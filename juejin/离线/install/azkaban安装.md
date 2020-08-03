集群规划：版本 2.5.0

| 服务名称 | 子服务                | hadoop102 | hadoop103 | hadoop104 |
| -------- | --------------------- | --------- | --------- | --------- |
| Azkaban  | AzkabanWebServer      | √         |           |           |
|          | AzkabanExecutorServer | √         |           |           |

Azkaban是由Linkedin开源的一个批量工作流任务调度器。用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban定义了一种KV文件格式来建立任务之间的依赖关系，并提供一个易于使用的web用户界面维护和跟踪你的工作流。

安装

密钥库相关知识了解
Keytool：是java数据证书的管理工具，使用户能够管理自己的公/私钥对及相关证书
-keystore：指定密钥库的名称及位置（产生的各类信息将不在.keystore文件中）
-genkey：在用户主目录中创建一个默认文件".keystore" 
-alias：对我们生成的.keystore进行指认别名；如果没有默认是mykey
-keyalg：指定密钥的算法 RSA/DSA 默认是DSA



```
cd /opt/software/# 下载下面三个文件放到/opt/software目录下
azkaban-web-server-2.5.0.tar.gz
azkaban-executor-server-2.5.0.tar.gz
azkaban-sql-script-2.5.0.tar.gz

mkdir -p /opt/module/azkaban
# 解压azkaban-web-server-2.5.0.tar.gz、azkaban-executor-server-2.5.0.tar.gz、azkaban-sql-script-2.5.0.tar.gz到/opt/module/azkaban目录下
tar -zxvf /opt/software/azkaban-web-server-2.5.0.tar.gz -C /opt/module/azkaban/
tar -zxvf /opt/software/azkaban-executor-server-2.5.0.tar.gz -C /opt/module/azkaban/
tar -zxvf /opt/software/azkaban-sql-script-2.5.0.tar.gz -C /opt/module/azkaban/
mv /opt/module/azkaban/azkaban-web-2.5.0/ /opt/module/azkaban/server
mv /opt/module/azkaban/azkaban-executor-2.5.0/ /opt/module/azkaban/executor
vim /etc/profile # 文件末尾加入下面内容
#AZKABAN_WEB
export AZKABAN_WEB=/opt/module/azkaban/server
export PATH=$PATH:$AZKABAN_WEB/bin
#AZKABAN_EXECUTOR
export AZKABAN_EXECUTOR=/opt/module/azkaban/server
export PATH=$PATH:$AZKABAN_EXECUTOR/bin

xsync /etc/profile
source /etc/profile # 每个节点都source一下
# 进入mysql，创建azkaban数据库，并将解压的脚本导入到azkaban数据库。source后跟.sql文件，用于批量处理.sql文件中的sql语句。
mysql -uroot -p1234 -e "create database azkaban;"
mysql -uroot -p1234 -e "use azkaban;source /opt/module/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql;"

# 密钥库生产，一开始设置密码Enter keystore password和确认秘密Re-enter new password都是123456，最低6位，“Is CN=Unknown, OU=Unknown..correct? [no]: ”这里写y。除开这三项其它回车即可。这样密钥库和jetty的秘密都是123456，方便记忆。
keytool -keystore keystore -alias jetty -genkey -keyalg RSA
keytool -keystore keystore -list #验证，执行此命令加设置的密钥库秘密出现“Your keystore contains 1 entry”表示成功
# 将keystore文件拷贝到azkaban web服务器根目录中，keystore命令在哪里使用就在哪里生成keystore文件
cp keystore /opt/module/azkaban/server/
vim /opt/module/azkaban/server/conf/azkaban.properties #  Web服务器配置，修改下面内容
web.resource.dir=/opt/module/azkaban/server/web/
default.timezone.id=Asia/Shanghai
user.manager.xml.file=/opt/module/azkaban/server/conf/azkaban-users.xml
executor.global.properties=/opt/module/azkaban/executor/conf/global.properties
mysql.host=hadoop102
mysql.user=root
mysql.password=1234
#SSL文件名（绝对路径）
jetty.keystore=/opt/module/azkaban/server/keystore
#SSL文件密码
jetty.password=123456
#Jetty主密码与keystore文件相同
jetty.keypassword=123456
#SSL文件名（绝对路径）
jetty.truststore=/opt/module/azkaban/server/keystore
#SSL文件密码
jetty.trustpassword=123456

vim /opt/module/azkaban/server/conf/azkaban-users.xml #  增加管理员用户，在azkaban-users内增加下面内容
<user username="admin" password="admin" roles="admin" />

vim /opt/module/azkaban/executor/conf/azkaban.properties #  执行服务器配置，修改下面内容
default.timezone.id=Asia/Shanghai
executor.global.properties=/opt/module/azkaban/executor/conf/global.properties
mysql.host=hadoop102
mysql.database=azkaban
mysql.user=root
mysql.password=1234

azkaban-executor-start.sh # 先启动executor服务器，避免Web Server会因为找不到执行器启动失败。
azkaban-web-start.sh # 再启动web服务器
jps # 查看进程
2273 AzkabanWebServer
2307 Jps
2217 AzkabanExecutorServer
# 启动完成后，在浏览器(建议使用谷歌浏览器)中输入https://服务器IP地址:8443，即可访问azkaban服务了。如https://hadoop102:8443/，注意是https，s别忘了加，不会http请求会访问不到。在登录中输入刚才在azkaban-users.xml文件中新添加的户用名及密码，点击login。
```

