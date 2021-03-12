---
title: "base64コマンドを使って文字をエンコード/デコードする"
date: 2021-03-10T11:10:30+09:00
draft: false
tags:
- linuxコマンド
---

base64コマンドバージョン：8.22の場合

### エンコード

```bash
$ echo 'password' | base64
cGFzc3dvcmQK
```

### デコード

```bash
$ echo 'cGFzc3dvcmQK' | base64 -e
password
```

簡単なのにいざ使おうとすると使い方忘れててググることが多い(汗
