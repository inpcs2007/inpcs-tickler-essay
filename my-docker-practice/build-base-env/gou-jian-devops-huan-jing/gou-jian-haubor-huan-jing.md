# 使用Ubuntu16.04\(64位\)构建Harbor私有docker仓库

### 参考：[https://www.jianshu.com/p/95191c4eed92](https://www.jianshu.com/p/2ebadd9a323d)

### 

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
＃
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

#### 1.创建用户

#### 2.创建项目

#### 3.为新项目添加成员

## 六、push镜像到镜像仓库

#### 1.镜像打标签

```
# 基本语法
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
# 样例
docker tag busybox 47.94.149.205/core/busybox
```

#### 2.查看镜像

```
root@xxxx:/dockerWork# docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
47.94.149.205/core/busybox   latest              8c811b4aec35        4 weeks ago         1.15MB
busybox                      latest              8c811b4aec35        4 weeks ago         1.15MB
```

#### 3.登录仓库

```
root@iZ2ze22o1dtiiawzopg0vnZ:/dockerWork# docker login -u inpcs -p xxxxxxxxxx 47.94.149.205
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```

#### 4.push镜像

```
root@xxxx:/dockerWork# docker push 47.94.149.205/inpcs-library/busybox:v2
The push refers to repository [47.94.149.205/inpcs-library/busybox]
432b65032b94: Layer already exists 
v2: digest: sha256:74f634b1bc1bd74535d5209589734efbd44a25f4e2dc96d78784576a3eb5b335 size: 527
```

* 说明：docker push 47.94.149.205/inpcs-library/busybox:v2
  * inpcs-library是在harbor的项目名称
  * 如果需要端口请在47.94.149.205后面添加【:端口】，本文是默认的【80】，所以省略
  * harbor中的管理员可以创建项目，开发人员可以push镜像，游客只能pull镜像。

## 七、harbor的主备镜像仓库

## 八、harbor的主备镜像仓库



