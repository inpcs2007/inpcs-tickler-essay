RabbitMQ是一个在AMQP基础上完成的，可复用的企业消息系统。  
因为RabbitMQ由Erlang实现，本机部署的话还要安装Erlang的开发环境，成本难免高些。然而，借助Docker的话，环境部署便会非常便捷。这次来使用docker快速搭建带web管理功能的RabbitMQ的环境。

# 查找镜像 {#查找镜像}

通过dockerhub搜索，可以找到[官方的RabbitMQ镜像](https://hub.docker.com/r/_/rabbitmq/)。  
在网页的[tag标签页](https://hub.docker.com/r/library/rabbitmq/tags/)下会列出所有可用的tag。  
当我们使用命令：

 docker pull rabbitmq:3.7.5



