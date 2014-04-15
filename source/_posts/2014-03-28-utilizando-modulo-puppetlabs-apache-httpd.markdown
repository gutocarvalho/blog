---
layout: post
title: "Utilizando módulo Puppetlabs Apache"
date: 2014-03-28 16:30
comments: true
categories: puppet
published: true
---

## 1. Módulos da Puppetlabs

Como informei no post [anterior](http://gutocarvalho.net/octopress/2014/03/26/utilizando-modulo-puppetlabs-mysql/), resolvi iniciar uma série de posts para divulgar os excelentes módulos da Puppetlabs, muita gente não os usa e não os conhece. Estes módulos são maduros e bem testados, afinal são construídos de forma coletiva pela comunidade puppet. 

Não vale a pena investir tempo reinventando a roda, use estes módulos!

## 2. Método aplicado

A ideia aqui não é mostrar como se instala e configura o PuppetMaster, isso já foi bem coberto neste blog e nas wikis do meu site, partirei da ideia de que você já tem um ambiente rodando e em produção. Eu simplesmente vou mostrar alguns exemplos de uso do módulo para facilitar a configuração.

## 3. Entendendo os módulos

Todos os módulos da Puppetlabs tem um arquivo **README**, este arquivo contém informações mais do que suficientes para entender seu funcionamento e fazer
uso dele.

Você vai encontrar os módulos da puppetlabs no [Forge](http://forge.puppetlabs.com/puppetlabs) e no [Github](https://github.com/puppetlabs/). 

No caso do Github existem outros projetos no repositório além dos módulos, normalmente os módulos terão o nome de puppetlabs-module, mas na descrição estará bem claro.

## 4. Sobre o módulo Puppetlabs Apache

Este módulo é bastante interessante, particularmente eu o uso para gerenciar meus servidores apache, com ele consigo instalar o Apache, habilitar módulos, criar vhost, configurar vhosts e muito mais.

### 4.1 Classes

#### 4.1.1 Public Classes

* apache: Guides the basic setup of Apache.
* apache::dev: Installs Apache development libraries. (Note: On FreeBSD, you must declare apache::package or apache before apache::dev.)
* apache::mod::[name]: Enables specific Apache HTTPD modules.

### 4.1.2 Private Classes

* apache::confd::no_accf: Creates the no-accf.conf configuration file in conf.d, required by FreeBSD's Apache 2.4.
* apache::default_confd_files: Includes conf.d files for FreeBSD.
* apache::default_mods: Installs the Apache modules required to run the default configuration.
* apache::package: Installs and configures basic Apache packages.
* apache::params: Manages Apache parameters.
* apache::service: Manages the Apache daemon.

### 4.2 Defined Types

#### 4.2.1 Public Defined Types

* apache::balancer: Creates an Apache balancer cluster.
* apache::balancermember: Defines members of mod_proxy_balancer.
* apache::listen: Based on the title, controls which ports Apache binds to for listening. Adds Listen directives to ports.conf in the Apache HTTPD configuration directory. Titles take the form '', ':', or ':'.
* apache::mod: Used to enable arbitrary Apache HTTPD modules for which there is no specific apache::mod::[name] class.
* apache::namevirtualhost: Enables name-based hosting of a virtual host. Adds all NameVirtualHost directives to the ports.conf file in the Apache HTTPD configuration directory. Titles take the form '*', '*:', '_default_:, '', or ':'.
* apache::vhost: Allows specialized configurations for virtual hosts that have requirements outside the defaults.

#### 4.2.2 Private Defined Types

* apache::peruser::multiplexer: Enables the Peruser module for FreeBSD only.
* apache::peruser::processor: Enables the Peruser module for FreeBSD only.

### 4.3 Templates

The Apache module relies heavily on templates to enable the vhost and apache::mod defined types. These templates are built based on Facter facts around your operating system. Unless explicitly called out, most templates are not meant for configuration.

## 5. Instalando o módulo Apache

No servidor puppetmaster acesse o diretório de módulos

    cd /etc/puppet/modules

Vamos instalar o módulo apache

    puppet module install puppetlabs-apache

Acompanhe a saída

```
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forge.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└─┬ puppetlabs-apache (v1.0.1)
  └── puppetlabs-concat (v1.0.2)
```

Veja que ele instalou o módulo concat que é dependência para este módulo.

## 5. Utilizando o módulo

Edite o arquivo **/etc/puppet/manifests/site.pp** e insira o código abaixo no node que receberá as configurações para instalação do Apache HTTPd.

### 5.1 Para instalar o Apache HTTPD Server

É importante você saber que configurações do apache que **não** são gerenciadas pelo Puppet serão removidas, esse módulo leva em conta que seu apache só será gerido pelo Puppet, logo tome cuidado ao usar em produção, pense em migrar o ambiente, será mais seguro. Eu criei uma nova VM, avaliei a VM antiga, escrevi toda a configuração, copiei os arquivos de aplicação e rodei o puppet, isso me permitiu desativar a VM antiga com segurança.

{% codeblock lang:puppet %}
node "apache.hacklab" {
        include apache
        include apache::mod::php
        include apache::mod::alias
        include apache::mod::rewrite
        include apache::mod::ssl
        include apache::mod::vhost_alias
}
{% endcodeblock %}

Eu coloquei neste exemplo as configurações que normalmente utilizo em meus ambientes, agora vamos aplicar.

    puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Loading facts in /etc/puppet/modules/concat/lib/facter/concat_basedir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/puppet_vardir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/root_home.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/pe_version.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/concat_basedir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395824850'
Notice: /Stage[main]/Apache/Package[httpd]/ensure: created
Info: /Stage[main]/Apache/Package[httpd]: Scheduling refresh of Class[Apache::Service]
Notice: /Stage[main]/Apache::Mod::Ssl/File[ssl.conf]/ensure: defined content as '{md5}a3d9c9dac9163536b63c2801bdd43ea8'
Info: /Stage[main]/Apache::Mod::Ssl/File[ssl.conf]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Apache/File[/etc/httpd/conf/httpd.conf]/content:
...
...
...
...
Info: FileBucket got a duplicate file {md5}27a5c8d9e75351b08b8ca1171e8a0bbd
Info: /Stage[main]/Apache/File[/etc/httpd/conf/httpd.conf]: Filebucketed /etc/httpd/conf/httpd.conf to puppet with sum 27a5c8d9e75351b08b8ca1171e8a0bbd
Notice: /Stage[main]/Apache/File[/etc/httpd/conf/httpd.conf]/content: content changed '{md5}27a5c8d9e75351b08b8ca1171e8a0bbd' to '{md5}227489018cada97a86d93b5ffafd6d4b'
Info: /Stage[main]/Apache/File[/etc/httpd/conf/httpd.conf]: Scheduling refresh of Class[Apache::Service]
Notice: /Stage[main]/Apache::Mod::Php/Apache::Mod[php5]/Package[php]/ensure: created
Notice: /Stage[main]/Apache::Mod::Ssl/Apache::Mod[ssl]/Package[mod_ssl]/ensure: created
Info: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Listen 443]: Filebucketed /var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Listen 443 to puppet with sum 462e40c14c182d965ae6448e832ebb9a
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Listen 443]/ensure: removed
Info: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_NameVirtualHost *:443]: Filebucketed /var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_NameVirtualHost *:443 to puppet with sum 235a73c5a3abcbafdd73c00f8c639b89
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_NameVirtualHost *:443]/ensure: removed
Info: /var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments: Scheduling refresh of Exec[concat_/etc/httpd/conf/ports.conf]
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/Exec[concat_/etc/httpd/conf/ports.conf]/returns: executed successfully
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/Exec[concat_/etc/httpd/conf/ports.conf]: Triggered 'refresh' from 1 events
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]/content:
--- /etc/httpd/conf/ports.conf	2014-03-26 05:46:43.562816901 -0300
+++ /tmp/puppet-file20140326-3322-1da7azk-0	2014-03-26 06:08:06.286816834 -0300
@@ -3,7 +3,5 @@
 # Managed by Puppet
 # ************************************

-Listen 443
 Listen 80
-NameVirtualHost *:443
 NameVirtualHost *:80

Info: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]: Filebucketed /etc/httpd/conf/ports.conf to puppet with sum bc388698a3c11221869ded2030460038
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]/content: content changed '{md5}bc388698a3c11221869ded2030460038' to '{md5}ac3b658e433eba85f350d4068b0ef826'
Info: Concat[/etc/httpd/conf/ports.conf]: Scheduling refresh of Class[Apache::Service]
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-intranet.seusite.com.br.conf]: Filebucketed /etc/httpd/conf.d/25-intranet.seusite.com.br.conf to puppet with sum d03969fa47021eceeadeea38ef4805fb
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-intranet.seusite.com.br.conf]/ensure: removed
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-www.seusite.com.br.conf]: Filebucketed /etc/httpd/conf.d/25-www.seusite.com.br.conf to puppet with sum 230bc8bf6cd2bab7019c1c9cfa62df11
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-www.seusite.com.br.conf]/ensure: removed
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-blog.seusite.com.br.conf]: Filebucketed /etc/httpd/conf.d/25-blog.seusite.com.br.conf to puppet with sum 0d1676f1185addac23548ca2c36a8759
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-blog.seusite.com.br.conf]/ensure: removed
Info: FileBucket got a duplicate file {md5}a3d9c9dac9163536b63c2801bdd43ea8
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/ssl.conf.rpmsave]: Filebucketed /etc/httpd/conf.d/ssl.conf.rpmsave to puppet with sum a3d9c9dac9163536b63c2801bdd43ea8
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/ssl.conf.rpmsave]/ensure: removed
Info: FileBucket got a duplicate file {md5}98540d3009ecc6435d8770c24a71258a
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/welcome.conf]: Filebucketed /etc/httpd/conf.d/welcome.conf to puppet with sum 98540d3009ecc6435d8770c24a71258a
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/welcome.conf]/ensure: removed
Info: FileBucket got a duplicate file {md5}50015ff8bb23090aae4f8556987c5e40
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/php.conf]: Filebucketed /etc/httpd/conf.d/php.conf to puppet with sum 50015ff8bb23090aae4f8556987c5e40
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/php.conf]/ensure: removed
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-mobile.seusite.com.br.conf]: Filebucketed /etc/httpd/conf.d/25-mobile.seusite.com.br.conf to puppet with sum ed928ada5c97974fddc0bc4cbf268483
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/25-mobile.seusite.com.br.conf]/ensure: removed
Info: FileBucket got a duplicate file {md5}41f59421026a473a0378c58d539069c6
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/ssl.conf.rpmnew]: Filebucketed /etc/httpd/conf.d/ssl.conf.rpmnew to puppet with sum 41f59421026a473a0378c58d539069c6
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/ssl.conf.rpmnew]/ensure: removed
Info: FileBucket got a duplicate file {md5}e92bea7e9d70a9ecdc61edd7c0a2f59a
Info: /Stage[main]/Apache/File[/etc/httpd/conf.d/README]: Filebucketed /etc/httpd/conf.d/README to puppet with sum e92bea7e9d70a9ecdc61edd7c0a2f59a
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/README]/ensure: removed
Info: /etc/httpd/conf.d: Scheduling refresh of Class[Apache::Service]
Info: Class[Apache::Service]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Apache::Service/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
Info: /Stage[main]/Apache::Service/Service[httpd]: Unscheduling refresh on Service[httpd]
Notice: Finished catalog run in 30.00 seconds
```

Veja que o puppet instalou o apache, ajustou os arquivos de configuração e reiniciou o serviço, tudo em 30 segundos, agora podemos verificar se a porta 80 está como LISTEN.

    netstat -ntpl|grep :80
    tcp        0      0 :::80        :::*     LISTEN       2414/httpd
    
Podemos verificar se está respondendo

    curl localhost

Acompanhe a saída

{% codeblock retorno do acesso localhost lang:html %}

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /</title>
 </head>
 <body>
<h1>Index of /</h1>
<table><tr><th><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr><tr><th colspan="5"><hr></th></tr>
<tr><th colspan="5"><hr></th></tr>
</table>
</body></html>

{% endcodeblock %}

Veja que o site está respondendo perfeitamente. Vale a pena mencionar que o diretório apache, seja no Debian ou no CentOS, sofrerá uma mudança na organização  interna de diretórios e arquivos, isto é uma característica deste módulo, vai ficar diferente, mas tudo vai funcionar e seu servidor web estará devidamente automatizado.
 
### 5.2 Criando vhosts

Vamos criar um VHOST para cada um dos seguintes sites

    http://www.seusite.com.br
    http://blog.seusite.com.br
    http://mobile.seusite.com.br
    http://intranet.seusite.com.br

Vamos ao código

{% codeblock lang:puppet %}

node "apache.hacklab" {

	include apache
	include apache::mod::php
	include apache::mod::alias
	include apache::mod::rewrite
	include apache::mod::ssl
	include apache::mod::vhost_alias
   
	# criando vhost simples
   
	apache::vhost { 'www.seusite.com.br':
		port          => '80',
		docroot       => '/var/www/seusite',
	}

	# criando vhost especificando owners e options
	
	apache::vhost { 'blog.seusite.com.br':
		port          => '80',
		docroot       => '/var/www/blog',
		docroot_owner => 'apache',
		docroot_group => 'apache',
		options       => ['Indexes','FollowSymLinks','MultiViews'],
		override      => ['none'],
	}
 
	# criando um vhost especificando configurações personalizadas de php
   
	apache::vhost { 'mobile.seusite.com.br':
                port            => '80',
                docroot         => '/var/www/activesync',
                docroot_owner   => 'apache',
                docroot_group   => 'apache',
                options         =>  ['All'],
                override        =>  ['All'],
                custom_fragment => [
                        "php_flag magic_quotes_gpc off \n",
                        "php_flag register_globals off \n",
                        "php_flag magic_quotes_runtime off \n",
                        "php_flag short_open_tag on \n",
                        "Alias /Microsoft-Server-ActiveSync /var/www/activesync/index.php"
                ],
        }


	# criando vhost ssl usando certificados default
   
	apache::vhost { 'intranet.seusite.com.br':
		port          => '443',
		docroot       => '/var/www/intranet',
		docroot_owner => 'apache',
		docroot_group => 'apache',
		options       => ['all'],
		override      => ['all'],
		ssl           => true,
		# pode especificar um certificado se quiser
		#ssl_cert => '/etc/ssl/intranet.cert',
		#ssl_key  => '/etc/ssl/intranet.key',
	}
}

{% endcodeblock %}

Cödigo no lugar, agora vamos rodar o puppet

    puppet agent -t

Acompanhe a saída

```
Info: Retrieving plugin
Info: Loading facts in /etc/puppet/modules/concat/lib/facter/concat_basedir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/puppet_vardir.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/root_home.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/pe_version.rb
Info: Loading facts in /etc/puppet/modules/stdlib/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Loading facts in /var/lib/puppet/lib/facter/concat_basedir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Caching catalog for puppet.hacklab
Info: Applying configuration version '1395825015'
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[intranet.seusite.com.br]/Apache::Listen[443]/Concat::Fragment[Listen 443]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Listen 443]/ensure: created
Info: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[intranet.seusite.com.br]/Apache::Listen[443]/Concat::Fragment[Listen 443]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Listen 443]: Scheduling refresh of Exec[concat_/etc/httpd/conf/ports.conf]
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[intranet.seusite.com.br]/Apache::Namevirtualhost[*:443]/Concat::Fragment[NameVirtualHost *:443]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_NameVirtualHost *:443]/ensure: created
Info: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[intranet.seusite.com.br]/Apache::Namevirtualhost[*:443]/Concat::Fragment[NameVirtualHost *:443]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_NameVirtualHost *:443]: Scheduling refresh of Exec[concat_/etc/httpd/conf/ports.conf]
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/Exec[concat_/etc/httpd/conf/ports.conf]/returns: executed successfully
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/Exec[concat_/etc/httpd/conf/ports.conf]: Triggered 'refresh' from 2 events
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]/content:
--- /etc/httpd/conf/ports.conf	2014-03-26 06:08:06.382816833 -0300
+++ /tmp/puppet-file20140326-3642-1h8p4wc-0	2014-03-26 06:10:34.465816827 -0300
@@ -3,5 +3,7 @@
 # Managed by Puppet
 # ************************************

+Listen 443
 Listen 80
+NameVirtualHost *:443
 NameVirtualHost *:80

Info: FileBucket got a duplicate file {md5}ac3b658e433eba85f350d4068b0ef826
Info: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]: Filebucketed /etc/httpd/conf/ports.conf to puppet with sum ac3b658e433eba85f350d4068b0ef826
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]/content: content changed '{md5}ac3b658e433eba85f350d4068b0ef826' to '{md5}bc388698a3c11221869ded2030460038'
Info: Concat[/etc/httpd/conf/ports.conf]: Scheduling refresh of Class[Apache::Service]
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[intranet.seusite.com.br]/File[25-intranet.seusite.com.br.conf]/ensure: created
Info: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[intranet.seusite.com.br]/File[25-intranet.seusite.com.br.conf]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[mobile.seusite.com.br]/File[25-mobile.seusite.com.br.conf]/ensure: created
Info: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[mobile.seusite.com.br]/File[25-mobile.seusite.com.br.conf]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[blog.seusite.com.br]/File[25-blog.seusite.com.br.conf]/ensure: created
Info: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[blog.seusite.com.br]/File[25-blog.seusite.com.br.conf]: Scheduling refresh of Service[httpd]
Info: Class[Apache::Service]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[www.seusite.com.br]/File[25-www.seusite.com.br.conf]/ensure: created
Info: /Stage[main]/Main/Node[puppet.hacklab]/Apache::Vhost[www.seusite.com.br]/File[25-www.seusite.com.br.conf]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Apache::Service/Service[httpd]: Triggered 'refresh' from 5 events
Notice: Finished catalog run in 10.81 seconds
```

Em menos de 11 segundos ele fez a configuração de 4 vhosts em nosso servidor, agora você só precisa colocar os arquivos de aplicação no diretório que foi declarado em seus vhosts, afinal o servidor já está pronto para rodando.

## 6. Amarrando as pontas

Neste post eu abordei a instalação do apache e a criação de vhosts simples, mas esse módulo tem muito, muito mais recursos, leia o README para ver, conhecer e entender seus vastos recursos.

O uso do Puppet torna a administração do seu servidor web um processo limpo, organizado, rápido e eficiente, vale a pena usá-lo.

## 7. Referências

* https://github.com/puppetlabs/puppetlabs-apache
* http://forge.puppetlabs.com/puppetlabs
* http://docs.puppetlabs.com/puppet/latest/reference/modules_fundamentals.html


