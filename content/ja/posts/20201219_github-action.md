---
title: "Github Pagesのドメイン設定が初期化される"
date: 2020-12-19T18:10:30+09:00
draft: false
categories:
- Blog制作
tags:
- github
---

## 概要

Hugoで自分専用のブログサイトを作ったものの、記事を追加する度にDomainの設定が初期化されるという問題発生。  
このままだと記事を描くのが億劫になってしまうので対処する。

## 対処内容

こちらが大変参考になりました。
* https://tech-wafter.net/2020/deploy-custom-domain-github-pages-on-github-actions/
* https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-disable-nojekyll

今のCIの設定をみてみると確かに`cname`入ってなかった。。。追加してpushしたらドメインの設定そのままになっていた。  
よかった！！！
```yaml
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          publish_dir: ./public
          publish_branch: main 
          cname: www.anypalette.com  # <-追加した
```
