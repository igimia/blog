---
title: "【Terraform】providerで詰まったところ"
date: 2021-01-15T19:10:30+09:00
draft: true 
tags:
- terraform
---

## 概要

custom-providerを作りたかったのだが、詰まったので詰まったところをメモする。

### custom-providerを作ることがなんなのかわからない

go言語を使うとか、提供されていないリソースの管理をすることができるとかはわかったのだが、
作ることで何がどうなるのかとか全然わからなかった。。。

いくつか日本語のサイトと公式HPを読んでふんわり概要を理解。     
日本語のものはあまりなかったので助かった。。。
(最後の参考にも同様のものを記載)
* [febc技術メモ(Terraform Provider実装 入門(1): Custom Providerの基礎)](https://febc-yamamoto.hatenablog.jp/entry/terraform-custom-provider-02)
* [HashiCorpLearn(CustomProvider)](https://learn.hashicorp.com/tutorials/terraform/provider-setup?in=terraform/providers)

とりあえず試しに動かしてみたいときは[Perform CRUD operations with Providers(公式HP)](Phttps://learn.hashicorp.com/tutorials/terraform/provider-use?in=terraform/providers)をやってみると良い。

調べてみて、手を動かしてみてリソースが適用されるまでの仕組みがようやくわかった。  
![image_1](/images/posts/20210109/terraform-provider_01.png)
`terraform plan`や`apply`が実行されるとterraform core は各プロバイダーやプロビジョナーに対し「このリソースをCRUDしたいからやってね」と依頼し、
プロバイダーやプロビジョナーはそれぞれのクライアントライブラリに対しAPIを発行することで要求を実現するということらしい。

ここで言うクライアントライブラリとは、例えばAWSであればAWS APIのこと。
今回私はRedashのプロバイダーが作りたかったので、[RedashのAPIドキュメント](https://redash.io/help/user-guide/integrations-and-api/api)になるわけだ。

どのリソースも基本的にやりたいことはCRUDなので、`.tf`ファイルと`state`の差分からCRUD情報を生成するところまではcoreが共通して担当し、
APIを発行するときは操作したいものによって指定の方法が違うので、それをproviderで作るということか。  
そうすることでcoreは操作したいリソースのAPIリファレンスについて知る必要はなく、互いが疎になるという。「プロバイダーはクライアントライブラリを抽象化したものとして機能する」とはそういうことか。

今までよくわかってなかったproviderの謎が少し解けた。

### custom-providerを作った後どう使うのかがわからない

これもそもそもの話になってしまうが。。。
providerを作ったあと何をどうすれば利用できるのかがわからなかった。。。

今回はmacを利用していたのでOSは`darwin_amd64`を指定した。
適当なフォルダを用意して、`main.tf`ファイルを作成し以下のような設定を記述する。

```main.tf
terraform {
  required_version = ">= 0.13.0"
  required_providers {
    redash = {                                     <- provider自体の名前（terraform-provider-{xxx}）
      source = "bambi-project.io/sample/redash"    <- sourceの場所を設定する
      version = "0.0.9"
    }
  }
}

provider redash {}      <- providerを利用するための宣言をする
```

上記の設定が終わった後、下記のフォルダを作成し、フォルダ配下に作成したproviderのバイナリファイルを設定する。

```
$ ~/.terraform.d/plugins/bambi-project.io/sample/redash/0.0.9/darwin_amd64
```
