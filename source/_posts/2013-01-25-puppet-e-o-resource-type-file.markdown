---
layout: post
title: "Puppet Resource Type File e Múltiplos Sources"
date: 2013-01-25 20:47
comments: true
categories: puppet
---


Há alguns dias um amigo (Elcimar Freitas) me disse que havia criado uma classe usando o modelo em trio, porém ele queria que certas máquinas recebessem arquivos personalizados, em um primeiro momento ele estava pensando em usar condicionais ou case para resolver o problema, o que é realmente possível, porém, não é muito prático, acabei dando uma dica que aprendi com o mago @dscobral - vulgo gandalf. Aqui neste post vamos resolver o problema dentro do recurso File, acompanhe a explicação e a solução.

### Estudando o problema

Veja abaixo um exemplo típico de configuração em trio:

{% codeblock lang:puppet %}

class ntp {

	package { 'ntp':
		ensure => present
	}

	file { '/etc/ntp.conf':
		ensure  => present,
		mode    => 644,
		owner   => root,
		group   => root,
		require => Package['ntp'],
		notify  => Service['ntp'],
		source  => "puppet:///files/ntp/ntp.conf"
	}
	
	service { 'ntp':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['/etc/ntp.conf'],
	}

}

{% endcodeblock %}

Poderíamos utilizar tratamento condicional para enviar diferentes arquivos para diferentes nodes, veja um exemplo abaixo:


{% codeblock lang:puppet %}
class ntp {

	if $fqdn == 'servidor1.hacklab' {
		$ntpfile = "ntp.conf.servidor1"
	}
	elsif $fqdn == "servidor2.hacklab" {
		$ntpfile = "ntp.conf.servidor2"
	}
	else {
		$ntpfile = "ntp.conf"
	}

	package { 'ntp':
		ensure => present
	}

	file { '/etc/ntp.conf':
		ensure  => present,
		mode    => 644,
		owner   => root,
		group   => root,
		require => Package['ntp'],
		notify  => Service['ntp'],
		source  => "puppet:///files/ntp/${ntpfile}"
	}
	
	service { 'ntp':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['/etc/ntp.conf'],
	}

}
{% endcodeblock %}


Ou então poderíamos utilizar o case para tratar a mesma questão de forma diferente, veja o exemplo abaixo:


{% codeblock lang:puppet %}
class ntp {

	case $fdqn {
		servidor1.hacklab: { $ntpfile = "ntp.conf.servidor1" }
		servidor2.hacklab: { $ntpfile = "ntp.conf.servidor2" }
		default:	   { $ntpfile = "ntp.conf" }
	}
    
 	package { 'ntp':
		ensure => present
	}

	file { '/etc/ntp.conf':
		ensure  => present,
		mode    => 644,
		owner   => root,
		group   => root,
		require => Package['ntp'],
		notify  => Service['ntp'],
		source  => "puppet:///files/ntp/${ntpfile}"
	}
	
	service { 'ntp':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['/etc/ntp.conf'],
	}
 
}
{% endcodeblock %}


Daria até para usar seletores, abaixo mais um exemplo:


{% codeblock lang:puppet %}
class ntp {
    
 	package { 'ntp':
		ensure => present
	}

	file { 'ntp.conf':
		ensure  => present,
		mode    => 644,
		owner   => root,
		group   => root,
		require => Package['ntp'],
		notify  => Service['ntp'],
		source    => $fqdn ? {
			'sevidor1.hacklab'   => 'puppet:///files/ntp/ntp.conf.servidor1',
			'servidor2.hacklab'  => 'puppet:///files/ntp/ntp.conf.servidor1',
			 default             => 'puppet:///files/ntp/ntp.conf',
		},
	
	}
	
	service { 'ntp':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['ntp.conf'],
	}
 
}

{% endcodeblock %}


### Solucionando o problema

Porém nos três casos teríamos que escrever muitas linhas caso fosse necessário ter arquivos diferentes para vários servidores, seria repetitivo e cansativo, e nós utilizamos o puppet justamente para evitar o trabalho repetitivo.

Se lermos com cuidado o manual de resource types, ou se você tiver a sorte de ter um gandalf (@dcsobral) fazendo mentoria de puppet contigo, você poderá encontrar a solução dentro do resource type file, veja como fica a solução mais elegante - na minha opinião.

{% codeblock lang:puppet %}
class ntp {
    
 	package { 'ntp':
		ensure => present
	}

	file { 'ntp.conf':
		ensure  => present,
		mode    => 644,
		owner   => root,
		group   => root,
		require => Package['ntp'],
		notify  => Service['ntp'],
		source  => [
                   "puppet:///files/ntp/ntp.conf.${fqdn}",
                   "puppet:///files/ntp/ntp.conf",
                   ],
	}
	
	service { 'ntp':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['ntp.conf'],
	}
 
}

{% endcodeblock %}

Observe que dentro de source eu declarei que se houver um arquivo chamado ntp.conf.nomedamaquina.dominio, ele vai usar esse arquivo, do contrário, se não existir algum arquivo com $fqdn no nome, ele passa para a próxima opção que seria o arquivo ntp.conf padrão.

Com este tipo de configuração podemos ter dezenas de arquivos de configuração no diretório /etc/puppet/files/ntp, cada um terminando com o nome da máquina a que se destina, isso seria entendido pelo source e processado na ordem determinada.

Eu chamo isso de múltiplos sources, veja mais um exemplo retirado do site da puppetlabs:

{% codeblock lang:puppet %}

file { "/etc/nfs.conf":
  source => [
    "puppet:///modules/nfs/conf.$host",
    "puppet:///modules/nfs/conf.$operatingsystem",
    "puppet:///modules/nfs/conf"
  ]
}
{% endcodeblock %}

No exemplo acima podem existir arquivos terminando com o nome da máquina, sistema operacional ou então a opção final é o arquivo padrão.

Dá para ver que existem muitas formas de se fazer a mesma coisa, o desafio é encontrar a mais eficiente para sua necessidade.

Eu normalmente uso CASE e SELETORES quando tenho que tratar nomes - diferentes - de pacotes em distros linux distintas, ou mesmo PATH para um arquivo de configuração que é diferente no Debian e no CENTOS, nestes casos são muito úteis de fato.

### Referências

* http://docs.puppetlabs.com
* http://gutocarvalho.net/dokuwiki/doku.php?id=puppet_serverless#case

[s]<br>
Guto
