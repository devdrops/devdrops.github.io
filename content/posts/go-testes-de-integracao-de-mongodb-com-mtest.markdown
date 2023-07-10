---
title: "[Go]: testes de integração de MongoDB com mtest"
date: 2022-01-05
draft: true
---

## A Situação

Certa vez, estava investigando algumas opções para testes de integração de uma camada de
[Repository](https://martinfowler.com/eaaCatalog/repository.html) de uma apĺicação que se comunicava com um banco de
dados MongoDB.

Minha intenção então foi de simular o comportamento do próprio SDK de MongoDB nessa aplicação, usando a estratégia de
[mocks](https://en.wikipedia.org/wiki/Mock_object). Dessa forma, poderia atingir meu objetivo.

Há algumas opções que normalmente a gente pensa quando falamos de mocks, como por exemplo:

- [gomock](https://github.com/golang/mock);
- Escrever o mock a partir de interfaces.

No entanto, eu particularmente não apliquei essas opções. Eu gosto do _gomock_, mas acho seu código final complexo para
uma pessoa entender "de cara". E embora eu goste de escrever mocks a partir de interfaces, a quantidade de métodos a
criar tornou o trabalho muito problemático.

Fuçando um pouco mais, acabei encontrando uma solução feita pelos mantenedores do próprio
[SDK](https://github.com/mongodb/mongo-go-driver): o
[mtest](https://pkg.go.dev/go.mongodb.org/mongo-driver/mongo/integration/mtest).

## A Solução

O pacote `mtest`, segundo sua própria documentação:

```
Package mtest is unstable and there is no backward compatibility guarantee. It is experimental and subject to change.
```

Por isso, reforço: essa é apenas **uma opção**; sabendo disso, use-a com cautela :wink:


```go
import (
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/mongo/integration/mtest"
)

func TestWhathever (t *testing.T) {
opt := mtest.NewOptions().ClientType(mtest.Mock).CreateClient(true).ShareClient(true)
       mt := mtest.New(t, opt)
       defer mt.Close()

       mc := &Whathever{
               client: mt.Client,
               db: "database",
               coll: "collection",
       }

       v := bson.A{"a", "b", "c", "d", "e", "f", "g", "h"}
       mt.AddMockResponses(bson.D{{"ok", 1}, {"values", v}})
       mt.Run("Should return success", func (mt *mtest.T) {
               res, err := mc.Whathever(context.TODO(), something)
               assert.NoErrorf(t, err, "Test failed: an error was returned (%v)", err)
               e := len(v)
               assert.Equalf(t, res, e, "Test failed: length should be %d but got %d", e, res)
       })

       mt.AddMockResponses(bson.D{{"ok", 0}})
       mt.Run("Should return an error", func (mt *mtest.T) {
               res, err := mc.Whathever(context.TODO(), something)
               assert.Error(t, err, "Test failed: an error was expected")
               assert.Equalf(t, res, 0, "Test failed: length should be 0 but got %d", res)
       })
}
```
