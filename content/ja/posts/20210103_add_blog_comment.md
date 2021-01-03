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

3. ブログ（HUGO）の設定追加
zzoテーマの`layouts/_default/single.html`をみてみると、コメントのテンプレートを利用しているところがあった。
    ```html:single.html
    </article>

    {{ partial "script/clipboard-script" . }}
    {{ partial "script/codeblock-script" . }}
    {{ partial "body/share" . }}
    {{ partial "body/donation" . }}
    {{ partial "body/whoami" . }}
    {{ partial "body/related" . }}
    {{ partial "pagination/pagination-single" . }}
    {{ partial "comments/comments.html" . }}
    {{ partial "body/photoswipe" . }}

    <div class="hide">
    ```
    `partials/comments/comments.html`をみてみるとparamsの`enableComment`と`disqus_shortname`
    に設定が入っていれば有効化されそう。
    ```html:comments.html
    {{ if $.Param "enableComment" }}
      {{ if $.Param "disqus_shortname" }}
        {{ partial "comments/disqus.html" . }}
      {{ end }}
    {{ end }}
    ```
    `config/_default/params.toml`の設定を修正する。
    disqus_shortnameはdiaqusの`General -> Shortname`から確認する。
    ```toml:params.toml
    enableComment = true
    disqus_shortname = "your_disqus_shortname"
    ```
    出てきた。
    
## 参考

参考にさせていただきました。ありがとうございました。

* [Hugo で作ったブログに Disqus を使ってコメント機能を追加する](https://michimani.net/post/blog-install-disqus-to-hugo/)
* [Add Disqus(HUGO公式)](https://gohugo.io/content-management/comments/)