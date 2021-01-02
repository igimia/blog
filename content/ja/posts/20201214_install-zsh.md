---
title: "zsh, preztoのセットアップ【Mac】"
date: 2020-12-14T11:10:30+09:00
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
- zsh
- mac
---

## 概要

macをもらってまずやるzshとpreztoのセットアップ手順についてまとめる。  
zshとpreztoが何かについては割愛する。

### 作業手順

#### zshのセットアップ

最近のMacはログインシェルの設定がデフォルトで`zsh`になっている。  

`chsh -s /bin/zsh` でログインシェルの変更が可能だが、Macの場合システム環境設定にログインシェルの設定がある。  
iTerm2を利用している場合、コマンドからログインシェルを変更してもシステム環境設定からログインシェルの変更をしないと反映されないため注意が必要。

##### ログインシェルの確認方法

`システム環境設定 → ユーザとグループ` を選択

![zsh_01.png](/images/posts/20201214/zsh_01.png)

鍵マークをクリックし、ユーザID, パスワードを入力

![zsh_02.png](/images/posts/20201214/zsh_02.png)

`ユーザ情報`を右クリックし、`詳細オプション` を選択する

![zsh_03.png](/images/posts/20201214/zsh_03.png)

ログインシェルに設定されているシェルの種類を確認する。

![zsh_04.png](/images/posts/20201214/zsh_04.png)

#### preztoのセットアップ

macにzsh入っていたため続いてpreztoのセットアップを行う。  
preztoのリポジトリをcloneする

```bash
$ git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"
```

インストール, prezto設定ファイルの作成

```bash
$ setopt EXTENDED_GLOB
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done
```

設定ファイルの修正(~/.zpreztorc)
```bash
$ vi ~/.zpreztorc

# 以下を貼り付ける
zstyle ':prezto:load' pmodule \
  'environment' \
  'terminal' \
  'editor' \
  'history' \
  'directory' \
  'spectrum' \
  'utility' \
  'completion' \
  'autosuggestions' \
  'syntax-highlighting' \
  'git' \
  'prompt'
```

#### powerline のインストール、設定

preztoなどのテーマにはpowerlineというフォントが使われていることが多くあり、デフォルトでインストールされていないためそのままだと文字化けして表示されてしまう。   
文字化けしないようにインストールする。

iTerm2やVSCodeを利用している場合は、そちらのterminalも文字化けしないように設定する必要がある。

##### インストール

git(powerline/fonts) をcloneしてインストール用のshellを叩くのが良い。

```bash
$ git clone https://github.com/powerline/fonts.git

$ sh ./install.sh
Copying fonts...
Resetting font cache, this may take a moment...
Powerline fonts installed to /Users/ayaka/Library/Fonts
```

##### 設定（VSCode）

VSCodeから設定（`setting.json`）を開く。   
検索ボックスに `terminal.integrated.fontFamily` を入力し検索する。

テキストボックスに `Source Code Pro for Powerline` を入力する。terminalの文字化けが直らない場合はVSCodeを再起動する。

##### 設定（iTerm2）

`profiles → Text → Font` より好きなフォントを選ぶ。

### 参考
参考にさせていただきました。ありがとうございます。  
* https://qiita.com/s_s_satoc/items/e3c1b9b3545fd572dd1c




