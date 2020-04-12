---
title: nodejs 操作 mysql
date: 2020-01-31 19:10:49
tags: nodejs
---

> nodejs 操作 mysql

<!-- more -->


### nodejs 操作 mysql 可利用 mysqljs ([https://github.com/mysqljs/mysql](https://github.com/mysqljs/mysql))
```js
npm install mysql
```


### 考虑到稳定性，建议使用 mysql 线程池的方式操作，当成一个 lib 提供给外部使用，如下

- mysql.js 文件

```js
// mysql.js
const mysql = require('mysql');

const pool  = mysql.createPool({
  connectionLimit : 10,
  host            : 'localhost',
  user            : 'root',
  password        : '',
  database        : 'stock'
});

const query=function(sql,callback){
  // 从线程池获取连接
  pool.getConnection(function(err, connection) {
    if (err) throw err; // not connected!

    // Use the connection
    connection.query(sql, function (error, results, fields) {
      // When done with the connection, release it.
      // 释放线程
      connection.release();

      // Handle error after the release.
      if (error) throw error;

      // Don't use the connection here, it has been returned to the pool.
      // 处理数据
      callback(results,fields);
    });
  });

};
module.exports=query

```

- 测试使用

```js
const query = require('./mysql.js');

query('select * from stock limit 2;',function(results,fields) {
  // body...
  console.log(results);
  console.log(fields)
});

```