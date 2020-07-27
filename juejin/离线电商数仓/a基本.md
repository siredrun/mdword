# 05dwd搭建

DWD(data warehouse detail)明细数据层：结构和粒度与原始表保持一致， 对ODS层数据进行清洗(去除空值， 脏数据，明田数据层超过极限范围的数据)。也有称呼为为DWI。注意：表都在数据gmall下创建，hsql执行建议使用可视化工具datagrip。

```
get_json_object(json字符串, 获取表达式)在hive中从json对象抽取指定的内容。需要指定要抽取的内容的path路径，如果函数传入的数据不是JSON,此时会返回Null。
$   : Root object     代表整个JSON对象
.   : Child operator   代表获取JSON对象中子元素(属性)的操作符
[]  : Subscript operator for array 获取JSONArray中的某个元素
例：
select get_json_object('{"name":"jack","age":18,"parents":[{"name":"oldjack","age":48}，{"name":"jackmom","age":48}]}','$.age'); # 获取18
select get_json_object('{"name":"jack","age":18,"parents":[{"name":"oldjack","age":48},{"name":"jackmom","age":48}]}','$.parents[1].age'); # 获取48
```

## 启动表

```sql
# 创建启动表
drop table if exists dwd_start_log;
CREATE EXTERNAL TABLE dwd_start_log(
`mid_id` string,
`user_id` string,
`version_code` string,
`version_name` string,
`lang` string,
`source` string,
`os` string,
`area` string,
`model` string,
`brand` string,
`sdk_version` string,
`gmail` string,
`height_width` string,
`app_time` string,
`network` string,
`lng` string,
`lat` string,
`entry` string,
`open_ad_type` string,
`action` string,
`loading_time` string,
`detail` string,
`extend1` string
)
PARTITIONED BY (dt string)
stored as parquet
location '/warehouse/gmall/dwd/dwd_start_log/'
TBLPROPERTIES('parquet.compression'='lzo');
# 向启动表导入数据
insert overwrite table dwd_start_log
PARTITION (dt='2020-07-08')
select 
    get_json_object(line,'$.mid') mid_id,
    get_json_object(line,'$.uid') user_id,
    get_json_object(line,'$.vc') version_code,
    get_json_object(line,'$.vn') version_name,
    get_json_object(line,'$.l') lang,
    get_json_object(line,'$.sr') source,
    get_json_object(line,'$.os') os,
    get_json_object(line,'$.ar') area,
    get_json_object(line,'$.md') model,
    get_json_object(line,'$.ba') brand,
    get_json_object(line,'$.sv') sdk_version,
    get_json_object(line,'$.g') gmail,
    get_json_object(line,'$.hw') height_width,
    get_json_object(line,'$.t') app_time,
    get_json_object(line,'$.nw') network,
    get_json_object(line,'$.ln') lng,
    get_json_object(line,'$.la') lat,
    get_json_object(line,'$.entry') entry,
    get_json_object(line,'$.open_ad_type') open_ad_type,
    get_json_object(line,'$.action') action,
    get_json_object(line,'$.loading_time') loading_time,
    get_json_object(line,'$.detail') detail,
    get_json_object(line,'$.extend1') extend1
from ods_start_log 
where dt='2020-07-08';
# 测试
select * from dwd_start_log limit 2;
# 启动脚本dwd_start_log
dwd_start_log 2020-07-05
select * from dwd_start_log where dt='2020-07-05' limit 2;
```

## 事件表

基础明细表dwd_base_event_log用于存储ODS层原始表转换过来的明细数据。

```
1594195017857| 服务器时间server_time
"cm": { 属于cm的都是公共字段，UDF函数（一进一出），如ln对应dwd_base_event_log.lng，sv对应dwd_base_event_log.sdk_version。
	"ln": "-81.6", 对应dwd_base_event_log.lng
	"sv": "V2.0.3", 对应dwd_base_event_log.sdk_version
	"os": "8.2.6",
	"g": "5Y27B7J6@gmail.com",
	"mid": "990",
	"nw": "4G",
	"l": "pt",
	"vc": "4",
	"hw": "750*1134",
	"ar": "MX",
	"uid": "990",
	"t": "1594189170143",
	"la": "-33.9",
	"md": "HTC-14",
	"vn": "1.0.8",
	"ba": "HTC",
	"sr": "B"
	},
"ap": "app", ap表示来源
"et": [ et表示event事件，是多个事件，采用UDTF函数（一进多出）。
	{ en事件名称event_name，kv事件详情event_json
	"ett": "1594111889459",
	"en": "display", 
	"kv": { 
	"goodsid": "233",
	"action": "1",
	"extend1": "2",
	"place": "1",
	"category": "90"
	}
	},
	{
	"ett": "1594120690516",
	"en": "newsdetail",
	"kv": {
	"entry": "1",
	"goodsid": "234",
	"news_staytime": "9",
	"loading_time": "54",
	"action": "4",
	"showtype": "0",
	"category": "92",
	"type1": ""
	}
	}...
]
```

# hive-function

本项目是为了做编写自定义函数解析基础明细表的相关字段

自定义UDF函数（解析公共字段）和自定义UDTF函数（解析具体事件字段）

创建一个普通maven，名字为hive-function（自定义函数解析），在pom.xml文件中添加如下内容

```xml
    <dependencies>
        <!--添加hive依赖-->
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <!-- 版本最好和节点上安装的hive一致 -->
            <version>2.3.7</version>
        </dependency>
    </dependencies>
```

## BaseFieldUDF解析公共字段

创建类：com.root.udf.BaseFieldUDF

```java
package com.root;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hive.ql.exec.UDF;
import org.codehaus.jettison.json.JSONException;
import org.codehaus.jettison.json.JSONObject;

/**
 * 继承UDF类，提供多个evaluate方法，evaluate方法不能返回void，但是可以返回null
 */
public class BaseFieldUDF extends UDF {

    public String evaluate(String line, String key) throws JSONException {

        if (!line.contains(key) && !"ts".equals(key)) {
            return "";
        }
        // 1 处理line   服务器时间 | json
        String[] log = line.split("\\|");

        //2 合法性校验
        if (log.length != 2 || StringUtils.isBlank(log[1])) {
            return "";
        }

        // 3 开始处理json
        JSONObject baseJson = new JSONObject(log[1].trim());

        String result = "";

        // 4 根据传进来的key查找相应的value
        if ("ts".equals(key)){
            return log[0].trim();
        }else if ("ap".equals(key)){
            return baseJson.getString("ap");
        }else if ("et".equals(key)){
            return baseJson.getString("et");
        }else{
            // 获取cm的某个属性
            return baseJson.getJSONObject("cm").getString(key);
        }
    }

    public static void main(String[] args) throws JSONException {

        String line = "1541217850324|{\"cm\":{\"mid\":\"m7856\",\"uid\":\"u8739\",\"ln\":\"-74.8\",\"sv\":\"V2.2.2\",\"os\":\"8.1.3\",\"g\":\"P7XC9126@gmail.com\",\"nw\":\"3G\",\"l\":\"es\",\"vc\":\"6\",\"hw\":\"640*960\",\"ar\":\"MX\",\"t\":\"1541204134250\",\"la\":\"-31.7\",\"md\":\"huawei-17\",\"vn\":\"1.1.2\",\"sr\":\"O\",\"ba\":\"Huawei\"},\"ap\":\"weather\",\"et\":[{\"ett\":\"1541146624055\",\"en\":\"display\",\"kv\":{\"goodsid\":\"n4195\",\"copyright\":\"ESPN\",\"content_provider\":\"CNN\",\"extend2\":\"5\",\"action\":\"2\",\"extend1\":\"2\",\"place\":\"3\",\"showtype\":\"2\",\"category\":\"72\",\"newstype\":\"5\"}},{\"ett\":\"1541213331817\",\"en\":\"loading\",\"kv\":{\"extend2\":\"\",\"loading_time\":\"15\",\"action\":\"3\",\"extend1\":\"\",\"type1\":\"\",\"type\":\"3\",\"loading_way\":\"1\"}},{\"ett\":\"1541126195645\",\"en\":\"ad\",\"kv\":{\"entry\":\"3\",\"show_style\":\"0\",\"action\":\"2\",\"detail\":\"325\",\"source\":\"4\",\"behavior\":\"2\",\"content\":\"1\",\"newstype\":\"5\"}},{\"ett\":\"1541202678812\",\"en\":\"notification\",\"kv\":{\"ap_time\":\"1541184614380\",\"action\":\"3\",\"type\":\"4\",\"content\":\"\"}},{\"ett\":\"1541194686688\",\"en\":\"active_background\",\"kv\":{\"active_source\":\"3\"}}]}";
        String x = new BaseFieldUDF().evaluate(line, "mid");
        System.out.println(x);
    }
}
```

## EventJsonUDTF解析具体事件字段

创建类：com.root.udtf.EventJsonUDTF

```java
mkdir $HIVE_HOME/auxlib
vim hivetest
1589904040011|{"cm":{"ln":"-113.3","sv":"V2.4.5","os":"8.1.2","g":"9J6D4C78@gmail.com","mid":"499","nw":"3G","l":"es","vc":"18","hw":"750*1134","ar":"MX","uid":"499","t":"1589883103852","la":"-47.5","md":"sumsung-0","vn":"1.0.8","ba":"Sumsung","sr":"B"},"ap":"app","et":[{"ett":"1589824937034","en":"display","kv":{"goodsid":"118","action":"1","extend1":"2","place":"0","category":"93"}},{"ett":"1589885144897","en":"ad","kv":{"entry":"3","show_style":"4","action":"1","detail":"102","source":"3","behavior":"1","content":"1","newstype":"4"}},{"ett":"1589888928984","en":"notification","kv":{"ap_time":"1589850530085","action":"3","type":"1","content":""}},{"ett":"1589820137819","en":"active_foreground","kv":{"access":"1","push_id":"1"}},{"ett":"1589845774345","en":"comment","kv":{"p_comment_id":0,"addtime":"1589839832384","praise_count":283,"other_id":1,"comment_id":4,"reply_count":16,"userid":6,"content":"赞谤隧遁雌恶霍嗡魂牵静默擒赵割"}},{"ett":"1589884580200","en":"praise","kv":{"target_id":9,"id":3,"type":3,"add_time":"1589812371891","userid":4}}]}

hive -d a=$(cat hivetest)
hive (gmall)> use gmall;
OK
Time taken: 0.401 seconds
hive (gmall)> set a;
a=1589904040011|{"cm":{"ln":"-113.3","sv":"V2.4.5","os":"8.1.2","g":"9J6D4C78@gmail.com","mid":"499","nw":"3G","l":"es","vc":"18","hw":"750*1134","ar":"MX","uid":"499","t":"1589883103852","la":"-47.5","md":"sumsung-0","vn":"1.0.8","ba":"Sumsung","sr":"B"},"ap":"app","et":[{"ett":"1589824937034","en":"display","kv":{"goodsid":"118","action":"1","extend1":"2","place":"0","category":"93"}},{"ett":"1589885144897","en":"ad","kv":{"entry":"3","show_style":"4","action":"1","detail":"102","source":"3","behavior":"1","content":"1","newstype":"4"}},{"ett":"1589888928984","en":"notification","kv":{"ap_time":"1589850530085","action":"3","type":"1","content":""}},{"ett":"1589820137819","en":"active_foreground","kv":{"access":"1","push_id":"1"}},{"ett":"1589845774345","en":"comment","kv":{"p_comment_id":0,"addtime":"1589839832384","praise_count":283,"other_id":1,"comment_id":4,"reply_count":16,"userid":6,"content":"赞谤隧遁雌恶霍嗡魂牵静默擒赵割"}},{"ett":"1589884580200","en":"praise","kv":{"target_id":9,"id":3,"type":3,"add_time":"1589812371891","userid":4}}]}
```



# 1

```

```



# 1