# zabbix

官网：https://www.zabbix.com/

Zabbix是一款能够监控各种网络参数以及服务器健康性和完整性的软件。Zabbix使用灵活的通知机制，允许用户为几乎任何事件配置基于邮件的告警。这样可以快速反馈服务器的问题。基于已存储的数据，Zabbix提供了出色的报告和数据可视化功能。

架构：

**Zabbix Agent**：部署在监控的目标上，主动监测本地的资源和应用(硬件驱户发送通知动，内存，处理器统计等)。

**Zabbix Server**：收集监控数据，计算是否满足触发器条件，向用户发送通知。

**Database**：存储所有配置信息。

**Zabbix-web**：用户操作界面，监控信息展示。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d515b5a344214324b4e99120aaafa7ab~tplv-k3u1fbpfcp-zoom-1.image)

术语：

Host（主机）：一台你想监控的网络设备，用IP或域名表示。
Item（监控项）：你想要接收的主机的特定数据，一个度量数据。
Trigger（触发器）：一个被用于定义问题阈值和“评估”监控项接收到的数据的逻辑表达式。
Action（动作）：一个对事件做出反应的预定义的操作，比如邮件通知。

# 安装配置

集群规划：

| 服务名称 | 子服务        | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------------- | ----------------- | ----------------- | ----------------- |
| Superset | zabbix-server | √                 |                   |                   |
|          | zabbix-web    | √                 |                   |                   |
|          | zabbix-agent  | √                 | √                 | √                 |

 注意：安装zabbix-server的节点必须保证已安装mysql，10051默认server端口，10050默认agent端口。

安装过程参考https://www.zabbix.com/download?zabbix=5.0&os_distribution=centos&os_version=7&db=mysql&ws=apache

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbb645d5f6314b9c85e33fa35cf4000d~tplv-k3u1fbpfcp-zoom-1.image)

主节点

```
sudo systemctl status firewalld # 3台节点关闭防火墙
sudo vim /etc/selinux/config # 3台节点关闭SELinux
SELINUX=disabled

reboot # 3台节点重启
# rpm -Uvh出现error: skipping repo.zabbix.rpm - transfer failed，是因为被强，换源
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm # 不用，被墙
rpm -Uvh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm # 

vim /etc/yum.repos.d/zabbix.repo 
[zabbix]
...
#baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/$basearch
...

[zabbix-frontend]
...
#baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/$basearch/frontend
enabled=1

yum clean all # Install Zabbix repository 创建初始数据库，mysql数据库主机上运行以下命令。
yum install zabbix-server-mysql zabbix-agent -y # Install Zabbix server and agent
yum install centos-release-scl -y # Install Zabbix frontend前端web
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y # Install Zabbix frontend packages.
mysql -uroot -p1234 # Create zabbix database
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@localhost identified by '1234';
grant all privileges on zabbix.* to zabbix@localhost;
quit;
# 解决mysql Your password does not satisfy the current policy requirements，没出现就略过
SHOW VARIABLES LIKE 'validate_password%'; 
set global validate_password_policy=LOW;
set global validate_password_length=4;

# Zabbix服务器主机上，导入初始架构和数据。系统将提示您输入新创建的密码。
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p 1234
# 如果出现 ERROR: ASCII '\0' 就按下面步骤来
gzip -d /usr/share/doc/zabbix-server-mysql*/create.sql.gz # gzip好像只能解压在当前目录，得到create.sql
mysql -uroot -p1234
use zabbix; 
source create.sql;

vim /etc/zabbix/zabbix_server.conf # 为Zabbix服务器配置数据库
DBUser=root
DBPassword=1234

vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf #  为Zabbix前端配置PHP，设置时区。

; php_value[date.timezone] = Asia/Shanghai

# php安装rh-php72
yum install centos-release-scl-rh
yum search php-fpm
yum install -y rh-php72

vim /etc/opt/rh/rh-php72/php.ini
[Date]
...
date.timezone = Asia/Shanghai #;去掉，;是ini文件注释符号

php -v # 查看版本以检测是否安装成功
# 启动Zabbix服务器和代理进程，并使其在系统启动时启动。
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
# Zabbix server日志查看/var/log/zabbix/zabbix_server.log
```

agent节点

```
rpm -Uvh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm # 

vim /etc/yum.repos.d/zabbix.repo 
[zabbix]
...
#baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/$basearch
...

[zabbix-frontend]
...
#baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/$basearch/frontend
enabled=1

yum install zabbix-agent -y
# 启动Zabbix服务器和代理进程，并使其在系统启动时启动。
systemctl restart zabbix-agent
systemctl enable zabbix-agent
```

## Zabbix web

可看https://www.zabbix.com/documentation/current/manual/installation/install#installing_frontend

1.访问：http：// <server_ip_or_name> / zabbix，如hadoop102/zabbix

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf2b82b45e234938ad654785fa154513~tplv-k3u1fbpfcp-zoom-1.image)

2.确保满足所有软件先决条件。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62a72e2e47664fddaa686ed63aa56057~tplv-k3u1fbpfcp-zoom-1.image)

3.输入用于连接数据库的详细信息。Zabbix数据库必须已经创建。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b15b68b6ed4646a0a8150a6b68bd7cfa~tplv-k3u1fbpfcp-zoom-1.image)

4.输入Zabbix服务器详细信息。Host填写host_name/ip，不要填写localhost。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86754cc3de0e4de886bdee8e9a64c1e8~tplv-k3u1fbpfcp-zoom-1.image)

5.查看设置摘要。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70124f80b69d480eb48c4ed162f66d00~tplv-k3u1fbpfcp-zoom-1.image)

6.下载配置文件zabbix.conf.php，并将其放在将Zabbix PHP文件复制到的Web服务器HTML文档子目录中的conf /下。如图中建议的/etc/zabbix/web/，点击finish完成安装。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f7f5dbdc2a547fdae5cd74382867146~tplv-k3u1fbpfcp-zoom-1.image)

7.Zabbix前端已经准备好！默认用户名为**Admin**，密码为**zabbix**。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/341e2a3f312c4ac5a8b61a1f0b091d31~tplv-k3u1fbpfcp-zoom-1.image)



# 使用

登录进去进行使用，参考https://www.zabbix.com/documentation/current/manual/quickstart

1.配置Host：点击Configuration/Hosts/Create hostst，配置Host，hadoop103和hadoop104都来一下

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abb6607c8253431788a61e323354497a~tplv-k3u1fbpfcp-zoom-1.image)

直到所有机器都展示出来为止

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a38eee141a9f4719ac5b0dbe0034099c~tplv-k3u1fbpfcp-zoom-1.image)

2.Items监控项配置：在Monitoring的hosts里面点击hadoop102的Items，再点击Create item。先启动hadoop集群

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc80e52429ce4c3d8b2cdbac98f73272~tplv-k3u1fbpfcp-zoom-1.image)

查看Item最新数据

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5af671fb5e654e3c86dc3803deb46314~tplv-k3u1fbpfcp-zoom-1.image)

3.创建Trigger：点击Conguration/Hosts/Triggers，点击Create Trigger

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c90fb23a0e974dc7893fb3ff4974a3cc~tplv-k3u1fbpfcp-zoom-1.image)

测试Trigger：关闭集群中的HDFS，会有如下效果

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f95934c9151144f0919be4f7afac303a~tplv-k3u1fbpfcp-zoom-1.image)

4.创建Media type：点击Administration/Media types/Email，编辑Email

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bac22a5760c242b9abac772776c704dc~tplv-k3u1fbpfcp-zoom-1.image)



测试Email，查看1519958263@163.com

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53a594bc33b94366848a51f13a8f5b30~tplv-k3u1fbpfcp-zoom-1.image)

Email绑定收件人

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e06522e001ca45dca2b604c1475fdbbe~tplv-k3u1fbpfcp-zoom-1.image)

# 1