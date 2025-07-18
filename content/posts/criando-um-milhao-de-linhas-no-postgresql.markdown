---
title: "Criando 1 Milhão de Linhas no PostgreSQL"
date: 2025-07-18
tags: ['PostgreSQL', 'pt-BR']
draft: false
---

Aqui eu mostro como criar 1 milhão de linhas em uma tabela do PostgreSQL, usando apenas o próprio PostgreSQL :wink:

<!--more-->

Imagine a seguinte tabela:

```sql
CREATE TABLE users (
        id         SERIAL       PRIMARY KEY,
        uuid       uuid         UNIQUE DEFAULT uuid_generate_v4 (),
        name       VARCHAR(100) NOT NULL,
        document   CHAR(11)     UNIQUE NOT NULL,
        created_on TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

Caso eu quiser popular a tabela para testes, eu posso fazer uso dos comandos e funções do PostgreSQL com a instrução
abaixo:

```sql
INSERT INTO users (
        uuid, name, document
)
SELECT
    GEN_RANDOM_UUID(),
    i::TEXT || '_' || LEFT(MD5(i::TEXT), 10) || ' das Couves',
    LEFT(MD5(i::TEXT), 14)
FROM GENERATE_SERIES(1, 1000000) s(i);
```

Ali temos:
- UUID aleatório;
- Nome (algo parecido :sweat_smile: );
- Documento.

Gerados na sequência de 1 a 1000000.

## Referências

- https://stackoverflow.com/a/24841269

