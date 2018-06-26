# 构建docker的基础环境

在Ubuntu14.04为基础系统环境构建docker-ce的基础环境，根据具体步骤构建出本文。

### _Ubuntu14.04下构建docker-ce环境_

#### 移除原有的docker

```
sudo apt-get remove docker docker-engine docker.io
```

#### _安装docker-ce前的准备_

> 1. 更新系统补丁，安装必备支撑软件

```
sudo apt-get update  
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```

> 1. 添加官方PGP秘钥

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

> 1. 添加apt源

```
＃
sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
```

#### _安装docker-ce_

```
sudo apt-get update  
sudo apt-get install docker-ce
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

```
sudo apt-get -y install python-pip

sudo pip -y install docker-compose
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



