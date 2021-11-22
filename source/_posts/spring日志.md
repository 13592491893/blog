---
abbrlink: 5502
title: spring日志
comments: true
toc: true
description: spring日志
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
tags:
  - spring
  - 日志
  - log
date: 2021-4-25 16:00:00
---
# spring日志

今天来谈一谈日志，主要是说一说springboot的日志，因为最近在学习springboot。首先在写代码的时候，要养成记日志的习惯，这点真的很重要，因为之前吃了很多亏。过去我对日志很不在意，该有的日志没有，不该有的日志却随意输出。新换的工作，上司对日志有严格的要求，也就慢慢开始注意了。

一般而言，一个接口或者说一段程序，其入口要有日志，记录传入的数据是什么；部分重要的处理逻辑要有日志输出；程序出口也要有日志，记录其最终的处理结果。这样在解决生产上的问题时，可以很快的定位问题的位置，是传入数据的问题还是我们代码逻辑写错了，总比凭空想象的好，要相信计算机，日志是不会骗人的。

还有一点，在生产上严禁使用System.out输出，性能太低，原因是System.out输出会导致线程等待（同步），而使用Logger输出线程不等待日志的输出（异步），而继续执行。

接下来看一看springboot的日志配置,说一下把日志记录到文件中的配置方式。

## 一、工具/原料

- springboot
- 日志

## 二、方法/步骤

1. springboot推荐的日志类库是slf4j、日志系统为logback,确实我回头一看项目中使用的都是slf4j，说明这个东西确实有他的优点。

   上文中也说了一点，slf4j有个接口叫Logger，提供了丰富的日志输出方法，包含了所有日志级别的输出。使用方式也是特别的简单，用slf4j的工厂类获取一个logger ，然后就可以输出日志了，默认情况下，日志只会输出到控制台。

   ![img](https://img-blog.csdn.net/20180918173714938?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xjaHExOTk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2. 通过在application.properties文件中配置logging.file、logging.path可以控制日志文件的输出路径和文件名。

   不过有些细节需要注意，否则配置不生效，我测试了几种情况。

   ![img](https://img-blog.csdn.net/20180918173738308?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xjaHExOTk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3. 如果，两者都配置了：logging.file=myLog.log、logging.path=D:/data/mylog，注意windos的路径（后面配置文件中也是/），此时并不会在d盘下生成日志文件，只会在项目的根目录下创建一个myLog.log的文件（workspace中，此项目的根目录）。

   其原因是，没有logback-spring.xml配置文件，系统只认识logging.file，不认识logging.path。

   ![img](https://img-blog.csdn.net/20180918173757844?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xjaHExOTk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

4. 所以要配置logback-spring.xml，spring boot会默认加载此文件，为什么不配置logback.xml,因为logback.xml会先application.properties加载，而logback-spring.xml会后于application.properties加载，这样我们在application.properties文中设置日志文件名称和文件路径才能生效。

   ![img](https://img-blog.csdn.net/20180918173824889?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xjaHExOTk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

5. 且看logback-spring.xml的配置详情。注意${LOG_PATH}和${LOG_FILE}分别是获取配置文件中的路径和文件名称，必须使用这两个全局的配置去获取。然后重启项目，发现在配置的目录下，有了相应的日志文件。

    

   ![img](https://img-blog.csdn.net/20180918173909141?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xjaHExOTk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

6. 日志文件的配置结构：

   （1）FILE_LOG_PATTERN：日志输出格式变量，在控制台输出和文件中输出的append中都引用了此变量。（2）consoleLog：定义一个控制台的appender（3）fileLog：定义一个日志文件的appender，这就是文件输出的详细配置，<File>是日志文件的输出地址：必须要为${LOG_PATH}/${LOG_FILE}，这样我们在application.properties中的配置才有效。level标签：如果我们设置了level为info,只会输出info的日志信息，其他日志级别的日志就会过滤掉，建议不配置level属性。

   （4）logger：其name就是项目中对应的包路径，appender-ref是appender的引用，在本配置文件中，意思就是com.example.xyx.MySpringBootTest包下文件的日志，按照fileLog的配置去输出，即按照FILE_LOG_PATTERN的格式，输出到D:/data/mylog/myLog.log文件中。

   标签level="debug"是设置日志级别：作用是debug级别及其以上级别的日志会输出（debug、info、warn、error,,,），注意此处的level是一个下线，比其日志级别高的日志信息也会输出，很重要。

   additivity="false"是配置此logger是否提交给其他的logger或者root节点，如果true，则root也会执行或者其他的可以拦截到的logger节点,且logger的level优先级高;否则不会执行，在本配置文件中即控制台不会输出com.example.xyx.MySpringBootTest包下文件的日志。

   （5）root：根节点，在logback-spring.xml中只引用了控制台日志输出配置，不会输出到文件，如果想输出到文件，可以写再写一个引用。level=info，在控制台输出into级别及其以上级别的日志。会拦截所有包下的日志，但是其输出会受到logger的影响，即注意logger中的additivity属性，如果为false，com.example.xyx.MySpringBootTest包下的日志不会输出到控制台。

   ![img](https://img-blog.csdn.net/20180918173942712?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xjaHExOTk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 三、肉制品同步程序的logback文件

![image-20210820150322211](https://gitee.com/gsshy/picgo/raw/master/img/image-20210820150322211.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="false">

    <conversionRule conversionWord="seq"
                    converterClass="com.shys.iiot.rzp.sync.utils.LocalSeqNumberConverter"/>

    <property resource="application.properties"/>

    <!--日志文件的名称，根据系统自动追加日期和后缀 -->
    <property name="LOG_FILE_NAME" value="shys-iiot-rzp-sync"/>

    <!-- 彩色日志依赖的渲染类 <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"
        /> <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"
        /> <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"
        /> -->

    <!-- %-5p %是格式修饰符， -指左对齐 5指最小字符 -->

    <!-- <property name="CONSOLE_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint}
        %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(-){faint} %clr([%15.15t]){faint}
        %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"
        /> -->

    <!-- 默认Sleuth会在MDC中添加[appname,traceId,spanId,exportable]： spanId - the id
        of a specific operation that took place appname - the name of the application
        that logged the span traceId - the id of the latency graph that contains
        the span exportable - whether the log should be exported to Zipkin or not.
        Whenwould you like the span not to be exportable? In the case in which you
        want towrap some operation in a Span and have it written to the logs only. -->

    <property name="CONSOLE_LOG_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-2seq %-5p --- serviceId:[${spring.application.name:-}] user[%X{userSessionId},%X{enterpriseID}] traceId:[%X{X-B3-TraceId}] [%15.15t] %-40.40logger{39} : %m%n"/>

    <!-- 控制台输出，开发调试可以在下文root中添加项 -->
    <appender name="stdout"
              class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!--dev环境 输出到LogStash property("apollo.env").equalsIgnoreCase("dev") <if
        condition='property("apollo.env").equalsIgnoreCase("dev")'> <then> </then>
        </if>
    <appender name="logstash"
        class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>10.1.5.20:5000</destination>
        <encoder
            class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <pattern>
                    <pattern>
                        {
                        "timestamp": "%date{\"yyyy-MM-dd'T'HH:mm:ss,SSSZ\"}",
                        "seq":"#asLong{%seq}",
                        "serviceId": "${spring.application.name}",
                        "traceId": "%X{X-B3-TraceId}",
                        "userId": "%X{userSessionId}",
                        "enterpriseID": "%X{enterpriseID}",
                        "log_level": "%level",
                        "thread": "%thread",
                        "class_name": "%class",
                        "message": "%message",
                        "stack_trace": "%exception{5}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
-->
    <!-- 每天生成一个日志文件 -->
    <appender name="logfile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/home/logs/${LOG_FILE_NAME}/${LOG_FILE_NAME}.log</file>
        <rollingPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!--日志文件输出的文件路径和文件名，不要修改 -->
            <FileNamePattern>/home/logs/${LOG_FILE_NAME}/${LOG_FILE_NAME}.%d{yyyy-MM-dd}.%i.log
            </FileNamePattern>
            <!--日志文件保留天数，默认30天 -->
            <maxFileSize>100MB</maxFileSize>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <appender name="errorfile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/home/logs/${LOG_FILE_NAME}/${LOG_FILE_NAME}_ERROR.log</file>
        <rollingPolicy
                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件路径和文件名，不要修改 -->
            <FileNamePattern>/home/logs/${LOG_FILE_NAME}_ERROR/${LOG_FILE_NAME}.%d{yyyy-MM-dd}.log
            </FileNamePattern>
            <!--日志文件保留天数，默认30天 -->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <logger name="com.netflix.discovery" level="WARN"/>
    <logger name="com.netflix.loadbalancer" level="WARN"/>
    <logger name="org.springframework" level="INFO"/>
    <logger name="org.apache" level="WARN"/>
    <logger name="com.shys" level="DEBUG"/>

    <logger name="druid.sql" level="DEBUG">
        <appender-ref ref="stdout"/>
        <appender-ref ref="logfile"/>
    </logger>

    <!--<logger name="com.ctrip" level="INFO">
    </logger>
-->
    <!-- <logger name="org.apache.ibatis" level="ERROR"> <appender-ref ref="stdout"
        /> <appender-ref ref="logfile" /> </logger> 就是这个监控了mybatis日志输出，配合上面的“dao”
        <logger name="mybatisDao" level="DEBUG"/> -->

    <!-- <logger name="java.sql.PreparedStatement" level="DEBUG"> <appender-ref
        ref="stdout" /> <appender-ref ref="logfile" /> </logger> -->

    <!-- <logger name="org.spring.springboot.dao" level="DEBUG"> <appender-ref
        ref="stdout"/> </logger> -->

    <logger name="org.redisson" level="ERROR">
        <appender-ref ref="stdout"/>
    </logger>

    <root level="DEBUG">
        <appender-ref ref="stdout"/>
        <appender-ref ref="logfile"/>
        <appender-ref ref="errorfile"/>
        <!-- <appender-ref ref="logstash" /> -->
    </root>

</configuration>

```

```java
package com.shys.iiot.rzp.sync.utils;

import ch.qos.logback.classic.pattern.ClassicConverter;
import ch.qos.logback.classic.spi.ILoggingEvent;

import java.util.concurrent.atomic.AtomicInteger;

public class LocalSeqNumberConverter extends ClassicConverter {

    long lastTimestamp = 0;

    AtomicInteger sequenceNumber = new AtomicInteger(1);

    @Override
    public String convert(ILoggingEvent le) {
        long now = le.getTimeStamp();
        if (lastTimestamp != now) {
            lastTimestamp = now;
            sequenceNumber.getAndSet(1);
        }

        return Long.toString(sequenceNumber.getAndIncrement());
    }

}


```

