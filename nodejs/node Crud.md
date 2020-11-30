### 从文件中读取数据 并渲染到页面

```javascript
var express = require('express')
var fs = require('fs')
var app = express()



app.use('/node_modules',express.static('./node_modules/'))
app.use('/public',express.static('./public/'))

app.engine('html', require('express-art-template'));

app.get('/',(req,res)=>{
    // 设置 utf8 和data.toString()方式相同
    fs.readFile('./views/db.json','utf8',(err,data)=>{
        if(err){
            return res.status(500).send('Server error.')
        }
        res.render('index.html',{
            fruits:[
                '苹果',
                '梨',
                '橙子'
            ],
            // 文件中读取的文件都是字符串,需要转换为对象
            students:JSON.parse(data).student
        })
    })
})

app.listen(3000,()=>{
    console.log('running...');
})
```





#### 起步

- 初始化
- 模板处理

#### 路由设计

| 请求方法 | 请求路径         | get参数 | post参数                   | 备注             |
| -------- | ---------------- | ------- | -------------------------- | ---------------- |
| GET      | /students        |         |                            | 渲染首页         |
| GET      | /students/new    |         |                            | 渲染添加学生页面 |
| POST     | /students/new    |         | name,age,gender,hobbies    | 处理添加学生请求 |
| GET      | /students/edit   | id      |                            | 渲染编辑页面     |
| POST     | /students/edit   |         | id,name,age,gender,hobbies | 处理编辑请求     |
| GET      | /students/delete | id      |                            | 处理删除请求     |

#### 提取路由模块

router.js:

```javascript
/**
 * router.js路由模块
 * 职责：
 *      处理路由
 *      根据不同的请求方法+请求路径设置具体的请求函数
 * 模块职责要单一，我们划分模块的目的就是增强代码的可维护性，提升开发效率
 */
/** 
方法一：
route.js

module.exports = function(app){
  app.get('/',(req,res)=>{
    // 设置 utf8 和data.toString()方式相同
    fs.readFile('./views/db.json','utf8',(err,data)=>{
        if(err){
            return res.status(500).send('Server error.')
        }
        res.render('index.html',{
            fruits:[
                '苹果',
                '梨',
                '橙子'
            ],
            // 文件中读取的文件都是字符串,需要转换为对象
            students:JSON.parse(data).student
        })
    })
})
}


app.js

var router = require('./router')
router(app)
**/
// 方法二
var fs = require('fs');

// Express专门提供了一种更好的方式
// 专门用来提供路由的
var express = require('express');
// 1 创建一个路由容器
var router = express.Router();
// 2 把路由都挂载到路由容器中

var Student = require('./student')

router.get('/', function(req, res) {
    // res.send('hello world');
    // readFile的第二个参数是可选的，传入utf8就是告诉他把读取到的文件直接按照utf8编码，直接转成我们认识的字符
    // 除了这样来转换，也可以通过data.toString（）来转换
    // fs.readFile('./db.json', 'utf8', function(err, data) {
    //     if (err) {
    //         return res.status(500).send('Server error.')
    //     }
    //     // 读取到的文件数据是string类型的数据
    //     // console.log(data);
    //     // 从文件中读取到的数据一定是字符串，所以一定要手动转换成对象
    //     var students = JSON.parse(data).student;
    //     res.render('index.html', {
    //         // 读取文件数据
    //         students:students
    //     })
    // })
    Student.find((err,students)=>{
        if (err) {
            return res.status(500).send('Server error.')
        }
        res.render('index.html', {
            // 读取文件数据
            fruits:[
                '苹果',
                '梨子'
            ],
            students:students
        })
    })
});

router.get('/students/new',function(req,res){
    res.render('new.html')
});
router.post('/students/new',function(req,res){
    Student.save(req.body,(err)=>{
        if (err) {
            return res.status(500).send('Server error.')
        }
        else{
            res.redirect('/')
        }
    })
});

router.get('/students/edit',function(req,res){
    Student.findById(parseInt(req.query.id),(err,student)=>{
        if (err) {
            return res.status(500).send('Server error.')
        }
        else{
            res.render('edit.html',{
                student:student
            })
        }
    })
});

router.post('/students/edit',function(req,res){
    Student.update(req.body,(err)=>{
        if (err) {
            return res.status(500).send('Server error.')
        }
        else{
            res.redirect('/')
        }
    })
});

router.get('/students/delete',function(req,res){
    Student.delete(parseInt(req.query.id),(err)=>{
        if (err) {
            return res.status(500).send('Server error.')
        }
        else{
            res.redirect('/')
        }
    })
});

// 3 把router导出
module.exports = router;
```

app.js:

```javascript
var router = require('./router');

// router(app);
// 把路由容器挂载到app服务中
// 挂载路由
app.use(router);
```

#### 设计操作数据的API文件模块

es6中的find和findIndex：

find接受一个方法作为参数，方法内部返回一个条件

find会便利所有的元素，执行你给定的带有条件返回值的函数

符合该条件的元素会作为find方法的返回值

如果遍历结束还没有符合该条件的元素，则返回undefined[![image-20200313103810731]()](https://github.com/smallC-L-Y/Demo/blob/notes/nodeJS学习笔记.md)

```javascript
/**
 * student.js
 * 数据操作文件模块
 * 职责：操作文件中的数据，只处理数据，不关心业务
 */
var fs = require('fs');
var dbPath = './db.json'
 /**
  * 获取所有学生列表
  * return []
  */

  // 回调函数：获取异步操作的结果
exports.find = function(callback){
    fs.readFile(dbPath,'utf8',(err,data)=>{
        if(err){
            return callback(err)
        }
        callback(null,JSON.parse(data).student)
    })
}

exports.findById = function(id,callback){
    fs.readFile(dbPath,'utf8',(err,data)=>{
        if(err){
            return callback(err)
        }
        var students = JSON.parse(data).student
        var stu = students.find(item =>{
            return item.id === parseInt(id)
        })
        callback(null,stu)
    })
}


 /**
  * 获取添加保存学生
  */
exports.save = function(student,callback){
    fs.readFile(dbPath,'utf8',(err,data)=>{
        if(err){
            return callback(err)
        }
        var students = JSON.parse(data).student
        student.id = students[students.length-1].id+1
        students.push(student)
        var ret = JSON.stringify({
            student:students
        })
        fs.writeFile(dbPath,ret,(err)=>{
            if(err){
                return callback(err)
            }
            // 成功则没错误，没错误传null
            callback(null)
        })
    })
}

/**
 * 更新学生
 */
exports.update = function(student,callback){
    fs.readFile(dbPath,'utf8',(err,data)=>{
        if(err){
            return callback(err)
        }
        student.id= parseInt(student.id)
        var students = JSON.parse(data).student
        var stu = students.find(item =>{
            return item.id === student.id
        })
        // 循环遍历对象
        for (var key in student){
            stu[key] = student[key]
        }
        var ret = JSON.stringify({
            student:students
        })
        fs.writeFile(dbPath,ret,(err)=>{
            if(err){
                return callback(err)
            }
            // 成功则没错误，没错误传null
            callback(null)
        })
    }) 
}

 /**
 * 删除学生
 */
exports.delete = function(id,callback){
    fs.readFile(dbPath,'utf8',(err,data)=>{
        if(err){
            return callback(err)
        }
        var students = JSON.parse(data).student
        var deleteId = students.findIndex(item=>{
            return item.id === parseInt(id)
        })
        students.splice(deleteId,1)
        var ret = JSON.stringify({
            student:students
        })
        fs.writeFile(dbPath,ret,(err)=>{
            if(err){
                return callback(err)
            }
            // 成功则没错误，没错误传null
            callback(null)
        })
    })
}
```

#### 步骤

- 处理模板
- 配置静态开放资源
- 配置模板引擎
- 简单的路由，/studens渲染静态页出来
- 路由设计
- 提取路由模块
- 由于接下来的一系列业务操作都需要处理文件数据，所以我们需要封装Student.js'
- 先写好student.js文件结构
  - 查询所有学生列别哦的API
  - findById
  - save
  - updateById
  - deleteById
- 实现具体功能
  - 通过路由收到请求
  - 接受请求中的参数（get，post）
    - req.query
    - req.body
  - 调用数据操作API处理数据
  - 根据操作结果给客户端发送请求
- 业务功能顺序
  - 列表
  - 添加
  - 编辑
  - 删除