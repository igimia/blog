---
title: "【Go 1.13】VsCodeでデバッグできなくなる"
date: 2021-06-21T14:46:36+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: false # 目次
tocPosition: inner
tags:
- go
- gin
- vscode
series:
-
categories:
-
---

## 概要

Go言語で、VsCodeからデバッグしようとしたところ急にデバッグできなくなる事象が発生した。
その時のメモ（対応した時から記事に起こすのにかなり時間がかかってしまった。。。）
デバッグ時に表示されたエラー↓
```
API server listening at: 127.0.0.1:48745
Version of Go is too old for this version of Delve (minimum supported version 1.14, suppress this error with --check-go-version=false)
Process exiting with code: 1
```

### バージョン

* Go: `v1.13.4`

### 原因と対応内容

VsCodeのコマンドパレットから`Go: Install/Update Tools`をダウンロードしたところ、dlv(1.6.0)がインストールされた。
このバージョンはv1.14(Go)より前のバージョンはサポートしていないらしい。。

インストール済のdlvを一旦削除する(複数ある場合は全部)。

```bash
$ which dlv
/Users/hoge/.goenv/shims/dlv

$ rm /Users/hoge/.goenv/shims/dlv
```

キャッシュを削除してからバージョンを指定してモジュールをインストールする。

```bash
$ go clean -modcache
$ go get github.com/go-delve/delve/cmd/dlv@v1.3.2
```

いつも通りVsCodeからデバッグできた。
