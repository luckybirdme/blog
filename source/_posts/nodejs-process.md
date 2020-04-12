---
title: nodejs 多进程操作
date: 2020-01-28 21:39:24
tags: nodejs
---

> nodejs 多进程的操作

<!-- more -->


### 1. process 是 node 的全局对象，可以直接使用，它包括主进程的运行信息

- 获取进程执行的输入参数

```js
// process.argv 第一个参数是 node 程序的目录，第二是执行文件的目录，后面的才是输入参数
// 比如 ./node index.js one two
let argv = process.argv.slice(2);

```

- 在进程退出时可指定返回码

```js
process.exit(1);
process.exit(0);

```

- process 包括进程的标准输入，输出和错误信息，即 process.stdin, process.stdout, process.stderr

```js
process.stdout.write('hello');

```

### 2. child_process 用于创建和管理子进程，并设置进程间的通信方式

- 利用 exec 直接执行 shell 命令，回调函数直接返回结果，最大 200kb，所以合适返回数据量较少的操作

```js
let cp=require('child_process');
cp.exec('echo hello',function(err,stdout){
  console.log(stdout);
});

```

- 利用 execFile 执行 shell 会检测传入实参的安全性，如果存在安全性问题，会抛出异常

```js
// 存在安全隐患
cp.exec('echo hello;rm -rf /',function(err,stdout){
  console.log(stdout);
});

// 检测参数，会抛出异常
cp.execFile('echo',['hello',';rm -rf /'],function(err,stdout){
   console.log(stdout);
});
```



- spawn 用于创建和管理子进程，并通过 Stream 返回，所以能返回大量数据。默认情况下，子进程的 stdin、 stdout 和 stderr 会被重定向到 ChildProcess 对象上相应的 stdin、 stdout 和 stderr 

```js

const { spawn } = require('child_process');
// 相当于通过子进程执行 ./node event.js
const cp = spawn('node', ['event.js']);

cp.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

cp.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

cp.on('close', (code) => {
  console.log(`子进程退出，退出码 ${code}`);
});
```

- 如果操作文件较大，以流的形式输入输出可以明显减小内存的占用，同时也可以提高输入输出的效率，比如以下例子，多个子进程，多层链式流

```js
let cp=require('child_process');
let cat=cp.spawn('cat',['input.txt']);
let sort=cp.spawn('sort');
let uniq=cp.spawn('uniq');

cat.stdout.pipe(sort.stdin);
sort.stdout.pipe(uniq.stdin);
uniq.stdout.pipe(process.stdout);
console.log(process.stdout);

```


- fork 可创建独立于主进程子进程，它基于 spawn, 但是一个特例，用于衍生新的 nodejs 进程，拥有自己的内存和 V8 实例，主进程和新的 nodejs 进程通过 IPC 通道进行通信。

```js
// parent.js
console.log('I am parent process');
let cp=require('child_process');
// 相当于 cp.spawn('node',['child.js']);
let child=cp.fork('./child.js');
child.on('message',function(msg){
  console.log('parent got a message is',msg);
});
child.send('hello world');

// child.js
console.log('I am child process')
process.on('message',function(msg){
   console.log('child got a message is',msg);
   process.send('Hi')
   process.exit(0);
})

```


### 3. cluster 集成了 child_process.fork 的特性，同时可将进程分发到多个 CPU 执行，最大限度利用机器性能。

- 官方的例子

```js

const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是一个 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    console.log('pid',process.pid)
    res.write(process.pid.toString());
    res.end('hello\n');
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

- 上述例子需要人工维护多进程，为了方便管理多进程，也为了实现进程平滑重启，可利用 nodejs 进程管理组件 [pm2](https://github.com/Unitech/pm2)

```js
// i参数告诉PM2，这段代码应该在cluster_mode启动，且新建 worker 进程的数量是4个
pm2 start app.js -i 4
```