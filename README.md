# Spring Cloud
## 01-Spring Security安全框架

|                          maven地址                           | artifactId(jar包)         | jar包作用         |
| :----------------------------------------------------------: | ------------------------- | ----------------- |
| https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security | spring-boot-starter-security      | spring security    |



---
# 数据库jar包

|                          maven地址                           | artifactId(jar包)         | jar包作用         |
| :----------------------------------------------------------: | ------------------------- | ----------------- |
| https://mvnrepository.com/artifact/com.mysql/mysql-connector-j | mysql-connector-j      | java数据库驱动    |
| https://mvnrepository.com/artifact/mysql/mysql-connector-java | mysql-connector-java      | java数据库驱动(官方)    |
|     https://mvnrepository.com/artifact/com.alibaba/druid     | druid                     | druid数据库连接池 |
| https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter | mybatis-plus-boot-starter | mybatis-plus      |
|    https://mvnrepository.com/artifact/com.github.yulichang/mybatis-plus-join    |  mybatis-plus-join                  |  mybatis-plus-join           |

### properties与yml配置文件任选其一

#### application.properties文件--mybatis配置及druid连接池配置

```properties
# 后台测试端口
server.port=8088

# 数据库连接配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/your_database?useSSL=false&serverTimeZone=Asia/Shanghai
spring.datasource.username=your_username
spring.datasource.password=your_password
# 配置Druid连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

# 配置xml的路径---resources目录下的mappers文件夹
mybatis.mapper-locations=classpath*:mappers/**/*.xml
#mybatis需要手动开启驼峰命名支持,mybatis-plus默认是开启的
mybatis.configuration.map-underscore-to-camel-case=true

# Mybatis日志xx.xx.mapper.*为映射的接口
logging.level.xx.xx.mapper.*=debug

#指定mybatis-plus的日志输出格式
mybatis-plus.configuration.log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

```


#### application.yml文件--mybatis配置及druid连接池配置
```yml
# 应用服务器(Tomcat)WEB端口号
# yml结构 相当于properties文件中的server.port=8088
server:
  port: 8088

# 加载当前的开发环境配置
spring:
  profiles:
    active: dev
```
> 在实际使用中合理结合,不过mybatis-plus默认集成mybatis,换句话说,引入mybatis-plus依赖后,mybatis照常能用,建议配置都加进来
#### application-dev.yml文件--mybatis配置及druid连接池配置
```yml

spring:
  # 数据库连接配置
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/your_database?useSSL=false&serverTimeZone=Asia/Shanghai
    username: your_username
    password: your_password
# 文件上传配置默认为1MB
  servlet:
    multipart:
      max-file-size: 10MB

# mybatis-plus配置
mybatis-plus:
  mapper-locations: classpath*:mappers/**/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# mybatis配置
mybatis:
  mapper-locations: classpath*:mappers/**/*.xml
  configuration:
    # 开启驼峰命名支持
    map-underscore-to-camel-case: true
  
#mybatis日志
logging:
  level:
    xxx:
      xxx:
        xxx: debug
```

#### bootstrap.yml文件--SpringCloud配置
##### others-service
```yml
spring:
  cloud:
    nacos:
      discovery:
        username: nacos
        password: nacos
        server-addr: ip:8848
        namespace: namespaceId
      config:
        username: nacos
        password: nacos
        file-extension: yaml
        server-addr: ip:8848
        namespace: namespaceId
  profiles:
    active: dev # 环境设置
  application:
    name: user-service # 服务名
```

##### gateway-service
```yml
spring:
  cloud:
    nacos:
      discovery:
        username: nacos
        password: nacos
        server-addr: ip:8848
        namespace: namespaceId
      config:
        username: nacos
        password: nacos
        file-extension: yaml
        server-addr: ip:8848
        namespace: namespaceId
    gateway:
      discovery:
        locator:
          enabled: true
      routes: # 路由
        - id: user-service #路由ID，没有固定要求，但是要保证唯一，建议配合服务名
          uri: lb://user-service # 匹配提供服务的路由地址
          predicates: # 断言
            - Path=/api/user/** # 断言，路径相匹配进行路由
            # http://localhost:8082/api/user/** 路由到 http://localhost:8084/api/user/**
            # 从而实现了用户访问网关，网关把各个请求路由到各种微服务中
  profiles:
    active: dev # 环境设置
  application:
    name: gateway-service

```
#### 更多关于Spring Gateway的用法请阅读手册: https://springdoc.cn/spring-cloud-gateway/

#### Mybatis-plus分页查询通用配置模板类
```java
package xxx.xxx.xxx.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor paginationInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        /**
         * DbType.XXX
         * XXX 为具体的数据库类型如MYSQL,ORACLE
         */
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```
---
### Mybatis-plus的使用
#### Service层
```java
// service接口需要继承IService<XXX>接口
public interface XXXService extends IService<XXX> {

}
// service实现类需要继承ServiceImpl<XXXMapper, User>接口
public class XXXServiceImpl extends ServiceImpl<XXXMapper, XXX> implements XXXService {

}
```

#### Mapper层
```java
public interface XXXMapper extends BaseMapper<User> {

}
```
---
### Swagger2通用配置模板类------Swagger2是能够以REST风格方式在本地网页测试接口的工具
```java
package xxx.xxx.xxx.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spi.service.contexts.SecurityContext;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.List;

@Configuration
@EnableSwagger2
public class Swagger2Config {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("xxx.xxx.xxx.controller"))
                .paths(PathSelectors.any())
                .build()
                //配置全局登录
                .securityContexts(securityContexts())
                .securitySchemes(securitySchemes());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("项目演示")
                .description("接口文档")
                .contact(new Contact("kakapreter", "http://localhost:8088/doc.html", ""))
                .version("1.0")
                .build();
    }

    private List<ApiKey> securitySchemes() {
        List<ApiKey> apiKeys = new ArrayList<>();
        apiKeys.add(new ApiKey("Authorization", "Authorization", "header"));
        return apiKeys;
    }
    private List<SecurityContext> securityContexts() {
        //设置需要登录认证的路径
        List<SecurityContext> list = new ArrayList<>();
        list.add(SecurityContext.builder().securityReferences(defaultAuth()).forPaths(PathSelectors.any()).build());
        return list;
    }

//    private SecurityContext getContextByPath(String pathRegex) {
//        return SecurityContext.builder()
//                .securityReferences(defaultAuth())
//                .forPaths(PathSelectors.regex(pathRegex))
//                .build();
//    }

    private List<SecurityReference> defaultAuth() {
        List<SecurityReference> list = new ArrayList<>();
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        list.add(new SecurityReference("Authorization", authorizationScopes));
        return list;
    }
}

```

#### aliyun的SpringBoot 2.7.5 Swagger2的jar包依赖
```xml
        <!--swagger2依赖-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.7.0</version>
        </dependency>

        <!--swagger第三方ui依赖-->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>swagger-bootstrap-ui</artifactId>
            <version>1.9.6</version>
        </dependency>
```

#### Swagger2的yml配置
```yml
spring:
  #Swagger2配置
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```

### Swagger默认的访问路径 http://localhost:8080/doc.html
### SpringBoot3的整合看这个文档 https://doc.xiaominfo.com/docs/quick-start

---
#### 基本测试
```java
package xxx.xxx.xxx.baseTest;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
@SpringBootTest
public class BaseTest {

    @Autowired
    DataSource dataSource;

    //测试数据库连接
    @Test
    public void druidTest() throws SQLException {
        //查看默认数据源
        System.out.println(dataSource.getClass());
        //获得数据库连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }

    //测试本地JDK
    @Test
    public void localJavaVersion(){
        System.out.println("当前系统的JDK版本:" + System.getProperty("java.version"));
        System.out.println("当前系统的JDK路径:" + System.getProperty("java.home"));
    }
}
```
#### XxxApplication --SpringBoot启动类模板
```java
package xxx.xxx.xxx;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.Environment;

@SpringBootApplication
@MapperScan("xxx.xxx.xxx.mapper")
public class XxxApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(XxxApplication.class, args);
        Environment environment = context.getBean(Environment.class);
        //单体项目测试
        System.out.println("前台访问链接为：http://localhost:" + environment.getProperty("local.server.port")+"/login.html");
        //前后端分离开发
        //System.out.println("前台访问链接为：http://localhost:8080");
        System.out.println("后台测试环境为: " + environment.getProperty( "spring.profiles.active") + " 占用端口为: "+environment.getProperty("local.server.port"));
        System.out.println("Swagger2接口测试网址为:http://localhost:"+environment.getProperty("local.server.port")+"/swagger-ui/index.html#/");
    }
}
```

#### Mybatis框架Xml文件通用模板
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="xxx.xxx.xxx.mapper.xxxMapper">
  
</mapper>
```

# SpringBoot-Web项目必引jar包

| maven地址                                                    | artifactId(jar包)        | jar包作用 |
| ------------------------------------------------------------ | ------------------------ | --------- |
| https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web | spring-boot-starter-web  | 1         |
| https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test | spring-boot-starter-test | 2         |

> 1.只要在 Spring Boot 项目的 pom.xml 中引入了 spring-boot-starter-web ,就不需进行任何配置，能直接使用 SpringMVC 进行 Web 开发。
>
> 2.Spring Test与JUnit等其他测试框架结合起来，提供了便捷高效的测试手段。而Spring Boot Test 是在Spring Test之上的再次封装，增加了切片测试，增强了mock能力。
>
> ### 日志依赖
>
> 3.spring-boot-starter-logging是Spring Boot 默认的日志框架 Logback + SLF4J 
>
> spring-boot-starter-web 
>
> ​	|-spring-boot-starter
>
> ​		  |-spring-boot-starter-logging
>
> 所以我们只需要引入web组件即可:

### 日志配置

src/main/resources/application.properties
```properties
logging.config=classpath:logback-spring.xml
logging.file.path=./logs/
```

src/main/resources/logback-spring.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="10 seconds">
    <contextName>MyContextName</contextName>
    <property name="LOG_HOME" value="./logs/"/>
    <springProperty name="serverName" source="logging.file.name" defaultValue="MyServerName"/>
    <springProperty name="logging.path" source="logging.file.path" defaultValue="./logs/"/>

    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>

    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
         <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logging.path}/web_debug.log</file>
        <encoder>
            <pattern> DEBUG: %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
          <fileNamePattern>${logging.path}/web-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
             <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
           <maxHistory>15</maxHistory>
        </rollingPolicy>
       <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logging.path}/web_info.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logging.path}/web-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logging.path}/web_warn.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logging.path}/web-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logging.path}/web_error.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logging.path}/web-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <springProfile name="dev">
        <logger name="com.xxx.xxx.controller" level="debug"/>
    </springProfile>
    <!--level = debug,info,warn,error-->
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="DEBUG_FILE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="WARN_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
</configuration>
```
> ### 服务器依赖
>
> 4.spring-boot-starter-tomcat是Spring Boot 默认服务器
>
> spring-boot-starter-web 
>
> ​	|-spring-boot-starter-tomcat
>
> 所以我们只需要引入web组件即可:
>
>### 单元测试依赖
> 
>5.spring-boot-starter-test包含junit-jupiter
> 
> spring-boot-starter-test
>
> ​	|-junit-jupiter        
> 
> 所以我们只需要引入test组件即可:
>

#### Cloud Native App Initializer https://start.spring.io , https://start.aliyun.com

#### 注意SpringBoot3.0.2的支持的mybatis-plus-boot-starter的jar包最低版本为3.5.3

#### 以下是aliyun的SpringBoot 3.0.2基本pom.xml配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId></groupId>
    <artifactId></artifactId>
    <version></version>
    <packaging></packaging>
    <name></name>
    <description></description>
    
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <!--启动类路径-->
                <configuration>
                    <mainClass>xxx.xxx.xxxApplication</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```
# 推荐jar包
| github地址                                                   | artifactId(jar包) | jar包作用 |
| ----------------------------------------------------------- | ----------------- | --------- |
| https://github.com/fengwenyi/mybatis-plus-code-generator | mybatis-plus-code-generator            | 1         |

| maven地址                                                   | artifactId(jar包) | jar包作用 |
| ----------------------------------------------------------- | ----------------- | --------- |
| https://mvnrepository.com/artifact/org.projectlombok/lombok | lombok            | 2         |
| https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools    |spring-boot-devtools     |   web项目热部署       |
|https://mvnrepository.com/artifact/cn.hutool/hutool-all|hutool-all|常用的java工具包|
|https://mvnrepository.com/artifact/com.github.xiaoymin/knife4j-openapi3-jakarta-spring-boot-starter| knife4j-openapi3-jakarta-spring-boot-starter| 后端文档|

> 1.MyBatis-Plus代码生成器,免除了写controller层,service层,mapper层,entity层,xml文件的麻烦,下载jar包在cmd命令窗口执行 java -jar mybatis-plus-code-generator-3.5.2.x.jar

![ac4bd11373f08202a4c845555bfbfbedab641b06](https://t3.picb.cc/2024/08/15/ibAeVs.png)
![7468686c6f2633337a757079](https://t3.picb.cc/2024/08/15/ibAlG7.png)

> 2.简化实体类的书写,只需在实体类上添加@Data,@AllArgsConstructor,@NoArgsConstructor

> lombok还支持@Slf4j调试工具

### 开发热环境部署
#### 第一步:添加spring-boot-devtools的依赖及spring-boot-devtools的yml配置
##### 在pom.xml中添加
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```
##### 在application.yml中添加
```yml
spring:
  devtools:
    restart:
      #热部署生效
      enabled: true
      #设置重启目录
      additional-paths: src/main/java
      #设置静态文件目录
      exclude: static/**
```
#### 第二步:调 Settings 

![7468686c6f2633337a7570796](https://t3.picb.cc/2024/08/15/ibAup6.png)
![7468686c6f26339](https://t3.picb.cc/2024/08/15/ibACYT.png)

> 旧版本的idea第二个图的配置如下
> 按 Ctrl + Shift + Alt + / 快捷键调出Maintenance页面,单击Registry,勾选compiler.automake.allow.wen.app.running复选框

# 常用设计
### 对于前端与后端数据交互通用类的设计
#### 通用设计,这里默认使用了lombok

```java
package xxx.xxx.xxx.exception;
import lombok.Getter;

@Getter
public class AppException extends RuntimeException{
    private int code = 50000;
    private String msg = "服务器异常";
    public AppException(AppExceptionCodeMsg appExceptionCodeMsg){
        super();
        this.code = appExceptionCodeMsg.getCode();
        this.msg = appExceptionCodeMsg.getMsg();
    }
    public AppException(int code,String msg){
        super();
        this.code = code;
        this.msg = msg;
    }
}
```

```java
package xxx.xxx.xxx.exception;
import lombok.Getter;

//这个枚举类中定义的都是跟业务有关的异常
@Getter
public enum AppExceptionCodeMsg {
    OK(20000,"成功"),
    ERR_BAD_REQUEST(40000,"请求参数格式错误"),
    ERR_UNAUTHORIZED(40100,"登录失败, 用户名或密码错误"),
    ERR_UNAUTHORIZED_DISABLED(40101,"登录失败, 用户被禁用"),
    ERR_FORBIDDEN(40300,"无权限"),
    ERR_NOT_FOUND(40400,"数据不存在"),
    ERR_CONFLICT(40900,"数据冲突"),
    ERR_EXISTS(40901,"数据冲突, 已经存在"),
    ERR_IS_ASSOCIATED(40902,"数据冲突, 已经关联"),
    ERR_SAVE_FAILED(50010,"插入数据错误"),
    ERR_DELETE_FAILED(50100,"删除数据错误"),
    ERR_UPDATE_FAILED(50200,"修改数据错误"),
    ERR_FILE_UPLOAD(50300,"文件上传错误"),
    ERR_JWT_EXPIRED(60000,"JWT已过期"),
    ERR_JWT_SIGNATURE(60100,"验证签名失败"),
    ERR_JWT_MALFORMED(60200,"JWT格式错误"),
    ERR_JWT_LOGOUT(60300,"JWT已退出登录"),
    ERR_UNKNOWN(99999,"未知错误")
    ;
    private final int code ;
    private final String msg ;
    AppExceptionCodeMsg(int code, String msg){
        this.code = code;
        this.msg = msg;
    }
}
```

```java
package xxx.xxx.xxx.exception;
import xxx.xxx.xxx.response.JsonResult;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = {Exception.class})
    @ResponseBody
    public <T> JsonResult<T> exceptionHandler(Exception e){
        //这里先判断拦截到的Exception是不是我们自定义的异常类型
        if(e instanceof AppException){
            AppException appException = (AppException)e;
            return JsonResult.error(appException.getCode(),appException.getMsg());
        }
        //如果拦截的异常不是我们自定义的异常(例如：数据库主键冲突)
        return JsonResult.error(50000,"服务器端异常");
    }
}
```

```java
package xxx.xxx.xxx.response;
import xxx.xxx.xxx.exception.AppExceptionCodeMsg;
import lombok.Getter;

@Getter
public class JsonResult<T> {
    //服务端返回的错误码
    private int code = 20000;
    //服务端返回的错误信息
    private String msg = "success";
    //我们服务端返回的数据
    private final T data;
    private JsonResult(int code,String msg,T data){
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
    private JsonResult(int code,String msg){
        this.code = code;
        this.msg = msg;
        this.data = null;
    }
    public static JsonResult<Void> success(){
        return null;
    }
    public static <T> JsonResult<T> success(int code,String msg){
        return new JsonResult<T>(code, msg);
    }
    public static <T> JsonResult<T> success(T data){
        return new JsonResult<T>(20000, "success", data);
    }
    public static <T> JsonResult<T> success(String msg,T data){
        return new JsonResult<T>(20000,msg, data);
    }
    public static <T> JsonResult<T> error(AppExceptionCodeMsg appExceptionCodeMsg){
        return new JsonResult<T>(appExceptionCodeMsg.getCode(), appExceptionCodeMsg.getMsg(), null);
    }
    public static <T> JsonResult<T> error(int code,String msg){
        return new JsonResult<T>(code,msg, null);
    }
}
```
---
## Controller层通用接口,需要整合Mybatis-Plus与上面的"通用设计"
```java
package xxx.xxx.xxx.controller.common;

import xxx.xxx.xxx.response.JsonResult;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import org.springframework.web.bind.annotation.*;

import java.util.List;

//@RequestMapping("/api/v1")
//@CrossOrigin
//@RestController
public interface CommonController<T> {

        JsonResult<Void> add(T object);

        JsonResult<Void> removeOne(Long objectId);

        JsonResult<Void> remove(List<Long> objectIds);

        JsonResult<Void> edit(T object);

        JsonResult<T> queryOne(Long objectId);

        JsonResult<List<T>> listData();

        JsonResult<Page<T>> listPage(Long current, Long size);


//    @PostMapping("/object/add")
//    public JsonResult<Void> add(@RequestBody T object){
//        return null;
//    }
//
//    @DeleteMapping("/object/removeOne/{objectId}")
//    public JsonResult<Void> removeOne(@PathVariable(name = "objectId")  Long objectId){
//        return null;
//    }
//    @DeleteMapping("/object/remove")
//    public JsonResult<Void> remove(@RequestBody List<Long> objectIds){
//          return null;
//    }
//    @PutMapping("/object/edit")
//    public JsonResult<Void> edit(@RequestBody T object){
//          return null;
//    }
//    @GetMapping("/object/queryOne/{objectId}")
//    public JsonResult<T> queryOne( @PathVariable(name = "objectId")  Long objectId){
//          return null;
//    }
//    @GetMapping("/object")
//    public JsonResult<List<T>> listData(){
//          return null;
//    }
//    @GetMapping("/object/{current}/{size}")
//    public JsonResult<Page<T>> listPage(@PathVariable(name = "current") Long current, @PathVariable(name = "size") Long size){
//          return null;
//    }
}


```
---

## 代码测速工具类
```java
public class TestCodeRunningSpeedUtil {
    public static long startMilliSecond(){
        long current = System.currentTimeMillis();
        System.out.println("staTime: " + current + " ms");
        return current;
    }
    public static long endMilliSecond(){
        long current = System.currentTimeMillis();
        System.out.println("endTime: " + current + " ms");
        return current;
    }
    public static long startNanSecond(){
        long current = System.nanoTime();
        System.out.println("staTime: " + current + " ns");
        return current;
    }
    public static long endNanSecond(){
        long current = System.nanoTime();
        System.out.println("endTime: " + current + " ns");
        return current;
    }
}
```

```java
import cn.tedu.award.service.utils.TestCodeRunningSpeedUtil;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        Integer[] arr = {1,5,2,7,8,3,6,4};
        List<Integer> list = Arrays.asList(arr);
        System.out.println("source: " + list);

        long staTime = TestCodeRunningSpeedUtil.startMilliSecond();
        Collections.sort(list);
        long endTime= TestCodeRunningSpeedUtil.endMilliSecond();
        
        System.out.println("Code takes "+(endTime-staTime)+" ms");

        System.out.println( "result: " + list);
    }
}
```
---

