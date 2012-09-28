---
layout: post
title: "Puppet: comparando a sintaxe com Salt"
date: 2012-09-28 10:38
comments: true
categories: tecnologia
---

Estive estudando a sintaxe do Salt, abaixo mostro como declarar um pacote, um serviço e um arquivo na DSL do Puppet e na SLS (YAML) do Salt, apenas com objetivo de comparar suas diferenças.


### Declarando no Puppet

{% codeblock lang:puppet %}

package { "openssh-server":
	ensure  => installed
}
	
service { "ssh":
	ensure  => running,
	enable  => true,
	require => Package["openssh-sever"]
}
	
file { "/etc/ssg/sshd_config":
	mode    => 644,
	owner   => root,
	group   => root,
	source  => "puppet:///files/httpd.conf",
	require => Package["openssh-server"]
	notify  => Service["ssh"]
}

{% endcodeblock %}


### Declarando no Salt


{% codeblock lang:python %}

openssh-server:
   pkg.installed

 sshd:
   service.running:
     - require:
       - pkg: openssh-client
       - pkg: openssh-server
       - file: /etc/ssh/sshd_config

 /etc/ssh/sshd_config:
   file.managed:
     - user: root
     - group: root
     - mode: 644
     - source: salt://ssh/sshd_config
     - require:
       - pkg: openssh-server
     - watch:
     	- file: /etc/ssh/sshd_config
     	
{% endcodeblock %}

Veja que é bem parecido, o watch do salt faz a mesma coisa que o notify do puppet, caso o arquivo sofra alguma modificação ele faz um resfresh no serviço.

E sigo estudando.

[s]<br>
Guto
