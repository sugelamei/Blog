---
title: Spring Cloud-05 Hystrix实现微服务的容错处理
date: 2019-11-24 09:20:13 
tags: 
  - Spring Cloud
  - Hystrix
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

​     

​        前面的文章已经使用`Eureka`实现了微服务的注册与发现，`Ribbon`实现了客户端侧的负载均衡，`Feign`实现了声明式的`API`调用。

​       下面我们就来讲解如何使用`Hystrix`实现微服务的容错。

### 1. 实现容错的手段

​        如果服务提供者响应非常缓慢，那么消费者对提供者的请求就会被强制等待，直到提供者响应或超时。在高负载场景下，如果不做任何处理，此类问题可能会导致服务消费者的资源耗竭甚至整个系统的崩溃。例如，曾经发生过一个案例——某电子商务网站在一个黑色星期五发生过载。过多的并发请求，导致用户支付的请求延迟很久都没有响应，在等待很长时间后最终失败。支付失败又导致用户重新刷新页面并再次尝试支付，进一步增加了服务器的负载，最终整个系统都崩溃了。

<!-- more -->    

​     当依赖的服务不可用时，服务自身会不会被拖垮？这是我们要考虑的问题。

#### 1.1  雪崩效应

​         微服务架构的应用系统通常包含多个服务层。微服务之间通过网络进行通信，从而支撑起整个应用系统，因此，微服务之间难免存在依赖关系。我们知道，任何微服务都并非100%可用，网络往往也很脆弱，因此难免有些请求会失败。

​         我们常把“基础服务故障”导致“级联故障”的现象称为雪崩效应。雪崩效应描述的是提供者不可用导致消费者不可用，并将不可用逐渐放大的过程。

​        如下图所示：A作为服务提供者（基础服务），B为A的服务消费者，C和D是B的服务消费者。当A不可用引起B的不可用，并将不可用像滚雪球一样的额放大到C和D时，雪崩效应就产生了。

 ![](/image/SpringCloud/image-20191214093251456.png)



#### 1.2  如何容错

​    要想防止雪崩效应，必须有一个强大的容错机制。该容错机制需实现以下两点：

- 为网络请求设置超时：必须为网络请求设置超时。正常情况下，一个远程调用一般在几十毫秒内就能得到响应了。如果依赖的服务不可用或者网络有问题，那么响应时间就会变得很长（几十秒）。

​        通常情况下，一次远程调用对应着一个线程/进程。如果响应太慢，这个线程/进程就得不到释放。而线程/进程又对应着系统资源，如果得不到释放的线程/进程越积越多，资源就会逐渐被耗尽，最终导致服务的不可用。

​       因此，必须为每个网络请求设置超时，让资源尽快释放。

- 使用断路器模式：试想一下，如果家里没有断路器，当电流过载时（例如功率过大、短路等），电路不断开，电路就会升温，甚至可能烧断电路、引发火灾。使用断路器，电路一旦过载就会跳闸，从而可以保护电路的安全。在电路超载的问题被解决后，只需关闭断路器，电路就可以恢复正常。

  ​        同理，如果对某个微服务的请求有大量超时（常常说明该微服务不可用），再去让新的请求访问该服务已经没有任何意义，只会无谓消耗资源。例如，设置了超时时间为1s，如果短时间内有大量的请求无法在1s内得到响应，就没有必要再去请求依赖的服务了。

  ​        断路器可理解为对容易导致错误的操作的代理。这种代理能够统计一段时间内调用失败的次数，并决定是正常请求依赖的服务还是直接返回。

  ​        断路器可以实现快速失败，如果它在一段时间内检测到许多类似的错误（例如超时），就会在之后的一段时间内，强迫对该服务的调用快速失败，即不再请求所依赖的服务。这样，应用程序就无须再浪费CPU时间去等待长时间的超时。

  ​        断路器也可自动诊断依赖的服务是否已经恢复正常。如果发现依赖的服务已经恢复正常，那么就会恢复请求该服务。使用这种方式，就可以实现微服务的“自我修复”——当依赖的服务不正常时，打开断路器时快速失败，从而防止雪崩效应；当发现依赖的服务恢复正常时，又会恢复请求。

  

   断路器状态转换的逻辑如下所示：

   

  ![](/image/SpringCloud/image-20191214100356789.png)

  - 正常情况下，断路器关闭，可正常请求依赖的服务。
  - 当一段时间内，请求失败率达到一定阈值（例如错误率达到50%，或100次/分钟等），断路器就会打开。此时，不会再去请求依赖的服务。
  - 断路器打开一段时间后，会自动进入“半开”状态。此时，断路器可允许一个请求访问依赖的服务。如果该请求能够调用成功，则关闭断路器；否则继续保持打开状态。

  综上，我们可通过以上两点机制保护应用，从而防止雪崩效应并提升应用的可用性。

### 2. 使用`Hystrix`实现容错

#### 2.1  `Hystrix`简介

​         `Hystrix`是由`Netflix`开源的一个延迟和容错库，用于隔离访问远程系统、服务或者第三方库，防止级联失败，从而提升系统的可用性与容错性。`Hystrix`主要通过以下几点实现延迟和容错：

- 包裹请求：使用`HystrixCommand`（或`HystrixObservableCommand`）包裹对依赖的调用逻辑，每个命令在独立线程中执行。这使用了设计模式中的“命令模式”。
- 跳闸机制：当某服务的错误率超过一定阈值时，`Hystrix`可以自动或者手动跳闸，停止请求该服务一段时间。
- 资源隔离：`Hystrix`为每个依赖都维护了一个小型的线程池（或者信号量）。如果该线程池已满，发往该依赖的请求就被立即拒绝，而不是排队等候，从而加速失败判定。
- 自我修复：断路器打开一段时间后，会自动进入“半开”状态。断路器打开、关闭、半开的逻辑转换。

具体可以参考如下：

`Hystrix的GitHub`：https://github.com/Netflix/Hystrix

命令模式：https://en.wikipedia.org/wiki/Command_pattern

#### 2.2  通用方式整合`Hystrix`

在Spring Cloud中，整合`Hystrix`非常方便,下面我们就来整合`Hystrix`。

1.新建项目`microservice-consumer-movie-ribbon-hystrix`将项目`microservice-consumer-movie-ribbon`中的代码复制过来。

2.修改`pom.xml`,完整`pom.xml`如下所示：

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
    <artifactId>microservice-consumer-movie-ribbon-hystrix</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-ribbon-hystrix</name>
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
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
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

3.在启动类上添加注解`@EnableCircuitBreaker`或`@EnableHystrix`，从而为项目启用断路器支持。

4.修改`MovieController`，让其中的`findById`方法具备容错能力。

```java
@RestController
public class MovieController {
    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "findByIdFallback")
    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id) {
        return restTemplate.getForObject("http://MICROSERVICE-PROVIDER-USER-RIBBON/user/" + id, User.class);
    }


    public User findByIdFallback(Long id) {
        User user = new User();
        user.setId(-1L);
        user.setName("default");
        user.setDataBase("default");
        
        return user;
    }

}
```

​       由代码可知，为`findById`方法编写了一个回退方法`findByIdFallback`，该方法与`findById`方法具有相同的参数与返回值类型，该方法返回了一个默认的User。

​     在`findById`方法上，使用注解`@HystrixCommand`的`fallbackMethod`属性，指定回退方法是`findByIdFallback`。注解`@HystrixCommand`由名为`javanica`（*https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica*）的`Hystrix contrib`库提供。`javanica`是一个`Hystrix`的子项目，用于简化`Hystrix`的使用。Spring Cloud自动将`Springbean`与该注解封装在一个连接到`Hystrix`断路器的代理中。

​      `@HystrixCommand`的配置非常灵活，可使用注解`@HystrixProperty`的`commandProperties`属性来配置`@HystrixCommand`。具体如下：



```java
    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500")
    },
            threadPoolProperties = {
                    @HystrixProperty(name = "coreSize", value = "30"),
                    @HystrixProperty(name = "maxQueueSize", value = "101"),
                    @HystrixProperty(name = "keepAliveTimeMinutes", value = "2"),
                    @HystrixProperty(name = "queueSizeRejectionThreshold", value = "15"),
                    @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "12"),
                    @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "1440")
            },fallbackMethod = "findByIdFallback")
    @GetMapping("user0/{id")
    public User getUserById(@PathVariable("id") Long id) {
        return restTemplate.getForObject("http://MICROSERVICE-PROVIDER-USER-RIBBON/user/" + id, User.class);
    }
```



**测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-ribbon-hystrix`。

4.访问*http://localhost:9084/user/1*，可获得如下结果。

![image-20191224222738107](/image/SpringCloud/image-20191214195055742.png)

5.停止`microservice-provider-user-ribbon`。

6.再次访问*http://localhost:9084/user/1*，获得如下结果。

![image-20191224222958474](/image/SpringCloud/image-20191214195202503.png)

说明当用户微服务不可用时，进入了回退方法。

​      我们知道，当请求失败、被拒绝、超时或者断路器打开时，都会进入回退方法。但进入回退方法并不意味着断路器已经被打开。那么，如何才能明确了解断路器当前的状态呢？•多数场景下，当发生业务异常时，我们并不想触发`fallback`。此时要怎么办呢？`Hystrix`有个`HystrixBadRequestException`类，这是一个特殊的异常类，当该异常发生时，不会触发回退。因此，可将自定义的业务异常继承该类，从而达到业务异常不回退的效果。

另外，`@HystrixCommand`为我们提供了`ignoreExceptions`属性，也可借助该属性来配置不想执行回退的异常类。

```java
    @HystrixCommand(fallbackMethod = "findByIdFallback",
            ignoreExceptions = {IllegalArgumentException.class, NullPointerException.class})
    @GetMapping("/user1/{id}")
    public User findById1(@PathVariable("id") Long id) {
        return restTemplate.getForObject("http://MICROSERVICE-PROVIDER-USER-RIBBON/user/" + id, User.class);
    }
```

这样，即使在`findById1`中发生`IllegalArgumentException或`者`NullPointerException`，也不会执行`findBy IdFallback`方法。



#### 2.3 `Hystrix`断路器的状态监控与深入理解

我们需要使用监控就需要引入`Spring Boot  Actuator`

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

​    断路器的状态也会暴露在Actuator提供的/health端点中，这样就可以直观地了解断路器的状态。下面来做一点实验，深入理解断路器的状态转换。

修改`application.yml`文件，修改后的文件如下：

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-ribbon-hystrix
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
management:
  endpoint:
    health:
      show-details: always
```





**测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-ribbon-hystrix`。

4.访问*http://localhost:9081/user/1*，可正常获得结果。

5.访问*http://localhost:9081/actuator/health*，可获得如下结果。

![image-20191214210938853](/image/SpringCloud/image-20191214210938853.png)

可以看到此时，`Hystrix`的状态是UP，也就是一切正常，此时断路器是关闭的。

6.停止`microservice-provider-user-ribbon`。

7.再次访问*http://localhost:9081/user/1*，获得如下结果。

![image-20191214211232708](/image/SpringCloud/image-20191214211232708.png)

8.再次访问*http://localhost:9081/actuator/health*，可获得如下结果。

![image-20191214211405323](/image/SpringCloud/image-20191214211405323.png)

​       我们发现，尽管执行了回退逻辑，返回了默认用户，但此时`Hystrix`的状态依然是UP，这是因为我们的失败率还没有达到阈值（默认是5s内20次失败）。这是很多初学者会遇到的误区，这里再次强调——执行回退逻辑并不代表断路器已经打开。请求失败、超时、被拒绝以及断路器打开时等都会执行回退逻辑。

9.持续快速地访问*http://localhost:9081/user/1*，直到请求快速返回。此时，访问*http://localhost:9081/actuator/health*可获得类似如下的结果：

![image-20191214211835422](/image/SpringCloud/image-20191214211835422.png)

可以看到，`Hystrix`的状态是`CIRCUIT_OPEN`，说明断路器已经打开，不会再去请求用户微服务了。我们也可使用类似的方法测试断路器从打开转半开以及从半开自动恢复等过程.

#### 2.4  `Hystrix`线程隔离策略与传播上下文



这个问题相对比较复杂，为了使大家更加好理解，先来阅读一下`Hystrix`官方`Wiki`（*https://github.com/Netflix/Hystrix/wiki/Configuration#execution.isolation.strategy*）。

> #### `execution.isolation.strategy`
>
> This property indicates which isolation strategy `HystrixCommand.run()` executes with, one of the following two choices:
>
> - `THREAD` — it executes on a separate thread and concurrent requests are limited by the number of threads in the thread-pool
> - `SEMAPHORE` — it executes on the calling thread and concurrent requests are limited by the semaphore count
>
> ##### Thread or Semaphore
>
> The default, and the recommended setting, is to run `HystrixCommand`s using thread isolation (`THREAD`) and `HystrixObservableCommand`s using semaphore isolation (`SEMAPHORE`).
>
> Commands executed in threads have an extra layer of protection against latencies beyond what network timeouts can offer.
>
> Generally the only time you should use semaphore isolation for `HystrixCommand`s is when the call is so high volume (hundreds per second, per instance) that the overhead of separate threads is too high; this typically only applies to non-network calls.
>
> 

![](/image/SpringCloud/image-20191214214456456.png)

> | Defaulter                    | `THREAD` (see `ExecutionIsolationStrategy.THREAD`)           |
> | ---------------------------- | ------------------------------------------------------------ |
> | Possible Values              | `THREAD`, `SEMAPHORE`                                        |
> | Default Property             | `hystrix.command.default.execution.isolation.strategy`       |
> | Instance Property            | `hystrix.command.*HystrixCommandKey*.execution.isolation.strategy` |
> | How to Set Instance Default: | `// to use thread isolation HystrixCommandProperties.Setter()   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.THREAD) // to use semaphore isolation HystrixCommandProperties.Setter()   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)` |

简单翻译一下，`Hystrix`的隔离策略有两种：分别是线程隔离和信号量隔离。

•THREAD（线程隔离）：使用该方式，`HystrixCommand`将在单独的线程上执行，并发请求受到线程池中的线程数量的限制。

•SEMAPHORE（信号量隔离）：使用该方式，`HystrixCommand`将在调用线程上执行，开销相对较小，并发请求受到信号量个数的限制。

`Hystrix`中默认并且推荐使用线程隔离（THREAD），因为这种方式有一个除网络超时以外的额外保护层。

一般来说，只有当调用负载非常高时（例如每个实例每秒调用数百次）才需要使用信号量隔离，因为在这种场景下使用THREAD开销会比较高。信号量隔离一般仅适用于非网络调用的隔离。

可使用`execution.isolation.strategy`属性指定隔离策略。

了解`Hystrix`的隔离策略后，再来看一下Spring Cloud官方的文档。

> If you want some thread local context to propagate into a `@HystrixCommand`, the default declaration does not work, because it executes the command in a thread pool (in case of timeouts). You can switch `Hystrix` to use the same thread as the caller through configuration or directly in the annotation, by asking it to use a different “Isolation Strategy”. The following example demonstrates setting the thread in the annotation:
>
> ```java
> @HystrixCommand(fallbackMethod = "stubMyService",
>     commandProperties = {
>       @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
>     }
> )
> ...
> ```
>
> The same thing applies if you are using `@SessionScope` or `@RequestScope`. If you encounter a runtime exception that says it cannot find the scoped context, you need to use the same thread.
>
> You also have the option to set the `hystrix.shareSecurityContext` property to `true`. Doing so auto-configures a `Hystrix` concurrency strategy `plugin` hook to transfer the `SecurityContext` from your main thread to the one used by the `Hystrix` command. `Hystrix` does not let multiple `Hystrix` concurrency strategy be registered so an extension mechanism is available by declaring your own `HystrixConcurrencyStrategy` as a Spring bean. Spring Cloud looks for your implementation within the Spring context and wrap it inside its own `plugin`.

简单翻译一下,大概意思如下：

如果你想传播线程本地的上下文到`@HystrixCommand`，默认声明将不会工作，因为它会在线程池中执行命令（在超时的情况下）。你可以使用一些配置，让`Hystrix`使用相同的线程，或者直接在注解中让`Hystrix`使用不同的隔离策略。例如：

```java
@HystrixCommand(fallbackMethod = "stubMyService",
    commandProperties = {
      @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
    }
)
...
```

这也适用于使用`@SessionScope`或者`@RequestSession`的情况。你会知道什么时候需要这样做，因为会发生一个运行时异常，说它找不到作用域上下文（scoped context）。

你还可将`hystrix.shareSecurityContext`属性设置为true，这样将自动配置一个`Hystrix`并发策略插件的hook，这个hook会将`SecurityContext`从主线程传输到`Hystrix`的命令。因为`Hystrix`不允许注册多个`Hystrix`并发策略，所以可以声明`HystrixConcurrencyStrategy`为一个Spring bean来实现扩展。Spring Cloud会在Spring的上下文中查找你的实现，并将其包装在自己的插件中。



将`SpringCloud`和`Hystrix`的文档对照阅读，就能很好地理解相关概念。在此，有如下总结：

- `Hystrix`的隔离策略有THREAD和SEMAPHORE两种，默认是THREAD。
- 正常情况下，保持默认即可。
- 如果发生找不到上下文的运行时异常，可考虑将隔离策略设置为SEMAPHORE。



#### 2.5  Feign使用`Hystrix`

前文是使用注解`@HystrixCommand`的`fallbackMethod`属性实现回退的。然而，Feign是以接口形式工作的，它没有方法体，前文讲解的方式显然不适用于Feign。

那么Feign要如何整合`Hystrix`呢？不仅如此，如何实现Feign的回退呢？

在Spring Cloud中，为Feign添加回退更加简单。事实上，Spring Cloud默认已为Feign整合了`Hystrix`，要想为Feign打开`Hystrix`支持，只需设置`feign.hystrix.enabled=true`。下面来详细探讨如何实现Feign的回退。

##### 2.5.1 为Feign添加回退

首先为前文编写的`UserFeignClient`添加回退。

1.新建`microservice-consumer-movie-feign-hystrix-fallback`项目，将`microservice-consumer-movie-feign`中的代码复制过来。

2.在`application.yml`中添加`feign.hystrix.enabled:true`，从而开启Feign的`Hystrix`支持，完整内容如下。

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-feign-hystrix-fallback
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
feign:
  hystrix:
    enabled: true
```

3.将之前编写的Feign接口修改成如下内容：

```java
@FeignClient(name = "microservice-provider-user-ribbon",fallback = FeignClientFallback.class)
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}

```



`FeignClientFallback`内容如下：

```java
@Component
public class FeignClientFallback implements UserFeignClient {
    @Override
    public User findById(Long id) {
        User user = new User();
        user.setId(-1L);
        user.setName("default");
        user.setDataBase("default");

        return user;
    }
}
```



由代码可知，只需使用`@FeignClient`注解的`fallback`属性，就可为指定名称的Feign客户端添加回退。

**测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-feign-hystrix-fallback`。

4.访问*http://localhost:9081/user/1*，可正常获得结果。

5.停止`microservice-provider-user-ribbon`。

6.再次访问*http://localhost:9081/user/1*，获得如下结果,说明当用户微服务不可

用时，进入了回退的逻辑。

![](/image/SpringCloud/image-20191214224632490.png)

在`SpringCloud Dalston`之前的版本中，Feign默认已开启`Hystrix`支持，无须设置`feign.hystrix.enabled=true`；从`SpringCloud Dalston`开始，Feign的`Hystrix`支持默认关闭，必须设置该属性。

##### 2.5.2 通过`Fallback Factory`检查回退原因

很多场景下，我们需要了解回退的原因，便于进行故障排查等。

前面讲过，当使用`@HystrixCommand`时，如需获得造成回退的原因，只需在回退方法上添加一个`Throwable`参数，例如：

```java
    @HystrixCommand(fallbackMethod = "findByIdFallback")
    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id) {
        return restTemplate.getForObject("http://MICROSERVICE-PROVIDER-USER-RIBBON/user/" + id, User.class);
    }


    public User findByIdFallback(Long id,Throwable throwable) {
        LOGGER.error("进入回退方法,异常:",throwable);
        User user = new User();
        user.setId(-1L);
        user.setName("default");
        user.setDataBase("default");
        return user;
    }
```

那么对于Feign，如何获得回退原因呢？可使用注解`@FeignClient`的`fallbackFactory`属性。

下面来编写一个示例，当回退发生时，打印日志。

1.新建项目`microservice-consumer-movie-feign-hystrix-fallback-factory`，复制`microservice-consumer-movie-feign`中的代码

2.在`application.yml`中添加`feign.hystrix.enabled:true`，从而开启`Feign`的`Hystrix`支持，完整配置文件如下：

```yaml
server:
  port: 9081
spring:
  application:
    name: microservice-consumer-movie-feign-hystrix-fallback-factory
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
feign:
  hystrix:
    enabled: true

```

3.将`UserFeignClient`改为如下内容。

```java
@FeignClient(name = "microservice-provider-user-ribbon",
        fallbackFactory = UserFeignClientFallbackFactory.class)
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}

```

`UserFeignClientFallbackFactory`的内容如下：

```java
@Component
public class UserFeignClientFallbackFactory implements FallbackFactory<UserFeignClient> {
    private static final Logger LOGGER = LoggerFactory.getLogger(UserFeignClientFallbackFactory.class);

    @Override
    public UserFeignClient create(Throwable throwable) {
        return new UserFeignClient() {
            @Override
            public User findById(Long id) {
            UserFeignClientFallbackFactory.LOGGER.info("fallback  reason was:",throwable);
                User user = new User();
                user.setId(-1L);
                user.setName("default");
                user.setDataBase("default");
                return user;
            }
        };
    }
}
```

这样，当Feign发生回退时，就会打印日志。

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-feign-hystrix-fallback-factory`。

4.访问*http://localhost:9081/user/1*，可正常获得结果。

5.停止`microservice-provider-user-ribbon`。

6.再次访问*http://localhost:9081/user/1*，获得如下结果,说明当用户微服务不可

用时，进入了回退的逻辑。

![](/image/SpringCloud/image-20191215191950770.png)

出现上面这种情况是在注册中心中存在该服务实例，但是无法访问；

![](/image/SpringCloud/image-20191215192128895.png)

出现上面这种情况是在注册中心中不存在该服务实例，无法访问；



`fallbackFactory`属性还有很多其他的用途，例如让不同的异常返回不同的回退结果，从而使Feign的回退更加灵活。例如：

```java
@Component
public class UserFeignClientFallbackFactory implements FallbackFactory<UserFeignClient> {
    private static final Logger LOGGER = LoggerFactory.getLogger(UserFeignClientFallbackFactory.class);

    @Override
    public UserFeignClient create(Throwable throwable) {

      return  new UserFeignClient() {
          @Override
          public User findById(Long id) {
              User user = new User();

              user.setName("default");
              user.setDataBase("default");

              if (throwable instanceof HystrixTimeoutException){
                  user.setId(-1L);
              }else {
                  user.setId(-2L);
              }
              return user;
          }
      };

    }
}
```



##### 2.5.3  为Feign禁用`Hystrix`

​      前文说过，在Spring Cloud中，只要`Hystrix`在项目的`classpath`中，Feign就会使用断路器包裹Feign客户端的所有方法。这样虽然方便，但很多场景并不需要该功能，那要如何为Feign禁用`Hystrix`呢？方法如下所示:

- 为指定Feign客户端禁用`Hystrix`：借助Feign的自定义配置，可轻松为指定名称的Feign客户端禁用`Hystrix`

```java
@Configuration
public class FeignDisableHystrixConfig {
    @Bean
    @Scope("prototype")
    public Feign.Builder feignBuilder() {

        return Feign.builder();
    }
}
```

想要禁用`Hystrix`的`@FeignClient`，引用该配置类即可

```java
@FeignClient(name ="microservice-provider-user-ribbon",configuration = FeignDisableHystrixConfig.class)
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

- 全局禁用`Hystrix`：也可为Feign全局禁用`Hystrix`。只需在`application.yml`中配置`feign.hystrix.enabled=false`即可。

### 3. `Hystrix`的监控

#### 3.1  普通的`Hystrix`项目

​         除实现容错外，`Hystrix`还提供了近乎实时的监控。`HystrixCommand`和`HystrixObservableCommand`在执行时，会生成执行结果和运行指标，比如每秒执行的请求数、成功数等，这些监控数据对分析应用系统的状态很有用。

​       使用`Hystrix`的模块`hystrix-metrics-event-stream`，就可将这些监控的指标信息以`text/event-stream`的格式暴露给外部系统。`spring-cloud-starter-netflix-hystrix`已包含该模块，在此基础上，只需为项目添加`spring-boot-starter-actuator`并且在配置文件中配置如下信息：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

就可使用`/actuator/hystrix.stream`端点获得`Hystrix`的监控信息了。

​    如上所述，前文的项目`microservice-consumer-movie-ribbon-hystrix`已具备监控`Hystrix`的能力。下面来做一点测试。

**测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-ribbon-hystrix`。

4.访问*http://localhost:9081/actuator/hystrix.stream*，可获得如下结果。

![image-20191217212143666](/image/SpringCloud/image-20191217212143666.png)

可看到浏览器一直处于请求的状态，页面空白。这是因为此时项目中注解了`@HystrixCommand`的方法还没有被执行，因此也没有任何的监控数据。

5.访问*http://localhost:9081/user/1*后，再次访问*http://localhost:9081/actuator/hystrix.stream*，可看到页面会重复出现类似如下的内容。

![image-20191217212349050](/image/SpringCloud/image-20191217212349050.png)

​     这是因为系统会不断地刷新以获得实时的监控数据。`Hystrix`的监控指标非常全面，例如`HystrixCommand`的名称、group名称、断路器状态、错误率、错误数等。

#### 3.2  `Feign`项目的`Hystrix`监控

​    在`microservice-consumer-movie-feign-hystrix-fallback`项目的配置文件中添加同样的配置

```yaml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

并使用类似的方式测试，然后访问*http://localhost:9081/actuator/hystrix.stream*，发现返回的是404。这是为什么呢？

![image-20191217213107895](/image/SpringCloud/image-20191217213107895.png)

查看项目的依赖树会发现，项目甚至连`hystrix-metrics-event-stream`的依赖都没有。那么如何解决该问题呢？

解决方案如下：

1.新建项目`microservice-consumer-movie-feign-hystrix-fallback-stream`,将`microservice-consumer-movie-feign-hystrix-fallback`中的代码复制过来。

2.为项目添加`spring-cloud-starter-netflix-hystrix`的依赖。

```xml
       <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

3.在启动类上添加`@EnableCircuitBreaker`或者`@EnableHystrix`，这样就可使用`/actuator/hystrix.stream`端点监控`Hystrix`了。



### 4. 使用`Hystrix Dashboard`可视化监控数据

​      前面讨论了`Hystrix`的监控，但访问`/actuator/hystrix.stream`端点获得的数据是以文字形式展示的。很难通过这些数据，一眼看出系统当前的运行状态。

​     可使用`Hystrix Dashboard`，让监控数据图形化、可视化。

1.新建一个项目`microservice-consumer-movie-feign-hystrix-dashboard`，复制项目`microservice-consumer-movie-feign-hystrix-fallback-stream`中的代码，并为项目添加一下依赖,完整依赖如下：

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
    <artifactId>microservice-consumer-movie-feign-hystrix-dashboard</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-feign-hystrix-dashboard</name>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
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

2.在启动类上添加`@EnableHystrixDashboard`注解；

3.在配置文件`application.yml`中添加如下内容

```yaml
server:
  port: 9083
spring:
  application:
    name: microservice-consumer-movie-feign-hystrix-dashboard
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
feign:
  hystrix:
    enabled: true
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

**测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-feign-hystrix-dashboard`

3.访问*http://localhost:9083/hystrix*，可获得如下结果。

![image-20191222221121959](/image/SpringCloud/image-20191217221004570.png)

4.在上图红色框内输入http://localhost:9083/actuator/hystrix.stream*，随意设置一个Title，单击Monitor Stream按钮后，可获得如下结果。

![image-20191219232224459](/image/SpringCloud/image-20191219232203662.png)

这是为什么呢？网络上的很多方式是无法解决这个问题的。

5.访问*http://localhost:9083/actuator/hystrix.stream*，可获得如下结果。

![image-20191222221345817](/image/SpringCloud/image-20191217212143666.png)

之前已经说过，这是因为此时项目中注解了`@HystrixCommand`的方法还没有被执行，因此也没有任何的监控数据。

6.访问*http://localhost:9083/user/1*后，再次访问*http://localhost:9083/actuator/hystrix.stream*，可看到页面会重复出现类似如下的内容。

![image-20191222221513012](/image/SpringCloud/image-20191217212349050.png)

7.再次刷新监控页面，得到如下结果，监控能够正常显示：

![image-20191219232741445](/image/SpringCloud/image-20191219232741445.png)

​       读者可尝试将隔离策略设为SEMAPHORE，此时图中的Thread Pool一栏将不再显示。这是由于THREAD、SEMAPHORE的实现机制不同所导致的。

那么此图中的各个指标是啥含义呢？请看下图：



![](/image/SpringCloud/image-20191219232945236.png)



### 5. 使用Turbine聚合监控数据

​      前文中使用`/actuator/hystrix.stream`端点监控单个微服务实例。然而，使用微服务架构的应用系统一般会包含若干个微服务，每个微服务通常都会部署多个实例。如果每次只能查看单个实例的监控数据，就必须在`Hystrix Dashboard`上切换想要监控的地址，这显然很不方便。那要如何解决该问题呢？请带着这个疑问读下去。

#### 5.1 Turbine简介

​       Turbine是一个聚合`Hystrix`监控数据的工具，它可将所有相关`/hystrix.stream`端点的数据聚合到一个组合的`/turbine.stream`中，从而让集群的监控更加方便。

引入Turbine后，架构如下图所示：

![](/image/SpringCloud/image-20191222214956123.png)



Turbine的GitHub：*https://github.com/Netflix/Turbine*.

#### 5.2  使用Turbine监控多个微服务

1.为了让Turbine监控2个以上的微服务，在`microservice-consumer-movie-feign-hystrix-fallback`中新增一个配置文件`application-9082.yml`具体内容如下：

```yaml
server:
  port: 9082
spring:
  application:
    name: microservice-consumer-movie-feign-hystrix-fallback
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

feign:
  hystrix:
    enabled: true
```



2.新建项目`microservice-consumer-movie-feign-hystrix-turbine`,添加完整依赖如下：

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
    <artifactId>microservice-consumer-movie-feign-hystrix-turbine</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-feign-hystrix-turbine</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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

2.为启动类加上注解`@EnableTurbine`

3.修改配置文件，完整配置文件如下:

```yaml
server:
  port: 9082
spring:
  application:
    name: microservice-consumer-movie-feign-hystrix-turbine
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
turbine:
  app-config: microservice-consumer-movie-feign-hystrix-fallback-stream,microservice-consumer-movie-feign-hystrix-dashboard,microservice-consumer-movie-ribbon-hystrix
  cluster-name-expression: "'default'"
```

​      使用以上配置，Turbine会在`EurekaServer`中找到`microservice-consumer-movie-feign-hystrix-fallback-stream`和`microservice-consumer-movie-ribbon`这两个微服务，并聚合两个微服务的监控数据。



**测试**



1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-feign-hystrix-fallback-stream`。

4.启动项目`microservice-consumer-movie-ribbon-hystrix`。

5.启动项目`microservice-consumer-movie-feign-hystrix-dashboard`。

6.启动项目`microservice-consumer-movie-feign-hystrix-turbine`。

7.访问`http://localhost:9081/user/1`，让`microservice-consumer-movie-feign-hystrix-fallback-stream`微服务产生监控数据。

8.访问`http://localhost:9083/user/1`，让`microservice-consumer-movie-feign-hystrix-dashboard`微服务产生监控数据。

9.访问`http://localhost:9084/user/1`，让`microservice-consumer-movie-ribbon-hystrix`微服务产生监控数据。

10.访问`http://localhost:9083/hystrix`，在URL一栏填入`http://localhost:9082/turbine.stream`，随意指定一个Title并单击Monitor Stream按钮后，显示如下结果：

![image-20191224223833675](/image/SpringCloud/image-20191224223833675.png)

配置文件中添加了三个微服务，为何只显示2个呢？因为有2个服务监控的方法一模一样，因此被合并为一个，在Hosts参数上变为2。

#### 5.3 使用消息中间件收集数据

​         一些场景下，前文的方式无法正常工作（例如微服务与Turbine网络不通），此时，可借助消息中间件实现数据收集。各个微服务将`Hystrix Command`的监控数据发送至消息中间件，Turbine消费消息中间件中的数据。

此时，架构图如图所示。

![](/image/SpringCloud/image-20191224225156789.png)

下面笔者以`RabbitMQ`为例进行演示。至于安装过程之类的自己网上搜一下，笔者是使用docker进行安装`RabbitMQ`环境的搭建。

##### 5.3.1  验证`RabbitMQ`

搭建完成后访问`http://192.168.10.21:15672/`，这个是笔者虚拟机的`IP`，各位读者测试时请使用自己的`RabbitMQ`所在的机子`IP`。`RabbitMQ`首页如下：

![image-20191224230046142](/image/SpringCloud/image-20191224230046142.png)

这样就可使用图形化的界面管理`RabbitMQ`了。

##### 5.3.2 改造微服务

要想使用消息中间件收集监控数据，需要改造前文编写的微服务，以`microservice-consumermovie-ribbon-hystrix`为例。

1.新建`microservice-consumer-movie-ribbon-hystrix-turbine-mq`项目，复制项目`microservice-consumer-movie-ribbon-hystrix`中代码。

2.添加`RabbitMQ`依赖，添加完后，完整的`pom.xml`如下：

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
    <artifactId>microservice-consumer-movie-ribbon-hystrix</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-ribbon-hystrix</name>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
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

3.在配置文件`application.yml`中添加`RabbitMQ`配置，添加完后完整内容如下：

```yaml
server:
  port: 9085
spring:
  application:
    name: microservice-consumer-movie-ribbon-hystrix
  rabbitmq:
    host: 192.168.10.21
    port: 5672
    username: guest
    password: guest
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

这样，微服务就改造完成了。

##### 5.3.3  新建Turbine

改造完微服务后，接下来改造Turbine。

1.新建`microservice-consumer-movie-feign-hystrix-turbine-mq`。

2.添加`RabbitMQ`依赖，添加完后完整的`pom.xml`如下：

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
    <artifactId>microservice-consumer-movie-feign-hystrix-turbine</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-consumer-movie-feign-hystrix-turbine</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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

3.在启动类上添加注解`@EnableTurbineStream`

4.在配置文件`application.yml`中添加如下内容：

```yaml
server:
  port: 9082
spring:
  application:
    name: microservice-consumer-movie-feign-hystrix-turbine
  rabbitmq:
    host: 192.168.10.21
    port: 5672
    username: guest
    password: guest
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

这样，Turbine就新建完成了。





**测试**

1.启动项目`microservice-discovery-eureka`。

2.启动项目`microservice-provider-user-ribbon`。

3.启动项目`microservice-consumer-movie-ribbon-hystrix-turbine-mq`

4.启动项目`microservice-consumer-movie-feign-hystrix-turbine-mq`。

7.访问`http://localhost:9085/user/1`，可正常获得结果。

8.访问`http://localhost:9082`，会发现Turbine能够持续不断地显示监控数据。



​         务必注意，依赖`spring-cloud-starter-netflix-turbine`不能与`spring-cloud-starter-netflix-turbine-stream`共存，否则应用将无法启动！不仅如此，这两个依赖所使用的Turbine版本也不相同——`spring-cloud-starter-netflix-turbine`使用的Turbine版本是1.0.0，而`spring-cloud-starter-netflix-turbine-stream`使用的Turbine版本是`2.0.0-DP.2`。

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