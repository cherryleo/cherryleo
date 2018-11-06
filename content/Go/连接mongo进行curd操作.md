---
weight: 1000
title: "连接mongo进行curd操作"
date: "2018-11-05"
lastmod: "2018-11-05"
description:  "go语言如何连接mongodb，如何对mongodb进行curd操作"
categories:  ["go"]
tags: ["go"]
---

Go语言连接Mongodb，并进行简单的增删查改操作。基于官方包封装了一些简单的方法，方便数据库连接，查询等操作。
查询单个文档，所有文档，返回子文档，返回JSON格式数据等，https://github.com/cherryleo/gomongo

## 1. mongodb官方包
```go
import "github.com/mongodb/mongo-go-driver/mongo"
```

## 2. 连接mongodb
设置mongodb连接信息，返回mongodb连接客户端
```go
var CS = connstring.ConnString{
	// Original: "mongodb://127.0.0.1:27017",
	// SSL:                         true,
	// SSLInsecure:                 true,
	// SSLCaFile:                   "",
	// SSLClientCertificateKeyFile: "",
	Hosts:    []string{"127.0.0.1:27017"},
	Username: "",
	Password: "",
}

func connectMongodb() (*mongo.Client, error) {
	client, err := mongo.NewClientFromConnString(CS)
	if err != nil {
		return nil, err
	}
	err = client.Connect(context.Background())
	if err != nil {
		return nil, err
	}
	return client, nil
}
```

## 3. 查询文档
提供数据库名，集合名，查询条件，返回数据库中所有匹配的文档
```go
func GetDocuments(db, collection string, filter *bson.Document) ([]*bson.Document, error) {
	// mongodb client
	client, err := connectMongodb()
	defer client.Disconnect(context.Background())
	if err != nil {
		return nil, err
	}

	// mongodb cursor
	c := client.Database(db).Collection(collection)
	cur, err := c.Find(context.Background(), filter, nil)
	if err != nil {
		return nil, err
	}
	defer cur.Close(context.Background())

	// get documents
	documentList := []*bson.Document{}
	for cur.Next(context.Background()) {
		elem := bson.NewDocument()
		err := cur.Decode(elem)
		if err != nil {
			return nil, err
		}
		documentList = append(documentList, elem)
	}
	if err := cur.Err(); err != nil {
		return nil, err
	}
	return documentList, nil
}
```

测试获取文档，需要提供过滤条件。当前过滤条件制定一对key/value，查询结果返回集合中所有匹配此key/value的文档
```go
func TestGetDocument(t *testing.T) {
	filter := bson.NewDocument(bson.EC.String("mongo-key", "mongo-value"))
	result, err := GetDocument(
		"db-name",
		"collection-name",
		filter,
	)
	if err != nil {
		t.Error(err)
	}
	t.Log(result)
}
```

## 4. 创建文档
提供数据库名，集合名，向指定数据库集合中插入文档
```go
func InsertDocument(db, collection string, document interface{}) error {
	// mongodb client
	client, err := connectMongodb()
	defer client.Disconnect(context.Background())
	if err != nil {
		return err
	}

	c := client.Database(db).Collection(collection)
	_, err = c.InsertOne(context.Background(), document)
	if err != nil {
		return err
	}
	return nil
}
```

测试插入文档代码
```go
func TestInsertOneDoucment(t *testing.T) {
	document := map[string]string{"foo": "bar"}
	err := InsertDocument("db-name", "collection-name", document)
	if err != nil {
		t.Error(err)
	}
}
```

## 5. 删除文档
提供数据库名，集合名，删除集合中所有匹配过滤条件的文档
```go
func DeleteDocument(db, collection string, filter *bson.Document) error {
	// mongodb client
	client, err := connectMongodb()
	defer client.Disconnect(context.Background())
	if err != nil {
		return err
	}

	c := client.Database(db).Collection(collection)
	_, err = c.DeleteOne(context.Background(), filter)
	if err != nil {
		return err
	}
	return nil
}
```

测试删除文档代码
```go
func TestDeleteDoucment(t *testing.T) {
	filter := bson.NewDocument(bson.EC.String("mongo-key", "mongo-value"))
	err := DeleteDocument("db-name", "collection-name", filter)
	if err != nil {
		t.Error(err)
	}
}
```