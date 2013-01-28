---
layout: post
title: "Puppet: Wiki com mudanças da V3"
date: 2012-10-03 17:24
comments: true
categories: puppet
---

A wikipage [BreakingChangesInTelly](http://projects.puppetlabs.com/projects/puppet/wiki/BreakingChangesInTelly) traz uma lista de mudanças importantes no puppet v3 em um formato mais amigável.

Dentre as novidades, acredito que uma das questões mais relevantes é a remoção dos comandos stand-alone, estes agora são chamados via comando puppet, veja na tabela abaixo como executar os comandos na versão nova.

Comandos pré 2.6 | Comandos pós 2.6-3.0
--------------- | ----------------
filebucket      | puppet filebucket
pi              | puppet describe
puppetdoc       | puppet doc 
ralsh           | puppet resource
pupetca         | puppet ca
puppetd         | puppet agent
puppetmasterd   | puppet master
pupptqd         | puppet queue
puppetrun       | puppet kick
<br>

Fica a dica.

[s]<br>
Guto
