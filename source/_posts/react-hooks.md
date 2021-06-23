---
title: React Hooks基础
date: 2021-06-22 17:26:00
tags: react hooks
---

**回顾class组件的问题**

- 大型组件很难拆分和重构，很难测试（即class不易拆分）
- 相同业务逻辑，分散到各个方法中，逻辑混乱
- 复用逻辑变得复杂，如HOC、Render Prop

---

### State Hook

函数组件是一个纯函数，执行完即销毁，无法存储state，需要State Hook，即把state功能”钩“到纯函数中

### Effect Hook

```jsx
import React, { useState, useEffect } from 'react'

function Index() {
	
	// 模拟class组价中 componentDidMount 和 componentDidUpdate
	useEffect(() => {
		console.log('在此发送一个ajax请求')
	})
	
	// 模拟class组件中 componentDidMount 挂载
	useEffect(() => {
		console.log('加载完了')
	}, []) // 第二个参数是 []（不依赖于任何 state）

	// 模拟class组件中 componetDidUpdate 更新
	useEffect(() => {
		console.log('更新了')
	}, [para1, para2]) // 第二个参数是依赖 state

	// 模拟class组件中 componentWillUnmount
	useEffect(() => {
		let timerId = window.setInterval(() => {
			console.log(Date.now())
		}, 1000)
		
		// 返回一个函数
		// 模拟class组件 componetWillUnmount 销毁
		return () => {
			window.clearInterval(timerId)
		}
	}, [])

	return <div>
		<div>Index test</div>
	</div>
}

export default Index
```

### ***所有的定时任务或全局事件绑定，一定要在组件销毁时，清除定时任务或解绑全局事件，防止内存泄漏**

```jsx
// demo 
useEffect(() => {
	function mouseMoveHandler(event) {
	    setX(event.clientX)
	    setY(event.clientY)
	}
	
	// 绑定事件
	document.body.addEventListener('mousemove', mouseMoveHandler)
	
	// 解绑事件
	return () => document.body.removeEventListener('mousemove', mouseMoveHandler)
}, [])
```

**useEffect让纯函数有了副作用**

- 默认情况下，执行纯函数，输入参数，返回结果，无副作用
- 所谓副作用，就是对函数之外造成影响，如设置全局定时任务
- 而组件需要副作用，所以有需要useEffect”钩“到纯函数中

**useEffect中返回函数callbackFn**

- useEffect依赖[]，组件销毁时执行fn，等同于WillUnmount
- useEffect无依赖或依赖[a, b]，组件更新时执行fn
- 即，下一次执行useEffect之前，就会执行fn，无论更新或卸载

```jsx
// demo
// DidMount 和 DidUpdate
useEffect(() => {
  console.log(`开始监听 ${friendId} 在线状态`)

  // 【特别注意】
  // 此处并不完全等同于 WillUnMount
  // props 发生变化，即更新，也会执行结束监听
  // 准确的说：返回的函数，会在下一次 effect 执行之前，被执行
  return () => {
      console.log(`结束监听 ${friendId} 在线状态`)
  }
})
```

### useRef 使用

```jsx
// demo
import React, { useRef, useEffect } from 'react'

function UseRef() {
	const btnRef = useRef(null) // 初始值
	
	// const numRef = useRef(0)
	// numRef.current
	
	useEffect(() => {
	  console.log(btnRef.current) // DOM 节点
	}, [])
	
	return <div>
	  <button ref={btnRef}>click</button>
	</div>
}

export default UseRef
```

### useContext 使用

```jsx
// demo 主题颜色传递
import React, { useContext } from 'react'

// 主题颜色
const themes = {
    light: {
        foreground: '#000',
        background: '#eee'
    },
    dark: {
        foreground: '#fff',
        background: '#222'
    }
}

// 创建 Context
const ThemeContext = React.createContext(themes.light) // 初始值

function ThemeButton() {
    const theme = useContext(ThemeContext)

    return <button style={{ background: theme.background, color: theme.foreground }}>
        hello world
    </button>
}

function Toolbar() {
    return <div>
        <ThemeButton></ThemeButton>
    </div>
}

function App() {
    return <ThemeContext.Provider value={themes.dark}>
        <Toolbar></Toolbar>
    </ThemeContext.Provider>
}

export default App
```

### useReducer 使用

```jsx
// demo
import React, { useReducer } from 'react'

const initialState = { count: 0 }

const reducer = (state, action) => {
    switch (action.type) {
        case 'increment':
            return { count: state.count + 1 }
        case 'decrement':
            return { count: state.count - 1 }
        default:
            return state
    }
}

function App() {
    // 很像 const [count, setCount] = useState(0)
    const [state, dispatch] = useReducer(reducer, initialState)

    return <div>
        count: {state.count}
        <button onClick={() => dispatch({ type: 'increment' })}>increment</button>
        <button onClick={() => dispatch({ type: 'decrement' })}>decrement</button>
    </div>
}

export default App
```

**useReducer和redux的区别**

- useReducer是useState的代替方案，用于state复杂变化
- useReducer是单个组件状态管理，组件通讯还需要props
- redux是全局状态管理，多组件共享数据

### useMemo 使用

```jsx
// demo
import React, { useState, memo, useMemo } from 'react'

// 子组件
// function Child({ userInfo }) {
//     console.log('Child render...', userInfo)

//     return <div>
//         <p>This is Child {userInfo.name} {userInfo.age}</p>
//     </div>
// }
// 类似 class PureComponent ，对 props 进行浅层比较
// 子组件需要用memo函数封装
const Child = memo(({ userInfo }) => {
    console.log('Child render...', userInfo)

    return <div>
        <p>This is Child {userInfo.name} {userInfo.age}</p>
    </div>
})

// 父组件
function App() {
    console.log('Parent render...')

    const [count, setCount] = useState(0)
    const [name, setName] = useState('双越老师')

    // const userInfo = { name, age: 20 }
    // 用 useMemo 缓存数据，有依赖
    const userInfo = useMemo(() => {
        return { name, age: 21 }
    }, [name])

    return <div>
        <p>
            count is {count}
            <button onClick={() => setCount(count + 1)}>click</button>
        </p>
        <Child userInfo={userInfo}></Child>
    </div>
}

export default App
```

**useMemo总结**

- React默认会更新所有子组件
- class组件使用shouldComponentUpdate和PureComponent做优化
- Hooks中使用useMemo，但优化的原理是相同的，子组件需要用memo函数封装

### useCallback使用

```jsx
// demo
import React, { useState, memo, useMemo, useCallback } from 'react'

// 子组件，memo 相当于 PureComponent
const Child = memo(({ userInfo, onChange }) => {
    console.log('Child render...', userInfo)

    return <div>
        <p>This is Child {userInfo.name} {userInfo.age}</p>
        <input onChange={onChange}></input>
    </div>
})

// 父组件
function App() {
    console.log('Parent render...')

    const [count, setCount] = useState(0)
    const [name, setName] = useState('双越老师')

    // 用 useMemo 缓存数据
    const userInfo = useMemo(() => {
        return { name, age: 21 }
    }, [name])

    // 用 useCallback 缓存函数
    const onChange = useCallback(e => {
        console.log(e.target.value)
    }, [])

    return <div>
        <p>
            count is {count}
           <button onClick={() => setCount(count + 1)}>click</button>
        </p>
        <Child userInfo={userInfo} onChange={onChange}></Child>
    </div>
}

export default App
```

**useCallback总结**

- useMemo缓存数据
- useCallback缓存函数
- 两者都是React Hooks的常见优化策略

---

### 自定义 Hook

```jsx
// demo
import { useState, useEffect } from 'react'
import axios from 'axios'

// 封装 axios 发送网络请求的自定义 Hook
function useAxios(url) {
	const [loading, setLoading] = useState(false)
	const [data, setData] = useState()
	const [error, setError] = useState()
	
	useEffect(() => {
	  // 利用 axios 发送网络请求
		setLoading(true)
		axios.get(url) // 发送一个 get 请求
	    .then(res => setData(res))
	    .catch(err => setError(err))
	    .finally(() => setLoading(false))
	}, [url])
	
	return [loading, data, error]
}
	
export default useAxios
```

**自定义Hook总结**

- 本质是一个函数，以use开头（重要）
- 内部正常使用useState，useEffect或者其他Hooks
- 自定义返回结果，格式不限

第三方Hooks

1. https://nikgraf.github.io/react-hooks
2. https://github.com/umijs/hooks

---

### Hooks使用规范

1. 只能用于React函数组件和自定义Hook中，其他地方不可以；
2. 只能用于顶层代码，不能在循环、判断中使用Hooks；
3. eslint插件eslint-plugin-react-hooks可以帮助你，（CRA脚手架是有配置的）；

    ```jsx
    // ESLint 配置文件
    // eslint-plugin-react-hooks
    {
    	"plugins": [
    		// ...此处省略 N 行...
    		"react-hooks"
    	],
    	"rules": {
    		// ...此处省略 N 行...
    		"react-hooks/rules-of-hooks": "error",  // 检查 Hook 的规则
    		"react-hooks/exhaustive-deps": "warn"   // 检查 effect 的依赖
    	}
    }
    ```

### 关于Hooks调用顺序

```jsx
// demo
import React, { useState, useEffect } from 'react'

function Teach({ couseName }) {
  // 函数组件，纯函数，执行完即销毁
  // 所以，无论组件初始化（render）还是组件更新（re-render）
  // 都会重新执行一次这个函数，获取最新的组件
  // 这一点和 class 组件不一样，class组价是有实例的，函数组件没有实例

  // render: 初始化 state 的值 '张三'
  // re-render: 读取 state 的值 '张三'
  const [studentName, setStudentName] = useState('张三')
	
	// 错误使用
  // if (couseName) {
  //     const [studentName, setStudentName] = useState('张三')
  // }

  // render: 初始化 state 的值 '双越'
  // re-render: 读取 state 的值 '双越'
  const [teacherName, setTeacherName] = useState('双越')
	
	// 错误使用
  // if (couseName) {
  //     useEffect(() => {
  //         // 模拟学生签到
  //         localStorage.setItem('name', studentName)
  //     })
  // }

  // render: 添加 effect 函数
  // re-render: 替换 effect 函数（内部的函数也会重新定义）
  useEffect(() => {
      // 模拟学生签到
      localStorage.setItem('name', studentName)
  })

  // render: 添加 effect 函数
  // re-render: 替换 effect 函数（内部的函数也会重新定义）
  useEffect(() => {
      // 模拟开始上课
      console.log(`${teacherName} 开始上课，学生 ${studentName}`)
  })

  return <div>
      课程：{couseName}，
      讲师：{teacherName}，
      学生：{studentName}
  </div>
}

export default Teach
```

**Hooks调用顺序必须保持一致**

- 无论是render还是re-render，Hooks调用顺序必须一致
- 如果Hooks出现在循环、判断里，则无法保证顺序一致
- Hooks严重依赖于调用顺序！重要！

---

### class组件逻辑复用

- 高阶组价HOC
    1. 组件层级嵌套过多，不易渲染，不易调试；
    2. HOC会劫持props，必须严格规范，容易出现疏漏；
- Render Prop
    1. 学习成本高，不易理解；
    2. 只能传递纯函数，而默认情况下纯函数功能有限；

---

- Mixins早已废弃
    1. 变量作用域来源不清；
    2. 属性重名；
    3. Mixins引入过多会导致顺序冲突；

---

### **Hooks组件逻辑复用好处**

- 完全符合Hooks原有规则，没有其他要求，易理解记忆
- 变量作用域明确
- 不会产生组件嵌套

---

### React Hooks注意事项（坑点）

- useState初始化值，只有第一次有效

```jsx
// demo
import React, { useState } from 'react'

// 子组件
function Child({ userInfo }) {
  // render: 初始化 state
  // re-render: 只恢复初始化的 state 值，不会再重新设置新的值
  //            只能用 setName 修改
  const [ name, setName ] = useState(userInfo.name)

  return <div>
      <p>Child, props name: {userInfo.name}</p> // 双越 click之后：慕课网
      <p>Child, state name: {name}</p>          // 双越 click之后：双越
  </div>
}

function App() {
  const [name, setName] = useState('双越')
  const userInfo = { name }

  return <div>
      <div>
          Parent &nbsp;
          <button onClick={() => setName('慕课网')}>setName</button>
      </div>
      <Child userInfo={userInfo}/>
  </div>
}

export default App
```

- useEffect内部不能修改state

```jsx
// demo
import React, { useState, useRef, useEffect } from 'react'

function UseEffectChangeState() {
    const [count, setCount] = useState(0)

    // 模拟 DidMount
    // const countRef = useRef(0) // 使用useRef来解决这个问题
    useEffect(() => {
        console.log('useEffect...', count)

        // 定时任务
        const timer = setInterval(() => {
            // console.log('setInterval...', countRef.current)
            console.log('setInterval...', count) // 打印一直是0
            setCount(count + 1)
            // setCount(++countRef.current)
        }, 1000)

        // 清除定时任务
        return () => clearTimeout(timer)
    }, []) // 依赖为 []

    // 依赖为 [] 时： re-render 不会重新执行 effect 函数
    // 没有依赖：re-render 会重新执行 effect 函数

    return <div>count: {count}</div> //这是输出一直是 1 
}

export default UseEffectChangeState
```

- useEffect可能出现死循环，useEffect第二个参数不能依赖引用类型，如Object，Array等，否则会出现死循环

```jsx
// demo
import { useState, useEffect } from 'react'
import axios from 'axios'

// 封装 axios 发送网络请求的自定义 Hook
function useAxios(url, config={}) {
  const [loading, setLoading] = useState(false)
  const [data, setData] = useState()
  const [error, setError] = useState()

  useEffect(() => {
      // 利用 axios 发送网络请求
      setLoading(true)
      axios(url, config) // 发送一个 get 请求
          .then(res => setData(res))
          .catch(err => setError(err))
          .finally(() => setLoading(false))
  }, [url， config])   // config是对象，不能作为第二个参数
											// 可以把对象解构出来使用，如config.a、config.b等
	// useEffect第二个参数比较是使用 Object.is(1, 1) 方法来判断两个值是否为同一个值

  return [loading, data, error]
}

export default useAxios

```