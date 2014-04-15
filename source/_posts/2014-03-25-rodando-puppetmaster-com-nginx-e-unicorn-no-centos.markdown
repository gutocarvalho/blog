---
layout: post
title: "Rodando PuppetMaster com Nginx e Unicorn no CentOS"
date: 2014-03-25 15:13
comments: true
categories: puppet
---

## 1. Acelerando e escalando master com Unicorn e Nginx

No post [anterior](http://gutocarvalho.net/octopress/2014/03/25/rodando-puppetmaster-com-nginx-e-unicorn-no-ubuntu/) demonstrei esse procedimento no Ubuntu, agora vamos ver como fazer no CentOS.

O UNICORN é um servidor web especializado em apps ruby, escrito em ruby e focado em performance, muito mais eficiente e rápido do que o webrick, e também com melhor performance se compararmos com apache + passenger.

## 2. Cenário

Neste cenário utilizei apenas uma VM para instalar e validar o funcionamento do puppetmaster com unicorn.

- CentOS 6.5
- 1 VM VMWARE, Puppet Master (1 VPCU, 1GB RAM, 4GB DISCO)

## 3. Procedimento

Abaixo segue o procedimento passo a passo para configuração deste cenário.

### 3.1 Configurando hostname

Ajuste o nome do seu servidor nos seguintes arquivos para puppet.hacklab

    # echo NETWORKING=yes > /etc/sysconfig/network
    # echo HOSTNAME=puppet.hacklab >> /etc/sysconfig/network
    # echo 127.0.0.1 localhost puppet.hacklab puppet > /etc/hosts
    # hostname puppet.hacklab

Faça logout e login e depois rode o comando abaixo

    # hostname
    
Ele precisa retornar o nome

    puppet.hacklab

### 3.2 Instalando repositório   
   
Faça o download do pacote release da puppetlabs

    yum install http://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm

### 3.3 Instalando puppetmaster
    
Instale o puppetmaster
 
    yum install puppet-server
    
Desative o daemon puppetmaster no boot pois vamos usar o unicorn
 
    chkconfig puppetmaster off

Desligue o daemon puppetmaster - se estiver ligado - para que a porta 8140 seja liberada para o nginx

    service puppetmaster stop
    
### 3.4 Instalando Unicorn

Instale as dependências necessárias

    yum install make gcc ruby-devel rubygems
    
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
I, [2014-03-23T15:14:30.028793 #2067]  INFO -- : Refreshing Gem list
I, [2014-03-23T15:14:31.142611 #2067]  INFO -- : unlinking existing socket=/var/run/puppet/puppetmaster_unicorn.sock
I, [2014-03-23T15:14:31.142844 #2067]  INFO -- : listening on addr=/var/run/puppet/puppetmaster_unicorn.sock fd=6
I, [2014-03-23T15:14:31.170260 #2073]  INFO -- : worker=0 spawned pid=2073
I, [2014-03-23T15:14:31.186108 #2073]  INFO -- : worker=0 ready
I, [2014-03-23T15:14:31.199679 #2074]  INFO -- : worker=1 spawned pid=2074
I, [2014-03-23T15:14:31.207474 #2074]  INFO -- : worker=1 ready
I, [2014-03-23T15:14:31.224235 #2075]  INFO -- : worker=2 spawned pid=2075
I, [2014-03-23T15:14:31.243385 #2075]  INFO -- : worker=2 ready
I, [2014-03-23T15:14:31.256398 #2076]  INFO -- : worker=3 spawned pid=2076
I, [2014-03-23T15:14:31.266525 #2076]  INFO -- : worker=3 ready
I, [2014-03-23T15:14:31.280448 #2077]  INFO -- : worker=4 spawned pid=2077
I, [2014-03-23T15:14:31.289763 #2077]  INFO -- : worker=4 ready
I, [2014-03-23T15:14:31.310051 #2078]  INFO -- : worker=5 spawned pid=2078
I, [2014-03-23T15:14:31.320665 #2078]  INFO -- : worker=5 ready
I, [2014-03-23T15:14:31.339841 #2079]  INFO -- : worker=6 spawned pid=2079
I, [2014-03-23T15:14:31.355532 #2079]  INFO -- : worker=6 ready
I, [2014-03-23T15:14:31.357241 #2067]  INFO -- : master process ready
I, [2014-03-23T15:14:31.362016 #2080]  INFO -- : worker=7 spawned pid=2080
I, [2014-03-23T15:14:31.367252 #2080]  INFO -- : worker=7 ready
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

exec /usr/bin/unicorn -c /etc/puppet/unicorn.conf -D
```

Inicie o serviço

     start unicorn

Verifique se o unicorn está realmente rodando

    ps aux|grep unicorn

A saída será similar a esta abaixo

```
puppet    2164  0.1  4.7 140108 48580 ?        S    15:37   0:01 unicorn master -D -c /etc/puppet/unicorn.conf
puppet    2170  0.0  4.5 140072 46376 ?        S    15:37   0:00 unicorn worker[0] -D -c /etc/puppet/unicorn.conf
puppet    2171  0.0  4.5 140076 46376 ?        S    15:37   0:00 unicorn worker[1] -D -c /etc/puppet/unicorn.conf
puppet    2172  0.0  4.5 140080 46376 ?        S    15:37   0:00 unicorn worker[2] -D -c /etc/puppet/unicorn.conf
puppet    2173  0.0  4.5 140084 46376 ?        S    15:37   0:00 unicorn worker[3] -D -c /etc/puppet/unicorn.conf
puppet    2174  0.0  4.5 140088 46380 ?        S    15:37   0:00 unicorn worker[4] -D -c /etc/puppet/unicorn.conf
puppet    2175  0.0  4.5 140092 46380 ?        S    15:37   0:00 unicorn worker[5] -D -c /etc/puppet/unicorn.conf
puppet    2176  0.0  4.5 140096 46380 ?        S    15:37   0:00 unicorn worker[6] -D -c /etc/puppet/unicorn.conf
puppet    2177  0.0  6.5 159236 66328 ?        S    15:37   0:00 unicorn worker[7] -D -c /etc/puppet/unicorn.conf
```

### 3.5 Instalando o Nginx

Instale o nginx

    yum install nginx

Ative o nginx no boot

    chkconfig nginx on

Crie o arquivo **/etc/nginx/conf.d/puppetmaster.conf** com o conteúdo abaixo

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
Lembre-se de ajustar o nome do certificado de acordo com seu FQDN, após r 	einicie o nginx

    /etc/init.d/nginx restart
   
Verfique se a porta 8140 está em modo listen

    root@puppet:/etc/nginx/sites-enabled# netstat -ntpl|grep 8140
    
Acompanhe a saída

    tcp    0      0 0.0.0.0:8140    0.0.0.0:*  OUÇA       2204/nginx
    
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

Acompanhe a saída do comando e observe o puppet aplicando a configuração

```
Info: Retrieving plugin
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395600179'
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Package[htop]/ensure: created
Info: Creating state file /var/lib/puppet/state/state.yaml
Notice: Finished catalog run in 3.56 seconds
```
Veja o log do Nginx

    tail -f /var/log/nginx/access.log

Acompanhe o nginx atuando como reverse proxy do unicorn

```
192.168.200.1 - - [21/Mar/2014:04:11:14 -0300] "GET /production/file_metadatas/plugins?links=manage&recurse=true&checksum_type=md5&ignore=.svn&ignore=CVS&ignore=.git HTTP/1.1" 200 46650 "-" "-" "-"
192.168.200.1 - - [21/Mar/2014:04:11:24 -0300] "POST /production/catalog/puppet.hacklab HTTP/1.1" 200 77809 "-" "-" "-"
192.168.200.1 - - [21/Mar/2014:04:11:26 -0300] "GET /production/file_metadata/modules/concat/concatfragments.sh?links=manage HTTP/1.1" 200 307 "-" "-" "-"
192.168.200.1 - - [21/Mar/2014:04:11:27 -0300] "GET /production/file_metadata/modules/postgresql/validate_postgresql_connection.sh?links=manage HTTP/1.1" 200 326 "-" "-" "-"
192.168.200.1 - - [21/Mar/2014:04:11:32 -0300] "GET /production/file_metadata/modules/puppetdb/routes.yaml?links=manage HTTP/1.1" 200 302 "-" "-" "-"
192.168.200.1 - - [21/Mar/2014:04:11:32 -0300] "PUT /production/report/puppet.hacklab HTTP/1.1" 200 9 "-" "-" "-"
192.168.200.1 - - [23/Mar/2014:15:42:55 -0300] "GET /production/node/puppet.hacklab? HTTP/1.1" 200 84 "-" "-" "-"
192.168.200.1 - - [23/Mar/2014:15:42:56 -0300] "GET /production/file_metadatas/plugins?ignore=.svn&ignore=CVS&ignore=.git&checksum_type=md5&links=manage&recurse=true HTTP/1.1" 200 283 "-" "-" "-"
192.168.200.1 - - [23/Mar/2014:15:43:00 -0300] "POST /production/catalog/puppet.hacklab HTTP/1.1" 200 1022 "-" "-" "-"
192.168.200.1 - - [23/Mar/2014:15:43:04 -0300] "PUT /production/report/puppet.hacklab HTTP/1.1" 200 9 "-" "-" "-"
```

Configuração aplicada, isto significa que o puppetmaster está rodando com Unicorn e Nginx, encaixe perfeito.

## 4. Amarrando as pontas

Como eu já mencionei no post anterior, é um processo relativamente simples e bastante rápido, utilizamos apenas pacotes oficiais da puppetlabs e neste caso CentOS, e usamos dois gems. Se quiser pode integrar o puppetdb no mesmo cenário. Com esse setup o puppetmaster fica rápido e escalável, recomendo.

## 5. Referências

Projects

* http://unicorn.bogomips.org/
* http://wiki.nginx.org/Main
* http://puppetlabs.com/

Blogs

* http://linuxmoz.com/rhel-centos-install-puppet-nginx-unicorn/
* http://tomayko.com/writings/unicorn-is-unix
* http://wiki.unixh4cks.com/index.php/Puppetmaster_:_Nginx_%2B_Unicorn
* http://projects.puppetlabs.com/projects/1/wiki/using_unicorn
* http://www.prontab.com/2011/01/this-page-should-outline-how-to-set-up.html
