---
title: "flask + mysql の環境をDocker（docker-compose）で構築する"
date: 2019-12-01T20:40:51+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
- docker
series:
-
image:
---

{{< notice warning "注意事項" >}}
この記事は`2019/12/01`にQrunchに掲載した内容です。
（Qrunchのサービス終了に伴いこちらに引っ越ししました）
{{< /notice >}}

## 背景
flaskで簡単なWebアプリを作りたい！
環境構築はDocker（docker-compose）でやりたいなぁ。

やりながら忘れないようにメモメモ。
ソースは[こちら](https://github.com/igimia/flask-sample-app)

# 目標
docker-composeを利用してflask + mysqlの環境を構築する
* flaskからmysqlへ接続できること
* 起動したwebアプリへローカルから接続できること

## ファイル構成
全体的にこんな感じのフォルダ構成。
flaskはflask-appbuilderを使う予定。
```
flask_sample
 ├ docker-compose.yml
 ├ db
     ├ Dockerfile
     └ my.cnf
 └ flask
     ├ Dockerfile
     └ requirements.txt
```

##### docker-compose.yml
`tty: true`を設定するとコンテナが起動しっぱなしになるのでデバッグしたい時とかに便利。設定を削除する場合は`command`に`flask run`などのコマンドを設定すること。
アプリ開発が軌道にのるまではオンにしておくと便利そう。

```yaml
version: "3"
services:
  web:
    build: ./flask
    volumes:
      - ./flask:/app
    ports:
      - 5000:5000
    links:
      - db
    tty: true

  db:
    build: ./db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: flask
      MYSQL_USER: flask
      MYSQL_PASSWORD: flask
      TZ: 'Asia/Tokyo'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
    volumes:
      - ./db/data:/var/lib/mysql
      - ./db/my.cnf:/etc/mysql/conf.d/my.cnf
      - ./db/sql:/docker-entrypoint-initdb.d
      - ./db/log:/var/log/mysql
    ports:
      - 3306:3306
```

## db 設定ファイル
#####  Dockerfile
特にすることがなければDockerfileにしなくてもOK。
その時はdocker-compose.ymlの`build: ./db`のところを`image: mysql:5.7`に修正する。

```docker
FROM mysql:5.7
```

##### mysql設定ファイル（my.cnf）

```ini
[mysqld]
character-set-server=utf8mb4
general-log=1
general-log-file=/var/log/mysql/mysqld.log  #ログの出力先

[client]
default-character-set=utf8mb4
```

## flask 設定ファイル
#####  Dockerfile
```docker
FROM python:3.7.5

ARG app_dir=/app/

ADD requirements.txt $app_dir

WORKDIR $app_dir

RUN pip install --upgrade pip

# 利用するパッケージがかたまるまでコメントアウトしてても良さそう
RUN pip install -r requirements.txt
```
#####  requirements.txt
```
apispec==1.3.3
attrs==19.3.0
Babel==2.7.0
Click==7.0
colorama==0.4.1
defusedxml==0.6.0
Flask==1.1.1
Flask-AppBuilder==2.2.0
Flask-Babel==0.12.2
Flask-JWT-Extended==3.24.1
Flask-Login==0.4.1
Flask-OpenID==1.2.5
Flask-SQLAlchemy==2.4.1
Flask-WTF==0.14.2
importlib-metadata==1.0.0
itsdangerous==1.1.0
Jinja2==2.10.3
jsonschema==3.2.0
MarkupSafe==1.1.1
marshmallow==2.19.5
marshmallow-enum==1.5.1
marshmallow-sqlalchemy==0.19.0
more-itertools==8.0.0
mysqlclient==1.4.6
prison==0.1.2
PyJWT==1.7.1
pyrsistent==0.15.6
python-dateutil==2.8.1
python3-openid==3.1.0
pytz==2019.3
PyYAML==5.1.2
six==1.13.0
SQLAlchemy==1.3.11
SQLAlchemy-Utils==0.35.0
Werkzeug==0.16.0
WTForms==2.2.1
zipp==0.6.0
```
## コンテナを起動する
```bash
# Dockerfileを元にコンテナのイメージを構築する
$ docker-compose build
# コンテナ起動
$ docker-compose up
```

## 起動後の確認
#####  コンテナ起動確認
`docker ps`コマンドもしくは`docker-compose ps`コマンドを利用して今起動しているコンテナの一覧を確認してみる。
```
# StateがUpになってればOK
$ docker-compose ps
   Name                  Command               State                 Ports
----------------------------------------------------------------------------------------
flask_db_1    docker-entrypoint.sh mysql ...   Up      0.0.0.0:3306->3306/tcp, 33060/tcp
flask_web_1   python3                          Up      0.0.0.0:5000->5000/tcp
```

やったね！

#####  ネットワークの確認
flaskとmysqlが同一のネットワーク上にいるか確認する。
`docker network ls`あたりのコマンドから確認しても良いが、skanehiraさん（ゴリラさん）の[docui](https://github.com/skanehira/docui)を入れておくとめっちゃ便利！（神ありがたや！！）

こちらがdocuiの実行画面。
ちゃんと同じネットワーク上にflaskとmysqlがわかる！
![flask_1.png](/images/posts/20191201/flask_1.png)

詳細をみてみる...
ローカルからはlocalhost(127.0.0.1)でアクセスする。

![flask_2.png](/images/posts/20191201/flask_2.png)

## flaskの起動
ようやくflaskの起動へ。色々自分で一から作るのは大変なので、ユーザ管理とかテンプレートが充実している[flask-appbuilder](https://flask-appbuilder.readthedocs.io/en/latest/)を利用することに。

#####  flask-appbuilderからスケルトンの作成
```
# 起動したコンテナに入って作業（docker exec -it [コンテナID] /bin/sh）
/app # flask fab create-app
Your new app name: app
Your engine type, SQLAlchemy or MongoEngine (SQLAlchemy, MongoEngine) [SQLAlchemy]:
Downloaded the skeleton app, good coding!
```

#####  flask設定ファイル修正
作成したflaskのスケルトンアプリフォルダ配下にあるconfig.pyを修正する。
修正内容は以下の通り。
* 日本語を利用するためのエンコード設定（utf8）
* 接続先データベースの設定
* デフォルト言語設定

```python
#coding:utf8 #   <-- 日本語を利用するための設定. 一番上に追加する

...

# The SQLAlchemy connection string.
# SQLALCHEMY_DATABASE_URI = "sqlite:///" + os.path.join(basedir, "app.db")

# 'mysql://[User]:[Password]@[Host]/[Database]'
# パスワードはなるべく直書きしないように（ゆくゆくは環境変数へ...）
SQLALCHEMY_DATABASE_URI = 'mysql://root:root@db/app'

...

# 日本語の設定を追加し、デフォルトの言語を日本語にする
# ---------------------------------------------------
# Babel config for translations
# ---------------------------------------------------
# Setup default language
BABEL_DEFAULT_LOCALE = "ja"
# Your application default translation path
BABEL_DEFAULT_FOLDER = "translations"
# The allowed translation for you app
LANGUAGES = {
    "en": {"flag": "gb", "name": "English"},
    "pt": {"flag": "pt", "name": "Portuguese"},
    "pt_BR": {"flag": "br", "name": "Pt Brazil"},
    "es": {"flag": "es", "name": "Spanish"},
    "de": {"flag": "de", "name": "German"},
    "zh": {"flag": "cn", "name": "Chinese"},
    "ru": {"flag": "ru", "name": "Russian"},
    "pl": {"flag": "pl", "name": "Polish"},
    "ja": {"flag": "jp", "name": u"日本語"},
}
```

#####  adminユーザの登録
適当にadminユーザを作成する。
```bash
flask fab create-admin

Username [admin]:
User first name [admin]:
User last name [user]:
Email [admin@fab.org]:
Password:
Password:

...
2019-12-01 10:04:56,305:INFO:flask_appbuilder.base:Registering class MyView on menu Method1
2019-12-01 10:04:56,306:INFO:flask_appbuilder.baseviews:Registering route /myview/method1/ ('GET',)
Recognized Database Authentications.
2019-12-01 10:04:56,450:INFO:flask_appbuilder.security.sqla.manager:Added user admin
Admin User admin created.     <--  adminユーザ作成成功！
```

#####  Viewの修正
そのまま起動するとInternalServerErrorが出てしまう。
せっかくなので適当なviewを設定する。
view.pyをこんな感じに書き換える。

```python
from flask import render_template
from flask_appbuilder.models.sqla.interface import SQLAInterface
from flask_appbuilder import BaseView, expose, has_access, ModelView, ModelRestApi

from . import appbuilder, db


class MyView(BaseView):

    default_view = 'method1'

    @expose('/method1/')
    @has_access
    def method1(self):
        # do something with param1
        # and return to previous page or index

        return 'Hello'


appbuilder.add_view(MyView, "Method1", category='My View')

db.create_all()
```

#####  flask起動
起動コマンド実行後、`* Running on http://0.0.0.0:5000/ ` が出てくればOK。
```python
flask run --host 0.0.0.0 --port 5000

...
2019-12-01 10:06:19,124:INFO:flask_appbuilder.api:Registering route /api/v1/menu/ ['GET']
2019-12-01 10:06:19,218:INFO:flask_appbuilder.base:Registering class MyView on menu Method1
2019-12-01 10:06:19,218:INFO:flask_appbuilder.baseviews:Registering route /myview/method1/ ('GET',)
2019-12-01 10:06:19,282:INFO:werkzeug: * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

#####  ローカルからアクセスする
ローカルのブラウザから`http://localhost:5000/`へアクセスする。
![flask_3.png](/images/posts/20191201/flask_3.png)


でけた。
`flask fab create-admin`で作成したadminユーザでログインすることもできる。

## 詰まったこと
最初、flaskのイメージにalpineを利用していたのだが、mysqlclientがpip installできなかった。
alpineだとmysqlclientをインストールするためのパッケージが足りないみたい...
いずれalpineにして必要なパッケージのみaddしていい感じにしたい。

また、最初何も考えずに`flask run`コマンドを実行していたせいでローカルから接続できない事態に陥った。hostを指定せずに起動すると`http://127.0.0.1:5000`で起動するのだが、コンテナに割り当てられたIPアドレスは別のIPなのでローカルから接続できなかった。

うっかりしてたww
コンテナから起動するときは`--host`オプションつけて、`コンテナに割り当てられたIPアドレス`か`0.0.0.0`で起動するように。

## ソース
ソースは[こちら](https://github.com/igimia/flask-sample-app)。
