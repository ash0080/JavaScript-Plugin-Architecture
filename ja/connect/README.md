# Connect

> 本文基于[Connect](https://github.com/senchalabs/connect "Connect") 3.4.0 版本.

[Connect](https://github.com/senchalabs/connect "Connect")是Node.js的一个HTTP Server Framework, 使用 _middleware_ (中间件)作为其展机制,connect本体只提供很少的功能。

本章我们来看看connect的 _middleware_ 原理

## 如何写(插件)?

让我们用Connect写一个简单的Echo server.
Echo server根据发送的请求返回内容作为响应.

[import, connect-echo-example.js](../../src/connect/connect-echo-example.js)

当发送如下request body到Echo server, server将返回原值作为响应.

```json
{
    "key": "value"
}
```

通过调用`app.use(middleware)`, `request`和`response`对象被传入 _middleware_.
`request`和`response`在 _middleware_ 进行处理、可以log或返回任意的响应内容.

在Echo server中, request和response以`req.pipe(res)`这种形式实现串流.

### 写一个middleware模组

让我们进一步了解下 _middleware_ 是如何实现为一个插件模组的。

在以下[connect-example.js](#connect-example.js)这个例子中、对应每一次的request请求,服务器返回一个由
`"response text"`(响应体)以及`"X-Content-Type-Options"`(响应头)组成的response.

不同功能的处理过程被分别实现为独立的 _middleware_ 文件, 并通过 `app.use(middleware)` 加载使用.

[import nosniff.js](../../src/connect/nosniff.js)

[import hello.js](../../src/connect/hello.js)

[import errorHandler.js](../../src/connect/errorHandler.js)

[import connect-example.js](../../src/connect/connect-example.js)

基本上任何 _middleware_ 都能用 `app.use(middleware)`这种扩展(app), _middleware_ 的模组化使得它可以非常容易地实现复用.

> **Note** 如果 _middleware_ 传入4个参数, 我们称这是个Error Handler _middleware_ , Connect对其有特别规定.

## 它是如何工作的?

让我们看看 Connect的 _middleware_ 是如何工作的.

为了在request请求发生时,能够调用`app`中注册(.use)的 _middleware_ ,我们可以猜想,首先在app中需要有个机制能够保存这些 _middleware_

Connect使用一个`app.stack` 数组来保存所有的 _middleware_,

下面我们展示了`app.stack`的内容,你可以看到 _middleware_ 是按照注册(.use)的顺序来存储的.

[import connect-trace-example.js](../../src/connect/connect-trace-example.js)

当server接收到request请求时, Connect按照一定顺序来调用 _middleware_ .

在上面的例子中, _middleware_ 按以下顺序被调用.

- nosniff
- hello
- errorHandler

执行过程中, errorHandler只在 _middleware_ 处理过程中发生错误时被调用(前面提到,Error Handler有特别规定)

所以(除去错误处理部分) 程序按照[nosniff.js](#nosniff.js) → [hello.js](#hello.js)的顺序执行。

[import nosniff.js](../../src/connect/nosniff.js)

`nosniff.js`在设置了HTTP响应头之后调用`next()`方法, 这里`next()`意味着执行下一个 _middleware_ 

接下来 `hello.js` 中并没有 `next()`

[import hello.js](../../src/connect/hello.js)

`hello.js`没有继续调用`next()`方法,这意味着它是 _middleware_ 执行序列的最后一个,这时,就算我们早前(在app中)注册了其他的 _middleware_ ,也将被略过.

也就是说, 整个过程是从stack里一个个取出 _middleware_ 依次执行直到结束.

Connect的处理机制可以抽象为以下的代码.。

```js
let req = "...",
    res = "...";
function next(){
    let middleware = app.stack.shift();
    // next是下一个middleware的回调函数
    middleware(req, res, next);
}
next();
```

我们称这种 _middleware_ 的串联形式 叫做 _middleware stack_ 

由 _middleware stack_ 这种形式组成的HTTP server, 还有Python的 WSGI middleware 和 Ruby的 Rack

Connect `use` _middleware_ 的方式和Rack相同,实际上它(Connect)就是参考Rack而来.

- [Ruby - Rack解説 - Rackの構造とRack DSL - Qiita](http://qiita.com/higuma/items/838f4f58bc4a0645950a#2-5 "Ruby - Rack解説 - Rackの構造とRack DSL - Qiita")

接下来,让我们尝试将之前的抽象代码转换为实际代码

## 模仿实现

让我们创建一个叫做Junction的类来实现与Connect类似的 _middleware_ 插件机制

Junction是一个拥有`use(middleware)` 和`process(value,(error, result) => {} );` 两个方法的类.

[import junction.js](../../src/connect/junction.js)

看看实现代码, `use`方法用于注册 _middleware_ `process`按顺序执行注册的 _middleware_ ,
`Junction`本身并不处理传入的数据, 数据的处理完全交由 _middleware_ 进行.

注册的 _middleware_ 和 Connect 一样, 通过调用`next` 可以将结果传递给下一个 _middleware_ .

这里我们传入的参数和Connect有所不同,不过使用方式是一样的.

[import junction-example.js](../../src/connect/junction-example.js)


## 适用于哪些场景?

Connect和Junction这样的架构,让我们可以在 _middleware_ 中实现具体的事务处理,
而本体(Connect本身)可以非常精简,只需要向 _middleware_ 提供相应的接口和错误处理.
尽管本文没有介绍,Connect还提供routing(路由)相关功能. 路由的实现原理也很简单, 「注册 _middleware_ ,仅对匹配指定路径的请求作出响应」.

```js
app.use("/foo", function fooMiddleware(req, res, next) {
    // req.url starts with "/foo"
    next();
});
```

这种架构,对于存在输入输出的情况,核心程序可以非常精简.
对于像Connect或Rack这种「接收请求,返回响应」的HTTP server, 这种架构是非常合适的.

## 不适用于哪些场景?

在这种架构中,具体的事务处理是在 _middleware_ 中实现, 如果 _middleware_ 的功能较为复杂, _middleware_ 之间可能会产生相互依赖关系,
这种情况下,`use(middleware)`注册 _middleware_可能需要按照一定顺序. 而这中间的依赖关系,必须由用户去解决.
译注: _middleware_ 间相互依赖并不是错.在实际使用中,这种情况常常会发生,例如bodyparser中间件.唯一的问题在于,当相互依赖发生时,需要用户自行去管理依赖.

如果两个插件之间存在强依赖关系,或者非常明确的依赖关系,这种架构就不合适了.译注:这种情况下,干脆把两个插件写成一个比较好.

## 生态

Connect本体十分轻量, 但是有大量的 _middleware_ 可供使用.

- [github.com/senchalabs/connect#middleware](https://github.com/senchalabs/connect#middleware)
- [Express middleware](http://expressjs.com/resources/middleware.html "Express middleware")

各种各样的 _middleware_ 都是很小的单一功能,多数情况下,我们会联合数个 _middleware_ 来一起使用.
_middleware_ 按层堆叠在一起, 多数情况下以 _middleware_ stack 这样的形式出现.  

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

本章我们学习了Connect 的plugin构建方式,可归纳为:

- Connect是一个使用 _middleware_ 的HTTP Server库
- Connect本身很简单
- 多个 _middleware_ 可以组合在一起使用,用来创建一个HTTP服务

## 参考资料

- [Ruby - Rack解説 - Rackの構造とRack DSL - Qiita](http://qiita.com/higuma/items/838f4f58bc4a0645950a#2-5)
- [Pylons Concept — Pylons 0.9.7 documentation](http://docs.pylonsproject.org/projects/pylons-webframework/en/latest/concepts.html)


