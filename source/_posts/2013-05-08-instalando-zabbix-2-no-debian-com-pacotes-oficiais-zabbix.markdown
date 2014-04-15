---
layout: post
title: "Instalando Zabbix 2 no Debian com pacotes oficiais Zabbix"
date: 2013-05-08 11:38
comments: true
categories: zabbix
---

Neste [post](http://gutocarvalho.net/octopress/2013/01/14/instalando-zabbix-2x-em-debian-squeeze) eu já mostrei como instalar o zabbix 2 compilando, agora
vou mostrar como instalar o zabbix 2 utilizando os pacotes oficiais da zabbix.org para debian.

## 1. Instalando chave

### 1.1 Adicionando chave manualmente

Faça o download da chave e adicionei ao seu chaveiro.

    wget http://repo.zabbix.com/zabbix-official-repo.key -O - | apt-key add -

### 1.2 Adicionando chave via pacote

Faça o download do pacote release

    wget http://repo.zabbix.com/zabbix/2.0/debian/pool/main/z/zabbix-release/zabbix-release_2.0-1squeeze_all.deb

Instale o pacote

    dpkg -i zabbix-release_2.0-1squeeze_all.deb

Esse pacote além de instalar a chave, já adiciona o repositório, portato se optar por ele, pule o passo 2.

## 2. Configurando repositório

Adicione a linha abaixo ao sources.list do seu squeeze

    deb http://repo.zabbix.com/zabbix/2.0/debian squeeze main

Atualize os índices de pacotes

    aptitude update

Pronto, já é possível encontrar os pacotes em uma pesquisa

    aptitude search zabbix

Acompanhe a saída

```
p   zabbix-agent                                      - network monitoring solution - agent
p   zabbix-frontend-php                               - network monitoring solution - PHP front-end
p   zabbix-get                                        - network monitoring solution - get
p   zabbix-java-gateway                               - network monitoring solution - java-gateway
p   zabbix-proxy-mysql                                - network monitoring solution - proxy (using MySQL)
p   zabbix-proxy-pgsql                                - network monitoring solution - proxy (using PostgreSQL)
p   zabbix-proxy-sqlite3                              - network monitoring solution - proxy (using SQLite3)
p   zabbix-release                                    - Zabbix official repository configuration
p   zabbix-sender                                     - network monitoring solution - sender
p   zabbix-server-mysql                               - network monitoring solution - server (using MySQL)
p   zabbix-server-pgsql                               - network monitoring solution - server (using PostgreSQL)
```

E podemos verificar a versão

    aptitude show zabbix-agent|grep ^Version

Acompanhe a saída

    Version: 1:2.0.6-1

Pacotes na última versão

## 3. Instalando o Zabbix Server

Acompanhe duas formas de instalar o zabbix-server

### 3.1 Zabbix Server com backend pgsql
  
Instalando o zabbix server com banco de dados postgresql
 
    aptitude install zabbix-server-pgsql

### 3.1 Zabbix Server com Backend mysql

Instalando o zabbix server com banco de dados mysql
	
    aptitude install zabbix-server-mysql

## 4. Instalando Zabbix Agent
	
Instalando o agente de monitoramento do zabbix

    aptitude install zabbix-agent

## 5. Instalando Zabbix Java Gateway

Instalando o gateway de monitoramento de aplicações JAVA e máquina virtual java.

    aptitude install zabbix-java-gateway

## 6. Instalando Zabbix Proxy

Acompanhe 3 formas de instalar o Zabbix Proxy.

## 6.1 Proxy com backend mysql
    
Instalando zabbix proxy com banco de dados mysql

    aptitude install zabbix-proxy-mysql

## 6.2 Proxy com backend pgsql

Instalando zabbix proxy com banco de dados pgsql
    
    aptitude install zabbix-proxy-pgsql

## 6.3 Proxy com backend sqlite3

Instalando zabbix proxy com banco de dados sqlite3
    
    aptitude install zabbix-proxy-sqlite3

## 7.Instalando Frontend PHP

Instalando frontend web para configuração e visualização de dados de monitoramento
	
    aptitude install zabbix-frontend-php

## 8. Instalando Get & Sender

Instalando ferramentas de consulta ao agente (get) e envio de dados (sender) ao servidor.

	aptitude install zabbix-get
	aptitude install zabbix-sender

Utilizar pacotes facilita e muito a administração de seu ambiente, principalmente
em caso de atualização, usar os pacotes mantidos pela zabbix.org é certeza de ter
sempre as últimas versões do Zabbix empacotadas.

Se precisa de agilidade use os pacotes/binários, não pense duas vezes.

[s]
Guto
