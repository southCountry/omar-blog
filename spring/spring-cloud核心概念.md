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