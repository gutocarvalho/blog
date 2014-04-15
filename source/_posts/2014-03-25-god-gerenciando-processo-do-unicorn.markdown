---
layout: post
title: "God gerenciando processo do Unicorn"
date: 2014-03-25 18:24
comments: true
categories: puppet
---

## Necessidade

Lembrando do poeta Juvenal e suas sátiras, temos a famosa frase:

    Quis custodiet ipsos custodes?
    
Que traduzindo para pt_BR seria algo como:

    Quem guardará os guardiões?

Ou em outra interpretação livre:
	
    Quem vigiará os vigilantes?

Isto se aplica a nossos cenários com o Puppetmaster. O UNICORN é o webserver do Puppet, o puppet é o nosso gerenciador de configurações, é ele que cuida de todo o nosso ambiente e infraestrutura. Através do Puppet podemos inclusive gerenciar serviços, mantendo estes habilitados e rodando, contudo, quem vai gerenciar o serviço do próprio Puppet?

É aqui que entra o God, será através dele que vamos garantir que o processo do UNICORN esteja sempre rodando, e por consequência, o próprio serviço do Puppet.

## Sobre o GOD

O [GOD](http://godrb.com/) é um framework de monitoramento todo escrito em Ruby, o seu criador propõe que o gerenciamento de processos e tarefas deve ser um procedimento simples, principalmente quando estamos fazendo o deploy de nossa aplicação, e isto o motivou a criar o GOD. Seu objetivo é torná-lo uma alternativa simples e robusta para monitoramento de aplicações. 

É possível além de manipular e monitorar processos e tarefas, enviar notificações para diversos serviços detre eles e-mail, jabber, twitter e outros.

O God consegue monitorar processos e tarefas simples, e você pode criar grupos de processos ou tarefas para monitorar. Todas as configurações e tratamento de de condições são feitos em ruby.

## Instalação do God

Instale o god

    gem install god

Crie o diretório para o god

    mkdir /etc/god

Crie os arquivos de log e ajuste suas permissões

    touch /var/log/unicorn_puppetmaster.log
    touch /var/log/unicorn_stderr.log
    chown puppet:puppet /var/log/unicorn_*.log

## Configuração do God

Crie o arquivo **/etc/god/puppetmaster.god** com o conteúdo abaixo

```
God.watch do |w|
  w.name = "puppetmaster"
  w.interval = 30.seconds
  w.pid_file = "/var/run/puppet/puppetmaster_unicorn.pid"
  w.log = "/var/log/unicorn_puppetmaster.log"
  w.dir = "/etc/puppet"

  w.start = "ulimit -v 750000 && exec /usr/bin/unicorn -c /etc/puppet/unicorn.conf -D"
  w.stop = "kill -QUIT `cat #{w.pid_file}`"
  w.restart = "kill -USR2 `cat #{w.pid_file}`"

  w.start_grace = 10.seconds
  w.restart_grace = 10.seconds

  w.uid = "puppet"
  w.gid = "puppet"

  w.behavior(:clean_pid_file)

  # determine the state on startup
  w.transition(:init, { true => :up, false => :start }) do |on|
    on.condition(:process_running) do |c|
      c.running = true
    end
  end

  # determine when process has finished starting
  w.transition([:start, :restart], :up) do |on|
    on.condition(:process_running) do |c|
      c.running = true
    end

    # failsafe
    on.condition(:tries) do |c|
      c.times = 5
      c.transition = :start
    end
  end

  # start if process is not running
  w.transition(:up, :start) do |on|
    on.condition(:process_exits)
  end

  w.start_if do |start|
    start.condition(:process_running) do |c|
      c.interval = 5.seconds
      c.running = false
    end
  end
end
```

Crie o arquivo /**etc/init/god.conf** com o conteúdo abaixo:

```
start on stopped rc RUNLEVEL=[2345]

stop on starting rc RUNLEVEL=[!2345]

env HOME=/etc/puppet

console output
respawn
respawn limit 10 120
exec /usr/bin/god --log-level info -D -c '/etc/god/*.god'
```
Inicie o god

    start god
    god start/running, process 1976
    
Verifique o status dos serviços mantidos pelo god

    god status
    puppetmaster: init
    
Aguarde e verifique novamente se o serviço subiu (up)

    god status
    puppetmaster: up
    
Quando aparecer UP, significará que o serviço subiu, você pode checar diretamente

    ps aux|grep unicorn

Acompanhe a saída do comando

```
puppet    1398  0.2  4.8 140252 48676 ?        S    18:22   0:01 unicorn master -c /etc/puppet/unicorn.conf -D
puppet    1404  0.0  4.5 140216 46484 ?        S    18:22   0:00 unicorn worker[0] -c /etc/puppet/unicorn.conf -D
puppet    1405  0.0  4.5 140220 46484 ?        S    18:22   0:00 unicorn worker[1] -c /etc/puppet/unicorn.conf -D
puppet    1406  0.0  4.5 140224 46484 ?        S    18:22   0:00 unicorn worker[2] -c /etc/puppet/unicorn.conf -D
puppet    1407  0.0  4.5 140228 46488 ?        S    18:22   0:00 unicorn worker[3] -c /etc/puppet/unicorn.conf -D
puppet    1408  0.0  4.5 140232 46488 ?        S    18:22   0:00 unicorn worker[4] -c /etc/puppet/unicorn.conf -D
puppet    1409  0.0  4.5 140236 46488 ?        S    18:22   0:00 unicorn worker[5] -c /etc/puppet/unicorn.conf -D
puppet    1410  0.0  4.5 140240 46492 ?        S    18:22   0:00 unicorn worker[6] -c /etc/puppet/unicorn.conf -D
puppet    1411  0.0  4.5 140244 46492 ?        S    18:22   0:00 unicorn worker[7] -c /etc/puppet/unicorn.conf -D
```

O unicorn está rodando, agora para testar se o god está funcionando na inicialização, reinicie o seu servidor e verifique se o serviço vai carregar automaticamente.

    reboot

## Manipulando o God

Para carregar um novo serviço

    god load servico

Para iniciar um serviço

    god start servico
    
Para parar um serviço

    god stop servico
    
Para reiniciar um serviço

    god restart servico
    
Para verificar o status dos serviços mantidos pelo god

    god status
    
Para verificar o status de um serviço específico

    god status servico
    
Para ver outros comandos do God

    god --help

## Amarrando as pontas

Com o GOD vamos ter a garantia de que alguém estará protegendo e monitorando o processo de nosso puppemaster, e caso o processo falhe, caso  o processo caia, o GOD irá iniciá-lo novamente. Isto traz maior disponibilidade para nosso ambiente e maior controle dos processos. Vale a pena investir nele.

Com o GOD, não há necessiade de criar um script INIT para o serviço já que será ele que vai cuidar do processo, no tutorial acima, criamos apenas um init no upstart para o GOD, o resto é com ele.

## Referências

* http://godrb.com/
* http://trateotu.proxy.nl/centos-6-5-puppet-v3-nginx-unicorn-and-god/
* http://wiki.unixh4cks.com/index.php/Puppetmaster_:_Nginx_%2B_Unicorn
* http://projects.puppetlabs.com/projects/1/wiki/using_unicorn