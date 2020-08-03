# ods搭建

这一层的主要目的是把放在hdfs里面的用户行为数据和业务数据根据日期分区放在hive里面。

ODS(Operation Data Store)原始数据层：

1）保持数据原貌不做任何修改，起到备份数据的作用。
2）数据采用LZO压缩，减少磁盘存储空间。100G数据可以压缩到10G以内。
3）创建分区表，防止后续的全表扫描，在企业开发中大量使用分区表。
4）创建外部表。在企业开发中，除了自己用的临时表，创建内部表外，绝大多数场景都是创建外部表。

## 数据生成回顾

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

## 用户行为数据

用户行为数据导入hive步骤就三步：创建分区表=》加载数据=》建立索引，分区表手动创建，加载数据和建立索引可以通过ODS层加载数据脚本hdfs_to_ods_log加日期去进行。

```
# 打开两个窗口分别启动metastore和hiveserver2
hive --service metastore
hiveserver2
# 注意：hdfs/origin_data/gmall/log/topic_event|start至少要有两天的数据，有类似“hive (default)>”的就表示要在hive里面执行。
hive (default)> create database gmall;
hive (default)> use gmall;
# 启动日志表ods_start_log：创建输入数据是lzo输出是text，外部表示支持json解析的分区表。
hive (gmall)> drop table if exists ods_start_log;
CREATE EXTERNAL TABLE ods_start_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_start_log';
# 创建事件日志表ods_event_log：创建输入数据是lzo输出是text，外部表示支持json解析的分区表。
hive (gmall)> drop table if exists ods_event_log;
CREATE EXTERNAL TABLE ods_event_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_event_log';
# 导入数据
hdfs_to_ods_log 2020-03-10
hdfs_to_ods_log 2020-03-11
# 验证
hive (gmall)> select * from ods_start_log where dt = '2020-03-10' limit 2;
hive (gmall)> select * from ods_start_log where dt = '2020-03-11' limit 2;
hive (gmall)> select * from ods_event_log where dt = '2020-03-10' limit 2;
hive (gmall)> select * from ods_event_log where dt = '2020-03-11' limit 2;
```

## 业务数据

进入hive选择gamll数据库，执行脚本与配置文件里面的ods_sql业务数据23张表的建表sql。

```
hive (gmall)> show tables;#查看gmall里面表的数量是否为25，用户行为数据2张表+业务数据23张表
# hdfs_to_ods_db业务数据加载脚本ODS层,执行方式：hdfs_to_ods_db first（第一次）或all（第n[n>1]次） 次
hdfs_to_ods_db first 2020-03-10
hdfs_to_ods_db all 2020-03-11
# 验证数据
hive (gmall)> select * from ods_order_info limit 2;
hive (gmall)> select * from ods_order_detail limit 2;
```

# dwd搭建

1）对用户行为数据解析。
2）对核心数据进行判空过滤。
3）对业务数据采用**维度模型**重新建模，即**维度退化**。
简单来说，就是对ODS层数据进行清洗(去除空值，脏数据，超过极限范围的数据)、维度退化、脱敏等。

函数介绍

**get_json_object函数**：解析json字符串，get_json_objec(json字符串, 解析格式)
**concat函数**：在连接字符串的时候，只要其中一个是NULL，那么将返回NULL
**concat_ws函数**：在连接字符串的时候，只要有一个字符串不是NULL，就不会返回NULL。concat_ws函数需要指定分隔符。
**STR_TO_MAP函数**：语法STR_TO_MAP(VARCHAR text, VARCHAR listDelimiter, VARCHAR keyValueDelimiter)，使用listDelimiter将text分隔成K-V对，然后使用keyValueDelimiter分隔每个K-V对，组装成MAP返回。默认listDelimiter为（ ，），keyValueDelimiter为（=）。

案例如下：

```sql
-- 取出第一个json对象
select get_json_object('[{"name":"大郎","sex":"男","age":"25"},{"name":"西门庆","sex":"男","age":"47"}]','$[0]');
{"name":"大郎","sex":"男","age":"25"}

-- 取出第一个json的age字段的值
SELECT get_json_object('[{"name":"大郎","sex":"男","age":"25"},{"name":"西门庆","sex":"男","age":"47"}]',"$[0].age");
25

select concat('a','b');
--ab

select concat('a','b',null);
--NULL
select concat_ws('-','a','b');
--a-b
select concat_ws('-','a','b',null);
--a-b

select concat_ws('','a','b',null);
--ab

select str_to_map('1001=2020-03-10,1002=2020-03-10',  ','  ,  '=')

--{"1001":"2020-03-10","1002":"2020-03-10"}
```

<font color='red'>注意</font>：如出现”FAILED: SemanticException Failed to get a spark session: org.apache.hadoop.hive.ql.metadata.HiveException: Failed to create Spark client for Spark session“，解决方式有两种：

1.重启hiveserver2和metastore；

2.在hive客户端设置加大client连接时间间隔

```sql
set hive.spark.client.server.connect.timeout=900000; 
```

3.在hive配置文件$HIVE_HOME/conf/hive-site.xml加入下面内容重启hive。

```xml
 <property>
    <name>hive.spark.client.server.connect.timeout</name>
    <value>900000</value>
  </property>
```

## 用户行为数据

开始：进入hive选择gamll数据库，执行脚本与配置文件里面的dwd_sql数据建表hive sql：生成26张表（包含业务数据和用户行为数据的表）

```
hive (gmall)> show tables;#查看gmall里面表的数量是否为53，ods层25张加dwd层28张等于53张
ods_to_dwd_start_log 2020-03-10 # 启动表加载数据，错误日志查看/tmp/admin/hive.log，执行时间大概30秒，请耐心等待。
ods_to_dwd_start_log 2020-03-11
# 注意执行的时候必须把hive客户端关掉，hive已做特殊处理但最大只能支持2个队列，hive客户端占1，执行ods_to_dwd_start_log占2个队列，这两个队列必须都执行完毕才算成功，如果hive客户端开着会因为一直不成功而挂掉，异常“: Failed to create Spark client for Spark session fe”。打印类似下面内容表示成功。
--------------------------------------------------------------------------------------
          STAGES   ATTEMPT        STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  
--------------------------------------------------------------------------------------
Stage-0 ........         0      FINISHED      1          1        0        0       0  
Stage-1 ........         0      FINISHED      1          1        0        0       0  
--------------------------------------------------------------------------------------
STAGES: 02/02    [==========================>>] 100%  ELAPSED TIME: 6.10 s     
--------------------------------------------------------------------------------------

# 将maven项目生成的hivefunction-1.0-SNAPSHOT.jar上传到HDFS上的/user/hive/jars路径下，Hadoop3版本可直接访问hadoop102:9870建立对应文件夹并上传。
hadoop fs -ls /user/hive/jars/
-rw-r--r--   1 admin supergroup       5205 2020-08-01 23:05 /user/hive/jars/hivefunction-1.0-SNAPSHOT.jar

# 将永久函数与开发好的java class关联
hive (gmall)> create function base_analizer as 'com.demo.udf.BaseFieldUDF' using jar 'hdfs://hadoop102:8020/user/hive/jars/hivefunction-1.0-SNAPSHOT.jar';
create function flat_analizer as 'com.demo.udtf.EventJsonUDTF' using jar 'hdfs://hadoop102:8020/user/hive/jars/hivefunction-1.0-SNAPSHOT.jar';

hive (gmall)> show functions;
...
gmall.base_analizer
gmall.flat_analizer

ods_to_dwd_base_log 2020-03-10 # 解析事件日志基础明细表，企业开发中一般在每日凌晨30分~1点
ods_to_dwd_base_log 2020-03-11 # 出现类似”STAGES: 02/02 ... 100%  ELAPSED TIME: 14.14 s“表示成功

hive (gmall)> select * from dwd_base_event_log where dt = '2020-03-10' limit 2;
select * from dwd_base_event_log where dt = '2020-03-11' limit 2;
# dwd_events_log事件日志具体表脚本DWD层，加载10张事件具体表的数据。
dwd_events_log 2020-03-10 # 会出现10个类似“STAGES: 02/02 ... 100%  ELAPSED TIME: 6.10 s”的打印
dwd_events_log 2020-03-11
hive (gmall)> select * from dwd_display_log where dt = '2020-03-10' limit 2;
select * from dwd_display_log where dt = '2020-03-11' limit 2;
```

事件具体详细表就是根据事件日志基础明细表ods_to_dwd_base_log的event_name事件名称把event_json事件json放到对应的明细表中

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d71bc7eca64b45e1bb7224efb2c910ff~tplv-k3u1fbpfcp-zoom-1.image)

## 业务数据

下面是业务数据事实表的简单概述：

订单明细事实表（事务型事实表）

|              | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| ------------ | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **订单详情** | √        |          | √        | √        |            |          |          | 件数/金额  |

支付事实表（事务型事实表）

|          | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **支付** | √        |          | √        |          |            |          |          | 金额       |

退款事实表（事务型事实表）：把ODS层ods_order_refund_info表数据导入到DWD层退款事实表，在导入过程中可以做适当的清洗。

|          | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **退款** | √        | √        |          | √        |            |          |          | 件数/金额  |

评价事实表（事务型事实表）：把ODS层ods_comment_info表数据导入到DWD层评价事实表，在导入过程中可以做适当的清洗。

|          | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **评价** | √        | √        |          | √        |            |          |          | 个数       |

加购事实表（周期型快照事实表，每日快照）：由于购物车的数量是会发生变化，所以导增量不合适。每天做一次快照，导入的数据是全量，区别于事务型事实表是每天导入新增。周期型快照事实表劣势：存储的数据量会比较大。解决方案：周期型快照事实表存储的数据比较讲究时效性，时间太久了的意义不大，可以删除以前的数据。

|          | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **加购** | √        | √        |          | √        |            |          |          | 件数/金额  |

收藏事实表（周期型快照事实表，每日快照）：收藏的标记，是否取消，会发生变化，做增量不合适。每天做一次快照，导入的数据是全量，区别于事务型事实表是每天导入新增。

|          | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **收藏** | √        | √        |          | √        |            |          |          | 个数       |

优惠券领用事实表（累积型快照事实表）：优惠卷的生命周期：领取优惠卷-》用优惠卷下单-》优惠卷参与支付，累积型快照事实表使用：统计优惠卷领取次数、优惠卷下单次数、优惠卷参与支付次数

|                | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **优惠券领用** | √        | √        |          |          | √          |          |          | 个数       |

订单事实表（累积型快照事实表）：订单生命周期：创建时间=》支付时间=》取消时间=》完成时间=》退款时间=》退款完成时间。由于ODS层订单表只有创建时间和操作时间两个状态，不能表达所有时间含义，所以需要关联订单状态表。订单事实表里面增加了活动id，所以需要关联活动订单表。

|          | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **编码** | **度量值** |
| -------- | -------- | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- |
| **订单** | √        | √        | √        |          |            | √        |          | 件数/金额  |

开始操作

```
# 时间维度表（特殊），创建临时表，非列式存储
# 将数据导入临时表，先把date_info.txt文件上传到hadoop102的/opt/module/db_log/路径。
hive (gmall)> load data local inpath '/opt/module/db_log/date_info.txt' into table dwd_dim_date_info_tmp;
hive (gmall)> insert overwrite table dwd_dim_date_info select * from dwd_dim_date_info_tmp; # 将数据导入正式表
hive (gmall)> select * from dwd_dim_date_info_tmp limit 730,1; # 验证总数731
hive (gmall)> select * from dwd_dim_date_info limit 730,1; # 验证总数731

# 用户维度表（拉链表）
# 步骤0：初始化拉链表（首次独立执行） dwd_dim_user_info_his初始化用户维度拉链表脚本DWD层，这里有点特殊，只执行一个日期
dwd_dim_user_info_his 2020-03-10 # 两个日期选择日期靠前的那一个
hive (gmall)> select * from dwd_dim_user_info_his where start_date = '2020-03-10';
# 步骤1：制作当日变动数据（包括新增，修改）每日执行。这一步略过，因为已经执行过，没执行过的可以重新执行以下。
# gmall_mysql_to_hdfs业务数据sqoop导入HDFS：gmall_mysql_to_hdfs all 日期；hdfs_to_ods_log用户行为数据加载脚本ODS层：hdfs_to_ods_log all 日期。如果制作当日请执行下面语句：
gmall_mysql_to_hdfs all yyyy-MM-dd(当日日期)
hdfs_to_ods_log all yyyy-MM-dd(当日日期)
# 步骤2：先合并变动信息，再追加新增信息，插入到临时表中。有个导入脚本在
# 步骤3：把临时表覆盖给拉链表 步骤2的导入脚本和步骤3的sql请看ods_to_dwd_db的sql3
# ods_to_dwd_db业务数据脚本DWD层包含业务表16张表的数据导入（时间维度表和用户维度表的临时拉链表已经包含）
ods_to_dwd_db first 2020-03-10 # 第一次使用first，后面使用all
ods_to_dwd_db all 2020-03-11

show tables like 'dwd_fact*';--事实表，8张
show tables like 'dwd_dim*';--维度表，8张，包含时间维度临时表和用户维度临时拉链表
hive (gmall)> select * from dwd_dim_user_info_his where start_date = '2020-03-11';#再执行会看到2020-03-11的数据
```

# dws搭建

将DWD层数据按天轻度聚合！每天一个分区，分区表，按照日期进行分区。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9ea3c0f97484b1ebdf55ab7952196f8~tplv-k3u1fbpfcp-zoom-1.image)



## 业务术语

1）用户
用户以设备为判断标准，在移动统计中，每个独立设备认为是一个独立用户。Android系统根据IMEI号，IOS系统根据OpenUDID来标识一个独立用户，每部手机一个用户。
2）新增用户
首次联网使用应用的用户。如果一个用户首次打开某APP，那这个用户定义为新增用户；卸载再安装的设备，不会被算作一次新增。新增用户包括日新增用户、周新增用户、月新增用户。
3）活跃用户
打开应用的用户即为活跃用户，不考虑用户的使用情况。每天一台设备打开多次会被计为一个活跃用户。
4）周（月）活跃用户
某个自然周（月）内启动过应用的用户，该周（月）内的多次启动只记一个活跃用户。
5）月活跃率
月活跃用户与截止到该月累计的用户总和之间的比例。
6）沉默用户
用户仅在安装当天（次日）启动一次，后续时间无再启动行为。该指标可以反映新增用户质量和用户与APP的匹配程度。
7）版本分布
不同版本的周内各天新增用户数，活跃用户数和启动次数。利于判断APP各个版本之间的优劣和用户行为习惯。
8）本周回流用户
上周未启动过应用，本周启动了应用的用户。
9）连续n周活跃用户
连续n周，每周至少启动一次。
10）忠诚用户
连续活跃5周以上的用户
11）连续活跃用户
连续2周及以上活跃的用户
12）近期流失用户
连续n（2<= n <= 4/即2-4周内）周没有启动应用的用户。（第n+1周没有启动过）
13）留存用户
某段时间内的新增用户，经过一段时间后，仍然使用应用的被认作是留存用户；这部分用户占当时新增用户的比例即是留存率。
例如，5月份新增用户200，这200人在6月份启动过应用的有100人，7月份启动过应用的有80人，8月份启动过应用的有50人；则5月份新增用户一个月后的留存率是50%，二个月后的留存率是40%，三个月后的留存率是25%。
14）用户新鲜度
每天启动应用的新老用户比例，即新增用户数占活跃用户数的比例。
15）单次使用时长
每次启动使用的时间长度。
16）日使用时长
累计一天内的使用时间长度。
17）启动次数计算标准
IOS平台应用退到后台就算一次独立的启动；Android平台我们规定，两次启动之间的间隔小于30秒，被计算一次启动。用户在使用过程中，若因收发短信或接电话等退出应用30秒又再次返回应用中，那这两次行为应该是延续而非独立的，所以可以被算作一次使用行为，即一次启动。业内大多使用30秒这个标准，但用户还是可以自定义此时间间隔。

## 系统函数

**collect_set函数**：在hive里执行

```sql
--创建原数据表
drop table if exists stud;
create table stud (name string, area string, course string, score int);
--向原数据表中插入数据
insert into table stud values('zhang3','bj','math',88);
insert into table stud values('li4','bj','math',99);
insert into table stud values('wang5','sh','chinese',92);
insert into table stud values('zhao6','sh','chinese',54);
insert into table stud values('tian7','bj','chinese',91);
--查询表中数据
select * from stud;
stud.name	stud.area	stud.course	stud.score
zhang3	bj	math	88
li4	bj	math	99
wang5	sh	chinese	92
zhao6	sh	chinese	54
tian7	bj	chinese	91
--把同一分组的不同行的数据聚合成一个集合
select course, collect_set(area), avg(score) from stud group by course;
chinese ["sh","bj"]     79.0
math    ["bj"]  93.5
--用下标可以取某一个
select course, collect_set(area)[0], avg(score) from stud group by course;
chinese sh      79.0
math    bj      93.5
```

**nvl函数**：NVL（表达式1，表达式2），如果表达式1为空值，NVL返回值为表达式2的值，否则返回表达式1的值。 该函数的目的是把一个空值（null）转换成一个实际的值。其表达式的值可以是数字型、字符型和日期型。但是表达式1和表达式2的数据类型必须为同一个类型。
**日期处理函数：在hive里执行

```sql
--date_format（根据格式整理日期）
select date_format('2020-03-10','yyyy-MM');
--2020-03
--date_add函数（加减日期）
select date_add('2020-03-10',-1);
--2020-03-09
select date_add('2020-03-10',1);
--2020-03-11
--next_day函数取当前天的下一个周一
select next_day('2020-03-12','MO');
--2020-03-16
--说明：星期一到星期日的英文（Monday，Tuesday、Wednesday、Thursday、Friday、Saturday、Sunday）
--取当前周的周一
select date_add(next_day('2020-03-12','MO'),-7);
--2020-03-11
--last_day函数（求当月最后一天日期）
select last_day('2020-03-10');
--2020-03-31
```

## 开始

DWS层业务：DWS层的宽表字段，是站在不同维度的视角去看事实表。重点关注事实表的度量值。dws层就五个任务，只有每日设备行为是用户行为，其它都是业务，如下：

```
每日设备行为dws_uv_detail_daycount：每日设备行为，主要按照设备id统计。
每日会员行为dws_user_action_daycount：注意：如果是23点59下单，支付日期跨天。需要从订单详情里面取出支付时间是今天，订单时间是昨天或者今天的订单。
每日商品行为dws_sku_action_daycount
每日活动统计dws_activity_info_daycount
每日地区统计dws_area_stats_daycount
```

sql重点分析：其它的字段查询也类似，下面是典型。

```sql
--每日会员行为之登录次数
	select
        user_id,
        count(*) login_count
    from dwd_start_log
    where dt='2020-03-10'
    and user_id is not null
    group by user_id
--每日会员行为之加入购物车次数
    select
        user_id,
        count(*) cart_count
    from dwd_fact_cart_info
    where dt='2020-03-10'
and date_format(create_time,'yyyy-MM-dd')='2020-03-10'
    group by user_id
--每日会员行为之下单次数、下单金额
    select
        user_id,
        count(*) order_count,
        sum(final_total_amount) order_amount
    from dwd_fact_order_info
    where dt='2020-03-10'
    group by user_id
--每日会员行为之支付次数、支付金额
    select
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from dwd_fact_payment_info
    where dt='2020-03-10'
    group by user_id
--每日会员行为之下单明细统计
--下单明细统计案例/json数组：[{"sku_id":"4","sku_num":1,"order_count":1,"order_amount":1442}]
    select
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'order_amount',order_amount)) order_stats
    from
    (
        select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(total_amount) as decimal(20,2)) order_amount
        from dwd_fact_order_detail
        where dt='2020-03-10'
        group by user_id,sku_id
    )tmp
    group by user_id
--每日商品行为之被下单次数、被下单件数、被下单金额
    select
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(total_amount) order_amount
    from dwd_fact_order_detail
    where dt='2020-03-10'
    group by sku_id
```

开始：进入hive选择gamll数据库，执行脚本与配置文件里面的dws_sql建表hsql：生成5张表，加上系统函数测试生成的表，dws层共六张表。

```
hive (gmall)> show tables;# 查看gmall里面表的数量是否为59，ods25+dwd28+dws6=59
dwd_to_dws 2020-03-10 # dwd_to_dws数据导入脚本DWS层
dwd_to_dws 2020-03-11
# 查看导入数据
hive (gmall)> 
select * from dws_uv_detail_daycount where dt='2020-03-10';
select * from dws_user_action_daycount where dt='2020-03-10';
select * from dws_sku_action_daycount where dt='2020-03-10';
select * from dws_activity_info_daycount where dt='2020-03-10';
select * from dws_area_stats_daycount where dt='2020-03-10';


select * from dws_uv_detail_daycount where dt='2020-03-11';
select * from dws_user_action_daycount where dt='2020-03-11';
select * from dws_sku_action_daycount where dt='2020-03-11';
select * from dws_activity_info_daycount where dt='2020-03-11';
select * from dws_area_stats_daycount where dt='2020-03-11';
```

# dwt搭建

DWT层： 数据主题层，围绕某个具体要统计的主题，例如日活，订单，用户等，一张总表，记录了从第一天截至今日的所有的围绕此主题的汇总数据。非分区表，一张普通表。

宽表字段来源：维度关联的事实表度量值+开头、结尾+累积+累积一个时间段。

主题宽表：设备、会员、商品、活动、地区五个。

```
设备主题宽表dwt_uv_topic
会员主题宽表dwt_user_topic
商品主题宽表dwt_sku_topic
活动主题宽表dwt_activity_topic
地区主题宽表dwt_area_topic
```

sql重点分析

```sql
--会员主题宽表之
```

开始：进入hive选择gamll数据库，执行脚本与配置文件里面的dwt_sql建表hsql：生成5张表。

```
hive (gmall)> show tables;# 查看gmall里面表的数量是否为64，ods25+dwd28+dws6+dwt5=564
dws_to_dwt 2020-03-10 # dws_to_dwt数据导入脚本DWT层
dws_to_dwt 2020-03-11
# 查看导入数据
hive (gmall)> 
select * from dwt_uv_topic limit 5;
select * from dwt_user_topic limit 5;
select * from dwt_sku_topic limit 5;
select * from dwt_activity_topic limit 5;
select * from dwt_area_topic limit 5;
```



# ads搭建

ADS层： 数据应用层。可以从DWS层或DWT层取需要的数据。非分区表，一张普通表。

开始

# 1