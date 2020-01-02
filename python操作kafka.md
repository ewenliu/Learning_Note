# python操作kafka

[toc]

## 一、环境介绍
- Python 3.7.0
- kafka-python 1.4.7
- kafka_2.11-2.4.0(windows)

## 二、连接器安装和topic创建

kafka-python安装：
> pip install kafka-python

测试topic创建
```cmd
D:\kafka\kafka_2.11-2.4.0\bin\windows>kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka_test
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic kafka_test.
```

## 三、kafka生产者

```python
# produer.py
# -*- coding: utf-8 -*-#

from kafka import KafkaProducer

# 此处ip可以是多个['0.0.0.1:9092','0.0.0.2:9092','0.0.0.3:9092' ]
producer = KafkaProducer(bootstrap_servers=['127.0.0.1:9092'])

for i in range(3):
    msg = "msg %d" % i
    # 生产者发过去的msg类型要属于(bytes, bytearray, memoryview, type(None))，否则会报错，所以将msg按照utf8转化为bytes
    msg_to_send = bytes(msg, encoding='utf8')
    producer.send(topic='kafka_test', value=msg_to_send)

producer.close()


```

## 四、kafka消费者

```python
# consumer.py
# -*- coding: utf-8 -*-#
from kafka import KafkaConsumer

consumer = KafkaConsumer('kafka_test', bootstrap_servers=["127.0.0.1:9092"])

for msg in consumer:
    recv = "%s:%d:%d: key=%s value=%s" % (msg.topic, msg.partition, msg.offset, msg.key, msg.value)
    print(recv)

```

## 运行测试代码

1. 先运行consumer.py
2. 再运行produer.py
3. 发现consumer的控制台有消息被消费