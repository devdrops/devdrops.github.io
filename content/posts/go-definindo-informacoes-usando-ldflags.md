---
title: "Go: Definindo informações usando ldflags"
date: 2024-03-15
tags: ['Go', 'pt-BR']
draft: false
---

Como definir informações como versão, datas etc usando `-ldflags`.

<!--more-->

**Obs**: todo o código usado aqui está [neste repositório](https://github.com/devdrops/ldflags).

## A Situação

Estava começando um novo projeto em Go, e decidi ter disponíveis, nos logs, informações a respeito da minha API, como
qual a versão, data de quando foi feito o binário etc. Algo como o exemplo abaixo:

```json
{
    "level": "info",
    "ts": 1710513266.1966991,
    "caller": "code/main.go:12",
    "msg": "application up and running",
    "app_name": "Payments API",
    "app_version": "develop",
    "app_build_at": "Today"
}
```

## A Solução

Uma forma de atingir isso é usando `-ldflags` no momento de criação do binário (a famosa _"build"_). Em Go, podemos usar
esse recurso para definir informações únicas nessa etapa, que ficarão marcadas no binário final.

Para essa tarefa, utilizei [go.uber.org/zap](https://github.com/uber-go/zap) para logs (mas pode ser qualquer outra lib
da sua escolha).

Primeiro, criei o lugar onde as informações estarão:

```go
package appinfo

const (
    AppName string = "Payments API"
)

var (
    AppVersion string = "develop"
    AppBuildAt string = "Today"
)
```

Como não quero mudar o valor de `AppName`, criei-o como uma constante. Mas os valores de `AppVersion` e `AppBuildAt`
serão alterados, no entanto, defini valores apenas para usar como padrão (por exemplo, executando apenas com `go run`).

Os logs, providenciei sem muitos detalhes, porém adicionei os valores de informação como campos padrão nos logs:

```go
package logger

import (
    "github.com/devdrops/ldflags/appinfo"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

func InitGlobalLogger() {
    logger, _ := zap.NewProduction(zap.AddCaller(), zap.AddStacktrace(zapcore.ErrorLevel))
    defer logger.Sync()

    logger = logger.With(
        zap.String("app_name", appinfo.AppName),
        zap.String("app_version", appinfo.AppVersion),
        zap.String("app_build_at", appinfo.AppBuildAt),
    )

    zap.ReplaceGlobals(logger)
}
```

E para finalizar, criei a entrada do projeto:

```go
package main

import (
	  "github.com/devdrops/ldflags/logger"

	  "go.uber.org/zap"
)

func main() {
	  logger.InitGlobalLogger()

	  zap.L().Info("application up and running")
}
```

Agora, usando `-ldflags` no momento de _"build"_, eu posso definir valores diferentes para `AppVersion` e `AppBuildAt`:

```
go build -ldflags "-X github.com/devdrops/ldflags/appinfo.AppBuildAt=$(date +%Y-%m-%dT%H:%M:%S_GMT%Z) -X github.com/devdrops/ldflags/appinfo.AppVersion=$GIT_TAG" main.go
```

**Nota**: `$GIT_TAG` pode ser qualquer variável que tenha a referência que você quer (commit, tag etc).

Com isso, o resultado de `go build` vai exibir o log com as informações passadas, como abaixo:

```json
{
    "level": "info",
    "ts": 1710513834.6334388,
    "caller": "code/main.go:12",
    "msg": "application up and running",
    "app_name": "Payments API",
    "app_version": "v1.2.3",
    "app_build_at": "2024-03-15T14:43:41_GMTUTC"
}
```

