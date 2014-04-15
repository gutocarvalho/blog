---
layout: post
title: "Octopress com aside de comentários recentes"
date: 2013-04-25 13:35
comments: true
categories: octopress
---

Pesquisando sobre aside (widget) para o disqus no octopress, encontrei um post no site [arshad](http://arshad.github.io/blog/2012/05/04/recent-comments-in-octopress/) que explica como fazer isto em poucos passos, vamos para a explicação.

## 1. Criando aside

Vamos criar um arquivo chamado **recent_comment.html** dentro de /caminho/para/octopress/source/_includes/custom/asides

    touch /path/to/octopress/source/_includes/custom/asides/recent_comment.html

## 2. Adicionando código ao arquivo

```
<section>
  <h1>Recent Comments</h1>
  <div id="dsq-recentcomments" class="dsq-widget"><script type="text/javascript" src="http://disqus.com/forums/{{ site.disqus_short_name }}/recent_comments_widget.js?hide_avatars=1;num_items=4"></script></div>
</section>
```

### 2.1 Sobre os parâmetros disponíveis

Para esconder avatar

    hide_avatars=0 (values : 0 show, 1 hide)

Quantidade de comentários a serem mostrados

    num_items=4

Tamanho máximo de caracteres por comentários

    xcerpt_length=140

## 3. Adicione o aside ao _config.yml

Procure o parâmetro default asides e insira após recent_posts.html

    custom/asides/recent_comment.html

Após o ajuste a configuração do parâmetro ficará desse jeito

    default_asides: [custom/asides/links.html, asides/recent_posts.html, custom/asides/recent_comment.html, asides/github.html, asides/twitter.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html]

## 4. Faça o rebuild do seu site

Agora basta fazer o rebuild

    rake generate && rake deploy
    
Após finalizar o comando, você verá o aside funcionando, simples assim.

[s]<br>
Guto
