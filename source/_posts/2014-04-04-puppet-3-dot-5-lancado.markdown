---
layout: post
title: "Puppet 3.5 Lançado"
date: 2014-04-04 12:13
comments: true
categories: puppet
---

Foi lançado no dia 03 de Abril a versão [3.5](http://docs.puppetlabs.com/puppet/latest/reference/release_notes.html#puppet-350) do [Puppet](http://www.puppetlabs.com), esta versão traz muitas novidades, correções e melhorias de performance.

Se você instalou o Puppet através do repositórios da Puppetlabs, faça inicialmente a atualização do puppemaster e depois dos agentes, esta é a recomendação Puppetlabs para atualizar o seu ambiente. 

Se você vai atualizar da versão 2.x para a 3.x não deixe de ler a página [upgrading](http://docs.puppetlabs.com/guides/upgrading.html) e  o release notes da versão [3.0.0](http://docs.puppetlabs.com/puppet/3/reference/release_notes.html) para saber o que pode quebrar com a atualização.

## 1. Principais Novidades

### 1.1. Environments

Nova forma de configurar environments, esse novo método é um substituto para o workaround "dynamic environments", agora podemos utilizar a diretiva **$confdir/environments**, e cada diretório que for criado lá dentro será um novo environment. Todas as configurações do diretório criado serão carregadas automaticamente, nenhuma configuração adicional será necessária para utilização deste recurso.

### 1.2 Import Deprecated

Nova forma de carregar manifests, a forma clássica **import "subdir/*.pp"** que normalmente utilizamos no **site.pp** pode ser deixada para trás, agora podemos setar a diretiva **$confdir/manifests** e definir o diretório em que estão nossos manifests, o puppet vai ler o diretório e carregar todos os manifests existentes - recursivamente.

### 1.3 Config Settings

Agora poderemos alterar as configurações do puppet sem necessariamente editar, modificar ou recarregar o arquivo **puppet.conf**.

Abaixo a sintaxe do comando

    # puppet config set <SETTING NAME> <VALUE> --section <CONFIG SECTION>
    
Abaixo veja um exemplo da seção agent do puppet.conf

```
[agent]
graph = true
pluginsync = true
```
Vamos adicionar uma diretiva a seção **agent** através do novo comando

    puppet config set report true --section agent
    
Após rodar o comando você vai verificar que a diretiva foi adicionada ao arquivo
```
[agent]
report = true
graph = true
pluginsync = true
```
Você pode adicionar, remover ou alterar o valor de uma diretiva  com ele.

### 1.5 Global Facts

Essa talvez seja uma das novidades mais interessantes desta versão, e vai certamente ajudar a diferenciar fatos de variáveis em nossos manifests e templates, algo que gerava - e ainda gera - confusão e dúvidas entre usuários iniciantes.

Até a versão 3.4 a forma recomendada de declarar um fato era assim

    $::operationalsystem

Veja que fatos deveriam ser declarados como se fossem variáveis globais - topo de escopo, era ai que morava a confusão, e além disto, neste modelo o nomes dos fatos **não** são protegidos, podendo ser sobrescritos por variáveis locais se estiverem declarados incorretamente. 

Com o recurso **global facts** podemos declarar fatos através da variável protegida **$facts**, essa variável vai conter todos os fatos de um node, e estes poderão ser extraídos da seguinte forma:

    $facts[operationalsystem]

Usando global facts a Puppetlabs pretende deixar o código do Puppet mais legível, limpo, evitando poluir os namespaces de variáveis globais.

Esse recurso vem desligado no Puppet 3.5, mas você pode ativá-lo com o comando abaixo
 
    puppet config set trusted_node_data true --section master
    
Ele provavelmente virá habilitado por padrão na versão 4.

### 1.6 Structured Facts (Early Version)

Todos os valores de fatos retornados na versão **1.x** do facter são strings, o que limita muito a construção de módulos com manifests e templates mais complexos. Já no facter **2.0** - que será padrão no Puppet 4 - temos suporte a vários tipos dados como Boleanos, Strings, Arrays e Hashs. Na versão 3.5 já é possível utilizar esse recurso, porém, ele precisa ser habilitado manualmente.

    puppet config set stringify_facts false --section master

Vale lembrar que isso ainda é bem novo no Puppet, está funcionando de forma estável, contudo, alguns sistemas externos como o PuppetDB ainda não tem suporte a esses diferentes tipos de dados, logo quando fatos personalizados forem enviados para o PuppetDB, estes serão convertidos em strings.

A Puppetlabs assegura que entre o Puppet e o Facter tudo funciona bem, isso em si já possibilta construir ou modificar nossos custom facts para usar esses novos tipos de dados, então aproveite.

### 1.7 Future Parser

Desde o Puppet 3.2.1 o Future Parser está presente, porém agora a Puppetlabs informa que ele está mais maduro para ambientes maiores e mais complexos. O Puppet Future Parser será o novo parser na versão 4 do Puppet, ele será compatível com a nova linguagem do Puppet, esta nova linguagem vai deixar de ser baseada em estados e passará a ser baseada em expressões, o que segundo a puppetlabs trará maior flexibilidade dentro do que se pode fazer hoje com o Puppet.

Na versão 3.5 é necessário habilitar esse recurso manualmente, mas fique ciente de que a Puppetlabs informa que não é aconselhável seu uso em produção.

    puppet config set parser future --section master

### 1.8 Ordering

Quem já tem alguma experiência com o Puppet sabe que é possível especificar a ordem de execução de nossos manifests criando dependências entre nossos rescursos, fazemos isto geralmente através dos meta-parâmetros before, require, notify e subscribe, com isto, temos como garantir a ordem em que as
coisas vão acontecer, porém, novos entusiastas do Puppet tem muita
dificuldade em estabelecer essa ordem, isto ocorre por não conhecerem a fundo
meta-parâmetros ou regras de relacionamento com chaining arrows.

Quem está começando com o Puppet geralmente espera que ele execute o manifest em modo na ordem em que escreve o seu manifest - TOP DOWN, porém não é assim que acontece.

Na série 3.x é possível definir esse comportamento ajustando o valor da diretiva **ordering** para **manifest**, podemos fazer isto com o comando abaixo.

    puppet config set ordering manifest --section master

Funciona bem, contudo, não é algo que eu indico ou recomendo, principalmente se você vai compartilhar código com alguém, ou se vai liberar seus módulos públicamente. Em classes simples até dá para pensar, mas em módulos e em ambiente produtivo é melhor estudar, entender e aplicar recursos de relacionamento, pois sem eles pode ficar bem difícil entender seu código.

### 1.9 Suporte a novos sistemas operacionais

O Puppet agora funciona no RHEL 7 e tem suporte pleno ao systemd. Em breve Fedora 19 e 20 também serão suportados, ambos estão em fase final de testes.

### 1.10 Deixa de ser suportado

A Puppetlabs deixa de suportar o sistema operacional Fedora 18.

### 1.11 Melhoria de performance

A versão 3.5 está bem mais rápida! A puppetlabs encontrou algumas situações relacionadas ao **puppet cert list**, **definitions** e **geração de módulos** que eram bem mais lentas do que deveriam ser, e após o devido tratamento a performance de todos estes recursos, e mais alguns outros, foi melhorada sensivelmente.

## 2. Correções

Acessa a lista completa de correções da versão 3.5.0

* [https://tickets.puppetlabs.com/browse/PUP/fixforversion/11009](https://tickets.puppetlabs.com/browse/PUP/fixforversion/11009)

## 3. Referências

##### Puppet 5.3.0

* [http://docs.puppetlabs.com/puppet/latest/reference/release_notes.html#puppet-350](http://docs.puppetlabs.com/puppet/latest/reference/release_notes.html#puppet-350)

##### Environments

* [http://docs.puppetlabs.com/puppet/latest/reference/environments.html](http://docs.puppetlabs.com/puppet/latest/reference/environments.html)
* [http://docs.puppetlabs.com/puppet/latest/reference/dirs_modulepath.html](http://docs.puppetlabs.com/puppet/latest/reference/dirs_modulepath.html)

##### Import

* [http://docs.puppetlabs.com/puppet/latest/reference/dirs_manifest.html](http://docs.puppetlabs.com/puppet/latest/reference/dirs_manifest.html)

##### Puppet Config

* [http://docs.puppetlabs.com/puppet/latest/reference/config_set.html](http://docs.puppetlabs.com/puppet/latest/reference/config_set.html)

##### Global Facts

* [http://docs.puppetlabs.com/puppet/latest/reference/config_important_settings.html#getting-new-features-early](http://docs.puppetlabs.com/puppet/latest/reference/config_important_settings.html#getting-new-features-early)

##### Structured Facts

* [http://docs.puppetlabs.com/puppet/latest/reference/config_important_settings.html#getting-new-features-early](http://docs.puppetlabs.com/puppet/latest/reference/config_important_settings.html#getting-new-features-early)

##### Future Parser

* [http://docs.puppetlabs.com/puppet/latest/reference/experiments_future.html](http://docs.puppetlabs.com/puppet/latest/reference/experiments_future.html)

* [http://puppetlabs.com/blog/puppet-3-2-introduces-an-experimental-parser-and-new-iteration-features](http://puppetlabs.com/blog/puppet-3-2-introduces-an-experimental-parser-and-new-iteration-features)

[s]<br>
Guto