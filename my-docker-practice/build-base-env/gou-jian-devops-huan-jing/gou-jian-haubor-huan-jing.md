### 参考：[https://www.jianshu.com/p/95191c4eed92](https://www.jianshu.com/p/95191c4eed92)

### 下载Harbor安装文件

```
＃在线安装包1.4.0版本
https://storage.googleapis.com/harbor-releases/release-1.4.0/harbor-online-installer-v1.4.0.tgz
tar -xzvf harbor-online-installer-v1.4.0.tgz
```

### 配置Harbor

解压缩之后，目录下回生成harbor.conf文件，该文件就是Harbor的配置文件。

```
配置hostname=<公网ＩＰ地址>47.95.2.44
```

运行Harbor

```
浏览器登陆Harbor[http://47.95.2.44]（默认用户密码：admin/Harbor12345）
```

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



