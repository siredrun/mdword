# 04ods搭建

ODS(Operation Data Store)原始数据层：存放原始数据， 直接加载原始日志、数据， 数据保原貌不做处理。这层的主要目的是把hdfs里面的数据放到hive里面。

```
============================回顾数据过程=============================
lg # 1.在hadoop102和hadoop103的/tmp/logs/生成日志（通过log-collector生成的jar包），格式app-日期.log，也可以结合ct集群时间更新和dt集群时间修改生成不同日期的日志。
f1 start # 2.把hadoop102和hadoop103的/tmp/logs/*.log日志采集过滤并放到kafka主题topic_start、topic_event里，要把日志过滤flume-interceptor生成的jar包放在flume的lib下。
f2 start # 3.启动两个消费者CG_Start、CG_Event分别消费kafka上的topic_start、topic_event里的日志数据，存到hdfs里面。
# 确保hadoop102里面有数据，通过lzo压缩的数据，后缀为lzo，如下
drwxr-xr-x   - root supergroup          0 2020-07-05 09:36 /origin_data/gmall/log/topic_event/2020-07-05
hadoop fs -ls /origin_data/gmall/log/topic_event/2020-07-05
-rw-r--r--   3 root supergroup     347040 2020-07-05 09:36 /origin_data/gmall/log/topic_event/2020-07-05/logevent-.1593912953492.lzo
===========================回顾数据完毕==============================
# 注意：有类似“hive (default)>”的就表示要在hive里面执行。
hive (default)> create database gmall;
hive (default)> use gmall;
======================启动日志表ods_start_log=======================
# 创建输入数据是lzo输出是text，外部表示支持json解析的分区表。
hive (gmall)> drop table if exists ods_start_log; # 如果存在就先删掉
hive (gmall)> CREATE EXTERNAL TABLE ods_start_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_start_log';

hadoop fs -ls /origin_data/gmall/log/topic_start/2020-07-05
-rw-r--r--   3 root supergroup      91344 2020-07-05 09:36 /origin_data/gmall/log/topic_start/2020-07-05/logstart-.1593912953491.lzo

hive (gmall)> load data inpath '/origin_data/gmall/log/topic_start/2020-07-05' into table gmall.ods_start_log partition(dt='2020-07-05');# 加载数据，请以自己hdf上面的数据为准注意：时间格式都配置成YYYY-MM-DD格式，这是Hive默认支持的时间格式。
hive (gmall)> select * from ods_start_log limit 2; # 验证存入的数据
OK
ods_start_log.line	ods_start_log.dt
{"action":"1","ar":"MX","ba":"Huawei","detail":"","en":"start","entry":"4","extend1":"","g":"5AIR60OU@gmail.com","hw":"750*1134","l":"pt","la":"19.5","ln":"-63.6","loading_time":"2","md":"Huawei-1","mid":"7","nw":"3G","open_ad_type":"2","os":"8.1.9","sr":"P","sv":"V2.6.6","t":"1593906311695","uid":"7","vc":"4","vn":"1.2.8"}	2020-07-05
{"action":"1","ar":"MX","ba":"HTC","detail":"","en":"start","entry":"5","extend1":"","g":"A71RTO6Q@gmail.com","hw":"1080*1920","l":"es","la":"5.9","ln":"-67.3","loading_time":"3","md":"HTC-8","mid":"10","nw":"3G","open_ad_type":"1","os":"8.2.2","sr":"B","sv":"V2.3.3","t":"1593840968917","uid":"10","vc":"13","vn":"1.2.7"}	2020-07-05
Time taken: 0.151 seconds, Fetched: 2 row(s)

hadoop jar $HADOOP_HOME/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/gmall/ods/ods_start_log/dt=2020-07-05 # 为lzo压缩文件创建索引
hadoop fs -ls /warehouse/gmall/ods/ods_start_log/dt=2020-07-05
Found 2 items
-rwxr-xr-x   3 root supergroup      91344 2020-07-05 09:36 /warehouse/gmall/ods/ods_start_log/dt=2020-07-05/logstart-.1593912953491.lzo
-rw-r--r--   3 root supergroup         16 2020-07-05 10:17 /warehouse/gmall/ods/ods_start_log/dt=2020-07-05/logstart-.1593912953491.lzo.index

======================事件日志表ods_event_log=======================
# 创建输入数据是lzo输出是text，外部表示支持json解析的分区表。
hive (gmall)> drop table if exists ods_event_log; # 如果存在就先删掉
hive (gmall)> CREATE EXTERNAL TABLE ods_event_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_event_log';

hadoop fs -ls /origin_data/gmall/log/topic_event/2020-07-05
-rw-r--r--   3 root supergroup     347040 2020-07-05 09:36 /origin_data/gmall/log/topic_event/2020-07-05/logevent-.1593912953492.lzo

hive (gmall)> load data inpath '/origin_data/gmall/log/topic_event/2020-07-05' into table gmall.ods_event_log partition(dt='2020-07-05'); # 加载数据，请以自己hdf上面的数据为准注意：时间格式都配置成YYYY-MM-DD格式，这是Hive默认支持的时间格式。
hive (gmall)> select * from ods_event_log limit 2; # 验证存入的数据
OK
ods_event_log.line	ods_event_log.dt
1593912554584|{"cm":{"ln":"-39.7","sv":"V2.4.3","os":"8.0.9","g":"R41A21I6@gmail.com","mid":"2","nw":"3G","l":"en","vc":"2","hw":"750*1134","ar":"MX","uid":"2","t":"1593830748998","la":"22.3","md":"HTC-1","vn":"1.2.0","ba":"HTC","sr":"B"},"ap":"app","et":[{"ett":"1593876122814","en":"newsdetail","kv":{"entry":"3","goodsid":"0","news_staytime":"0","loading_time":"3","action":"1","showtype":"4","category":"7","type1":"542"}},{"ett":"1593867654077","en":"loading","kv":{"extend2":"","loading_time":"24","action":"3","extend1":"","type":"3","type1":"102","loading_way":"2"}},{"ett":"1593870529063","en":"ad","kv":{"entry":"1","show_style":"0","action":"5","detail":"325","source":"4","behavior":"1","content":"1","newstype":"0"}},{"ett":"1593848965351","en":"active_foreground","kv":{"access":"","push_id":"3"}},{"ett":"1593895712213","en":"error","kv":{"errorDetail":"at cn.lift.dfdfdf.control.CommandUtil.getInfo(CommandUtil.java:67)\\n at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\\n at java.lang.reflect.Method.invoke(Method.java:606)\\n","errorBrief":"at cn.lift.dfdf.web.AbstractBaseController.validInbound(AbstractBaseController.java:72)"}},{"ett":"1593841455609","en":"praise","kv":{"target_id":8,"id":6,"type":3,"add_time":"1593857205264","userid":8}}]}	2020-07-05
1593912554642|{"cm":{"ln":"-56.7","sv":"V2.3.3","os":"8.0.4","g":"2OYHT5C2@gmail.com","mid":"4","nw":"4G","l":"es","vc":"12","hw":"640*960","ar":"MX","uid":"4","t":"1593897609955","la":"19.7","md":"Huawei-17","vn":"1.1.1","ba":"Huawei","sr":"M"},"ap":"app","et":[{"ett":"1593885876607","en":"newsdetail","kv":{"entry":"3","goodsid":"0","news_staytime":"0","loading_time":"0","action":"2","showtype":"5","category":"80","type1":"102"}},{"ett":"1593833837489","en":"loading","kv":{"extend2":"","loading_time":"42","action":"1","extend1":"","type":"3","type1":"433","loading_way":"1"}},{"ett":"1593859829339","en":"ad","kv":{"entry":"1","show_style":"1","action":"5","detail":"201","source":"3","behavior":"2","content":"1","newstype":"8"}},{"ett":"1593860697509","en":"active_foreground","kv":{"access":"","push_id":"3"}},{"ett":"1593825504072","en":"active_background","kv":{"active_source":"2"}},{"ett":"1593864674685","en":"error","kv":{"errorDetail":"java.lang.NullPointerException\\n    at cn.lift.appIn.web.AbstractBaseController.validInbound(AbstractBaseController.java:72)\\n at cn.lift.dfdf.web.AbstractBaseController.validInbound","errorBrief":"at cn.lift.appIn.control.CommandUtil.getInfo(CommandUtil.java:67)"}},{"ett":"1593907753501","en":"comment","kv":{"p_comment_id":1,"addtime":"1593820528368","praise_count":690,"other_id":6,"comment_id":6,"reply_count":80,"userid":5,"content":"漆掂扣特沂弹御"}},{"ett":"1593865659284","en":"praise","kv":{"target_id":6,"id":8,"type":3,"add_time":"1593844949877","userid":9}}]}	2020-07-05
Time taken: 0.188 seconds, Fetched: 2 row(s)

hadoop jar $HADOOP_HOME/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/gmall/ods/ods_event_log/dt=2020-07-05 # 为lzo压缩文件创建索引。
hadoop fs -ls /warehouse/gmall/ods/ods_event_log/dt=2020-07-05
Found 2 items
-rwxr-xr-x   3 root supergroup     347040 2020-07-05 09:36 /warehouse/gmall/ods/ods_event_log/dt=2020-07-05/logevent-.1593912953492.lzo
-rw-r--r--   3 root supergroup         48 2020-07-05 10:31 /warehouse/gmall/ods/ods_event_log/dt=2020-07-05/logevent-.1593912953492.lzo.index

==========================最后测试=================================
关于上面启动日志表和事务日志操作比较类似，所以专门使用一个脚本来ods_log启用数据。
# 先启动hd zk kf
dt '2020-05-20'
------------同步hadoop102时间--------------
Wed May 20 00:00:00 CST 2020
------------同步hadoop103时间--------------
Wed May 20 00:00:00 CST 2020
------------同步hadoop104时间--------------
Wed May 20 00:00:00 CST 2020
xcall date +%F
要执行的命令是:date +%F
-----------------------hadoop102---------------------
hadoop102 date +%F
2020-05-20
-----------------------hadoop103---------------------
hadoop103 date +%F
2020-05-20
-----------------------hadoop104---------------------
hadoop104 date +%F
2020-05-20

lg
xcall ls /tmp/logs/ | grep 'app-2020-05-20'
app-2020-05-20.log
app-2020-05-20.log
ls: cannot access /tmp/logs/: No such file or directory
f1 start # 启动后记得通过kafka tools看看topic_start和topic_event两个主题的message total数量变化
f2 start # 查看hdfs有无生成2020-05-20的数据
hadoop fs -ls /origin_data/gmall/log/topic_start/2020-05-20
-rw-r--r--   3 root supergroup      96812 2020-05-20 00:03 /origin_data/gmall/log/topic_start/2020-05-20/logstart-.1589904154823.lzo

hadoop fs -ls /origin_data/gmall/log/topic_event/2020-05-20
-rw-r--r--   3 root supergroup     328234 2020-05-20 00:03 /origin_data/gmall/log/topic_event/2020-05-20/logevent-.1589904154823.lzo

hiveserver2 #启动hive，记得运行mysql和keepalived（mysql高可用），保证虚拟ip可用
ods_log 2020-05-20 # 注意因为hive sql执行的特殊性，此命令必须在有安装hive的节点上面执行。
===日志日期为2020-05-20===
......
OK
Time taken: 1.349 seconds
Loading data to table gmall.ods_start_log partition (dt=2020-05-20)
OK
Time taken: 2.122 seconds
Loading data to table gmall.ods_event_log partition (dt=2020-05-20)
OK
Time taken: 1.319 seconds
......

hadoop fs -ls /warehouse/gmall/ods/ods_event_log/dt=2020-05-20
Found 2 items
-rwxr-xr-x   3 root supergroup     328234 2020-05-20 00:03 /warehouse/gmall/ods/ods_event_log/dt=2020-05-20/logevent-.1589904154823.lzo
-rw-r--r--   3 root supergroup         48 2020-05-20 01:23 /warehouse/gmall/ods/ods_event_log/dt=2020-05-20/logevent-.1589904154823.lzo.index

hadoop fs -ls /warehouse/gmall/ods/ods_start_log/dt=2020-05-20
Found 2 items
-rwxr-xr-x   3 root supergroup      96812 2020-05-20 00:03 /warehouse/gmall/ods/ods_start_log/dt=2020-05-20/logstart-.1589904154823.lzo
-rw-r--r--   3 root supergroup         16 2020-05-20 01:23 /warehouse/gmall/ods/ods_start_log/dt=2020-05-20/logstart-.1589904154823.lzo.index

ct # 注意把集群时间更新回来。
-----------hadoop102---------------
 8 Jul 17:23:08 ntpdate[7834]: step time server 120.25.115.20 offset -1.457613 sec
-----------hadoop103---------------
 8 Jul 17:23:14 ntpdate[5282]: step time server 120.25.115.20 offset 4290746.782269 sec
-----------hadoop104---------------
 8 Jul 17:23:19 ntpdate[3694]: step time server 120.25.115.20 offset -1.768443 sec

xcall date +%F
要执行的命令是:date +%F
-----------------------hadoop102---------------------
hadoop102 date +%F
2020-07-08
-----------------------hadoop103---------------------
hadoop103 date +%F
2020-07-08
-----------------------hadoop104---------------------
hadoop104 date +%F
2020-07-08
```
