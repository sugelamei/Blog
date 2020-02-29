---
title: Spring Cloud-03 Ribbon实现客户端侧负载均衡
date: 2019-11-19 10:29:00 
tags: 
  - Spring Cloud
  - Ribbon
---



### 0.说在前面

```java
- JDK：Spring Cloud官方建议使用JDK 1.8;
- Spring Boot：笔者使用Spring Boot 2.1.10.RELEASE;
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；
```

​       经过前面的讲解，已经实现了微服务的注册与发现。启动各个微服务时，Eureka Client会把自己的网络信息注册到Eureka Server上。世界似乎更美好了一些。

​       然而，这样的架构依然有一些问题，比如负载均衡。一般来说，在生产环境中，各个微服务都会部署多个实例。那么服务消费者要如何将请求分摊到多个服务提供者实例上呢？

<!-- more -->

### 1.`Ribbon`简介

​        Ribbon是`Netflix`发布的负载均衡器，它有助于控制HTTP和TCP客户端的行为。为Ribbon配置服务提供者地址列表后，Ribbon就可基于某种负载均衡算法，自动地帮助服务消费者去请求。Ribbon默认为我们提供了很多的负载均衡算法，例如轮询、随机等。当然，我们也可为Ribbon实现自定义的负载均衡算法。

​         在Spring Cloud中，当Ribbon与Eureka配合使用时，Ribbon可自动从Eureka Server获取服务提供者地址列表，并基于负载均衡算法，请求其中一个服务提供者实例。如下图所示：

![](/image/SpringCloud/image-20191120210323456.png)

### 2.为服务者整合Ribbon

1.使用 `Spring Initializr`快速创建`microservice-consumer-movie-ribbon`，将`microservice-consumer-movie`项目中的内容复制过来，`pom.xml`如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.sugelamei</groupId>
    <artifactId>microservice-consumer-movie-ribbon</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-ribbon</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
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
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

由于前面已为电影微服务添加了`spring-cloud-starter-netflix-eureka-client`，该依赖已包含`spring-cloud-starter-netflix-ribbon`，也可以不引入此包。

2.为`RestTemplate`添加`@LoadBalanced`注解。

```java
@Configuration
public class ConsumerConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

只需添加注解`@LoadBalanced`注解，就可为`RestTemplate`整合Ribbon，使其具备负载均衡的能力。



3.对Controller代码进行修改

```java
@RestController
public class MovieController {
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id){
      return   restTemplate.getForObject("http://microservice-provider-user-ribbon/user/"+id,User.class);
    }

}
```

​    由代码可知，我们将请求的地址改成了http://microservice-provider-user/。microservice-provider-user是用户微服务的**虚拟主机名**（virtual hostname），当Ribbon和Eureka配合使用时，会自动将虚拟主机名映射成微服务的网络地址。

### 3.调整消费者

1.使用 `Spring Initializr`快速创建`microservice-provider-user-ribbon`，将`microservice-provider-user`项目中的内容复制过来，`pom.xml`如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.sugelamei</groupId>
    <artifactId>microservice-provider-user-ribbon</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-provider-user-ribbon</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
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
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

2.添加`application-8081.yml`

```yaml
#服务端口
server:
  port: 8081
spring:
  application:
    name: microservice-provider-user-ribbon
  datasource:
    username: root
    password: root
    url: jdbc:mysql://192.168.10.21:3306/microservice01?useUnicode=true;characterEncoding=utf8;allowMultiQueries=true;autoReconnect=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    platform: mysql
    schema: classpath:schema.sql
    data: classpath:data.sql
    initialization-mode: always

mybatis:
  mapper-locations: classpath*:mapper/*Mapper.xml

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

```

3.添加`application-8082.yml`

```yaml
#服务端口
server:
  port: 8080
spring:
  application:
    name: microservice-provider-user-ribbon
  datasource:
    username: root
    password: root
    url: jdbc:mysql://192.168.10.21:3306/microservice02?useUnicode=true;characterEncoding=utf8;allowMultiQueries=true;autoReconnect=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    platform: mysql
    schema: classpath:schema.sql
    data: classpath:data.sql
    initialization-mode: always

mybatis:
  mapper-locations: classpath*:mapper/*Mapper.xml

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

```

​       对比2个配置文件可知，2个配置文件的不同之处在于8081端口对应数据库`microservice01`，8082端口对应数据库`microservice02`，这个就能通过不同数据库的不同数据来区分不同的微服务。

**测试**

1.启动`microservice-discovery-eureka`。

2.基于2个不同的配置文件启动2个`microservice-provider-user-ribbon`实例。

3.启动`microservice-consumer-movie-ribbon`。

4.访问*http://localhost:8761*；

![](/image/SpringCloud/image-20191125231049426.png)

5.访问http://localhost:9081/user/1

![](/image/SpringCloud/image-20191125230855707.png)

![](/image/SpringCloud/image-20191125230930137.png)

刷新以后将数据库切换，说明默认的负载均衡方式为轮询。



-  虚拟主机名与虚拟`IP`非常类似，如果大家接触过`HAProxy`或Heartbeat，理解虚拟主机名就非常容易了。如果无法理解虚拟主机名，可将其简单理解成为提供者的服务名称，因为在默认情况下，虚拟主机名和服务名称是一致的。当然，也可使用配置属性`eureka.instance.virtual-host-name`或者`eureka.instance.secure-virtual-host-name`指定虚拟主机名。

- 不能将`restTemplate.getForObject(...)与loadBalancerClient.choose(...)`写在同一个方法中，两者之间会有冲突——因为此时代码中的`restTemplate`实际上是一个Ribbon客户端，本身已经包含了“choose”的行为。
-  虚拟主机名不能包含“_”之类的字符，否则Ribbon在调用时会报异常。 

### 4.`Ribbon`配置自定义

​      很多场景下，可能根据需要自定义Ribbon的配置，例如修改Ribbon的负载均衡规则等。`Spring Cloud Edgware`允许使用Java代码或属性自定义Ribbon的配置，两种方式是等价的。 

#### 4.1   使用Java代码自定义Ribbon配置

本小节来讨论如何使用代码自定义Ribbon配置。

#####      4.1.1  配置指定名称的Ribbon Client 

在Spring Cloud中，Ribbon默认的配置类是`RibbonCl ientConfiguration`。也可使用一个`POJO`自定义Ribbon的配置（自定义配置会覆盖默认配置）。这种配置是细粒度的，不同名称的Ribbon客户端可使用不同配置。 

 下面来为名为`microservice-provider-user`的Ribbon客户端自定义配置。 

1.新建 `microservice-consumer-movie-ribbon-customizing`项目，复制 `microservice-consumer-movie-ribbon` 中相关代码；

2.创建Ribbon的配置类。 

```java
@Configuration
public class RibbonConfig {

    @Bean
    public IRule iRule() {
        //修改负载均衡算法为随机算法
        return new RandomRule();
    }
}

```

注意：该类比应该在主应用程序上下问的`@ComponentScan`所扫描的包中。

3. 创建一个空类，并在其上添加@Configuration注解和`@RibbonClient`注解。 

```java
/*
 使用RibbonClient为特定name的RibbonClient自定义配置。
 使用@RibbonClient的configuration属性，指定Ribbon的配置类。
*/
@Configuration
@RibbonClient(name = "microservice-provider-user-ribbon",configuration = RibbonConfig.class)
public class RibbonClientConfig {

}

```

 由代码可知，使用`@RibbonClient`注解的configuration属性，即可自定义指定名称Ribbon客户端的配置。 



**测试**

1.启动`microservice-discovery-eureka`。

2.基于2个不同的配置文件启动2个`microservice-provider-user-ribbon`实例。

3.启动`microservice-consumer-movie-ribbon-customizing`。

4.访问*http://localhost:8761*；

![](/image/SpringCloud/image-20191201225224049.png)

5.访问http://localhost:9081/user/1

![](/image/SpringCloud/image-20191201225446934.png)

每次刷新后会出现不同的`dataBase`，出现任意一个`dataBase`的概率是随机的。也就是在使用的负载均衡算法为随机算法，会随机访问有个服务获取数据。 此时请求会随机分布到两个用户微服务节点上，说明已经实现了Ribbon的自定义配置。 

**注意**：

​    必须注意的是，本例中的`RibbonConfiguration`类不能存放在主应用程序上下文的`@ComponentScan`所扫描的包中，否则该类中的配置信息将被所有的`@RibbonClient`共享。

​       因此，如果只想自定义某一个Ribbon客户端的配置，必须防止@Configuration注解的类所在的包与`@ComponentScan`扫描的包重叠，或应显式指定`@ComponentScan`不扫描@Configuration类所在的包。



##### 4.1.2 全局配置 

可使用`@RibbonClients`注解为所有`RibbonClient`提供默认配置

```java
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public  class DefaultRibbonClientConfig {
    public static class DefaultServerList extends ConfigurationBasedServerList{

        public  DefaultServerList(IClientConfig clientConfig) {
            super.initWithNiwsConfig(clientConfig);
        }
    }
}
```



```java
@Configuration
public class DefaultRibbonConfig {

    @Bean
    public IRule rule(){
        return  new BestAvailableRule();
    }
    @Bean
    public IPing ping(){
        return new PingUrl();
    }

    @Bean
    public ServerList<Server> serverServerList(IClientConfig config){
        return new DefaultRibbonClientConfig.DefaultServerList(config);
    }

    @Bean
    public ServerListSubsetFilter serverListSubsetFilter(){
        return new ServerListSubsetFilter();
    }
}
```

#### 4.2 使用属性自定义Ribbon配置

​     从`Spring Cloud Netflix 1.2.0`开始（即从Spring Cloud Camden RELEASE开始，Ribbon支持使用属性自定义。这种方式比使用Java代码配置的方式更加方便。

​    支持的属性如下，配置的前缀是<clientName>.ribbon.。其中，<clientName>是Ribbon Client的名称，**如果省略，则表示全局配置**。

- `NFLoadBalancerClassName`：配置`ILoadBalancer`的实现类。

- `NFLoadBalancerRuleClassName`：配置`IRule`的实现类。

- `NFLoadBalancerPingClassName`：配置`IPing`的实现类。

- `NIWSServer ListClassName`：配置Server List的实现类。

- `NIWSServer ListFilterClassName`：配置`Server ListFilter`的实现类。

下面，我们来用属性来修改名为`microservice-provider-user`的Ribbon Client的负载均衡规则。

​    新建`microservice-consumer-movie-ribbon-customizing-properties`项目，复制项目`microservice-consumer-movie-ribbon`中的内容；在项目的`application.yml`中内容即可：

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-ribbon-customizing-properties
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

microservice-provider-user-ribbon:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

这样，就可将名为`microservice-provider-user`的Ribbon Client的负载均衡规则设为随机。

若配置成如下形式，则表示对所有`RibbonClient`都使用`RandomRule`：

```yaml
ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

由代码不难看出，使用属性自定义的方式比用Java代码配置方便很多。

### 5.脱离Eureka使用Ribbon

​         在前面的描述中，是将Ribbon与Eureka配合使用的。但现实中可能不具备这样的条件，例如一些遗留的微服务，它们可能并没有注册到Eureka Server上，甚至根本不是使用Spring Cloud开发的，此时要想使用Ribbon实现负载均衡，要怎么办呢？

Ribbon支持脱离Eureka使用，如图所示：

![](/image/SpringCloud/image-20191204213656231.png)

​      下面将通过一个简单的示例为大家讲解如何脱离Eureka使用Ribbon。

1.新建`microservice-consumer-movie-without-eureka`项目，复制项目`microservice-consumer-movie-ribbon`中的内容

2.为了让测试更具说服力，干脆为项目去掉Eureka的依赖`spring-cloud-starter-netflix-eureka-client`，只使用Ribbon的依赖`spring-cloud-starter-netflix-ribbon`。在项目的`pom.xml`中，找到：

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

将此依赖删除。

3.去掉启动类上的`@EnableDiscoveryClient`注解（如果有的话）。

4.将`application.yml`改成如下：

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-without-eureka
#Ribbon客户端设置请求的地址列表
microservice-provider-user-ribbon:
  ribbon:
    listOfServers: localhost:8000,localhost:8001
```

5.在`microservice-provider-user-ribbon`中新增`application-8000.yml`和`application-8001.yml`配置文件，具体内容如下：

```yaml
#服务端口  8000对应application-8000.yml 8001对应application-8001.yml 
server:
  port: 8000
spring:
  application:
    name: microservice-provider-user-ribbon
  datasource:
    username: root
    password: root
    url: jdbc:mysql://192.168.10.21:3306/microservice01?useUnicode=true;characterEncoding=utf8;allowMultiQueries=true;autoReconnect=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    platform: mysql
    schema: classpath:schema.sql
    data: classpath:data.sql
    initialization-mode: always

mybatis:
  mapper-locations: classpath*:mapper/*Mapper.xml
  eureka:
  client:
    enabled: false
```

6.修改`MovieController`

```java
@RestController
public class MovieController {
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id) {
        return restTemplate.getForObject("http://microservice-provider-user-ribbon/user/" + id, User.class);
    }

}
```



**测试**

1.根据2个不同的配置文件启动2个端口分别为8000和8001的`microservice-provider-user-ribbon`服务；

2.启动`microservice-consumer-movie-without-eureka`

3.访问

```
http://localhost:9081/user/1
```

![](/image/SpringCloud/image-20191201225446934.png)



由结果可知，尽管电影微服务和用户微服务此时并没有注册到Eureka上，Ribbon仍可正常工作，请求依旧会分摊到两个用户微服务节点上。

**注意**

当`EurekaClient`依赖在项目的`classpath`下时，如果想单独使用Ribbon，而不使用`Eurkea`的服务发现功能，需添加配置`ribbon.eureka.enabled=false`。

某些场景下，我们可能只想让指定名称的Ribbon Client去使用指定的URL请求，其他Ribbon Client依旧与Eureka配合使用，那么可配置如下：

```yaml

<clientName>:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    listOfServers: localhost:8000,localhost:8001
```

这样，对于名为<clientName>的Ribbon Client，即可从地址列表localhost:8000,localhost:8001中选择地址去请求，而其他名称的Ribbon Client依旧可与Eureka配合使用——自动从Eureka Server获得目标服务的地址，并选择一个去请求。



### 6.饥饿加载

Spring Cloud会为每个名称的Ribbon Client维护一个子应用程序上下文（还记得Spring Framework中的父子上下文吗？），这个上下文默认是懒加载的。指定名称的Ribbon Client第一次请求时，对应的上下文才会被加载，因此，首次请求往往会比较慢。从Spring Cloud Dalston开始，我们可配置饥饿加载。例如：

```yaml
ribbon:
  eager-load:
    enabled: true
    clients: client1,client2
```

这样，对于名为`client1、client2`的Ribbon Client，将在启动时就加载对应的子应用程序上下文，从而提高首次请求的访问速度。

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```

源码地址：

```
https://github.com/superdevops-cn/spring-cloud-microservice
```

### 888.参考资料

[《Spring Cloud 与Docker 微服务架构实战》](https://book.douban.com/subject/30278673/)