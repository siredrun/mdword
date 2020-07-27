# flume安装

集群信息：版本1.9

| 服务名称           | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| ------------------ | ------ | ----------------- | ----------------- | ----------------- |
| Flume(采集日志)    | flume  | √                 | √                 |                   |
| Flume（消费Kafka） | flume  |                   |                   | √                 |

安装

```
xcall yum install -y nc
cd /opt/software/
wget http://archive.apache.org/dist/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
tar -zxvf apache-flume-1.9.0-bin.tar.gz -C /opt/module/
mv /opt/module/apache-flume-1.9.0-bin/ /opt/module/fl # 注意这里l是字母L，不是数字1
vim /etc/profile # 文件某位加入下面内容
#FL_HOME
export FL_HOME=/opt/module/fl
export PATH=$PATH:$FL_HOME/bin

xsync /etc/profile
source /etc/profile # 每个节点都source一下
xcall rm /opt/module/fl/lib/guava-11.0.2.jar # 将lib文件夹下的guava-11.0.2.jar删除以兼容Hadoop 3.1.3
xcall cp /opt/module/hd/share/hadoop/hdfs/lib/guava-27.0-jre.jar /opt/module/fl/lib/
xcall ls /opt/module/fl/lib/guava-27.0-jre.jar 
要执行的命令是:ls /opt/module/fl/lib/guava-27.0-jre.jar
-----------------------ls /opt/module/fl/lib/guava-27.0-jre.jar---------------------
hadoop102 ls /opt/module/fl/lib/guava-27.0-jre.jar
/opt/module/fl/lib/guava-27.0-jre.jar
-----------------------ls /opt/module/fl/lib/guava-27.0-jre.jar---------------------
hadoop103 ls /opt/module/fl/lib/guava-27.0-jre.jar
/opt/module/fl/lib/guava-27.0-jre.jar
-----------------------ls /opt/module/fl/lib/guava-27.0-jre.jar---------------------
hadoop104 ls /opt/module/fl/lib/guava-27.0-jre.jar
/opt/module/fl/lib/guava-27.0-jre.jar

mv $FL_HOME/conf/flume-env.sh.template $FL_HOME/conf/flume-env.sh
vim $FL_HOME/conf/flume-env.sh #加入Java路径
export JAVA_HOME=/opt/module/java

xsync $FL_HOME
xcall $FL_HOME/bin/flume-ng version
要执行的命令是:/opt/module/fl/bin/flume-ng version
-----------------------/opt/module/fl/bin/flume-ng version---------------------
hadoop102 /opt/module/fl/bin/flume-ng version
Flume 1.9.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: d4fcab4f501d41597bc616921329a4339f73585e
Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
From source with checksum 35db629a3bda49d23e9b3690c80737f9
-----------------------/opt/module/fl/bin/flume-ng version---------------------
hadoop103 /opt/module/fl/bin/flume-ng version
Flume 1.9.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: d4fcab4f501d41597bc616921329a4339f73585e
Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
From source with checksum 35db629a3bda49d23e9b3690c80737f9
-----------------------/opt/module/fl/bin/flume-ng version---------------------
hadoop104 /opt/module/fl/bin/flume-ng version
Flume 1.9.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: d4fcab4f501d41597bc616921329a4339f73585e
Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
From source with checksum 35db629a3bda49d23e9b3690c80737f9
```

# guava包冲突

注意：关于删除所有节点的$FL_HOME/lib/guava-11.0.2.jar在hadoop102、hadoop103（把日志采集到kafka）上采集日志会报ClassNotFoundException: com.google.common.collect.Lists（这个类属于guava jar包）。如果所有节点都不删除，那在hadoop104消费日志（kafka消费数据到hdfs）会因为hdfs和flume都有guava包产生冲突，报异常，如下：
ERROR hdfs.HDFSEventSink: process failed java.lang.NoSuchMethodError:com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
......
Exception in thread "SinkRunner-PollingRunner-DefaultSinkProcessor" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V。
解决方案：2种
1.在需要采集日志的节点如Hadoop102和Hadoop103上不删除$FL_HOME/lib/guava-11.0.2.jar，消费日志节点如Hadoop104把hadoop的guava包（路径$HD_HOME/hd/share/hadoop/hdfs/lib/guava-27.0-jre.jar）往$FL_HOME/lib下拷贝一份。
2.直接把所有节点$FL_HOME/lib下的guava-11.0.2.jar删除，然后把所有节点的$HD_HOME/hd/share/hadoop/hdfs/lib的/guava-27.0-jre.jar往FL_HOME/lib下都拷贝一份。建议使用，本教程采用的是第二种。

# Flume内存优化

如果启动消费Flume抛出如下异常

```
ERROR hdfs.HDFSEventSink: process failed
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

解决方案步骤：
/opt/module/f1/conf/flume-env.sh文件中增加如下配置并同步

```
export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"
```

Flume内存参数设置及优化
JVM heap一般设置为4G或更高，部署在单独的服务器上（4核8线程16G内存）
-Xmx与-Xms最好设置一致，减少内存抖动带来的性能影响，如果设置不一致容易导致频繁fullgc。
-Xms表示JVM Heap(堆内存)最小尺寸，初始分配；-Xmx 表示JVM Heap(堆内存)最大允许的尺寸，按需分配。如果不设置一致，容易在初始化时，由于内存不够，频繁触发fullgc。