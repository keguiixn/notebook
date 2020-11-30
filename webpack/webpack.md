npm init -y 生成一个package.json 文件初始化管理配置文件

如何在项目中安装和配置webpack

1.运行npm install webpack webpack-cli -D 命令，安装webpack相关的包
2.在项目根目录中，创建名为webpack.config.js的webpack配置文件
3.在webpack的配置文件中，初始化配置
<code>
module.exports={
    mode:'development'     //mode用来指定构建模式开发者模式
}
</code>
4.在package.json配置文件中的scripts节点下，新增dev脚本如下
<code>
"scripts":{
    "dev":"webpack"  //script节点下的脚本，可以通过npm run执行
}
</code>
5.在终端中运行npm run dev命令，启动webpack进行项目打包
dist 是webpack自动建立的



配置webpack的自动打包功能
1.运行 npm install webpack-dev-server -D 命令，安装支持项目的自动打包的工具
2.修改 package.json ->scripts中的dev命令如下：
   "scirpt"：{
       "dev":"webpack-dev-server" 
   }
3.将src ->  idnex.html中 ，script脚本的引用路径,修改为"/buldle.js"



在实际开发的过程中，webpack默认只能打包处理以.js后缀名结尾的模块，其他非.js后缀名结尾的模块，webpack处理不了，需要
调用loader加载器才可以正常打包,否则会错
loader 加载器可以协助webpack打包处理特定的文件模块,比如：
less-loader可以打包处理.less相关的文件
sass-loader可以打包处理.scss相关的文件
url-loader可以打包处理css中与url路径相关的文件


打包处理js文件中的高级语法
1.安装babel 转换器相关的包 :npm install babel-loader @babel/core @babel/runtime -D
2.安装babel语法插件相关的包：npm install @babel/preset-env @babel/plugin-transform-runtime @babel/plugin-proposal-class-properties -D
3.在项目根目录下，创建babel配置文件babel.config.js并初始化基本配置如下:
module.exports={
    presets:['@babel/preset-env'],
    plugins:['@babel/plugin-transform-runtime','@babel/plugin-proposal-class-properties']
}
4.在webpack.config.js的module->rules数组中，添加loader规则
{test:/\.js$/,use:'babel-loader',exclude:/node_modules/}





webpack配置vue组件的加载器
1.运行 npm install vue-loader vue-template-compiler -D 命令
2.在webpack.config.js配置文件中，添加vue-loader 的配置项
const VueLoaderPlugin = require('vue-loader/lib/plugin')
增加规则 {test: /\.vue$/,loader:'vue-loader'}
增加插件 new VueLoaderPlugin() 

在webpack 项目中使用vue
1.运行 npm install vue -S 安装vue
2.导入vue构造函数   import Vue from 'vue'
3. 创建vue实例对象并指定要控制的el区域，通过render函数渲染App组件


import Vue from 'vue'
import App from './component/App.vue'

const vm = new Vue({
    el:'#app',
    render: h => h(App)   
})




