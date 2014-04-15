---
layout: post
title: "Puppet Recall da versão 3.5"
date: 2014-04-06 08:00
comments: true
categories: puppet	
---

A Puppetlabs chamou um [Recall](http://docs.puppetlabs.com/puppet/latest/reference/release_notes.html#recalled-on-april-4-2014) da versão 3.5, isto significa que essa versão foi removida dos repositórios da puppetlabs e teremos que aguardar até o dia 07 ou 08 de Abril para encontrá-la novamente - após algumas correções.

Alguns usuários reportaram problemas no Puppet 3.5.0 em ambientes que usam o workaround **Dynamic Environments**, além disto, algumas mudanças no resource type YUMREPO - recurso que foi totalmente refatorado para melhorar a qualidade e resolver alguns problemas - geraram novos bugs que estão sendo analisados e corrigidos.

~~Se você já atualizou e não usa nenhum dos recursos acima mencionados, fique tranquilo, não há necessidade de retorno.
~~

Atualizei para o 3.5.0 para testar e mesmo não tendo nada que usa YUMREPO ou ENVIRONMENTS meu ambiente deixou de funcionar e gerenciar os nodes, ainda não encontrei o problema, vou clonar a VM e insistir para tentar abrir um ticket pelo menos, mas no ambiente de produção fiz o downgrade para 3.4.3.

Acompanhe a [página]((http://docs.puppetlabs.com/puppet/latest/reference/release_notes.html) de release notes para saber mais novidades.

[s]<br>
Guto