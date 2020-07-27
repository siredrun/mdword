# Kafka安装

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Kafka    | Kafka  | √                 | √                 | √                 |

安装

```
cd /opt/software/
wget https://downloads.apache.org/kafka/2.4.1/kafka_2.11-2.4.1.tgz
tar -zxvf kafka_2.11-2.4.1.tgz -C /opt/module
mv /opt/module/kafka_2.11-2.4.1/ /opt/module/kf
vim /etc/profile # 文件某位加入下面内容
#KF_HOME
export KF_HOME=/opt/module/kf
export PATH=$PATH:$KF_HOME/bin

xsync /etc/profile
source /etc/profile # 每个节点都source一下
mkdir $KF_HOME/logs
vim $KF_HOME/config/server.properties #添加修改下面四处，分发后记得把broker.id改一下，不能重复
#broker的全局唯一编号，不能重复
broker.id=1
#删除topic功能
delete.topic.enable=true
#kafka运行日志存放的路径
log.dirs=/opt/module/kf/logs
#配置连接Zookeeper集群地址，末尾加/mykafka表示kafka相关的东西在zookeeper的/kafka，而不是根目录下，方便管理
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka

xsync $KF_HOME
vim $KF_HOME/config/server.properties #修改对应broker.id，不能重复
broker.id=2
broker.id=3

xcall grep 'broker.id' $KF_HOME/config/server.properties
要执行的命令是:grep broker.id /opt/module/kf/config/server.properties
-----------------------grep broker.id /opt/module/kf/config/server.properties---------------------
hadoop102 grep broker.id /opt/module/kf/config/server.properties
broker.id=1
-----------------------grep broker.id /opt/module/kf/config/server.properties---------------------
hadoop103 grep broker.id /opt/module/kf/config/server.properties
broker.id=2
-----------------------grep broker.id /opt/module/kf/config/server.properties---------------------
hadoop104 grep broker.id /opt/module/kf/config/server.properties
broker.id=3

# 出现$KF_HOME/bin/kafka-run-class.sh: line 315: exec: java: not found，jdk安装的地方和kafka默认的java安装目录不一致导致java命令找不到，添加一个java命令的link软链接到kafka默认的java安装目录去
xcall ln -s $JAVA_HOME/bin/java /usr/bin/java

#启动集群，依次在hadoop102、hadoop103、hadoop104节点上启动kafka
xcall $KF_HOME/bin/kafka-server-start.sh -daemon /opt/module/kf/config/server.properties
#关闭集群，依次在hadoop102、hadoop103、hadoop104节点上关闭kafka
xcall $KF_HOME/bin/kafka-server-stop.sh

# 群起脚本kf进行测试
kf start # 启动前必须保证zookeeper集群的启动 zk start
jpsall | grep Kafka #通过jps查看是否有实例，这里的K是大写
49386 Kafka
31760 Kafka
34792 Kafka

ls /kafka/brokers/ids # 也可以通过zkCli.sh进入客户端查看Kafka服务id
[1, 2, 3]
```

Kafka机器数量（经验公式）=2*（峰值生产速度*副本数/100）+1
先拿到峰值生产速度，再根据设定的副本数，就能预估出需要部署Kafka的数量。
比如我们的峰值生产速度是50M/s。副本数为2：Kafka机器数量=2\*（50\*2/100）+ 1=3台

# 命令行操作

```
kafka-topics.sh --zookeeper hadoop102:2181/kafka --list # 查看当前服务器中的所有topic
kafka-topics.sh --zookeeper hadoop102:2181/kafka \
--create --replication-factor 3 --partitions 1 --topic first # 创建topic
选项说明：
--topic 定义topic名
--replication-factor 定义副本数
--partitions 定义分区数
kafka-topics.sh --zookeeper hadoop102:2181/kafka \ --delete --topic first # 删除topic，需要server.properties中设置delete.topic.enable=true否则只是标记删除。
kafka-console-producer.sh --broker-list hadoop102:9092 --topic first 发送消息
kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first # 消费消息
--from-beginning 会把主题中以往所有的数据都读取出来。
kafka-topics.sh --zookeeper hadoop102:2181/kafka --describe --topic first #查看某个Topic的详情
kafka-topics.sh --zookeeper hadoop102:2181/kafka --alter --topic first --partitions 6 # 修改分区数
```

kafka tool2

```
下载安装并打开：https://www.kafkatool.com/download.html
在clusters右键 add new connection 在properties里面做一下配置
cluster name：随意填，如demo
zookeeper host：zookeeper主机，多个以“,”分割。如：hadoop102,hadoop103,hadoop104
zookeeper port：默认2181，不用管它
chroot path：/mykafka，zookeeper下的kafka路径 也就是config/server.properties文件里面的zookeeper.connect属性后面跟的值
```

# 压力测试

用Kafka官方自带的脚本，对Kafka进行压测。Kafka压测时，可以查看到哪个地方出现了瓶颈（CPU，内存，网络IO）。一般都是网络IO达到瓶颈。 

```
kafka-consumer-perf-test.sh
kafka-producer-perf-test.sh
```

Kafka Producer压力测试

```
kafka-producer-perf-test.sh --topic test --record-size 100 --num-records 100000 --throughput -1 --producer-props bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
100000 records sent, 78186.082877 records/sec (7.46 MB/sec), 277.65 ms avg latency, 513.00 ms max latency, 325 ms 50th, 503 ms 95th, 512 ms 99th, 513 ms 99.9th.

# record-size是一条信息有多大，单位是字节。
# num-records是总共发送多少条信息。
# throughput 是每秒多少条信息，设成-1，表示不限流，可测出生产者最大吞吐量。
# 打印参数解析：本例中一共写入10w条消息，吞吐量为7.46 MB/sec，每次写入的平均延迟为277.65毫秒，最大的延迟为513.00毫秒。
```

Kafka Consumer压力测试，如果这四个指标（IO，CPU，内存，网络）都不能改变，考虑增加分区数来提升性能。

```
kafka-consumer-perf-test.sh --broker-list hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic topic_start --fetch-size 10000 --messages 10000000 --threads 1
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
WARNING: Exiting before consuming the expected number of messages: timeout (10000 ms) exceeded. You can use the --timeout option to increase the timeout.
2020-07-15 23:12:49:816, 2020-07-15 23:13:00:806, 3.4430, 0.3133, 10864, 988.5350, 1594825970051, -1594825959061, -0.0000, -0.0000

# --zookeeper 指定zookeeper的链接信息 或--broker-list 指定kafka集群信息
# --topic 指定topic的名称
# --fetch-size 指定每次fetch的数据的大小
# --messages 总共要消费的消息个数
# 测试结果说明：
# 开始测试时间，测试结束数据，共消费数3.4430MB，吞吐量0.3133MB/s，共消费10864条，平均每秒消费**2988.5350条。
```