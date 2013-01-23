---
layout: post
title: "Puppet: Configuração em Trio"
date: 2012-10-29 20:25
comments: true
categories: 
---

Existe uma estrutura que gosto de chamar de 'configuração em trio' que é a base para automatizar vários tipos de serviços comuns em ambientes linux/unix, eu costumo utilizar esse modelo em classes que crio dentro e fora de módulos, é eficiente e prático, basta combinar três resource types (trio) e dois meta-parâmetros, são eles:

Resource Types

- package
- service
- file

MetaParameters

- require
- notify

### 1. Exemplo Conceitual

Antes de um exemplo real, vamos primeiro entender um modelo conceitual.

#### 1.1 Requisitos do modelo conceitual

Se formos traduzir o código em requisitos teríamos uma lista como esta abaixo:

* Um pacote deve ser declarado como presente/instalado
* O serviço relacionado ao pacote deve estar habilitado
* O serviço relacionado ao pacote deve estar rodando
* O serviço relacionado ao pacote aceita comando restart em seu init script
* O serviço relacionado ao pacote aceita comando status em seu init script
* O serviço depende do objeto que declara o arquivo de configuração
* Um arquivo de configuração do serviço deve ser declarado como presente
* O arquivo de configuração depende do objeto que instala o pacote
* O arquivo deve pertencer ao usuário root
* O arquivo deve pertecer ao grupo root
* O arquivo deve ter permissão 644
* O arquivo de configuração deve notificar o serviço caso ocorra mudança na origem 

É uma lista densa e complicada, vamos traduzir isto para código puppet.

#### 1.2 Solução para modelo conceitual

{% codeblock lang:puppet %}

	# declarando pacote
    
	package { 'pacote':
        	ensure => installed,
	}
      
	# declarando arquivo de configuracao
 	
	file { 'pacote.conf':
		ensure  => present,
		path    => "/etc/pacote/pacote.conf",
		source  => "/root/puppet/pacote.conf",
		owner   => 'root',
		group   => 'root',
		mode    => 644,
		require => Package['pacote'],
		notify  => Service['servico'],
	}       	

	# declarando servico
 	
	service { 'servico':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['pacote.conf'],
	}
     
{% endcodeblock %}

Veja que a tradução dos requisitos em código oferece uma visualização e um entendimento bem mais claro.

### 2. Exemplo Real

Agora vamos para dois exemplos do mundo real, o primeiro acerca do serviço Postfix e o segundo acerca do serviço NTP. Perceba que a base é a mesma, fiz só alguns pequenos ajustes.

#### 2.1 Postfix

{% codeblock lang:puppet %}

	# declarando pacote

	package { 'postfix':
		ensure => present,
	}
 
	# declarando serviço
 	
	service { 'postfix':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['/etc/postfix/main.cf'],
	}
 
	# declarando arquivo de configuração
 	
	file { '/etc/postfix/main.cf':
		source  => '/root/puppet/main.cf',
		owner   => 'root',
		group   => 'root',
		mode    => 644,
		require => Package['postfix'],
		notify  => Service['postfix'],
	}
    
{% endcodeblock %}


#### 2.1 NTP

{% codeblock lang:puppet %}

	# declarando pacote
	
	package { 'ntp':
        	ensure => present,
	}
 
	# declarando serviço
 	
	service { 'ntp':
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => File['/etc/ntp.conf'],
	}
 	
	# declarando arquivo de configuração
 	
	file { '/etc/ntp.conf':
		source  => '/root/puppet/ntp.conf',
		owner   => 'root',
		group   => 'root',
		mode    => 644,
		require => Package['ntp'],
		notify  => Service['ntp'],
	}
    
{% endcodeblock %}

### 3. Conclusão

O modelo aplica-se a muitas demandas de automatização de serviços, e a partir dele podemos evoluir o código até atender todas das necessidades e requisitos.

Em caso de uso de puppet __cliente/servidor__ haveria uma pequena mudança no parâmetro source do resource type **file**.

Mudaríamos de

    source  => '/root/puppet/ntp.conf',

para 

    source  => 'puppet:///files/ntp/ntp.conf'
    
se estivermos criando uma classe simples com arquivos armazenados em /etc/puppet/files, agora se fosse um módulo, seria algo como

    source  => 'puppet:///modules/ntp/ntp.conf'

e o arquivo estaria provavelmente em /etc/puppet/modules/ntp/files.

Fica a dica para puppetizar suas configurações ;)

[s]<br>
Guto
