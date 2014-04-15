---
layout: post
title: "Ativando ActiveSync em Zimbra com Z-PUSH"
date: 2014-01-25 12:07
comments: true
published: true
toc: true
categories: zimbra
---

Em um de meus clientes, instalei o Zimbra 7.x versão Community, essa versão funciona muito bem, é uma ferramenta excelente, particularmente a melhor suite de correio livre - IMHO, contudo, ela não possui dois recursos importantes, o primeiro é o sistema granular de BKP de caixas postais, e o segundo é o sistema de acesso mobile através do protocolo ActiveSync. Estes dois recursos só estão presentes na versão paga da ferramenta.

Sem o ActiveSync só é possível configurar o recebimento e envio de mensagens via IMAP e SMTP em seu dispositivo móvel, a sincronização de agenda, contatos, notas e tarefas não está disponível.

Este post pretende abordar como habilitar o ActiveSync no Zimbra através do projeto [Z-PUSH](http://z-push.sourceforge.net/soswp/) + [ZimbraBackend](http://sourceforge.net/projects/zimbrabackend/).

## 1. Requisitos

* Você precisa criar um domínio para acesso externo, ex: mobile.seudominio.com.br
    
* Você precisa de um servidor de APP WEB com suporte a aplicações PHP já que o Z-PUSH foi escrito nesta linguagem.

* O seu servidor WEB precisa enxergar o servidor Zimbra.
 
* Recomenda-se utilização de conexão segura SSL neste serviço.

### 1.1 Versões

* Z-PUSH versão 2.1.1

* ZimbraBackend release 58.

* CentOS 6.5 - 64 bits

* Apache HTTPd 2.2.x como servidor de APP WEB.

* PHP 5.3.x


## 2. Instalando e configurando HTTPd e PHP

    # yum install httpd php5 php-process
    
### 2.1 Configurando VHOST

Crie o arquivo abaixo:

    /etc/httpd/conf.d/mobile.seudominio.com.br.conf

Ajuste o vhost conforme suas necessidades, veja o exemplo abaixo:

```
<VirtualHost *:80>
  ServerName mobile.seudominio.com.br
  Alias /Microsoft-Server-ActiveSync /srv/activesync/index.php
  DocumentRoot /srv/activesync
  <Directory /srv/activesync>
    Options All
    AllowOverride All
    Order allow,deny
    Allow from all
  </Directory>
  ErrorLog /var/log/httpd/mobile.seudominio.com.br_error.log
  ServerSignature Off
  CustomLog /var/log/httpd/mobile.seudominio.com.br_access.log combined
  php_flag magic_quotes_gpc off
  php_flag register_globals off
  php_flag magic_quotes_runtime off
  php_flag short_open_tag on
</VirtualHost>
```

Observe que estou usando HTTP puro, contudo, recomendo uso de HTTPS.

### 2.2 Criando diretórios

    # mkdir /srv/activesync
    # mkdir /srv/activesync/state
    # mkdir /var/log/z-push

## 3. Instalando Z-PUSH

Acesse o diretório tmp

    # cd /tmp

Faça o download do projeto

    # wget http://zarafa-deutschland.de/z-push-download/final/2.1/z-push-2.1.1-1788.tar.gz
    
Descompacte o projeto    
    
    # tar zxvf z-push-2.1.1-1788.tar.gz
    
Acesse o diretório do projeto

    # cd z-push-2.1.1-1788

Copie os arquivos para o diretório documentroot configurado no vhost

    # mv * /srv/activesync/

## 4. Instalando Backend Zimbra no Z-PUSH

Acesse o diretório tmp

    # cd /tmp

Faça download do ZimbraBackend no site do projeto

    # wget http://ufpr.dl.sourceforge.net/project/zimbrabackend/Release58/zimbra58.tgz     
    
Descompacte o projeto    
    
    # tar zxvf zimbra58.tgz
    
Acesse o diretório do projeto

    # cd zimbra58
    
Mova o diretório z-push-2 para /srv/activesync/backend/

    # mv z-push-2 /srv/activesync/backend/zimbra

## 5. Configurando

### 5.1 Permissões

Ajuste owner, group e as permissões de arquivos e diretórios

    # chown -R apache:apache /srv/activesync
    # chown -R apache:apache /var/log/z-push
    # find /srv/activesync -type f -exec chmod 664 {} \;
    # find /srv/activesync -type d -exec chmod 775 {} \;
    
### 5.2 Z-PUSH

Edite o arquivo /srv/activesync/config.php e ajuste as linhas abaixo

    define('TIMEZONE', 'America/Sao_Paulo');
    define('STATE_DIR', '/srv/activesync/state/');
    define('BACKEND_PROVIDER', 'BackendZimbra');

### 5.3 Zimbra Backend

Edite o arquivo /srv/activesync/backend/zimba/config.php e ajuste as linhas abaixo

     define('ZIMBRA_URL', 'https://ip-ou-dns-do-zimbra');
     define('ZIMBRA_USER_DIR', 'zimbra');
     define('ZIMBRA_SYNC_CONTACT_PICTURES', true);
     define('ZIMBRA_VIRTUAL_CONTACTS',true);
     define('ZIMBRA_VIRTUAL_APPOINTMENTS',true);
     define('ZIMBRA_VIRTUAL_TASKS',true);
     define('ZIMBRA_IGNORE_EMAILED_CONTACTS',true);
     define('ZIMBRA_HTML',true);
     define('ZIMBRA_TIMEZONE', 'America/Sao_Paulo');
     define('ZIMBRA_ENFORCE_VALID_EMAIL', true);
     define('ZIMBRA_SMART_FOLDERS',true);
     define('ZIMBRA_RETRIES_ON_HOST_CONNECT_ERROR',5);

## 6. Testando

Agora acesse o endereço abaixo para testar a autenticação

    http://mobile.seudominio.com.br/Microsoft-Server-ActiveSync

Após autenticar você ira cair em uma página com os seguintes dados

```
Z-Push - Open Source ActiveSync

Version 2.1.1-1788
GET not supported

This is the Z-Push location and can only be accessed by Microsoft ActiveSync-capable devices

More information about Z-Push can be found at:
Z-Push homepage
Z-Push download page at BerliOS
Z-Push Bugtracker and Roadmap

All modifications to this sourcecode must be published and returned to the community.

Please see AGPLv3 License for details.
```
Se essa tela aparecer, sua autenticação funcionou, agora você pode partir para configuração do seu IOS ou Android, basta adicionar uma conta EXCHANGE preenchendo os dados corretamente.

## 8. Verificando logs

Você pode acompanhar sua conexão pelos logs.

    # tail -f /var/log/z-push/z-push.log /var/log/z-push/z-push-error.log

## 9. Amarrando as pontas

Os passos são simples e o Z-PUSH implementa muito bem o ActiveSync. O mais interessante disso tudo é que você poderá desabilitar o IMAP e SMTP externo uma vez que você poderá enviar e receber mensagens utilizando ACTIVESYNC, além disto, sua agenda, contatos, notas e tarefas estarão disponíveis para sincronização em seu dispositivo móvel.

## 7. Referências

* http://www.bluhm-de.com/installing-z-push-for-zimbra-8-on-centos-6
* http://aubreykloppers.wordpress.com/2012/10/16/z-push-and-zimbrabackend-the-zimbra-companion-for-mobility/
* http://b3n.org/setup-your-own-activesync-server-with-zimbra-and-z-push/
* http://www.andreabalboni.com/cms/microsoft-active-sync-vmware-zimbra-8-z-push-backend-zimbra/
* http://blog.iwayvietnam.com/tuanta/2013/03/12/setup-z-push-with-zimbra-to-synchronize-emails-calendar-and-contacts-with-mobile-devices/

No próximo post irei abordar como fazer o BKP granular de caixas postais com alguns scripts.

[s]<br>
Guto
