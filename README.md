# node.js培训教程
---

##搭建 Node.js 开发环境
###NVM安装
[安装nvm-windows](https://github.com/coreybutler/nvm-windows/wiki#manual-installation)

1. [download nvm-noinstall.zip](https://github.com/coreybutler/nvm/releases)
2. Update the system environment variables:
*NVM_HOME*, *NVM_SYMLINK* (C:\Users\Program Files\nodejs **This directory should not exist**)
3. Create *settings.txt* file
````
root: C:\Users\qinayan\bin\nvm
path: C:\Program Files\nodejs
arch: 64
proxy: none
```
**tips**
Please do not forget to run `install` in *NVM_HOME*
###node.js安装
```
安装特定版本的nodejs
> nvm install 0.12.10
安装最新版本的nodejs
> nvm install latest
查看本地安装了哪些nodejs
> nvm ls
 * 5.7.1 (Currently using 64-bit executable)
   0.12.10
验证安装完成   
> node -v
  v5.7.1
> node
...> console.log("hello world");
  hello world
```
##模块
每个文件就是一个模块，文件的路径名就是模块的名字
### require
类似于Java中的`import`关键字，导入不同的包。
```
var express = require('express');
```
### exports
导出模块的公有方法和属性。可以理解为Java中的`public`方法和属性。
```
// util.js
exports.greeting = function(name) {
   return "hello, " + name;   
}
// index.js
var greeting = require('./util').greeting;
console.log(greeting("lambeta"))
>  hello, lambeta
```
### module
包含当前模块的一些信息，常用的做法是替换当前模块的导出对象。
```
// util.js
console.log(module)
>>
Module {
  id: '.',
  exports: { greeting: [Function] },
  parent: null,
  filename: 'C:\\Users\\qianyan\\Projects\\lesson2\\util.js',
  loaded: false,
  children: [],
  paths:
   [ 'C:\\Users\\qianyan\\Projects\\lesson2\\node_modules',
     'C:\\Users\\qianyan\\Projects\\node_modules',
     'C:\\Users\\qianyan\\node_modules',
     'C:\\Users\\node_modules',
     'C:\\node_modules' ] }

// replace with obj
module.exports = {greeting: {}};
>>
Module {
  id: '.',
  exports: { greeting: {} },
  parent: null,
  filename: 'C:\\Users\\qianyan\\Projects\\lesson2\\util.js',
  loaded: false,
  children: [],
  paths:
   [ 'C:\\Users\\qianyan\\Projects\\lesson2\\node_modules',
     'C:\\Users\\qianyan\\Projects\\node_modules',
     'C:\\Users\\qianyan\\node_modules',
     'C:\\Users\\node_modules',
     'C:\\node_modules' ] }
```
###module initialize发生的时机
模块中的代码只会在首次被使用的时候才会执行一次，同时初始化该模块的导出对象，之后导出对象会被缓存到内存当中，供任意使用。

##小结
* NVM是Node Version Manager，管理node的版本的工具。使用NVM，可以保证同一个操作系统下，多个不同版本的node得以共存。
* node作为javascript的解析器，可以在终端下进入交互式模式(repl read-eval-print-loop)，很方便快速地反馈我们程序的结果。
* nodeJS的模块系统实现了CMD标准，即[CommonJS Module Definition](http://wiki.commonjs.org/)标准；而对于运行在浏览器上的javascript的模块化，因为需要异步加载js文件，所以由require.js实现了AMD (Asynchronous Module Definition)标准

```
CMD
require('express')

AMD
define("alpha", ["require", "exports", "beta"],
function (require, exports, beta) {
       exports.verb = function() {
           return beta.verb();
           //Or:
           return require("beta").verb();
       }
});
```

##问题
* 是否可以使用`require('./data.json'`)将json文件引入到我们的程序当中呢？
* 有两个js文件同时引入了`data.json`，先执行a.js，后执行b.js。下面的程序会输出什么？
```
//data.json
{"hello": "world"}

//a.js
var data = require('./data.json');
data = {};

//b.js
var data = require('./data.json');
console.log(data)
```
---
##代码组织
###模块解析路径
####node_modules
不想直接require文件路径名，因为这样一旦所依赖的文件路径发生变化，牵扯的文件会很多。所以我们需要一个约定的根目录。这个根目录就是`node_modules`
以这个文件的路径为例：`C:\\Users\\qianyan\\Projects\\lesson2\\util.js`，node搜索的路径如下。
```
 paths:
   [ 'C:\\Users\\qianyan\\Projects\\lesson2\\node_modules',
     'C:\\Users\\qianyan\\Projects\\node_modules',
     'C:\\Users\\qianyan\\node_modules',
     'C:\\Users\\node_modules',
     'C:\\node_modules' ]
```
####NODE_PATH
我们知道java中依赖包的搜索路径是通过classpath这个JVM的参数控制的。其实node也有这样的变量提供支持。这个变量就是`NODE_PATH`
windows下
```
cmd
set NODE_PATH="your_path"
powershell
env:NODE_PAHT="your_path"
```
NODE_PATH中的路径被遍历是发生在从项目的根位置递归搜寻 node_modules 目录，直到文件系统根目录的node_modules，如果还没有查找到指定模块的话，就会去 NODE_PATH中注册的路径中查找。
####内置模块
如`fs`, `http`等，不做路径解析就直接使用其导出对象`require('fs')`, `require('http')`

###Package（包）
包就是封装多个子模块，同时提供入口的大模块。这个大模块的功能是内聚的。举个例子：
```
C:\USERS\QIANYAN\PROJECTS\LESSON2
└───plane
        body.js
        engine.js
        main.js
        wing.js
```
其中plane目录定义了一个包，其中包含了4个子模块。main.js作为入口模块，如下：
```
var engine = require('./engine');
var wing = require('./wing');
var body = require('./body');

exports.plane = {
    type: "747",
    engine: engine,
    wing: wing,
    body: body
}
```
这里有两种方法可以省去写文件名`main`。
####index.js
其他模块需要使用plane这个包时，可以使用`require('./plane/main')`才行。这里有个约定，如果main.js->index.js，那么就不需要写出文件名字，直接`require('./plane')`就可以了。

这样模块显得更内聚，和Clojure中的`(use namespace)`的用法类似。以下两条语句等价。
```
require('./plane/main')
require('./plane')
```
####package.json

如果想自定义入口模块的文件名和存放位置，就需要在包目录下包含一个package.json文件，并在其中指定入口模块的路径。上例中的模块可以重构如下。
```
C:\USERS\QIANYAN\PROJECTS\LESSON2\PLANE
│   package.json //包描述文件
│
├───doc //文档
├───lib //API文件
│       body.js
│       engine.js
│       main.js
│       wing.js
│
└───tests //测试文件
```
其中package.json内容如下。
```
{
    "name": "plane",
    "main": "./lib/main.js" //这里入口文件的名字可以按自己的喜好更改
}
```
如此一来，就同样可以使用require('./lib/plane')的方式加载模块。NodeJS会根据包目录下的package.json找到入口模块所在位置。

###命令行程序
node.js的程序是跑在命令行之中的，命令行程序长得类似`cmd --name=value`这样的形式。
####创建目录
在windows下创建一个`greeting`程序的目录，如下：
```
C:\USERS\QIANYAN\PROJECTS\LESSON2\GREETING
│   package.json
│   README.md
│
├───bin
│       greeting.cmd
│
├───doc
├───lib
│       index.js
│
├───node_modules
└───tests
```
package.json的内容如下：
```
{
  "name": "greeting",
  "version": "1.0.0",
  "description": "say hi to everyone",
  "main": "lib/index.js",
  "directories": {
    "doc": "doc",
    "test": "tests"
  },
  "bin": {
    "greeting": "bin/greeting.cmd"  
  },
  "scripts": {
    "test": "node test"
  },
  "author": "lambeta",
  "license": "MIT"
}
```
在windows下，如果我们想要实现`cmd --name=value`的效果，就必须使用cmd后缀的文件，如下：
```
//greeting.cmd
@node "lib/index.js" %*
```
我们实现一个接受人的名字作为参数的命令行程序`lib/index.js`，如下：
```
console.log('hello,', process.argv[2]);
```
到这里，可以直接这样运行`./bin/greeting.cmd lambeta`，输出`hello, lambeta`。不过，还是没有预期的样子。
####npm link
我们再运行一条npm的命令
```
> npm link
C:\Program Files\nodejs\greeting -> C:\Program Files\nodejs\node_modules\greeting\bin\greeting.cmd
C:\Program Files\nodejs\node_modules\greeting -> C:\Users\qianyan\Projects\lesson2\greeting
```
这条命令帮助我们设置两个软链接。第一个链接使得我们可以直接运行`greeting lambeta`；第二个则在全局范围内，其他的模块得以引入greeting这个包。
此时，我们可以直接使用`greeting lambeta`来运行程序了
####依赖第三方库
为了实现真正的`cmd --name=value`，我们使用一个第三方库`yargs`。
1. 安装yargs: `npm install yargs --save`
2. 修改index.js文件如下
```
var argv = require('yargs').argv;
console.log('hello,', argv.name);
```
3. 运行
```
> greeting --name=lambeta
  hello, lambeta
```

最后再来看看一个完整的node.js的整体结构
```
C:\USERS\QIANYAN\PROJECTS\LESSON2\GREETING
│   package.json //包描述文件
│   README.md    //说明文件
│
├───bin
│       greeting.cmd //命令行相关代码
│
├───doc   //文档
├───lib   //API代码
│       index.js
│
├───node_modules //第三方依赖
└───tests //测试
```
##小结
* 按照标准目录结构
* 分模块管理项目
* 使用NPM管理第三方模块和命令行程序
* 使用package.json描述项目信息和依赖

##问题
* 下载一个第三方命令行程序到本地`npm install es-checker`，不要使用`-g`参数，如何运行起来这个程序？
* 了解一下[npm scripts](https://docs.npmjs.com/misc/scripts)，在上题的基础上，添加包含下面的内容的package.json，运行`npm test`。思考这样做是否可行？
```
//package.json
{
    "scripts": {
        "test": "es-checker"
    }
}
```

##文件操作
前置条件：[安装离线文档工具](https://zealdocs.org/)
###文件操作相关的API
####buffer对象（数据块）
Javascript语言本身只支持字符串操作，没有提供针对二进制数据流的操作。NodeJS提供了一个与`String`对等的全局对象`Buffer`. Buffer和整数的数组很类似，但是它是固定长度，一旦创建就不能修改。

```
var bin = new Buffer('hello', 'utf8');// <Buffer 68 65 6c 6c 6f>
bin.toString(); //'hello'
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
bin.toString(); //'hello'

var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var dump = new Buffer(bin.length);
bin.copy(dump);
```
####stream模块（数据流）
Stream是一个抽象的接口，所有的stream都是EventEmitter的实例。
当内存中无法一次装下需要处理的数据时，或者一边读取一边处理更加高效时，我们就需要用到数据流。NodeJS中通过各种Stream来提供对数据流的操作。
```
var fs = require('fs');
var readStream = fs.createReadStream('README.md'); //readStream是EventEmitter的实例。

readStream.on('data', function(chunk) {
   console.log(chunk.toString()); 
});

readStream.on('end', function() {
   console.log('end.');
});
```
####fs模块
NodeJS通过`fs`内置模块提供对文件的操作。`fs`模块提供的API基本可以分为以下三类：

* 文件属性读写。
其中常用的有fs.stat、fs.chmod、fs.chown等等。

* 文件内容读写。
其中常用的有fs.readFile、fs.readdir、fs.writeFile、fs.mkdir等等。

* 底层文件操作。
其中常用的有fs.open、fs.read、fs.write、fs.close等等。

我们可以通过`require('fs')`来引用这个模块，而且该模块下的每个方法都有同步和异步的形式。
```
// read sync
var file = fs.readFileSync(process.cwd() + '/README.md', 'utf8');
console.log(file);

// read async
fs.readFile(process.cwd() + '/README.md', 'utf8', function (err, data) {
    if(err) {
        console.log(err);
        return;
    }
    console.log(data)
});
```
一段遍历当前目录的程序
```
var fs = require('fs');
var p = require('path');

function recursiveRead(path) {
    if(fs.statSync(path).isFile()) {
        console.log("File:", path);
    } else {
        fs.readdir(path, function(err, files) {
            if(err) {
                console.log(err);
                return;
            }
            
            files.forEach(function(file) {
                var absPath = p.join(path, file);
                if(fs.statSync(absPath).isFile()) {
                    console.log("File:", absPath);
                } else {
                    console.log("Directory:", absPath);
                    recursiveRead(absPath);
                }
            });
        });  
    }
}
recursiveRead(__dirname); //全局的对象__dirname，当前脚本执行的目录。
```

####path模块
和`java`类似，NodeJS提供path来简化对文件路径的操作。

* path.normalize
```
var path = require('path');
path.normalize('foo/bar/..'); // 'foo'
```
* path.join & path.sep
```
path.join('foo', '/bar/', '/baz', 'par/') // 'foo\\bar\\baz\\par\\'
path.sep // '\\'
```
* path.extname
```
path.extname('node.js') //'.js'
```
##小结
* `Buffer`提供了NodeJS操作二进制的机制；
* `Stream`是一个抽象的接口，每种stream都是EventEmitter的实例。当我们在读取大文件时，可以使用数据流一边读取，一边处理；
* `fs`提供了文件属性读写，内容读写以及底层文件操作。
* 不要使用字符串拼接，使用`path`简化操作

##问题
* 使用`fs`的API创建一个copy的函数；
* NodeJS对文本编码的处理；
* 使用第三方包`findit`重写遍历当前目录的程序。

##网络操作
###简单的HTTP服务器
使用`http`实现一个简单的HTTP服务器。
```
var http = require('http');
http.createServer(function(req, res) {
    res.writeHead(200, {'Content-type': 'text/plain'});
    res.end('Hello Node.js');
}).listen(12306);
```
###http模块
http模块提供两种使用方式：

* 作为服务端使用时，创建一个HTTP服务器，监听HTTP客户端请求并返回响应；
* 作为客户端使用时，发起一个HTTP客户端请求，获取服务端响应。

先创建一个HTTP服务器
```
//http-server.js
var http = require('http');

http.createServer(function(req, res) {
   var body = [];
   res.writeHead(200, {'Content-type': 'text/plain'});
   
   req.on('data', function(chunk) {
      res.write(chunk);
      body.push(chunk);
   });
   
   req.on('end', function() {
      body = Buffer.concat(body);
      console.log(body.toString());
      res.end();
   })
}).listen(12306);
```
再创建一个HTTP客户端
```
//http-client.js
var http = require('http');
var options = {hostname: 'localhost',
    port: 12306,
    method: 'POST',
    headers: {
        'Content-type': 'text/plain'
    }};
var req = http.request(options, function(res) {
    res.on('data', function(chunk) {
       console.log('res:', chunk.toString());
    });
});

req.write('hello world');
req.end();
```
###url模块
* `parse`
使用url解析成URL对象
```
> url.parse('http://user:pass@host.com:8080/p/a/t/h?query=string#hash
Url {
  protocol: 'http:',
  slashes: true,
  auth: 'user:pass',
  host: 'host.com:8080',
  port: '8080',
  hostname: 'host.com',
  hash: '#hash',
  search: '?query=string',
  query: 'query=string',
  pathname: '/p/a/t/h',
  path: '/p/a/t/h?query=string',
  href: 'http://user:pass@host.com:8080/p/a/t/h?query=string#hash' }
```
* `format`
format方法允许将一个URL对象转换为URL字符串
```
> url.format({
...     protocol: 'http:',
...     host: 'www.example.com',
...     pathname: '/p/a/t/h',
...     search: 'query=string'
... });
//'http://www.example.com/p/a/t/h?query=string'
```
* `resolve`
resolve方法拼接两个URL
```
> url.resolve('http://www.baid.com/path', '../www.google.com')
'http://www.baid.com/www.google.com'
> url.resolve('http://example.com/one', '/two')
'http://example.com/two'
```
### querystring
```
> querystring.parse('foo=bar&baz=qux&baz=quux&corge');
{ foo: 'bar', baz: [ 'qux', 'quux' ], corge: '' }
```
##小结
* http模块支持服务端模式和客户端模式两种使用方式；
* request和response对象除了用于读写头数据外，可以当作数据流来操作；
* url.parse方法加上request.url属性是处理HTTP请求时的固定搭配。

##问题
* http模块和https模块的区别？
* 如何创建一个https服务器？

##进程操作
###API一览
####Process
`process`是一个全局的对象，可以在node环境中随处访问。并且它是EventEmitter的实例。
一个进程对象里头到底包含些什么属性？
```
pid
stdio
argv
env
```
只在POSIX平台支持的函数
```
getuid
getgid
geteuid 
getegid
```
进程ID、标准输入输出以及错误流、启动进程的参数、运行环境、运行时权限。
#####应用场景
* 获取命令行参数
```
// index.js
console.log(process.argv);

> node index.js helo
[ 'C:\\Program Files\\nodejs\\node.exe', //node的执行路径
  'C:\\Users\\qianyan\\Projects\\lesson5\\index.js', //文件路径
  'hello' ] //参数

```
一般获取参数的写法
```
process.argv.splice(2)
```
* 退出程序
类似Java中的`System.exit(1)`，当我们捕获一个异常，同时觉得程序需要立即停止时，就执行`process.exit(1)`来表示非正常退出。

* 控制输入和输出
`stdin`是只读流，而`stdout`和`stderr`都是只写流。`console.log`等价于
```
console.log = (msg) => {
  process.stdout.write(`${msg}\n`);
};
```
####Child Process
`child_process`是一个内置模块，可以创建和控制子进程。该模块的主要功能都是`child_process.spawn()`函数提供的。其余诸如`exec`, `fork`, `execFile`等都是对`spawn()`进行的封装。

#####应用场景
* 创建子进程

```javascript
//(command[, args][, options])
const spawn = require('child_process').spawn;
const echo = spawn('cmd', ['/c', 'env'], {env: process.env});//尝试设置{env: {}}，观察结果。 

echo.stdout.on('data', (data) => {
    console.log(`stdout: ${data}`);
})
```
第一参数是可执行文件的路径，第二参数是数组对应可执行文件接收的参数，第三参数用于配置子进程运行的环境和行为。
* 进程间通信
如果父子进程都是Node.js的进程，那么就可以通过IPC通道通信。

```javascript
//parent.js
const spawn = require('child_process').spawn;
const child = spawn('node', ['child.js'], {
    stdio: [process.stdin, process.stdout, process.stderr, 'ipc']
});

child.on('message', (msg) => {
    console.log('parent:', msg);
});

child.send({hello: 'world'});

//child.js
process.on('message', (msg) => {
    console.log('child:', msg);
    msg.hello = msg.hello.toUpperCase();
    process.send(msg);
})
=>
child: { hello: 'world' }
parent: { hello: 'WORLD' }
```
父进程在创建子进程的时候，使用了`options.stdio`的`ipc`额外开辟了一条通道，之后开始监听子进程的`message`事件来接收子进程的消息，同时通过`send`方法给子进程发送消息。子进程则通过`process`对象监听来自父进程的消息，并通过`process.send`方法向父进程发送消息。
####Cluster
单个实例的Node.js运行在单独的进程当中。但是我们有时候可能需要利用多核处理器的优势，在每个单独的核上跑一个Node.js的进程。
`Cluster`就是创造出来简化多进程服务程序开发的，让每一个核上面运行一个工作进程，并统一通过主进程监听端口和分发请求。
#####应用场景
```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);
}
=>
NODE_DEBUG=cluster node server.js
isMaster
worker
worker
worker
worker
worker
worker
worker
worker
```
该模块很简单地创建多个共享一个服务端口的子进程，而这些子进程是通过IPC和Master，也即父进程进行信息交互的，可应用于负载均衡。 

##小结
* 使用`process`对象管理进程
* 使用`child_process`对象管理子进程，其最主要的方法就是`spawn`
##问题

##异步编程
NodeJS最大的卖点——事件机制和异步IO，开发者需要按照异步的方式去组织代码。
###回调
异步编程的直接体现就是回调函数，但是不是有回调函数，就是异步编程呢？
```javascript
function sum(arr, callback) {
    var sum = 0;
    for(var i=0; i<arr.length; i++) {
        sum = sum + arr[i];
    }
    
    callback(sum);
}

sum([1,2,3,4,5], console.log);
=>
15
```
显然，这个`callback`还是顺序（同步）执行的。我们知道，JS本身是单线程的，所以不具备多线程并发执行的特点，那么异步从何体现呢？
我们再看一段程序：
```javascript
setTimeout(function() {
    console.log("world")
}, 1000);

console.log("hello");
=>
hello
world
```
上面的例子先打印出“hello”，然后打印出“world”。看上去好像是`setTimeout()`另外启动了一个“平行线程”，等待了1秒钟之后，调用回调函数打印“world”。
JS中提供了两大类异步函数，一种是计时函数，如：`setTimeout`和`setInterval`。另外一类是I/O异步函数，如：`fs.readFile`。

但是JS是单线程的。也就是说如果“主”线程一直处于忙碌状态，即使“平行”线程完成工作，通知“主”线程调用它的回调函数，也会等到“主”线程空闲了才能真正去调用。
```javascript
var t = new Date();
setTimeout(function () {
    console.log("waiting time: ", new Date() - t);
}, 1000);

while(new Date() - start < 1000);
=>
waiting time: 1094 //大于我们设置的1000毫秒
```

###返回值
我们分别使用同步和异步实现一个函数，判断当前目录下的文件是否都是File，最终程序返回一个布尔值的数组，如：`[true, false]`
当前目录文件结构如下：
```
|_async.js
|_sync.js
|_ dir/
```
####比较中学习
* 同步方式下

```javascript
const fs = require('fs');
const path = require('path');

const dirs = fs.readdirSync('.');
const areFiles = dirs.map((dirName) => {    
    return fs.statSync(path.join('.', dirName)).isFile();
});

console.log(areFiles);
=> [true, false, true]
```
* 异步方式下
*失败的尝试*

```javascript
const fs = require('fs');
const path = require('path');

fs.readdir('.', (err, dirs) => {
    const areFiles = [];
    
    dirs.forEach((dirName) => {  
        fs.stat(path.join('.', dirName), (err, stat) => {
            areFiles.push(stat.isFile());
        })
    });
    
    console.log(areFiles);
});
=> [] //思考为何是空数组？
```
*成功的尝试*

```javascript
const fs = require('fs');
const path = require('path');

fs.readdir('.', (err, dirs) => {
    const areFiles = [];
    
    dirs.forEach((dirName) => {  
        fs.stat(path.join('.', dirName), (err, stat) => {
            areFiles.push(stat.isFile());
            
            if(areFiles.length == dirs.length) { //使用标志来位判断所有的回调都已经调用完毕
                console.log(areFiles);
            }
        })
    });
});
=>[ true, true, false ] or [true, false, true]
```
####总结
1. 同步方法顺序取返回值，而异步方法总是在回调函数的取返回值
2. 循环遍历中调用同步方法很容易，但是同样地在异步方法中，需要使用**标志位**来判断是否所有回调函数都已经调用完毕
3. 异步函数的执行回调是无序的

###数组的串行处理
我们看到上个例子里的异步的写法，最后的返回结果其实是无序的。使用标志位只能保证数组中的所有数据对应的回调函数都得以执行，但不能保证哪个回调函数先返回。要想顺序执行，那么必须是一个回调函数中包含另一个回调函数。拿上面的例子尝试：

```javascript
const fs = require('fs');
const path = require('path');

fs.readdir('.', (err, dirs) => {
    (function iterate(index, areFiles, callback) {
        if(index < dirs.length) {
            fs.stat(path.join('.', dirs[index]), (err, stat) => {
                areFiles.push(stat.isFile());
                iterate(index + 1, areFiles, callback);
            });  
        } else {
           callback(areFiles);
        }
    }(0, [], (result) => {
        console.log(result);
    }));
});
```
####在场景中学习
假如我们有这样一个场景：有一系列的HTTP请求的URL构成的数组和一个初始值。这些HTTP请求是有依赖的，后一个的执行必须依赖前一个HTTP请求的响应。如果只是两个请求，我们可以很轻松地写出这样的代码：

```javascript
const http = require('http');
const urls = ['localhost', 'www.baidu.com']; //多个urls的数组

const req = http.request(optionsWithHostname(urls[0]), (res) => { //第一次请求
    res.on('data', (chunk) => {
        const req2 = http.request(optionsWithHostname(urls[1]), (res2) => {  //第二次请求
            res2.on('data', (chunk2) => {
                //dosometing here...
            });
        });
        req2.write(chunk.toString() + 'agian');
        req2.end();
    });
});

req.write('hello world');
req.end();

function optionsWithHostname(hostname) {
    return {
    hostname: hostname,
    port: 12306,
    method: 'POST',
    headers: {
        'Content-type': 'text/plain'
    }};
}
```
但如果是十个或者更多，这样的写法就不好使了。

我们知道异步函数必须在回调中才能使用其返回值，这样会很容易写出类似于`>`形状的回调套回调的写法。而递归的写法也正好符合这样的形状，所以尝试一下：

```javascript
const urls = ['localhost', 'www.baidu.com']; //多个urls的数组

(function next(i, len, initValue, callback) {
    if (i < len) {
    // 将http请求过程简化成了async
        async(urls[i], (value) => {
            console.log(value);
            next(i + 1, len, value, callback);
        });
    } else {
        callback();
    }
}(0, urls.length, 'hello world', () => {
   //dosomething here...
}));
```
####总结
* 在异步函数想要保证执行的顺序，就必须一个回调套一个回调
* 可以利用递归的写法，在保证执行顺序的同时，处理系列或者不定长度的数据

###异常处理
####在比较中学习
* 同步方式下

```javascript
//try ... catch ...

try {
    x.func();
} catch (err) {
   console.log("I catch you ", err);
}
=> I catch you  [ReferenceError: x is not defined]
```

* 异步方式下

```javascript
try {
    setTimeout(() => {
        x.func();
    }, 0);
} catch (err) {
   console.log("I catch you ", err);
}
=> C:\Users\qianyan\Projects\lesson6\exception\async.js:5
        x.func();
        ^

ReferenceError: x is not defined
    at null._onTimeout (C:\Users\qianyan\Projects\lesson6\exception\async.js:5:9)
```
可以看到，同步方式下异常会沿着代码执行路径一直冒泡，直到遇到第一个try语句时被捕获住。但由于异步函数会打断代码执行路径，异步函数执行过程中以及执行之后产生的异常冒泡到执行路径被打断的位置时，如果一直没有遇到try语句，就作为一个全局异常抛出。

解决方式就是在异常被作为全局异常抛出之前，try-catch住，如下：

```javascript
setTimeout(() => {
    try {
        x.func();
    } catch (err) {
        console.log("I catch you ", err);
    }
}, 0);
=> I catch you  [ReferenceError: x is not defined]
```
这样异常又被捕获了。不妨，对`setTimetout`做一次封装

```javascript
function wrapSetTimeout(fn, callback) {
    setTimeout(() => {
        try {
            callback(null, fn());
        } catch (err) {
            callback(err);
        }
    }, 0);
}

wrapSetTimeout(() => {x.func()}, (err, data) => {
    if(err) console.log("I catch you again", err);
})
=>I catch you again [ReferenceError: x is not defined
```
Node.js的整个异步函数的异常设计都是如此，callback的首个参数都是err。

####总结
* try-catch在同步方式下很有效，但在异步方式下做不到
* callback首个参数是err，是因为大多数API都遵循了一致的风格

##小结
* 不掌握异步编程就不算学会NodeJS
* 异步编程依托于回调来实现，而使用回调不一定就是异步编程
* 异步编程下的函数间数据传递、数组遍历和异常处理与同步编程有很大差别

