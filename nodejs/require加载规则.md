## require的加载规则：

- 优先从缓存加载

- 判断模块标识符

  - 核心模块

  - 自己写的模块（路径形式的模块）

  - 第三方模块（node_modules）

    - 第三方模块的标识就是第三方模块的名称（不可能有第三方模块和核心模块的名字一致）

    - npm

      - 开发人员可以把写好的框架库发布到npm上
      - 使用者通过npm命令来下载

    - 使用方式：

      ```
      var 名称 = require('npm install【下载包】 的包名')
      ```

      - node_modules/express/package.json main
      - 如果package.json或者main不成立，则查找被选择项：index.js
      - 如果以上条件都不满足，则继续进入上一级目录中的node_modules按照上面的规则依次查找，直到当前文件所属此盘根目录都找不到最后报错

```javascript
// 如果非路径形式的标识
// 路径形式的标识：
    // ./  当前目录 不可省略
    // ../  上一级目录  不可省略
    //  /xxx也就是D:/xxx
    // 带有绝对路径几乎不用（D:/a/foo.js）
// 首位表示的是当前文件模块所属磁盘根目录
// require('./a'); 


// 核心模块
// 核心模块本质也是文件，核心模块文件已经被编译到了二进制文件中了，我们只需要按照名字来加载就可以了
require('fs'); 

// 第三方模块
// 凡是第三方模块都必须通过npm下载（npm i node_modules），使用的时候就可以通过require('包名')来加载才可以使用
// 第三方包的名字不可能和核心模块的名字是一样的
// 既不是核心模块，也不是路径形式的模块
//      先找到当前文所述目录的node_modules
//      然后找node_modules/art-template目录
//      node_modules/art-template/package.json
//      node_modules/art-template/package.json中的main属性
//      main属性记录了art-template的入口模块
//      然后加载使用这个第三方包
//      实际上最终加载的还是文件

//      如果package.json不存在或者mian指定的入口模块不存在
//      则node会自动找该目录下的index.js
//      也就是说index.js是一个备选项，如果main没有指定，则加载index.js文件
//      
        // 如果条件都不满足则会进入上一级目录进行查找
// 注意：一个项目只有一个node_modules，放在项目根目录中，子目录可以直接调用根目录的文件
var template = require('art-template');
```





## 模块标识符中的`/`和文件操作路径中的`/`

文件操作路径：

```javascript
// 咱们所使用的所有文件操作的API都是异步的
// 就像ajax请求一样
// 读取文件
// 文件操作中 ./ 相当于当前模块所处磁盘根目录
// ./index.txt    相对于当前目录
// index.txt    相对于当前目录
// /index.txt   绝对路径,当前文件模块所处根目录
// d:express/index.txt   绝对路径
fs.readFile('./index.txt',function(err,data){
    if(err){
       return  console.log('读取失败');
    }
    console.log(data.toString());
})
```

模块操作路径：

```javascript
// 在模块加载中，相对路径中的./不能省略
// 这里省略了.也是磁盘根目录
require('./index')('hello')
```

