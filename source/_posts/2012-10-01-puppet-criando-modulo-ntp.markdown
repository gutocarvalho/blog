---
layout: post
title: "Puppet: Criando módulo NTP"
date: 2012-10-01 12:09
comments: true
categories: tecnologia
---

Hoje trago um exemplo de criação de módulo no Puppet com tratamento de condicionais e fatos, espero que os novos entusiastas consigam enxergar com mais
clareza como fazer esse tipo de tratamento.

## 1. requisitos

- Verificar se a distribuição é Debian, Ubuntu, RHEL ou CENTOS
- Declarar as variáveis $package_name, $service_name e $conf_file de acordo com o SO
- Instalar o pacote NTP
- Empurrar o arquivo de configuração NTP
  - Só fazer isto se o pacote estiver instalado
  - O arquivo deve pertencer ao usuário root, grupo root e ter permissão 644
- Declarar que o serviço NTP deve estar rodando e habilitado no boot
  - Só fazer isso se o pacote estiver instalado e se o arquivo de configuração estiver presente

### mão-na-massa

crie o diretório ntp

    mkdir ntp

entre no diretório

    cd ntp

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

crie o arquivo ntp.pp

    vim ntp.pp 

insira os seguinte conteúdo

{% codeblock lang:puppet %}
class ntp {
 
	# tratando distribuições
 
	case $operatingsystem {
        	CentOS,RedHat: { 
			$package_name = 'ntp'
	        	$service_name = 'ntpd'
	        	$conf_file    = 'ntp.conf.el'
		}
		Debian,Ubuntu: { 
			$package_name = 'ntp'
			$service_name = 'ntp'
			$conf_file    = 'ntp.conf.debian'
		}
	}

	# declarando o pacote
 
	package { "$package_name":
		ensure => present,
	}
 
	# declarando o arquivo de configuração
 
	file { '/etc/ntp.conf':
		ensure  => present,
		source  => "puppet:///ntp/${conf_file}",
		notify  => Service["$service_name"],
		require => Package["$package_name"],
	}
 
	# declarando o serviço
 
	service { "$service_name":
		ensure     => running,
		enable     => true,
		hasrestart => true,
		hasstatus  => true,
		require    => [ Package['ntp'], File['/etc/ntp.conf' ], ],
	}
 
}
{% endcodeblock %}

salve o arquivo

    :wq

retorne ao diretório manifests

    cd ..

crie o arquivo init.pp

    vim init.pp

edite arquivo

    vim init.pp

insira a linha abaixo

    import "classes/*.pp"

salve o arquivo

    :wq

crie os arquivos **ntp.conf.el** e **ntp.conf.debian** no diretório files. Fazendo isto, nosso módulo estará pronto para testarmos.

## 2. testando módulo

partindo do pressuposto de que o módulo já foi carregado no arquivo site.pp, siga e declare esse módulo em algum node, veja o exemplo abaixo:

```
node xpto {
    include ntp
}
```

no node, execute o puppet

    puppet agent --test

acompanhe a saída

```
info: Caching catalog for puppetmaster.hacklab
info: Applying configuration version '1349105440'
notice: /Stage[main]/Ntp/Package[ntp]/ensure: ensure changed 'purged' to 'present'
notice: /Stage[main]/Ntp/File[/etc/ntp.conf]/content: 
--- /etc/ntp.conf	2010-10-17 11:35:26.000000000 -0200
+++ /tmp/puppet-file20121001-2558-1l2emqx-0	2012-10-01 12:33:20.000000000 -0300
@@ -1,55 +1,16 @@
-# /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help
-
 driftfile /var/lib/ntp/ntp.drift
-
-
-# Enable this if you want statistics to be logged.
-#statsdir /var/log/ntpstats/
-
 statistics loopstats peerstats clockstats
+
 filegen loopstats file loopstats type day enable
 filegen peerstats file peerstats type day enable
 filegen clockstats file clockstats type day enable
 
+server ntp.dominio iburst dynamic
 
-# You do need to talk to an NTP server or two (or three).
-#server ntp.your-provider.example
-
-# pool.ntp.org maps to about 1000 low-stratum NTP servers.  Your server will
-# pick a different set every time it starts up.  Please consider joining the
-# pool: <http://www.pool.ntp.org/join.html>
-server 0.debian.pool.ntp.org iburst
-server 1.debian.pool.ntp.org iburst
-server 2.debian.pool.ntp.org iburst
-server 3.debian.pool.ntp.org iburst
-
-
-# Access control configuration; see /usr/share/doc/ntp-doc/html/accopt.html for
-# details.  The web page <http://support.ntp.org/bin/view/Support/AccessRestrictions>
-# might also be helpful.
-#
-# Note that "restrict" applies to both servers and clients, so a configuration
-# that might be intended to block requests from certain clients could also end
-# up blocking replies from your own upstream servers.
-
-# By default, exchange time with everybody, but don't allow configuration.
 restrict -4 default kod notrap nomodify nopeer noquery
 restrict -6 default kod notrap nomodify nopeer noquery
-
-# Local users may interrogate the ntp server more closely.
 restrict 127.0.0.1
 restrict ::1
 
-# Clients from this (example!) subnet have unlimited access, but only if
-# cryptographically authenticated.
-#restrict 192.168.123.0 mask 255.255.255.0 notrust
-
-
-# If you want to provide time to your local subnet, change the next line.
-# (Again, the address is an example only.)
-#broadcast 192.168.123.255
-
-# If you want to listen to time broadcasts on your local subnet, de-comment the
-# next lines.  Please do this only if you trust everybody on the network!
-#disable auth
-#broadcastclient
+disable kernel
+logfile   /var/log/ntp.log

info: /Stage[main]/Ntp/File[/etc/ntp.conf]: Filebucketed /etc/ntp.conf to main with sum 3e250ecaf470e1d3a2b68edd5de46bfd
notice: /Stage[main]/Ntp/File[/etc/ntp.conf]/content: content changed '{md5}3e250ecaf470e1d3a2b68edd5de46bfd' to '{md5}b54a36abe675e96723eb2e5562195c91'
info: /Stage[main]/Ntp/File[/etc/ntp.conf]: Scheduling refresh of Service[ntp]
notice: /Stage[main]/Ntp/Service[ntp]: Triggered 'refresh' from 1 events
notice: Finished catalog run in 16.80 seconds

```

vamos conferir se o pacote está instalado

    root@puppetmaster:~# dpkg --list|grep ntp
    ii  ntp                                 1:4.2.6.p2+dfsg-1+b1         Network Time Protocol daemon and utility programs

vamos conferir se o arquivo está com as permissões declaradas

    root@puppetmaster:~# ls -la /etc/ntp.conf 
    -rw-r--r-- 1 root root 446 Oct  1 12:33 /etc/ntp.conf

vamos conferir se o serviço está rodando

    root@puppetmaster:~# /etc/init.d/ntp status
    NTP server is running.
   
bacana, fez tudo que o foi declarado, com isto o módulo atendeu a todos os requisitos.

## 3. Amarrando as pontas

Espero que o exemplo lhe ajude a compreender melhor o tratamento de codicionais e o uso de fatos em um manifest.

Esse módulo está disponível no Github => [https://github.com/gutocarvalho/puppet-ntp](http://github.com/gutocarvalho/puppet-ntp)

[s]<br>
Guto
