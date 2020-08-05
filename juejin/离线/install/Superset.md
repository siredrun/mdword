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

# 使用