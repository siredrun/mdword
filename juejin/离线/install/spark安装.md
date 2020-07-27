# spark安装

注意：不能随意选择spark版本，要与其它相关的大数据组件兼容才行。

集群规划：跟随hive，hive安装在哪个节点spark就在哪个节点。

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Hive     | Hive   | √                 |                   |                   |

鉴于hive和spark的特殊原因，hive也得重新使用提供的已经编译的hive安装，当然也可以以开始就使用编译好的hive进行安装。

hive替换

```
cd /opt/software
mv /opt/module/hive /opt/module/hive_bak #原来的hive先备份一份
tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /opt/module/ #把提供已编译好的hive上传到linux
mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
vim /etc/profile
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile
cp /opt/module/hive_bak/conf/hive-site.xml /opt/module/hive/conf/
cp /opt/module/hive_bak/lib/mysql-connector-java-5.1.48.jar /opt/module/hive/lib/
启动metastore和hiveserver2，查看/tmp/admin/hive.log有无报错，能否正常访问hadoop:102


# 解决日志Jar包冲突，进入/opt/module/hive/lib目录和"Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V"
mv /opt/module/hive/lib/log4j-slf4j-impl-2.10.0.jar /opt/module/hive/lib/log4j-slf4j-impl-2.10.0.jar.bak
mv /opt/module/hive/lib/guava-19.0.jar /opt/module/hive/lib/guava-19.0.jar.bak
cp /opt/module/hd/share/hadoop/hdfs/lib/guava-27.0-jre.jar /opt/module/hive/lib
wget https://cdn.mysql.com/archives/mysql-connector-java-5.1/mysql-connector-java-5.1.48.tar.gz
tar -zxvf mysql-connector-java-5.1.48.tar.gz
cp mysql-connector-java-5.1.48/mysql-connector-java-5.1.48.jar /opt/module/hive/lib/
hive --service metastore
hiveserver2
# 启动后要等两分钟左右才访问连接，访问hadoop102:10002和使用datagrip（和idea同一家公司出品）能成功表示hive启动没问题。
```

安装

```
cd /opt/software
wget http://archive.apache.org/dist/spark/spark-2.4.5/spark-2.4.5.tgz
tar -zxvf spark-2.4.5.tgz
./spark-2.4.5/dev/make-distribution.sh --name without-hive --tgz -Pyarn -Phadoop-3.1 -Dhadoop.version=3.1.3 -Pparquet-provided -Porc-provided -Phadoop-provided #会生成spark-2.4.5-bin-without-hive.tgz

# 注意：Hive on Spark编译，截止2020-07-19，官方没有针对spark对hadoop3.1版本的编译tgz包，需要进行编译，但是编译会出各种问题，比较麻烦，不太建议使用，直接跳过上面步骤使用提供已经编译好spark-2.4.5-bin-without-hive.tgz即可。当然如果hadoop是2.7.x，可以下载http://archive.apache.org/dist/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz。
tar -zxvf /opt/software/spark-2.4.5-bin-without-hive.tgz -C /opt/module
mv /opt/module/spark-2.4.5-bin-without-hive/ /opt/module/spark
vim /etc/profile
#SPARK_HOME
export SPARK_HOME=/opt/module/spark
export PATH=$PATH:$SPARK_HOME/bin

xsync /etc/profile
source /etc/profile
mv /opt/module/spark/conf/spark-env.sh.template /opt/module/spark/conf/spark-env.sh
vim /opt/module/spark/conf/spark-env.sh # 添加
export SPARK_DIST_CLASSPATH=$(hadoop classpath)

# 连接sparkjar包到hive，如果hive中已存在则跳过
ln -s /opt/module/spark/jars/scala-library-2.11.12.jar /opt/module/hive/lib/scala-library-2.11.12.jar
ln -s /opt/module/spark/jars/spark-core_2.11-2.4.5.jar /opt/module/hive/lib/spark-core_2.11-2.4.5.jar
ln -s /opt/module/spark/jars/spark-network-common_2.11-2.4.5.jar /opt/module/hive/lib/spark-network-common_2.11-2.4.5.jar
vim $HIVE_HOME/conf/spark-defaults.conf # 添加如下内容
spark.master                                    yarn
spark.eventLog.enabled                          true
spark.eventLog.dir                              hdfs://hadoop102:8020/spark-history
spark.driver.memory                             2g
spark.executor.memory                           2g

hadoop fs -mkdir /spark-history # HDFS创建路径
hadoop fs -mkdir /spark-jars # 上传Spark依赖到HDFS
hadoop fs -put $SPARK_HOME/jars/* /spark-jars
vim $HIVE_HOME/conf/hive-site.xml # 加入修改
  <!--Spark依赖位置-->
  <property>
    <name>spark.yarn.jars</name>
    <value>hdfs://hadoop102:8020/spark-jars/*</value>
  </property>
  
  <!--Hive执行引擎-->
  <property>
    <name>hive.execution.engine</name>
    <value>spark</value>
  </property>

hive (default)> create external table student(id int, name string) location '/student';
# 通过insert测试效果，注意hive在这里一定要使用提供的重新编译过的hive安装包重新解压安装，不然容易报错。
hive (default)> insert into table student values(1,'abc');
Query ID = admin_20200719233906_da264107-0c51-4be1-ae4a-ad43110d3d1e
Total jobs = 1
Launching Job 1 out of 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Running with YARN Application = application_1595088857356_0003
Kill Command = /opt/module/hd/bin/yarn application -kill application_1595088857356_0003
Hive on Spark Session Web UI URL: http://hadoop102:41270

Query Hive on Spark job[0] stages: [0, 1]
Spark job[0] status = RUNNING
--------------------------------------------------------------------------------------
          STAGES   ATTEMPT        STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  
--------------------------------------------------------------------------------------
Stage-0 ........         0      FINISHED      1          1        0        0       0  
Stage-1 ........         0      FINISHED      1          1        0        0       0  
--------------------------------------------------------------------------------------
STAGES: 02/02    [==========================>>] 100%  ELAPSED TIME: 8.06 s     
--------------------------------------------------------------------------------------
Spark job[0] finished successfully in 8.07 second(s)
WARNING: Spark Job[0] Spent 19% (997 ms / 5319 ms) of task time in GC
Loading data to table default.student
OK
col1	col2
Time taken: 27.975 seconds
hive (default)> select * from student;
OK
student.id	student.name
1	abc
Time taken: 0.171 seconds, Fetched: 1 row(s)
```

## Yarn容量调度器队列配置

正常启动hive客户端不退访问http://hadoop103:8088/会看到State为RUNNING、Queue为default的任务。

![](https://user-gold-cdn.xitu.io/2020/7/20/17367dac77eb3eb0?w=1559&h=146&f=png&s=29264)

再开一个新的窗口执行下面命令会发现到“ INFO mapreduce.Job: Running job: job_1595088857356_0004”就不能继续往下执行了。

```
hadoop jar $HD_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar pi 1 1
Number of Maps  = 1
Samples per Map = 1
...
2020-07-20 00:13:02,951 INFO mapreduce.Job: The url to track the job: http://hadoop103:8088/proxy/application_1595088857356_0004/
2020-07-20 00:13:02,952 INFO mapreduce.Job: Running job: job_1595088857356_0004
```

再刷新http://hadoop103:8088/，application_1595088857356_0004就是刚刚执行的那个hadoop jar任务，注意观察会发现0004和0003任务的Queue都是default，yarn默认的队列长度是1，任务按先来后到执行，0003hive客户端在运行中，0004mapreduce就必须等待着，除非把hive客户端停掉：hive (default)> quit;。0004mapreduce才能执行打印执行。

![](https://user-gold-cdn.xitu.io/2020/7/20/17367e5bd176d741?w=1581&h=268&f=png&s=51970)

默认Yarn的配置下，容量调度器只有一条Default队列。在$HD_HOME/etc/hadoop/capacity-scheduler.xml中可以配置多条队列，**修改**以下属性，增加hive队列。

编译$HD_HOME/etc/hadoop/capacity-scheduler.xml修改下面内容

```xml
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>50</value>
    <description>Default queue target capacity.</description>
  </property>
```

shift+g到达某位加上

```xml
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
<value>50</value>
    <description>
      hive队列的容量为50%
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
<value>1</value>
    <description>
      一个用户最多能够获取该队列资源容量的比例
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
<value>80</value>
    <description>
      hive队列的最大容量
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
<value>*</value>
    <description>
      访问控制，控制谁可以将任务提交到该队列
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
<value>*</value>
    <description>
      访问控制，控制谁可以管理(包括提交和取消)该队列的任务
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
<value>*</value>
<description>
      访问控制，控制用户可以提交到该队列的任务的最大优先级
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
<value>-1</value>
    <description>
      hive队列中任务的最大生命时长
</description>
</property>
<property>
    <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
<value>-1</value>
    <description>
      default队列中任务的最大生命时长
</description>
</property>
```

分发并重启hd

```
xsync $HD_HOME/etc/hadoop/capacity-scheduler.xml
hd stop
hd sart
```

访问http://hadoop103:8088/cluster/scheduler，出现表示“Queue：hive”表示成功。

![](https://user-gold-cdn.xitu.io/2020/7/20/1736801da09e9108?w=1422&h=118&f=png&s=16408)

**设置hive客户端任务提交到hive队列**：为方便后续hive客户端的测试和shell脚本中的任务能同时执行，我们将hive客户端的测试任务提交到hive队列，让shell脚本中的任务使用默认值，提交到default队列。每次进入hive客户端时，执行以下命令即可。

```
hive (default)> set mapreduce.job.queuename=hive;
```

当然也可以在hive-site.xml直接配置写死，但容易出现queue阻塞，不推荐。