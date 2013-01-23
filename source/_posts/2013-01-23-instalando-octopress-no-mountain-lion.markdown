---
layout: post
title: "Instalando Octopress no Mountain Lion"
date: 2013-01-23 09:23
comments: true
categories: tecnologia, octopress
---

## 1. Sobre octopress

O Octopress é um framework para o Jekyll. O Jekyl por sua vez é um gerador de sistes estáticos. Quando usamos o Jekyll diretamente precisamos criar toda a estrutura HTML, CSS, Javascript e configurá-la, com o octopress isso já tudo já está pronto.

Tanto o Octopress quanto o Jekyll foram escritos em Ruby.

Podemos instalar o Octopress diretamente em nosso notebook, podemos utilizar git para manter seus arquivos e empurrar estes para um servidor na internet via git ou rsync, simples assim.

No octopress utilizamos a linguagem markdown para escrever os posts, e tudo é feito pela linha de comando, veja alguns exemplos de manipulação do Octopress.

Criar um post

    rake new_post['Post sobre Octopress']

Editar um novo post

    vim sources/_posts/2013-01-23-post-sobre-octoprress.markdown
    
Criar uma página

    rake new_page['howtos']
    
Editar uma nova página

    vim sources/howtos/index.markdown        
    
Gerar o site após mudanças em configurações/posts/páginas/tema.

    rake generate
    
Ver um preview do site no seu notebook

    rake preview

Após rodar o preview acesse 

    http://localhost:4000

Para publicar uma mudança em seu site de produção

    rake deploy

Existem duas formas de fazer o deploy, uma via RSYNC e outra via GIT, normalmente o deploy via GIT é utilizado para alimentar o GITHUB PAGES e o RSYNC para enviar para algum outro host, no meu caso dreamhost.

## 2. Minhas necessidades
    
Recentemente eu reinstalei o meu OSX e precisei preparar todo o ambiente para rodar o octopress novamente, abaixo segue meu roteiro para resolver isto usando o homebrew e RVM.

## 3. Pré-requisitos

O octopresss roda em virtualmente qualquer sistema operacional que tenha suporte ruby, porém no meu caso estou usando OSX Mountain Lion.

- instale o último xcode
- instale console tools do xcode

Partindo do pressuposto de que você já tem o xcode e xcode console tools, vamos prosseguir.

## 4. Homebrew

O homebrew é um gerenciador de pacotes para OSX, tal como fink ou macports, porém no momento acho o homebrew mais interessante e com mais opções do que os demais, vale lembrar que se for usar o homebrew, remova antes o fink ou macports para evitar conflitos.

### 4.1 instalando homebrew

Instale o homebrew

    ruby -e "$(curl -fsSkL raw.github.com/mxcl/homebrew/go)"

## 5. RVM

O rvm é uma ferramenta que te permite selecionar a versão do ruby para aquela aplicação que você deseja utilizar, no meu caso eu uso puppet que necessita de um ruby mais antigo, porém também uso o octopress que precisa do ruby mais novo, portando o rvm é uma excelente alternativa para gerenciar este tipo de coisa.

### 5.1 instalando rvm

Atualize os índices do homebew

    brew update

Instale o git

    brew install git

Ative o uso de fórmulas externas

    brew tap homebrew/dupes

Instale pacotes necessarios para compilação do ruby

    brew install autoconf automake apple-gcc42

Ajuste seu path no arquivo .bash_profile ou .profile

    export PATH="/usr/local/sbin:/usr/local/bin:$PATH"

Instale o rvm e a última versão do ruby

    curl -L https://get.rvm.io | bash -s stable --ruby
    
Instale as dependências para o ruby (recomendando pelo rvm)    
    
    rvm pkg install openssl

Se quiser, instale o ruby 1.8.7

    rvm install 1.8.7 --with-gcc=clang --without-tcl --without-tk

Se quiser, instale o ruby 1.9.2

    rvm install 1.9.2 --with-gcc=clang --without-tcl --without-tk

Defina que o ruby 1.9.3 é o default

    rvm usr 1.9.3 --default
    
## 6. Octopress

### 6.1 octopress, minha primeira instalação

Siga estes passos se você deseja instalar o octopress pela primeira vez.

Faça o clone do octopress via git

    git clone git://github.com/imathis/octopress.git 
    
Acesse o diretório octopress

    cd octopress

Instale o bundler

    gem install bundler

Rode o bundle para instalar todas as dependências do octopress

    blunde install

Instale o tema default do octopress

    rake install

### 6.2 octopress, reinstalei meu OSX e agora?

Se você reinstalou seu OSX e quer fazer seu octopress funcionar novamente, faça o seguinte:

Acesse seu diretório do octopress

    cd /caminho/para/octopress

Instale o bundler

    gem install bundler

Rode o bundle para instalar as dependências do octopress

    bundle install

Está tudo pronto para voltar a utilizar seu octopress, é só sair blogando.

### 6.3 utilizando o octopress

Como configurar

    http://octopress.org/docs/configuring/

O básico para blogar

    http://octopress.org/docs/blogging
    
Posts com trechos de código

    http://octopress.org/docs/blogging/code/    
    
Sobre plugins nativos

    http://octopress.org/docs/blogging/plugins/
    
Tema e personalização deste

    http://octopress.org/docs/theme/

Mais informações em 

    http://octopress.org/docs/

## 7. Dicas Soltas

Eu uso o Mou como editor Markdown para Octopress, é possível alterar o código do octopress para ele abrir o Mou no momento em que você criar um post ou página.

É perfeitamente possível blogar através do VIM, eventualmente eu faço isto, porém acho o MOU mais confortável.

Para comentários eu uso o Disqus.

O restante é o padrão do Octopress.

Eu uso muito o plugin nativo Codeblock.

Eu comecei a migrar os post do Wordpress - quase 500 posts, mas achei cansativo e chato arrumar post a post devido aos problemas de sintaxe markdown do script exitwp, acabei deixando o wordpress no ar e quando estiver com tempo volto a pensar nessa migração.

## 8. Referências 

* http://stackoverflow.com/questions/11664835/mountain-lion-rvm-install-1-8-7-x11-error
* http://www.webdevotion.be/blog/2012/06/05/ruby-rvm-homebrew-xcode43/
* http://amaras-tech.co.uk/article/182/Mountain_Lion_Setup,_Part_2_Homebrew_%26_RVM
* http://blog.dojimmy.com/blog/2012/12/07/como-instalar-o-octopress-no-mac-osx-moutain-lion/


[s]<br>
Guto