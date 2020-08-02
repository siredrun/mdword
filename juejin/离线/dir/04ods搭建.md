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

## 用户行为数据

开始：进入hive选择gamll数据库，执行脚本与配置文件里面的dwd_sql数据建表hive sql：生成26张表（包含业务数据和用户行为数据的表）

```
hive (gmall)> show tables;#查看gmall里面表的数量是否为51，ods层25张加dwd层26张等于51张
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







阿道夫

```
# 时间维度表（特殊），创建临时表，非列式存储
hive (gmall)> DROP TABLE IF EXISTS `dwd_dim_date_info_tmp`;
CREATE EXTERNAL TABLE `dwd_dim_date_info_tmp`(
    `date_id` string COMMENT '日',
    `week_id` int COMMENT '周',
    `week_day` int COMMENT '周的第几天',
    `day` int COMMENT '每月的第几天',
    `month` int COMMENT '第几月',
    `quarter` int COMMENT '第几季度',
    `year` int COMMENT '年',
    `is_workday` int COMMENT '是否是周末',
    `holiday_id` int COMMENT '是否是节假日'
)
row format delimited fields terminated by '\t'
location '/warehouse/gmall/dwd/dwd_dim_date_info_tmp/';
# 将数据导入临时表，先把date_info.txt文件上传到hadoop102的/opt/module/db_log/路径。
hive (gmall)> load data local inpath '/opt/module/db_log/date_info.txt' into table dwd_dim_date_info_tmp;
hive (gmall)> insert overwrite table dwd_dim_date_info select * from dwd_dim_date_info_tmp; # 将数据导入正式表
hive (gmall)> select * from dwd_dim_date_info_tmp limit 730,1; # 验证总数731
hive (gmall)> select * from dwd_dim_date_info limit 730,1; # 验证总数731

# ods_to_dwd_db业务数据脚本DWD层包含业务表12张表的数据导入（时间维度表和用户维度表需手动执行）
ods_to_dwd_db first 2020-03-10 # 第一次使用first，后面使用all
ods_to_dwd_db all 2020-03-11
```



# 1