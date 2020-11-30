

```html
<form action = '/pinglun' method = 'get'>
    
</form>
```





```javascript
var url = require ('url')

// 使用 url.parse 方法将路径解析为一个方便操作的对象，第二个参数为true 表示直接将查询字符串转换为对象 query属性
var parseObj = url.parse(req.url,true)

// 单独获取不包含查询字符串的路径部分
var pathname=parseObj.pathname

if (pathname === '/pinglun'){
// res.end(JSON.stringfy(parseObj.query))
    var comment = parseObj.query
    comments.push (comment)
    
    // 如何通过服务器让客户端重定向
       // 1. 将状态码设置为 302 临时重定向 statusCode
       // 2. 在响应头中通过 Location 告诉客户端往哪重定向 setHeader
    // 如果客户端发现收到服务器的响应状态码是302 就会自动去响应头中找Location 然后对该地址发起新的请求 
    res.statusCode = 302
    res.setHeader('Location' , '/')
    res.end()
}
```

