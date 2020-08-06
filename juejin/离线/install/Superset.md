官网：http://superset.apache.org/

概述：Apache Superset是一个开源的、现代的、轻量级BI分析工具，能够对接多种数据源、拥有丰富的图标展示形式、支持自定义仪表盘，且拥有友好的用户界面，十分易用。一个由Python语言编写的Web应用。
应用场景：由于Superset能够对接常用的大数据分析工具，如Hive、Kylin、Druid等，且支持自定义仪表盘，故可作为数仓的可视化工具。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56e25a8ba9a041289b84e31644b7fb57~tplv-k3u1fbpfcp-zoom-1.image)

Superset是由Python语言编写的Web应用，必须要求**Python3.6**的环境。系统一般自带Python2.7，某些系统资源依赖Python2.7，为避免出现问题不要卸载Python2.7。conda是一个开源的包、环境管理器，可以用于在同一个机器上安装不同Python版本的软件包及其依赖，并能够在不同的Python环境之间切换，Anaconda包括Conda、Python以及一大堆安装好的工具包，比如：numpy、pandas等；Miniconda包括Conda、Python。此处不需要如此多的工具包，故选择MiniConda。

# 安装配置

集群规划

| 服务名称 | 子服务 | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ------ | ----------------- | ----------------- | ----------------- |
| Superset |        | √                 |                   |                   |

python安装

```
cd /opt/software
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh # 下载Miniconda（Python3版本）
bash Miniconda3-latest-Linux-x86_64.sh # 安装，安装下面操作
Please, press ENTER to continue
>>> 空格
Do you accept the license terms? [yes|no]
[no] >>> yes
Miniconda3 will now be installed into this location:
/home/admin/miniconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/admin/miniconda3] >>> /opt/module/miniconda3 # 选择安装路径
Do you wish the installer to initialize Miniconda3
by running conda init? [yes|no]
[no] >>> yes
Thank you for installing Miniconda3! #出现这个表示成功

source ~/.bashrc # 加载环境变量配置文件，使之生效，conda在初始化的时候已经把相关的环境配置到bashrc里面
(base) [admin@hadoop102 ~]$ python -V#前面的bash表示已经进入conda创建的一个虚拟环境中
Python 3.8.3

# Miniconda安装完成后，每次打开终端都会激活其默认的base环境/(base) ，可通过以下命令，禁止激活默认base环境。执行完毕重新打开窗口
conda config --set auto_activate_base false
```

创建Python3.6环境
conda环境管理常用命令
创建环境：conda create -n env_name
查看所有环境：conda info --envs
删除一个环境：conda remove -n env_name --all

进入环境：conda activate env_name

退出环境：conda deactivate env_name

```
# 配置conda国内镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --set show_channel_urls yes
conda create --name superset python=3.6 # 创建Python3.6环境
conda activate superset # 激活superset环境，效果如：(superset) [admin@hadoop102 ~]$
python -V # 查看python版本
conda deactivate # 退出当前环境，前面的(superset) 会去掉。
```

Superset部署

gunicorn命令说明： 
--workers：指定进程个数
--timeout：worker进程超时时间，超时会自动重启
--bind：绑定本机地址，即为Superset访问地址
--daemon：后台运行

```
conda activate superset # 进入superset环境，前面带(superset)，下面的所有操作都在superset环境里进行
sudo yum install -y python-setuptools gcc gcc-c++ libffi-devel python-devel python-pip python-wheel openssl-devel cyrus-sasl-devel openldap-devel # 安装依赖
# 安装（更新）setuptools和pip。pip是python的包管理工具，可以和centos中的yum类比
pip install --upgrade setuptools pip -i https://pypi.douban.com/simple/
# 安装Supetset，-i的作用是指定镜像，这里选择国内镜像，-i指定镜像地址
pip install apache-superset -i https://pypi.douban.com/simple/ # 出现Successfully installed ...表示安装成功
superset db upgrade # 初始化Supetset数据库
export FLASK_APP=superset # 创建管理员用户，flask是一个python web框架，Superset使用的就是flask
flask fab create-admin # fab，与Makefiles类似，是一个能够在远程服务器上执行命令的Python工具
# 出现选项全部回车，Password和Repeat for confirmation（确认密码）都写为123456，最后出现下面语句表示成功
Recognized Database Authentications.
Admin User admin created.

superset init # Superset初始化
# 安装gunicorn，gunicorn是一个Python Web Server，类似java的Tomcat
pip install gunicorn -i https://pypi.douban.com/simple/
gunicorn --workers 5 --timeout 120 --bind hadoop102:8787  "superset.app:create_app()" --daemon # 启动
ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | xargs kill -9 # 停止
conda deactivate # 退出superset环境，个人建议开两个窗口，一个进入superset环境不退出，方便使用。
访问http://hadoop102:8787，并使用创建的管理员账号秘密admin/123456登录
```

# 使用之数据源配置

```
# 安装依赖，对接不同的数据源，需安装不同的依赖，以下地址为官网说明。http://superset.apache.org/installation.html#database-dependencies
conda install mysqlclient
# 重启Superset
ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | xargs kill -9
gunicorn --workers 5 --timeout 120 --bind hadoop102:8787  "superset.app:create_app()" --daemon
```

访问http://hadoop102:8787数据源配置

0.在mysql的gmall_report数据库执行下面sql

```sql
DROP TABLE IF EXISTS `ads_area_topic`;
CREATE TABLE `ads_area_topic`  (
  `dt` date NOT NULL,
  `id` int(11) DEFAULT NULL,
  `province_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
  `area_code` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
  `iso_code` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `region_id` int(11) DEFAULT NULL,
  `region_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
  `order_day_count` bigint(255) DEFAULT NULL,
  `order_day_amount` double(255, 2) DEFAULT NULL,
  `payment_day_count` bigint(255) DEFAULT NULL,
  `payment_day_amount` double(255, 2) DEFAULT NULL,
  PRIMARY KEY (`dt`, `iso_code`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '地区主题表' ROW_FORMAT = Compact;

-- ----------------------------
-- Records of ads_area_topic
-- ----------------------------
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 1, '北京', '110000', 'CN-11', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 2, '天津市', '120000', 'CN-12', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 5, '河北', '130000', 'CN-13', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 3, '山西', '140000', 'CN-14', 1, '华北', 1, 15927.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 4, '内蒙古', '150000', 'CN-15', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 17, '辽宁', '210000', 'CN-21', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 16, '吉林', '220000', 'CN-22', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 15, '黑龙江', '230000', 'CN-23', 3, '东北', 1, 1569.00, 1, 1569.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 6, '上海', '310000', 'CN-31', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 7, '江苏', '320000', 'CN-32', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 8, '浙江', '330000', 'CN-33', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 9, '安徽', '340000', 'CN-34', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 10, '福建', '350000', 'CN-35', 2, '华东', 1, 297.00, 1, 297.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 11, '江西', '360000', 'CN-36', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 12, '山东', '370000', 'CN-37', 2, '华东', 1, 500.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 23, '河南', '410000', 'CN-41', 4, '华中', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 24, '湖北', '420000', 'CN-42', 4, '华中', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 25, '湖南', '430000', 'CN-43', 4, '华中', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 26, '广东', '440000', 'CN-44', 5, '华南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 27, '广西', '450000', 'CN-45', 5, '华南', 1, 14594.00, 1, 14594.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 28, '海南', '460000', 'CN-46', 5, '华南', 1, 7371.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 13, '重庆', '500000', 'CN-50', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 31, '四川', '510000', 'CN-51', 6, '西南', 1, 233.00, 1, 230.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 32, '贵州', '520000', 'CN-52', 6, '西南', 1, 3124.00, 1, 3124.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 33, '云南', '530000', 'CN-53', 6, '西南', 1, 1558.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 34, '西藏', '540000', 'CN-54', 6, '西南', 1, 459.00, 1, 459.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 18, '陕西', '610000', 'CN-61', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 19, '甘肃', '620000', 'CN-62', 7, '西北', 2, 26509.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 20, '青海', '630000', 'CN-63', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 21, '宁夏', '640000', 'CN-64', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 22, '新疆', '650000', 'CN-65', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 14, '台湾', '710000', 'CN-71', 2, '华东', 0, 0.00, 1, 17352.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 29, '香港', '810000', 'CN-91', 5, '华南', 0, 0.00, 1, 3116.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-10', 30, '澳门', '820000', 'CN-92', 5, '华南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 1, '北京', '110000', 'CN-11', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 2, '天津市', '120000', 'CN-12', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 5, '河北', '130000', 'CN-13', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 3, '山西', '140000', 'CN-14', 1, '华北', 1, 15927.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 4, '内蒙古', '150000', 'CN-15', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 17, '辽宁', '210000', 'CN-21', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 16, '吉林', '220000', 'CN-22', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 15, '黑龙江', '230000', 'CN-23', 3, '东北', 1, 1569.00, 1, 1569.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 6, '上海', '310000', 'CN-31', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 7, '江苏', '320000', 'CN-32', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 8, '浙江', '330000', 'CN-33', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 9, '安徽', '340000', 'CN-34', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 10, '福建', '350000', 'CN-35', 2, '华东', 1, 297.00, 1, 297.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 11, '江西', '360000', 'CN-36', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 12, '山东', '370000', 'CN-37', 2, '华东', 1, 500.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 23, '河南', '410000', 'CN-41', 4, '华中', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 24, '湖北', '420000', 'CN-42', 4, '华中', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 25, '湖南', '430000', 'CN-43', 4, '华中', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 26, '广东', '440000', 'CN-44', 5, '华南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 27, '广西', '450000', 'CN-45', 5, '华南', 1, 14594.00, 1, 14594.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 28, '海南', '460000', 'CN-46', 5, '华南', 1, 7371.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 13, '重庆', '500000', 'CN-50', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 31, '四川', '510000', 'CN-51', 6, '西南', 1, 233.00, 1, 230.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 32, '贵州', '520000', 'CN-52', 6, '西南', 1, 3124.00, 1, 3124.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 33, '云南', '530000', 'CN-53', 6, '西南', 1, 1558.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 34, '西藏', '540000', 'CN-54', 6, '西南', 1, 459.00, 1, 459.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 18, '陕西', '610000', 'CN-61', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 19, '甘肃', '620000', 'CN-62', 7, '西北', 2, 26509.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 20, '青海', '630000', 'CN-63', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 21, '宁夏', '640000', 'CN-64', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 22, '新疆', '650000', 'CN-65', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 14, '台湾', '710000', 'CN-71', 2, '华东', 0, 0.00, 1, 17352.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 29, '香港', '810000', 'CN-91', 5, '华南', 0, 0.00, 1, 3116.00);
INSERT INTO `ads_area_topic` VALUES ('2020-03-11', 30, '澳门', '820000', 'CN-92', 5, '华南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 1, '北京', '110000', 'CN-11', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 2, '天津市', '120000', 'CN-12', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 5, '河北', '130000', 'CN-13', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 3, '山西', '140000', 'CN-14', 1, '华北', 2, 16376.00, 2, 16376.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 4, '内蒙古', '150000', 'CN-15', 1, '华北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 17, '辽宁', '210000', 'CN-21', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 16, '吉林', '220000', 'CN-22', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 15, '黑龙江', '230000', 'CN-23', 3, '东北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 6, '上海', '310000', 'CN-31', 2, '华东', 1, 6634.00, 1, 6634.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 7, '江苏', '320000', 'CN-32', 2, '华东', 1, 17816.00, 1, 17816.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 8, '浙江', '330000', 'CN-33', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 9, '安徽', '340000', 'CN-34', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 10, '福建', '350000', 'CN-35', 2, '华东', 1, 14297.00, 1, 14297.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 11, '江西', '360000', 'CN-36', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 12, '山东', '370000', 'CN-37', 2, '华东', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 23, '河南', '410000', 'CN-41', 4, '华中', 1, 6914.00, 1, 6914.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 24, '湖北', '420000', 'CN-42', 4, '华中', 1, 3107.00, 1, 3107.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 25, '湖南', '430000', 'CN-43', 4, '华中', 1, 10306.00, 1, 10306.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 26, '广东', '440000', 'CN-44', 5, '华南', 1, 2460.00, 1, 2460.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 27, '广西', '450000', 'CN-45', 5, '华南', 1, 675.00, 1, 675.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 28, '海南', '460000', 'CN-46', 5, '华南', 3, 15174.00, 2, 10495.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 13, '重庆', '500000', 'CN-50', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 31, '四川', '510000', 'CN-51', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 32, '贵州', '520000', 'CN-52', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 33, '云南', '530000', 'CN-53', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 34, '西藏', '540000', 'CN-54', 6, '西南', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 18, '陕西', '610000', 'CN-61', 7, '西北', 2, 6228.00, 2, 6228.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 19, '甘肃', '620000', 'CN-62', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 20, '青海', '630000', 'CN-63', 7, '西北', 2, 10204.00, 2, 10204.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 21, '宁夏', '640000', 'CN-64', 7, '西北', 1, 295.00, 1, 295.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 22, '新疆', '650000', 'CN-65', 7, '西北', 0, 0.00, 0, 0.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 14, '台湾', '710000', 'CN-71', 2, '华东', 1, 1569.00, 1, 1569.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 29, '香港', '810000', 'CN-91', 5, '华南', 2, 9189.00, 1, 4267.00);
INSERT INTO `ads_area_topic` VALUES ('2020-08-05', 30, '澳门', '820000', 'CN-92', 5, '华南', 1, 7546.00, 1, 7546.00);

SET FOREIGN_KEY_CHECKS = 1;
```

1.点击Sources/Databases，再点击+

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6267a2b0f6824bc1a64e1b13b959eee8~tplv-k3u1fbpfcp-zoom-1.image)

2.点击填写Database及SQL Alchemy URI。注：SQL Alchemy URI编写规范：mysql://账号:密码@IP/数据库名称，如mysql://root:1234@hadoop102/gmall_report?charset=utf8；点击Test Connection，出现“Seems Ok！”提示即表示连接成功；最后点击保存。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29d3889b8ab5420da8f840439d73c42c~tplv-k3u1fbpfcp-zoom-1.image)

3.点击Sources/Tables，再点击+号，选择数据库和填写表名。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e09c7c5d4dea44fd8c9d90e57e92f2a4~tplv-k3u1fbpfcp-zoom-1.image)

# 使用之仪表盘制作

1.点击Dashboards，选择+号；title值随意填写然后点击保存。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43a954c3c8224d83a46270954f12f489~tplv-k3u1fbpfcp-zoom-1.image)

2.点击Charts，选择+号；选择一张表作为数据源，然后点击Table选择可视化/图表类型，选择Country Map，然后点击Create new chart。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7831ecfc5d04d19a429c9b1f352bbcc~tplv-k3u1fbpfcp-zoom-1.image)

3.语言切换为中文，方便处理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4441bff724f4050a9734c8766018347~tplv-k3u1fbpfcp-zoom-1.image)



4.解决Control labeled **"地区/省/部门ISO3166-2代码"** 不能为空，把地区/省/部门ISO3166-2代码的值改为iso_code，设置下面选项，点击run query出来下面结果。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adb9f50fdc3c49b6bc7e6a03c7b346e9~tplv-k3u1fbpfcp-zoom-1.image)

5.保存看板，看板名字为“全国各省份订单个数”

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/684956b2e37f4a27983f54192ada93cc~tplv-k3u1fbpfcp-zoom-1.image)

6.保存并转到看板会访问http://hadoop102:8787/superset/dashboard/1/

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/367cc53660ff4c13900a4e9acbfa10ce~tplv-k3u1fbpfcp-zoom-1.image)

# 使用之自动刷新

访问仪表盘http://hadoop102:8787/superset/dashboard/1/，查看所有保存的chart图表/看板，设置自动刷新10秒一次。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c65f8b2a780442faf071ed18710a4dc~tplv-k3u1fbpfcp-zoom-1.image)

选择mysql的gmall_report数据库执行下面语句，对order_day_count每日支付总数做稍微修改。

```sql
UPDATE ads_area_topic SET order_day_count = 45 where dt = '2020-03-10'
and area_code = 120000
and region_id = 1
and region_name = '华北'
and province_name = '天津市';
```

等待10秒再看仪表盘http://hadoop102:8787/superset/dashboard/1/

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a262e8691cbd42099934bf16340b0242~tplv-k3u1fbpfcp-zoom-1.image)

# 1