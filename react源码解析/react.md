#### JSX和虚拟DOM

##### 准备工作

为了集中精力编写逻辑,在代码打包工具上选择了火热的零配置打包工具parcel

##### **全局安装parcel**

```js
npm i parcel-bundler -g
复制代码
```

新建项目react_simple,新建index.html和index.js,将index.js引入到index.html中

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22a5445af754422d9b4df59afebc8bc6~tplv-k3u1fbpfcp-zoom-1.image)

**执行**

```
parcel index.html
复制代码
```

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 项目依赖配置

安装parcel-bundler

```js
npm i parcel-bundler --save-dev 
复制代码
```

配置`package.json`

```json
{
  "name": "react_simple",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "parcel index.html"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.3",
    "babel-plugin-transform-react-jsx": "^6.24.1",
    "babel-preset-env": "^1.7.0",
    "parcel-bundler": "^1.12.3"
  }
}

复制代码
```

安装babel插件,将jsx的语法转化成js对象(这个js对象即所谓的虚拟DOM)

```js
npm i babel-core babel-preset-env babel-plugin-transform-react-jsx --save-dev
复制代码
```

同时`.babelrc`配置如下

```js
{
    "presets": ["env"],
    "plugins": [
        ["transform-react-jsx", {
            "pragma": "React.createElement"
        }]
    ]
}

复制代码
```

##### 先看一个react的最简单,也是最熟悉的例子

```js
import React from 'react';
import ReactDOM from 'react-dom';

// 这并不是合法的js代码,它是一种被称为jsx的语法扩展,通过它我们可以很方便的在js代码中书写html片段
// 本质上,jsx是语法糖
const title = <h2 className = 'title'> <span> hello React </span></h2> ;

// 上面代码会被babel转义成下面对象 而这个对象就是所谓的虚拟DOM
/* 
var title = React.createElement("h2", {
    className: "title"
}, " hello React ");
*/
ReactDOM.render(title,document.querySelector('#root'))

/* 
第一个参数是DOM节点的标签名，它的值可能是div，h1，span等等
第二个参数是一个对象，里面包含了所有的属性，可能包含了className，id等等
从第三个参数开始，就是它的子节点
*/
// 从jsx转译结果来看 createElement(tag,attrs,child1,child2,child3,child4...)
复制代码
```

因此，我们得出结论：JSX 语法糖经过 Babel 编译后转换成一种对象，该对象即所谓的`虚拟 DOM`，使用虚拟 DOM 能让页面进行更为高效的渲染。

函数构造

index.js

```js
const React = {
    createElement
}
function createElement(tag,attrs,...childrens){
    return {
        tag,
        attrs,
        childrens
    }
}
const element = (
    <div className='title'>
        hello <span>react</span>
    </div>
)
console.log(element);
复制代码
```

打印结果如下:

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

```
createElement`方法返回的对象记录了这个DOM节点所有的信息,换句话说,通过它我们可以生成真正的DOM,这个记录信息的对象的我们称之为`虚拟DOM
```

接下来,我们一一来实现想要的功能

##### ReactDOM.render

接下来是ReactDOM.render方法,

```jsx
ReactDOM.render(
    <h1>Hello,react</h1>,
    document.getElementById( 'root' )
);
复制代码
```

经过转换

```jsx
ReactDOM.render(
    React.createElement( 'h1', null, 'Hello, react' ),
    document.getElementById('root')
);
复制代码
```

所以`render`的第一个参数实际上接受的是createElement返回的对象，也就是虚拟DOM .而第二个参数则是挂载的目标DOM

总而言之，render方法的作用就是**将虚拟DOM渲染成真实的DOM**，下面是它的实现：

```jsx
const ReactDOM = {
     render
}
function render(vnode, container) {
    // 如果vnode为定义,则什么都不做
    if (vnode == undefined) return;
    // 如果vnode是字符串是,渲染结果是一段文本
    if (typeof vnode === 'string') {
        const textNode = document.createTextNode(vnode);
        return container.appendChild(textNode);
    }
    // 否则是一个虚拟DOM对象
    const {
        tag
    } = vnode;
    // 创建节点对象
    const dom = document.createElement(tag);
    console.log(dom);

    if (vnode.attrs) {
        // 有属性
        Object.keys(vnode.attrs).forEach(key => {
            // 取值
            const val = vnode.attrs[key];
            // 为上面的节点对象设置属性
            setAttribute(dom, key, val);
        })
    }
    // 递归渲染子节点
    vnode.childrens.forEach(child => render(child, dom));
    // 将渲染结果挂载到真正的DOM上
    return container.appendChild(dom);

}
复制代码
```

设置属性需要考虑一些特殊情况(className,onClick,其它属性等),我们单独将其封装一个方法`setAttribute`

```jsx
function setAttribute(dom, key, value) {
    // 如果属性名是className,则改为class
    if (key === 'className') {
        key = 'class';
        // return key
    }

    // 如果属性名是onClick onBlur onChange.... 则是事件监听的方法
    if (/on\w+/.test(key)) {
        // 转小写
        key = key.toLowerCase();
        dom[key] = value || '';

    } else if (key === 'style') { //如果属性名是style,则更新style对象
        if (!value || typeof value === 'string') {
            dom.style.cssText = value || '';
        } else if (value && typeof value === 'object') {
            // 更新style对象
            for (let k in value) {
                // 可以通过style={width:20}这种形式来设置样式,可以省略掉单位px
                if (typeof value[k] === 'number') {
                    dom.style[k] = value[k] + 'px';
                } else {
                    dom.style[k] = value[k]
                }
                //  dom.style[ k ] = typeof value[ k ] === 'number' ? value[ k ] + 'px' : value[ k ];
            }
        }
    } else { //普通属性则直接更新属性
        if (key in dom) {
            dom[key] = value || '';
        }
        if (value) { //有值则更新值
            dom.setAttribute(key, value);
        } else { //没有值 移除此属性
            dom.removeAttribute(key);
        }
    }
}
复制代码
```

注意:多次地调用render函数时,不会清除原来的内容 先清除一下挂载目标DOM的内容

修改代码如下:

```jsx
const ReactDOM = {
    render:(vnode,container)=>{
        container.innerHTML = '';
        return render(vnode,container);
    }
}
复制代码
```

测试1:

```jsx
const ele = <div className='title' title='123'>
    <h3 className='3'>hello <span><a href='http://www.baidu.com'>react</a> </span></h3>
</div>
ReactDOM.render(ele, document.getElementById('root'));
复制代码
```

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

测试2:

```jsx
function tick() {
    const element = (
        <div>
            <h1>Hello, world!</h1>
            <h2>It is {new Date().toLocaleTimeString()}.</h2>
        </div>
      );
    ReactDOM.render(
        element,
        document.getElementById( 'root' )
    );
}

setInterval( tick, 1000 );
复制代码
```

问题:

在定义React组件或者书写React相关代码,不管代码中有没有用到React这个对象,我们都必须将其import进来,这是为什么?

```jsx
import React from 'react';    // 下面的代码没有用到React对象，为什么也要将其import进来
import ReactDOM from 'react-dom';

ReactDOM.render( <App />, document.getElementById( 'editor' ) );
复制代码
```

其实上面就是答案了!!!!

#### 组件和生命周期

在react中,组件有两种:函数组件和类组件,函数组件可以看作是类定义的一种简单形式

上面咱们说道:`React.createElement`的实现

```jsx
const React = {
    createElement
}

function createElement(tag, attrs, ...childrens) {
    return {
        tag,
        attrs,
        childrens
    }
}
export default React;

复制代码
```

这种实现不得不说我们只是来渲染原生DOM元素,而对于组件来说,`createElement`得到的参数略有不同:如果JSX片段中的每个元素是组件,那么`createElement`的第一个参数tag将会是一个方法,而不是字符串

> 区分组件和原生DOM的工作,是`babel-plugin-transform-react-jsx`帮我们实现的

例如在处理`<Home name='react' />`时,`createElement`方法的第一个参数`tag`,实际上就是我们定义的`Home`方法:

```jsx
function Home(props){
    return <h1>hello,{props.name}</h1>
}
复制代码
```

我们不需要对createElement做修改，只需要知道如果渲染的是组件，tag的值将是一个函数

##### 组件基类React.Component

通过类的方式定义组件,我们需要继承`React.Component`

```jsx
class Home extends React.Component {
    render() {
        return (
            <h1>hello,{this.props.name}</h1>
        );
    }
}
ReactDOM.render(<Home name='react' />, document.getElementById('root')); 
复制代码
```

##### Component

React.Component包含了一些预先定义好的变量和方法啊,我们一一实现它:

react文件夹下,新建component.js,定义一个`Component`类:

```jsx
import {
    renderComponent
} from '../react-dom';
// React.Component包含了一些预先定义好的变量和方法
class Component{
    // 通过继承React.Component定义的组件有自己的私有状态state，可以通过this.state获取到。同时也能通过this.props来获取传入的数据。
    // 所以在构造函数中， 我们需要初始化state和props
    constructor(props = {}) {
        this.state = {};
        this.props = props;
    }
    setState(statChange){
        // 将修改合并到state
        Object.assign(this.state,stateChange);
        // 重新渲染组件
        renderComponent(this);
    }
}
export default Component;
复制代码
```

> 你可能听说过React的`setState`是异步的，同时它有很多优化手段，这里我们暂时不去管它，在以后会有一篇文章专门来讲`setState`方法。

组件内部的`state`和渲染结果无关,当`state`改变时通常会触发渲染,为了让React知道我们改变了state,我们只能通过setState方法来修改数据,我们可以通过`Object.assign`来做一个简单的实现,在每次更新`state`后,我们需要调用`renderComponent`方法来重新渲染组件,`renderComponent`方法的实现

```jsx
export function renderComponent(comp) {
    // 声明一个初始化变量,用来保存当前js节点对象
    let base;
    const renderer = comp.render(); //调用render方法之后,返回的是jsx对象
    // 返回js节点对象
    base = _render(renderer);
	
    // 如果调用了setState,节点替换
    if (comp.base && comp.base.parentNode) {
        comp.base.parentNode.replaceChild(base, comp.base);
    }
    // 为组件挂载js节点对象
    comp.base = base;
}
复制代码
```

##### render方法

```jsx
const ReactDOM = {
    render
}

function render(vnode,container) {

    return container.appendChild(_render(vnode));
}
复制代码
```

实现的render方法只支持渲染原生的DOM元素,不支持渲染组件,我们需要修改`ReactDOM.render`方法,让它支持渲染组件

思路:如果是函数组件或者类组件,则vnode.tag的类型为函数

在render方法中添加代码如下:

```jsx
// 如果是一个函数,则渲染组件
if (typeof vnode.tag === 'function') {
    // 1.创建组件
    const comp = createComponent(vnode.tag, vnode.attrs)
    // 2.设置组件的属性
    setComponentProp(com,vnoe.attrs);
    // 3.返回当前组件的jsx对象
    return comp.base;
}
复制代码
```

##### 组件渲染

`createComponent`方法用来创建组件实例,并且我想将函数定义组件扩展成为类定义的组件,为了后面方便处理

```jsx
/* 
    @comp: 为当前函数组件还是类定义的组件  (都是函数)
    @props: 组件属性
    @return: 返回当前组件
*/
function createComponent(comp,props) {
    let inst;
    // 如果是类定义的组件,则直接返回实例
    if (comp.prototype && comp.prototype.render){  //有原型,原型上有render函数,则是类
       inst = new comp(props);
    }else{
        /* 
            function Home(){
                return <div>hello</div>
            }
        */
        // 是函数组件,则将函数组件扩展成类定义的组件 方便后面的统一处理
        inst = new Component(props);
        // 改变构造函数指向
        inst.constructor = comp;
        // 定义render函数
        inst.render = function() {
            return this.constructor(props);
        }
    }
    return inst;
}

复制代码
setComponentPorps`方法用来更新`props
function setComponentProp(comp,props) {
    //更新props
    comp.props = props;
    //重新渲染组件
    renderComponent(comp)
}
复制代码
```

##### 生命周期方法实现

在`setComponentProps`方法中可以实现`componentWillMount`和`componentWillReceiveProps`两个生命周期方法

```jsx
function setComponentProp(comp,props) {
    // 实现componentWillMount，componentWillReceiveProps两个生命周期方法
    if(!comp.base){
        if (comp.componentWillMount) comp.componentWillMount();
    }else if(comp.componentWillReceiveProps){
        comp.componentWillReceiveProps(props);
    }
    comp.props = props;
    renderComponent(comp)
}
复制代码
```

基本的组件渲染和声明周期方法已经实现

测试1:

```jsx
import React from './react';
import ReactDOM from './react-dom/index.js';
class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            num: 0
        }
    }
    handlerClick(){
        this.setState({
            num: this.state.num + 1
        })
    }
    componentWillMount(){
        console.log('组件将要挂载');
        
    }
    componentWillReceiveProps(props){
        console.log('将要接收属性',props);
        
    }
     componentWillUpdate() {
         console.log('组件将要更新');
     }
     componentDidMount(){
         console.log('组件已经挂载完成');
     }

    render() {
        console.log(this.state.num);
        return (
            <div>
                <h3>number:{this.state.num}</h3>
                <button onClick={this.handlerClick.bind(this)}>add</button>
            </div>
        );
    }
}
ReactDOM.render(<Counter name='react'/>, document.getElementById('root')); 
复制代码
```

效果:

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

组件挂载输出一次,后面每次更新会输出update

React的核心功能基本已经实现,但是我们目前的做法是每次更新都重新渲染整个组件甚至整个应用,这样的做法在页面复杂时会暴露出性能的问题,为了减少DOM操作,React又使用diff算法对DOM进行操作

#### diff算法

diff算法?what?什么玩意

如何减少DOM更新:我们需要找出渲染前后真正变化的部分,只更新这一部分.而对比变化,找出需要更新部分的算法称之为**diff算法**

##### 对比策略

在前面我们实现了`render`方法,它能讲虚拟DOM转换成真正的DOM

但是我们需要改进它,不要让它傻乎乎地重新渲染整个DOM数,而是找出真正变化的部分进行替换

这部分很多类似React框架实现方式都不太一样，有的框架会选择保存上次渲染的虚拟DOM，然后对比虚拟DOM前后的变化，得到一系列更新的数据，然后再将这些更新应用到真正的DOM上。

**我们会选择直接对比虚拟DOM和真实DOM，这样就不需要额外保存上一次渲染的虚拟DOM，并且能够一边对比一边更新，这也是我们选择的方式。**

不管是DOM还是虚拟DOM，它们的结构都是一棵树，完全对比两棵树变化的算法时间复杂度是O(n^3)，但是考虑到我们很少会跨层级移动DOM，所以我们只需要对比同一层级的变化。

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

总而言之,我们的diff算法有两个原则

- 对比当前真实的DOM和虚拟DOM,在对比过程中直接更新真实DOM
- 只对比同一层级的变化

##### 起步

先修改render函数,将_render方法渲染的方式改为我们即将写的diff算法方式

```
/react-dom/index.js
import diff from './diff';
const ReactDOM = {
    render
}

function render(vnode, container) {
    // return container.appendChild(_render(vnode));
    return diff(dom,vnode,container);
}
/* 
复制代码
```

由上面的`diff()`可以看出,传入了 `真实DOM对象,虚拟DOM对象,根元素`

##### 实现

实现一个diff算法,它的作用是对比真实的DOM和虚拟DOM,最后返回更新后的DOM

```
/react-dom/diff.js
/*
dom:真实DOM
vnode:虚拟DOM
container: 容器
return : 更新后的DOM
*/
export function diff(dom,vnode,container) {
    // 返回更新后的节点
    let ret =  diffNode(dom,vnode);

    return ret;
}
function diffNode(dom,vnode){
	//......
}
复制代码
```

接下来实现这个方法

在这之前先来回忆一下我们虚拟DOM的结构:

虚拟DOM的结构可以分为三种,分别表示文本,原生DOM节点以及组件

```jsx
// 原生DOM节点的vnode
{
    tag: 'div',
    attrs: {
        className: 'container'
    },
    children: []
}

// 文本节点的vnode
"hello,world"

// 组件的vnode
{
    tag: ComponentConstrucotr,
    attrs: {
        className: 'container'
    },
    children: []
}
复制代码
```

##### 对比文本节点

首先考虑最简单的文本节点,如果当前的DOM就是文本节点,则直接更新内容,否则就新建一个文本节点,并移除原来的DOM

```jsx
function diffNode(dom,vnode) {
    let out = dom;
    if(typeof vnode === 'string'){
      // 如果当前的dom是文本节点,则直接更新内容
      if(dom && dom.nodeType === 3){
          if(dom.textContent !== vnode) dom.textContent = vnode;
      } else{
        //  如果dom不是文本节点,则新建一个文本节点dom,并移除原来的dom
        out = document.createTextNode(vnode);
        if(dom && dom.parentNode){
            dom.parentNode.replaceChild(out,dom);
        }
      }
        return out;
    }
    return out;
}
复制代码
```

文本节点十分简单,它没有属性,也没有子元素

##### 对比组件

之前也说过,react组件分为函数组件和类组件,我们定制一个方法`diffComponent`

```jsx
function diffNode(dom,vnode){
    //...
    //如果是一个组件
    if (typeof vnode.tag === 'function') {
        return diffComponent(dom, vnode);
    }
    //....
}
function diffComponent(dom, vnode) {
    let comp = dom;
    // 如果组件没有变化,则重新设置 props;   执行
    if (comp && comp.constructor === vnode.tag) {
        // 重新设置 props
        setComponentProp(comp, vnode.attrs);
        // 赋值
        dom = comp.base;
    } else {
        // 如果组件类型变化,则移除掉原来组件,并渲染新组件
        // 移除
        if (comp) {
            unmountComponent(comp);
            comp = null;
        }
        //核心代码
        // 1.创建新组件
        comp = createComponent(vnode.tag, vnode.attrs);
        // 2.设置组件属性
        setComponentProp(comp, vnode.attrs);
        dom = comp.base;
    }
    return dom;
}
复制代码
```

##### 对比非文本DOM节点

如果vnode表示的是一个非文本DOM节点,分两种情况分析:

情况一: 如果真实DOM不存在,表示此节点是新增的

```jsx
if(!dom){
    updateDOM = document.createElement(vnode.tag);
}
复制代码
```

情况二:如果真实DOM存在,需要`对比属性`和`对比子节点`

##### 对比属性

找出来节点的属性以及事件监听的变化 单独起一个`diffAttributes`方法

```jsx
//设置属性和移除属性的时候 使用到了此方法.注意:移除属性 只需要设置属性值为undefined就可以
import {
    setAttribute
} from './index';
function diffNode(dom,vnode){
    let out = dom;
	//  ....
    // 如果是非文本DOM节点
    // 真实DOM不存在
    if(!dom){
      out = document.createElement(vnode.tag);
	}
    // 如果真实DOM存在,需要对比属性和对比子节点
    diffAttributes(out,vnode);
    //...
    return out;
}
function diffAttributes(dom, vnode) {
    // 保存之前的真实DOM的所有属性
    const oldAttrs = {};
    const newAttrs = vnode.attrs; //虚拟DOM的属性 (也是最新的属性)
    // 获取真实DOM属性
    const domAttrs = dom.attributes;
    // const domAttrs =  document.querySelector('#root').attributes;
    [...domAttrs].forEach(item => {
        oldAttrs[item.name] = item.value;
    })
    // 比较
    // 如果原来的属性不在新的属性当中,则将其移除掉  (属性值直接设置undefined)
    for (let key in oldAttrs) {
        if (!(key in newAttrs)) {
            // 移除
            setAttribute(dom, key, undefined);
        }
    }
    // 更新新的属性值
    for (let key in newAttrs) {
        if (oldAttrs[key] !== newAttrs[key]) {
            console.log(dom, newAttrs[key], key);
            // 只更值不相等的属性
            setAttribute(dom, key, newAttrs[key]);
        }
    }
}
复制代码
```

##### 对比子节点

节点对比完成之后,接下来对比它的子节点

这个时候会有一个问题,前面我们实现的不同的diff算法,都是明确知道哪一个是真实DOM和虚拟DOM对比,但是子节点childrens是一个数组,他们可能改变顺序,或者数量有所变化,我们很难确定是和虚拟DOM对比的是哪一个?

> 思路:给节点设置一个key值,重新渲染时对比key值相同的节点,这样我们就能找到真实DOM和哪个虚拟DOM进行对比了

对比子节点的方法有点复杂,在这里理解一下原理

```jsx
function diffNode(dom,vnode){
     if (vnode.childrens && vnode.childrens.length > 0 || (out.childNodes && out.childNodes.length > 0)) {
        diffChildren(out, vnode.childrens);
    }
}
复制代码
// 对比子节点
function diffChildren(dom, vchildren) {
    const domChildren = dom.childNodes;
    const children = [];
    const keyed = {};
    // 将有key的节点(用对象保存)和没有key的节点(用数组保存)分开
    if (domChildren.length > 0) {
        [...domChildren].forEach(item => {
            // 获取key
            const key = item.key;
            if (key) {
                // 如果key存在,保存到对象中
                keyed[key] = item;
            } else {
                // 如果key不存在,保存到数组中
                children.push(item)
            }

        })
    }
    if (vchildren && vchildren.length > 0) {
        let min = 0;
        let childrenLen = children.length; //2
        [...vchildren].forEach((vchild, i) => {
            // 获取虚拟DOM中所有的key
            const key = vchild.key;
            let child;
            if (key) {
                // 如果有key,找到对应key值的节点
                if (keyed[key]) {
                    child = keyed[key];
                    keyed[key] = undefined;
                }
            } else if (childrenLen > min) {
                // alert(1);
                // 如果没有key,则优先找类型相同的节点
                for (let j = min; j < childrenLen; j++) {
                    let c = children[j];
                    if (c) {
                        child = c;
                        children[j] = undefined;
                        if (j === childrenLen - 1) childrenLen--;
                        if (j === min) min++;
                        break;
                    }
                }
            }
            // 对比
            child = diffNode(child, vchild);
            // 更新DOM
            const f = domChildren[i];
            if (child && child !== dom && child !== f) {
                // 如果更新前的对应位置为空，说明此节点是新增的
                if (!f) {
                    dom.appendChild(child);
                    // 如果更新后的节点和更新前对应位置的下一个节点一样，说明当前位置的节点被移除了
                } else if (child === f.nextSibling) {
                    removeNode(f);
                    // 将更新后的节点移动到正确的位置
                } else {
                    // 注意insertBefore的用法，第一个参数是要插入的节点，第二个参数是已存在的节点
                    dom.insertBefore(child, f);
                }
            }
        })
    }
}
复制代码
```

最后再修改`renderComponent`方法的两个地方

```jsx
export function renderComponent(comp) {
    let base;
    // 调用render之后 返回jsx对象
    const renderer = comp.render();
    // 实现componentWillUpdate componentDidUpdate componentDidMount生命周期方法
    if (comp.base && comp.componentWillUpdate) {
        comp.componentWillUpdate();
    }

    // 调用_render方法,返回当前js节点对象
    // base = _render(renderer);
    //修改
    base = diffNode(comp.base,renderer);
    comp.base = base;
    
    
    if (comp.base) {
        if (comp.componentDidUpdate) comp.componentDidUpdate();
    } else if (comp.componentDidMount) {
        comp.componentDidMount();
    }

    // 节点替换 删掉这部分
    // if (comp.base && comp.base.parentNode) {
    //     comp.base.parentNode.replaceChild(base, comp.base);
    // }
    // 为当前组件挂载当前jsx节点对象(貌似也可以不加)
    // comp.base = base;
    // console.log(comp);
    // base._component = comp;
}
复制代码
```

看一下有无diff算法之后的网页效果:

**diff算法网页效果**

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

**无diff算法网页效果**

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

​	我们实现了diff算法，通过它做到了每次只更新需要更新的部分，极大地减少了DOM操作。React实现远比这个要复杂，特别是在React 16之后还引入了Fiber架构，但是主要的思想是一致的。

​	实现diff算法可以说性能有了很大的提升，但是在别的地方仍然后很多改进的空间：每次调用setState后会立即调用renderComponent重新渲染组件，但现实情况是，我们可能会在极短的时间内多次调用setState。 假设我们在上文的Counter组件中写出了这种代码

```jsx
handlerClick() {
    for ( let i = 0; i < 100; i++ ) {
        this.setState( { num: this.state.num + 1 } );
    }
}
复制代码
```

那以目前的实现，每次点击都会渲染100次组件，对性能肯定有很大的影响。

下节课改进setState方法

#### 异步的setState

上节,虽然我们实现了diff算法,性能有非常大的改进.同时我们也指出了问题:按照目前的实现,每次调用setState都会触发更新,如果组件内执行这样一段代码:

```jsx
for ( let i = 0; i < 100; i++ ) {
    this.setState( { num: this.state.num + 1 } );
}
复制代码
```

那么执行这段代码会导致这个组件被重新渲染100次,这对性能是一个非常大的负担

##### 真正的React是怎么做的

```jsx
import React,{Component}from 'react';
import ReactDOM from 'react-dom';
class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            num: 0
        }
    }
    componentDidMount(){
        for (let i = 0; i < 100; i++) {
            this.setState({
                num: this.state.num + 1
            });
            console.log(this.state.num);
            
        }
    }
    
    render() {
        return (
            <div>
                <h3>{this.state.num}</h3>
            </div>
        );
    }
}
ReactDOM.render(<App name='react'/>,document.querySelector('#root'))
复制代码
```

效果显示: 你会发现..... What?为什么输出了100个0,我不是想让它每次循环+1么?而且你还发现页面还渲染了1

这说明每次循环中,拿到的state仍然是更新之前的

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

看到这种效果,不要惊讶,这是React的优化手段,但是显然它会导致一些不符合直觉的问题,所以针对这种情况,React给出了一种解决方法:setState接收的参数还可以是一个函数,在这个函数中可以获取先前的状态,并通过这个函数的返回值得到下一个状态

修改组件:

```jsx
  componentDidMount(){
        for (let i = 0; i < 100; i++) {
            this.setState((prevState)=>{
                console.log(prevState.num);
                
                return {
                    num : prevState.num + 1
                }
            })  
        }
    }
复制代码
```

> 这种用法是不是很像数组的reduce方法

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

通过以上的演示,我们要做两个事情

1. 异步更新state,将短时间内的多个setState合并成一个
2. 为了解决异步更新导致的问题,增加另一种形式的setSatet:接受一个函数作为参数,在函数中可以得到前一个状态并返回下一个状态

##### 合并setState

```jsx
setState(stateChange) {
    // 1.外部调用setState之后,更新state
    Object.assign(this.state, stateChange);
    // 2.渲染组件 参数为当前组件(函数),容器
    renderComponent(this);
}
复制代码
```

这种实现,每次调用setState都会更新state并马上渲染一次

##### setState队列

为了合并setState,我们需要一个队列来保存每次setState的数据,然后在一段时间后,清空这个队列来渲染组件

> 队列是一种数据结构,它的特点是先进先出,可以通过js数组的push和shift方法模拟,然后需要定义一个"入队"的方法,用来将更新添加进队列

```jsx
const setStateQueue = [];
// 创建队列  
// 队列: 先进先出
export function enqueueSetState(stateChange, component) {
    // 入队 让所有设置的状态 都放入到队列中
    setStateQueue.push({
        stateChange,
        component
    })
}
复制代码
```

然后修改组件的setState方法,不再直接更新state和渲染组件,而是添加进队列中

```jsx
import {
    enqueueSetState
} from './set_state_queue';
class Component {
    constructor(props = {}) {
        this.state = {};
        this.props = props;
    }
    setState(stateChange) {
        // 1.外部调用setState之后,更新state
        // Object.assign(this.state, stateChange);
        // 2.渲染组件 参数为当前组件(函数),容器
        // renderComponent(this);
        enqueueSetState(stateChange,this);
    }
}
复制代码
```

现在队列是有了,怎么清空队列并渲染组件呢?

##### 清空队列

定义一个flush方法,它的作用就是清空队列

```jsx
function flush() {
    let item;
    // 遍历state
    while (item = setStateQueue.shift()) {
        const {stateChange,component} = item;
        // 如果没有prevState,则将当前的state作为初始的prevState
        if (!component.prevState) {
            component.prevState = Object.assign({}, component.state);
        }
        // 如果stateChange是一个方法,也就是setState的第一种形式
        if (typeof stateChange === 'function') {
            Object.assign(component.state, stateChange(component.prevState, component.props))
        } else {
            // 如果stateChange是一个对象,则直接合并到setState中
            Object.assign(component.state, stateChange);
        }
        component.prevState = component.state;
    }

    // 遍历组件
    while (component = renderQueue.shift()) {
        renderComponent(component);
    }
}
复制代码
```

这只是实现state的更新,接下来渲染组件,渲染组件不能遍历时进行,因为同一个组件可能会多次添加到队列中,我们需要另一个队列来保存所有组件,不同在于,这个队列内不会有重复的组件

我们在`enqueueSetState`方法中,可以做这件事

```jsx
const setStateQueue = [];
const renderQueue = [];
function enquequSetState(stateChange,component){
    setStateQueue.push({
        stateChange,
        component
    })
    // 如果renderQueue里没有当前组件,则添加到队列中
    let r = renderQueue.some(item => {
        return item === component;
    })
    if (!r) {
        renderQueue.push(component);
    } 
}
复制代码
```

在`flush`方法中,我们还需要遍历renderQueue,来渲染每一个组件

```jsx
function flush(){
	let item,component;
    while(item = quequ.shift()){
        //....
    }
    // 遍历组件
    while (component = renderQueue.shift()) {
        renderComponent(component);
    }
}
复制代码
```

以上操作,添加状态队列和组件队列,以及更新状态和更新组件的任务都已完成,接下来还剩下最重要的一件事情:**什么时候执行flush方法**

我们需要合并一段时间内所有的setState,也就是在一段时间后才执行flush方法来清空队列,关键是这`一段时间`怎么决定

答案毫无疑问就是利用js的事件队列机制

```jsx
setTimeout( () => {
    console.log( 2 );
}, 0 );
Promise.resolve().then( () => console.log( 1 ) );
console.log( 3 );
复制代码
```

你可以打开浏览器的调试工具运行一下，它们打印的结果是：

```js
3
1
2
复制代码
```

我们可以利用事件队列,让flush的所有同步任务后执行

```jsx
function defer( fn ) {
    return Promise.resolve().then( fn );
}
export function enqueueSetState(stateChange, component) {
    // 如果setStateQueue的长度是0,也就是在上次flush执行之后第一次往队列里添加
    if (setStateQueue.length === 0) {
        defer(flush);
    }
    // 入队
    setStateQueue.push({
        stateChange,
        component
    })
    // 如果renderQueue里没有当前组件,则添加到队列中
    let r = renderQueue.some(item => {
        return item === component;
    })
    if (!r) {
        renderQueue.push(component);
    }    
}
复制代码
```

这样在一次“事件循环“中，最多只会执行一次flush了，在这个“事件循环”中，所有的setState都会被合并，并只渲染一次组件。

我们又实现了一个很重要的优化：合并短时间内的多次setState，异步更新state。 到这里我们已经实现了React的大部分核心功能和优化手段了

到此为止。手写自己的一个React基本完成了


