```javascript
var http = require('http')
var fs = require('fs')
var server = http.createServer()
// fs.readdir 读取目录
server.on('request',(req,res)=>{
    var url = req.url
    var path = 'E:/exam01'
    var filePath = '/object.html'
    if (url != '/'){
        filePath = url
    }
    fs.readFile(path + filePath,(err,data)=>{
        if (err) {
            // return 有两个作用
            // 1.方法返回值
            // 2.阻止代码继续往后执行
            return res.end('404 not found')
         }
        res.end(data)
    })
})

server.listen(3000,()=>{
    console.log('服务启动成功');
})
```

