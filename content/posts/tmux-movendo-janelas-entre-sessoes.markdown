---
title: "tmux: Movendo Janelas entre Sessões"
date: 2024-03-04
tags: ['tmux', 'pt-BR']
draft: false
---

Dica quente para quem usa [tmux](https://github.com/tmux/tmux).

<!--more-->

## A Situação

Normalmente, eu trabalho com várias sessões, uma para cada projeto e cada uma com as suas janelas. Eu criei uma sessão
para um projeto (vou chamá-la de **A**), e nela eu tinha as seguintes janelas, cada uma para a sua finalidade:

```
└── A
    ├── Janela 0: Vim
    ├── Janela 1: Comandos de terminal
    ├── Janela 2: Comandos de git
    └── Janela 3: Monitorando imagens Docker
```

Só que, por engano, eu criei uma nova janela com um comando que eu gostaria de organizar em outra sessão, só para eu não
me confundir (uma sessão por projeto), ficando assim:

```
└── A
    ├── Janela 0: Vim
    ├── Janela 1: Comandos de terminal
    ├── Janela 2: Comandos de git
    ├── Janela 3: Monitorando imagens Docker
    └── Janela 4: Lens desktop
```

Como eu não queria perder a janela para recriá-la em outro projeto do zero, procurei então uma forma de mover a janela
do índice **4** para uma nova sessão.

## A Solução

Com tmux, isso é molezinha, basta usar o comando `move-window`. Segundo a doc:

```
move-window [-abrdk] [-s src-window] [-t dst-window]
              (alias: movew)
        This is similar to link-window, except the window at src-window is moved to dst-window.  With -r, all windows in
        the session are renumbered in sequential order, respecting the base-index option.
```

Criei então uma nova sessão, que vou chamá-la de **B**, como abaixo:

```
└── B
    └── Janela 0: Vim
```

Naveguei até a janela que quero mover, na sessão **A**, janela **4**, e usei então o comando `move-window` pelo prompt
de comando do tmux (prefixo + `:`), como abaixo:

```
move-window -t B:1
```

Dessa forma, obtive o seguinte resultado:

```
├── A
│   ├── Janela 0: Vim
│   ├── Janela 1: Comandos de terminal
│   ├── Janela 2: Comandos de git
│   └── Janela 3: Monitorando imagens Docker
└── B
    ├── Janela 0: Vim
    └── Janela 1: Lens desktop
```
