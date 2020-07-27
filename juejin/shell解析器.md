引言：更多相关请看 [个人系列](https://juejin.im/post/5e3d90ba518825494066f491)  
大数据程序员学习Shell的目的：  
1）需要看懂运维人员编写的Shell程序。  
2）偶尔会编写一些简单Shell程序来管理集群、提高开发效率。  
shell的中文意思就是贝壳，其实比较类似于我们内核的壳。简而言之就是只要能够操作应用程序的接口都能够称为SHELL。狭义的shell指的是命令行方面的软件，广义的SHELL则包括图形界面。

![](https://user-gold-cdn.xitu.io/2020/3/14/170d68c4ea130e57?w=604&h=308&f=png&s=20636)
# shell解析器
（1）Linux提供的Shell解析器有：  
命令：cat /etc/shells
```
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/tcsh
/bin/csh
```
重要的是sh和bash。  
（2）bash和sh的关系  
使用sh调用执行脚本，相当于打开了bash的POSIX标准模式 （等效于bash的 --posix 参数）。一般来讲，sh是bash的“子集”。其实sh调用的就是bash。  
（3）Centos默认的解析器是bash  
命令：echo $SHELL
```shell
/bin/bash
```
# 脚本入门

脚本格式：通常以#!/bin/bash开头（指定解析器）

## helloworld

（1）需求：创建一个Shell脚本，输出helloworld

（2）案例实操：

命令： vim helloworld.sh建立脚本文件，内容如下

```shell
#!/bin/bash

echo "helloworld"
```

（3）脚本的常用执行方式

第一种：采用bash或sh+脚本的相对路径或绝对路径（不用赋予脚本+x权限）

如：sh helloworld.sh 或 sh /root/aa/helloworld.sh；bash helloworld.sh 或 bash /root/aa/helloworld.sh

第二种：采用输入脚本的绝对路径或相对路径执行脚本（必须具有可执行权限+x）

（a）首先要赋予helloworld.sh 脚本的+x权限

命令：chmod 777 helloworld.sh 或chmod +x helloworld.sh

（b）执行脚本

相对路径./helloworld.sh  和 绝对路径/root/aa/helloworld.sh。

==注意：第一种执行方法，本质是bash解析器帮你执行脚本，所以脚本本身不需要执行权限。第二种执行方法，本质是脚本需要自己执行，所以需要执行权限。==

## 多命令处理

（1）需求：在/root下创建一个a.txt,在a.txt文件中增加“I love cls”。

（2）案例实操：vim batch.sh

在batch.sh中输入如下内容

```shell
#!/bin/bash

 

cd /root

touch cls.txt

echo "I love cls" >>cls.txt
```

执行脚本就能完成需求。

# 变量

## 系统变量

常用系统变量：

```shell
$HOME、$PWD、$SHELL、$USER
```

使用：echo加系统变量，如 echo $HOME

## 自定义变量

1．基本语法

（1）定义变量：变量=值 

（2）撤销变量：unset 变量

（3）声明静态变量：readonly变量，注意：不能unset

2．变量定义规则

（1）变量名称可以由字母、数字和下划线组成，但是不能以数字开头，环境变量名建议大写。

（2）等号两侧不能有空格

（3）在bash中，变量默认类型都是字符串类型，无法直接进行数值运算。

（4）变量的值如果有空格，需要使用双引号或单引号括起来。

3．案例实操

（1）定义变量A

```shell
[root@yxc ~]# a=5
[root@yxc ~]# echo $a
5
```

（2）给变量A重新赋值

```shell
[root@yxc ~]# a=8
[root@yxc ~]# echo $a
8
```

​    （3）撤销变量A，撤销后为空

```shell
[root@yxc ~]# unset a
[root@yxc ~]# echo $a

```

​    （4）声明静态的变量B=2，不能unset

```shell
[root@yxc ~]# readonly a=2
[root@yxc ~]# echo $a
2
[root@yxc ~]# a=3
-bash: a: readonly variable
```

​    （5）在bash中，变量默认类型都是字符串类型，无法直接进行数值运算

```powershell
[root@yxc ~]# C=1+2
[root@yxc ~]# echo $C
1+2
```

（6）变量的值如果有空格，需要使用双引号或单引号括起来

```powershell
[root@yxc ~]# D=I love banzhang
-bash: world: command not found
[root@yxc ~]# D="I love banzhang"
[root@yxc ~]# echo $D
I love banzhang
```

（7）可把变量提升为全局环境变量，可供其他Shell程序使用。export 变量名

命令：vim helloworld.sh建立文件并在helloworld.sh文件中增加echo $B

```shell
\#!/bin/bash
echo "helloworld"
echo $B
```

执行脚本并没有打印输出变量B的值。

```shell
[root@yxc ~]# export B
[root@yxc ~]# ./helloworld.sh 
helloworld
```

## 特殊变量：$n

基本语法： $n  （功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十以上的参数，十以上的参数需要用大括号包含，如${10}）。

案例实操

（1）输出该脚本文件名称、输入参数1和输入参数2 的值

```shell
[root@yxc ~]# touch parameter.sh 
[root@yxc ~]# vim parameter.sh
\#!/bin/bash
echo "$0 $1  $2"
[root@yxc ~]# chmod 777 parameter.sh
[root@yxc ~]# ./parameter.sh cls xz
./parameter.sh cls  xz
```

## 特殊变量：$#

基本语法：$#  （功能描述：获取所有输入参数个数，常用于循环）。

案例实操

（1）获取输入参数的个数

```
[root@yxc ~]#  vim parameter.sh
\#!/bin/bash
echo "$0 $1  $2"
echo $#
[root@yxc ~]#  chmod 777 parameter.sh
[root@yxc ~]#  ./parameter.sh cls xz
parameter.sh cls xz
2
```



## 特殊变量：$*、$@

1．基本语法

    $* （功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体）
    $@ （功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待）

2．案例实操

（1）打印输入的所有参数

```
[root@yxc ~]# vim parameter.sh
\#!/bin/bash
echo "$0 $1  $2"
echo $#
echo $*
echo $@

[root@yxc ~]# bash parameter.sh 1 2 3
parameter.sh 1  2
3
1 2 3
1 2 3
```



## 特殊变量：$？

基本语法：$？ （功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。）

2．案例实操

    判断helloworld.sh脚本是否正确执行

```
[root@yxc ~]# ./helloworld.sh 
hello world
[root@yxc ~]# echo $?
0
```

# 运算符		

基本语法
（1）“$((运算式))”或“$[运算式]”
（2）expr  + , - , \*,  /,  %    加，减，乘，除，取余
==注意：expr运算符间要有空格==
案例实操： 
（1）计算3+2的值

```
[root@yxc ~]# expr 2 + 3
5
```

（2）计算3-2的值

```
[root@yxc ~]# expr 3 - 2 
1
```

（3）计算（2+3）X4的值
（a）expr一步完成计算

```
[root@yxc ~]# expr `expr 2 + 3` \* 4
20
```

（b）采用\$[运算式]方式

```
[root@yxc ~]# S=$[(2+3)*4]
[root@yxc ~]# echo $S
```

# 条件判断

1．基本语法
[ condition ]（注意condition前后要有空格）
注意：条件非空即为true，[ atguigu ]返回true，[] 返回false。

常用判断条件
（1）两个整数之间比较
= 字符串比较
-lt 小于（less than）			-le 小于等于（less equal）
-eq 等于（equal）				-gt 大于（greater than）
-ge 大于等于（greater equal）	-ne 不等于（Not equal）
（2）按照文件权限进行判断
-r 有读的权限（read）			-w 有写的权限（write）
-x 有执行的权限（execute）
（3）按照文件类型进行判断
-f 文件存在并且是一个常规的文件（file）
-e 文件存在（existence）		-d 文件存在并是一个目录（directory）
3．案例实操

注意：0为false，1为true

（1）23是否大于等于22

```
[root@yxc ~]# [ 23 -ge 22 ]
[root@yxc ~]# echo $?
0

```

（2）helloworld.sh是否具有写权限

```
[root@yxc ~]# [ -w helloworld.sh ]
[root@yxc ~]# echo $?
0
```

（3）/home/cls.txt目录中的文件是否存在

```
[root@yxc ~]# [ -e /home/atguigu/cls.txt ]
[root@yxc ~]# echo $?
1
```

（4）多条件判断（&& 表示前一条命令执行成功时，才执行后一条命令，|| 表示上一条命令执行失败后，才执行下一条命令）

```
[root@yxc ~]$ [ condition ] && echo OK || echo notok
OK
[root@yxc ~]# [ condition ] && [ ] || echo notok
notok
```

# 流程控制（重点）

## if 判断

1．基本语法
if [ 条件判断式 ];then 
  程序 
fi 
或者 
if [ 条件判断式 ] 
  then 
    程序 
fi

完整结构
if [条件表达式]
	then
		程序
elif [条件表达式]
	then
		程序
else
	程序
fi
	注意事项：
（1）[ 条件判断式 ]，中括号和条件判断式之间必须有空格
（2）if后要有空格
2．案例实操
（1）输入一个数字，如果是1，则输出banzhang zhen shuai，如果是2，则输出cls zhen mei，如果是其它，什么也不输出。

```
[root@yxc ~]#  touch if.sh
[root@yxc ~]#  vim if.sh

#!/bin/bash

if [ $1 -eq "1" ]
then
        echo "banzhang zhen shuai"
elif [ $1 -eq "2" ]
then
        echo "cls zhen mei"
fi

[root@yxc ~]#  chmod 777 if.sh 
[root@yxc ~]#  ./if.sh 1
banzhang zhen shuai
```



## case 语句

1．基本语法
case $变量名 in 
  "值1"） 
    如果变量的值等于值1，则执行程序1 
    ;; 
  "值2"） 
    如果变量的值等于值2，则执行程序2 
    ;; 
  …省略其他分支… 
  *） 
    如果变量的值都不是以上的值，则执行此程序 
    ;; 
esac
注意事项：
1)	case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。
2)	双分号“;;”表示命令序列结束，相当于java中的break。
3)	最后的“*）”表示默认模式，相当于java中的default。
2．案例实操
（1）输入一个数字，如果是1，则输出banzhang，如果是2，则输出cls，如果是其它，输出renyao。

```
[root@yxc ~]#  touch case.sh
[root@yxc ~]#  vim case.sh

!/bin/bash

case $1 in
"1")
        echo "banzhang"
;;

"2")
        echo "cls"
;;
*)
        echo "renyao"
;;
esac

[root@yxc ~]#  chmod 777 case.sh
[root@yxc ~]#  ./case.sh 2
renyao
```



##  for 循环

1．基本语法1
	for (( 初始值;循环控制条件;变量变化 )) 
  do 
    程序 
  done
2．案例实操
（1）从1加到100

```shell
[root@yxc ~]#  touch for1.sh
[root@yxc ~]#  vim for1.sh

#!/bin/bash

s=0
for((i=0;i<=100;i++))
do
        s=$[$s+$i]
done
echo $s

[root@yxc ~]#  chmod 777 for1.sh 
[root@yxc ~]#  ./for1.sh 
5050
```

3．基本语法2
for 变量 in 值1 值2 值3… 
  do 
    程序 
  done
4．案例实操
（1）打印所有输入参数

```
[root@yxc ~]#  touch for2.sh
[root@yxc ~]#  vim for2.sh

#!/bin/bash
#打印数字

for i in $*
    do
      echo "ban zhang love $i "
    done

[root@yxc ~]#  chmod 777 for2.sh 
[root@yxc ~]#  bash for2.sh cls xz bd
ban zhang love cls
ban zhang love xz
ban zhang love bd
```

（2）比较$*和$@区别
（a）$*和$@都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以$1 $2 …$n的形式输出所有参数。

```
[root@yxc ~]#  touch for.sh
[root@yxc ~]#  vim for.sh

#!/bin/bash 

for i in $*
do
      echo "ban zhang love $i "
done

for j in $@
do      
        echo "ban zhang love $j"
done

[root@yxc ~]#  bash for.sh cls xz bd
ban zhang love cls 
ban zhang love xz 
ban zhang love bd 
ban zhang love cls
ban zhang love xz
ban zhang love bd
```

（b）当它们被双引号“”包含时，“$*”会将所有的参数作为一个整体，以“$1 $2 …$n”的形式输出所有参数；“$@”会将各个参数分开，以“$1” “$2”…”$n”的形式输出所有参数。

```
[root@yxc ~]#  vim for.sh

#!/bin/bash 

for i in "$*" 
#$*中的所有参数看成是一个整体，所以这个for循环只会循环一次 
        do 
                echo "ban zhang love $i"
        done 

for j in "$@" 
#$@中的每个参数都看成是独立的，所以“$@”中有几个参数，就会循环几次 
        do 
                echo "ban zhang love $j" 
done

[root@yxc ~]#  chmod 777 for.sh
[root@yxc ~]#  bash for.sh cls xz bd
ban zhang love cls xz bd
ban zhang love cls
ban zhang love xz
ban zhang love bd
```



## while 循环

1．基本语法
while [ 条件判断式 ] 
  do 
    程序
  done
2．案例实操
（1）从1加到100

```
[root@yxc ~]#  touch while.sh
[root@yxc ~]#  vim while.sh

#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done

echo $s

[root@yxc ~]#  chmod 777 while.sh 
[root@yxc ~]#  ./while.sh 
5050
```

# read读取控制台输入

1．基本语法
read(选项)(参数)
选项：
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）。
参数
	变量：指定读取值的变量名
2．案例实操
（1）提示7秒内，读取控制台输入的名称

```
[root@yxc ~]#  touch read.sh
[root@yxc ~]#  vim read.sh

#!/bin/bash

read -t 7 -p "Enter your name in 7 seconds " NAME
echo $NAME

[root@yxc ~]#  ./read.sh 
Enter your name in 7 seconds xiaoze
xiaoze
```

注意：执行后请在七秒内输入并按enter键打印出输入的内容。

# 函数

## 系统函数

1.basename基本语法
basename [string / pathname] [suffix] （功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。
选项：suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。
2.案例实操
（1）截取该/home/atguigu/banzhang.txt路径的文件名称

```
[atguigu@hadoop101 datas]$ basename /home/banzhang.txt 
banzhang.txt
[atguigu@hadoop101 datas]$ basename /home/banzhang.txt .txt
banzhang
```

3.dirname基本语法

dirname 文件绝对路径。功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））
4.案例实操
（1）获取banzhang.txt文件的路径

```
[root@yxc ~]#  dirname /home/banzhang.txt 
/home
```

## 自定义函数

1.基本语法

[ function ] funname[()]
{
    Action;
    [return int;]
}

funname	

2.经验技巧
（1）必须在调用函数地方之前，先声明函数，shell脚本是逐行运行。不会像其它语言一样先编译。
（2）函数返回值，只能通过\$?系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)
3．案例实操
（1）计算两个输入参数的和

```
[root@yxc ~]#  touch fun.sh
[root@yxc ~]#  vim fun.sh

#!/bin/bash
function sum()
{
    s=0
    s=$[ $1 + $2 ]
    echo "$s"
}

read -p "Please input the number1: " n1;
read -p "Please input the number2: " n2;
sum $n1 $n2;

[root@yxc ~]#  chmod 777 fun.sh
[root@yxc ~]#  ./fun.sh 
Please input the number1: 2
Please input the number2: 5
7

注意：
#!/bin/bash
function sum(){
        s=0
        s=$[$1 + $2] 这里的参数为sum函数的参数
        echo $s
}
sum $1 $2 
```

# Shell工具（重点）

## cut

cut的工作就是“剪”，具体的说就是在文件中负责剪切数据用的。cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出。

1.基本用法

cut [选项参数]  filename

说明：默认分隔符是制表符

2.选项参数说明

表1-55

| 选项参数 | 功能                         |
| -------- | ---------------------------- |
| -f       | 列号，提取第几列             |
| -d       | 分隔符，按照指定分隔符分割列 |

3.案例实操

（0）数据准备

```
[root@yxc ~]#  touch cut.txt
[root@yxc ~]#  vim cut.txt
dong shen
guan zhen
wo  wo
lai  lai
le  le
```

（1）切割cut.txt第一列

```
[root@yxc ~]#  cut -d " " -f 1 cut.txt 
dong
guan
wo
lai
le
```

（2）切割cut.txt第二、三列

```
[root@yxc ~]#  cut -d " " -f 2,3 cut.txt 
shen
zhen
wo
lai
le
```

（3）在cut.txt文件中切割出guan

```
[root@yxc ~]#  cat cut.txt | grep "guan" | cut -d " " -f 1
guan
```

（4）选取系统PATH变量值，第2个“：”开始后的所有路径：

```
[root@yxc ~]# echo $PATH
/opt/java/bin:/opt/maven/bin:/opt/git/bin:/usr/local/lib/erlang/bin:/usr/local/rabbitmq//sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

3-表示第三列以后的所有列

```
[root@yxc ~]# echo $PATH |cut -d ":" -f 3-
/opt/git/bin:/usr/local/lib/erlang/bin:/usr/local/rabbitmq//sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

（5）切割ifconfig 后打印的IP地址

```
[root@yxc ~]#  ifconfig eth0 | grep "inet addr" | cut -d: -f 2 | cut -d" " -f1
192.168.1.102
```



## sed

sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。

1.基本用法

sed [选项参数]  ‘command’ filename

2.选项参数说明

表1-56

| 选项参数 | 功能                                  |
| -------- | ------------------------------------- |
| -e       | 直接在指令列模式上进行sed的动作编辑。 |

3.命令功能描述

表1-57

| 命令 | 功能描述                              |
| ---- | ------------------------------------- |
| *a*  | 新增，a的后面可以接字串，在下一行出现 |
| d    | 删除                                  |
| s    | 查找并替换                            |

4.案例实操

（0）数据准备

```
[root@yxc ~]# touch sed.txt
[root@yxc ~]# vim sed.txt
dong shen
guan zhen
wo wo
lai lai
le le
```

（1）将“mei nv”这个单词插入到sed.txt第二行下，打印。

```
[root@yxc ~]# sed '2a mei nv' sed.txt 
dong shen
guan zhen
mei nv
wo wo
lai lai
le le
[root@yxc ~]# cat sed.txt 
dong shen
guan zhen
wo wo
lai lai
le le
```

==注意：文件并没有改变==

（2）删除sed.txt文件所有包含wo的行

```
[root@yxc ~]# sed '/wo/d' sed.txt
dong shen
guan zhen
lai lai
le le
```

（3）将sed.txt文件中wo替换为ni

```
[root@yxc ~]# sed 's/wo/ni/g' sed.txt 
dong shen
guan zhen
ni ni
lai lai
le le
```

==注意：‘g’表示global，全部替换==

（4）将sed.txt文件中的第二行删除并将wo替换为ni

```
[root@yxc ~]# sed -e '2d' -e 's/wo/ni/g' sed.txt 
dong shen
ni ni
lai lai
le le
```

**注意：如果执行多条相同的命令，不加-e只会执行一个**

## awk

一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

1.基本用法

awk [选项参数] ‘pattern1{action1} pattern2{action2}...’ filename

pattern：表示AWK在数据中查找的内容，就是匹配模式

action：在找到匹配内容时所执行的一系列命令

**注意：这里的模式匹配需要使用单引号，因为双引号会对正则的一些特殊符号处理**

2.选项参数说明

表1-55

| 选项参数 | 功能                 |
| -------- | -------------------- |
| -F       | 指定输入文件折分隔符 |
| -v       | 赋值一个用户定义变量 |

3.案例实操

（0）数据准备

```
[root@yxc ~]# sudo cp /etc/passwd ./
```

（1）搜索passwd文件以root关键字开头的所有行，使用冒号切分，并输出该行的第7列。

```
[root@yxc ~]# awk -F: '/^root/{print $7}' passwd 
/bin/bash
```

（2）搜索passwd文件以root关键字开头的所有行，使用冒号切分，并输出该行的第1列和第7列，中间以“，”号分割。

```
[root@yxc ~]# awk -F: '/^root/{print $1","$7}' passwd 
root,/bin/bash
```

==注意：只有匹配了pattern的行才会执行action==

（3）对/etc/passwd用冒号切分后，只显示第一列和第七列，然后以逗号分割，且在所有行前面添加列名user，shell在最后一行添加"banzhang，/bin/zuishuai"。

```
[root@yxc ~]# awk -F : 'BEGIN{print "user, shell"} {print $1","$7} END{print "banzhang,/bin/zuishuai"}' passwd
user, shell
root,/bin/bash
bin,/sbin/nologin
...
banzhang,/bin/zuishuai
```

注意：BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行。

（4）将passwd文件中的用户id增加数值1并输出 **注意：i取值的时候不需要加$**

```
[root@yxc ~]# awk -v i=1 -F: '{print $3+**i**}' passwd
1
2
3
4
```

4.awk的内置变量

表1-56

| 变量     | 说明                                   |
| -------- | -------------------------------------- |
| FILENAME | 文件名                                 |
| NR       | 已读的记录数(行数)                     |
| NF       | 浏览记录的域的个数（切割后，列的个数） |

5.案例实操

（1）统计使用冒号切分passwd后，文件名，每行的行号，每行的列数

```
[root@yxc ~]# awk -F: '{print "filename:" FILENAME ", linenumber:" NR ",columns:" NF}' passwd 
filename:passwd, linenumber:1,columns:7
filename:passwd, linenumber:2,columns:7
filename:passwd, linenumber:3,columns:7
```

 （2）切割IP

```
[root@yxc ~]# ifconfig eth0 | grep "inet addr" | awk -F: '{print $2}' | awk -F " " '{print $1}' 

192.168.1.102
```

 （3）查询sed.txt中空行所在的行号**（面试题）**

```
[root@yxc ~]# awk '/^$/{print NR}' sed.txt 

5
```

[**Linux:awk与cut命令的区别**](https://www.cnblogs.com/xwb583312435/p/9040790.html)

**awk 以空格为分割域时,是以单个或多个连续的空格为分隔符的; cut则是以单个空格作为分隔符。**

例如：awk -F " " '{print $2}' cut.txt

## sort

sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出。

1.基本语法

sort(选项)(参数)

表1-57

| 选项 | 说明                     |
| ---- | ------------------------ |
| -n   | 依照数值的大小排序       |
| -r   | 以相反的顺序来排序       |
| -t   | 设置排序时所用的分隔字符 |
| -k   | 指定需要排序的列         |

参数：指定待排序的文件列表

2.案例实操

（0）数据准备

```
[root@yxc ~]# touch sort.txt
[root@yxc ~]# vim sort.txt
aaa:10:1.1
ccc:30:3.3
ddd:40:4.4
bbb:20:2.2
eee:50:5.5
```

（1）   默认排序

```
[root@yxc ~]# sort sort.txt 
aaa:10:1.1
bbb:20:2.2
ccc:30:3.3
ddd:40:4.4
eee:50:5.5
```

（2）   按照“：”分割后的第三列倒序排序。

```
[root@yxc ~]# sort -t : -nrk 3 sort.txt 
eee:50:5.5
ddd:40:4.4
ccc:30:3.3
bbb:20:2.2
aaa:10:1.1
```

# 企业真实面试题（重点）

## 京东

问题1：使用Linux命令查询file1中空行所在的行号
答案：

```
[root@yxc ~]# awk '/^$/{print NR}' sed.txt 
5
```

在实际工作中，过滤空行，空值，404，500等场景比较常见，这个属于数据清洗（ETL）
问题2：有文件chengji.txt内容如下:
张三 40
李四 50
王五 60
使用Linux命令计算第二列的和并输出

```
[root@yxc ~]# cat chengji.txt | awk -F " " '{sum+=$2} END{print sum}'
150
```

## 搜狐&和讯网

问题1：Shell脚本里如何检查一个文件是否存在？如果不存在该如何处理？

```
!/bin/bash

if [ -f file.txt ]; then
   echo "文件存在!"
else
   echo "文件不存在!"
fi
```

## 新浪

问题1：用shell写一个脚本，对文本中无序的一列数字排序

```
[root@CentOS6-2 ~]# cat test.txt
9
8
7
6
5
4
3
2
10
1
[root@CentOS6-2 ~]# sort -n test.txt|awk '{a+=$0;print $0}END{print "SUM="a}'
1
2
3
4
5
6
7
8
9
10
SUM=55
```

## 金和网络

问题1：请用shell脚本写出查找当前文件夹（/home）下所有的文本文件内容中包含有字符”shen”的文件名称

```
[root@yxc ~]# grep -r "shen" /home | cut -d ":" -f 1
/home/root/datas/sed.txt
/home/root/datas/cut.txt
```

