---
layout: post
title: "Puppet Melhores Práticas"
date: 2013-04-22 13:35
comments: true
toc: true
categories: puppet
---

## 1. Melhores Práticas

Neste post vou agrupar algumas dicas aprendidas no dia-a-dia em produção, são dicas que me ajudaram, espero que lhe seja útil também.

## 2. Organizando o site.pp

Evite deixar o arquivo site.pp lotado de código, use o recurso **import**, separe as configurações em arquivos para ficar mais organizado.

{% codeblock lang:puppet %}

# /etc/puppet/manifests/site.pp
  
# importando manifests em diretórios

import "nodes/*.pp"      # importando manifest com declaração de nodes
import "classes/*.pp"    # importando classes simples

# importando manifests

import "filebucket.pp"   # importando configurações do filebucket
import "providers.pp"    # importando configurações globais de providers
import "globals.pp"      # importando configuraçoes globais variadas
import "extdata.pp"      # importando configurações de extlookup

{% endcodeblock %}

Sempre que houver alguma alteração em um manifest importado no site.pp, será necessário reiniciar o puppetmaster ou apache (se estiver usando passenger).

### 1.1 Arquivo filebucket.pp:

Neste arquivo eu normalmente insiro configurações do filebucket.

{% codeblock lang:puppet %}
# Define the bucket
filebucket { 'main':
  server => puppet,
  path   => false,
  # Due to a known issue, path must be set to false for remote filebuckets.
}

# Specify it as the default target
File { backup => main }
{% endcodeblock %}

### 1.2 Arquivo providers.pp:

Neste arquivo normalmente insiro configurações acerca de providers.

{% codeblock lang:puppet %}
case $operatingsystem {
    'Debian': {
        Package { provider => 'aptitude' }
    }
    'RedHat': {
        case $operatingsystemrelease {
            '4'    : {
                Package { provider => 'up2date' }
            }
        }
    }
}
{% endcodeblock %}

### 1.3 Arquivo extdata.pp:

Neste arquivo determino o escopo da busca para external data.

{% codeblock lang:puppet %}
$extlookup_datadir = "/etc/puppet/manifests/extdata"
$extlookup_precedence = ["host/%{fqdn}", "host/%{hostname}", "domain/%{domain}", "env/%{environment}", "common"]
{% endcodeblock %}

Apesar do recurso extlookup não ser muito utilizado depois da criação do HIERA,
ainda tenho módulos que utilizam esse tipo de configuração.

### 1.4 Arquivo globals.pp:

Neste arquivo coloco configurações globais, normalmente variáveis
que possam ser de interesse de vários módulos.

{% codeblock lang:puppet %}
$dnssearch = [ 'dominio', 'dominio.com.br', ]
$nameservers = [ '172.16.1.1', ]
$zabbixserver = '192.168.20.134'
$baculaserver = '192.168.20.135'
{% endcodeblock %}

Lembre-se de declarar corretamente as variáveis de topo de escopo, veja o exemplo abaixo:

    $::dnsseearch
    $::nameservers
    $::zabbixserver
    $::baculaserver

### 1.5 Diretório classes

Neste diretório costumo colocar classes simples, normalmente classes organizacionais que contém outras classes, ou mesmo classes **base** como a classe linux-server que é carregada em todos os nodes linux sem exceção.

Exemplo de manifest classes/linux-server.pp (classe organizacional):

{% codeblock lang:puppet %}
class linux-server {
 
        include hosts             # configura arquivo hosts
        include mailaliases       # configura aliases de correio local
        include utils             # instala pacotes dos sysadmins
        include aptrepos          # configura repositórios apt
        include rcfiles           # empurra screenrc e bashrc do root
        include vimeditor         # instala e configura vim e vimrc
        include ntp               # instala e cofigura ntp server
        include locale            # configura locale
        include timezone          # configure timezone
        include users             # cria usuários dos sysadmins
        include suoders           # configura sudoers
        include ssh:server        # instala e configura ssh server
        include snmp::server      # instala e configura snmp server
        include zabbix::agent     # instala e configura zabbix agent
        include puppet::agent     # configura puppet agent
        include bacula::agent     # instala e configura bacula agent
        include postfix::local    # instala e configura postfix local
}
{% endcodeblock %}

Exemplo de manifest classes/utils.pp (classe simples):

{% codeblock lang:puppet %}
class utils { 
 
  # declarando que o tzdata tem que estar sempre em sua ultima versao
 
  package { 'tzdata': ensure => latest }
 
  # declarando que os seguintes pacotes devem estar presentes (sysadmin packages)
 
  package { [ 'screen',
    'lynx',
    'elinks',
    'rsync',
    'telnet',
    'ftp',
    'wget',
    'bzip2',
    'unzip',
    'traceroute',
    'tcpdump',
    'iptraf',
    'ccze',
    'htop',
    'less',
    'dnsutils',
    'nmap', ]:
    ensure  => installed,
  }
}
{% endcodeblock %} 

Considero uma classe simples algo que tem um manifest e no máximo um arquivo estático ou dinâmico (template), qualquer coisa maior recomendo construir um módulo, fica mais organizado.

Minhas classes simples utilizam os diretórios abaixo:

    /etc/puppet/files      # arquivos estáticos
    /etc/puppet/templates  # arquivos dinâmicos

### 1.6 Diretório nodes

No diretório nodes costumo criar manifests separados para agrupar declarações de nodes em um mesmo contexto, fica mais organizado, 
fácil de entender e administrar.

Exemplo de manifest nodes/apache.pp:    

{% codeblock lang:puppet %}
node "apache01.dominio" {
   include apache2
   include php5
   include aplicacao
}
node "apache02.dominio" {
   include apache2
   include php5
   include aplicacao
}
node "apache03.dominio" {
   include apache2
   include php5
   include aplicacao
}
{% endcodeblock %} 

Exemplo de manifest nodes/haproxy.pp:

{% codeblock lang:puppet %}
node "haproxy01.dominio" {
   include haproxy
   include haproxy::httpmode::phppool
   include haproxy::httmode::phppool
}
node "haproxy02.dominio" {
   include haproxy
   include haproxy::tcpmode::imap
   include haproxy::tcpmode::smtp
}
{% endcodeblock %} 

## 2 Puppet.conf

Algumas configurações importantes no puppet.conf dos agentes

### 2.1 runinterval

O parâmetro **runinterval** define o intervalo entre as checagens que o agente faz ao servidor puppet, neste exemplo ele fará a checagem a cada 3600 segundos ou 60 minutos.

    runinterval=3600

### 2.2 splay & splaylimit

Podemos utilizar o splay para evitar que todos os agentes solicitem o catálogo ao mesmo tempo.

Com esse recurso ativo, o agente vai entrar em **sleep** quando chegar no momento de consultar o servidor. O agente será acionado dentro da
janela **splaylimit** de forma **randômica**, isto é fundamental para aliviar
a carga no puppetmaster em um cenário com muitos nodes.

Veja um exemplo de configuração de splay:

    splay=true
    splaylimit=1800
    runinterval=3600

No exemplo acima o **splay** está ligado e o limit está definido para 1800, logo ele vai trabalhar em uma janela de 1800 segundos após o tempo do **runinterval** ter se esgotado, com isto,
o tempo máximo entre checagens poder checar até 5400 segundos ou 90 minutos, lembrando que isso é o tempo máximo uma vez que os acionamentos são randômicos.

## 3. Módulos

### 3.1 Pense em bibliotecas

Pense em módulos como se fossem bibliotecas, quando for construir um módulo tente ao máximo construí-lo de forma que se alguém colocar este módulo no diretório /etc/puppet/modules tudo funcione de primeira, sem necessidade de alteração de código, permitindo
que alguma funcionalidade nova seja agregada ao seu ambiente
de forma simples e direta.

### 3.2 Carregamento do módulo

Construa módulos que tenham capacidade de serem lidos pelo auto-loader, isto significa que eles devem ter a estrutura recomendada
pela puppetlabs.

Módulo que não é carregado pelo auto-loader (jeito antigo):

    # /etc/puppet/modules/modulename/init.pp
    import "classes/*.pp"
    import "definitions/*.pp"

Módulo que é carregado pelo auto-loader (jeito certo):

    # /etc/puppet/modules/modulename/init.pp
    class modulename {}

Se você construir um módulo incompatível com o auto-loader, você precisa fazer o import no arquivo site.pp

    import modulename

Se não fizer isso, o módulo não será carregado, por isto, procure seguir sempre as recomendações e crie módulos compatíveis com o auto-loader.

### 3.3 Namespace

Ao criar classes em módulos siga a nomenclatura abaixo:

    nome_do_modulo
    nome_do_modulo::nome_da_classe
    nome_do_modulo::nome_do_diretorio::nome_da_classe

Veja o exemplo do módulo apache:

{% codeblock lang:puppet %}
  # /etc/puppetlabs/puppet/modules/apache/manifests
    # init.pp
      class apache { }
    # ssl.pp
      class apache::ssl { }
    # vhost.pp
      define apache::vhost () { }
    # params/confs.pp
      class apache::params::confs { }
{% endcodeblock %} 

Essa estrutura em escopo torna o nome das classes do módulo única, diferenciando dos demais módulos, evitando conflitos e problemas de carregamento, siga sempre esse modelo.

### 3.4 Forge e Github

Não comece do zero, pesquise e reutilize código existente no Puppet Forge e Github.

### 3.5 Documentação

Escreva um README detalhado, não deixe de preencher os metadados do seu módulo, crie a documentação de suas classes e definições usando
RDoc Markup.

Use os comandos **puppet module** e **puppet doc** para lhe ajudar com a documentação.

## 4. Estilo de código

Acesse a página [style guide](http://docs.puppetlabs.com/guides/style_guide.html), ou o meu post [guia de estilos](http://gutocarvalho.net/octopress/2013/04/20/puppet-style-guide) para aprender a escrever seus manifests de forma mais eficiente.

### 4.1 Puppet parser validate

Para validar a sintaxe de código em um manifest você pode utilizar o comando puppet parser, veja abaixo o exemplo:

    puppet parser validate manifest.pp

### 4.2 Puppet-lint

Use o [puppet lint](http://puppet-lint.com) para verificar se o seu manifest está com a estrutura de acordo com o [style guide](http://docs.puppetlabs.com/guides/style_guide.html) do Puppet.

## 5. ENC (External node classifier)

Instale um ENC em seu ambiente, isto te permite acompanhar o ciclo de vida de seus node, e além disto, oferece controles para associar
classes parametrizadas a nodes ou grupos de nodes.

Os ENCs oferecem visualizações de status do seus nodes, é possível visualizar quando ocorreu uma mudança, é possível visualizar erros
decorrentes de mudanças, é possível extrair dados (fatos) dos nodes. 

Os ENCs mais conhecidos são Puppet Dashboard e Foreman, o que possui
mais recursos é o Foreman.

## 6. Repositório Puppetlabs

Recomendo a utilização de pacotes do repositório oficial da Puppetlabs. Eles oferecem repositórios para Debian, Ubuntu, RHEL, Centos e derivados EL (Enterprise Linux).

Repositório APT

    apt.puppetlabs.com

Repositório YUM
 
    yum.puppetlabs.com

## 7. Passenger

O pacote puppetmaster por default usa um servidor web embutido chamado  'webrick', este servidor web deve ser utilizado apenas para fins de desenvolvimento e testes, ele não suportará mais do que 25
conexões simultâneas, e se suportar irá alocar grande quantidade
de recursos de seu servidor, gerando extrema lentidão e alto
load. Em resumo o puppetmaster usando webrick não escala.

Em ambientes de produção é recomendando utilizar o pacote puppetmaster-passenger que já integra o puppetmaster ao Apache2
com mod-passenger, com isto, sua capacidade e robustez serão equivalentes a capacidade do apache2, este por sua vez é um servidor web reconhecidamente eficaz, rápido e escalável.

O passenger pode ser utilizado inclusive para rodar o puppet-dashboard.

## 8. Orquestrador e MQ (Message Queue)

É importante ter um orquestrador funcionando, eventualmente pode surgir a necessidade de rodar algum comando de forma paralela e
em tempo real em seus nodes, e você não pode ficar na mão nesse
momento.

O orquestrador que recomendo é o Marionette Collective (Mcollective ou MCO).

O Mcollective depende de um sistema MQ para funcionar, recomendo
o ActiveMQ. É possível utilizar o RabbitMQ, porém o criador do MCO
não é muito fã desse projeto, e algumas features do MCO só vão
funcionar com ActiveMQ.

Veja o post [Intalando o Mcollective em Debian Squeeze](http://gutocarvalho.net/octopress/2013/02/07/instalando-mcollective-em-debian-squeeze-com-ambiente-puppet/) para aprender a instalar e configurar este orquestrador.

## 9. Manuteção em Puppet Dashboard

Se você está utilizando o ENC Puppet Dashboard, tome cuidado como crescimento do banco, se você não configurar o MYSQL corretamente
ou não levar em conta espaço em disco, pode ter alguma surpresa
depois de alguns meses, além da lentidão, um disk full pode para
seu ENC e será trabalhoso resolver o problema.

A puppetlabs oferece um [manual](http://docs.puppetlabs.com/dashboard/manual/1.2/maintaining.html) com dados e comandos para fazer manutenção na base do dashboard, isto é muito útil e pode
te livrar de um grande problema.

### 9.1 Otimização da base

Para otimizar a sua base, basta rodar o comando abaixo na VM do ENC dentro do diretório do dashboard.

    sudo -u puppet-dashboard rake RAILS_ENV=production db:raw:optimize

### 9.2 Limpeza da base

Para limpar a base e manter os dados dos últimos 30 dias, basta rodar o comando abaixo no diretório do dashboard.

    sudo -u puppet-dashboard rake RAILS_ENV=production reports:prune upto=1 unit=mon

### 9.3 Limpeza agendada

Para inserir uma rotina de limpeza mensal do banco no cron, use o comando abaixo dentro do diretório do dashboard.  

    sudo -u puppet-dashboard rake cron:cleanup

## 10. Ambiente Puppet

Dicas gerais:

* Se possível instale o PuppetMaster em uma VM dedicada.
* Se possível instale o ENC em uma VM dedicada.
* Se possível separe o banco do ENC em uma VM dedicada.
* Se possível instale o Orquestrador + MQ em uma VM dedicada.

Não vejo necessidade de rodar o Puppet, ENC ou Orquestrador em máquinas físicas, seu consumo de I/O e recursos não justifica isto, já o banco do ENC é interessante rodar em uma máquina física ligada
a um STORAGE com uma LUN em um RAID GROUP com discos rápidos.

### 10.1 Características

#### 10.1.1 PuppetMaster

Para atender até 1000 nodes.

* 4 Processadores
* 6 Gigas de RAM
* BARRA com 30 GB
* /VAR  com 120 GB (no caso de ter storeconfig ligado)
* Distribuição 64 Bits
* Recomendo Debian

#### 10.1.2 ENC (app web sem banco)

Para atender até 1000 nodes.

* 2 Processadores
* 4 Gigas de RAM
* BARRA com 30 GB
* /VAR  com 60 GB
* Distribuição 64 Bits
* Recomendo Debian

ENCs: Puppet Dashboard ou Foreman

Dá para colocar o PuppetDB na mesma máquina - se for o caso.

#### 10.1.3 ENC BANCO

Para armazenar dados de até 1000 nodes.

* 4 Processadores
* 6 Gigas de RAM
* BARRA com 30 GB
* /VAR  com 300 GB
* Distribuição 64 Bits
* Recomendo Debian

#### 10.1.4 ORQUESTRADOR e MQ

Para orquestrar até 1000 nodes.

* 2 Processadores
* 4 Gigas de RAM
* BARRA com 30 GB
* /VAR  com 30 GB
* Distribuição 64 Bits
* Recomendo Debian

ActiveMQ/RabbitMQ, Mcollective Client/Server.

[s]<br>
Guto
