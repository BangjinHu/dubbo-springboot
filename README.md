# SpringBoot + Dubbo + zookeeper 搭建简单分布式服务

[![](https://img.shields.io/github/issues/BillyYangOne/dubbo-springboot.svg)](https://github.com/BillyYangOne/dubbo-springboot/issues)  [![](https://img.shields.io/github/forks/BillyYangOne/dubbo-springboot.svg)](https://github.com/BillyYangOne/dubbo-springboot/network) [![](https://img.shields.io/github/stars/BillyYangOne/dubbo-springboot.svg)](https://github.com/BillyYangOne/dubbo-springboot/stargazers)  [![](https://img.shields.io/github/release/BillyYangOne/dubbo-springboot.svg)](https://github.com/BillyYangOne/dubbo-springboot/releases)

> springboot 简易集成 dubbo，使用 zookeeper 作为注册中心。

## 一、Dubbo 简介
>Apache Dubbo |ˈdʌbəʊ| 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。
 
### 1. 架构
![架构框架](https://github.com/BillyYangOne/dubbo-springboot/blob/master/dubbo-interface/src/main/resources/image/architecture.png)
### 2. 特征
* 2.1、 **面向接口代理的高性能RPC调用**

提供高性能的基于代理的远程调用能力，服务以接口为粒度，为开发者屏蔽远程调用底层细节。
* 2.2、 **智能负载均衡**

内置多种负载均衡策略，智能感知下游节点健康状况，显著减少调用延迟，提高系统吞吐量。
* 2.3、 **服务自动注册与发现**

支持多种注册中心服务，服务实例上下线实时感知。
* 2.4、 **高度可扩展能力**

遵循微内核+插件的设计原则，所有核心能力如Protocol、Transport、Serialization被设计为扩展点，平等对待内置实现和第三方实现。
* 2.5、 **运行期流量调度**

内置条件、脚本等路由策略，通过配置不同的路由规则，轻松实现灰度发布，同机房优先等功能。
* 2.6、 **可视化的服务治理与运维**   

提供丰富服务治理、运维工具：随时查询服务元数据、服务健康状态及调用统计，实时下发路由策略、调整配置参数。

## 二、搭建 zookeeper 环境
### 1、下载安装包
通过网站 [http://mirror.bit.edu.cn/apache/zookeeper/](http://mirror.bit.edu.cn/apache/zookeeper/) 下载安装包。
### 2、安装
* 2.1、 **windows 环境**

启动 bin 下的 cmd 文件即可， zookeeper-3.4.14\bin\zkServer.cmd
* 2.2、**linux 环境**

1. 解压
```
tar -zxvf zookeeper-3.4.14.tar.gz
```
2. 进入 zookeeper 目录，创建 data 文件夹
```
mkdir data
```
3. 赋值 conf 目录下 zoo_sample.cfg 为 zoo.cfg
```
cp zoo_sample.cfg zoo.cfg
```
4. 修改 zoo.cfg 中的 data 属性
```
dataDir=/usr/local/zookeeper/data
```
5. 启动 zookeeper 
```
./zkServer.sh start
```
## 三、springboot 集成 dubbo
### 1、项目目录结构
![目录结构](https://github.com/BillyYangOne/dubbo-springboot/blob/master/dubbo-interface/src/main/resources/image/structure.png)
### 2、代码编写
* 2.1、 **父项目编写**

 创建maven项目 dubbo-springboot ，添加以下依赖到 pom.xml 文件中：
```xml
   <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/>
    </parent>
    
    <groupId>com.billy</groupId>
    <artifactId>dubbo-springboot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>1.8</java.version>
        <spring-boot.version>2.2.1.RELEASE</spring-boot.version>
        <dubbo.version>2.7.3</dubbo.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <!--  引入 dubbo-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <!-- Zookeeper dependencies -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>
```
* 2.2、 **接口项目编写**
  
为了减少冗余，编写接口项目 dubbo-interface ，为服务端和消费端提供接口。编写简单接口如下：
```java
/**
 * 接口类
 */
public interface HelloService {

    String sayHello(String name);
}
```
* 2.3、 **provider 项目编写**

服务提供者 dubbo-provider ，用于提供服务。
1. `pom.xml` 中引入接口依赖，
```xml
    <parent>
        <artifactId>dubbo-springboot</artifactId>
        <groupId>com.billy</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbo-provider</artifactId>

    <dependencies>
        <!-- 引入接口模块 -->
        <dependency>
            <groupId>com.billy</groupId>
            <artifactId>dubbo-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
```
2. 编写服务层，如下：
```java
/**
 * 服务层
 */
@Service
public class HelloServiceImpl implements HelloService {

    /**
     * The default value of ${dubbo.application.name} is ${spring.application.name}
     */
    @Value("${dubbo.application.name}")
    private String serviceName;

    @Override
    public String sayHello(String name) {
        return String.format("[%s] : hell, %s", serviceName, name);
    }
}
```
注意： 此处 @Service 引自dubbo.

3. 启动类增加 `@EnableDubbo`， 用于加载 dubbo 配置
```java
@EnableDubbo
@SpringBootApplication
public class DubboProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboProviderApplication.class, args);
    }

}
``` 
4. 配置文件中增加 dubbo 配置
```properties
# Spring boot application
spring.application.name=dubbo-provider
server.port=8661

# 当前dubbo应用ID
dubbo.application.id=dubbo-provider
# 当前dubbo应用name
dubbo.application.name=dubbo-provider
# Base packages to scan Dubbo Component: @org.apache.dubbo.config.annotation.Service
dubbo.scan.base-packages=com.billy.service

# Dubbo Protocol 生产者暴露给消费者协议
dubbo.protocol.name=dubbo
# 生产者暴露给消费者端口
dubbo.protocol.port=20880

## Dubbo Registry 注册中心
dubbo.registry.address=zookeeper://xxxxx:2181
```

* 2.4、 **consumer 项目编写**
1. `pom.xml` 文件中引入接口依赖，同 2.3.1

2. 编写测试类 HelloController,如下：
```java
@RestController
public class HelloController {

    @Reference
    private HelloService helloService;

    @RequestMapping("hello")
    public String hello(String name) {

        return helloService.sayHello(name);
    }
}

```

3. 编写 `application.properties`.

 ```properties
spring.application.name=dubbo-consumer
server.port=8660

# 当前dubbo应用ID
dubbo.application.id=dubbo-consumer
# 当前dubbo应用名称
dubbo.application.name=dubbo-consumer
# 注册地址
dubbo.registry.address=zookeeper://xxxx:2181

# 生产者提供的协议ID
dubbo.protocol.id = dubbo
# 生产者提供的协议名称
dubbo.protocol.name = dubbo
# 生产者提供的协议端口号
dubbo.protocol.port = 20880
 ```
### 3、测试结果
![test result](https://github.com/BillyYangOne/dubbo-springboot/blob/master/dubbo-interface/src/main/resources/image/test_result.png)
