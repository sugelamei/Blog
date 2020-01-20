```yaml
title: Spring Cloud-01 开始使用微服务
date: 2019-11-13 10:29:54 
tags: 
  - SpringCloud
```

[TOC]

### 0.说在前面

```java
- JDK：Spring Cloud官方建议使用JDK 1.8。
- Spring Boot：笔者使用Spring Boot 2.1.10.RELEASE
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3;
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；
```



### 1.Spring Cloud使用的前提

​     Spring Cloud 不一定适合所有人，想要玩转Spring Cloud 需要具备什么的技术能力，以及在实战中会使用哪些工具。

#### 1.1 技术储备

Spring Cloud 并不是面向零基础的开发人员，它有一定的学习曲线；

- 语言基础：Spring Cloud是一个基于Java语言的工具套件，所以学习它需要一定的Java基础。当然，Spring Cloud同样也支持使用Scala、Groovy等语言进行开发。本书的示例代码都是使用Java编写的。

- Spring Boot：SpringCloud是基于Spring Boot构建的，因此它延续了Spring Boot的契约模式以及开发方式。如果大家对Spring Boot不熟悉，建议花一点时间入门。

- 项目管理与构建工具：目前业界比较主流的项目管理与构建工具有Maven和Gradle等，本书采用的是目前相对主流的Maven。

  #### 1.2 工具及软件版本

-  JDK：Spring Cloud官方建议使用JDK 1.8。
- Spring Boot：笔者使用Spring Boot 2.2.1.RELEASE
- Spring Cloud：笔者使用Spring Cloud  Greenwich.SR3
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用IntelliJIDEA；
- Maven：笔者使用Maven 3.6.1构建项目；

### 2.服务提供者与服务消费者

使用微服务构建的是分布式系统，微服务之间通过网络进行通信。我们使用服务提供者与服务消费者来描述微服务之间的调用关系。

- 服务提供者：服务的被调用方（为其他服务提供服务的服务）
- 服务消费者：服务的调用方（依赖其他服务的服务）

以一个电影售票系统为例，架构图如下所示：

<img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191113214123622.png" style="zoom:80%;" />

围绕该场景，先来编写一个用户微服务，然后编写一个电影微服务。

### 3.编写服务提供者

​    编写一个服务提供者（用户微服务），该服务可通过主键查询用户信息。为方便测试与代码的编写，使用Mybatis作为持久层框架并且使用，使用Mysql作为数据库。

#### 3.1 使用 Spring Initializr快速创建Spring Boot项目（microservice-simple-provider-user）

- 使用IntelliJIDEA创建一个空的maven项目
- 新建一个Moudule 选择Spring Initializr 如下图：

<img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191113215322376.png" alt="image-20191113215322376" style="zoom:25%;" />

-  Next 输入微服务的名字以及相关的信息

​                                <img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191113215820445.png" alt="image-20191113215820445" style="zoom: 25%;" />       

- Next 选择需要的模块，Web，Mysql，JDBC，Mybatis和SpringBoot 2.1.10

<img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191114191207504.png" alt="image-20191118225743998" style="zoom:25%;" />

- Next 然后Finish

<img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191113220404252.png" alt="image-20191113220404252" style="zoom:25%;" />

  到此，项目创建就算成功了。

#### 3.2 准备数据库脚本,在项目的classpath下建立schema.sql,并添加如下内容：

```sql
drop table if exists   user ;

create table user
(
    id bigint not null,
    name varchar(40) ,
    age int,
    balance decimal(10,2),
    data_base varchar(40),
    primary key (id)
);
```

#### 3.3 准备几条数据,在项目的classpath下建立文件data.sql，并添加如下内容：

```sql
INSERT INTO user(id, name, age, balance,data_base)VALUES(1, '张三', 18, 100,DATABASE());
INSERT INTO user(id, name, age, balance,data_base)VALUES(2, '李四', 28, 200,DATABASE());
INSERT INTO user(id, name, age, balance,data_base)VALUES(3, '王五', 38, 300,DATABASE());
```

#### 3.4 编写配置文件，命名为application.yml

在传统的Web开发中，常使用properties格式文件作为配置文件。Spring Boot以及Spring Cloud支持使用properties或者yml格式的文件作为配置文件。

 yml文件格式是YAML（Yet Another Markup Language）编写的文件格式，YAML和properties格式的文件可互相转换，例如本节中的application.yml，就等价于如下的properties文件。

```yaml
#服务端口
server:
  port: 8080
spring:
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
```

 启动程序，会自动建表并且把数据入数据库。

从中不难看出，YAML比properties结构清晰；可读性、可维护性也更强，并且语法非常简捷。因此，本书使用YAML格式作为配置文件。**yml有严格的缩进，并且key与value之间使用**:**分隔，冒号后的空格不能少，请大家注意**。

#### 3.5使用 springboot + mybatis generator自动生成代码

  为了方便后续的使用，单独写一个springboot项目来生成代码，具体步骤如下，

- 选择Mysql，JDBC，Mybatis和SpringBoot 2.1.10；

<img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191114193448063.png" alt="image-20191118230016612" style="zoom:25%;" />

- 加入 mybatis generator自动生成代码插件 pom

  ```xml
  <!-- mybatis generator 自动生成代码插件 -->
              <plugin>
                  <groupId>org.mybatis.generator</groupId>
                  <artifactId>mybatis-generator-maven-plugin</artifactId>
                  <version>1.3.2</version>
                  <configuration>
                      <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
                      <overwrite>true</overwrite>
                      <verbose>true</verbose>
                  </configuration>
                  <dependencies>
                      <dependency>
                          <groupId>mysql</groupId>
                          <artifactId>mysql-connector-java</artifactId>
                          <version>8.0.18</version>
                      </dependency>
                  </dependencies>
              </plugin>
  ```

  

-  在pom文件中配置的Mybatis-Generator 工具配置文件的位置新建一个generatorConfig.xml

   

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
          "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
  <generatorConfiguration>
      <!--加载配置文件，为下面读取数据库信息准备-->
      <properties resource="application.properties"/>
  
      <!--defaultModelType="flat" 大数据字段，不分表 -->
      <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
          <property name="autoDelimitKeywords" value="true" />
          <property name="beginningDelimiter" value="`" />
          <property name="endingDelimiter" value="`" />
          <property name="javaFileEncoding" value="utf-8" />
          <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
  
          <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
  
          <!-- 注释 -->
          <commentGenerator >
              <property name="suppressAllComments" value="true"/><!-- 是否取消注释 -->
              <property name="suppressDate" value="true" /> <!-- 是否生成注释代时间戳-->
          </commentGenerator>
  
          <!--数据库链接地址账号密码-->
          <jdbcConnection driverClass="${spring.datasource.driver-class-name}"
                          connectionURL="${spring.datasource.url}"
                          userId="${spring.datasource.username}"
                          password="${spring.datasource.password}">
               <property name="nullCatalogMeansCurrent" value="true"/>
          </jdbcConnection>
  
          <!-- 类型转换 -->
          <javaTypeResolver>
              <!-- 是否使用bigDecimal， false可自动转化以下类型（Long, Integer, Short, etc.） -->
              <property name="forceBigDecimals" value="false"/>
          </javaTypeResolver>
  
          <!--生成Model类存放位置-->
          <javaModelGenerator targetPackage="com.sugelamei.entity" targetProject="src/main/java">
              <property name="enableSubPackages" value="true"/>
              <property name="trimStrings" value="true"/>
          </javaModelGenerator>
  
          <!-- 生成mapxml文件 -->
          <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources" >
              <property name="enableSubPackages" value="false" />
          </sqlMapGenerator>
  
          <!-- 生成mapxml对应client，也就是接口dao -->
          <javaClientGenerator targetPackage="com.sugelamei.mapper" targetProject="src/main/java" type="XMLMAPPER" >
              <property name="enableSubPackages" value="false" />
          </javaClientGenerator>
  
          <table tableName="article" enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true">
              <generatedKey column="id" sqlStatement="Mysql" identity="true" />
          </table>
          <!---->
          <table  tableName="user" enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true">
              <generatedKey column="id" sqlStatement="Mysql" identity="true" />
          </table>
  
      </context>
  </generatorConfiguration>
  ```
  
  注意:
  
  最好单独用一个模块或者项目生成代码和对应的*Mapper.xml,每次生成完以后就删除，重复运行生成代码的mybatis-generator:generate 不会覆盖以前的文件 会直接追加，在执行的时候就会报错！！！
  
-   在application.properties中加入如下配置

    ```properties
    # MyBatis 配置
    ## mapper xml 文件地址
    mybatis.mapper-locations=classpath*:mapper/*Mapper.xml
    ##数据库url 注意不要加数据库名字
    spring.datasource.url=jdbc:mysql://192.168.10.21:3306/microservice01
    ##数据库用户名
    spring.datasource.username=root
    ##数据库密码
    spring.datasource.password=root
    ##数据库驱动
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    
    ```

-   在代码生成**前**目录结构如下图所示：

    <img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191116213057227.png" alt="image-20191116213057227" style="zoom: 50%;" />

- 然后点击生成代码

  <img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191116213243818.png" alt="image-20191116213243818" style="zoom:50%;" />

- 在代码生成**后**目录结构如下图所示：

  <img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191116235956434.png" alt="image-20191116235956434" style="zoom:50%;" />

- 至此，代码自动生成完毕，复制生成的代码到你自己的项目目录下.



#### 3.6 添加service和controller

-   添加 UserService和UserServiceImpl

```java
public interface UserService {
    
    int deleteByPrimaryKey(Long id);

    int insert(User record);

    User selectByPrimaryKey(Long id);

    List<User> selectAll();

    int updateByPrimaryKey(User record);
}
```

```java
@
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;
    @Override
    public int deleteByPrimaryKey(Long id) {
        return userMapper.deleteByPrimaryKey(id);
    }

    @Override
    public int insert(User record) {
        return insert(record);
    }

    @Override
    public User selectByPrimaryKey(Long id) {
        return userMapper.selectByPrimaryKey(id);
    }

    @Override
    public List<User> selectAll() {
        return userMapper.selectAll();
    }

    @Override
    public int updateByPrimaryKey(User record) {
        return userMapper.updateByPrimaryKey(record);
    }
}
```

- 添加UserController（为了测试就写一个方法）

  ```java
  @RestController
  public class UserController {
  
      @Autowired
      private UserService userService;
  
      @GetMapping("/user/{id}")
      public User findUserById(@PathVariable("id") Long id) {
          return userService.selectByPrimaryKey(id);
      }
  }
  ```

  UserController中用到的@GetMapping和@RestController注解。

  @GetMapping它是一个组合注解，等价于@RequestMapping(method=RequestMethod.GET)，用于简化开发。同理还有@PostMapping、@PutMapping、@DeleteMapping、@PatchMapping等。

  @RestController也是一个组合注解，等价于@Controller + @ResponseBody

  

  #### 3.7 最终项目的目录结构如下图所示：

  <img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191116235837020.png" alt="image-20191116235837020" style="zoom:50%;" />

#### 3.8 启动并测试

-  在MicroserviceSimpleProviderUserApplication上加上@MapperScan(basePackages = "com.sugelamei.mapper")

```java
@MapperScan(basePackages = "com.sugelamei.mapper")
@SpringBootApplication
public class MicroserviceSimpleProviderUserApplication {

    public static void main(String[] args) {
        SpringApplication.run(MicroserviceSimpleProviderUserApplication.class, args);
    }

}
```



- 启动MicroserviceSimpleProviderUserApplication

-  访问http://localhost:8080/user/1 获取结果

  ![image-20191120220758616](G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191116235215043.png)



### 4.编写服务消费者

​    前文编写了一个服务提供者（用户微服务），本节来编写一个服务消费者（电影微服务）。该服务非常简单，它使用RestTemplate调用用户微服务的API，从而查询指定ID的用户信息。

- 使用Spring Initializr快速创建，只需要加入web即可，ArtifactId是microservice-simple-consumer-movie。

- 将entity包复制到com.sugelamei目录下

- 编写config配置类如下：

  ```java
  @Configuration
  public class ConsumerConfig {
      
      @Bean
      public RestTemplate restTemplate(){
          return new RestTemplate();
      }
  }
  
  ```

  

@onfiguration标注在类上，相当于把该类作为spring的xml配置文件中的<beans>，作用为：配置spring容器(应用上下文) , 等价于<Beans></Beans> 

@Bean是一个方法注解，作用是实例化一个Bean并使用该方法的名称命名。在本例中，添加@Bean注解的restTemplate()方法，等价于RestTemplate restTemplate=new RestTemplate();, @Bean等价于<Bean></Bean> 。

- .创建MovieController，在其中使用RestTemplate请求用户微服务的API。

```java
@RestController
public class MovieController {
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("consume/user/{id}")
    public User findById(@PathVariable("id") Long id){
      return   restTemplate.getForObject("http://localhost:8080/user/"+id,User.class);
    }

}
```

- 创建application.yml配置文件，并配置端口：

```yaml
server:
  port: 8081
```

- 最终项目的目录结构如下图所示：

<img src="G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191117002302313.png" alt="image-20191117002302313" style="zoom:50%;" />

- 启动MicroserviceSimpleConsumerMovieApplication

- 访问

  ```
  http://localhost:8081/consume/user/1
  ```
  
   获取结果（注意端口和请求路径不一样，服务提供者也需要在运行状态）
  
  ![image-20191120220651049](G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191117002651887.png)

说明电影微服务可以正常使用RestTemplate调用用户微服务的API。

### 5.硬编码有哪些问题

至此，已经实现了一个用户微服务和电影微服务，并在电影微服务中使用RestTemplate调用用户微服务中的RESTfulAPI。一切都是那么的自然、简单、perfect！

 

那么真的完美吗？来分析一下电影微服务的代码，在MovieController.java中：

```java
     @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id){
      return        restTemplate.getForObject("http://localhost:8080/user/"+id,User.class);
    }
    
```

代码可知，我们是把提供者的网络地址（IP和端口等）硬编码在代码中的，当然，也可将其提取到配置文件中去。例如：

```yaml
user:
  userServiceUrl: http://localhost:8080/
```

代码改为：

```java
@Value("${user.userServiceUrl}")
private String userServiceUrl;
 @GetMapping("/user/{id}")
    public User findById(@PathVariable("id") Long id){
      return   restTemplate.getForObject(userServiceUrl+id,User.class);
    }

```

在传统的应用程序中，一般都是这么做的。然而，这种方式有很多问题，如下所示。

•适用场景有局限：如果服务提供者的网络地址（IP和端口）发生了变化，会影响服务消费者。例如，用户微服务的网络地址发生变化，就需要修改电影微服务的配置，并重新发布,这显然是不可取的。

•无法动态伸缩：在生产环境中，每个微服务一般都会部署多个实例，从而实现容灾和负载均衡。在微服务架构的系统中，还需要系统具备自动伸缩的能力，例如动态增减节点等,硬编码无法适应这种需求。

那么要如何解决这些问题呢？

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![img](G:\Blog\source\image\common\Java橙序猿.png) 

关注博客：

```
 http://superdevops.cn
```

### 参考资料：

[《Spring Cloud 与Docker 微服务架构实战》](https://book.douban.com/subject/30278673/)

