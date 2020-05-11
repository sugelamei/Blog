---
title: Spring Cloud-04 Feign实现声明式REST调用
date: 2019-11-21 10:28:34 
tags: 
  - Spring Cloud
  - Feign
typora-root-url: ..
---



### 0. 说在前面

```yaml
- JDK：Spring Cloud官方建议使用JDK 1.8;
- Spring Boot：笔者使用Spring Boot 2.1.10.RELEASE;
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；
```



前文的示例中是使用`RestTemplate`实现`REST API`调用的，代码大致如下：

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

<!-- more -->

由代码可知，我们是使用拼接字符串的方式构造URL的，该URL只有一个参数。然而在现实中，URL中往往有多个参数。如果这时还使用这种方式构造URL，那么就会变得很低效，并且难以维护。举个例子，想要请求这样的URL：

```java
localhost:8081/search?name=张三&age=18&dataBase=microservice01
```

若使用拼接字符串的方式构建请求URL，那么代码可编写如下：

```java
  public User[] search(String name,Integer age,String dataBase){
      Map<String,Object> map = new HashMap<>();
      map.put("name",name);
      map.put("age",age);
      map.put("dataBase",dataBase);
      return  restTemplate.getForObject("http://microservice-provider-user-ribbon/search?name={name}&age={age}&dataBase={dataBase}",User[].class,map);
  }
```

在这里，URL仅包含3个参数。如果URL更加复杂，例如有10个以上的参数，那么代码会变得难以维护。

如何解决这种问题呢？请大家带着疑问继续往下看。

### 1. Feign简介

​      Feign是`Netflix`开发的声明式、模板化的HTTP客户端，其灵感来自`Retrofit、JAXRS-2.0`以及`WebSocket`。Feign可帮助我们更加便捷、优雅地调用`HTTPAPI`。

​      在Spring Cloud中，使用Feign非常简单——创建一个接口，并在接口上添加一些注解，代码就完成了。Feign支持多种注解，例如Feign自带的注解或者`JAX-RS`注解等。

​     Spring Cloud对Feign进行了增强，使Feign支持了`SpringMVC`注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便。

Feign的GitHub：`https://github.com/OpenFeign/feign`

### 2. 为服务消费者整合Feign

1.新建项目`microservice-consumer-movie-feign`，将项目`microservice-consumer-movie`中的相关代码复制过来；

2.整体`pom.xml`如下：

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
    <groupId>com.sugalemei</groupId>
    <artifactId>microservice-consumer-movie-feign</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-feign</name>
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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

</project>

```

3.创建一个Feign接口，并添加`@FeignClient`注解。

```java
@FeignClient(name = "microservice-provider-user-ribbon")
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

`@FeignClient`注解中的`microservice-provider-user-ribbon`是一个任意的客户端名称，用于创建Ribbon负载均衡器。在本例中，由于使用了Eureka，所以Ribbon会把`microservice-provider-user-ribbon`解析成`EurekaServer`服务注册表中的服务。

​      当然，如果不想使用Eureka，可使用`<serviceClientName>.ribbon.listOfServers`属性配置服务器列表。

还可使用`url`属性指定请求的URL（URL可以是完整的URL或者主机名），

例如

```java
@FeignClient(name="microservice-provider-user-ribbon",ur l="http://localhost:8000/")。
```

4.修改Controller代码，让其调用Feign接口。

```java
@RestController
public class MovieController {
    @Autowired
    private UserFeignClient userFeignClient;

    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id) {
        return userFeignClient.findById(id);
    }

}
```



5.修改启动类，为其添加`@EnableFeignClients`注解，如下：

```java
@SpringBootApplication
@EnableFeignClients
public class MicroserviceConsumerMovieFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(MicroserviceConsumerMovieFeignApplication.class, args);
    }

}
```

这样，电影微服务就可以用Feign去调用用户微服务的`API`了。

**测试**

1.启动`microservice-discovery-eureka`。

2.启动2个或更多`microservice-provider-user`实例。

3.启动`microservice-consumer-movie-feign`。

4.多次访问http://localhost:9081/user/1，返回如下结果并能看到`dataBase`字段的变化。

![](/image/SpringCloud/image-20191205233918071.png)

以上结果说明，我们不但实现了声明式的`REST API`调用，同时还实现了客户端侧的负载均衡。

### 3. 自定义Feign配置

​     很多场景下，我们需要自定义Feign的配置，例如配置日志级别、定义拦截器等。从`Spring Cloud Edgware`开始允许使用Java代码或属性自定义Feign的配置，两种方式是等价的。

#### 3.1  使用Java代码自定义Feign配置

下面来讨论如何使用Java代码自定义Feign的配置。

​     在Spring Cloud中，Feign的默认配置类是`FeignClientsConfiguration`，该类定义了Feign默认使用的编码器、解码器、所使用的契约等。

​     Spring Cloud允许通过注解`@FeignClient`的configuration属性自定义Feign的配置，自定义配置的优先级比`FeignClientsConfiguration`要高。

​        在`SpringCloud`文档中可看到以下段落，描述了`SpringCloud`提供的默认配置。另外，有的配置尽管没有提供默认值，但是Spring也会扫描其中列出的类型（也就是说，这部分配置也能自定义）。



Spring Cloud Netflix provides the following beans by default for feign (`BeanType` beanName: `ClassName`):

- `Decoder` feignDecoder: `ResponseEntityDecoder` (which wraps a `SpringDecoder`)
- `Encoder` feignEncoder: `SpringEncoder`
- `Logger` feignLogger: `Slf4jLogger`
- `Contract` feignContract: `SpringMvcContract`
- `Feign.Builder` feignBuilder: `HystrixFeign.Builder`
- `Client` feignClient: if Ribbon is enabled it is a `LoadBalancerFeignClient`, otherwise the default feign client is used.

The OkHttpClient and ApacheHttpClient feign clients can be used by setting `feign.okhttp.enabled` or `feign.httpclient.enabled` to `true`, respectively, and having them on the classpath. You can customize the HTTP client used by providing a bean of either `ClosableHttpClient` when using Apache or `OkHttpClient` when using OK HTTP.

Spring Cloud Netflix *does not* provide the following beans by default for feign, but still looks up beans of these types from the application context to create the feign client:

- `Logger.Level`
- `Retryer`
- `ErrorDecoder`
- `Request.Options`
- `Collection`
- `SetterFactory`

##### 3.1.1  配置指定名称的Feign Client

​     由此可知，在`Spring Cloud`中，Feign默认使用的契约是`SpringMvcContract`，因此它可以使用`SpringMVC`的注解。下面来自定义Feign的配置，让它使用Feign自带的注解进行工作。

1.新建项目`microservice-consumer-movie-feign-customizing`,复制项目`microservice-consumer-movie-feign`。

2.创建Feign的配置类。

```java
/*
 该类为Feign的配置类
 注意:该类可以不写@Configuration注解；如果加了@Configuration注解，
 就不能放在主应用程序上下文@ComponentScan所扫描的包中。
 */
public class FeignConfig {
    /*
     将契约改为fegin原生的默认契约，
     这样就可以使用feign自带的注解了。
     */
    @Bean
    public Contract feignContract(){
        return new Contract.Default();
    }
}
```

3.`Feign`接口修改为如下，使用`@FeignClient`的configuration属性指定配置类，同时，将findById上的`SpringMVC`注解修改为Feign自带的注解。

类似地，还可自定义Feign的编码器、解码器、日志打印，甚至为Feign添加拦截器。

```java
   /*
   使用默认的编码器
     */
    @Bean
    public Encoder feignEncoder() {
        return new Encoder.Default();
    }

    /*
    使用默认的解码器
     */
    @Bean
    public Decoder feignDecoder() {
        return new Decoder.Default();
    }
```



**测试**

1.启动`microservice-discovery-eureka`。

2.启动`microservice-consumer-movie-ribbon`。

3.启动`microservice-consumer-movie-feign-customizing`。

4.访问`http://localhost:9081/user/1`，可获得如下内容：

![](/image/SpringCloud/image-20191207172024438.png)

说明已经实现Feign的配置自定义。

​      在Spring Cloud 中，Feign的配置类（例如本例中的`FeignConfig`类）**无须**添加@Configuration注解；如果加了@Configuration注解，那么该类不能存放在主应用程序上下文`@ComponentScan`所扫描的包中。否则，该类中的配置的`Decoder`、`Encoder`、`Contract`等配置就会被所有的`@FeignClient`共享。

​      为避免造成问题,最佳实践是不在指定名称的Feign配置类上添加@Configuration注解。

##### 3.1.2  全局配置

注解`@EnableFeignClients`为我们提供了`defaultConfiguration`属性，用来指定默认的配置类

```java
@EnableFeignClients(defaultConfiguration = FeignConfig.class)
```

#### 3.2  使用属性自定义Feign配置

​    从`Spring Cloud Netflix 1.4.0`开始（即从`Spring Cloud Edgware`开始，Feign支持使用属性自定义。这种方式比使用Java代码配置的方式更加方便。

##### 3.2.1  配置指定名称的Feign Client

​     对于指定名称的Feign Client，配置如下,其中名称为`@FeignClient(name="microservice-provider-user-ribbon"）`中的name对应的值。

```yaml
feign:
  client:
    config:
      microservice-provider-user-ribbon:
        loggerLevel: BASIC
        connectTimeout: 5000
        readTimeout: 5000
```

##### 3.2.2  通用配置

​     上面讨论了如何配置指定名称的Feign Client，如果想配置所有的Feign Client，该怎么办呢？只需做如下配置即可：

```yaml
feign:
  client:
    config:
      default:
       loggerLevel: BASIC
       connectTimeout: 5000
       readTimeout: 5000
```

属性配置的方式比Java代码配置的方式优先级更高。如果你想让Java代码配置方式优先级更高，可使用这个属性：

```yaml
feign:
  client:
    default-to-properties: false
```



### 4. Feign对继承的支持

   Feign支持继承。使用继承，可将一些公共操作分组到一些父接口中，从而简化Feign的开发，下面展示了简单的例子。

基础接口：`UserService.java`

```java
public interface UserService {
  @GetMapping("/user/{id}")
    User  getUser(@PathVariable("id") Long id);
}
```



服务提供者Controller：`UserController.java`

```java
@RestController
public class UserController implements UserService {
    @Override
    public User getUser(Long id) {
        return null;
    }
}
```



服务消费者：`UserClient.java`

```java
@FeignClient(name = "user")
public interface UserClient extends UserService {

}
```



尽管Feign的继承可帮助我们进一步简化Feign的开发，但Spring Cloud官方指出——通常情况下，不建议在服务器端与客户端之间共享接口，因为这种方式造成了服务器端与客户端代码的紧耦合。并且，Feign本身并不使用`Spring MVC`的工作机制（方法参数映射不被继承）。

笔者认为，应客观看待“紧耦合”与“方便性”，并在权衡利弊后做出取舍——放弃开发的方便性或接受代码的紧耦合。以下几篇帖子对“紧耦合”与“方便性”进行了深入地讨论，有兴趣的读者可前往查看：

•*https://github.com/spring-cloud/spring-cloud-netflix/issues/951*

•*https://github.com/spring-cloud/spring-cloud-netflix/issues/659*

•*https://github.com/spring-cloud/spring-cloud-netflix/issues/646*

•*https://jmnarloch.wordpress.com/2015/08/19/spring-cloud-designing-feignclient/*



### 5. Feign对压缩的支持

在一些场景下，可能需要对请求或响应进行压缩，此时可使用以下属性启用Feign的压缩功能。

```yaml
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

对于请求的压缩，Feign还提供了更为详细的设置，例如：

```yaml
feign.compression.request.enabled=true
feign.compression.request.mime-types= "text/xml"
feign.compression.request.min-request-size= 2048
```

其中，`feign.compression.request.mime-types`用于支持的媒体类型列表，默认是`text/xml`、`application/xml`以及`application/json`。

`feign.compression.request.min-request-size`用于设置请求的最小阈值，默认是2048。

### 6. Feign的日志

在很多场景下，需要了解Feign处理请求的具体细节，那么如何满足这种需求呢？

Feign对日志的处理非常灵活，可为每个Feign客户端指定日志记录策略，每个Feign客户端都会创建一个logger。默认情况下，logger的名称是Feign接口的完整类名。需要注意的是，Feign的日志打印只会对DEBUG级别做出响应。

我们可为每个Feign客户端配置各自的`Logger.Level`对象，告诉Feign记录哪些日志。`Logger.Level`的值有以下选择。

- NONE：不记录任何日志（默认值）。

- BASIC：仅记录请求方法、URL、响应状态代码以及执行时间。

- HEADERS：记录BASIC级别的基础上，记录请求和响应的header。

- FULL：记录请求和响应的header、body和元数据。



#### 6.1 编码方式设置日志级别

下面来为前面编写的`UserFeignClient`添加日志打印，将它的日志级别设置为FULL。

1.新建`microservice-consumer-movie-feign-logging`项目，复制项目`microservice-consumer-movie-feign`中的代码。

2.编写Feign配置类：

```java
@Configuration
public class FeignLogConfig {

    @Bean
    Logger.Level  feignLogLevel(){
        return Logger.Level.FULL;
        //return Logger.Level.BASIC;
    }
}

```



3.修改Feign的接口，指定配置类：

```java

@FeignClient(name = "microservice-provider-user-ribbon",configuration = FeignLogConfig.class)
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```



4.在`application.yml`指定Feign接口的日志级别为DEBUG，添加后的内容如下：

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-feign
eureka:
  client:
     service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

logging:
  level:
     com.sugelamei.client.UserFeignClient: debug
```



**测试**

1.启动`microservice-discovery-eureka`。

2.启动`microservice-provider-user-ribbon`。

3.启动`microservice-consumer-movie-feign-logging`。

4.访问`http://localhost:9081/user/1`，可以看到类似如下的日志。

![](/image/SpringCloud/image-20191210195340951.png)

可以看到，Feign已将请求的各种细节非常详细地记录了下来。

5.将日志Feign的日志级别改为BASIC，重启项目后再次访问`http://localhost:9081/user/1`，会发现此时只会打印请求方法、请求的URL、响应的状态码和响应的时间等信息。

![](/image/SpringCloud/image-20191210200341006.png)

#### 6.2  使用属性配置日志级别

 从`Spring Cloud Edgware`开始，也可使用配置属性直接定义Feign的日志级别，例如：

```yaml
logging:
  level:
     com.sugelamei.client.UserFeignClient: debug
#代码和配置二选一   
feign:
  client:
    config:
      microservice-consumer-movie-feign:
        loggerLevel: full
```

### 7. 使用Feign构造多参数请求

​     本节探讨如何使用Feign构造多参数的请求，以GET以及POST方法的请求为例，其他方法（例如DELETE、PUT等）的请求原理相同；

假设请求的URL包含多个参数，例如`http://microservice-provider-user/get?id=1&name=张三`，要如何构造呢？

#### 7.1 修改用户微服务

1.新建`microservice-provider-user-multiple-params`项目，将`microservice-provider-user`代码复制过来；

2.修改`UserController`，修改后如下：

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/user/{id}")
    public User findUserById(@PathVariable("id") Long id) {
        return userService.selectByPrimaryKey(id);
    }

 
    @GetMapping("/get")
    public User get(User user){
        return user;
    }


    @PostMapping("/post")
    public User post(@RequestBody User user){
        return user;
    }
}

```

3.修改`application.yml`，修改后的内容如下：

```yaml
#服务端口
server:
  port: 8080
spring:
  application:
    name: microservice-provider-user-multiple-params
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



#### 7.2  修改电影微服务

1.新建`microservice-consumer-movie-feign-multiple-params`，将`microservice-consumer-movie-feign`项目中的代码复制过来；

2.修改`UserFeignClient` ，修改后的代码如下：

```java
@FeignClient(name = "microservice-provider-user-multiple-params")
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);

    // 该请求不会成功
    @GetMapping("/get")
    User get(User user);

    @GetMapping("/get")
    User get0(@SpringQueryMap User user);

    @GetMapping("/get")
    User get1(@RequestParam("id") Long id, @RequestParam("name") String name);

    @GetMapping("/get")
    User get2(@RequestParam Map<String, Object> map);

    @PostMapping("/post")
    User post(@RequestBody User user);
}
```



3.修改`MovieController` ，修改后的代码如下：

```java
@RestController
public class MovieController {
    @Autowired
    private UserFeignClient userFeignClient;

    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id) {
        return userFeignClient.findById(id);
    }


    /**
     * 测试URL：http://localhost:9081/user/get?id=1&name=张三
     * 该请求不会成功。
     * @param user user
     * @return 用户信息
     */
    @GetMapping("/user/get")
    public User get(User user) {
        return this.userFeignClient.get0(user);
    }

    /**
     * 测试URL：http://localhost:9081/user/get0?id=1&name=张三
     * @param user user
     * @return 用户信息
     */
    @GetMapping("/user/get0")
    public User get0(User user) {
        return this.userFeignClient.get0(user);
    }
    /**
     * 测试URL：http://localhost:9081/user/get1?id=1&name=张三
     * @param user user
     * @return 用户信息
     */
    @GetMapping("/user/get1")
    public User get1(User user) {
        return this.userFeignClient.get1(user.getId(), user.getName());
    }

    /**
     * 测试URL：http://localhost:9081/user/get2?id=1&name=张三
     * @param user user
     * @return 用户信息
     */
    @GetMapping("/user/get2")
    public User get2(User user) {
        Map<String, Object> map = Maps.newHashMap();
        map.put("id", user.getId());
        map.put("name", user.getName());
        return this.userFeignClient.get2(map);
    }

    /**
     * 测试URL:http://localhost:9081/user/post?id=1&username=张三
     * @param user user
     * @return 用户信息
     */
    @GetMapping("/user/post")
    public User post(User user) {
        return this.userFeignClient.post(user);
    }
}
```



3.修改`application.yml`,修改内容如下：

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-feign-multiple-params
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```



#### 7.3  测试

1.启动`microservice-discovery-eureka`。

2.启动`microservice-provider-user-multiple-params`。

3.启动`microservice-consumer-movie-feign-multiple-params`。

##### 7.3.1  GET请求多参数的URL

我们知道，Spring Cloud为Feign添加了`SpringMVC`的注解支持，那么不妨按照`Spring MVC`的写法尝试一下：

```java
@FeignClient(name = "microservice-provider-user-multiple-params")
public interface UserFeignClient {
    @GetMapping("/get")
    User get0(User user);
}

```

然而，这种写法并不正确，控制台会输出类似如下的异常。

![](/image/SpringCloud/image-20191210231151525.png)



那么应该如何正确的写呢？

- 方式一:

```java
@FeignClient(name = "microservice-provider-user-multiple-params")
public interface UserFeignClient {
    @GetMapping("/get")
    User get0(@SpringQueryMap User user);

}
```

在调用时，可使用类似以下的代码：

```java
    @GetMapping("/user/get0")
    public User get0(User user) {
        return this.userFeignClient.get0(user);
    }
```

使用`@SpringQueryMap`注解(老版本可能不支持，笔者使用的版本支持此功能)。

访问`http://localhost:9081/user/get1?id=1&name=张三`，结果如下所示：

![](/image/SpringCloud/image-20191211214150460.png)



- 方式二：

```java
@FeignClient(name = "microservice-provider-user-multiple-params")
public interface UserFeignClient {
    @GetMapping("/get")
    User get1(@RequestParam("id") Long id, @RequestParam("name") String name);

}
```

在调用时，可使用类似以下的代码：

```java
    @GetMapping("/user/get1")
    public User get1(User user) {
        return this.userFeignClient.get1(user.getId(), user.getName());
    }
```



这是最为直观的方式，URL有几个参数，Feign接口中的方法就有几个参数。使用@RequestParam注解指定具体的请求参数。

访问`http://localhost:9081/user/get1?id=1&name=张三`，结果如下所示：

![](/image/SpringCloud/image-20191210231934979.png)





- 方式三：

多参数的URL也可使用Map来构建。当目标URL参数非常多时，可使用这种方式简化Feign接口的编写。

```java
@FeignClient(name = "microservice-provider-user-multiple-params")
public interface UserFeignClient {
    @GetMapping("/get")
    User get2(@RequestParam Map<String, Object> map);

}

```

在调用时，可使用类似以下的代码：

```java
    @GetMapping("/user/get2")
    public User get2(User user) {
        Map<String, Object> map = Maps.newHashMap();
        map.put("id", user.getId());
        map.put("name", user.getName());
        return this.userFeignClient.get2(map);
    }
```

访问`http://localhost:9081/user/get2?id=1&name=张三`，结果如下所示：

![image-20191210231733711](/image/SpringCloud/image-20191210231733711.png)

##### 7.3.2  POST请求包含多个参数

​     如何使用Feign构造包含多个参数的POST请求。假设服务提供者的Controller是这样编写的：

```java
@RestController
public class UserController {

    @PostMapping("/post")
    public User post(@RequestBody User user){
       //此处用于测试直接返回接受到的数据
        return user;
    }
```

要如何使用Feign去请求呢？

```java
@FeignClient(name = "microservice-provider-user-multiple-params")
public interface UserFeignClient {
    @PostMapping("/post")
    User post(@RequestBody User user);
}

```

访问`http://localhost:9081/user/post?id=1&username=张三`，结果如下所示：

![](/image/SpringCloud/image-20191211210159571.png)

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

