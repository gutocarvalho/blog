---
layout: post
title: "Bash Hacks: Histórico de comandos"
date: 2012-11-28 17:20
comments: true
categories: console
---

## 1. Entendendo o history

O history é um comando muito utilizado por sysadmins, no entanto, existem algumas funcionalidades muito interessantes e pouco exploradas nesta ferramenta.

Vamos rodar o comando history:

    # history
    
Acompanhe a saída:

```
...
492  reboot
493  ps aux
494  df -h
495  vim /etc/hosts
```

A informação  que estamos vendo é armazenada em memória e também em disco. Todo o usuário possui em seu diretório pessoal um arquivo com seu histórico de comandos, se este estiver utilizando o interpretador de comandos bash, por default, o arquivo com estes dados tem o nome .bash_history, esse arquivo armazena os comandos digitados em sua última sessão ativa, isto significa que no momento do logoff esse arquivo recebe todo o fluxo de comandos digitados por aquele usuário em sua última sessão.

O nome e o path para este arquivo é definido através da variável HISTFILE, por default a variável tem o valor abaixo:

    HISTFILE=/root/.bash_history

Para verificar o valor desta variável para um usuário, use um dos comandos abaixo:

    # echo $HISTFILE

ou utilize o comando set, veja o exemplo:

    # set |grep HISTFILE

Se desejar mudar o local de armazenamento do arquivo e o nome do arquivo, você pode especificar isto manualmente, veja o exemplo:

    # export HISTFILE=/home/usuario/logs/.commands_history

Se modificarmos o valor da variável isto vai valer para sessão em que estamos logados, para que esta configuração se torne permanente, esta linha pode ser inserida no seu arquivo .bashrc ou no arquivo .bash_profile, com isso durante o logon a variável será carregada e ao fazer logoff o arquivo receberá os comandos digitados pelo usuário.

Apesar do .bash_history conter todo o histórico de comandos das últimas sessões, o comando history também traz o histórico de comandos da sessão atual, combinando estes comandos com o conteúdo do arquivo. A diferença é que os dados mais recentes ainda estão em memória e serão descarregados no arquivo durante o logoff do usuário.

### 1.1 History time format

É possível personalizar as informações de histórico, inserindo data e hora em que o comando foi executado, para isto basta configurar a variável HISTTIMEFORMAT, veja o exemplo abaixo:

    # export HISTTIMEFORMAT="%F %T "
    
Ao configurar esta variável a saída do history ficará assim:

```
...
547 2012-11-25 12:08:27 cd /etc/puppet/
548 2012-11-25 12:08:28 ls -al
549 2012-11-25 12:08:31 cd manifests/classes/ 550 2012-11-25 12:08:31 ls
551 2012-11-25 12:08:37 vim linux-server.pp 552 2012-11-25 12:08:42 vim users.pp
553 2012-11-25 12:08:57 openssl passwd -1 ...
```

### 1.2 History file size

Você pode também especificar quantas linhas o arquivo .bash_history deve ter, isto é controlado através da variável HISTFILESIZE, para setá-la use o comando abaixo:

    # export HISTFILESIZE=1500

Neste exemplo estamos especificando que o arquivo .bash_history deve possuir até no máximo 1500 linhas. O history sabe manipular o arquivo e apagar as entradas mais antigas para manter os 1500 comandos recentes.

### 1.3 History size

Existe também a variável HISTSIZE que determina quantos comandos serão
retornados ao usar o comando HISTORY, veja o exemplo a abaixo: 

    # export HISTSIZE=100

Neste exemplo eu digo ao comando history que ele deve me retornar os últimos 100 comandos. 

### 1.4 History control 

Existe uma variável chamada HISTCONTROL que nos permite especificar alguns filtros no histórico de comandos do usuário, vamos conhecer esses filtros.

#### 1.4.1 ignoredups

Se quiser ignorar os comandos idênticos repetidos em sequência no histórico, podemos utilizar o filtro ignoredumps, acompanhe o teste:

```
# w
# w
# w
# history | tail -4
```

Acompanhe a saída do history:

```
44  w
45  w
46  w
47  history | tail -4
```

Observe que o comando w apareceu 3 vezes em sequência, se quisermos ignorar as repetirções durante a auditoria, podemos especificar o ignoredups para os usuários de nossos servidores, com isto, não teríamos repetições durante as pesquisas, veja o exemplo abaixo.

    # export HISTCONTROL=ignoredups

Acompanhe o teste:

```
# uname -a
# uname -a
# uname -a
# history | tail -3￼￼
```
Acompanhe a saída do history:

```
56  export HISTCONTROL=ignoredups
57  uname -a
58  history | tail -3
```

Veja que apesar de termos digitado três vezes o comando **uname -a**, o histórico apresentou apenas uma entrada, essa é a função do ignoredups.

#### 1.4.2 erasedups

O filtro erasedups remove entradas repetidas em todo o histórico, mesmo que elas não estejam em sequência, veja o exemplo abaixo:

```
# export HISTCONTROL=erasedups
# pwd
# /etc/init.d/puppet stop
# history | tail -3
```

Acompanhe a saída do history:

```
38  pwd
39  /etc/init.d/puppet s top
40  history | tail -3
```

Agora vamos repetir alguns comandos:

```
# ls -ler
# /etc/init.d/puppet stop
# history | tail -6
```
Acompanhe a saída do history:

```
35  export HISTCONTROL=erasedups
36  pwd
37  history | tail -3
38  ls -ltr
39  /etc/init.d/puppet stop
40  history | tail -6
```
Note que que o comando **/etc/init.d/puppet stop** não foi repetido no histórico, essa é a função do erasedups.

#### 1.4.3 ignorespace

O filtro ignorespace nos ajuda a evitar que alguns comandos digitados entrem no history, com ele se começarmos o comando com um espaço, este será ignorado, veja o exemplo:

```
# export HISTCONTROL=ignorespace
# ls -ltr
# pwd
#  puppet agent --test
# history | tail -3
```
Acompanhe a saída

```
67  ls -ltr
68  pwd
69  history | tail -3
```
Observe que o comando **puppet agent --test** não apareceu no histórico pois antes do comando eu dei um espaço.

#### 1.5 History ignore

Ao setar a variável HISTIGNORE estamos orientando o history a ignorar alguns comandos, evitando inseri-los no histórico, acompanhe o exemplo:

```
# export HISTIGNORE="pwd:ls:ls -ltr:"
# pwd
# ls
# ls -ltr
# service httpd stop
# history | tail -3
```
Acomopanhe a saída do history:

```
79  export HISTIGNORE="pwd:ls:ls -ltr:"
80  service httpd stop
81  history
```

Veja que os comandos pwd, ls e lst -ltr não foram mencionados pois estão declarados como valor do HISTIGNORE.

#### 1.6 Fixando configurações de history

Quando especificamos um valor para uma variável, mesmo se essa variável for exportada, esse procedimento só valerá para aquela sessão de login e seus processos filhos, ao deslogar perderemos todas as configurações de history.

Para fixar essas configurações e fazer com que elas sejam carregadas durante o logon, devemos inseri-las no arquivo .bashrc ou no arquivo .bash_profile, desta forma estaremos garantindo que as configurações sejam permanentes para nosso usuário.

#### 1.7 Pesquisando e executando comandos no history

Existem diversas formas de fazer pesquisa no histórico de comandos de usuários, vamos conhecer algumas delas:

Para fazer a paginação do resultado podemos usar o more ou less:

    # history|more
    # history|less

Se quiser ver os últimos 100 comandos do history use:

    # history 100

Se quiser pesquisar um padrão específico use:

    # history|grep comando

Para ver o último comando digitado você pode usar a seta para cima, combinar as teclas CTRL+P ou ainda concatenar a saída com o tail, veja o exemplo:

    # history|tail -1

Se quiser repetir o último comando digitado use:

    # !!

ou

    # !-1

Onde -1 é o último comando, -2 o penúltimo e assim por diante.

Se você quiser executar algum comando do history, por exemplo o comando 510, basta chamá-lo desta forma:

    # !510

Outras duas dicas importantes, se por alguma razão desejar rodar o último
comando com um padrão específico, mesmo que você não se lembre exatamente
do comando e parâmetros, faça o seguinte:

    #!?un?

No caso acima, o último comando recente como o padrão digitado era o
'uname -a', e ele foi executado, agora caso saiba exatamente o comando,
use o formato abaixo:

    #!uname

Quando desejar fazer uma pesquisa ativa, pode ainda usar a combinação de teclas CTRL+R, após digitar a combinação de teclas você precisa especificar um padrão de pesquisa, após digitar o padrão vá pressionando a combinação para continuar a busca, quando encontrar o comando desejado pressione ESC para que ele vá para o prompt ou RETURN caso deseje executá-lo.


## 2. Amarrando as pontas

O history é um comando importante, é fundamental saber utilizá-lo de forma ágil e eficiente, as principais informações do history estão no manual do BASH, basta pesquisar para encontrar o que precisa.


[s] <br>
Guto
