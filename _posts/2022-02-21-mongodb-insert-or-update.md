---
layout: post
title: Mongodb使用复合唯一索引实现mysql on duplication update  
tags: linux CentOS mongo  
categories: mongo  
---


### 创建复合唯一索引：
```shell
admin > use dbname
dbname> db.test.createIndex({one_id:1,tow_user_name:1}, {unique:true})
one_id_1_tow_user_name_1
dbname> db.test.getIndexes()
[
{ v: 2, key: { _id: 1 }, name: '_id_' },
{
v: 2,
key: { one_id: 1, tow_user_name: 1 },
name: 'one_id_1_tow_user_name_1',
unique: true
}
]
```

### 伪代码：

```go

    import(
        "go.mongodb.org/mongo-driver/bson"
	    "go.mongodb.org/mongo-driver/mongo"
	    "go.mongodb.org/mongo-driver/mongo/options"
    )

    MongoConnect := MongodbConnect()
	var modelsWrite []mongo.WriteModel
	for i := 1; i < 100; i++ {
		insertDataMap := make(map[string]interface{})
		insertDataMap["one_id"] = 1
		insertDataMap["tow_user_name"] = userName
		userName := "hash insertDataMap"
		timeNow := time.Now().Format("2006-01-02 15:04:05")
		filterData := bson.D{
			{"one_id", i},
			{"hash_string", hashStringData},
		}
		user := fmt.Sprintf("Man_%s", timeNow)
		updateData := bson.D{
			{"$set", bson.D{
				{"tow_user_name", userName},
				{"three_user_gender", 0},
				{"regist_time", timeNow},
				{"login_time", timeNow},
			}},
			{"$setOnInsert", bson.D{
				{"regist_time", timeNow},
			}},
		}
		modelsWrite = append(
            modelsWrite,
			mongo.NewUpdateManyModel().SetFilter(filterData).SetUpdate(updateData).SetUpsert(true),
		)
	}
	ops := options.BulkWrite().SetOrdered(false)
	retInsert, err := MongoCon.Collection(table).BulkWrite(ctx, modelsWrite, ops)
	if err != nil {
		log.Debugf("InsertOrUpdateUserData error, err:%+v", err)
	}
	log.Infof("InsertOrUpdateUserData success return %+v", retInsert)

```