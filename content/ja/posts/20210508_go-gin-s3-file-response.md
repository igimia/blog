---
title: "【Go】GinでAWS S3からダウンロードしたデータを圧縮して返却する"
date: 2021-05-06T18:46:36+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
- go
- gin
series:
-
categories:
-
---

## 概要

APIにファイルダウンロードを機能追加する際、Go言語のginを使うとどんな感じにできるのか気になったのでやってみた。その時のメモ。
ファイルは事前にAWS S3にアップロードされている想定。

### バージョン

* Go: `v1.13.4`
* gin: `v1.7.1`
* aws-sdk-go: `v1.38.36`

#### S3からファイルをダウンロードする

初めにAWS S3からファイルをダウンロードして内容を標準出力することをやってみる。

ファイルのダウンロードはローカルにファイルを落としてくる方式とStreamとして扱う方法とあるみたい。
ローカルにダウンロードする必要はないので[GetObject](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/#S3.GetObject)を利用する。

```go
package main

import (
    "bytes"
    "fmt"

    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/s3"
)

func main() {
    sess := session.Must(session.NewSession(&aws.Config{
        Region: aws.String("ap-northeast-1"),
    }))
    s3handler := s3.New(sess)

    obj, _ := s3handler.GetObject(&s3.GetObjectInput{
        Bucket: aws.String("2021-test-bucket"),     // ダウンロードしたいS3オブジェクトがアップロードされているS3バケット
        Key:    aws.String("sample.txt"),           // キー (ファイル)名
    })

    body := obj.Body
    defer obj.Body.Close()

    buf := new(bytes.Buffer)
    buf.ReadFrom(body)
    fmt.Print(buf.String())
}
```

実行するとダウンロードしたファイルの内容が表示された。

```bash
$ go run main.go
# this is sample text
```

#### ginを使ってレスポンスに圧縮したファイルを返す

stackoverflowにファイルの圧縮について質問しているQAがあった（[stackoverflow](https://stackoverflow.com/questions/57429256/how-to-generate-zip-7z-archive-on-the-fly-in-a-http-server-using-gin)）
これと、[【Golang】S3上のデータをメモリ上に乗せずにストリーミングしてレスポンスへ流す【AWS】](https://tkzo.jp/blog/golang-streaming-download-then-send-http-response/)が非常に参考になった。

レスポンスに設定したzipへ追加したfileのWriterにS3からダウンロードしたobjectのReaderを流している。
S3のデータをローカルにダウンロードしたり、zipファイルを作成することなくレスポンスを返すことができる。すごい。

```go
package main

import (
    "archive/zip"
    "io"
    "log"

    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/s3"
    "github.com/gin-gonic/gin"
)

func main() {
    engine := gin.Default()
    engine.GET("/", func(c *gin.Context) {

        sess := session.Must(session.NewSession(&aws.Config{
            Region: aws.String("ap-northeast-1"),
        }))
        s3handler := s3.New(sess)

        obj, err := s3handler.GetObject(&s3.GetObjectInput{
            Bucket: aws.String("2021-test-bucket"),
            Key:    aws.String("sample.txt"),
        })

        if err != nil {
            log.Print(err)
        }

        body := obj.Body
        defer obj.Body.Close()

        c.Stream(func(w io.Writer) bool {
            ar := zip.NewWriter(w)
            defer ar.Close()

            c.Writer.Header().Set("Content-Disposition", "attachment; filename=download.zip")

            f, _ := ar.Create("sample.txt")
            io.Copy(f, body)

            return false
        })
    })

    engine.Run(":3000")
}

```
