---
title: "【Terraform】providerを作ってみる"
date: 2021-01-09T19:10:30+09:00
draft: true
tags:
- terraform
---

## 概要

作りたかったのものがあったのでやってみる。
どう作ればいいのか全くわからんのでそれを調べるところから。

## 作業

### terraform providerについて調べる

いくつか日本語のサイトと公式HPを読んでふんわり概要を理解。     
日本語のものはあまりなかったので助かりました。。。
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

今更だけどterraform のアーキテクチャって素敵！！
（今までよく理解してなくてごめんなさいorz）

### 試しに作ってみる

今回は試しにredashのdata_sourceの操作を実装してみる。  
ま図、terraform実行時に発行されるAPIのリクエスト情報を確認する。
    
```bash
$ curl http://127.0.0.1:5000/api/data_sources?api_key=YOUR_API_KEY
[
    {
        "id": 1,
        "name": "python",
        "type": "python",
        "syntax": "python",
        "paused": 0,
        "pause_reason": null,
        "view_only": false
    },
    {
        "id": 2,
        "name": "test",
        "type": "json",
        "syntax": "yaml",
        "paused": 0,
        "pause_reason": null,
        "view_only": false
    },
    {
        "id": 3,
        "name": "world",
        "type": "mysql",
        "syntax": "sql",
        "paused": 0,
        "pause_reason": null,
        "view_only": false
    }
]
```

↑をschemaに設定する。schemaに設定する時の型（`Type`）で何を指定すれば良いかわからないときは
[Schema Attributes and Types](https://www.terraform.io/docs/extend/schemas/schema-types.html)
を参考にするといい。




## 参考

とても参考になりました。ありがとうございました。

* [febc技術メモ(Terraform Provider実装 入門(1): Custom Providerの基礎)](https://febc-yamamoto.hatenablog.jp/entry/terraform-custom-provider-02)
* [HashiCorpLearn(CustomProvider)](https://learn.hashicorp.com/tutorials/terraform/provider-setup?in=terraform/providers)
