---
title: React基础
date: 2021-06-22 11:05:59
tags:
---

### JSX语法

```bash
const text = {
	__html: '<div>hello world</div>'
}

return <div dangerouslySetInnerHTML={text}></div>
```

### class组件和函数组件

```jsx
// class组件
class List extends React.Component {
	constructor(props) {
		super(props)
	}
	
	render() {
		const { list } = this.props
	
		return <ul>{list.map((item, index) => {
			return <li key={item.id}>
				<span>{item.title}</span>
			</li>
		})}</ul>
	}
}

// 函数组件
function List(props) {
	const { list } = props

	return <ul>{list.map(item => {
		return <li key={item.id}>
			<span>{item.title}</span>
		</li>
	)}</ul>
}
```

<!-- more -->

### React event 事件

```jsx
clickHandle = (event) => {
	console.log(event)
}

1. event 是 SyntheticEvent, 模拟出来 DOM 事件所有能力；
2. event.navtionEvent 是原生事件对象；
3. 所有的事件，都被挂载到 document 上；
4. 和 DOM 事件不一样，和 Vue 事件也不一样；
```

### Form表单

```jsx
// 用 htmlFor 代替 for
<label htmlFor="inputName">姓名</label>
```

### 受控组件 vs 非受控组件

- 优先使用受控组件，符合React设计原理；
- 必须操作DOM时，再使用非受控组件；

```jsx
// 受控组件
1. 使用 this.setState 改变值；
```

```jsx
// 非受控组件 - 使用场景

1. 必须手动操作DOM元素，setState实现不了；
2. 文件上传 <input type="file" />
3. 某些富文本编辑器，需要传入DOM元素；

```

### setState

1.  不可变值（函数式编程，纯函数）；

```jsx
/** 不能直接修改 this.state 里的属性的值 **/

1. 数组
export class index extends Component {
	constructor(props) {
		super(props)
		this.state = {
			list: [1,2,3]
		}
	}
	// 可使用方式：concat map filter slice等，会返回新数组的方法；
	// 注意，不能直接对 this.state.list 进行 push pop splice 等，这样违反不可变值；

	const listCopy = this.state.list.slice()
	listCopy.splice(1, 0, 'a')

	this.setState({
		list: listCopy
	})

	render() {
		return <div>
			<div>setState</div>
		</div>
	}
}

export default index

2. 对象
this.setState({
	obj1: Object.assgin({}, this.state.obj1, {a: 100}) 
	obj2: {...this.state.obj2, a:100}
})

// 注意，不能直接对 this.state.obj 进行属性设置，这样违反不可变值
```

2. 可能是异步更新；

```jsx
1. this.setState 直接使用是异步的，需要回调里接收最新值
this.setState({
    count: this.state.count + 1
}, () => {
    // 联想 Vue $nextTick - DOM
    console.log('count by callback', this.state.count) // 回调函数中可以拿到最新的 state
})
console.log('count', this.state.count) // 异步的，拿不到最新值

2. setTimeout 中 setState 是同步的
setTimeout(() => {
  this.setState({
    count: this.state.count + 1
	 })
   console.log('count in setTimeout', this.state.count)
}, 0)

3. 自己定义的 DOM 事件，setState 是同步的
bodyClickHandler = () => {
  this.setState({
      count: this.state.count + 1
  })
  console.log('count in body event', this.state.count)
}
componentDidMount() {
  // 自己定义的 DOM 事件，setState 是同步的
  document.body.addEventListener('click', this.bodyClickHandler)
}
componentWillUnmount() {
  // 及时销毁自定义 DOM 事件
  document.body.removeEventListener('click', this.bodyClickHandler)
  // clearTimeout
}

```

3. 可能会被合并；

```jsx
1. 传入对象，会被合并（类似 Object.assign ）。执行结果只一次 +1
this.setState({
  count: this.state.count + 1
})
this.setState({
  count: this.state.count + 1
})
this.setState({
  count: this.state.count + 1
})

2. 传入函数，不会被合并。执行结果是 +3
this.setState((prevState, props) => {
  return {
      count: prevState.count + 1
  }
})
this.setState((prevState, props) => {
  return {
      count: prevState.count + 1
  }
})
this.setState((prevState, props) => {
  return {
      count: prevState.count + 1
  }
})
```

### 组件生命周期

![Class生命周期](https://z3.ax1x.com/2021/06/22/RVOWKf.png)

### 函数组件和Class组件的区别

函数组件

1. 纯函数，输入props，输出JSX；
2. 没有实例，没有生命周期，没有state；
3. 不能扩展其他方法；

### Portals 作用和使用场景

子组件逃离父组件

- overflow: hidden
- 父组件 z-index 值太小
- fixed 需要放在 body 第一层级

```jsx
class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
        }
    }
    render() {
        // // 正常渲染
        // return <div className="modal">
        //     {this.props.children} {/* vue slot */}
        // </div>

        // 使用 Portals 渲染到 body 上。
        // fixed 元素要放在 body 上，有更好的浏览器兼容性。
        return ReactDOM.createPortal(
            <div className="modal">{this.props.children}</div>,
            document.body // DOM 节点
        )
    }
}

export default App
```

### Context

- 主要用于数据的传递，逻辑不复杂的场景，如主题切换，语言切换等；
- 主要api，1. 生产数据：Context.Provider；
2. 消费数据 函数组件：Context.Consumer，Class组件：static contextType = ThemeContex；

```jsx
// demo 主题切换
import React from 'react'

// 创建 Context 填入默认值（任何一个 js 变量）
const ThemeContext = React.createContext('light')

// 底层组件 - 函数是组件使用方式
function ThemeLink (props) {
    // const theme = this.context // 会报错。函数式组件没有实例，即没有 this

    // 函数式组件可以使用 Consumer
    return <ThemeContext.Consumer>
        { value => <p>link's theme is {value}</p> }
    </ThemeContext.Consumer>
}

// 底层组件 - class 组件使用方式
class ThemedButton extends React.Component {
    // 指定 contextType 读取当前的 theme context。
    // static contextType = ThemeContext // 也可以用 ThemedButton.contextType = ThemeContext

    render() {
        const theme = this.context // React 会往上找到最近的 theme Provider，然后使用它的值。
        return <div>
            <p>button's theme is {theme}</p>
        </div>
    }
}
ThemedButton.contextType = ThemeContext // 指定 contextType 读取当前的 theme context。

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar(props) {
    return (
        <div>
            <ThemedButton />
            <ThemeLink />
        </div>
    )
}

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: 'light'
        }
    }
    render() {
        return <ThemeContext.Provider value={this.state.theme}>
            <Toolbar />
            <hr/>
            <button onClick={this.changeTheme}>change theme</button>
        </ThemeContext.Provider>
    }
    changeTheme = () => {
        this.setState({
            theme: this.state.theme === 'light' ? 'dark' : 'light'
        })
    }
}

export default App
```

### 异步组件

- import()
- React.lazy
- React.Suspense

```jsx
import React from 'react'

const ContextDemo = React.lazy(() => import('./ContextDemo'))

class App extends React.Component {
    constructor(props) {
        super(props)
    }
    render() {
        return <div>
            <p>引入一个动态组件</p>
            <hr />
            <React.Suspense fallback={<div>Loading...</div>}>
                <ContextDemo/>
            </React.Suspense>
        </div>

        // 1. 强制刷新，可看到 loading （看不到就限制一下 chrome 网速）
        // 2. 看 network 的 js 加载
    }
}

export default App
```

### *性能优化（重要）

- shouldComponentUpdate 简称（SCU）
    1. SCU默认返回 true，即 React 默认重新渲染所有子组件；
    2. 必须配合”不可变值“一起使用；
    3. 可先不用SCU，有性能问题时在考虑使用；
- PureComponent 和 React.memo
    1. Class组件 PureComponent，SCU中实现了浅比较；
    2. memo，函数组件中的PureComponent；
    3. 浅比较已使用大部门情况（尽量不要做深度比较）；
- 不可变值 immutable.js
    1. 彻底拥抱”不可变值“；
    2. 基于共享数据（不是深拷贝），速度好；
    3. 有一定学习和迁移成本，按需使用；

```jsx
// shouldComponentUpdate Demo
import React from 'react'

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            count: 0
        }
    }
    render() {
        return <div>
            <span>{this.state.count}</span>
            <button onClick={this.onIncrease}>increase</button>
        </div>
    }
    onIncrease = () => {
        this.setState({
            count: this.state.count + 1
        })
    }
    // 演示 shouldComponentUpdate 的基本使用
    shouldComponentUpdate(nextProps, nextState) {
        if (nextState.count !== this.state.count) {
            return true // 可以渲染
        }
        return false // 不重复渲染
    }
		
		// React 默认：父组件有更新，子组件则无条件也更新！！！
}

export default App

// PureComponent Demo
import React from 'react'

class App extends React.PureComponent {
	constructor(props) {
		super(props)
	}
}

export default App

// memo Demo
import React from 'react'

function MyComponent(props) {
  /* 使用 props 渲染 */
}
function areEqual(prevProps, nextProps) {
  /*
  如果把 nextProps 传入 render 方法的返回结果与
  将 prevProps 传入 render 方法的返回结果一致则返回 true，
  否则返回 false
  */
}
export default React.memo(MyComponent, areEqual)

```

### 关于组件公共逻辑的抽离

- mixin，已被 React 弃用
- 高阶组件 HOC
- Render Props

HOC vs Render Props

1. HOC：模式简单，但会增加组件层级；
2. Render Props：代码简洁，学习成本高；
3. 按需使用；

```jsx
// 高阶组件 HOC Demo 7-24
// 高阶组件不是一种功能，而是一种模式
import React from 'react'

// 高阶组件
const withMouse = (Component) => {
  class withMouseComponent extends React.Component {
      constructor(props) {
          super(props)
          this.state = { x: 0, y: 0 }
      }

      handleMouseMove = (event) => {
          this.setState({
              x: event.clientX,
              y: event.clientY
          })
      }

      render() {
          return (
              <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
                  {/* 1. 透传所有 props 2. 增加 mouse 属性 */}
                  <Component {...this.props} mouse={this.state}/>
              </div>
          )
      }
  }
  return withMouseComponent
}

const App = (props) => {
  const a = props.a
  const { x, y } = props.mouse // 接收 mouse 属性
  return (
      <div style={{ height: '500px' }}>
          <h1>The mouse position is ({x}, {y})</h1>
          <p>{a}</p>
      </div>
  )
}

export default withMouse(App) // 返回高阶函数
```

```jsx
// Render Props 7-25
// Render Props 的核心思想
// 通过一个函数将 Class 组件的 state 作为props传递给纯函数组件
class Factory extends React.Component {
	constructor(props) {
		this.state = {
			// state 即多个组件的公共逻辑的数据
		}
	}

	// 修改 state
	render() {
		return <div>{this.props.render(this.state)}</div>
	}
}

const App = () => {
	<Factory render={
		// render 是一个函数组件
		(props) => <p>{props.a} {props.b} ...</p>
	}/>	
}
 

// Demo
import React from 'react'
import PropTypes from 'prop-types'

class Mouse extends React.Component {
  constructor(props) {
      super(props)
      this.state = { x: 0, y: 0 }
  }

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    })
  }

  render() {
    return (
      <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
          {/* 将当前 state 作为 props ，传递给 render （render 是一个函数组件） */}
          {this.props.render(this.state)}
      </div>
    )
  }
}
Mouse.propTypes = {
  render: PropTypes.func.isRequired // 必须接收一个 render 属性，而且是函数
}

const App = (props) => (
  <div style={{ height: '500px' }}>
      <p>{props.a}</p>
      <Mouse render={
          /* render 是一个函数组件 */
          ({ x, y }) => <h1>The mouse position is ({x}, {y})</h1>
      }/>
      
  </div>
)

/**
 * 即，定义了 Mouse 组件，只有获取 x y 的能力。
 * 至于 Mouse 组件如何渲染，App 说了算，通过 render prop 的方式告诉 Mouse 。
 */

export default App
```

### React.createRef() 使用

```jsx
// demo
class MyComponent extends React.Component {
  constructor(props) {
    super(props)

    this.inputRef = React.createRef()
  }

  render() {
    return <input type="text" ref={this.inputRef} />
  }

  componentDidMount() {
    this.inputRef.current.focus()
  }
}
```