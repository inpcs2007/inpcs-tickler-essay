# 构建容器之jenkins环境

### 安装jenkins

启动一个jenkins容器

docker run -d --name jenkins -p 18081:8080 -v /home/incps/devsoft/dockerWork/jenkins\_home:/home/jenkins\_home jenkins

查看jenkins服务   docker ps \| grep jenkins

启动服务端 。localhost:8081/jenkins

去容器内部找密码

进入容器：docker exec -it jenkins bash

执行：cat /var/jenkins\_home/secrets/initialAdminPassword  红框即为jenkins初始密码

输入密码之后，重启docker镜像  docker restart   {CONTAINER ID}

### 配置Jenkins

参见　[https://blog.csdn.net/m0\_37106742/article/details/78815753](https://blog.csdn.net/m0_37106742/article/details/78815753)

打开浏览器，最好使用谷歌，或者火狐。在地址栏输入[http://IP:8080/jenkins](http://IP:8080/jenkins)

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601153503805-1152040019.png)

这里需要输入密码，密码按照提示寻找，正常情况下应该是在：/root/.jenkins/secrets/initialAdminPassword

输完密码点击 Continue 按钮进行下一步![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601154122930-79347306.png)

左边是默认安装，右边是自定义安装，本地实验不需要特殊需求，所以选择默认安装了。

附张正在装的图，页面还是蛮精美的

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601154429446-1590176109.png)

如果无法安装，就是你的网络有问题，自己先把网络搞定了在说吧！

安装完成后输入自己的用户名，密码等个人信息。然后Save and Finish一下，之后就可以启动Jenkins了

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601155929524-305250595.png)

至此Jenkins 安装完成。

### 创建一个项目

先配置一下Jenkins 

　　　　首页–&gt; 系统管理 –&gt;[Global Tool Configuration](http://118.89.192.184:8080/jenkins/configureTools) –&gt; 新增“jdk Maven Docker”　　\#注意：是三个，不是一个，如果路径不对，会有报错的

　　　　最后不要忘记，Save下　　　　\# 使用svn的小伙伴注意了：“如果没有新增svn 可以在插件管理选项中安装svn 插件后即可新增svn，这个自行百度。”

　　3.1、创建一个新任务

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170602100647493-1770913656.png)

 点击图片中的 开始创建一个新任务，这里创建一个自由风格的软件项目，并书写上项目名称,点击OK进行下一步操作

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170602110838164-2138881770.png)

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170602111103243-1973307965.png)

**源码管理，我这里使用git的方式来管理项目，我这里去开源中国找一个免费开源项目来进行测试。**

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170602111147118-180996251.png)

　　　　正常的构建项目只需要做好源码管理即可，如需要使用Docker  使用shell即可调用。触发器的作用相当于触发某个条件，就会立即去构建，不必后期的手工构建了。

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170602112230133-848332959.png)

　　　　　　点击类似时钟的图标，点击立即构建，然后他会自动去打包，如果成功，S下面的图标会显示为蓝色，右边为太阳的图标。至此项目构建完成。

      1 Docker 命令集
      2     寻找网络镜像命令
      3         docker search centos
      4             
      5             [root@test ~]# docker search centos
      6                         名字                                             描述                                             下载次数  是否官方   是否是Dockerfile构建的 
      7             INDEX       NAME                                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMAT
      8             docker.io   docker.io/centos                                 The official build of CentOS.                   3301      [OK]       
      9 
     10     下载镜像
     11         docker pull centos
     12     查看镜像
     13         docker images
     14     删除镜像
     15         docker rmi
     16     
     17 容器命令
     18     启动容器
     19         docker run --name -h hostname
     20     启动容器2
     21         docker start CONTAINER ID
     22     停止容器
     23         docker stop CONTAINER ID
     24     查看容器
     25         docker ps -a 
     26     进入容器
     27         docker exec | docker attach
     28     删除容器
     29         docker rm
     30 
     31 进入后台运行容器
     32     docker attach 68e5c66ee5c9        退出自动停止运行容器
     33 
     34 进入容器
     35     docker run --name mydocker -it docker.io/centos /bin/bash
     36         -d        进入后台运行
     37         --run    运行
     38         --name 指定名字
     39         -i        输入终端打开
     40         -t        开一个伪终端
     41         
     42 进入容器不退出
     43 1、进入容器
     44     docker run --name mydocker -it docker.io/centos /bin/bash
     45 2、退出
     46 3、启动容器
     47     docker ps -a查询ID号
     48     docker start ID号
     49 4、获取Pid号
     50     docker inspect --format "{{.State.Pid}}" 68e5c66ee5c9
     51 5、进入容器而不退出
     52     nsenter --target 19205 --mount --uts --ipc --net --pid    退出不停止运行容器
     53     如果没有这个命令，需要安装util-linux包    nsenter
     54 
     55 6、懒人写法
     56     nsenter --target `docker inspect --format "{{.State.Pid}}" ID` --mount --uts --ipc --net --pid    
     57 docker 网络访问
     58     docker run -P 
     59         -P    随机映射
     60         -p    hostport:containerPort
     61         -p    ip:hostPort:containerPort
     62         -p  ip::containerPort
     63         -p  hostPort:containerPort
     64         -p  hostPort:containerPort
     65         
     66 数据卷管理
     67     docker 
     68         -v /data
     69         -v /src:/dsrc
     70         -v /src:/src:ro 
     71 
     72 容器的制作
     73     docker commit -m "my nginx" f443e801f545 shijia/my-nginx:v1
     74 Docker file 的方式构建docker镜像
     75     
     76     FROM         他的妈妈是谁（基础镜像）
     77     MAINTAINER    告诉被人，你创造了他（维护者信息）
     78     RUN            你想让他干啥（把命令前面加上RUN）
     79     ADD         相当于cp命令（COPY文件，会自动解压）
     80     WORKDIR        相当于cd命令（当前工作目录）
     81     VOLUME        给我一个放行李的地方（目录挂载）
     82     EXPOSE        我要打开的门是啥（端口）
     83     RUN            奔跑吧，兄弟！（进程要一直运行下去）
     84     
     85     Docker 案例
     86     vim /opt/docker-file/nginx/Dockerfile
     87     
     88     # This is My first Dockerfile
     89     # Version 1.0
     90     # Author : Shijia Zhang
     91 
     92     #Base images
     93     FROM centos
     94 
     95     #MAINTAINER
     96     MAINTAINER ShiJia Zhang
     97 
     98     #ADD
     99     ADD pcre-8.37.tar.gz    /usr/local/src
    100     ADD nginx-1.9.3.tar.gz  /usr/local/src
    101 
    102     #RUN
    103     RUN yum -y install wget gcc gcc-c++ make openssl-devel
    104     RUN useradd -s /sbin/nologin -M www
    105 
    106     #WORKDIR
    107     WORKDIR /usr/local/src/nginx-1.9.3
    108 
    109     #RUN
    110     RUN ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --with-pcre=/usr/local/src/pcre-8.37 && make && make install
    111 
    112     RUN echo "daemon off;" >> /usr/local/nginx/conf/nginx.conf
    113 
    114     ENV PATH=/usr/local/nginx/sbin:$PATH
    115     EXPOSE 80
    116 
    117     CMD ["nginx"]
    118 运行Dockerfile
    119     docker build -t nginx-file:v1 /opt/docker-file/nginx/
    120 其他命令
    121     docker run -it --rm stree --cpu 1
    122     --rm    容器运行完即可删除
    123     --cpu    指定分多少颗CPU 



