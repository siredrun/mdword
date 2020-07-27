# mysql HA

集群规划

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| mysql HA | mysql  | √                 | √                 |                   |

![](https://imgkr.cn-bj.ufileos.com/0fb49724-70f4-49cb-85dc-0ed65c8af296.png)

搭建高可用MySQL，两个mysql相互为主从，一旦一个出现问题，另外一个马上补上。核心两点：active检测keepalive和数据同步。keepalive服务是检测本机的mysql是否在运行，两个keepalive组成一个keepalive组，这个组会对外面提供一个虚拟IP，客户端只要访问这个虚拟IP就行。虚拟IP同时只能由一台是active状态的mysql占有。一旦Active状态的mysql出现问题，我们需要让当前机器监控mysql的keepAlive进程结束，结束后当前占用的虚拟IP才可以被放弃放弃后其他的keepAlive进程才会抢占，将请求路由到当前机器的Msql。

主从复制原理：slave会从master读取binlog来进行数据同步。

步骤：
master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志事件，binary log events；
slave将master的binary log events拷贝到它的中继日志（relay log）；
slave重做中继日志中的事件，将改变应用到自己的数据库中。 MySQL复制是异步且串行化的。

复制的问题：延时。

基本原则：每个slave只有一个master、每个slave有且只有一个唯一的服务器ID、每个master可以有多个slave。

确保hadoop102和hadoop103上有安装mysql，没有的安装下面重新安装一遍。

```
cd /opt/software
yum -y install yum-utils # yum-config-manager命令在yum-utils包里
#下载相关https://dev.mysql.com/downloads/repo/yum/
wget https://repo.mysql.com//mysql80-community-release-el7-3.noarch.rpm
rpm -Uvh mysql80-community-release-el7-3.noarch.rpm # -Uvh：升级软件包
# 默认情况下，默认启用最新GA系列（当前为MySQL 8.0）的子存储库，而所有其他系列（例如，MySQL 5.7系列）的子存储库均被禁用。使用此命令可查看MySQL Yum存储库中的所有子存储库，并查看已启用或禁用了哪些子存储库。
yum repolist all | grep mysql # 列出所有版本
# 发现8.0版本是enabled的，5.7版本是disabled的，如果需要安装5.7版本的，所以把8.0的进行禁用，然后再启用5.7版本
yum-config-manager --disable mysql80-community # 禁用8.0版本
yum-config-manager --enable mysql57-community # 启用5.7版本
yum repolist enabled|grep mysql #检查启用版本，进行安装时请确保只有一个版本启用，否则会显示版本冲突
yum install -y mysql-community-server 
mysql -V # 验证安装
systemctl enable mysqld
systemctl start mysqld
systemctl status mysqld
grep 'temporary password' /var/log/mysqld.log #查看root用户密码 grep '关键字' 文件路径 ：直接在文件里查看包括内容的那一行内容。把"generated for root@localhost: "后面的内容（密码）按ctrl+insert复制，shift+insert粘贴在执行。注意：如果结果有多行，选择时间最新的那条记录的密码。
mysql -uroot -p #加密码登录
#SHOW VARIABLES LIKE 'validate_password%'; #mysql初始密码策略
set global validate_password_policy=LOW; # 设置密码的验证强度等级，先设置，不然会报"password does not satisfy"
set global validate_password_length=4; # 设置密码长度为4
ALTER USER 'root'@'localhost' IDENTIFIED BY '1234'; #重置密码
grant all privileges on *.* to 'root' @'%' identified by '1234'; # 配置root用户远程访问权限
flush privileges;
mysql -uroot -p1234 #exit;退出中心以密码1234登录验证
mysql -h hadoop103 -uroot -p1234 # 测试root是否可以从外部地址(如hadoop103)主机名登录
mysqladmin processlist -uroot -p1234 #查看当前mysql服务器收到了哪些客户端连接请求
```

mysql主从配置，hadoop102和hadoop103都要。

```
vim /etc/my.cnf # 在[mysqld]下配置：
server_id = 103 # 另外一台，配置也一样，只需要修改servei_id
log-bin=mysql-bin
binlog_format=mixed
relay_log=mysql-relay #中继日志

xcall cat /etc/my.cnf | grep 'server.id'
server_id = 102
server_id = 103

xcall systemctl restart mysqld # 重启mysql服务
# 在主机mysql上使用root@localhost登录,授权从机可以使用哪个用户登录。双向主从，hadoop102和hadoop103的mysql里都要都执行。
GRANT replication slave ON *.* TO 'slave'@'%' IDENTIFIED BY '1234'; 
select * from mysql.user where user = 'slave'; # 查看是否有信息
# 测试
mysql -uroot -p1234 # hadoop102、hadoop103启动mysql
show master status; #查看主机如hadoop103的binlog文件的最新位置，记住File和Position（master_log_pos）的值。
#在从机如hadoop102上执行以下语句，master_log_file值为File、master_log_pos值为Position。
change master to master_user='slave', master_password='1234',master_host='172.31.82.16',master_log_file='mysql-bin.000002',master_log_pos=588;
start slave; # 在从机如hadoop102上开启同步线程
show slave status; # 查看从机如hadoop102同步线程的状态，看到Slave_IO_Running为Yes、Slave_IO_SQL为Yes表示成功。
create database test; # 主机如hadoop103创建一个数据库。
show databases; #查看从机如hadoop102上有没有数据test，有就表示成功，没有就失败。
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+

# 下面再按照hadoop102主、hadoop103从设置。
show master status; #hadoop102
#hadoop103
change master to master_user='slave', master_password='1234',master_host='172.31.82.15',master_log_file='mysql-bin.000002',master_log_pos=438; 
start slave; #hadoop103
show slave status \G; #hadoop103
create database test1;  #hadoop102
show databases; #hadoop103
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
| test1              |
+--------------------+
```

mysql HA 高可用，hadoop102和hadoop103都要。

```
xcall yum install -y keepalived
xcall cat /dev/null > /etc/keepalived/keepalived.conf
vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
   router_id MySQL-ha
}
vrrp_instance VI_1 {
    state master # 初始状态
    interface eth0 # 网卡
    virtual_router_id 51 # 虚拟器路由ID
    priority 100 # 优先级
    advert_int 1 # Keepalived心跳间隔
    nopreempt # 只在高优先级配置， 原master恢复之后不重新上位
    authentication {
        auth_type PASS # 认证相关
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.200.100 # 虚拟ip
    }
}
#声明虚拟服务器
virtual_server 192.168.200.100 3306 {
    delay_loop 6
    persistence_timeout 30
    protocol TCP
	# 声明真实服务器
    real_server 172.31.82.15 3306 {
    	notify_down /var/lib/mysql/killkeepalived.sh # 真实服务故障后调用脚本
    	TCP_CHECK {
    		connect_timeout 3 # 超时时间
    		nb_get_retry 1 # 重试次数
    		delay_before_retry 1 # 重试时间间隔
    	}
    }
}

xsync /etc/keepalived/keepalived.conf
xcall cat /etc/keepalived/keepalived.conf | grep 'real_server' # 查看真实服务器ip，要和服务器一一对应。
    real_server 172.31.82.15 3306 {
    real_server 172.31.82.16 3306 {

vim /var/lib/mysql/killkeepalived.sh # 当前机器keepalived检测到mysql故障时的通知脚本
#!/bin/bash
#停止当前机器的keepalived进程
sudo systemctl stop keepalived

xcall cat /var/lib/mysql/killkeepalived.sh | grep stop
sudo systemctl stop keepalived
sudo systemctl stop keepalived

xcall chmod +x /var/lib/mysql/killkeepalived.sh #记得加权限，不然执行会失败，关不掉keepalived
xcall systemctl enable keepalived # 开机自启动keepalived服务
xcall systemctl start keepalived # 启动keepalived服务，只需要当前启动，以后都可以开机自启动
xcall ip a |grep 'scope global' # 查看当前机器是否持有虚拟ip。
    inet 172.31.82.15/20 brd 172.31.95.255 scope global dynamic eth0
    inet 192.168.200.100/32 scope global eth0
    inet 172.31.82.16/20 brd 172.31.95.255 scope global dynamic eth0
    inet 192.168.200.100/32 scope global eth0

mysql -h192.168.200.100 -uroot -p1234 # 验证登录，能成功表示高可用成功，反之失败需要重来。
mysql -h192.168.200.100 -uroot -p1234 -se "show databases;" # 不登陆数据库执行MySQL命令
mysql -h192.168.200.100 -uroot -p1234 -se "create database ha1;"
mysql -hhadoop102 -uroot -p1234 -se "show databases;"|grep 'ha1'
# 查看hadoop102和hadoop103下是否有ha1数据库，没有就失败，hadoop102先启动keepalive，所以虚拟Ip使用的就是hadoop102，hadoop102和hadoop103都双向主从，所以在虚拟IP上改变数据，hadoop102会改变，hadoop102改变后hadoop103也会改变。
mysql: [Warning] Using a password on the command line interface can be insecure.
ha1

mysql -hhadoop103 -uroot -p1234 -se "show databases;"|grep 'ha1'
mysql: [Warning] Using a password on the command line interface can be insecure.
ha1

# 尝试关闭hadoop102的mysql，看下虚拟IP是否能使用。
ssh hadoop102 systemctl stop mysqld
ssh hadoop102 systemctl status keepalived|grep 'Active'
Active: inactive (dead) since Thu 2020-07-02 16:07:16 CST; 1min 35s ago

mysql -hhadoop103 -uroot -p1234 -se "show databases;"
mysql -h192.168.200.100 -uroot -p1234 -se "show databases;" # 隔4秒左右执行
```

补充： mysql和keepalived服务都是开机自启动，keepalived服务一启动就需要向mysql的3306端口发送心跳，所以需要保证在开机自启动时，keepalived一定要在mysql启动之后再启动。如何查看一个自启动服务在开机时的启动顺序？所有自启动的开机服务，都会在/etc/init.d下生成一个启动脚本。例如mysql的开机自启动脚本就在 /etc/init.d/mysql。chkconfig: 2345(启动级别，-代表全级别) 64(开机的启动的顺序，号小的先启动) 36(关机时服务停止的顺序) 。

上面是centos6的自启服务顺序概述，centos7待了解，可以直接把keepalived服务开机后重启一下，暴力解决。
