# React 入门


# React

react是用于构建web和原生交互界面的库

![1.png](images%2F1.png)
创建
```
npx create-react-app react-basic
```

## JSX
概念：JSX是javascript和XML(HTML)的缩写，表示在JS代码中编写HTML模版结构，它是React中编写模版的方式。
![2.png](images%2F2.png)

优势
1、HTML的声明式的模版语法
2、JS的可编程能力
### 本质<!-- {"fold":true} -->
JSX并不是标准的JS语法，它是JS的语法扩展，浏览器本身不能识别，需要通过解析工具做解析后才能在浏览器运行
[Babel · Babel](https://babeljs.io/)
![3.png](images%2F3.png)


### 基础
- 使用引号传递字符串
- 使用javascript变量
- 函数调用和方法调用
- 使用javascript对象
```
//项目的根组件
// APP->index.js->public/index.html(root)
const count = 100

function getName() {
    return 'hello'
}

function App() {
    return (
        <div className="App">
            this is app
            {/*  使用引号传递字符串  */}
            {'this is message'}
            {/*    识别js变量*/}
            {count}
            {/*    函数调用*/}
            {getName()}
            {/*    方法调用*/}
            {Date.now()}
            {/*    js对象*/}
            <div style={{color:'red'}}>
                this is div
            </div>
        </div>
    );
}

export default App;


```
#### 列表
```
//项目的根组件
// APP->index.js->public/index.html(root)
const count = 100

function getName() {
    return 'hello'
}

const list = [
    {id: 1001, name: 'vue'},
    {id: 1002, name: 'react'},
    {id: 1003, name: 'angular'}
]

function App() {
    return (
        <div className="App">
            {/*注意:需要加一个独一无二的key，key的作用：React框架内部使用，用来提升列表更新性能*/}
            <ul>
                {list.map(item => <li key={item.id}>{item.name}</li>)}
            </ul>
        </div>
    );
}

export default App;



```
#### 条件渲染
```
//项目的根组件
// APP->index.js->public/index.html(root)
const count = 100

function getName() {
    return 'hello'
}

const isLogin = true
// const list = [
//     {id: 1001, name: 'vue'},
//     {id: 1002, name: 'react'},
//     {id: 1003, name: 'angular'}
// ]

function App() {
    return (
        <div className="App">
            {/*逻辑与*/}
            {isLogin && <div>{getName()}</div>}
            {isLogin ? <span>登录</span>: <span>未登录</span>}
        </div>
    );
}

export default App;


```
### 事件
语法：on+事件名称={事件处理程序}
```
//项目的根组件
// APP->index.js->public/index.html(root)
function App() {
    const handleClick = () => {
        console.log('click')
    }
    return (
        <div className="App">
            <button onClick={handleClick}>click me</button>
        </div>
    );
}

export default App;


```
==🔵同时传递事件对象和自定义参数==
```
//项目的根组件
// APP->index.js->public/index.html(root)
function App() {
    const handleClick = (e, name) => {
        console.log("button 按钮点击了", name, e)
    }
    return (
        <div className="App">
            <button onClick={(e) => handleClick(e, 'jack')}>click me</button>
        </div>
    );
}

export default App;

```
### 组件
概念：一个组件就是用户界面的一部分，它可以有自己的逻辑和外观，组件之间可以互相嵌套，也可以复用多次。
![4.png](images%2F4.png)
组件开发可以让开发者像搭积木一样构建一个完整的庞大应用
在react中，一个组件就是首字母大写的函数，内部存放了组件的逻辑和视图UI，渲染组件只需要把组件当成标签书写即可。
![5.png](images%2F5.png)
#### 通信
![6.png](images%2F6.png)
##### 父子通信
###### 父传子
![7.png](images%2F7.png)
```
import './App.scss'

/*父传子
* 1、父组件传递数据 子组件标签身上绑定属性
* 2、子组件接收数据，props.属性名
*
* */
function Son(props) {
    // props：对象里面包含了父组件传递过来的所有的属性，prpos也可以定义成别的，一般定义成props
    return (
        <div className="son">
            <p>{props.name}</p>
        </div>
    )
}

const App = () => {
    const name = "this is app name"
    return (
        <div className="app">
            <Son name={name}></Son>
        </div>
    )
}

export default App

```
###### 子传父
![8.png](images%2F8.png)
```
import './App.scss'
import {useState} from "react";

/*父传子
* 1、父组件传递数据 子组件标签身上绑定属性
* 2、子组件接收数据，props.属性名
*
* */
function Son(props) {
    // props：对象里面包含了父组件传递过来的所有的属性，prpos也可以定义成别的，一般定义成props
    props.sendMsg('this is son msg')
    return (
        <div className="son">
            <p>{props.name}</p>
        </div>
    )
}

/** 子传父
 * 1、父组件定义一个函数，用于接收子组件传递过来的数据
 * 2、子组件调用父组件传递过来的函数，并将数据作为参数传递过去
 * 3、父组件通过props接收子组件传递过来的数据
 *
 * @returns {JSX.Element}
 * @constructor
 */

const App = () => {
    const [msg, setMsg] = useState('')
    const sendMsg = (msg) => {
        setMsg(msg)
    }
    return (
        <div className="app">
            {msg}
            <Son sendMsg={sendMsg}>
            </Son>
        </div>
    )
}

export default App

```
###### 兄弟通信
![9.png](images%2F9.png)
```
import './App.scss'
import {useState} from "react";

/*父传子
* 1、父组件传递数据 子组件标签身上绑定属性
* 2、子组件接收数据，props.属性名
*
* */
function A(props) {
    // props：对象里面包含了父组件传递过来的所有的属性，prpos也可以定义成别的，一般定义成props
    const msg = "this is A name"
    return (
        <div className="son">
            <p>{msg}</p>
            <button onClick={() => props.sendMsg(msg)}>send</button>
        </div>
    )
}

function B(props) {
    return (
        <div className="son">
            <p>{props.msg}</p>
        </div>
    )
}

/** 子传父
 * 1、父组件定义一个函数，用于接收子组件传递过来的数据
 * 2、子组件调用父组件传递过来的函数，并将数据作为参数传递过去
 * 3、父组件通过props接收子组件传递过来的数据
 *
 * @returns {JSX.Element}
 * @constructor
 */

const App = () => {
    const [msg, setMsg] = useState('')
    const sendMsg = (msg) => {
        setMsg(msg)
    }
    return (
        <div className="app">
            <A sendMsg={sendMsg}>
            </A>
            <B msg={msg}></B>
        </div>
    )
}

export default App

```
###### props
说明
- 可以传递任意数据
- props是只读对象
    - 子组件只能读取props中的数据，不能直接进行修改，父组件的数据只能有父组件来修改
###### Prop children
![10.png](images%2F10.png)
```
import './App.scss'

/*父传子
* 1、父组件传递数据 子组件标签身上绑定属性
* 2、子组件接收数据，props.属性名
*
* */
function Son(props) {
    // props：对象里面包含了父组件传递过来的所有的属性，prpos也可以定义成别的，一般定义成props
    return (
        <div className="son">
            <p>{props.name}</p>
        </div>
    )
}

const App = () => {
    const name = "this is app name"
    return (
        <div className="app">
            <Son name={name}>
                <span>this is span</span>
            </Son>
        </div>
    )
}

export default App

```
##### 兄弟通信
![11.png](images%2F11.png)
```
import './App.scss'
import {useState} from "react";

/*父传子
* 1、父组件传递数据 子组件标签身上绑定属性
* 2、子组件接收数据，props.属性名
*
* */
function A(props) {
    // props：对象里面包含了父组件传递过来的所有的属性，prpos也可以定义成别的，一般定义成props
    const msg = "this is A name"
    return (
        <div className="son">
            <p>{msg}</p>
            <button onClick={() => props.sendMsg(msg)}>send</button>
        </div>
    )
}

function B(props) {
    return (
        <div className="son">
            <p>{props.msg}</p>
        </div>
    )
}

/** 子传父
 * 1、父组件定义一个函数，用于接收子组件传递过来的数据
 * 2、子组件调用父组件传递过来的函数，并将数据作为参数传递过去
 * 3、父组件通过props接收子组件传递过来的数据
 *
 * @returns {JSX.Element}
 * @constructor
 */

const App = () => {
    const [msg, setMsg] = useState('')
    const sendMsg = (msg) => {
        setMsg(msg)
    }
    return (
        <div className="app">
            <A sendMsg={sendMsg}>
            </A>
            <B msg={msg}></B>
        </div>
    )
}

export default App

```
##### 跨层通信（context）
![12.png](images%2F12.png)
```
import './App.scss'
import {createContext, useContext} from "react";
// 1、createContext 创建上下文
const MsgContext = createContext()
function A() {
    return (
        <div>
            this is A compent
            <B></B>
        </div>
    )
}

function B() {
    // 在底层组件，通过useContext钩子函数使用数据
    const msg =useContext(MsgContext)
    return (
        <div>
            this is B compent
            {msg}
        </div>
    )
}

const App = () => {
    const msg = 'this is app msg'
    return (
        <div className="app">
            {/*在顶层组件，通过Provider组件提供数据*/}
            <MsgContext.Provider value={msg}>
                <A></A>
            </MsgContext.Provider>
        </div>
    )
}

export default App

```
### 样式
```
// useState实现一个计数器按钮
import './index.css'

const style = {
    color: 'red'
}

function App() {
    return (
        <div>
            {/*第一种写法，不推荐*/}
            <span style={{color: 'red'}}>this is span one</span>
            <br/>
            <span style={style}>this is span one</span>
            <br/>
            {/*第三种写法*/}
            <span className="foo">this is span</span>
        </div>
    );
}

export default App;


```
### hooks
#### useState
useState是一个React Hook（函数）,它允许我们向组件天添加一个状态变量，从而控制影响组件的渲染结果。
```
// useState实现一个计数器按钮
import {useState} from "react";

function App() {
    // count 状态变量 setCount修改状态变量的方法
    const [count, setCount] = useState(0)
    // 点击事件回调
    const handleClick = () => {
        setCount(count + 1)
    }
    return (
        <div>
            <button onClick={handleClick}>{count}</button>
        </div>
    );
}

export default App;

```
==注意：状态不可变==
在React中，状态被认为是只读的，我们应该始终替换它而不是修改它，直接修改状态不会引发视图更新。
对象类型
规则：对于对象类型的状态变量，应该始终传给set方法一个全新的对象来进行修改。
```
// useState实现一个计数器按钮
import {useState} from "react";

function App() {
    // count 状态变量 setCount修改状态变量的方法
    const [count, setCount] = useState(0)
    // 点击事件回调
    const handleClick = () => {
        setCount(count + 1)
    }
    const [form, setForm] = useState({name: "jack", age: "18"})
    const handleForm = () => {
        //其中三个点表示ES6的扩展运算符，简化原有的对象拷贝操作
        setForm({
            ...form,
            name: "tom",
            age: "19"
        })
    }
    return (
        <div>
            <button onClick={handleClick}>{count}</button>
            <button onClick={handleForm}>{form.name}-{form.age}</button>
        </div>
    );
}

export default App;


```
##### 传递参数
useState本身是泛型参数，可以传入具体的自定义参数类型
![13.png](images%2F13.png)
```
import {useState} from 'react'

type User = {
    name: string
    age: number
}

function App() {
    // const [user, setUser] = useState<User>({
    //     name: 'john',
    //     age: 20
    // })
    // const [user, setUser] = useState<User>(() => {
    //     return {
    //         name: 'john',
    //         age: 20
    //     }
    // })
    const [user, setUser] = useState<User | null>(null)
    //     return {
    const changeHandler = () => {
        setUser({
            name: 'john',
            age: 30
        })
    }
    return (
        <>
		{/*user?为了类型安全 可选链做类型守卫*/}
		{/*只有user不为null（不为空）的时候才进行运算*/}

            this is app,{user?.name},年龄为{user?.age}
            <button onClick={changeHandler}>点击</button>
        </>
    )
}

export default App




```
#### useEffect
useEffect是一个React Hook函数，用于在React组件中创建不是由事件引起而是由渲染本身引起的操作，比如发送Ajax请求，更改DOM等
```
import './App.scss'
import {useEffect, useState} from "react";

const url = "http://geek.itheima.net/v1_0/channels"
const App = () => {
    const [list, setList] = useState([])
    useEffect(() => {
        // 额外操作，获取频道列表
        // 异步
        async function getList() {
            // 等待结果
            const res = await fetch(url)
            const jsonRes = await res.json()
            setList(jsonRes.data.channels)
        }

        getList()
    }, [])
    return (
        <div className="app">
            this is app
            <ul>
                {list.map(item => <li key={item.id}>{item.name}</li>)}
            </ul>
        </div>
    )
}

export default App

```
##### 依赖项
![14.png](images%2F14.png)
##### 清除副作用
在useEffect中编写的由渲染本身引起的对组件外的操作，社区也经常叫做副作用操作。比如在useEffect中开启一个定时器，我们想在组件卸载的时候把定时器清理掉，这个过程急事清理副作用。
```
import './App.scss'
import {useEffect, useState} from "react";

const Son = () => {
    useEffect(() => {
        const timer = setInterval(() => {
            console.log('定时器执行中')
        }, 1000)
        return () => {
            //组件卸载时候，会自动执行return
            clearInterval(timer)
            console.log('定时器被销毁')
        }
    })
    return (
        <div>
            this is son
        </div>
    )
}

const App = () => {
    const [show, setShow] = useState(true)
    return (
        <div className="app">
            {
                show && <Son></Son>
            }
            <button onClick={() => setShow(false)}>卸载Son组件</button>
        </div>
    )
}

export default App

```

#### 自定义Hook
- 只能在组件中或者其它自定义的Hook函数中调用
- 只能在组件顶层调用，不能嵌套在if for 中
```
import './App.scss'
import {useState} from "react";

// 封装自定义Hook
function useToggle() {
    //  可复用的逻辑代码
    const [value, setValue] = useState(true)
    const toggle = () => {
        setValue(!value)
    }
    // 那些状态和函数需要再其他函数中使用，return
    return {
        value,
        toggle
    }
}
// 封装自定义hook的通用思路
// 1. 定义一个函数，函数名就是自定义hook的名字
// 2. 函数内部定义状态和函数，返回需要暴露出去的状态和函数
// 3. 在函数内部使用状态和函数，暴露出去的状态和函数
const App = () => {
    const {value, toggle} = useToggle()
    return (
        <div className="app">
            {
                value && <div>this is div</div>
            }
            <button onClick={toggle}>toggole</button>
        </div>
    )
}

export default App

```
#### useMemo
作用：在组件每次重新渲染的时候缓存计算结果
![15.png](images%2F15.png)
#### useCallback
作用：在组件多次重新渲染的时候缓存函数
说明：使用useCallback包裹函数之后，函数可以保证在App重新渲染的时候保持引用稳定
```
// useState实现一个计数器按钮
import './index.css'
import {memo, useCallback, useMemo, useState} from "react";

// React.memo props比较机制
// 1、传递一个简单类型prop prop变化时候组件重新渲染
// 2、传递一个引用类型的prop 比较的是最新值和旧值的引用是否相等， 当父组件的函数重新执行时，实际上形成的是新的数组引用
// 3、保证引用稳定 ->useMemo 在组件的渲染过程缓存一个值
function Son({count}) {
    console.log("子组件重新渲染了")
    return <div>this is son {count}</div>
}

const MemoSon = memo(Son)

function App() {
    const [count, setCount] = useState(0)
    const changHandler = useCallback((value) => console.log(value), [])
    return (
        <div>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <MemoSon count={changHandler}></MemoSon>
        </div>
    );
}

export default App;


```
#### forwardRef
场景：通过ref获取到子组件内部的input元素属性
```
import {forwardRef, useRef} from "react";

// function Son() {
//     return <input type={'text'}/>
// }

const Son = forwardRef((props, ref) => {
    return <input type={'text'} ref={ref}/>
})

function App() {
    const sonRef = useRef(null)
    const showRef = () => {
        sonRef.current.focus()
    }
    return (
        <>
            <Son ref={sonRef}/>
            <button onClick={showRef}>focus</button>
        </>
    )
}

export default App

```
#### useInperativeHandler
场景：父组件通过ref调用子组件内部的focus方法实现聚焦。
```
import {forwardRef, useImperativeHandle, useRef} from "react";

// function Son() {
//     return <input type={'text'}/>
// }

const Son = forwardRef((props, ref) => {
    const inputRef = useRef(null)
    const focusHandler = () => {
        inputRef.current.focus()
    }
    useImperativeHandle(ref, () => {
        return {
            focusHandler
        }
    })
    return <input type={'text'} ref={inputRef}/>
})

// 父组件
function App() {
    const sonRef = useRef(null)
    const focusHandler = () => {
        sonRef.current.focusHandler()
    }
    return (
        <>
            <Son ref={sonRef}/>
            <button onClick={focusHandler}>focus</button>
        </>
    )
}

export default App

```
#### useRef
获取dom场景，可以直接把要获取的dom元素的类型当成泛型参数传递个useRef,可以推导出.current属性的类型
#### props
##### props和Ts一起使用-基础类型
```
import {useState} from 'react'

// type Props = {
//     className: string
// }

interface Props {
    className: string
    title?: string
}

function Button(props: Props) {
    const {className} = props
    return (
        <button>
            Click me
        </button>
    )
}

function App() {

    return (
        <>
            <Button className={'test'} title=""></Button>
        </>
    )
}

export default App


```
##### props与Ts-为children添加类型
Children是一个比较特殊的prop，支持多种类型数据传入，需要通过一个内置的ReactNode类型来做注解
```
import {useState} from 'react'
import react from "@vitejs/plugin-react";

// type Props = {
//     className: string
// }

interface Props {
    className: string
    children: react.ReactNode
}

function Button(props: Props) {
    const {className, children} = props
    return (
        <button className={className}>
            {children}
        </button>
    )
}

function App() {

    return (
        <>
            <Button className={'test'}>clike me !</Button>
            <Button className={'test'}><span>this is span</span></Button>
        </>
    )
}

export default App


```
##### props与ts-为事件prop添加类型
组件经常执行类型为函数的prop实现子传父，这类prop重点在于函数参数类型的注解
```
import {useState} from 'react'
import react from "@vitejs/plugin-react";

// type Props = {
//     className: string
// }
type Props = {
    onGetMsg: (msg: string) => void
}

function Son(props: Props) {
    const {onGetMsg} = props
    return <button onClick={() => onGetMsg?.(100)}></button>
}

function App() {
    const getMsgHandler = (msg: string) => {
        console.log(msg)
    }
    return (
        <>
            <Son onGetMsg={(msg) => console.log(msg)}></Son>
            <Son onGetMsg={getMsgHandler}></Son>
        </>
    )
}

export default App


```
### DOM
```
// useState实现一个计数器按钮
import './index.css'
import {useRef} from "react";

// React中获取DOM
// 1.声明useRef,绑定到dom标签身上
// 2、dom可用时，ref.current获取dom（渲染完毕之后dom生成之后才可用）

function App() {
    const inputRef = useRef(null)
    const showDom = () => {
        console.dir(inputRef.current)
    }
    return (
        <div>
            <input type="text" ref={inputRef}/>
            <button onClick={showDom}>获取dom</button>
        </div>
    );
}

export default App;

```
### Memo
```
// useState实现一个计数器按钮
import './index.css'
import {memo, useMemo, useState} from "react";

// React.memo props比较机制
// 1、传递一个简单类型prop prop变化时候组件重新渲染
// 2、传递一个引用类型的prop 比较的是最新值和旧值的引用是否相等， 当父组件的函数重新执行时，实际上形成的是新的数组引用
// 3、保证引用稳定 ->useMemo 在组件的渲染过程缓存一个值
function Son({count}) {
    return <div>this is son {count}</div>
}

const MemoSon = memo(Son)

function App() {
    const [count, setCount] = useState(0)
    const num = 100
    const list = useMemo(() => {
        return [1, 2, 3, 4, 5]
    }, [])
    return (
        <div>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <MemoSon count={list}></MemoSon>
        </div>
    );
}

export default App;



```
## Redux
Redux是React最常用的集中管理工具，类型vuex，可以独立于框架运行。
作用：通过集中管理的方式管理应用状态
### 配套工具
在React使用Redux，官方要求安装两个插件 -Redux Toolkit和react-redux
- Redux Toolkit（RTK）-官方推荐的编写Redux逻辑的方式，是一套工具的集合集，简化书写方式
- react-redux 用来链接redux和React组件的中间件
  安装
```
npm i @reduxjs/toolkit react-redux
```
### 修改store数据
React组件中修改store中的数据需要借助另外一个hook函数-useDispatch，它的作用是生成条件action对象的dispatch函数。
### 异步操作
```
import {createSlice} from '@reduxjs/toolkit'
import axios from "axios"

const url = "http://geek.itheima.net/v1_0/channels"
// 创建store
const channelStore = createSlice({
    name: 'channel',
    initialState: {
        channelList: []
    },
    reducers: {
        setChannels(state, action) {
            state.channelList = action.payload
        }
    }
})
const {setChannels} = channelStore.actions
// 创建方法
const fetchChannelList = async (dispatch) => {
    const res = await axios.get(url)
    dispatch(setChannels(res.data.data.channels))
}
export {fetchChannelList}
const channelReducer = channelStore.reducer
export default channelReducer

```
### Demo
创建状态类
```
import {createSlice} from "@reduxjs/toolkit"

const counterSlice = createSlice({
    name: "counter",
    //初始化
    initialState: {
        count: 0
    },
    // 修改状态方法
    reducers: {
        increment(state) {
            state.count++
        },
        decrement(state) {
            state.count--
        },
        addToNum(state, action) {
            state.count = action.payload
        }
    }
})
// 结构出来actionCreater函数
const {increment, decrement, addToNum} = counterSlice.actions
// 获取 reducer
const countReducer = counterSlice.reducer
// 按需导出
export {increment, decrement, addToNum}
// 导出
export default countReducer

```
声明store
```
const store = configureStore({
    reducer: {
        counter: countReducer
    }
})

export default store

```
```
import './App.css';
import {useDispatch, useSelector} from "react-redux";
//导入actionCreater
import {increment, decrement, addToNum} from './store/modules/counterStore'

function App() {
    const {count} = useSelector(state => state.counter)
    const dispatch = useDispatch()
    return (
        <div className="App">
            <button onClick={() => dispatch(decrement())}>-</button>
            {count}
            <button onClick={() => dispatch(increment())}>+</button>
            <button onClick={() => dispatch(addToNum(10))}>add to 10</button>
            <button onClick={() => dispatch(addToNum(20))}>add to 20</button>
        </div>
    );
}

export default App;

```
### 调试
浏览器安装devTools
![16.png](images%2F16.png)
## 路由
前端路由：一个路径path对应一个组件的component当我们在浏览器中访问一个path的时候，path对应的组件会在页面中渲染。
```
npm i react-router-dom
```
demo
```
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import {createBrowserRouter, RouterProvider} from "react-router-dom"
// 创建路由对应关系
const router = createBrowserRouter(
    [
        {
            path: "/login",
            element: <div>我是登录面</div>
        },
        {
            path: "/article",
            element: <div>我是文章页</div>
        }
    ]
)

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
    <React.StrictMode>
       <RouterProvider router={router}></RouterProvider>
    </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```
#### 路由导航
路由系统中的多个路由之间需要进行路由跳转，并且在跳转的同时有可能需要传递参数进行通信。
##### 声明式导航
![17.png](images%2F17.png)
##### 编程式导航
![18.png](images%2F18.png)
##### 导航传参
![19.png](images%2F19.png)

##### demo
```
import {Link, useNavigate} from 'react-router-dom'

const Login = () => {
    const navigate = useNavigate()
    return (
        <div>我是登录页
            {/*声明式写法*/}
            <Link to={"/article"}>跳转到文章页</Link>
            {/*命令式写法*/}
            <button onClick={() => navigate('/article')}>跳转到文章页</button>
        </div>
    )
}
export default Login

```
#### 嵌套路由
==🔴注意：Outlet是渲染二级路由，如果屏蔽无法渲染出跳转的二级路由==
```
import Article from "../page/Article";
import Login from "../page/Login";
import {createBrowserRouter, RouterProvider} from "react-router-dom"
import Layout from "../page/Layout";
import Board from "../page/Board";
import About from "../page/About";

// 创建路由对应关系
const router = createBrowserRouter(
    [
        {
            path: "/",
            element: <Layout></Layout>,
            children: [
                {
                    // 设置为默认路由，去掉path，设置index为true
                    element: <Board></Board>,
                    index: true
                },
                {
                    path: "about",
                    element: <About></About>
                },
            ]
        },
        {
            path: "/login",
            element: <Login></Login>
        },
        {
            path: "/article/:id",
            element: <Article></Article>
        }
    ]
)
export default router
```
#### 路由模式
常见路由模式有两种：history模式和hash模式。

| 路由模式    | url表现       | 底层原理                  | 是否需要后端支持    |
|---------|-------------|-----------------------|-------------|
| history | url/login   | history对象+pushState事件 | 需要（nginx配置） |
| hash    | url/#/login | 监听hashChange事件        | 不需要         |

## ~~Class 类组件~~
目前新版本已废弃，只需要了解
类组件就是通过js中的类来组织组件的代码
```
import {Component, forwardRef, useImperativeHandle, useRef} from "react";

class Counter extends Component {
    // 1、状态变量
    state = {
        count: 0
    }

    // 组件渲染完毕后执行一次，发送网络请求
    componentDidMount() {
        console.log('Counter组件挂载完毕')
        this.timer = setInterval(() => {
            console.log("定时器运行中")
        }, 1000)
    }

    // 组件卸载的时候执行，副作用清楚工作，清楚定时器，清楚事件绑定
    componentWillUnmount() {
        console.log('Counter组件即将卸载')
        clearInterval(this.timer)
    }

    // 定义事件回调修改状态数据
    setCount = () => {
        this.setState({count: this.state.count + 1})
    }

    render() {
        return (
            <button onClick={this.setCount}>{this.state.count}</button>
        )
    }
}

// 父组件
function App() {
    return (
        <>
            <Counter></Counter>
        </>
    )
}

export default App

```
### 生命周期
![20.png](images%2F20.png)
### 组件通信
概念：类组件和Hooks编写的组件在组件通信的思想上完全一致。
## Zustand(替代redux)
地址：[一个小型、快速、可扩展的基本状态管理解决方案](https://zustand-cn.js.org/)
简单的状态管理工具
### 基础使用
```
// zustand
import {create} from 'zustand'
import {useEffect} from "react";

const URL = "http://geek.itheima.net/v1_0/channels"
// 创建store
// 1.函数参数必须返回一个对象，对象内部编写状态数据和方法
// 2.set是用来修改数据的专门方法必须调用他来修改数据
// 语法1：参数是函数，需要用到老数据场景
// 语法2：参数是对象，直接修改数据场景
const useStore = create((set) => {
    return {
        count: 0,
        inc: () => set((state) => ({count: state.count + 1})),
        channelList: [],
        fetchGetList: async () => {
            const res = await fetch(URL)
            const jsonRes = await res.json()
            set(
                {channelList: jsonRes.data.channels}
            )
        }
    }
})

// 父组件
function App() {
    const {count, inc, fetchGetList, channelList} = useStore()
    useEffect(() => {
        fetchGetList()
    }, [fetchGetList])
    return (
        <>
            <button onClick={inc}>{count}</button>
            <ul>
                {channelList.map(item => <li key={item.id}>{item.name}</li>)}
            </ul>
        </>
    )
}

export default App


```
### 切片模式
场景：当单个store比较大的时候，可以采用切片模式进行模式拆分组合。
![](image%2050.png)
```
// zustand
import {create} from 'zustand'
import {useEffect} from "react";

const URL = "http://geek.itheima.net/v1_0/channels"
// 1、拆分子模块 在组合起来
const createCouterStore = (set) => {
    return {
        count: 0,
        inc: () => set((state) => ({count: state.count + 1})),
    }
}
const createChannelStore = (set) => {
    return {
        channelList: [],
        fetchGetList: async () => {
            const res = await fetch(URL)
            const jsonRes = await res.json()
            set(
                {channelList: jsonRes.data.channels}
            )
        }
    }
}
const useStore = create((...a) => {
    return {
        ...createCouterStore(...a),
        ...createChannelStore(...a)
    }
})

// 父组件
function App() {
    const {count, inc, fetchGetList, channelList} = useStore()
    useEffect(() => {
        fetchGetList()
    }, [fetchGetList])
    return (
        <>
            <button onClick={inc}>{count}</button>
            <ul>
                {channelList.map(item => <li key={item.id}>{item.name}</li>)}
            </ul>
        </>
    )
}

export default App

```
## [Vite](https://vitejs.cn/vite5-cn/)
下一代前端开发与构建工具
## 库
### [lodash](https://www.lodashjs.com/)
Lodash 是一个一致性、模块化、高性能的 JavaScript 实用工具库。
> Lodash 遵循 MIT 开源协议发布，并且支持最新的运行环境。 查看各个构件版本的区别并选择一个适合你的版本。
```
npm i lodash
```

### Classnames
classname是一个简单的JS库，可以非常方便的通过条件动态空中class类名的显示
```
npm i classnames --save

```
### [DAY](https://day.js.org/zh-CN/)
Day.js 是一个轻量的处理时间和日期的 JavaScript 库
### [json-server](https://github.com/typicode/json-server?tab=readme-ov-file)
json-server是一个快速模拟json数据的服务
### [Axios](https://www.axios-http.cn/)
Axios 是一个基于 promise 的网络请求库，可以用于浏览器和 node.js
Axios 使用简单,包尺寸小且提供了易于扩展的接口。
### [Antnd-mobile](https://ant-design-mobile.antgroup.com/zh)
阿里移动端组件库
### **[Ant Design](https://ant-design.antgroup.com/index-cn)**
是基于 Ant Design 设计体系的 React UI 组件库，适合企业级中后台产品与前台桌面网站。
### Craco
CRA本身把webpack配置包装到黑盒里面无法修改，需要借助一个插件 -craco
```
npm i -D @craco/craco
```
### [Normalize](https://necolas.github.io/normalize.css/)
样式全局初始化
[￼Echarts](https://echarts.apache.org/zh/index.html)
图表
### [react-quill](https://github.com/zenoamaro/react-quill)
富文本编辑器

## 项目打包
命令
```
npm run build
```
### 本地预览
本地预览是指本地通过静态服务模拟项目
```
npm i -g serve
serve -s build
```
### 打包优化
#### 路由懒加载
```
const Home = lazy(() => import("@/pages/Home"))
const Article = lazy(() => import("@/pages/Article"))
const Publish = lazy(() => import("@/pages/Publish"))
```
#### 体积分析
需要安装插件
```
npm i -d source-map-explorer 
```
#### CDN优化
CDN是一种内容分发网络服务，当用户请求网站内容时，由离用户最近的服务器将缓存的资源内容传递给用户。
那些资源可以放到CDN服务器？
- 体积较大的非业务的JS文件，比如react、react-dom
- 非业务js文件，不需要经常变更的，CDN不用频繁更新缓存
  项目中怎么做？
- 把需要做CDN缓存的文件排除在打包之外（react、react-dom）
- 以CDN的方式重新引入资源
## 常见问题
1、var、const和let的区别
https://www.freecodecamp.org/chinese/news/var-let-and-const-whats-the-difference/
2、别名路径配置
- 路径解析配置（webpack），把@/解析为src/
  3、webpack无法找到@
  需要新增jsconfig.json文件
```
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  }
}


```
4、forEach和map区别
- forEach不返回结果，返回的是一个undefined
- map返回的是一个新数组
  5、<>{children}</>是什么意思
  <>{children}</> 是 React 中的一种语法，称为 Fragment。它用于将多个子元素包裹在一起，而不增加额外的 DOM 节点。children 是一个特殊的属性，包含了传递给组件的所有子元素。使用 Fragment 可以让你在返回多个元素时保持结构的简洁。例如：
```
const MyComponent = ({ children }) => {
  return (
    <>
      <h1>标题</h1>
      {children}
    </>
  );
};

```
在这个例子中，MyComponent 会渲染一个标题和传递给它的任何子元素，而不会在 DOM 中增加额外的包裹元素。


---

> 作者:   
> URL: http://localhost:1313/posts/notes/study/web/react/1-react-basic/  

