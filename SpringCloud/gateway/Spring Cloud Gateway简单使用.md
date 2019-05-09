---

---

![中文测试](../../图床/title1.jpg)

## Spring Cloud Gateway 简单使用

 	Spring Cloud Gateway作为spring新推出的网关，与zuul的主要区别是采用了非阻塞式API。(emmm，其实个人阻塞非阻塞感觉提升一般般)。还有一些长连接啥的估计一般也用不到，先入个门再说。

#### 本节目标

> 浏览器访问 http://网关地址:网关端口/service-id/** 转发到 http://服务地址:服务端口/**

#### 环境:

pom.xml

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
</properties>

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
<dependencyManagement>

<dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
</dependencies>
```

网关信息可以在配置文件或者代码中配置。

##### 配置文件方式

------

根据路径转发:

```yml
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
      - id: test
        uri: http://localhost:8081	#目标地址
        predicates:
        	- Path=/spring-cloud	#路径匹配
```

这段配置的作用是访问 http://localhost:8080/spring-cloud/test 时会转发到 http://localhost:8081/spring-cloud/test 上。

初次之外，spring cloud gateway最大的特点还是根据service-id转发。(service-id就是在eureka上边看见的那个注册名)。

根据service-id转发:

```yml
server:
  port: 8080

spring:
  application:
    name: gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true		#开启根据服务转发
          lower-case-service-id: true	#服务名默认注册是大写，开启后输入小写即可访问
```

