# [SpringBoot之读取配置文件](https://segmentfault.com/a/1190000017578623)

 

引入依赖

```xml
<!-- Spring Boot 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

application.properties `默认配置文件`

```properties
local.ip=192.168.1.1
local.port=8080
local.test=this is test multi file
```

- 第一种方式 注入Environment，同时也可以指定默认值

```java
package vip.fkandy.demo01;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;
import lombok.extern.slf4j.Slf4j;

@Component
@Slf4j
public class UserConfig {
    @Autowired
    private Environment env; //不需要getter setter方法
    public void show() {
        log.info("local.ip: " + env.getProperty("local.ip"));
    }
}
```

- 第二种方式 使用@Value("${local.ip}")方式

```java
package vip.fkandy.demo01;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class JdbcConfig {
    @Value("${local.ip}")
    private String localIp;  //不需要getter setter方法
    public void show() {
        log.info("Jdbc ip: " + localIp);
    }
}
```

- 使用@Value("${local.ip}")方式,并指定默认值

```java
package vip.fkandy.demo01;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import lombok.extern.slf4j.Slf4j;

@Component
@Slf4j
public class DogConfig {
    @Value("${local.ip:8080}")
    private String localPort;//不需要getter setter方法
    public void show() {
        log.info("dog ip :" + localPort);
    }
}
```

配置文价默认名字为application.properties,默认位置在classpath根目录，或者classpath:/config,file:/,file:config目录默认配置文件名字可以使用--spring.config.name来指定，只需要指定文件名字，文件扩展名可以省略，默认的配置文件路径可以使用--spring.config.location=classpath:conf/app.properties指定具体文件位置，配置文件名需要指定全路径，包括目录和文件名字，还可以指定多个，用逗号隔开	

- 【自定义配置文件】定义一个配置类@Configuration和@PropertySource(value = {classpath:application.properties" })，之后再其他容器bean中使用@Value取值

```java
@Configuration
@PropertySource(value = {"classpath:a.properties" })    //可以引用自定义名字的配置文件
public class CatConfig {

}
```

也可以指定多个配置文件

```java
@PropertySources({@PropertySource("classpath:a.properties" ),@PropertySource("classpath:b.properties" )})
```

- 第三种方式 根据前缀指定

```java
package vip.fkandy.demo01;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;

@Component //需要加入到Spring容器
@ConfigurationProperties(prefix = "local")
@Slf4j
@Data
public class LastConfig {
    private String ip;
    private String port;
    public void show() {
        log.info("ip:{},port:{}",ip,port);
    }
}
```

测试类

```java
package vip.fkandy.demo01;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import lombok.extern.slf4j.Slf4j;

@SpringBootApplication
@Slf4j
public class Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);
        log.info(context.getEnvironment().getProperty("local.ip"));
        log.info("-------测试 @PropertySource 指定配置文件---------------------------");
        log.info(context.getEnvironment().getProperty("local.address"));
        log.info("-------测试默认配置文件和@PropertySource 指定配置文件同时能生效-----");
        log.info(context.getEnvironment().getProperty("local.test"));
        log.info("-------@Autowired Environment method---------------------------");
        context.getBean(UserConfig.class).show();
        log.info("-------@Value(\"${local.ip}\") method --------------------");
        context.getBean(DogConfig.class).show();
        log.info("-------@Value(\"${local.ip:8080}\") default value method ------");
        context.getBean(JdbcConfig.class).show();
        log.info("-------@ConfigurationProperties method ------------------------");
        context.getBean(LastConfig.class).show();
    }
}
```

支持在配置中使用${}引用之前写好的配置

- 注入集合

```java
ds.hosts[0]=192.168.0.1
ds.hosts[1]=192.168.0.2
@Component
@ConfigurationProperties("ds")
public class TomcatProperties{
    private List<String> hosts = new ArrayList<String>();
    public List<String> getHosts(){
        return hosts;
    }
    public void setHosts(List<String> hosts){
        this.hosts = hosts;
    }
}
```

- 动态引用配置文件-配置文件集中化管理-可以通过HttpClient读取集中化管理的配置文件

```java
package com.example.demo;

import java.io.FileInputStream;
import java.io.InputStream;
import java.util.Properties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.env.EnvironmentPostProcessor;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.PropertiesPropertySource;
/**
 * 动态读取d:/tmp/springboot.properties
 */
public class MyEnvironmentPostProcessor implements EnvironmentPostProcessor {
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        try {
            InputStream input = new FileInputStream("d:/tmp/springboot.properties");
            Properties source = new Properties();
            source.load(input);
            PropertiesPropertySource propertySource = new PropertiesPropertySource("my", source);
            environment.getPropertySources().addLast(propertySource);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

srcmainresourcesMETA-INFspring.factories

```java
org.springframework.boot.env.EnvironmentPostProcessor=com.example.demo.MyEnvironmentPostProcessor
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class Springboot15DynamicConfigApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Springboot15DynamicConfigApplication.class, args);
        System.out.println(context.getEnvironment().getProperty("local.ip"));
    }
}
```

通过运行环境动态注入Bean--->基于代码层面，也可以直接动态指定peroperties文件

```java
public class MyConfig{
    
    @Bean
    @Profile("test")//这个注解也可以放在类上
    public Runnable createRunnable(){
        return ()->{};
    }
    
    @Bean
    @Profile("dev")
    public Runnable createRunnable(){
        return ()->{};
    }
}
```

--spring.profiles.active=dev 表示激活dev配置