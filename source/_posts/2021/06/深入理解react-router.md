---
title: 深入理解react router
date: 2021-06-05 11:17:23
tags: react
categories: router
---

## 浏览器导航相关

URI（统一资源标识符）的通用语法： `URI = scheme:[//authority]path[?query][#fragment]`

其中 authority 可由以下三部分组成：`authority = [userinfo@]host[:port]`

fragment 为片段标识符，通常标记为已获取资源的子资源，可选

URL 为 URI 的一种，统一资源定位器

未保留字符不需要进行百分号编码`（a~z, A~Z, 0~9, - _ . ~）`共 66 个，保留字符需要进行对应的编码，因为其有特殊的含义在 URL 中。`（!*'();:@&=+$,/?#[]）`共 18 个

encodeURI 对 66 个未保留字符，18 个保留字符，除去`[]`，不对这 82 个字符编码，对于非 ASCII 字符，其将转换成 UTF-8 编码字节序，然后放置%进行编码，也就是会将中文等字符进行编码

encodeURIComponent 将转义除字母、数字、( ) . ! ~ \* ' - \_ 以外的所有字符。encodeURI 适合对一个完整的 URI 进行编码，而 encodeURIComponent 则被用作对 URI 中的一个组件或者一个片段进行。

浏览器记录没有直接的 api 可以获取，可以通过 window.history.length 来获取当前记录栈的长度信息。由浏览器统一管理，不属于哪一个具体的页面。

```JavaScript
interface History{
    readonly length: number;
    scrollRestoration: 'auto' | 'manual';
    readonly state: any;
    back(): void; // 栈指针后退一位
    forward(): void; // 跳转到当前栈指针所指前一个记录的方法，等同于history.go(1)，是否刷新取决于栈记录是如何得到的
    go(delta?: number): void; // -1表示后退到上一个页面，1表示前进一个页面，0表示刷新当前页面，与location.reload方式行为一致，此方法会刷新页面，会触发popstate事件，pushState永远产生新的栈顶并指向它
    pushState(data: any, title: string, url?: string | null): void; // 无刷新增加历史栈记录，改变浏览器的url，有中文也会URF-8编码，即便浏览器显示的还是中文，但是已经是编码过的。第一个参数是传入的状态，设置了第一个参数之后，可以通过history.state读取。firfox中state大小限制640k，并且跳转的url有同源策略。执行一次会增加一个历史栈，history.length会发生对应的变化，即使url参数不传。data对象采用了结构化拷贝算法，对象中不能设置函数
    replaceState(data: any, title: string, url?: string | null): void; // 类似pushState，但是是修改当前的历史记录，不处于栈顶的情况下修改的也是当前的，且不会将指针指向栈顶，history.length不会发生变化。url同pushstate都支持绝对和相对路径

    // 当前路径为/one/two/three
    // window.history.pushState(null, null, './four')
    // 当前路径为/one/two/four
    // window.history.pushState(null, null, './')
    // 当前路径为/one/two/

    // 当前路径为/one/two/
    // window.history.pushState(null, null, 'four')
    // 当前路径为/one/two/four

}
```

调用 history.pushState 改变 search 的值时，hash 的值会被清理

base 元素存在的情况下，进行添加和修改浏览器记录`<base href='/base/bar'>`，pushState 以/开头的绝对路径跳转的时候，base 是被忽略的，如果是相对路径，则会使用 base 作为基准

window.location.href 与 window.location 行为一致，与 pushState 不同的是其可以刷新对应的页面并重新加载 URL 指定的内容

window.location.hash 改变 URL 的 hash 值，改变 hash 同样会产生新的历史栈记录，如果设置的 hash 与当前的 hash 值相同，则不会产生任何事件和历史记录，如果改变 hash 不进行入栈的操作，通过 location.repalce 来实现

window.location.replace 替换栈记录，设置绝对路径时，会刷新页面

当移动栈指针的时候会触发 popstate 事件，通过 window.addEventListener 进行监听，对于回调函数的 event 事件，event.state 是重点关注的，其值为移动后栈中记录的 state 对象。pushState 以及 repalceState 不会触发 popstate 事件。当然你也可以使用 history.state 来获取 state 对象。location.href 设置 hash 的时候，一样的 hash 也会触发 popstate，但是栈没变，location.hash 则不会触发 popstate。

hashchange 用来监听浏览器的 hash 值变化，pushState 不触发 hashchange 事件

可以调用 window.dispatchEvent(new PopStateEvent('popstate'))来主动出发 popstate 事件

## react 的 history 库详解

history 是一个单独的库，提供了 createBrowserHistory，createHashHistory，creatMemoryHistory。其中个历史对象都具备

- 监听外界地址变化的能力
- 获取当前地址
- 增加和修改历史栈
- 在栈中移动当前历史栈指针
- 阻止跳转，定义跳转提示
- 获取当前历史栈长度以及最后一次导航行为
- 转换地址对象

```JavaScript
interface History<HistoryLocationState = LocationState> {
    length: number;
    action: Action; //最后一次导航的导航行为
    location: Location<HistoryLocationState>; // 当前历史地址
    push(path: Path, state?: HistoryLocationState): void; // 添加历史栈记录
    push(location: LocationDescriptorObject<HistoryLocationState>): void; //重载方法
    replace(path: Path, state?: HistoryLocationState): void; // 修改历史栈记录
    replace(location: LocationDescriptorObject<HistoryLocationState>): void; // 重载方法
    go(n: number): void;
    goBack(): void;
    goForward(): void;
    block(prompt?: boolean | string | TransitionPromptHook): UnregisterCallback; // 阻止导航行为
    listen(listener: LocationListener): UnregisterCallback; // 监听地址变化，传入一个回调函数，参数1为location，参数2为 action（不同的跳转类型），页面初始化不会触发。location前后值一致的情况下也会触发
    createHref(location: LocationDescriptorObject<HistoryLocationState>): Href; // 地址对象转换
}

interface BrowserHistoryBuildOptions {
    basename?: string;
    forceRefresh?: boolean; // 跳转是否刷新页面，默认不刷新
    getUserConfirmation?: typeof getUserConfirmation;
    keyLength?: number; // 历史栈中栈记录的key字符串的长度，默认6
    // ...通用配置
}

// browser history.push 底层使用history.pushState，无刷新，但是push会触发history的listen监听的回调
interface History<any> {
    push(path: string, state?: any): void;
    push(location: {
        pathname?: string;
        search?: string;
        state?: any;
        hash?: string;
    }): void;
}

// hashHistory 在创建hashHistory时，除了一些公共的配置，还可以设置hash类型，不可设置keyLength，forceRefresh配置
type HashType = 'hashbang' | 'noslash' | 'slash'; // 默认slash：其#号后面都会跟上/，noslash则#号后面都没有/，hashbang为#号后会跟上!和/，页面信息抓取用hashbang更好，页面初始化的过程中，encodedPath和hashPath不同，所以会进行一次初始化，因而每次会在页面后追加/#/
```

window.location.replace 仅传入 hash 字符串的调用会将 window.location.pathname 改成 base 的值。后续高版本中修复了这个问题。监听还是采用 listen 的方式进行

history.createHref 可以将 location 对象转换成对应的 URL 字符串，不会对原字符做任何编码处理，其会判断文档流中有没有 base 元素

memoryHistory，basename 在此中不被支持，其所有的信息都保存在 location 中。

```JavaScript
interface MemoryHistoryBuildOptions {
    getUserConfirmation?: typeof getUserconfirmation;
    initialEntries?: string[]; // 类似历史栈记录，默认['/']
    initialIndex?: number;
    keyLength?: number;
}
interface MemoryHistory<HistoryLocationState = LocationState> extends History<HistoryLocationState> {
    index: number;
    entries: Location<HistoryLocationState>[]; // 历史栈数组
    canGo(n: number): boolean; // 是否可以跳转到位置n
}
```

history 库的整体运行流程

![history](https://gitee.com/RenYaNan/wx-photo/raw/master/2021-6-7/1623047216789-%E5%9B%BE%E7%89%87.png)

问题：

1. 刷新页面后历史栈丢失
2. 移动端限制
3. hash 路由问题，手动修改页面 hash

## React 相关

creatContext 创建 context 容器，Provider 提供跨层级数据，consumer 进行接收，如果传递的数据进行了对应的更新，触发 render，那么 Provider 会将最新的 value 传递给所有的 Consumer。当提供的数据更新时，没 Consumer 的组件不会触发更新。当有多个 Provider 时，Consumer 使用的是最近一级的 Provider 提供的值。

useEffect 不会阻碍浏览器的绘制，使用了一种特殊手段保证了 effect 在绘制后触发，其使用了 MessageChannel 的方式，结合 requestAnimationFrame 达到绘制完成后触发 effect 函数，effect 适合执行无 dom 依赖，不阻碍主线程渲染的副作用，如网络请求，事件绑定等。

对于清除副作用，每个 effect 都返回一个清除函数。执行的时机是在执行当前 effect 之前对上一个 effect 进行清除。除了每次更新会清除，在组件卸载的时候也会清除

第二个参数为依赖列表，那么 React 是如何知道依赖列表的值发生的改变呢？其通过 Object.is 进行元素前后的比较。其比较的是对象的引用。可以使用社区的 use-deep-compare-effect 方式进行比较

useEffect 特性

1. Capture Value 特性
2. async 函数特性，由于 async 返回的是 promise，和 effect 函数返回的 clean 函数冲突
3. 空数组依赖

useLayoutEffect，都可从 DOM 中获取其变更后的属性。layoutEffect 运行是同步的，影响浏览器的渲染

useRef，不仅可以保存对应的 dom，current 属性还能保存一些数据，数据变化时 dom 不刷新

useMemo 用来缓存某些函数的返回值。使用缓存避免每次渲染都重新执行相关函数。
useCallback，等同于 useMemo(() => fn,deps)
useContext，调用了 useContext 的组件总会在 Context 值发生变化的时候重新渲染

自定义 hook，hooks 的基本准则，不能在条件循环中使用，不能在普通函数中使用

forwardRef 可以对 ref 进行传递

React.memo 可以对组件进行缓存，第二个参数可以决定是否渲染。不可以把 react.memo 放在组件渲染过程中

## 认识 React Router

Route Link history

StaticRouter 为静态路由，也称为无状态路由。通常在服务端 Nodejs 中使用。特征是在开发阶段或者程序运行前就已经确定
MemoryRouter 和 NativeRouter 在测试场景或 ReactNative 中使用较多

## Router 源码解析

三部分：history 监听，提供初始 context 以及提前监听

```JavaScript
// history监听
this.unlisten = props.history.listen(location => {
    this.setState({ location })
})
// context
import React from 'react'
import RouterContext from './RouterContext'
class Router extends React.Component {
    static computeRootMatch(pathname) {
        return { path: '/', url: '/', params: {}, isExact: pathname === '/' }
    }
    constructor(props) {
        super(props)
        this.state = {
            // 初始location从history中获得
            location: props.history.location
        }
        this._isMounted = false
        this._pendingLocation = null
        if(!props.staticContext) {
            this.unlisten = props.history.listen(location => {
                if(this._isMounted){
                    // 仅在挂在成功后进行设置，防止出现未挂在成功而props.history.location已经变化的情况
                    this.setState({
                        location
                    })
                }else{
                    // 先将最近的location变化挂载
                    this._pendingLocation = location
                }
            })
        }
        componentDidMount() {
            // 用于记录组件是否挂载成功
            this._isMounted = true
            if(this._ispendingLocation){
                // 将挂载的location设置到state.location中
                this.setState({
                    location: this._pendingLocation
                })
            }
        }
        componentWillUnmount() {
            // unmount在销毁时应该取消监听
            if(this.unlisten) this.unlisten()
        }
        render() {
            return (
                <RouterContext.Provider children={this.props.children || null} value={{
                    history: this.props.history,
                    // state.location发生了变化，触发重新渲染
                    location: this.state.location,
                    match: Router.computeRootMatch(this.state.location.pathname),
                    staticContext: this.props.staticContext
                }} />
            )
        }
    }
}
export default Router

// RouterContext
import createContext form 'mini-create-react-context'
const createNamedContext = mame => {
    const context = createContext()
    context.displayName = name
    return context
}
const context = /* #__PURE__*/ createNamedContext('Router')
export default context
```

- useRouterContext：获得组件树中距离当前组件最近的 RouterContext 值，可获得 history、location、match
- useHistory：获取 history 对象
- useLocation：获取 location 对象

## Route

两个基本要素，path 和组件渲染方式。path 表示用来匹配何种浏览器路径，component 用来定义匹配成功后渲染哪个组件。匹配并渲染

- path：基本路径格式字符串、正则表达形式、综合形式，符合 Express.js 的路径声明风格，可传入数组。对于():\*\需要转义，否则识别为正则符号，如果没有传入 path，那么其视为匹配成功
- 组件渲染方式：component 方式，render 方式
- children 属性渲染：最高自由度，即便路由 path 没有 match，如果 child 不是函数，为 react 组件，则命中路由后才做对应的渲染。

渲染优先级：children，component，render

对于 Route 组件的三个参数说明

- match 对象：params（路由匹配出的键值对），isExact（是否完全匹配），path（匹配到的路由的 path 属性），url（真实的 url 匹配命中的部分）
- location 对象：key 在 hash 路由下不存在，为 null，在浏览器路由中为一个随机值。pathname 为 url 路径的 path 部分，search 为 query 部分，包括？，hash 为 hash 部分，包括#，state 对象，持久化存储状态。注意 hash 路由的 pathname 是从#后开始计算路由，解析 query 可以通过 new URLSearchParams(location.search)进行格式化
- history 对象：建议每次使用 props 的 location 对象，而非 history 的 location 对象

Route 的其它配置

1. location，允许自行传入而不是使用上下文中的 location
2. exact，路由配置是否完全匹配
3. strict，路由严格模式
4. senssitive，路由大小写是否敏感，默认不敏感

整体流程如下

![router流程](https://gitee.com/RenYaNan/wx-photo/raw/master/2021-6-22/1624329218023-%E5%9B%BE%E7%89%87.png)

useRouteMatch，获取某路径的路由匹配情况或者获取上下文中的 match 命中情况
useLocation，获取当前的 URL 路径
useParams，获取上下文中的命名参数匹配结果，源码为使用 useContext 获得上下文中的 match，路由未命中时 match 为 null，命中之后返回 match.params
RouterContext 上下文跨组件获取组件树中距离本级组件最近的 Router 的匹配信息
IndexRedirect 组件的作用是在父路由命中的情况下，将父路由重定向到一个新的路由，在 Router4.x 版本以后被取消了，但是自己可以模拟实现

路由缓存问题：1. 状态存在 redux 等内存中 2. 不销毁 dom 节点 3.渲染优化

```JavaScript
// CacheRoute
import * as React form 'react'
import { Route, RouteChildrenProps, RouteProps } from 'react-router'
import { omit } from 'lodash' // 忽略对象中某些属性
import MemoChildrenWithRouteMatch, { MemoChildrenWithRouteMatchExact } from './MemoChildrenWithRouteMatchExact'
// 文件内容如下MemoChildrenWithRouteMatchExact start
interface Props extends RouteChildrenProps {
    children?: any;
}
function MemoChildrenWithRouteMatch(props: Props) {
    return props.children
}
export default React.memo(MemochildrenWithRouteMatch, (prvious, nextProps) => {
    // 不命中就不渲染，仅在match有值才渲染
    return !nextProps.match
})
function MemoChildrenWithRouteExactMaatch(props: Props){
    return props.children
}
export const MemoChildrenWithRouteMatchExact = React.memo(MemoChildrenWithRouteExactMatch, (prvious, nextProps) => !(nextProps.match && nextProps.match.isExact))
//MemoChildrenWithRouteMatchExact end

import Remount from './Remount'
// 文件内容如下Remount start
export default function Remount(props) {
    // 初始化key
    const keyRef = React.useRef(Math.random() + Date.now())
    if(props.shouldRemountComponent) {
        keyRef.current = Math.random() + Date.now()
    }
    return React.cloneElement(React.children.only(props.children), {
        key: keyRef.current
    })
}
// Remount end

interface Props {
    forceHide?: boolean; // 强制隐藏
    shouldRemount?: boolean; // 是否需要销毁组件并重新渲染dom
    shouldDestroyDomWhenNotMatch?: boolean; // dom渲染模式为销毁模式
    shouldMatchExact?: boolean; // 判断组件缓存时是全匹配还是模糊匹配
}

export default function CacheRoute(props: RouteProps & Props) {
    const routeHadRenderRef = React.useRef(false)
    return (
        <Route
            {...omit(props, 'component', 'render', 'children')}
            children={
                (routerProps: RouteChildrenProps) => {
                    const Component = props.component
                    // 获取Route命中结果
                    const routeMatch = routeProps.match
                    let match = !!routeMatch
                    if(props.shouldMatchExact){
                        match = routeMatch && routeMatch.isExact
                    }
                    if(props.shouldDestroyDomWhenNotMatch){
                        if(!match) routeHadRenderRef.current = false
                        // 按react-router包中的Route逻辑
                        if(props.render){
                            return match && props.render(routeProps)
                        }
                        return (
                            match && Component && React.createElement(Component, routeProps)
                        )
                    }else{
                        const matchStyle = {
                            // 隐藏
                            display: match && !props.forceHide ? 'block' : 'none'
                        }
                        if(match && !routeHadRenderRef.current){
                            // 将渲染标记设置为true
                            routeHadRenderRef.current = true
                        }
                        let shouldRender = true
                        if(!match && !routeHadRenderRef.current){
                            shouldRender =false
                        }
                        // 选择对应的memo
                        const MemoCache = props.shouldMatchExact ? MemoChiildrenWithROuteMaatchExact : MemeChildrenWithRouteMatch
                        // css隐藏保留dom
                        let component
                        if(props.render){
                            component = props.render(routeProps)
                        }else{
                            component = <Component {...routeProps} />
                        }
                        return (
                            shouldRender && (
                                {/* 提供css属性 */}
                                <div style={matchStyle}>
                                    {/* 提供remount能力 */}
                                    <Remount shouldRemountComponent={props.shouldRemount}>
                                        <MemoCache {...routeProps}>
                                            {component}
                                        </MemoCache>
                                    </Remount>
                                </div>
                            )
                        )
                    }
                }
            }
        >
        </Route>
    )
}
```

## Link

Link 要从 react-router-dom 中引入，其 props 定义为

```javascript
export interface LinkProps<S = H.LocationState>
  extends React.AnchorHTMLAttributes<HTMLAnchorElement> {
  to: H.LocationDescriptor<S>;
  replace?: boolean;
  innerRef?: React.Ref<HTMLAnchorElement>;
}
```

如果不需要 a 标签发送 Referer 字段，可以设置 rel="noreferrer"

NavLink，带激活状态的 Link

```javascript
export interface NavLinkProps<S = H.LocationState>
  extends LinkProps<S> {
  activeClassName?: string;
  activeStyle?: React.CSSProperties;
  exact?: boolean;
  strict?: boolean;
  isActive?<Params extends { [K in keyof Params]?:string }>(
      match: match<Params>,
      location: H.Location<S>,
  ): boolean;
  location?: H.Location<S>;
}
```

## 其它路由组件及方法

- Switch 路由匹配组件，跟 Route 结合使用

## 之后必备

- 结构化克隆算法
- loadhash
