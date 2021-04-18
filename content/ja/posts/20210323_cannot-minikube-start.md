---
title: "minikubeの起動に失敗した時の対処"
date: 2021-03-15T11:10:30+09:00
draft: false
tags:
- minikube
---

## 内容

久しぶりにMacでminikube起動しようと思ったら起動に失敗した。  
minikubeをupgradeしたり`miniube delete`したりしたけど全然起動できない、、、詰む。

```
minikube start --driver=hyperkit 

😄  Darwin 11.2.2 上の minikube v1.16.0
✨  プロフィールを元に、 hyperkit ドライバを使用します
👍  コントロールプレーンのノード minikube を minikube 上で起動しています
🔄  既存の hyperkit VM を "minikube" のために再起動しています...
🐳  Docker 20.10.0 で Kubernetes v1.20.0 を準備しています...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Kubernetes コンポーネントを検証しています...
❗  'default-storageclass' を有効にする際にエラーが発生しました。running callbacks: [Error making standard the default storage class: Error listing StorageClasses: Unauthorized]
🌟  有効なアドオン: storage-provisioner

❌  Exiting due to GUEST_START: wait 6m0s for node: wait for healthy API server: controlPlane never updated to v1.20.0

😿  If the above advice does not help, please let us know:
👉  https://github.com/kubernetes/minikube/issues/new/choose
```

## 内容

minikubeのissueに解決策あった。

あんまり古いバージョンからupgradeするときは`.minikube`フォルダを削除する必要があり、`minikube delete --all --purge`すべしとな。

参考：[https://github.com/kubernetes/minikube/issues/8844]https://github.com/kubernetes/minikube/issues/8844

```
minikube delete --all --purge    
🔥  hyperkit の「minikube」を削除しています...
💀  クラスタ "minikube" の全てのトレースを削除しました。
🔥  Successfully deleted all profiles
💀  Successfully purged minikube directory located at - [/Users/xxx/.minikube]
```

その後ようやく起動に成功した。

うまくいかないはずがないと思ってupgrade前のバージョン控えてなかったが、相当古かったんだろう。。