# SpringCloud Finchley 三版本微服务实战

## 第1章 课程介绍

###  1-1 SpringCloud导学

* Eurake
  * Eurake Server
  * Eurake Client
  * Eurake高可用
  * 服务发现机制
* Config
  * Config Server
  * Config Client
  * Spring Cloud Bus（结合RabbitMQ自动刷新）
* Ribbon
  * RestTemplate
  * Feign
  * Ribbon（分析源码，了解底层）
* Zuul
  * 动态路由
  * 校验
* Hystrix
  * Hystrix Dashboar
  * 熔断机制
* 容器编排和服务追踪
  * docker + rancher
  * Spring clould sleuth + zipkin

## 第2章 微服务介绍

### 2-1 微服务和其他常见架构

![](E:\markdown笔记\笔记图片\21\1.png)

![2](E:\markdown笔记\笔记图片\21\2.png)

![3](E:\markdown笔记\笔记图片\21\3.png)

![4](E:\markdown笔记\笔记图片\21\4.png)

![5](E:\markdown笔记\笔记图片\21\5.png)

* 单体架构的优点：
  * 容易测试
  * 容易部署
* 单体架构的缺点
  * 开发效率低
  * 代码维护难
  * 部署不灵活
  * 稳定性不高
  * 扩展性不够
* 参考文章：[《Web 研发模式演变》](https://github.com/lifesinger/blog/issues/184)

![](E:\markdown笔记\笔记图片\21\6.png)

* 集群是个物理形态，分布式是个工作方式。
  - 分布式：一个业务拆分成多个子业务，每个子业务分别部署在不同的服务器上
  - 集群：同一个业务，部署在多个服务器上
* 比喻：厨房里有两个厨师，他们做相同的事情，那这就是集群；如果他们做不同的事情，那这就是分布式

### 2-2 从一个极简的微服务架构开始

![](E:\markdown笔记\笔记图片\21\7.png)

![8](E:\markdown笔记\笔记图片\21\8.png)

![9](E:\markdown笔记\笔记图片\21\9.png)

从上图中我们可以看到，后端通用服务和前端服务是比较臃肿的。如果去治疗这部分肥胖的服务呢

![](E:\markdown笔记\笔记图片\21\10.png)

![11](E:\markdown笔记\笔记图片\21\11.png)

## 第3章 服务注册与发现

### 3-1 Spring Cloud Eureka

![](E:\markdown笔记\笔记图片\21\12.png)

### 3-2 Eureka Server

* 项目的搭建详情参见：https://github.com/depers/spring-cloud-sell/tree/master/eureka-server

* 参考文章

  http://cmsblogs.com/?p=13061

  http://blog.didispace.com/spring-cloud-starter-dalston-1/

  https://www.fangzhipeng.com/springcloud/2017/06/01/sc01-eureka.html

  [https://www.extlight.com/2018/07/04/Spring-Cloud-%E5%85%A5%E9%97%A8-%E4%B9%8B--Eureka-%E7%AF%87%EF%BC%88%E4%B8%80%EF%BC%89/

### 3.3 Eureka Client

* 项目搭建的详情参见：https://github.com/depers/spring-cloud-sell/tree/master/eureka-client

## 第8章 服务网关

###  8-1 服务网关和Zuul

1. 服务网关的要素
   * 稳定性，高可用
   * 性能，并发性
   * 安全性
   * 可扩展性
2. 常用的网关方案
   * Nginx+Lua
   * Kong
   * Tyk
   * Spring Cloud Zuul

3. Zuul的特点
   * 路由+过滤器=Zuul
   * 核心就是一系列的过滤器

4. Zuul的四种过滤器
   * 前置（Pre）
   * 路由（Route）
   * 后置（Post）
   * 错误（Rrror）

![](E:\markdown笔记\笔记图片\21\16.png)

在上图中，微信点餐项目使用nginx做负载均衡和反向代理，这点不变，我们可以使用Zuul代替Tomcat。

![17](E:\markdown笔记\笔记图片\21\17.png)

Zuul组件的架构图。

![18](E:\markdown笔记\笔记图片\21\18.png)

Zuul中一个请求的生命周期，其中除了四种过滤器之外。还有一个custom filters，这个是我们自定义的过滤器。

### 8-2 Zuul：路由转发，排除和自定义

1. 路由转发
2. 自定义路由
3. 禁止路由访问

### 8-3 Zuul：Cookie和动态路由

###  8-4 Zuul：路由和高可用小结

典型应用场景

* 前置(Pre)
  * 限流
  * 简权
  * 参数校验调整
* 后置(Post)
  * 统计
  * 日志
* Zuul的高可用
  * 多个Zuul节点注册到Eureka Server
  * Nginx和Zuul的“混搭”