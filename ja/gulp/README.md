# gulp

> 本文基于[gulp](http://gulpjs.com/ "gulp") 3.9.0 版本.

[gulp](http://gulpjs.com/ "gulp")是Node.js的自动化脚本工具. 可以执行build, test之类的任务, 每条任务都可以用javascript书写.

这里所说的task是由多个步骤组成的过程块.
`Gulp.task`作为API用于定义task
另外、多个process通过node.js的[Stream](https://nodejs.org/api/stream.html "Stream")连接,过程中无需产生临时文件.
用户可以通过载入模组,使用pipe()连接模组来定义task.

## 如何写(插件)?

举例来说, 我们需要处理一个[Sass](http://sass-lang.com/ "Sass")格式的文件.

1. 读取`sass/*.scss`文件
2. 使用`sass`编译读取的sass文件.
3. 使用`autoprefixture`预处理前序生成的css文件
4. 用`minify`压缩每个css文件
5. 将压缩后的CSS文件循环放置到`css`文件夹

这一串连续处理的进程可以定义为以下的tasks

```js
import gulp from "gulp";
import sass from "gulp-sass";
import autoprefixer from "gulp-autoprefixer";
import minify from "gulp-minify-css";

gulp.task("sass", function() {
    return gulp.src("sass/*.scss")
        .pipe(sass())
        .pipe(autoprefixer())
        .pipe(minify())
        .pipe(gulp.dest("css"));
});
```
因为本文主要是要了解gulp插件的实现原理,所以gulp的详细使用方法可以参考以下文档.

- [gulp/docs at master · gulpjs/gulp](https://github.com/gulpjs/gulp/tree/master/docs)
- [現場で使えるgulp入門 - gulpとは何か | CodeGrid](https://app.codegrid.net/entry/gulp-1)
- [gulp入門 (全12回) - プログラミングならドットインストール](http://dotinstall.com/lessons/basic_gulp)

## 它如何工作的?

让我们实际看一下gulp的插件是如何在一起连携工作的.

在之前的gulp task的例子中,我们只是简单地将每个模组用`pipe`连在一起,这里我并不知道每一个处理是如何实现的.

接下来,我要写一个叫做`gulp-prefixer`的插件, `gulp-prefixer`插件在每一个给定的文件前加上一个指定的字符串.

在官方文档「how to write a plug-in」中,有一个相似的插件,所以你也可以同时看一下官方文档中的这个例子.

- [gulp/docs/writing-a-plugin](https://github.com/gulpjs/gulp/tree/master/docs/writing-a-plugin "gulp/docs/writing-a-plugin at master · gulpjs/gulp")
- [gulp/dealing-with-streams.md](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/dealing-with-streams.md "gulp/dealing-with-streams.md at master · gulpjs/gulp")

有很多gulp插件接受options参数并实现返回Node's Stream对象.

[import gulp-prefixer.js](../../src/gulp/gulp-prefixer.js)

我们写的这个`gulp-prefixer`插件, 可按以下方式嵌入task中.

[import gulpfile.babel.js](../../src/gulp/gulpfile.babel.js)

`default` task 包含以下处理步骤.

1. 从`./*.*`获取文件(当前目录下全部文件)
2. 添加`prefix text`到全部获取的文件头部
3. 把修改后的文件输出到`build/`文件夹

### Stream

调用[gulp-prefixer.js](#gulp-prefixer.js)文件中的`gulpPrefixer`方法将返回一个[Transform Stream](https://nodejs.org/api/stream.html#stream_class_stream_transform "stream.Transform")实例

```js
let gulpPrefixer = function (prefix) {
    // enable `objectMode` of the stream for vinyl File objects.
    return new Transform({
        // Takes in vinyl File objects
        writableObjectMode: true,
        // Outputs vinyl File objects
        readableObjectMode: true,
        transform: function (file, encoding, next) {
            if (file.isBuffer()) {
                file.contents = prefixBuffer(file.contents, prefix);
            }

            if (file.isStream()) {
                file.contents = file.contents.pipe(prefixStream(prefix));
            }
            this.push(file);
            next();
        }
    });
};

export default gulpPrefixer;
```

我们这里用了 Transform Stream, Node.js的Stream类一共有4种:

- Readable Stream
- Transform Stream
- Writable Stream
- Duplex Stream

我们定义的`default` task按以下形式传递stream

1. 从`./*.*`获取文件(当前目录下全部文件) = Readable Stream
2. 添加`prefix text`到全部获取的文件头部 = Transform Stream
3. 把修改后的文件输出到`build/`文件夹 = Writable Stream

整个过程可以理解为, 文件被 _Read_, 接着 _Transform_, 最后被 _Write_ 的数据操作流程

在[gulp-prefixer.js](#gulp-prefixer.js)中,data以stream的形式被传入,转换为Transform Stream的形式传递到下一步骤.

为了处理「gulp中的数据流」`readableObjectMode`和`writableObjectMode`分别被设置为`true`.
这里的 _ObjectMode_ 用于设置Stream对象的读写权限.

一般来说Node.js Stream 操作[Buffer](https://nodejs.org/api/buffer.html)这种二进制数据,[Buffer](https://nodejs.org/api/buffer.html)可以与string互相转换,但是,不可以操作复杂类型比如Object.

因此, Node.js的Stream 提供了一种[Object Mode][Object Mode](https://nodejs.org/api/stream.html#stream_object_mode "Object Mode")如果启用此模式,Buffer或String以外的Javascript Object,也可以被Stream流化操作.

关于Node.js Stream更进一步的阅读可以参考以下文章.

- [Stream Node.js Manual & Documentation](https://nodejs.org/api/stream.html "Stream Node.js Manual &amp; Documentation")
- [substack/stream-handbook](https://github.com/substack/stream-handbook "substack/stream-handbook")

### vinyl

gulp的[vinyl](https://github.com/gulpjs/vinyl "vinyl")Object也是一种Stream 流化对象。
vinyl 被称为 _Virtual file format_ (虚拟文件格式) 是gulp创建的一种包含了文件信息与文件内容的抽象格式.

通过以下的几条需求,可以很容易理解我们为什么需要这种抽象文件格式

- 我们需要获知流数据的文件后缀
- 我们需要获知流数据的可读属性
- 我们需要写文件到与流数据相同的路径

如果流数据中只包含内容,我们就无法获知文件的路径,可读属性等细节.因此,通过`gulp.src`读取的文件被包装成vinyl虚拟文件,文件的内容可以通过`contents`引用.

### 在vinyl中处理contents

让我们看看Transform Stream的具体实现

```js
// file is a `vinyl` Object
if (file.isBuffer()) {
    file.contents = prefixBuffer(file.contents, prefix);
}

if (file.isStream()) {
    file.contents = file.contents.pipe(prefixStream(prefix));
}
```

文件可能读取为Buffer或Stream存储于`vinyl`抽象格式的`contents`属性中, 所以两种patterns都需要写出.

> **NOTE**: gulp plugin 并不强制要求两种pattern, 很多插件仅支持Buffer, 所以, 官方guideline建议添加异常处理.- [gulp/guidelines.md at master · gulpjs/gulp](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/guidelines.md "gulp/guidelines.md at master · gulpjs/gulp")

`contents`中究竟存储着哪种类型取决于前序Stream

```js
gulp.src("./*.*")
    .pipe(gulpPrefixer("prefix text"))
    .pipe(gulp.dest("build"));
```

在上面的例子中, 取决于`gulp.src`
因为`gulp.src` 默认存成buffer形式,所以这里会按照buffer进行处理.

`gulp.src` 通过传入`{ buffer: false }`参数也可以指定输出为Stream形式.

```js
gulp.src("./*.*", { buffer: false })
        .pipe(gulpPrefixer("prefix text"))
        .pipe(gulp.dest("build"));
```

### 变换处理

最后,我们看一下Buffer和Stream之间的转换处理

```js
export function prefixBuffer(buffer, prefix) {
    return Buffer.concat([Buffer(prefix), buffer]);
}

export function prefixStream(prefix) {
    return new Transform({
        transform: function (chunk, encoding, next) {
            // ObjectMode:falseのTransform Stream
            // StreamのchunkにはBufferが流れてくる
            let buffer = prefixBuffer(chunk, prefix);
            this.push(buffer);
            next();
        }
    });
}
```

这样,`prefix` 字符串就合并到文件Buffer的前部了.

转换过程并不依赖于gulp,你也可以传入到其它的库中进行处理.因为Buffer和String可以直接变换,很多gulp插件可能只会实现`gulpPrefixer`和`prefixBuffer`部分.

换句话说,像这种添加prefix的变换处理很可能通过加载既有库来完成.

gulp在插件间以[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象的形式传递数据,可以说(vinyl)使用Node.js Stream 作为它的interface.

## 生态

gulp单个插件的的运作过程可以总结为「输出数据作为(下一)插件的输入」。而这是通过[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象将Node.js Stream作为interface来实现的.

gulp 建议插件按单一功能设计(single function)

> Your plugin should only do one thing, and do it well.
> -- [gulp/guidelines.md](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/guidelines.md "gulp/guidelines.md at master · gulpjs/gulp")

gulp直接使用了Node.js Stream而避免了开发自己的API

Transform Stream 在单一变换处理中具有很好的表现,所以你可以通过`pipe`将多个处理进程串联.

因为gulp是个自动化任务工具,所以让既有的库能够很容易的被当做task来使用是非常重要的.而Node.js的Stream默认以`Buffer`传递数据,这使得既有库难以处理这些数据,因此引入了[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象作为数据.

通过这种方式,gulp让使用既有库开发单功能插件变得简单.
从生态上来说,这让gulp拥有大量可重用的插件变得可能.

## 适用于哪些场合?

gulp自身仅仅管理数据流,具体的任务由插件完成,根据不同的任务需求,插件的种类也多种多样.

gulp将[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象定义为一种中间格式,它让使用既有库创建插件变得十分简单.

另外,与Grunt不同的是,gulp直接用javascript定义任务.有时候当无法通过一组插件完成某些任务时,你可以直接写代码解决.

因此,在插件的处理过程不可确定的情况下,只要解决如何将数据通过中间格式传递就可以了.

结论

- 既有库可以很容易转换成task插件
- 如果没有现成的插件,也可以直接写在setting里

## 不适用于哪些场合?

通常我们(完成一个任务)需要用到多个插件的组合,但是多个插件组合也容易发生问题.
プラグインを複数組み合わせ扱うものに共通することですが、プラグインの組み合わせ問題はgulpでも発生します。

比如,[Browserify](https://github.com/substack/node-browserify)也使用Node.js的Stream.
但如果不指定起始点,就会抛出错误. 译注:这里指browserify需要在插件的options里指定entries,具体看以下文档

- [gulp/browserify-transforms.md at master · gulpjs/gulp](https://github.com/gulpjs/gulp/blob/master/docs/recipes/browserify-transforms.md "gulp/browserify-transforms.md at master · gulpjs/gulp")

另外,虽然gulp建议插件按单一功能设计,但这只是`建议`, gulp并没有从API层面作出这样的限制.

gulp`解决`这个问题的方法只是给出了guidelines和recipes这样的文档.

- [gulp/docs at master · gulpjs/gulp](https://github.com/gulpjs/gulp/tree/master/docs "gulp/docs at master · gulpjs/gulp")

容易将既有库包装成插件,也意味着插件之间的options并未遵循统一的规则,每一个插件都有自己的一套规则,用户需要自己学习每一个插件的配置方式.

(因为插件开发十分容易)经常出现相似的功能,有好几个不同作者写的插件, 这也导致插件的质量难以保证,选择起来较为麻烦.

结论

- 插件组合的问题要由用户自行解决
- 插件功能重复的情况很常见

## 使用这种机制的案例

- [sighjs/sigh](https://github.com/sighjs/sigh "sighjs/sigh")
    - 兼容gulp插件

## 总结

本章我们学习了gulp plugin的构建方式

- gulp是自动化任务工具
- 任务可以用javascript书写
- gulp管理中间格式和数据流
- 中间格式[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象
- 数据流通过 Node.js Stream 实现
- 使用既有库创建插件十分容易
- 插件功能重复的情况很常见


