---
title: "MongoDB e Docker"
date: 2023-06-27
tags: ['MongoDB', 'Docker', 'mongosh']
draft: false
---

Anotação simples e rápida de como usei MongoDB com Docker apenas para aprendizado.

## Antes de Seguir...

Todos os comandos abaixo foram feitos em ambiente Linux (no caso, Linux Mint), usando apenas Docker.

## A Situação

Eu queria usar o `mongosh` via terminal em uma imagem Docker, para executar comandos apenas via linha de comando.

Fui atrás de uma imagem que tivesse o que precisava, e escolhi a
[mongodb/mongodb-community-server](https://hub.docker.com/r/mongodb/mongodb-community-server). Porém, ao executá-la,
percebi que o entrypoint (comando configurado para iniciar a execução da imagem) já estava definido para inicializar o
banco de dados, o que impedia de usar o `mongosh` "de cara". Sim, eu poderia driblar o entrypoint, mas aí eu perderia a
inicialização automática do `mongod` e outras dependências, o que iria atrapalhar tudo.

Tentei outras formas até finalmente conseguir o que queria, com os comandos abaixo:

## A Solução

1. Criei uma network para que as imagens Docker "conversem" entre si:

```bash
docker network create mongodb-network
```

2. Iniciei a execução da imagem de MongoDB que será o meu servidor (onde os comandos terão efeito), em modo desanexado
   (_detached_), usando a network criada no passo anterior e expondo a porta padrão **27017**:

```bash
docker run -d --name mongodb-server --network mongodb-network -ti --rm -p 27017:27017 mongodb/mongodb-community-server:latest
```

3. Iniciei a execução de outra imagem de MongoDB que será o cliente (onde vou executar os comandos), que vai se conectar
   à imagem de servidor, usando a mesma network, abrindo uma sessão do shell:

```bash
docker run --name mongodb-client --network mongodb-network -ti --rm mongodb/mongodb-community-server:latest bash
```

4. Com o shell da minha imagem cliente em execução, executei o comando para me conectar à imagem servidor:

```bash
mongosh mongodb-server:27017
```

Importante lembrar de usar o nome dado à imagem servidor, para que seja reconhecida na network :wink:

5. Feito esses passos, consegui meu objetivo!

```bash
mongodb@9f2fac4c832d:/$ mongosh mongodb-server:27017
Current Mongosh Log ID: 649af9c2e2abf96f648a6456
Connecting to:          mongodb://mongodb-server:27017/?directConnection=true&appName=mongosh+1.10.1
Using MongoDB:          6.0.6
Using Mongosh:          1.10.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2023-06-27T14:37:12.016+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2023-06-27T14:37:12.735+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2023-06-27T14:37:12.737+00:00: vm.max_map_count is too low
------

test>

```

Apenas para validar, fui checar os logs da imagem servidor para garantir o sucesso de tudo:

```bash
docker logs -f mongodb-server
```

```json
{"t":{"$date":"2023-06-27T15:01:22.880+00:00"},"s":"I",  "c":"NETWORK",  "id":22943,   "ctx":"listener","msg":"Connection accepted","attr":{"remote":"172.18.0.3:37788","uuid":"d970dc63-cd9a-4ec2-a2e0-0bc2f07d2a2e","connectionId":6,"connectionCount":1}}
{"t":{"$date":"2023-06-27T15:01:22.886+00:00"},"s":"I",  "c":"NETWORK",  "id":51800,   "ctx":"conn6","msg":"client metadata","attr":{"remote":"172.18.0.3:37788","client":"conn6","doc":{"application":{"name":"mongosh 1.10.1"},"driver":{"name":"nodejs|mongosh","version":"5.6.0|1.10.1"},"platform":"Node.js v16.20.1, LE","os":{"name":"linux","architecture":"x64","version":"5.4.0-152-generic","type":"Linux"}}}}
```

