---
title: "PostgreSQL: Inicializar Banco de Dados com Docker"
date: 2023-10-24
tags: ['PostgreSQL', 'Docker']
draft: false
---

Este texto serve como um complemento a um texto anterior daqui do blog, [PostgreSQL: Dicas para Uso de Arquivos SQL]().

## Antes de Prosseguir

Esse texto oferece uma abordagem do PostgreSQL com imagem Docker. Caso esse não seja o seu caso, recomendo conferir a
documentação do banco de dados e/ou das ferramentas do ambiente que você está usando.

## A Situação

Na criação de um projeto, eu já tinha um arquivo SQL preparado para a criação do meu banco de dados, semelhante ao
arquivo descrito no post citado acima. Vou chamar aqui nesse texto esse arquivo de `env/db/database.sql`, como abaixo:

```sql
/* application será o nome do banco de dados */
SELECT 'CREATE DATABASE application'
	WHERE NOT EXISTS
		(SELECT FROM pg_database WHERE datname = 'application')\gexec

/* Selecionando o banco application */
\c application

/* Definindo timezone */
SET timezone = 'America/Sao_Paulo';

/* UUID */
BEGIN;
  CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
COMMIT;

/* Tabela users */
BEGIN;
  CREATE TABLE IF NOT EXISTS users (
	  id         SERIAL       PRIMARY KEY,
	  uuid       uuid         UNIQUE DEFAULT uuid_generate_v4 (),
	  name       VARCHAR(100) NOT NULL,
	  document   CHAR(11)     UNIQUE NOT NULL,
	  created_on TIMESTAMPTZ  NOT NULL DEFAULT NOW()
  );
COMMIT;
```

Observação: adicionei o uso de transações nesse projeto, pois diferente da amostra acima, o arquivo original possui
várias operações que precisavam de uma garantia de transação. Contudo, as instruções acima podem ser executadas sem
`BEGIN;` e `COMMIT;` :wink:

Em outros projetos, a criação de um banco de dados "pronto para ação" era lenta, dependendo da disponibilidade do
Postgres. Eu precisava de algo rápido, quase instantâneo, e efetivo.

## A Solução

Lendo a documentação da imagem oficial do PostgreSQL [no Docker Hub](https://hub.docker.com/_/postgres), encontrei algo
que parecia atender o que eu queria.

No tópico **Initialization scripts**, há um parágrafo explicando:

> _If you would like to do additional initialization in an image derived from this one, add one or more `*.sql`,_
> _`*.sql.gz`, or `*.sh` scripts under `/docker-entrypoint-initdb.d` (creating the directory if necessary). After the_
> _entrypoint calls `initdb` to create the default `postgres` user and database, it will run any `*.sql` files, run any_
> _executable `*.sh` scripts, and source any non-executable `*.sh` scripts found in that directory to do further_
> _initialization before starting the service._

Traduzindo (de forma simples):

> _Para executar qualquer ação ao initializar uma imagem derivada da imagem oficial, você pode colocar seus arquivos_
> _`*.sql`, `*.sql.gz` ou `*.sh` (não executável) no diretório `/docker-entrypoint-initdb.d`, que esses arquivos serão_
> _executados logo após a criação do banco de dados e usuário padrões e antes de inicializar o serviço._

Então, criei uma nova imagem Docker a partir da imagem oficial, `./env/Dockerfile.db` como abaixo:

```dockerfile
FROM postgres:9.6

COPY ./db/database.sql /docker-entrypoint-initdb.d/
```

Nele, há 2 instruções:

Com a instrução `FROM`, eu defino que esta imagem tem como base a imagem `postgres`, na tag `9.6`. E com a instrução
`COPY`, eu copio o arquivo `./env/db/database.sql` na minha máquina para o diretório `/docker-entrypoint-initdb.d` da
minha imagem.

Com isso, ao inicializar o meu ambiente, a imagem criada com o arquivo acima executava o meu arquivo SQL imediatamente,
sem que eu precise me preocupar com essa tarefa.
