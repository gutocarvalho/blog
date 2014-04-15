---
layout: post
title: "Entenda o Bug Heartbleed"
date: 2014-04-10 15:55
comments: true
categories: security
---

## 1. Sobre o Bug

No dia 07 de Abril de 2014 o projeto OpenSSL divulgou um [boletim](https://www.openssl.org/news/secadv_20140407.txt) de segurança informando sobre uma grave falha de implementação nas versões 1.0.1f, 1.0.1e, 1.0.1d, 1.0.1c, 1.0.1b, 1.0.1a e 1.0.1 do OpenSSL, a falha foi chamada de Heartbleed.

O Bug foi reportado por Neel Mehta do time de segurança do google, a correção foi escrita pelos desenvolvedores Adam Langley (agl@chromium.org) e Bodo Moeller (bmoeller@acm.org).

A equipe do OpenSSL recomenda a atualização da lib para versão 1.0.1g ou então a recompilação do pacote OpenSSL com a seguinte flag habilitada

    -DOPENSSL_NO_HEARTBEATS

O bug foi inicialmente registrado no [CVE](https://cve.mitre.org/) sob o ID [2014-0160](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160). 

O CVE é uma das maiores e mais respeitadas base de dados de segurança que contém dados do tipo vulnerability (erros de implementação em softwares) e exposure (falhas de configuração ou de uso de software).	


## 2. Entenda o Bug

A falha é tão séria que permite a leitura de fragmentos de memória dos sistemas operacionais que utilizam o OpenSSL como biblioteca de segurança para seus sistemas e serviços.

Esses fragmentos são obtidos através de clientes ou até mesmo servidores SSL/TLS maliciosos.

Estes clientes ou sevidores podem enviar pacotes DTLS com a extensão Heartbeat (recurso da versão 1.0.1) e durante esse processo de comunicação, a cada requisição eles conseguem interceptar um pequeno vazamento de dados da memória do sistema operacional, no total cerca 64kb de dados são expostos. Vale informar que essas requisições podem ocorrer repetidas vezes, vazando mais e mais dados.

Isto significa que um atacante pode interceptar informações sensíveis ou sigilosas em sistemas que utilizam o openSSL como mecanismo de segurança. 



## 3. O que pode ser interceptado?

O pessoal da [codenomicon](http://www.codenomicon.com) fez diversos testes para simular e validar o bug e os resultados foram surpreendentes, algo que chocou especialistas de segurança no mundo todo.

Eles atacaram seus próprios servidores - externamente - explorando a falha no OpenSSL e conseguiram capturar informações muito sensíveis sem serem percebidos pelos sistemas de segurança - o exploit não deixa nenhum rastro.

Não foi necessário utilizar nenhum tipo de informação interna, não foi necessário conhecer o ambiente, tão pouco utilizar credenciais de acesso locais (user/senha). Explorando o bug de forma simples e direta foi possível acessar dados sensíveis como por exemplo nomes de usuários, senhas, mensagens instantâneas que deveriam estar codificadas, mensagens de correio eletrônico e até fragmentos de documentos internos da empresa.

O mais grave de tudo isto é que foi possível inclusive interceptar e capturar as chaves privadas x.509 que são utilizadas para criptografar tais informações, e uma vez de posse destas chaves, é possível decriptar todos os pacotes de uma determinada origem ou serviço, comprometendo de forma perigosa o sigilo na comunicação de ambientes de toda a empresa.



## 4. Como se proteger do HeartBleed?

A primeira coisa a se fazer é atualizar o OpenSSL, possívelmente sua distribuição Linux ou Unix já deve ter disponibilizado pacotes com essa correção, atualize seu ambiente e reinicie os serviços que utilizam a biblioteca OpenSSL, o restart é necessário para que a versão antiga do OpenSSL seja descarregada da memória, algumas pessoas para ter 100% certeza, reiniciam até mesmo o OS, se você tem janela para isto, é algo que pode ser feito.


## 5. Por que o bug recebeu o nome HeartBleed?

O BUG está relacionado a uma falha na implementação da extensão Heartbeat no protocolo TLS/DTSL (transport layer security protocol), e como ele gera um LEAK (vazamento) foi apelidado assim Heart (de HeartBeat) + Bleed (de sangrar, escorrer), HeartBleed.

## 6. Quais medidas devo tomar além da atualização do OpenSSL?

### 6.1 Medidas para o usuário final

O ideal é que você troque suas senhas nos sistemas e serviços que são oferecidos via HTTPS pela Internet, isto é necessário pois todos os dados que trafegavam em sistemas afetados estão potencialmente comprometidos já que podiam ser interceptados, e neste processo está incluso a sua senha pessoal.

### 6.2 Medidas para o sysadmin e developer

É importante atualizar o OpenSSL, reiniciar os serviços que usam a biblioteca - e se possível o servidor - para liberar a memória e carregar a nova lib OpenSSL.

É importante revogar certificados antigos, gerar novas chaves e certificados para seus serviços internos. Tem gente até regerando a CA local, mas não é preciso tanto, normalmente a chave do CA está bem protegida e encriptada (pelo menos deveria).

No caso de serviços externos que usam certificados gerados por terceiros é importante verificar informações no site do seu provedor de certificados, é possível que ele gere uma nova CA, se for esse o caso, você vai revogar os certificadoa antigos e a partir daí você vai gerar uma requisição de assinatura de certificados para cada um dos seus certificados e vai enviar estes arquivos para seu provedor assinar com a nova CA - já criada em um ambiente pós aplicação do bugfix.

Se eles não publicarem nova CA, você vai então revogar seus certificados antigos e gerar outros novos pelo sistema do provedor, seguindo a mesma lógica, gere as requisições de assinatura e assine no sistema deles, ou então, envie para eles assinarem.

Pensando no pior cenário, caso você tenha alguma chave ssh para acesso a servidores externos como sua vm pessoal ou github por exemplo, se essa chave foi enviada por IM ou EMAIL, vale a pena gerar outra e trocar - por segurança e paz de espírito, assim você dormirá traquilo.

## 7. Informações sobre distros linux

Como já mencionei acima, a maioria das distros já publicou atualizações para o pacote OpenSSL, então verifique no site do projeto de sua distro qual é a versão corrigida e verifique a versão do pacote instalado em seu servidor - via terminal - para ter certeza de que o pacote foi atualizado. 

Abaixo uma lista das principais distros linux com as versões corrigidas do OpenSSL, acompanhe:

CentOS 6

    [gutocarvalho@exige ~]$ rpm -qa|grep openssl
    openssl-1.0.1e-16.el6_5.7.x86_64

Fedora 19 

     openssl-1.0.1e-37.fc19.1 

Ubuntu 13.10

     1.0.1e-3ubuntu1.2
     
Debian 7

     1.0.1e-2+deb7u6

## 8. Quais serviços são afetados?

Todos os serviços que utilizam de alguma forma SSL/TLS são afetados, como por exemplo Apache, Nginx, OpenVPN, Postfix, Dovecot, OpenLDAP, Samba, Jabber, dentre outros.

Atualizando seu ambiente, trocando suas senhas, regerando suas chaves e certificados, o problema se resolve.

## 9. O SSH foi afetado pelo BUG?

O serviço SSH não foi afetado, este é um protocolo distinto, fique tranquilo.

Agora, se o seu servidor roda qualquer outro tipo de serviço que usa SSL, e se o seu servidor disponibiliza esse serviço externamente, e se ele rodava uma versão do OpenSSL com o bug, logo este ambiente está potencialmente comprometido, portanto, é recomendável que você troque suas senhas locais, remotas e chaves ssh para ficar principalmente com a consciência tranquila, além do ambiente protegido. 

## 10. Há quanto tempo estamos vulneráveis?

Apartentemente o BUG foi descoberto em Dezembro de 2013, contudo, o recurso HeartBeat está disponível desde o início de 2012, com isso nossos ambientes ficaram cerca de 2 anos expostos e vulneráveis ao HeartBleed.

## 11. Tirinha do XKCD que explica o BUG

![xkcd heartbleed](http://imgs.xkcd.com/comics/heartbleed_explanation.png)

## 12. Referências


* https://www.openssl.org/news/secadv_20140407.txt

* https://rhn.redhat.com/errata/RHSA-2014-0376.html

* http://heartbleed.com/

* http://xkcd.com/1354/

* https://www.digitalocean.com/community/articles/how-to-protect-your-server-against-the-heartbleed-openssl-vulnerability

* https://community.openvpn.net/openvpn/wiki/heartbleed

* http://serverfault.com/questions/587414/do-i-need-to-replace-keys-for-openssh-in-response-to-heartbleed

* http://superuser.com/questions/739349/does-heartbleed-affect-ssh-keys

* http://security.stackexchange.com/questions/55076/what-should-a-website-operator-do-about-the-heartbleed-openssl-exploit

[s]<br>
Guto