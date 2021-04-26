---
title: "git submoduleについてと主な使い方"
date: 2021-04-15T23:10:30+09:00
draft: false
tags:
- git
---


## 内容

git submoduleの使い方についてまとめる。

git submoduleコワクナイ！！

## 概要

### git submoduleとは

Git公式の説明は[こちら](https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E3%82%B5%E3%83%96%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB)。

ソース管理しているプロジェクトから別のプロジェクトのソースを利用したいな〜って時とかに利用する。
例えば、アプリケーションの開発をする際にログ出力など複数のアプリケーションに共通して実装する部分を切り出しして別のプロジェクトとして管理し、アプリケーションのプロジェクトではsubmoduleとして呼び出すようにしたりとか。。

特定のプロジェクトのソースをまるっとコピーしたりリンクでもいいんだけど。。いちいちコピーしたりバージョン管理が面倒だなぁという時に使えると便利。
個人的には「あっ。これ面倒くさいな」と思ったタイミングでsubmodule化を検討すればいいかと思う。

私のblogのプロジェクトもsubmoduleを利用している（コミットメッセージが適当すぎて少し恥ずかしい...）。

![image_1](/images/posts/20210416/git-submodule_01.png)

`@`以降にある英数字の文字列はsubmoduleとして追加(更新)した時のコミットハッシュを指している。

[リンク](https://github.com/bambi-san/blog/tree/gh-pages/themes)の`zzo`をクリックすると別のプロジェクト(zzossig/hugo-theme-zzo)に遷移する。

例では`290800d...`になっており、リンクよりzzoのプロジェクトに遷移するとコミットハッシュが自動的に`290800d...`になっているのが確認できる。

![image_2](/images/posts/20210416/git-submodule_02.png)

追加したsubmoduleのコミットハッシュがすぐに、明確にわかるおかげで追加した後の管理が非常に楽になる。わからないと参照しているリポジトリはいつのコミットだったか何かしらの方法で管理しなければならず、管理しているコミットハッシュと実際のコミットハッシュがズレてしまうと場合によってはバグに...

複数のsubmoduleを追加した時も同様にそれぞれのリポジトリで参照しているコミットハッシュが表示される。

また、submoduleは自動でソースが更新されることはないので注意。
(コミットハッシュを参照しているため自動更新されないことにより参照元の動作が担保されるので安心)

## git submodule 使い方

アプリケーションの開発をしていると仮定して、開発中のリポジトリ、参照先のリポジトリをそれぞれ以下の名前とする。

* 開発中のリポジトリ：`app_repo`
* submoduleとして参照される先のリポジトリ名：`submodule_repo`

### submoduleの追加

```bash
# git submodule add ${submoduleとして追加したいリポジトリのURL} ${submoduleを追加したいパス}
# コマンド入力配下に追加したい場合、${submoduleを追加したいパス}の指定は不要
$ git submodule add https://xxx/submodule_repo.git
```

`${submoduleを追加したいパス}`に相対パスを指定した場合はコマンドを実行したパスからの相対パスとなる。

上記のコマンドを実行すると以下4つのファイルやフォルダが修正、作成される。

1. `.gitmodules`
2. `.git/config`
3. `.git/modules/submodule_repo`
4. submoduleとして追加したリポジトリの中身

git submoduleの操作をしていてなんだかよく分からないがエラーが出てしまってどうしようもなさそうな時や最初からやり直したい時は
この3つの内容をそれぞれ以下のように初期化することで最初から操作することができる。
 * .gitmodules ファイル内の設定を削除 (submoduleの設定が一つの場合はファイルごと削除でもOK)
 * .git/config ファイル内にあるsubmoduleの設定を削除
 * .git/modules/submodule_repo フォルダを削除
 * 追加したsubmoduleのリポジトリフォルダを削除

中途半端に削除した状態で操作しようとすると`'xxx(submodule名)' already exists in the index`というエラーが出てきて深みにハマる。。。ﾋｨ
削除の仕方については`git rm`コマンドを使った方が良いか、`rm`コマンドなどでそのまま削除した方が良いかは`git status`で状態を確認しながら行うとよいかもしれない。

ちなみに、submoduleとして追加されたリポジトリのフォルダ配下には`.git`というファイルが作成され、中身をみると以下の様にgitdirという設定が書き込みされている。
これはgitの管理用のファイル群(.git)がどこにあるのかを示していて、submoduleの操作に限らずgitを初期化する際などにオプションをつけることで管理を分けることができる(`--separate-git-dir=`)。
普通に利用していて.gitを分けたいと思った場面に出会ったことがない(そういう話も聞いたことがない)ため利用する場面はあまりないかもしれない。。サブモジュール特有のものではないということだけ覚えておけばよさそう。
```
gitdir: ../.git/modules/submodule_repo
```

上記3つのファイル、フォルダにどのような内容が追加、作成されるのかについてざっくりにまとめる。

#### 1. .gitmodules

`git init`により初期化された状態ではこのファイルが作られることはなく、サブモジュールを追加することで作成されるファイル。
どのリポジトリをどんな名前で、どこにsubmoduleとして登録しているかの設定が追加される。

```ini
[submodule "submodule_repo"]
	path = submodule_repo
	url = https://xxx/submodule_repo.git
```

#### 2. .git/config

リポジトリのローカルブランチについての設定やsubmoduleの設定が記載されている。
末尾に以下のような設定が追加される。

```ini
[submodule "submodule_repo"]
         url = https://xxx/submodule_repo
         active = true
```

#### 3. .git/modules/submodule_repo

submoduleとして追加したリポジトリのgitに関する設定が入ってる。

#### 4. submoduleとして追加したリポジトリの中身

追加したリポジトリがごっそりそのまま追加される。


### submoduleの状態確認

submoduleとして参照しているリポジトリについて、どのコミットハッシュを参照しているのか確認することができる。
(私自身は今までこのコマンドあまり使ったことないけど...使えると便利なのかも)

```bash
$ git submodule status
# 7xxxx1966a4b3bba7391322a3a151xxxxxxxxxx submodule_repo (heads/main)
```
### submoduleの更新

git submodule コマンドを使った方法とsubmoduleプロジェクト配下で更新する方法がある。
複数管理していて一度に全てを更新したい時は前者の方が、特定のsubmoduleのみをupdateしたいときは後者の方が便利。

#### git submodule コマンドを利用した更新方法

fetch は通常の`git fetch --all`コマンドと同じ意味合いのため、単に最新版に追従したいだけであれば2つ目のコマンドのみで良い。
submoduleが複数ある場合は複数のsubmoduleに対して同じ操作が行われる。submoduleとして追加したリポジトリの状態によってはコマンド実行時にエラーが出る場合もあるため注意する。
(例えば以下の場合、`master`ブランチが存在しないリポジトリをsubmoduleとして登録しているとgit pullに失敗しエラーが発生する)

```
$ git submodule foreach git fetch --all
$ git submodule foreach git pull origin master
```

#### submoduleプロジェクト配下で更新する方法

`submoduleとして登録したリポジトリ(フォルダ)`配下 or `.git/modules/対象のsubmodule`配下 でそれぞれgitの操作を行う。
今回はプロジェクトroot配下にsubmodule_repo の名前で登録しているため、実行するときは以下のようになる。

```bash
# submoduleとして登録したリポジトリ(フォルダ) 配下で更新する場合
$ cd submodule_repo
$ git fetch
$ git merge master

# .git/modules/対象のsubmodule 配下で更新する場合
$ cd .git/modules/submodule_repo
$ git fetch
$ git merge master
```

.git/modules/対象のsubmodule 配下で実行する方法は少し奇妙に感じるが、サブモジュールを追加するとそのリポジトリのgit管理は`.git/modules/追加したsubmodule` で行われる(submoduleフォルダ配下の.gitファイル参照)ため、サブモジュールとして登録したリポジトリ(フォルダ)配下で更新
するのと変わらない。やりやすい方でやればよさそう。

submoduleを更新したあとは本体(今回の例ではapp_repo)でコミットするのを忘れないように。。。

### submoduleの削除

```bash
$ git submodule deinit submodule_repo

$ rm -rf submodule_repo

# .gitmodules を見て設定が残っていたら削除する
```

上記のコマンド実行でうまくいかない時はsubmoduleを追加した時に追加されたリソースを初期化・削除すればOK。

## 参考

わかりやすい素敵な記事がたくさんありました。ありがとうございます。

* https://www.m3tech.blog/entry/git-submodule
* http://transitive.info/article/git/command/submodule/
