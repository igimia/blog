---
title: "Re:VIEWの使い方(v5.1)"
date: 2021-05-02T14:46:36+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
- Re:VIEW
series:
-
categories:
- 文書作成
---

## 概要

書籍を作成するのに非常に便利なツール[Re:VIEW](https://reviewml.org/ja/)の環境構築や使い方について。
久しぶりにRe:VIEWを使いたいなぁと思ったら色々忘れかけてたので思い出しながらまとめた。
製本（reviewコマンドの実行）はコンテナでやる。

## やってみる

### 事前準備

作業用フォルダを作成する

```bash
$ mkdir sample-book
```

#### 実行環境の作成

reviewの実行環境を作成する。
ローカルにrubyの実行環境を作成するのが少し手間なのでコンテナ上で実行する。　　
こちら（([vvakame/review](https://hub.docker.com/r/vvakame/review))）と[Re:VIEW(Git)](https://github.com/kmuto/review)のQuick Startを参考に作成し実行する。

ボリュームの永続化設定を忘れずに。

```bash
$ vi docker-compose.yml
```

```yaml:docker-compose.yml
version: '3'
services:
  review:
    image: vvakame/review:5.1
    volumes:
      - .:/work
    working_dir: /work
    # 作業用のフォルダを作成するためのコマンド.
    command: >
      sh -c "
        review init hello
      "
```

実行する。
```bash
$ docker-compose up
# Starting sample-book_review_1 ... done
# Attaching to sample-book_review_1
# sample-book_review_1 exited with code 0
```

完了後、helloフォルダが作成されている。
```bash
$ ls hello
Gemfile     catalog.yml doc         images      lib         style.css
Rakefile    config.yml  hello.re    layouts     sty
```

#### 製本する

試しに現時点の設定でPDF形式で製本する。

```bash
# helloディレクトリ配下のRe:VIEWを利用するのに必要なセットをdocker-composeと同じ階層に移動する
$ mv hello/* .
```

コンテナ起動時のコマンドを書き換え、実行する

```bash
$ vi docker-compose.yml
# "review init hello" -> "review-pdfmaker config.yml" に書き換える

$ docker-compose up
# Recreating sample-book_review_1 ... done
# Attaching to sample-book_review_1
# review_1  | INFO: compiling hello.tex
#  :
# (省略)
#  :
# sample-book_review_1 exited with code 0
```

完了後、実行したフォルダ配下をみるとpdfファイルが作成されてる。
開いてみるとこんな感じ。

![image_1](/images/posts/20210502/review_01.png)

### カスタマイズする

自分用に設定やフォルダ構成をカスタマイズしていく。

#### config.ymlの設定

全ての設定にコメントで注釈が書いてあり、非常にわかりやすい。
とりあえず以下の設定を修正。かき始めたらまた適宜修正する

```yaml:config.yml
booktitle: 2021年テスト書籍
aut: ["BAMBi"]

# コメントされているので外す。
highlight:
  html: "rouge"
  latex: "listings"

# 本編はこのディレクトリ配下に書いていく
contentdir: contents
```

contentdirで指定したフォルダを作成し、`.re`ファイルはcontentsフォルダ配下に移動する。

```bash
$ mkdir contents
```

#### catalog.ymlの設定

本全体の構成をここで設定する。
どうなるかわからないので適当に仮決めで設定。

```yaml:catalog.yml
PREDEF:
  - intro.re

CHAPS:
  - ch00/01.re
  - 第一部:
    - ch01/01.re

APPENDIX:

POSTDEF:
  - postscript.re
```

設定した内容と同じになるように`contents`フォルダ配下に`.re`ファイルを作成していく。

#### フォルダ構成

このような構成にした。

```text:フォルダ構成
.
├── Gemfile
├── Rakefile
├── book.pdf
├── catalog.yml
├── config.yml
├── contents
│   ├── ch00
│   ├── ch01
│   ├── intro.re
│   └── postscript.re
├── doc
├── docker-compose.yml
├── images
├── lib
└── sty
```

### 文章校正ツールを設定する

[textlint](https://textlint.github.io/)を使って"てにをは"とかをチェックしてもらう。
最後はCIに設定してpushの都度チェックするようにしたい。

#### Dockerfileの作成

プロジェクトルート直下に`textlint`を作成し、その下にDockerfileを作成する。
以下のプラグインを使えるようにインストールしておく。

* [textlint-plugin-review](https://github.com/orangain/textlint-plugin-review): Re:VIEWの文書をtextlintでチェック対象とするためのプラグイン
* [textlint-filter-rule-comments](https://github.com/textlint/textlint-filter-rule-comments): lint対象の文書について、任意の箇所を対象外とすることができるプラグイン
* [textlint-rule-prh](https://github.com/prh/prh): 表記揺れに関するルール。リポジトリ内の`WEB+DB_PRESS.yml`を校正のルールとして利用する。
* [textlint-rule-preset-ja-technical-writing](https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing): 技術文書向けのルール
* [textlint-rule-preset-ja-spacing](https://github.com/textlint-ja/textlint-rule-preset-ja-spacing): 日本語のスペース有無に関するルール

```Dockerfile:textlint/Dockerfile
FROM node:15.5.1-stretch-slim

RUN npm install -g textlint \
    && npm install -g \
       textlint-plugin-review \
       textlint-rule-prh \
       textlint-rule-preset-ja-technical-writing \
       textlint-rule-preset-ja-spacing \
       textlint-filter-rule-comments
WORKDIR /work
```

#### docker-compose.ymlの修正

textlintが利用できるようにサービスを追加する。

```yaml:docker-compose.yml
  textlint:
    build:
      context: .
      dockerfile: ./textlint/Dockerfile
    volumes:
      - .:/work
    working_dir: /work/textlint
    # textlintの初期化
    command: >
      sh -c "
        cd textlint
        textlint --init
       "
```

作成後、`docker-compose up`を実行するとtextlintフォルダ配下に`.textlintrc`が作成される。

#### ルールの設定

##### ルールファイルのダウンロード
[こちら](https://github.com/prh/rules/tree/master/media)にある2つのルールファイルをどこかに保存する。
(サンプルは`textlint/rules`配下に保存している)

##### .textlintrcの修正

インストールしたプラグインを利用する設定を追加する。
執筆作業を進める中で適宜修正する。

```json:.textlintrc
{
  "plugins": [
    "review"
  ],
  "filters": {
    "comments": true
  },
  "rules": {
    "prh": {
      "rulePaths": [
        "./rules/WEB+DB_PRESS.yml",
        "./rules/techbooster.yml"
      ]
    },
    "preset-ja-technical-writing": true,
    "preset-ja-spacing": true
  }
}
```

##### docker-compose.ymlの修正

textlintのcommandを修正する。
引数に指定しているフォルダ名は校正したいファイル(以下の例のように正規表現を利用して設定することもできる)を設定する。

オプションはお好みで。。。

```yaml
    command: >
      sh -c "
        textlint ../contents/*.re -f pretty-error
       "
```

設定後`docker-compose up textlint` でtextlintのみを実行することが可能になる。

↓サンプルプロジェクトで実行した時の結果。

```text:コマンド実行結果ログ
docker-compose up textlint
Starting sample-book_textlint_1 ... done
Attaching to sample-book_textlint_1
textlint_1  | ✓ prh: はじめ => 始め
textlint_1  | /work/contents/intro.re:1:3
textlint_1  |           v
textlint_1  |     0.
textlint_1  |     1. = はじめに
textlint_1  |     2. ここに前書きをかく
textlint_1  |           ^
textlint_1  |
textlint_1  | ja-technical-writing/ja-no-mixed-period: 文末が"。"で終わっていません。
textlint_1  | /work/contents/intro.re:2:9
textlint_1  |                         v
textlint_1  |     1. = はじめに
textlint_1  |     2. ここに前書きをかく
textlint_1  |     3.
textlint_1  |                         ^
textlint_1  |
textlint_1  | ja-technical-writing/ja-no-mixed-period: 文末が"。"で終わっていません。
textlint_1  | /work/contents/postscript.re:2:10
textlint_1  |                           v
textlint_1  |     1. = あとがき
textlint_1  |     2. ここにあとがきを書く
textlint_1  |     3.
textlint_1  |                           ^
textlint_1  |
textlint_1  | ✖ 3 problems (3 errors, 0 warnings)
textlint_1  | ✓ 1 fixable problem.
textlint_1  | Try to run: $ textlint --fix [file]
textlint_1  |
sample-book_textlint_1 exited with code 1
```

### GitHub Actionsの設定

最後にGitHubActionsの設定を行う。
後述するサンプルプロジェクトに全て置いてあるので良かったら使ってください。

設定したジョブと内容について

1. `lint`: textlintを利用した文章の校正を行う
2. `deploy`: 製本する(lintの結果によらず必ず実行する。結果はCI実行から5日間以内であればダウンロードが可能。)

以下改善点
* lintのjobをテンプレート化して引数とかでフォルダをlistで渡して実行できるようにしたい。
   文書が増えてきて章ごとにフォルダが別れた時lintの行を増やさないといけないのが若干面倒なので。そもそもできるかわからないけど多分できそう。
* ジョブが完了したら結果をSlackの通知に加えても良さそう。
* lintが失敗した場合に、textllintのオプション`--fix`を使って自動修正してその内容でプルリク作成するジョブがあると便利。

GitHub Actionsでは`docker-compose`コマンドが使えるとのことで試してみたら爆速でCIできた。

```yaml:.github/workflows/review-workflow.yml
# project root配下に.github/workflows/review-workflow.yml を作成する。

name: review-github-actions
on: [push]
jobs:
  lint:
    name: lint-contents
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
      - run: docker-compose build textlint
      - run: docker-compose up textlint

  deploy:
    name: make-book
    # lintが失敗しても製本は常に行う
    if: ${{ always() }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: docker-compose up review
      # 成果物をWeb上からダウンロードできるように設定する
      - name: 'Upload Book'
        uses: actions/upload-artifact@v2
        with:
          name: mybook
          path: book.pdf
          retention-days: 5  # 5日間ダウンロードが可能になる
```

CIの挙動が`docker-compose.yml`に依存するのは個人的にモヤッと感があるのでdocker-composeコマンド使わない版も作成した。

```yaml:review-workflow.yml
name: review-github-actions
on: [push]
jobs:
  lint:
    name: lint-contents
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
      - name: npm install
        run: |
          npm install -g textlint
          npm install -g textlint-plugin-review textlint-rule-prh textlint-rule-preset-ja-technical-writing textlint-rule-preset-ja-spacing textlint-filter-rule-comments
      - name: textlint
        run: |
          cd textlint && \
          textlint ../contents/*.re \
          -f pretty-error
  deploy:
    container: vvakame/review:5.1
    name: make-book
    # lintが失敗しても製本は常に行う
    if: ${{ always() }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      # 成果物をWeb上からダウンロードできるように設定する
      # 出力形式を変更する際は後述するuploadのファイル名も変更する
      - run: |
          sh -c "review-pdfmaker config.yml"
      - name: 'Upload Book'
        uses: actions/upload-artifact@v2
        with:
          name: myboo
          path: book.pdf
          retention-days: 5  # 5日間ダウンロードが可能になる
```

製本ジョブが成功するとGitHub画面上のActionsタブから成果物をダウンロードすることができる。
![image_2](/images/posts/20210502/review_02.png)

### 最終的なプロジェクトの構成とサンプルプロジェクトについて

#### プロジェクト構成

```text
.
├── Gemfile
├── Rakefile
├── book.pdf
├── catalog.yml
├── config.yml
├── contents             <- このフォルダ配下に記事を書いていく
│   ├── ch00
│   ├── ch01
│   ├── intro.re
│   └── postscript.re
├── doc
├── docker-compose.yml
├── images
│   ├── cover-a5.ai
│   └── cover.jpg
├── lib
│   └── tasks
├── sty
├── style.css
└── textlint            <- textlintの設定はこの配下で行う
    ├── Dockerfile
    ├── intro.re
    ├── .textlintrc
    └── rules
```

#### サンプルプロジェクトについて

GitHub Actionsの設定までが入っているものを以下URLにpush済みです。良かったら何かにお使いください。
https://github.com/bambi-san/createbook-sample.git

### 参考

* [textlint公式](https://textlint.github.io/)
* [textlint校正ツールの日本語周りスペースルールをカスタマイズする方法](http://my-web-site.iobb.net/~yuki/2018-05/e-book/review-textlint-ja-spacing/)
* [textlint と VS Code で始める文章校正](https://qiita.com/takasp/items/22f7f72b691fda30aea2#%E8%A1%A8%E8%A8%98%E6%8F%BA%E3%82%8C---textlint-rule-spellcheck-tech-word)
* [textlintでRe:VIEWの原稿をチェックする](https://backport.net/blog/2019/02/09/textlint_for_review/)
