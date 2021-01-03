---
title: "ブログにコメント機能を追加する(disqus)"
date: 2021-01-03T19:10:30+09:00
draft: false
tags:
categories:
- Blog制作
---

## 概要

コメントを投稿できる機能を追加する。  
調べてみたら`disqus`を利用した方法が多かったのでそれを利用してさっくりやってみる。

## やってみる

1. ユーザ登録  
[disqus](https://disqus.com)にアクセスし、SignUpする。

2. サイトの登録  
利用したいサイトを登録する。私の場合こんな感じ。
![disqus_1](/images/posts/20210103/disqus_01.png)  
コメントの機能だけ使いたいのでBasic（無料）を選択する。  
![disqus_2](/images/posts/20210103/disqus_02.png)
対応するプラットフォームがなかったためinstall manuallyを選択する。   
![disqus_3](/images/posts/20210103/disqus_03.png)
コメント入力時のcolor schemaなどdisqusに関する設定を行う。     
今回はこんな感じにした。  
![disqus_3](/images/posts/20210103/disqus_03.png)
モデレーションのタイプを選択する。   
後から変更可能とのことなので一旦strictを設定。




## 参考

参考にさせていただきました。ありがとうございました。

* [Hugo で作ったブログに Disqus を使ってコメント機能を追加する](https://michimani.net/post/blog-install-disqus-to-hugo/)
* [Add Disqus(HUGO公式)](https://gohugo.io/content-management/comments/)