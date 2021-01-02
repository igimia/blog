---
title: "pytestを使ってみる②【実行前、実行後処理編】"
date: 2021-01-01T19:10:30+09:00
draft: false
tags:
- Python
- test
- pytest
---

## 概要

今回はテスト実行前、実行後（setup, teardown）の処理についてまとめる。

## 使い方

`@pytest.fixture()`というデコレータを設定すればよい。
`yield`以降はテストが実行された後に実行される（teardown）。

使いたいテスト関数の引数にfixtureの関数名を指定すると使うことができる。

```python
import pytest


@pytest.fixture
def pre_function():
    print("\n実行前")
    yield "以降は試験実行後に処理される"
    print("\n実行後")


def test_setup_teardown_1(pre_function):
    print("test_1")
    # 試験データ
    a = 1
    b = 1
    assert a == b, "aaa"


def test_setup_teardown_2():
    print("test_2")
    # 試験データ
    a = 2
    b = 2
    assert a == b
```

実行結果は以下のようになる。  

```bash
 pytest tests/test_setup_teardown.py --capture=no                                                                                                            1 ✘  23:14:41  
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.9.0, pytest-6.2.1, py-1.10.0, pluggy-0.13.1
rootdir: /Users/hogehoge/
plugins: html-3.1.1, metadata-1.11.0, cov-2.10.1
collected 2 items                                                                                                                                                                                   

tests/test_setup_teardown.py 
実行前
test_1
.
実行後
test_2
.

========================================================================================= 2 passed in 0.01s =========================================================================================
```

`@pytest.fixture`には色々なオプションを指定することが可能（詳細は[公式](https://docs.pytest.org/en/stable/fixture.html)参照）。    
中でも`scope`と`autouse`はよく使うと思うので一応まとめてみる。    

### scope

`scope="スコープ名"`の形式で設定する（デフォルトはfunction）。fixtureのスコープ（適用範囲）を指定できる。
`function`, `class`, `module`, `package`, `session`のうちから選ぶ。

| スコープ名 | 説明                                                            | 例(以下`試してみた`で利用した名前) |
|------------|-----------------------------------------------------------------|-----------------|
| function   | テスト関数の実行前後で setup/teardownが実行される。デフォルト。 | `test_function_1` |
| class      | テストクラスの実行前後で    〃 。                                  | `TestClass`       |
| module     | テストモジュールの実行前後で    〃 。                              | `test_scope_1.py` |
| package    | テストパッケージの前後で    〃 。                                  | `scope/package1`     |
| session    | 全体通して一回setup/teardownが実行される。                      | -               |

### autouse

`autouse=True or False`の形式で設定する（デフォルトはFalse）。Trueを設定すると対象のscopeのテストケースを実行するとき自動的にsetup/teardownしてくれる。  
試験パターンが多くなってくると個別にfixtureの設定をするのがかなり面倒になるため、共通的なものはautouseを利用すると便利。

#### 試してみた

確認かねて設定したらどんなもんか試してみた。  
moduleとpackageの動作確認を行うため以下のような構成にした。 

```
└── tests
    └── scope
         ├── package1 
         |  ├── test_scope_1.py   # function ~ session まで全種類のfixture設定
         |  └── test_scope_2.py   # fixtureの設定なし（module スコープの動作確認用）
         └── package2             
           └── test_package_2.py  # fixtureの設定なし（package スコープの動作確認用）
```

`package1/test_scope_1.py`  
全種類のfixtureを設定し、自動的に対象のスコープの前後処理が行われるように全てautouseはTrueに設定した。

```python:test_scope_1.py
import pytest


@pytest.fixture(scope="function", autouse=True)
def pre_function():
    scope = "Function"
    print(f"\n=== SETUP {scope} ===")
    yield
    print(f"\n=== TEARDOWN {scope} ===\n")


@pytest.fixture(scope="class", autouse=True)
def pre_class():
    scope = "Class"
    print(f"\n*** SETUP {scope} ***")
    yield
    print(f"\n*** TEARDOWN {scope} ***\n")


@pytest.fixture(scope="module", autouse=True)
def pre_module():
    scope = "Module"
    print(f"\n--- SETUP {scope} ---")
    yield
    print(f"\n--- TEARDOWN {scope} ---\n")


@pytest.fixture(scope="package", autouse=True)
def pre_package():
    scope = "Package"
    print(f"\n<<< SETUP {scope} >>>")
    yield
    print(f"\n<<< TEARDOWN {scope} >>>\n")


@pytest.fixture(scope="session", autouse=True)
def pre_session():
    scope = "Session"
    print(f"\n@@@ SETUP {scope} @@@")
    yield
    print(f"\n@@@ TEARDOWN {scope} @@@\n")


def test_function_1():
    print("test_function_1")
    # 試験データ
    a = 1
    assert a == 1

class TestClass:
    def test_class_1(self):
        print("test_class_1")
        # 試験データ
        a = 1
        assert a == 1

    def test_class_2(self):
        print("test_class_2")
        # 試験データ
        a = 1
        assert a == 1
```

package1/test_scope_2.py

```python:test_scope_2.py
class TestClass:
    def test_class_no_fixture(self):
        print("\ntest_class_no_fixture")
        # 試験データ
        a = 1
        assert a == 1


def test_function_no_fixture():
    print("\ntest_function_no_fixture")
    # 試験データ
    a = 1
    assert a == 1
```

実行結果は以下の通り。  
大体想像通りだったが、メンバ関数ではない普通の関数テストも前後にclass scopeの処理が行われるのが想定外だった。  
(python本体のソース見てないが、関数は規定クラスのメンバ関数という位置付けなのかもなと予想。そのうち本体のソース見てみたい。。)

```bash
$  pytest tests/scope --capture=no
======================================================================================== test session starts ========================================================================================
platform darwin -- Python 3.9.0, pytest-6.2.1, py-1.10.0, pluggy-0.13.1
rootdir: /Users/hogehoge
plugins: html-3.1.1, metadata-1.11.0, cov-2.10.1
collected 6 items                                                                                                                                                                                   

tests/scope/package1/test_scope_1.py 
@@@ SETUP Session @@@

<<< SETUP Package >>>

--- SETUP Module ---

*** SETUP Class ***

=== SETUP Function ===
test_function_1
.
=== TEARDOWN Function ===


*** TEARDOWN Class ***


*** SETUP Class ***

=== SETUP Function ===
test_class_1
.
=== TEARDOWN Function ===


=== SETUP Function ===
test_class_2
.
=== TEARDOWN Function ===


*** TEARDOWN Class ***


--- TEARDOWN Module ---


tests/scope/package1/test_scope_2.py 
test_class_no_fixture
.
test_function_no_fixture
.
tests/scope/package2/test_package_2.py 
test_package_2
.
<<< TEARDOWN Package >>>


@@@ TEARDOWN Session @@@

========================================================================================= 6 passed in 0.06s =========================================================================================

```



## 感想

testsフォルダ直下には一つ前の記事のpytest用スクリプトがあったのでscope動作確認時のフォルダ構成が気持ち悪い感じになった。記事ごとにフォルダ分ければよかった。  
公式をみてみると他にも便利なオプションがたくさんありそうなので、新しいオプション利用する度追記していきたい。


## 参考
* [How do I correctly setup and teardown for my pytest class with tests?](https://stackoverflow.com/questions/26405380/how-do-i-correctly-setup-and-teardown-for-my-pytest-class-with-tests)
* [docs.pytest.org](https://docs.pytest.org/en/stable/fixture.html)









