---
title: "Vim: Definindo syntax Sem Mudar filetype"
date: 2023-07-24
tags: ['Vim']
draft: false
---

Essa é uma dica rápida para quem usa [Vim](https://www.vim.org/) no dia a dia.

## A Situação

Uns tempos atrás, estava observando um projeto feito com TypeScript + React, com arquivos `.tsx`. No meu caso, minha
instalação de Vim não sabia identificar a sintaxe (`syntax`) desses arquivos, deixando tudo com cara de texto comum.

Como gosto de usar estilo nos arquivos, comecei a pensar em como poderia trazer a sintaxe sem sobrescrever o tipo de
arquivo (`filetype`), que o Vim reconhecia como `typescriptreact`. Ou seja, eu queria manter o `filetype` e mudar apenas
a sintaxe, aplicando um estilo parecido.

## A Solução

A solução encontrada foi definir uma instrução de uma só linha para isso:

```vim
autocmd BufNewFile,BufRead *.tsx set syntax=typescript
```

Com essa instrução, que pode ser colocada no arquivo `.vimrc`, eu consigo reproduzir a mesma sintaxe usada para arquivos
TypeScript para esses arquivos com extensão `*.tsx`, tanto ao criar como ao abrir.

Você pode conferir essa instrução no meu repositório de "dot files"
[aqui](https://github.com/devdrops/my-dotfiles/blob/main/.vim/ftdetect/typescriptreact.vim).
