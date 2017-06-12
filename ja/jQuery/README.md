# jQuery

> 本文基于[jQuery](http://jquery.com/ "jQuery") 2.1.4版本

jQuery允许我们通过`$.fn`,给jQuery对象添加带有返回值`$()`的method来扩展jQuery.

如下例 `greenify` 插件, 让我们能够调用 `$(document.body).greenify();` 

[import, greenify.js](../../src/jQuery/greenify.js)

在实际使用时, 我们需要在载入`greenify.js`前,先载入`jquery.js`

```html
<script src="jquery.js"></script>
<script src="greenify.js"></script>
```

## 它是如何工作的?

首先让我们看看jQuery的插件是如何工作的

jQuery的插件写法如`$.fn.greenify = function (){}`应用prototype的扩展规则. 

`jQuery.fn`的实体是`jQuery.prototype`,其实质是prototype扩展.

```js
// https://github.com/jquery/jquery/blob/2.1.4/src/core.js#L39
jQuery.fn = jQuery.prototype = {
    // prototype的实现
};
```


`$()`内部返回一个`new` jQuery 对象、这个jQuery 对象可以使用prototype进行扩展.

```js
$(document.body); // 返回值是 jQuery instance
```

也就是说, jQuery插件直接使用prototype.

## 适用于哪些场景?

我们了解了jQuery插件的实现原理,现在来考虑一下这种方式适用于哪些场合.

首先,因为这是prototype的简单封装,所以适用于prototype同样的场合.
其次,因为可以动态添加methods,所以你可以添加例如monkey patch来覆盖已存在的插件方法.

## 不适用于哪些场景?

与prototype相似, prototype扩展方式过于灵活,jQuery本身几乎无法约束这些插件的行为.

另外,扩展依赖于jQuery的实现,插件很可能因为jQuery的版本更新而不可用.

尽管jQuery声称不会去更改那些未文档化的API代码,但也不总是严格遵守.

## 模仿实现

我们现在来写一个可扩展的`calculator`计算器,来实现jQuery plugin类似的机制.

`calculator` 代码如下

[import, calculator.js](../../src/jQuery/calculator.js)

与`$.fn`相似,我们使用`fn`作为`prototype`的别名

```js
calculator.fn = calculator.prototype;
```

我们在此声明了一个`calculator(初始值)`的稍微有点特殊的构造器,但我们暂未加入任何实际功能.

[calculator.js](#calculator.js)接下来,我们来给计算器添加四则运算操作.

[import, calculator-plugin.js](../../src/jQuery/calculator-plugin.js)

在[calculator-plugin.js](#calculator-plugin.js)里,我们通过`calculator.fn.add`的方式添加一个`add`方法.

这里,module的依存关系与jQuery相同,我们需要先引用[calculator.js](#calculator.js),然后再引用[calculator-plugin.js](#calculator-plugin.js)

```html
<script src="calculator.js"></script>
<script src="calculator-plugin.js"></script>
```

通过这种方式,我们现在实现了例如`calculator#add`这样的方法,所以,你可以这样写:

[import, calculator-example.js](../../src/jQuery/calculator-example.js)

通过这个例子,我们看到了使用javascript的`prototype`进行扩展的方法,并没有什么特别之处. 
我们可以简单总结为「用`calculator.fn`代替`calculator.prototype`进行扩展」

## 生态

这种插件机制依赖于确定的global对象,应用的前提是要能通过`<script>`标签引用脚本.

```html
<script src="jquery.js"></script>
<script src="greenify.js"></script>
```

这种实现方法在CommonJS或ES6模组的形态下不能使用,因此在Node.js生态里是不存在的.
换句话说,这种机制不适用于CommonJS或ES6.

## 总结

本章我们学习了jQuery plugin 的建构方式,可归纳为:

- jQuery plugin 通过 `jQuery.fn` 扩展
- `jQuery.fn` 等于 `jQuery.prototype`
- jQuery plugin 扩展在 `jQuery.prototype` 
- (jQuery本身)很难控制插件的行为,因为(prototype)可以做任何事,非常自由

## 参考资料

- [Plugins | jQuery Learning Center](https://learn.jquery.com/plugins/ "Plugins | jQuery Learning Center")
- [jQuery拡張の仕組み 〜 JSおくのほそ道 #013 - Qiita](http://qiita.com/hosomichi/items/29b19ed3ebd0df9361ae)
- [The npm Blog — Using jQuery plugins with npm](http://blog.npmjs.org/post/112064849860/using-jquery-plugins-with-npm "The npm Blog — Using jQuery plugins with npm")


