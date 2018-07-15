# 在Centos6.9构建docker的基础环境

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

### 易用性配置

> 使用docker时取消sudo命令

```
sudo gpasswd -a username docker  
sudo service docker restart
```

## 安装Python2.7.15 {#step-2-—-installing-docker-compose}

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

> 更新系统默认 Python 版本

```
# 先把系统默认的旧版 Python 重命名
mv /usr/bin/python /usr/bin/python.old
# 再删除系统默认的 python-config 软链接
rm -f /usr/bin/python-config
# 最后创建新版本的 Python 软链接
ln -s /usr/local/bin/python /usr/bin/python
ln -s /usr/local/bin/python-config /usr/bin/python-config
```

> 最后查看

```
[root@xxxx Python-2.7.15]# ll -tr /usr/bin/python*
-rwxr-xr-x  1 root root 1418 8月  18 2016 /usr/bin/python2.6-config
-rwxr-xr-x. 2 root root 4864 8月  18 2016 /usr/bin/python.old
-rwxr-xr-x. 2 root root 4864 8月  18 2016 /usr/bin/python2.6
lrwxrwxrwx. 1 root root    6 3月  27 04:52 /usr/bin/python2 -> python
lrwxrwxrwx  1 root root   21 6月  26 19:48 /usr/bin/python -> /usr/local/bin/python
lrwxrwxrwx  1 root root   28 6月  26 19:49 /usr/bin/python-config -> /usr/local/bin/python-config
```

> 查看python版本

```
[root@xxxx Python-2.7.15]# python --version
Python 2.7.15
```

> 为新版 Python 安装 setuptools

```
wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

> 为新版 Python 安装 pip

```
easy_install pip
```

> 为新版 Python 安装 distribute 包（可选）

```
pip install distribute
```

* 注意：升级 Python 可能会导致 yum 命令不可用。解决方法如下：

  * 编辑 /usr/bin/yum 文件，将开头第一行的

```
#!/usr/bin/python
改为
#!/usr/bin/python2.6
```

## 安装docker-compose

```
　# 制定安装1.5.2对应docker 1.7.1
 pip install docker-compose==1.5.2
```

> V2 版本的 docker-compose.yml

```
version: "2"
services:
  redis:
    image: "redis:alpine"
    ports:
      - "6389:6379"
    volumes:
      - "./data/redis/data:/data"
  mysql:
    build: ./builds/mysql
    ports:
      - "3386:3306"
    volumes:
      - "./data/mysql/data:/var/lib/mysql"
      - "./data/mysql/conf:/etc/mysql/conf.d"
    restart: always   
    environment:
      MYSQL_DATABASE: solar
      MYSQL_USER: root
      MYSQL_PASSWORD: Huofigo2015
      MYSQL_ROOT_PASSWORD: Huofigo2015
  api:
    depends_on:
      - mysql
      - redis
    build:
      context: ./builds/api
    ports:
      - "8388:8080"
    volumes:
      - "./target/solar-app-0.0.1-SNAPSHOT.jar:/app/solar-app.jar"
    entrypoint:
      - "java"
      - "-jar"
      - "/app/solar-app.jar"
    restart: always
```

> V1 版本的 docker-compose.yml

```
redis:
  image: "redis:alpine"
  ports:
    - "6389:6379"
  volumes:
    - "./data/redis/data:/data"
mysql:
  build: ./builds/mysql
  ports:
    - "3386:3306"
  volumes:
    - "./data/mysql/data:/var/lib/mysql"
    - "./data/mysql/conf:/etc/mysql/conf.d"
  restart: always
  environment:
    MYSQL_DATABASE: solar
    MYSQL_USER: root
    MYSQL_PASSWORD: Huofigo2015
    MYSQL_ROOT_PASSWORD: Huofigo2015
api:
  build: ./builds/api
  ports:
    - "8388:8080"
  volumes:
    - "./target/solar-app-0.0.1-SNAPSHOT.jar:/app/solar-app.jar"
  links:
    - mysql
    - redis
  entrypoint:
    - "java"
    - "-jar"
    - "/app/solar-app.jar"
  restart: always
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
service docker restart
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



