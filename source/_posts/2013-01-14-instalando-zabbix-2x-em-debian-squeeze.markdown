---
layout: post
title: "Instalando Zabbix 2.0.4 Server, Proxy, Frontend, JMX e Agent em Debian Squeeze"
date: 2013-01-14 15:27
comments: true
categories: zabbix, tecnologia
---

Neste post vou mostrar uma das muitas formas de instalar o Zabbix 2.0.4 em modo Server, Agent, Proxy e JMX Gateway.

## Entendendo alguns conceitos básicos

### Zabbix

O Zabbix é uma suite de monitoramento open-source extretamente eficiente, flexível e poderosa, e um dos seus principais diferenciais é utilizar mínimos recursos dos hosts para fazer o monitoramento. 

O Zabbix Atualmente se encontra na versão 2.0.4.

O Zabbix é uma solução composta por Zabbix Server, Zabbix Agent, Zabbix Proxy e a partir da versão 2.0.x oferece monitoramento de aplicações java via JMX Gateway.

#### Zabbix Server

O Zabbix Server é o 'sistema nervoso' da suite de monitoramento Zabbix, é ele que coleta os dados, é ele que calcula e executa as triggers, é ele que envia notificações aos usuários. É o componente central da suite de monitoramento, é a ele que o Zabbix Agent e Zabbix Proxy se reportam e enviam dados para processamento, análise e arquivamento. 

Todas as configurações e informações estatísticas, todos os dados operacionais são armazenados no Zabbix Server.

O Zabbix Server é composto pelo server, frontend PHP e backend SQL que pode ser o PostgreSQL, MySQL ou SQLite.

#### Zabbix Agent

O Agent é quem monitora os hosts, é ele que extrai os dados e envia para o Zabbix Server processar.

O Agent pode funcionar de forma passiva ou ativa.

Na forma passiva, o servidor Zabbix Server solicita dados ao Agente que verifica, extrai e devolve os dados para o Server processar.

Na forma ativa, o agente verifica com o sevidor a lista de itens que precisa checar, a partir daí ele extrai e envia periódicamente para o servidor.

A forma mais comum de uso do agente é o modo passivo. 

#### Zabbix Proxy

O Zabbix Proxy pode coletar informações de um ou mais hosts e enviá-las para o Zabbix Server processar.

O Zabbix Proxy é uma solução que foi desenhada para dividir a carga do Zabbix Server, uma vez que o proxy pode coletar os dados de forma direta, logo pode-se ter vários proxys coletando dados de vários ambientes e enviando para o Zabbix Server processar.

A coleta de dados é um procedimento que consome muitos recursos do servidor, passar isso para o proxy pode ser uma boa ideia para diminuir a carga no master.

Ele também pode ser muito útil para redes muito segmentadas ou para coletar dados em pontos remotos da rede onde a conexão não é muito boa, assim ele centraliza todos os dados e envia de uma vez só para o Zabbix Server processar.

##### Cenários de Zabbix Proxy

Imagine cerca 1000 agentes enviando dados ao mesmo tempo para o Zabbix Server processar, certamente o consumo e memória e processamento seriam enormes   tanto na coleta quanto no processamento. Agora imagine 4 Proxys conectados ao Server enviando dados de forma ordenada, e neste caso cada proxy poderia coletar dados de cerca de 250 agentes, esse cenário certamente ofereceria uma melhor performance para toda a suite Zabbix.

#### Zabbix Java (JMX Gateway)

O Zabbix JMX Gateway é um daemon escrito em JAVA que tem o objetivo de monitorar aplicações JAVA. Quando o Zabbix Server quer saber alguma informação de um ambiente JMX, ele pergunta ao JMX GATEWAY que por sua vez se conecta via API de gerenciamento JMX na aplicação, assim ele extrai o dado e devolve para o Zabbix Server processá-lo. 

Este recurso só está disponível a partir do Zabbix 2.0.x.

Na versão 1.8.x era necessário usar uma aplicação externa chamada ZAPCAT para fazer tal monitoramento.

## Ambiente

Para este projeto eu criei três VMs no virtualbox com duas interfaces de rede.

A primeira interface NAT para baixar pacotes, a segunda interface em modo HostOnly ou Internal usando IP's da faixa 192.168.56.xx

O sistema operacional escolhido foi o Debian Squeeze - 64 bits.

#### Orientações 

Deixe uma partição maior para /opt, é lá que o zabbix e seus arquivos ficarão após compilado.

##### Disposição das VM's utilizadas

    VM1 - IP 192.168.56.80 zabbix server/database/jmx/agent/frontend - 512 ram - 1 processador
    VM2 - IP 192.168.56.81 zabbix proxy/database/agent - 128 ram - 1 processador
    VM3 - IP 192.168.56.82 zabbix agent (usando proxy) - 128 ram - 1 processador
    
Prepare este ambiente antes de prosseguir.

Fixe o IP nas interfaces para evitar problemas ao reiniciar as VMs.

## Mão na Massa

## 1. Zabbix Server

Acesse a VM reservada para o Zabbix Server (192.168.56.80).

A configuração de repositório APT é a padrão.

### 1.1. Pacotes

instale ferramentas de compilação

    aptitude install build-essential

instale pacotes dev do postgresql

    aptitude install postgresql-server-dev-8.4

instale pacotes dev da libcurl

    aptitude install libcurl3-gnutls-dev

instale pacotes dev para jabber

    aptitude install libiksemel-dev
    aptitude install libjabberd2-dev

instale pacote dev para odbc

    aptitude install unixodbc-dev

instale pacote dev para snmp

    aptitude install libsnmp-dev

instale pacote dev para libssh

    aptitude install libssh2-1-dev
    
instale pacote dev para openipmi
    
    aptitude install libopenipmi-dev

instale pacote java jdk

    aptitude install sun-java6-jdk
    
instale o fping

    aptitude install fping
        
juntando tudo

    aptitude install build-essentials postgresql-server-dev-8.4 libcurl3-gnutls-dev libiksemel-dev libjabberd2-dev unixodbc-dev libsnmp-dev libssh2-1-dev libopenipmi-dev sun-java6-jdk fping

### 1.2. Usuário e Grupo

crie o usuário e grupo zabbix

    groupadd zabbix
    useradd -g zabbix zabbix

### 1.3. Instalação do Zabbix Server/Agent/Jmx Gateway

faça download dos sources do zabbix

    wget http://ufpr.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.0.4/zabbix-2.0.4.tar.gz

descompacte o tarball

    tar zxvf zabbix-2.0.4.tar.gz

acesse o diretório

    cd zabbix-2.0.4

configure a compilação

```
./configure --prefix=/opt/zabbix --enable-server --enable-agent --enable-java --enable-ipv6 --with-ldap --with-ssh2 with-net-snmp --with-jabber --with-postgresql --with-libcurl --with-unixodbc --with-openimpi
```

compile o zabbix

    make

instale os arquivos compilados

    make install

crie um diretório para o init

    mkdir /opt/zabbix/init
    
copie os scripts de init default para o diretório criado   
    
    cp /root/zabbix-2.0.4/misc/init.d/debian/zabbix-agent /opt/zabbix/init
    cp /root/zabbix-2.0.4/misc/init.d/debian/zabbix-server /opt/zabbix/init

corrija o path dentro dos scripts para /opt/zabbix

    sed -i 's/\/usr\/local\/sbin/\/opt\/zabbix\/sbin/' /opt/zabbix/init/zabbix-*
    
crie o script de init para o jmx gateway em /opt/zabbix/init/zabbix-java

```
#!/bin/sh
NAME=zabbix-java
DAEMON=/opt/zabbix/sbin/zabbix_java/startup.sh
DOWN=/opt/zabbix/sbin/zabbix_java/shutdown.sh
DESC="Zabbix Java Gateway daemon"
PID=/opt/zabbix/run/zabbix_java.pid

test -f $DAEMON || exit 0
set -e

case "$1" in
  start)
        echo "Starting $DESC"
        $DAEMON
        ;;
  stop)
        echo "Stopping $DESC"
        $DOWN
        ;;
  *)
        echo "Usage: $NAME {start|stop}"
        exit 1
        ;;
esac
```
ajuste a permissão do arquivo

    chmod 755 /opt/zabbix/init/zabbix-java

após ajustar os scripts, copie para /etc/init.d/
    
    cp /opt/zabbix/init/* /etc/init.d

crie os diretórios run e log

    mkdir /opt/zabbix/run/
    mkdir /var/zabbix/log/

ajuste as permissões do diretório

    chown zabbix. /opt/zabbix/log -R
    chown zabbix. /opt/zabbix/run -R

crie links simbólicos para facilitar a administração

    ln -s /opt/zabbix/log /var/log/zabbix
    ln -s /opt/zabbix/etc /etc/zabbix
    
#### 1.3.1 configurando zabbix server

modifique as seguintes linhas no arquivo /opt/zabbix/etc/zabbix_server.conf

    PidFile=/opt/zabbix/run/zabbix_server.pid
    LogFile=/opt/zabbix/log/zabbix_server.log
    DBHost=localhost 
    DBName=zabbixdb
    DBUser=zabbixuser
    DBPassword=zabbixpassword
    DBPort=5432
    StartPollers=8
    StartPollersUnreachable=8
    StartTrappers=8
    StartPingers=4
    JavaGateway=127.0.0.1
    JavaGatewayPort=10052
    StartJavaPollers=5
    JavaGateway=127.0.0.1
    JavaGatewayPort=10052

#### 1.3.2 configurando zabbix agent

modifique as seguintes linhas no arquivo /opt/zabbix/etc/zabbix_agentd.conf    

    PidFile=/opt/zabbix/run/zabbix_agentd.pid
    LogFile=/opt/zabbix/log/zabbix/zabbix_agentd.log
    Server=127.0.0.1
    DebugLevel=3
    StartAgents=4
    Hostname=nomedamaquina

#### 1.3.3 configurando zabbix jmx gateway

Com o JMX rodando, só precisamos configurar a monitoração no HOST via FRONTEND.

### 1.4. Instalação e configuração do Banco de Dados

Neste estudo o banco de dados foi instalado na mesma VM do Zabbix server, no entanto, o ideal é ficar em uma VM separada, e de preferência em um outro HYPERVISOR para não ter muita concorrência de I/O com as VM's já criadas. Se tiver condições separe a VM e se tiver condições coloque em outro servidor HYPERVISOR.

Abaixo segue o processo de instalação e configuração do PostgreSQL como backend do Zabbix.

instale os pacotes do postgresql

    aptitude install postgresql-8.4 postgresql-client-8.4

acesse o banco via psql

    su postgres
    psql
 crie o usuário zabbixuser
   
    postgres=# create user zabbixuser with password 'zabbixpassword';
    CREATE ROLE

crie o banco zabbixdb

    postgres=# create database zabbixdb;
    CREATE DATABASE

defina permissão para o usuário acessar o banco

    postgres=# grant all privileges on database zabbixuser to zabbixdb;
    GRANT

altere o owner do banco zabbixdb para zabbixuser

    postgres=# alter database zabbixdb owner to zabbixuser;
    ALTER DATABASE

teste a conexão com o banco

    psql -U zabbixuser -h 127.0.0.1 zabbixdb
    
    psql (8.4.11)
    conexao SSL (cifra: DHE-RSA-AES256-SHA, bits: 256)
    Digite "help" para ajuda.
    zabbix=> \q

se a conexão for bem sucedida vá para o próximo passo, do contrário revise os passos anteriores.

#### 1.4.1 Instalação limpa - sem migração.

Se não é seu caso, passe para o procedimento 1.4.2, se for seu caso siga os passos abaixo:

acesse o diretório database nos sources do Zabbix

    cd /root/zabbix-2.0.4/database/postgresql

rode os scripts

    psql -U zabbixuser -h localhost -W zabbixdb < schema.sql
    psql -U zabbixuser -h localhost -W zabbixdb < images.sql
    psql -U zabbixuser -h localhost -W zabbixdb < data.sql

#### 1.4.2 Migração de base do zabbix 1.8.x para zabbix 2.0.x 

faça o dump da base de produção

    pg_dump -U zabbixuser -h ip-de-producao zabbixdb > /tmp/zabbix-2013-01-07.sql

faça o restore da base no banco criado

    psql -U zabbix -h 127.0.0.1 -f /tmp/zabbix-2013-01-07.sql  zabbixdb

acesse o diretório de upgrades
 
    cd /root/zabbix-2.0.4/upgrades/dbpatches/2.0/postgresql

rode o patch para atualizar a estrutura do banco

    pgsql -U zabbix -h 127.0.0.1 zabbix < patch.sql

### 1.5 Serviços Zabbix

inicie o server

    /etc/init.d/zabbix-server start

inicie o agent

    /etc/init.d/zabbix-agent start

inicie o jmx gateway

    /etc/init.d/zabbix-java start

verifique as portas
 
    netstat -ntpl|grep 1005

observe a saída do netstat

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      5030/zabbix_agentd
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      4972/zabbix_server
tcp6       0      0 :::10050                :::*                    LISTEN      5030/zabbix_agentd
tcp6       0      0 :::10051                :::*                    LISTEN      4972/zabbix_server
tcp6       0      0 :::10052                :::*                    LISTEN      5040/java    
   
```

ative a inicialização dos serviços no boot

    update-rc.d -f zabbix-server defaults
    update-rc.d -f zabbix-agent defaults
    update-rc.d -f zabbix-java defaults

## 6. Frontend Zabbix 

Se tiver condições separe uma VM só para isto, se não for possível, mantenha junto com o zabbix-server.

instale os pacote do apache e php5

    aptitude install apache2 php5 php5-pgsql php5-gd

crie o diretório /var/www/zabbix

    mkdir /var/www/zabbix

acesse o diretório

    cd /var/www/zabbix

copie os arquivos do frontend a partir do source

    cp -Rav /root/zabbix-2.0.4/frontends/php/*

corrija as permissões dos arquivos

    chown www.data /var/www/zabbix -R

ajustes os seguintes parâmetros do arquivo /etc/php5/apache2/php.ini

    memory_limit = 128M
    post_max_size = 16M
    upload_max_filesize = 16M
    max_execution_time = 300
    max_input_time = 300
    date.timezone = America/Sao_Paulo

o memory_limit deve ser ajustado de acordo com a memória disponível, após os ajustes reinicie o apache2

    /etc/init.d/apache2 restart

## 7. Acesso ao frontend

O frontend foi acessado pelo endereço abaixo
 
    http://192.168.56.80/zabbix
    
Siga os passos da configuração (next-next-finish).

## 8. Zabbix Proxy

Acesse a VM destinada ao proxy (192.168.56.81).

### 8.1 pacotes

instale os pacotes necessários para compilação

    aptitude install build-essentials libcurl3-gnutls-dev unixodbc-dev libsnmp-dev libssh2-1-dev libopenipmi-dev sun-java6-jdk fping libsqlite3-dev sqlite3

### 8.2 compilação

copie os sources do zabbix server, descompacte e faça a configuração para compilação.

    ./configure --prefix=/opt/zabbix --enable-proxy --enable-agent --enable-java --enable-ipv6 --with-ssh2 --with-net-snmp --with-libcurl --with-unixodbc --with-openipmi --with-sqlite3 
compile
    
    make
    
instale
    
    make install	

Veja que eu estou compilando um JMX Gateway caso queria usar o proxy para monitorar uma aplicação JAVA, no entanto não vamos configurar mais nada, deixo normalmente compilado caso eu precise.
        
### 8.3 Usuário e grupo
 
crie o usuário e grupo do zabbix

    groupadd zabbix
    useradd -g zabbix zabbix
 
### 8.4 Configurações
 
ajuste as configurações do arquivo /opt/zabbix/etc/zabbix_proxy.conf

    Server=ip-do-servidor-zabbix 
    Hostname=nome-do-servidor-proxy
    LogFile=/op/zabbix/log/zabbix_proxy.log
    DBName=/opt/zabbix/sqlite/zabbixprx.db
    PidFile=/opt/zabbix/run/zabbix_proxy.pid
    DebugLevel=3
    StartPollers=8
    StartPollersUnreachable=8
    StartTrappers=8
    StartPingers=4
    
ajuste as configurações do arquivo /opt/zabbix/etc/zabbix_agentd.conf 

    PidFile=/opt/zabbix/run/zabbix_agentd.pid
    LogFile=/opt/zabbix/log/zabbix/zabbix_agentd.log
    Server=ip-do-servidor-zabbix
    DebugLevel=3
    StartAgents=4
    Hostname=nomedamaquina  
    
crie os diretórios run, log, init e sqlite

    mkdir /opt/zabbix/{run,log,init,sqlite}

ajuste as permissões dos diretórios criados 

    chown zabbix.zabbix /opt/zabbix/{run,log,init,sqlite}
    
crie os links simbólicos para /var/log/zabbix e /etc/zabbix/

    ln -s /opt/zabbix/log /var/log/zabbix
    ln -s /opt/zabbix/etc /etc/zabbix

crie um arquivo init para o proxy

    cp /root/zabbix-2.0.4/misc/init.d/debian/zabbix-server /opt/zabbix/init/zabbix-proxy
        
altere as seguintes linhas do script init zabbix-proxy    

    NAME=zabbix_proxy
    DAEMON=/opt/zabbix/sbin/${NAME}
    DESC="Zabbix proxy daemon"
    
copie o arquivo init do agent

    cp /root/zabbix-2.0.4/misc/init.d/debian/zabbix-agent /opt/zabbix/init/

ajuste o path dentro dos arquivos

    sed -i 's/\/usr\/local\/sbin/\/opt\/zabbix\/sbin/' /opt/zabbix/init/zabbix-*

copie os arquivos init para /etc/init.d/
    
    cp /opt/zabbix/init/zabbix-* /etc/init.d/
    
ative os serviços zabbix no boot

    update-rc.d -f zabbix-proxy defaults
    update-rc.d -f zabbix-agent defaults    
    
### 8.5 Serviços

execute os serviços

    /etc/init.d/zabbix-proxy start
    /etc/init.d/zabbix-agent start

verifique as portas

    netstat -ntpl|grep zabbix

observe a saída

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      1852/zabbix_agentd
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      1385/zabbix_proxy
tcp6       0      0 :::10050                :::*                    LISTEN      1852/zabbix_agentd
tcp6       0      0 :::10051                :::*                    LISTEN      1385/zabbix_proxy
```

## 9. Agente

Acesse a VM destinada ao agente (192.168.56.82).

### 9.1 pacotes

instale os pacotes necessários para compilação

    aptitude install build-essentials libcurl3-gnutls-dev unixodbc-dev libsnmp-dev libssh2-1-dev libopenipmi-dev

### 9.2 compilação

copie o tarball do server e descompacte, após descompactar configure

    ./configure --prefix=/opt/zabbix --enable-agent --enable-ipv6 --with-ssh2 --with-net-snmp --with-libcurl --with-unixodbc --with-openipmi
    
compile    
    
    make
    
instale
    
    make install
        
### 9.3 Usuário e grupo
 
crie o usuário e grupo zabbix

    groupadd zabbix
    useradd -g zabbix zabbix
 
### 9.4 Configurações 

#### 9.4.1 Enviando dados para servidor zabbix server
     
crie os diretórios run, init e log no /opt/zabbix e ajuste as permissões

    mkdir /opt/zabbix/{run,log,init}
    chown zabbix.zabbix /opt/zabbix/{run,log,init}
     
crie os links simbólicos para /var/log/zabbix e /etc/zabbix

    ln -s /opt/zabbix/log /var/log/zabbix
    ln -s /opt/zabbix/etc /etc/zabbix
     
copie os arquivos default de init

    cp /root/zabbix-2.0.4/misc/init.d/debian/zabbix-agent /opt/zabbix/init

altere o path no arquivo /etc/init.d/zabbix-agent

    sed -i 's/\/usr\/local\/sbin/\/opt\/zabbix\/sbin/' /opt/init/zabbix-agent

copie o arquivo para o diretório /etc/init.d/

    cp /opt/zabbix/init/* /etc/init.d

ajuste as linhas abaixo no arquivo /opt/zabbix/etc/zabbix_agentd.conf    

    PidFile=/opt/zabbix/run/zabbix_agentd.pid
    LogFile=/opt/zabbix/log/zabbix/zabbix_agentd.log
    Server=ip-do-servidor-zabbix
    DebugLevel=3
    StartAgents=4
    Hostname=nomedamaquina
 
ative o zabbix-agent no boot

    update-rc.d -f zabbix-agent defaults

#### 9.4.2 Enviando dados para servidor zabbix proxy

Os procedimentos são os mesmos do item 9.3.1, com exceção da seguinte linha que precisa modificada no arquivo /opt/zabbix/etc/zabbix_agentd.conf

    Server=ip-zabbix-proxy
    
### 9.5 Serviço

inicie o agente

    /etc/init.d/zabbix-agent start

verifique se está rodando

    netstat -ntpl|grep zabbix

observe a saída

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      8089/zabbix_agentd
```

### 9.6 Configurações no Frontend

No Frontend adicione o proxy em Administration/DM e associe a máquina 82 (agente) ao proxy, adicione o proxy com o mesmo nome da diretiva Hostname em zabbix_proxy.conf.

Você também pode criar o proxy e depois associar o HOST ao PROXY pelas configurações do HOST se preferir.

### 10. Amarrando as pontas

Neste post foi possível aprender a instalar o Zabbix 2.0.4 separando os serviços em várias VMs. Abordei também como migrar a base do 1.8.x para 2.0.x através de alguns passos rápidos e já deixamos o JMX Gateway instalado e rodando.

Nos próximos posts pretendo abordar o processo de criação de templates SNMP para monitorar Switchs, Roteadores e Appliances. 

Outra coisa interessante que devo mostrar é a monitoração de aplicações JAVA usando o Zabbix JMX Gateway.

Parece que devo continuar na trilha do Zabbix por mais uns dias aqui em BSB, confesso que estou aprendendo muita coisa nova, isto tem sido muito divertido, portanto, nada melhor do que compartilhar o que eu venho aprendendo aqui neste blog para ficar ainda melhor.

### 11. Referências

Durante meus estudos passei por muitos links, segue a lista abaixo:

#### 11.1 Zabbix.org

Configurações Server/Proxy/Agent/java

* https://www.zabbix.com/documentation/2.0/manual/appendix/config/zabbix_server
* https://www.zabbix.com/documentation/2.0/manual/appendix/config/zabbix_proxy
* https://www.zabbix.com/documentation/2.0/manual/appendix/config/zabbix_agent
* https://www.zabbix.com/documentation/2.0/manual/appendix/config/zabbix_java

Conceitos

* https://www.zabbix.com/documentation/2.0/manual/concepts/definitions
* https://www.zabbix.com/documentation/2.0/manual/concepts/server
* https://www.zabbix.com/documentation/2.0/manual/concepts/agent
* https://www.zabbix.com/documentation/2.0/manual/concepts/proxy
* https://www.zabbix.com/documentation/2.0/manual/concepts/java
* https://www.zabbix.com/documentation/2.0/manual/concepts/sender
* https://www.zabbix.com/documentation/2.0/manual/concepts/get

Bancos

* https://www.zabbix.com/documentation/2.0/manual/appendix/install/db_scripts

Templates

* https://www.zabbix.com/documentation/2.0/manual/config/templates/template

Triggers

* https://www.zabbix.com/documentation/2.0/manual/config/triggers/trigger
* https://www.zabbix.com/documentation/2.0/manual/config/triggers/expression
* https://www.zabbix.com/documentation/2.0/manual/config/triggers/severity
* https://www.zabbix.com/documentation/2.0/manual/config/triggers/dependencies

Usuários/Grupos/Permissões

* https://www.zabbix.com/documentation/2.0/manual/config/users_and_usergroups/permissions

Interface Web

* https://www.zabbix.com/documentation/2.0/manual/installation/install#installing_zabbix_web_interface

Gráficos, Mapas, Telas

* https://www.zabbix.com/documentation/2.0/manual/config/visualisation/maps
* https://www.zabbix.com/documentation/2.0/manual/config/visualisation/graphs
* https://www.zabbix.com/documentation/2.0/manual/config/visualisation/screens
* https://www.zabbix.com/documentation/2.0/manual/config/visualisation/slides

Monitoração Distribuída

* https://www.zabbix.com/documentation/2.0/manual/distributed_monitoring/proxies
* https://www.zabbix.com/documentation/2.0/manual/distributed_monitoring/nodes

JMX Gateway

* https://www.zabbix.com/documentation/2.0/manual/config/items/itemtypes/jmx_monitoring
* https://www.zabbix.com/documentation/2.0/manual/appendix/config/zabbix_java

Tuning

* https://www.zabbix.com/documentation/2.0/manual/appendix/performance_tuning

API

* https://www.zabbix.com/documentation/2.0/manual/appendix/api/api

#### 11.2 zabbixbrasil

O material desses caras é muito fera.

* http://zabbixbrasil.org/files/criando_um_template_zabbix.pdf
* http://zabbixbrasil.org/wiki/tiki-index.php?page=Usando+Zabbix+para+Monitorar+Clientes+SNMP+v1+e+v2c
* http://zabbixbrasil.org/?page_id=7

Para quem deseja monitorar GlassFish com JMX do Zabbix:

* http://zabbixbrasil.org/files/Monitorando_Glassfish2.1_com-Zabbix_2.0.pdf

#### 11.3 Outros variados

Peformance do Zabbix

* http://blog.zabbix.com/monitoring-how-busy-zabbix-processes-are/
* http://openingyourmind.wordpress.com/2012/12/19/melhor-performance-com-o-zabbix-trapper/

SNMP e Afins

* http://www.hjort.co/2010/09/monitorando-servicos-com-snmp-e-zabbix.html
* http://pierky.wordpress.com/2012/04/24/zabbix-a-lightweight-dynamic-template-for-snmp-routers/
* http://linuxtechres.blogspot.com.br/2012/05/setup-snmp-and-snmptrap-monitoring.html
* http://www.oidview.com/mibs/1916/md-1916-1.html
* http://www.iana.org/assignments/enterprise-numbers

Para quem deseja monitorar JBOSS com JMX do Zabbix:

* http://janssenlima.blogspot.com/2012/07/monitorando-servidor-jboss-no-zabbix.html


[s]
Guto
