# hadoop安装

集群规划

| 服务名称 | 子服务            | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ----------------- | ----------------- | ----------------- | ----------------- |
| HDFS     | NameNode          | √                 |                   |                   |
|          | DataNode          | √                 | √                 | √                 |
|          | SecondaryNameNode |                   |                   | √                 |
| Yarn     | NodeManager       | √                 | √                 | √                 |
|          | Resourcemanager   |                   | √                 |                   |

## 基本安装

```
cd /opt/software
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz
tar -zxvf /opt/software/hadoop-2.7.2.tar.gz -C /opt/module/
vim /etc/profile # 环境变量，在profile文件末尾添加JDK路径：（shitf+g）
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
xsync /opt/module/hadoop-2.7.2
xsync /etc/profile
source /etc/profile #每个节点都来一下
xcall $HADOOP_HOME/bin/hadoop version
```

## hadoop目录

```
bin目录：存放对Hadoop相关服务（HDFS,YARN）进行操作的脚本
etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
sbin目录：存放启动或停止Hadoop相关服务的脚本
share目录：存放Hadoop的依赖jar包、文档、和官方案例
```

## 配置文件说明

所有的配置文件都在$HADOOP_HOME/etc/hadoop下。

必须配置

```
core-site.xml
hdfs-site.xml
yarn-site.xml
mapred-site.xml
slaves
```

配置Java_home的页面

```
hadoop-env.sh
yarn-env.sh
mapred-env.sh
```

上面三个配置文件直接加入JAVA安装路径即可

```
export JAVA_HOME=/opt/module/java
```

## core-site.xml

```
vim $HADOOP_HOME/etc/hadoop/core-site.xml # <configuration>内加入配置
<!-- 指定HDFS中NameNode的地址 -->
<property>
		<name>fs.defaultFS</name>
      <value>hdfs://hadoop102:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/module/hadoop-2.7.2/data/tmp</value>
</property>
```

## hdfs-site.xml

```
vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml 

<!-- SecondaryNameNode：指定Hadoop辅助名称节点主机配置 -->
<property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop104:50090</value>
</property>
```

## yarn-site.xml

```
vim etc/hadoop/yarn-site.xml
<!-- 日志聚集功能使能 -->
<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
</property>

<!-- Reducer获取数据的方式 -->
<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoop103</value>
</property>
```

## mapred-site.xml

```
mv $HADOOP_HOME/etc/hadoop/mapred-site.xml.template $HADOOP_HOME/etc/hadoop/mapred-site.xml
vim $HADOOP_HOME/etc/hadoop/mapred-site.xml
<!-- 指定MR运行在Yarn上 -->
<property>
   <name>mapreduce.framework.name</name>
   <value>yarn</value>
</property>
<!-- 历史服务器端地址 -->
<property>
	<name>mapreduce.jobhistory.address</name>
	<value>hadoop102:10020</value>
</property>
<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>
<!--第三方框架使用yarn计算的日志聚集功能 -->
<property>         
	<name>yarn.log.server.url</name>      
	<value>http://hadoop102:19888/jobhistory/logs</value>
</property>
```

## slaves

```
vim $HADOOP_HOME/etc/hadoop/slaves 
hadoop102
hadoop103
hadoop104
```

依次配置完后分发

```
xsync $HADOOP_HOME
```

## 集群启动

如果集群是第一次启动，需要格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除$HADOOP_HOME下的data和log数据）。

```
hdfs namenode -format # 在主节点上格式化，两个命令都可以
hadoop namenode -format
```

出现“Storage directory /opt/module/hadoop-2.7.2/data/tmp/dfs/name has been successfully formatte”表示格式成功。

在有resourcemanager的节点启动start-dfs.sh和start-yarn.sh

start-dfs.sh

```
start-dfs.sh
Starting namenodes on [hadoop102]
hadoop102: starting namenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-namenode-hadoop102.out
hadoop102: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop102.out
hadoop104: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop104.out
hadoop103: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop103.out
Starting secondary namenodes [hadoop104]
hadoop104: starting secondarynamenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-secondarynamenode-hadoop104.out
```

start-yarn.sh

```
start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-resourcemanager-hadoop103.out
hadoop104: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop104.out
hadoop102: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop102.out
hadoop103: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop103.out
```

jps查看实例

```
jpsall
========== hadoop102 ==========
9728 NodeManager
9522 NameNode
9611 DataNode
9838 Jps
0
========== hadoop103 ==========
4641 ResourceManager
4745 NodeManager
4379 DataNode
5037 Jps
0
========== hadoop104 ==========
4647 NodeManager
4555 SecondaryNameNode
4765 Jps
4462 DataNode
0
```

访问http://hadoop102:50070/ 

![](https://user-gold-cdn.xitu.io/2020/6/29/172fde5ca16fcbad?w=1198&h=791&f=png&s=71520)

访问http://hadoop104:50090/

![](https://user-gold-cdn.xitu.io/2020/6/29/172fde5ca9bcc7a1?w=1452&h=550)

## 启动测试

hd群启hadoop脚本

测试hd start

```
hd start #任意服务器执行
Starting namenodes on [hadoop102]
hadoop102: starting namenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-namenode-hadoop102.out
hadoop102: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop102.out
hadoop104: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop104.out
hadoop103: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop103.out
Starting secondary namenodes [hadoop104]
hadoop104: starting secondarynamenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-secondarynamenode-hadoop104.out
starting yarn daemons
starting resourcemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-resourcemanager-hadoop103.out
hadoop104: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop104.out
hadoop102: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop102.out
hadoop103: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop103.out
starting historyserver, logging to /opt/module/hadoop-2.7.2/logs/mapred-root-historyserver-hadoop102.out
jpsall #查看实例
========== hadoop102 ==========
10388 NodeManager
10615 Jps
10283 DataNode
10189 NameNode
10477 JobHistoryServer
0
========== hadoop103 ==========
6118 NodeManager
5755 DataNode
6012 ResourceManager
6430 Jps
0
========== hadoop104 ==========
5008 DataNode
5108 SecondaryNameNode
5318 Jps
5182 NodeManager
0
```

测试hd stop

```
[root@hadoop104 ~]# hd stop
Stopping namenodes on [hadoop102]
hadoop102: stopping namenode
hadoop102: stopping datanode
hadoop104: stopping datanode
hadoop103: stopping datanode
Stopping secondary namenodes [hadoop104]
hadoop104: stopping secondarynamenode
stopping yarn daemons
stopping resourcemanager
hadoop102: stopping nodemanager
hadoop104: stopping nodemanager
hadoop103: stopping nodemanager
no proxyserver to stop
stopping historyserver
[root@hadoop104 ~]# jpsall
========== hadoop102 ==========
10762 Jps
0
========== hadoop103 ==========
6874 Jps
0
========== hadoop104 ==========
5434 Jps
0
```

测试restart

```
hd restart
---------------stop---------------
Stopping namenodes on [hadoop102]
hadoop102: stopping namenode
hadoop102: stopping datanode
hadoop104: stopping datanode
hadoop103: stopping datanode
Stopping secondary namenodes [hadoop104]
hadoop104: stopping secondarynamenode
stopping yarn daemons
stopping resourcemanager
hadoop102: stopping nodemanager
hadoop103: stopping nodemanager
hadoop104: stopping nodemanager
no proxyserver to stop
stopping historyserver
---------------start---------------
Starting namenodes on [hadoop102]
hadoop102: starting namenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-namenode-hadoop102.out
hadoop102: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop102.out
hadoop104: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop104.out
hadoop103: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop103.out
Starting secondary namenodes [hadoop104]
hadoop104: starting secondarynamenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-secondarynamenode-hadoop104.out
starting yarn daemons
starting resourcemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-resourcemanager-hadoop103.out
hadoop104: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop104.out
hadoop102: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop102.out
hadoop103: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop103.out
starting historyserver, logging to /opt/module/hadoop-2.7.2/logs/mapred-root-historyserver-hadoop102.out
```

## 访问

| **服务名称** | **子服务**        | 服务器        | 端口  |
| ------------ | ----------------- | ------------- | ----- |
| hdfs         | NameNode          | hadoop102     | 50070 |
| yarn         | Resourcemanager   | hadoop103     | 8088  |
| hdfs         | SecondaryNameNode | hadoop104     | 50090 |
|              | jobhistory        | hadoop102     | 19888 |
| yarn         | NodeManager       | hadoop10[2-4] | 8042  |

相关网站

```
http://hadoop102:50070/  NameNode
http://hadoop103:8088/ Resourcemanager
http://hadoop104:50090/ SecondaryNameNode
```

访问http://hadoop102:50070/  NameNode

![](https://user-gold-cdn.xitu.io/2020/6/29/172fde5ca16fcbad?w=1198&h=791&f=png&s=71520)

访问http://hadoop103:8088/

![](https://user-gold-cdn.xitu.io/2020/6/29/172fde5caa3ed51c?w=1707&h=472&f=png&s=119872)

访问http://hadoop104:50090/

![](https://user-gold-cdn.xitu.io/2020/6/29/172fde5ca9bcc7a1?w=1452&h=550)

## hadoop-lzo编译

基本依赖

```
xcall yum -y install gcc
xcall yum -y install gcc-c++
xcall yum -y install lzo-devel
xcall yum -y install zlib-devel
xcall yum -y install autoconf
xcall yum -y install automake
xcall yum -y install libtool
xcall yum -y install git
wget https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /opt/module/
mv /opt/module/apache-maven-3.6.3/ /opt/module/mvn
vim /etc/profile #文件末尾加入下面内容
#MVN_HOME
export MVN_HOME=/opt/module/mvn
export PATH=$PATH:$MVN_HOME/bin
vim /opt/module/mvn/conf/settings.xml # 修改sitting.xml加阿里云镜像
	<mirror>  
		<id>alimaven</id>  
		<name>aliyun maven</name>  
		<url>https://maven.aliyun.com/nexus/content/groups/public/</url>  
		<mirrorOf>central</mirrorOf>          
    </mirror>
xsync /opt/module/mvn
xsync /etc/profile
source /etc/profile #每个节点都执行一下
mvn -v #版本验证
```

下载安装编译

```
wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz
tar -zxvf lzo-2.10.tar.gz
xsync lzo-2.10
cd lzo-2.10 #每个节点都执行
./configure -prefix=/usr/local/hadoop/lzo/ && make && make install #每个节点都执行
git clone https://github.com/twitter/hadoop-lzo/
vim hadoop-lzo/pom.xml 修改pom.xml，写自己的Hadoop版本号<hadoop.current.version>2.7.2</hadoop.current.version>
# 声明两个临时环境变量
export C_INCLUDE_PATH=/usr/local/hadoop/lzo/include
export LIBRARY_PATH=/usr/local/hadoop/lzo/lib 
cd hadoop-lzo
mvn package -Dmaven.test.skip=true # 进入hadoop-lzo，执行maven编译，时间稍微有点长，请耐心等待。
cd target # 进入target，hadoop-lzo-0.4.21-SNAPSHOT.jar 即编译成功的hadoop-lzo组件
stat hadoop-lzo-0.4.21-SNAPSHOT.jar # 使用stat(显示inode信息)命令可查看一个文件的某些信息
  File: ‘hadoop-lzo-0.4.21-SNAPSHOT.jar’
  Size: 188934    	Blocks: 376        IO Block: 4096   regular file
Device: fd01h/64769d	Inode: 788007      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-06-29 08:25:40.991523773 +0800
Modify: 2020-06-29 08:24:13.244178460 +0800
Change: 2020-06-29 08:24:13.244178460 +0800
 Birth: -
```

## hadoop支持lzo压缩

LzopCodec是LzoCodec的升级版（Plus），LzopCodec会压缩的文件中添加一些header信息，保存压缩的元数据。LzoCodec后缀 .lzo_deflate，LzopCodec后缀  .lzo。.lzo_deflate格式的压缩文件，作为MR的输入时，无法使用程序对此文件进行切片；.lzo_deflate格式被Mapper读取时，会乱码。一般都使用且一定使用LzopCodec格式，尤其作为reduce的输出时。在shuffle阶段的话，用哪个都行。

将编译好后的hadoop-lzo-0.4.21-SNAPSHOT.jar 放入$HADOOP_HOME/share/hadoop/common/

```
cp hadoop-lzo-0.4.21-SNAPSHOT.jar $HADOOP_HOME/share/hadoop/common/
xsync $HADOOP_HOME/share/hadoop/common/
vim $HADOOP_HOME/etc/hadoop/core-site.xml #加入配置
<property>
	<name>io.compression.codecs</name>
	<value>
	org.apache.hadoop.io.compress.GzipCodec,
	org.apache.hadoop.io.compress.DefaultCodec,
	org.apache.hadoop.io.compress.BZip2Codec,
	org.apache.hadoop.io.compress.SnappyCodec,
	com.hadoop.compression.lzo.LzoCodec,
	com.hadoop.compression.lzo.LzopCodec
	</value>
</property>

<property>
    <name>io.compression.codec.lzo.class</name>
    <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>

xsync $HADOOP_HOME/etc/hadoop/core-site.xml
xcall $HADOOP_HOME/bin/hadoop checknative # 查询所有节点是否有显示lz4，hadoop checknative查看hadoop支持的压缩方式。
hd stop #集群停止
xcall rm -rf $HADOOP_HOME/data
xcall rm -rf $HADOOP_HOME/logs
hadoop namenode -format #主节点上执行
hd start #集群开启
echo hello > hello
hadoop fs -mkdir /wc
hadoop fs -put hello /wc
```

执行（就是这么长，没有报错表示成功）

```
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=com.hadoop.compression.lzo.LzopCodec /wc /wcoutput
```

测试

```
hadoop fs -ls /wcoutput #会生成一个/wcoutput/part-r-00000.lzo 后缀lzo的文件
Found 2 items
-rw-r--r--   3 root supergroup          0 2020-06-29 09:40 /wcoutput/_SUCCESS
-rw-r--r--   3 root supergroup         58 2020-06-29 09:40 /wcoutput/part-r-00000.lzo
# 为lzo文件创建索引
hadoop jar $HADOOP_HOME/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /wcoutput

hadoop fs -ls /wcoutput # 会增一个*.lzo.index文件。
Found 3 items
-rw-r--r--   3 root supergroup          0 2020-06-29 09:40 /wcoutput/_SUCCESS
-rw-r--r--   3 root supergroup         58 2020-06-29 09:40 /wcoutput/part-r-00000.lzo
-rw-r--r--   3 root supergroup          8 2020-06-29 09:47 /wcoutput/part-r-00000.lzo.index
```

## 基准测试

测试HDFS写性能

```
# 向HDFS集群写10个128M的文件
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB

# 下面是测试展示的部分结果
20/06/29 09:57:41 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
20/06/29 09:57:41 INFO fs.TestDFSIO:            Date & time: Mon Jun 29 09:57:41 CST 2020
20/06/29 09:57:41 INFO fs.TestDFSIO:        Number of files: 10 # 文件数量
20/06/29 09:57:41 INFO fs.TestDFSIO: Total MBytes processed: 1280.0 # 文件总大小
20/06/29 09:57:41 INFO fs.TestDFSIO:      Throughput mb/sec: 28.051720359412666 # 吞吐量/每秒 单位：mb
20/06/29 09:57:41 INFO fs.TestDFSIO: Average IO rate mb/sec: 28.580001831054688 # 平均值/每秒 单位：mb
20/06/29 09:57:41 INFO fs.TestDFSIO:  IO rate std deviation: 4.190169301401879 # 标准差 std standard
20/06/29 09:57:41 INFO fs.TestDFSIO:     Test exec time sec: 31.763
```

总结：一秒10mb（大概） 1h大概36000mb 36G。

测试HDFS读性能

```
# 读取HDFS集群10个128M的文件
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -read -nrFiles 10 -fileSize 128MB

...
20/06/29 10:08:05 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
20/06/29 10:08:05 INFO fs.TestDFSIO:            Date & time: Mon Jun 29 10:08:05 CST 2020
20/06/29 10:08:05 INFO fs.TestDFSIO:        Number of files: 10
20/06/29 10:08:05 INFO fs.TestDFSIO: Total MBytes processed: 1280.0
20/06/29 10:08:05 INFO fs.TestDFSIO:      Throughput mb/sec: 254.6250248657251
20/06/29 10:08:05 INFO fs.TestDFSIO: Average IO rate mb/sec: 276.227294921875
20/06/29 10:08:05 INFO fs.TestDFSIO:  IO rate std deviation: 83.65365900630701
20/06/29 10:08:05 INFO fs.TestDFSIO:     Test exec time sec: 23.601
20/06/29 10:08:05 INFO fs.TestDFSIO: 
```

总结：一秒270mb，一小时972000mb 972G。

生成文件

```
hadoop fs -ls /benchmarks/TestDFSIO
Found 4 items
drwxr-xr-x   - root supergroup          0 2020-06-29 10:07 /benchmarks/TestDFSIO/io_control
drwxr-xr-x   - root supergroup          0 2020-06-29 09:57 /benchmarks/TestDFSIO/io_data
drwxr-xr-x   - root supergroup          0 2020-06-29 10:08 /benchmarks/TestDFSIO/io_read
drwxr-xr-x   - root supergroup          0 2020-06-29 09:57 /benchmarks/TestDFSIO/io_write
```

删除测试生成数据，hdfs路径/benchmarks文件夹里的内容将被清空

```
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -clean
```