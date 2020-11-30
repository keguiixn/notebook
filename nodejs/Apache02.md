```javascript
var http = require('http')
var fs = require('fs')
var server = http.createServer()

server.on('request',(req,res)=>{
    var url = req.url
     var path = 'E:/exam01'
    fs.readFile('./template.html',(err,data)=>{
        if (err) {
            // return 有两个作用
            // 1.方法返回值
            // 2.阻止代码继续往后执行
            return res.end('404 not found')
         }
        fs.readdir(path,(err,files){     // files :['apple', 'orange']
          if (err) {
             return res.end('404 not found')
          }
        var content = ''
        files.forEach((item) =>{
            content += `${item}`
        })
        data = data.toString()
        data = data.replace('abc',content)
        })
        res.end(data)
    })
})

server.listen(3000,()=>{
    console.log('服务启动成功');
})
```

