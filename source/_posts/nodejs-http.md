---
title: nodejs 网络交互
date: 2020-01-28 13:44:58
tags: nodejs
---

> nodejs 网络交互的常用方法

<!-- more -->

### 1. 创建 http server
```js
const http = require('http');
let data = '';
http.createServer(function (request, response) {
  response.writeHead(200, { 'Content-Type': 'text/plain' });
  request.on('data', function (chunk) {
    console.log('data',chunk)
    data += chunk;
  });
  request.on('end', function () {
    console.log('request end',data);
    response.write('hello');
    response.end();
  });
}).listen(8080);
```

### 2. 模拟 http client

```js
const getData = 'hi';
let data = '';
const options = {
  hostname: 'localhost',
  port: 8080,
  path: '/',
  method: 'GET',
  headers: {
    'Content-Type': 'text/plain',
    'Content-Length': Buffer.byteLength(getData)
  }
};

const req = http.request(options, (res) => {
  res.setEncoding('utf8');
  res.on('data', (chunk) => {
    data += chunk;
  });
  res.on('end', () => {
    console.log('res end',data);
  });
});

req.write(getData);
req.end();
```

### 3. 更便捷的 get 方法

```js
let data = '';
http.get('http://localhost:8080/', function (res) {
  res.on('data', (chunk) => {
    data += chunk;
  });
  res.on('end', () => {
    console.log('res end',data);
  });   
});
```


### 4. 支持 https 协议
```js

const options = {
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('hello world\n');
}).listen(8000);


```

### 5. 支持 gzip 压缩和解压
```js
// http server 根据浏览器标识，是否返回压缩形式的数据
http.createServer(function (request, response) {
    let data = 'hello';
    if ((request.headers['accept-encoding'] || '').indexOf('gzip') !== -1) {
        zlib.gzip(data, function (err, data) {
            response.writeHead(200, {
                'Content-Type': 'text/plain',
                'Content-Encoding': 'gzip'
            });
            response.end(data);
        });
    } else {
        response.writeHead(200, {
            'Content-Type': 'text/plain'
        });
        response.end(data);
    }
}).listen(80);


// http client 根据服务器返回数据标识，是否解压数据
http.request(options, function (response) {
    let body = [];

    response.on('data', function (chunk) {
        body.push(chunk);
    });

    response.on('end', function () {
        body = Buffer.concat(body);

        if (response.headers['content-encoding'] === 'gzip') {
            zlib.gunzip(body, function (err, data) {
                console.log(data.toString());
            });
        } else {
            console.log(data.toString());
        }
    });
}).end();
```


### 6. 支持分片报文传输 Transfer-Encoding: chunked
```js
const getData = 'hi';
let data = '';
const options = {
  hostname: 'localhost',
  port: 8080,
  path: '/',
  method: 'GET',
  headers: {
    'Content-Type': 'text/plain',
    // 非持续链接，需要计算报文长度，以便知道是否传输完毕
    //'Content-Length': Buffer.byteLength(getData)
    // 长链接支持分片报文传输，共用 TCP 链接进行多次传输
    'Connection': 'keep-alive',
    'Transfer-Encoding': 'chunked'
  }
};

const req = http.request(options, (res) => {
  res.on('data', (chunk) => {
    data += chunk;
  });
  res.on('end', () => {
    console.log('res end',data);
  });
});

req.write(getData);
req.end();

```

### 7. 借助 URL 类解析请求路径
``` js
const url = require('url');
const myURL = url.parse('https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash');
console.log(myURL);
{
  protocol: 'https:',
  slashes: true,
  auth: 'user:pass',
  host: 'sub.host.com:8080',
  port: '8080',
  hostname: 'sub.host.com',
  hash: '#hash',
  search: '?query=string',
  query: 'query=string',
  pathname: '/p/a/t/h',
  path: '/p/a/t/h?query=string',
  href: 'https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash'
}

```

### 8. 借助 querystring 类转换请求参数
```js
const querystring = require('querystring');
var obj = querystring.parse('foo=bar&baz=qux&baz=quux&corge');
console.log(obj);

let str = querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' });
console.log(str);
```