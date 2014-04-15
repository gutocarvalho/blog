---
layout: post
title: "Mcollective Shell Agent"
date: 2013-03-09 18:51
comments: true
toc: true
categories: mcollective
---

## 1. Sobre o Marionette Collective

O Mcollective é um framework que te permite trabalhar com orquestração de servidores ou execução paralela de tarefas em seus nodes.

Em essência, orquestração significa invocar ações em paralelo em todo o seu parque de servidores de uma forma direta, eficiente e além disto em tempo real.

## 2. Agente Shell

No post anterior, apresentei alguns agentes do Mcollective que executam ações pontuais, no entanto, não é possível através destes agentes rodar comandos arbitários em nossos nodes.

O próprio criador do Mcollective ([R.I.Pienaar](http://www.devco.net/)) não recomenda que sejam escritos plugins para execução de comandos arbitrários, e ele tem razão, é perigoso e você não conseguirá prever a saída do comando no outro lado.

Porém, eventualmente precisamos fazer isto, e neste caso existem alguns plugins disponíveis no GITHUB, aqui neste post vou mostrar como instalar um plugin que contém um agente para execução de comandos em ambiente shell.

### 2.1 Instalando o agente

Partindo do pressuposto de que você já tem o Mcollective rodando, vamos apenas instalar o agente shell e ver como ele funciona. Aqui neste post vou usar o agente escrito por [Jeremy Caroll](https://github.com/phobos182/mcollective-plugins/).

Faça download do arquivo shell.rb

    wget https://raw.github.com/phobos182/mcollective-plugins/master/agent/shell/shell.rb

Faça download do arquivo shell.ddl

    wget https://raw.github.com/phobos182/mcollective-plugins/master/agent/shell/shell.ddl

Mova os arquivos para o diretório agent do mcollective

    mv shell.rb shell.ddl /usr/share/mcollective/plugins/mcollective/agent

Agora faça download do arquivo de aplicação shell.rb (é outro arquivo)

    wget https://raw.github.com/phobos182/mcollective-plugins/master/agent/shell/application/shell.rb

Mova o arquivo para o diretório application do mcollective

    mv shell.rb /usr/share/mcollective/plugins/mcollective/application
    
Agora reinicie o agente.

    /etc/init.d/mcollective restart
    
Instale os arquivos em todos os nodes que deseja utilizar o agente shell.

### 2.2 Utilizando o agente

A utilização é bastante simples, observe a sintaxe:

    mco shell <comando>

Vamos a um exemplo

    mco shell uptime

Acompanhe a saída

```
Do you really want to send this command unfiltered? (y/n): y
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

[puppet.hacklab] exit=0:
[debian.hacklab] exit=0:
```

Veja que o comando foi executado em dois nodes, mas repare que apenas fomos informados disto, caso deseje ver a saída, será necessário adicionar um parâmetro.

    mco shell uptime --verbose

Acompanhe a saída

```
Do you really want to send this command unfiltered? (y/n): y
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

[puppet.hacklab] exit=0:  13:56:48 up 28 min,  2 users,  load average: 0.00, 0.00, 0.00
[debian.hacklab] exit=0:  13:56:44 up 28 min,  2 users,  load average: 0.00, 0.00, 0.00
```

### 2.3 Exemplos de uso do agente


#### 2.3.1 Lendo arquivo /etc/issue

Vamos usar o cat para ler o conteúdo do arquivo issue

    mco shell "cat /etc/issue" --verbose
    
Acompanhe a saída

```
Do you really want to send this command unfiltered? (y/n): y
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

[debian.hacklab] exit=0: Debian GNU/Linux 6.0 \n \l

[puppet.hacklab] exit=0: Debian GNU/Linux 6.0 \n \l
```

#### 2.3.2 Rodando comando df

Vamos rodar o comando df -h para ver dados de uso das partições e disco.

    mco shell "df -h" --verbose

Acompanhe a saída

```
Do you really want to send this command unfiltered? (y/n): y
Discovering hosts using the mc method for 2 second(s) .... 2

 * [ ============================================================> ] 2 / 2

[puppet.hacklab] exit=0: Filesystem            Size  Used Avail Use% Mounted on
/dev/sda1             3.8G  2.1G  1.6G  57% /
tmpfs                 188M     0  188M   0% /lib/init/rw
udev                  184M  124K  184M   1% /dev
tmpfs                 188M     0  188M   0% /dev/shm

[debian.hacklab] exit=0: Filesystem            Size  Used Avail Use% Mounted on
/dev/sda1             3.8G 1002M  2.6G  28% /
tmpfs                 125M     0  125M   0% /lib/init/rw
udev                  121M  124K  120M   1% /dev
tmpfs                 125M     0  125M   0% /dev/shm
```

## 3. Amarrando as pontas

O criador do Mcollective ([R.I.Pienaar](http://www.devco.net/)) recomenda que você escreva seus agentes usando simpleRPC, agentes que executem tarefas pontais, agentes no quais você tem controle, agentes nos quais você pode confiar, agentes nos quais você conhece o risco de utilizar em sua infra, e tenho que dizer que eu concordo com isto, portanto, use esse agente ssh por sua conta e risco ;)

Eu assumo o risco pois o que preciso fazer com ele é bem simples, mas cada caso é um caso.

### 3.1 Exemplificando os riscos

Só para você ter uma ideia de tragédias possíveis, imagine o risco potencial de alguém rodar - sem querer - algo como abaixo:

    mco shell "rm -rf /"
    
Isto simplesmente apagaria todo o seu parque, é muito poder para pouca segurança. Espero que este exemplo tenha sido suficientemente claro :)

### 3.1 Facilitando a instalação

Eu escrevi dois módulos para Puppet que facilitam a instalação do Mcollective e dos agentes, segue abaixo os links.

[https://github.com/gutocarvalho/puppet-mcollective-debian](https://github.com/gutocarvalho/puppet-mcollective-debian)

[https://github.com/gutocarvalho/puppet-rabbitmq-debian](https://github.com/gutocarvalho/puppet-rabbitmq-debian)

O agente SSH já está incluso no módulo.

## 4. Referências

* http://www.devco.net
* https://github.com/phobos182/mcollective-plugins
* https://github.com/puppetlabs/mcollective-plugins
* https://puppetlabs.com/mcollective/introduction

[s]<br>
Guto
