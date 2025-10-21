---
title: mongodb api
date: 2025-07-01 +0800
categories: [工作日志, 备忘]
tags: [mongodb]
author: <yurongku>  
mermaid: true
---

```sh
show dbs; // 显示 database
show databases;

use hello-mongo; // 切换或者创建 database

db; // 查看当前 database

db.dropDatabase(); // 删除 database

db.createCollection('aaa'); // 创建 collection

db.xxx.drop() //删除 collection

db.xxx.insertOne({ name: 'guang', age: 20, phone: '13222222222'});// 插入 document

db.xxx.insertMany([{ name: 'dong', age: 21}, {name: 'xxx', hobbies: ['writing']}]);

db.xxx.find({age: 20}); // 查询 collection

db.xxx.findOne({ age: 20});

db.xxx.findOne(); // 查询所有 collection

db.xxx.find({ age: { $in: [20, 21]}}) // 使用 in、nin 查询多个值

db.xxx.find({ $and: [{age: { $gte: 20 }}, { name: /dong\d/}]}) // 使用 and 指定多个条件同时成立

db.xxx.find({ $or: [{age: { $gt: 20 }}, { name: /dong*/}]}) // 使用 or 表示或者

db.xxx.find().skip(1).limit(2) // 使用 skip + limit 实现分页

db.xxx.updateOne({ name: 'guang'}, { $set: {age: 30}, $currentDate: { aaa: true } }) // updateOne、updateMany 更新

db.xxx.replaceOne({name: 'guang'}, { age: 30}) // 整体替换

db.xxx.deleteMany({ age: { $gt: 20 }}); // deleteOne、deleteMany 删除

db.xxx.count({ name: /guang/})

db.xxx.find().sort({ age: -1, name: 1})

```