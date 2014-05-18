---
layout: post
title: "Vagrant Multi Machines"
date: 2014-05-12 10:07
comments: true
toc: true
categories: vagrant
---

## 1. Conceito Multi Machines

Após a introdução ao [vagrant](http://gutocarvalho.net/octopress/2014/05/09/entenda-o-vagrant/) no post anterior, vamos falar rapidamente sobre multi machines. 

O conceito é simples, com "multi machines" você pode subir várias VMs de uma vez só. 

Imagine um ambiente LAMP, você pode ter uma VM para o Apache e outra para o MYSQL, tudo no mesmo projeto/diretório/vagranfile.

### 1.1 Características do ambiente

VM1

```
VM1 (apache)
Base box CentOS 6.5 64Bits
1 Processador
600 MB de RAM
NIC 1 em modo NAT
NIC 2 c/ ip 192.168.200.30, rede privada
Encaminhamento de porta 80 do guest paraporta 9090 no host (osx)
```

VM2

```
VM2 (mysql)
Base box CentOS 6.5 64Bits
1 Processador
600 MB de RAM
NIC 1 em modo NAT
NIC 2 c/ ip 192.168.200.31, rede privada
Encaminhamento de porta 3306 do guest paraporta 9091 no host (osx)
```

## 2. Criando Multi Machines

Deixo claro que eu estou partindo da ideia que você já leu o post anterior e já configurou o vagrant em seu ambiente, então vamos lá

    mkdir ~/vagrant/projects/lamp
    cd ~/vagrant/projects/lamp
    vim Vagrantinit

Vamos traduzir as características da seção 1.1 para o Vagrantfile

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

  end
  # End Mysql

end
```

Agora vamos iniciar as duas VMs

    vagrant up

Acompanhe a saída

```
==> apache: Importing base box 'centos6x64'...
==> apache: Matching MAC address for NAT networking...
==> apache: Setting the name of the VM: lamp_apache_1399901503605_21761
==> apache: Fixed port collision for 22 => 2222. Now on port 2200.
==> apache: Clearing any previously set network interfaces...
==> apache: Preparing network interfaces based on configuration...
    apache: Adapter 1: nat
    apache: Adapter 2: hostonly
==> apache: Forwarding ports...
    apache: 22 => 2200 (adapter 1)
    apache: 80 => 9090 (adapter 1)
==> apache: Running 'pre-boot' VM customizations...
==> apache: Booting VM...
==> apache: Waiting for machine to boot. This may take a few minutes...
    apache: SSH address: 127.0.0.1:2200
    apache: SSH username: vagrant
    apache: SSH auth method: private key
    apache: Warning: Connection timeout. Retrying...
    apache: Warning: Connection timeout. Retrying...
    apache: Warning: Remote connection disconnect. Retrying...
==> apache: Machine booted and ready!
==> apache: Checking for guest additions in VM...
==> apache: Setting hostname...
==> apache: Configuring and enabling network interfaces...
==> apache: Mounting shared folders...
    apache: /vagrant => /Users/gutocarvalho/vagrant/projects/lamp
==> mysql: Importing base box 'centos6x64'...
==> mysql: Matching MAC address for NAT networking...
==> mysql: Setting the name of the VM: lamp_mysql_1399901623062_42651
==> mysql: Fixed port collision for 22 => 2200. Now on port 2201.
==> mysql: Clearing any previously set network interfaces...
==> mysql: Preparing network interfaces based on configuration...
    mysql: Adapter 1: nat
    mysql: Adapter 2: hostonly
==> mysql: Forwarding ports...
    mysql: 22 => 2201 (adapter 1)
    mysql: 3306 => 9091 (adapter 1)
==> mysql: Running 'pre-boot' VM customizations...
==> mysql: Booting VM...
==> mysql: Waiting for machine to boot. This may take a few minutes...
    mysql: SSH address: 127.0.0.1:2201
    mysql: SSH username: vagrant
    mysql: SSH auth method: private key
    mysql: Warning: Connection timeout. Retrying...
    mysql: Warning: Connection timeout. Retrying...
    mysql: Warning: Remote connection disconnect. Retrying...
    mysql: Warning: Remote connection disconnect. Retrying...
==> mysql: Machine booted and ready!
==> mysql: Checking for guest additions in VM...
==> mysql: Setting hostname...
==> mysql: Configuring and enabling network interfaces...
==> mysql: Mounting shared folders...
    mysql: /vagrant => /Users/gutocarvalho/vagrant/projects/lamp
```

Em cerca de 4 minutos as duas VMs subiram, vamos verificar o status das VMs

    vagrant status

Acompanhe a saída

```
Current machine states:

apache                    running (virtualbox)
mysql                     running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

Perfeito, estão funcionando :)

## 3. Manipulando Multi Machines

Subindo todas as VMs

    vagrant up
    
Subindo apenas a VM apache

    vagrant up apache
    
Parando todas as VMs
 
    vagrant halt
    
Parando apenas a VM mysql

     vagrant mysql halt
     
Recarregando todas as VMs

     vagrant reload
     
Recarregando apenas a VM apache

     vagrant reload apache
     
Você pode usar expressões regulares se quiser ( no caso de várias vms )

     vagrant up /apache[0-9]/
     
Para entrar na VM mysql

     vagrant ssh mysql
     
Para entrar na VM apache

     vagrant ssh apache

É bastante simples!

## 4. Dicas soltas

### 4.1 Primary machine

Observe a ordem em que as máquinas subiram, foi a mesma ordem declarada no Vagranfile, contudo, você pode definir uma VM que deve subir antes das outras, isto é chamado de **primary**, veja o exemplo de sintaxe abaixo

```
config.vm.define "mysql" primary: true do |web|
  code
end
```

Basta especificar isto e ela subirá primeiro que as outras.

### 4.2 Autostart

Se quiser que uma das VMs não inicie automaticamente, especifique isso no define dela

    config.vm.define "db_slave", autostart: false

Feito isto, o vagrant up vai ignorar ela.

## 5. Multi Machines com diferentes boxes

É possível usar diferentes tipos de box, basta especificar, eu tenho aqui um projeto que sobe um puppetmaster em centos e mais 3 vms com agentes, um debian, um ubuntu e um centos, funciona muito, muito bem.

## 6. Conclusão

Quanto mais estudo o vagrant, melhor entendo o seu funcionamento e acabo conhecendo novas e excelentes características como esta. 

Com multi machines, construir ambientes complexos passa a ser fácil e rápido, imagina na hora que eu integrar com o o Puppet :)

Será o mesmo que juntar o queijo e a goiabada, perfeição. 

## 7. Referências

* https://docs.vagrantup.com/v2/multi-machine/
