---
title: "tmux: Movendo entre Sessões"
date: 2023-11-17
draft: true
---

Dica quente para quem usa [tmux](https://github.com/tmux/tmux).

## A Situação

Eu criei uma sessão para um projeto (vamos chamá-la de **A**), e nela eu tinha as seguintes janelas:

- 0 para editor;
- 1 para execução de comandos;
- 2 para git;
- 3 para tarefa de monitoramento;

E essa janela 3

## A Solução

`tmux move-window -t <session_name>:<session_window>`
