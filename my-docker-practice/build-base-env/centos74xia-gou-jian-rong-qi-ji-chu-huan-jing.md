# 在Centos7.4构建docker的基础环境

在Ubuntu\(7.4\)为基础系统环境构建docker-ce的基础环境，根据具体步骤构建出本文。

### Centos7.4_下构建docker环境_

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

> 1. 安装依赖

```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

> 阿里云源

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> 国外官方源

```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

> 重建软件源缓存

```
sudo yum makecache fast
```

#### _安装docker_

```
sudo yum install docker-ce
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

## 安装docker-compose

```
 pip install docker-compose
```

### 阿里云docker加速

```
https://dev.aliyun.com/search.html
```

进入docker的目录/etc/docker/

```
# 配置阿里云docker加速
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://nactxuae.mirror.aliyuncs.com"],
"insecure-registries": ["47.95.2.44"]
}
EOF

# 重启docker
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



