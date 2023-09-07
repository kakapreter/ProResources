# 数据库jar包

|                          maven地址                           | artifactId(jar包)         | jar包作用         |
| :----------------------------------------------------------: | ------------------------- | ----------------- |
| https://mvnrepository.com/artifact/com.mysql/mysql-connector-j | mysql-connector-j      | java数据库驱动    |
| https://mvnrepository.com/artifact/mysql/mysql-connector-java | mysql-connector-java      | java数据库驱动(官方)    |
|     https://mvnrepository.com/artifact/com.alibaba/druid     | druid                     | druid数据库连接池 |
| https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter | mybatis-plus-boot-starter | mybatis-plus      |
|    https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter    | mybatis                   | mybatis           |

### properties与yml配置文件任选其一

#### application.properties文件--mybatis配置及druid连接池配置

```properties
# 数据库连接配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/your_database?useSSL=false&serverTimeZone=Asia/Shanghai
spring.datasource.username=your_username
spring.datasource.password=your_password
# 配置Druid连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

# 配置xml的路径---resources目录下的mappers文件夹
mybatis.mapper-locations=classpath:mappers/*.xml
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
# yml结构 相当于properties文件中的server.port=8080
server:
  port: 8080

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

#指定mybatis-plus的日志输出格式
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# 加载MyBatis配置文件
mybatis:
  mapper-locations: classpath:mappers/*.xml
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
#------------------------------------------------------------------------------------------------
#### Swagger2通用配置模板类------Swagger2是能够以REST风格方式在本地网页测试接口的工具
```java
package xxx.xxx.xxx.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn")) //写项目的根包如cn.xxx.xxx.xxx的根包为cn
                .paths(PathSelectors.any()).build();
    }
    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("演示项目API")
                .description("演示项目")
                .version("1.0")
                .build();
    }
}

```
#### aliyun的SpringBoot 3.0.2 Swagger2的jar包依赖
```xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
```
#### Swagger的yml配置
```yml
spring:
  # swagger-ui custom path
  mvc:
    path match:
      matching-strategy: ant_path_matcher
```

### Swagger默认的访问路径http://localhost:8080/swagger-ui/index.html#/
#------------------------------------------------------------------------------------------------
#### 测试Druid连接池配置
```java
package xxx.xxx.xxx;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
@SpringBootTest
public class DruidTest {
    @Autowired
    DataSource dataSource;
    @Test
    public void druidTest() throws SQLException {
        //查看默认数据源
        System.out.println(dataSource.getClass());
        //获得数据库连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        //close
        connection.close();
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
@MapperScan("xxx.xxx.xxx.Mapper")
public class XxxApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(XxxApplication.class, args);
        Environment environment = context.getBean(Environment.class);
        System.out.println("当前测试环境为: " + environment.getProperty( "spring.profiles.active") + " 占用端口为: "+environment.getProperty("local.server.port"));
        System.out.println("Web首页访问链接为：http://localhost:" + environment.getProperty("local.server.port")+"/login.html");
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
>
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
| maven地址                                                   | artifactId(jar包) | jar包作用 |
| ----------------------------------------------------------- | ----------------- | --------- |
| https://mvnrepository.com/artifact/org.projectlombok/lombok | lombok            | 1         |
| https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools    |spring-boot-devtools     |   web项目热部署       |

> 1.简化实体类的书写,只需在实体类上添加@Data,就省略了setter,getter,toString方法,有参和无参构造方法还是需要写的
>
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

![](https://pic.imgdb.cn/item/64f91121661c6c8e54282d08.png)
![](https://pic.imgdb.cn/item/64f91519661c6c8e542a0ca9.png)
> 旧版本的idea第二个图的配置如下
> 按 Ctrl + Shift + Alt + / 快捷键调出Maintenance页面,单击Registry,勾选compiler.automake.allow.wen.app.running复选框
