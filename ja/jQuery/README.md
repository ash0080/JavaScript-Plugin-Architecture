# jQuery

> 本文基于[jQuery](http://jquery.com/ "jQuery") 2.1.4版本

jQuery允许我们通过`$.fn`,给jQuery对象添加带有返回值`$()`的method来扩展jQuery.

如下例 `greenify` 插件, 让我们能够 call `$(document.body).greenify();` 

[import, greenify.js](../../src/jQuery/greenify.js)

在实际应用场合, 我们必须在载入`greenify.js`之前,先载入`jquery.js`

```html
<script src="jquery.js"></script>
<script src="greenify.js"></script>
```

## 它如何工作的?

首先让我们看看jQuery的插件是如何工作的

jQuery的插件写法如`$.fn.greenify = function (){}`,很像prototype扩展的方式. 

我们看一下`jQuery.fn`的具体实现、实际就是`jQuery.prototype`的封装,所以这就是一种prototype扩展.

```js
// https://github.com/jquery/jquery/blob/2.1.4/src/core.js#L39
jQuery.fn = jQuery.prototype = {
    // prototype的实现
};
```


`$()`内部返回一个`new` jQuery 对象、这个jQuery 对象允许使用prototype扩展方法.

```js
$(document.body); // 返回值是 jQuery instance
```

也就是说, 你可以认为jQuery插件就是直接使用prototype.

## 适用于哪些场合?

我们了解了jQuery插件的实现原理,现在来考虑一下这种方式适用于哪些场合.

首先,因为这是prototype的简单封装,所以适用于prototype同样的场合.
其次,因为可以动态添加methods,所以你可以添加例如monkey patch来覆盖已存在的插件方法.

## 不适用于哪些场合?

与prototype相似, prototype扩展方式过于灵活,jQuery本身几乎无法约束这些插件的行为.

另外,扩展依赖于jQuery的实现,插件很可能因为jQuery的版本更新而不可用.

尽管jQuery制定了不动undocumented APIs的规则,但也不总是被严格遵守.

## 模仿实现

我们现在来写一个可扩展的`calculator`计算器,来实现jQuery plugin类似的机制.

`calculator` 代码如下

[import, calculator.js](../../src/jQuery/calculator.js)

与`$.fn`相似,我们使用`fn`作为`prototype`的别名

```js
calculator.fn = calculator.prototype;
```

我们在此声明了一个`calculator(初始值)`的稍微有点特殊的构造器,但我们暂未加入任何扩展功能.

[calculator.js](#calculator.js)接下来,我们要给计算器加入四则运算操作.

[import, calculator-plugin.js](../../src/jQuery/calculator-plugin.js)

在[calculator-plugin.js](#calculator-plugin.js)里,我们通过`calculator.fn.add`的方式添加一个`add`方法.

这里,module的依存关系与jQuery相同,我们需要先引用[calculator.js](#calculator.js),然后再引用[calculator-plugin.js](#calculator-plugin.js)

```html
<script src="calculator.js"></script>
<script src="calculator-plugin.js"></script>
```

通过这种方式,我们现实现了例如`calculator#add`这样的方法,所以,你可以这样写:

[import, calculator-example.js](../../src/jQuery/calculator-example.js)

现在,我们通过实例理解了使用javascript的`prototype`进行扩展的这种方法,这并不需要什么特殊的实现手段. 我们也可以简单总结为一句话,「用`calculator.fn`替代`calculator.prototype`来进行扩展」

## 生态

这种插件机制依赖于确定的global对象,应用的前提是要能通过`<script>`标签引用脚本.

```html
<script src="jquery.js"></script>
<script src="greenify.js"></script>
```

这种实现方法在CommonJS或ES6模组的形式下不能使用,因此在Node.js生态里是不存在的.也可以说,这种机制有点不适合CommonJS或ES6.

## 总结

本章我们学习了jQuery plugin 的建构方式:

- jQuery plugin 通过 `jQuery.fn` 扩展
- `jQuery.fn` 等同于 `jQuery.prototype`
- jQuery plugin 是 `jQuery.prototype` 的一种衍生
- 很难控制插件的行为,因为你可以做任何事情,非常自由

## 参考资料

- [Plugins | jQuery Learning Center](https://learn.jquery.com/plugins/ "Plugins | jQuery Learning Center")
- [jQuery拡張の仕組み 〜 JSおくのほそ道 #013 - Qiita](http://qiita.com/hosomichi/items/29b19ed3ebd0df9361ae)
- [The npm Blog — Using jQuery plugins with npm](http://blog.npmjs.org/post/112064849860/using-jquery-plugins-with-npm "The npm Blog — Using jQuery plugins with npm")


