---
layout: post
title: "Octopress com textos justificados"
date: 2013-01-25 08:07
comments: true
categories: octopress
---

Por padrão o Octopress não justifica os posts e páginas, vou mostrar como resolver isto.

Edite o arquivo sass/custom/_styles.scss

    vim sass/custom/_styles.scss 

E insira o código abaixo no final do arquivo

```
article p{
    text-align:justify;
}
```

Gere novamente o conteúdo e ative o preview

    rake generate && rake preview
    
Acesse seu blog para verificar se os textos estão justificados

    http://localhost:4000

Referências

    https://github.com/imathis/octopress/issues/409

    
[s]<br>
Guto
