参考资料：

https://www.bilibili.com/video/BV1a34y167AZ/?share_source=copy_web&vd_source=eb6ade7f410b79ba275b327b6011d56e

# 初识 Node.js

https://nodejs.cn/en/learn

## 是什么？

> Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine.

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境,

## 在 Node.js 环境中执行 JavaScript 代码

1 打开终端
2 输入 node 要执行的 js 文件的路径

```bash
node xxx.js
```

# 操作文件

## fs 文件系统模块

### 路径动态拼接的问题

在使用 fs 模块操作文件时，如果提供的操作路径是以 `.` 或 `../` 开头的相对路径时，很容易出现路径动态拼接错误的问题

原因：代码在运行的时候，会==以执行 node 命令时所处的目录==，动态拼接出被操作文件的完整路径

解决方案：在使用 fs 模块操作文件时，直接提供完整的路径

```js
// __dirname表示当前文件所在目录
console.log(__dirname)

const fs = require('fs')

fs.readFile(__dirname + '/1.txt', 'utf8', function (err, data) {
    if (err) {
        return console.log('读取文件失败！' + err.message)
    }
    console.log('读取文件成功！' + data)
})
```

> 这里个人目前觉得“相对路径”没毛病

## Path 模块

https://nodejs.cn/en/learn/manipulating-files/nodejs-file-paths

# 模块化

模块化是指解决一个复杂问题时，自顶向下逐层把系统划分成若干模块的过程。对于整个系统来说，模块是==可组合、分解和更换==的单元。

编程领域中的模块化，就是遵守固定的规则，把一个大文件拆成独立并互相依赖的多个小模块。

把代码进行模块化拆分的好处：
- 提高了代码的复用性
- 提高了代码的可维护性
- 可以实现==按需加载==

## 模块的分类

Node.js 中根据模块来源的不同，将模块分为了 3 大类，分别是:

- **内置模块**：由 Node.js 官方提供的，例如 fs、path、http 等
- **自定义模块**：用户创建的每个 js 文件，都是自定义模块
- **第三方模块**：由第三方开发出来的模块，并非官方提供的内置模块，也不是用户创建的自定义模块，使用前需要先下载

## CommonJS 模块化规范

Node.js 遵循了 CommonJS 模块化规范，CommonJS 规定了模块的特性和各模块之间如何相互依赖。

CommonJS 规定：
1. 每个模块内部，module 变量代表当前模块
2. module 变量是一个对象，它的 exports 属性，即 module.exports 是对外的接口。
3. 加载某个模块，其实是加载该模块的 module.exports 属性。`require()` 方法用于加载模块

## 加载模块

 ```js
// 1. 加载内置模块  
const fs = require('fs')  
  
// 2.加载用户的自定义模块，可以省略.js后缀  
// const custom = require('./custom.js')  
const custom = require('./custom.js')  
  
// 3.加载第三方模块  
const moment = require('moment')
```

使用 `require()` 方法加载其它模块时，会==执行被加载模块中的代码==

## 模块作用域

在自定义模块中定义的变量、方法等成员，==只能在当前模块内被访问==

好处：防止了全局变量污染的问题

## 向外共享模块作用域中的成员

在每个 js 自定义模块中都有一个 module 对象，它里面存储了和当前模块有关的信息

```js
console.log(module)

{
  id: '.',
  path: 'C:\\Users\\Boer\\Desktop\\nodejs-demo',
  exports: {},
  filename: 'C:\\Users\\Boer\\Desktop\\nodejs-demo\\hello.js',
  loaded: false,
  children: [
  ],
  paths: [
    'C:\\Users\\Boer\\Desktop\\nodejs-demo\\node_modules',
    'C:\\Users\\Boer\\Desktop\\node_modules',
    'C:\\Users\\Boer\\node_modules',
    'C:\\Users\\node_modules',
    'C:\\node_modules'
  ]
}
```

在自定义模块中，可以使用 module.exports 对象，将模块内的成员共享出去，供外界使用
- 默认 `module.exports={}` （空对象）

外界用 `require()` 方法导入自定义模块时，得到的就是 module.exports 所指向的对象。

```js
module.exports.username = 'zs'
// 向 module.exports 对象上挂载 sayHello 方法
module.exports.sayHello = function () {
    console.log('Hello!')
}

*********************************

const custom = require('./custom.js')  
console.log(custom)
// { username: 'zs', sayHello: [Function (anonymous)] }
```

由于 module.exports 单词写起来比较复杂，为了简化向外共享成员的代码，Node 提供了 exports 对象。==默认情况下，exports 和 module.exports 指向同一个对象==

最终共享的结果，还是以 module.exports 指向的对象为准。

```js
console.log(module.exports === exports)
```

# npm 与包

Node.js 中的第三方模块又叫做包。

包是基于内置模块封装出来的，提供了更高级、更方便的 API，极大的提高了开发效率

> 国外有一家 IT 公司，叫做 npm, Inc. 这家公司旗下有一个非常著名的网站: https://www.npmjs.com/ ，它是==全球最大的包共享平台==，你可以从这个网站上搜索到任何你需要的包，只要你有足够的耐心!
> 
> npm,lnc. 公司提供了一个地址为 https://registry.npmjs.org/ 的服务器 ，来对外共享所有的包，我们可以从这个服务器上下载自己所需要的包。

npm,lnc. 公司提供了一个包管理工具 **Node Package Manager** (npm)，这个包管理工具随着 Node.js 的安装包一起被安装到了用户的电脑上。

## npm 安装包

在项目中安装指定名称的包

```bash
npm install 完整包名
npm install moment
# 简写
npm i moment
# 默认安装最新版本，指定版本：
npm i moment@2.22.2
# 多个包可以用空格分隔
npm i jquery art-template
```

初次装包完成后，在项目文件夹下多一个叫做 node modules 的文件夹和 package-lock.json 的配置文件。

- node_modules 文件夹用来存放所有已安装到项目中的包。`require()` 导入第三方包时，就是从这个目录中查找并加载包
- package-lock.json 配置文件用来记录 node modules 目录下的每一个包的下载信息，例如包的名字、版本号、下载地址等

## 包管理配置文件

npm 规定，在项目根目录中，必须提供一个叫做 package.json 的包管理配置文件。用来记录与项目有关的一些配置信息。例如:

- 项目的名称、版本号、描述等
- 项目中都用到了哪些包
- 哪些包只在开发期间会用到
- 哪些包在开发和部署时都需要用到

> 可以运行 `npm uninstall` 命令，来卸载指定的包

### 快速创建 package.json

npm 包管理工具提供了一个快捷命令，可以在执行命令时所处的目录中，快速创建 package.json

```bash
npm init -y
```

### dependencies 节点

package.json 中有一个 **dependencies 节点**，专门用来记录您使用 `npm install` 命令安装了哪些包。

```json
{  
  "dependencies": {  
    "art-template": "^4.13.2",  
    "jquery": "^3.7.1",  
    "moment": "^2.30.1"  
  }  
}
```

### 一次性安装所有的包

第三方包的体积过大，不方便团队成员之间共亨项目源代码。==共享时剔除 node_modules==

当我们拿到一个剔除了 node_modules 的项目之后，需要先把所有的包下载到项目中，才能将项目运行起来

```bash
npm intall 
npm i
```

### devDependencies 节点

如果某些包只在项目开发阶段会用到，在项目上线之后不会用到，则建议把这些包记录到 devDependencies 节点中。

```bash
npm i 包名 -D
```

## 包的分类

- 项目包
	- 开发依赖包（被记录到 devDependencies 节点中的包）
	- 核心依赖包（被记录到 dependencies 节点中的包）
- 全局包
	- 全局包会被安装到 `C:\Users\用户目录\AppData\Roaming\npm\node modules` 目录下
	- 只有工具性质的包，才有全局安装的必要性。因为它们提供了好用的终端命令

```bash
# 全局包安装
npm i 包名 -g
```

## 发布包 #todo 

