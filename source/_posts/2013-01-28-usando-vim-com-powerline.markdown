---
layout: post
title: "Usando VIM com Powerline no Mountain Lion"
date: 2013-01-28 15:50
comments: true
categories: vim-hacks
---

## Sobre a Powerline para o VIM

A Powerline é um daqueles plugins essenciais para o VIM, ele personaliza a statusline  deixando com um visual bem mais agradável e polido, facilitando a leitura e o entendimento dos modos de operação do VIM. A powerline foi criada pelo [Kim Silkebækken](https://github.com/Lokaltog) e está disponível no Github.

Veja um exemplo retirado do site [spifftastic.net](http://www.spifftastic.net/).

{% img center /images/posts/2013-01-28/powerline-ruby.png [Powerline Normal Mode] %}

Conheci a Powerline através de um amigo (Magnun Leno) e desde então sempre uso esse plugin no VIM.

Eu estou usando diretamente no VIM, a maioria das pessoas no mundo OSX prefere usá-la no MacVim, a configuração é muito similar por isso não vou entrar em detalhes do MacVim. Abaixo veja algumas screenshots do vim-powerlone em meu notebook pessoal.

{% img center /images/posts/2013-01-28/powerline-normal-mode.png [Powerline Normal Mode] %}

{% img center /images/posts/2013-01-28/powerline-insert-mode.png [Powerline Insert Mode] %}

{% img center /images/posts/2013-01-28/powerline-visual-mode.png [Powerline Visual Mode] %}

{% img center /images/posts/2013-01-28/powerline-replace-mode.png [Powerline Replace Mode] %}

Note que ele apresenta o modo do VIM (Normal, Inserção, Visual, Substituição, etc…), mostra o nome do arquivo que está sendo editado, o tipo do arquivo (unix), a codificação (utf-8), o filetype (puppet), o percentual lido, a linha e a posição do caracter na linha.

## PreReqs

* OSX Mountain Lion;
* Homebrew instalado em seu OSX;
* iTerm2 instalado e usando terminal em modo xterm-256color;
* Conhecimentos básicos sobre vim e vimrc;

## Instalando Vundle

Para instalar o powerline vamos usar o vundle que é um gerenciador de pacotes para o vim, primeiro faça o clone do vundle com o comando abaixo:

    git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle

Depois ajuste seu .vimrc para usar o vundle

```
"vundle
set nocompatible
filetype off

set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" let Vundle manage Vundle
" required! 
Bundle 'gmarik/vundle'

 " My Bundles here:
 "
Bundle 'Lokaltog/vim-powerline'

 " original repos on github
 " Bundle 'tpope/vim-rails.git'

 " non github repos
 " Bundle 'git://git.wincent.com/command-t.git'
 
filetype plugin on
filetype plugin indent on
```
Pronto, a configuração do vundle já foi adicionada, inclusive já mencionei que desejamos instalar o vim-powerline na lista de bundles, agora abra o vim e vamos pedir para o vundle instalar os plugins.

    :BundleInstall

## Configurando a Powerline

Agora abra seu arquivo .vimrc e adicione as linhas abaixo:

```
set laststatus=2
set encoding=utf-8
let g:Powerline_symbols = 'fancy'
```
Isso é suficiente para configurar e ativar a powerline, porém, ao abrir o Vim você verá que ela não está bonita como nas screenshost acima, isso acontece pois é necessário fazer um patch na fonte do OSX para ter aquele visual polido.

## Instalando fontes no OSX

### Fontes que já receberam patchs

Existem diversas fontes já preparadas para powerline, basta procurar no github. 

Para facilitar, escolha uma fonte com patch a partir do site abaixo:

    https://github.com/Lokaltog/powerline-fonts
 
Para este exemplo eu fiz o download da fonte Menlo.

    wget https://github.com/Lokaltog/powerline-fonts/blob/master/Menlo/Menlo%20Regular%20for%20Powerline.otf

Depois de fazer download, rode o comando abaixo

    open Menlo+Regular+for+Powerline.otf
     
Quando a fonte abrir, clique em instalar fonte.

Configure a fonte nova como opção default em seu iTerm2, reinicie o iTerm2.

Abra o Vim e veja se a powerline está funcionando com o visual correto.

### Fontes sem patchs

Caso deseje aplicar um patch em uma fonte específica, primeiro verifique se você tem o fontforge instalado rodando o comando abaixo:

```
$ fontforge -version
Copyright (c) 2000-2011 by George Williams.
Executable based on sources from 13:48 GMT 22-Feb-2011-D.
Library based on sources from 13:48 GMT 22-Feb-2011.
fontforge 20110222
libfontforge 20110222
```

Caso ele não esteja instalado, atualize os índices do homebrew

    brew update

Instale o fontforge

    brew install fontforge --use-clang

Faça o patch da sua fonte

    fontforge -script ~/.vim/bundle/vim-powerline/fontpatcher/fontpatcher CONSOLA.TTF 
    
Acompanhe a saída    

```    
Copyright (c) 2000-2012 by George Williams.
 Executable based on sources from 14:57 GMT 31-Jul-2012-D.
 Library based on sources from 14:57 GMT 31-Jul-2012.
The following table(s) in the font have been ignored by FontForge
  Ignoring 'DSIG' digital signature table
```

Veja a fonte criada

    -rw-r--r--  1 gutocarvalho  staff  96584 28 Jan 19:40 CONSOLA-Powerline.TTF
    -r--r--r--  1 gutocarvalho  staff  98520 28 Jan 19:39 CONSOLA.TTF

Após fazer o patch, clique duas vezes na fonte gerada e instale em seu OSX.

Configure a fonte nova como opção default em seu iTerm2, reinicie o iTerm2.

Abra o vim e veja se a powerline está funcionando com o visual correto.

## Amarrando as pontas

Esse plugin é um grande utilitário para o VIM ou MACVIM, recomendo para UNIX e LINUX.

No Linux não é necessário aplicar patch nas fontes.

O vim-powerline está congelado, seu criador está desenvolvendo um novo projeto para substituí-lo chamado apenas de [powerline](https://github.com/Lokaltog/powerline), este foi todo escrito em python, no momento ainda é beta.

## Referências

* https://github.com/Lokaltog/vim-powerline/blob/develop/doc/Powerline.txt
* https://github.com/Lokaltog/vim-powerline
* https://github.com/gmarik/vundle



