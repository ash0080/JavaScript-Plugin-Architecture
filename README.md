# JavaScript Plugin Architecture [![Build Status](https://travis-ci.org/azu/JavaScript-Plugin-Architecture.svg?branch=master)](https://travis-ci.org/azu/JavaScript-Plugin-Architecture)

本书主要讨论各种javascript库和工具的插件架构设计

提供以下阅读格式

- [Web版](https://ash0080.gitbooks.io/javascript-plugin-architecture/content/)
- [PDF格式](https://www.gitbook.com/download/pdf/book/ash0080/javascript-plugin-architecture)
- [ePub格式](https://www.gitbook.com/download/epub/book/ash0080/javascript-plugin-architecture)
- [Mobi格式](https://www.gitbook.com/download/mobi/book/ash0080/javascript-plugin-architecture)

你也可以从[GitHub](https://github.com/azu/JavaScript-Plugin-Architecture)直接获取Markdown格式
不过我们建议阅读[Web版](https://ash0080.gitbooks.io/javascript-plugin-architecture/content/)

本文关联的Twitter话题标签是[#js_plugin_book](https://twitter.com/search?f=tweets&q=%23js_plugin_book&src=typd "Twitter #js_plugin_book")

更新信息可通过[RSS](https://github.com/ash0080/JavaScript-Plugin-Architecture/releases.atom)或[发布节点](https://github.com/ash0080/JavaScript-Plugin-Architecture/releases)订阅。

<!-- textlint-disable -->

<a aria-label="Star azu/JavaScript-Plugin-Architecture on GitHub" href="https://github.com/azu/JavaScript-Plugin-Architecture" class="github-button"><img src="https://monosnap.com/file/MZsfLjZNkSNwTJ33apkwpBjlBZLbSh.png" alt="GitHub"></a> <a href="http://b.hatena.ne.jp/entry/https://github.com/azu/JavaScript-Plugin-Architecture" class="hatena-bookmark-button" data-hatena-bookmark-title="JavaScript Plugin Architecture" data-hatena-bookmark-layout="standard-balloon" data-hatena-bookmark-lang="ja" title="はてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="はてなブックマークに追加" width="20" height="20" style="border: none;" /></a>

<!-- textlint-enable -->

## 译者序

Javascript是一门十分自由的语言.相比其他语言,Javascript的既有库可谓浩瀚,实际工作中我们往往`拿来主义`,通过数个既有库,我们能够较为轻松地完成工作,
当我们有能力完成Application层面的工作之后,下一步该往何处去? 我们该如何写出`框架`层级的东西呢?
这时我们面对的第一个问题往往是: 如何去规划一个插件架构?
带着这个问题,我很偶然地发现了@azu的这本书,受益良多,于是花了一些时间翻译,希望也能与更多人分享,不仅仅是这些分析案例,更重要的是学习方法.

原文:      [gitbook](https://azu.gitbooks.io/javascript-plugin-architecture/content/)
原Repo:     [github](https://github.com/azu/JavaScript-Plugin-Architecture) 

## 简介

在Javascript的世界中, 存在着多种编码风格,用于将许多细小的功能组合在一起,而不是直接创建一个臃肿庞大的库.
为了将细小的功能组合在一起,就需要一种称作「插件扩展」的机制.
另外, 插件架构对于创建一个拥有诸多插件的软件生态非常重要.

> 插件架构的引入会推进从「用户社区」向「开发者社区」, 从「软件开发」向「软件生态」的质的转变
> -- [开源软件开发和维护活跃度与良好的软件设计之间是否存在着必然的联系? - t-wada 的博客](http://t-wada.hatenablog.jp/entry/active-oss-development-vs-simplicity "OSS開発の活発さの維持と良いソフトウェア設計の間には緊張関係があるのだろうか? - t-wadaのブログ")

本书聚焦于从这类生态中研究和学习javascript的插件架构

## 本书内容

### [jQuery](ja/jQuery/README.md)

展示了jQuery 插件机制
展示了一种基于`<script>`标签的插件架构

### [ESLint](ja/ESLint/README.md)

本章会解释ESLint的rules的扩展机制
ESLint将javascript代码转换为AST,然后在AST的基础上实现代码检查
通过试写一个实例插件来了解ESLint rules是如何工作的.

### [Connect](ja/connect/README.md)

展示了Connect的称为**middleware**的插件架构
这种分层插件结构常见于HTTP servers库,除Node.js之外,还可以在比如 _Rack_ 中见到

### [gulp](ja/gulp/README.md)

展示了著名**自动化任务工具**gulp的插件架构
Gulp使用Node.js的Stream实现其数据流,并使用vinyl对象作为其数据
通过实际写一个gulp插件来学习gulp的插件架构

### [Redux](ja/Redux/README.md)

解释了Redux应用状态(state)管理框架的插件架构
Reduxe使用**middelware**作为其扩展机制, 不过这里的**middelware**和Connect的存在相似之处也有不同之处.
我们通过实际编写一个Redex的**middleware** 我来学习Redux的插件架构

## Contributing

本书供免费阅读,同时您也有权利增改其内容.

[CONTRIBUTING.md](https://github.com/ash0080/JavaScript-Plugin-Architecture/blob/master/CONTRIBUTING.md)
关于提交本书建议, Pull Request, commit 等, 可阅读此链接

关于错误,本文使用的 library 更新, 请通过Issue 或 Pull Request提交

所有源代码均公开于Github.

- [azu/JavaScript-Plugin-Architecture](https://github.com/azu/JavaScript-Plugin-Architecture)

## License

MIT/CC BY-NC © azu
本中文版由[圈爷](https://github.com/ash0080) 翻译

- 源代码基于MIT license
- 文章基于[CC BY-NC 4.0](http://creativecommons.org/licenses/by-nc/4.0/ "CC BY-NC 4.0") license
