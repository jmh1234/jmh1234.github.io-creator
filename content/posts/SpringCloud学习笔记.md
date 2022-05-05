---
title: "SpringCloud学习笔记"
date: 2022-05-05T09:31:42+08:00
draft: false
categories: ["Java", "SpringCloud"]
tags: ["Java", "SpringCloud"]
author: 小叽
---

# 微服务 #
1. 什么是微服务

   微服务是一种构建应用的架构方案。微服务架构有别于更为传统的单体式方案，可将应用拆分成多个核心功能。每个功能都被称为一项服务，可以单独构建和部署。
2. 分布式部署

3. 微服务应用之间通信
  restful api
RestTemplate.postForProject
            .getForProject

4. 优缺点
 good：
  1. 易于开发和维护：一个微服务只会关注一个特定的业务功能，所以业务清晰、代码量较少。开发和 维护单个微服务相对简单。 
  2. 单个微服务启动较快 
  3. 局部修改容易部署：单一应用只要有修改，就得重新部署整个应用。微服务解决了这样的问题。一 般来说，对某个微服务进行修改，只             需要重新部署这个服务即可。 
  4. 技术栈不受限制：在微服务架构中，可以结合项目业务及团队的特点，合理的选择技术栈。
  5. 按需伸缩：更易扩展程序，添加新的功能
 bad: 
  1. 运维要求高
  2. 分布式固有的复杂性：使用微服务构建的是分布式系统。对于一个分布式系统，系统容错、网络延 迟、分布式事务等都会带来巨大的问 题。 
  3. 接口调整成本高：微服务之间通过接口进行通信。如果修改某一个微服务的API，可能所有用到这 个接口的微服务都需要进行调整。


# springCloud #
1. 技术栈
   服务开发                 Spring Boot、Spring、Spring MVC等
   服务注册与发现           Eureka、Zookeeper等
   服务调用                 Rest、RPC等
   服务熔断器               Hystrix、Envoy等
   负载均衡                 Ribbon、Nginx等
   服务接口调用(客户端调用服务的简化工具)     Feign等
   消息队列                 Kafka、ActiveMQ等
   服务配置中心管理         Spring Cloud Conﬁg等
   服务路由（API网关）      Zuul等
   服务监控                 Zabbix、Nagios等
   全链路追踪               Zipkin，Brave等
   服务部署                 Docker、OpenStack等
   数据流处理               Spring Cloud Stream（Redis,Rabbit,Kafka等发送接收消息）
   事件消息总线             Spring Cloud Bus
2. 什么是springCloud
   基于springBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型 中立的开源组件。 


# eurake #
eurake显示status 为instanceID

Eureka心跳健康检查机制：
Eureka服务器向客户端公开下面资源以让其发送心跳：
PUT /eureka/apps/{app id}/{instance id}?status={status} 

{instance id}采用  hostname:app id:port，其中app id代表标识唯一的Eureka客户端实例，Eureka服务器会识别一些状态数值：UP; DOWN; STARTING; OUT_OF_SERVICE; UNKNOWN.

自我保护机制：
https://www.jdon.com/springcloud/eureka-self-preservation.html
在短时间内丢失过多客户端
当超过15%的应用未发送心跳，会启动自我保护机制，再eurake服务器中的未响应应用不会立即删除，依旧会对该微服务的信息进行保存

自我保护的优点
1.没有接收心跳的服务器可能是由于网络分区，接受分区，保留暂时的不一致，提高可用性。
2.即使服务器和某些客户端之间的连接丢失，客户端也可能相互连接。即，在网络分区期间，实例2还是具有与实例4的连接。

为什么要使用集群的eurake
负载均衡 + 故障容错
易于维护，维护单独一台服务器时，不会影响其他服务器，整个应用不会宕机

为什么使用eurake
1.eurake来自生产环境（netflix开发）
2.支持完善


eruake原理


# Ribbon #
1. 什么是负载均衡（Load Balance）
 负载均衡是我们处理高并发，缓解网络压力和进行服务端扩容的重要手段之一，将用户的请求平摊到这个服务器上，从而实现系统的高可用性。

2. 客户端负载均衡和服务端负载均衡：  @LoadBalanced
客户端的负载均衡实在springCloud分布式框架组件Ribbon中定义的。
我们在使用springCloud分布式框架时，同一个sercer大概率会启动多个，当一个请求奔过来时，那么这多个service，Ribbon通过策略决定本次请求使用哪个service的方式就是客户端负载均衡

ribbon提供的负载均衡算法
- 随机 RandomRule
- 轮询 RoundRobinRule
- AailabilityFilteringRule
  会过滤掉由于多次访问故障而处于断路器跳闸状态的服务，还有病并发的连接数量超过阀值的服务，然后对剩余的服务列表按照轮询策略进行访问
- WeightdResponseTimeRue
  根据平均响应时间计算所有服务的权重，响应速度越快权重越大被选中的几率越高，刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够，会切换到AailabilityFilteringRule
- RetryRule
  先按照RoundRobinRule的策略获取服务，如果获取服务失败则在制定的时间内进行重试，获取可用的服务。（刚开始会有几次访问到不可用的服务，后面会跳过这个不可用的服务。）
- BestAvailableRule
  会过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量小的服务
- ZoneAvoidanceRule
  默认规则，复合判断server所在区域的性能和server的可用性选择服务器。

切换规则
    @Bean
    public IRule myRule(){
        return new RoundRobinRule();
    }

可以自定义一个规则


ngnix ？？？？？？？
# Feign # 

微服务之前互相调用，不再使用http通讯的restTemplate，可以简化调用流程，真正能感觉到是微服务之前的调用。

Feign通过接口的方法调用Rest服务（之前是Ribbon+RestTemplate），请求发送给 Eureka 服务器（http://MICR OSERVICE-PRODUCT/product/list）, 通过Feign直接找到服务接口 ，因为集成了 Ribbon 技术，Feign 自带负载均 衡配置功能。

@FeignClient(value = "microservice-product" ,fallback = ProductClientServiceHystrix.class) //指定调用的微服务名称

注意事项：
1. @FeignClient接口方法有基本类型参数在参数必须加@PathVariable("XXX") 或 @RequestParam("XXX") 
2. @FeignClient接口方法返回值为复杂对象时，此类型必须有无参构造方法。 

原理：

# Hystrix # 
