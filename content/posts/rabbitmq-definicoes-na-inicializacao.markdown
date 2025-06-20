---
title: "RabbitMQ: Carregando Definições na Inicialização"
date: 2025-06-20
tags: ['RabbitMQ', Docker', 'pt-BR']
draft: false
---

Dica rápida no uso de RabbitMQ, via Docker, para facilitar o trabalho ao criar um ambiente.

## O Problema

Você tem um projeto que precisa publicar mensagens (vamos chamá-lo de _producer_). É comum que, no momento de conexão,
esse _producer_ seja responsável em criar a `exchange`, a `queue`, o `binding`, ou seja, tudo o que é necessário para
que a publicação de mensagens ocorra com sucesso.

Isso pode trazer alguns problemas, como por exemplo:

- Caso a conexão se perca, ao refazê-la, o projeto pode acabar tendo que repetir esses passos.
- A aplicação agora, além de ter a tarefa de publicar mensagens,  acaba assumindo uma responsabilidade de criar um
  recurso que ela precisa para publicar mensagens.

## A Proposta

Podemos delegar essa responsabilidade para nível de infraestrutura usando **definitions**. Segundo a documentação do
RabbitMQ:

> _Nodes and clusters store information that can be thought of schema, metadata or topology. Users, vhosts, queues,_
> _exchanges, bindings, runtime parameters all fall into this category. This metadata is called definitions in RabbitMQ_
> _parlance._

Ou seja, o **definitions** contém as definições de vhosts, queues, exchanges, bindings, e muito mais. Vamos aqui focar
nos seguintes ítens: vhosts, queues, exchanges e bindings, que são fundamentais para que nosso _producer_ consiga
realizar sua tarefa de publicar mensagens - e mais nada :wink:

Primeiro, vamos criar nosso arquivo de **definitions**, que nada mais é do que um arquivo JSON. Nele vamos colocar as
seguintes informações:

- O vhost será a raiz, ou seja, `\`;
- A queue se chamará `users.queue`;
- A exchange se chamará `users.exchange`, do tipo `direct`;
- E teremos um binding entre a exchange e a queue.

Para isso, temos o conteúdo abaixo:

```json
{
  "vhosts": [
    {
      "name": "/"
    }
  ],
  "queues":[
    {
      "name": "users.queue",
      "vhost": "/",
      "durable": true,
      "auto_delete": true,
      "arguments": {}
    }
  ],
  "exchanges": [
    {
      "name": "users.exchange",
      "vhost": "/",
      "type": "direct",
      "durable": true,
      "auto_delete": false,
      "internal": false,
      "arguments": {}
    }
  ],
  "bindings":[
    {
      "source": "users.exchange",
      "vhost": "/",
      "destination": "users.queue",
      "destination_type": "queue",
      "routing_key": "users.routingkey",
      "arguments": {}
    }
  ]
}
```

Salvamos isso em `definitions.json`. Não há regras de nome, apenas uso esse para manter um padrão.

Uma vez feito isso, precisamos agora declarar para o RabbitMQ que temos uma **definitions** que deve ser considerada.
Isso pode ser feito de várias formas, como
[via terminal com `rabbitmqctl`](https://www.rabbitmq.com/docs/definitions#import) por exemplo, mas no nosso caso,
queremos que isso seja feito assim que o RabbitMQ for iniciado. Para fazer isso, crie um arquivo com o nome de
`rabbitmq.conf` com o conteúdo abaixo:

```sh
# Isto impede que o definitions seja recarregado caso nada de sua estrutura tenha sido alterada.
# É muito importante para evitar que o node fica carregando a definitions em todas as vezes.
definitions.skip_if_unchanged = true
# local_filesystem indica que a definitions se encontra como arquivo em disco local.
definitions.import_backend = local_filesystem
# Aqui você declara o local onde a definitions será colocada.
definitions.local.path = /etc/rabbitmq/definitions.json
```

Agora, com os dois arquivos. `definitions.json` e `rabbitmq.conf`, você pode testar executando o comando Docker abaixo:

```sh
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -p 25672:25672 \
  -p 5552:5552 \
  -v $(pwd)/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
  -v $(pwd)/definitions.json:/etc/rabbitmq/definitions.json \
  -e RABBITMQ_DEFAULT_VHOST="/" \
  -e RABBITMQ_DEFAULT_USER="guest" \
  -e RABBITMQ_DEFAULT_PASS="guest" \
  rabbitmq:3-management-alpine
```

Observe no comando que estamos compartilhando através de volumes os nossos arquivos.

Agora, acesse [localhost:15672](http://localhost:15672), faça o login e verifique que o conteúdo da nossa
**definitions** foi carregado com sucesso.

## Referências

- https://www.rabbitmq.com/docs/definitions
- https://www.rabbitmq.com/docs/definitions#import-on-boot

