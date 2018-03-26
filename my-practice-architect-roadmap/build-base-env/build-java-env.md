# 构建java的基础环境

在Centos6为基础系统环境构建java的基础环境，根据具体步骤构建出本文。

### 创建用户

> 切换到 root 用户

 ```
     su root
 ```

> 创建 channelmonitor  用户、 用户组、设置密码

```
[root@manage ~]# groupadd -g 1000 channelmonitor
[root@manage ~]# useradd -u 2000 -g channelmonitor channelmonitor
[root@manage ~]# passwd channelmonitor
```

### 设置sudo权限

> 查看/etc/sudoers 文件权限

```
[root@manage ~]# ll /etc/sudoers
-r--r-----. 1 root root 3729 12月  8 2015 /etc/sudoers
[root@manage ~]#
```

> 修改sudoers文件权限

```
[root@manage ~]# ll /etc/sudoers
-r--r-----. 1 root root 3729 12月  8 2015 /etc/sudoers
[root@manage ~]# sudo chmod 775 /etc/sudoers
[root@manage ~]# ll /etc/sudoers
-rwxrwxr-x. 1 root root 3729 12月  8 2015 /etc/sudoers
[root@manage ~]#
```

> 使用快捷执行命令

```
    [root@manage ~]# echo 'channelmonitor ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
    // 内容含义：用户名 channelmonitor 网络中主机=（全部用户）不需要密码：全部范围
```

> 恢复访问权限

```
[root@manage ~]# sudo chmod 400 /etc/sudoers
[root@manage ~]# ll /etc/sudoers
-r--------. 1 root root 3767 9月   7 10:03 /etc/sudoers
[root@manage ~]#
```

### 删除Centos6.8 的原生OpenJdk
> 查看原生OpenJdk

```
[channelmonitor@manage ~]$ java -version
java version "1.7.0_99"
OpenJDK Runtime Environment (rhel-2.6.5.1.el6-x86_64 u99-b00)
OpenJDK 64-Bit Server VM (build 24.95-b01, mixed mode)
[channelmonitor@manage ~]$
```

\[channelmonitor@manage ~\]$ sudo rpm -qa \| grep java  
tzdata-java-2016c-1.el6.noarch  
java-1.6.0-openjdk-1.6.0.38-1.13.10.4.el6.x86\_64  
java-1.7.0-openjdk-1.7.0.99-2.6.5.1.el6.x86\_64  
\[channelmonitor@manage ~\]$

> 卸载CentOS的Java

```
yum -y remove java-1.6.0-openjdk-1.6.0.38-1.13.10.4.el6.x86_64
yum -y remove java-1.7.0-openjdk-1.7.0.99-2.6.5.1.el6.x86_64
```

\[channelmonitor@manage ~\]$ sudo rm -f /var/run/yum.pid  
\[channelmonitor@manage ~\]$ sudo yum -y remove java-1.6.0-openjdk-1.6.0.38-1.13.10.4.el6.x86\_64  
\[channelmonitor@manage ~\]$ yum -y remove java-1.7.0-openjdk-1.7.0.99-2.6.5.1.el6.x86\_64

> 查看 java 版本  
> \[channelmonitor@manage ~\]$ java -version  
> bash: /usr/bin/java: 没有那个文件或目录  
> \[channelmonitor@manage ~\]$
>
> 安装sun的jdk

```
[channelmonitor@manage devsoft]$ sudo tar -xzvf jdk-8u144-linux-x64.tar.gz -C /usr/

[channelmonitor@manage usr]$ sudo mv jdk1.8.0_144 java

[channelmonitor@manage usr]$ sudo chown channelmonitor:channelmonitor java
```

> 编辑 profile 文件  
> \[channelmonitor@manage usr\]$ sudo vim /etc/profile
>
> ```
> export JAVA_HOME=/usr/java  
> export JRE_HOME=/usr/java/jre   
> export CLASSPATH=$JAVA_HOME/lib   
> export PATH=:$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
> ```
>
> 修改 使之生效  
> \[channelmonitor@manage usr\]$ source /etc/profile
>
> 验证 java 版本
>
> ```
> [channelmonitor@manage usr]$ java -version
> java version "1.8.0_144"
> Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
> Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
> [channelmonitor@manage usr]$
> ```



