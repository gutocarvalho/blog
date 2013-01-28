---
layout: post
title: "Octopress e novas fontes"
date: 2013-01-25 11:34
comments: true
categories: octopress 
---

Eu não gosto da fonte padrão do Octopresss, já havia lido que ele usa fontes do Google, pesquisando no repositório [google webfonts](http://www.google.com/webfonts), encontrei uma fonte mais interessante - Oxygen, após alguma pesquisa descobri que é bem fácil alterar as fontes, vamos ao procedimento que eu executei.

## Modificando a fonte

Primeiro edite o arquivo sass/custom/_fonts.scss e insira

{% codeblock lang:css %}
$sans: "Oxygen", sans-serif;
$serif: "Oxygen", sans-serif;
$heading-font-family: 'Oxygen', sans-serif;
$header-title-font-family: "Oxygen", Helvetica, Arial, sans-serif;
{% endcodeblock %}

Depois edite o arquivo source/_includes/custom/head.html e modifique o nome da fonte

{% codeblock lang:html %}
<link href="http://fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
<link href="http://fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/cs
{% endcodeblock %}

para Oxygen

{% codeblock lang:html %}
link href="http://fonts.googleapis.com/css?family=Oyxgen:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
<link href="http://fonts.googleapis.com/css?family=Oxygen:regular,italic,bold,bolditalic" rel="stylesheet" type="text/cs
{% endcodeblock %}

Rode o generate e preview

    rake generate && rake preview
    
Acesse seu site localmente e verifique se a fonte mudou

    http://localhost:4000
    
Depois de confirmar o mudança, faça o deploy

    rake deploy
    
Pronto, fontes novas.    
    
[s]<br>
Guto
