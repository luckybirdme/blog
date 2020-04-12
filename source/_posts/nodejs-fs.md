---
title: nodejs 文件操作
date: 2020-01-26 13:52:00
tags: nodejs
---

> nodejs 文件操作的常用方法

<!-- more -->

### 1. 大部分文件操作方法都包括异步和同步的函数，同步函数多了 Sync，而且可以立马 catch 错误
- 异步读取

```js
fs.readFile('hello.txt','utf-8', (err, data) => {
  if (err) throw err;
  console.log(data);
});


```

- 同步读取

```js

try {
  // 指定编码读取
  const data = fs.readFileSync('hello.txt','utf-8');
  console.log(data);
} catch (err) {
  // 处理错误
  console.log(err);
}

```


### 2. nodejs 文件读取时支持设置编码， 比如 utf-8, 但是不支持 gbk, 需要第三方库进行转码

```js
const iconv = require('iconv-lite');
let bin = fs.readFileSync('hello.txt');
let str = iconv.decode(bin, 'gbk');
```


### 3. 数据处理除了字符串 String 外，还支持 Buffer (数据块，二进制) 和 Stream (数据流，管道流) 
- Buffer

```js

try {
  // 默认返回是 Buffer
  const buf = fs.readFileSync('hello.txt');
  console.log(buf);
  const bts = buf.toString('utf-8');
  console.log(bts);
} catch (err) {
  // 处理错误
  console.log(err);
}

```

- Stream

```js
let data = '';
let readerStream = fs.createReadStream('hello.txt',{ encoding : 'utf-8', highWaterMark : 1});
let writerStream = fs.createWriteStream('output.txt');

readerStream.on('data', function(chunk) {
  console.log(chunk);
  data += chunk;
});
readerStream.on('end',function(){
  console.log("read end");
  console.log(data);
  writerStream.write(data,'utf-8');
  writerStream.end();
});

writerStream.on('finish', function() {
  console.log("write finish");
});

```

- 链式 Stream

```js
const zlib = require('zlib');
fs.createReadStream('hello.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('hello.txt.gz'));

```


### 4. 为了防止大文件操作时占满内存，通过 Stream 方式进行数据流转，比如复制大文件
- 小文件

```js
const src = '/tmp/little';
const dst = '/tmp/little_copy';
fs.writeFileSync(dst, fs.readFileSync(src));

```

- 大文件

```js
const src='/tmp/large';
const dst='/tmp/large_copy';
fs.createReadStream(src).pipe(fs.createWriteStream(dst));
```


### 5. 利用递归遍历多层目录，包括同步和异步的方式
- 同步遍历

```js
function traverSync(src, cb){
  let pathname = path.join(src);
  let files = fs.readdirSync(pathname);
  for(let i in files) {
    let newsrc = path.join(src,files[i]);
    if(fs.statSync(newsrc).isDirectory()){
      traverSync(newsrc, cb);
    }else{
      cb(newsrc)
    }
  }
}

```

- 异步遍历

```js
function traver(src, cb) {
  fs.readdir(src, (err, files) => {
    if(err) throw err;
    // 必须使用 let 块变量，不能使用 var 
    // 因为是异步执行，var 变量只会保存最后的取值
    // let 块变量，只在当前区域有效，不受外界影响，所以异步执行依然是当时的变量
    for(let i in files) {
      let newsrc = path.join(src, files[i]);
      fs.stat(newsrc, (err, file) => {
        if(err) throw err;

        if(file.isDirectory()){
          traver(newsrc, cb)
        }else{
          cb(newsrc)
        }
        console.log(i);
      })
    }
  });
}
```