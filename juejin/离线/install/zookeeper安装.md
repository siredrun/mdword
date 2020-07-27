# 分布式安装部署

集群规划

| 服务名称  | 子服务           | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| --------- | ---------------- | ----------------- | ----------------- | ----------------- |
| Zookeeper | Zookeeper Server | √                 | √                 | √                 |

安装zookeeper的前提是安装jdk，本地安装的话cluster和datas里面的myid就不用配置。

```
cd /opt/software
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.5.7/apache-zookeeper-3.5.7-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module/
mv /opt/module/apache-zookeeper-3.5.7-bin/ /opt/module/zk
cd /opt/module/zk
vim /etc/profile # 下面内容添加到末尾
#ZK_HOME
export ZK_HOME=/opt/module/zk
export PATH=$PATH:$ZK_HOME/bin

xsync /etc/profile
source /etc/profile # 每个节点都source一下
mkdir -p $ZK_HOME/zkData
echo 1 > $ZK_HOME/zkData/myid # 分发后注意改为对应的server ID
mv $ZK_HOME/conf/zoo_sample.cfg $ZK_HOME/conf/zoo.cfg
vim $ZK_HOME/conf/zoo.cfg # 修改增加下面两处
dataDir=/opt/module/zk/zkData
#######################cluster##########################
server.1=hadoop102:2888:3888
server.2=hadoop103:2888:3888
server.3=hadoop104:2888:3888

# 为解决 “Error: JAVA_HOME is not set and java could not be found in PATH.”
vim $ZK_HOME/bin/zkEnv.sh # 添加Java路径
export JAVA_HOME=/opt/module/java

xsync $ZK_HOME
echo 2 > $ZK_HOME/zkData/myid # 修改server ID
echo 3 > $ZK_HOME/zkData/myid # 修改server ID
xcall cat $ZK_HOME/zkData/myid
要执行的命令是:cat /opt/module/zk/zkData/myid
-----------------------hadoop102---------------------
hadoop102 cat /opt/module/zk/zkData/myid
1
-----------------------hadoop103---------------------
hadoop103 cat /opt/module/zk/zkData/myid
2
-----------------------hadoop104---------------------
hadoop104 cat /opt/module/zk/zkData/myid
3
```

测试脚本zk

```
zk start
# 任意节点执行下面任意服务器就表示成功
zkCli.sh -server hadoop102:2181
zkCli.sh -server hadoop103:2181
zkCli.sh -server hadoop104:2181
```