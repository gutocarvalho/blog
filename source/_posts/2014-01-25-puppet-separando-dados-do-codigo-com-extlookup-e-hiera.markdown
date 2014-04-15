---
layout: post
title: "Puppet, separando dados do código com extlookup e hiera"
date: 2014-01-25 07:53
comments: true
toc: true
categories: puppet
---

## 1. Separando dados de código

Durante a implantação do Puppet nós vamos migrando todos aqueles 
procedimentos manuais, artesanais, sejam checklists, sejam tutoriais, 
fazendo isto de forma gradativa e progressiva, no entanto, ao escrever 
novo código, para deixá-lo flexível, é necessário utilizar variáveis,
constantes, fatos e parâmetros personalizados em classes e definições.

Para entender este conceito de forma mais fácil, procure lembrar se você já precisou escrever uma classe genérica, ou mesmo uma base class para aplicar em vários nodes, porém em um node, ou um grupo de nodes, algumas pequenas mudanças deveriam ser levadas em conta. Quando temos esse tipo de cenário nós normalmente usamos e abusamos de tratamentos condicionais, expressões, cases e seletores em busca de um código mais flexível e inteligente, com isto, nosso código acaba se tornando uma mescla de dados (variáveis, constantes, fatos, parâmetros e atributos) com código, veja esse exemplo simples:

```
$dnsserver    = '8.8.8.8'
$searchdomain = 'hacklab'
 
file { '/etc/resolv.conf':
  ensure  => present
  content => "search ${searchdomain}\n nameserver ${dnsserver}\n",
}
```

Observe que os dados e o código estão no mesmo arquivo que chamamos de manifest, e se desejamos alterar por exemplo o endereço **8.8.8.8** ou o domínio de pesquisa **hacklab** precisaremos editar o manifest que também possui o código, e isto pode gerar algum tipo de problema caso a edição não seja cuidadosa e altere a sintaxe do manifest.

Pensando nisto, a Puppetlabs criou algumas ferramentas para separar dados de código, facilitando a edição e preservando os manifests de nossos módulos.

## 2. O pioneiro: Extlookup

O primeiro recurso criado para resolver este problema foi o extlookup, com ele é possível especificar um local externo para buscar dados.

O exemplo abaixo foi retirado do site do R.I.Piennar (http://www.devco.net) o criador do extlookup.

```
class snmp::config {

   if $fqdn == "www.mesintogravida.com.br" {
      $contactname = 'webmaster'
      $contactemail = 'webmaster@mesintogravida.com.br'
   }
 
   elsif $domain == "mesintogravida.com.br" {
      $contactname = 'contato'
      $contactemail = 'contato@mesintogravida.com.br'
   }
   else {
      $contactname = 'Jessica Macedo'
      $contactemail = 'jessica@mesintogravida.com.br'
   }
}
```

Observe que para cada tratamento algum valor é associado a uma variável, tudo dentro do código, veja que são as mesmas variáveis repetidas diversas vezes, não há reutilização de código, temos escrever mais e ainda corremos o risco de alterar os dados e causar um problema de sintaxe no código, basta esquecer de fechar uma aspa para gerar um erro em diversos nodes.

No exemplo acima, primeiro é verificado o FQDN, se for www.mesintogravida.com.br valores específicos são associados as duas variáveis abaixo, caso o FQDN não seja o esperado, é verificado então se o domínio é mesintogravida.com.br, se for, outros valores serão associados as variáveis, caso o domínio não seja hacklab, um valor comum será associado aquelas variáveis no último tratamento.

Usando o conceito do extlookup, faríamos apenas a seguinte definição dentro de nosso manifest (removendo todo o tratamento condicional if elsif else).

    $contactname = extlookup("contactname")
    $contactemail = extlookup("contactemail")
    
Bem simples e elegante, contudo, em nosso site.pp precisamos determinar como o extlookup vai funcionar, o primeiro passo é determinar a precedência de pesquisa , faremos isto da seguinte forma:

    $extlookup_precedence = ["%{fqdn}", "domain/%{domain}", "common"]
    
E depois será necessário definir o local onde estão os arquivos com os dados:

    $extlookup_datadir = /etc/puppet/manifests/extdata

No caso do extlookup o formato para armazenar os dados é o CSV, dentro do diretório **/etc/puppet/manifests/extdata** serão criados arquivos CSV como os exemplos abaixo:

Arquivo **/etc/puppet/manifests/extdata/hosts/www.mesintogravida.com.br.csv**

    contactemail,webmaster@mesintogravida.com.br
    contactname,webmaster

Arquivo **/etc/puppet/manifests/extdata/domain/mesintogravida.com.br.csv**

    contactemail,contato@mesintogravida.com.br
    contactname,contato

Arquivo **/etc/puppet/manifests/extdata/common.csv**

    contactemail,jessica@mesintogravida.com.br
    contactname,jessica
    
Com a pesquisa seguindo a precedência definida no site.pp, economizamos 15 linhas em nosso manifest e não precisamos fazer o tratamento condicional.
    
### 2.1 Entendendo pesquisa extlookup
   
Entenda a forma de pesquisa que será feita pelo extlookup conforme configurações feitas no site.pp

#### 2.1.1 FQDN
   
Primeiro a ordem de pesquisa verifica se existe um arquivo dentro de **/etc/puppet/manifests/extdata/hosts** com o nome **www.mesintogravida.com.br.csv**, se existir e ele vai ler e atribuir os valores presentes no arquivo para as variáveis.

#### 2.1.2 DOMAIN

Caso ele não encontre um arquivo com o nome completo do host (fqdn), ele vai procurar por um arquivo com o nome do domínio dentro de **/etc/puppet/manifests/extdata/domain** com o nome **mesintogravida.com.br.csv**, se existir, ele vai ler o arquivo e atribuir o valor presente no arquivo para as variáveis.

#### 2.1.3 COMMON

Caso ele não encontre um arquivo com o nome do domínio, ele lerá por último o arquivo **/etc/puppet/manifests/extdata/common/common.csv**, que pela lógica terá as configurações comuns para nodes que não tenham tratamentos diferenciados.

### 2.2 Vantagens no uso do extlookup

Observe que retiramos os dados de dentro do código, agora só é preciso editar arquivos CSV (comma separeted values) para mudar o comportamento de um manifest, isto facilita a administração, torna nossos manifests mais flexíveis e simples de manipular. 

Nós também diminuímos o risco de falhas decorrentes de mudanças de dados em nossos manifests e além disto, otimizamos o código de nosso módulo, no exemplo mostrado nós economizamos cerca de 15 linhas.

### 2.2 Limitações do extlookup

Infelizmente o extlookup só consegue ler dados de arquivos CSV (comma separeted values), cada linha no CSV traz algum tipo de informação separada por virgulas, isso para alguns cenários engessa e limita as possibilidades de tratamento.

Com o tempo os admins foram querendo mais recursos e mais facilidades de uso, suporte a outros tipos de armazenamento e o extlookup precisou evoluir. É importante falar que ele foi um passo importante antes da criação de seu sucessor chamado HIERA.

Antes de entrarmos no hiera é importante dizer que o extlookup continua disponível e funcionando no Puppet 3.x.x.

## 3. O sucessor: Hiera

O Hiera é uma ferramenta que faz lookup (busca) por dados de configuração de seus manifests, ele foi criado pelo R.I.Piennar (que também crio ou o extlookup) para tornar o puppet mais eficiente e permitir que você defina dados relacionados a um node sem precisar repetir código.

Podemos dizer que o HIERA é uma evolução do extlookup, porém ele foi escrito do zero, logo é um evolução inspirada no extlookup, mas com código completamente novo.

### 3.1 Mas porque o Hiera?

O Hiera torna o puppet melhor ao permitir separar os dados do código, ele retira essencialmente os dados de dentro de uma manifest e armazena externamente. É através do HIERA que você irá consultar estes dados conforme sua necessidade - tal como o extlookup fazia.

O Hiera facilita:

* A configuração de seus nodes: Podemos ter configurações com dados default ou então vários níveis de overrides nestes dados.

* A reutilização de módulos: Você não precisa editar código, precisa apenas colocar os dados no hiera.

* A publicação do seu módulo: Você não precisa mais ficar limpando vários pedaços do seu módulo para publicá-lo no forge ou github, os dados estão fora do manifest, tornado essa tarefa bem mais simples.

* A elaboração e um código sem repetição!

Com o Hiera você pode:

* Escrever dados comuns para a maioria dos seus nodes

* Sobrescrever dados para nodes específicos

* Definir a hierarquia de pesquisa de dados

### 3.2 Hiera datasources

No momento o HIERA suporta nativamente os datasources YAML e JSON.

Exemplo de datasource YAML

```
---
# array
apache-packages:
    - apache2
    - apache2-common
    - apache2-utils

# string
apache-service: apache2

# interpolated facter variable 
hosts_entry: sandbox.%{fqdn}

# hash
sshd_settings: 
    root_allowed: "no"
    password_allowed: "yes"

# alternate hash notation
sshd_settings: {root_allowed: "no", password_allowed: "yes"}

# to return "true" or "false"
sshd_settings: {root_allowed: no, password_allowed: yes}
```

Exemplo de datasource JSON

```
{   
    "apache-packages" : [
    "apache2",
    "apache2-common",
    "apache2-utils"
    ],

    "hosts_entry" :  "sandbox.%{fqdn}",

    "sshd_settings" : {
                        "root_allowed" : "no", 
                        "password_allowed" : "no"
                      }
}
```

O Hiera não suporta o datasource CSV.

### 3.2.1 Outros tipos de datasource

Você certamente vai encontrar projetos no github que permitem que o hiera use outros tipos de backend, eu particularmente recomendo os projetos do desenvolvedor Craig Dunn, ele criou 3 plugins que permitem que o HIERA use backends GPG, HTTP e MYSQL.

Visite os projetos:

    https://github.com/crayfishx/hiera-gpg
    https://github.com/crayfishx/hiera-http
    https://github.com/crayfishx/hiera-mysql

## 5. Instalando Hiera

Para instalar o Hiera habilite os repositórios puppetlabs no Debian ou CentOS.

    apt.puppetlabs.com (se for debian)
    yum.puppetlabs.com (se for centos/rhel)

Para instalar no Debian

    aptitude install hiera

Para instalar no CentOS

    yum install hiera

Se preferir instale usando o RAL (tanto faz o OS).

    puppet resource package hiera ensure=present

Se você estiver utilizando o Puppet 3 ele já deve estar instalado em seu ambiente. Aqui no meu ambiente estou usando CentOS 6.5 - 64 Bits e Puppet 3.4.2.

## 5. Configurando Hiera

A primeira coisa a se fazer é criar o arquivo **/etc/puppet/hiera.yaml** e definir a hierarquia de pesquisa, o backend e o local onde ele deverá procurar os arquivos, veja o modelo abaixo:

```
---
:hierarchy:
   - host/%{fqdn}
   - host/%{hostname}
   - os/%{osfamily}
   - domain/%{domain}
   - common
:backends:
   - yaml
:yaml:
   :datadir: /etc/puppet/hieradata 
```

Observe que na configuração do arquivo, estamos definindo a seguinte precedência de pesquisa:

1. HOST/FQDN
2. HOST/HOSTNAME
3. OS/OSFAMILY
4. DOMAIN/DOMAIN
6. COMMON

E definimos também que o backend é o YAML e que os arquivos YAML ficarão no diretório /etc/puppet/hieradata, crie o diretório se ele não existir.

Neste modelo eu estou utilizando o backend YAML.

### 5.1 Tipos de datasources 

O Hiera pode trabalhar tanto com data sources estáticos, quanto dinâmicos (com interpolação de tokens e variáveis).

Exemplo de datasource estático (common):

```
---
:hierarchy:
   - common
:backends:
   - yaml
:yaml:
   :datadir: /etc/puppet/hieradata 
```

Exemplo de datasource dinâmico (osfamily):

```
---
:hierarchy:
   - %{osfamily}
:backends:
   - yaml
:yaml:
   :datadir: /etc/puppet/hieradata 
```

Exemplo de datasource combinado (dinâmico e estático):

```
---
:hierarchy:
   - host/%{fqdn}
   - host/%{hostname}
   - os/%{osfamily}
   - domain/%{domain}
   - common
:backends:
   - yaml
:yaml:
   :datadir: /etc/puppet/hieradata 
```

## 6. Criando Manifest

Agora vamos criar um manifest sem separação de dados e logo após vamos mostrar o mesmo manifest utilizando o Hiera.

### 6.1 Sem Hiera

{% codeblock lang:puppet %}
class webapps {

    ### tratamentos e definiçoes #############

    # primeiro tratamento: verifica fqdn

    case $::fqdn {

        'defiant.hackab': {
            $nameserver = [ "8.8.8.8", "8.8.4.4" ]
        }
        'voyager.hacklab': {
            $nameserver = [ "208.67.222.222", "208.67.220.220" ]
        }
        default: {
            $nameserver = [ "10.0.1.1" ]
        }

    }

    # segundo tratamento: verifica dominio

    case $::domain {
        'hacklab': {
            $user = 'admin'
        }
        default: {
            $user = 'gutocarvalho'
        }
    }

    # terceiro tratamento: verifica o sistema operacional

    case $::operatingsystem {
        Debian: {
            $editor_package = 'vim'
            $http_package = 'apache2'
            }
        RedHat: {
            $editor_package = 'vim-enhanced'
            $http_package = 'httpd'
            }
    }

    # definindo uma constante simples

    $puppet_ipaddr = '192.168.56.60'

    ### declarando recursos ###############################

    # gerenciando resolv.conf (fqdn)

    file { '/etc/resolv.conf':
        ensure  => present,
        owner   => 'root',
        group   => 'root',
        mode    => '0644',
        content => template('webapps/resolv.conf.erb'),
    }

    # gerenciando usuarios (domain)

    user { $user:
        ensure => present,
    }

    # gerenciando pacotes (operatingsystem)

    package { $editor_package:
        ensure => present,
    }

    package { $http_package:
        ensure => present,
    }

    # gerenciando host puppet (common)

    host { 'puppet.hacklab':
        ensure     => present,
        ip         => $puppet_ipaddr,
        host_alias => 'puppet',
    }
}
{% endcodeblock %}

Veja o template resolv.conf.erb definido neste manifest.

```
<% hiera_nameserver.each do |val| -%>
  nameserver <%= val %>  
<% end -%>
```

Observe quantas vezes repetimos as mesmas variáveis em cada tratamento.

### 6.2 Com Hiera

Agora vamos utilizar o HIERA e modificar o manifest, veja como fica:

{% codeblock lang:puppet %}
class webapps::hiera {

    ### tratamentos e definiçoes #############

    $hiera_nameserver = hiera_array('nameserver')
    $hiera_user = hiera_array('users')
    $hiera_editor_package = hiera('editor_package')
    $hiera_http_package = hiera('http_package')
    $hiera_puppet_ipaddr = hiera('puppet_ipaddr')

    ### declarando recursos ###############################

    # gerenciando resolv.conf (fqdn)

        file { '/etc/resolv.conf':
                ensure  => present,
                owner   => 'root',
                group   => 'root',
                mode    => '0644',
                content => template('webapps/resolv.conf.erb'),
        }

        # gerenciando usuarios (domain)

        user { $hiera_user:
                ensure => present,
        }

        # imprimindo nome do ambiente (environment)

        notify { "Ambiente de ${hiera_myenv}": }

        # gerenciando pacotes (operatingsystem)

        package { $hiera_editor_package:
                ensure => present,
        }

        package { $hiera_http_package:
                ensure => present,
        }

        # gerenciando host puppet (common)

        host { 'puppet.hacklab':
                ensure       => present,
                ip           => $hiera_puppet_ipaddr,
                host_aliases => puppet,
        }

} 
{% endcodeblock %}

Observe que não há dados no manifest, apenas código, agora vamos ver como criar os arquivos que conterão os dados.

#### 6.2.1 Arquivos YAML

Acompanhe abaixo o conteúdo dos arquivos.

Arquivo /etc/puppet/hieradata/host/defiant.hacklab.yaml
```
---
nameserver:
   - 8.8.8.8
   - 8.8.4.4
```
   
Arquivo /etc/puppet/hieradata/host/voyager.hacklab.yaml

```
---
nameserver:
   - 208.67.222.222
   - 208.67.220.220
```

Arquivo /etc/puppet/hieradata/domain/hacklab.yaml

```
---
nameserver:
   - 10.0.0.10
user:
   - admin
```

Arquivo /etc/puppet/hieradata/os/Debian.yaml
```
---
editor_package: vim
http_package: apache2
```

Arquivo /etc/puppet/hieradata/os/RedHat.yaml
```
---
editor_package: vim-enhanced
http_package: httpd
```

Arquivo /etc/puppet/hieradata/common.yaml
```
---
nameserver:
  - 10.0.1.1
  - 8.8.8.8
  - 8.8.4.4
  - 208.67.222.222
  - 208.67.220.220
user:
  - gutocarvalho
puppet_ipaddr:
   - 192.168.56.60
```

Com o Hiera os dados ficam fora do arquivo deixando o código mais limpo, coeso, evitando repetição e erros de edição.

## 7. Comandos e consultas Hiera

Para invocar o hiera para uma pesquisa seguindo a hierarquia definida.

    # hiera ntp_server

Para invocar o hiera especificando parâmetros de busca

    # hiera ntp_server -yaml web01.example.com.yaml 
    
É bem simples fazer a pesquisa e testar se vai retornar o que você está esperando.

## 8. Referências

Puppetlabs Misc

* https://puppetlabs.com/blog/the-problem-with-separating-data-from-puppet-code/
* https://puppetlabs.com/blog/first-look-installing-and-using-hiera/

Hiera

* http://docs.puppetlabs.com/hiera/1/index.html
* http://docs.puppetlabs.com/hiera/1/installing.html
* http://docs.puppetlabs.com/hiera/1/configuring.html
* http://docs.puppetlabs.com/hiera/1/lookup_types.html
* http://docs.puppetlabs.com/hiera/1/data_sources.htm
* http://docs.puppetlabs.com/hiera/1/variables.html
* http://docs.puppetlabs.com/hiera/1/puppet.html
* http://docs.puppetlabs.com/hiera/1/complete_example.html
* http://docs.puppetlabs.com/hiera/1/command_line.html
* http://docs.puppetlabs.com/hiera/1/custom_backends.html
* http://docs.puppetlabs.com/hiera/1/release_notes.html

Misc

* http://pt.wikipedia.org/wiki/Comma-separated_values
* http://www.devco.net/archives/2009/08/31/complex_data_and_puppet.php

[s]<br>
Guto

