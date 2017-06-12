# Connect

> 本文基于[Connect](https://github.com/senchalabs/connect "Connect") 3.4.0 版本.

[Connect](https://github.com/senchalabs/connect "Connect")是Node.js的一个中间件HTTP服务框架.
 _middleware_ (中间件)是一种扩展机制, connect本身只提供很少的一些功能。

本章我们来看看connect的 _middleware_ 原理

## 如何写(插件)?

让我们用Connect写一个简单的Echo server.
Echo server 能够根据发送的请求返回内容作为响应.

[import, connect-echo-example.js](../../src/connect/connect-echo-example.js)

当你发送如下的请求体到Echo server, server会返回相同的值作为响应.

```json
{
    "key": "value"
}
```

通过`app.use(middleware)` 这种形式, `request`和`response`对象被传入_middleware_.
`request`和`response`在_middleware_ 处理后、可以log或返回任意的响应内容.

Echo server中, request 和 response 以 `req.pipe(res)`这种形式实现.

### 写一个middleware模组

让我们进一步了解下 _middleware_ 插件模组是如何实现的。

在[connect-example.js](#connect-example.js)这个例子中、对应每一次的request请求,`"response text"`响应以及`"X-Content-Type-Options"`响应头被server返回.

每一个 _middleware_ 组件被分别加载,通过 `app.use(middleware)` 添加至整个处理进程中.

[import nosniff.js](../../src/connect/nosniff.js)

[import hello.js](../../src/connect/hello.js)

[import errorHandler.js](../../src/connect/errorHandler.js)

[import connect-example.js](../../src/connect/connect-example.js)

一般来说, 任何 _middleware_ 都能够以`app.use(middleware)`这种形式进行组装, _middleware_ 模组形式令其可以非常容易的实现复用.

> **Note** 如果 _middleware_ 引入4个参数, 其中一个是Connect特别规定的 error handler.

## 它如何工作的?

让我们看看 Connect的 _middleware_ 是如何工作的.

`app`中注册的 _middleware_ 可以在request时调用.

Connect用一个`app.stack` 数组来管理所有的 _middleware_

下面我们展示了`app.stack`的内容,你可以看到 _middleware_ 是按照注册顺序来存储的.

[import connect-trace-example.js](../../src/connect/connect-trace-example.js)

当server收到request时, Connect按照顺序来调用 _middleware_ .

在上面的例子中, _middleware_ 按一下顺序被调用.

- nosniff
- hello
- errorHandler

执行过程中, _middleware_ errorHandler只在错误发生时被调用

所以,一般来说按照 [nosniff.js](#nosniff.js) → [hello.js](#hello.js) 的顺序调用。

[import nosniff.js](../../src/connect/nosniff.js)

`nosniff.js`在设置了HTTP header之后调用`next()`方法, 这里`next()`意味着执行下一个 _middleware_ 

接下来 `hello.js` 中并没有 `next()`

[import hello.js](../../src/connect/hello.js)

缺少`next()`表明`hello.js`是 _middleware_ 执行序列的最后一个, 这时,就算我们早前注册了其他的 _middleware_ 也将被忽略掉.

也就是说, 处理的顺序是从stack的头部一个个依次取出.

Connect的处理机制可以抽象为以下的代码.。

```js
let req = "...",
    res = "...";
function next(){
    let middleware = app.stack.shift();
    // nextが呼ばれれば次のmiddleware
    middleware(req, res, next);
}
next();// 初回
```

这一串联 _middleware_ 的方式有时被称作 _middleware stack_ 

与 _middleware stack_ 这种 HTTP server构成方式类似的, 在Python中有WSGI中间件, Ruby有Rack

实际上Connect的这种 _middleware_ `use`方式和Rack相似,实际上它就是参考Rack来实现的.

- [Ruby - Rack解説 - Rackの構造とRack DSL - Qiita](http://qiita.com/higuma/items/838f4f58bc4a0645950a#2-5 "Ruby - Rack解説 - Rackの構造とRack DSL - Qiita")

接下来,让我们尝试将之前的抽象代码具体实现

## 模仿实现

让我创建一个叫做Junction的类来实现与Connect类似的 _middleware_ 

Junction是一个简单类, 拥有`use(middleware)` 和`process(value,(error, result) => {} );`两个方法.

[import junction.js](../../src/connect/junction.js)

看看实现代码, `use`方法用于注册 _middleware_ `process`按顺序执行注册的 _middleware_ 
因此, `Junction`本身并不处理传入的数据, 数据的处理完全依赖于 _middleware_ .

注册的 _middleware_ 和 Connect 一样, 调用`next` 交给下一_middleware_ 进行处理.

这里我们传入的参数和Connect有所不同,不过使用方式是一样的.

[import junction-example.js](../../src/connect/junction-example.js)


## 适用于哪些场合?

通过Connect和Junction这样的架构,我们可以在 _middleware_ 中实现方法的细节

因此,程序本体可以十分简单, 仅仅需要实现供 _middleware_ 使用的 interface以及 error handling.

尽管这次我们没有展开介绍, Connect还提供routing功能.这一功能的实现也很简单, 「注册 _middleware_ ,对与给定路径匹配的请求作出响应」.

```js
app.use("/foo", function fooMiddleware(req, res, next) {
    // req.url starts with "/foo"
    next();
});
```

这种架构展示了,如果存在输入输出的情况下, 程序核心可以十分简单.

因此, 像Connect或Rack这种「对请求作出响应」的HTTP server, 这种架构是非常合适的.

## 不适用于哪些场合?

在这种架构中, 程序的功能细节在 _middleware_ 中实现, 但如果你要在 _middleware_ 中实现复杂的功能, 可能 _middleware_ 间会产生相互依赖.

在此例中, `use(middleware)`注册 _middleware_ 可以存在不同的顺序.而这中间的依赖关系,必须由用户去解决.

因此, 如果你的插件存在强依赖,或者明确的依赖关系, 这种架构可能就不合适了.

因此, _middle stack_ 被引入来解决这些问题. 

## 生态

Connect本体十分轻量, 但是有大量的 _middleware_ 可供使用.

- [github.com/senchalabs/connect#middleware](https://github.com/senchalabs/connect#middleware)
- [Express middleware](http://expressjs.com/resources/middleware.html "Express middleware")

每一个 _middleware_ 都是很小的单一功能, 多数情况下, 我们会联合数个 _middleware_ 来一起使用.

_middleware_ 按层堆叠在一起, 常常以 _middleware_ stack 这样的形式出现.  

![pylons_as_onion](img/pylons_as_onion.png)

> middleware的概念结构上很像洋葱
> 引用于 [WSGI middleware](http://docs.pylonsproject.org/projects/pylons-webframework/en/v1.0.1rc1/concepts.html#wsgi-middleware "WSGI middleware")


## 使用这种机制的案例

- [Express](http://expressjs.com/ "Express")
    - 与 Connect 的 _middleware_ 互相兼容 
    - 之前直接使用Connect, 不过自[4.0.0](https://github.com/strongloop/express/blob/4.0.0/History.md "4.0.0")改为自行实现
- [wooorm/retext](https://github.com/wooorm/retext "wooorm/retext")
    - 使用`use`注册middleware的自然语言处理器
- [r7kamura/stackable-fetcher](https://github.com/r7kamura/stackable-fetcher "r7kamura/stackable-fetcher")
    - 使用`use`注册middele的HTTP客户端库

## 总结

本章我们学习了Connect 的plugin构建方式

- Connect是 一个使用 _middleware_ 的 HTTP Server 库
- Connect 本身很简单
- 多个 _middleware_ 可以组合在一起使用, 以创建一个HTTP服务器

## 参考资料

- [Ruby - Rack解説 - Rackの構造とRack DSL - Qiita](http://qiita.com/higuma/items/838f4f58bc4a0645950a#2-5)
- [Pylons Concept — Pylons 0.9.7 documentation](http://docs.pylonsproject.org/projects/pylons-webframework/en/latest/concepts.html)


