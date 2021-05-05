---
title: 【Flask】flask-appbuilder を使ったwebアプリにfaviconを表示させたい
date: 2019-12-25 00:11:02 +09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: false
tocPosition: inner
tags:
- flask
- flask-appbuilder
series:
-
categories:
- web
---

{{< notice warning "注意事項" >}}
この記事は`2019/12/12`にQrunchに掲載した内容です。
（Qrunchのサービス終了に伴いこちらに引っ越ししました）
{{< /notice >}}

まとめてアウトプットすることも大事ですが、少しずつ小さな単位でアウトプットし、後からまとめることの大切さも実感する今日この頃です。

必要最低限の記述ですみません。

## やりたいこと
flask（flask-appbuilderプラグイン利用）でwebアプリ起動時faviconを表示するようにしたい。
イメージはこんな感じ（ブラウザはsafariです）。
![image_1](/images/posts/20191225/1225_1.png)

## やりかた
GitHubに良い情報が。 [参考リンク](https://github.com/dpgaspar/Flask-AppBuilder/issues/1036)
`I nice way to do it is to override base.html template and add it to the block head_css`

base.htmlをオーバーライドしてhead_cssをごにょゴニョすれば良さそう。

## やってみる
関連するフォルダ構成はこんな感じ。
（config.pyとかは省略してます）
```
 .
├─ static
│      └─ favicon.png
├─ templates
│      ├─ custom_base.html
│      └─ sample.html
├─ __init__.py
└─ views.py

```

#### custom_base.html
```jinja2
{% extends 'appbuilder/baselayout.html' %}

{% block head_css %}
    {{ super() }}
    <link rel="shortcut icon" href="{{ url_for('static', filename='favicon.png') }}">
{% endblock %}
```

#### sample.html
```jinja2
{% extends "./custom_base.html" %}

{% block content %}
  <h2>favicon表示させたい</h2>
{% endblock %}
```

#### views.py
```python
from flask import render_template
from flask_appbuilder import BaseView, expose, has_access

from . import appbuilder, db


class MyView(BaseView):
    default_view = 'method1'

    @expose('/method1/')
    @has_access
    def method1(self):
        # do something with param1
        # and return to previous page or index

        return (
            render_template(
                "sample.html", appbuilder=appbuilder
            ),
            200,
        )

appbuilder.add_view(MyView, "Method1", category='My View')


db.create_all()
```

これでFlaskを起動させ、`{baseurl}/myview/method1`にアクセスすると、staticフォルダ配下に設置したfaviconファイルの内容が表示される。

...しかし、この方法だと他のurlにアクセスするとfaviconが表示されません。ぐぬっ。

AppBuilderインスタンス生成時にベーステンプレートの設定を加えられるようなので`__init__.py`に設定を追加します。

#### `__init__.py`
```python
from flask import Flask
from flask_appbuilder import AppBuilder, SQLA

app = Flask(__name__)
db = SQLA(app)
appbuilder = AppBuilder(app, db.session, base_template='custom_base.html')
```

再びflaskを起動しなおすと全てのrouteにfaviconが表示されます( ´∀｀)ﾜｰｲ
