# Kafka安装

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Kafka    | Kafka  | √                 | √                 | √                 |

安装

```
cd /opt/software/
wget https://downloads.apache.org/kafka/2.5.0/kafka_2.12-2.5.0.tgz
tar -zxvf kafka_2.12-2.5.0.tgz -C /opt/module
mv /opt/module/kafka_2.12-2.5.0/ /opt/module/kf
vim /etc/profile # 文件某位加入下面内容
#KF_HOME
export KF_HOME=/opt/module/kf
export PATH=$PATH:$KF_HOME/bin

xsync /etc/profile
source /etc/profile # 每个节点都source一下
mkdir $KF_HOME/datas
vim $KF_HOME/config/server.properties #添加修改下面四处，分发后记得把broker.id改一下，不能重复
#broker的全局唯一编号，不能重复
broker.id=102
#删除topic功能使能
delete.topic.enable=true
#kafka运行日志存放的路径	
log.dirs=/opt/module/kf/datas
#配置连接Zookeeper集群地址，末尾加/mykafka表示kafka相关的东西在zookeeper的/mykafka，而不是根目录下，方便管理。
进入命令行：zkCli.sh -server hadoop103:2181
输入ls /mykafka出现厦门内容表示成功
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/mykafka

xsync $KF_HOME
vim $KF_HOME/config/server.properties #修改对应broker.id，不能重复
broker.id=103
broker.id=104

xcall grep 'broker.id' $KF_HOME/config/server.properties
要执行的命令是:grep broker.id /opt/module/kf/config/server.properties
-----------------------hadoop102---------------------
hadoop102 grep broker.id /opt/module/kf/config/server.properties
broker.id=102
-----------------------hadoop103---------------------
hadoop103 grep broker.id /opt/module/kf/config/server.properties
broker.id=103
-----------------------hadoop104---------------------
hadoop104 grep broker.id /opt/module/kf/config/server.properties
broker.id=104

# 出现$KF_HOME/bin/kafka-run-class.sh: line 315: exec: java: not found，jdk安装的地方和kafka默认的java安装目录不一致导致java命令找不到，添加一个java命令的link软链接到kafka默认的java安装目录去
xcall ln -s $JAVA_HOME/bin/java /usr/bin/java
# 群起脚本kf进行测试
kf start # 启动前必须保证zookeeper集群的启动 zk start
jpsall|grep 'Quo' #通过jps查看是否有实例
4516 QuorumPeerMain
11622 QuorumPeerMain
5969 QuorumPeerMain
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