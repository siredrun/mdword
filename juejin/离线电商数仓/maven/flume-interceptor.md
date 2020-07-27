# flume-interceptor

本项目中自定义两个拦截器，分别是：ETL拦截器、日志类型区分拦截器。

ETL拦截器作用：过滤时间戳不合法和Json数据不完整的日志。

日志类型区分拦截器作用：将启动日志和事件日志区分开来，方便发往Kafka的不同Topic。

# pom.xml

创建一个Maven工程flume-interceptor，并在在pom.xml文件中添加如下配置。

```xml
	<dependencies>
        <dependency>
            <groupId>org.apache.flume</groupId>
            <artifactId>flume-ng-core</artifactId>
            <!-- 版本号最好和集群上安装的一致 -->
            <version>1.7.0</version>
        </dependency>
    </dependencies>
```

# ETL拦截器LogETLInterceptor

```java
package com.root.flume;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;

/**
 * ETL拦截器
 */
public class LogETLInterceptor implements Interceptor {

    public void initialize() {

    }

    public Event intercept(Event event) {

        // 1 获取数据
        byte[] body = event.getBody();
        String log = new String(body, Charset.forName("UTF-8"));

        // 2 判断数据类型并向Header中赋值
        if (log.contains("start")) {
            if (LogUtils.validateStart(log)) {
                return event;
            }
        } else {
            if (LogUtils.validateEvent(log)) {
                return event;
            }
        }

        // 3 返回校验结果
        return null;
    }

    public List<Event> intercept(List<Event> events) {

        ArrayList<Event> interceptors = new ArrayList<Event>();

        for (Event event : events) {
            Event intercept1 = intercept(event);

            if (intercept1 != null) {
                interceptors.add(intercept1);
            }
        }

        return interceptors;
    }

    public void close() {

    }

    public static class Builder implements Interceptor.Builder {

        public Interceptor build() {
            return new LogETLInterceptor();
        }

        public void configure(Context context) {

        }
    }
}
```

# 日志类型区分拦截器LogTypeInterceptor

类似MyInterceptor

```java
package com.root.flume;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * 日志类型区分拦截器
 */
public class LogTypeInterceptor implements Interceptor {
    public void initialize() {

    }

    public Event intercept(Event event) {

        // 区分日志类型：   body  header
        // 1 获取body数据
        byte[] body = event.getBody();
        String log = new String(body, Charset.forName("UTF-8"));

        // 2 获取header
        Map<String, String> headers = event.getHeaders();

        // 3 判断数据类型并向Header中赋值
        if (log.contains("start")) {
            headers.put("topic","topic_start");
        }else {
            headers.put("topic","topic_event");
        }

        return event;
    }

    public List<Event> intercept(List<Event> events) {

        ArrayList<Event> interceptors = new ArrayList<Event>();

        for (Event event : events) {
            Event intercept1 = intercept(event);

            interceptors.add(intercept1);
        }

        return interceptors;
    }

    public void close() {

    }

    public static class Builder implements  Interceptor.Builder{

        public Interceptor build() {
            return new LogTypeInterceptor();
        }

        public void configure(Context context) {

        }
    }

}
```

# 日志过滤工具类LogUtils

```java
package com.root.flume;

import org.apache.commons.lang.math.NumberUtils;

/**
 * 日志过滤工具类
 */
public class LogUtils {

    public static boolean validateEvent(String log) {
        // 服务器时间 | json
        // 1549696569054 | {"cm":{"ln":"-89.2","sv":"V2.0.4","os":"8.2.0","g":"M67B4QYU@gmail.com","nw":"4G","l":"en","vc":"18","hw":"1080*1920","ar":"MX","uid":"u8678","t":"1549679122062","la":"-27.4","md":"sumsung-12","vn":"1.1.3","ba":"Sumsung","sr":"Y"},"ap":"weather","et":[]}

        // 1 切割
        String[] logContents = log.split("\\|");

        // 2 校验
        if(logContents.length != 2){
            return false;
        }

        //3 校验服务器时间
        if (logContents[0].length()!=13 || !NumberUtils.isDigits(logContents[0])){
            return false;
        }

        // 4 校验json
        if (!logContents[1].trim().startsWith("{") || !logContents[1].trim().endsWith("}")){
            return false;
        }

        return true;
    }

    public static boolean validateStart(String log) {
        // {"action":"1","ar":"MX","ba":"HTC","detail":"542","en":"start","entry":"2","extend1":"","g":"S3HQ7LKM@gmail.com","hw":"640*960","l":"en","la":"-43.4","ln":"-98.3","loading_time":"10","md":"HTC-5","mid":"993","nw":"WIFI","open_ad_type":"1","os":"8.2.1","sr":"D","sv":"V2.9.0","t":"1559551922019","uid":"993","vc":"0","vn":"1.1.5"}

        if (log == null){
            return false;
        }

        // 校验json
        if (!log.trim().startsWith("{") || !log.trim().endsWith("}")){
            return false;
        }

        return true;
    }

    public static void main(String[] args) {
        String start = "{\"action\":\"1\",\"ar\":\"MX\",\"ba\":\"HTC\",\"detail\":\"201\",\"en\":\"start\",\"entry\":\"4\",\"extend1\":\"\",\"g\":\"37S06O79@gmail.com\",\"hw\":\"750*1134\",\"l\":\"pt\",\"la\":\"-30.7\",\"ln\":\"-98.4\",\"loading_time\":\"19\",\"md\":\"HTC-17\",\"mid\":\"947\",\"nw\":\"4G\",\"open_ad_type\":\"2\",\"os\":\"8.2.8\",\"sr\":\"V\",\"sv\":\"V2.6.6\",\"t\":\"1593445707878\",\"uid\":\"947\",\"vc\":\"16\",\"vn\":\"1.1.8\"}";
        String event = "1593463233058|{\"cm\":{\"ln\":\"-42.2\",\"sv\":\"V2.9.1\",\"os\":\"8.2.5\",\"g\":\"BNV4DR87@gmail.com\",\"mid\":\"948\",\"nw\":\"3G\",\"l\":\"pt\",\"vc\":\"3\",\"hw\":\"750*1134\",\"ar\":\"MX\",\"uid\":\"948\",\"t\":\"1593445865220\",\"la\":\"-24.5\",\"md\":\"HTC-11\",\"vn\":\"1.3.9\",\"ba\":\"HTC\",\"sr\":\"E\"},\"ap\":\"app\",\"et\":[{\"ett\":\"1593375210165\",\"en\":\"notification\",\"kv\":{\"ap_time\":\"1593417019576\",\"action\":\"1\",\"type\":\"4\",\"content\":\"\"}},{\"ett\":\"1593409732369\",\"en\":\"active_foreground\",\"kv\":{\"access\":\"\",\"push_id\":\"3\"}},{\"ett\":\"1593405199765\",\"en\":\"active_background\",\"kv\":{\"active_source\":\"2\"}},{\"ett\":\"1593420337649\",\"en\":\"error\",\"kv\":{\"errorDetail\":\"java.lang.NullPointerException\\\\n    at cn.lift.appIn.web.AbstractBaseController.validInbound(AbstractBaseController.java:72)\\\\n at cn.lift.dfdf.web.AbstractBaseController.validInbound\",\"errorBrief\":\"at cn.lift.dfdf.web.AbstractBaseController.validInbound(AbstractBaseController.java:72)\"}},{\"ett\":\"1593410111548\",\"en\":\"comment\",\"kv\":{\"p_comment_id\":2,\"addtime\":\"1593418637738\",\"praise_count\":425,\"other_id\":5,\"comment_id\":7,\"reply_count\":40,\"userid\":7,\"content\":\"固店终片尤坷昭凄\"}}]}";

        //两个都为true表示成功
        System.out.println(validateEvent(event));
        System.out.println(validateStart(start));

    }
}
```

# 打包

打包生成flume-interceptor-1.0-SNAPSHOT.jar