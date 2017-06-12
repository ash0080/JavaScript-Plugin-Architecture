# ESLint

> 本文基于[ESLint](http://eslint.org/ "ESLint") 1.3.0 版本

[ESLint](http://eslint.org/ "ESLint")是一个用javascript书写的规则来验证javascript代码的检验工具. 译注:有点绕,其实就是ESLint的rule都是用js写的.

大致来说, ESLint根据javascript代码先生成AST(abstract syntax tree 抽象语法树), 然后按照写好的rule验证AST,并输出 errors 和 warnings

rule可用插件的形式书写, 实际上所有ESLint的rules都是以插件形式存在的.

> The pluggable linting utility for JavaScript and JSX

在ESLint官网, 你可以看到以上这句话, 说明ESLint的设计架构十分强调插件.

接下来,就让我们看看ESLint的插件架构.

## 如何写(插件)?

ESLint使用`.eslintrc`配置文件定义使用的rules,具体操作方法请参考官方文档.

- [Documentation - ESLint - Pluggable JavaScript linter](http://eslint.org/docs/user-guide/configuring "Documentation - ESLint - Pluggable JavaScript linter")

ESLint里的rule可以表述为一个导出以下function的模组,该function传入一个`context`对象,并返回一个与之对应的对象.

<a name="no-console.js"></a>
[import, no-console.js](../../src/ESLint/no-console.js)

ESLint基于AST检测代码,而不是直接检查字符串,我们这里略过AST的实现细节,我们大致可以将AST理解为表征javascript代码的树状结构对象.

```js
console.log("Hello!");
```

代码经过转换, 会变成以下的AST对象

```json
{
    "type": "Program",
    "body": [
        {
            "type": "ExpressionStatement",
            "expression": {
                "type": "CallExpression",
                "callee": {
                    "type": "MemberExpression",
                    "computed": false,
                    "object": {
                        "type": "Identifier",
                        "name": "console"
                    },
                    "property": {
                        "type": "Identifier",
                        "name": "log"
                    }
                },
                "arguments": [
                    {
                        "type": "Literal",
                        "value": "Hello!",
                        "raw": "\"Hello!\""
                    }
                ]
            }
        }
    ],
    "sourceType": "script"
}
```

- [JavaScript AST explorer](http://astexplorer.net/#/FNrLHi8ngW "JavaScript AST explorer")

ESLint基于rules验证AST, 比如我们定义在[no-console.js](#no-console.js)中的`console.log`.

回到如何书写rules的话题, `context`实际上是一个function, 而rule的主体部分返回一个包含了`context`操作的方法的对象.译注:看代码

这个对象有个以节点类型命名的方法. 遍历AST时, 当「访问到`"MemberExpression"` 类型的节点时」,我们在rule中所声明的通知(方法)被触发.

之前所写的`console.log`在`MemberExpression`类型节点的AST中,可以表述为以下对象.

```json
{
    "type": "MemberExpression",
    "computed": false,
    "object": {
        "type": "Identifier",
        "name": "console"
    },
    "property": {
        "type": "Identifier",
        "name": "log"
    }
}
```
如[no-console.js](#no-console.js)rule所示,当一个`MemberExpression`类型节点的`node.object.name === "console"`条件满足时, report 一个 error.

如果AST检索比较难以理解,这里提供了一个工具帮助理解检索过程.

- [azu.github.io/visualize_estraverse/](http://azu.github.io/visualize_estraverse/ "visualize estraverse step")

```js
function debug(string){
    console.log(string);
}
debug("Hello");
```

<video controls>
<source src="./movie/traverse.webm" type="video/webm">
<source src="./movie/traverse.mp4" type="video/mp4">
<p>若需正常浏览本视频,浏览器需支持webm或mp4格式</p>
</video>

书写ESLint 规则的其他方法,你可以查看官方文档或下面列出的文章.

- [Documentation - ESLint - Pluggable JavaScript linter](http://eslint.org/docs/developer-guide/working-with-rules "Documentation - ESLint - Pluggable JavaScript linter")
- [Find bugs in code with code! | Cyber Agent Official Engineer Blog](http://ameblo.jp/principia-ca/entry-11837554210.html "Find bugs in code with code! | Cyber Agent Official Engineer Blog")

## 它如何工作的?

大致来说,ESLint的工作原理是将代码解析为AST,然后利用js写的rule来检测AST.

接下来,我们就来看看这个机制是怎样把一条rule当成plugin来工作的.

1. 每一条rule都需注册`node.type`事件
2. 当遍历AST时,`node.type`事件被击发
3. 通过`context.report()`方法输出内容

EventEmitter被用于注册和击发事件,ESLint本身一般存在复数条rules,所以这是一个典型的 `Pub/Sub` 设计模式.

当表述为伪代码时,处理Lint的过程可以表达为以下的流程

```js
import {parse} from "esprima";
import {traverse} from "estraverse";
import {EventEmitter} from "events";

function lint(code){
    // 解析 code to AST
    let ast = parse(code);
    // 注册事件
    let emitter = new EventEmitter();
    let results = [];
    emitter.on("report", message => {
        // 3. 输出内容
        results.push(message);
    });
    // 列出所有的 rules
    let ruleList = getAllRules();
    // 1. 为每个rules注册`node.type`事件
    ruleList.forEach(rule => {
        // 比如) MemberExpression(node){}
        // => {"MemberExpression" : function(node){}, ... }
        // 将rule的方法列表作为一个对象获取
        let methodObject = getDefinedMethod(rule);
        Object.keys(methodObject).forEach(nodeType => {
            emitter.on(nodeType, methodList[nodeType]);
        });
    });
    // 2. 遍历AST时,击发`node.type`事件
    traverse(ast, {
        // 1. 如果`node.type`已注册,击发
        enter: (node) => {
            emitter.emit(node.type, node);
        },
        leave: (node) => {
            emitter.emit(`${node.type}:exit`, node);
        }
    });
    // 2. 从rule获取内容并显示
    console.log(results.join("\n"));
}
```

使用Pub/Sub 设计模式, AST只需遍历一次,即可`emit`出每条rule中所定义的信息.

为了让我们更进一步理解, 现在让我们用相同的原理来写一段可以实际运行的代码.

## 模仿实现

这次,让我们来尝试模仿ESLint写一个自己的Lint处理工具.

之前,我们已经写了[no-console.js](#no-console.js)这个自定义rule, 这次我们就写一个自己的`MyLinter`来尝试使用这个rule检验javascript代码.

### MyLinter

我们把MyLinter简单定义为一个拥有两个方法的类.

- `MyLinter#loadRule(rule): void`
    - 注册使用的rules
    - `rule`是[no-console.js](#no-console.js)export的内容
- `MyLinter#lint(code): string[]`
    - 接受`code`并返回根据rule Lint之后的结果
    - Lint结果存为一个error messages数组形式

实现代码如下,

[import, src/ESLint/MyLinter.js](../../src/ESLint/MyLinter.js)

使用方法:通过`MyLinter#load`方法读取[no-console.js](#no-console.js)规则

```js
function add(x, y){
    console.log(x, y);
    return x + y;
}
add(1, 3);
```

Lint 代码

[import, src/ESLint/MyLinter-example.js](../../src/ESLint/MyLinter-example.js)

因为我们的代码中包含`console`对象, 执行后会抛出一个_"Unexpected console statement."_ 的错误信息.   

### RuleContext (格则上下文)

我们再看一下[MyLinter.js](#MyLinter.js),你可能会注意到里面有个叫做`RuleContext`的类.

这个`RuleContext`实际包含了所有在rule中的可用方法.

在MyLinter中我们这次仅使用了`RuleContext#report`方法实现了错误信息的返回.

在rule的实现代码中,我们并没有直接返回对象, 你可以看到rule通过`context`参数,传入一个`RuleContext`的实例.

[import, no-console.js](../../src/ESLint/no-console.js)

通过这种方式,rule可以使用所给的'context'而并不需要知道Mylinter的具体实现细节. 译注:依赖注入

## 适用于哪些场合?

这种插件架构得益于Pub/Sub设计模式,适用于ESLint这样需要读取并检测所给代码的场合.

换句话说, 我们也可以说它是一种 read-only 的插件架构.

并且,`context`被注入rule中使用,rule被设计为仅仅是被动`给`,rule和Linter本体是一种松耦合的关系. 

因此,通过控制`context`所暴露的接口功能,可以很容易控制rule所能影响的范围. 译注: 也就是说控制rule能做多少事.

## 不适用于哪些场合?

相反的,如果我们想改写代码(AST)时,因为rule也在同时进行处理,所以如果rules之间存在冲突,处理过程会很容易崩.

因此,(如果要进行写操作),不添加另一个抽象层几乎不可能.

也就是说, 对于read-write插件架构, 我认为使用这一简单的设计模式是难以实现的.

> **NOTE** ESLint 2.0 计划添加autofixing功能,引入写操作。
> 一个抽象层用于从rules中聚合需要改写的命令到一个`SourceCode`对象,最终再执行实际的改写操作.
> - [Implement autofixing · Issue #3134 · eslint/eslint](https://github.com/eslint/eslint/issues/3134 "Implement autofixing · Issue #3134 · eslint/eslint")

## 使用这种机制的案例

- [azu/textlint](https://github.com/azu/textlint "azu/textlint")
    - Markdown文本检测工具

## 生态

由于ESLint rule是javascript模组, 所以rule本身可以在(https://www.npmjs.com/ "npm")公开发布.

不过,ESLint并没有默认生效的rule.所以使用ESLint的时候,必须创建配置文件或者ESLint wrapper,例如[sindresorhus/xo](https://github.com/sindresorhus/xo "sindresorhus/xo")

ESLint有一套`eslint:recommended`推荐设定, 你可以用`extends`继承这些设定.

```json
{
    "extends": "eslint:recommended"
}
```
因为这些设定可以写成javascript(模组),所以也可以用npm发布.

- [Shareable Configs - ESLint - Pluggable JavaScript linter](http://eslint.org/docs/developer-guide/shareable-configs "Documentation - ESLint - Pluggable JavaScript linter")

coding有各种各样的rule.需要什么样的rule因人而异.
不应用配置文件实现起来当然十分简单,但是要想每个人都能当做工具来使用就十分困难了.
能够简单的制作并分享各种各样的设定就是为了解决这一问题.

这就是_The pluggable linting utility_的实现原理

## 总结

本章我们学习了ESLint plugin 的构建方式:

- ESLint 的rule可以用javascript写
- AST遍历使用Pub/Sub设计模式
- rule并不需要知道传入的`context`实现细节
- 对于rule这种read-only的形式十分简单高效
- 如果要实现read-write操作则需要小心了
- 配置文件可以写成javascript
- 可以通过npm分享配置


