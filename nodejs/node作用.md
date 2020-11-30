
# node 服务器开发 javascript 运行时环境 既不是语言也不是框架只是个平台
1. ## node中的javascript时EcmaScript基本的JavaScript语言部分 

2. ## 没有BOM 和 DOM 

3. ## 提供了一些服务器级别的API 核心模块： 文件操作（fs) http服务构建 (http) path路径操作模块 os操作系统信息模块


作用：1.解析.js文件
     2.读取文件
     3.http  构建一个web服务器模块  服务器的作用：提供服务：对数据的服务 发请求，接收请求，处理请求，发送响应
    

```javascript

    var fs = require('fs')
    <!-- fs 是 file-system的简写，文件系统提供了所有文件操作的相关API 例如  fs.readFile读取文件 -->
    <!-- 成功
      data为数据，error为null
    失败
      data为null,error为错误对象 -->
    <code>
    fs.readFile('路径',funciton(error,data){
    });

    fs.wirteFile('路径','内容',(error)=>{    
    })
```



####     require 是一个方法 作用为加载模块  拿到被加载文件模块导出的接口对象 exports 默认是个空对象

####  在node中模块有三种  

1. 具名的核心模块 fs. http
2. 用户自己编写的文件模块  ./  文件名 可以省略后缀名
3. node中，没有全局作用域，只有模块作用域 外部访问不到内部 ，内部也访问不到外部



```javascript
var a = require('./b.js')

require('./b.js')
console.log(a.foo)    // hello


// b.js
var foo = 'bbn'
console.log('ssss')
exports.foo = 'hello'
```

​    

##### ip地址用来定位计算机 端口号用来定位具体的应用程序  所有需要联网通信的应用程序都会占用一个端口号



##### 中文乱码问题: 服务器端默认发送的数据是utf-8编码的内容，但浏览器在不知道服务器响应的情况下默认会按照默认编码去解析 中文为gbk，解决方法：

```javascript
res.setHeader('Content-Type','text/plain;charset=utf-8')
```



默认情况下浏览器能解析html的字符串 但加入下图后，无法解析，需要更改为text/html

```javascript
res.setHeader('Content-Type','text/plain;charset=utf-8')   // 普通文本

res.setHeader('Content-Type','text/html;charset=utf-8')    // html 渲染
res.end('<p>你好世界<a href=''>点我</a></p>')
```

将资源文件通过服务器返回到浏览器的方式

Content-type 

​     http://tool.oschina.net/commons

​     不同的资源对应的Content-Type 是不一样的

​     图片不需要指定编码，一般只有字符数据才指定

```javascript
var fs = require ('fs')
var http = require ('http')

var server = http.createServer()
server.on('request',(req,res)=>{
    if (req.url === '/'){
        fs.readFile('文件路径',(err,data)=>{
            if(err){
                res.setHeader('Content-Type','text/plain;charset=utf-8') 
                console.log('文件读取失败')
            }
            else{
                res.setHeader('Content-Type','text/html;charset=utf-8') 
                res.end(data)
            }
        })
    }
})

server.listen(3000,()=>{
    consoe.log('服务器已经启动了')
})
```

