---
title: Spring Boot Log4J2
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---
## 1. Springboot log4j2

The apache software foundation 에서는 Log4j 는 개발이 중단되었기 때문에, Log4j 2를 사용하기를 권고하고 있습니다.

> Apache™ Logging Services™ Project Announces Log4j™ 1 End-Of-Life; Recommends Upgrade to Log4j 2





Log4j2 뿐만 아니라, Logback 또한 로깅의 대안이 될 수는 있지만, 성능면에서는 Log4j2가 더 좋습니다.

<br/>

![async-throughput-comparison](/assets/images/springboot/log4j2/async-throughput-comparison.png)



<br/>

이런 이유로 만약, 스프링부트에서 로깅을 고려 해야 하는 상황이라면 `Log4j2` 가 좋은 대안입니다. 



<br/>

#### 1-1. pom.xml 

json 포멧의 로깅을 하기 위해서 `log4j2-logstash-layout` 라이브러리를 추가하고,

스프링 부트에서 log4j2를 사용하기 위해서 `spring-boot-starter-log4j2`를 추가합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>spring-boot-log4j2</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-log4j2</name>
    <description>Spring Boot Log4j2 Example</description>

    <properties>
        <java.version>11</java.version>
        <jackson.version>2.11.0</jackson.version>
        <lombok.version>1.18.12</lombok.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <!-- 스프링 부트에서 기본 제공하는 Logback을 제거함. -->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <version>2.4.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-collections4</artifactId>
            <version>4.4</version>
        </dependency>

        <!-- 스프링부트에서 log4j2를 사용하기 위해서 추가 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

        <!-- json 포멧으로 로깅하기 위해서, log4j2-logstash-layout 추가 -->
        <dependency>
            <groupId>com.vlkan.log4j2</groupId>
            <artifactId>log4j2-logstash-layout</artifactId>
            <version>1.0.3</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <!--
                junit-vintage-engine is included by default to allows easy migration from Junit 4 to Junit 5,
                by allowing both Junit 4 and Junit 5 based tests to run in parallel.
                You can exclude junit-vintage-engine if you do not have any Junit 4 based testcase in your application.
            -->
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-jsr310 -->
        <dependency>
            <groupId>com.fasterxml.jackson.datatype</groupId>
            <artifactId>jackson-datatype-jsr310</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.0</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```



#### 1-2. /src/main/resources/log4j2.xml

Console : 콘솔 로깅

RollingFile : 파일 로깅

RollingJsonFile : json 포멧 파일 로깅

RollingXmlFile : xml 포멧 파일 로깅

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Properties>
        <Property name="layoutPattern">%d{yyyy/MM/dd HH:mm:ss.SSS} [%t] %-5p %c(%M:%L) - %m%n</Property>
    </Properties>

    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${layoutPattern}" />
        </Console>

        <RollingFile name="RollingFile"
                     fileName="./logs/app.log"
                     filePattern="./logs/$${date:yyyy_MM}/app_%d{yyyyMMdd}_%i.log">
            <PatternLayout>
                <pattern>%d %p %C{1.} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!-- rollover on startup, daily and when the file reaches 10 MegaBytes -->
                <OnStartupTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="2 MB">
                </SizeBasedTriggeringPolicy>
                <TimeBasedTriggeringPolicy />
            </Policies>
            <DefaultRolloverStrategy max="5" />
        </RollingFile>

        <RollingFile name="RollingJsonFile"
                     fileName="./logs/app.json"
                     filePattern="./logs/$${date:yyyy_MM}/app_%d{yyyyMMdd}_%i.json">
            <Policies>
                <!-- rollover on startup, daily and when the file reaches 10 MegaBytes -->
                <OnStartupTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="2 MB">
                </SizeBasedTriggeringPolicy>
                <TimeBasedTriggeringPolicy />
            </Policies>
            <DefaultRolloverStrategy max="5" />

            <!--
            <JsonLayout complete="false" locationInfo="true"  includeTimeMillis="true" compact="false"  eventEol="true">
                <KeyValuePair key="myCustomField" value="myCustomValue" />
                <KeyValuePair key="timestampFormat" value="$${date:yyyy-MM-dd'T'HH:mm:ss.SSSZ}" />
            </JsonLayout>
            -->
            <LogstashLayout dateTimeFormatPattern="yyyy-MM-dd'T'HH:mm:ss.SSSZZZ"
                            eventTemplateUri="classpath:LogstashJsonEventLayoutV1.json"
                            locationInfoEnabled="true"
                            emptyPropertyExclusionEnabled="true"
                            prettyPrintEnabled="true"
                            stackTraceEnabled="true"/>
        </RollingFile>

        <RollingFile name="RollingXmlFile"
                     fileName="./logs/app.xml"
                     filePattern="./logs/$${date:yyyy_MM}/app_%d{yyyyMMdd}_%i.xml">
            <Policies>
                <!-- rollover on startup, daily and when the file reaches 10 MegaBytes -->
                <OnStartupTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="2 MB">
                </SizeBasedTriggeringPolicy>
                <TimeBasedTriggeringPolicy />
            </Policies>
            <DefaultRolloverStrategy max="5" />

            <XMLLayout>
                <KeyValuePair key="myCustomField" value="myCustomValue" />
            </XMLLayout>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- LOG everything at INFO level -->
        <Root level="info">
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFile" />
            <AppenderRef ref="RollingJsonFile" />
            <AppenderRef ref="RollingXmlFile" />
        </Root>

        <!-- LOG "com.example*" at TRACE level -->
        <Logger name="com.example" level="info" />
        <Logger name="com.example.app.ElasticsearchIndexTests" level="debug" />
        <Logger name="com.example.app.SpringBootConsoleApplicationTests" level="debug" />
    </Loggers>

</Configuration>
```



<br/>

#### 1-3. Log4j2 사용 예제

```java
package com.example.app;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;

@Slf4j
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
public class SpringBootConsoleApplicationTests {

    @Test
    public void logTest() throws Exception {
        log.info("hello log4j2");

        try {
            throw new RuntimeException("my exception");
        } catch (Exception e) {
            log.error("", e);
        }
    }
}
```



<br/>

#### 1-4. Log4j2 실행 결과

app.log

```
2020-07-20 16:26:16,126 INFO o.s.b.StartupInfoLogger [main] Started SpringBootConsoleApplicationTests in 1.235 seconds (JVM running for 4.721)
2020-07-20 16:26:16,614 INFO c.e.a.SpringBootConsoleApplicationTests [main] hello log4j2
2020-07-20 16:26:16,615 ERROR c.e.a.SpringBootConsoleApplicationTests [main] 
java.lang.RuntimeException: my exception
	at com.example.app.SpringBootConsoleApplicationTests.logTest(SpringBootConsoleApplicationTests.java:23) ~[test-classes/:?]
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[?:?]
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[?:?]
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:?]
	at java.lang.reflect.Method.invoke(Method.java:566) ~[?:?]
	at org.junit.platform.commons.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:686) ~[junit-platform-commons-1.6.2.jar:1.6.2]
	at org.junit.jupiter.engine.execution.MethodInvocation.proceed(MethodInvocation.java:60) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation.proceed(InvocationInterceptorChain.java:131) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.extension.TimeoutExtension.intercept(TimeoutExtension.java:149) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestableMethod(TimeoutExtension.java:140) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestMethod(TimeoutExtension.java:84) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall.lambda$ofVoidMethod$0(ExecutableInvoker.java:115) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.ExecutableInvoker.lambda$invoke$0(ExecutableInvoker.java:105) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation.proceed(InvocationInterceptorChain.java:106) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.proceed(InvocationInterceptorChain.java:64) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.chainAndInvoke(InvocationInterceptorChain.java:45) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.invoke(InvocationInterceptorChain.java:37) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:104) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:98) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$invokeTestMethod$6(TestMethodTestDescriptor.java:212) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.invokeTestMethod(TestMethodTestDescriptor.java:208) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:137) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:71) ~[junit-jupiter-engine-5.6.2.jar:5.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:135) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at java.util.ArrayList.forEach(ArrayList.java:1540) ~[?:?]
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:38) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:139) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at java.util.ArrayList.forEach(ArrayList.java:1540) ~[?:?]
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:38) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:139) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:32) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:51) ~[junit-platform-engine-1.6.2.jar:1.6.2]
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:248) ~[junit-platform-launcher-1.6.2.jar:1.6.2]
	at org.junit.platform.launcher.core.DefaultLauncher.lambda$execute$5(DefaultLauncher.java:211) ~[junit-platform-launcher-1.6.2.jar:1.6.2]
	at org.junit.platform.launcher.core.DefaultLauncher.withInterceptedStreams(DefaultLauncher.java:226) [junit-platform-launcher-1.6.2.jar:1.6.2]
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:199) [junit-platform-launcher-1.6.2.jar:1.6.2]
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:132) [junit-platform-launcher-1.6.2.jar:1.6.2]
	at com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:69) [junit5-rt.jar:?]
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33) [junit-rt.jar:?]
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230) [junit-rt.jar:?]
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58) [junit-rt.jar:?]
```





app.json

```json
{
  "line_number" : 20,
  "class" : "com.example.app.SpringBootConsoleApplicationTests",
  "@version" : 1,
  "source_host" : "N19074_W1",
  "message" : "hello log4j2",
  "thread_name" : "main",
  "@timestamp" : "2020-07-20T16:26:16.614+09:00",
  "level" : "INFO",
  "file" : "SpringBootConsoleApplicationTests.java",
  "method" : "logTest",
  "logger_name" : "com.example.app.SpringBootConsoleApplicationTests"
}
{
  "exception" : {
    "exception_class" : "java.lang.RuntimeException",
    "exception_message" : "my exception",
    "stacktrace" : "java.lang.RuntimeException: my exception\r\n\tat com.example.app.SpringBootConsoleApplicationTests.logTest(SpringBootConsoleApplicationTests.java:23)\r\n\tat java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.base/java.lang.reflect.Method.invoke(Method.java:566)\r\n\tat org.junit.platform.commons.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:686)\r\n\tat org.junit.jupiter.engine.execution.MethodInvocation.proceed(MethodInvocation.java:60)\r\n\tat org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation.proceed(InvocationInterceptorChain.java:131)\r\n\tat org.junit.jupiter.engine.extension.TimeoutExtension.intercept(TimeoutExtension.java:149)\r\n\tat org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestableMethod(TimeoutExtension.java:140)\r\n\tat org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestMethod(TimeoutExtension.java:84)\r\n\tat org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall.lambda$ofVoidMethod$0(ExecutableInvoker.java:115)\r\n\tat org.junit.jupiter.engine.execution.ExecutableInvoker.lambda$invoke$0(ExecutableInvoker.java:105)\r\n\tat org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation.proceed(InvocationInterceptorChain.java:106)\r\n\tat org.junit.jupiter.engine.execution.InvocationInterceptorChain.proceed(InvocationInterceptorChain.java:64)\r\n\tat org.junit.jupiter.engine.execution.InvocationInterceptorChain.chainAndInvoke(InvocationInterceptorChain.java:45)\r\n\tat org.junit.jupiter.engine.execution.InvocationInterceptorChain.invoke(InvocationInterceptorChain.java:37)\r\n\tat org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:104)\r\n\tat org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:98)\r\n\tat org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$invokeTestMethod$6(TestMethodTestDescriptor.java:212)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.invokeTestMethod(TestMethodTestDescriptor.java:208)\r\n\tat org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:137)\r\n\tat org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:71)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:135)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125)\r\n\tat org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80)\r\n\tat java.base/java.util.ArrayList.forEach(ArrayList.java:1540)\r\n\tat org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:38)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:139)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125)\r\n\tat org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80)\r\n\tat java.base/java.util.ArrayList.forEach(ArrayList.java:1540)\r\n\tat org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:38)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:139)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125)\r\n\tat org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123)\r\n\tat org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122)\r\n\tat org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80)\r\n\tat org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:32)\r\n\tat org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)\r\n\tat org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:51)\r\n\tat org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:248)\r\n\tat org.junit.platform.launcher.core.DefaultLauncher.lambda$execute$5(DefaultLauncher.java:211)\r\n\tat org.junit.platform.launcher.core.DefaultLauncher.withInterceptedStreams(DefaultLauncher.java:226)\r\n\tat org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:199)\r\n\tat org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:132)\r\n\tat com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:69)\r\n\tat com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)\r\n\tat com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230)\r\n\tat com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58)\r\n"
  },
  "line_number" : 25,
  "class" : "com.example.app.SpringBootConsoleApplicationTests",
  "@version" : 1,
  "source_host" : "N19074_W1",
  "thread_name" : "main",
  "@timestamp" : "2020-07-20T16:26:16.615+09:00",
  "level" : "ERROR",
  "file" : "SpringBootConsoleApplicationTests.java",
  "method" : "logTest",
  "logger_name" : "com.example.app.SpringBootConsoleApplicationTests"
}
```

app.xml

```xml
  <Event xmlns="http://logging.apache.org/log4j/2.0/events" timeMillis="1595229976614" thread="main" level="INFO" loggerName="com.example.app.SpringBootConsoleApplicationTests" endOfBatch="false" loggerFqcn="org.apache.logging.slf4j.Log4jLogger" threadId="1" threadPriority="5">
    <Instant epochSecond="1595229976" nanoOfSecond="614357900"/>
    <Message>hello log4j2</Message>
    <myCustomField>myCustomValue</myCustomField>
  </Event>

  <Event xmlns="http://logging.apache.org/log4j/2.0/events" timeMillis="1595229976615" thread="main" level="ERROR" loggerName="com.example.app.SpringBootConsoleApplicationTests" endOfBatch="false" loggerFqcn="org.apache.logging.slf4j.Log4jLogger" threadId="1" threadPriority="5">
    <Instant epochSecond="1595229976" nanoOfSecond="615356000"/>
    <Message></Message>
    <Thrown commonElementCount="0" localizedMessage="my exception" message="my exception" name="java.lang.RuntimeException">
      <ExtendedStackTrace>
        <ExtendedStackTraceItem class="com.example.app.SpringBootConsoleApplicationTests" method="logTest" file="SpringBootConsoleApplicationTests.java" line="23" exact="false" location="test-classes/" version="?"/>
        <ExtendedStackTraceItem class="jdk.internal.reflect.NativeMethodAccessorImpl" method="invoke0" file="NativeMethodAccessorImpl.java" line="-2" exact="false" location="?" version="?"/>
        <ExtendedStackTraceItem class="jdk.internal.reflect.NativeMethodAccessorImpl" method="invoke" file="NativeMethodAccessorImpl.java" line="62" exact="false" location="?" version="?"/>
        <ExtendedStackTraceItem class="jdk.internal.reflect.DelegatingMethodAccessorImpl" method="invoke" file="DelegatingMethodAccessorImpl.java" line="43" exact="false" location="?" version="?"/>
        <ExtendedStackTraceItem class="java.lang.reflect.Method" method="invoke" file="Method.java" line="566" exact="false" location="?" version="?"/>
        <ExtendedStackTraceItem class="org.junit.platform.commons.util.ReflectionUtils" method="invokeMethod" file="ReflectionUtils.java" line="686" exact="false" location="junit-platform-commons-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.MethodInvocation" method="proceed" file="MethodInvocation.java" line="60" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation" method="proceed" file="InvocationInterceptorChain.java" line="131" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.extension.TimeoutExtension" method="intercept" file="TimeoutExtension.java" line="149" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.extension.TimeoutExtension" method="interceptTestableMethod" file="TimeoutExtension.java" line="140" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.extension.TimeoutExtension" method="interceptTestMethod" file="TimeoutExtension.java" line="84" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall" method="lambda$ofVoidMethod$0" file="ExecutableInvoker.java" line="115" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.ExecutableInvoker" method="lambda$invoke$0" file="ExecutableInvoker.java" line="105" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation" method="proceed" file="InvocationInterceptorChain.java" line="106" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.InvocationInterceptorChain" method="proceed" file="InvocationInterceptorChain.java" line="64" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.InvocationInterceptorChain" method="chainAndInvoke" file="InvocationInterceptorChain.java" line="45" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.InvocationInterceptorChain" method="invoke" file="InvocationInterceptorChain.java" line="37" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.ExecutableInvoker" method="invoke" file="ExecutableInvoker.java" line="104" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.execution.ExecutableInvoker" method="invoke" file="ExecutableInvoker.java" line="98" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor" method="lambda$invokeTestMethod$6" file="TestMethodTestDescriptor.java" line="212" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor" method="invokeTestMethod" file="TestMethodTestDescriptor.java" line="208" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor" method="execute" file="TestMethodTestDescriptor.java" line="137" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor" method="execute" file="TestMethodTestDescriptor.java" line="71" exact="false" location="junit-jupiter-engine-5.6.2.jar" version="5.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$5" file="NodeTestTask.java" line="135" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$7" file="NodeTestTask.java" line="125" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.Node" method="around" file="Node.java" line="135" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$8" file="NodeTestTask.java" line="123" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="executeRecursively" file="NodeTestTask.java" line="122" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="execute" file="NodeTestTask.java" line="80" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="java.util.ArrayList" method="forEach" file="ArrayList.java" line="1540" exact="false" location="?" version="?"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService" method="invokeAll" file="SameThreadHierarchicalTestExecutorService.java" line="38" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$5" file="NodeTestTask.java" line="139" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$7" file="NodeTestTask.java" line="125" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.Node" method="around" file="Node.java" line="135" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$8" file="NodeTestTask.java" line="123" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="executeRecursively" file="NodeTestTask.java" line="122" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="execute" file="NodeTestTask.java" line="80" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="java.util.ArrayList" method="forEach" file="ArrayList.java" line="1540" exact="false" location="?" version="?"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService" method="invokeAll" file="SameThreadHierarchicalTestExecutorService.java" line="38" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$5" file="NodeTestTask.java" line="139" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$7" file="NodeTestTask.java" line="125" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.Node" method="around" file="Node.java" line="135" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="lambda$executeRecursively$8" file="NodeTestTask.java" line="123" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.ThrowableCollector" method="execute" file="ThrowableCollector.java" line="73" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="executeRecursively" file="NodeTestTask.java" line="122" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.NodeTestTask" method="execute" file="NodeTestTask.java" line="80" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService" method="submit" file="SameThreadHierarchicalTestExecutorService.java" line="32" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor" method="execute" file="HierarchicalTestExecutor.java" line="57" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine" method="execute" file="HierarchicalTestEngine.java" line="51" exact="false" location="junit-platform-engine-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.launcher.core.DefaultLauncher" method="execute" file="DefaultLauncher.java" line="248" exact="false" location="junit-platform-launcher-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.launcher.core.DefaultLauncher" method="lambda$execute$5" file="DefaultLauncher.java" line="211" exact="false" location="junit-platform-launcher-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.launcher.core.DefaultLauncher" method="withInterceptedStreams" file="DefaultLauncher.java" line="226" exact="true" location="junit-platform-launcher-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.launcher.core.DefaultLauncher" method="execute" file="DefaultLauncher.java" line="199" exact="true" location="junit-platform-launcher-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="org.junit.platform.launcher.core.DefaultLauncher" method="execute" file="DefaultLauncher.java" line="132" exact="true" location="junit-platform-launcher-1.6.2.jar" version="1.6.2"/>
        <ExtendedStackTraceItem class="com.intellij.junit5.JUnit5IdeaTestRunner" method="startRunnerWithArgs" file="JUnit5IdeaTestRunner.java" line="69" exact="true" location="junit5-rt.jar" version="?"/>
        <ExtendedStackTraceItem class="com.intellij.rt.junit.IdeaTestRunner$Repeater" method="startRunnerWithArgs" file="IdeaTestRunner.java" line="33" exact="true" location="junit-rt.jar" version="?"/>
        <ExtendedStackTraceItem class="com.intellij.rt.junit.JUnitStarter" method="prepareStreamsAndStart" file="JUnitStarter.java" line="230" exact="true" location="junit-rt.jar" version="?"/>
        <ExtendedStackTraceItem class="com.intellij.rt.junit.JUnitStarter" method="main" file="JUnitStarter.java" line="58" exact="true" location="junit-rt.jar" version="?"/>
      </ExtendedStackTrace>
    </Thrown>
    <myCustomField>myCustomValue</myCustomField>
  </Event>
```



