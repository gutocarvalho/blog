---
layout: post
title: "HAPROXY balanceando JBOSS"
date: 2013-03-15 15:20
comments: true
toc: true
categories: tecnologia
---

Este post estava encostado há algum tempo, finalmente consegui tempo para publicá-lo.

Há alguns meses um leitor me pediu ajuda com o HAPROXY, ele disse que havia visto em minha wiki alguns exemplos de uso, mas queria algo mais focado no JBOSS, este leitor tinha uma necessidade específica e resolvi ajudá-lo, mesmo não sendo especialista e não conhecendo a fundo a ferramenta.

Abaixo segue o roteiro completo de nossos estudos e descobertas.

## 1. Sobre o HAPROXY

O [HAProxy](http://haproxy.1wt.eu/) é uma solução - opensource - que oferece recursos de High Availbility, Load Balance e PROXY para aplicações baseadas em TCP e HTTP. Ele foi particularmente desenhado e pensado para web sites com grande carga que necessitam de persistência ou processamentos específicos na camada de aplicação (layer7). Ele suporta milhares de conexões em hardwares muito modestos. É uma ferramenta fácil de instalar, configurar, operar e principalmente fácil de integrar aos ambientes e arquiteturas existentes.

Existem casos documentados de ambientes HAPROXY atendendo demandas que geravam de 3 a 6 Gibabits de tráfego - por segundo. Existem [testes](http://haproxy.1wt.eu/10g.html) de laboratório feitos pelo criador da ferramenta demonstram que ele chegou a atender cerca de 106 mil requisições HTTP por segundo, e encaminhar (HTTP FORWARD) cerca de 40 mil requisições por segundo para backends internos.

Mas quem usa o HAPROXY?

AMAZON, TWITTER, REDDIT, FARMVILLE, GITHUB, TUMBLR, DISQUS, FEDORA, REDHAT CLOUD, STACKOVERFLOW.

\* Dados retirados de [http://haproxy.1wt.eu/they-use-it.html](http://haproxy.1wt.eu/they-use-it.html)

## 2. HAPROXY

### 2.1 Instalando 

A instalação é bastante simples, vou mostrar como instalar no Debian e CentOS.

No Debian

    aptitude install haproxy

No CentOS

    yum install haproxy
    
Se preferir compile - recomendado - a última versão disponível, basta fazer o download dos sources no site do [haproxy](http://haproxy.1wt.eu/download/1.4/src/).
    
### 2.2 Cenário

O leitor queria balancear uma aplicação de ensino a distância, tipo moodle, porém a ferramenta foi escrita em Java. Em seu ambiente ele tinha dois servidores bem robustos rodando JBOSS, e um servidor mais modesto em standby, estes por sua vez, sustentavam a aplicação de EAD.

### 2.3 Configuração

O arquivo de configuração tem o nome de haproxy.cfg, ele normalmente está no diretório /etc/haproxy.

A configuração proposta foi a seguinte:

```
global
	log 127.0.0.1   local0
	log 127.0.0.1   local1 notice info
	maxconn 25000   # numero maximo de conexoes que ele vai atender
	user haproxy
	group haproxy
	daemon
	nbproc  2 # numero de processadores

defaults
     contimeout      5000
     clitimeout      50000
     srvtimeout      50000
     retries  3

listen  sistema    0.0.0.0:80
 
     log     global
     mode    http
     option httplog
     option dontlognull
     option redispatch
     option forwardfor
     option httpclose
     option forceclose
     option persist
     stats enable
     stats auth admin:proinfo
     stats uri /haproxy_stats
     stats realm HAProxy\ Statistics
     cookie  SERVERID insert indirect nocache
     balance roundrobin
     server  app01 10.1.1.1:8080 cookie A weight 1 maxconn 5000 inter 2s rise 2 fall 3 check
     server  app02 10.1.1.2:8080 cookie B weight 1 maxconn 5000 inter 2s rise 2 fall 3 check
     server  app03 10.1.1.3:8080 cookie B weight 1 maxconn 5000 inter 2s rise 2 fall 3 check backup
```

#### 2.3.1 section global

A seção global do arquivo de configurações define algumas configurações globais, são elas:

```
log              => configurações de log
maxconn          => número máximo global de conexões que ele vai atender
user             => usuário que vai rodar o serviço
group            => grupo
daemon           => forma de funcionamento, no caso standalone
nbproc           => número de processadores
```

#### 2.3.2 section defaults

A seção defaults define algumas configurações padrão para os listeners e backends.

```
contimeout       => tempo máximo que o servidor aguarda para um conexão ser estabelecida
clitimeout       => tempo máximo de inatividade no lado do cliente
srvtimeout       => tempo máximo de inatividade no lado do servidor
retries          => qts vezes o clt ainda deve tentar se conectar após uma falha
```

#### 2.3.3 section listen

Está seção define as portas que o HAPROXY vai abrir e escutar, além disto é aqui que fazemos todas as configurações de balanceamento. Abaixo a explicação de cada parâmetro:

     log                                      => confs de log, no caso confs do global

     mode    http                             => modo de funcionamento (http ou tcp )

     option httplog                           => habilita log http
     option dontlognull                       => não loga conexões de checagem, ajuda a evitar log poluído
     option redispatch                        => limpar o cookie caso o backend caia
     option forwardfor                        => para empurrar o ip do cliente para o backend
     option httpclose                         => ajuda a fechar conexões que já deveriam ter sido encerradas
     option forceclose                        => ajuda a fechar conexões que já deveriam ter sido encerradas
     option persist                           => habilitando persistentica

     stats enable                             => habilita estatísticas
     stats auth admin:password                => usuário/senha para acessar stats
     stats uri /haproxy_stats                 => url das estatísticas
     stats realm HAProxy\ Statistics          => realm de autenticação

     cookie  SERVERID insert indirect nocache => injeta cookie na conexao
     
     balance roundrobin                       => tipo de balanceamento

##### 2.3.3.1 parâmetro cookie     

    SERVERID   => nome do cookie que será inserido, caso necessário
    indirect   => não insere cookie se o cliente já tiver um cookie válido
    insert     => insere um cookie para controle de persistência de sessão caso seja necessário
    nocache    => se cookie for inserido (insert) orienta a não fazer cache
 
##### 2.3.3.2 parâmetro balancer e server     

Veja abaixo que a configuração de balanceamento possui três backends com diversos parâmetros definidos para eles.
     
     server  sistema_01 10.1.1.1:8080 cookie A weight 1 maxconn 5000 inter 2s rise 2 fall 3 check
     server  sistema_02 10.1.1.2:8080 cookie B weight 1 maxconn 5000 inter 2s rise 2 fall 3 check
     server  sistema_03 10.1.1.3:8080 cookie C weight 1 maxconn 5000 inter 2s rise 2 fall 3 check backup

Vamos entender agora para que serve cada parâmetro:

    check     => verifica se a porta está aberta no servidor do pool
    cookie    => nome do cookie que será injetado no lado do cliente
    backup    => essa maquina só vai entrar se uma instância do pool estiver fora
    inter     => significa que ele fará checagens no backend a cada 2 segundos
    rise      => significa que a maquina só volta para o pool se tiver 2 checagens positivas
    fall      => significa que a maquina só sairá do pool se tiver pelo menos 3 checagens negativas
    maxconn   => número máximo de conexões que a instância deve receber
    weight    => peso que a instância terá no pool, quando maior mais conexões vão para ele
        
#### 2.4 Estatísticas

Para acessar as estatísticas habilitadas na configuração proposta, basta apontar seu navegador para o endereço abaixo:

    http://ip-do-servidor-haproxy/haproxy_stats

Através desta ferramenta de estatística, você terá dados completos sobre o balanceamento.

\* Vai pedir o usuário e senha definidos na configuração.
        
#### 2.5 Entendendo a configuração

Abaixo explico em 6 passos rápidos a configuração criada.

1. A configuração **option persist** força o acesso a um servidor do balanceamento que esteja demorando a responder, em caso de servidores que atendem muitas requisições isso é importante, as vezes ele não está fora só está lento.
2. Estamos injetando um cookie (**cookie  SERVERID insert indirect nocache**) com nome SERVERID para organizar o balanceamento e manter a persistência de sessão. Quando um cliente chega, o haproxy injeta um cookie SERVERID=A/B/C para ele, assim quando ele voltar a conectar, vai trazer a informação do cookie (SERVERID=A) e a partir disto cair no mesmo servidor que ele iniciou a conexão, assim ele mantém a persistência na sessão.
3. É muito bom ter a persistência configurada, porém se uma máquina do balanceamento sair do ar, alguns clientes não irão conseguir fazer flush no cookie, com isso eles ficarão perdidos, com o **option REDISPATCH** isso é resolvido, o cookie é resetado caso o backend tenha sido removido do balanceamento e a partir daí a requisição vai para o próximo servidor disponível.
4. O **option forwardfor** é utilizado para empurrar o IP do cliente para o backend ao invés de empurrar o IP do HAPROXY, isso é bom quando queremos gerar estatísticas usando ferramentas como AWSTATS, estas ferramentas normalmente fazem leitura dos logs dos backends.
5. O **option httpclose** e **option forceclose** são utilizados para fechar conexões. Algumas vezes alguns servidores ao receberem o **httpclose**, mesmo assim ainda mantém a conexão aberta com o cliente, deixando ela cair por timeout, isso pode ocasionar centenas e as vezes milhares de conexões desnecessárias esperando timeout, esses dois parâmetros resolvem isto, fechando a conexão assim que o servidor terminar de responder.
6. Configuramos 3 backends, 2 ativos e um em standby (backup), esse último só entrará no balanceamento caso um dos backends principais seja removido por alguma razão, essa máquina é mais modesta, por isto está em modo backup.

## 3. Amarrando as pontas

Segundo o leitor, até hoje o balanceamento está funcionando redondinho, e ele por sinal está bastante feliz.

Tive que ler bastante para ajudá-lo na configuração, isso me ajudou a desenferrujar no HAPROXY, e olha que havia um bom tempo em que eu não usava ele para algo, finalmente surgiu uma boa oportunidade.

O HAPROXY ainda tem centenas de recursos para conhecer, alguns muito interessantes para PROXY e LOAD BALANCER, ele tem suporte a ACLs para redirecionamentos especiais, podemos construir regras fantásticas usando ACLs.

Outra questão sensacional é o balanceamento TCP, posso virtualmente criar um balanceamento na camada 4 utilizando apenas esta ferramenta, com isso posso balancear IMAP, POP, SMTP e qualquer outra aplicação que atenda clientes utilizando uma porta TCP.

Logo podemos dizer que com o HAPROXY você pode construir seu próprio SWITCH de conteúdo.

Se quiser criar um ambiente com HA para seu HAPROXY recomendo o uso do KEEPALIVED. Acesse o site do [Andy Leonard](http://andyleonard.com/2011/02/01/haproxy-and-keepalived-example-configuration/) para ler um bom tutorial sobre esse assunto.

Eu já vi casos de uso do NGINX para fazer proxy reverso e o HAPROXY para balanceamento, afinal ele é especializado nisto.

Já vi cenários com NGINX + HAPROXY + VARNISH (PROXY => BALANCEAMENTO => CACHE).

A grande vantagem do HAPROXY na minha opinião é que ele é uma ferramenta especializada em balanceamento, diferente de outras ferramentas que também fazem balanceamento, mas não são especializadas nisto. O HAPROXY é incrivelmente performatico e usa poucos recursos do servidor (MEM/PROC) para fazer seu trabalho.

Eu particularmente gosto muito desta ferramenta e das possibilidades que ela me oferece, recomendo de olhos fechados.

\* Caso alguém que conheça melhor a ferramenta tenha visto algo errado, por favor nos avise e ajude melhorar este estudo, como eu já disse, sou um admirador da ferramenta mas não um grande conhecedor :)

## 4. Referências

* http://haproxy.1wt.eu/
* https://code.google.com/p/haproxy-docs
* http://gutocarvalho.net/dokuwiki/doku.php/haproxy_exemplos
