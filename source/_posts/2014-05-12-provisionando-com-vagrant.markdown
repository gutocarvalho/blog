---
layout: post
title: "Provisionando com Vagrant"
date: 2014-05-12 15:44
comments: true
toc: true
categories: vagrant
---

## 1. Provisioners

Após a introdução ao [vagrant](http://gutocarvalho.net/octopress/2014/05/09/entenda-o-vagrant/) e uma rápida passagem pelo conceito de [multi machines](http://gutocarvalho.net/octopress/2014/05/12/vagrant-multi-machines/), vamos começar a entrar no provisionamento.

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

Eu vou abordar neste post o provisionamento **shell**.

### 1.1 Requisitos

É importante que você tenha executado os exemplos nos posts anteriores e criado o projeto lamp, este post é uma sequência e vai utilizar os arquivos e projetos criados anteriormente.

Entre no diretório do projeto lamp

    cd ~/vagrant/projects/lamp

## 2. Shell Provisioner

O provisionamento shell pode ser feito de duas formas, a primeira é o chamado INLINE e a segunda EXTERNAL SCRIPT.

### 2.1 Shell INLINE

Vamos fazer um exemplo simples em que invoco os comandos abaixo

    yum clean all
    yum install tcpdump -y

Traduzindo para a sintaxe do vagrant ficaria assim
    
    vm.provision :shell, :inline => "yum clean all;yum install tcpdump -y"

Agora vamos inserir no Vagrantfile do projeto 'lamp'.

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
    apache.vm.provision :shell, :inline => "yum clean all;yum install tcpdump -y"
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
    mysql.vm.provision :shell, :inline => "yum clean all;yum install tcpdump -y"
    mysql.vm.provider "virtualbox" do |y|
        y.customize [ "modifyvm", :id, "--cpus", "1" ]
        y.customize [ "modifyvm", :id, "--memory", "600" ]
    end

  end
  # End Mysql

end
```

Arquivo ajustado vamos fazer um teste em nosso ambiente, primeiro desligue as VMs

    cd ~/vagrant/projects/lamp
    vagrant halt

Verifique se as VMs desligaram
    
    vagrant status

Acompanhe a saída

```
Current machine states:

apache                    poweroff (virtualbox)
mysql                     poweroff (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
``` 
    
Agora vamos religar e verificar se o provisionamento ocorreu.

    vagrant up
  
Perceba que vamos receber as seguintes mensagens

```
==> apache: VM already provisioned. Run `vagrant provision` or use `--provision` to force it

==> mysql: VM already provisioned. Run `vagrant provision` or use `--provision` to force it
	
```

Essa mensagem diz que sua máquina já foi provisionada na primeira vez que rodamos vagrant up, é importante entender isto, o provisionamento só é executado quando a VM é criada, no momento do UP, depois o vagrant considera que a VM já foi devidamente provisionada.

Bom, caso queiramos executar novamente as tarefas de provisionamento, principalmente se elas foram adicionadas após o processo de criação da VM, teremos que ser mais explícitos, então vamos digitar

    vagrant provision
    
Acompanhe a saída

```
==> apache: Running provisioner: shell...
    apache: Running: inline script
Loaded plugins: fastestmirror, security
Cleaning repos: base extras puppetlabs-deps puppetlabs-products updates
Cleaning up Everything
Cleaning up list of fastest mirrors
Loaded plugins: fastestmirror, security
Determining fastest mirrors
 * base: mirror.globo.com
 * extras: mirror.globo.com
 * updates: mirror.globo.com
Setting up Install Process
Package 14:tcpdump-4.0.0-3.20090921gitdf3cb4.2.el6.x86_64 already installed and latest version
Nothing to do
==> mysql: Running provisioner: shell...
    mysql: Running: inline script
Loaded plugins: fastestmirror, security
Cleaning repos: base extras puppetlabs-deps puppetlabs-products updates
Cleaning up Everything
Loaded plugins: fastestmirror, security
Determining fastest mirrors
 * base: mirror.globo.com
 * extras: mirror.globo.com
 * updates: mirror.globo.com
Setting up Install Process
Package 14:tcpdump-4.0.0-3.20090921gitdf3cb4.2.el6.x86_64 already installed and latest version
Nothing to do
```

Veja que o provisionamento foi realizado em ambas as VMs e o pacote solicitado já havia sido instalado corretamente, portanto, nada foi realizado.

Em relação a esse provisioner, sua criatividade é o limite, você pode intalar pacotes, criar usuários, arquivos, diretórios, links, alterar configurações, fazer um deploy de aplicações e até instalar sua ferramenta preferida de gerência de configurações ( puppet :).

Podemos usar constantes para colocar mais comandos no velho estilo EOF, veja o exemplo abaixo

```
$script = <<SCRIPT
echo "Estou provisionando via inline shell"
yum clean all
yum install tcpdump -y
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: $script
end
```
Essa forma também é bastante prática, porém seu Vagrantfile pode ficar um pouco poluído.

### 2.2 External Script

Para evitar poluição do Vagrantfile podemos construir scripts mais complexos do lado de fora em um arquivo externo.

crie o arquivo users.sh dentro do diretorio do projeto

    cd ~/vagrant/projects/lamp
    vim users.sh

Coloque o conteúdo abaixo

    #!/bin/bash
    useradd gutocarvalho
    
Agora vamos ver a sintaxe para uso do external script

    apache.vm.provision :shell, :path => "users.sh"
    mysql.vm.provision :shell, :path => "users.sh"

Veja como fica o arquivo completo

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
    apache.vm.provision :shell, :path => "users.sh"
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
    mysql.vm.network "private_network", ip: "192.168.200.31
    mysql.vm.provision :shell, :path => "users.sh"
    mysql.vm.provider "virtualbox" do |y|
        y.customize [ "modifyvm", :id, "--cpus", "1" ]
        y.customize [ "modifyvm", :id, "--memory", "600" ]
    end

  end
  # End Mysql

end
```

E agora vamos rodar o provisionamento

    vagrant provision

Acompanhe a saída

```
==> apache: Running provisioner: shell...
    apache: Running: /var/folders/ty/7b7lzgzd5wl_4hgncfgctf3r0000gn/T/vagrant-shell20140513-61540-1tolxu9
==> mysql: Running provisioner: shell...
    mysql: Running: /var/folders/ty/7b7lzgzd5wl_4hgncfgctf3r0000gn/T/vagrant-shell20140513-61540-jmxaov
```

O provisionamento aconteceu, agora vamos verificar se o usuário foi criado em ambas VMs.

```
[kaiten@gutocarvalho lamp]$ vagrant ssh apache
Last login: Tue May 13 04:16:37 2014 from 10.0.2.2
Welcome to your Packer-built virtual machine.
[vagrant@apache ~]$ id gutocarvalho
uid=501(gutocarvalho) gid=501(gutocarvalho) grupos=501(gutocarvalho)
[vagrant@apache ~]$ exit
logout
Connection to 127.0.0.1 closed.

[kaiten@gutocarvalho lamp]$ vagrant ssh mysql
Last login: Tue May 13 04:28:22 2014 from 10.0.2.2
Welcome to your Packer-built virtual machine.
[vagrant@mysql ~]$ id gutocarvalho
uid=501(gutocarvalho) gid=501(gutocarvalho) grupos=501(gutocarvalho)
[vagrant@mysql ~]$ exit
logout
Connection to 127.0.0.1 closed.
```

O usuário foi criado nas duas VMs.

Um dica rápida, é possível utilizar um arquivo remoto como external script, veja o exemplo abaixo

```
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "https://example.com/provisioner.sh"
end
```

## 6. Conclusão

O uso de provisionadores facilita a vida do sysadmin, o vasto suporte a diferentes tipos de provisionadores atende amplamente as mais diferentes modalidades de gerência de configurações, é um resurso imprescindível e deve ser utilizado para extrairmos o máximo do vagrant.

O próximo post será sobre o provisionador Puppet Apply, até lá ;)

## 7. Referências

https://docs.vagrantup.com/v2/provisioning/shell.html
