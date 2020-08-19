# React学习总结

[TOC]

## React 入门

React 起源于 Facebook 的内部项目，因为该公司对市场上所有 JavaScript MVC 框架，都不满意，就决定自己写一套，用来架设 Instagram 的网站。做出来以后，发现这套东西很好用，就在2013年5月开源了。



### 安装

react.js是react的核心库, react-dom.js提供与dom相关操作的功能。另外由于ReactJS使用`JSX`语法（浏览器无法识别），所以需要额外的工具将`JSX`转成浏览器认识的JS和HTML。例如：`Babel`。

- 安装react cli `npm install -g create-react-app`
- 安装React/ReactDOM `npm install react reactDOM --save`



### ReactDOM.render(jsx, container[callback])

ReactDOM.render 是 React 的最基本方法，用于将ReactElement（JSX）转为 HTML，并插入指定的 DOM 节点。ReactElement被插入到真正的DOM之前是存在于内存中的虚拟DOM。根据 React 的设计，所有的 DOM 变动，都先在虚拟 DOM 上发生，然后再将实际发生变动的部分，反映在真实 DOM上，这种算法叫做 [DOM diff](http://calendar.perfplanet.com/2013/diff/) ，它可以极大提高网页的性能表现。

```react
// Hello react
ReactDOM.render(<h1>Hello react</h1>, document.body)
```



### JSX语法

JSX语法更像是javascript的扩展，而不是HTML的template。JSX可替换正常的Javascript的表达式。

JSX 的基本语法规则：遇到 HTML 标签（以 `<` 开头），就用 HTML 规则解析；遇到代码块（以 `{` 开头），就用 JavaScript 规则解析。JSX标签属性可以是""或者{}但是不能混用。标签必须闭合。{}的值会被escape。JSX会被Babel解析成 React.createElement(标签类型，属性，子标签...)->Object/React Element

JSX 允许直接在模板插入 JavaScript 变量。如果这个变量是一个数组，则会展开这个数组的所有成员。



### React Element

ReactElement使用JSX语法书写和HTML Element不同，React Element是plain object。由Babel翻译成ReactElement对象。



### React Component

React组件像是一个生产React Element的函数，接受props作为参数，返回React Element。组件的定义可以使用`functional`也可以使用ES6的`class`。

```react
//一个大栗子
// functional
function Hello(props){
  return <h1>Hello {props.name}</h1>
}

// ES6 function
const Hello = (props)=>(<h1>props.name</h1>)

// ES6 class
class Hello extends React.Component {
  constructor(){
    super()
    //默认state
    this.state={marked:true}
    //绑定事件函数
    this.changeColor = this.changeColor.bind(this)
  }
  // 声明周期函数
  // 自定义属性
  render() {
    return <h1>this.props.name</h1>
  }
}

changeColor(e){
  // 取得实际的dom元素
  console.log(this.refs.hello.id)
  this.setState((prevState, props)=>({
      marked:!prevState.marked
    }));
}

// 默认props
Hello.defaultProps={
  name:'JK'
}

// props类型校验, 需添加PropTypes依赖
Hello.propTypes={
  name: React.PropTypes.string.isRequired // 可以链式使用 string + required
}

// 引用Componet
reactDOM.render(<Hello ref='hello' name='JK' onClick={this.changeColor} style={{color:this.marked?'red':'black'}}/>, document.body, ()=>(console.log('Done!')))
```



组件定义的时候需要注意的事项：

1. 组件的定义首字母必须大写
2. 必须有`render`函数且返回的模板只有一个根节点
3. 如果`render`函数返回null则组件不显示，但是生命周期函数依然执行
4. `constructor`函数中取不到`this.props`的值




#### props

React的组件定义中可以通过`this.props.[propName]`取得传入组件的属性值。其值可以是任何的javascript类型，而不单单是`string`。

注意事项：

1. 属性`class`和`for`是JavaScript保留字，需要使用`className`和`htmlFor`代替

2. 使用`this.props.children`取得模板的子节点（多个返回数组类型，单个返回对象类型）。

   `React.Children.map(this.props.children, function (child) {})`

3. 使用`[class].propTypes`设置属性的类型检查：`属性名=验证类型`

4. 使用`[class].defaultProps`方法设置属性的默认值

   ​

#### refs

HTML DOM是由React维护的，我们操作的只是ReactElement（js obj）。那么有的时候需要取到实际怎么办呢？

需要在组件上添加`ref`属性，使用的时候在组件中`this.refs.[refId]`就可以了。



#### state

React component的state属性有特殊的意义，每当调用`this.setState`方法时React都会调用Component的`render`方法重新渲染UI。如果一个属性没有在`render`函数返回的React Element中出现，那么他就不应该放在state中。

注意事项：

1. this.state={}这种写法不会重新渲染React Component
2. `this.state`保存组件的状态，使用`this.setState`方法设置属性，没有赋值的属性会被忽略。
3. `this.setState`不要在`constructor`中调用
4. `setState`是异步的，如果需要根据之前state的值计算新的state值那么应该这么写： `setState((prevState, props)=>())`




#### 事件

1. 在React Element上定义事件属性，属性名用`camel case`(因为element是js obj)，
2. 事件的注册只应该发生在定义模板的时候。
3. 事件的值是函数类型，如果函数中使用了`this`,那么应该使用bind，或者箭头函数绑定Object。




#### 生命周期

React Component的生命周期是由React维护的，它分为下面几个阶段：

```react
// Mounting, 组件的挂载阶段, 组件的实例被创建并插入到DOM中。通常第一调用reactDOM.render
constructor()
componentWillMount()
render()
componentDidMount()

// Updating, 用户交互发出事件，触发定义好的事件处理函数，调用setState方法
componentWillReceiveProps()
shouldComponentUpdate()
componentWillUpdate()
render()
componentDidUpdate()

// Unmounting, 组件从DOM上移除,如：ReactDOM.unmountComponentAtNode()
componentWillUnmount()

```



### React App的开发流程

1. 设计UI和API
2. 将UI拆分成树状组件
3. 创建静态页面(不用state)
4. 识别state，如果有下面情况之一，那么就不应该定义state
   1. state是否从props传入
   2. state是否不变
   3. state是否可以通过其他state计算得到
5. 思考state所需放入的组件
   1. 找到state影响的所有组件
   2. 找到步骤一组件的最近的父节点
   3. 就是它...
6. 定义用户交互事件




## React-Router



##参考

-  [阮一峰：react实例入门](http://www.ruanyifeng.com/blog/2015/03/react.html)

-  [React官方网站](https://facebook.github.io/react/)


