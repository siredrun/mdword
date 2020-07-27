# sqoop安装

Sqoop是一个用于hdoop和关系型数据库间数据相互转移的工具。前提：安装好hadoop和mysql。集群信息：版本1.4.6。

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Sqoop    | Sqoop  | √                 |                   |                   |

安装

```
cd /opt/software
wget https://archive.apache.org/dist/sqoop/1.4.6/sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz
tar -zxvf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
mv /opt/module/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/ /opt/module/sqoop
sudo vim /etc/profile # 末尾加入
#SQOOP_HOME
export SQOOP_HOME=/opt/module/sqoop
export PATH=$PATH:$SQOOP_HOME/bin

xsync /etc/profile
source /etc/profile
mv /opt/module/sqoop/conf/sqoop-env-template.sh /opt/module/sqoop/conf/sqoop-env.sh
vim /opt/module/sqoop/conf/sqoop-env.sh # 增加如下内容
export HADOOP_COMMON_HOME=/opt/module/hd
export HADOOP_MAPRED_HOME=/opt/module/hd
export HIVE_HOME=/opt/module/hive
export ZOOKEEPER_HOME=/opt/module/zk
export ZOOCFGDIR=/opt/module/zk/conf

cd /opt/software
wget https://cdn.mysql.com/archives/mysql-connector-java-5.1/mysql-connector-java-5.1.48.tar.gz
tar -zxvf mysql-connector-java-5.1.48.tar.gz
cp mysql-connector-java-5.1.48/mysql-connector-java-5.1.48.jar /opt/module/sqoop/lib/
sqoop version
# 测试Sqoop是否能够成功连接数据库并显示出所有数据库列表
sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 1234 
```