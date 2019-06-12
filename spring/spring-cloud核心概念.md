## 简介
spring cloud是一个基于spring boot的微服务开发工具集。

## 服务治理：spring cloud eureka
服务治理主要用来实现各个微服务实例的自动化注册与发现。

dubbo也内置了服务自动化注册与发现功能。用户可以在dubbo配置文件中指定注册中心，一般为zk集群。

### 搭建服务注册中心
1. 使用@EnableEurekaServer注解指定当前进程为服务注册中心。
2. 可以在application.properties中修改注册中心相关配置。

### 高可用注册中心
1. 启动多个注册中心实例即可

### 注册服务提供者
1. 在主类中通过加上@EnableDiscoveryClient注解，激活DiscoverClient的实现
2. 在application.properties中指定注册中心，使用restTemplate实现对服务提供者的调用。

### 注册服务消费者
1. 在主类中通过加上@EnableDiscoveryClient注解，让该应用注册为Eurka客户端应用，以获得服务发现能力。

**因为大部分公司的dubbo是有一个独立的团队在维护，因此，对于注册中心的操作比较少。spring cloud基于rest接口的方式实现服务注册和发现，不需要显示依赖服务接口，因此服务升级较为方便。dubbo服务是基于rpc代理，使用和理解起来都更加直观。**

## 客户端负载均衡：spring cloud ribbon
使用@LoadBalance注解注释RestTemplate，然后调用RestTemplate的方法即可自动实现客户端负载均衡。

## 服务容错保护 spring cloud hystrix

## 声明式服务调用 spring cloud feigh
1. 使用@EnableFeignClients注解注释Application主类。
2. 定义service接口，使用@FeignClient注解注释该接口

## API网关服务 spring cloud zuul
1. 解决路由规则以及服务实例维护问题
2. 解决签名校验、登录校验在微服务架构中的冗余问题

## 分布式配置中心 spring cloud config
1. 使用@EnableConfigServer注解注释Application主类
2. 在application.properties中添加配置中心的基本信息

## 消息驱动的微服务 spring cloud stream

## 分布式服务跟踪 spring cloud sleuth