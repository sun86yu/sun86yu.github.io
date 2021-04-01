---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: springboot多套环境切换打包
tags:
- springboot
- maven
- 多环境
---

介绍
===
开发部署时涉及到打包。但通常我们的开发环境、测试环境、正式环境都是分开的。在部署的时候就要特别小心。这里提供一种思路：在项目中配置多套环境，多个配置文件。要部署到不同的环境时，使用不同的配置文件打包。

构建
===

pom.xml 中添加:

```
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <!-- 测试 -->
    <profile>
        <id>test</id>
        <properties>
            <spring.profiles.active>test</spring.profiles.active>
        </properties>
    </profile>
    <!-- 发布 -->
    <profile>
        <id>release</id>
        <properties>
            <spring.profiles.active>release</spring.profiles.active>
        </properties>
    </profile>
</profiles>

<!--spring boot打包指定一个唯一的入口-->
<build>
    <plugins>
        <!--多环境处理-->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <version>${maven-resources-plugin.version}</version>
            <executions>
                <execution>
                    <id>default-resources</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/classes</outputDirectory>
                        <useDefaultDelimiters>false</useDefaultDelimiters>
                        <delimiters>
                            <delimiter>#</delimiter>
                        </delimiters>
                        <resources>
                            <resource>
                                <directory>src/main/resources/</directory>
                                <filtering>true</filtering>
                                <includes>
                                    <include>**/*.xml</include>
                                    <include>**/*.yml</include>
                                </includes>
                            </resource>
                            <resource>
                                <directory>src/main/resources/</directory>
                                <filtering>false</filtering>
                                <excludes>
                                    <exclude>**/*.xml</exclude>
                                    <exclude>**/*.yml</exclude>
                                </excludes>
                            </resource>
                        </resources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>

    <resources>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>

</build>
```

这里定义了三种环境。将对应三个配置文件。并且在 plugins 里加了上多环境处理插件。

在项目的 src/main/resource 目录下有四个文件：

application.yml, 这个是公共的配置。不管需要哪个环境，该文件都会被加载，所以可以放一些能用的，不会随环境切换变化的配置。内容可能如下：

```
server:
  port: 8080
spring:
  profiles:
    active: #spring.profiles.active#
  thymeleaf:
    prefix: classpath:/templates/
  datasource:
    name: ga
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    filters: stat
    maxActive: 20
    initialSize: 1
    maxWait: 60000
    minIdle: 1
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 'x'
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxOpenPreparedStatements: 20
  servlet:
    multipart:
      max-file-size: 50MB
      max-request-size: 50MB
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.gongan.model

pagehelper:
  helper-dialect: mysql
  reasonable: true
  support-methods-arguments: true
  params: count=countSql

logging:
  level:
    root: info
    com.gongan.dao : debug

weblogPath: ${logging.path}/demo.log
httpHeaderName: Authorization
fileUploadSize: 31457280
```

其它三套环境则对应不同的配置文件：

application-dev.yml
---

```
spring:
  datasource:
    url: jdbc:mysql://172.27.100.105/ga
    username: root
    password: 123456
  data:
    neo4j:
      uri: http://172.27.100.105:7474
      username: neo4j
      password: 123456

filePath: /Users/sunyu/Documents/log/insight
filePath1: /Users/sunyu/Documents/log/insight/data/
csvPath: /Users/sunyu/Developer/bigdata

logging:
  file: /Users/sunyu/Documents/log/demo.log

logging.path: /Users/sunyu/Documents/log/insight/log
```

application-test.yml
---

```
spring:
  datasource:
    url: jdbc:mysql://localhost/ga
    username: root
    password: 123456
  data:
    neo4j:
      uri: http://localhost:7474
      username: neo4j
      password: 123456

filePath: D:\insight\log
filePath1: D:\insight\fileupload
csvPath: D:\insight\neo4j-community-3.4.10\import

logging:
  file: D:\insight\log\demo.log

logging.path: D:\insight\log
```

application-release
---

```
spring:
  datasource:
    url: jdbc:mysql://172.27.100.113/ga
    username: root
    password: 123456
  data:
    neo4j:
      uri: http://192.168.100.113:7474
      username: neo4j
      password: 123456

filePath: /Users/sunyu/Documents/log/insight
filePath1: /Users/sunyu/Documents/log/insight/data/
csvPath: /Users/sunyu/Developer/bigdata

logging:
  file: /Users/sunyu/Documents/log/demo.log

logging.path: /Users/sunyu/Documents/log/insight/log
```

打包时，可以使用命令参数打对应的包：

```
mvn package -Ptest
mvn package -Prelease
```

本地 debug 也可以启动不同的环境:

![](/images/mutienv_01.jpg)

这里 ```-Dspring.profiles.active=dev```里 ```spring.profiles.active```的值就是 application.yml 里这一项配置的值: ```spring.profiles.active: #spring.profiles.active#```

也就是说，我们也可以通过改这里的配置进行打包时的环境配置。
