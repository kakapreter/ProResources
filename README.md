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
  mapper-locations: classpath:mappers/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# mybatis配置
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
#------------------------------------------------------------------------------------------------
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
#------------------------------------------------------------------------------------------------
#### Swagger2通用配置模板类------Swagger2是能够以REST风格方式在本地网页测试接口的工具
```java
package xxx.xxx.xxx.config;

import io.swagger.v3.oas.models.ExternalDocumentation;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {
    @Bean
    public OpenAPI springShopOpenAPI() {
        return new OpenAPI()
                .info(new Info().title("项目演示")
                        .description("项目演示API文档")
                        .version("v1.0.0")
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")))
                .externalDocs(new ExternalDocumentation()
                        .description("外部文档")
                        .url("https://springshop.wiki.github.org/docs"));
    }
}
```

#### aliyun的SpringBoot 3.0.2 Swagger2的jar包依赖
```xml
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-api</artifactId>
            <version>2.0.2</version>
        </dependency>
```

#### Swagger的yml配置
```yml
springdoc:
  swagger-ui:
    path: /swagger-ui.html
```

### Swagger默认的访问路径http://localhost:8080/swagger-ui/index.html#/
#------------------------------------------------------------------------------------------------
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
| github地址                                                   | artifactId(jar包) | jar包作用 |
| ----------------------------------------------------------- | ----------------- | --------- |
| https://github.com/fengwenyi/mybatis-plus-code-generator | mybatis-plus-code-generator            | 1         |

| maven地址                                                   | artifactId(jar包) | jar包作用 |
| ----------------------------------------------------------- | ----------------- | --------- |
| https://mvnrepository.com/artifact/org.projectlombok/lombok | lombok            | 2         |
| https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools    |spring-boot-devtools     |   web项目热部署       |
|https://mvnrepository.com/artifact/cn.hutool/hutool-all|hutool-all|常用的java工具包|


> 1.MyBatis-Plus代码生成器,免除了写controller层,service层,mapper层,entity层,xml文件的麻烦,下载jar包在cmd命令窗口执行 java -jar mybatis-plus-code-generator-3.5.2.x.jar

![](https://pic.imgdb.cn/item/651ff859c458853aefc7343a.png)

![](https://pic.imgdb.cn/item/64fed547661c6c8e547a64c4.png)

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

![](https://pic.imgdb.cn/item/65084f46204c2e34d3a9511b.png)

![](https://pic.imgdb.cn/item/651c5b8ac458853aef6cf31d.png)

> 旧版本的idea第二个图的配置如下
> 按 Ctrl + Shift + Alt + / 快捷键调出Maintenance页面,单击Registry,勾选compiler.automake.allow.wen.app.running复选框

# 常用设计
### 对于前端与后端数据交互通用类的设计
#### 通用R类,RCodeEnum类设计,这里默认使用了lombok
#### 加入依赖库
```xml
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-annotations</artifactId>
            <version>1.5.22</version>
        </dependency>
```
#### R类
```java
package xxx.xxx.xxx.entity.common;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import java.util.HashMap;
import java.util.Map;

//设置统一资源返回结果集
@Data
@ApiModel(value = "全局统一返回结果")
public class R {
    @ApiModelProperty(value = "返回码")
    private  Integer code;
    @ApiModelProperty(value = "返回消息")
    private  String message;
    @ApiModelProperty(value = "是否成功")
    private  Boolean success;
    @ApiModelProperty(value = "返回数据")
    private Map<String,Object> data = new HashMap<>();
    private R() {}
    //返回成功的结果集
    public static R success(){
        R r = new R();
        r.setSuccess(RCodeEnum.SUCCESS.isSuccess());
        r.setCode(RCodeEnum.SUCCESS.getCode());
        r.setMessage(RCodeEnum.SUCCESS.getMessage());
        return r;
    }

    //返回失败的结果集
    public static R error(){
        R r = new R();
        r.setSuccess(RCodeEnum.UNKNOWN_REASON.isSuccess());
        r.setCode(RCodeEnum.UNKNOWN_REASON.getCode());
        r.setMessage(RCodeEnum.UNKNOWN_REASON.getMessage());
        return r;
    }
    /**
     * @param rCodeEnum
     * @return
     */
    public static R setResult(RCodeEnum rCodeEnum){
        R r = new R();
        r.setSuccess(rCodeEnum.isSuccess());
        r.setCode(rCodeEnum.getCode());
        r.setMessage(rCodeEnum.getMessage());
        return r;
    }

    public R success(Boolean success){
        this.setSuccess(success);
        return this;
    }

    public R message(String message){
        this.setMessage(message);
        return this;
    }

    public R code(Integer code){
        this.setCode(code);
        return this;
    }

    public R data(String key, Object value) {
        this.data.put(key, value);
        return this;
    }

    public R data(Map<String,Object> map){
        this.setData(map);
        return this;
    }
    
}
      
```

#### RCodeEnum类
```java
package xxx.xxx.xxx.entity.common;
import lombok.Getter;

@Getter
public enum RCodeEnum {
    SUCCESS(true, 20000, "成功"),
    UNKNOWN_REASON(false, 20001, "未知错误"),
    BAD_SQL_GRAMMAR(false, 21001, "sql语法错误"),
    JSON_PARSE_ERROR(false, 21002, "json解析异常"),
    PARAM_ERROR(false, 21003, "参数不正确"),
    FILE_UPLOAD_ERROR(false, 21004, "文件上传错误"),
    EXCEL_DATA_IMPORT_ERROR(false, 21005, "Excel数据导入错误");

    private final boolean success;
    private final Integer code;
    private final String message;

    RCodeEnum(Boolean success, Integer code, String message) {
        this.success = success;
        this.code = code;
        this.message = message;
    }
}

```

#### 全局异常处理类
```java
package xxx.xxx.xxx.service.common;

import xxx.xxx.xxx.entity.common.R;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public R error(Exception e){
        e.printStackTrace();
        return R.error().message("执行了全局异常处理");
    }
}

```
#### 跨域配置类
```java
package xxx.xxx.xxx.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

/**
 * 这里是跨域的设置
 */

@Configuration
public class CorsConfig {

    // 当前跨域请求最大有效时长。这里默认1天
    private static final long MAX_AGE = 24 * 60 * 60;

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 1 设置访问源地址
        corsConfiguration.addAllowedHeader("*"); // 2 设置访问源请求头
        corsConfiguration.addAllowedMethod("*"); // 3 设置访问源请求方法
        corsConfiguration.setMaxAge(MAX_AGE);
        source.registerCorsConfiguration("/**", corsConfiguration); // 4 对接口配置跨域设置
        return new CorsFilter(source);
    }
}
```
