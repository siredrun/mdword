# presto

集群规划：版本 0.189

| 服务名称 | 子服务      | hadoop102 | hadoop103 | hadoop104 |
| -------- | ----------- | --------- | --------- | --------- |
| Presto   | Coordinator | √         |           |           |
|          | Worker      |           | √         | √         |

注意：命令行Client和可视化Client安装在Coordinator子服务所在节点。

官网：https://prestodb.io/

Presto是一个开源的分布式SQL查询引擎，适用于交互式分析查询，数据量支持GB到PB字节。

Presto的设计和编写完全是为了解决像Facebook这样规模的商业数据仓库的交互式分析和处理速度的问题。

架构：Presto由一个Coordinator和多个Worker组成。

1)由客户端提交查询，从Presto命令行CLI提交到Coordinator。

2) Coordinator解析查询计划，然后把任务分发给CatalogWorker执行。

3) Worker负责执行任务和处理数据。

4) Catalog表示数据源。一个Catalog包含Schema和Connector。

5) Connector是适配器，用于Presto和数据源(如Hive、Redis) 的连接， 类似于JDBC。

6) Schema类似于Mysql中数据库， Table类似于MySQL中表。

7) Coordinator是负责从Worker获取结果并返回最终结果给Client。

```mermaid
graph LR

a[Client] -->b[Coordinator]

    b --> ca[Worker]
    
    b --> cb[Worker]
    
    b --> cc[Worker]
    
    ca --> caa[Hive] --> cab[Connector]  
    
    caa --> cac[Schema and Table]
    
    cb --> cba[Kafka] --> cbb[Connector]  
    
    cba --> cbc[Schema and Table]
    
    cc --> cca[Redis] --> ccb[Connector]  
    
    cca --> ccc[Schema and Table]

    Z[Hive Metastore] --> b
```

**优点**：Presto基于内存运算， 减少了硬盘IO， 计算更快；能够连接多个数据源，跨数据源连表查，如从Hive查询大量网站访问记录， 然后从Mysql中匹配出设备信息。

**缺点**：Presto能够处理PB级别的海量数据分析， 但Presto并不是把PB级数据都放在内存中计算的。而是根据场景， 如Count、AVG等聚合运算， 是边读数据边计算， 再清内存， 再读数据再计算，这种耗的内存并不高。但是连表查，就可能产生大量的临时数据，因此速度会变慢。

**Presto、Impala性能比较**：可看https://blog.csdn.net/u012551524/article/details/79124532，测试结论：Impala性能稍领先于Presto，但是Presto在数据源支持上非常丰富，包括Hive、图数据库、传统关系型数据库、Redis等。

## 安装

```
cd /opt/software
# presto server
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.238.2/presto-server-0.238.2.tar.gz
# presto command client
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.238.2/presto-cli-0.238.2-executable.jar
# presto jdbc jar
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-jdbc/0.238.2/presto-jdbc-0.238.2.jar
cp /opt/software/presto-jdbc-0.238.2.jar $JAVA_HOME/lib
xsync $JAVA_HOME/lib
tar -zxvf presto-server-0.238.2.tar.gz -C /opt/module/
mv /opt/module/presto-server-0.238.2/ /opt/module/presto
vim /etc/profile
#PRESTO_HOME
export PRESTO_HOME=/opt/module/presto
export PATH=$PATH:$PRESTO_HOME/bin

xsync /etc/profile
source /etc/profile
mkdir /opt/module/presto/data
mkdir /opt/module/presto/etc
vim /opt/module/presto/etc/jvm.config
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError

# 命令行Client安装
mv /opt/software/presto-cli-0.238.2-executable.jar /opt/software/prestocli
sudo chmod +x /opt/software/prestocli
mv /opt/software/prestocli /opt/module/presto/bin

# Presto支持多个数据源，在Presto里面叫catalog，这里配置支持Hive的数据源，配置一个Hive的catalog
mkdir /opt/module/presto/catalog
vim /opt/module/presto/etc/catalog/hive.properties # hive catalog
connector.name=hive-hadoop2
hive.metastore.uri=thrift://hadoop102:9083

vim /opt/module/presto/etc/catalog/mysql.properties # mysql catalog
connector.name=mysql
connection-url=jdbc:mysql://hadoop102:3306
connection-user=root
connection-password=1234

vim /opt/module/presto/etc/node.properties
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
node.data-dir=/opt/module/presto/data

xsync $PRESTO_HOME
# 分发之后，修改配置下几台机器的node属性，node id每个节点都不一样。
xcall cat /opt/module/presto/etc/node.properties | grep 'node.id'
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
node.id=ffffffff-ffff-ffff-ffff-fffffffffffe
node.id=ffffffff-ffff-ffff-ffff-fffffffffffd

vim /opt/module/presto/etc/config.properties # hadoop102配置coordinator节点
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8881
query.max-memory=50GB
discovery-server.enabled=true
discovery.uri=http://hadoop102:8881

vim /opt/module/presto/etc/config.properties # hadoop103、hadoop104上配置worker节点
coordinator=false
http-server.http.port=8881
query.max-memory=50GB
discovery.uri=http://hadoop102:8881

hive --service metastore # 启动hive
xcall $PRESTO_HOME/bin/launcher start # 前台启动Presto：launcher run，控制台显示日志；后台启动Presto：launcher start，日志查看路径/opt/module/presto/data/var/log，三台一起启动。
jpsall|grep 'Pre'
115822 PrestoServer
61486 PrestoServer
64524 PrestoServer

xcall $PRESTO_HOME/bin/launcher stop # 关闭
prestocli --server hadoop102:8881 --catalog mysql --schema sys # 启动prestocli，出现类似presto:*
presto:sys> show tables;
prestocli --server hadoop102:8881 --catalog hive --schema default
presto:default> show tables;
```

可视化安装

```
# 将yanagishima-18.0.zip下载上传到/opt/module目录
unzip /opt/module/yanagishima-18.0.zip
mv /opt/module/yanagishima-18.0 /opt/module/yanagishima
mv /opt/module/conf/yanagishima.properties /opt/module/conf/yanagishima.properties.bak
vim /opt/module/conf/yanagishima.properties
jetty.port=7080
presto.datasources=admin-presto
presto.coordinator.server.admin-presto=http://hadoop102:8881
catalog.admin-presto=hive
schema.admin-presto=default
sql.query.engines=presto

xcall $PRESTO_HOME/bin/launcher start #保证Presto启动
/opt/module/yanagishima/bin/yanagishima-start.sh
# 启动web页面http://hadoop102:7080 
```

就是一个简单业务，hive和mysql因为在presto catalog里面配置了hive和mysql

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d9968d838a14803a609945de0803a9c~tplv-k3u1fbpfcp-zoom-1.image)

## 优化

**数据存储**

1.合理设置分区：与Hive类似，Presto会根据元数据信息读取分区数据，合理的分区能减少Presto数据读取量，提升查询性能。
使用列式存储。Presto对ORC文件读取做了特定优化，因此在Hive中创建Presto使用的表时，建议采用ORC格式存储。相对于Parquet，Presto对ORC支持更好。
2.使用压缩：数据压缩可以减少节点间数据传输对IO带宽压力，对于即席查询需要快速解压，建议采用Snappy压缩。

**查询SQL**

1.只选择使用的字段：由于采用列式存储，选择需要的字段可加快字段的读取、减少数据量。避免采用*读取所有字段。

```sql
[GOOD]: SELECT time, user, host FROM tbl
[BAD]:  SELECT * FROM tbl
```

2.过滤条件必须加上分区字段：对于有分区的表，where语句中优先使用分区字段进行过滤。acct_day是分区字段，visit_time是具体访问时间。

```sql
[GOOD]: SELECT time, user, host FROM tbl where acct_day=20171101
[BAD]:  SELECT * FROM tbl where visit_time=20171101
```

3.Group By语句优化：合理安排Group by语句中字段顺序对性能有一定提升。将Group By语句中字段按照每个字段distinct数据多少进行降序排列。

```sql
[GOOD]: SELECT GROUP BY uid, gender
[BAD]:  SELECT GROUP BY gender, uid
```

4.Order by时使用Limit：Order by需要扫描数据到单个worker节点进行排序，导致单个worker需要大量内存。如果是查询Top N或者Bottom N，使用limit可减少排序计算和内存压力。

```sql
[GOOD]: SELECT * FROM tbl ORDER BY time LIMIT 100
[BAD]:  SELECT * FROM tbl ORDER BY time
```

5.使用Join语句时将大表放在左边：Presto中join的默认算法是broadcast join，即将join左边的表分割到多个worker，然后将join右边的表数据整个复制一份发送到每个worker进行计算。如果右边的表数据量太大，则可能会报内存溢出错误。

```sql
[GOOD] SELECT ... FROM large_table l join small_table s on l.id = s.id
[BAD] SELECT ... FROM small_table s join large_table l on l.id = s.id
```

**注意事项**

1.字段名引用：避免和关键字冲突：MySQL对字段加反引号`、Presto对字段加双引号分割。当然，如果字段名称不是关键字，可以不加这个双引号。
2.时间函数：对于Timestamp，需要进行比较的时候，需要添加Timestamp关键字，而MySQL中对Timestamp可以直接进行比较。

```sql
/*MySQL的写法*/
SELECT t FROM a WHERE t > '2017-01-01 00:00:00'; 

/*Presto中的写法*/
SELECT t FROM a WHERE t > timestamp '2017-01-01 00:00:00';
```

3.不支持INSERT OVERWRITE语法：Presto中不支持insert overwrite语法，只能先delete，然后insert into。
4.PARQUET格式：Presto目前支持Parquet格式，支持查询，但不支持insert。