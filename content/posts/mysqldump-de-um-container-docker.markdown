---
title: "Mysqldump de um container Docker"
date: 2022-05-17
tags: ['MySQL', 'Docker', 'pt-BR']
draft: false
---

Este texto é uma tradução de um escrito que fiz em 2016, em inglês, e até hoje
ainda recebe comentários. Por isso, decidi traduzir para Português do Brasil,
mas, se quiser conferir o original, o link é [este
aqui](https://gist.github.com/devdrops/c59ee84504d128a7913a480b1191d66e). Para
melhorar o entendimento, vou adicionar alguns comentários e explicações a mais,
que possam enriquecer as informações.

<!--more-->

---

## Antes de prosseguir

Este tutorial parte do princípio que você tem [Docker](https://www.docker.com/)
instalado corretamente, com pelo menos uma imagem do banco de dados MySQL, em um
ambiente do SO Linux. O mesmo comando pode ser feito no SO Mac, mas algumas
alterações devem ser feitas para que o comando funcione no SO Windows.

## Como fazer

O comando abaixo gera a exportação do conteúdo de um banco de dados a partir de
uma imagem Docker em execução que contém um banco de dados MySQL em correto
funcionamento:

```bash
docker exec -i {MYSQL_IMAGEM} mysqldump -uroot -proot --databases {BANCODEDADOS} --skip-comments > {/CAMINHO/PARA/O/ARQUIVO/dump.sql}
```

Explicando o comando por partes:

- `docker exec` é o comando Docker para executar tarefas em imagens em execução.
- `-i` é a opção de modo interativo do comando `docker exec`, que mantém aberta
a entrada de dados para seu uso.
- `{MYSQL_IMAGEM}` é o nome da imagem contendo MySQL e que se encontra em
execução. Exemplo: `mysql:latest`.
- `mysqldump -uroot -proot` é a ferramenta de exportação e importação de dados
do MySQL. As opções `-uroot` e `-proot` indicam usuário e senha e devem ser
escritas dessa forma mesmo.
  - **Importante**: o usuário aqui usado deverá ter privilégios de exportar e/ou
  importar dados. Verifique se o usuário possui esses privilégios para ter
  sucesso nessa operação e evitar um cenário de `access denied`. Esses
  privilégios estão listados na documentação oficial, que deixo o link no final
  deste texto.
- `--databases {BANCODEDADOS}` é a opção para informar qual (ou quais) banco(s)
de dados você deseja exportar. `{BANCODEDADOS}` é, no caso, o nome do banco.
- `--skip-comments` é uma opção do `mysqldump` para que sejam ignorados
comentários que são inseridos no arquivo exportado.

## Referências

- [Docker](https://docker.com/)
- [mysqldump - A Database Backup
Program](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)
