# 使用Ubuntu16.04\(64位\)构建Harbor私有docker仓库

### 参考：[https://www.jianshu.com/p/95191c4eed92](https://www.jianshu.com/p/95191c4eed92)

### https://www.jianshu.com/p/79eebbcc5da0

## 一、基础环境介绍

```
操作系统：ubuntu16.04 64位
主机：1核2G40G系统盘
```

## 二、安装前准备

#### 1.下载Harbor安装文件

```
＃在线安装包1.4.0版本
wget https://storage.googleapis.com/harbor-releases/release-1.4.0/harbor-online-installer-v1.4.0.tgz
```

#### 2.解压缩Harbor文件

```
＃解压缩文件
tar -xzvf harbor-online-installer-v1.4.0.tgz
```

#### 3.配置Harbor.conf文件

cd 进入harbor目录,会看见harbor.conf文件，该文件就是Harbor的配置文件。在里面配置主机名称（尽量公网IP）地址。

```
＃<公网ＩＰ地址>不能使用127.0.0.1或者localhost
配置hostname=47.94.149.205
```

## 三、安装Harbor

#### 1.安装Harbor

在harbor目录下执行install.sh脚本，开始从dockerHub上拉取Harbor的镜像

```
./install.sh
```

#### 2.运行Harbor

在浏览器的地址栏中输入　[http://47.94.149.205](http://47.94.149.205)　（你配置好的主机名称）

```
＃默认用户：admin
＃密码：Harbor12345
```

## 四、配置私有仓库地址

由于我们搭建的仓库仅提供了http服务，而docker默认采用https。所以需要在所有使用的docker主机上将这个harbor注册到 daemon.json中，并重启docker。

#### 1.注册私有仓库地址到本机docker中。（所有访问harbor的docker都需要进行注册，避免无法登录访问）

```
root@xxxx:/etc/docker# cat daemon.json 
{
  "registry-mirrors": ["https://nactxuae.mirror.aliyuncs.com"],
  "insecure-registries": ["47.94.149.205"]
}
```

#### 2.重启docker

```
systemctl restart docker.service
```

#### 3.终端登录harbor私有仓库

```
root@xxxx:/etc/docker# docker login 47.94.149.205
Username: admin
Password: 
Login Succeeded

```

## 五、harbor仓库web配置



配置docker私有库

```
可能会出现无法push镜像到私有仓库的问题。这是因为我们启动的registry服务不是安全可信赖的。
这是我们需要修改docker的配置文件/etc/default/docker，添加下面的内容，

修改Docker配置文件
vim /etc/default/docker
增加以下一行
DOCKER_OPTS="$DOCKER_OPTS --insecure-registry=47.95.2.44:5000"
重启Docker
sudo service docker restart

$ sudo service docker restart

ExecStart=/usr/bin/dockerd --insecure-registry=47.95.2.44




OPTIONS='--selinux-enabled=false --log-driver=journald --insecure-registry=192.168.3.42:8060'

ADD_REGISTRY='--add-registry 47.95.2.44'
INSECURE_REGISTRY='--insecure-registry=47.95.2.44'
```



