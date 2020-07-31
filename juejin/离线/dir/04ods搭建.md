这一层的主要目的是把放在hdfs里面的用户行为数据和业务数据根据日期分区放在hive里面

# 数据生成回顾

注意：**把用户行为数据和业务数据都生成两天的数据，如2020-03-10、2020-03-11两天。**

用户行为数据

```
lg # 1.在hadoop102和hadoop103的/tmp/logs/生成日志（通过log-collector生成的jar包），格式app-日期.log，也可以结合ct集群时间更新和dt集群时间修改生成不同日期的日志。
f1 start # 2.把hadoop102和hadoop103的/tmp/logs/*.log日志采集过滤并放到kafka主题topic_start、topic_event里，要把日志过滤flume-interceptor生成的jar包放在flume的lib下。
f2 start # 3.启动两个消费者分别消费kafka上的topic_start、topic_event里的日志数据，存到hdfs里面。
# 确保hadoop102里面有数据，通过lzo压缩的数据，后缀为lzo，如下
drwxr-xr-x   - root supergroup          0 2020-07-05 09:36 /origin_data/gmall/log/topic_event/2020-07-05
hadoop fs -ls /origin_data/gmall/log/topic_event/2020-07-05
-rw-r--r--   3 root supergroup     347040 2020-07-05 09:36 /origin_data/gmall/log/topic_event/2020-07-05/logevent-.1593912953492.lzo
```

业务数据

```
mkdir -p /opt/module/db_log/
# 把gmall-mock-db.jar上传到hadoop102的/opt/module/db_log下，有做了的可以省略
# 注意mock.date可以设置为2020-03-10和2020-03-11，分别生成对应日期的业务数据。mock.clear是否重置，0否，1是。
vim /opt/module/db_log/application.properties
......
#业务日期
mock.date=2020-03-10
#是否重置
mock.clear=0
......

jar -jar /opt/module/db_log/gmall-mock-db.jar
......
tarted GmallMockDbApplication in 3.641 seconds (JVM running for 4.123)
--------开始生成数据--------
--------开始生成用户数据--------
共生成50名用户
......

vim /opt/module/db_log/application.properties #修改mock-date
mock.date=2020-03-10

jar -jar /opt/module/db_log/gmall-mock-db.jar
mysql -uroot -p1234
mysql> use gmall;
mysql> select * from user_info where create_time = '2020-03-10' limit 1;
mysql> select * from user_info where create_time = '2020-03-11' limit 1;
gmall_mysql_to_hdfs first 2020-03-10 # 初次导入 导入时间会相对比较长，大概20分钟左右
gmall_mysql_to_hdfs all 2020-03-11 # 每日导入

hadoop fs -ls /origin_data/gmall/db/activity_info/2020-03-10
Found 3 items
-rw-r--r--   1 admin supergroup          0 2020-07-18 21:58 .../2020-03-10/_SUCCESS
-rw-r--r--   1 admin supergroup        135 2020-07-18 21:58 .../2020-03-10/part-m-00000.lzo
-rw-r--r--   1 admin supergroup          8 2020-07-18 21:58 .../2020-03-10/part-m-00000.lzo.index

```

# 用户行为数据

ODS(Operation Data Store)原始数据层：

1）保持数据原貌不做任何修改，起到备份数据的作用。
2）数据采用LZO压缩，减少磁盘存储空间。100G数据可以压缩到10G以内。
3）创建分区表，防止后续的全表扫描，在企业开发中大量使用分区表。
4）创建外部表。在企业开发中，除了自己用的临时表，创建内部表外，绝大多数场景都是创建外部表。

```
# 启动metastore和hiveserver2
# 注意：hdfs/origin_data/gmall/log/topic_event|start至少要有两天的数据，有类似“hive (default)>”的就表示要在hive里面执行。
hive (default)> create database gmall;
hive (default)> use gmall;
```

## 启动日志表ods_start_log

```
# 建表语句，创建输入数据是lzo输出是text，外部表示支持json解析的分区表。建输入数据是lzo输出是text，支持json解析的分区表。
hive (gmall)> drop table if exists ods_start_log;
CREATE EXTERNAL TABLE ods_start_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_start_log';

# 加载数据 注意：时间格式都配置成YYYY-MM-DD格式，这是Hive默认支持的时间格式
hive (gmall)> load data inpath '/origin_data/gmall/log/topic_start/2020-03-10' into table gmall.ods_start_log partition(dt='2020-03-10');
hive (gmall)> select * from ods_start_log limit 1; # 验证存入的数据

# 为lzo压缩文件创建索引
hadoop jar $HD_HOME/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/gmall/ods/ods_start_log/dt=2020-03-10 

```

## 创建事件日志表ods_event_log

```
# 创建输入数据是lzo输出是text，外部表示支持json解析的分区表。
hive (gmall)> drop table if exists ods_event_log; # 如果存在就先删掉
hive (gmall)> CREATE EXTERNAL TABLE ods_event_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_event_log';

# 加载数据，请以自己hdf上面的数据为准注意：时间格式都配置成YYYY-MM-DD格式，这是Hive默认支持的时间格式。
hive (gmall)> load data inpath '/origin_data/gmall/log/topic_event/2020-03-10' into table gmall.ods_event_log partition(dt='2020-03-10'); 
hive (gmall)> select * from ods_event_log limit 2; # 验证存入的数据

# 为lzo压缩文件创建索引。
hadoop jar $HD_HOME/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/gmall/ods/ods_event_log/dt=2020-03-10 
```

## 总结

根据上面发现步骤就三步：创建分区表=》加载数据=》建立索引，除了时间和表名不一样而已。可以通过ODS层加载数据脚本hdfs_to_ods_log加日期去建立。

# 业务数据

```

```

