---
layout: post
title: "Octopress com alinhamento em listas"
date: 2013-01-25 08:13
comments: true
categories: octopress
---

O Octopress vem com uma configuração de CSS em que as listas (UL/OL) vem com os itens desalinhados em relação ao texto, isso não deixa o post bonito, para resolver é bastante simples.

Edite o arquivo sass/custom/_styles.scss

    vim sass/custom/_styles.scss

Insira o código abaixo no final do arquivo

```
article {
  ol, ul {
    padding-left: 3em;
  }
}
```

Gere novamente o conteúdo e ative o preview

    rake generate && rake preview
    
Acesse o site para verificar

    http://localhost:4000/

Referências

    https://github.com/imathis/octopress/issues/417
