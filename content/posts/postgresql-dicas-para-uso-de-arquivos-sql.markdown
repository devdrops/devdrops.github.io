---
title: "PostgreSQL: Dicas para Uso de Arquivos SQL"
date: 2022-08-03
tags: ['PostgreSQL', 'Docker', 'Docker-Compose']
draft: false
---

Recentemente, tive que fazer algumas operações com banco de dados PostgreSQL
através de arquivos SQL, e pude aprender um bocado de dicas. Vou compartilhar
esses conhecimentos abaixo, espero que te ajude também - como me ajudaram muito!

Ao final deste texto, você poderá:

- Criar banco de dados e tabelas apenas se os mesmos não existirem;
- Selecionar em qual banco de dados as tabelas serão criadas;
- Trabalhar com campos incrementais e valores padrão;
- Trabalhar com timezones;
- Trabalhar com criação automática de UUIDs;
- Tudo isso em um só arquivo SQL, para importar e criar o que você precisa.

---

## Antes de Prosseguir

As dicas aqui escritas foram feitas usando banco de dados
[PostgreSQL](https://www.postgresql.org/), usando imagens
[Docker](https://www.docker.com/) com
[Docker Compose](https://docs.docker.com/compose/). Os mesmos comandos puros de
PostgreSQL devem funcionar da mesma forma, sem problemas, apenas fazendo poucas
adaptações.

E outro detalhe muito importante: **eu não sou uma pessoa especialista em bancos
de dados**. Apenas trabalho com desenvolvimento backend da melhor forma
possível. Se prefere aprender dicas com quem manja mesmo do assunto, eu
recomendo demais as seguintes fontes:

- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/) é um site em inglês
cheio de dicas muito úteis.
- [Db4Beginners](http://db4beginners.com/) é o blog da mestra (pois mestrado)
 [Dani Monteiro](https://www.linkedin.com/in/danimonteirodba), para mim a pessoa
referência em banco de dados aqui no Brasil.

---

## Vamos por Partes

### A Ideia

Na situação, precisava de uma forma de criar/recriar o banco de dados através de
um único arquivo SQL, em um ambiente local com Docker e Docker Compose, de forma
que qualquer pessoa pudesse executar um só comando para cumprir essa tarefa com
100% de sucesso.

Bora lá? :wink:

### Criando o Ambiente com Docker e Docker Compose

Primeiro, eu precisava do ambiente com banco de dados onde a tarefa seria
executada. Para isso, criei o arquivo `docker-compose.yml` da seguinte forma:

```yaml
version: "3.7"

services:
    db:
        # Uma imagem oficial de PostgreSQL, na versão 9.6
        image: postgres:9.6
        # Volume apenas para persistir os dados
        volumes:
            - ./data:/var/lib/postgresql
        # Porta exposta para comunicação a partir do host
        ports:
            - "5432:5432"
        # Arquivo contendo variáveis de ambiente importantes
        env_file: ./.env

volumes:
    # Instruções fundamentais para o uso do volume
    data:
        driver: 'local'
```

O arquivo acima permite que você tenha uma imagem Docker do PostgreSQL, na
versão 9.6, compartilhando a porta padrão para sua máquina host.

Para completar, também criei o arquivo `.env`, com variáveis de ambiente
fundamentais para uso com Docker, conforme abaixo:

```conf
# Coloque aqui o valor para o usuário padrão do PostgreSQL
POSTGRES_USER=DBUSER
# E coloque aqui a senha desse usuário padrão
POSTGRES_PASSWORD=PREULA
```

As variáveis ali serão lidas e criadas como variáveis de ambiente apenas no
container de banco de dados, assim que a imagem do serviço `db` estiver em
execução.

> **Dica**: crie um arquivo `.env.sample` e adicione o arquivo `.env` no arquivo
`.gitignore`, para evitar que você coloque no seu repositório senhas ou qualquer
conteúdo sensível :wink:

Uma vez criados os arquivos acima, podemos começar a construção do arquivo SQL.

### Arquivo SQL: Criando o Banco de Dados

Vamos agora criar o banco de dados. Comece o arquivo SQL assim:

```sql
/* application será o nome do banco de dados */
CREATE DATABASE application;
```

Isso é suficiente para criar o banco, mas apenas da primeira vez. E se o banco
já existe, e você não quer um erro ao importar o arquivo?

É comum nos bancos de dados usar a abordagem _crie X apenas se não existe_,
porém aí vem uma parte chata do PostgreSQL: para criar bancos de dados, essa
opção não existe. Ao invés disso, muitos textos na internet indicam uma outra
abordagem, que funciona assim:

1. Veja se o banco existe.
2. Se não, crie.
3. Se sim, não crie.

Para fazer isso, vamos fazer uma mudança no nosso arquivo SQL:

```sql
/* application será o nome do banco de dados */
SELECT 'CREATE DATABASE application'
  WHERE NOT EXISTS
    (SELECT FROM pg_database WHERE datname = 'application')\gexec
```

A tabela `pg_database` contém diversas informações dos bancos de dados, e a
coluna `datname` é o nome deles. O que essa instução faz é, primeiro, checar se
existe o registro `application` nessa tabela; se não, executa o comando `CREATE
DATABASE application'; se sim, ignora o comando. Tudo em uma só linha (a
indentação aqui foi só para organizar).

Dessa forma, podemos repetir a importação quantas vezes a gente quiser que o
banco só será criado se não existir, sem gerar erros :+1:

O próximo passo é definir o banco onde vamos criar as tabelas.

### Arquivo SQL: Definindo Qual Banco de Dados Usar

O PostgreSQL vem com muitos utilitários (sério, é um banco de dados muito
bacana). Um deles, o `\c`, permite selecionar o banco de dados em um arquivo SQL
(e também em comandos SQL no terminal).

Vamos usá-lo no nosso arquivo:

```sql
/* Selecionando o banco application */
\c application
```

De agora em diante, todos os comandos serão realizados no banco `application`.

O próximo passo é trabalhar com timezones.

### Arquivo SQL: Definindo Timezones

Uma necessidade era de operar ações no banco de dados com campos do tipo
`TIMESTAMP`. Porém, era interessante usar o timezone local nesse tipo, e para
isso o PostgreSQL tem um tipo especial, `TIMESTAMPTZ`. Para isso, precisei
adicionar a seguinte instrução:

```sql
/* Definindo timezone */
SET timezone = 'America/Sao_Paulo';
```

Com isso, todo campo do tipo `TIMESTAMPTZ` vai usar o fuso horário
`America/Sao_Paulo`.

Feito isso, a próxima coisa a fazer era facilitar meu trabalho com UUIDs.

### Arquivo SQL: Gerando UUIDs de Forma Automática

A aplicação em si precisava de valores
[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier), também
conhecidos como _Universally Unique IDentifier_ (IDentificador Único Universal).
Esse tipo de campo é importante quando queremos oferecer um identificador
externo para usuários sem expor um identificador de uso exclusivamente interno,
que pode expor informações sobre a sua aplicação que você não quer que usuários
saibam, sem perder a funcionalidade de identificar recursos.

Isso pode ser feito de várias formas. Uma das mais comuns é delegar a criação
desse campo no código da aplicação, usando libs e etc. Mas eu queria uma
solução direto no banco de dados, e o PostgreSQL tem uma solução para isso: a
extensão [uuid-ossp](https://www.postgresql.org/docs/current/uuid-ossp.html).
Para usá-la, é preciso fazer a declaração abaixo:

```sql
/* UUID */
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

Felizmente aqui podemos usar a declaração `IF NOT EXISTS`! Com a linha acima, os
próximos comandos podem fazer uso das funções dessa extensão.

E por falar nisso, o próximo passo é justamente criar tabelas.

### Arquivo SQL: Criando Tabelas

Eu busquei uma forma de criar tabelas da forma mais simples possível, delegando
tarefas mais complexas para o banco de dados. Fiz da seguinte forma:

```sql
/* Tabela users */
CREATE TABLE IF NOT EXISTS users (
  id         SERIAL       PRIMARY KEY,
  uuid       uuid         UNIQUE DEFAULT uuid_generate_v4 (),
  name       VARCHAR(100) NOT NULL,
  document   CHAR(11)     UNIQUE NOT NULL,
  created_on TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

O que está acontecendo:

- Cria-se uma tabela, chamada `users`, apenas se a mesma não existir.
- Nela, temos um campo `id` que é declarado como `SERIAL`, uma forma mais
simples de dizer que é de valor numérico inteiro e auto incrementável. E também,
esse campo é a chave primária da tabela.
- Depois, temos o campo `uuid`, que será o identificador externo de cada
registro. Esse campo é do tipo `uuid`, de valor único, e onde o valor padrão
será obtido pela função `uuid_generate_v4()`, vinda da extensão `uuid-ossp`.
- Os campos `name` e `document` são campos comuns dos tipos `VARCHAR` e `CHAR`.
- E o campo `created_on` é do tipo `TIMESTAMPTZ`, influenciado pela declaração
de fuso horário. Seu valor padrão é a data e hora atuais.

Dessa forma, posso fazer comandos de `INSERT` da seguinte forma:

```sql
INSERT INTO users (name, document) VALUES ('José das Couves', '0111406');
```

Informando apenas `name` e `document`, enquanto o banco cria os demais valores
para mim. Olha só:

```sql
INSERT INTO users (name, document) VALUES ('José das Couves', '0111406') RETURNING *;

 id |                 uuid                 |      name       |  document   | created_on
----+--------------------------------------+-----------------+-------------+-------------------------------
  1 | b758aac0-ddd1-42e7-a6da-ccd7a0173857 | José das Couves | 0111406     | 2022-08-04 03:16:15.822798+00
(1 row)

INSERT 0 1
```

### Arquivo SQL: Resultado do Arquivo

Com todas as alterações acima, nosso arquivo SQL ficou assim:

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
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

/* Tabela users */
CREATE TABLE IF NOT EXISTS users (
	id         SERIAL       PRIMARY KEY,
	uuid       uuid         UNIQUE DEFAULT uuid_generate_v4 (),
	name       VARCHAR(100) NOT NULL,
	document   CHAR(11)     UNIQUE NOT NULL,
	created_on TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

Agora, finalmente, como podemos importar o arquivo para cumprir sua tarefa.

### Arquivo SQL: Importando para o Container Docker

Com o ambiente em execução, podemos executar o comando abaixo:

```shell
docker-compose exec -T db psql < ./database.sql -U DBUSER
```

O comando acima vai executar a instrução no container em execução. É importante
usar ali o usuário informado no arquivo `.env`, pois esse usuário terá
privilégios para todos os comandos do arquivo.

Ao executar o comando acima, podemos ver o seguinte resultado:

```shell
CREATE DATABASE
You are now connected to database "application" as user "DBUSER".
SET
CREATE EXTENSION
CREATE TABLE
```

E, se você repetir o comando ainda com o ambiente em execução:

```shell
You are now connected to database "application" as user "DBUSER".
SET
NOTICE:  extension "uuid-ossp" already exists, skipping
CREATE EXTENSION
NOTICE:  relation "users" already exists, skipping
CREATE TABLE
```

De forma segura, o comando foi executado sem sobrescrever nada, e sem gerar
erros :tada:
