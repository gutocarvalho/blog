---
layout: post
title: "Utilizando módulo Puppetlabs MySQL"
date: 2014-03-26 13:30
comments: true
categories: puppet
published: true
---

## 1. Módulos da Puppetlabs

Resolvi iniciar uma série de posts para divulgar os excelentes módulos da Puppetlabs, muita gente não os usa e não os conhece. Estes módulos são maduros e bem testados, afinal são construídos de forma coletiva pela comunidade puppet. 

Não vale a pena investir tempo reinventando a roda, use estes módulos!

## 2. Método aplicado

A ideia aqui não é mostrar como se instala e configura o PuppetMaster, isso já foi bem coberto neste blog e nas wikis do meu site, partirei da ideia de que você já tem um ambiente rodando e em produção. Eu simplesmente vou mostrar alguns exemplos de uso do módulo para facilitar a configuração.

## 3. Entendendo os módulos

Todos os módulos da Puppetlabs tem um arquivo **README**, este arquivo contém informações mais do que suficientes para entender seu funcionamento e fazer
uso dele.

Você vai encontrar os módulos da puppetlabs no [Forge](http://forge.puppetlabs.com/puppetlabs) e no [Github](https://github.com/puppetlabs/). 

No caso do Github existem outros projetos no repositório além dos módulos, normalmente os módulos terão o nome de puppetlabs-module, mas na descrição estará bem claro.

## 4. Sobre o módulo Puppetlabs MySQL

Este módulo é bastante interessante, particularmente eu o uso para gerenciar todos os meus servidores mysql, ele possui as seguintes classes.

Public classes

* mysql::server: Installs and configures MySQL.
* mysql::server::account_security: Deletes default MySQL accounts.
* mysql::server::monitor: Sets up a monitoring user.
* mysql::server::mysqltuner: Installs MySQL tuner script.
* mysql::server::backup: Sets up MySQL backups via cron.
* mysql::bindings: Installs various MySQL language bindings.
* mysql::client: Installs MySQL client (for non-servers).

Private classes

* mysql::server::install: Installs packages.
* mysql::server::config: Configures MYSQL.
* mysql::server::service: Manages service.
* mysql::server::root_password: Sets MySQL root password.
* mysql::server::providers: Creates users, grants, and databases.
* mysql::bindings::java: Installs Java bindings.
* mysql::bindings::perl: Installs Perl bindings.
* mysql::bindings::python: Installs Python bindings.
* mysql::bindings::ruby: Installs Ruby bindings.
* mysql::client::install: Installs MySQL client.

Cada uma destas classes tem diversos parâmetros, portanto, recomendo uma leitura completa no README do módulo antes de iniciar suas aventuras.

* https://github.com/puppetlabs/puppetlabs-mysql

## 4. Instalando o módulo Mysql

No servidor puppetmaster acesse o diretório de módulos

    cd /etc/puppet/modules

E inicie uma busca por módulos da puppetlabs

    puppet module search puppetlabs

Veja os módulos disponíveis (filtrado)

```
puppetlabs-activemq
puppetlabs-apache
puppetlabs-appdirector
puppetlabs-apt
puppetlabs-awsdemo_profiles
puppetlabs-azure
puppetlabs-bacula
puppetlabs-boundary
puppetlabs-ceilometer
puppetlabs-cinder
puppetlabs-cloud_provisioner
puppetlabs-cloudformation
puppetlabs-concat
puppetlabs-corosync
puppetlabs-dashboard
puppetlabs-denyhosts
puppetlabs-dism
puppetlabs-drbd
puppetlabs-f5
puppetlabs-firewall
puppetlabs-gcc
puppetlabs-gce_compute
puppetlabs-git
puppetlabs-glance
puppetlabs-grizzly
puppetlabs-haproxy
puppetlabs-havana
puppetlabs-heat
puppetlabs-horizon
puppetlabs-inifile
puppetlabs-java
puppetlabs-java_ks
puppetlabs-keystone
puppetlabs-kwalify
puppetlabs-lib_puppet
puppetlabs-logentries
puppetlabs-lvm
puppetlabs-mcollective
puppetlabs-mongodb
puppetlabs-motd
puppetlabs-mount_providers
puppetlabs-mrepo
puppetlabs-mssql
puppetlabs-mysql
puppetlabs-neutron
puppetlabs-newrelic
puppetlabs-nginx
puppetlabs-node_gce
puppetlabs-node_openstack
puppetlabs-nodejs
puppetlabs-nova
puppetlabs-ntp
puppetlabs-opennebula
puppetlabs-openstack
puppetlabs-passenger
puppetlabs-pe_gem
puppetlabs-pe_upgrade
puppetlabs-postgresql
puppetlabs-puppetdb
puppetlabs-quantum
puppetlabs-rabbitmq
puppetlabs-razor
puppetlabs-reboot
puppetlabs-registry
puppetlabs-rsync
puppetlabs-ruby
puppetlabs-sqlite
puppetlabs-stdlib
puppetlabs-stunnel
puppetlabs-swift
puppetlabs-tftp
puppetlabs-vcenter
puppetlabs-vcli_rsyslog
puppetlabs-vcsrepo
puppetlabs-vswitch
puppetlabs-win_desktop_shortcut
puppetlabs-xinetd
```
Vamos instalar o módulo mysql

    puppet module install puppetlabs-mysql

Acompanhe a saída

```
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forge.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└─┬ puppetlabs-mysql (v2.2.3)
  └── puppetlabs-stdlib (v4.1.0)
```

Veja que ele instalou o módulo stdlib que é dependência para este módulo.

## 5. Utilizando o módulo

Edite o arquivo /etc/puppet/manifests/site.pp e insira as configuações abaixo no node que receberá as configurações para instalação do MYSQL Server.

### 5.1 Para instalar o Mysql Server

{% codeblock lang:puppet %}
node "mysql.hacklab" {

	class { 'mysql::server':

		root_password    => 'suasenha',

		override_options => {
			'mysqld' => {
				'connect_timeout'                 => '60',
				'bind_address'                    => '0.0.0.0',
				'max_connections'                 => '100',
				'max_allowed_packet'              => '512M',
				'thread_cache_size'               => '16',
				'query_cache_size'                => '128M',
			}
		}
	}
}
{% endcodeblock %}

Observe que é uma configuração bastante objetiva e direta, e eu uso a diretiva override_options para definir algumas configurações diretamente no aquivo my.cnf, sem depender dos parâmetros do módulo diretamente. Agora vamos aplicar a configuração.

    puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/puppet_vardir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/root_home.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/pe_version.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395658656'
Notice: /Stage[main]/Mysql::Server::Install/Package[mysql-server]/ensure: created
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/mysql]/ensure: created
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/my.cnf]/content:
--- /etc/my.cnf	2014-02-12 17:42:13.000000000 -0200
+++ /tmp/puppet-file20140324-2025-10nhr9y-0	2014-03-24 07:58:32.036141657 -0300
@@ -1,10 +1,46 @@
+[client]
+port = 3306
+socket = /var/lib/mysql/mysql.sock
+
+[isamchk]
+key_buffer_size = 16M
+
 [mysqld]
-datadir=/var/lib/mysql
-socket=/var/lib/mysql/mysql.sock
-user=mysql
-# Disabling symbolic-links is recommended to prevent assorted security risks
-symbolic-links=0
+basedir = /usr
+bind_address = 0.0.0.0
+connect_timeout = 60
+datadir = /var/lib/mysql
+expire_logs_days = 10
+key_buffer_size = 16M
+log-error = /var/log/mysqld.log
+max_allowed_packet = 512M
+max_binlog_size = 100M
+max_connections = 1400
+myisam_recover = BACKUP
+pid-file = /var/run/mysqld/mysqld.pid
+port = 3306
+query_cache_limit = 1M
+query_cache_size = 128M
+skip-external-locking
+socket = /var/lib/mysql/mysql.sock
+ssl = false
+ssl-ca = /etc/mysql/cacert.pem
+ssl-cert = /etc/mysql/server-cert.pem
+ssl-key = /etc/mysql/server-key.pem
+thread_cache_size = 16
+thread_stack = 256K
+tmpdir = /tmp
+user = mysql

 [mysqld_safe]
-log-error=/var/log/mysqld.log
-pid-file=/var/run/mysqld/mysqld.pid
+log-error = /var/log/mysqld.log
+nice = 0
+socket = /var/lib/mysql/mysql.sock
+
+[mysqldump]
+max_allowed_packet = 16M
+quick
+quote-names
+
+
+!includedir /etc/mysql/conf.d/

Info: /Stage[main]/Mysql::Server::Config/File[/etc/my.cnf]: Filebucketed /etc/my.cnf to puppet with sum 8ace886bbe7e274448bc8bea16d3ead6
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/my.cnf]/content: content changed '{md5}8ace886bbe7e274448bc8bea16d3ead6' to '{md5}995ee9b1c5173f66fe1e8adf81d1e0f0'
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/mysql/conf.d]/ensure: created
Notice: /Stage[main]/Mysql::Server::Service/Service[mysqld]/ensure: ensure changed 'stopped' to 'running'
Info: /Stage[main]/Mysql::Server::Service/Service[mysqld]: Unscheduling refresh on Service[mysqld]
Notice: /Stage[main]/Mysql::Server::Root_password/Mysql_user[root@localhost]/password_hash: defined 'password_hash' as '*686FAFB0D22504AC4005869A48377E6056C40F6A'
Notice: /Stage[main]/Mysql::Server::Root_password/File[/root/.my.cnf]/ensure: defined content as '{md5}c51fe9a952a98fdd64ad44e0f4b9e660'
Notice: Finished catalog run in 53.98 seconds
```

Veja que o puppet instalou o mysqlserver, ajustou a senha de root, e alterou o arquivo de configuração. Agora verifique se o serviço está rodando

    netstat -ntpl|grep 3306
    tcp        0      0 0.0.0.0:3306     0.0.0.0:*  OUÇA     2495/mysqld
    
Tente se conectar no mysql

    mysql

Acompanhe

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 74
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>    
```
Tudo perfeito.   
 
### 5.2 Criando database, usuário e grant

Abaixo vamos ajustar a configuração do node para criar um banco de dados para a aplicação teampass, já vamos definir o nome do banco, nome de usuário de acesso ao banco, senha do usuário e qual o host autoriado a acessar este banco com estas credenciais.

{% codeblock lang:puppet %}
node "mysql.hacklab" {

	# instalando mysql server

	class { 'mysql::server':
		root_password    => 'suasenha',
		override_options => {
			'mysqld' => {
				'connect_timeout'                 => '60',
				'bind_address'                    => '0.0.0.0',
				'max_connections'                 => '1400',
				'max_allowed_packet'              => '512M',
				'thread_cache_size'               => '16',
				'query_cache_size'                => '128M',
			}
		}
	}

	# sistema teampass

        mysql::db { 'prod_teampass':
                user     => 'teampass_user',
                password => 'senha_teampass',
                host     => 'teampass.hacklab',
                grant    => ['all'],
        }
}
{% endcodeblock %}

Feito os ajustes vamos rodar o puppet

    puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/puppet_vardir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/root_home.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/pe_version.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395659211'
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Mysql::Db[prod_teampass]/Mysql_database[prod_teampass]/ensure: created
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Mysql::Db[prod_teampass]/Mysql_user[teampass_user@teampass.hacklab]/ensure: created
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Mysql::Db[prod_teampass]/Mysql_grant[teampass_user@teampass.hacklab/prod_teampass.*]/ensure: created
Notice: Finished catalog run in 5.91 seconds
```

Base criada, usuário criado, grant criado, simples, rápido e fácil.

### 5.3 Criando usuário gutocarvalho

Agora vamos criar o usuário gutocarvalho, vamos configurar uma senha e determinar o host de origem dos acessos deste usuário.

{% codeblock lang:puppet %}
node "mysql.hacklab" {

	# instalando mysql server

	class { 'mysql::server':
		root_password    => 'suasenha',
		override_options => {
			'mysqld' => {
				'connect_timeout'                 => '60',
				'bind_address'                    => '0.0.0.0',
				'max_connections'                 => '1400',
				'max_allowed_packet'              => '512M',
				'thread_cache_size'               => '16',
				'query_cache_size'                => '128M',
			}
		}
	}

	# sistema teampass

        mysql::db { 'prod_teampass':
                user     => 'teampass_user',
                password => 'senha_teampass',
                host     => 'teampass.hacklab',
                grant    => ['all'],
        }

	# usuario gutocarvalho

        mysql_user{ 'gutocarvalho@10.138.25.102':
                ensure        => present,
                password_hash => mysql_password('minhasenha'),
                require       => Class['mysql::server'],
        }
}
{% endcodeblock %}

Vamos aplicar a configuração

    puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/puppet_vardir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/root_home.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/pe_version.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395659615'
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Mysql_user[gutocarvalho@10.138.25.102]/ensure: created
Notice: Finished catalog run in 5.79 seconds
```
Veja que o usuário foi criado no mysql.

### 5.4 Configurando grant para usuário

Agora vamos ver como definir um grant para um usuário, no caso vamos dizer ao puppet que o usuário gutocarvalho terá permissão de SELECT no banco prod_teampass.

{% codeblock lang:puppet %}
node "mysql.hacklab" {

	# instalando mysql server

	class { 'mysql::server':
		root_password    => 'suasenha',
		override_options => {
			'mysqld' => {
				'connect_timeout'                 => '60',
				'bind_address'                    => '0.0.0.0',
				'max_connections'                 => '1400',
				'max_allowed_packet'              => '512M',
				'thread_cache_size'               => '16',
				'query_cache_size'                => '128M',
			}
		}
	}

	# sistema teampass

        mysql::db { 'prod_teampass':
                user     => 'teampass_user',
                password => 'senha_teampass',
                host     => 'teampass.hacklab',
                grant    => ['all'],
        }

	# usuario gutocarvalho

		mysql_user { 'gutocarvalho@10.138.25.102':
                ensure        => present,
                password_hash => mysql_password('minhasenha'),
                require       => Class['mysql::server'],
       }

	# grant para usuario gutocarvalho no banco teampass

		mysql_grant { 'gutocarvalho@10.138.25.102/prod_teampass.*':
                user       => 'gutocarvalho@10.138.25.102',
                table      => 'prod_teampass.*',
                privileges => ['SELECT'],
		}
}
{% endcodeblock %}

Vamos aplicar a configuração

    puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/puppet_vardir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/root_home.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/pe_version.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395660070'
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Mysql_grant[gutocarvalho@10.138.25.102/prod_teampass.*]/ensure: created
Notice: Finished catalog run in 7.50 seconds
```

O grant foi criado.

## 6. Amarrando as pontas

Neste post eu abordei apenas estes três procedimentos, são os mais comuns, os mais usados. Com o que você fez até aqui, já dá para ter uma ideia de como instalar, configurar e utilizar um módulo da puppetlabs.

Observe que a declaração dos recursos no node serve como uma documentação, como o Miguel Filho diz, é uma verdadeira documentação executável, tudo o que você precisa saber sobre esse servidor de banco está ali, e com esse dados você inclusive consegue reconstruir a máquina rapidamente.

O uso do Puppet torna a administração do sgbd um processo limpo, organizado, rápido e eficiente, vale a pena usá-lo.

## 7. Referências

* https://github.com/puppetlabs/puppetlabs-mysql
* http://forge.puppetlabs.com/puppetlabs
* http://docs.puppetlabs.com/puppet/latest/reference/modules_fundamentals.html


