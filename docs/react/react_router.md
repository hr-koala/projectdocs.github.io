# React 路由

## React-Router 的实现原理是什么？

客户端路由实现的思想：

- 基于 `hash` 的路由：通过**锚点**来改变浏览器的 URL,通过监听 `hashchange` 事件，感知 `hash` 的变化
  - 改变 hash 可以直接通过 `location.hash="#xxx"` 改变 url，触发 hashchange 事件<br/>
    **优点**
    - 无需向后端发起请求，在页面的 `hash` 值发生变化时，无需向后端发起请求, window.hashChange 就可以监听事件的改变，并按规则加载相应的代码就好
    - 兼容性较好，大部分浏览器支持 <br/>
      **缺点**
    - '#'不太美观；
    - 由于 Hash 对于服务端来说是不可见的，所以对于 SEO 不友好
- 基于 `H5 history` 路由：(`pushState() 、replaceState() + popstate 事件`)
  - 改变 url 可以通过 `history.pushState` 和 `replaceState`等，会将 `URL` 压入**堆栈**，同时能够应用 `history.go()` 等 API
  - 监听 `url` 的变化可以通过自定义事件触发实现

react-router 实现的思想：

- 基于 `history` 库来实现上述不同的客户端路由实现思想，并且能够保存历史记录等，磨平浏览器差异，上层无感知
- 通过维护的列表，在每次 `URL` 发生变化的回收，通过配置的 路由路径，匹配到对应的 `Component`，并且 render

## 如何配置 React-Router 实现路由切换

（1）**使用 `<Route> `组件**

路由匹配是通过比较 `<Route>` 的 `path` 属性和当前地址的 `pathname` 来实现的。当一个 `<Route>` 匹配成功时，
它将渲染其内容，当它不匹配时就会渲染 `null`。没有路径的 `<Route>` 将始终被匹配。

```tsx
// when location = { pathname: '/about' }
<Route path='/about' component={About}/> // renders <About/>
<Route path='/contact' component={Contact}/> // renders null
<Route component={Always}/> // renders <Always/>
```

（2）**结合使用 `<Switch>` 组件和 `<Route>` 组件**

`<Switch>` 用于将 `<Route>` 分组。

```tsx
<Switch>
  <Route exact path="/" component={Home} />
  <Route path="/about" component={About} />
  <Route path="/contact" component={Contact} />
</Switch>
```

`<Switch>` 不是分组`<Route>`所必须的，但它通常很有用。 一个 `<Switch>` 会遍历其所有的子 `<Route>`元素，并仅渲染与当前地址匹配的第一个元素。

（3）**使用 `<Link>`、 `<NavLink>`、`<Redirect>` 组件**

`<Link>` 组件来在你的应用程序中创建链接。无论你在何处渲染一个 `<Link>` ，都会在应用程序的` HTML` 中渲染锚（`<a>`）。

```tsx
<Link to="/">Home</Link>
// <a href='/'>Home</a>
```

`<NavLink>` 是一种特殊类型的 `<Link>` 当它的 `to`属性与当前地址匹配时，可以将其定义为"活跃的"。

```tsx
// location = { pathname: '/react' }
<NavLink to="/react" activeClassName="hurray">
  React
</NavLink>
// <a href='/react' className='hurray'>React</a>
```

当我们想强制导航时，可以渲染一个`<Redirect>`，当一个`<Redirect>`渲染时，它将使用它的`to`属性进行定向。

## React-Router 怎么设置重定向？

使用`<Redirect>`组件实现路由的重定向：

```tsx
<Switch>
  <Redirect from="/users/:id" to="/users/profile/:id" />
  <Route path="/users/profile/:id" component={Profile} />
</Switch>
```

当请求 `/users/:id` 被重定向去 '`/users/profile/:id`' ：

- 属性 `from: string` ：需要匹配的将要被重定向路径。
- 属性 `to: string` ：重定向的 URL 字符串
- 属性 `to: object `：重定向的 location 对象
- 属性 `push: bool` ：若为真，重定向操作将会把新地址加入到访问历史记录里面，并且无法回退到前面的页面。

## react-router 里的 Link 标签和 a 标签的区别

从最终渲染的 `DOM` 来看，这两者都是链接，都是 标签，区别是 ∶

`<Link>`是`react-router` 里实现路由跳转的链接，一般配合`<Route>` 使用，`react-router`接管了其默认的链接跳转行为，区别于传统的页面跳转，`<Link>` 的“跳转”行为只会触发相匹配的`<Route>`对应的页面内容更新，而不会刷新整个页面。

`<Link>`做了 3 件事情:

- 有`onclick`那就执行 onclick
- `click`的时候阻止`a`标签默认事件
- 根据跳转`href`(即是 to)，用`history` (web 前端路由两种方式之一，history & hash)跳转，此时只是链接变了，并没有刷新页面而`<a>`标签就是普通的超链接了，用于从当前页面跳转到 href 指向的另一 个页面(非锚点情况)。

`a`标签默认事件禁掉之后做了什么才实现了跳转?

```tsx{3-4}
let domArr = document.getElementsByTagName('a')
[...domArr].forEach(item=>{
  item.addEventListener('click',function () {
    location.href = this.href
  })
})
```

## React-Router 如何获取 URL 的参数和历史对象？

（1）**获取 URL 的参数**

- `get` 传值

路由配置还是普通的配置，如： 'admin' ，传参方式如： 'admin?id='1111'' 。通过 `this.props.location.search` 获取 url 获取到一个字符串 '?id='1111'<br/>
可以用 `url，qs，querystring`，浏览器提供的 api `URLSearchParams` 对象或者自己封装的方法去解析出 `id` 的值。

- 动态路由传值

路由需要配置成动态路由：如 `path='/admin/:id'` ，传参方式，如 'admin/111' 。通过 `this.props.match.params.id` 取得 url 中的动态路由 id 部分的值，除此之外还可以通过 `useParams（Hooks）` 来获取

- 通过 `query` 或 `state` 传值

传参方式如：在 `Link` 组件的 `to` 属性中可以传递对象 `{pathname:'/admin',query:'111',state:'111'}`; 。
通过 `this.props.location.state` 或 `this.props.location.query` 来获取即可，传递的参数可以是对象、数组等，但是存在**缺点**就是只要刷新页面，参数就会丢失。

（2）**获取历史对象**

如果 React >= 16.8 时可以使用 React Router 中提供的 `Hooks`

```tsx{1}
import { useHistory } from "react-router-dom";
let history = useHistory();
```

2.使用 `this.props.history` 获取历史对象

```tsx
let history = this.props.history;
```

## React-Router 4 怎样在路由变化时重新渲染同一个组件？

当路由变化时，即组件的`props`发生了变化，会调用`componentWillReceiveProps`等生命周期钩子。那需要做的只是：
当路由改变时，根据路由，也去请求数据：

```tsx
class NewsList extends Component {
  componentDidMount () {
    this.fetchData(this.props.location);
  }
  fetchData(location) {
    const type = location.pathname.replace('/', '') || 'top'
    this.props.dispatch(fetchListData(type))
  }
  componentWillReceiveProps(nextProps) {
    if (nextProps.location.pathname != this.props.location.pathname) {
      this.fetchData(nextProps.location);
    }
  }
  render () {
  ...
  }
}
```

利用生命周期`componentWillReceiveProps`，进行重新`render`的预处理操作。

## React-Router 的路由有几种模式？

React-Router 支持使用 `hash`（对应 `HashRouter`）和 `browser`（对应 `BrowserRouter`） 两种路由规则， react-router-dom 提供了 `BrowserRouter` 和 `HashRouter` 两个组件来实现应用的 UI 和 URL 同步：

- `BrowserRouter` 创建的 URL 格式：http://xxx.com/path
- `HashRouter` 创建的 URL 格式：http://xxx.com/#/path

（1）**`BrowserRouter`**

它使用 HTML5 提供的 `history API`（**pushState、replaceState + popstate 事件**）来保持 UI 和 URL 的同步。由
此可以看出，**`BrowserRouter` 是使用 HTML5 的 history API 来控制路由跳转的**：

```tsx
<BrowserRouter
  basename={string}
  forceRefresh={bool}
  getUserConfirmation={func}
  keyLength={number}
/>
```

其中的属性如下：

- `basename` 所有路由的基准 URL。`basename` 的正确格式是前面有一个前导斜杠，但不能有尾部斜杠；

```tsx
<BrowserRouter basename="/calendar">
 <Link to="/today" />
</BrowserRouter>
// 等同于
 <a href="/calendar/today" />
```

- `forceRefresh` 如果为 true，在导航的过程中整个页面将会刷新。一般情况下，只有在不支持 HTML5 history API 的浏览器中使用此功能；
- `getUserConfirmation` 用于确认导航的函数，默认使用 `window.confirm`。例如，当从 /a 导航至 /b 时，会使用默认的 `confirm` 函数弹出一个提示，用户点击确定后才进行导航，否则不做任何处理；

```tsx
// 这是默认的确认函数
const getConfirmation = (message, callback) => {
  const allowTransition = window.confirm(message);
  callback(allowTransition);
};
<BrowserRouter getUserConfirmation={getConfirmation} />;
```

需要配合 `<Prompt>` 一起使用。

- `KeyLength` 用来设置 `Location.Key` 的长度。

（2）**`HashRouter`**

使用 URL 的 hash 部分（即 `window.location.hash`）来保持 UI 和 URL 的同步。由此可以看出，**`HashRouter` 是通过 URL 的 hash 属性来控制路由跳转的**：

```tsx
<HashRouter basename={string} getUserConfirmation={func} hashType={string} />
```

其中的参数如下：

- `basename`, `getUserConfirmation` 和 `BrowserRouter` 功能一样；
- hashType `window.location.hash` 使用的 hash 类型，有如下几种：
  - slash - 后面跟一个斜杠，例如 #/ 和 #/sunshine/lollipops；
  - noslash - 后面没有斜杠，例如 # 和 #sunshine/lollipops；
  - hashbang - Google 风格的 ajax crawlable，例如 #!/ 和 #!/sunshine/lollipops。

## React-Router 4 的 Switch 有什么用？

`Switch` 通常被用来包裹 Route，用于渲染与路径匹配的第一个子 `<Route>` 或 `<Redirect> `，它里面不能放其他元素。
假如不加 `<Switch>` ：

```tsx
import { Route } from 'react-router-dom'
<Route path="/" component={Home}></Route>
<Route path="/login" component={Login}></Route>
```

Route 组件的 `path` 属性用于匹配路径，因为需要匹配 / 到 Home ，匹配 /login 到 Login ，所以需要两个 Route，但是不能这么写。
这样写的话，当 URL 的 path 为 “/login” 时， `<Route path="/" />` 和 `<Route path="/login" />` 都会被匹配，因此页面会展示 Home 和 Login 两个组件。这时就需要借助 `<Switch>` 来做到只显示一个匹配组件：

```tsx
import { Switch, Route } from "react-router-dom";
<Switch>
  <Route path="/" component={Home}></Route>
  <Route path="/login" component={Login}></Route>
</Switch>;
```

此时，再访问 “`/login`” 路径时，却**只**显示了 `Home` 组件。这是就用到了 `exact` 属性，它的作用就是**精确匹配**路径，经常与 `<Switch>` 联合使用。只有当 URL 和该 `<Route>` 的 path 属性完全一致的情况下才能匹配上：

```tsx
import { Switch, Route } from "react-router-dom";
<Switch>
  <Route exact path="/" component={Home}></Route>
  <Route exact path="/login" component={Login}></Route>
</Switch>;
```