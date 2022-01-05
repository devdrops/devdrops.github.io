---
title: "Go: mocks de MongoDB com mtest"
date: 2022-01-05
draft: true
---

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


