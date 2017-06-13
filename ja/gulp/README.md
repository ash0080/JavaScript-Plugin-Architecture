# gulp

> 本文基于[gulp](http://gulpjs.com/ "gulp") 3.9.0 版本.

[gulp](http://gulpjs.com/ "gulp")是Node.js的自动化脚本工具. 可以执行build, test之类的任务, 每条任务都可以用javascript书写.

这里所说的task是由多个处理步骤组成的过程块.
`Gulp.task`作为API用于定义task,
多个处理步骤通过node.js的[Stream](https://nodejs.org/api/stream.html "Stream")连接,过程中不产生临时文件.
用户可以通过载入模组,使用pipe()连接模组来组成task.

## 如何写(插件)?

比如说, 我们要处理一个[Sass](http://sass-lang.com/ "Sass")格式的文件,步骤如下:

1. 读取`sass/*.scss`文件
2. 使用`sass`编译读取的sass文件.
3. 使用`autoprefixture`预处理前序编译成的css文件
4. 用`minify`压缩各css文件
5. 将压缩后的CSS文件循环放置到`css`文件夹

这一串连续处理步骤可以写成以下的task

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
因为本文主要是要了解gulp插件的实现原理(并不打算就gulp如何使用展开),所以gulp的详细使用方法可以参考以下文档.

- [gulp/docs at master · gulpjs/gulp](https://github.com/gulpjs/gulp/tree/master/docs)
- [現場で使えるgulp入門 - gulpとは何か | CodeGrid](https://app.codegrid.net/entry/gulp-1)
- [gulp入門 (全12回) - プログラミングならドットインストール](http://dotinstall.com/lessons/basic_gulp)

## 它如何工作的?

让我们实际看一下gulp的插件是如何在一起连携工作的.
在之前的gulp task的例子中,我们只是简单地将每个模组`pipe`在一起,这里我们并不知道每一个处理功能的细节是如何实现的.
接下来,我们来写一个`gulp-prefixer`插件, `gulp-prefixer`插件在每一个指定的文件头部加上一个指定的字符串.

在官方文档「how to write a plug-in」中有一个同名的例子,最好能同时看一下这个例子.

- [gulp/docs/writing-a-plugin](https://github.com/gulpjs/gulp/tree/master/docs/writing-a-plugin "gulp/docs/writing-a-plugin at master · gulpjs/gulp")
- [gulp/dealing-with-streams.md](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/dealing-with-streams.md "gulp/dealing-with-streams.md at master · gulpjs/gulp")

多数gulp插件接受options参数,并被定义为一个能返回(Node.js)Stream对象的函数.

[import gulp-prefixer.js](../../src/gulp/gulp-prefixer.js)

我们实现的这个`gulp-prefixer`插件, 能够按下面方式在task中使用.

[import gulpfile.babel.js](../../src/gulp/gulpfile.babel.js)

`default`task按以下处理步骤执行.

1. 从`./*.*`获取文件(当前目录下全部文件)
2. 添加`prefix text`到全部文件的头部
3. 把修改后的文件输出到`build/`文件夹

### Stream

如[gulp-prefixer.js](#gulp-prefixer.js)所示, 调用`gulpPrefixer`方法将返回一个[Transform Stream](https://nodejs.org/api/stream.html#stream_class_stream_transform "stream.Transform")实例

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

这里用到Transform Stream, Node.js的Stream类一共有4种:

- Readable Stream
- Transform Stream
- Writable Stream
- Duplex Stream

`default`task的处理过程(传递Stream)如下:

1. 从`./*.*`获取文件(当前目录下全部文件) = Readable Stream
2. 添加`prefix text`到全部获取的文件头部 = Transform Stream
3. 把修改后的文件输出到`build/`文件夹 = Writable Stream

这里存在着,文件被 _Read_(读取), 接着 _Transform_(转换), 最后被 _Write_(写入)的数据流.

在[gulp-prefixer.js](#gulp-prefixer.js)中,data以stream的形式被传入,转换为Transform Stream的形式传递到下一步骤.

为了实现「gulp中的数据流传递」,`readableObjectMode`和`writableObjectMode`分别被设置为`true`.
这里的 _ObjectMode_ 用于设置Stream对象(的读写权限).

一般,Node.js Stream用于处理[Buffer](https://nodejs.org/api/buffer.html)这种二进制数据格式,[Buffer](https://nodejs.org/api/buffer.html)可以与string互相转换,但不可处理复杂类型比如Object.

因此,Node.js的Stream提供了一种[Object Mode][Object Mode](https://nodejs.org/api/stream.html#stream_object_mode "Object Mode"),用于处理Buffer或String以外的Javascript Object对象,令其也可以被Stream流化.

关于Node.js Stream更进一步的了解可以参考以下文章.

- [Stream Node.js Manual & Documentation](https://nodejs.org/api/stream.html "Stream Node.js Manual &amp; Documentation")
- [substack/stream-handbook](https://github.com/substack/stream-handbook "substack/stream-handbook")

### vinyl

gulp使用[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象在Stream中传递。
vinyl也称作 _Virtual file format_ (虚拟文件格式) 是gulp创造的一种包含了文件信息与文件内容的抽象格式.

通过以下几条需求,可以帮助我们理解为什么要创造这种抽象文件格式

- 我们需要获知流数据的文件后缀
- 我们需要获知流数据的可读属性
- 我们需要写文件到与流数据相同的路径(译注:需要知道文件路径)

如果流数据中仅包含文件内容,我们就无法获知文件的路径或可读属性等信息.因此,通过`gulp.src`读取的文件被封装成vinyl虚拟文件,文件的内容则可通过`contents`引用.

### 获取vinyl的内容

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

`vinyl`抽象格式的`contents`属性, 可能是buffer格式也可能是Stream格式,所以如果你要满足两者,两种可能都需要在代码中实现,否则其中一种情况下就数据就不能继续传递.

> **NOTE** gulp plugin并不强制要求实现两种格式,很多插件仅支持Buffer,所以,官方guideline建议的是添加异常处理.- [gulp/guidelines.md at master · gulpjs/gulp](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/guidelines.md "gulp/guidelines.md at master · gulpjs/gulp")

`contents`中究竟存储着哪种格式取决于前序Stream

```js
gulp.src("./*.*")
    .pipe(gulpPrefixer("prefix text"))
    .pipe(gulp.dest("build"));
```

在上面的例子中, 取决于`gulp.src`
因为`gulp.src` 默认存成buffer形式,所以这里会按照buffer格式进行处理.

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

只是将前缀字符串转换成buffer加到传入的buffer之前.
转换过程并不依赖于gulp,你也可以在这里调用既有库处理数据.因为Buffer和String可以直接转换,很多gulp插件可能只会实现`gulpPrefixer`和`prefixBuffer`部分.(译注:还是在说很多插件只实现buffer格式)

换句话说,像这种添加prefix的变换处理很可能通过加载既有库来完成.

gulp插件彼此间使用[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象传递数据,可以说(gulp插件)是使用Node.js Stream 作为它的interface.

## 生态

gulp插件的主要工作可简单总结为「输入输出」.
数据通过[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象传递, 使用Node.js Stream作为其数据传递API的interface.

gulp建议单一插件只完成单一功能(single function)

> Your plugin should only do one thing, and do it well.
> -- [gulp/guidelines.md](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/guidelines.md "gulp/guidelines.md at master · gulpjs/gulp")

gulp直接使用了Node.js Stream而避免了开发另外的API
Transform Stream对于完成单一变换步骤很好用.而对于多个变换步骤,可以通过`pipe`串联.

因为gulp是个自动化任务工具,所以让既有的库能够很容易的在task中使用是非常重要的.而Node.js的Stream默认以`Buffer`传递数据,于是引入了[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象来传递数据,让使用既有库变得更容易实现.

通过这种方式,gulp让使用既有库开发单功能插件变得简单.
从生态上来说,这让gulp拥有大量可重用的插件变得可能.

## 适用于哪些场景?

gulp自身仅仅管理数据流,具体的任务由插件完成,根据不同的任务需求,插件的种类也多种多样.

gulp将[vinyl](https://github.com/gulpjs/vinyl "vinyl")对象定义为一种中间格式,它让使用既有库创建插件变得十分简单.

另外,与Grunt不同的是,gulp直接用javascript定义任务.有时候当无法通过一组插件完成某个任务时,你可以直接写代码解决.

因此,如果前序插件具体做了什么不可知,我们只需要知道(传递出的)中间数据格式和数据流就可以了.

结论

- 既有库可以很容易转换成task插件
- 如果没有现成的插件,也可以直接写在setting里

## 不适用于哪些场景?

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


