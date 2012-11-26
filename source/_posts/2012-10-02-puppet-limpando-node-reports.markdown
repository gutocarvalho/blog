---
layout: post
title: "Puppet: Limpando Node Reports"
date: 2012-10-02 11:56
comments: true
categories: [ tecnologia, puppet ]
---

### Node Reports

Quando usamos o puppet-dashboard, precisamos habilitar em cada agente o parâmetro `report = true`. Com isso, a cada execução do puppet em um node, um relatório é enviado para o puppetmaster, este por sua vez empurra os dados recebidos para o puppet-dashboard.

Se quiseremos, podemos gravar esses **relatórios** em disco, um arquivo YAML será gerado no servidor puppetmaster no diretório /var/lib/puppet/reports a cada execução do puppet em nossos nodes.

{% blockquote %}
Mas qual seria a finalidade de manter esses arquivos?
{% endblockquote %}

É muito simples, você pode popular a base do puppet-dashboard utilizando esses arquivos no caso de um corrompimento de sua base (mysql) de produção.

No caso de um desastre, para importar esses arquivos bastaria rodar o comando abaixo:

    puppet-dashboard rake RAILS_ENV=production reports:import

Eu mantenho por segurança cerca de 2 semanas de arquivos, assim caso ocorra alguma coisa eu posso re-popular os dados do dashboard dos últimos 14 dias.

### 1. Transtornos ocasionais

O problema de armazenar esses arquivos é que o disco vai enchendo e pode acontecer o famoso DISK FULL, fazendo seu puppetmaster parar de funcionar.

### 2. Limpando diretório

Precisamos evitar esse tipo de transtorno, vou mostrar algumas formas de limpar o disco, deixando apenas os relatórios das duas últimas semanas.


#### 2.1 Usando Find

Podemos fazer isto usando simplesmente o comando abaixo.

    find /var/lib/puppet/reports -name *.yaml -mtime +14 -exec rm -rf {} \;

Esse comando vai apagar todos os arquivos com extensão `yaml` do diretório `reports` com idade maior ou igual a `14` dias.

#### 2.2 Usando Find dentro de Cron no Puppet

Podemos ainda criar no puppet um cron para executar isto diariamente, veja o exemplo abaixo:

{% codeblock lang:puppet %}
class cleanreports {

	cron { "clean reports":
		command => "find /var/lib/puppet/reports -name *.yaml -mtime +14 -exec rm -rf {} \;"
		user => root, 
		hour => 01, 
		minute => 00,  
	}
} 
{% endcodeblock %}

Neste exemplo estou orientando que o comando seja executado diariamente às 01:00 da manhã.

#### 2.3 Usando o recurso Tidy do Puppet

Ou podemos ainda usar o tidy que é um recurso nativo do puppet para limpeza de arquivos em diretórios com base em um critério pré-definido.

{% codeblock lang:puppet %}
class cleanreports {
 
        tidy { "/var/lib/puppet/reports/":
                age     => "2w",
                matches => "*.yaml",
                recurse => true ,
        }
}
{% endcodeblock %}

Veja que estou dizendo ao Tidy que ele deve limpar arquivos com extensão `yaml` , com idade igual ou maior a `2 semanas`, de forma recursiva.

O único inconveniente desta alternativa é que ele vai rodar o tidy a cada execução do puppet no master, e caso sua janela seja curta (10 min) isso pode ser um problema, causando um aumento excessivo do load de seu puppetmaster de forma regular.

### 3. Amarrando as pontas

Existem diferentes formas de evitar um disk full, podemos fazer a limpeza de arquivos usando comandos do linux (find), podemos usar recursos nativos do puppet (tidy), podemos combinar comandos do sistema (find) com recursos do puppet (cron), isso vai depender de sua criatividade e necessidade.

Eu apresentei apenas três exemplos que podem resolver problemas deste tipo,
porém cada cenário normalmente pede uma solução específica.

É sempre bacana ler a documentação do puppet para entender e encontrar recursos que possam nos ajudar a resolver um problema, eu não conhecia o TIDY, mas vi que ele pode ser muito útil nesta situação.

Ainda há muito a ler e aprender, sigo compartilhando o que descubro.

[s]<br>
Guto
