# 在Centos构建docker的基础环境

在Ubuntu\(6.9\)为基础系统环境构建docker-ce的基础环境，根据具体步骤构建出本文。

### Centos6.9_下构建docker环境_

#### 移除原有的docker

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

#### _安装docker-ce前的准备_

> 1. 安装yum源

```
yum -y install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

#### _安装docker_

```
yum install docker-io
```

#### 重启docker

```
# 重启docker
service docker restart
# 查看docker 版本
docker version
```

#### _升级docker-ce_

```
sudo apt-get upgrade docker-engine
```

### 易用性配置

> 使用docker时取消sudo命令

```
sudo gpasswd -a username docker  
sudo service docker restart
```

## 安装Docker Compose {#step-2-—-installing-docker-compose}

> 安装前准备工作

```
# 安装所有的开发工具包
yum groupinstall -y "Development tools"

# 下载python2.7.15
wget https://www.python.org/ftp/python/2.7.15/Python-2.7.15.tar.xz

#　安装必须包
yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel

# 进入Python源代码目录
cd Python-2.7.15
# 配置检查项，生成makefile
./configure --prefix=/usr/local

# 编译
make && make install
```

> 查看Python2.7.15安装结果

```
[root@xxxx Python-2.7.15]# ll -tr /usr/local/bin/python*
-rwxr-xr-x 1 root root 6283209 6月  26 19:32 /usr/local/bin/python2.7
-rwxr-xr-x 1 root root    1687 6月  26 19:32 /usr/local/bin/python2.7-config
lrwxrwxrwx 1 root root       7 6月  26 19:32 /usr/local/bin/python -> python2
lrwxrwxrwx 1 root root       9 6月  26 19:32 /usr/local/bin/python2 -> python2.7
lrwxrwxrwx 1 root root      16 6月  26 19:32 /usr/local/bin/python2-config -> python2.7-config
lrwxrwxrwx 1 root root      14 6月  26 19:32 /usr/local/bin/python-config -> python2-config
```

> 查看系统自带的Python2.6

```
[root@xxxx Python-2.7.15]# ll -tr /usr/bin/python*
-rwxr-xr-x  1 root root 1418 8月  18 2016 /usr/bin/python2.6-config
-rwxr-xr-x. 2 root root 4864 8月  18 2016 /usr/bin/python2.6
-rwxr-xr-x. 2 root root 4864 8月  18 2016 /usr/bin/python
lrwxrwxrwx. 1 root root    6 3月  27 04:52 /usr/bin/python2 -> python
lrwxrwxrwx  1 root root   16 3月  27 04:56 /usr/bin/python-config -> python2.6-config

```

### 阿里云docker加速

```
https://dev.aliyun.com/search.html
```

进入docker的目录/etc/docker/

```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://nactxuae.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

容器的批零操作命令

```
＃stop停止所有容器
docker stop $(docker ps -a -q) 
# remove删除所有容器
docker  rm $(docker ps -a -q)
# 删除所有id为<None>的image
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
# remove删除全部image
docker rmi $(docker images -q)
```



