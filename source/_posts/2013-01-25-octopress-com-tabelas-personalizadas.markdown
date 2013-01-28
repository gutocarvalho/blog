---
layout: post
title: "Octopress com tabelas personalizadas"
date: 2013-01-25 13:05
comments: true
categories: octopress
---

Trabalhar com tabelas em markdown não é complicado, no octopress funciona da mesma forma que em qualquer editor markdown, porém, por alguma razão a configuração definida na folha de estilos (CSS) para tabelas no octopress não formata as bordas, tabela sem borda nem parece tabela.

Para resolver o problema, novamente pesquisei e achei algumas referências, veja abaixo como resolvi em meu ambiente.

### Ajustando o Octopress


Primeiro passo, crie um arquivo chamado data-table.css em source/stylesheets com o conteúdo abaixo:

{% codeblock lang:css %}

* + table {
  border-style:solid;
  border-width:1px;
  border-color:#e7e3e7;
}

* + table th, * + table td {
  border-style:dashed;
  border-width:1px;
  border-color:#e7e3e7;
  padding-left: 3px;
  padding-right: 3px;
}

* + table th {
  border-style:solid;
  font-weight:bold;
}

* + table th[align="left"], * + table td[align="left"] {
  text-align:left;
}

* + table th[align="right"], * + table td[align="right"] {
  text-align:right;
}

* + table th[align="center"], * + table td[align="center"] {
  text-align:center;
}

{% endcodeblock %}

Segundo passo, adicione a linha abaixo ao arquivo source/_includes/head.html

{% codeblock lang:html %}

<link href="/stylesheets/data-table.css" media="screen, projection" rel="stylesheet" type="text/css" />

{% endcodeblock %}

A partir deste momento a tabelas que você criar via markdown

```
Nome                    | Twitter      
----------------------- | --------------
Guto Carvalho           | @gutocarvalho
Jose Augusto Carvalho   | @joseaugustocc

```

Vão ter bordas, o resultado final é este abaixo

Nome                    | Twitter      
----------------------- | --------------
Guto Carvalho           | @gutocarvalho
Jose Augusto Carvalho   | @joseaugustocc

<br>

Viram como foi fácil, e via CSS você pode personalizar ainda mais o estilo de suas tabelas.

No site [smashingmagazine](http://coding.smashingmagazine.com/2008/08/13/top-10-css-table-designs/) tem vários exemplos de CSS para tabelas.

### Referências

* http://samwize.com/2012/09/24/octopress-table-stylesheet/
* http://programus.github.com/blog/2012/03/07/add-table-data-css-for-octopress