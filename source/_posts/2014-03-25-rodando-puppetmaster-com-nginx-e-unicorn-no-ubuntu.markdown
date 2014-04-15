---
layout: post
title: "Rodando PuppetMaster com Nginx e Unicorn no Ubuntu"
date: 2014-03-25 10:33
comments: true
categories: puppet
---

## 1. Acelerando e escalando master com Unicorn e Nginx

Neste post pretendo demonstrar como instalar e rodar o PuppetMaster utilizando o UNICORN como servidor de aplicação e NGINX como proxy reverso.

Se deseja fazer esse procedimento no CentOS, acesse esse [link](http://gutocarvalho.net/octopress/2014/03/25/rodando-puppetmaster-com-nginx-e-unicron-no-centos/).

O UNICORN é um servidor web especializado em apps ruby, escrito em ruby e focado em performance, muito mais eficiente e rápido do que o webrick, e também com melhor performance se compararmos com apache + passenger.

## 2. Cenário

Neste cenário utilizei apenas uma VM para instalar e validar o funcionamento do puppetmaster com unicorn.

- Ubuntu 12.04
- 1 VM VMWARE, Puppet Master (1 VPCU, 1GB RAM, 4GB DISCO)

## 3. Procedimento

Abaixo segue o procedimento passo a passo para configuração deste cenário.

### 3.1 Configurando hostname

Ajuste o nome do seu servidor nos seguintes arquivos para puppet.hacklab

    # echo puppet.hacklab > /etc/hostname
    # echo 127.0.0.1 localhost puppet.hacklab puppet > /etc/hosts
    # hostname puppet.hacklab

Faça logout e login e depois rode o comando abaixo

    # hostname
    
Ele precisa retornar o nome

    puppet.hacklab

### 3.2 Instalando repositório   
   
Faça o download do pacote release da puppetlabs

    wget http://apt.puppetlabs.com/puppetlabs-release-precise.deb

Instalando o repositório da puppetlabs

    dpkg -i puppetlabs-release-precise.deb

### 3.3 Instalando puppetmaster
    
Instale o puppetmaster
 
    aptitude install puppetmaster
    
Desative o daemon puppetmaster no boot pois vamos usar o unicorn
 
    update-rc.d -f puppetmaster disable

Desligue o daemon puppetmaster para que a porta 8140 seja liberada para o nginx

    /etc/init.d/puppetmaster stop
    
Comente as linhas abaixo no /etc/puppet/puppet.conf (**importante!**)

    ssl_client_header = SSL_CLIENT_S_DN
    ssl_client_verify_header = SSL_CLIENT_VERIFY

### 3.4 Instalando Unicorn

Instale as dependências necessárias

    aptitude install make gcc ruby-dev rubygems
    
Instale o unicorn e rack utilizando gem (não tem pacotes deb)

    gem install unicorn rack

Crie o arquivo **/etc/puppet/config.ru** como seguinte conteúdo

```
# a config.ru, for use with every rack-compatible webserver.
# SSL needs to be handled outside this, though.

# if puppet is not in your RUBYLIB:
# $LOAD_PATH.unshift('/opt/puppet/lib')

$0 = "master"

# if you want debugging:
# ARGV << "--debug"

ARGV << "--rack"

# Rack applications typically don't start as root.  Set --confdir and --vardir
# to prevent reading configuration from ~puppet/.puppet/puppet.conf and writing
# to ~puppet/.puppet
ARGV << "--confdir" << "/etc/puppet"
ARGV << "--vardir"  << "/var/lib/puppet"

# NOTE: it's unfortunate that we have to use the "CommandLine" class
#  here to launch the app, but it contains some initialization logic
#  (such as triggering the parsing of the config file) that is very
#  important.  We should do something less nasty here when we've
#  gotten our API and settings initialization logic cleaned up.
#
# Also note that the "$0 = master" line up near the top here is
#  the magic that allows the CommandLine class to know that it's
#  supposed to be running master.
#
# --cprice 2012-05-22

require 'puppet/util/command_line'
# we're usually running inside a Rack::Builder.new {} block,
# therefore we need to call run *here*.
run Puppet::Util::CommandLine.new.execute
```

Caso as configurações do puppet estejam em um diretório diferente do padrão, ajuste a linha confdir, a mesma coisa para o diretório em que ficam os arquivos variáveis

    ARGV << "--confdir" << "/etc/puppet"
    ARGV << "--vardir"  << "/var/lib/puppet"

Crie o arquivo **/etc/puppet/unicorn.conf** com o seguinte conteúdo

```
 worker_processes 8
    working_directory "/etc/puppet"
    listen '/var/run/puppet/puppetmaster_unicorn.sock', :backlog => 512
    timeout 120
    pid "/var/run/puppet/puppetmaster_unicorn.pid"

    preload_app true
    if GC.respond_to?(:copy_on_write_friendly=)
      GC.copy_on_write_friendly = true
    end

    before_fork do |server, worker|
      old_pid = "#{server.config[:pid]}.oldbin"
      if File.exists?(old_pid) && server.pid != old_pid
        begin
          Process.kill("QUIT", File.read(old_pid).to_i)
        rescue Errno::ENOENT, Errno::ESRCH
          # someone else did our job for us
        end
      end
    end
```

Caso as configurações do puppet estejam em diretório diferente do padrão, ajuste a diretiva working_directory.

    working_directory "/etc/puppet"

Inicie o serviço para testar o unicorn

    root@ubuntu:/etc/puppet# unicorn -c unicorn.conf

Veja a saída esperada

```
I, [2014-03-22T00:09:14.089524 #2415]  INFO -- : Refreshing Gem list
I, [2014-03-22T00:09:15.402416 #2415]  INFO -- : listening on addr=/var/run/puppet/puppetmaster_unicorn.sock fd=8
I, [2014-03-22T00:09:15.432006 #2438]  INFO -- : worker=0 spawned pid=2438
I, [2014-03-22T00:09:15.444172 #2438]  INFO -- : worker=0 ready
I, [2014-03-22T00:09:15.462684 #2439]  INFO -- : worker=1 spawned pid=2439
I, [2014-03-22T00:09:15.476063 #2439]  INFO -- : worker=1 ready
I, [2014-03-22T00:09:15.494481 #2440]  INFO -- : worker=2 spawned pid=2440
I, [2014-03-22T00:09:15.519071 #2440]  INFO -- : worker=2 ready
I, [2014-03-22T00:09:15.566899 #2441]  INFO -- : worker=3 spawned pid=2441
I, [2014-03-22T00:09:15.579692 #2441]  INFO -- : worker=3 ready
I, [2014-03-22T00:09:15.600440 #2442]  INFO -- : worker=4 spawned pid=2442
I, [2014-03-22T00:09:15.652091 #2442]  INFO -- : worker=4 ready
I, [2014-03-22T00:09:15.663205 #2443]  INFO -- : worker=5 spawned pid=2443
I, [2014-03-22T00:09:15.676469 #2443]  INFO -- : worker=5 ready
I, [2014-03-22T00:09:15.724854 #2444]  INFO -- : worker=6 spawned pid=2444
I, [2014-03-22T00:09:15.735700 #2415]  INFO -- : master process ready
I, [2014-03-22T00:09:15.751096 #2445]  INFO -- : worker=7 spawned pid=2445
I, [2014-03-22T00:09:15.758621 #2444]  INFO -- : worker=6 ready
I, [2014-03-22T00:09:15.760709 #2445]  INFO -- : worker=7 ready
```

Se houver uma saída similar é sinal de que está fucionando como esperado, dê um CTRL+C para interromper o processo.

    ^C

Crie o arquivo **/etc/init/unicorn** com o conteúdo abaixo

```
# When to start the service
start on runlevel [2345]

# When to stop the service
stop on runlevel [016]

# Automatically restart process if crashed
respawn
respawn limit 5 15

# Upstart will expect the process executed to call fork(2) exactly twice.
expect daemon

exec /usr/local/bin/unicorn -c /etc/puppet/unicorn.conf -D
```
   
Inicie o serviço

     start unicorn

Verifique se o unicorn está realmente rodando

    ps aux|grep unicorn

A saída será similar a esta abaixo

```
puppet    1889  0.5  9.1 119412 45380 ?        S    20:05   0:01 unicorn master -D -c /etc/puppet/unicorn.conf
puppet    1896  0.0  8.7 119376 43416 ?        S    20:05   0:00 unicorn worker[0] -D -c /etc/puppet/unicorn.conf
puppet    1897  0.0  8.7 119380 43416 ?        S    20:05   0:00 unicorn worker[1] -D -c /etc/puppet/unicorn.conf
puppet    1898  0.0  8.7 119384 43416 ?        S    20:05   0:00 unicorn worker[2] -D -c /etc/puppet/unicorn.conf
puppet    1899  0.0  8.7 119388 43416 ?        S    20:05   0:00 unicorn worker[3] -D -c /etc/puppet/unicorn.conf
puppet    1900  0.0  8.7 119392 43416 ?        S    20:05   0:00 unicorn worker[4] -D -c /etc/puppet/unicorn.conf
puppet    1901  0.0  8.7 119396 43416 ?        S    20:05   0:00 unicorn worker[5] -D -c /etc/puppet/unicorn.conf
puppet    1902  0.0  8.7 119400 43416 ?        S    20:05   0:00 unicorn worker[6] -D -c /etc/puppet/unicorn.conf
puppet    1903  0.0  8.7 119404 43416 ?        S    20:05   0:00 unicorn worker[7] -D -c /etc/puppet/unicorn.conf
``

### 3.5 Instalando o Nginx

Instale o nginx

    aptitude install nginx

Ative o nginx no boot

    update-rc.d -f nginx defaults

Crie o arquivo **/etc/nginx/sites-available/puppetmaster** com o conteúdo abaixo

```
    upstream puppetmaster_unicorn {
        server unix:/var/run/puppet/puppetmaster_unicorn.sock fail_timeout=0;
    }

    server {
        listen 8140;

        ssl on;
        ssl_session_timeout 5m;
        ssl_certificate /var/lib/puppet/ssl/certs/puppet.hacklab.pem;
        ssl_certificate_key /var/lib/puppet/ssl/private_keys/puppet.hacklab.pem;
        ssl_client_certificate /var/lib/puppet/ssl/ca/ca_crt.pem;
        ssl_ciphers SSLv2:-LOW:-EXPORT:RC4+RSA;
        ssl_verify_client optional;

        root /usr/share/empty;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Client-Verify $ssl_client_verify;
        proxy_set_header X-Client-DN $ssl_client_s_dn;
        proxy_set_header X-SSL-Issuer $ssl_client_i_dn;
        proxy_read_timeout 120;

        location / {
            proxy_pass http://puppetmaster_unicorn;
            proxy_redirect off;
        }
    }
```
Lembre-se de ajustar o nome do certificado de acordo com seu FQDN, crie o link simbólico em seguida

    ln -s /etc/nginx/sites-available/puppetmaster /etc/nginx/sites-enabled/puppetmaster

Reinicie o nginx

    /etc/init.d/nginx restart
   
Verfique se a porta 8140 está em modo listen

    root@puppet:/etc/nginx/sites-enabled# netstat -ntpl|grep 8140
    
Acompanhe a saída

    tcp        0      0 0.0.0.0:8140            0.0.0.0:*               LISTEN      9960/nginx

### 3.7 Testando o Puppet

Crie o arquivo **/etc/puppet/manifests/site.pp** com o conteúdo abaixo

```
node "puppet.hacklab" {

        package { 'htop':
                ensure => present,
        }

}
```
Agora vamos testar se o puppet está funcionando.

    root@puppet:/etc/puppet# puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395768087'
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Package[htop]/ensure: ensure changed 'purged' to 'present'
Notice: Finished catalog run in 11.98 seconds
```

Veja o log do Nginx

    tail -f /var/log/nginx/access.log

Acompanhe o nginx atuando como reverse proxy do unicorn

```
172.16.184.153 - - [25/Mar/2014:17:19:54 +0000] "POST /production/catalog/puppet.hacklab HTTP/1.1" 400 121 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:19:55 +0000] "PUT /production/report/puppet.hacklab HTTP/1.1" 200 9 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:20:45 +0000] "GET /production/node/puppet.hacklab? HTTP/1.1" 200 4115 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:20:45 +0000] "GET /production/file_metadatas/plugins?ignore=.svn&ignore=CVS&ignore=.git&checksum_type=md5&links=manage&recurse=true HTTP/1.1" 200 283 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:20:50 +0000] "POST /production/catalog/puppet.hacklab HTTP/1.1" 400 164 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:20:50 +0000] "PUT /production/report/puppet.hacklab HTTP/1.1" 200 9 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:21:23 +0000] "GET /production/node/puppet.hacklab? HTTP/1.1" 200 4115 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:21:23 +0000] "GET /production/file_metadatas/plugins?ignore=.svn&ignore=CVS&ignore=.git&checksum_type=md5&links=manage&recurse=true HTTP/1.1" 200 283 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:21:28 +0000] "POST /production/catalog/puppet.hacklab HTTP/1.1" 200 1022 "-" "-"
172.16.184.153 - - [25/Mar/2014:17:21:41 +0000] "PUT /production/report/puppet.hacklab HTTP/1.1" 200 9 "-" "-"
```

Configuração aplicada, isto significa que o puppetmaster está rodando com Unicorn e Nginx, encaixe perfeito.

## 4. Amarrando as pontas

É um processo relativamente simples e bastante rápido, utilizamos apenas pacotes oficiais da puppetlabs e ubuntu, e dois gems. Se quiser pode integrar o puppetdb no mesmo cenário. Com esse setup o puppetmaster fica rápido e escalável, recomendo.

## 5. Referências

Projects

* http://unicorn.bogomips.org/
* http://wiki.nginx.org/Main
* http://puppetlabs.com/

Blogs

* http://tomayko.com/writings/unicorn-is-unix
* http://linuxmoz.com/rhel-centos-install-puppet-nginx-unicorn/
* http://tomayko.com/writings/unicorn-is-unix
* http://wiki.unixh4cks.com/index.php/Puppetmaster_:_Nginx_%2B_Unicorn
* http://projects.puppetlabs.com/projects/1/wiki/using_unicorn
* http://www.prontab.com/2011/01/this-page-should-outline-how-to-set-up.html
