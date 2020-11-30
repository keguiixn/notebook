###### 使用Node编写应用程序主要是在使用：

- EcmaScript 语言
  - 和浏览器不一样，在node中没有BOM，DOM
- 核心模块
  - 文件操作的fs
  - http服务的http
  - url路径处理模块
  - os操作系统信息
- 第三方模块
  - art-template
  - 必须通过npm 来下载才可以使用
- 自定义模块 (路径形式模块)
  - 自己创建的文件



###### 什么是模块化

- 文件作用域
- 通信规则
  - 加载 require
  - 导出

###### CommonJS模块规范

在Node中的JavaScript 还有一个很重要的概念: 模块系统

- 模块作用域

- 使用require方法用来加载模块

  语法：

  `var 自定义变量名称 = require('模块')`

  两个作用：

  - 执行被加载模块中的代码
  - 得到被加载模块中的`exports` 导出接口对象

- 使用exports接口对象用来导出模块中的成员

  - Node 中是模块作用域，默认文件中所有的成员只在当前文件模块有效
  - 对于希望可以被其他模块访问的成员，我们就需要把这些公开的成员都挂载到`exports`接口对象中就可以了

导出多个成员（必须在对象中）：

``` javascript
exports.a = 123
exports.b = 'sssad'
exports.c = function(){}
exports.d = {
    foo : 'bar'
}
module.exports={}


var module = {
    exports:{}
}
//等价于
console.log(exports === module.exports)   //true

// 只要 exports更改赋值就不等于module.exports
module.exports.a = 123


// 最后return 的是 module.exports
```



导出单个成员（拿到的就是：函数，字符串）

```javascript
module.exports = function (){}
module.exports = 'hello'
```

