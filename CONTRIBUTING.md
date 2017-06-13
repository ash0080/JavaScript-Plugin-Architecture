# Contribute

## Installation

需要安装Node.js

    git clone https://github.com/azu/JavaScript-Plugin-Architecture.git
    cd JavaScript-Plugin-Architecture
    npm install

## Usage

本书使用[GitBook](https://github.com/GitbookIO/gitbook "GitBook")编写.

### 显示

`npm start`将运行GitBook本地服务器显示本书内容.

    npm start

### 测试

`npm test`运行代码和文本检查

    npm test

### 文本覆盖率检查

[textlint](https://github.com/textlint/textlint "textlint")和[textlint-formatter-codecov](https://github.com/azu/textlint-formatter-codecov "textlint-formatter-codecov")用于文本覆盖率检查.

理想目标是100%, 实际值一百分比呈现

[![codecov.io](https://codecov.io/github/azu/JavaScript-Plugin-Architecture/coverage.svg?branch=master)](https://codecov.io/github/azu/JavaScript-Plugin-Architecture?branch=master)

![coverage graph](https://codecov.io/github/azu/JavaScript-Plugin-Architecture/branch.svg?branch=master)

也可以使用以下命令检查文本覆盖率
```
npm run textlint:coverage
```

## Contribute 贡献

本文通过多个库和工具讨论javascript的插件架构设计

Contribute大致可分为已有文章修正和建议


## 文章的修正

如果发现书写错误,即使是一个字,也欢迎您发送Pull Request给我.

如果您发现书中存在着用词或术语使用不统一的情况, 我也很乐意接受您通过Pull Request发送的修订.

另外,本书使用[test/prh-rule.yaml](test/prh-rule.yaml)中所定义的辞典,来检查用词统一情况,所以辞典中的词条检测是可能的,如果存在问题也欢迎指正

- [textlint + prhで表記ゆれを検出する | Web Scratch](http://efcl.info/2015/09/14/textlint-rule-prh/ "textlint + prhで表記ゆれを検出する | Web Scratch")

```
## 建议

Proposalとは、書籍に載せたいプラグインアーキテクチャについてのIssueを立てることを言います。

例如,要提交一个关于库/工具xxx的issue,可以
たとえば、XXXというライブラリ/ツールのアーキテクチャについてのIssueを立てる場合、
次のようなことが1行とかでもいいので書かれていれば参考になります。

如果某插件机制难以理解,您也可以跳过它.
Javascript是非常自由的语言, 我认为即使是了解一种plugin的形式,也是非常有用的.

# XXX的架构

## 如何写(插件)?

- 实际代码例子

## 它是如何工作的?

- 核心机制,例如prototype扩展
- 关于这种机制的代码链接
- 关于这种机制和插件的文档链接

## 适用于哪些场景?

- どういう用途で使われてる(ユースケース)
- 変換する毎にファイルとして吐き出さないので、高速に複数の変換をするのに向いている等

## 不适用于哪些场景?

- 変換の仕組み上書き換え等を行うプラグインを扱いにくい等

## 使用这种机制的案例

- XXX以外にも同様の仕組みを使っているものがあるなら

----

## チェックリスト

- [ ] どう書ける?
- [ ] どういう仕組み?
- [ ] どういうことに向いている?
- [ ] どういうことに向いていない?
- [ ] この仕組みを使っているもの
- [ ] 実装してみよう
- [ ] エコシステム
```

你可以使用以下格式提交issue.

- [新しいProposalを書く](https://github.com/azu/JavaScript-Plugin-Architecture/issues/new?title=Proposal:XXX&body=%23+XXXのアーキテクチャ%0D%0AURL%3A)

### Proposalの具体例

現在ある[Proposal一覧](https://github.com/azu/JavaScript-Plugin-Architecture/labels/proposal)を参考にしてみるとよいかもしれません。

- [jQuery Plugin · Issue #8 · azu/JavaScript-Plugin-Architecture](https://github.com/azu/JavaScript-Plugin-Architecture/issues/8 "jQuery Plugin · Issue #8 · azu/JavaScript-Plugin-Architecture")

## テスト

`$ npm test` を実行するとコードや文章に対するテストが実行されます。

```sh
npm test
```

文章は[textlint](https://github.com/azu/textlint "textlint")による単語のチェックが行われます。

## コミットメッセージ

AngularJSのGit Commit Guidelinesをベースとしています。

- [conventional-changelog/angular.md at master · ajoslin/conventional-changelog](https://github.com/ajoslin/conventional-changelog/blob/master/conventions/angular.md "conventional-changelog/angular.md at master · ajoslin/conventional-changelog")

次のような形で1行目に概要、3行目から本文、最後に関連するIssue(任意)を書きます。

```
feat(ngInclude): add template url parameter to events

The `src` (i.e. the url of the template to load) is now provided to the
`$includeContentRequested`, `$includeContentLoaded` and `$includeContentError`
events.

Closes #8453
Closes #8454
```


```
                         scope        commit title

        commit type       /                /      
                \        |                |
                 feat(ngInclude): add template url parameter to events

        body ->  The 'src` (i.e. the url of the template to load) is now provided to the
                 `$includeContentRequested`, `$includeContentLoaded` and `$includeContentError`
                 events.

 referenced  ->  Closes #8453
 issues          Closes #8454
```

1行の`feat`や`fix`といったcommit typeは、迷ったらとりあえず`chore`と書いて、`scope`も省略して問題ないので次のような形でも問題ありません。

```
chore: コミットメッセージ
```

このコミットメッセージの規約は[conventional-changelog](https://github.com/ajoslin/conventional-changelog "conventional-changelog")による自動生成のためでもありますが、
取り込むときに調整できるので無視してもらっても問題はありません。

以下を見てみるとよいかもしれません。

- [良いChangeLog、良くないChangeLog | Web Scratch](http://efcl.info/2015/06/18/good-changelog/ "良いChangeLog、良くないChangeLog | Web Scratch")
- [われわれは、いかにして変更点を追うか](http://azu.github.io/slide/cto/changelog.html "われわれは、いかにして変更点を追うか")
