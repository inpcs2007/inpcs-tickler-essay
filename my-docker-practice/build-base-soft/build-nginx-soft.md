
# 基于docker的nginx的使用

## 1.　安装nginx
```
docker pull nginx
```
## 2. 运行nginx
### 2.1 运行nginx　并挂接外部的配置文件及目录
```
docker run -p 8999:80 --name mynginx -v $PWD/www:/www -v $PWD/conf/conf.d:/etc/nginx/conf.d -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf -v $PWD/logs:/wwwlogs -d nginx
```

### 2.2 宿主机操作容器
 docker cp 容器名：要拷贝的文件在容器里面的路径       要拷贝到宿主机的相应路径
```
 docker cp testtomcat：/usr/local/tomcat/webapps/test/js/test.js /opt
```
### 2.３ 容器推送到宿主机
docker cp 要拷贝的文件路径 容器名：要拷贝到容器里面对应的路径
```
docker cp /opt/test.js testtomcat：/usr/local/tomcat/webapps/test/js
```
### 2.4 如何获取 docker 容器(container)的 ip 地址
```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID> 
```