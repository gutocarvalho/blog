---
layout: post
title: "Vagrant com Provisioner Puppet Apply"
date: 2014-05-13 09:45
comments: true
toc: true
categories: [ puppet, vagrant ]
---

## 1. Provisioners

Após a introdução ao [vagrant](http://gutocarvalho.net/octopress/2014/05/09/entenda-o-vagrant/) fizemos uma rápida passagem pelo conceito de [multi machines](http://gutocarvalho.net/octopress/2014/05/12/vagrant-multi-machines/) para então entrarmos em [provisionamento](http://gutocarvalho.net/octopress/2014/05/12/provisionando-com-vagrant/), começamos com provisionamento em [shell](http://gutocarvalho.net/octopress/2014/05/12/provisionando-com-vagrant/) e agora vamos falar de Puppet e Vagrant.

Como eu já disse a vocês, o vagrant tem suporte a diversos provisionadores, isto significa que você pode automatizar a instalação e configuração de sua VM, tornando isto parte do processo **vagrant up**.

O vagrant atualmente suporta os seguintes provisionadores

* File
* Shell
* Ansible
* CFEngine
* Chef Solo
* Chef Client
* Docker
* Puppet Apply
* Puppet Agent
* Salt

Eu vou abordar neste post o provisionamento **Puppet Apply**.

### 1.1 Requisitos

É importante que você tenha executado os exemplos nos posts anteriores e criado o projeto lamp, este post é uma sequência e vai utilizar os arquivos e projetos criados anteriormente.

É importante que você tenha o puppet instalado em suas VMs, como eu utilizei a VM da Puppetlabs não precisarei instalar, mas verifique e se preciso instale nas VMS.

Entre no diretório do projeto lamp

    cd ~/vagrant/projects/lamp

Vamos em frente

## 2. Puppet Apply

Basicamente temos duas formas de usar o Puppet, vamos ver a primeira forma que não necessita de um puppet master, com o comando apply podemos aplicar nossos manifests localmente.

### 2.1 Ambiente e estrutura

Por padrão o vagrant vai procurar os manifests do puppet no diretório manifests dentro do diretório do projeto, em nosso caso seria

    /~/vagrant/projects/lamp/manifests

Então para começarmos, vamos criar esse diretório

    mkdir -p /~/vagrant/projects/lamp/manifest
    
Outro detalhe interessante é que ele vai procurar por padrão o arquivo **default.pp** no diretório manifests. Caso queira utilizar outro diretório ou outro arquivo padrão basta especificar o path e arquivo através das diretivas **manifest_file** e **manifest_path**.

### 2.2 Declarando provisioner

Abaixo a sintaxe de configuração do provisioner

```
Vagrant.configure("2") do |config|
  config.vm.provision "puppet"
end
```

A configuração acima é suficiente para aplicar algo a sua VM, mas isso só vai acontecer se o diretório manifest e arquivo default.pp existirem.

Vamos criar o arquivo default.pp com o conteúdo abaixo

```
class utils {

	package { 'iptraf':
		ensure => present,
	}

}

include utils

```   

E agora vamos adicionar o provisioner ao Vagrantfile do projeto lamp

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Apache
  config.vm.define "apache" do |apache|

    apache.vm.hostname = "apache.hacklab"
    apache.vm.box = "centos6x64"
    apache.vm.network "forwarded_port", guest: 80, host: 9090
    apache.vm.network "private_network", ip: "192.168.200.30"
    
    apache.vm.provision "puppet"
   
    apache.vm.provider "virtualbox" do |v|
        v.customize [ "modifyvm", :id, "--cpus", "1" ]
        v.customize [ "modifyvm", :id, "--memory", "600" ]
    end

  end
  # End apache

  # Mysql
  config.vm.define "mysql" do |mysql|

    mysql.vm.hostname = "mysql.hacklab"
    mysql.vm.box = "centos6x64"
    mysql.vm.network "forwarded_port", guest: 3306, host: 9091
    mysql.vm.network "private_network", ip: "192.168.200.31"
    
    mysql.vm.provision "puppet"
   
    mysql.vm.provider "virtualbox" do |y|
        y.customize [ "modifyvm", :id, "--cpus", "1" ]
        y.customize [ "modifyvm", :id, "--memory", "600" ]
    end

  end
  # End Mysql

end
```

Feito o ajuste no Vagrantfile vamos rodar o provision

    vagrant provision
   
Acompanhe a saída   

```
==> apache: Running provisioner: puppet...
Shared folders that Puppet requires are missing on the virtual machine.
This is usually due to configuration changing after already booting the
machine. The fix is to run a `vagrant reload` so that the proper shared
folders will be prepared and mounted on the VM.
```

Veja que o vagrant não conseguiu provisionar, isto ocorre pois não havia diretório manifest ou arquivo default.pp quando a VM foi iniciada, então ele recomenda um 'reload' para reiniciar suas VMs, vamos fazer isto

    vagrant reload
   
Durante o reload vamos receber a seguinte mensagem

```
==> apache: VM already provisioned. Run `vagrant provision` or use `--provision` to force it
==> mysql: VM already provisioned. Run `vagrant provision` or use `--provision` to force it
```

Essa mensagem já é conhecida, vimos no post anterior, ela nos diz que sua máquina já foi provisionada na primeira vez que rodamos o comando **vagrant up**, é importante entender isto, o provisionamento só é executado quando a VM é criada, no momento do UP, depois o vagrant considera que a VM já foi devidamente provisionada.

Bom, caso queiramos executar novamente as tarefas de provisionamento, principalmente se elas foram adicionadas após o processo de criação da VM, teremos que ser mais explícitos, então vamos digitar

    vagrant provision

Acompanhe a saída

```
==> apache: Running provisioner: puppet...
Running Puppet with default.pp...
Notice: Compiled catalog for apache.hacklab in environment production in 1.32 seconds
Notice: /Stage[main]/Utils/Package[iptraf]/ensure: created
Notice: Finished catalog run in 17.60 seconds
==> mysql: Running provisioner: puppet...
Running Puppet with default.pp...
Notice: Compiled catalog for mysql.hacklab in environment production in 1.83 seconds
Notice: /Stage[main]/Utils/Package[iptraf]/ensure: created
Notice: Finished catalog run in 16.68 seconds
```

Maravilha, o pacote iptraf foi provisionado com sucesso em ambas VMs, vamos seguir estudando.

### 2.2 Personalizando

Eu prefiro criar um diretório puppet na raiz do meu projeto e dentro dele o diretório manifests, além disto, eu prefiro usar o arquivo **site.pp** como default, para fazer estes pequenos ajustes a sintaxe no Vagrantfile vai ficar assim

```
vm.provision "puppet" do |puppet|
		puppet.manifests_path = "puppet/manifests"
		puppet.manifest_file  = "site.pp"
end
```

Vamos ajustar a estrutura de diretórios e arquivos

     mkdir ~/vagrant/projects/lamp/puppet
     cd ~/vagrant/projects/lamp
     mv manifests puppet
     cd puppet/manifests
     mv default.pp site.pp
   
Vamos ajustar o Vagrantfile do projeto lamp

```
# -*- mode: ruby -*-

# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Apache
  config.vm.define "apache" do |apache|

    apache.vm.hostname = "apache.hacklab"
    apache.vm.box = "centos6x64"
 
    apache.vm.network "forwarded_port", guest: 80, host: 9090
    apache.vm.network "private_network", ip: "192.168.200.30"
 
    apache.vm.provider "virtualbox" do |v|
        v.customize [ "modifyvm", :id, "--cpus", "1" ]
        v.customize [ "modifyvm", :id, "--memory", "600" ]
    end

    apache.vm.provision "puppet" do |puppet|
	    puppet.manifests_path = "puppet/manifests"
	    puppet.manifest_file  = "site.pp"
    end

  end
  # End apache

  # Mysql
  config.vm.define "mysql" do |mysql|

    mysql.vm.hostname = "mysql.hacklab"
    mysql.vm.box = "centos6x64"

    mysql.vm.network "forwarded_port", guest: 3306, host: 9091
    mysql.vm.network "private_network", ip: "192.168.200.31"

    mysql.vm.provider "virtualbox" do |y|
        y.customize [ "modifyvm", :id, "--cpus", "1" ]
        y.customize [ "modifyvm", :id, "--memory", "600" ]
    end

    mysql.vm.provision "puppet" do |puppet|
	    puppet.manifests_path = "puppet/manifests"
	    puppet.manifest_file  = "site.pp"
    end
  end
  # End Mysql

end
```
    
Vamos dar um reload para o vagrant ler os novos diretórios

    vagrant reload

E quando as máquinas retornarem vamos invocar o provision

     vagrant provision

Acompanhe a saída

```
==> apache: Running provisioner: puppet...
Running Puppet with site.pp...
Notice: Compiled catalog for apache.hacklab in environment production in 2.10 seconds
Notice: Finished catalog run in 1.12 seconds
==> mysql: Running provisioner: puppet...
Running Puppet with site.pp...
Notice: Compiled catalog for mysql.hacklab in environment production in 2.02 seconds
Notice: Finished catalog run in 1.28 seconds
```

Veja que apesar de termos invocado o puppet nenhuma configuração foi aplicada, isto ocorreu graças a característica idempotente do puppet, o pacote já estava instalado, como o estado do sistema já é o esperado, nada foi feito ;)

### 2.3 Classificando Nodes

Até agora tudo funcionou perfeitamente, mas se observamos com cuidado, veremos que as duas VMs estão recebendo a mesma configuração do puppet.

Se quisermos aplicar diferentes configurações em cada VM preciso proceder de outra maneira, neste caso específico temos duas formas de determinar o que será aplicado em cada uma delas.

#### 2.3.1 Manifest File

Uma das formas mais fáceis de resolver o problema é criar um manifest específico para aquela VM e chamá-lo diretamente através da diretiva **manifest_file**, veja o exemplo abaixo:

```
...
apache.vm.provision "puppet" do |puppet|
	puppet.manifests_path = "puppet/manifests"
	puppet.manifest_file  = "apache.pp"
end
...
...
...
mysql.vm.provision "puppet" do |puppet|
	puppet.manifests_path = "puppet/manifests"
	puppet.manifest_file  = "mysql.pp"
end
...
```

É uma forma simples e rápida de separar e determinar as configurações que serão aplicadas	 a cada VM.

#### 2.3.2 Node Name

Outra forma de resolver o problema é manter apenas um arquivo e declarar as configurações utilizando a definição node.
```
...
apache.vm.provision "puppet" do |puppet|
	puppet.manifests_path = "puppet/manifests"
	puppet.manifest_file  = "site.pp"
end
...
...
...
mysql.vm.provision "puppet" do |puppet|
	puppet.manifests_path = "puppet/manifests"
	puppet.manifest_file  = "site.pp"
end
...
```

Veja que usamos o mesmo arquivo para as duas VMs, contudo a forma de declarar as configurações irá mudar, veja o exemplo:

```
node "apache.hacklab" {

	bloco de código

}

node "mysql.hacklab" {

	bloco de código

}
```

É igual ao que fazemos no puppetmaster, só que faremos no esquema serverless, desta forma tudo fica bem organizado e talvez mais elegante, além de exigir menor esforço de edição do Vagrantfile em caso de um projeto multi-machines.

### 2.4 Trabalhando com módulos

É possível também trabalhar com módulos normalmente, só precisamos adicionar a diretiva **module_path** na configuração do provisioner puppet.

    puppet.module_path    = "puppet/modules"

A configuração do provisioner completa ficaria assim:

```
config.vm.provision "puppet" do |puppet|
	puppet.manifests_path = "puppet/manifests"
	puppet.module_path    = "puppet/modules"
	puppet.manifest_file  = "site.pp"
end
```

É necessário criar o diretório modules

    mkdir -p ~/vagrant/projects/lamp/puppet/modules
    
Para instalar módulos no diretório que você criou, use o comando puppet module

    puppet module install usuario/modulo --target-dir ~/vagrant/projects/lamp/puppet/modules
    
Dê um reload para ler o diretório modules

    vagrant reload    
    
O Vagrantfile atualizado ficaria assim

```
# -*- mode: ruby -*-

# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Apache
  config.vm.define "apache" do |apache|

    apache.vm.hostname = "apache.hacklab"
    apache.vm.box = "centos6x64"
 
    apache.vm.network "forwarded_port", guest: 80, host: 9090
    apache.vm.network "private_network", ip: "192.168.200.30"
 
    apache.vm.provider "virtualbox" do |v|
        v.customize [ "modifyvm", :id, "--cpus", "1" ]
        v.customize [ "modifyvm", :id, "--memory", "600" ]
    end

   apache.vm.provision "puppet" do |puppet|
		puppet.manifests_path = "puppet/manifests"
       puppet.module_path    = "puppet/modules"
       puppet.manifest_file  = "site.pp"
   end

  end
  # End apache

  # Mysql
  config.vm.define "mysql" do |mysql|

    mysql.vm.hostname = "mysql.hacklab"
    mysql.vm.box = "centos6x64"

    mysql.vm.network "forwarded_port", guest: 3306, host: 9091
    mysql.vm.network "private_network", ip: "192.168.200.31"

    mysql.vm.provider "virtualbox" do |y|
        y.customize [ "modifyvm", :id, "--cpus", "1" ]
        y.customize [ "modifyvm", :id, "--memory", "600" ]
    end

    mysql.vm.provision "puppet" do |puppet|
	    puppet.manifests_path = "puppet/manifests"
	    puppet.module_path    = "puppet/modules"
	    puppet.manifest_file  = "site.pp"
    end
  end
  # End Mysql

end
```

#### 2.4.1 Módulo puppetlabs-apache

Vamos instalar o módulo apache da puppetlabs e passar algumas configurações para VM apache.

    puppet module install puppetlabs/apache --target-dir ~/vagrant/projects/lamp/puppet/modules
    
Agora vamos editar o arquivo site.pp e declarar a classe apache e um virtualhost no node apache.hacklab.

```
class utils {

    package { 'iptraf':
        ensure => present,
    }

}

node "apache.hacklab" {

	include utils

	include apache

	apache::vhost { 'www.hacklab':
		port    => '80',
		docroot => '/var/www/html',
	}

}

node "mysql.hacklab" {

	include utils

}

```

Feito isto, rode o provision apenas na vm apache

     vagrant provision apache

Acompanhe a saída

```
==> apache: Running provisioner: puppet...
Running Puppet with site.pp...
Warning: Config file /etc/puppet/hiera.yaml not found, using Hiera defaults
Notice: Compiled catalog for apache.hacklab in environment production in 14.28 seconds
Notice: /Stage[main]/Concat::Setup/File[/var/lib/puppet/concat]/ensure: created
Notice: /Stage[main]/Concat::Setup/File[/var/lib/puppet/concat/bin]/ensure: created
Notice: /Stage[main]/Concat::Setup/File[/var/lib/puppet/concat/bin/concatfragments.sh]/ensure: defined content as '{md5}e7aaa4c45316eb97d2d88b57334c4060'
Notice: /Stage[main]/Apache::Mod::Mime/Package[mailcap]/ensure: created
Notice: /Stage[main]/Apache/Package[httpd]/ensure: created
Notice: /Stage[main]/Apache::Mod::Mime/Apache::Mod[mime]/File[mime.load]/ensure: defined content as '{md5}e36257b9efab01459141d423cae57c7c'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[expires]/File[expires.load]/ensure: defined content as '{md5}f0825bad1e470de86ffabeb86dcc5d95'
Notice: /Stage[main]/Apache::Mod::Dav/Apache::Mod[dav]/File[dav.load]/ensure: defined content as '{md5}588e496251838c4840c14b28b5aa7881'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authz_owner]/File[authz_owner.load]/ensure: defined content as '{md5}f30a9be1016df87f195449d9e02d1857'
Notice: /Stage[main]/Apache::Mod::Prefork/File[/etc/httpd/conf.d/prefork.conf]/ensure: defined content as '{md5}109c4f51dac10fc1b39373855e566d01'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authz_groupfile]/File[authz_groupfile.load]/ensure: defined content as '{md5}ae005a36b3ac8c20af36c434561c8a75'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authn_dbm]/File[authn_dbm.load]/ensure: defined content as '{md5}90ee8f8ef1a017cacadfda4225e10651'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authz_user]/File[authz_user.load]/ensure: defined content as '{md5}63594303ee808423679b1ea13dd5a784'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[log_config]/File[log_config.load]/ensure: defined content as '{md5}785d35cb285e190d589163b45263ca89'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[logio]/File[logio.load]/ensure: defined content as '{md5}084533c7a44e9129d0e6df952e2472b6'
Notice: /Stage[main]/Apache::Mod::Mime/File[mime.conf]/ensure: defined content as '{md5}2fa646fe615e44d137a5d629f868c107'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[version]/File[version.load]/ensure: defined content as '{md5}1c9243de22ace4dc8266442c48ae0c92'
Notice: /Stage[main]/Apache::Mod::Setenvif/File[setenvif.conf]/ensure: defined content as '{md5}c7ede4173da1915b7ec088201f030c28'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[actions]/File[actions.load]/ensure: defined content as '{md5}599866dfaf734f60f7e2d41ee8235515'
Notice: /Stage[main]/Apache::Mod::Deflate/File[deflate.conf]/ensure: defined content as '{md5}44d54f557a5612be8da04c49dd6da862'
Notice: /Stage[main]/Apache/File[/etc/httpd/conf/httpd.conf]/content: content changed '{md5}27a5c8d9e75351b08b8ca1171e8a0bbd' to '{md5}227b5a08e18b6086ae8b654e142fad67'
Notice: /Stage[main]/Apache::Mod::Negotiation/File[negotiation.conf]/ensure: defined content as '{md5}47284b5580b986a6ba32580b6ffb9fd7'
Notice: /Stage[main]/Apache::Mod::Alias/Apache::Mod[alias]/File[alias.load]/ensure: defined content as '{md5}3cf2fa309ccae4c29a4b875d0894cd79'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[env]/File[env.load]/ensure: defined content as '{md5}d74184d40d0ee24ba02626a188ee7e1a'
Notice: /Stage[main]/Apache::Mod::Negotiation/Apache::Mod[negotiation]/File[negotiation.load]/ensure: defined content as '{md5}d262ee6a5f20d9dd7f87770638dc2ccd'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authz_dbm]/File[authz_dbm.load]/ensure: defined content as '{md5}c1363277984d22f99b70f7dce8753b60'
Notice: /Stage[main]/Apache::Mod::Dir/File[dir.conf]/ensure: defined content as '{md5}c741d8ea840e6eb999d739eed47c69d7'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[usertrack]/File[usertrack.load]/ensure: defined content as '{md5}e95fbbf030fabec98b948f8dc217775c'
Notice: /Stage[main]/Apache::Mod::Vhost_alias/Apache::Mod[vhost_alias]/File[vhost_alias.load]/ensure: defined content as '{md5}eca907865997d50d5130497665c3f82e'
Notice: /Stage[main]/Apache::Mod::Setenvif/Apache::Mod[setenvif]/File[setenvif.load]/ensure: defined content as '{md5}ec6c99f7cc8e35bdbcf8028f652c9f6d'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[auth_basic]/File[auth_basic.load]/ensure: defined content as '{md5}494bcf4b843f7908675d663d8dc1bdc8'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authn_file]/File[authn_file.load]/ensure: defined content as '{md5}d41656680003d7b890267bb73621c60b'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authn_alias]/File[authn_alias.load]/ensure: defined content as '{md5}58fea0bcb12624191c07106c359d1134'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[ext_filter]/File[ext_filter.load]/ensure: defined content as '{md5}76d5e0ac3411a4be57ac33ebe2e52ac8'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[speling]/File[speling.load]/ensure: defined content as '{md5}f82e9e6b871a276c324c9eeffcec8a61'
Notice: /Stage[main]/Apache::Mod::Dir/Apache::Mod[dir]/File[dir.load]/ensure: defined content as '{md5}1bfb1c2a46d7351fc9eb47c659dee068'
Notice: /Stage[main]/Apache::Mod::Dav_fs/Apache::Mod[dav_fs]/File[dav_fs.load]/ensure: defined content as '{md5}2996277c73b1cd684a9a3111c355e0d3'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[include]/File[include.load]/ensure: defined content as '{md5}88095a914eedc3c2c184dd5d74c3954c'
Notice: /Stage[main]/Apache::Mod::Alias/File[alias.conf]/ensure: defined content as '{md5}7c1f62cc583b16885ce93d449df6a3ce'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authn_default]/File[authn_default.load]/ensure: defined content as '{md5}871572d647c4604e3f91c3bed92f496e'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[suexec]/File[suexec.load]/ensure: defined content as '{md5}c7d5c61c534ba423a79b0ae78ff9be35'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[substitute]/File[substitute.load]/ensure: defined content as '{md5}8077c34a71afcf41c8fc644830935915'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[auth_digest]/File[auth_digest.load]/ensure: defined content as '{md5}df9e85f8da0b239fe8e698ae7ead4f60'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authz_default]/File[authz_default.load]/ensure: defined content as '{md5}bda328d6f1ccfb2d51610953a9352eeb'
Notice: /Stage[main]/Apache::Mod::Deflate/Apache::Mod[deflate]/File[deflate.load]/ensure: defined content as '{md5}2d1a1afcae0c70557251829a8586eeaf'
Notice: /Stage[main]/Apache::Mod::Autoindex/Apache::Mod[autoindex]/File[autoindex.load]/ensure: defined content as '{md5}515cdf5b573e961a60d2931d39248648'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authn_anon]/File[authn_anon.load]/ensure: defined content as '{md5}bf57b94b5aec35476fc2a2dc3861f132'
Notice: /Stage[main]/Apache::Mod::Autoindex/File[autoindex.conf]/ensure: defined content as '{md5}2421a3c6df32c7e38c2a7a22afdf5728'
Notice: /Stage[main]/Apache::Mod::Cache/Apache::Mod[cache]/File[cache.load]/ensure: defined content as '{md5}01e4d392225b518a65b0f7d6c4e21d29'
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf]/ensure: created
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments.concat.out]/ensure: created
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments]/ensure: created
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments.concat]/ensure: created
Notice: /Stage[main]/Apache::Mod::Rewrite/Apache::Mod[rewrite]/File[rewrite.load]/ensure: defined content as '{md5}26e2683352fc1599f29573ff0d934e79'
Notice: /Stage[main]/Apache::Default_mods/Apache::Mod[authz_host]/File[authz_host.load]/ensure: defined content as '{md5}d1045f54d2798499ca0f030ca0eef920'
Notice: /Stage[main]/Apache::Mod::Mime_magic/File[mime_magic.conf]/ensure: defined content as '{md5}b258529b332429e2ff8344f726a95457'
Notice: /Stage[main]/Apache::Mod::Mime_magic/Apache::Mod[mime_magic]/File[mime_magic.load]/ensure: defined content as '{md5}cb8670bb2fb352aac7ebf3a85d52094c'
Notice: /Stage[main]/Apache/Apache::Vhost[default]/Apache::Listen[80]/Concat::Fragment[Listen 80]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Listen 80]/ensure: created
Notice: /Stage[main]/Apache/Apache::Vhost[default]/Apache::Namevirtualhost[*:80]/Concat::Fragment[NameVirtualHost *:80]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_NameVirtualHost *:80]/ensure: created
Notice: /Stage[main]/Apache/Concat::Fragment[Apache ports header]/File[/var/lib/puppet/concat/_etc_httpd_conf_ports.conf/fragments/10_Apache ports header]/ensure: created
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/Exec[concat_/etc/httpd/conf/ports.conf]/returns: executed successfully
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/Exec[concat_/etc/httpd/conf/ports.conf]: Triggered 'refresh' from 5 events
Notice: /Stage[main]/Apache/Concat[/etc/httpd/conf/ports.conf]/File[/etc/httpd/conf/ports.conf]/ensure: defined content as '{md5}ac3b658e433eba85f350d4068b0ef826'
Notice: /Stage[main]/Apache::Mod::Dav_fs/File[dav_fs.conf]/ensure: defined content as '{md5}899a57534f3d84efa81887ec93c90c9b'
Notice: /Stage[main]/Apache::Mod::Cgi/Apache::Mod[cgi]/File[cgi.load]/ensure: defined content as '{md5}ac20c5c5779b37ab06b480d6485a0881'
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/welcome.conf]/ensure: removed
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/README]/ensure: removed
Notice: /Stage[main]/Apache/Apache::Vhost[default]/File[15-default.conf]/ensure: created
Notice: /Stage[main]/Main/Node[apache.hacklab]/Apache::Vhost[www.hacklab]/File[25-www.hacklab.conf]/ensure: created
Notice: /Stage[main]/Apache::Service/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
Notice: Finished catalog run in 116.67 seconds
```

Nosso servidor apache foi provisionado utilizando o módulo apache, agora ele roda o servidor http papache e já tem o vhost www.hacklab configurado

```
Notice: /Stage[main]/Apache/Apache::Vhost[default]/File[15-default.conf]/ensure: created
Notice: /Stage[main]/Main/Node[apache.hacklab]/Apache::Vhost[www.hacklab]/File[25-www.hacklab.conf]/ensure: created
Notice: /Stage[main]/Apache::Service/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
Notice: Finished catalog run in 116.67 seconds
```
Maravilha, vamos fazer o mesmo para o servidor mysql.

#### 2.4.2 Módulo puppetlabs-mysql

Seguindo em frente, tal como fizemos no exemplo 2.4.1, vamos utilizar um módulo para provisionar o mysql-server na VM mysql, primeiro vamos instalar o módulo.

    puppet module install puppetlabs/mysql --target-dir ~/vagrant/projects/lamp/puppet/modules

Acompanhe a saída

```
Notice: Preparing to install into /Users/gutocarvalho/vagrant/projects/lamp/puppet/modules ...
Notice: Downloading from https://forge.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/Users/gutocarvalho/vagrant/projects/lamp/puppet/modules
└── puppetlabs-mysql (v2.2.3)
```

Reinicie as VMs para ler os novos diretórios

    vagrant reload

Agora vamos ajustar nosso site.pp e declarar algumas configurações para a VM mysql

```
class utils {

    package { 'iptraf':
        ensure => present,
    }

}

node "apache.hacklab" {

	include utils

	include apache

	apache::vhost { 'www.hacklab':
		port    => '80',
		docroot => '/var/www/html',
	}

}

node "mysql.hacklab" {

	include utils
	
	class { 'mysql::server':

		root_password    => 'suasenha',

		override_options => {
			'mysqld' => {
				'connect_timeout'                 => '60',
				'bind_address'                    => '0.0.0.0',
				'max_connections'                 => '100',
				'max_allowed_packet'              => '128M',
				'thread_cache_size'               => '16',
				'query_cache_size'                => '64M',
			}
		}
	}

	# site www

        mysql::db { 'prod_www':
                user     => 'www_user',
                password => 'senha_www',
                host     => 'www.hacklab',
                grant    => ['all'],
        }
}
```
Agora vamos provisionar a VM

    vagrant provision mysql
    
Acompanhe a saída

```
=> mysql: Running provisioner: puppet...
Running Puppet with site.pp...
Warning: Config file /etc/puppet/hiera.yaml not found, using Hiera defaults
Notice: Compiled catalog for mysql.hacklab in environment production in 6.31 seconds
Notice: /Stage[main]/Mysql::Client::Install/Package[mysql_client]/ensure: created
Notice: /Stage[main]/Mysql::Server::Install/Package[mysql-server]/ensure: created
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/mysql]/ensure: created
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/my.cnf]/content: content changed '{md5}8ace886bbe7e274448bc8bea16d3ead6' to '{md5}4f689e5e30c31ecbac3ae64179f74668'
Notice: /Stage[main]/Mysql::Server::Config/File[/etc/mysql/conf.d]/ensure: created
Notice: /Stage[main]/Mysql::Server::Service/Service[mysqld]/ensure: ensure changed 'stopped' to 'running'
Notice: /Stage[main]/Mysql::Server::Root_password/Mysql_user[root@localhost]/password_hash: defined 'password_hash' as '*686FAFB0D22504AC4005869A48377E6056C40F6A'
Notice: /Stage[main]/Mysql::Server::Root_password/File[/root/.my.cnf]/ensure: defined content as '{md5}c51fe9a952a98fdd64ad44e0f4b9e660'
Notice: /Stage[main]/Main/Node[mysql.hacklab]/Mysql::Db[prod_www]/Mysql_database[prod_www]/ensure: created
Notice: /Stage[main]/Main/Node[mysql.hacklab]/Mysql::Db[prod_www]/Mysql_user[www_user@www.hacklab]/ensure: created
Notice: /Stage[main]/Main/Node[mysql.hacklab]/Mysql::Db[prod_www]/Mysql_grant[www_user@www.hacklab/prod_www.*]/ensure: created
Notice: Finished catalog run in 99.14 seconds
```   

Maravilha, a VM mysql foi provisionada, o mysql-server foi instalado, o banco prod_www foi criado, o usuário www_user foi criado e um grant foi aplicado.

Mais uma vez utilizamos o Puppet e módulos para provisionar uma VM, com sucesso e agilidade.

#### 2.5. Compartilhando diretório Puppet em vários projetos

## 7. Perguntas frequentes

##### 7.1. Posso usar mais de um tipo de provisioner ao mesmo tempo? 

Pode sim, normalmente as pessoas usam o provisioner SSH para instalar seu gestor de configurações, por exemplo o puppet, e dali para frente utilizam o puppet para configurar a VM.

##### 7.2 Posso alterar o default.pp e rodar o provision sem necessidade de reload? 

Pode, só precisa de reload quando você acabou de criar um arquivo ou diretório, alterações em arquivos existentes são lidas e aplicadas sem necessidade de reload.

## 6. Conclusão

A relação entre vagrant e puppet é na minha opinião uma união muito equilibrada, conseguimos rapidamente configurar um ambiente e testar uma aplicação. O Puppet tem recursos muito interessantes e vai lhe ajudar a construir ambientes complexos de forma fácil e objetiva.

O próximo post será sobre o provisionador Puppet Agent, até lá ;)

## 7. Referências

https://docs.vagrantup.com/v2/provisioning/puppet_apply.html
