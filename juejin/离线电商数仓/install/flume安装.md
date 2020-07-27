# flume安装

集群信息：版本1.7

| 服务名称           | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| ------------------ | ------ | ----------------- | ----------------- | ----------------- |
| Flume(采集日志)    | flume  | √                 | √                 |                   |
| Flume（消费Kafka） | flume  |                   |                   | √                 |

安装

```
xcall yum install -y nc
cd /opt/software/
wget http://archive.apache.org/dist/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz
tar -zxvf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
mv /opt/module/apache-flume-1.7.0-bin/ /opt/module/fl # 注意这里l是字母L，不是数字1
vim /etc/profile # 文件某位加入下面内容
#FL_HOME
export FL_HOME=/opt/module/fl
export PATH=$PATH:$FL_HOME/bin

xsync /etc/profile
source /etc/profile # 每个节点都source一下
mv $FL_HOME/conf/flume-env.sh.template $FL_HOME/conf/flume-env.sh
vim $FL_HOME/conf/flume-env.sh #加入Java路径
export JAVA_HOME=/opt/module/java

mkdir $FL_HOME/myagents #myagents存放收集配置文件agent目录
xsync $FL_HOME
```