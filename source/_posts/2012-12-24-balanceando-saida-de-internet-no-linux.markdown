---
layout: post
title: "Balanceando links de internet no linux"
date: 2012-12-24 07:37
comments: true
categories: firewall 
---

Em um projeto recente precisei fazer o balanceamento de links no linux, o cliente possuía saída por dois provedores, sendo o primeiro NET/Virtua e o segundo Embratel, seu link Embratel estava ocioso e ele queria acabar com essa ociosidade.

A solução foi usar o iproute2 para criar uma tabela com balanceamento de links para alguns pacotes, em conjunto
usei o iptables para marcar os pacotes que deveriam sair
por essa tabela.

Além disto o cliente usava o Embratel para alguns serviços, logo existia um redirecionamento DNAT para rede interna e isso precisava ser levado em conta.

Vamos a solução para essa necessidade.

## Ambiente

Vamos descrever as configurações de rede do ambiente

    Interface eth0 está com rede interna (10.1.x.x/xx)
    Interface eth1 está conectado ao modem virtua (189.x.x.x)
    Interface eth2 está conectado ao modem embratel (200.x.x.x)

Se isto está entendido, vamos continuar.

## Procedimento manual

### 1. Criando tabelas no rt_tables 

Primeiro edite o arquivo

    vim /etc/iproute2/rt_tables

E adicione as seguintes tabelas ao final do arquivo

    20 VIRTUA
    30 EMBRATEL
    40 BALANCEAMENTO

Ótimo, isso é suficiente, elas precisam existir para que possamos criar regras em cada uma destas tabelas.

### 2. Criando regras no iproute2

#### 2.1 configurando tabela virtua

Antes de começar, vou colocar nomes ao invés de endereços IPs para facilitar o entendimento, veja o que significa cada coisa.


    VIRTUA_NET é o endereço de rede
    VIRTUA_NIC é a placa de rede (eth1)
    VIRTUA_GAT é o endereço do GW virtua
    VIRTUA_IPA é o ip da VIRTUA_NIC

Vamos a configuração básica da tabela virtua

    ip route add VIRTUA_NET dev VIRTUA_NIC src VIRTUA_IPA table VIRTUA

Agora vamos especificar quem é o defautl gateway da virtua

    ip route add default via VIRTUA_GAT table VIRTUA

#### 2.2 configurando tabela embratel

Novamente eu vou colocar nomes ao invés de endereços IPs para facilitar o entendimento, veja o que significa cada coisa.


    EMBRATEL_NET é o endereço de rede
    EMBRATEL_NIC é a placa de rede (eth1)
    EMBRATEL_GAT é o endereço do GW
    EMBRATEL_IPA é o ip da EMBRATEL_NIC

Agora vamos configurar a tabela EMBRATEL

    ip route add EMBRATEL_NET dev EMBRATEL_NIC src EMBRATEL_IPA table EMBRATEL

Defina o gateway padrão da tabela EMBRATEL
    
    ip route add default via EMBRATEL_GAT table EMBRATEL

#### 2.3 configurando tabela balanceamento

Agora vamos criar a tabela que fará o balanceamento entre os dois links

    ip route add default scope global table BALANCEAMENTO nexthop via VIRTUA_GAT dev VIRTUA_NIC weight 2 nexthop via EMBRATEL_GAT dev EMBRATEL_NIC weight 1

Pronto, configuramos a tabela BALANCEAMENTO.

Veja que o peso do VIRTUA é maior que o EMBRATEL, fiz isso pois o link VIRTUA é mais parrudo e por isso seu peso no balanceamento será maior (analogia: 2 conexões vão para o virtua e 1 para a Embratel).

#### 2.4 Definido regras de roteamento

Definindo que pacotes vindos do IP do VIRTUAL usarão a tabela VIRTUA

    ip rule add from VIRTUA_IPA table VIRTUA

Definindo que pacotes vindos do IP da EMBRATEL usarão a tabela embratel

    ip rule add from EMBRATEL_IPA table EMBRATEL

Definindo que pacotes com a marca 2 usarão a tabela de balanceamento

    ip rule add fwmark 2 table BALANCEAMENTO

Definindo que pacotes coma marca 1 usarão a tabela Embratel
 
    ip rule add fwmark 1 table EMBRATEL

Essa última regra estou criando pois existe um redirect para para rede interna, e portanto preciso tratar a ida e a volta para que não saia pelo balanceamento, e sim pela interface
de origem do redirect que é a EMBRATEL.

#### 2.5 Configurando rota padrao

Preciso configurar qual será a rota padrão

    ip route add default via EMBRATEL_GAT
    
É possível fazer o balanceamento usando a tabela default, no final darei um exemplo disto.

#### 2.6 Limpando cache de tabelas

Necessário para fazer a limpeza de informação que não é útil, como por exemplo tabelas que foram deletadas.

     ip route flush cache

### 3. Criando regras no netfilter/iptables

#### 3.1 Regras de redirecionamento

Primeiro redireciono para rede interna

    iptables -t nat -A PREROUTING -i eth2 -d 200.252.xx.xxx -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.1.0.xxx:80

Depois marco o pacote de retorno para ele usar a tabela embratel.

    iptables -t mangle -A PREROUTING -i eth0 -s 10.1.xx.xx -p tcp --sport 22 -j MARK --set-mark 1

#### 3.2 Regras para balanceamento

Aqui eu marco os pacotes que desejo que usem a tabela de balanceamento.

     iptables -t mangle -A PREROUTING -s 10.1.x.x/24 -d 0/0 -j MARK --set-mark 1
     
#### 3.3 Mascarando pacotes

Aqui vamos mascarar os pacotes da rede interna, necessário para que as máquinas consigam sair para internet e para o redirect.

     iptables -t nat -A POSTROUTING -s 10.1.x.x/24 -j MASQUERADE
     
#### 3.4 Ativando ip forward

Precisamos ativar o encaminhamento de pacotes para que tudo funcione.

    echo 1 > /proc/sys/net/ipv4/ip_forward

Pronto, balanceamento feito, você pode usar o comando IPTRAF no servidor para avaliar se os dois links estão sendo utilizados.

Nas estações recomendo que use o mtr-tiny para ver por onde seus pacotes estão saindo.
 
     mtr terra.com.br

Se preferir use o traceroute

     traceroute uol.com.br

## Automatizado

Não dá para executar todas as regras toda a vez que a máquina inciar, seria cansativo e poderíamos esquecer
algo, logo criei um script para me ajudar com isto.

### 4. Script de roteamento

{% codeblock lang:bash %}
#!/bin/bash

# variaveis/constantes

VIRTUA_IPA="192.168.xxx.xxx"
VIRTUA_NET="192.168.xxx.xxx/24"
VIRTUA_GAT="192.168.xxx.xxx"
VIRTUA_NIC="eth1"

EMBRATEL_IPA="200.252.xx.xxx"
EMBRATEL_NET="200.252.xx.xxx/26"
EMBRATEL_GAT="200.252.xx.xxx"
EMBRATEL_NIC="eth2"

# limpando tabelas

ip route flush table VIRTUA
ip route flush table EMBRATEL
ip route flush table BALANCEAMENTO

# limpando regras

ip rule del from 200.252.xxx.xxx table EMBRATEL
ip rule del from 192.168.xxx.xxx table VIRTUA
ip rule del fwmark 0x2 table BALANCEAMENTO
ip rule del fwmark 0x1 table EMBRATEL
ip route del default

# configuracoes tabela VIRTUA

ip route add $VIRTUA_NET dev $VIRTUA_NIC src $VIRTUA_IPA table VIRTUA
ip route add default via $VIRTUA_GAT table VIRTUA

# configuracoes tabela EMBRATEL

ip route add $EMBRATEL_NET dev $EMBRATEL_NIC src $EMBRATEL_IPA table EMBRATEL
ip route add default via $EMBRATEL_GAT table EMBRATEL

# trafico da eth1 sai pela tabela VIRTUA

ip rule add from $VIRTUA_IPA table VIRTUA

# trafico da eth2 sai pela tabela EMBRATEL

ip rule add from $EMBRATEL_IPA table EMBRATEL

# definindo regra para marcacao de pacotes da intranet sairem pelo BALANCEAMENTO

ip rule add fwmark 2 table BALANCEAMENTO

# definindo regra para pacotes marcados sairem pela EMBRATEL
 
ip rule add fwmark 1 table EMBRATEL

# Criando balanceamento multilink para tabela BALANCEAMENTO

ip route add default scope global table BALANCAMENTO nexthop via $VIRTUA_GAT dev $VIRTUA_NIC weight 1 nexthop via $EMBRATEL_GAT dev $EMBRATEL_NIC weight 1

# definindo rota padrao

ip route add default via $EMBRATEL_GAT

# fazendo flush no cache de rotas que foram deletadas

ip route flush cache
{% endcodeblock %}


### 5. Script de firewall

{% codeblock lang:bash %}
#!/bin/bash

# variaveis/constantes

IPTABLES="/sbin/iptables"

### limpando tabela filter

$IPTABLES -t filter -F
$IPTABLES -t filter -P INPUT ACCEPT
$IPTABLES -t filter -P OUTPUT ACCEPT
$IPTABLES -t filter -P FORWARD ACCEPT
$IPTABLES -t filter -X

### limpando tabela nat

$IPTABLES -t nat -F
$IPTABLES -t nat -P PREROUTING ACCEPT
$IPTABLES -t nat -P OUTPUT ACCEPT
$IPTABLES -t nat -P POSTROUTING ACCEPT
$IPTABLES -t nat -X

### limpando tabela mangle

$IPTABLES -t mangle -F
$IPTABLES -t mangle -P PREROUTING ACCEPT
$IPTABLES -t mangle -P INPUT ACCEPT
$IPTABLES -t mangle -P FORWARD ACCEPT
$IPTABLES -t mangle -P OUTPUT ACCEPT
$IPTABLES -t mangle -P POSTROUTING ACCEPT
$IPTABLES -t mangle -X

### ativando ip forward

echo 1 > /proc/sys/net/ipv4/ip_forward

### Fazendo redirecionamento para servidores na rede interna

# tratando a ida
$IPTABLES -t nat -A PREROUTING -i eth2 -d 200.252.xxx.xxx -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.1.0.20:80

# tratando a volta
$IPTABLES -t mangle -A PREROUTING -i eth0 -s 10.1.0.20 -p tcp --sport 80 -j MARK --set-mark 1

### Marcando pacotes que serao direcionados para tabela BALANCEAMENTO

# especificando uma maquina da rede para usar balanceamento
$IPTABLES -A PREROUTING -t mangle -s 10.1.0.100/32 -d 0/0 -j MARK --set-mark 2

# Mascarando conexoes

$IPTABLES -t nat -A POSTROUTING -s 10.1.0.20/24 -j MASQUERADE
$IPTABLES -t nat -A POSTROUTING -s 10.1.0.100/24 -j MASQUERADE
{% endcodeblock %}

### 6. Colocando scripts para rodar durante a inicialização

Você pode simplesmente chamar os scripts no arquivo /etc/rc.local, vamos supor que os scripts estejam dentro de /root/rules e tenham o nome de rc.routes e rc.firewall.

Edite o arquivo

    vim /etc/rc.local

Adicione

    /root/rules/rc.routes
    /root/rules/rc.firewall

Salve

    :wq!

Pronto, desde que eles tenham permissão de execução tudo está pronto para funcionar, se quiser testar para valer dê um reboot.

## Outras dicas

### 7. Usando Proxy Transparente na mesma máquina

Se quiser configurar um proxy transparente a regra abaixo resolve.

    $IPTABLES -t nat -A PREROUTING -i eth0 -p tcp -s $LAN_NET --dport 80 -j REDIRECT --to-port 3128
    
Você precisa ter o SQUID na mesma máquina.

### 8. Saída por link específico para destino específico

Se quiser acessar um site sempre por uma das saídas, por exemplo embratel, basta marcar assim:

     $IPTABLES -A PREROUTING -t mangle -s 10.1.xxx.xxx/xx -d 186.202.xxx.xxx -j MARK --set-mark 2
     
Como estou marcando com 2, ele sairia para embratel toda a vez que o destino fosse 186.202.xxx.xxx

#### 8.1 Serviço específico saindo por link específico
    
Podemos especificar que toda o envio de e-mail (SMTP) será feito pelo link EMBRATEL que é mais estável, seguro e tem menos chances se estar em um lista de bloqueio.    
    
    $IPTABLES -A PREROUTING -t mangle -s 10.1.xxx.xxx/xx -d 0/0 --dport 25 -j MARK --set-mark 2
    
### 9. SQUID usando Balanceamento

Se tiver um squid na mesma máquina e desejar que ele use a tabela de balanceamento ao invés da rota padrão, defina a configuração abaixo no squid.conf

    tcp_outgoing_tos 0x1 redelocal
    
0x1 é referente a marcação no iptables para saída pela tabela de balanceamento e rede local seria relativa a uma ACL que tem como conteúdo o endereço da rede local.

#### 7.1 Exemplo de squid.conf
 
Aqui um exemplo de conf de squid3 que só faz cache na porta 80.
 
```
#### porta #################################
 
http_port 3128
 
### hostname ###################
 
visible_hostname fw01
 
### dns servers #################
 
dns_nameservers xxx.xxx.xxx.xxx
 
### configuracoes de cache ############
# referencia: http://www.visolve.com/squid/squid24s1/cache_size.php
# referencia: http://www.visolve.com/squid/squid24s1/cache_size.php#cache_replacement_policy
 
# setando para 2 gigas, os outros 2 gigas vamos deixar 1 giga para sistema e
# 1 giga para operação do cache em disco
 
cache_mem 2048 MB
 
# tamanho de objetos em memoria e disco
 
maximum_object_size_in_memory 512 KB
maximum_object_size 64 MB
minimum_object_size 0 KB
 
# quais o rate para objetos devem serem swapados
cache_swap_low 90
cache_swap_high 95
 
# heap GDSF: otimiza o "hit rate" por manter objetos pequenos e
# e populares no cache, guardando assim um numero maior de objetos
# ao inves de buscar no disco ja esta na memoria, maior velocidade
# na resposta ao usuario
 
memory_replacement_policy heap GDSF
 
# heap LFUDA: otimiza o "byte hit rate" por manter objetos populares
# no cache sem levar em conta o tamanho. Se for utilizado este, o
# maximum_object_size devera ser aumentado para otimizar o LFUDA.
 
cache_replacement_policy heap LFUDA
 
# Lembrando que cada 10GB de cache o squid consome 100MB de ram para gerenciar isto
# colocando 100GB de STORAGE o squid vai usar 1 Giga da RAM para gerenciar o cache do disco
# estou reservando entao 3 gigas para o squid, 2 para cache_mem e 1 para cuidar do cache em disco, sobrando 1 GB para o sistema usar.
# Em relacao ao metodo de gerenciamento do cache, aufs é + rapido que ufs
 
cache_dir aufs /var/spool/squid3 4096 16 256
 
# user/group/manager
 
cache_mgr infraestrutura@empresa.com.br
cache_effective_user proxy
cache_effective_group proxy
 
### ttl de objetos no cache ###########################
# http://www.squid-cache.org/Doc/config/refresh_pattern/
 
refresh_pattern ^ftp:                1440        20%        10080
refresh_pattern ^gopher:        1440        0%        1440
refresh_pattern -i (/cgi-bin/|\?) 0        0%        0
refresh_pattern (Release|Package(.gz)*)$        0        20%        2880
refresh_pattern .                0        20%        4320
 
 
###
### listas de controle #######################
###
 
hierarchy_stoplist cgi-bin ?
 
# acl ligada a autenticacao
 
# acesso padrao daemon squid
acl all src all
acl manager proto cache_object
acl localhost src 127.0.0.1/32
acl to_localhost dst 127.0.0.0/8
 
# redes para icp (proxy filho)
acl localnet src 10.1.xxx.xxx/24
 
# portas seguras
acl SSL_ports port 443                # https
acl SSL_ports port 465                # https
acl SSL_ports port 587                # https
acl SSL_ports port 993                # https
acl SSL_ports port 997                # https
 
# portas comuns
acl Safe_ports port 80                 # http
acl Safe_ports port 8080               # http
acl Safe_ports port 21                 # ftp
acl Safe_ports port 25                 # smtp
acl Safe_ports port 110                # pop
 
# acl que especifica metodos de conectividade
acl purge method PURGE
acl CONNECT method CONNECT
 
# acl que especifica tipo de consulta QUERY em cgi-bin
acl QUERY urlpath_regex cgi-bin \?
 
###
### controle de acesso #############################################
###
 
# liberacoes padrao daemon/localhost
http_access allow manager localhost
http_access deny manager
http_access allow purge localhost
http_access deny purge
 
# nao libera portas diferentes de Safe_ports e SSL_ports
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
 
# nao cacheia cgi-bin
cache deny QUERY
 
# libera rede local
http_access allow localnet

# bloqueia qualquer acesso que nao tenha casado com as regras acima
http_access deny all

### especificando uso da tabela BALANCEAMENTO (roteamento) 

tcp_outgoing_tos 0x1 localnet
 
###
### processando em paralelo ###########
###
 
# http://www.visolve.com/squid/squid24s1/delaypool.php
 
pipeline_prefetch on
 
### reiniciando rapidamente #######
 
shutdown_lifetime 1 second
 
### estatisticas de conexao #######
 
# If enabled, squid will keep statistics on each client.
# This can become a memory hog after a while, so it’s best to keep it disabled.
 
client_db off
 
### conexoes #################
 
# Sends a connection-close to clients
# that leave a half open connection to the squid server.
 
half_closed_clients off
 
### snmp ####################
 
acl snmpcommunity snmp_community empresa
snmp_port 3401
snmp_access allow snmpcommunity localhost
snmp_access deny all
 
### logs #################
 
access_log /var/log/squid3/access.log squid
cache_log /var/log/squid3/cache.log
 
# Log de objetos guardados. Pode ser desativado para melhorar a performance

cache_store_log none
 
### idioma das mensagens do squid para usuarios ################
 
error_directory /usr/share/squid3/errors/Portuguese
 
### arquivos ###################
 
hosts_file /etc/hosts
coredump_dir /var/spool/squid3

```

### 7.2 Exemplo de balanceamento usando tabela default

Ao invés de criar uma tabela específica para balanceamento, podemos definir o balanceamento na tabela default.

    ip route add default scope global nexthop via 189.xxx.xxx.xxx \
    dev eth1 weight 1 nexthop via 200.xxx.xxx.xxx dev eth2 weight 1

É apenas uma outra forma de fazê-lo, mais direta.

## Amarrando as pontas
 
### 8. Conclusão

O Netfilter/iptables e Iproute2 possuem juntos um universo de funcionalidades e comandos para te ajudar a resolver os mais diversos problemas.

O exemplo deste post é bem básico e simples, espero que sirva de referência para aqueles que necessitem balancear dois ou mais links.

### 8.2 Cuidado com testes MEUIP

Se estiver tentando ver se o balanceamento funciona usando sites MEUIP, você está fazendo errado. O balanceamento possui um cache, e no caso do mesmo site ele vai analisar por
qual link voce acessou aquele site e vai sair sempre por ele, isso ocorre para
evitar problemas ao acessar sites que precisam de persistência de sessão.

O balanceamento vai funcionar, mas acesse sites diferentes para testar e use o MTR no console para avaliar por onde você está saindo, isso será mais eficiente.

### 9. Referências de pesquisa

### 9.1 Principais 

* http://www.policyrouting.org/iproute2.doc.html
* http://netfilter.org/documentation/

### 9.2 Secundários

* http://lartc.org/howto/lartc.rpdb.multiple-links.html
* http://www.debian-administration.org/articles/377
* http://www.enterprisenetworkingplanet.com/netos/article.php/3512836/Tunnels-Routes-and-Rules-Theyre-Easier-with-iproute2.htm
* http://blog.nielshorn.net/2008/09/load-balancing-two-isps/

Att.<br>
Guto
