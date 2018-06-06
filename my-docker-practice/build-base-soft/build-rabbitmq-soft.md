RabbitMQ是一个在AMQP基础上完成的，可复用的企业消息系统。  
因为RabbitMQ由Erlang实现，本机部署的话还要安装Erlang的开发环境，成本难免高些。然而，借助Docker的话，环境部署便会非常便捷。这次来使用docker快速搭建带web管理功能的RabbitMQ的环境。

# 查找镜像 {#查找镜像}

通过dockerhub搜索，可以找到[官方的RabbitMQ镜像](https://hub.docker.com/r/_/rabbitmq/)。  
在网页的[tag标签页](https://hub.docker.com/r/library/rabbitmq/tags/)下会列出所有可用的tag。  
当我们使用命令：

```
docker pull rabbitmq:3.7.5
```

默认使用的RabbitMQ最新的新镜像。

目前最新的是3.7.5，通过查看他的

[rabbitmq/3.6/debian/Dockerfile](https://github.com/docker-library/rabbitmq/blob/28001b529f28ed0d8e8297f8b603a4cc93a846a3/3.6/debian/Dockerfile)

生成文件，可以发现并没有我们需要的rabbitmq\_management。

所以，需要去tag查找下，带rabbitmq\_management功能的tag是什么。通过查找，使用的tag是management，或者版本号-3.7.5-management。

当我们使用命令：

```
docker pull rabbitmq:3.7.5-management
```

# 创建镜像 {#创建镜像}

创建容器使用如下命令：

```
docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
```

命令很简单：run创建容器，-d后台运行，–name命名容器为rabbitmq，-p将容器内端口映射到本机。  
至于为什么要映射这些端口，可以通过查看[rabbitmq:management的Dockerfile](https://github.com/docker-library/rabbitmq/blob/b9eda3e4665c24db70a9a290fddf33bc5c567b10/3.6/debian/management/Dockerfile)文件找到原因。  
首先，rabbitmq:management的Dockerfile最后指出：

> EXPOSE 15671 15672

所以，web管理服务最终使用容器内的这两个端口。  
其次，rabbitmq:management的Dockerfile开始的时候指明：

> FROM rabbitmq:3.7.5

所以rabbitmq:management的Dockerfile是基于rabbitmq镜像创建的，[rabbitmq的Dockerfile文件](https://github.com/docker-library/rabbitmq/blob/b9eda3e4665c24db70a9a290fddf33bc5c567b10/3.6/debian/management/Dockerfile)最后定义了：

> EXPOSE 4369 5671 5672 25672

所以，容器使用的所有端口就明确了。

成功创建容器后，就可以访问web 管理端了[http://127.0.0.1:15672](http://127.0.0.1:15672/)，默认创建了一个 guest 用户，密码也是 guest。



# 创建镜像（用户名和密码） {#创建镜像}

创建容器使用如下命令：

```
docker run -d --hostname rabbit --name rabbit -e RABBITMQ_DEFAULT_USER=rabbitadmin -e RABBITMQ_DEFAULT_PASS=rabbitpwd -p 15672:15672 rabbitmq:management
```









