### 5.**为项目整合Spring BootActuator**

​     Spring Boot Actuator提供了很多监控端点。可使用http://{ip}:{port}/{endpoint}的形式访问这些端点，从而了解应用程序的运行状况。

Actuator提供的端点，如下图所示：

| ID               | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| auditevents      | Exposes audit events information for the current application. Requires an AuditEventRepository bean. |
| beans            | Displays a complete list of all the Spring beans in your application. |
| caches           | Exposes available caches.                                    |
| conditions       | Shows the conditions that were evaluated on configuration and auto configuration classes and the reasons why they did or did not match. |
| configprops      | Displays a collated list of all @ConfigurationProperties.    |
| env              | Exposes properties from Spring’s ConfigurableEnvironment.    |
| flyway           | Shows any Flyway database migrations that have been applied. Requires one or more Flyway beans. |
| health           | Shows application health information.                        |
| httptrace        | Displays HTTP trace information (by default, the last 100 HTTP request-response exchanges). Requires an HttpTraceRepository bean. |
| info             | Displays arbitrary application info.                         |
| integrationgraph | Shows the Spring Integration graph. Requires a dependency on spring-integration-core. |
| loggers          | Shows and modifies the configuration of loggers in the application. |
| liquibase        | Shows any Liquibase database migrations that have been applied. Requires one or more Liquibase beans. |
| metrics          | Shows ‘metrics’ information for the current application.     |
| mappings         | Displays a collated list of all @RequestMapping paths.       |
| scheduledtasks   | Displays the scheduled tasks in your application.            |
| sessions         | Allows retrieval and deletion of user sessions from a Spring Session backed session store. Requires a Servlet-based web application using Spring Session. |
| shutdown         | Lets the application be gracefully shutdown. Disabled by default. |
| threaddump       | Performs a thread dump                                       |

由于在后面有大量的章节需要用到Actuator，不妨先来为项目整合Actuator，以项目microservice-simple-provider-user为例。

- 为项目添加以下依赖，在创建项目时也可以选择Actuator，则不需要添加依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

这样，就整合好Actuator了。是不是十分简单呢？同理，也可为项目microservice-simpleconsumer-movie整合Actuator。

- 重启MicroserviceSimpleProviderUserApplication

- 测试**/health端点**，访问 

  ```
  http://localhost:8080/actuator/health 
  ```

![image-20191117005136194](G:\Blog\source\image\SpringCloud与Docker微服务实战\image-20191117005136194.png)

此时，可展示当前应用的健康状况。其中，UP表示运行正常，除UP外，还有DOWN、OUT_OF_SERVICE、UNKNOWN等状态。此时，只显示了一个概要情况，如需展示详情，可为应用添加

```properties
management.endpoint.health.show-details=always
```

```
http://localhost:8080/actuator/auditevents

```

