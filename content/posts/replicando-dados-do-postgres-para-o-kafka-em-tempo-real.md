---
title: "Replicando dados do Postgres para o Kafka em tempo real"
date: 2024-04-11
tags: ['PostgreSQL', 'Kafka', 'Debezium', 'Docker', 'pt-BR']
draft: true
---

Como fiz para monitorar alterações em um banco de dados Postgres diretamente para um tópico no Kafka, em tempo real.

<!--more-->


1.

docker run --name postgres -p 5000:5432 -e POSTGRES_HOST_AUTH_METHOD=trust debezium/postgres

2.

docker run -it --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper

3.

docker run -it --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka

4.

docker run -it --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my-connect-configs -e OFFSET_STORAGE_TOPIC=my-connect-offsets -e ADVERTISED_HOST_NAME=$(echo $DOCKER_HOST | cut -f3 -d'/' | cut -f1 -d':') --link zookeeper:zookeeper --link postgres:postgres --link kafka:kafka debezium/connect

5.

psql -h localhost -p 5000 -U postgres
CREATE DATABASE inventory;
CREATE TABLE dumb_table(id SERIAL PRIMARY KEY, name VARCHAR);

6.

curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '
{
 "name": "inventory-connector",
 "config": {
 "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
 "tasks.max": "1",
 "database.hostname": "postgres",
 "database.port": "5432",
 "database.user": "postgres",
 "database.password": "postgres",
 "database.dbname" : "inventory",
 "database.server.name": "dbserver1",
 "database.whitelist": "inventory",
 "database.history.kafka.bootstrap.servers": "kafka:9092",
 "database.history.kafka.topic": "schema-changes.inventory",
 "topic.prefix": "testtopic"
 }
}'


bin/kafka-console-consumer.sh --topic testtopic.public.dumb_table --from-beginning --bootstrap-server 172.17.0.4:9092

bin/kafka-topics.sh --list --bootstrap-server 172.17.0.4:9092 localhost:2181



{"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"default":0,"field":"id"},{"type":"string","optional":true,"field":"name"}],"optional":true,"name":"testtopic.public.dumb_table.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"default":0,"field":"id"},{"type":"string","optional":true,"field":"name"}],"optional":true,"name":"testtopic.public.dumb_table.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false,incremental"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"sequence"},{"type":"int64","optional":false,"field":"ts_us"},{"type":"int64","optional":false,"field":"ts_ns"},{"type":"string","optional":false,"field":"schema"},{"type":"string","optional":false,"field":"table"},{"type":"int64","optional":true,"field":"txId"},{"type":"int64","optional":true,"field":"lsn"},{"type":"int64","optional":true,"field":"xmin"}],"optional":false,"name":"io.debezium.connector.postgresql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"int64","optional":true,"field":"ts_us"},{"type":"int64","optional":true,"field":"ts_ns"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"name":"event.block","version":1,"field":"transaction"}],"optional":false,"name":"testtopic.public.dumb_table.Envelope","version":2},"payload":{"before":null,"after":{"id":5,"name":"EITA 5"},"source":{"version":"2.6.0.Final","connector":"postgresql","name":"testtopic","ts_ms":1712872115106,"snapshot":"false","db":"inventory","sequence":"[\"23751552\",\"23752040\"]","ts_us":1712872115106271,"ts_ns":1712872115106271000,"schema":"public","table":"dumb_table","txId":559,"lsn":23752040,"xmin":null},"op":"c","ts_ms":1712872115134,"ts_us":1712872115134298,"ts_ns":1712872115134298000,"transaction":null}}


- https://hevodata.com/learn/postgresql-kafka-connector/
- https://debezium.io/documentation/reference/stable/connectors/postgresql.html
- https://github.com/docker-library/docs/blob/master/postgres/README.md
- https://docs.confluent.io/kafka/operations-tools/kafka-tools.html
- https://kafka.apache.org/quickstart
- https://www.baeldung.com/ops/kafka-list-topics


