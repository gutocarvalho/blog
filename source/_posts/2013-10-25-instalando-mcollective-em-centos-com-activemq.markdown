---
layout: post
title: "Instalando Mcollective em CentOS com ActiveMQ"
date: 2013-10-25 10:41
comments: true
categories: 
published: false
---

Eu já publiquei um post sobre a instalação do mcollective no [Debian Squeeze](http://gutocarvalho.net/octopress/2013/02/07/instalando-mcollective-em-debian-squeeze-com-ambiente-puppet/) utilizando RabbitMQ, agora vou mostrar como fazê-lo no CentOS6 utilizando o ActiveMQ. Recomendo a leitura do post anterior que inclusive demonstra como utilizar diversos agentes do MCO.

## 1. Ambiente

Vamos trabalhar com pelo menos 2 VMs, a primeira vai ter o ActiveMQ (sistema de mensageria), mcollective e mcollective cliente.

VM 1 (ActiveMQ/Mcollective/Mcollective Client)

* CentOS 6.4 64 Bits
* 1 PROC
* 1 GB RAM

VM2 (Mcollective)

* CentOS 6.4 64 Bits
* 1 PROC
* 1 GB RAM

## 2. Configurando repositórios (ambas vms)

atualizei seu ambiente

    yum update -y

instale o vim e cliente ssh

    yum install vim wget openssh-clients

instale o repositório epel

    rpm -Uvh http://ftp-stud.hs-esslingen.de/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

instale o repositório puppet

    rpm -Uvh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-6.noarch.rpm

## 3. Configurando VM 1

### 3.1 ActiveMQ

##### 3.1.1 Instalando

yum install activemq

##### 3.1.2 Configurando

Edite a linha 109 do arquivo de configuração /etc/activemq/activemq.xml

    <authenticationUser username="mcollective" password="secret" groups="mcollective,admins,everyone"/>

Modifique secret para senha que deseja utilizar no mcollective, apenas isto, reincide o serviço após o suste

    service activemq restart

### 3.2 Configurando ActiveMQ

### Mcollective

    yum install mcollective

### 
