---
title: "Autenticação http-basic no Composer"
date: 2023-07-18
tags: ['PHP', 'Composer', 'Docker', 'Docker-Compose']
draft: false
---

Segue uma dica rápida, com uso tanto para ambiente com ou sem Docker :wink:

## Antes de Seguir...

As instruções aqui informam dicas de como usar autenticação para download de dependências com Composer. Meu foco maior é
usando Docker e Docker Compose, mas os comandos em essência podem ser usados em qualquer ambiente.

## A Situação

Uma vez, trabalhando com um projeto em PHP, observei que uma das dependências a instalar com
[Composer](https://getcomposer.org/) exigia autenticação para o download. O processo era simples: autenticação básica,
com usuário e senha. Mas como eu queria algo automático, precisava de uma forma mais prática do que informar dados
sensíveis "na unha". Dado que era uma dependência privada, e eu estava usando Docker, pensei em como resolver isso de
forma simples e segura.

## A Solução

Eu precisava resolver duas coisas:

1. Como passar as informações de senha e usuário para o Composer;
2. Como fazer isso de forma segura, sem versionar dados sensíveis.

### Com Docker Compose

#### Arquivo `.gitignore`:

```
.env
auth.json
```

O arquivo `.env` é onde vamos colocar os dados de usuário e senha. Dessa forma, o arquivo será ignorado pelo controle de
versão. Já o arquivo `auth.json` será criado automaticamente pelo Composer, e como também pode conter informações
sensíveis, recomendo que seja também ignorado da mesma forma.

#### Arquivo `.env`:

```bash
USUARIO=SECRETUSER
SENHA=SECRETPASSWORD
```

No arquivo `.env`, criei duas variáveis, uma para o usuário e outra para a senha, ambos no formato `NOME=VALOR`.

#### Arquivo `docker-compose.yml`:

```yaml
composer:
    image: devdrops/php-toolbox:8.2
    env_file: .env
    command: >
        sh -c "composer config http-basic.site.com ${USUARIO} ${SENHA} && composer install"
    volumes:
        - .:/app
    working_dir: /app
```

E no arquivo `docker-compose.yml`, temos algumas instruções importantes:

- O parâmetro `image` aponta para a imagem Docker a ser usada. No caso, a
[devdrops/php-toolbox](https://github.com/devdrops/php-toolbox), que possui Composer e outras ferramentas, usando PHP
8.2.
- O parâmetro `env_file` aponta o arquivo `.env` como fonte de variáveis de ambiente.
- O parâmetro `command` tem 2 comandos:
  1. `composer config http-basic.<ENDEREÇO> <USUARIO> <SENHA>` instrui ao Composer que para o endereço específico, é
     preciso usar o usuário e senha informados.
  2. `composer install`, agora com as instrução de autenticação, pode instalar as dependências com sucesso.
- Enquanto isso, os demais parâmetros (`volumes` e `working_dir`) servem para uso correto de volumes, de forma que as
alterações feitas no container sejam refletidas no mesmo diretório, na sua máquina.

No final, ao executar `docker-compose up`, o Composer será capaz de fazer o download corretamente das dependências,
privadas ou públicas.

## Referências

- https://getcomposer.org/doc/articles/authentication-for-private-packages.md#http-basic

