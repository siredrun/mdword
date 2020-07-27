# hive安装

前提：安装好mysql或高可用mysql、Java、hadoop3，hive2.xy匹配hadoop2.xy，hive3.xy匹配hadoop3.xy。

集群规划：版本3.1.2

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Hive     | Hive   | √                 |                   |                   |

安装

```
cd /opt/software
wget https://downloads.apache.org/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /opt/module/
mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
vim /etc/profile
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin

xsync /etc/profile
source /etc/profile

# 解决日志Jar包冲突，进入/opt/module/hive/lib目录和"Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V"
mv /opt/module/hive/lib/log4j-slf4j-impl-2.10.0.jar /opt/module/hive/lib/log4j-slf4j-impl-2.10.0.jar.bak
mv /opt/module/hive/lib/guava-19.0.jar /opt/module/hive/lib/guava-19.0.jar.bak
cp /opt/module/hd/share/hadoop/hdfs/lib/guava-27.0-jre.jar /opt/module/hive/lib
wget https://cdn.mysql.com/archives/mysql-connector-java-5.1/mysql-connector-java-5.1.48.tar.gz
tar -zxvf mysql-connector-java-5.1.48.tar.gz
cp mysql-connector-java-5.1.48/mysql-connector-java-5.1.48.jar /opt/module/hive/lib/
```

vim $HIVE_HOME/conf/hive-site.xml # 添加如下内容

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/hive?useSSL=false</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>1234</value>
    </property>

    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>

    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop102:9083</value>
    </property>

    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>

    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>

    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
    
    <property>
        <name>hive.server2.active.passive.ha.enable</name>
        <value>true</value>
    </property>
</configuration>
```

继续配置

```
mysql -uroot -p1234 # 登陆MySQL
mysql> create database hive; # 新建Hive元数据库
mysql> quit;
schematool -initSchema -dbType mysql -verbose # 初始化Hive元数据库
mysql -uroot -p1234
mysql> use hive;
mysql> show tables;
| WM_POOL                       |
| WM_POOL_TO_TRIGGER            |
| WM_RESOURCEPLAN               |
| WM_TRIGGER                    |
| WRITE_SET                     |
+-------------------------------+
74 rows in set (0.00 sec)

mysql> quit;
# Hive 2.x以上版本，要先启动metastore和hiveserver2，这两个服务，否则会报错"FAILED: HiveException java.lang.RuntimeException: Unable to instantiate "
hive --service metastore
hive --service hiveserver2或 hiveserver2
tail -f /tmp/admin/hive.log 
2020-07-18T23:41:22,249  INFO [main] handler.ContextHandler: Started o.e.j.s.ServletContextHandler@4eb9f2af{/logs,file:///tmp/admin/,AVAILABLE}
2020-07-18T23:41:22,249  INFO [main] server.AbstractConnector: Started ServerConnector@2b1cd7bc{HTTP/1.1,[http/1.1]}{0.0.0.0:10002}
2020-07-18T23:41:22,250  INFO [main] server.Server: Started @63701ms
2020-07-18T23:41:22,250  INFO [main] server.HiveServer2: Web UI has started on port 10002

# 启动后要等两分钟左右才访问连接，访问hadoop102:10002和使用datagrip（和idea同一家公司出品）能成功表示hive启动没问题。
mv $HIVE_HOME/conf/hive-env.sh.template $HIVE_HOME/conf/hive-env.sh
vim $HIVE_HOME/conf/hive-env.sh # 加入下面配置
export HADOOP_HOME=/opt/module/hd # 配置HADOOP_HOME路径
export HIVE_CONF_DIR=/opt/module/hive/conf # 配置HIVE_CONF_DIR路径

# hive log默认存放在/tmp/root/hive.log目录下（当前用户名下）,这一项可不配，不是强制性要求。
mv $HIVE_HOME/conf/hive-log4j2.properties.template $HIVE_HOME/conf/hive-log4j2.properties
vim  $HIVE_HOME/conf/hive-log4j2.properties
hive.log.dir=/opt/module/hive/logs
```

**关于metastore和hiveserver2启动**

metastore：源数据，就是mysql下hive数据库下面的一系列表。如“show tables;”就需要用到，访问源数据有两种方式：

1.直接访问，每次都要重新去访问连接mysql，性能不佳。

2.通过启动metastore服务去访问源数据，metastore服务只要一启动，就会自动连接好mysql。hive客户端没必要每次都要去重新连接mysql。在$HIVE_HOME/conf/hive-site.xml中做下列配置：

```xml
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop102:9083</value>
    </property>
```

做好配置后必须先启动metastore服务，才能使用hive客户端。不然会

```
hive (default)> show tables;
FAILED: HiveException java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```

hiveserver2：需要远程通过可视化工具如datadrip连接时必须启动。注意：如果hive.metastore.uris有配置metastore必须启动datagrip才能连接。