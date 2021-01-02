---
title: "【AWS, SES】python(boto3)を利用してメールを送信する"
date: 2020-11-22T21:00:30+09:00
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
- aws
- ses
---

## 背景
処理の結果に応じて自動的にメールを送信する機能を作りたかった。
最終的にファイルを添付したメールを送れるようにしたい。
試してみる。

## 試してみる

### [step1] aws cliを利用したメールの送信方法

AWSCLIを利用して送るとどうなるのか試してみた。
リージョンはオレゴンで利用していたのでオレゴンのIDを指定。

```bash
$ aws --region us-west-2 --profile=<プロファイル名> ses send-email --to send_to@example.com --from send_from@example.com --subject sample --text "`echo -e hogehoge`"

# 送信完了するとMessageIdが表示される
# {
#     "MessageId": "01010175efc9f343-a80ee339-a377-4228-bbfb-83c3d2e4ad40-000000"
# }
```

使っていたgmailを宛先にして実施し、送信後メールボックスを確認すると無事にメールが届いていた！
次はファイルを添付して送信してみたいところだが、aws cliでファイルを添付する方法がすぐに検索できなかった。。。

そのため、次はpython(boto3)を利用してメールを送れるように試してみる。

### [step2] python(boto3)を利用してメールを送信する

ファイルの添付などせず、そのまま送信する。
以下のサイトが参考になった。

* [boto3 Docs(SES)](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_raw_email)
* [PythonでAmazon SESからメールを送ってみた（Qiita）](https://qiita.com/takeh/items/2dee4f93bee05ec64e18)

````python
import boto3

client = boto3.client(
    'ses',
    aws_access_key_id='YourAccessKey',     # アクセスキー
    aws_secret_access_key='YourSecretKey', # シークレットキー
    region_name = 'us-west-2'              # リージョン（今回はオレゴン）
)

# メール送信
client.send_email(
    Source = 'from@example.com',              # 送信元メールアドレス
    Destination = {
        'ToAddresses': ['to@example.com']     # 宛先メールアドレス
    },
    Message = {
        'Subject': {
            'Data': 'title',
            'Charset': 'UTF-8'
        },
        'Body': {
            'Text': {
                'Data': 'hello',
                'Charset': 'UTF-8'
            }
        }
    }
)
````

### [step3] ファイルを添付してメールを送信する（python, boto3）

公式をみてみると、ファイルを添付してメールを送信する場合、`send_email`ではなく`send_raw_email`を利用するらしい。
`send_raw_email`はsend_emailと引数が異なるため少し詰まった。
公式にある例を参考にしてなんとかメールを送信することはできた。


````python
import boto3


client = boto3.client(
    'ses',
    aws_access_key_id='YourAccessKey',
    aws_secret_access_key='YourSecretKey',
    region_name = 'us-west-2'
)

# メール送信
client.send_raw_email(
    Source = 'from@example.com',   # 送信元メールアドレス
    Destinations=[                 # 送信先メールアドレス
        "to_1@example.com",
        "to_2@example.com"
    ],
    RawMessage={
        'Data': 'Subject: Test email (contains an attachment)\nMIME-Version: 1.0\nContent-type: Multipart/Mixed; boundary="NextPart"\n\n--NextPart\nContent-Type: text/plain\n\nThis is the message body.\n\n--NextPart\nContent-Type: text/plain;\nContent-Disposition: attachment; filename="attachment.txt"\n\nThis is the text in the attachment.\n\n--NextPart--',
    },
)
````

受信されたメールをみてみると、
````
This is the text in the attachment.
````
という内容ファイル（`attachment.tx`）が添付されたメールが届いていた。

また、`Destinations`に複数設定すると複数のメールアドレスがToに設定されるものかと思ったが、そうではなく、それぞれ別々のメールとして送信されているようだった。

一応メールを送信することはできたものの、あまりにも不格好なのでもう少しやってみる。
To, Cc, Bccや添付ファイルの中身、内容などを全てRawMessageに含めるのではなく、それぞれ分けて設定できるようにしたい。
(最悪最後にRawMessageとして結合するしかないのかもだが。。。)

このあたりを参考に。。
* https://stackoverrun.com/ja/q/11824951
* https://blog.serverworks.co.jp/tech/2018/12/16/ses_lambda_attachment/

作ったサンプルソースがこちら。
最初よりかなりよくなった。

````python
import boto3

from email.mime.application import MIMEApplication
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText


class Email(object):
    def __init__(self, frm, to, subject):
        self.frm = frm
        self.to = to
        self.subject = subject
        self.text = None
        self.attachment = None

    def send(self):
        client = boto3.client(
            "ses",
            aws_access_key_id="YourAccessKey",
            aws_secret_access_key="YourSecretKey",
            region_name="us-west-2",
        )

        # メール送信
        msg = MIMEMultipart()
        msg["Subject"] = self.subject
        msg["From"] = self.frm
        msg["To"] = self.to

        filepath = "/tmp/attachment.txt"    # 添付ファイルの絶対パス
        part = MIMEApplication(open(filepath, "rb").read())

        part.add_header("Content-Disposition", "attachment", filename="添付ファイル.txt")
        msg.attach(part)

        part = MIMEText("sample mail")      # メッセージBody
        msg.attach(part)

        client.send_raw_email(
            RawMessage={
                "Data": msg.as_string(),
            },
            Destinations=[],
        )


if __name__ == "__main__":
    email = Email(
        frm="from@example.com",
        to="to_1@example.com",
        subject="件名"
    )
    # you could use StringIO.StringIO() to get the file value
    email.send()
````

受信したメールをみてみると`添付ファイル.txt`という名前のファイルが添付されていて、中身は指定した`/tmp/attachment.txt`と一致した。
メールと MIME オブジェクトを作るライブラリを利用してデータを設定し、最後に`as_string()｀でstringにして`Data`に設定するとメールの作成が結構楽になりそう。

公式（https://docs.python.org/ja/3/library/email.mime.html）も参考にすると良さそうだ。





















