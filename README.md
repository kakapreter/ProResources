
# 数据库

|                          maven地址                           | artifactId(jar包)         | jar包作用         |
| :----------------------------------------------------------: | ------------------------- | ----------------- |
| https://mvnrepository.com/artifact/mysql/mysql-connector-java | mysql-connector-java      | java数据库驱动    |
|     https://mvnrepository.com/artifact/com.alibaba/druid     | druid                     | druid数据库链接池 |
| https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter | mybatis-plus-boot-starter | mybatis-plus      |
|    https://mvnrepository.com/artifact/org.mybatis/mybatis    | mybatis                   | mybatis           |



#### application.properties文件

```properties
#数据库连接配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/your_database?useSSL=false&serverTimeZone=Asia/Shanghai
spring.datasource.username=your_username
spring.datasource.password=your_password
#配置Druid连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

#Mybatis日志xx.xx.mapper.*为映射的接口
logging.level.xx.xx.mapper.*=debug

#配置xml的路径---resources目录下的mappers文件夹
mybatis.mapper-locations=classpath:mappers/*.xml
#开启驼峰命名支持
mybatis.configuration.map-underscore-to-camel-case=true
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
> ​	|-spring-boot-starter,spring-boot-starter 
>
> ​		|-spring-boot-starter-logging
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

#### Cloud Native App Initializer https://start.spring.io , https://start.aliyun.com , aliyun基本pom.xml配置
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

