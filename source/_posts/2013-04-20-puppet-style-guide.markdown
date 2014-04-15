---
layout: post
title: "Puppet Style Guide"
date: 2013-04-20 16:03
comments: true
toc: true
categories: puppet
---

## 1. Puppet Guia de estilos

Neste post eu vou trazer algumas dicas da puppetlabs sobre estilo de código. No site da Puppetlabs há uma página chamada [style guide](http://docs.puppetlabs.com/guides/style_guide.html) explicando isto em inglês, eu pretendo me inspirar nesta página e estender as dicas na medida do possível.

O principal objetivo do style guide é apresentar como utilizar a sintaxe do Puppet de forma mais eficiente. Quando usamos a sintaxe coerentemente, temos ganhos na velocidade de processamento de nossos manifests, além disto, nosso código fica mais legível e elegante.

O parser do Puppet é bastante flexível e mesmo quando você produz algo fora padrões esperados, ele consegue entender, processar e aplicar as configurações descritas em seu manifest, contudo, ele levará mais tempo para processar o manifest, logo podemos dizer que certos tipos de código dificultam vida do parser, isto diminui a performance do agente quando ele for aplicar suas configurações, além disto, a aplicação do manifest pode ter algum comportamento inesperado.

Acompanhe o artigo para entender a melhor forma de usar sintaxe do Puppet.

## 2. Dicas Gerais

### 2.1 Código de fácil leitura

Se você tiver que optar em escrever um código obscuro e complexo e um código mais legível, procure sempre optar por escrever um código legível, seja para você, seja para outras pessoas. Isto pode parecer subjetivo, mas quando trabalhamos em equipe ou quando compartilhamos nosso código, quanto mais legível melhor. Além disto, isto evita situações em que você relê um código escrito meses atrás e não consegue entender o que estava fazendo.

### 2.2 Herança em classes

A Puppetlabs em regra geral pede para evitarmos o uso de herança entre classes, eles acreditam que isso dificulta a leitura e o entendimento do código.

Em minha opinião esse recurso pode ser muito útil em alguns casos, logo recomendo que você estude bem como funciona o conceito antes de utilizar, mais a frente voltaremos a falar disto.

### 2.3 Módulos

Quando for escrever módulos já planeje e desenhe seu módulo primeiro para funcionar com auto loading, segundo para que funcione tanto com um ENC quanto sem um ENC (External Node Classifier). O Foreman e Puppet Dashboard são exemplos de ENC.

Pense também em criar seu módulo utilizando o comando **puppet module generate** para que você siga a estrutura correta e recomendada,
escreva o README bem detalhado e não deixe de preencher os metadados.

### 2.4 Declaração de classes

Antes de entrar no assunto, vamos entender os conceitos DEFINIR e DECLARAR no universo Puppet.

Quando falarmos **definir**, significa que vamos escrever nosso manifest, ou seja, vamos colocar código lá.

{% codeblock lang:puppet %}

class guto {
	user { 'guto':
		ensure => present,
	}
}

{% endcodeblock %}

Quando falarmos **declarar**, significa que vamos executar o código de nosso manifest, ou seja, vamos aplicar a configuração.

{% codeblock lang:puppet %}
class { 'guto': }
{% endcodeblock %}

A puppetlabs não recomenda que durante a **definição** de uma classe, seja feita também **declaração** de outra classe.

Na minha opinião essa recomendação é feita pois este tipo declaração pode dificultar a ação do parser, aumentando o tempo de processamento do catálogo e além disto, pode gerar algum tipo de comportamento indesejável ao aplicar o manifest.

{% codeblock lang:puppet %}
class sudo {

	package { 'sudo':
		ensure => present,
	}
	
	class { 'guto': }
}
{% endcodeblock %}

A declaração de classes deve ser utilizada o mais próximo possível do escopo do **node**, eles recomendam que sejamos conservadores com isto, contudo, há o recurso **include** que permite múltipla declaração de classes, se você precisar declarar outra classe, use include ou então verifique se não é melhor utilizar herança.

## 3. Metadados do módulo

Todo o módulo deve ter um arquivo de metadados, essa é uma recomendação importante principalmente se você deseja compartilhar seu módulo no forge ou github.

Quando você gera seu módulo usando o comando **puppet module**, o exemplo de metadados abaixo é criado, a partir daí é só ajustar.

```
name 'myuser-mymodule'
version '0.0.1'
author 'Author of the module - for shared modules this is Puppet Labs'
summary 'One line description of the module'
description 'Longer description of the module including an example'
license 'The license the module is release under - generally GPLv2 or Apache'
project_page 'The URL where the module source is located'
dependency 'otheruser/othermodule', '>= 1.2.3'
```
Um guia completo sobre a estrutura de um módulo pode ser encontrado no projeto [puppet-module-tool](https://github.com/puppetlabs/puppet-module-tool/blob/master/README.markdown) no github.

## 4. Indentação

Se o seu manifest seguir as recomendações abaixo, estará dentro dos
moldes descritos no guia de estilo.

O seu manifest…

* Deve usar **two-space soft tab** (tab convertida para 2 espaços)
* Não deve usar **tab** (literal)
* Não deve ter **trailing whitespace** em seu código
* Não deve extraploar **80 caracteres** por linha
* Deve ter todas as **setas** (comma arrows) **=>** alinhadas

Eu particularmente uso soft tab no **vim** ou *sublime text 2** com 4 espaços, mas a recomendação deles é clara, 2 espaços para estar dentro dos padrões puppetlabs.

### 4.1 Puppet-lint

Use o [puppet lint](http://puppet-lint.com) para verificar se o seu manifest está de acordo com o [style guide](http://docs.puppetlabs.com/guides/style_guide.html) do Puppet.

## 5. Comentários

O Puppet permite vários tipo de comentários, porém o recomendado é utilizar hash.

A forma **certa** de fazer:

    # Este é o tipo comentário recomendado
    
A puppetlabs não recomenda o uso dos estilos de comentários abaixo:

    // Comentário em formato não recomendado
    /* Comentário em formato não recomendado */

## 6. Quoting

Todas as **strings** que não contenham **variáveis** devem usar **single quotes**.

{% codeblock lang:puppet %}
'string em single quote'
{% endcodeblock %}
    
Quando for necessário **interpolar** strings e variáveis, use **double quoting** e **chaves** nas variáveis para identificá-las.

{% codeblock lang:puppet %}
"string e ${variável}"
{% endcodeblock %}

Valores específicos de **atributos** não precisam de quote.

{% codeblock lang:puppet %}
ensure => present,
{% endcodeblock %}

O uso de **quotes** é opcional quando uma **string** for **alfa-numérica** e desde que não seja o **título** de um recurso.

### 6.1 Interpolação de variável

A forma **certa** de fazer:

{% codeblock lang:puppet %}
"/etc/${file}.conf"
"${::operatingsystem} is not supported by ${module_name}"
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
"/etc/$file.conf"
"$::operatingsystem is not supported by $module_name"
{% endcodeblock %}

### 6.2 Valor de atributos

A forma **certa** de fazer:

{% codeblock lang:puppet %}
mode => $my_mode
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
mode => "$my_mode"
mode => "${my_mode}"
{% endcodeblock %}

## 7. Resources

### 7.1 Resource names 

**Todos** os resource names devem usar **quote**, no entanto, o puppet permite o uso de **nomes** para **títulos** sem quotes desde que o nome não tenha hífen ou espaços, contudo, para seguir um padrão consistente, use sempre quote. 

**Variáveis** ou **fatos** não necessitam de **quotes** quando forem utilizadas como título.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
package { 'htop': ensure => present }
package { $package_name: ensure => present }
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
    package { htop: ensure => present }
    package { "${package_name}": ensure => present }
{% endcodeblock %}

### 7.2. Alinhamento

Todos os atributos e valores de um recurso devem estar alinhados em seu bloco,
as setas devem ter um espaço após o nome do atributo mais longo.

Apesar de ser uma recomendação estética, isto facilita muito a leitura.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
package { 'pacote':
       ensure => present,
}
 
service { 'servico':
	ensure     => running,
	enable     => true,
	hasrestart => true,
	hasstatus  => true,
	require    => Package['pacote'],
}

file { 'main.cf':
	path    => "/etc/pacote/pacote.conf",
	source  => "/root/puppet/pacote.conf",
	owner   => 'root',
	group   => 'root',
	mode    => 644,
	require => Package['pacote'],
	notify  => Service['servico'],
}
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
package { 'pacote':
	ensure => present,
}

service { 'servico':
	ensure => running,
	enable => true,
	hasrestart => true,
	hasstatus => true,
	require => Package['pacote'],
}
 
file { 'main.cf':
	path => "/etc/pacote/pacote.conf",
	source => "/root/puppet/pacote.conf",
	owner => 'root',
	group => 'root',
	mode => 644,
	require => Package['pacote'],
	notify => Service['servico'],
}

{% endcodeblock %}

### 7.3. Ordenação de atributos

Se a declaração de um recurso possuir o atributo **ensure**, este deve preceder qualquer outro atributo, ele sempre vem primeiro.

{% codeblock lang:puppet %}
file { '/tmp/readme.txt':
	ensure => file,
	owner  => '0',
	group  => '0',
	mode   => '0644',
}
{% endcodeblock %}

### 7.4 Múltiplos resources

A puppetlabs só recomenda o uso de múltiplos resources para tornar o código mais legível, portando, evite o uso a não ser que isso seja realmente necessário.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
file { '/tmp/a':
	content => 'a',
}

file { '/tmp/b':
	content => 'b',
}

file { '/tmp/c':
	content => 'c',
}
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
file {
	"/tmp/a":
		content => "a";
	"/tmp/b":
		content => "b";
	"/tmp/c":
		content => "c";
}
{% endcodeblock %}

### 7.5 Links simbólicos

Apenas para deixar o código mais limpo e legível, recomenda-se declarar o link simbólico de forma explícita.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
    file { '/var/log/syslog':
      ensure => link,
      target => '/var/log/messages',
    }
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
    file { '/var/log/syslog':
      ensure => '/var/log/messages',
    }
{% endcodeblock %}

### 7.6 File modes

Devemos representar as permissões de arquivos e diretórios com 4 dígitos ao invés de 3 dígitos. 

As permissões devem ser declaradas usando **single-quote**, **sempre**.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
file { '/var/log/syslog':
	ensure => present,
	mode   => '0644',
}
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
file { '/var/log/syslog':
	ensure => present,
	mode   => 644,
}
{% endcodeblock %}

### 7.7 Resource Defaults

Podemos declarar alguns padrões para uso de recursos, porém, isso deve ser utilizado com cuidado, este tipo de declaração deve ser feita apenas no nível mais alto do seu ecosistema de manifests. Especificamente, eles devem ser declarados:

* No topo de escopo dentro do arquivo site.pp
* Em classes que não serão herdadas
* Em classes em que não você não declara outras classes

Se essas recomendações não forem seguidas poderão ocorrer comportamentos inesperados durante a execução de seus manifests, principalmente se
essa configuração for propagada ou herdada de alguma forma.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
# /etc/puppetlabs/puppet/manifests/site.pp:
File {
	mode  => '0644',
	owner => 'root',
	group => 'root',
}
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
#/etc/puppetlabs/puppet/modules/ssh/manifests/init.pp
File {
	mode  => '0600',
	owner => 'nobody',
	group => 'nogroup',
}

class {'ssh::client':
	ensure => present,
}
{% endcodeblock %}

## 8. Condicionais

### 8.1 Mantenha suas declarações simples

A Puppetalabs não recomenda misturar resource types e condicionais, isto está bem ligado ao uso de **seletores**, portanto, faça o tratamento condicional fora do resource type, é mais eficaz do ponto de vista de processamento do catálogo e além disto, fica mais legível e elegante.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
$file_mode = $::operatingsystem ? {
	debian => '0007',
	redhat => '0776',
	fedora => '0007',
}

file { '/tmp/readme.txt':
	content => "Hello World\n",
	mode    => $file_mode,
}
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
file { '/tmp/readme.txt':
	mode => $::operatingsystem ? {
	debian => '0777',
	redhat => '0776',
	fedora => '0007',
	}
}
{% endcodeblock %}

### 8.2 Estado Default em Case e Seletores

Tanto em CASE quanto em SELETORES, podemos utilizar o statement DEFAULT, ou seja, isto determina um valor default se não casar com nenhuma opção do CASE ou SELETOR.

case:

{% codeblock lang:puppet %}
case $var {
	valor1:  { bloco de código }
	valor2:  { bloco de código }
	default: { bloco de codigo }
}
{% endcodeblock %}

seletor:

{% codeblock lang:puppet %}
$file_mode = $::operatingsystem ? {
	debian  => '0007',
	redhat  => '0776',
	fedora  => '0007',
	default => '0077',
}
{% endcodeblock %}
   
A puppetlabs recomenda que a opção default esteja presente em qualquer tratamento condicional que o suporte, e recomenda ainda que seja associada a função fail, esta função interrompe o processamento do catálogo caso o sistema
não seja compatível com o tratamento condicional definido.

exemplo:

{% codeblock lang:puppet %}
case $::operatingsystem {
	centos, redhat: { $apache = "httpd" }
	debian, ubuntu: { $apache = "apache2" }
	default: { fail("Sistema operacional não reconhecido") }
}
{% endcodeblock %}
    
No exemplo acima, o **CASE** verifica qual é o sistema operacional do node - usando o **fato**, caso o valor detectado não case com nenhuma das opções do CASE, o **default** é acionado e executa a função **fail**, isto interrompe o processamento do catálogo e retorna um erro que poderá ser visualizado em seu ENC ou no log do agente.

## 9. Classes

Recomenda-se separar classes e definições em arquivos distintos no diretório manifest de seu módulo.

{% codeblock lang:puppet %}
 # /etc/puppetlabs/puppet/modules/apache/manifests

    # init.pp
      class apache { }
    # ssl.pp
      class apache::ssl { }
    # virtual_host.pp
      define apache::virtual_host () { }
{% endcodeblock %}

É importante entender que isto não altera em nada o funcionamento do módulo,
tem o mesmo efeito de declarar tudo no arquivo init.pp, contudo, separando
os arquivos temos uma estrutura que facilita o entendimento e torna o todo
mais legível.

### 9.1 Nomes de classes

Observe que o nome das classes dentro dos arquivos utiliza escopos, a regra é clara, o nome da classe recebe o nome do módulo + o nome da classe, veja o exemplo:
{% codeblock lang:puppet %}
# /etc/puppetlabs/puppet/modules/apache/manifests
 # ssl.pp
  class apache::ssl
{% endcodeblock %}
    
Caso você queria criar um diretório para organizar melhor seu módulo a regra passa a incluir o nome do diretório, veja o exemplo:
  
{% codeblock lang:puppet %}    
# /etc/puppetlabs/puppet/modules/apache/manifests
 # conf/parameters.pp
  class apache::conf::parameters
{% endcodeblock %}
      
Essa estrutura em escopo torna o nome das classes do módulo única, diferenciando dos demais módulos, evitando conflitos e problemas
de carregamento, siga sempre essa estrutura.

### 9.2 Organização interna

As classes precisam ser organizadas de forma a manter um estilo e estrutura consistente. Abaixo uma lista com itens importantes encontrados em um classe
que foi construída seguindo a lógica do style guide.

A classe...

* Deve definir parâmetros
* Deve validar parâmetros
* Deve interromper a execução em falha de validação
* Deve oferecer parâmetros default
* Pode ter declaração de variáveis locais
* Pode ter relação com outras classes
  * Class['apache'] -> Class['local_yum'] 
* Pode sobrescrever (override) resources
* Pode declarar resources
* Pode declarar relação entre resources dentro de condicionais

Exemplo de uma classe que segue essas recomendações:

{% codeblock lang:puppet %}

class myservice($ensure='running') {

	  # checagem condicional
	 
      if $ensure in [ running, stopped ] {  
        $ensure_real = $ensure
      } else {
        # interrompendo execução em caso de falta de parâmetro
        fail('ensure parameter must be running or stopped')
      }
      case $::operatingsystem {
        centos: {
          $package_list = 'openssh-server'
        }
        solaris: {
          $package_list = [ SUNWsshr, SUNWsshu ]
        }
        default: {
      	   # interrompendo execução caso o sistema operacional seja desconhecido
          fail("Module ${module_name} does not support ${::operatingsystem}")
      
        }
      
      }

      # variável de uso local
      $variable = 'something'

      Package { ensure => present, }

      File { owner => '0', group => '0', mode => '0644' }

      package { $package_list: }

      file { "/tmp/${variable}":
        ensure => present,
      }

      service { 'myservice':
        ensure    => $ensure_real,
        hasstatus => true,
      }
    }
{% endcodeblock %}

### 9.3 Relações encadeadas

O puppet nos oferece duas formas de trabalhar com relações, a primeira é usando meta-parâmetros como **require**, **before**, **notify** e **subscribe** e a segunda é usando uma cadeia de relações, para entender veja dois exemplos básicos, o primeiro precedência e o segundo notificação.

**Precedência via meta-parâmetro**

{% codeblock lang:puppet %}

package { 'httpd':
	ensure => installed,
	before => Service[httpd],
}

service { 'httpd':
	ensure => running,
}

{% endcodeblock %}

**Precedência usando cadeia de relacionamento**

{% codeblock lang:puppet %}

Package['httpd'] -> Service['httpd']

{% endcodeblock %}

**Notificando via meta-parâmetro**

{% codeblock lang:puppet %}

file { 'httpdconf':
	path   => '/etc/httpd/httpd.conf'
	ensure => present,
	notify => Service[httpd]

service { 'httpd':
	ensure => running,
}

{% endcodeblock %}

**Notificando via cadeia de relacionamento**

{% codeblock lang:puppet %}
File['httpdconf'] ~> Service['httpd']
{% endcodeblock %}

Cadeias de relacionamento são recursos úteis mas devem ser utilizados com cuidado, recomenda-se
utilizar cadeias com regras da esquera para direta sempre, evite outras variações

A forma **certa** de fazer:

{% codeblock lang:puppet %}
Package['httpd'] -> Service['httpd']
File['httpconf'] ~> Service['httpd']
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
    Service['httpd'] <- Package['httpd']
    Service['httpd'] <~ File['httpdconf']
{% endcodeblock %}

Quando possível, utilize meta-parâmetros ao invés de relacionamento interno.

### 9.4 Classes internas

No início já falamos disto, devemos evitar criar classes dentro de classes, ou mesmo definições dentro de classes.

A forma **não** recomendada:

{% codeblock lang:puppet %}
    class apache {
      class ssl { ... }
    }
{% endcodeblock %}

Outra forma **não** recomendada:

{% codeblock lang:puppet %}
    class apache {
      define config() { ... }
    }
{% endcodeblock %}

Prefira sempre criar um arquivo separado e declarar uma única classe ou uma única definição neste arquivo.

### 9.5 Classes e herança

Herança pode ser usada em módulos, mas deve ser utilizada entre as classes do módulo. Se o seu objetivo é  herdar uma classe de outro módulo, pense em usar o **include** ao invés do **inherits**. O simples ato de fazer herança entre módulos já viola o proprio conceito de modularidade, evite isto ao máximo.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
    class ssh { ... }

    class ssh::client inherits ssh { ... }

    class ssh::server inherits ssh { ... }

    class ssh::server::solaris inherits ssh::server { ... }
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
    class ssh inherits server { ... }

    class ssh::client inherits workstation { ... }

    class wordpress inherits apache { ... }
{% endcodeblock %}

### 9.6 Variáveis e namespace

Quando usarmos váriaveis de topo de escopo ou fatos, devemos declará-las de forma a evitar carregamento de uma variável com o mesmo nome
mas em escopo local.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
$::operatingsystem
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
$operatingsystem
{% endcodeblock %}

Se não fizermos isto e houver uma variável com o mesmo nome em escopo
local, ou seja, se você por alguma razão criou uma variável em um
manifest do seu módulo com o nome **operatingsystem**, por ordem
de precedência a variável de escopo **local** será carregada e não a  variável - ou fato - de topo de escopo, desta forma seu código pode ter um comportamento indesejado.

### 9.7 Nome de variáveis

Quando for definir variáveis, procure usar apenas letras, números
e underscore. Não use hífen ou travessão.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
$foo_bar123
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
$foo-bar123
{% endcodeblock %}

### 9.8 Classes e Parâmetros

Em classes parametrizadas e definições, devemos tomar um cuidado especial na hora da declaração. Parâmetros obrigatórios devem ser listados antes de parâmetros opcionais.

A forma **certa** de fazer:

{% codeblock lang:puppet %}
    class ntp (
    $servers,
    $options   = "iburst",
    $multicast = false
    ) {}
{% endcodeblock %}

A forma **não** recomendada:

{% codeblock lang:puppet %}
class ntp (
    $options   = "iburst",
    $servers,
    $multicast = false
    ) {}
{% endcodeblock %}

## 10. Testes

Todos os manifests de um módulo devem ter um manifest de teste no diretório tests, isso é chamado de smoking test, o objetivo do
smoking test é demonstrar como uma classe ou definição deve
ser declarada.

exemplo de manifest:

{% codeblock lang:puppet %}
# /etc/puppet/modules/ntp/manifests/init.pp
class ntp {
    package { 'ntp':
        ensure => present,
    }
}
{% endcodeblock %}

exemplo de smoking test:

{% codeblock lang:puppet %}
# /etc/puppet/modules/ntp/tests/init.pp
class { ntp: }
{% endcodeblock %}

## 11. Puppet DOC

O Puppet Doc é um recurso que nos permite documentar nossas classes e definições utilizando RDoc markup. Esse documentação inline é
importante para facilitar a geração de documentação online, isto
significa que uma vez gerada, poderá ser extraída pelo própio puppet
doc quando desejar informações e comentários das classes e definções
de seu módulo.

Modelo para classes:

```
    # == Class: example_class
    #
    # Full description of class example_class here.
    #
    # === Parameters
    #
    # Document parameters here.
    #
    # [*ntp_servers*]
    #   Explanation of what this parameter affects and what it defaults to.
    #   e.g. "Specify one or more upstream ntp servers as an array."
    #
    # === Variables
    #
    # Here you should define a list of variables that this module would require.
    #
    # [*enc_ntp_servers*]
    #   Explanation of how this variable affects the funtion of this class and if it
    #   has a default. e.g. "The parameter enc_ntp_servers must be set by the
    #   External Node Classifier as a comma separated list of hostnames." (Note,
    #   global variables should not be used in preference to class parameters  as of
    #   Puppet 2.6.)
    #
    # === Examples
    #
    #  class { 'example_class':
    #    ntp_servers => [ 'pool.ntp.org', 'ntp.local.company.com' ]
    #  }
    #
    # === Authors
    #
    # Author Name <author@example.com>
    #
    # === Copyright
    #
    # Copyright 2011 Your name here, unless otherwise noted.
    #
    class example_class {

    }

```

Modelo para definições:

```
    # == Define: example_resource
    #
    # Full description of defined resource type example_resource here.
    #
    # === Parameters
    #
    # Document parameters here
    #
    # [*namevar*]
    #   If there is a parameter that defaults to the value of the title string
    #   when not explicitly set, you must always say so.  This parameter can be
    #   referred to as a "namevar," since it's functionally equivalent to the
    #   namevar of a core resource type.
    #
    # [*basedir*]
    #   Description of this variable.  For example, "This parameter sets the
    #   base directory for this resource type.  It should not contain a trailing
    #   slash."
    #
    # === Examples
    #
    # Provide some examples on how to use this type:
    #
    #   example_class::example_resource { 'namevar':
    #     basedir => '/tmp/src',
    #   }
    #
    # === Authors
    #
    # Author Name <author@example.com>
    #
    # === Copyright
    #
    # Copyright 2011 Your name here, unless otherwise noted.
    #
    define example_class::example_resource($basedir) {

    }

 ```

[s]<br>
Guto
