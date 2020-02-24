---
title: Spring Cloud-07 使用Spring Cloud Config统一管理微服务配置
date: 2020-12-01 10:20:13 
tags: 
  - Spring Cloud
  - Config
---



### 0.说在前面

```yaml
- JDK：Spring Cloud官方建议使用JDK 1.8;
- Spring Boot：笔者使用Spring Boot 2.1.10.RELEASE;
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；
```



### 1.为什么要统一管理微服务配置

​        对于传统的单体应用，常使用配置文件管理所有配置。例如一个Spring Boot开发的单体应用，可将配置内容放在`application.yml`文件中。如果需要切换环境，可设置多个Profile，并在启动应用时指定`spring.profiles.active={profile}`。

​      然而，在微服务架构中，微服务的配置管理一般有以下需求：

<!--more-->

- 集中管理配置。一个使用微服务架构的应用系统可能会包含成百上千个微服务，因此集中管理配置是非常有必要的。

- 不同环境，不同配置。例如，数据源配置在不同的环境（开发、测试、预发布、生产等）中是不同的。

- 运行期间可动态调整。例如，可根据各个微服务的负载情况，动态调整数据源连接池大小或熔断阈值，并且在调整配置时不停止微服务。

- 配置修改后可自动更新。如配置内容发生变化，微服务能够自动更新配置。

综上所述，对于微服务架构而言，一个通用的配置管理机制是必不可少的，常见做法是使用配置服务器管理配置。

### 2.`Spring Cloud Config`简介

​      `Spring Cloud Config`为分布式系统外部化配置提供了服务器端和客户端的支持，它包括`Config Server`和`Config Client`两部分。由于`Config Server`和`Config Client`都实现了对Spring Environment和`PropertySource`抽象的映射，因此，`Spring Cloud Config`非常适合Spring应用程序，当然也可与任何其他语言编写的应用程序配合使用。

​     `Config Server`是一个可横向扩展、集中式的配置服务器，它用于集中管理应用程序各个环境下的配置，默认使用Git存储配置内容（也可使用Subversion、本地文件系统或Vault存储配置），因此可以很方便地实现对配置的版本控制与内容审计。

​     `Config Client`是`Config Server`的客户端，用于操作存储在`Config Server`中的配置属性。如图所示，所有的微服务都指向`Config Server`。各个微服务在启动时，会请求`Config Server`以获取所需要的配置属性，然后缓存这些属性以提高性能。

![](/image/SpringCloud/image-20200112211823789.png)

### 3.编写`Config Server`

下面来编写一个`Config Server`。在本例中，使用Git作为`Config Server`的后端存储。

1.在Git仓库https://github.com/superdevops-cn/spring-cloud-config-repo.git中新建几个配置文件，例如：

```java
    microservice-dev.properties
    microservice-pro.properties
    microservice-test.properties
    microservice-superdevops.properties
```

内容分别是：

```java
    profile=dev-1.0
    profile=pro-1.0
    profile=test-1.0
    profile=superdevops-1.0
```

为了测试版本控制，为该Git仓库创建`devops`分支，并将各个配置文件中的1.0改为2.0。

2.创建项目`microservice-config-server`，并为项目添加以下依赖。

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
    <artifactId>microservice-config-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-config-server</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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

3.在启动类上添加注解`@EnableConfigServer`，声明这是一个`Config Server`。

4.编写配置文件`application.yml`，并在其中添加以下内容。

```yaml
server:
  port: 8180
spring:
  application:
    name: microservice-config-server
  cloud:
    config:
      server:
        git:
          #配置Git仓库地址
          uri: https://github.com/superdevops-cn/spring-cloud-config-repo
          #Git 仓库的账号
          username: superdevops-cn
          #Git 仓库的密码
          password: 
```

这样，一个`Config Server`就完成了。

可以使用`Config Server`的端点获取配置文件的内容。端点与配置文件的映射规则如下：

```xml
{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

以上端点都可映射到{application}-{profile}.proper ties这个配置文件，{application}表示微服务的名称，{label}对应Git仓库的分支，默认是master。

按照以上规则，对于本例，可使用以下URL访问到Git仓库master分支的`microservice-dev.properties`，例如：

- *http://localhost:8180/microservice/dev*
- *http://localhost:8180/microservice-dev.yml*
- *http://localhost:8180/microservice-dev.properties*



**测试**

1.启动项目`microservice-config-server`

2.访问*http://localhost:8180/microservice/dev*，可获得如下结果

![image-20200118160651470](/image/SpringCloud/image-20200118160651470.png)

从结果可以直观地看到应用名称、项目profile、Git label、Git version、配置文件URL、配置详情等信息。

3.访问*http://localhost:8180/microservice-dev.properties*，返回配置文件中的属性：

```java
profile: dev-1.0
```

4.访问*http://localhost:8180/devops/microservice-dev.properties*，可获得如下结果：

```java
profile: dev-2.0
```

说明获得了Git仓库`devops`分支中的配置信息。

至此，已成功构建了`Config Server`，并通过构造URL的方式，获取了Git仓库中的配置信息。

### 4. 编写`Config Client`

​       前文已经构建了一个`Config Server`，并使用`Config Server`端点获取配置内容。本节来讨论Spring Cloud微服务如何获取配置信息。

下面来编写一个微服务，该微服务整合了`Config Client`。

1.新建`microservice-config-client`，并为项目添加如下依赖。

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
    <artifactId>microservice-config-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-config-client</name>
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
            <artifactId>spring-cloud-starter-config</artifactId>
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

2.编写配置文件`application.yml`，并在其中添加如下内容。

```yaml
server:
  port: 8181
```

3.创建配置文件`bootstrap.yml`，并在其中添加如下内容。

```yaml
spring:
  application:
    #对应config server 所获取的配置文件的{application}
    name: microservice
  cloud:
    config:
      uri: http://localhost:8180
      #profile对应config server 所获取的配置文件的{profile}
      profile: dev
      #指定Git 仓库的分支，对应config server 所获取的配置文件的{label}
      label: master

```

其中：

•`spring.application.name`：对应`Config Server`所获取的配置文件中的{application}。

•`spring.cloud.config.uri：`指定`Config Server`的地址，默认是*http://localhost:8888*。

•`spring.cloud.config.profile`：profile对应`Config Server`所获取的配置文件中的{profile}。

•`spring.cloud.config.label`：指定Git仓库的分支，对应`Config Server`所获取配置文件的{label}。

​       值得注意的是，以上属性应配置在`bootstrap.yml`，而不是`application.yml`中。如果配置在`application.yml`中，该部分配置就不能正常工作。例如，`Config Client`会连接`spring.cloud.config.uri`的默认值*http://localhost:8888*，而并非配置的*http://localhost:8180/*。

​     Spring Cloud有一个“引导上下文”的概念，这是主应用程序的父上下文。引导上下文负责从配置服务器加载配置属性，以及解密外部配置文件中的属性。和主应用程序加载application.*(`yml`或`properties)`中的属性不同，引导上下文加载bootstrap.*中的属性。配置在bootstrap.*中的属性有更高的优先级，因此默认情况下它们不能被本地配置覆盖。

若需禁用引导过程，可设置`spring.cloud.bootstrap.enabled=false`。

4.编写Controller。

```java
@RestController
public class ConfigClientController {
    @Value("${profile}")
    private  String profile;

    @GetMapping("/profile")
    public String getProfile(){
        return profile;
    }
}
```

**注意**

在Controller中，通过注解@Value("${profile}")，绑定Git仓库配置文件中的profile属性。



**测试**

1.启动`microservice-config-server`。

2.启动`microservice-config-client`。

3.访问*http://localhost:8181/profile*，可获得如下的结果。

```java
dev-1.0
```

说明`Config Client`能够正常通过`Config Server`获得Git仓库中对应环境的配置。



### 5. `Config Server`的Git仓库配置详解

​      前文使用`spring.cloud.conf ig.server.git.uri`指定了一个Git仓库，事实上，该属性非常灵活。本节来详细讨论`Config Server`的Git仓库配置。

#### 5.1 占位符支持

​    `Config Server`的占位符支持{application}、{profile}和{label}。

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/superdevops-cn/{application}
```
使用这种方式，即可轻松支持一个应用对应一个Git仓库。同理，也可支持一个profile对应一个Git仓库。

#### 5.2　模式匹配和多个配置仓库

​        模式匹配指的是带有通配符的{application}/{profile}名称的列表。如果{application}/{profile}不匹配任何模式，它将使用`spring.cloud.config.server.git.uri`定义的URI。

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
```

​       该例中，对于simple仓库，它只匹配所有配置文件中名为simple的应用程序。local仓库则匹配所有配置文件中以local开头的所有应用程序的名称。

#### 5.3 搜索目录

​        很多场景下，可能把配置文件放在了Git仓库子目录中，此时可以使用search-path指定，search-path同样支持占位符。

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          search-paths: foo,bar*
```

#### 5.4 启动时加载配置文件

​         默认情况下，在配置被首次请求时，C`onfig Server`才会`clone Git`仓库。也可让`Config Server`在启动时就clone Git仓库，例如:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          repos:
            team-a:
                pattern: team-a-*
                cloneOnStart: true
                uri: https://git/team-a/config-repo.git
            team-b:
                pattern: team-b-*
                cloneOnStart: false
                uri: https://git/team-b/config-repo.git
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```

​           将属性`spring.cloud.conf ig.server.git.repos.*.clone-on-start`设为true，即可让`Config Server`启动时clone指定Git仓库。

​          当然，也可使用`spring.cloud.conf ig.server.git.clone-on-start=true`进行全局配置。配置clone-on-start=true，可帮助`Config Server`启动时快速识别错误的配置源（例如无效的Git仓库）。

​       将以下包的日志级别设为DEBUG，就可打印`Config Server`请求Git仓库的细节。可以通过这种方式，更好地理解`Config Server`的Git仓库配置，同时也便于快速定位问题。

```yaml
logging:
  level: 
    org.springframework.cloud: debug
    org.springframework.boot: debug
```

### 6. `Config Server`的健康状况指示器

​          `Config Server`自带了一个健康状况指示器，用于检查所配置的`EnvironmentRepository`是否正常工作。可使用`Config Server`的/health端点查询当前健康状态。默认情况下，健康指示器向`EnvironmentRepository`请求的{application}是`app`，{profile}和{label}是对应`EnvironmentRepository`实现的默认值。对于Git，{profile}是default，{label}是master。

​           同样也可以自定义健康状况指示器的配置，从而检查更多的{application}、自定义的{profile}以及自定义的{label}，例如：

```yaml
spring:
  cloud:
    config:
      server:
        health:
          repositories:
           microservice:
              label: devops
              name: microservice
              profiles: dev
```



​          若需禁用健康状况指示器，可设置`spring.cloud.conf ig.server.health.enabled=false`。



### 7. 配置内容的加解密

#### 7.1 安装`JCE`

​            前文是在Git仓库中明文存储配置属性的。很多场景下，对于某些敏感的配置内容（例如数据库账号、密码等），应当加密存储。

​         `        Config Server`为配置内容的加密与解密提供了支持。

​         `Config Server`的加解密功能依赖`JavaCryptography Extension（JCE）`。

​         `  Java8JCE`的地址：*http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html*

​        下载`JCE`并解压，按照其中的`README.txt`的说明即可安装。`JCE`的安装非常简单，其实就是将`JDK/jre/lib/security`目录中的两个jar文件替换为压缩包中的jar文件。

其他Java版本的`JCE`地址，在`SpringCloud`文档中都有提及，详见：*http://cloud.spring.io/spring-cloud-static/Edgware.RELEASE/single/spring-cloud.html#_encryption_and_decryption*.



#### 7.2  `Config Server`的加解密端点

​        `Config Server`提供了加密与解密的端点，分别是/encrypt与/decrypt。可使用以下代码来加密明文：

```yaml
curl  $CONFIG_SERVER_URL/encrypt -d 想要加密的明文
```

​      使用以下代码来解密密文：

```shell
curl  $CONFIG_SERVER_URL/decrypt -d 想要解密的密文
```



#### 7.3 对称加密

1.新建项目`microservice-config-server-encryption`，复制项目`microservice-config-server`中的内容

2.创建`bootstrap.yml`，添加以下内容。

```yaml
encrypt:
  key: superdevops
  #对称加密密钥
```

这样，代码就编写完成了。

**测试**

1.启动项目`microservice-config-server-encryption`

2.输入以下命令

```shell
curl localhost:8180/encrypt --data-urlencode  sugelamei
```

可返回，说明已被加密。

```java
e4ed1718089bfe5c096b0aeabbafa21fd9e7582353ea02ef324e70106a68af37
```



3.输入以下命令。

```shell
curl localhost:8180/decrypt --data-urlencode e4ed1718089bfe5c096b0aeabbafa21fd9e7582353ea02ef324e70106a68af37
```



可返回`sugelamei`，说明能够正常解密。



- 以encrypt开头的配置，必须放在`bootstrap.yml`中，否则将无法正常加解密。



#### 7.4  存储加密的内容

​    加密后的内容，可使用{cipher}密文的形式存储。

1.准备一个配置文件，命名为`encryption.yml`。

```yaml
spring:
  datasource:
    username: dbuser
    password: '{cipher}e4ed1718089bfe5c096b0aeabbafa21fd9e7582353ea02ef324e70106a68af37'
```

并将其push到Git仓库（）。此处需注意`spring.datasource.password`上的单引号不能少。如读者使用properties格式管理配置，则不能使用单引号，否则该值不会被解密，正确写法如下：

```yaml
spring.datasource.username: dbuser
spring.datasource.password: {cipher}e4ed1718089bfe5c096b0aeabbafa21fd9e7582353ea02ef324e70106a68af37
```

2.使用*http://localhost:8180/encryption-default.yml*可获得如下结果。

```yaml
spring:
  datasource:
    password: sugelamei
    username: dbuser
```

说明`Config Server`能自动解密配置内容。

​        一些场景下，想要让`Config Server`直接返回密文本身，而并非解密后的内容，可设置`spring.cloud.config.server.encrypt.enabled=false`，这时可由`Config Client`自行解密。

#### 7.5  非对称加密

​     前文讨论了对称加密的方式，`Spring Cloud`同样支持非对称加密。

1.新建项目`microservice-config-server-encryption-rsa`复制项目`microservice-config-server`中的内容。

2.执行以下命令，并按照提示操作，即可创建一个Key Store。

```shell
keytool -genkeypair -alias mytestkey -keyalg RSA -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" -keypass changeme -keystore server.jks -storepass letmein
```

其中

```
-alias:别名
-keyalg: 指定密钥使用的加密算法（如 RSA ）
-validity:过期时间，单位天
-keystore: 指定存储密钥的密钥库的名称（二进制文件）
-keypass: 指定生成密钥的密码。
-storepass: 指定访问密钥库的密码
```

3.将生成的`server.jks`文件复制到项目的`classpath`下。

4.创建`bootstrap.yml`，添加以下内容。

```yaml
encrypt:
  keyStore:
    location: classpath:/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```

这样，使用以下命令：

```shell
curl localhost:8280/encrypt --data-urlencode  sugelamei
```

尝试加密时，就会得到类似以下的结果：

```java
AQBaQ198ouBcmwvl8TN5ZY4W435K4Eg2v0OQNnJQDVtdtJYKlL9CrwzBz0dZDyDqpqgrJr50LHN4qqSbspTxR8TwO9fO/mtgXwqsI8biPYQVycf2XRUj+rPJL85/QOKZewxZR0fxSbBI5hJwFcCf9GWzur2YQDR/ZjYy0SjsuZ1VVX3Kk/hOI6SEQyYEla8hufCwjHZU7LF6yakXhPFF9kWswKokmu62VvFi2k66huvhGERV3paCvWAYXRwpkgQk5YXsDwhK5UvMK3AbYqs+yyGlHaWhpRwYWV6WYmkpfvEIzqVa1eqY4QQi2Ic3Wv/kzvpnvJ6rNqxEk68kGRwBoKX84gJ0RdtsZTMcTZbkXZcAcxqw7asE6lsAPKFK1tqB7C8=
```

  相对于对称加密，非对称加密的安全性更高，但对称加密相对方便。读者可按照需求，自行选择加密方案。

以encrypt开头的配置，必须放在`bootstrap.yml`中，否则将无法正常加解密。

### 8. 使用/refresh端点手动刷新配置

​          很多场景下，需要在运行期间动态调整配置。如果配置发生了修改，微服务要如何实现配置的刷新呢？

​          要想实现配置刷新，需对之前的代码进行一点改造。

 1.新建项目   `microservice-config-client-refresh` 复制项目`microservice-config-client`中的内容

 2.在Controller上添加注解`@RefreshScope`。添加`@RefreshScope`的类会在配置更改时得到特殊的处理。

```java
@RestController
@RefreshScope
public class ConfigClientController {
    @Value("${profile}")
    private String profile;

    @GetMapping("/profile")
    public String getProfile(){
        return profile;
    }
}
```

这样就完成了代码改造。

3.在`application.yml`中添加如下内容：

```yaml
server:
  port: 8281
management:
  endpoints:
    web:
      exposure:
        include: refresh

```

**测试**

1.启动`microservice-config-server`。

2.启动`microservice-config-client-refresh`。

3.访问*http://localhost:8281/profile*，获得结果：dev-1.0。

4.修改Git仓库中`microservice-dev.properties`文件内容为profile=dev-1.0-change。

5.重新访问*http://localhost:8281/profile*，发现结果依然是dev-1.0，说明配置尚未刷新。

6.发送POST请求到http://localhost:8281/actuator/refresh，例如：

```shell
curl -X POST http://localhost:8281/actuator/refresh
```



返回结果：["profile"]，表示profile这个配置属性已被刷新。

7.再次访问*http://localhost:8281/profile*，返回dev-1.0-change，说明配置已经刷新。

   

### 9. 使用Spring Cloud Bus自动刷新配置

​         前文讨论了使用/refresh端点手动刷新配置，但如果所有微服务节点的配置都需要手动去刷新，工作量可想而知。不仅如此，随着系统的不断扩张，会越来越难以维护。因此，实现配置的自动刷新是很有必要的，本节将使用Spring Cloud Bus实现配置的自动刷新。

#### 9.1  Spring Cloud Bus简介

​      Spring Cloud Bus使用轻量级的消息代理（例如`RabbitMQ、Kafka`等）连接分布式系统的节点，这样就可以广播传播状态的更改（例如配置的更新）或者其他的管理指令。可将Spring Cloud Bus想象成一个分布式的Spring Boot  Actuator。使用Spring Cloud Bus后的架构如图所示。

![](/image/SpringCloud/image-20200206223415678.png)



​        由上图可知，微服务A的所有实例都通过消息总线连接到了一起，每个实例都会订阅配置更新事件。当其中一个微服务节点的/bus/refresh端点被请求时，该实例就会向消息总线发送一个配置更新事件，其他实例获得该事件后也会更新配置。



#### 9.2  实现自动刷新

​     安装`RabbitMQ`（不会的同学自己网上找找教程，笔者使用docker 容器中启动`RabbitMQ`）后，接下来为项目整合Spring Cloud Bus并实现自动刷新。

1新建项目 `microservice-config-client-refresh-cloud-bus。` 复制项目`microservice-config-client-refresh`中的内容。

2.为项目添加如下的依赖。

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
    <artifactId>microservice-config-client-refresh-cloud-bus</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-config-client-refresh-cloud-bus</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
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
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <artifactId>objenesis</artifactId>
                    <groupId>org.objenesis</groupId>
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

</project>

```

3.在`bootstrap.yml`中添加以下内容：

```yaml
spring:
  application:
    #对应config server 所获取的配置文件的{application}
    name: microservice
  cloud:
    config:
      uri: http://localhost:8180
      #profile对应config server 所获取的配置文件的{profile}
      profile: dev
      #指定Git 仓库的分支，对应config server 所获取的配置文件的{label}
      label: master
  rabbitmq:
    host: 192.168.10.21
    port: 5672
    username: guest
    password: guest

```

**注意**：`rabbitmq`相关的信息，一定要放在`bootstrap.yml`中。

4.在`application.yml`中添加以下内容：

```yaml
server:
  port: 8281
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```



**测试**

1.启动`microservice-config-server`。

2.启动`microservice-config-client-refresh-cloud-bus`

3.将`microservice-config-client-refresh-cloud-bus`的端口改为8282，再启动一个节点。

4.访问*http://localhost:8281/profile*，可获得结果：`dev-1.0-change`。

5.将Git仓库中的`microservice-foo-dev.properties`文件内容修改为如下内容:

```java
dev-1.0-change-bus
```

6.发送POST请求到其中一个`Config Client`实例的/bus-refresh端点，例如：

```java
curl -X POST http://localhost:8281/actuator/bus-refresh
```

7.访问两个Config Client节点的/profile端点，发现两个节点都会返回`dev-1.0-change-bus`，说明配置内容已被刷新。

8.借助Git仓库的WebHooks，即可轻松实现配置的自动刷新

![image-20200210232754526](/image/SpringCloud/image-20200210232754526.png)

#### 9.3  局部刷新

​         某些场景下（例如灰度发布等），若只想刷新部分微服务的配置，可通过/bus/refresh端点的destination参数来定位要刷新的应用程序。

​         例如/bus-refresh?destination=`customers:9000`，这样消息总线上的微服务实例就会根据destination参数的值来判断是否需要刷新。其中，`customers:9000`指的是各个微服务的`ApplicationContext ID`。默认情况下，`ApplicationContext ID`是`spring.application.name:server.port`。

​         destination参数也可以用来定位特定的微服务。例如/bus-refresh?destination=customers:**，这样就可以触发customers微服务所有实例的配置刷新。

#### 9.4 架构改进

​       在前面的示例中，通过请求某个微服务/bus-refresh端点的方式来实现配置刷新，但这种方式并不优雅。原因如下：

​      •破坏了微服务的职责单一原则。业务微服务只应关注自身业务，不应承担配置配置刷新的职责。

​      •破坏了微服务各节点的对等性。

​      •有一定的局限性。例如，微服务在迁移时，网络地址常常会发生变化。此时如果想自动刷新配置，就不得不修改`WebHook`的配置。

如下图所示，将`Config Server`也加入消息总线中，并使用`Config Server`的/bus-refresh端点来实现配置的刷新。这样，各个微服务只需要关注自身的业务，而不再承担配置刷新的职责。

![](/image/SpringCloud/image-20200210233245123.png)

##### 9.4.1 使用`Config Server`实现自动刷新

​     安装`RabbitMQ`（不会的同学自己网上找找教程，笔者使用docker 容器中启动`RabbitMQ`）后，接下来为项目整合Spring Cloud Bus并使用`Config Server`实现自动刷新。

1新建项目 `microservice-config-server-refresh-cloud-bus` 复制项目`microservice-config-server`中的内容。

2.为项目添加如下的依赖。

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
    <artifactId>microservice-config-server-refresh-cloud-bus</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-config-server-refresh-cloud-bus</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
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

3.在`application.yml`中添加以下内容：

```yaml
server:
  port: 8180
spring:
  application:
    name: microservice-config-server-refresh-cloud-bus
  cloud:
    config:
      server:
        git:
          #配置Git仓库地址
          uri: https://github.com/superdevops-cn/spring-cloud-config-repo
          #Git 仓库的账号
          username: superdevops-cn
          #Git 仓库的密码
          password: wxc291748zy
    bus:
      trace:
        enabled: true     # 开启cloud bus的跟踪
  rabbitmq:
    host: 192.168.10.21
    port: 5672
    username: guest
    password: guest
```

**测试**

1.启动`microservice-config-server-refresh-cloud-bus`。

2.启动`microservice-config-client-refresh-cloud-bus`

3.将`microservice-config-client-refresh-cloud-bus`的端口改为8282，再启动一个节点。

4.访问*http://localhost:8281/profile*，可获得结果：`dev-1.0-change-bus`。

5.将Git仓库中的`microservice-foo-dev.properties`文件内容修改为如下内容:

```java
dev-1.0-config-server
```

6.发送POST请求到其中一个`Config Server`实例的/bus-refresh端点，例如：

```shell
curl -X POST http://localhost:8081/actuator/bus-refresh
```

7.访问两个~节点的/profile端点，发现两个节点都会返回`dev-1.0-config-server`，说明配置内容已被刷新。 



### 10. `Spring Cloud Config`与Eureka配合使用

​         前文在微服务中指定了`Config Server`地址，这种方式无法利用服务发现组件的优势。本节将讨论`Config Server`注册到Eureka Server上时，如何使用`Spring Cloud Config`。

#### 10.1 改造`Config Server`

1.新建项目`microservice-config-server-eureka`，将`microservice-config-server`中的代码复制过来。

2.将`Config Server`都注册到Eureka Server上。

3.`Config Server`的`application.yml`可配置如下。

```yaml
server:
  port: 8180
spring:
  application:
    name: microservice-config-server-eureka
  cloud:
    config:
      server:
        git:
          #配置Git仓库地址
          uri: https://github.com/superdevops-cn/spring-cloud-config-repo
          #Git 仓库的账号
          username: superdevops-cn
          #Git 仓库的密码
          password: 

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```



#### 10.2 改造`Config Client`

1.新建项目`microservice-config-client-eureka`，将`microservice-config-client`中的代码复制过来。

2.将`Config Server`都注册到Eureka Server上。

3.`Config Client`的`bootstrap.yml`可配置如下。

```yaml
spring:
  application:
    #对应config server 所获取的配置文件的{application}
    name: microservice
  cloud:
    config:
      uri: http://localhost:8180
      #profile对应config server 所获取的配置文件的{profile}
      profile: dev
      #指定Git 仓库的分支，对应config server 所获取的配置文件的{label}
      label: master
      discovery:
        enabled: true
        service-id: microservice-config-server-eureka
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

> 下面简要说明一下`bootstrap.yml`中的配置属性。
>
> `spring.cloud.config.discovery.enabled=true`：表示开启通过服务发现组件访问`Config Server`的功能。
>
> `spring.cloud.config.discovery.service-id`：指定`Config Server`在服务发现组件中的`serviceId`。



### 11. `Spring Cloud Config`的用户认证

在前文的示例中，`Config Server`是允许匿名访问的。为了防止配置内容的外泄，应该保护`Config Server`的安全。有多种方式做到这一点，例如通过物理网络安全，或者为`Config Server`添加用户认证等。

本节来为`Config Server`添加基于`HTTPBasic`的用户认证。

先来构建一个需要用户认证的`Config Server`。

1.新建项目`microservice-config-server-authenticating`，复制项目`microservice-config-server`中内容 。

2.为项目添加以下依赖。

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
    <artifactId>microservice-config-server-authenticating</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice-config-server-authenticating</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
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

3.在`application.yml`中添加如下内容

```yaml
server:
  port: 8180
spring:
  application:
    name: microservice-config-server
  cloud:
    config:
      server:
        git:
          #配置Git仓库地址
          uri: https://github.com/superdevops-cn/spring-cloud-config-repo
          #Git 仓库的账号
          username: superdevops-cn
          #Git 仓库的密码
          password: 
  security:
    user:
      name: user
      password: 123456


```

​         这样就为`Config Server`添加了基于`HTTPbasic`的认证。如果不设置这段内容，账号默认是user，密码是一个随机值，该值会在启动时打印出来。

​     此时启动该`Config Server`，会看到如下的登录对话框如下：

![image-20200212184348008](/image/SpringCloud/image-20200212184348008.png)

`Config Client`有两种方式使用需用户认证的`Config Server`。

1.新建项目`microservice-config-client-authenticating`，复制`microservice-config-client`项目中的代码。

•方式一：使用curl风格的URL。示例：

```yaml
spring:
  cloud:
    config:
      uri: http://user:123456@localhost:8180
```

•方法二：指定`Config Serve`r的账号与密码。示例：

```yaml
spring:
  cloud:
    config:
      uri: http://localhost:8180
      username: user
      password: 123456
```

需要注意的是`spring.cloud.config.password`和`spring.cloud.config.username`的优先级较高，它们会覆盖URL中包含的账号与密码。



### 12. `Config Server`的高可用

​         前文构建的都是单节点的`Config Server`，本节来讨论如何构建高可用的`Config Server`集群，包括`Config Server`的高可用依赖Git仓库的高可用以及`RabbitMQ`的高可用。

下面先来讨论Git仓库的高可用。

#### 12.1　Git仓库的高可用

由于配置内容都存储在Git仓库中，所以要想实现`Config Server`的高可用，必须有一个高可用的Git仓库。有两种方式可以实现Git仓库的高可用。

•使用第三方Git仓库：这种方式非常简单，可使用例如`GitHub、BitBucket、git@osc、Coding`等提供的仓库托管服务，这些服务本身就已实现了高可用。

•自建Git仓库管理系统：使用第三方服务的方式虽然省去了很多烦恼，但是很多场景下，倾向于自建Git仓库管理系统。此时就需要保证自建Git的高可用。

以`GitLab`为例，读者可参照官方文档搭建高可用的`GitLab`：

> •高可用GitLab复杂度分析：*https://about.gitlab.com/high-availability/*.
>
> •高可用GitLab搭建文档：*https://docs.gitlab.com/ce/administration/high_availability/README.html*.
>

#### 12.2　`RabbitMQ`的高可用

​     还记得前文使用Spring Cloud Bus实现了配置的自动刷新吗？由于Spring Cloud Bus依赖`RabbitMQ`（当然也可使用其他`MQ`），所以`RabbitMQ`的高可用也是必不可少的。

​        搭建高可用`RabbitMQ`的资料，读者可详见：*https://www.rabbitmq.com/ha.html*。由于比较简单，笔者不作赘述。当然，也可使用云平台的提供的`RabbitMQ`服务。

#### 12.3　`Config Serve`r自身的高可用

本小节来讨论如何实现Config Server自身的高可用。笔者分两种场景进行讨论。

##### 12.3.1　`Config Server`未注册到Eureka Server上

对于这种情况，`Config Server`的高可用可借助一个负载均衡器来实现，如下图所示：

![](/image/SpringCloud/image-20200212184756123.png)

各个微服务将请求发送到负载均衡器，负载均衡器将请求转发到其代理的其中一个`Config Server`节点。这样，就可以实现`Config Server`的高可用。

##### 12.3.2　`Config Server`注册到Eureka Server上

这种情况下，`Config Server`的高可用相对简单，只需将多个`Config Server`节点注册到Eureka Server上，即可实现`Config Server`的高可用。架构如图所示：

![](/image/SpringCloud/image-20200212184812456.png)

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