---
layout: post
title: "Limpando fila de jobs do Puppet Dashboard"
date: 2013-06-22 10:11
comments: true
categories: puppet
---

## 1. Sobre o dashboard

O puppet dashboard é a ferramenta que nos permite acompanhar as mudanças que estão ocorrendo em um parque gerenciado pelo puppet, com ele todo o ciclo de vida de um node estará dentro do dashboard a partir do momento que um host começa a ser gerenciado.

## 2. Como ele funciona?

Para o puppet dashboard ser atualizado, a configuração do puppet agente precisa ter a diretiva report ativada, com isto, toda a vez que o puppet agent rodar na máquina gerenciada, um relatório será enviado ao master.

    report = true
    
Além desta diretiva, precisamos configurar o puppetmaster para enviar os reports recebido dos agentes para o dashboard. No dashboard, precisamos ativar os serviço workers, este serviço processa os relatórios recebidos do puppetmaster e alimenta a base do dashboard, isto atualiza os dados na interface web e você poderá então acompanhar o que está acontecendo em seus nodes.

## 3. Problemas comuns

Quando o dashboard para de atualizar as informações, você normalmente verá no canto superior esquerdo uma mensagem com o título 'Pending Tasks' e ao lado dela o número de tarefas que estão aguardando processamento.

Quando isto ocorre, provavelmente seu processo workers está parado, essa é a causa mais comum, basta reiniciar o processo e isso se resolve.

No debian

    /etc/init.d/puppet-dashboard-workers restart

No CentOS

    service puppet-dashboard-workers restart
    
No manual do Puppet Dashboard as dicas param ai, qualquer problema além disto você precisa se virar.

## 4. Problemas incomuns

Já ocorreu comigo duas vezes um problema bastante incomum que não estava relacionado aos processos workers, mas sim aos relatórios que ficam no diretório /usr/share/puppet-dashboard/spool, este diretório acolhe todos os arquivos recebidos do puppetmaster para processamento e inserção na base do dashboard.

### 4.1 Entendendo o problema

As vezes os workers não conseguem processar algum relatório e isso trava toda a fila de processamento. Na primeira vez que tive esse problema eu apelei, removi o dashboard, recriei a base e reinstalei o dashboard, meu dia estava corrido e era mais fácil e rápido fazer isto, contudo ontem (2013-06-21) eu tive novamente esse problema, desta vez com mais calma pesquisei e achei uma solução mais elegante no fórum do Puppet.

### 4.2 Solucionando o problema

A primeira coisa ser feita é analisar o arquivo /usr/share/puppet-dashboard/log/delayed_job.log para tentar achar o report que está quebrando o processamento, contudo nem sempre é fácil interpretar esse arquivo, principalmente se você tem uma quantidade relativamente grande de nodes sendo gerenciados pelo puppet.

Caso não consiga identificar o problema para destravar o processamento da fila, você pode simplesmente parar os workers, limpar o spool, limpar os jobs e reativar os workers, veja o passo a passo.

#### 4.2.1 Passo a passo

Acesse o diretório do spool

    cd /usr/share/puppet-dashboard/spool

Pare os processos worker

    /etc/init.d/puppet-dashboard-workers stop

Apague os relatórios YAML
  
    rm -rf *.yaml

Limpe os jobs

    rake jobs:clear RAILS_ENV=production

Inicie o workers

    /etc/init.d/puppet-dashboard-workers start

Isso deve ser suficiente para que seu dashboard volte a atualizar as informações referentes aos seus nodes.

### 4.3 Avaliando as possíveis causas

Não posso afirmar com 100% de certeza, contudo, alguns nodes que estive configurando em versões antigas do CentOS (5.x) reclamaram do certificado do master (CRL) e me apresentaram vários erros estranhos quando rodei o comando "puppet agent --test" neles. Acredito que  a tentativa de rodar o puppet sem sucesso pode ter gerado algum tipo de relatório mal formado que o workers não conseguiu processar, esse é o meu chute pelo menos, quando tiver dado mais concretos passo para vocês.˚

## 5. Referências

 * http://www.mail-archive.com/puppet-users@googlegroups.com/msg40277.html