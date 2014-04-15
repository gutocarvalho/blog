---
layout: post
title: "Saltstack"
date: 2012-09-27 19:20
comments: true
categories: tecnologia
---

Há algum tempo venho compartilhando no blog meus estudos sobre gerência de configurações, e nestes estudos venho apresentando o Puppet como uma excelente ferramenta para implantar esse tipo de controle, porém ele não é a única opção, já mencionei o CHEF como uma ferramenta  alternativa e recentemente meu bom amigo @douglasandrade me apresentou o SaltStack.

O Salt como o puppet é uma ferramenta para gerenciar sua infraestrutura, ele tem condições de controlar o estado das configurações dos sistemas sob sua gerência e também tem recursos nativos para executar comandos remotamente em centenas ou milhares de nodes.

Veja abaixo a definição do propio site.

{% blockquote %}
Salt is an open source tool to manage your infrastructure. Easy enough to get running in minutes and fast enough to manage tens of thousands of servers (and still get a response back in seconds).
{% endblockquote %}

Eu recentemente instalei o salt em um dos clientes que também estamos usando o puppet, pelo puppet, mas o foco nesse caso foi a execução remota de comandos, a instalação do SALT foi muito rápida e simples, em poucos minutos ele estava instalado e eu já havia conseguido executar comandos em nodes remotos.

É uma ferramenta interessante, recomendo conhecê-la se estiver pensando em implementar gerência de configurações.

Eu particularmente ainda prefiro o puppet para controlar o estado das configurações, acho sua sintaxe mais clara, elegante, porém o SALT provou seu valor na execução remota de comandos de forma rápida e simplificada.

Abaixo um screencast do SALT.

<iframe src="http://blip.tv/play/AYLc3HMC.html?p=1" width="550" height="443" frameborder="0" allowfullscreen></iframe><embed type="application/x-shockwave-flash" src="http://a.blip.tv/api.swf#AYLc3HMC" style="display:none"></embed>

Em breve falarei mais do Salt.

[s]<br>
Guto
