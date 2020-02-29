---
title: Spring Cloud-02 微服务注册与发现Eureka
date: 2019-11-17 11:29:00 
tags: 
  - Spring Cloud
  - Eureka
---



### 0.说在前面

```java
- JDK：Spring Cloud官方建议使用JDK 1.8;
- Spring Boot：笔者使用Spring Boot 2.1.10.RELEASE;
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；
```

### 1.服务发现简介

​         通过上一篇博客的讲解，我们知道硬编码提供者地址的方式有不少问题。要想解决这些问题，服务消费者需要一个强大的服务发现机制，服务消费者使用这种机制获取服务提供者的网络信息。不仅如此，即使服务提供者的信息发生变化，服务消费者也无须修改配置文件。 

​        服务发现组件提供这种能力。在微服务架构中，服务发现组件是一个非常关键的组件。

<!-- more -->

​       使用服务发现组件后的架构图，如下所示：

  ![](/image/SpringCloud/image-20191117114531311.png)



服务提供者、服务消费者、服务发现组件这三者之间的关系大致如下：

- 各个微服务在启动时，将自己的网络地址等信息注册到服务发现组件中，服务发现组件会存储这些信息。

- 服务消费者可从服务发现组件查询服务提供者的网络地址，并使用该地址调用服务提供者的接口。

- 各个微服务与服务发现组件使用一定机制（例如心跳）通信。服务发现组件若长时间无法与某微服务实例通信，就会注销该实例。

- 微服务网络地址发生变更（例如实例增减或者`IP`端口发生变化等）时，会重新注册到服务发现组件。使用这种方式，服务消费者就无须人工修改提供者的网络地址了。

  综上，服务发现组件应具备以下功能：

  - 服务注册表：是服务发现组件的核心，它用来记录各个微服务的信息，例如微服务的名称、`IP`、端口等。服务注册表提供查询`API`和管理`API`，查询`API`用于查询可用的微服务实例，管理`API`用于服务的注册和注销。

  - 服务注册与服务发现：服务注册是指微服务在启动时，将自己的信息注册到服务发现组件上的过程。服务发现是指查询可用微服务列表及其网络地址的机制。

  - 服务检查：服务发现组件使用一定机制定时检测已注册的服务，如发现某实例长时间无法访问，就会从服务注册表中移除该实例。

  综上，使用服务发现的好处是显而易见的。Spring Cloud提供了多种服务发现组件的支持，例如Eureka、Consul和`ZooKeeper`等。笔者将以Eureka为例，为大家详细讲解服务注册与发现。



### 2.`Eureka`简介

​        Eureka是`Netflix`开源的服务发现组件，本身是一个基于REST的服务。它包含Server和Client两部分。Spring Cloud将它集成在子项目`Spring Cloud Netflix`中，从而实现微服务的注册与发现。

- `Eureka的GitHub`：

  ```
  https://github.com/Netflix/Eureka
  ```

- `Netflix`是一家在线影片租赁提供商。

### 3.`Eureka`原理

在分析Eureka的原理之前，先来了解一下Region和Availability Zone，如下图所示

![](/image/SpringCloud/image-20191117114312345.png)

​       Region和Availability Zone均是`AWS`的概念。其中，Region表示`AWS`中的地理位置，每个Region都有多个Availability Zone，各个Region之间完全隔离。`AWS`通过这种方式实现了最大的容错和稳定性。

​       `SpringCloud`默认使用的Region是us-east-1，在非`AWS`环境下，可以将Availability Zone理解成机房，将Region理解为跨机房的Eureka集群。

   理解Region和Availability Zone后，来分析一下Eureka的原理，Eureka架构如图下图所示：

![](/image/SpringCloud/image-20191117161543123.png)

​     上图来自Eureka官方的架构图，该图比较详细地描述了Eureka集群的工作原理。图中的组件非常多，概念也比较抽象，笔者先来用通俗易懂的文字翻译一下：

-  Application Service相当于服务提供者。

- Application Client相当于服务消费者。

-  Make Remote Call可以理解成调用`RESTful API`的行为。

-  `us-east-1c、us-east-1d`等都是zone，它们都属于us-east-1这个region。



Eureka包含两个组件：`EurekaServer`和`EurekaClient`，它们的作用如下：

- `EurekaServer`提供服务发现的能力，各个微服务启动时，会向`EurekaServer`注册自己的信息（例如`IP`、端口、微服务名称等），Eureka Server会存储这些信息。

- `EurekaClient`是一个Java客户端，用于简化与`EurekaServer`的交互。

- 微服务启动后，会周期性（默认30s）地向`EurekaServer`发送心跳以续约自己的“租期”。

- 如果Eureka Server在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将注销该实例（默认90s）。

- 默认情况下，Eureka Server同时也是`EurekaClient`。多个Eureka Server实例互相之间通过复制的方式来实现服务注册表中数据的同步。

- `EurekaClient`会缓存服务注册表中的信息。这种方式有一定的优势——首先，微服务无须每次请求都查询Eureka Server，从而降低了Eureka Server的压力；其次，即使Eureka Server所有节点都宕掉，服务消费者依然可以使用缓存中的信息找到服务提供者并完成调用。

  综上，Eureka通过心跳检查、客户端缓存等机制，提高了系统的灵活性、可伸缩性和可用性。

### 4.编写Eureka Server 

1.使用 `Spring Initializr`快速创建`microservice-discovery-eureka`微服务，添加Eureka Server模块`pom.xml`如下：

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
    <artifactId>microservice-discovery-eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-discovery-eureka</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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



2.在启动类上添加`@EnableEurekaServer`注解，声明这是一个`EurekaServer`。

```java
@SpringBootApplication
@EnableEurekaServer
public class MicroserviceDiscoveryEurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(MicroserviceDiscoveryEurekaApplication.class, args);
    }

}

```

3.在配置文件`application.yml`中添加以下内容：

```yaml
server:
  port: 8761
  
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka/
      
```



下面简要讲解一下`application.yml`中的配置属性。

 

- `eureka.client.register-with-eureka`：表示是否将自己注册到`EurekaServer`，默认为true。由于当前应用就是Eureka Server，故而设为false。

- `eureka.client.fetch-registry`：表示是否从Eureka Server获取注册信息，默认为true。因为这是一个单点的Eureka Server，不需要同步其他的Eureka Server节点的数据，故而设为false。

- `eureka.client.serviceUrl.defaultZone`：设置与`EurekaServer`交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka；多个地址间可使用,分隔。

 这样一个Eureka Server就编写完成了。

4.测试

   启动Eureka Server，访问

```
http://localhost:8761
```

![](/image/SpringCloud/image-20191117164630623.png)

  由图可知，Eureka Server的首页展示了很多信息，例如当前实例的系统状态、注册到Eureka Server上的服务实例、常用信息、实例信息等。显然，当前还没有任何微服务实例被注册到Eureka Server上。



### 5.将微服务注册到Eureka Server上

本节将之前编写的用户微服务注册到Eureka Server上。

1.使用 `Spring Initializr`快速创建`microservice-provider-user`微服务。

2.`pom.xml`如下：

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
    <artifactId>microservice-provider-user</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-provider-user</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
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

3.将`microservice-simple-provider-user`中的代码复制过来；

4.在配置文件`application.yml`中添加以下配置。

  

```yaml
spring:
  application:
    name: microservice-provider-user
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true  
```

​       其中，`spring.application.name`用于指定注册到Eureka Server上的应用名称；`eureka.instance.prefer-ip-address=true`表示将自己的`IP`注册到Eureka Server。若不配置该属性或将其设置为false，则表示注册微服务所在操作系统的`hostname`到Eureka Server。

​        这样即可将用户微服务注册到`EurekaServer`上。同理，将电影微服务也注册到Eureka Server上，配置电影微服务的`spring.application.name`为`microservice-consumer-movie`。

在配置文件`application.yml`中添加以下配置：

```yaml
server:
  port: 8081
spring:
  application:
    name: microservice-consumer-movie
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

**测试**

1.启动`microservice-discovery-eureka`。

2.启动`microservice-provider-user`。

3.启动`microservice-consumer-movie`。

4.访问

```
http://localhost:8761/
```

结果如下图所示：![](/image/SpringCloud/image-20191117204851752.png)

由图可知，此时用户微服务、电影微服务已经被注册到Eureka Server上了。

- 在`Spring Cloud Edgware`之前，要想将微服务注册到Eureka Server或其他服务发现组件上，**必须**在启动类上添加`@EnableEurekaClient`或@`EnableDiscoveryClient`。

- 在`SpringCloud Edgware`以及更高版本中，**只需添加相关依赖，即可自动注册**。这是由于在实际项目中，我们可能希望实现“不同环境不同配置”的效果，例如：在开发环境中，不注册到`EurekaServer`上，而是服务提供者、服务消费者直连，便于调测；在生产环境中，我们又希望能够享受服务发现的优势——服务消费者无须知道服务提供者的绝对地址。

- **若不想将服务注册到**`EurekaServer`，只需设置

  ```yaml
  spring:
    cloud:
      service-registry:
        auto-registration:
          enabled: false
  ```

  或`@EnableDiscoveryClient`(auto-Register=false)即可。

### 6.`Eureka Server`的高可用

​          有分布式应用开发经验的读者应该能够看出，前文编写的单节点`EurekaServer`并不适合线上生产环境。Eureka Client会定时连接Eureka Server，获取服务注册表中的信息并缓存在本地。微服务在消费远程`API`时总是使用本地缓存中的数据。因此一般来说，即使Eureka Server发生宕机，也不会影响服务之间的调用。但如果Eureka Server宕机时，某些微服务也出现了不可用的情况，Eureka Client中的缓存若不被更新，就可能会影响微服务的调用，甚至影响整个应用系统的高可用性。因此，在生产环境中，通常会部署一个高可用的Eureka Server集群。

#### 6.1 编写高可用Eureka Server

Eureka Server可以通过运行多个实例并相互注册的方式实现高可用部署，Eureka Server实例会彼此增量地同步信息，从而确保所有节点数据一致。事实上，节点之间相互注册是`EurekaServer`的默认行为，因此在前面的`application.yml`文件中这样配置的：

```yaml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

 在前文的基础上，构建一个双节点Eureka Server集群。

1.使用 `Spring Initializr`快速创建`microservice-discovery-eureka-cluster`微服务，`pom.xm`l如下：

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
    <artifactId>microservice-discovery-eureka-cluster</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-discovery-eureka-cluster</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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

2.配置系统的hosts，Windows系统的hosts文件路径是`C:\Windows\System32\drivers\etc\hosts`；

Linux及`MacOS`等系统的文件路径是/etc/hosts

```
127.0.0.1  eureka01
127.0.0.1  eureka02
127.0.0.1  eureka03
```

3.将`application.yml`修改如下，让两个节点的Eureka Server相互注册。

```yaml
spring:
  application:
    name: microservice-discovery-eureka-cluster
---
spring:
  profiles: eureka01
server:
  port: 8761
eureka:
  instance:
    hostname: eureka01
  client:
    service-url:
      defaultZone: http://eureka02:8762/eureka/,http://eureka03:8763/eureka/
---
spring:
  profiles: eureka02
server:
  port: 8762
eureka:
  instance:
    hostname: eureka02
  client:
    service-url:
      defaultZone: http://eureka01:8761/eureka/,http://eureka03:8763/eureka/
---
spring:
  profiles: eureka03
server:
  port: 8763
eureka:
  instance:
    hostname: eureka03
  client:
    service-url:
      defaultZone: http://eureka01:8761/eureka/,http://eureka02:8762/eureka/
  
```

​       如上，使用连字符（---）将该`application.yml`文件分为三段。第二段和第三段分别为`spring.properties`指定了一个值，该值表示它所在的那段内容应用在哪个Profile里。第一段由于并未指定`spring.profiles`，因此这段内容会对所有Profile生效。

​      经过以上分析，不难理解，我们定义了`eureka01，eureka02，eureka03`三个profile。当我们以`eureka01`这个Profile启动时，会将其注册到其他两个`EurekaServer`上，也就是每个`EurekaServer`会将自己注册到当前`EurekaServe`上，也会注册到其他2个`EurekaServer`上。这样，三个`EurekaServer`上均会有其他2个`EurekaServer`以及自己所在的`EurekaServer`。

**测试**

这里有2种测试方式：

方式一：打包项目，并使用以下命令启动两个Eureka Server节点

```
java -jar microservice-discovery-eureka-cluster-0.0.1.SNAPSHOT.jar --spring.profiles.active=eureka01
java -jar microservice-discovery-eureka-cluster-0.0.1.SNAPSHOT.jar --spring.profiles.active=eureka02
java -jar microservice-discovery-eureka-cluster-0.0.1.SNAPSHOT.jar --spring.profiles.active=eureka03
```

方式二：

直接在`IntelliJIDEA`中指定启动参数：

​     具体的看下图所示：

![](/image/SpringCloud/image-20191117220620900.png)

修改应用名字为：

`MicroserviceDiscoveryEurekaClusterApplication01，`

`MicroserviceDiscoveryEurekaClusterApplication02，`

`MicroserviceDiscoveryEurekaClusterApplication03，`

以及对应的`profiles：eureka01，eureka02，eureka03`。



注意：启动第一，二个的时候会报错，这是因为第一个启动好，想要注册到第二个的时候发现第二个不存在（第二个没启动），遇到错误直接忽略，三个都启动好就没问题了。

访问

```
http://localhost:8761/
或者
http://localhost:8762/
或者
http://localhost:8763/
```

![](/image/SpringCloud/image-20191117221354387.png)

访问8761时，你会发现registered-replicas有 `eureka02， eureka03`；

访问8762时，你会发现registered-replicas有 `eureka01， eureka03`；

访问8763时，你会发现registered-replicas有 `eureka01， eureka02`；

到此`EurekaServer` 集群也就搞定了。

#### 6.2将应用注册到Eureka Server集群上

以`microservice-provider-user`项目为例，只需修改`eureka.client.serviceUrl.defaultZone`，配置多个Eureka Server地址，就可以将其注册到Eureka Server集群了，

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka01:8761/eureka/,http://eureka02:8762/eureka/,http://eureka03:8763/eureka/
```

这样就可以将服务注册到Eureka Server集群上了。

 

当然，微服务即使只配置`EurekaServer`集群中的某个节点，也能正常注册到`EurekaServer`集群，因为多个Eureka Server之间的数据会相互同步，我们以`microservice-consumer-movie`项目为例，

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka01:8761/eureka/
      
```

在正常情况下，这种方式与配置多个Server节点的效果是一样的。不过为适应某些极端场景，建议在客户端配置多个Eureka Server节点。

在正常情况下，这种方式与配置多个Server节点的效果是一样的。不过为适应某些极端场景，笔者建议在客户端配置多个Eureka Server节点。

在正常情况下，这种方式与配置多个Server节点的效果是一样的。不过为适应某些极端场景，笔者建议在客户端配置多个Eureka Server节点。

**测试**

1.启动`microservice-provider-user`和`microservice-consumer-movie`

2.访问

```
http://localhost:8761/
或者
http://localhost:8762/
或者
http://localhost:8763/
```

![](/image/SpringCloud/image-20191117224625490.png)

随意访问一个，2个服务都注册到三个`EurekaServer`上。

### 7.用户认证

​       在前面的示例中，Eureka Server是允许匿名访问的，在实际项目中，可能希望必须经过用户认证才允许访问Eureka Server。下面来详细探讨Eureka的用户认证。

#### 7.1为Eureka Server添加用户认证

 下面就来构建一个需要登录的才能访问的Eureka Server；

1.使用 `Spring Initializr`快速创建`microservice-discovery-eureka-authenticating`微服务，`pom.xml`如下：

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
    <artifactId>microservice-discovery-eureka-authenticating</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-discovery-eureka-authenticating</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
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



2.创建`application.yml`并添加以下内容：

```yaml
server:
  port: 8761

spring:
  security:
    user:
      name: user
      password: 1234

```

若不指定name和 password，name会默认使用user，password默认会自动生成在控制台输出。

```java
Using generated security password: 392dc44e-9e51-409a-b42c-d7ea1a0169b6
```

3.将Eureka Server中的`eureka.client.serviceUrl.defaultZone`修改为http://user:password@EUREKA_HOST:EUREKA_PORT/eureka/的形式

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://user:1234@localhost:8761/eureka/
    fetch-registry: false
    register-with-eureka: false
```

4.测试

   a.启动`microservice-discovery-eureka-authenticating`

   b.访问

```
http://localhost:8761/
```

![](/image/SpringCloud/image-20191118200851644.png)

c.输入账号user、密码1234，即可登录并访问Eureka Server。

#### 7.2将微服务注册到需认证的Eureka Server

​     我们需要在`microservice-discovery-eureka-authenticating`项目中加入以下配置类，才能使其他服务注册到Eureka Server中，不然会有报错出现：

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable(); //关闭csrf
        super.configure(http); //开启认证
    }
}
```

​      如何才能将微服务注册到需认证的Eureka Server上呢？答案非常简单，只需将`eureka.client.serviceUrl.defaultZone`配置为**http://user:password@EUREKA_HOST:EUREKA_PORT/eureka/**的形式，即可将微服务注册到本例的Eureka Server。

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://user:1234@localhost:8761/eureka/
    fetch-registry: false
    register-with-eureka: false
```

### 8.`Eureka`的元数据

​         Eureka的元数据有两种，分别是标准元数据和自定义元数据。标准元数据指的是主机名、`IP`地址、端口号、状态页和健康检查等信息，这些信息都会被发布在服务注册表中，用于服务之间的调用。自定义元数据可以使用`eureka.instance.metadata-map`配置，这些元数据可以在远程客户端中访问，但一般不会改变客户端的行为，除非客户端知道该元数据的含义。

   下面通过一个简单的示例，帮助大家更好地理解Eureka的元数据。

#### 8.1改造用户微服务

   1.使用 `Spring Initializr`快速创建`microservice-provider-user-metadata`，将`microservice-provider-user`项目中的内容复制过来。

  2.修改`application.yml`，使用`eureka.instance.metadata-map`属性为该微服务添加应自定义的元数据，如下：

```yaml
#服务端口
server:
  port: 8080
spring:
  application:
    name: microservice-provider-user-metadata
  datasource:
    username: root
    password: root
    url: jdbc:mysql://192.168.10.21:3306/microservice?useUnicode=true;characterEncoding=utf8;allowMultiQueries=true;autoReconnect=true
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
      defaultZone: http://user:1234@localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    metadata-map:
                 k1: v1
                 k2: v2
```



#### 8.2改造电影微服务

   1.使用 `Spring Initializr`快速创建`microservice-consumer-movie-metadata`，将`microservice-consumer-movie`项目中的内容复制过来。

  2.修改`MovieController`：

```java
@RestController
public class MovieController {
    @Autowired
    RestTemplate restTemplate;

    @Autowired
    DiscoveryClient discoveryClient;

    @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id){
      return   restTemplate.getForObject("http://localhost:8080/user/"+id,User.class);
    }

    @GetMapping("/service/instances")
    public List<ServiceInstance> serviceInstances(){
        return discoveryClient.getInstances("microservice-provider-user-metadata");
    }

}
```

使用`DiscoveryClient.getInstances(serviceId)`，可查询指定微服务在Eureka上的实例列表。

3.修改`application.yml`

```yaml
server:
  port: 8081
spring:
  application:
    name: microservice-consumer-movie-metadata
eureka:
  client:
    service-url:
      defaultZone: http://user:1234@localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

**测试**

1.启动`microservice-discovery-eureka-authenticating`；

2.启动`microservice-provider-user-metadata`；

3.启动`microservice-consumer-movie-metadata`；

4.访问

```
http://localhost:8761/eureka/apps
```

可查看Eureka的`metadata`。

![](/image/SpringCloud/image-20191118221222073.png)

5.访问*http://localhost:8081/service/instances*，可以返回类似如下的内容

![](/image/SpringCloud/image-20191118221044975.png)

2个访问结果相似，一个是`json`格式，一个是`xml`格式。

### 9.`Eureka`的自我保护模式

   下面我们来看一下，Eureka的自我保护模式。进入自我保护模式最直观的体现，是`EurekaServer`首页输出的警告，如下图所示：

![](/image/SpringCloud/image-20191118221822073.png)

默认情况下，如果`EurekaServer`在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将注销该实例（默认为90s）。但是当网络分区故障发生时，微服务与`EurekaServer`之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。

Eureka通过“自我保护模式”来解决这个问题——当Eureka Server节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

在Spring Cloud中，可以使用`eureka.server.enable-self-preservation=false`禁用自我保护模式。



### 10.多网卡环境下的`IP`选择 

​     对于多网卡的服务器，各个微服务注册到Eureka Server上的IP要如何指定呢？

​     指定`IP`在某些场景下很有用。例如某台服务器有`eth0、eth1和eth2`三块网卡，但是只有`eth1`可以被其他的服务器访问；如果`EurekaClient`将`eth0`或者`eth2`注册到`EurekaServer`上，其他微服务就无法通过这个`IP`调用该微服务的接口。

​    Spring Cloud提供了按需选择`IP`的能力，从而避免以上的问题，具体如下：

  1.忽略指定名称的网卡

```yaml
spring:
  cloud:
    inetutils:
      ignored-interfaces:
        - eth0
        - ens.*
eureka:
  instance:
    prefer-ip-address: true
```

这样就可以忽略`eth0`网卡以及所有以ens开头的网卡。

2.使用正则表达式，指定使用的网络地址

```yaml
spring:
  cloud:
    inetutils:
      preferred-networks: 
              - 192.168
              - 10.8
eureka:
  instance:
    prefer-ip-address: true
```

3.只使用站点本地地址

```yaml
spring:
  cloud:
    inetutils:
      use-only-site-local-interfaces: true
eureka:
  instance:
    prefer-ip-address: true
```

4.手动指定`IP`地址。在某些极端场景下，可以手动指定注册到Eureka Server的微服务IP。

 

```yaml
eureka:
  instance:
    prefer-ip-address: true
    ip-address: 127.0.0.1
```



### 11.`Eureka`的健康检查

   先来看一下Eureka首页

![](/image/SpringCloud/image-20191118223645316.png)

​       由图可见，在Status一栏有个UP，表示应用程序状态正常。应用状态还有其他取值，例如DOWN、OUT_OF_SERVICE、UNKNOWN等。只有标记为“UP”的微服务会被请求。

​        前面讲过，`EurekaServer`与`EurekaClient`之间使用心跳机制来确定`EurekaClient`的状态，默认情况下，服务器端与客户端的心跳保持正常，应用程序就会始终保持“UP”状态。

​       以上机制并不能完全反映应用程序的状态。举个例子，微服务与Eureka Server之间的心跳正常，Eureka Server认为该微服务“UP”；然而，该微服务的数据源发生了问题（例如因为网络抖动，连不上数据源），根本无法正常工作。

​       前文说过，`Spring BootActuator`提供了/health端点，该端点可展示应用程序的健康信息。那么如何才能将该端点中的健康状态传播到Eureka Server呢？

​      要实现这一点，只需启用Eureka的健康检查。这样，应用程序就会将自己的健康状态传播到Eureka Server。开启的方法非常简单，只需为微服务配置以下内容，就可以开启健康检查。

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

某些场景下，可能希望更细粒度地控制健康检查，此时可实`com.netflix.appinfo.HealthCheckHandler`接口。

`eureka.client.heal thcheck.enabled=true`只能配置在`application.yml`中，如果配置在`bootstrap.yml`中，可能会导致一些不良的后果，例如应用注册到Eureka Server上的状态是UNKNOWN。

------

今天的内容到这里就结束了，想要一起学习交流以及指正文章中的错误；

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

### 888.参考资料：

[《Spring Cloud 与Docker 微服务架构实战》](https://book.douban.com/subject/30278673/)