---
layout: post
title: "Instalando Mcollective em Debian Squeeze com Ambiente Puppet"
date: 2013-02-07 11:06
comments: true
toc: true
categories: mcollective
---
## 1. Sobre o Marionette Collective

O Mcollective é um framework que te permite trabalhar com orquestração de servidores ou execução paralela de tarefas em seus nodes.

Em essência, orquestração significa invocar ações em paralelo em todo o seu parque de servidores de uma forma direta, eficiente e além disto em tempo real.

## 2. Entendendo a necessidade

Vamos supor que por alguma razão você precise rodar algum comando em todos os seus servidores, seja para atualizar um pacote, seja para desligar um serviço, seja para remover um usuário ou aplicar uma regra e firewall, enfim, por qualquer que seja a razão. 

Sabemos que o puppet permite esse tipo de abordagem, porém não te permite fazer isto em tempo real, com o puppet você precisar criar um manifest, declarar a classe de seu manifest para seus nodes e esperar que os agentes consultem o servidor, recebam o catálogo e iniciem a aplicação das mudanças declaradas no manifest. Isto pode levar minutos ou horas dependendo do tamanho do seu parque e de como os agentes estão configurados.

Se sua necessidade é executar algo em tempo real, de forma paralela, o mcollective é a ferramenta certa para fazer isto em seu parque de servidores.

## 3. Objetivos deste artigo

Aqui pretendo abordar a instalação do Mcollective em ambiente Debian.

## 4. Ambiente

2 VMs' Virtualbox - Debian Squeeze - 64 Bits

    vm1: 192.168.56.50 puppet.hacklab (puppet master)
    vm2: 192.168.56.51 debian.hacklab (puppet agent )

## 5 Pre-Reqs

### 5.1 Repositório debian

Nas duas VM's adicione os repositórios abaixo ao seu sources.list

Edite o arquivo

    vim /etc/apt/sources.list

Adicione as linhas ao final do arquivo

    deb http://apt.puppetlabs.com squeeze main
    deb http://www.rabbitmq.com/debian/ testing main

Salve as alterações

	:wq

Atualize os índices
	
	aptitude update
	
Pronto, agora você poderá instalar o mcollective usando o aptitude.

### 5.2 Ambiente Puppet

Nas duas VM's é necessário que já exista um ambiente Puppet instalado e rodando em modo cliente/servidor, não vou entrar em detalhes pois já cobri esse processo de forma completa em minha [wiki](http://gutocarvalho.net/dokuwiki/doku.php/puppetmaster).

    vm1: puppet.hacklab (puppet master)
    vm2: debian.hacklab (puppet agent )

Ressalto que é importante ter o puppet funcionando nas VM's para podermos utilizar o agente puppet, package e service do Mcollective.

## 6. Configurando VM1 (puppet master)

### 6.1 Instalando RabbitMQ

#### 6.1.1 Sobre o RabbitMQ

O Mcollective (MCO) depende de algum sistema de mensageria, é possível utilizar tanto o ApacheMQ quanto o RabbitMQ, ambos tem suporte ao protocolo STOMP, necessário para o funcionamento do MCO.

Neste estudo vou utilizar o RabbitMQ, mas é possível - e até recomendado - utilizar o ActiveMQ.

#### 6.1.2 Instalando o ActiveMQ

Instale o rabbitmq 3.0.2 (vigente neste momento)

    aptitude install rabbitmq-server

Edite o arquivo /etc/rabbitmq/rabbitmq.config e coloque apenas o conteúdo abaixo, limpe o resto

```
[
  {rabbitmq_stomp, [{tcp_listeners, [{"0.0.0.0", 6163},
                                     {"::1",       6163}]}]}
].
```   

Edite o arquivo /etc/default/rabbitmq-server e descomente a linha abaixo

    ulimit -n 1024

reinicie o serviço

    /etc/init.d/rabbitmq-server restart

Verfique o status do serviço
            
    /etc/init.d/rabbitmq-server status

Acompanhe a saída

```
Status of node rabbit@puppet ...
[{pid,7390},
 {running_applications,[{rabbit,"RabbitMQ","3.0.2"},
                        {mnesia,"MNESIA  CXC 138 12","4.4.14"},
                        {os_mon,"CPO  CXC 138 46","2.2.5"},
                        {sasl,"SASL  CXC 138 11","2.1.9.2"},
                        {stdlib,"ERTS  CXC 138 10","1.17"},
                        {kernel,"ERTS  CXC 138 10","2.14"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R14A (erts-5.8) [source] [rq:1] [async-threads:30] [kernel-poll:true]\n"},
 {memory,[{total,12888616},
          {connection_procs,1344},
          {queue_procs,2688},
          {plugins,0},
          {other_proc,4592340},
          {mnesia,28704},
          {mgmt_db,0},
          {msg_index,7020},
          {other_ets,363324},
          {binary,3680},
          {code,6394787},
          {atom,796661},
          {other_system,698068}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,157546905},
 {disk_free_limit,1000000000},
 {disk_free,2638680064},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,121}]},
 {run_queue,0},
 {uptime,5}]
...done.
```

Crie um usuário para o mcollective

    rabbitmqctl add_user mcollective senhapuppet

Especifique as permissões para o usuário do mcollective

    rabbitmqctl set_permissions -p / mcollective "^amq.gen-.*" ".*" ".*"

Remova o usuário guest

    rabbitmqctl delete_user guest
    
Habilite os plugins stomp e amqp

    rabbitmq-plugins enable rabbitmq_stomp
    rabbitmq-plugins enable amqp_client

### 6.2 Instalando MCollective

Instale a biblioteca stomp para ruby

    aptitude install libstomp-ruby

Instale os pacotes mcollective da puppetlabs

    aptitude install mcollective mcollective-client

Configure o arquivo /etc/mcollective/server.cfg conforme abaixo

```
topicprefix = /topic/
main_collective = mcollective
collectives = mcollective
libdir = /usr/share/mcollective/plugins
logfile = /var/log/mcollective.log
loglevel = info
daemonize = 1

# Plugins
securityprovider = psk
plugin.psk = hacklab

connector = stomp
plugin.stomp.host = 192.168.56.50
plugin.stomp.port = 6163
plugin.stomp.user = mcollective
plugin.stomp.password = senhapuppet

# Facts
factsource = yaml
plugin.yaml = /etc/mcollective/facts.yaml

```

Configure o arquivo /etc/mcollective/client.cfg conforme abaixo

```
topicprefix = /topic/
main_collective = mcollective
collectives = mcollective
libdir = /usr/share/mcollective/plugins
logger_type = console
loglevel = warn

# Plugins
securityprovider = psk
plugin.psk = hacklab

connector = stomp
plugin.stomp.host = 192.168.56.50
plugin.stomp.port = 6163
plugin.stomp.user = mcollective
plugin.stomp.password = senhapuppet

# Facts
factsource = yaml
plugin.yaml = /etc/mcollective/facts.yaml
```
Reinicie o mcollective

    /etc/init.d/mcollective restart

Faça o teste de ping em nodes ativos usando o mco

    mco ping

Acompanhe a saída

    puppet.hacklab                           time=89.34 ms
    ---- ping statistics ----
    1 replies max: 89.34 min: 89.34 avg: 89.34

Faça o teste de find em nodes ativos usando o mco

    mco find

Acompanhe a saída

    puppet.hacklab
    
Podemos ver que o MCO está funcionando corretamente, por hora detectando apenas um node.

## 7. Configurando VM2 (puppet agent)

Para instalar o Mcollective na VM2, siga as referências da seção 1.2 e tenha os repositórios devidamente configurados.

Após a instalação e a configuração do mcollective, faça o teste de ping em nodes ativos com mco

    mco ping

Acompanhe a saída

```
debian.hacklab                           time=188.49 ms
puppet.hacklab                           time=191.91 ms
---- ping statistics ----
2 replies max: 191.91 min: 188.49 avg: 190.20 
```

Faça uma busca por nodes ativos com o mco

    mco find

Acompanhe a saída

```
puppet.hacklab
debian.hacklab
```

Solicite o inventário de uma máquina

    mco inventory debian.hacklab
    
Acompanhe a saída

```
Inventory for debian.hacklab:

   Server Statistics:
                      Version: 2.2.2
                   Start Time: Wed Feb 06 14:39:55 -0200 2013
                  Config File: /etc/mcollective/server.cfg
                  Collectives: mcollective
              Main Collective: mcollective
                   Process ID: 1744
               Total Messages: 19
      Messages Passed Filters: 18
            Messages Filtered: 1
             Expired Messages: 0
                 Replies Sent: 17
         Total Processor Time: 0.52 seconds
                  System Time: 0.27 seconds

   Agents:
      discovery       puppet          rpcutil        

   Data Plugins:
      agent           fstat           puppet         
      resource                                       

   Configuration Management Classes:
      apt                            apt::params                   
      debian.hacklab                 hostname                      
      hosts                          linux-server                  
      openssh-server                 puppet-agent                  
      repos                          resolv                        
      salt-minion                    settings                      
      usuarios                       utils                         

   Facts:
      mcollective => 1
```

Podemos ver que o MCO está funcionando corretamente e detectando os dois nodes.

## 8. Agentes para Mcollective

O Mcollective possui vários agentes que nos ajudam a manipular os nodes de nosso parque de forma direta, aqui vou abordar a instalação e utilização de alguns destes agentes.

Instale os pacotes em ambos os nodes

    aptitude install mcollective-puppet-agent mcollective-puppet-client mcollective-package-agent mcollective-package-client mcollective-service-agent mcollective-service-client

Reinicie o mcollective em ambos os nodes

    /etc/init.d/mcollective restart

### 8.1 Agente puppet

#### 8.1.1 runonce/runall

Após instalar, podemos mandar executar o puppet em um node

    mco puppet runonce -I debian.hacklab
   
Acompanhe a saída 
    
```     
* [ ============================================================> ] 1 / 1

Finished processing 1 / 1 hosts in 112.56 ms
```

Podemos mandar executar o agente em todos os nodes

    mco puppet runonce -v

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

puppet.hacklab                          : Puppet is currently applying a catalog, cannot run now
    {:summary=>"Puppet is currently applying a catalog, cannot run now"}

debian.hacklab                          : OK
    {:summary=>"Signalled the running Puppet Daemon"}

---- rpc stats ----
           Nodes: 2 / 2
     Pass / Fail: 1 / 1
      Start Time: Wed Feb 06 18:08:20 -0200 2013
  Discovery Time: 2013.47ms
      Agent Time: 43.72ms
      Total Time: 2057.18ms
```
    
Existe ainda o parâmetro **runall** onde você pode especificar a quantidade de ações simultâneas que serão executadas, ou seja, quantidade de agentes que serão executados de forma paralela.

    mco puppet runall 10
    
Neste caso ele vai executar o agente em todos os nodes, rodando de 10 em 10, essa pode ser um estratégia para evitar esgotamento dos recursos do servidor em parques muito grandes, desta forma, ao invés de deixar o agente ligado e consultando o servidor a cada N minutos, você pode usar o MCO para rodar o agente de forma controlada em todos os seus nodes.
 
#### 8.1.2 enable/disable 
    
Desabilitando puppet em um node
    
    mco puppet disable -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

Summary of Enabled:

   disabled = 1

Finished processing 1 / 1 hosts in 126.27 ms
```

Habilitando puppet em um node

    mco puppet enable -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

Summary of Enabled:

   enabled = 1

Finished processing 1 / 1 hosts in 91.71 ms    
```    
 
Desabilitando puppet em todos os nodes

    mco rpc puppet disable message="doing some hand hacking"

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Enabled:

   disabled = 2

Finished processing 2 / 2 hosts in 201.09 ms
```

Habilitando puppet em todos os nos nodes

    mco rpc puppet enable

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Enabled:

   enabled = 2

Finished processing 2 / 2 hosts in 45.02 ms
```

#### 8.1.3 summary

Solicitando o relatório last run

    mco puppet summary

Acompanhe a saída

```
Summary statistics for 2 nodes:

                  Total resources: ▇▁▁▇▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁  min: 62.0   max: 66.0  
            Out Of Sync resources: ▇▇▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁  min: 0.0    max: 2.0   
                 Failed resources: ▇▇▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁  min: 0.0    max: 2.0   
                Changed resources: ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁  min: 0.0    max: 0.0   
  Config Retrieval time (seconds): ▇▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁  min: 0.7    max: 1.5   
         Total run-time (seconds): ▇▁▁▁▁▇▁▁▁▁▁▁▁▁▁▁▁▁▁▁  min: 4.4    max: 12.5  
    Time since last run (seconds): ▇▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▇  min: 791.0  max: 1.5k  
```

#### 8.1.4 count
 
Solicitando contagem dos nodes habilitados, desabiliados e em execução 
   
    mco puppet count

Acompanhe a saída

```
Total Puppet nodes: 2

          Nodes currently enabled: 2
         Nodes currently disabled: 0

Nodes currently doing puppet runs: 0
          Nodes currently stopped: 2

       Nodes with daemons started: 2
    Nodes without daemons started: 0
       Daemons started but idling: 2
```

#### 8.1.5 stats

Solicitando status dos agentes

    mco puppet status
    
Acompanhe a saída

```
 * [ ============================================================> ] 2 / 2

   debian.hacklab: Currently idling; last completed run 30 minutes 34 seconds ago
   puppet.hacklab: Currently applying a catalog; last completed run 6 hours 37 minutes 18 seconds ago

Summary of Applying:

    true = 1
   false = 1

Summary of Daemon Running:

   running = 2

Summary of Enabled:

   enabled = 2

Summary of Idling:

    true = 1
   false = 1

Summary of Status:

               idling = 1
   applying a catalog = 1


Finished processing 2 / 2 hosts in 67.61 ms
```

### 8.2 Agente service

#### 8.2.1 agente service: stop

Parando um serviço em um node específico

    mco rpc service stop service=ssh -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

Summary of Service Status:

   stopped = 1

Finished processing 1 / 1 hosts in 1166.06 ms
```
Parando um serviço em todos os nodes

    puppet@root ~]# mco rpc service stop service=ssh

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Service Status:

   stopped = 2

Finished processing 2 / 2 hosts in 2546.92 ms
```

#### 8.2.2 plugin service: start

Iniciando o serviço em um node específico

    mco rpc service start service=ssh -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

Summary of Service Status:

   running = 1

Finished processing 1 / 1 hosts in 1204.95 ms
```

Iniciando um serviço em todos os nodes

    mco rpc service start service=ssh

Acompanhe a saída


```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Service Status:

   running = 2

Finished processing 2 / 2 hosts in 3271.52 ms
```

#### 8.2.3 plugin service: restart

Reiniciando um serviço em um node específico

    mco rpc service restart service=ssh -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1


Summary of Service Status:

   running = 1

Finished processing 1 / 1 hosts in 1015.66 ms
```

Reiniciando um serviço em todos os nodes

    mco rpc service restart service=ssh

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Service Status:

   running = 2

Finished processing 2 / 2 hosts in 5466.57 ms
```

### 8.3 Agente package

#### 8.3.1 agente package: install

Instalando o pacote em um node específico

    mco rpc package install package=htop -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

debian.hacklab                           Unknown Request Status
   Package is already installed

Summary of Ensure:

   0.8.3-1 = 1

Finished processing 1 / 1 hosts in 856.85 ms
```
Instalando um pacote em todos os nodes
   
    mco rpc package install package=htop

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Ensure:

   0.8.3-1 = 2

Finished processing 2 / 2 hosts in 4972.04 ms
```

#### 8.3.2 plugin package: uninstall

Removendo um pacote de um node específico

    mco rpc package uninstall package=htop -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

Summary of Ensure:

   absent = 1

Finished processing 1 / 1 hosts in 7515.24 ms
```

Removendo um pacote em todos os nodes

    mco rpc package uninstall package=htop

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Ensure:

   absent = 2

Finished processing 2 / 2 hosts in 4230.82 ms
```

#### 8.3.3 plugin package: status

Verificando o status de um pacote em um node específico

    mco rpc package status package=htop -I debian.hacklab

Acompanhe a saída

```
 * [ ============================================================> ] 1 / 1

debian.hacklab                           
       Arch: nil
    desired: install
     Ensure: 0.8.3-1
      Epoch: nil
      error: ok
       Name: htop
     Output: nil
   Provider: :apt
    Release: nil
     status: installed
    Version: nil

Summary of Arch:

     No aggregate summary could be computed

Summary of Ensure:

   0.8.3-1 = 1

Finished processing 1 / 1 hosts in 391.55 ms
```

Verificando o status de um pacote em todos os nodes

    mco rpc package status package=htop

Acompanhe a saída
  
```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

debian.hacklab                           
       Arch: nil
    desired: install
     Ensure: 0.8.3-1
      Epoch: nil
      error: ok
       Name: htop
     Output: nil
   Provider: :apt
    Release: nil
     status: installed
    Version: nil

puppet.hacklab                           
       Arch: nil
    desired: install
     Ensure: 0.8.3-1
      Epoch: nil
      error: ok
       Name: htop
     Output: nil
   Provider: :apt
    Release: nil
     status: installed
    Version: nil

Summary of Arch:

     No aggregate summary could be computed

Summary of Ensure:

   0.8.3-1 = 2

Finished processing 2 / 2 hosts in 174.41 ms
```

#### 8.3.4 plugin package: update

Atualizando um pacote em um node específico

    mco rpc package update package=facter -I debian.hacklab

Acompanhe a saída

```

 * [ ============================================================> ] 1 / 1

Summary of Ensure:

   1.6.17-1puppetlabs1 = 1


Finished processing 1 / 1 hosts in 11307.56 ms
```

Atualizando um pacote em todos os nodes

    mco rpc package update package=htop

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

Summary of Ensure:

   0.8.3-1 = 2

Finished processing 2 / 2 hosts in 2999.96 ms
```

### 8.4 Agente facter facts

O Mcollective possui suporte a mostrar certos fatos do sistema, mas se quisermos utilizar os fatos do facter, precisamos fazer alguns ajustes na configuração e adicionar um plugin manualmente, isto é necessário pois infelizmente o plugin **facter facts** não está empacotado para o debian.

Acesse do diretório tmp

    cd /tmp

Faça o clone do repo mcollective-plugins

    git clone https://github.com/puppetlabs/mcollective-plugins.git

Copie o plugin para o diretório do mcollective

    cp mcollective-plugins/facts/facter/facter_facts.rb /usr/share/mcollective/plugins/mcollective/facts/

Ajuste o arquivo client.cfg e server.cfg do mcollective, remova ou comente o trecho abaixo

```
# Facts
factsource = yaml
plugin.yaml = /etc/mcollective/facts.yaml
```

E insira o seguinte no lugar

```
# Facts
factsource = facter
```

Após alterar, reinicie o mcollective 

    /etc/init.d/mcollective restart
    
Consulte o fato virtual em um node específico

    mco facts virtual  -v -I debian.hacklab

Acompanhe a saída

```
Report for fact: virtual

        virtualbox                              found 1 times

            debian.hacklab


---- rpc stats ----
           Nodes: 1 / 1
     Pass / Fail: 1 / 0
      Start Time: Wed Feb 06 20:57:39 -0200 2013
  Discovery Time: 0.00ms
      Agent Time: 136.37ms
      Total Time: 136.37ms    
```

Consulte o fato ipaddress do facter em todos os nodes

    mco facts ipaddress -v

Acompanhe a saída

```
Discovering hosts using the mc method for 2 second(s) .... 2
Report for fact: ipaddress

        192.168.56.50                           found 1 times

            puppet.hacklab

        192.168.56.51                           found 1 times

            debian.hacklab

---- rpc stats ----
           Nodes: 2 / 2
     Pass / Fail: 2 / 0
      Start Time: Wed Feb 06 20:52:29 -0200 2013
  Discovery Time: 2007.25ms
      Agent Time: 54.72ms
      Total Time: 2061.97ms
```
Consulte o fato kernelversion do facter em todos os nodes

    mco facts kernelversion -v

Acompanhe a saída:

```
Discovering hosts using the mc method for 2 second(s) .... 2
Report for fact: kernelversion

        2.6.32                                  found 2 times

            debian.hacklab
            puppet.hacklab


---- rpc stats ----
           Nodes: 2 / 2
     Pass / Fail: 2 / 0
      Start Time: Wed Feb 06 20:52:47 -0200 2013
  Discovery Time: 2008.67ms
      Agent Time: 59.92ms
      Total Time: 2068.59ms
```

Podemos ver que o MCO está conseguindo trazer os fatos do facter corretamente.

## 9. Amarrando as pontas

Essa foi uma instalação básica do Mcollective em ambiente debian - usando RabbitMQ, além disto, apresentei alguns comandos iniciais usando o agente/client puppet criado pela Puppetlabs para o MCO.

Mas não pense que é só isto, ele é muito mais poderoso, existem diversos outros agentes para utilizar e você pode construir seus próprios agentes usando RPC se achar necessário, e ainda existem os plugins em ruby, ele é muito extensível.

Vale comentar que apesar de ter utilizado o RabbitMQ, o criador do MCO recomenda o uso do ActiveMQ a partir da versão 2.x.x, aparentemente alguns dos recursos desta versão só irão funcionar com o ActiveMQ. Em breve espero tratar o uso do ActiveMQ também.

Preciso explicar que esse post apenas agrupa o que eu estudei até o momento, a partir daqui é hora de aprofundar ainda mais.

## 10. Referências


**Plugins Github** <br>

* https://github.com/puppetlabs/mcollective-plugins
* https://github.com/puppetlabs/mcollective-puppet-agent#readme

**Introdução ao MCO** <br>

* https://puppetlabs.com/mcollective/introduction/
* http://docs.puppetlabs.com/mcollective/reference/basic/gettingstarted_debian.html
* http://docs.puppetlabs.com/mcollective/reference/basic/gettingstarted.html
* http://docs.puppetlabs.com/mcollective/reference/basic/configuration.html

**Construindo Agentes e Clientes com SimpleRPC** <br>

* http://docs.puppetlabs.com/mcollective/simplerpc/

**Slides MCO** <br>

* http://www.slideshare.net/PuppetLabs/presentation-16281121

**RabbitMQ** <br>

* http://www.rabbitmq.com/stomp.html

O livro **Pro Puppet** da Puppetlabs também foi uma ótima referência de estudo.

[s]<br>
Guto
