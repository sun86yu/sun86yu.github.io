---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: springboot多模块项目构建
tags:
- springboot
- 多模块
---

介绍
===
基于 Spring 框架。以注解为主进行开发。并嵌入 Tomcat、Jetty 或者 Undertow，无须部署  WAR 文件。可以通过 maven 来根据需要获取 starter。

多模块的构建要依赖于 maven 的 pom 配置。最终的项目结构如下：

![](/images/springboot/muti-1.jpg)

模块
===

测试项目分 ```common```, ```model```, ```service```, ```web```几个模块。分别表示能用功能（可以独立建git地址，公司多个项目通用该模块），数据层、逻辑层、展现层。每个模块有自己单独的 ```pom.xml```，配置自己模块自己的依赖。整个项目也有自己的```pom.xml```，用来把各个模块整理起来。

如果项目最终发布时只想生成一个整的 jar 包，对外提供 WEB 服务，则只需要在 ```web```模块里添加打包的配置。

整体项目
---

pom.xml 内容:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sunyu</groupId>
    <artifactId>base-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 表示是一个被继承的模块 -->
    <packaging>pom</packaging>

    <!-- 声明所有模块 -->
    <modules>
        <module>pro-common</module>
        <module>pro-model</module>
        <module>pro-service</module>
        <module>pro-web</module>
    </modules>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>

    <dependencies>
        <!-- Spring boot 核心web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.2</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
    </dependencies>

    <!--指定使用maven打包-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.20.1</version>
                <configuration>
                    <!--默认关掉单元测试 -->
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

common 模块
---

pom.xml 内容:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.sunyu</groupId>
        <artifactId>base-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatisplus-spring-boot-starter</artifactId>
            <version>1.0.5</version>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>2.9.9</version>
        </dependency>

    </dependencies>

    <artifactId>pro-common</artifactId>
    <packaging>jar</packaging>

</project>
```

model 模块
---

pom.xml 内容:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.sunyu</groupId>
        <artifactId>base-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>pro-model</artifactId>
    <packaging>jar</packaging>

    <dependencies>

        <dependency>
            <groupId>com.sunyu</groupId>
            <artifactId>pro-common</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
            <version>5.1.30</version>
        </dependency>

    </dependencies>
</project>
```

>可以看到，model 模块继承 common 模块。

service 模块
---

pom.xml 内容:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.sunyu</groupId>
        <artifactId>base-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>pro-service</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <!-- pro-service 需要使用到 pro-model 中的类，所以需要添加对 pro-model 模块的依赖 -->
        <dependency>
            <groupId>com.sunyu</groupId>
            <artifactId>pro-model</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>

</project>
```

web 模块
---

pom.xml 内容:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.sunyu</groupId>
        <artifactId>base-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>pro-web</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <!-- pro-web 需要使用到 pro-service 中的类，所以需要添加对 pro-service 模块的依赖.pro-service 依赖的包也会被引入 -->
        <dependency>
            <groupId>com.sunyu</groupId>
            <artifactId>pro-service</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.0</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.33</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.7.0</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.7.0</version>
        </dependency>

    </dependencies>

    <!--spring boot打包指定一个唯一的入口-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- 指定该Main Class为全局的唯一入口 -->
                    <mainClass>com.sunyu.web.WebApplication</mainClass>
                    <layout>ZIP</layout>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <!--可以把依赖的包都打包到生成的Jar包中-->
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

构建
===

由于项目最终打包成一个整体的 jar，而且启动类在 web 模块中，所以将配置文件放在 web 的 resource 目录下。

配置文件 ```application.yml``` 内容是:

```
spring:
  output:
    ansi:
      enabled: always
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
  datasource:
    url: jdbc:mysql://localhost/mydb
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
    maxActive: 20
    initialSize: 1
    maxWait: 60000
    minIdle: 1
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    poolPreparedStatements: true
    maxOpenPreparedStatements: 20
  cache:
    type: REDIS
    cache-names: redisCache
mybatis-plus:
  mapper-locations: classpath:/mapper/*.xml
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.sunyu.model.entity
  #自动处理枚举类型
  typeEnumsPackage: com.sunyu.model.entity.enums
  global-config:
    # 数据库相关配置
    db-config:
      #主键类型  AUTO:"数据库ID自增", INPUT:"用户输入ID",ID_WORKER:"全局唯一ID (数字类型唯一ID)", UUID:"全局唯一ID UUID";
      id-type: id_worker
      #字段策略 IGNORED:"忽略判断",NOT_NULL:"非 NULL 判断"),NOT_EMPTY:"非空判断"
      field-strategy: not_empty
      #驼峰下划线转换
      column-underline: true
      #逻辑删除配置
      logic-delete-value: 0
      logic-not-delete-value: 1
  # 原生配置
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: false
debug: false
jwt:
  # 加密秘钥
  secret: kpi
  # token有效时长，7天，单位秒
  expire: 604800
  header: token

logging:
  level:
    com.idss.cop_kpi.mapper: debug
  file: /data/logs/base-project.log

```

项目启动类: ```WebApplication.java```:

```
package com.idss.capital.web;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@SpringBootApplication(scanBasePackages = {
        "com.sunyu",
})
@EnableSwagger2
public class WebApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
```

数据库表对应的 xml 文件则放在 model 模块的 resource 目录下。

各个被依赖的项目模块也要有自己的模块启动类，如 ```model```模块的启动类 ```ModelApplication.java```:

```
package com.sunyu.model;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ModelApplication {

    public static void main(String[] args) {
        SpringApplication.run(ModelApplication.class, args);
    }
}
```

同理，service 模块也要有。

>有了启动类，在打包时该模块会被打成 jar 包，然后最终放到一个大包里。