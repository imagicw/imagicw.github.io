# 全栈公开课2020学习笔记 01 - JavaScript 和 React

<!--more-->
{{< admonition info "关于笔记内容" open >}}
本文是博主学习赫尔辛基大学的 [全栈公开课2020](https://fullstackopen.com/) 过程中的学习笔记，文章分类结构与课程会有些许不同，本文部分内容引用自原文。
{{< /admonition >}}

## JavaScript

### 认识 Javascript

JavaScript 标准的正式名称是ECMAScript。 目前最新的版本是2019年6月发布的，名为ECMAScript 2019 ，即ES10。

#### Variables 变量

JavaScript中有 `const` `let` `var` 这几种定义变量的方法，其中 `const` 实际上是定义了一个常量，也就是其值不能再修改。 `let` 定义了一个普通的变量。 `var` 在很长一段时间里，是 JavaScript的唯一定义变量的方法，其余两种是在 ES6 版本添加的。在一些特定情况，var 的工作方式与大多数语言中的变量定义相比是十分不同的。所以不建议使用 `var`。

值得注意的是 `let` 声明一个块作用域的**局部变量**，**在该块区域以外是无法被调用的**。

```js
const a = 100
let b = 200

b += 10          // console.log(b) -> 210
b = '你好'       // console.log(b) -> 你好
a = 1000        // 报错
```

#### Array 数组

数组可以使用 `const` 定义，因为数组是一个对象，而且数组的变量总是指向这同一个对象，当添加新元素的时，数组的内容也将发生变化。

```js
const array = [1, 3, -3]    // 定义 array 变量

array.push(9)               // 向 array 数组添加元素 数字9

console.log(array.length)   // 数组 length 为 4
console.log(array[1])       // 打印 array 的第 2 个元素 为 3

// 遍历数组使用 forEach

array.forEach(value =>
{
    console.log(value)      // 将数组所有内容依次打印出来 1, 3, -3, 9
})

```

上述案例中使用的是 push 方法将新元素添加到数组中。在使用 React 时，经常使用函数式编程的技巧。函数编程范型的一个特点，就是使用不可变的数据结构。在React代码中，最好使用concat方法，因为它不向数组中添加元素，而是创建一个新数组，新数组中包含了旧数组和新的元素。

```js
const array = [1, 3, -3]
const array2 = array.concat(5)

// 此时 array 没有改变
// array2 为 [1, 3, -3, 5]
```

`array.concat(5)` 这种方法调用不会向 `array` 数组添加新的元素，而是直接返回一个包含 `array` 以及 `concat` 的元素的一个新数组。

`map` 方法遍历

以下案例使用了 `map` 方法创建了一个新数组，将 `array` 的每一个元素作为函数的入参来创建新的元素。

```js
const array3 = array.map(value => value * 10)
// array3 为 [10, 30, -30]
```

`map` 方法还可以将数组转换成完全不同的内容，下例将 array 的整数值转换成了包含 HTML 字符串的数组。

```js
const array4 = array.map(value => '<li>' + value + '</li>')
// array4 为 ['<li>1</li>', '<li>3</li>', '<li>-3</li>']
```

数组中的单个元素可以使用[解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)赋给变量。

```js
const array = ['apple', 'banana', 'cherry', 'durian', 'fig']
const [fruit1, fruit2, ... restFruit] = array

// fruit1 为 apple
// fruit2 为 banana
// restFruit 为 [cherry, durian, fig]
````

#### Objects 对象

定义对象的不同方式
- 对象字面量 (Object Literals)，使用大括号 {}
- 使用句点 .
- 使用中括号 []

对象属性的值可以是任何类型：`整数` `字符串` `数组` `对象`等。

```js
// 对象字面量
const object =
{
    name: 'imagic',
    age: 100,
    company:
    {
        name: 'BBC',
        location: 'UK',
    },
    devices: ['iPhone','MacBook Pro', 'raspberry']
}

// 句点方式 和 中括号

// 引用
console.log(object.name)            // 打印出 imagic
console.log(object['name'])         // 打印出 imagic

// 添加
object.address = '221B Baker Street'
object['nick name'] = 'carrot'      // 因为 nick name 中间有空格，只能使用中括号 [] 来完成。
```

#### Functions 函数

```js
// 定义函数
const sum = (num1, num2) =>
{
    return num1 + num2
}

// 调用函数
const result = sum(10,3)    // result 为 13
```

如果只有一个参数，我们可以在定义中去掉括号

```js
const square = num =>
{
    return p * p
}
```

如果函数只包含一个表达式，则不需要大括号。

```js
const square = num => num * num
```

这个方式在操作数组时特别方便。如数组的 `map` 方法。

箭头函数是随 ES6 一起添加到 JavaScript. 在之前定义函数的唯一方法是使用 `function`。

```js
// 定义函数
function square(num1, num2)
{
    return num * num2
}

// 函数表达式方法
const average = function(num1, num2)
{
    return (num1 + num2) / 2
}
```

#### Object method and "this" 对象方法以及“ this”关键字

箭头函数和使用 function 关键字的函数，在涉及到 this 关键字（指向对象本身）的行为上，有很大的不同。

function 方法

```js
const object =
{
    name: 'imagic',
    age: 100,
    greet: function ()
    {
        console.log('Hi, I am ' + this.name)
    }
}

object.greet()  // 打印出 'Hi, I am imagic'

// 也可以在对象创建之后再赋值给对象
object.growOlder = function ()
{
    this.age += 1
}

console.log(object.age)     // 打印出 100
object.growOlder()
console.log(object.age)     // 打印出 101
```

对象中定义的 `greet` 方法可以通过赋值给变量的方法引用，但是要注意以下情况：

```js
object.greet()      // 打印出 'Hi, I am imagic'

const referenceToGreet = object.greet()
referenceToGreet()  // 打印出 'Hi, I am undefined'
```

这是因为通过引用调用 `referenceToGreet()` 方法时，该方法不认识 `this` 是什么。与其他语言相反，在 JavaScript 中，`this` 的值是根据 方法如何调用来定义的。

当通过引用调用该方法时，`this` 的值就变成了所谓的全局对象，这并不是我们所想要的结果。所以在开发时要尽量避免使用 `this` 来避免潜在的问题。

消除由 `this` 引起的问题的方法有几种办法：

- 使用 setTimeout 方法
- 使用 bind 方法

```js
setTimeout(object.greet, 1000)
```

当使用 `setTimeout` 来调用 `object.greet` 方法时，实际上是 JavaScript 引擎在调用该方法，此时的 `this` 是指向的是全局对象。

```js
setTimeout(object.greet.bind(object), 1000)
```

使用了 `bind` 方法创建了一个新函数，它将 `this` 绑定指向到了 `object`，这与方法的调用位置和方式无关。

使用箭头函数可以解决与 `this` 相关的一系列问题。 但是，**它不能当做对象的方法来使用**，因为那样的话 `this` 就不起作用了。

#### Classes 类

JavaScript 中并没有像面向对象程序语言中的类机制。 然而，JavaScript 中的一些新特性使得它能够「模拟」面向对象中的类。

与 ES6 一起引入到 JavaScript 中的类语法，它在很大程度上简化了 JavaScript 中的类(或者说像是类)的定义。

```js
// 定义名为 Person 的 class
class Person
{
    constructor(name, age)
    {
        this.name = name
        this.age = age
    }
    greet()
    {
        console.log('Hi, I am ' + this.name)
    }
}

// 创建两个 Person 对象
const imagic = new Person('imagic', 100)
imagic.greet()      // 'Hi, I am imagic'

const w = new Person('W', 99)
w.greet()           // 'Hi, I am W'
```

在语法方面，类以及由它们创建的对象非常类似于 Java 的类和对象。 它们的行为也非常类似于 Java 对象。 但在本质上，它们仍然是基于 JavaScript 的原型继承的对象。 这两个对象的类型实际上都是 Object，因为 JavaScript 实质上只定义了Boolean，Null，Undefined，Number，String，Symbol，BigInt，以及 Object几种类型。

{{< admonition tip "更加了解 JavaScript" false >}}
Mozillas [重新认识 JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)

深入了解 JavaScript [You Dont Know JS](https://github.com/getify/You-Dont-Know-JS)
{{< /admonition >}}

---

## React

### 认识 React

React 是一个用于构建用户界面的 JavaScript 库。[^react]

#### 安装 React

以下默认系统已经安装了 `node`，如果无法按照以下命令操作，请查看 `node` 版本是否需要更新。

在 `终端 terminal` 使用 `create-react-app` 工具创建 React 应用。

```bash
npx create-react-app project
cd project
```
以上创建了 `project` 应用，并进入了它的目录，使用以下命令运行这个应用。

```bash
npm start
```

默认情况下访问应用的地址为 `localhost:3000`。

#### Component 组件

- React 组件命名**必须首字母大写**。
- React 组件内容**只能有一个根元素**。（虽然可以使用创建组件数组的方式来规避这个问题，但会使代码看起来不好看，所以不使用）
- 根元素可以使用空元素 `<></>`，这样在 Dom 树中不会有额外的 `<div>` 元素了。
- **不要在组件内定义组件**，最大的问题是 React 在每次渲染时，会将内部的组件当作一个新的组件。这会导致 React 无法去优化组件。

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

// 定义 App 组件
const App = () =>
{
    const now = new Date()
    const a = 10
    const b = 20

    return (
        <div>
            <p>Hello world, it is {now.toString()}</p>
            <p>
                {a} plus {b} is {a + b}
            </p>
        </div>
    )
}

// 将组件 App 渲染到 id 为 'root' 的 div 中，该元素在 public/index.html 中定义。
ReactDOM.render(<App />, document.getElementById('root'))
```

#### JSX

React 组件的布局大部分使用 JSX 编写的，JSX 和 HTML 非常相似，React 组件返回的 JSX 会被编译成 JavaScript。

JSX 是一种「类 XML」语言，所以每个标签都需要关闭。如 `<br>` 必须写成 `<br />`。


#### Multiple Component 多组件

React 的核心理念就是通过将许多定制化的、可重用的组件组合成应用。有一个约定，就是应用的组件树顶部都要有一个 root 组件叫做 App。

```jsx
const Hello = () =>
{
    return (
        <div>
            <p>你好，世界！</p>
        </div>
    )
}

const App = () =>
{
    return (
        <div>
            <h1>欢迎</h1>
            <Hello />
            // Hello 组件可以重复使用 
            <Hello />
            <Hello />
        </div>
    )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

#### Props: passing data to components Props: 向组件传递数据

先看一段代码

```jsx
const Hello = (props) =>
{
    return (
        <div>
            <p>你好，{props.age} 岁的 {props.name}</p>
        </div>
    )
}
```

我们给 `Hello` 组件设置了一个参数 props，它接收了一个对象，对象具有组件中所定义的所有`属性`对应的字段。

```jsx
const App = () =>
{
    const name = 'W'
    const age = 100

    return (
        <div>
            <h1>欢迎</h1>
            <Hello name='imagic' age={100} />
            <Hello name={name} age={age+100} />
        </div>
    )
}
``` 

`props` 的值可以是字符串，也可以是 JavaScript 表达式的结果。如果是表达式，则需要用花括号`{}`括起来。

### 组件状态，事件处理

#### Component helper functions 组件辅助函数

组件辅助函数实际上是在另一个函数中定义的，这个函数是用来定义组件的行为的。

`bornYear` 函数可以访问 `Hello` 组件中所有的 `props`，不需要单独将 `props.age` 单独传参给 `bornYear`。

```jsx
const Hello = (props) =>
{
    const bornYear = () =>
    {
        const yearNow = new Date().getFullYear()
        return yearNow - props.age
    }

    return (
        <div>
            <p>
                Hellp {props.name}, you are {props.age} years old.
            </p>
            <p>
                So you were probably born in {bornYear()}
            </p>
        </div>
    )
}
```

#### Desctructuring 解构

结构是 ES6 规范中添加的一个特性，它允许我们在赋值时从对象和数组中解构出值。

使用解构的方法可以将属性直接赋值给变量，从而简化组件。

以下是常规的方法

```jsx
const props =
{
    name: 'imagic',
    age: 100
}

const Hello = (props) =>
{
    const name = props.name
    const age = props.age

    const bornYear = () => new Date().getFullYear() - age

    return (
        <div>
            <p>
                Hellp {name}, you are {age} years old.
            </p>
            <p>
                So you were probably born in {bornYear()}
            </p>
        </div>
    )
}
```

解构可以更加方便的将对象的属性的值提取到单独的变量中：

```jsx
const props =
{
    name: 'imagic',
    age: 100
}

const Hello = (props) =>
{
    const { name, age } = props
    const bornYear = () => new Date().getFullYear() -age
    // 以下与上述案例相同，省略
}
```

也可以使用如下方法直接将属性值赋给变量：

```jsx
const Hello = ({ name, age }) =>
```

#### Page re-rendering 页面重渲染

页面初次渲染之后，外观是不会变化的，所以如果需要页面变化，则需要将页面重新渲染。

*此处使用的是 `ReactDOM.render` 方法，不是重新渲染组件的推荐方法，以下只作为了解即可。*

```jsx
const App = (props) =>
{
    const { counter } = props
    return(
        <div>{counter}</div>
    )
}

let counter = 1

ReactDOM.render(
    <App counter={counter} />,
    document.getElementById('root')
)
```

以上代码如果添加 `counter += 1` 命令，`App` 部件是不会重新渲染，我们需要再次调用 `ReactDOM.render` 的方法让组件重新渲染：

```jsx
const App = (props) =>
{
    const { counter } = props
    return (
        <div>{counter}</div>
    )
}

let counter = 1

const refresh = () =>
{
    ReactDOM.render(
        <App counter={counter} />,
        document.getElementById('root')
    )
}

refresh()       // 第一次渲染，页面显示 1
counter += 1
refresh()       // 第二次渲染，页面显示 2
counter += 1
refresh()       // 第三次渲染，页面显示3  
```

由上面可以看到，页面渲染了3次，页面从1变到了3，因为渲染速度较快，很难注意到1和2。如果想要看到看到1变到3的过程，我们可以使用 `setInterval` 或者 `setTimeout` 的方法增加延迟。

**不推荐使用这样的方法重新渲染，下面有更好的方法可以实现。**

#### Stateful conmponent 有状态组件

以上的组件都不包含任何组件的状态「无状态组件」，可以通过 `React` 的 `state hook` 向组件添加状态。

```jsx
import React, { useState } from 'react'     // 引入 useState 函数
import ReactDOM from 'react-dom'

const App = () =>
{
    /**
     *  使用 useState 函数并解构赋值将元素分配给 counter 和 setCounter 两个变量
     *  counter 变量被赋予初始值 state 为 0
     *  setCounter 被分配给一个函数，该函数用于修改 state，也就是用来修改 counter 的值
     */
    const [ counter, setCounter ] = useState(0)


    /**
     *  设置1秒后调用 setCounter 函数
     *  将 counter + 1 赋值给 counter
     */
    setTimeout(
        () => setCounter(counter + 1),
        1000
    )

    return (
        <div>{counter}</div>
    )
}

ReactDOM.render(
    <App />,
    document.getElementById('root')
)
```

当 1 秒后调用 setCounter 时，React 会重新渲染这个组件，此时 `counter` 由原先的值变为 `counter + 1`，也就是由 0 变为 1 ，然后 `setTimeout` 继续倒数 1 秒，接着第二次调用 `setCounter`，然后 `counter` 又会变为 2，如此往复。

#### Event handling 事件处理

事件处理程序，是指（被注册为）在特定事件发生时进行调用。例如用户点击按钮产生的 `点击 Click` 事件。

```jsx
const App = () =>
{
    const [ counter, setCounter ] = useState(0)

    const handleClick = () =>
    {
        setCounter(counter + 1)
    }

    return (
        <>
            <div>Like {counter}</div>
            <button onClick={ handleClick }>
                赞
            </button>
        </>
    )
}
```

按钮的 `onClick` 调用了 `handleClick` 函数，每次点击 `赞` 按钮都会调用 `handleClick` 函数，也就是 `Click` 事件都会 `setCounter` 将 `counter` 的值 +1。

也可以直接在 onClick 属性的值直接定义事件处理函数：

```jsx
<button onClick={() => setCounter(counter +1)}>
    赞
</button>
```

##### Event handler is a function 事件处理是一个函数

事件处理是一个函数，也就意味着我们不能将事件处理程序作出以下的定义：

- 定义为一个字符串，如 `<button onClick="字符串...">按钮</button>`
- 计算结果，如 `<button onClick="counter + 1">👍</button>`
- 赋值，如 `<button onClick="counter = 0">重置</button>`

下述情况也不可以：

```jsx
<button onClick={console.log('按钮被点击')}按钮</button>
```

当这个组件渲染出来时，`按钮被点击` 会在 console 控制台被打印出来，但点击按钮后不会发生任何事情，`console.log` 函数调用只在渲染组件时被执行。

这里的问题是，我们的事件处理被定义为 function call，这意味着事件处理程序实际上被分配了函数返回的值，而 console.log 的返回值是undefined。


要注意的是，此处不能简略的写成如下：

```jsx
<button onClick={setCounter(counter +1)}>
    赞
</button>
```

当我们点击后，应用会奔溃并报错 `Error: Too many re-renders. .....`。

这是因为事件处理器被定义成了一个函数调用，在很多情况下这是可行的，但是在特殊情况下就不行了。

这样的情况是因为，页面第一次渲染的时候，它执行的函数调用 `setCounter(0+1)`，并将组件状态的值改成了1。这就导致组件重新渲染，重新渲染又会执行 `setCounter` 函数，导致组件状态发生变化，然后重新渲染，这样就进入了一个无限循环，导致 `React` 报错。

要使 `setCounter(counter +1)` 在事件处理程序中正常被调用，可以使用钩子函数，将组件改成 `<button onClick={() => setCounter(counter +1)}>赞</button>`。但在按钮中单独定义时间处理函数不是一个好方法。

所以，最好的方式还是将事件处理程序分离成单独的函数，就跟最初那样定义一个 `handleClick` 函数并在组件内调用。




#### Passing state to child components 将状态传递给子组件

> Often, several components need to reflect the same changing data. We recommend lifting the shared state up to their closest common ancestor.

[状态提升](https://zh-hans.reactjs.org/docs/lifting-state-up.html) 是编写可复用、跨应用、跨项目的 React 组件的最佳方式。React 的数据流是自上而下的。

下面的例子中，应用的状态是放在 App 组件中的，并通过 props 将其传递给 App 中的子组件，例中为 `Display`  `Button` 组件。

```jsx
// 定义可复用的组件
const Display = (props) =>
{
    return (
        <div>Like {props.counter}</div>
    )
}

const Button = (props) =>
{
    return (
        <button onClick={props.handleClick}>
            {props.text}
        </button> 
    )
}

const App = () =>
{
    const [ counter, setCounter ] = useState(0)

    const increaseByOne = () => setCounter(counter + 1)
    const decreaseByOne = () => setCounter(counter - 1)
    const setToZero = () => setCounter(0)

    return (
        <>
            <Display counter={counter} />
            <Button
                handleClick={increaseByOne}
                text='👍'
            />
            <Button
                handleClick={setToZero}
                text='归零'
            />
            <Button
                handleClick={decreaseByOne}
                text='👎'
            />
        </>
    )
}
```


#### Changes in state cause rerendering 状态的改变导致重新渲染

上例 App 在执行时的流程如下

- 应用启动，执行 App 中的代码 > useState hook 创建了 `counter` 的初始值。
- 渲染 `Display` `Button` 组件，页面显示 `counter` 的初始值 `0` 以及将 `increaseByOne` `setToZero` `decreaseByOne` 分别与3个 `Button` 组件的 `onClick` 事件绑定。
- 当单击一个按钮时，事件处理程序使用 `setCounter` 函数更改 App 组件状态。
- 调用一个改变状态的函数使组件重新渲染。
    - 点击 `👍` 按钮，将 `counter` 值改为 `1` 并重新渲染 App 组件，同时子组件也会重新渲染。
    - `Display` 接收到 `counter` 的值，并显示 `1`。
    - `Button` 接收到事件处理程序并与之绑定

#### Refactoring the components 重构组件

将上例组件进行重构，简化代码。

重构 `Display` 组件

```jsx
// 原先定义的组件
const Display = (props) =>
{
    return (
        <div>{props.counter}</div>
    )
}

// 重构
const Display = ({ counter }) => <div>{counter}</div>
```

重构 `Button` 组件

```jsx
// 原先定义的组件
const Button = (props) =>
{
    return (
        <button onClick={props.handleClick}>
            {props.text}
        </button> 
    )
}

// 重构
const Button = ({ handleClick, text}) => <button onClick={handleClick}>{text}</button>
```

### 深入 React 应用调试

#### Complext state 复杂状态

大多数情况，复杂的状态**最简单和最好的方法**是多次使用 `useState` 函数来创建单独的状态。

关于使用单个和多个变量的情况，可以查阅官方FAQ：[我应该使用单个还是多个 state 变量？](https://zh-hans.reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables)

下面是使用复杂状态的案例

```jsx
const App = () =>
{
    const [clicks, setClicks] = useState(
    {
        left: 0,
        right: 0,
    })

    const handleLeftClick = () =>
    {
        const newClicks = 
        {
            left: clicks.left + 1,
            right: clicks.right,
        }
        setClicks(newClicks)
    }

    const handleRightClick = () =>
    {
        const newClicks = 
        {
            left: clicks.left,
            right: clicks.right + 1,
        }
        setClicks(newClicks)
    }

    return (
        <>
            {clicks.left}
            <button onClick={handleLeftClick}>Left</button>
            <button onClick={handleRightClick}>Right</button>
            {clicks.right}
        </>
    )
}
```

可以看到，点击 Left 按钮时，right 的值是不变的，同样的，点击 Right 按钮时， left 的值是不变的。此处使用展开语法来定义新的状态对象，使代码更简洁。

```jsx
const handleLeftClick = () =>
{
    const newClicks =
    {
        ...clicks,
        left: clicks.left + 1,
    }
    setClicks(newClicks)
}

const handleRightClick = () =>
{
    const newClicks =
    {
        ...clicks,
        right: clicks.right + 1,
    }
    setClicks(newClicks)
}
```

将对象分配给事件处理中的变量是没有必要的，上述代码可以简写成如下
```jsx
const handleLeftClick = () => setClicks({ ...clicks, left: clicks.left + 1 })

const handleRightClick = () => setClicks({ ...clicks, right: clicks.right + 1 })
```

以下代码违反了 React 中状态不可以直接修改的原则，这会导致很多副作用，应杜绝这样的方式。

```jsx
const handleLeftClick = () =>
{
    clicks.left++
    setClicks(clicks)
}
```

#### Handling arrays 处理数组

下例增加了点击历史记录的功能，需要添加一个新的状态。使用 `useState([])` 初始化该数组。

使用 `concat` 方法向数组中添加元素，该方法不改变现有数组，而是返回数组的新副本，并将元素添加到数组中。

虽然 `push` 方法也可以向数组添加元素，但因 React 组件的状态不能直接更改，所以不推荐这样使用。

下例使用了 `join` 方法，该数组将所有项目连接到一个字符串中，由作为函数参数传递的字符串「例中为一个空格」分隔。

```jsx
const App = () =>
{
    const [left, setLeft] = useState(0)
    const [right, setRight] = useState(0)
    const [allClicks, setAll] = useState([])

    const handleLeftClick = () =>
    {
        setAll(allClicks.concat('点击了Left'))
        setLeft(left + 1)
    }

    const handleRightClick = () =>
    {
        setAll(allClicks.concat('点击了Right'))
        setRight(right + 1)
    }

    return (
        <div>
            {left}
            <button onClick={handleLeftClick}>left</button>
            <button onClick={handleRightClick}>right</button>
            {right}
            <p>{allClicks.join(' ')}</p>
        </div>
    )
}
```

#### Conditional rendering 条件渲染

下例将操作历史记录交由 `History` 组件来处理，使用 `if` 判断渲染历史记录。

```jsx
const History = (props) =>
{
    if (props.allClicks.length === 0)
    {
        return (
            <div>
                暂无操作历史
            </div>
        )
    }

    return (
        <div>
            操作历史：{props.allClicks.join(' ')}
        </div>
    )
}

const App = () => 
{
    // ... 同上例

    return (
        <div>
            {left}
            <button onClick={handleLeftClick}>left</button>
            <button onClick={handleRightClick}>right</button>
            {right}
            <History allClicks={allClicks} />
        </div>
    )
}
```

可以看出，`allClicks` 为空数组的时候，`History` 组件将会渲染 *暂无操作历史* 的内容，其他情况下，将会渲染操作历史。

#### Debugging React applications 调试 React 应用

- web 开发第一原则
    - 始终打开浏览器开发控制台
    - 尤其是 Console 选项卡应该始终处于打开状态，除非有特定的原因需要查看另一个选项卡。
- 保持代码和网页同时打开
- 编译失败时，应立即找到并修复问题
- 不能正常工作时，可以使用 `console.log` 来查看。`console.log('props value is', props)` 与 `props` 不要使用 + 号，因为这样会打印出 `props value is [object object]`。
- 也可在代码中写入 `debugger`，chrome 开发者控制台会在执行到该代码时暂停。
- `debugger` 还可以使用在 Source 选项卡右侧找到空间一行一行地执行代码。
- 通过在 Sources 选项卡中添加断点，您还可以在不使用 `debugger` 命令的情况下访问调试器。
- 在 Chrome 安装 React developer tools 扩展，可以用来检查不同的React 元素，以及它的属性、状态。

#### Rules of Hooks Hook的规则

- 不能从循环、条件表达式或任何不适定义组件的函数的地方调用 `useState` 「同样的还有 `useEffect` 函数」。*这样做是为了确保Hook总是以相同的顺序调用，如果不是这样，应用的行为就会不规则。*

```jsx
const App = () =>
{
    // 这里使用时没有问题的
    const [age, setAge] = useState(0)

    if ( age > 10 )
    {
        // 这里使用时没有效果的
        const [range, setRange] = useState('少年')
    }

    for ( let i = 0; i < age; i++ )
    {
        // 这里依然无效
        const [nothing, setNothing] = useState(false)
    }

    const notGood = () =>
    {
        // 这里也不行
        const [x, setX] = useState(null)
    }

    return (
        // ...
    )
}

```

#### Function that returns a function 返回函数的函数

定义事件处理程序的另外一种方法是使用返回函数的函数。因为函数返回的是函数，所以事件处理程序是可以使用这个函数。

使用返回函数的函数，将可以使用参数自定义的通用函数添加到不同组件的事件处理程序中。

```jsx
const hello = (who) =>
{
    const handler = () => console.log('hello', who)

    return handler
}
```

当 React 渲染时， `<button onClick={hello('world')}>button</button>` 本质上会被转换成 `<button onClick={()=> console.log('hello', 'world')}>button</button>`。

这样我们就可以在不同组件中调用了。

```jsx
<button onClick={hello('imagic')}>imagic</button>
<button onClick={hello('w')}>w</button>
```

我们将 hello 函数重构一下

```jsx
// 省去 handler
const hello = (who) =>
{
    return () =>
    {
        console.log('hello', who)
    }
}

// 再来
const hello = (who) =>
    () =>
    {
        console.log('hello', who)
    }

// 将上放再紧凑一些

const hello = (who) => () =>
{
    console.log('hello', who)
}

```

也可以这样使用

```jsx
const hello = (who) =>
{
    console.log('hello', who)
}
```

相应的，事件处理程序需要按照如下设置，这么做的原因在 *事件处理是一个函数* 章节说过了。

```
<button onClick={() => hello('imagic')}>
    imagic
</button>
```

{{< admonition tip "更加了解 React" false >}}
React [官方文档](https://zh-hans.reactjs.org/docs/hello-world.html)

[Egghead.io 课程](https://egghead.io/)
{{< /admonition >}}

[^react]: [React 官方中文文档](https://zh-hans.reactjs.org/)
[^1]: https://fullstackopen.com/zh/part1/java_script#classes
