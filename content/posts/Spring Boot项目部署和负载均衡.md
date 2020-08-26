---
title: "Spring Boot项目部署以及负载均衡"
date: 2020-08-21T14:07:45+08:00
draft: false
author: 小叽
---

本文将介绍Spring Boot项目的三种部署方式以及使用Nginx作为负载均衡器。

<!--more-->
# 集群、负载均衡、分布式的区别与联系
## 集群
集群是指同一种组件的多个实例形成的逻辑上的整体。即同一个系统，部署在多台服务器上，将多个服务器集中起来同时进行一种服务，
在客户端看来就像是只有一个服务器，而且其中的一个节点挂了不影响其他节点的对外提供服务。

## 分布式
分布式是指通过网络连接的多个组件，为了完成共同的任务而通过交换信息协作的计算机节点组成的系统。
将一个大的业务拆分成多个小的业务分别部署到不同的服务器上。其目的是利用更多的机器，处理更多的数据。

## 负载均衡
负载均衡是指将请求分摊到多个操作单元也就是分开部署的服务器上，Nginx是常用的反向代理服务器，可以用来做负载均衡。

## 负载均衡集群
负载均衡集群为企业需求提供了更实用的系统。如名称所暗示，该系统使负载(流量)可以在计算机集群中尽可能平均分摊处理，这样的系统非常适合于运行同一组应用程序的大量用户。
每个节点都可以处理一部分负载，并且可以在节点之间动态分配负载，以实现平衡，对于网络流量也是如此。
通常，网络服务器应用程序接受了太多入网流量，以致无法迅速处理，这就需要将流量发送给在其它节点上运行的网络服务器应用。
还可以根据每个节点上不同的可用资源或网络的特殊环境来进行优化。

# Spring Boot项目的部署
本次部署的Spring Boot项目中涉及到的mysql数据库以及redis缓存都在docker容器中，所以首先要将它们在docker中启动。
1. 启动[docker mysql](https://hub.docker.com/_/mysql/)数据库：`docker run --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=yourPassword -d mysql`
2. 启动[docker redis](https://hub.docker.com/_/redis/)缓存：`docker run -p 6379:6379 -d redis`
3. 使用flyway进行数据迁移：`mvn flyway:migrate`

## maven插件exec部署
pom.xml文件配置：
````
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <executable>java</executable>
        <arguments>
            <argument>-classpath</argument>
            <!-- automatically creates the classpath using all project dependencies,
                 also adding the project build directory -->
            <classpath/>
            <argument>com.demo.test.Application</argument> <!--Spring Boot 项目启动的入口文件-->
        </arguments>
    </configuration>
</plugin> <!--maven exec plugin-->
````

1. 在pom文件中引入上述配置
2. 在控制台窗口执行`mvn exec:exec`即可部署该项目。
3. 或者在`cmd`窗口中奖路径设置到当前项目的根路径下后执行命令`mvn exec:exec`同样可以部署该项目。
4. 该项目的默认端口号为8080

## jar包部署Spring Boot项目
### 生成Jar文件
pom.xml文件配置：
````
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin><!--打jar包-->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin><!--打jar包-->
````

在pom文件中引入上述配置后在控制台窗口执行`mvn package`获取该项目的jar文件，该文件位于target目录下。

### 部署
1. 在控制台窗口将路径设置到target下，执行命令`java -Dserver.port=8081 -jar spring-boot-project-0.0.1.jar`即可部署该项目。
2. 或者在`cmd`窗口中将路径设置到当前项目的target目录下，执行命令`java -Dserver.port=8081 -jar spring-boot-project-0.0.1.jar`同样可以部署该项目。
3. -Dserver.port=8081 指令为了指定额外的端口号为8081(8080端口已经被前一个部署的项目占用)。

## docker 部署
dockerfile文件内容
````
FROM java:openjdk-8u111-alpine

RUN mkdir /app

WORKDIR /app

COPY target/spring-boot-project-0.0.1.jar /app

EXPOSE 8080

CMD ["java","-jar","spring-boot-project-0.0.1.jar"]

````

1. 配置好dockerfile文件后通过`docker build .`将该项目制作成images。
2. 通过`docker images` 查看当前容器中的镜像，找到刚刚制作的镜像的ID。
3. 通过`docker run --restart=always --name=mySpringBootApp -p 8082:8080 -d <镜像id>` 启动该镜像
ps：docker可以看做是一台独立的计算机要时刻注意ip地址的问题。

## Nginx流量转发

docker nginx官网：[https://hub.docker.com/_/nginx/](https://hub.docker.com/_/nginx/)  

外部 nginx.conf 文件配置:
````
events{ }

http {
    upstream myapp1 {
        server 192.168.1.164:8080;
        server 192.168.1.164:8081;
        server 192.168.99.100:8082;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
````

1. 其中192.168.1.164路由器分配给为宿主机的ip地址，192.168.99.100为docker容器的地址。
2. Nginx监听80端口，将流量均匀的转发给myapp1中的三个服务。其中8080通过exec插件部署、8081通过jar文件方式部署、8082通过docker部署。
3. 启动Nginx：docker run -v "$(pwd)"/nginx.conf:/etc/nginx/nginx.conf:ro -p 80:80 -d nginx(当前的文件路径(pwd)为C:\Users\用户\tmp)
4. Nginx启动成功后就可以通过 192.168.99.100 + 后台服务接口名称访问项目
