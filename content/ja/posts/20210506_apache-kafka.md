---
title: "Apache kafkaを使ってみる"
date: 2021-05-06T18:46:36+09:00
description:
draft: true
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
- kafka
series:
-
categories:
-
---

## 概要

どんなものか自分で試してみたかった。
公式にあるniaru`APACHE KAFKA QUICKSTART`を参考にやってみる

### kafkaのインストール

最新版をインストールする。
ダウンロードページにtarファイルのパスが複数あったがとりあえず適当に。

```bash
$ curl -O https://ftp.riken.jp/net/apache/kafka/2.8.0/kafka_2.13-2.8.0.tgz
$ tar -xzf kafka_2.13-2.8.0.tgz
$ cd kafka_2.13-2.8.0
```

### kafkaサービスの起動

#### javaバージョンの確認

起動するのにJava8以上が必要らしい。
調べてみたら大丈夫だった。

```bash
$ java --version
# openjdk 12.0.1 2019-04-16
# OpenJDK Runtime Environment (build 12.0.1+12)
# OpenJDK 64-Bit Server VM (build 12.0.1+12, mixed mode, sharing)
```

#### ZooKeeperサービスの起動

公式の通りにコマンドを実行する。

```bash
$ bin/zookeeper-server-start.sh config/zookeeper.properties
#  .
#  .
#  .
# [2021-05-06 18:49:59,022] INFO PrepRequestProcessor (sid:0) started, reconfigEnabled=false (org.apache.zookeeper.server.PrepRequestProcessor)
# [2021-05-06 18:49:59,038] INFO Using checkIntervalMs=60000 maxPerMinute=10000 (org.apache.zookeeper.server.ContainerManager)
```

起動できたっぽい。次にkafkaサービスを起動する。こちらも公式のままコマンド実行する。

```bash
$ bin/kafka-server-start.sh config/server.properties
# [2021-05-06 18:52:56,405] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Started socket server acceptors and processors (kafka.network.SocketServer)
# [2021-05-06 18:52:56,414] INFO Kafka version: 2.8.0 (org.apache.kafka.common.utils.AppInfoParser)
# [2021-05-06 18:52:56,415] INFO Kafka commitId: ebb1d6e21cc92130 (org.apache.kafka.common.utils.AppInfoParser)
# [2021-05-06 18:52:56,415] INFO Kafka startTimeMs: 1620294776406 (org.apache.kafka.common.utils.AppInfoParser)
# [2021-05-06 18:52:56,422] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
# [2021-05-06 18:52:56,475] INFO [broker-0-to-controller-send-thread]: Recorded new controller, from now on will use broker 192.168.0.141:9092 (id: 0 rack: null) (kafka.server.BrokerToControllerRequestThread)
```

起動できたっぽい。

#### Topicの作成

```bash
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092

$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
Topic: quickstart-events	TopicId: eE4y5bTrRT2VUXXxGSXw0w	PartitionCount: 1	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: quickstart-events	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

Topic作成できた。

#### Topicにイベントを書き込む

```bash
$ bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
>This is my first event
>This is my second event
>^C
```

#### Topicのイベントを読み込む
```
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```
