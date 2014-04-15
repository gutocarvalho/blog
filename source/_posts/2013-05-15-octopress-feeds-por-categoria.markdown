---
layout: post
title: "Octopress feeds por categoria"
date: 2013-05-15 17:10
comments: true
categories: octopress
---

Hoje eu precisei aprender a habilitar feeds xml por categoria em meu octopress, esse procedimento é simples, o recurso é nativo, contudo, vem desabilitado.

Primeiro acesse o feed principal e veja que o arquivo chama-se atom.xml

    http://gutocarvalho/octopress/atom.xml

Agora ative o feed de categorias no arquivo _config.yml

    category_feeds: true
    
E depois rode um generate e deploy
 
    rake generate && rake deploy

A partir daí basta entrar na página de uma categoria

    http://gutocarvalho.net/octopress/categories/puppet
    
Neste mesmo diretório teremos um arquivo chamado atom.xml, acesse o arquivo para visualizar o feed XML.

    http://gutocarvalho.net/octopress/categories/puppet/atom.xml
    
Simples, rápido e sem gambiarras.

[s]<br>
Guto
