# node.js培训教程
---

##搭建 Node.js 开发环境
###NVM安装
[安装nvm-windows](https://github.com/coreybutler/nvm-windows/wiki#manual-installation)
1. [download nvm-noinstall.zip](https://github.com/coreybutler/nvm/releases)
2. Update the system environment variables:
*NVM_HOME*, *NVM_SYMLINK* (C:\Users\Program Files\nodejs **This directory should not exist**)
3. Create *settings.txt* file
> root: C:\Users\qinayan\bin\nvm
path: C:\Program Files\nodejs
arch: 64
proxy: none

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
##进程操作
##网络操作
##异步编程
##es6和es5
