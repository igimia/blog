---
title: "Hugo + GithubPages でブログを構築する"
date: 2020-10-12T20:40:51+09:00
description: hugo, github
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
- hugo
- github
series:
-
categories:
- Blog制作
image: images/feature2020/1012_1.jpg
---


## 経緯

今までメモがてら使っていたサイトの閉鎖に伴い、いよいよ自分で作ることを決意。
GitHub Pagesを使うとタダ！でできるのでそれでやってみる。
静的サイトジェネレータはいくつかあったが、お試しでHUGOでやってみた。

## 作ってみる

### インストール, サンプル起動

[公式](https://gohugo.io/getting-started/quick-start/)の手順を見ながら構築していく

```bash
$ brew install hugo
$ hugo version
# -> Hugo Static Site Generator v0.76.3/extended darwin/amd64 BuildDate: unknown

$ hugo new site quickstart
$ cd quickstart
$ git init
```

themeは[公式](https://themes.gohugo.io/zzo)から自分の気に入るのを探してsubmodule化して追加する（`zzo`を利用することにした）。
```bash
$ git submodule add https://github.com/zzossig/hugo-theme-zzo.git themes/zzo
```

もうblogを立ち上げられるらしい。
試しに立ち上げてみる
```bash
$ cd themes/zzo/exampleSite
$ hugo server --themesDir ../..
# Serving pages from memory
# Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
# Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

`http://localhost:1313/`にアクセスしたら公式のthemeと同じものが出てきた。しゅごい...

### CI/CDを設定する

GitHub Actionを使い、ビルド〜デプロイを自動でやってもらうようにする。

[ここ](https://github.com/peaceiris/actions-gh-pages)を見ながらこんな感じのCIを作成した。
gh-pagesブランチにpushすると発火し、`main`ブランチにデプロイされる。

```yaml 
name: github pages

on:
  push:
    branches:
      - gh-pages  # 発火させたいブランチ名を設定

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          publish_dir: ./public
          publish_branch: main  # デプロイ対象のブランチ名を設定
```

GitHub Actionを利用するのは初めてだったが...少し詰まった

* GitHub_Token を利用すると初回デプロイに必ず失敗する
  公式にも説明があるが、GitHub_Tokenには利用するのに制限があるらしく、初回デプロイ時に失敗するらしい。
  正直よくわらからなかったと、deploy_keyを利用してSSH経由でpushすればそういう制限はないらしいので使うのをやめた。
* ACTIONS_DEPLOY_KEY の利用に失敗する
  Secretsに登録した変数の名前を変えたらデプロイに失敗した。GitHub Pages専用変数のようなので名前を変えてはいけないよう。

### 自分好みにカスタマイズする

#### 準備

[Zzo](https://themes.gohugo.io/hugo-theme-zzo/)公式に細かい設定が書いてある。が、一旦テンプレートをそのまま利用するのでカスタマイズする前に`themes/zzo/exampleSite/`配下をrootにまるっとコピーしておくと良い。

Zzoの公式に全て書いてあるが...今回私がカスタマイズしたところはこのあたり。

#### ホーム画面

* タイトル,サブタイトルの変更
  `content/en/_index.md` このファイルをいじると色々カスタムできる。
  `en`は使う言語に応じて設定を修正する（まだjp設定できていないので一旦en配下を修正した）。

#### プロフィール

* アバターの変更
  `static/images/whoami/avatar.jpeg` のファイルを適用したいものに差し替える

* 名前、プロフィール文の変更
  `config/_default/params.toml`ファイルを修正する。
  名前やプロフィール、SNSへのリンク先は`whoami`あたりを修正するとよい。

#### 言語

* デフォルト言語の変更
  デフォルトは英語なので日本語に設定変更する。

  ```toml:config.toml
  defaultContentLanguage = "ja"
  ```

* 言語の設定にjaを加える

  ```toml:languages.toml
  [ja]
    title = "Hugo Zzo Theme"
    languageName = "日本語"
    weight = 1
  ```

* content/[言語] のコピー
  上記の設定だけではエラーが出てきた。
  contentなど言語ごとにフォルダが分かれているものがあったので`ja`用を作成する。

  ``` bash
  $ cp content/en content/ja
  ```
  何も表示されない。まだ何か足りないらしい。
  ![image_2](/images/posts/20201012/1012_2.jpg)
  config配下に`menus.en.toml`があった。
  細かいチューニングは置いといて、一旦Copy、Renameしてそのままjaファイルとする。
  ```bash
  $ cp -a config/_default/menus.en.toml config/_default/menus.ja.toml
  ```

  ![image_1](/images/posts/20201012/1012_1.jpg)
  コンテンツ表示された。心なしかあひるも少し嬉しそう。

* 表示言語切り替え機能の無効化

  英語はしばらく使うことなさそうだがそのうち英語でも...という日が来るかもしれないのでそのままにしておくことにする。
  その間表示言語の切り替え機能は無効化しておく。

  ```toml:params.toml
  # footer
  showPoweredBy = true
  showFeedLinks = true
  showSocialLinks = true
  enableLangChange = false  # ここを修正
  enableThemeChange = true
  ```


## 参考

構築時にとても参考になりました。

* GitHub Pages × Hugo で技術ブログを始めた: https://reona.dev/posts/20200331

* Hugoで記事内に画像を貼り付ける方法 : https://qiita.com/atuyosi/items/4100bd502e373c088c74

* Hugo(quickstart) : https://gohugo.io/getting-started/quick-start/

* GitHubAction(GitHubPages, Hugo) : https://github.com/peaceiris/actions-gh-pages

