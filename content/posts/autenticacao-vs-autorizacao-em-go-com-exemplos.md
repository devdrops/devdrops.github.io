---
title: "Autenticação vc Autorização em Go (com exemplos)"
date: 2024-03-11
tags: ['Go']
draft: true
---


<!--more-->

- Autenticação: quem você é
- Autorização: o que você pode

```go
package main

import (
        "net/http"

        "github.com/gin-gonic/gin"
)

func main() {
        r := gin.Default()
        r.GET("/resource", gin.BasicAuth(gin.Accounts{
                "admin": "secret",
        }), func(c *gin.Context) {
                c.JSON(http.StatusOK, gin.H{
                        "data": "resource data",
                })
        })
        r.Run()
}
```

Referências:
- https://www.jetbrains.com/guide/go/tutorials/authentication-for-go-apps/auth/

