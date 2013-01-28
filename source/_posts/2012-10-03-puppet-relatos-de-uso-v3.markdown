---
layout: post
title: "Puppet: Relatos de uso V3"
date: 2012-10-03 16:22
comments: true
categories: puppet
---

Os usuários de puppet estão concentrando as informações relativas a dificuldades encontradas no upgrade para a versão 3.0. Se você identificou algum problema de fucionamento em seu ambiente após a atualização, por gentileza informe nesta [wiki](http://projects.puppetlabs.com/projects/puppet/wiki/Telly_Upgrade_Issues).

Hoje ministrei um workshop de puppet já usando a v3, percebi - pelo menos no debian - que o ralsh não vem mais no pacote do puppet.

Na introdução de puppet no workshop eu usava o RAL para demonstrar a abstração no uso de recuros.

RALSH significava `ral shell`.

### Exemplos de uso do RALSH até a versão 2.7

testando resource type **user**

    # ralsh user guto ensure=present

testando resource type **package**

    # ralsh package htop ensure=installed

testando resource type **service**

    # ralsh service ssh ensure=running

### Adaptando para a versão 3.0

testando resource type **user**

    # puppet resource user guto ensure=present

testando resource type **package**

    # puppet resource package htop ensure=installed

testando resource type **service**

    # puppet resource service ssh ensure=running

Até o momento eu só peguei isso, ainda estou confirmado no changelog se esta mudança foi devidamente informada.

[s]<br>
Guto
