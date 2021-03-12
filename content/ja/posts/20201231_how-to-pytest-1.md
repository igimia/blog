---
title: "pytestを使ってみる①【準備・実行編】(python3.9.0)"
date: 2020-12-31T18:10:30+09:00
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
- Python
- test
- pytest
---

## 背景

最近仕事でPytestを使ったUnitTestをかく機会があったので使い方を忘れないようにメモ。参考にもあるが、[こちらのサイト](https://www.magata.net/memo/index.php?pytest%C6%FE%CC%E7)がすごくわかりやすかった。
感謝。。。！

ちなみに実行環境はmacで利用しているpythonのバージョンは`python3.9.0`です。

## pytestとは

Python製のテストフレームワークの一つ。似たようなものにunittestがある。unittestは標準で使えるライブラリだが、pytestはライブラリのインストールが必要となる。
カバレッジを算出したり、結果をhtmlファイルに出力することができて便利。

## 基本的な使い方

### インストール方法

pipコマンドからインストールが可能
```bash
$ pip install pytest
```

### ディレクトリ構成

```
.
├── src
│   ├── main.py
│   └── util
│       ├── __init__.py
│       └── util.py
└── tests
    ├── __init__.py
    ├── conftest.py
    └── test_main.py
```

`src`と`tests`は分けた方が管理しやすいので分割。
CIなどでtestを実行した時にPATH通るよう、`conftest.py`は以下のように書いた。

```python
import sys
import os

sys.path.append(os.path.abspath(os.path.dirname(os.path.abspath(__file__)) + "/../src/"))
```

実行したいテストのファイル名は`test_xxx.py`の名前をつけると良い。自動でテスト対象としてみなしてくれる。

### 基本的な書き方と実行方法

#### 基本的な書き方

```python
def test_main_01():
    # 試験実施
    a = 1
    b = 1

    # 結果の検証
    assert a == b
```

#### 実行方法

##### テストケースをまとめて実行する

root（今回の例では`.`）で実行。
`tests/`配下のテストが実行される。

````bash
$ pytest .
````

##### 試験ケースを指定して実行する

例えば`test_main.py`ファイル内の`test_main_01`を実行したい時は以下のように指定する。

```bash
$ pytest tests/test_main.py::test_main_2

# "-v"オプションをつけると指定の仕方を確認することができて便利。
$ pytest . -v
======================================================================================== test session starts ========================================================================================
cachedir: .pytest_cache
rootdir: /Users/hogehoge/
collected 2 items

tests/test_main.py::test_main_01 PASSED  [ 50%]
tests/test_main.py::test_main_02 PASSED  [100%]
```

### そのほか便利な機能

#### カバレッジを出したい時

[pytest-cov](https://pypi.org/project/pytest-cov/)を利用する。

```bash
$ pip install pytest-cov
```

インストール後、pytestコマンドにオプションをつけることでカバレッジを表示することが可能になる。
今回の例では`src/`配下にソースを格納しているためsrcを設定した。

```bash
# "-v"オプションをつけると一つ一つの試験結果が表示され結果が見やすくなる
$ pytest -v --cov=src
======================================================================================== test session starts ========================================================================================
cachedir: .pytest_cache
rootdir: /Users/hogehoge/
plugins: cov-2.10.1
collected 2 items

tests/test_main.py::test_main_1 PASSED  [ 50%]
tests/test_main.py::test_main_2 PASSED  [100%]

---------- coverage: platform darwin, python 3.9.0-final-0 -----------
Name                   Stmts   Miss  Cover
------------------------------------------
src/main.py                9      1    89%
src/util/__init__.py       0      0   100%
src/util/util.py           2      0   100%
------------------------------------------
TOTAL                     11      1    91%

========================================================================================= 2 passed in 0.08s =========================================================================================
```

#### カバレッジをHTMLで表示したい

##### テストできているところ / いないところを確認する

```
$  pytest --cov=src --cov-report=html
```

コマンドを実行したフォルダ配下に`htmlcov`フォルダが作成されている。フォルダ内の`index.html`を開くとファイルごとのカバレッジが表示されている。それぞれリンクになっていて、クリックするとテストができているところとそうでないところを確認することができる。

![pytest_1](/images/posts/20201231/pytest_1.png)

src/main.pyをクリックするとどこがテストできていないのかすぐわかる。

![pytest_2](/images/posts/20201231/pytest_2.png)

##### テスト結果のサマリをHTMLで表示する

[pytest-html](https://pypi.org/project/pytest-html/)をインストールする。

```bash
# インストール
$ pip install pytest-html

# 実行方法
# --html=<出力したいファイル名>
$ pytest . --html=cov.html
```

以下のように試験結果が表示される。

![pytest_3](/images/posts/20201231/pytest_3.png)

#### テスト対象外のファイルを設定したい

pytest対象外としたいファイルがある時の設定方法。
rootディレクトリに`.coveragerc`を作成する（今回の例ではsrc, testsフォルダと同じ階層に作成）

```
.
├── src
├── tests
└── .coveragerc
```

`.coveragerc`に対象外としたいフォルダやファイル名を記述する。

```ini
[run]
omit =
   # __init__.py ファイルはテスト対象外とする
   */__init__.py
```

試験を実行すると設定したフォルダやファイルがテスト対象外となっている。

```bash
$ pytest --cov=src
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.9.0, pytest-6.2.1, py-1.10.0, pluggy-0.13.1
rootdir: /Users/hogehoge/
plugins: html-3.1.1, metadata-1.11.0, cov-2.10.1
collected 2 items

tests/test_main.py ..   [100%]

---------- coverage: platform darwin, python 3.9.0-final-0 -----------
Name               Stmts   Miss  Cover
--------------------------------------
src/main.py            9      1    89%
src/util/util.py       2      0   100%
--------------------------------------
TOTAL                 11      1    91%


========================================================================================= 2 passed in 0.09s =========================================================================================
```

#### テストで失敗した時にメッセージを出力したい

assert後にカンマ(`,`)をつけ、その後メッセージを書く。

```python
def test_main_failure_1():
    # 試験データ
    a = 1
    b = 5

    # テストに失敗した時メッセージが表示される
    assert a == b, "a not equal b"
```

```python
pytest tests/test_main_failure.py
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.9.0, pytest-6.2.1, py-1.10.0, pluggy-0.13.1
rootdir: /Users/hogehoge/
plugins: html-3.1.1, metadata-1.11.0, cov-2.10.1
collected 1 item

tests/test_main_failure.py F    [100%]

============================================================================================= FAILURES ==============================================================================================
________________________________________________________________________________________ test_main_failure_1 ________________________________________________________________________________________

    def test_main_failure_1():
        # 試験データ
        a = 1
        b = 5

        # 想定値
>       assert a == b, "a not equal b"
E       AssertionError: a not equal b
E       assert 1 == 5

tests/test_main_failure.py:7: AssertionError
====================================================================================== short test summary info ======================================================================================
FAILED tests/test_main_failure.py::test_main_failure_1 - AssertionError: a not equal b
========================================================================================= 1 failed in 0.05s =========================================================================================
```

#### どんなデータが入っていてもassertの結果をTrueにしたい時

pytestではなく、標準ライブラリとして用意されている`unittest`の`ANY`を利用することで可能。    
滅多に使うケースはないが、知っているとすごく便利。例えばmockにした関数に渡された引数のassertをしたいがその一部が実行時点のタイムスタンプやランダム値など予測不能な値である場合に対象項目の予測値に`ANY`を利用する。  

どんなデータでも通ってしまうので利用する時は注意が必要。。

```python
from unittest.mock import ANY

def test_main_any_1():
    # 試験データ
    a = None

    # こんなassertも問題なく通る
    assert a == ANY
```

## 参考  
大変参考にさせていただきました。ありがとうございます。

* [PyPi(pytest)](https://pypi.org/project/pytest/)
* [pytest入門 - 闘うITエンジニアの覚え書き](https://www.magata.net/memo/index.php?pytest%C6%FE%CC%E7)
* [知っておくと便利！Pytestコマンドライン小ネタ集](https://dev.classmethod.jp/articles/pytest-tips-cmd-options/)
* [.coveragercの書き方について](https://stackoverflow.com/questions/1628996/is-it-possible-exclude-test-directories-from-coverage-py-reports)
