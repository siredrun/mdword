# kylin

集群规划：

| 服务名称 | 子服务 | hadoop102 | hadoop103 | hadoop104 |
| -------- | ------ | --------- | --------- | --------- |
| Kylin    |        | √         |           |           |

## 概述

官网：http://kylin.apache.org/

Apache Kylin是一个开源的分布式分析引擎，提供Hadoop/Spark之上的SQL查询接口及多维分析（OLAP）能力以支持超大规模数据，最初由eBay Inc开发并贡献至开源社区。它能在亚秒内查询巨大的Hive表。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fa927e22d154b9a88bfa0e29aa97f6b~tplv-k3u1fbpfcp-zoom-1.image)

1）REST Server
REST Server是一套面向应用程序开发的入口点，旨在实现针对Kylin平台的应用开发工作。 此类应用程序可以提供查询、获取结果、触发cube构建任务、获取元数据以及获取用户权限等等。另外可以通过Restful接口实现SQL查询。
2）查询引擎（Query Engine）
当cube准备就绪后，查询引擎就能够获取并解析用户查询。它随后会与系统中的其它组件进行交互，从而向用户返回对应的结果。 
3）路由器（Routing）
在最初设计时曾考虑过将Kylin不能执行的查询引导去Hive中继续执行，但在实践后发现Hive与Kylin的速度差异过大，导致用户无法对查询的速度有一致的期望，很可能大多数查询几秒内就返回结果了，而有些查询则要等几分钟到几十分钟，因此体验非常糟糕。最后这个路由功能在发行版中默认关闭。
4）元数据管理工具（Metadata）
Kylin是一款元数据驱动型应用程序。元数据管理工具是一大关键性组件，用于对保存在Kylin当中的所有元数据进行管理，其中包括最为重要的cube元数据。其它全部组件的正常运作都需以元数据管理工具为基础。 Kylin的元数据存储在hbase中。 
5）任务引擎（Cube Build Engine）
这套引擎的设计目的在于处理所有离线任务，其中包括shell脚本、Java API以及Map Reduce任务等等。任务引擎对Kylin当中的全部任务加以管理与协调，从而确保每一项任务都能得到切实执行并解决其间出现的故障。

Kylin的主要特点包括支持SQL接口、支持超大规模数据集、亚秒级响应、可伸缩性、高吞吐率、BI工具集成等。
1）标准SQL接口：Kylin是以标准的SQL作为对外服务的接口。
2）支持超大数据集：Kylin对于大数据的支撑能力可能是目前所有技术中最为领先的。早在2015年eBay的生产环境中就能支持百亿记录的秒级查询，之后在移动的应用场景中又有了千亿记录秒级查询的案例。
3）亚秒级响应：Kylin拥有优异的查询相应速度，这点得益于预计算，很多复杂的计算，比如连接、聚合，在离线的预计算过程中就已经完成，这大大降低了查询时刻所需的计算量，提高了响应速度。
4）可伸缩性和高吞吐率：单节点Kylin可实现每秒70个查询，还可以搭建Kylin的集群。
5）BI工具集成
Kylin可以与现有的BI工具集成，具体包括如下内容。
ODBC：与Tableau、Excel、PowerBI等工具集成
JDBC：与Saiku、BIRT等Java工具集成
RestAPI：与JavaScript、Web网页集成
Kylin开发团队还贡献了Zepplin的插件，也可以使用Zepplin来访问Kylin服务。

## 安装

```
cd /opt/software  # 安装Kylin前需先部署好Hadoop、Hive、Zookeeper、HBase。
wget https://mirrors.tuna.tsinghua.edu.cn/apache/kylin/apache-kylin-3.1.0/apache-kylin-3.1.0-bin-hadoop3.tar.gz
wget http://mirror.bit.edu.cn/apache/kylin/apache-kylin-3.1.0/apache-kylin-3.1.0-bin-hbase1x.tar.gz
tar -zxvf apache-kylin-3.1.0-bin-hadoop3.tar.gz -C /opt/module/
mv /opt/module/apache-kylin-3.1.0-bin-hadoop3/ /opt/module/kylin
vim /etc/profile
#KYLIN_HOME
export KYLIN_HOME=/opt/module/kylin
export PATH=$PATH:$KYLIN_HOME/bin

xsync /etc/profile
source /etc/profile
vim /opt/module/kylin/bin/find-hbase-dependency.sh # 修改39行
原来：result=`echo $data | grep -e 'hbase-common[a-z0-9A-Z\.-]*jar' | grep -v tests`
新：result=`echo $data | grep -e 'hbase-[common-shaded\-client][a-z0-9A-Z\.-]*jar' | grep -v tests`

kylin.sh start # 出现“Web UI is at“成功，访问http://hadoop102:7070/kylin，初始账号密码DMIN/KYLIN（大写）。服务器启动后，可查看运行时日志$KYLIN_HOME/logs/kylin.log。停止麒麟：kylin.sh stop。启动前，需先启动Hadoop（hdfs，yarn，jobhistoryserver）、Zookeeper、Hbase
```

## Cube构建原理

**维度和度量**
维度：即观察数据的角度。比如员工数据，可以从性别角度来分析，也可以更加细化，从入职时间或者地区的维度来观察。维度是一组离散的值，比如说性别中的男和女，或者时间维度上的每一个独立的日期。因此在统计时可以将维度值相同的记录聚合在一起，然后应用聚合函数做累加、平均、最大和最小值等聚合计算。
度量：即被聚合（观察）的统计值，也就是聚合运算的结果。比如说员工数据中不同性别员工的人数，又或者说在同一年入职的员工有多少。
**Cube和Cuboid**
有了维度跟度量，一个数据表或者数据模型上的所有字段就可以分类了，它们要么是维度，要么是度量（可以被聚合）。于是就有了根据维度和度量做预计算的Cube理论。
给定一个数据模型，我们可以对其上的所有维度进行聚合，对于N个维度来说，组合`的所有可能性共有2n种。对于每一种维度的组合，将度量值做聚合计算，然后将结果保存为一个物化视图，称为Cuboid。所有维度组合的Cuboid作为一个整体，称为Cube。
下面举一个简单的例子说明，假设有一个电商的销售数据集，其中维度包括时间[time]、商品[item]、地区[location]和供应商[supplier]，度量为销售额。那么所有维度的组合就有24 = 16种，如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6edbe91864044c5bbe3a9e1779b539ce~tplv-k3u1fbpfcp-zoom-1.image)

一维度（1D）的组合有：[time]、[item]、[location]和[supplier]4种；
二维度（2D）的组合有：[time, item]、[time, location]、[time, supplier]、[item, location]、[item, supplier]、[location, supplier]3种；
三维度（3D）的组合也有4种；
最后还有零维度（0D）和四维度（4D）各有一种，总共16种。
注意：每一种维度组合就是一个Cuboid，16个Cuboid整体就是一个Cube。
 **Cube存储原理**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e81a16adeab40c8856d4ed3c06ba48d~tplv-k3u1fbpfcp-zoom-1.image) 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5377b6eb4e944ad19a1ec798f968f858~tplv-k3u1fbpfcp-zoom-1.image)

**Cube构建算法**
1.**逐层构建算法（layer）**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99ed95942c4f43c2b8ee80640998227d~tplv-k3u1fbpfcp-zoom-1.image)

我们知道，一个N维的Cube，是由1个N维子立方体、N个(N-1)维子立方体、N*(N-1)/2个(N-2)维子立方体、......、N个1维子立方体和1个0维子立方体构成，总共有2^N个子立方体组成，在逐层算法中，按维度数逐层减少来计算，每个层级的计算（除了第一层，它是从原始数据聚合而来），是基于它上一层级的结果来计算的。比如，[Group by A, B]的结果，可以基于[Group by A, B, C]的结果，通过去掉C后聚合得来的；这样可以减少重复计算；当 0维度Cuboid计算出来的时候，整个Cube的计算也就完成了。
每一轮的计算都是一个MapReduce任务，且串行执行；一个N维的Cube，至少需要N次MapReduce Job。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7900c2f4ead47f0b5021fc7686995e3~tplv-k3u1fbpfcp-zoom-1.image)

算法优点：
1）此算法充分利用了MapReduce的优点，处理了中间复杂的排序和shuffle工作，故而算法代码清晰简单，易于维护；
2）受益于Hadoop的日趋成熟，此算法非常稳定，即便是集群资源紧张时，也能保证最终能够完成。
算法缺点：
1）当Cube有比较多维度的时候，所需要的MapReduce任务也相应增加；由于Hadoop的任务调度需要耗费额外资源，特别是集群较庞大的时候，反复递交任务造成的额外开销会相当可观；
2）由于Mapper逻辑中并未进行聚合操作，所以每轮MR的shuffle工作量都很大，导致效率低下。
3）对HDFS的读写操作较多：由于每一层计算的输出会用做下一层计算的输入，这些Key-Value需要写到HDFS上；当所有计算都完成后，Kylin还需要额外的一轮任务将这些文件转成HBase的HFile格式，以导入到HBase中去；
总体而言，该算法的效率较低，尤其是当Cube维度数较大的时候。
2.**快速构建算法（inmem）**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf6eedb8467642c588e7dbb1b6bfe343~tplv-k3u1fbpfcp-zoom-1.image)

也被称作“逐段”(By Segment) 或“逐块”(By Split) 算法，从1.5.x开始引入该算法，该算法的主要思想是，每个Mapper将其所分配到的数据块，计算成一个完整的小Cube 段（包含所有Cuboid）。每个Mapper将计算完的Cube段输出给Reducer做合并，生成大Cube，也就是最终结果。如图所示解释了此流程。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32e9496cb2e5423fbdaa2902e8a36b8f~tplv-k3u1fbpfcp-zoom-1.image)

与旧算法相比，快速算法主要有两点不同：
1） Mapper会利用内存做预聚合，算出所有组合；Mapper输出的每个Key都是不同的，这样会减少输出到Hadoop MapReduce的数据量，Combiner也不再需要；
2）一轮MapReduce便会完成所有层次的计算，减少Hadoop任务的调配。

## Cube构建优化

1.**使用衍生维度（derived dimension）**
衍生维度用于在有效维度内将维度表上的非主键维度排除掉，并使用维度表的主键（其实是事实表上相应的外键）来替代它们。Kylin会在底层记录维度表主键与维度表其他维度之间的映射关系，以便在查询时能够动态地将维度表的主键“翻译”成这些非主键维度，并进行实时聚合。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2183c7f748ef47eb976284cad43f6d91~tplv-k3u1fbpfcp-zoom-1.image)

虽然衍生维度具有非常大的吸引力，但这也并不是说所有维度表上的维度都得变成衍生维度，如果从维度表主键到某个维度表维度所需要的聚合工作量非常大，则不建议使用衍生维度。
2.**使用聚合组（Aggregation group）**
聚合组（Aggregation Group）是一种强大的剪枝工具。聚合组假设一个Cube的所有维度均可以根据业务需求划分成若干组（当然也可以是一个组），由于同一个组内的维度更可能同时被同一个查询用到，因此会表现出更加紧密的内在关联。每个分组的维度集合均是Cube所有维度的一个子集，不同的分组各自拥有一套维度集合，它们可能与其他分组有相同的维度，也可能没有相同的维度。每个分组各自独立地根据自身的规则贡献出一批需要被物化的Cuboid，所有分组贡献的Cuboid的并集就成为了当前Cube中所有需要物化的Cuboid的集合。不同的分组有可能会贡献出相同的Cuboid，构建引擎会察觉到这点，并且保证每一个Cuboid无论在多少个分组中出现，它都只会被物化一次。
对于每个分组内部的维度，用户可以使用如下三种可选的方式定义，它们之间的关系，具体如下。
1）强制维度（Mandatory），如果一个维度被定义为强制维度，那么这个分组产生的所有Cuboid中每一个Cuboid都会包含该维度。每个分组中都可以有0个、1个或多个强制维度。如果根据这个分组的业务逻辑，则相关的查询一定会在过滤条件或分组条件中，因此可以在该分组中把该维度设置为强制维度。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d187a6f0ed5430ca09c6ce2846547ff~tplv-k3u1fbpfcp-zoom-1.image)

2）层级维度（Hierarchy），每个层级包含两个或更多个维度。假设一个层级中包含D1，D2…Dn这n个维度，那么在该分组产生的任何Cuboid中， 这n个维度只会以（），（D1），（D1，D2）…（D1，D2…Dn）这n+1种形式中的一种出现。每个分组中可以有0个、1个或多个层级，不同的层级之间不应当有共享的维度。如果根据这个分组的业务逻辑，则多个维度直接存在层级关系，因此可以在该分组中把这些维度设置为层级维度。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/431a0b974ff7420aaac3055a5497cc96~tplv-k3u1fbpfcp-zoom-1.image)

3）联合维度（Joint），每个联合中包含两个或更多个维度，如果某些列形成一个联合，那么在该分组产生的任何Cuboid中，这些联合维度要么一起出现，要么都不出现。每个分组中可以有0个或多个联合，但是不同的联合之间不应当有共享的维度（否则它们可以合并成一个联合）。如果根据这个分组的业务逻辑，多个维度在查询中总是同时出现，则可以在该分组中把这些维度设置为联合维度。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cd052967379458f828f039e594173e9~tplv-k3u1fbpfcp-zoom-1.image)

这些操作可以在Cube Designer的Advanced Setting中的Aggregation Groups区域完成，如下图所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af7309a6fa59419aaa17f96b00facf9a~tplv-k3u1fbpfcp-zoom-1.image)

聚合组的设计非常灵活，甚至可以用来描述一些极端的设计。假设我们的业务需求非常单一，只需要某些特定的Cuboid，那么可以创建多个聚合组，每个聚合组代表一个Cuboid。具体的方法是在聚合组中先包含某个Cuboid所需的所有维度，然后把这些维度都设置为强制维度。这样当前的聚合组就只能产生我们想要的那一个Cuboid了。
再比如，有的时候我们的Cube中有一些基数非常大的维度，如果不做特殊处理，它就会和其他的维度进行各种组合，从而产生一大堆包含它的Cuboid。包含高基数维度的Cuboid在行数和体积上往往非常庞大，这会导致整个Cube的膨胀率变大。如果根据业务需求知道这个高基数的维度只会与若干个维度（而不是所有维度）同时被查询到，那么就可以通过聚合组对这个高基数维度做一定的“隔离”。我们把这个高基数的维度放入一个单独的聚合组，再把所有可能会与这个高基数维度一起被查询到的其他维度也放进来。这样，这个高基数的维度就被“隔离”在一个聚合组中了，所有不会与它一起被查询到的维度都没有和它一起出现在任何一个分组中，因此也就不会有多余的Cuboid产生。这点也大大减少了包含该高基数维度的Cuboid的数量，可以有效地控制Cube的膨胀率。
3.**Row Key优化**
Kylin会把所有的维度按照顺序组合成一个完整的Rowkey，并且按照这个Rowkey升序排列Cuboid中所有的行。
设计良好的Rowkey将更有效地完成数据的查询过滤和定位，减少IO次数，提高查询速度，维度在rowkey中的次序，对查询性能有显著的影响。
Row key的设计原则如下：
1）被用作过滤的维度放在前边。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54bb611ccb234ccaa7ec960ad2411855~tplv-k3u1fbpfcp-zoom-1.image)

2）基数大的维度放在基数小的维度前边。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cec9aeb6d648407b878595704ce1b45a~tplv-k3u1fbpfcp-zoom-1.image)

4.**并发粒度优化**
当Segment中某一个Cuboid的大小超出一定的阈值时，系统会将该Cuboid的数据分片到多个分区中，以实现Cuboid数据读取的并行化，从而优化Cube的查询速度。具体的实现方式如下：构建引擎根据Segment估计的大小，以及参数“kylin.hbase.region.cut”的设置决定Segment在存储引擎中总共需要几个分区来存储，如果存储引擎是HBase，那么分区的数量就对应于HBase中的Region数量。kylin.hbase.region.cut的默认值是5.0，单位是GB，也就是说对于一个大小估计是50GB的Segment，构建引擎会给它分配10个分区。用户还可以通过设置kylin.hbase.region.count.min（默认为1）和kylin.hbase.region.count.max（默认为500）两个配置来决定每个Segment最少或最多被划分成多少个分区。

由于每个Cube的并发粒度控制不尽相同，因此建议在Cube Designer 的Configuration Overwrites（上图所示）中为每个Cube量身定制控制并发粒度的参数。假设将把当前Cube的kylin.hbase.region.count.min设置为2，kylin.hbase.region.count.max设置为100。这样无论Segment的大小如何变化，它的分区数量最小都不会低于2，最大都不会超过100。相应地，这个Segment背后的存储引擎（HBase）为了存储这个Segment，也不会使用小于两个或超过100个的分区。我们还调整了默认的kylin.hbase.region.cut，这样50GB的Segment基本上会被分配到50个分区，相比默认设置，我们的Cuboid可能最多会获得5倍的并发量。

## Kylin BI工具集成

可以与Kylin结合使用的可视化工具很多，例如：
ODBC：与Tableau、Excel、PowerBI等工具集成
JDBC：与Saiku、BIRT等Java工具集成
RestAPI：与JavaScript、Web网页集成
Kylin开发团队还贡献了Zepplin的插件，也可以使用Zepplin来访问Kylin服务。
**JDBC**
1.maven项目并导入依赖

```xml
		<dependency>
            <groupId>org.apache.kylin</groupId>
            <artifactId>kylin-jdbc</artifactId>
            <version>2.5.1</version>
        </dependency>
```

2.编码

```java
package com.demo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class TestKylin {
    public static void main(String[] args) throws Exception {

        //Kylin_JDBC 驱动
        String KYLIN_DRIVER = "org.apache.kylin.jdbc.Driver";

        //Kylin_URL
        String KYLIN_URL = "jdbc:kylin://hadoop102:7070/gmall";

        //Kylin的用户名
        String KYLIN_USER = "ADMIN";

        //Kylin的密码
        String KYLIN_PASSWD = "KYLIN";

        //添加驱动信息
        Class.forName(KYLIN_DRIVER);

        //获取连接
        Connection connection = DriverManager.getConnection(KYLIN_URL, KYLIN_USER, KYLIN_PASSWD);

        //预编译SQL
        PreparedStatement ps = connection.prepareStatement("select bp.province_name,sum(od.total_amount) from dwd_fact_order_detail od join dwd_dim_base_province bp on od.province_id = bp.id group by bp.province_name");

        //执行查询
        ResultSet resultSet = ps.executeQuery();
        //遍历打印
        while (resultSet.next()) {
            System.out.println("省份：" + resultSet.getString(1) + "订单总金额" + resultSet.getDouble(2) );
        }
    }

}
```

3.结果展示

```
省份：山东订单总金额488.0
省份：黑龙江订单总金额1553.0
省份：西藏订单总金额488.0
省份：福建订单总金额290.0
省份：四川订单总金额222.0
省份：贵州订单总金额3106.0
```

**Zepplin**

```
# 文件比较大，大概1G
wget https://archive.apache.org/dist/zeppelin/zeppelin-0.8.0/zeppelin-0.8.0-bin-all.tgz
tar -xvf zeppelin-0.9.0-preview2-bin-all.tgz -C /opt/module
mv /opt/module/zeppelin-0.8.0-bin-all/ /opt/module/zeppelin
/opt/module/zeppelin/bin/zeppelin-daemon.sh start # http://hadoop102:8080
/opt/module/zeppelin/bin/zeppelin-daemon.sh stop
# 可登录网页查看，web默认端口号为8080，日志查看/opt/module/zeppelin/logs/zeppelin-admin-hadoop102.log
```

配置Zepplin支持Kylin
1.点击右上角anonymous选择Interpreter，搜索Kylin插件并修改相应的配置，修改完成点击Save完成

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17d7212774b9469ba41b2c7660c69104~tplv-k3u1fbpfcp-zoom-1.image)

**案例实操**：查询员工详细信息，并使用各种图表进行展示
（1）点击Notebook创建新的note，随意填写Note Name，Default Interpreter选择kylin再点击Create 

（2）执行查询

```sql
select bp.province_name,sum(od.total_amount) from dwd_fact_order_detail od join dwd_dim_base_province bp on od.province_id = bp.id group by bp.province_name
```

结果

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/459674e3fb2d4b0a8ee4a78b1863d0b8~tplv-k3u1fbpfcp-zoom-1.image)

（3）其他图表格式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/960c8ec4e6f7482c9aba968d3cc37e0c~tplv-k3u1fbpfcp-zoom-1.image)

 

 