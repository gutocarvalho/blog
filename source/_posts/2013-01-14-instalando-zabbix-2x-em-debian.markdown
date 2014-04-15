---
layout: post
title: "Instalando Zabbix 2.0.x Server, Proxy, Frontend, JMX e Agent em Debian"
date: 2013-01-14 15:27
comments: true
toc: true
categories: zabbix
---

Neste post vou mostrar uma das muitas formas de instalar o Zabbix 2.0.6 em modo Server, Agent, Proxy e JMX Gateway.

## 1. Entendendo alguns conceitos básicos

### 1.1 Zabbix

O Zabbix é uma suite de monitoramento open-source extretamente eficiente, flexível e poderosa, e um dos seus principais diferenciais é utilizar mínimos recursos dos hosts para fazer o monitoramento. 

O Zabbix Atualmente se encontra na versão 2.0.6.

O Zabbix é uma solução composta por Zabbix Server, Zabbix Agent, Zabbix Proxy e a partir da versão 2.0.x oferece monitoramento de aplicações java via JMX Gateway.

### 1.2 Zabbix Server

O Zabbix Server é o 'sistema nervoso' da suite de monitoramento Zabbix, é ele que coleta os dados, é ele que calcula e executa as triggers, é ele que envia notificações aos usuários. É o componente central da suite de monitoramento, é a ele que o Zabbix Agent e Zabbix Proxy se reportam e enviam dados para processamento, análise e arquivamento. 

Todas as configurações e informações estatísticas, todos os dados operacionais são armazenados no Zabbix Server.

O Zabbix Server é composto pelo server, frontend PHP e backend SQL que pode ser o PostgreSQL, MySQL ou SQLite.

### 1.3 Zabbix Agent

O Agent é quem monitora os hosts, é ele que extrai os dados e envia para o Zabbix Server processar (modo ativo) ou disponibiliza para o Zabbix Server quando ele o cosulta (modo passivo).

O Agent pode funcionar de forma passiva ou ativa.

Na forma passiva, o servidor Zabbix Server solicita dados ao Agente que verifica, extrai e devolve os dados para o Server processar.

Na forma ativa, o agente verifica com o sevidor a lista de itens que precisa checar, a partir daí ele extrai e envia periódicamente para o servidor.

A forma mais comum de uso do agente é o modo passivo. 

### 1.4 Zabbix Proxy

O Zabbix Proxy pode coletar informações de um ou mais hosts e enviá-las para o Zabbix Server processar.

O Zabbix Proxy é uma solução que foi desenhada para dividir a carga do Zabbix Server, uma vez que o proxy pode coletar os dados de forma direta, logo pode-se ter vários proxys coletando dados de vários ambientes e enviando para o Zabbix Server processar.

A coleta de dados é um procedimento que consome muitos recursos do servidor, passar isso para o proxy pode ser uma boa ideia para diminuir a carga no master.

Ele também pode ser muito útil para redes muito segmentadas ou para coletar dados em pontos remotos da rede onde a conexão não é muito boa, assim ele centraliza todos os dados e envia de uma vez só para o Zabbix Server processar.

### 1.5 Zabbix GET

O Zabbix GET é um comando utilizado para consultar informações em agentes Zabbix, é como o Zabbix Server se comunica com os agentes, contudo ele pode ser instalado separadamente do server para fazer testes e consultas em agentes diretamente.

    zabbix_get -s ip-do-host-monitorado -p 10050 -k agent.version

### 1.6 Zabbi SENDER

O Zabbix SENDER é um comando utilizado para enviar informações ao Zabbix Server diretamente, sem uso de agente. No Zabbix server você precisa criar itens 
chamados Zabbix TRAP para receber e processar informações enviadas via SENDER. Normalmente o SENDER está associado a scripts que geram determinadas métricas e usam depois o SENDER para enviar estes dados coletados ao Zabbix Server.

    zabbix_sender -z zabbix -s "Apache Conexoes Estabelecidas" -k httpd.established.connections -o 178

#### 1.4.1 Cenários de Zabbix Proxy

Imagine que o servidor zabbix precise consultar informações em cerca de 1000 agentes e depois, precisa processar os dados recebidos de todo eles, certamente esta tarefa irá consumir boa parte da memória e processamento disponíveis, e fará isto tanto na coleta quanto no processamento. Agora imagine 4 Proxys conectados ao Server, coletando, armazenando e enviando dados ao zabbix server de forma ordenada, e neste caso, cada proxy poderia coletar dados de cerca de 250, diminuindo a carga de consulta aos agentes, esse cenário certamente ofereceria uma melhor performance para toda a suite Zabbix.

### 1.5 Zabbix JMX Gateway

O Zabbix JMX Gateway é um daemon escrito em JAVA que consegue monitorar JMX Counters de aplicações JAVA. Quando o Zabbix Server quer saber alguma informação de um ambiente JMX, ele pergunta ao JMX GATEWAY que por sua vez se conecta via API de gerenciamento JMX na aplicação, assim ele extrai o dado e devolve para o Zabbix Server processá-lo. 

Este recurso só está disponível a partir do Zabbix 2.0.x.

Na versão 1.8.x era necessário usar aplicações externas como o ZAPCAT ou TWEEDLE para fazer tal monitoramento.

## 2. Mão na Massa

### 2.1 Ambiente

Para este projeto eu criei quatro VMs no virtualbox com duas interfaces de rede em cada.

A primeira interface NAT para baixar pacotes, a segunda interface em modo HostOnly ou Internal usando IP's da faixa 192.168.56.xx

O sistema operacional escolhido foi o Debian Wheezy - 64 bits.

#### 2.1.1 Orientações 

Se possível, deixe uma partição maior para /opt, é lá que o zabbix e seus arquivos ficarão após a compilação.

#### 2.1.2 Disposição das VM's utilizadas

```
VM1 - IP 192.168.56.80 zabbix server/jmx/agent            - 512 MB RAM - 1 processador
VM2 - IP 192.168.56.81 zabbix database/agent              - 512 MB RAM - 1 processador
VM3 - IP 192.168.56.82 zabbix frontend/agent              - 128 MB RAM - 1 processador
VM4 - IP 192.168.56.83 zabbix proxy/database/agent        - 128 MB RAM - 1 processador
```    
Prepare este ambiente antes de prosseguir.

Fixe o IP nas interfaces para evitar problemas ao reiniciar as VMs.

### 2.2. Zabbix Server

Acesse a VM reservada para o Zabbix Server (192.168.56.80).

A configuração de repositório APT é a padrão.

#### 2.2.1 Pacotes

instale ferramentas de compilação

    aptitude install build-essential

instale pacotes dev do postgresql

    aptitude install postgresql-server-dev-9.1

instale pacotes dev da libcurl

    aptitude install libcurl3-gnutls-dev

instale pacotes dev para jabber

    aptitude install libiksemel-dev

instale pacote dev para odbc

    aptitude install unixodbc-dev

instale pacote dev para snmp

    aptitude install libsnmp-dev

instale pacote dev para libssh

    aptitude install libssh2-1-dev
    
instale pacote dev para openipmi
    
    aptitude install libopenipmi-dev

instale pacote java jdk

    aptitude install openjdk-6-jdk
    
instale o fping

    aptitude install fping
        
juntando tudo

    aptitude install build-essential postgresql-server-dev-9.1 libcurl3-gnutls-dev libiksemel-dev libjabberd2-dev unixodbc-dev libsnmp-dev libssh2-1-dev libopenipmi-dev openjdk-6-jdk fping

#### 2.2.2 Usuário e Grupo

crie o grupo zabbix

    groupadd zabbix

crie o usuário zabbix

    useradd -g zabbix zabbix

#### 2.2.3 Compilando

faça download dos sources do zabbix

    wget http://ufpr.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.0.6/zabbix-2.0.6.tar.gz

descompacte o tarball

    tar zxvf zabbix-2.0.6.tar.gz

acesse o diretório

    cd zabbix-2.0.6

configure a compilação

```
./configure --prefix=/opt/zabbix --enable-server --enable-agent --enable-java --enable-ipv6 --with-ldap --with-ssh2 --with-net-snmp --with-jabber --with-postgresql --with-libcurl --with-unixodbc --with-openipmi
```

compile o zabbix

    make

instale os arquivos compilados

    make install

crie um diretório para os scripts de init

    mkdir /opt/zabbix/init

ajuste as permissões

    chown -R zabbix. /opt/zabbix
    
copie os scripts de init default para o diretório criado   
    
    cp /root/zabbix-2.0.6/misc/init.d/debian/zabbix-agent /opt/zabbix/init
    cp /root/zabbix-2.0.6/misc/init.d/debian/zabbix-server /opt/zabbix/init

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
    mkdir /opt/zabbix/log/

ajuste as permissões do diretório

    chown zabbix. /opt/zabbix/log -R
    chown zabbix. /opt/zabbix/run -R

crie links simbólicos para facilitar a administração

    ln -s /opt/zabbix/log /var/log/zabbix
    ln -s /opt/zabbix/etc /etc/zabbix
    

apague os arquivos desnecessários

    cd /opt/zabbix/etc
    rm -rf zabbix_agent.conf zabbix_agent.conf.d

#### 2.2.4 Configurando

##### 2.2.4.1 Configurando server

modifique as seguintes linhas no arquivo /opt/zabbix/etc/zabbix_server.conf

```
    PidFile=/opt/zabbix/run/zabbix_server.pid
    LogFile=/opt/zabbix/log/zabbix_server.log
    LogFileSize=10

    DBHost=192.168.56.81
    DBName=zabbixdb
    DBUser=zabbixuser
    DBPassword=zabbixpassword
    DBPort=5432

    StartPollers=8
    StartPollersUnreachable=8
    StartTrappers=8
    StartPingers=4
    StartDiscoverers=2
    StartHTTPPollers=2

    JavaGateway=127.0.0.1
    JavaGatewayPort=10052
    StartJavaPollers=5

    HousekeepingFrequency=24
    MaxHousekeeperDelete=10000

    CacheSize=32M

    StartDBSyncers=5

    HistoryCacheSize=32M
    TrendCacheSize=32M
    HistoryTextCacheSize=64M
```

##### 2.2.4.2 Configurando zabbix agent

modifique as seguintes linhas no arquivo /opt/zabbix/etc/zabbix_agentd.conf    

    PidFile=/opt/zabbix/run/zabbix_agentd.pid
    LogFile=/opt/zabbix/log/zabbix_agentd.log
    LogFileSize=10
    Server=127.0.0.1
    DebugLevel=3
    StartAgents=4
    Hostname=nomedamaquina

#### 2.2.5. Configurando jmx gateway

Com o JMX rodando, só precisamos configurar a monitoração no HOST via FRONTEND.

#### 2.2.6. Zabbix Database

Acesse a VM reservada para o Zabbix Database (192.168.56.81).

O ideal é ficar em uma VM separada, e de preferência em um outro HYPERVISOR para não ter muita concorrência de I/O com as VM's já criadas. 

Abaixo segue o processo de instalação e configuração do PostgreSQL como backend do Zabbix.

instale os pacotes do postgresql

    aptitude install postgresql-9.1 postgresql-client-9.1

ajuste o postgresql.conf alterando listen_address para o valor abaixo

    listen_addresses = '*'

ajuste o pg_hba.conf adiciona a linha abaixo ao final do arquivo

    host    zabbixdb       zabbixuser       192.168.56.80/32        md5

reinicie o postgresql

    /etc/init.d/postgresql restart

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

    postgres=# grant all privileges on database zabbixdb to zabbixuser;
    GRANT

altere o owner do banco zabbixdb para zabbixuser

    postgres=# alter database zabbixdb owner to zabbixuser;
    ALTER DATABASE

teste a conexão com o banco

    psql -U zabbixuser -h 127.0.0.1 zabbixdb
    
    psql (9.1.9)
    conexao SSL (cifra: DHE-RSA-AES256-SHA, bits: 256)
    Digite "help" para ajuda.
    zabbix=> \q

se a conexão for bem sucedida vá para o próximo passo, do contrário revise os passos anteriores.

##### 2.2.6.1 Instalação limpa - sem migração.

Se não é seu caso, passe para o procedimento 1.4.2, se for seu caso siga os passos abaixo:

acesse o diretório database nos sources do zabbix no seu servidor zabbix server (192.168.56.80)

    cd /root/zabbix-2.0.6/database/postgresql

instale o postgresql-client

    aptitude install postgresql-client-9.1

rode os scripts

    psql -U zabbixuser -h 192.168.56.81 -W zabbixdb < schema.sql
    psql -U zabbixuser -h 192.168.56.81 -W zabbixdb < images.sql
    psql -U zabbixuser -h 192.168.56.81 -W zabbixdb < data.sql

##### 2.2.6.2 Migração de base do zabbix 1.8.x para zabbix 2.0.x 

faça o dump da base de produção

    pg_dump -U zabbixuser -h ip-de-producao zabbixdb > /tmp/zabbix-2013-01-07.sql

faça o restore da base no banco criado

    psql -U zabbixuser -h 192.168.56.81 -f /tmp/zabbix-2013-01-07.sql zabbixdb

acesse o diretório de upgrades
 
    cd /root/zabbix-2.0.6/upgrades/dbpatches/2.0/postgresql

rode o patch para atualizar a estrutura do banco

    pgsql -U zabbixuser -h 192.168.56.81 zabbixdb < patch.sql

insira as novas imagens de mapas

    psql -U zabbixuser -h 192.168.56.81 -W zabbixdb < images.sql

#### 2.2.7 Serviços Zabbix

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

    insserv zabbix-agent
    insserv zabbix-server
    insserv zabbix-java

#### 2.2.8 Frontend Zabbix 

Acesse a VM 192.168.56.82

instale os pacote do apache e php5

    aptitude install apache2-mpm-prefork php5 php5-pgsql php5-gd

crie o diretório /var/www/zabbix

    mkdir /var/www/zabbix

acesse o diretório

    cd /var/www/zabbix

copie os arquivos do frontend a partir do source que está no servidor zabbix

    scp -r root@192.168.56.80:/root/zabbix-2.0.6/frontends/php/* .

corrija as permissões dos arquivos

    chown www-data. /var/www/zabbix -R

ajustes os seguintes parâmetros do arquivo /etc/php5/apache2/php.ini

    memory_limit = 128M
    post_max_size = 16M
    upload_max_filesize = 16M
    max_execution_time = 300
    max_input_time = 300
    date.timezone = America/Sao_Paulo

o memory_limit deve ser ajustado de acordo com a memória disponível, após os ajustes reinicie o apache2

    /etc/init.d/apache2 restart

adicione a seguinte linha ao pg_hba.conf do servidor de banco de dados (192.168.56.81)

    host    zabbixdb       zabbixuser       192.168.56.82/32        md5

reinicie o banco de dados (192.168.56.81)

    /etc/init.d/postgres restart

##### 2.2.8.1 Acesso ao frontend

O frontend foi acessado pelo endereço abaixo
 
    http://192.168.56.80/zabbix

Siga os passos da configuração (next-next-finish), preencha os dados conforme abaixo:

```
Database type	PostgreSQL
Database server	192.168.56.81
Database port	5432
Database name	zabbixdb
Database user	zabbixuser
Database password	zabbixpassword

Zabbix server	192.168.56.80
Zabbix server port	10051
Zabbix server name	zabbixserver
```

Depois de configurar é possível acessar o dashboard web com o usuário Admin e senha zabbix.

### 2.3. Zabbix Proxy

Acesse a VM destinada ao proxy (192.168.56.83).

#### 2.3.1 pacotes

instale os pacotes necessários para compilação

    aptitude install build-essentials libcurl3-gnutls-dev unixodbc-dev libsnmp-dev libssh2-1-dev libopenipmi-dev sun-java6-jdk fping libsqlite3-dev sqlite3

#### 2.3.2 compilação

entre no diretório root

    cd /root

copie os sources do zabbix server
    
    scp -r root@192.168.56.80:/root/zabbix-2.0.6.tar.gz .

descompacte

    tar zxvf zabbix-2.0.6.tar.gz

entre no diretório

    cd zabbix-2.0.6

configure os parâmetros de compilação

    ./configure --prefix=/opt/zabbix --enable-proxy --enable-agent --enable-java --enable-ipv6 --with-ssh2 --with-ldap --with-net-snmp --with-libcurl --with-unixodbc --with-openipmi --with-sqlite3

faça a compilação
    
    make

instale os binários compilados
    
    make install	

#### 2.3.3 Usuário e grupo
 
crie o grupo zabbix

    groupadd zabbix

crie o usuário zabbix

    useradd -g zabbix zabbix
 
#### 2.3.4 Configurações
 
ajuste as configurações do arquivo /opt/zabbix/etc/zabbix_proxy.conf

    Server=192.168.56.80
    Hostname=zabbixproxy
    LogFile=/opt/zabbix/log/zabbix_proxy.log
    DBName=/opt/zabbix/sqlite/zabbixprx.db
    PidFile=/opt/zabbix/run/zabbix_proxy.pid
    DebugLevel=3
    ProxyOfflineBuffer=3
    StartPollers=8
    StartPollersUnreachable=8
    StartTrappers=8
    StartPingers=4
    StartDiscoverers=2
    StartHTTPPollers=2
    JavaGateway=127.0.0.1
    JavaGatewayPort=10052
    StartJavaPollers=2   
 
ajuste as configurações do arquivo /opt/zabbix/etc/zabbix_agentd.conf 

    PidFile=/opt/zabbix/run/zabbix_agentd.pid
    LogFile=/opt/zabbix/log/zabbix_agentd.log
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
    ln -s /opt/zabbix/log /var/log/zabbix

apague os arquivos desnecessários

    cd /opt/zabbix/etc
    rm -rf zabbix_agent.conf zabbix_agent.conf.d

crie um arquivo init para o proxy

    cp /root/zabbix-2.0.6/misc/init.d/debian/zabbix-server /opt/zabbix/init/zabbix-proxy
        
altere as seguintes linhas do script init zabbix-proxy    

    NAME=zabbix_proxy
    DAEMON=/opt/zabbix/sbin/${NAME}
    DESC="Zabbix proxy daemon"
    
copie o arquivo init do agent

    cp /root/zabbix-2.0.6/misc/init.d/debian/zabbix-agent /opt/zabbix/init/

ajuste o path dentro dos arquivos

    sed -i 's/\/usr\/local\/sbin/\/opt\/zabbix\/sbin/' /opt/zabbix/init/zabbix-*

copie os arquivos init para /etc/init.d/
    
    cp /opt/zabbix/init/zabbix-* /etc/init.d/
    
ative os serviços zabbix no boot

    insserv -d zabbix-proxy
    insserv -d zabbix-agent    
    

#### 2.3.5 Serviços

execute os serviços

    /etc/init.d/zabbix-proxy start
    /etc/init.d/zabbix-agent start

verifique as portas

    netstat -ntpl|grep 1005

observe a saída

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      1852/zabbix_agentd
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      1385/zabbix_proxy
tcp6       0      0 :::10050                :::*                    LISTEN      1852/zabbix_agentd
tcp6       0      0 :::10051                :::*                    LISTEN      1385/zabbix_proxy
```

### 2.4 Zabbix Agente

Agora vamos instalar o agente nas VMs FrontEnd (192.168.56.82) e Database (192.168.56.81)

#### 2.4.1 Instalando agente na Database

Vamos utilizar o frontend para compilar o agente e gerar um tarball para usarmos no Database, ela será nossa máquina referência em relação ao agente.

##### 2.4.1.1 pacotes

instale os pacotes necessários para compilação e funcionamento do agente

    aptitude install build-essential libcurl4-gnutls-dev libssh2-1-dev libldap-2.4-2 

##### 2.4.1.2 compilação

entre no diretório root

    cd /root

copie os sources do zabbix server
    
    scp -r root@192.168.56.80:/root/zabbix-2.0.6.tar.gz .

descompacte

    tar zxvf zabbix-2.0.6.tar.gz

entre no diretório

    cd zabbix-2.0.6

copie o tarball do server e descompacte, após descompactar configure

    ./configure --prefix=/opt/zabbix --enable-agent --enable-ipv6 --with-ldap --with-ssh2 --with-libcurl

compile    
    
    make

instale
    
    make install

##### 2.4.1.3 Usuário e grupo
 
crie o usuário e grupo zabbix

    groupadd zabbix
    useradd -g zabbix zabbix
 
##### 2.4.1.4 Configurações 

###### 2.4.1.4.1 Enviando dados para servidor zabbix server
     
crie os diretórios run, init e log no /opt/zabbix e ajuste as permissões

    mkdir /opt/zabbix/{run,log,init}
    chown zabbix.zabbix /opt/zabbix/{run,log,init}
     
crie os links simbólicos para /var/log/zabbix e /etc/zabbix

    ln -s /opt/zabbix/log /var/log/zabbix
    ln -s /opt/zabbix/etc /etc/zabbix

apague os arquivos desnecessários

    cd /opt/zabbix/etc
    rm -rf zabbix_agent.conf zabbix_agent.conf.d
     
copie os arquivos default de init

    cp /root/zabbix-2.0.6/misc/init.d/debian/zabbix-agent /opt/zabbix/init

altere o path no arquivo /etc/init.d/zabbix-agent

    sed -i 's/\/usr\/local\/sbin/\/opt\/zabbix\/sbin/' /opt/zabbix/init/zabbix-agent

copie o arquivo para o diretório /etc/init.d/

    cp /opt/zabbix/init/* /etc/init.d

ajuste as linhas abaixo no arquivo /opt/zabbix/etc/zabbix_agentd.conf    
    PidFile=/opt/zabbix/run/zabbix_agentd.pid
    LogFile=/opt/zabbix/log/zabbix_agentd.log
    Server=192.168.56.80
    DebugLevel=3
    StartAgents=4
    Hostname=nomedamaquina
 
ative o zabbix-agent no boot

    insserv -d zabbix-agent

###### 2.4.1.4.2 Enviando dados para servidor zabbix proxy

Os procedimentos são os mesmos do item 9.3.1, com exceção da seguinte linha que precisa modificada no arquivo /opt/zabbix/etc/zabbix_agentd.conf

    Server=192.168.56.83

##### 2.4.1.5 Serviço

inicie o agente

    /etc/init.d/zabbix-agent start

verifique se está rodando

    netstat -ntpl|grep 1005

observe a saída

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      3335/zabbix_agentd
tcp6       0      0 :::10050                :::*                    LISTEN      3335/zabbix_agentd
```

##### 2.4.1.6 Gerando tarball

Acesse o diretório opt

    cd /opt

Compacte o diretório

    tar jcvf zabbix-2.0.6-vV-AAAA-MM-DD.tar.bz2 zabbix

Exemplo

    tar jcvf zabbix-2.0.6-v1-2013-05-17.tar.bz2 zabbix    

Agora o targall está pronto para ser utilizado na database.

#### 2.4.2 Instalando agente no frontend

copie o tarball da database
    
    cd /opt
    cp -r root@192.168.56.81:/opt/zabbix-2.0.6-v1-2013-05-17.tar.bz2 .

descompacte

    tar jxvf zabbix-2.0.6-v1-2013-05-17.tar.bz2

entre no diretório

    cd zabbix-2.0.6

ajuste o arquivo zabbix_agentd.conf

```
PidFile=/opt/zabbix/run/zabbix_agentd.pid
LogFile=/opt/zabbix/log/zabbix_agentd.log
Server=192.168.56.80
DebugLevel=3
StartAgents=4
Hostname=zabbixfrontend
```

crie os links simbólicos para /var/log/zabbix e /etc/zabbix

    ln -s /opt/zabbix/log /var/log/zabbix
    ln -s /opt/zabbix/etc /etc/zabbix
    
copie o arquivo para o diretório /etc/init.d/

    cp /opt/zabbix/init/* /etc/init.d

ative o zabbix-agent no boot

    insserv -d zabbix-agent

crie o usuário e grupo zabbix

    groupadd zabbix
    useradd -g zabbix zabbix

inicie o agente

    /etc/init.d/zabbix-agent start

verifique se está rodando

    netstat -ntpl|grep 1005

observe a saída

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      3335/zabbix_agentd
tcp6       0      0 :::10050                :::*                    LISTEN      3335/zabbix_agentd
```

pronto.

### 2.5. Zabbix Frontend & Proxy

No Frontend adicione o proxy em Administration/DM e associe a máquina 81 (database) e 82 (frontend) ao proxy, adicione o proxy com o mesmo nome da diretiva Hostname em zabbix_proxy.conf.

Você também pode criar o proxy em Administration/DM e depois associar o HOST ao PROXY pelas configurações do HOST se preferir.

## 3. Amarrando as pontas

Neste post foi possível aprender a instalar o Zabbix 2.0.6 separando os serviços em várias VMs. Abordei também como migrar a base do 1.8.x para 2.0.x através de alguns passos rápidos e já deixamos o JMX Gateway instalado e rodando.

Nos próximos posts pretendo abordar o processo de criação de templates SNMP para monitorar Switchs, Roteadores e Appliances. 

Outra coisa interessante que devo mostrar é a monitoração de aplicações JAVA usando o Zabbix JMX Gateway.

Parece que devo continuar na trilha do Zabbix por mais uns dias aqui em BSB, confesso que estou aprendendo muita coisa nova, isto tem sido muito divertido, portanto, nada melhor do que compartilhar o que eu venho aprendendo aqui neste blog para ficar ainda melhor.

## 4. Referências

Durante meus estudos passei por muitos links, segue a lista abaixo:

### 4.1 Zabbix.org

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

### 4.2 ZabbixBrasil

O material desses caras é muito fera.

* http://zabbixbrasil.org/files/criando_um_template_zabbix.pdf
* http://zabbixbrasil.org/wiki/tiki-index.php?page=Usando+Zabbix+para+Monitorar+Clientes+SNMP+v1+e+v2c
* http://zabbixbrasil.org/?page_id=7

Para quem deseja monitorar GlassFish com JMX do Zabbix:

* http://zabbixbrasil.org/files/Monitorando_Glassfish2.1_com-Zabbix_2.0.pdf

### 4.3 Outros

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

[s]<br>
Guto
