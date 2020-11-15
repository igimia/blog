---
title: "MySQLでランダムデータを生成したい"
date: 2010-09-26T20:40:51+09:00
description: tool
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
- mysql
series:
-
image:
---

{{< notice warning "注意事項" >}}
この記事は`2019/09/26`にQrunchに掲載した内容です。
（Qrunchのサービス終了に伴いこちらに引っ越ししました）
{{< /notice >}}

# 背景
DB（MySQL）の性能試験をしているときに、ダミーデータを素早く積み込みたい想いに駆られました。
結果、私的にはtmpテーブルとMySQLのランダム関数を利用したデータの投入が一番早いやんww となりまして。
その話もまたいずれ...
知っておくと便利なSQLなので、備忘も兼ねてまとめました。

ちなみにMySQLのクライアントツールは[sequel](https://sequelpro.com/)がおすすめです。
本家のクライアントアプリ[MySQL Workbench](https://www.mysql.com/jp/products/workbench/)もめっちゃいいですが、サクッと使いたい人はsequelで十分です。　　

# ランダムデータ生成SQL
ローカルでさっくり試してみたいけど環境構築が...という方のため、後述でMySQLをコンテナで立てる方法を記載してますよかったら参考にしてください。

ちなみにMySQLで複数の文字列を結合して出力したい場合は`concat`を使うとよかです。
```sql
mysql> select concat('SAM', 'PLE', '00001');
+-------------------------------+
| concat('SAM', 'PLE', '00001') |
+-------------------------------+
| SAMPLE00001                   |
+-------------------------------+
```
`select xxx`という書き方はecho的なことがしたいなぁという時に使えるので、覚えておくと便利そうです。

## ランダムデータ生成（文字列編）
### 英小文字 + 数字
32桁までのランダムデータを生成する場合は`uuid()`関数を利用するのが一番楽です。
後述するチェックサムを利用する方法だと積みこむデータの量や長さよっては重複が結構発生して面倒です（私のデータ生成方法が悪いのか...）。
```sql
mysql> select concat(LEFT(REPLACE(uuid(),'-',''), 8)) as RandomData;
+------------+
| RandomData |
+------------+
| db8a8aff   |
+------------+
```

チェックサムを利用してデータを生成する場合はこちら
```sql
mysql> select concat(LEFT(MD5(RAND()*100+1),8)) as RandomData;
+------------+
| RandomData |
+------------+
| 678b5019   |
+------------+
```
各関数の詳細はMySQLの公式リファレンスを参照してください。
* [RAND](https://dev.mysql.com/doc/refman/5.6/ja/mathematical-functions.html#function_rand)
* [MD5](https://dev.mysql.com/doc/refman/5.6/ja/encryption-functions.html#function_md5)

ざっくり説明すると、1.xxx ~ 100.xxx の少数を生成し、MD5関数を使ったチェックサムを生成、生成したチェックサムの左から引数に指定した桁数（今回は6桁）分切り取って返却しています。

この方法だと英語の大文字や記号入りません。
固定長でいいから大文字や記号を入れたい時はこんな感じでデータを生成すれば良いです。
```sql
mysql> select concat('SAMPLE_', LEFT(MD5(RAND()*100+1),6)) as RandomData;
+---------------+
| RandomData    |
+---------------+
| SAMPLE_cc0c31 |
+---------------+
```

### 特定の文字種を利用したランダムデータ

`RAND() * n` のn には定義した文字種の桁数を設定してください。
```sql
mysql> select CONCAT(substring('あいうえおabcde12345', CEIL(RAND() * 15),1)
    ->  , substring('あいうえおabcde12345', CEIL(RAND() * 15),1)
    ->  , substring('あいうえおabcde12345', CEIL(RAND() * 15),1)
    ->  ) as RandomData;
+------------+
| RandomData |
+------------+
| 53え       |
+------------+
```

mysqlのユーザ定義変数を使うと少しだけ実行が楽になります。
```sql
mysql> SET @var1 = 'あいうえおabcde12345';
Query OK, 0 rows affected (0.00 sec)

mysql> select @var1 ;
+---------------------------+
| @var1                     |
+---------------------------+
| あいうえおabcde12345      |
+---------------------------+

mysql> select CONCAT(substring(@var1, CEIL(RAND() * 15),1)
    ->  , substring(@var1, CEIL(RAND() * 15),1)
    ->  , substring(@var1, CEIL(RAND() * 15),1)
    ->  ) as RandomData;
+------------+
| RandomData |
+------------+
| あああ     |
+------------+
```

## ランダムデータ生成（数字編）
1~100 まで
```sql
mysql> select concat(ceil(rand() * 100 + 0)) as RandomData;
+------------+
| RandomData |
+------------+
| 51         |
+------------+
```
1001 ~ 1099 まで
```sql
mysql> SELECT CONCAT(CEIL(RAND() * 99 + 1000)) as RandomData;
+------------+
| RandomData |
+------------+
| 1085       |
+------------+
```

## ランダムデータ生成（日付編）
```sql
mysql> select FROM_UNIXTIME(UNIX_TIMESTAMP('2019-03-01 00:00:00')  + FLOOR(0 + (RAND() * 86400))) as RandomData;
+---------------------+
| RandomData          |
+---------------------+
| 2019-03-01 17:32:54 |
+---------------------+
 ```
**86400**は1日間の秒数`24*60*60 = 86400`です。


# サクッとお試し環境構築手順
一応MySQLの環境構築設定もメモします。
コンテナ（docker-compose）で構築します。

## docker-composeファイルの作成

```yml
version: '3'

services:
  # MySQL
  db:
    image: mysql:5.7
    container_name: mysql_host
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test_database
      MYSQL_USER: docker
      MYSQL_PASSWORD: docker
      TZ: 'Asia/Tokyo'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
    volumes:
    - ./db/data:/var/lib/mysql
    - ./db/my.cnf:/etc/mysql/conf.d/my.cnf
    - ./db/sql:/docker-entrypoint-initdb.d
    ports:
    - 3306:3306
```

## コンテナ起動
作成したdocker-compose.yamlファイルがあるパス上で`docker-compose up`コマンドを実行する。
mysqlが立ち上がったらローカルから`mysql -uroot -h<hostnameコマンドの実行結果名> -p`で実行し、パスワードは`root`を入れてあげれば入れると思います。
mysql-clientをダウンロードしていない場合はダウンロードしてから実行するか、`docker exec -it` でmysqlのコンテナにアタッチしてください。
