# hive安装

前提：安装好高可用mysql、Java、hadoop2，hive2.xy匹配hadoop2.xy，hive3.xy匹配hadoop3.xy。

集群规划

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Hive     | Hive   | √                 |                   |                   |

安装在hadoop102

```
cd /opt/software
wget https://downloads.apache.org/hive/hive-2.3.7/apache-hive-2.3.7-bin.tar.gz
tar -zxvf apache-hive-2.3.7-bin.tar.gz -C /opt/module/
mv /opt/module/apache-hive-2.3.7-bin/ /opt/module/hive
vim /etc/profile
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile
mv $HIVE_HOME/conf/hive-env.sh.template $HIVE_HOME/conf/hive-env.sh
vim $HIVE_HOME/conf/hive-env.sh # 加入下面配置
export HADOOP_HOME=/opt/module/hadoop-2.7.2 # 配置HADOOP_HOME路径
export HIVE_CONF_DIR=/opt/module/hive/conf # 配置HIVE_CONF_DIR路径

# hive log默认存放在/tmp/root/hive.log目录下（当前用户名下）,这一项可不配，不是强制性要求。
mv $HIVE_HOME/conf/hive-log4j2.properties.template $HIVE_HOME/conf/hive-log4j2.properties
vim  $HIVE_HOME/conf/hive-log4j2.properties
hive.log.dir=/opt/module/hive/logs

#显示当前数据库，以及查询表的头信息配置
hd start # 保证启动start-dfs.sh和start-yarn.sh
Starting namenodes on [hadoop102]
hadoop102: starting namenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-namenode-hadoop102.out
hadoop103: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop103.out
hadoop104: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop104.out
hadoop102: starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-datanode-hadoop102.out
Starting secondary namenodes [hadoop104]
hadoop104: starting secondarynamenode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-root-secondarynamenode-hadoop104.out
starting yarn daemons
starting resourcemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-resourcemanager-hadoop103.out
hadoop104: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop104.out
hadoop102: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop102.out
hadoop103: starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-root-nodemanager-hadoop103.out
starting historyserver, logging to /opt/module/hadoop-2.7.2/logs/mapred-root-historyserver-hadoop102.out

hadoop fs -mkdir /tmp
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -chmod +x /tmp
hadoop fs -chmod +x /user/hive/warehouse
```

## 配置hive的元数据存储在mysql

```
下载地址：https://dev.mysql.com/downloads/connector/j/
wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.49.tar.gz
tar -zxvf mysql-connector-java-5.1.49.tar.gz
cp mysql-connector-java-5.1.49/mysql-connector-java-5.1.49-bin.jar $HIVE_HOME/lib
ls $HIVE_HOME/lib |grep 'mysql-con'
mysql-connector-java-5.1.49-bin.jar

ip a|grep 'scope global eth0' # 查看MySQL高可用虚拟IP，保证keepalive、mysql服务的启动。
inet 192.168.200.100/32 scope global eth0

vim $HIVE_HOME/conf/hive-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
        <property>
          <name>javax.jdo.option.ConnectionURL</name>
          <!-- 有MySQL高可用虚拟IP就用高可用IP，没有采用实际IP -->
          <value>jdbc:mysql://192.168.200.100:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
          <description>JDBC connect string for a JDBC metastore</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionDriverName</name>
          <value>com.mysql.jdbc.Driver</value>
          <description>Driver class name for a JDBC metastore</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>root</value>
          <description>username to use against metastore database</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>1234</value>
          <description>password to use against metastore database</description>
        </property>
        <!-- 显示当前数据库，以及查询表的头信息配置 -->
        <property>
                <name>hive.cli.print.header</name>
                <value>true</value>
        </property>

        <property>
                <name>hive.cli.print.current.db</name>
                <value>true</value>
        </property>
</configuration>

#创建hive数据库和建立hive相关表
cd  $HIVE_HOME/scripts/metastore/upgrade/mysql
mysql -uroot -p1234
create database hive;
use hive;
source hive-schema-2.3.0.mysql.sql; #hive版本是2.3.7，所以选择2.3.0.

vim $HADOOP_HOME/etc/hadoop/core-site.xml
<!-- 允许运行任何账户 -->
<property>
				<name>hadoop.proxyuser.root.hosts</name>
				<value>*</value>
</property>
<property>
				<name>hadoop.proxyuser.root.groups</name>
				<value>*</value>
</property>

xsync $HADOOP_HOME/etc/hadoop/core-site.xml
hd restart
hiveserver2 # 启动hiveserver2
启动成功访问http://hadoop102:10002/ HiveServer2: Web UI
beeline #另外启动一个窗口
beeline> !connect jdbc:hive2://hadoop102:10000
Connecting to jdbc:hive2://hadoop102:10000
Enter username for jdbc:hive2://hadoop102:10000: root
Enter password for jdbc:hive2://hadoop102:10000: (直接回车)
Connected to: Apache Hive (version 2.3.7)
Driver: Hive JDBC (version 2.3.7)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://hadoop102:10000>

hive日志/tmp/root/hive.log 家目录下，有什么问题请查看此日志。
# hive查询异常：Cannot create directory /tmp/hive-root/。。。Name node is in safe mode
hadoop为了防止数据丢失，启动了“安全模式”的设置，每次启动hadoop后一段时间内集群处于安全模式，在该模式下，文件系统中的内容不允许修改也不允许删除，直到安全模式结束。运行期通过命令也可以进入 安全模式。系统启动的时候去修改和删除文件也会有安全模式不允许修改的出错提示，只需要等待一会儿即可。     

 直接在bash输入指令脱离安全模式（推荐）

在安全模式下输入指令： hadoop dfsadmin -safemode leave
```

## Hive运行引擎Tez

```
Tez是一个Hive的运行引擎，性能优于MR，支持DAG。用Hive直接编写MR程序，假设有四个有依赖关系的MR作业，需要将中间结果持久化写到HDFS。Tez可以将多个有依赖的作业转换为一个作业，这样只需写一次HDFS，且中间节点较少，从而大大提升作业的计算性能。Tez安装在有hive的机器上。

cd /opt/software
wget https://downloads.apache.org/tez/0.9.2/apache-tez-0.9.2-bin.tar.gz
tar -axvf apache-tez-0.9.2-bin.tar.gz -C /opt/module/
mv /opt/module/apache-tez-0.9.1-bin/ /opt/module/tez
hadoop fs -mkdir /tez
hadoop fs -put /opt/module/tez/share/tez.tar.gz /tez # 一定是拷贝压缩包解压出来的share文件夹下的jar.gz包。
hadoop fs -chmod +x /tez/tez.tar.gz
hadoop fs -ls /tez
Found 1 items
-rwxr-xr-x   3 root supergroup   46254263 2020-07-04 14:09 /tez/tez.tar.gz

vim $HIVE_HOME/conf/tez-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
        <name>tez.lib.uris</name>
        <value>${fs.defaultFS}/tez/apache-tez-0.9.1-bin.tar.gz</value>
</property>
</configuration>

vim $HIVE_HOME/conf/hive-env.sh # 添加tez环境变量配置和依赖包环境变量配置，加入下面内容
export TEZ_HOME=/opt/module/tez    #是你的tez的解压目录
export TEZ_JARS=""
for jar in `ls $TEZ_HOME |grep jar`; do
    export TEZ_JARS=$TEZ_JARS:$TEZ_HOME/$jar
done
for jar in `ls $TEZ_HOME/lib`; do
    export TEZ_JARS=$TEZ_JARS:$TEZ_HOME/lib/$jar
done
# 注意本机的hadoop-lzo jar包位置
export HIVE_AUX_JARS_PATH=/opt/module/hadoop-2.7.2/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar

vim $HIVE_HOME/conf/hive-site.xml # 加入下面内容
<!-- 更改hive计算引擎 -->
<property>
    <name>hive.execution.engine</name>
    <value>tez</value>
</property>
<!-- 关闭元数据检查 -->
<property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
</property>

cp /opt/module/tez/*.jar $HIVE_HOME/lib #把tez的相关jar都复制到$HIVE_HOME/lib，不然hive可能找不到相关类
cp /opt/module/tez/lib/*.jar $HIVE_HOME/lib
hd resart
hive #启动hive
create table student(id int,name string);
hive (default)> insert into student values(1,"zhangsan");
select * from student;
1       zhangsan
vim $HADOOP_HOME/etc/hadoop/yarn-site.xml # 加入内容
<!-- 关掉虚拟内存检查，修改后一定要分发，并重新启动hadoop集群。-->
<property>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>false</value>
</property>
<property>
   <name>yarn.scheduler.minimum-allocation-mb</name>
   <value>2048</value>
   <description>default value is 1024</description>
</property>

xsync $HADOOP_HOME/etc/hadoop/yarn-site.xml
hd restart

hive #启动hive
create table student(id int,name string);
insert into student values(1,"zhangsan"); #出现类似问题表示成功
Query ID = root_20200704141246_8b43702f-e752-4df7-8c22-71426891f8f0
Total jobs = 1
Launching Job 1 out of 1
Status: Running (Executing on YARN cluster with App id application_1593842044632_0004)

----------------------------------------------------------------------------------------------
        VERTICES      MODE        STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED  
----------------------------------------------------------------------------------------------
Map 1 .......... container     SUCCEEDED      1          1        0        0       0       0  
----------------------------------------------------------------------------------------------
VERTICES: 01/01  [==========================>>] 100%  ELAPSED TIME: 4.92 s     
----------------------------------------------------------------------------------------------
Loading data to table default.student
OK
_col0	_col1
Time taken: 8.648 seconds

select * from student;
1       zhangsan

远程hive连接工具有datagrip(与idea同一家公司，推荐)和dbeaver，通过远程连接必须能正常启动hiveserver2。
```
