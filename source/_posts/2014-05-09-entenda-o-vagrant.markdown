---
layout: post
title: "Entenda o vagrant"
date: 2014-05-09 08:09
comments: true
toc: true
categories: vagrant
---

## 1. Sobre o Vagrant

O [Vagrant](http://www.vagrantup.com) é uma ferramenta que permite que criemos rapidamente ambientes virtuais para fazermos testes, desenvolvimento ou provisionmento de ambientes utilizando as soluções de virtualização mais comuns como o Virtualbox e o VMWare, sendo também compatível com os principais provedores cloud como AWS, Rackspace e Digitalocean. Além disto, tem suporte a várias tecnologias de provisionamento como Puppet, Chef, Salt, Ansible e CFEngine, com isto, você já cria e configura o ambiente em um único processo, é realmente mágico.

Outros aspecto bacana é que você tem a portabilidade de criar e recriar esses ambientes em qualquer lugar de forma simples e descomplicada bastando apenas ter a 'receita mágica' do vagrant.

## 2. Entendendo a necessidade

Desde 2010 eu conhecia o vagrant, mas confesso que não havia dado muita bola para o projeto, como eu usava o [ganeti](https://code.google.com/p/ganeti/) na época, para mim era muito prático criar e recriar vms baseadas em templates, no servidor da empresa, não havia a menor necessidade de fazer isso no meu ambiente local (mbp).

Contudo, nos últimos 2 anos tenho trabalhado intensamente com virtualização e implantação de ambientes Puppet, e a construção de módulos em ambiente controlado é muito importante. 

Eu vinha sofrendo no OSX tanto com o VirtualBox quanto com o VMWARE Fusion para clonar e reconfigurar VMs sempre que eu tinha que construir um novo módulo ou criar um ambiente para algum treinamento ou palestra. 

Era sempre chato no caso do VMWARE clonar (copiando o disco) uma VM, dar boot, corrigir problemas de interface de rede, hostname, dar boot de novo, configurar outras coisas, etc, dar outro boot, era um clico cansativo. No virtualbox idem, apesar de ter o clone a um clique de distância, o que já ajuda, ainda era necessário fazer vários ajustes bem manuais para ter o ambiente do jeito que eu queria, e nem preciso dizer que isso consumia muito, mas muito tempo mesmo, tempo que hoje é precisoso demais para perder com isso, tempo que posso utilizar com minha família por exemplo.

Para resolver esses problemas e mais alguns outros, depois de alguma insistência de alguns amigos, resolvi brincar com o vagrant. Posso dizer que a brincadeira foi longe e os resultados me impressionaram.

### 3. Instalando Vagrant

A instalação é descomplicada, basta entrar na área de [downloads](http://www.vagrantup.com/downloads.html) do site e escolher o pacote para o seu sistema operacional. Você vai encontrar pacotes para o CentOS, Ubuntu, Debian, MacOX e Windows. É possível instalar via [gem](http://docs.vagrantup.com/v2/installation/index.html) se preferir - sim o vagrant foi escrito em Ruby.

Depois de fazer o dowload acesse a [documentação](http://docs.vagrantup.com/v2/) que é bem clara e siga a risca.

Ter o virtualbox instalado é pré-requisito, de preferência a última versão.

#### 3.1 OSX

No meu caso eu peguei o pacote do OSX e instalei com bastante facilidade com alguns cliques.

### 4. Iniciando o uso do vagrant

Um vez instalado ele está disponível para uso, basta abrir o console e digitar

    vagrant

Acompanhe a saída
 
```  
Usage: vagrant [options] <command> [<args>]

    -v, --version                    Print the version and exit.
    -h, --help                       Print this help.

Common commands:
     box          manages boxes: installation, removal, etc.
     connect      connect to a remotely shared Vagrant environment
     destroy      stops and deletes all traces of the vagrant machine
     halt         stops the vagrant machine
     help         shows the help for a subcommand
     init         initializes a new Vagrant environment by creating a Vagrantfile
     login        log in to Vagrant Cloud
     package      packages a running vagrant environment into a box
     plugin       manages plugins: install, uninstall, update, etc.
     provision    provisions the vagrant machine
     reload       restarts vagrant machine, loads new Vagrantfile configuration
     resume       resume a suspended vagrant machine
     share        share your Vagrant environment with anyone in the world
     ssh          connects to machine via SSH
     ssh-config   outputs OpenSSH valid configuration to connect to the machine
     status       outputs status of the vagrant machine
     suspend      suspends the machine
     up           starts and provisions the vagrant environment

For help on any individual command run `vagrant COMMAND -h`

Additional subcommands are available, but are either more advanced
or not commonly used. To see all subcommands, run the command
`vagrant list-commands`.
```

#### 4.1 Diretório de trabalho

Eleja um diretório pra trabalhar, no meu caso eu criei um diretório chamado vagrant no diretório home do meu usuário e dentro dele um diretório chamado projects.

    mkdir -p ~/vagrant/projects
    
#### 4.2 Boxes

Ao invés de trabalharmos com templates, o vagrant usa o conceito chamado box, que são imagens de sistemas operacionais pré-configurados que vão servir como base para a construção de nossas vms.

Hoje existe o vagrant cloud que é um repositório de boxes interessante

    https://vagrantcloud.com
    
Além deste, existe um base pública de boxes no site abaixo

    http://www.vagrantbox.es

E está é a base de boxes públicas da Puppetlabs

    http://puppet-vagrant-boxes.puppetlabs.com
 
##### 4.2.1 Instalando um box via vagrant cloud

Primeiro procure a box que deseja na cloud

    https://vagrantcloud.com/discover/featured

Depois insira a box em seu ambiente vagrant

    vagrant box add puppetlabs/centos-6.5-32-puppet

E depois verifique se a box foi adicionada corretamente

    vagrant box list

Acompanhe a saída

    centos-6.5-32    (virtualbox, 0)
   
Maravilha, box adicionada e pronta para uso.

##### 4.2.2 Instalando uma box manualmente

Você pode adicionar boxes manualmente, eu particularmente gosto das boxes da puppetlabs por já virem com o puppet pré-configurado

    cd /tmp
    wget http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-puppet.box
 
Agora vamos adicionar a box
 
    vagrant box add /tmp/centos-65-x64-virtualbox-puppet.box --name centos6x64pl
    
E vamos verificar a box

    vagrant box list

Acompanhe a saída

    centos-6.5-32    (virtualbox, 0)
    centos6x64pl     (virtualbox, 0)
 
 Maravilha, adicionamos mais uma box com sucesso.   
 
#### 4.3 Criando a VM

Para criar uma nova VM, vamos basicamente criar um diretório para o projeto e iniciar o projeto.

    cd ~/vagrant/projects
    mkdir meuprojeto
    cd meuprojeto

Depois de criar o diretório precisamos rodar o comando init para criar o arquivo de configuração

    vagrant init

Acompanhe a saída

```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

Veja que o vagrant nos disse que criou um arquivo de configuração em nosso diretório, vamos olhar esse arquivo

    cat Vagrantfile

Acompanhe a saída

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "base"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # Enable provisioning with CFEngine. CFEngine Community packages are
  # automatically installed. For example, configure the host as a
  # policy server and optionally a policy file to run:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.am_policy_hub = true
  #   # cf.run_file = "motd.cf"
  # end
  #
  # You can also configure and bootstrap a client to an existing
  # policy server:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.policy_server_address = "10.0.2.15"
  # end

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file default.pp in the manifests_path directory.
  #
  # config.vm.provision "puppet" do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "site.pp"
  # end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { :mysql_password => "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"
end
```

É bem grande mas é bem simples, vamos retirar os comentários e preparar ele para usar a VM que acabamos de baixar.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

		config.vm.hostname = "suavm.seudominio"
		config.vm.box = "centos6x64pl"
		
		config.vm.provider "virtualbox" do |virtualbox|
			virtualbox.customize [ "modifyvm", :id, "--cpus", "1" ]
			virtualbox.customize [ "modifyvm", :id, "--memory", "600" ]
		end
end
```

Veja como ficou bem mais simples de entender, eu disse no arquivo que vou usar a box centos6x64pl, defini memória e quantidade de processadores, mais nada, no caso dessa box ela já vem com uma interface de rede configurada em modo nat.

#### 4.4 Iniciando a VM

Para iniciar a VM

    vagrant up
   
Agora aguarde ela carregar, a primeira vez ai demorar alguns minutos pois ele vai copiar o BOX para esse projeto, depois passa a ser bem rápido.

```
vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'centos6x64'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: meuprojeto_default_1399893568659_66343
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection timeout. Retrying...
    default: Warning: Connection timeout. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
==> default: Setting hostname...
==> default: Mounting shared folders...
    default: /vagrant => /Users/gutocarvalho/vagrant/projects/
```
No caso do meu macbookpro core2duo com disco mecanico, a vm levou 1.48 minutos para ser criada, agora vamos entrar nela.

    vagrant ssh

E você já está rodando sua vm.

```
[vagrant@meuprojeto ~]$ ifconfig
eth0      Link encap:Ethernet  Endereço de HW 08:00:27:73:BF:1C
          inet end.: 10.0.2.15  Bcast:10.0.2.255  Masc:255.255.255.0
          endereço inet6: fe80::a00:27ff:fe73:bf1c/64 Escopo:Link
          UP BROADCASTRUNNING MULTICAST  MTU:1500  Métrica:1
          RX packets:448 errors:0 dropped:0 overruns:0 frame:0
          TX packets:348 errors:0 dropped:0 overruns:0 carrier:0
          colisões:0 txqueuelen:1000
          RX bytes:47075 (45.9 KiB)  TX bytes:42215 (41.2 KiB)

lo        Link encap:Loopback Local
          inet end.: 127.0.0.1  Masc:255.0.0.0
          endereço inet6: ::1/128 Escopo:Máquina
          UP LOOPBACKRUNNING  MTU:16436  Métrica:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          colisões:0 txqueuelen:0
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

[vagrant@meuprojeto ~]$ cat /etc/issue
CentOS release 6.5 (Final)
Kernel \r on an \m

[vagrant@meuprojeto ~]$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root  8,4G  1,1G  6,9G  14% /
tmpfs                         289M     0  289M   0% /dev/shm
/dev/sda1                     485M   34M  426M   8% /boot
/vagrant                      931G  853G   78G  92% /vagrant

[vagrant@meuprojeto ~]$ free -m
             total       used       free     shared    buffers     cached
Mem:           576        109        466          0          9         45
-/+ buffers/cache:         55        521
Swap:          991          0        991

[vagrant@meuprojeto ~]$
```

Se quiser criar outro ambiente, basta criar outro diretório, iniciar o seu projeto e configurar o Vagrantfile.

#### 4.4 Manipulando a VM

Para iniciar a configuração de uma VM

    vagrant init

Para iniciar uma VM

     vagrant up
    
Para desligar a VM
    
    vagrant halt
    
Para recarregar uma VM (reboot)

     vagrant reload
     
Para suspender uma VM

     vagrant suspend
     
Para fazer resume de uma vm suspensa

     vagrant resume
 
Para verificar o status de uma VM

     vagrant status
     
Para acessar sua vm

     vagrant ssh
     
Para destruir a VM, apagando ela do seu disco

    vagrant destroy

Para outros comandos e ajuda

    vagrant help
    
## 5. Dicas de configuração

Abaixo uma configuração que cria uma máquina virtual baseada no box centos6x84, configura um encaminhamento de portas da porta 80 no guest (vm) para porta 8080 em meu host (osx), isto significa que eu poderei acessar a aplicação da VM como se estivesse rodando localmente na porta 8080. 

Outro detalhe legal é que montaremos o diretório /Users/gutocarvalho/Sites do host (osx) dentro da VM no destino /srv/sites, ou seja, meus arquivos podem ficar no meu host, sendo editados pela minha IDE de preferência estarão sendo servidos pelo Apache do meu guest e meus sites serão acessíveis pela porta 8080, sensacionalmente fácil e prático.

Independente do Synced Folder específico que está presente no arquivo, o vagrant por DEFAULT monta na VM a raiz do seu projeto, então se eu colocar um arquivo em /Users/gutocarvalho/vagrant/projects/meuprojeto/arquivo.txt (host) ele estará disponível no guest em /vagrant/arquivo.txt.

Além disto, crio uma segunda interface de rede, em modo rede privada com o endereço IP 192.168.200.20 para comunicação futura com outra máquina.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

		config.vm.hostname = "apache.hacklab"
		config.vm.box = "centos6x64"
		config.vm.network "forwarded_port", guest: 80, host: 8080
		config.vm.network "private_network", ip: "192.168.200.20"
		config.vm.synced_folder "/Users/gutocarvalho/vagrant/projects/meuprojeto", "/srv/sites"

		config.vm.provider "virtualbox" do |virtualbox|
			virtualbox.customize [ "modifyvm", :id, "--cpus", "1" ]
			virtualbox.customize [ "modifyvm", :id, "--memory", "600" ]
		end
end
```
Existem centenas de outros parâmetros que podemos utilizar, acesse a documentação e leia bastante => http://docs.vagrantup.com.

## 6. Conclusão

Começar a trabalhar com vagrant me possibilitou aumentar radicalmente a minha agilidade na criação, testes e desenvolvimento de qualquer coisa em ambiente virtualizado, é realmente uma ferramenta fantástica e simples, recomendo.

Nos próximos posts eu vou falar sobre configuração de Multi Machines e provisionamento SHELL e PUPPET ;)

Conforme eu for aprendendo vou compartilhando com vocês.

## 7. Referências

* http://docs.vagrantup.com/v2/installation/index.html
* http://docs.vagrantup.com/v2/getting-started/index.html
* http://docs.vagrantup.com/v2/networking/index.html
* http://docs.vagrantup.com/v2/networking/forwarded_ports.html
* http://docs.vagrantup.com/v2/networking/private_network.html
* http://docs.vagrantup.com/v2/synced-folders/
