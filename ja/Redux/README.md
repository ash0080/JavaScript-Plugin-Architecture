# Redux

>本文基于[Redux][] [3.5.2](https://github.com/reactjs/redux/releases/tag/v3.5.2 "3.5.2")版本.

[Redux][]是[React](https://github.com/facebook/react "React")的状态管理工具。

Redux的架构和[Flux](https://facebook.github.io/flux/ "Flux")类似.所以你可以先了解一下Flux.

Redux建立在三原则的基础上[Three Principles](http://cn.redux.js.org/docs/introduction/ThreePrinciples.html "Three Principles | Redux")(以下简称 三原则)

- 单一数据源
    - 整个应用的state存于唯一一个状态树
- State 只读
    - State仅可通过触发action来修改
- 使用纯函数执行修改
    - 接收Action,修改State的纯函数称为Reducer
    
要详细了解三原则, 可查看以下文档.译者:已替换为中文文档链接

- [Read Me | Redux](http://redux.js.org/)
- [Getting Started with Redux - Course by @dan_abramov @eggheadio](https://egghead.io/series/getting-started-with-redux)

在深入Redux之前,我们先说明一点,Redux也使用 _middleware_ 进行插件扩展, _middleware_ 基于三原则.

在[Connect](../connect/README.md)中我们讲过 _middleware_, Redux的 _middleware和 Connect的 _middleware既有相同之处,又有不同之处,
接下来让我们来看一下Redex的 _middleware_ 实现机制.

## 如何写(插件)?

简要描述Redex的流程如下:

- 表示(数据)操作的对象,称为Action
    - 基本上是一个command设计模式
- 接收Action,实际去修改State的函数是Reducer
    - Reducer需事先注册到Store
- Dispatch(`store.dispatch(action)`)Action,会通知Reducer

Redux的例子,见以下代码

[import, redux-example.js](../../src/Redux/redux-example.js)

1. call `createStore`方法,加载`logger`和`crashReporter`中间件
2. 创建Store对象,并注册Reducer
3. (Store数据的变更)Action被dispatch(派发)
4. Reducer接收Action的通知并返回新的State
5. State变更发出通知

流程就是这样

_middleware_ 存在于上述步骤的3和4之间

```
dispatch(action) -> (_middleware_ 处理) -> reducer返回新的State -> (如果State发生了改变) -> 执行所有通过subscribe注册了的回调
```

![Redux flow](./img/redux-unidir-ui-arch.jpg)

via [staltz.com/unidirectional-user-interface-architectures.html](http://staltz.com/unidirectional-user-interface-architectures.html)

接下来,我们将看看 _middleware_ 能做些什么(扩展)

## middleware

在Redux里,我们称 _middleware_ 是一种供第三方扩展的机制

- [Middleware | Redux](http://cn.redux.js.org/docs/advanced/Middleware.html "Middleware | Redux")

我们来通过一个实际的例子来学习 _middleware_ 是如何扩展的. 这个 _middleware_  会log出action派发前后Store内的数据变化. 

[import, logger.js](../../src/Redux/logger.js)

这个 _middleware_ 可以通过以下方式加载.

```js
import {createStore, applyMiddleware} from "redux";
const createStoreWithMiddleware = applyMiddleware(createLogger())(createStore);
```
这里看上去 _middleware_ 加载(apply)到`store`对象, 而实际上,它是加载到`store.dispatch`对象,并且,创建了一个继承(extend) `dispatch`的方法

通过这种方式,当`dispatch`被执行时,_middleware_也会被执行.这就是Redux里 _middleware_ 的扩展要点.

```js
store.dispatch({
    type: "AddTodo",
    title: "Todo title"
});
```

我们看一下`logger.js`

```js
export default function createLogger(options = defaultOptions) {
    const logger = options.logger || defaultOptions.logger;
    return store => next => action => {
        logger.log(action);
        const value = next(action);
        logger.log(store.getState());
        return value;
    };
}
```

`createLogger`将options传递给logger
`return`返回一个高阶函数(多层包裹的callback), 这就是 _middleware_ 的主体部分

```js
const middleware = store => next => action => {};
```

上面这串=>function第一眼看上去难以理解,
我们可以把它展开:

```js
const middleware = (store) => {
    return (next) => {
        return (action) => {
            // Middlewareの処理
        };
    };
};
```

简单来说,高阶函数就是返回函数的函数.

让我们再看一眼`logger.js`, 你会发现,在next(action)方法调用的前后,都调用了一个.log方法;

[import, logger.js](../../src/Redux/logger.js)

这个 _middleware_ 的执行流程可参考下图

![dispatch-log.js flow](./img/dispatch-log.js.png)

在此例中, `next`可以理解为就是`dispatch`,不过如果存在多个 _middleware_ 的情况下,就不一定了,只能理解为 **下一个** _middleware_

Redux的 _middleware_ 机制其实非常简单,但因为这是一种不常见的设计,所以看起来很复杂.

为了让我们能够实现相似的架构,让我们深入学习一下Redux的 _middleware_

## 它是如何工作的?

_middleware_ 是`dispatch`的封装, 但是`dispatch`在这里做了些什么呢?

简要来说, Redux的`store.dispatch(action)`派发action执行(call)`store.subscribe(callback)`注册了的callback回调

这是一种典型的 Pub/Sub 设计模式, 这次让我们实际实现一下这一设计模式.

### Dispatcher

就像[ESLint](../ESLint/README.md)使用EventEmitter,这里我们用`dispatch`和`subscribe`来实现`Dispatcher`

[import, Dispatcher.js](../../src/Redux/Dispatcher.js)

`Dispatcher`用于`dispatch`(派发)一个Action对象, 并调用通过`subscribe`注册了的回调函数

需要特别说明的是,我们这里`Dispatcher`的实现与Redux实际并不相同,这里仅仅作为一个帮助我们理解原理的例子.

> Unlike Flux, Redux does not have the concept of a Dispatcher.
> This is because it relies on pure functions instead of event emitters
> -- [Prior Art | Redux](http://redux.js.org/docs/introduction/PriorArt.html "Prior Art | Redux")

### applyMiddleware

接下来, 我们要实现添加 _middleware_ 的 `applyMiddleware`方法.
如前面所述, _middleware_ 是`dispatch`的扩展.

`applyMiddleware`接收`dispatch`和 _middleware_ 两个参数, 返回 _middleware_ extend 后的 `dispatch` 

[import, apply-middleware.js](../../src/Redux/apply-middleware.js)

这里的`applyMiddleware`和Redux相同. 我们接着来实现适用于 _middleware_ 的`dispatch`方法(译注:applyMiddleware和Redux的实现相同, dispatch和Redux实现实际是不一样的)

[import, apply-middleware-example.js](../../src/Redux/apply-middleware-example.js)


通过`applyMiddleware`注册 _middleware_时添加的`timestamp`,应用于 _middleware_ 的Action.
当每次`dispatchWithMiddleware(action)`时,会自动添加时间戳

```js
const dispatchWithMiddleware = applyMiddleware(createLogger(), timestamp)(middlewareAPI);
dispatchWithMiddleware({type: "FOO"});
```

这里, _middleware_ 被传入一个拥有两个方法的`middlewareAPI`对象,
其中,`getState`是只读方法, _middleware_ 并不直接修改State内容,另一个`dispatch`方法可以修改Action对象, 但只能够返回`dispatch`

如此, 我们令 _middleware_ 满足了三原则 

- State is read-only
    - State 只能通过Action来修改

_middleware_ 本身和[Connect](../connect/README.md)中的实现非常相似, 只是 _middleware_ 不能直接修改结果(State)

Connect中的 _middleware_ 能够修改最终结果(`response`)
也可以说Redux _middleware_ 的不同之处在于其作用范围仅限于「从`dispatch`到Reducer为止」

## 适用于哪些场合?

Redex的 _middleware_ 本身是基于三原则的一种实现.
_middleware_ 可以自由的修改(或者不修改)Action对象.
但 _middleware_ 不能直接修改State.

很多插件构架都带有权限限制, Redex的 _middleware_ 也限制了写入权限

供 _middeleware_ 调用的API方法`getState()`和`dispatch()`方法均限制了写入.

通过这种限制了写入权限的方式, 实现了三原则的约定.

## 不适用于哪些场合?

另一方面, 因为要限制插件的写入权限,必须在插件间传递中间数据.

在Redux中,为了实现以Action对象形式出现的指令(command)操作,需要一个机制,让Reducer方法生成新的State.

也就是说, 插件本身并不能完成全部的处理工作.
得到插件处理的结果的同时,还需实现结果的实际操作(译注:指用Reducer改写state)

Redux里,很多处理都是以 _middleware_ 形式实现的.
这样插件和具体实现(译注:改写State)会非常紧密

因此,这并不是一种能够完全依赖插件完成全部工作的机制.

## 总结

本章我们学习了[Redux][] plugin的建构方式,可归纳为:

- Redux _middleware_ 主要针对Action对象写(代码)
- _middleware_ 满足三原则
- _middleware_ 的功能限制容易实现
- _middleware_ 本身并不能完成全部的工作

## 参考

- [Middleware | Redux](http://redux.js.org/docs/advanced/Middleware.html)
- [10. Middleware · happypoulp/redux-tutorial Wiki](https://github.com/happypoulp/redux-tutorial/wiki/10.-Middleware)
- [Brian Troncone – Redux Middleware: Behind the Scenes](http://briantroncone.com/?p=529)
- [ReduxのMiddlewareについて理解したいマン | moxt](https://hogehuga.com/post-1123/)
- [Understanding Redux Middleware — Medium](https://medium.com/@meagle/understanding-87566abcfb7a#.8fr4jmjwz)

[Redux]: https://github.com/reactjs/redux  "reactjs/redux: Predictable state container for JavaScript apps"


