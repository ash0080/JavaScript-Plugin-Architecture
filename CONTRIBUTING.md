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

例如,要提交一个关于库/工具 xxx 的 issue.
たとえば、XXXというライブラリ/ツールのアーキテクチャについてのIssueを立てる場合、
次のようなことが1行とかでもいいので書かれていれば参考になります。

如果某章节的内容
仕組みについて調べるのが大変な場合は飛ばしても問題ありません。
JavaScriptはとにかく柔軟な言語なので、こういうプラグインの形式を取ってるというのを知らせるだけでも有用だと思います。

# XXXのアーキテクチャ

## どう書ける?

- 実際のコード例

## どういう仕組み?

- prototypeを拡張しているなど具体的な仕組み
- その機構のコードへのリンク
- その仕組みやプラグインについてドキュメントへのリンク

## どういうことに向いている?

- どういう用途で使われてる(ユースケース)
- 変換する毎にファイルとして吐き出さないので、高速に複数の変換をするのに向いている等

## どういうことに向いていない?

- 変換の仕組み上書き換え等を行うプラグインを扱いにくい等

## この仕組みを使っているもの

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

以下からこのテンプレートで使ったIssueを立てることができます。

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
