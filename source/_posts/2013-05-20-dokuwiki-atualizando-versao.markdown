---
layout: post
title: "Dokuwiki atualizando versão"
date: 2013-05-20 09:04
comments: true
categories: tecnologia
---

## 1. Dokuwiki

O dokuwiki é um sistema de blog que não usa banco de dados SQL, leve, simples e rápido, foi a minha escolha para aposentar o velho mediawiki.

Esse post é uma nota mental para me lembrar dos passos para atualizar o dokuwiki ;)

## 2. Atualização do dokuwiki

### 2.1 Backup

Faça o backup do seu dokuwiki.

    tar jcvf dokuwiki-AAAA-MM-DD.tar dokuwiki/
    
Pronto o backup completo está feito, e como não tem banco SQL, é só isso mesmo, está tudo em disco, arquivos texto.

### 2.2 Download

Faça o download da versão mais nova.

    wget http://www.splitbrain.org/_media/projects/dokuwiki/dokuwiki-2013-05-10.tgz
    
No momento deste post esta é a versão mais nova.

### 2.3 Atualização

Acesse o diretório do dokuwiki

    cd dokuwiki/
    
Atualize os arquivos

    tar -xzvf ../dokuwiki-2013-05-10.tgz --strip-components=1
    
Pronto, dokuwiki atualizado, essa foi rápida ;)

### 2.4 Mensagem de upgrade

Pode ser que ele continue mostrando que há atualização, mesmo após atualizar, então você tem duas opções, limpar o cache, aguardar 1 dia ou rodar um touch no arquivo doku.php, eu faço sempre a terceira.

    touch doku.php
    
E pronto, mensagem a de atualização sumiu.

## 3. Referências

* https://www.dokuwiki.org/changes
* https://www.dokuwiki.org/install:upgrade
* https://www.dokuwiki.org/install:unpacking    
