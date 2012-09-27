---
layout: post
title: "Primeiro post"
date: 2012-09-26 22:56
comments: true
categories: tecnologia
---
Este é meu primeiro post no Octopress, estou de mudança do Wordpress para cá, em breve vou trazer os posts mais interessantes.



### 1. Por que estou de mudança?

Eu queria algo mais simples para blogar, principalmente algo mais rápido.



### 2. Minha história com o WP

O Wordpress me atendeu por muitos e muitos anos, construí diversos projetos nele, porém nos últimos tempos eu tenho tido uma sensação de que a cada versão ele fica mais lento e mais vulnerável a medida que surgem novas funcionalidades.

Além disto, não consigo enxergar a necessidade de ter um banco de dados relacional para armazenar posts de um blog.

Tive recentes experiências com ferramentas sem banco que me provaram que é possível ter qualidade sem ter que usar algum banco SQL.

A experiência que tive com o DOKUWIKI foi realmente esclarecedora, esta ferramenta funciona de forma maravilhosa, armazena tudo em arquivos, é rápido e simples de usar, tem centenas de plugins, foi escrita em PHP e fazer backup é uma maravilha, basta compactar um diretório e acabou o drama.

Perseguindo essa ideia de simplificar, dei uma olhada no Flatpress, Pixie, PivotX, gpEasy e GetSimple, em sua maioria são CMS que não usam banco de dados, começava então uma longa pesquisa para substituir o WP.



### 3. Mas afinal o que vinha me desagradando no WP?

Eventualmente quando eu ministrava alguma palestra ou quando meu blog era mencionado em algum outro blog grande, o Wordpress claramente não escalava - a.k.a - sentava bonito.

Nos últimos tempos, percebi que havia diminuído minhas postagens devido a insatisfação com a ferramenta de blog, só o tempo que eu demorava para logar, criar um novo post e começar a escrever me desanimava profundamente a prosseguir.

Além disto, havia a necessidade de uma desagradável manutenção, era backup de banco, backup do WP, atualizações de segurança constantes no core do WP, atualizações de plugin quase diárias, tudo isso já estava me cansando, as vezes eu logava para blogar, e quando percebia já estava fuçando e atualizando plugins e o própio WP.

A combinação plugins + banco é uma âncora, para melhorar algo que já é lento, precisa-se usar mais plugins de cache ou colocar sistemas como varnish ou nginx na frente do WP fazendo o cacheamento pesado pra que ele escale.

O Editor rico (WYSIWYG) me deixava louco as vezes, copiar e colar algo era penoso e isso gerava muito retrabalho.



### 4. O que eu queria?

Eu não queria uma ferramente que usasse banco de dados relacional  

Eu queria que a ferramenta gerasse HTML Puro

Eu queria algo que pudesse controlar via GIT

Eu queria algo em Ruby, Python ou Perl - usei muito PHP, era hora de experimentar outras coisas.

Eu queria sair do mainstream e ver as novidades recentes



### 5. Octopress

Após muita pesquisa ainda não me via satisfeito, achei muitas ferramentas interessantes, porém a maioria havia sido feita usando PHP, e como disse, queria algo que não fosse PHP, eu queria algo mais simples, queria algo que gerasse HTML puro. 

Nessa linha de pesquisa, encontrei a engine Jekill, o framework Octopress (ruby) e um outro framework inspirado neste chamado Hyde (python), me identifiquei mais com o Octopress, testei, gostei e resolvi mudar.



### 6. Posts antigos

Se vou migar os posts antigos? Provavelmente, ainda estou testando isso, existe um script chamado exitwp que migra os posts exportados do wordpress em formato XML para o formato Markdown, assim que eu tiver mais tempo termino esse teste.



### 7. O Octopress me atende?

No meu caso sim, para boa fatia dos blogueiros que usam WP, provavelmente não. 

O conceito dele é bem diferente, para você ter uma ideia eu uso o MOU para escrever os posts (editor markdown) em meu OSX, uso o git para versionar meus arquivos, e posso via git ou rsync, publicar meus posts e páginas. 

A principal vantagem é que eu tenho o blog localmente no meu notebook, com isso posso trabalhar nele e empurrar para meu provedor quando houver conectividade.



### 7. O que espero obter usando o Octopress?

Agilidade na publicação de meus posts

Agilidade na formatação dos meus posts (markdown)

Conseguir aumentar a frequência de posts

Oferecer um site mais rápido para visitantes

Usar quase a mesma sintaxe que uso dokuwiki

Blogar diretamente do meu note



### 8. Já foi o tempo do Wordpress? 

Na minha opinião, o Wordpress é uma ótima ferramenta, mas hoje temos uma gama de tecnologias surgindo, coisas muito interessantes, mais rápidas, mais eficientes, novas abordagens para gerenciar conteúdo e armazenar informações, soluções que merecem nossa atenção, portanto, acho que é hora de inovar, o wordpress é sempre uma alternativa, mas eu sou da turma que gosta de novas experiências, portanto, na minha opinião o tempo do WP já passou e em meus novos projetos, certamente vou escolher outro tipo de ferramenta.

[s]<br>
Guto
