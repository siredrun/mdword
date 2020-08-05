# hbase安装

前提：安装java、hadoop、zookeeper、hive

集群规划：版本2.2。hbase2.2兼容hadoop3.1和3.2、java8

| 服务名称 | 子服务        | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------------- | ----------------- | ----------------- | ----------------- |
| Hbase    | HMaster       | √                 |                   |                   |
|          | HRegionServer | √                 | √                 | √                 |

安装

```
cd /opt/software
wget https://mirror.bit.edu.cn/apache/hbase/stable/hbase-2.2.4-bin.tar.gz
tar -zxvf hbase-2.2.4-bin.tar.gz -C /opt/module/
mv /opt/module/hbase-2.2.4/ /opt/module/hbase
vim /etc/profile
#HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin

xsync /etc/profile
source /etc/profile
vim $HBASE_HOME/conf/hbase-env.sh
export HBASE_MANAGES_ZK=false
export JAVA_HOME=/opt/module/java

vim $HBASE_HOME/conf/hbase-site.xml
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop102:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop102,hadoop103,hadoop104</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
<property>
<name>hbase.wal.provider</name>
<value>filesystem</value>
</property>

vim $HBASE_HOME/conf/regionservers：
hadoop102
hadoop103
hadoop104

# 软连接hadoop配置文件到HBase：
ln -s $HD_HOME/etc/hadoop/core-site.xml /opt/module/hbase/conf/core-site.xml
ln -s $HD_HOME/etc/hadoop/hdfs-site.xml /opt/module/hbase/conf/hdfs-site.xml
xsync $HBASE_HOME # 分发 
# 在哪台服务器启动master，就表示此台机器启动HMaster服务
hbase-daemon.sh start master #第一种方式
hbase-daemon.sh start regionserver
start-hbase.sh # 第二种方式 推荐，关闭stop-hbase.sh
jpsall|grep H
45884 HRegionServer
44543 HMaster
19634 HRegionServer
17874 HRegionServer

提示：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出ClockOutOfSyncException异常。
修复提示：
a、同步时间服务
b、属性：hbase.master.maxclockskew设置更大的值
vim $HBASE_HOME/conf/hbase-site.xml
<property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>
        <description>Time difference of regionserver from master</description>
</property>
```

访问web UI：http://hadoop102:16010/