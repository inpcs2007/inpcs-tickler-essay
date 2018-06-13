* eureka的高可用

java -jar microservice-discovery-eureka-ha-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer1

java -jar microservice-discovery-eureka-ha-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer2

* 在集成rabbin（软件的负载均衡）后，运行多个实例（相同实例名称不同的端口），在eureka注册中心上就会看到。

使用LoadBalanceClient中会轮训调用不同的实例。

将请求改成\[[http://microservice-provider-user/+id\]的形式（是用户服务的虚拟主机名）](http://microservice-provider-user/+id]的形式（是用户服务的虚拟主机名）)

当Ribbon和Eureka配合使用的时候，会自动讲虚拟主机名称映射成微服务的网络地址

# SpringCloud进阶教程

 \* 1.注册中心Eureka

1. Spring Cloud之一 Eureka简介及原理　[http://itmuch.com/spring-cloud-1/](http://itmuch.com/spring-cloud-1/)

2. Spring Cloud之二 创建一个Eureka Server [http://itmuch.com/spring-cloud-2/](http://itmuch.com/spring-cloud-2/)

3. Spring Cloud之三 创建一个Eureka Server集群　[http://www.cnblogs.com/ityouknow/p/6854805.html](http://www.cnblogs.com/ityouknow/p/6854805.html)

4. Spring Cloud之四 创建一个Eureka Server集群[http://itmuch.com/spring-cloud-sum-eureka/](http://itmuch.com/spring-cloud-sum-eureka/)



