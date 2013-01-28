---
layout: post
title: "Octopress com lista de categorias"
date: 2013-01-25 10:22
comments: true
categories: octopress
---

O Octopress não vem com um plugin para fazer a lista de categorias de forma nativa, pesquisando encontrei várias soluções a que mais gostei foi a dotnetguy, acompanhe o que eu fiz.

### Instalando

Primeiro criei um arquivo chamado category_list_tab.rb no diretório plugins do octopress

    vim category_list_tab.rb

Coloque o conteúdo abaixo no arquivo

```
module Jekyll
  class CategoryListTag < Liquid::Tag
    def render(context)
      html = ""
      categories = context.registers[:site].categories.keys
      categories.sort.each do |category|
        posts_in_category = context.registers[:site].categories[category].size
        category_dir = context.registers[:site].config['category_dir']
        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase)
        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n"
      end
      html
    end
  end
end

Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
```

Depois crie o arquivo category_list.html em source/_includes/custom/asides

    vim sources/_includes/custom/asides/category_list.html

Insira o conteúdo abaixo

{% include_code category_list.html %}

E para finalizar edite o arquivo _config.yml e insira uma referência para o category_list

    asides/category_list.html

Veja como ficou no meu arquivo

    default_asides: [custom/asides/about.html, asides/twitter.html, asides/recent_posts.html, custom/asides/category_list.html, asides/github.html,  asides/delicious.html, asides/pinboard.html, asides/googleplus.html]
    
Agora basta acessar o blog e verificar se a lista de categorias apareceu.

### Referências

    http://www.dotnetguy.co.uk/post/2012/06/25/octopress-category-list-plugin/
