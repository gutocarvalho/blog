---
layout: post
title: "Puppet: Criando módulo Saltstack"
date: 2012-09-28 19:40
comments: true
categories: tecnologia
---

Eu já mostrei como instalar o Salt manualmente, agora vou compartilhar com vocês o módulo que criei para instalá-lo via puppet em todo o nosso parque de servidores.

### Criando o módulo

crie o diretório

    mkdir saltstack

entre no diretório

    cd saltstack

crie o subdir files

    mkdir files

crie o subdir manifests

    mkdir manifests

entre no subdir manifests

    cd manifests

crie o subdir classes

    mkdir classes

entre no subdir classes

    cd classes

crie o arquivo salt-master.pp

    vim salt-master.pp

insira os seguinte conteúdo 

{% codeblock lang:puppet %}

class salt-master {

	if $lsbdistcodename == 'squeeze' {
	
		package { "salt-master":
			ensure => present,
		}

		file { "/etc/salt/master":
			ensure  => present,
			mode    => 644,
			owner   => root,
			group   => root,
			source  => "puppet:///salt/master",
			require => Package['salt-master'],
			notify  => Service['salt-master'],
		}

		service { "salt-master":
			ensure     => running,
			enable     => true,
			hasstatus  => true,
			hasrestart => true,
			require    => [ Package['salt-master'], File["/etc/salt/master"] ],
		}
	}
}

{% endcodeblock %}

crie o arquivo salt-minion.pp com o conteúdo abaixo

{% codeblock lang:puppet %}

class salt-minion {

	if $lsbdistcodename == 'squeeze' {

		package { "salt-minion":
			ensure => present,
		}

		file { "/etc/salt/minion":
			ensure  => present,
			mode    => 644,
			owner   => root,
			group   => root,
			source  => "puppet:///salt/minion",
			require => Package['salt-minion'],
			notify  => Service['salt-minion'],
		}

		service { "salt-minion":
			ensure     => running,
			enable     => true,
			hasstatus  => true,
			hasrestart => true,
			require    => [ Package['salt-minion'], File["/etc/salt/minion"] ],
		}
	}
}

{% endcodeblock %}

suba um nível

    cd ..
    
crie o arquivo init.pp

    vim init.pp
    
insira o conteúdo abaixo

    import "classes/*.pp"
    
coloque os arquivos de configuração master e minion no diretório files.

### Declarando módulo no site.pp

Após a construção do módulo precisamos de alguma forma carregá-lo, após isto poderemos incluir suas classes nas configurações dos nodes.

O arquivo /etc/puppet/manifests/site.pp é o arquivo em que você precisa carregar os módulos, neste arquivo você poderá colocar uma linha como a abaixo:

    import "saltstack"
    
Isso será suficiente para o carregamento do módulo.

#### Declarando classes em nodes

Declarando a classe salt-master ao node master

{% codeblock lang:puppet %}

node "master" {
    include salt-master
}

{% endcodeblock %}

Declarando a classe salt-minion ao node minion

{% codeblock lang:puppet %}

node "master" {
    include salt-master
}

{% endcodeblock %}

#### Amarrando as pontas

Este módulo é simples, porém é o suficiente para instalarmos o servidor salt-master e também o salt-minion em nossos nodes, com isso poderemos executar comandos nos nodes em tempo real, algo que faz muita falta no puppet.
