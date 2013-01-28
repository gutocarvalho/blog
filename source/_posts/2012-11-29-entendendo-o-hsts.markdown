---
layout: post
title: "Entendendo o HSTS"
date: 2012-11-29 09:56
comments: true
categories: security
---

## 1. HSTS

No dia 19 de Novembro de 2012 a IETF publicou um novo padrão chamado HSTS via [RFC6797](http://tools.ietf.org/html/rfc6797).

HSTS significa **HTTP Strick Transport Security**, essa tecnologia não é nova, ela já vem sendo desenvolvida desde 2010. O HSTS foi criado para permitir que sites possam se declarar acessíveis apenas de forma segura utilizando o protocolo HTTPS, isto ajuda a minimizar problemas que ocorrem nos sites mistos que oferecem tanto acesso HTTP quanto HTTPS. Neste tipo de site, normalmente ocorre um redirect para HTTPS no primeiro acesso do usuário.

Sites mistos sofrem devido a falhas de implementação do HTTPS ou dos desenvolvedores, este tipo de coisa ocorre primeiro pois o HTTPS é protocolo antigo, já tem mais de 18 anos, e claro que não previu em sua criação a maioria dos cenários de internet que temos hoje. O outro problema são os desenvolvedores que nem sempre seguem as boas práticas de desenvolvimento e segurança. 

Devido ao uso misto de HTTP e HTTPS em alguns sites, atacantes podem explorar brechas de implementação e tentar então comprometer estes sites.

## 1.1 Entendendo o problema

Imagine que você digitou no navegador dominio.com.br, normalmente fazemos isto, não especificamos http ou https no início, com isto o navegador vai tentar primeiro acessar a parte insegura do site http://dominio.com.br, e caso exista um redirect o servidor de aplicação vai lhe forcar a carregar a página de forma segura (HTTPS), porém você já fez um contato não seguro com o website.

Atacantes podem se aproveitar desse tipo de situação e explorar falhas com métodos conhecidos como man-in-the-middle.

## 1.2 Solução proposta

Para resolver isto, a IETF reuniu um grupo de especialistas com o objetivo de desenvolver um mecanismo que previna este tipo de problema. Dentre os membros deste seleto grupo podemos citar Jeff Hodge (PayPal), Collin Jackson (Carnegie Mellon University) e Adam Barth (Google).

O objeto principal do projeto era desenvolver algum mecanismo que force que o site seja visitado apenas via HTTPS, convertendo então qualquer url **http://dominio.com.br/xxx** para **htps://dominio.com.br**, fazendo isto sem nem mesmo se comunicar com o website, tudo ocorre no lado do cliente (navegador).

O mecanismo foi criado e chamado de HSTS e o resultado do trabalho destas pessoas está publicado na [RFC6797](http://tools.ietf.org/html/rfc6797), portando podemos dizer que o conteúdo desta RFC é um consenso de toda a comunidade científica (IETF), interessados e colaboradores.

Já temos grandes empresas como Google, Twitter e Paypal que disponibilizam a tecnologia HSTS em seus sites. Isso significa que se você tiver um navegador com suporte HSTS, ele vai saber se o site tem suporte ao HSTS, se tiver já vai tentar estabelecer uma conexão segura, ignorando as formas inseguras de acesso.

## 2. Exemplos de uso em Apache HTTPD

## 2.1 RHEL/CENTOS

```
 # carregando modulo headers

LoadModule headers_module modules/mod_headers.so
 
 # HTTPS-Host-Configuration

<VirtualHost 10.0.0.1:443>
      # Usando HTTP Strict Transport Security para forçar conexao segura
      Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
      # O restante da configuração segue abaixo
      [...]
</VirtualHost>
```

## 2.2 Debian

Primeiro devemos carregar o módulo do apache2

    # a2enmod headers

E depois a configuração no vhost

```
<VirtualHost 10.0.0.1:443>
      # Usando HTTP Strict Transport Security para forçar conexao segura
      Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
      # O restante da configuração segue abaixo
      [...]
</VirtualHost>
```

## 3. Exemplos de uso em NGINX

No Nginx você deve especificar o HSTS na diretiva server que configura o acesso seguro.

```
server {
    listen 443 default_server ssl;
    ssl_certificate /etc/nginx/ssl/example.com_chain.pem;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    add_header Strict-Transport-Security max-age=31536000;
}
```
 
## 4. Outros exemplos de uso

Você pode ainda declarar HSTS no código de sua aplicação, na página HSTS da [wikipedia](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) existem vários exemplos, recomendo.
 
## 5. Entendendo a configuação

## 5.1 Apache

Com a linha abaixo ativamos o HSTS no VHOST do Apache

    Header always set Strict-Transport-Security

Com o parâmetro abaixo definimos o tempo em que o navegador deve lembrar dessa configuração, no caso do exemplo são 365 dias.

    max-age=31536000

E para definirmos que a configuração deve se estender aos subdomínios podemos utilizar a diretiva abaixo:

    includeSubDomains

## 5.2 NGINX

Para ativar o HSTS na diretiva server devemos usar:

    add_header Strict-Transport-Security 

E para definirmos o tempo em que o navegador deve se lembrar disto, neste caso 365 dias.

    max-age=31536000

## 4.Conclusão

Essa tecnologia é aliada do usuário e aumenta consideravelmente a segurança em sua navegação, mas para funcionar seu navegador tem de ter suporte ao HSTS. O Firefox e Chrome já suportam HSTS e já avisaram que em suas próximas versões devem trazer uma lista de sistes HSTS pré-carregados.

;)

[s]<br>
Guto
