---
title: Spring Cloud-06 使用Zuul构建微服务网关
date: 2019-11-27 11:24:13 
tags: 
  - Spring Cloud
  - Zuul
---



### 0.说在前面

```yaml
- JDK：Spring Cloud官方建议使用JDK 1.8;
- Spring Boot：笔者使用Spring Boot 2.1.10.RELEASE;
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；
```

### 1. 为什么要使用微服务网关 

​       经过前文的讲解，微服务架构已经初具雏形，但还有一些问题——不同的微服务一般会有不同的网络地址，而外部客户端（例如手机`APP`）可能需要调用多个服务的接口才能完成一个业务需求。例如一个电影购票的手机`APP`，可能调用多个微服务的接口才能完成一次购票的业务流程，

如下图所示

<!--more-->

![](/image/SpringCloud/image-20200102213856456.png)

如果让客户端直接与各个微服务通信，会有以下的问题：

- 客户端会多次请求不同的微服务，增加了客户端的复杂性。

- 存在跨域请求，在一定场景下处理相对复杂。

- 认证复杂，每个服务都需要独立认证。

- 难以重构，随着项目的迭代，可能需要重新划分微服务。例如，可能将多个服务合并成一个或者将一个服务拆分成多个。如果客户端直接与微服务通信，那么重构将很难实施。

- 某些微服务可能使用了对防火墙/浏览器不友好的协议，直接访问时会有一定的困难。

以上问题可借助微服务网关解决。微服务网关是介于客户端和服务器端之间的中间层，所有的外部请求都会先经过微服务网关。使用微服务网关后，架构可演变成下图所示：

![](/image/SpringCloud/image-20200102214045789.png)

​          这样，微服务网关封装了应用程序的内部结构，客户端只用跟网关交互，而无须直接调用特定微服务的接口。这样，开发就可以得到简化。不仅如此，使用微服务网关还有以下优点：

- 易于监控。可在微服务网关收集监控数据并将其推送到外部系统进行分析。

- 易于认证。可在微服务网关上进行认证，然后再将请求转发到后端的微服务，而无须在每个微服务中进行认证。

- 减少了客户端与各个微服务之间的交互次数。





### 2. `Zuul`简介

`Zuul`是`Netflix`开源的微服务网关，它可以和Eureka、Ribbon、`Hystrix`等组件配合使用。

`Zuul`的核心是一系列的过滤器，这些过滤器可以完成以下功能。

- 身份认证与安全：识别每个资源的验证要求，并拒绝那些与要求不符的请求。

- 审查与监控：在边缘位置追踪有意义的数据和统计结果，从而带来精确的生产视图。

- 动态路由：动态地将请求路由到不同的后端集群。

- 压力测试：逐渐增加指向集群的流量，以了解性能。

- 负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求。

- 静态响应处理：在边缘位置直接建立部分响应，从而避免其转发到内部集群。

- 多区域弹性：跨越`AWSRegion`进行请求路由，旨在实现`ELB（Elastic Load Balancing）`使用的多样化，以及让系统的边缘更贴近系统的使用者。

`SpringCloud`对`Zuul`进行了整合与增强。目前，`Zuul`使用的默认HTTP客户端是`Apache HTTPClient`，也可以使用`RestClien`t或者`okhttp3.OkHttpClient`。

如果想要使用`RestClient`，可以设置`ribbon.restclient.enabled=true`；

想要使用`okhttp3.OkHttpClient`，可以设置`ribbon.okht tp.enabled=true`。



•`Zuul`的GitHub：*https://github.com/Netflix/zuul*.

•`Netflix`如何使用Zuul：*https://github.com/Netflix/zuul/wiki/How-We-Use-Zuul-At-Netflix*.

### 3. 编写`Zuul`微服务网关

​    下面将编写一个简单的微服务网关。在本例中会将`Zuul`注册到Eureka Server上。

1.新建`microservice-gateway-zuul`，并为项目添加以下完整依赖。

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
    <artifactId>microservice-gateway-zuul</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-gateway-zuul</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
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

2.在启动类上添加注解`@EnableZuulProxy`，声明一个`Zuul`代理。该代理使用Ribbon来定位注册在Eureka Server中的微服务；同时，该代理还整合了`Hystrix`，从而实现了容错，所有经过`Zuul`的请求都会在`Hystrix`命令中执行。

3.编写配置文件`application.yml`。

  ```yaml
server:
  port: 8888
spring:
  application:
    name: microservice-gateway-zuul
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
  ```



​       这样，一个简单的微服务网关就编写完成了。从配置可知，此时仅添加了`Zuul`的依赖，并将`Zuul`注册到Eureka Server上。

4.在项目`microservice-provider-user`中添加配置文件`application-8081.yml`和`application-8082.yml`，两者的区别仅仅是端口和数据库的区别：

8081端口对应`microservice01`数据库；

8082端口对应`microservice02`数据库；

这样这是为了方便测试.

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



#### 3.1  测试路由规则

1.启动项目`microservice-discovery-eureka`。

2.使用`application-8081.yml`启动项目`microservice-provider-user`。

3.启动项目`microservice-consumer-movie-ribbon`。

4.启动项目`microservice-gateway-zuul`。

5.访问*http://localhost:8888/microservice-consumer-movie-ribbon/user/1*，请求会被转发到*http://localhost:9081/user/1*（电影微服务）。

6.访问*http://localhost:8888/microservice-provider-user/user/1*，请求会被转发到*http://localhost:8081/user/1*（用户微服务）。

说明在默认情况下`，Zuul`会代理所有注册到Eureka Server的微服务，并且`Zuul`的路由规则如下：

http://ZUUL_HOST:ZUUL_PORT/微服务在Eureka上的`serviceId/**`会被转发到`serviceId`

对应的微服务。

#### 3.2  测试负载均衡

1.启动项目`microservice-discovery-eureka`。

2.分别使用`application-8081.yml`和`application-8082.yml`启动项目`microservice-provider-user`。

3.启动项目`microservice-gateway-zuul`。此时，Eureka Server首页如图下图所示。

![image-20200105191531194](/image/SpringCloud/image-20200105191531194.png)

4.多次访问*http://localhost:8888/microservice-provider-user/user/1*，会发现两个用户微服务节点都返回如下结果：

![image-20200105191639341](/image/SpringCloud/image-20200105191639341.png)

![image-20200105191703537](/image/SpringCloud/image-20200105191703537.png)

说明`Zuul`可以使用Ribbon达到负载均衡的效果。

#### 3.3  测试`Hystrix`容错与监控

1.启动项目`microservice-discovery-eureka`。

2.使用`application-8081.yml`启动项目`microservice-provider-user`。

3.启动项目`microservice-consumer-movie-ribbon`。

4.启动项目`microservice-gateway-zuul`。

5.启动项目`microservice-consumer-movie-feign-hystrix-dashboard`。

6.访问*http://localhost:8888/microservice-consumer-movie-ribbon/user/1*，可获得预期结果。

7.访问http://localhost:9083/hystrix

8.在`Hystrix Dashboard`中输入http://localhost:8888/actuator/hystrix.stream，随意指定某Title并单击Monitor Stream按钮后会显示相应的监控界面。

![image-20200108215320537](/image/SpringCloud/image-20200105215320537.png)

说明`Zuul`已经整合了`Hystrix`。

### 4. 管理端点

​       当`@EnableZuulProxy`与Spring Boot  Actuator配合使用时，`Zuul`会暴露两个端点：`/routes`和`filters`。借助这些端点，可方便、直观地查看以及管理`Zuul`。

#### 4.1 routes端点



`/routes`端点的使用非常简单，具体如下：

1.使用GET方法访问该端点，即可返回`Zuul`当前映射的路由列表。

2.使用POST方法访问该端点就会强制刷新`Zuul`当前映射的路由列表。尽管路由会自动刷新，Spring Cloud依然提供了强制立即刷新的方式。使用`endpoints.routes.enabled` = `false`来禁止此端点。

3.`SpringCloud Edgware`对/routes端点进行了进一步的增强，我们可使用/routes?format=details查看更多与路由相关的详情设置。

由于`spring-cloud-starter-netflix-zuul`已经包含了spring-boot-starter-actuator，因此之前编写的`microservice-gateway-zuul`已具备路由管理的能力。

4.修改`microservice-gateway-zuul`的配置，设置如下配置：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: routes
```

**测试**

1.启动项目`microservice-discovery-eureka`。

2.使用`application-8081.yml`启动项目`microservice-provider-user`。

3.启动项目`microservice-consumer-movie-ribbon`。

4.启动项目`microservice-gateway-zuul`。

5.启动项目`microservice-consumer-movie-feign-hystrix-dashboard`。

6.使用浏览器访问http://localhost:8888/actuator/routes，可获得如下的结果：

![image-20200105223319853](/image/SpringCloud/image-20200105223319853.png)

从中可以直观地看出从路径到微服务的映射。

这段`JSON`表示：访问`Zuul`的`/microservice-consumer-movie-ribbon/**`路径，会转发到microservice-consumer-movie-ribbon微服务的/**。

7.访问*http://localhost:8888/actuator/routes/details*，可获得类似如下的结果：

![image-20200105223910488](/image/SpringCloud/image-20200105223910488.png)

由结果可知，此时可看到更多与路由相关的配置，例如路由id、转发路径、是否重试等，这些对于我们调测应用很有用。

也可使用类似方式测试`Zuul`路由的自动刷新与强制刷新。

> `Zuul`路由相关代码：`org.springframework.cloud.netflix.zuul.RoutesEndpoint`.





#### 4.2 filters端点

​       从`SpringCloud Edgware`版本开始，`Zuul`提供了/filters端点。访问该端点即可返回`Zuul`中当前所有过滤器的详情，并按照类型分类，后面将会详细讲解`Zuul`的过滤器.

​        如下是/filters端点的展示结果，从中，我们可以了解当前`Zuul`中`，error、post、pre、route`四种类型的过滤器分别有哪些，每个过滤器的order（执行顺序）是多少，以及是否启用等信息。这对`Zuul`问题的定位很有用。

![image-20200105224530825](/image/SpringCloud/image-20200105224530825.png)

### 5. 路由配置详解

​      前面已经编写了一个简单的`Zuul`网关，并让该网关代理了所有注册到Eureka Server的微服务。但在现实中可能只想让`Zuul`代理部分微服务，又或者需要对URL进行更加精确的控制。

`Zuul`的路由配置非常灵活、简单，本节通过几个示例，详细讲解`Zuul`的路由配置。

#### 5.1    自定义指定微服务的访问路径

   配置`zuul.routes.`指定微服务的`serviceId=`指定路径即可

```yaml
zuul:
  routes:
    microservice-provider-user: /user/**
```



  完成设置后，`microservice-provider-user`微服务就会被映射到/user/**路径。

#### 5.2 忽略指定微服务

忽略服务非常简单，可以使用`zuul.ignored-services`配置需要忽略的服务，多个服务间用逗号分隔.

```yaml
zuul:
  ignored-services:  microservice-provider-user,microservice-consumer-movie-ribbon 
```

这样就可让`Zuul`忽略`microservice-provider-user`和`microservice-consumer-movie`微服务，只代理其他微服务。

#### 5.3 代理指定的微服务

​       忽略所有微服务，只路由指定微服务很多场景下，可能只想要让`Zuul`代理指定的微服务，此时可以将`zuul.ignored-services`设为'*'。

```yaml
zuul:
  ignored-services: '*'
  routes: microservice-provider-user: /user/**
```

这样就可以让`Zuul`只路由`microservice-provider-user`微服务。

#### 5.4 同时指定微服务的`serviceId`和对应路径

```yaml
 zuul:
  routes:
    user-route:  
      path: /user/**
      serviceId: microservice-provider-user
```

#### 5.5 同时指定path和URL

```yaml
 zuul:
  routes:
     user-route:
      path: /user/**
      url: http://localhost:8081/
```

这样就可以将`/user/**`映射到`http://localhost:8000/**`

> 需要注意的是，使用这种方式配置的路由不会作为`HystrixCommand`执行，同时也不能使用Ribbon来负载均衡多个URL

#### 5.6 同时指定path和URL，并且不破坏`Zuul`的`Hystrix、Ribbon`特性

```yaml
zuul:
  routes:
    echo:
      path: /user/**
      serviceId: microservice-provider-user
      strip-prefix: true

hystrix:
  command:
    microservice-provider-user:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: ...

microservice-provider-user:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    listOfServers:  http://localhost:8081/, http://localhost:8082/
    ConnectTimeout: 1000
    ReadTimeout: 3000
    MaxTotalHttpConnections: 500
    MaxConnectionsPerHost: 100
```

或者

```yaml
uul:
  routes:
    user-route:
      path: /user/**
      serviceId: microservice-provider-user

ribbon:
  eureka:
    enabled: false

microservice-provider-user:
  ribbon:
    listOfServers: localhost:8081,localhost:8082
```

这样就可以既指定path与URL，又不破坏`Zuul的Hystrix`与Ribbon特性了。

#### 5.7 使用正则表达式指定`Zuul`的路由匹配规则

```java
@Bean
public PatternServiceRouteMapper serviceRouteMapper() {
    //构造函数PatternServiceRouteMapper(String servicePattern, String routePattern)
    //servicePattern 指定微服务的正则
    //routePattern 指定路由正则
    return new PatternServiceRouteMapper(
        "(?<name>^.+)-(?<version>v.+$)",
        "${version}/${name}");
}
```

通过这段代码即可实现将诸如`microservice-provider-user-v1`这个微服务，映射到`/v1/microservice-provider-user/**`这个路径。可以接受任何正则表达式，但所有命名组都必须同时存在于`servicePattern`和中`routePattern`。如果`servicePattern`与不匹配`serviceId`，则使用默认行为。



#### 5.8 路由前缀

```yaml
zuul:
  prefix: /api
  strip-prefix: false
  routes: microservice-provider-user: /user/**
```

这样，访问`Zuul的/api/microservice-provider-user/1`路径，请求将被转发到`microserviceprovider-user的/api/1`。

```yaml
zuul:
  routes:
    microservice-provider-user:
      path: /user/**
      strip-prefix: true
```

这样访问`Zuul`的/user/1路径，请求将被转发到`microservice-provider-user的/user/1`。

> •可参考该Issue辅助理解：*https://github.com/spring-cloud/spring-cloudnetflix/issues/1365*.
>
> •该特性可能有Bug，相关Issue：*https://github.com/spring-cloud/springcloud-netflix/issues/1514*.

#### 5.9 忽略某些路径

​      前面讲解了如何忽略微服务，但有时还需要更细粒度的路由控制。例如，想让`Zuul`代理某个微服务，同时又想保护该微服务的某些敏感路径。此时，可使用ignored-Patterns，指定忽略的正则。

```yaml
zuul:
  ignored-patterns: /**/admin/**  #忽略所有包含/admin/的路径
  routes:  microservice-provider-user: /user/**
```

   这样就可将`microservice-provider-user`微服务映射到/user/**路径，但会忽略该微服务中所有包含`/admin/`的路径。

#### 5.10 本地转发

```yaml
zuul:
  routes: 
    route-name:
      path: /path-b/**
      url: forward: /path-b/**
```

这样，当访问`Zuul`的`/path-a/**`路径，将转发到`Zuul`的`/path-b/**`。

​      读者若无法掌握`Zuul`路由的规律，可将`com.netflix`包的日志级别设为DEBUG。这样，`Zuul`就会打印转发的具体细节，从而有助于更好地理解`Zuul`的路由配置

```yaml
logging:
  level: 
    com.netflix: debug
```



### 6. `Zuul`的安全与Header

#### 6.1 敏感Header的设置

​      您可以在同一系统中的服务之间共享标头，但是您可能不希望敏感标头泄漏到下游到外部服务器中。您可以在路由配置中指定忽略的标头列表。Cookies发挥着特殊的作用，因为它们在浏览器中具有定义明确的语义，并且始终将它们视为敏感内容。如果代理的使用者是浏览器，那么下游服务的cookie也会给用户带来麻烦，因为它们都混杂在一起（所有下游服务看起来都来自同一位置）。

​     如果您对服务的设计很谨慎（例如，如果只有一个下游服务设置cookie），则可以让它们从后端一直流到调用者。另外，如果您的代理设置cookie，并且所有后端服务都属于同一系统，则自然可以简单地共享它们（例如，使用Spring Session将它们链接到某些共享状态）。除此之外，由下游服务设置的任何cookie可能对调用者都没有用，因此建议您（至少）将`Set-Cookie`其`Cookie`放入不属于域的路由的敏感标头中。即使对于属于您网域的路由，在让Cookie在它们和代理之间流动之前，也请尝试仔细考虑其含义。

​    一般来说，可在同一个系统中的服务之间共享Header。不过应尽量防止让一些敏感的Header外泄。因此，在很多场景下，需要通过为路由指定一系列敏感Header列表

可以将敏感头配置为每个路由的逗号分隔列表，如以下示例所示：

```yaml
zuul:
  routes:
  microservice-provider-user:
      path: /user/**
      ensitive-headers: Cookie,Set-Cookie,Authorization
      url: https://downstream
```

这样就可为`microservice-provider-user`微服务指定敏感Header了。

也可用`zuul.sensitive-headers`全局指定敏感Header,如以下示例所示：

```yaml
 zuul:
  sensitive-headers: Cookie,Set-Cookie,Authorization  #默认是Cookie,Set-Cookie,Authorization
```

   需要注意的是，如果使用`zuul.routes.*.sensitive-headers`的配置方式，会覆盖全局的配置。

#### 6.2   忽略Header

可使用`zuul.ignored-headers`属性丢弃一些Header

```yaml
zuul:
  ignored-headers: Header1,Header2
```

这样设置后，`Header1`和`Header2`将不会传播到其他的微服务中。

​      默认情况下，`zuul.ignored-headers`是空值，但如果Spring Security在项目的`classpath`中，那么z`uul.ignored-headers`的默认值就是`Pragma,Cache-Cont rol,X-Frame-Options,X-Content-Type-Options,X-XSS-Protection,Expires`。所以，当Spring Security在项目的`classpath`中，同时又需要使用下游微服务的Spring Security的Header时，可以将`zuul.ignore-security-headers`设置为false。

> - `GitHub`上提的Issue：*https://github.com/spring-cloud/spring-cloudnetflix/issues/1487*，希望可以帮助大家理解。
>
> - sensitive-headers与ignored-headers的单元测试：*https://github.com/springcloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/test/java/org/springframework/cloud/netflix/zuul/filters/pre/PreDecorationFilterTests.java*.
>
> - 事实上，sensitive-headers会被添加到ignored-headers中，详见代码`org.springframework.cloud.netflix.zuul.fi lters.ProxyRequestHelper.add Ignored-Headers(String…)和org.springframework.cloud.netflix.zuul.fi lters.pre.Pre-DecorationFilter.run()。`



### 7. 使用`Zuul`上传文件

​     对于小文件（`1M`以内）上传，无须任何处理，即可正常上传。对于大文件（`10M`以上）上传，需要为上传路径添加`/zuul`前缀。也可使用`zuul.servlet-path`自定义前缀。

​       也就是说，假设`zuul.routes.microservice-file-upload=/microservice-file-upload/**`，如果`http://{HOST}:{PORT}/upload`是微服务`microservice-file-upload`的上传路径，则可使用`Zuul`的`/zuul/microservice-file-upload/upload`路径上传大文件。

如果`Zuul`使用了Ribbon做负载均衡，那么对于超大的文件（例如`500M`），需要提升超时设置

```yaml
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
ribbon:
  ConnectTimeout: 3000
  ReadTimeout: 60000
```

1.新建`microservice-file-upload`项目，并为其添加以下依赖

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
    <artifactId>microservice-file-upload</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-file-upload</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

2.在启动类上添加`@EnableEurekaClient`注解。

3.编写`FileUploadController`

```java
@RestController
public class FileUploadController {

    @PostMapping("/upload")
    public String handleFileUpload(@RequestParam("file") MultipartFile file) throws IOException {
        File fileToSave = new File(file.getOriginalFilename());
        FileCopyUtils.copy(file.getInputStream(),new FileOutputStream(fileToSave));
        return fileToSave.getAbsolutePath();
    }
}

```

4.配置文件`application.yml`,在其中添加如下内容。

```yml
server:
  port: 8888
spring:
  application:
    name: microservice-gateway-zuul-file-upload
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
management:
  endpoints:
    web:
      exposure:
        include: routes,filters
```

​          由配置可知，已经将该服务注册到Eureka Server上，并配置了文件上传大小的限制。这样一个文件上传的微服务就编写完成了。可使用以下命令测试文件上传。

```shell
$ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@文件全名" localhost:9999/upload
```

**测试**

目前使用的测试工具是CURL，大家也可使用其他工具进行测试。

1.准备1个小文件（`1M`以下），记为`small.file`；1个超大文件（`1G以上，2G以上`），记为`large.file`。

2.启动`microservice-discovery-eureka`。

3.启动`microservice-file-upload`。

4.启动`microservice-gateway-zuul`。

5.测试直接上传到`microservice-file-upload`上：使用命令`curl-F"file=@large.file"localhost:8888/upload`，上传大文件，发现可以正常上传。同理，小文件也可以上传。

6.测试通过`Zuul`上传小文件。

```shell
$ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@small.file" localhost:8888/microservice-file-upload/upload
```

7.测试通过`Zuul`上传大文件，不添加`/zuul`前缀。

```shell
$ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@large.file" localhost:8888/microservice-file-upload/upload
```

发现`Zuul`会报类似如下的异常：

![image-20200106224033124](/image/SpringCloud/image-20200106224033124.png)

8.测试通过`Zuul`上传大文件，添加`/zuul`前缀。

```shell
$ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@large.file" localhost:8888/zuul/microservice-file-upload/upload
```

此时，`Zuul`会报以下异常：

![image-20200106224510415](/image/SpringCloud/image-20200106224510415.png)

可以看到，此时已经不是文件大小限制的异常了，而只是`Hystrix`的超时异常。

9.新建项目`microservice-gateway-zuul-file-upload`，复制项目`microservice-gateway-zuul`，并在`application.yml`中添加如下内容。

```yaml
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
ribbon:
  ConnectTimeout: 30000
  ReadTimeout: 60000
```

添加好以后，完整`application.yml`如下：

```yaml
server:
  port: 8888
spring:
  application:
    name: microservice-gateway-zuul-file-upload
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
management:
  endpoints:
    web:
      exposure:
        include: routes,filters
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
ribbon:
  ConnectTimeout: 30000
  ReadTimeout: 60000
```



10.关闭`microservice-gateway-zuul`，启动`microservice-gateway-zuul-file-upload`再次

使用如下命令进行测试，此时已可正常上传文件。

```shell
$ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@large.file" localhost:8888/microservice-file-upload/upload
```



### 8. `Zuul`的过滤器

   过滤器是`Zuul`的核心组件，下面来详细讨论`Zuul`的过滤器。

#### 8.1 过滤器类型与请求生命周期

​      `Zuul`大部分功能都是通过过滤器来实现的。`Zuul`中定义了4种标准过滤器类型，这些过滤器类型对应请求的典型生命周期。

- `PRE`：这种过滤器在请求被路由之前调用。可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。

- `ROUTING`：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用`ApacheHttpClient`或`Netfilx Ribbon`请求微服务。

- POST：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的`HTTP Header`、收集统计信息和指标、将响应从微服务发送给客户端等。

- ERROR：在其他阶段发生错误时执行该过滤器。

​          除了默认的过滤器类型，`Zuul`还允许创建自定义的过滤器类型。例如，可以定制一种STATIC类型的过滤器，直接在`Zuul`中生成响应，而不将请求转发到后端的微服务。



`Zuul`请求的生命周期如图下图所示，该图详细描述了各类过滤器的执行顺序。

![](/image/SpringCloud/image-20200107210456789.png)

#### 8.2 内置过滤器详解

   Spring Cloud默认为`Zuul`编写并启用了一些过滤器，这些过滤器有什么作用呢？我们不妨结合`@EnableZuulServer、@EnableZuulProxy`两个注解进行讨论。

​      可将`@EnableZuulProxy`简单理解为`@EnableZuulServer`的增强版。事实上，当`Zuul`与`Eureka、Ribbon`等组件配合使用时，`@EnableZuulProxy`是我们最常用的注解，本书前文所使用的也是`@EnableZuulServer`。

​       探讨内置过滤器之前，我们先来了解一下什么是`RequestContext`，其用于在过滤器之间传递消息。它的数据保存在每个请求的`ThreadLocal`中。它用于存储请求路由到哪里、错误、`HttpServletRequest`、`HttpServletResponse`等信息。`RequestContext`扩展了`ConcurrentHashMap`，所以，理论上任何数据都可以存储在`RequestContext`中。

##### 8.2.1 `@EnableZuulServer`所启用的过滤器

- `pre`类型过滤器
  - `ServletDetectionFilter`：该过滤器用于检查请求是否通过`SpringDispatcher`。检查后，通过`FilterConstants.IS_DISPATCHER_SERVLET_REQUEST_KEY`设置布尔值。
  - `FormBodyWrapperFilter`：解析表单数据，并为请求重新编码。
  - `DebugFilter`：顾名思义，调试用的过滤器。当设置`zuul.include-debug-header=true`抑或设置`zuul.debug.request=true`，并在请求时加上了debug=true的参数，例如`http://ZUUL_HOST:ZUUL_PORT/some-path?debug=true`，就会开启该过滤器。该过滤器会把`RequestContext.setDebugRouting()`以及`RequestContext.setDebugRequest()`设为true。

- route类型过滤器

  - `SendForwardFilter`：该过滤器使用`Servlet RequestDispatcher`转发请求，转发位置存储在`RequestContext`的属性`FilterConstants.FORWARD_TO_KEY`中。这对转发到`Zuul`自身的端点很有用。可将路由设成：

    ```yaml
    zuul:
      routes: 
        abc:
          path: /path-a/**
          url: forward:/path-b
    ```

    然后访问`http://ZUUL_HOST:ZUUL_PORT/path-a/**`，观察该过滤器的执行过程。

- post类型过滤器
  
- `SendResponseFilter`：将代理请求的响应写入当前响应。
  
- error类型过滤器
  
  -   `SendErrorFilter`：若`RequestContext.getThrowable()`不为null，则默认转发到/error，也可以设置`error.path`属性来修改默认的转发路径。

##### 8.2.2  `@EnableZuulProxy`所启用的过滤器

​    如果使用注解`@EnableZuulProxy`，那么除上述过滤器之外，Spring Cloud还会安装以下过滤器。

- pre类型过滤器
  - `PreDecorationFilter`：该过滤器根据提供的`RouteLocator`确定路由到的地址，以及怎样去路由。同时，该过滤器还为下游请求设置各种代理相关的header。

- route类型过滤器

  -   `SimpleHostRoutingFilter`：该过滤器通过`Apache HttpClient`向指定的URL发送请求。URL在`RequestContext.getRouteHost()`中。

  -  `RibbonRoutingFilter`：该过滤器使用`Ribbon、Hystrix`和可插拔的HTTP客户端发送请求。`serviceId`在`RequestContext`的属性`FilterConstants.SERVICE_ID_KEY`中。该过滤器可使用如下这些不同的HTTP客户端。
    - `Apache HttpClient`：默认的HTTP客户端。
    - `Squareup OkHttpClient v3`：若需使用该客户端，需保证`com.squareup.okhttp3`的依赖在`classpath`中，并设置`ribbon.okhttp.enabled=true`。
    - `Netflix Ribbon HTTP Client`：设置`ribbon.restclient.enabled=true`即可启用该HTTP客户端。该客户端有一定限制，例如不支持PATCH方法，另外，它有内置的重试机制。



> -  建议读者阅读以上过滤器的源码，将对`Zuul`有一个更加深入的认识。其中，最重要的过滤器当属`RibbonRoutingFilter`了，因为整合`Ribbon、Hystrix`以及发送请求都在该过滤器中完成。
>
> - 形如如下内容的路由不会经过`RibbonRoutingFilter`，而是走`SimpleHostRoutingFilter`。
>
>   ```yaml
>   zuul:
>     routes: 
>       abc:
>         path: /user/**
>         url: http://localhost:8080/
>   ```
>
>   
>
> - 读者可借助/filters端点查看过滤器详情。
>
> - 目前`FormBodyWrapperFilte`r的代码实现并不高效，若你的应用没有Form表单提交，可禁用该过滤器，从而获取更好的性能表现。

#### 8.3　编写`Zuul`过滤器

​         理解过滤器类型和请求生命周期后，来编写一个`Zuul`过滤器。编写`Zuul`的过滤器非常简单，只需继承抽象类`ZuulFilter`，然后实现几个抽象方法就可以了。

那么现在来编写一个简单的`Zuul`过滤器，让该过滤器打印请求日志。

1.新建项目`microservice-gateway-zuulfilter`，复制`microservice-gateway-zuul`中的内容

2.编写`Pre`类型的过滤器

   前置过滤器会在`RequestContext`中设置数据，以供下游过滤器使用。主要用例是设置路由过滤器所需的信息。

```java
public class PreLogZuulFilter extends ZuulFilter {

    private static  final Logger LOGGER = LoggerFactory.getLogger(PreLogZuulFilter.class);
    @Override
    public String filterType() {
        //"pre"类型过滤器
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        //org.springframework.cloud.netflix.zuul.filters.pre.PreDecorationFilter执行之前执行
        return FilterConstants.PRE_DECORATION_FILTER_ORDER-1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();
        LOGGER.info("request.getMethod()=={},request.getRequestURI()=={}",request.getMethod(),request.getRequestURI());
        return null;
    }
}
```



3.编写Route类型的过滤器

路由过滤器在 前置过滤器之后运行，并向其他服务发出请求。这里的许多工作是将请求和响应数据与客户端所需的模型相互转换。

```java
public class RouteLogZuulFilter extends ZuulFilter {
    private  static final Logger  LOGGER = LoggerFactory.getLogger(RouteLogZuulFilter.class);
    @Override
    public String filterType() {
        //"route"类型过滤器
        return FilterConstants.ROUTE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SIMPLE_HOST_ROUTING_FILTER_ORDER-1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {

        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();
        LOGGER.info("request.getMethod()=={},request.getRequestURI()=={}",request.getMethod(),request.getRequestURI());
        return null;
    }
}
```



4.编写Post类型的过滤器

​     后置过滤器通常操纵响应。

```java
public class PostLogZuulFilter extends ZuulFilter {

    private  static  final Logger LOGGER = LoggerFactory.getLogger(PostLogZuulFilter.class);
    @Override
    public String filterType() {
        return FilterConstants.POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SEND_RESPONSE_FILTER_ORDER-1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();
        LOGGER.info("request.getMethod()=={},request.getRequestURI()=={}",request.getMethod(),request.getRequestURI());
        return  null;
    }
}
```

由代码可知，自定义的ZuulFilter需实现以下几个方法。

- `filterType`：返回过滤器的类型。有`pre、route、post、error`等几种取值，分别对应上文的几种过滤器。详细可以参考`com.netflix.zuul.Zuul Filter.filterType()`中的注释。

- `filterOrder`：返回一个int值来指定过滤器的执行顺序，不同的过滤器允许返回相同的数字。

- `shouldFilter`：返回一个boolean值来判断该过滤器是否要执行，true表示执行，false表示不执行。

- run：过滤器的具体逻辑。本例中让它打印了请求的HTTP方法以及请求的地址。

5.编写配置类

```java
@Configuration
public class ZuulFilterConfig {

    @Bean
    public PreLogZuulFilter preLogZuulFilter(){
        return new PreLogZuulFilter();
    }

    @Bean
    public PostLogZuulFilter postLogZuulFilter(){
        return  new PostLogZuulFilter();
    }

    @Bean
    public RouteLogZuulFilter routeLogZuulFilter(){
        return  new RouteLogZuulFilter();
    }
}
```

**测试**

1.启动`microservice-discovery-eureka`。

2.启动`microservice-provider-user`。

3.启动`microservice-gateway-zuul-filter`。

4.访问*http://localhost:8888/microservice-provider-user/user/1*，可获得类似如下的日志。

![image-20200107225825215](/image/SpringCloud/image-20200107225825215.png)

说明编写的自定义`Zuul`过滤器被执行了，而且执行顺序是`pre，route，post`；



> - 本小节只是演示了一个非常简单的`Zuul`过滤器。事实上，我们可使用过滤器做很多事情，例如安全认证、灰度发布、限流等。
>
> - 使用`Zuul`过滤器实现限流可参考：*http://www.itmuch.com/spring-cloudsum/spring-cloud-ratelimit/*.
>
> - 使用`Zuul`过滤器传播安全Token可参考：*https://gitee.com/itmuch/spring-cloud-yes/blob/master/zuul-server/src/main/java/com/itmuch/yes/KeycloakRouteZuulFilter.java*.



#### 8.4 禁用`Zuul`过滤器

​        Spring Cloud默认为`Zuul`编写并启用了一些过滤器，例如`DebugFilter、Form BodyWrapperFilter、PreDecorationFilter`等。这些过滤器都存放在s`pring-cloud-netflix-core`这个Jar包的`org.springframework.cloud.netflix.zuul.filters`包中。

在一些场景下，要想禁用部分过滤器，该怎么办呢？

答案非常简单，只需设置`zuul.<SimpleClassName>.<filterType>.disable=true`，即可禁用`SimpleClassName`所对应的过滤器。以过滤器`org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter`为例，只需设置`zuul.SendResponseFilter.post.disable=true`，即可禁用该过滤器。

同理，如果想禁用上面编写的过滤器，只需设置`zuul.PreLogZuulFilter.pre.disable=true`即可。



### 9. `Zuul`的容错与回退

1.启动项目`microservice-discovery-eureka`。

2.使用`application-8081.yml`启动项目`microservice-provider-user`。

3.启动项目`microservice-consumer-movie-ribbon`。

4.启动项目`microservice-gateway-zuul`。

5.启动项目`microservice-consumer-movie-feign-hystrix-dashboard`。

6.访问*http://localhost:8888/microservice-consumer-movie-ribbon/user/1*，可获得预期结果。

7.访问http://localhost:9083/hystrix

8.在`Hystrix Dashboard`中输入http://localhost:8888/actuator/hystrix.stream，随意指定某Title并单击Monitor Stream按钮后会显示相应的监控界面。

![image-20200108220804020](/image/SpringCloud/image-20200108220804020.png)

   由图可知，`Zuul`的`Hystrix`监控的粒度是微服务，而不是某个`API`；同时也说明，所有经过`Zuul`的请求，都会被`Hystrix`保护起来。    

9.关闭项目`microservice-consumer-movie-ribbon`，再次访问*http://localhost:8888/microservice-consumer-movie-ribbon/user/1*，会看到页面输出类似如下的异常：

![image-20200108221018008](/image/SpringCloud/image-20200108221018008.png)

  要想为`Zuul`添加回退，需要实现`ZuulFallbackProvider`接口。在实现类中，指定为哪个微服务提供回退，并提供一个`ClientHttpResponse`作为回退响应。

1.新建项目`microservice-gateway-zuul-fallback`，复制项目`microservice-gateway-zuul`；

2.编写`Zuul`回退类

```java
class AllFallbackProvider implements FallbackProvider {
    @Override
    public String getRoute() {
        //*代表回退所有服务，也可以指定的服务名
        return "*";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable throwable) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
} 
```

3.关闭`microservice-gateway-zuul`项目，启动`microservice-gateway-zuul-fallback`项目

添加回退之后，重复之前的实验，当`Zuul`所代理的任意微服务无法正常响应时，会返回如下内容：

![image-20200108223340653](/image/SpringCloud/image-20200108223340653.png)

​    在`Spring Cloud Edgware`版本之前，要想为`Zuul`回退，需实现`ZuulFallbackProvider`接口，从`Spring Cloud Edgware`开始`，ZuulFallbackProvider`接口已被标注为@Deprecated，由`FallbackProvider`替代。`FallbackProvider`提供了`ClientHttpResponse  fallbackResponse(Throwable cause)`;方法，为我们提供获取造成回退原因的能力。

### 10. 饥饿加载

我们知道，`Zuul`整合了Ribbon实现负载均衡，而Ribbon默认是懒加载的（），可能会导致首次请求较慢的问题。可使用如下配置配置饥饿加载。

```yaml
zuul:
  ribbon:
    eager-load:
      enabled: true
```



### 11. Query String编码

​         当处理请求时，`query param`会被解码，因此，可在`Zuul`过滤器中进行一些适当的修改。这些参数在route过滤器中构建请求时，将被重新编码。如果`query param`使用JavaScript的`encodeURIComponent()`方法进行编码，那么重新编码后的结果可能与原始值不同——尽管在大多数情况下不会引起问题，但某些Web服务器可能会对复杂的query string进行编码。

要强制让`query string与HttpServletRequest.getQueryString()`保持一致，可使用如下配置：

```yaml
 zuul:
  forceOriginalQueryStringEncoding: true
```

> - 该特殊标志只适用于`SimpleHostRoutingFilter`，并且由于query string在原始的`HttpServletRequest`上获取，你将无法使用`RequestContext.getCurrentContext().setRequestQueryParams(someOver riddenParameters)`重写`query param`。
>
> - `forceOriginalQuerySt ringEncoding`处理的相关代码可详见`org.springf ramework.cloud.netflix.zuul.filters.route.SimpleHostRoutingFilter#buildHttpRequest`。



### 12. `Hystrix`隔离策略与线程池

我们知道`Zuul`集成了`Hystrix`，而`Hystrix`有隔离策略——THREAD及SEMAPHORE；

#### 12.1   隔离策略

不妨做一个简单的实验。

1.为项目`microservice-gateway-zuul-fallback`设置`zuul.ribbon-isolation-strategy=thread`。

2.按照**9. `Zuul`的容错与回退**的操作步骤会得到下图图表：

![image-20200108224105516](/image/SpringCloud/image-20200108224105516.png)

对比2次图表，会发现前面的Thread Pools 一栏没有数据，而后面的有，这说明两个问题：

- 默认情况下，`Zuul`的`Hystrix`隔离策略是SEMAPHORE。

- 可使用`zuul.ribbon-isolation-strategy=thread`将隔离策略改为THREAD。



#### 12.2 线程池配置

​     由上图可知，当`zuul.ribbon-isolation-strategy=thread`时，`Hystrix`的线程隔离策略将作用于所有路由，`HystrixThreadPoolKey`默认为`RibbonCommand`，这意味着，所有路由的`HystrixCommand`都会在相同的`Hystrix`线程池中执行。

可使用以下配置，让每个路由使用独立的线程池：

```yaml
zuul:
  ribbon-isolation-strategy: thread
  thread-pool:
    use-separate-thread-pools: true
```

这样，`HystrixThreadPoolkey`将与路由的服务标识相同。对于本例，则是`microservice-provider-user`。

如果想为`HystrixThreadPoolKey`添加前缀，可使用类似如下的配置：

```yaml
zuul:
  ribbon-isolation-strategy: thread
  thread-pool:
    use-separate-thread-pools: true
    thread-pool-key-prefix: prefix-
```

这样，`HystrixThreadPoolkey`将变成${prefix}-{服务标识}。对于本例，则是`prefix-microservice-provider-user`。

![image-20200108224945089](/image/SpringCloud/image-20200108224945089.png)

> 相关PullRequest：*https://github.com/spring-cloud/spring-cloud-netflix/pull/2074*.





### 13.`Zuul`的高可用

​          ` Zuul`的高可用非常关键，因为外部请求到后端微服务的流量都会经过`Zuul`。故而在生产环境中一般都需要部署高可用的`Zuul`以避免单点故障。

​    下面分两种场景讨论`Zuul`的高可用。

 本节中的“`Zuul客户端`”是指广义上的“客户端”，即向`Zuul`发送的浏览器、手机等终端。

#### 13.1 `Zuul`客户端也注册到了Eureka Server上

​       这种情况下，`Zuul`的高可用非常简单，只需将多个`Zuul`节点注册到`EurekaServer`上，就可实现`Zuul`的高可用。此时，`Zuul`的高可用与其他微服务的高可用没什么区别。

​     如下图，当`Zuul`客户端也注册到`EurekaServer`上时，只需部署多个`Zuul`节点即可实现其高可用。`Zuul`客户端会自动从`EurekaServer`中查询`ZuulServer`的列表，并使用Ribbon负载均衡地请求`Zuul`集群。

![](/image/SpringCloud/image-20200108225545789.png)

#### 13.2  `Zuul`客户端未注册到Eureka Server上

​     现实中，这种场景往往更常见，例如，`Zuul`客户端是一个手机`APP`——不可能让所有的手机终端都注册到Eureka Server上。这种情况下，可借助一个额外的负载均衡器来实现`Zuul`的高可用，例如`Nginx`、`HAProxy`、`F5`等。

如下图，`Zuul`客户端将请求发送到负载均衡器，负载均衡器将请求转发到其代理的其中一个`Zuul`节点。这样，就可以实现`Zuul`的高可用。

![](/image/SpringCloud/image-20200108225945654.png)

### 14. 使用`Zuul`聚合微服务

​          许多场景下，外部请求需要查询`Zuul`后端的多个微服务。举个例子，一个电影售票手机`APP`，在购票订单页上，既需要查询“电影微服务”获得电影相关信息，又需要查询“用户微服务”获得当前用户的信息。如果让手机端直接请求各个微服务（即使使用`Zuul`进行转发），那么网络开销、流量耗费、耗费时长可能都无法令我们满意。那么对于这种场景，可使用`Zuul`聚合微服务请求——手机`APP`只需发送一个请求给`Zuul`，由`Zuul`请求用户微服务以及电影微服务，并组织好数据给手机`APP`。

​          使用这种方式，在手机端只需发送一次请求即可，简化了客户端侧的开发；不仅如此，由于`Zuul`、用户微服务、电影微服务一般都在同一个局域网中，因此速度会非常快，效率会非常高。

下面围绕以上场景，来编写代码示例。在本例中，使用`RxJava`结合`Zuul`来实现微服务请求的聚合。

1.新建`microservice-gateway-zuul-aggregation`，复制项目`microservice-gateway-zuul`

2.创建`ZuulAggregationConfig`类

```java
@Configuration
public class ZuulAggregationConfig {

    @Bean
    public RestTemplate restTemplate(){
        return  new RestTemplate();
    }
}
```

3.创建实体类User

```java
public class User implements Serializable {
    private Long id;

    private String name;

    private Integer age;

    private BigDecimal balance;

    private String dataBase;

    private static final long serialVersionUID = 1L;
    //get set 。。。。
    }
```

4.创建Java类，名为`AggregationService`：

```java
@Service
public class AggregationService {
    @Autowired
    private RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "fallback")
    public Observable<User> getUserById(Long id) {
        //创建一个被观察者
        return Observable.create(observer -> {
            //请求用户微服务的user/{id}端点
            User user = restTemplate.getForObject("http//:MICROSERVICE-PROVIDER-USER/user/{id}",
                    User.class, id);
            observer.onNext(user);
            observer.onCompleted();
        });
    }

    @HystrixCommand(fallbackMethod = "fallback")
    public Observable<User> getUserByIdToMovie(Long id) {
        //创建一个被观察者
        return Observable.create(observer -> {
            //请求电影微服务的user/{id}端点
            User user = restTemplate.getForObject("http://MICROSERVICE-CONSUMER-MOVIE-RIBBON/user/{id}",
                    User.class, id);
            observer.onNext(user);
            observer.onCompleted();
        });
    }

    public User fallback(Long id) {
        User user = new User();
        user.setId(-1L);
        return user;
    }
}
```

5.创建Controller，在Controller中聚合多个请求：

```java
@RestController
public class AggregationController {
    
    private static  final Logger LOGGER = LoggerFactory.getLogger(AggregationController.class);

    @Autowired
    private AggregationService aggregationService;

    @GetMapping("/aggregation/{id}")
    public DeferredResult<HashMap<String, User>> aggregation(@PathVariable("id") Long id){
        Observable<HashMap<String, User>> result = this.aggregationObservable(id);
        return this.toDeferredResult(result);
    }
    public Observable<HashMap<String, User>> aggregationObservable( Long id){
        //合并2个或者多个Observable发射出的数据项，根据指定的函数变化它们
        return Observable.zip(
                aggregationService.getUserById(id),
                aggregationService.getUserByIdToMovie(id),
                (user,movieUser)->{
                    HashMap<String,User> map = Maps.newHashMap();
                    map.put("user",user);
                    map.put("movieUser",movieUser);
                    return map;
                }
        );

    }
    public DeferredResult<HashMap<String, User>> toDeferredResult(Observable<HashMap<String, User>>  datails){

        DeferredResult<HashMap<String, User>> result=  new DeferredResult<>();
        //订阅
        datails.subscribe(new Observer<HashMap<String, User>>() {
            @Override
            public void onCompleted() {
                LOGGER.info("onCompleted");
            }

            @Override
            public void onError(Throwable throwable) {
                LOGGER.info("onError",throwable);
            }

            @Override
            public void onNext(HashMap<String, User> stringUserHashMap) {
               result.setResult(stringUserHashMap);
            }
        });
        return result;
    }
}
```

​     这样，代码就编写完成了。相信熟悉`RxJava`的读者朋友们能非常轻松地阅读本段代码的含义，对于不了解的`RxJava`的读者朋友们，建议花一点时间入门`RxJava`。

**测试一、微服务聚合测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user`。

3.启动项目`microservice-consumer-movie-ribbon`。

4.启动项目`microservice-gateway-zuul-aggregation`。

5.访问*http://localhost:8888/aggregation/1*，可获得如下结果：

![image-20200109001127642](/image/SpringCloud/image-20200109001103639.png)

说明已成功用`Zuul`聚合了用户微服务以及电影微服务的`RESTful`   `API`。

**测试二、**`Hystrix`**容错测试**

1.在测试一的基础上，停止项目`microservice-provider-user`以及`microservice-consumer-movie-ribbon`。

2.访问*http://localhost:8888/aggregation/1*，可获得如下结果：

![image-20200108235930414](/image/SpringCloud/image-20200108235930414.png)

说明`fallback`方法正常被触发，能够正常回退。



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