# 构建容器之jenkins环境

### 安装jenkins

启动一个jenkins容器

docker run -d --name jenkins -p 18081:8080 -v /home/incps/devsoft/dockerWork/jenkins\_home:/home/jenkins\_home jenkins

查看jenkins服务   docker ps \| grep jenkins

启动服务端 。localhost:8081/jenkins

去容器内部找密码

进入容器：docker exec -it jenkins bash 

执行：cat /var/jenkins\_home/secrets/initialAdminPassword  红框即为jenkins初始密码

输入密码之后，重启docker镜像  docker restart   {CONTAINER ID}

### 配置Jenkins

参见　https://blog.csdn.net/m0\_37106742/article/details/78815753



