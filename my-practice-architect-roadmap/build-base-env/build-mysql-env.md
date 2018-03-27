# 构建mysql的基础环境

在Centos6为基础系统环境构建mysql的基础环境，包含自动重启等功能，根据具体步骤构建出本文。

## 构建mysql5.5的基础环境

在Centos6的系统基础环境下，构建mysql5.5的单机数据库基础环境。

### _mysql 安装前的准备工作_

> 查看系统中已经存在的mysql安装包

```
[channelmonitor@manage ~]$ sudo yum list installed | grep mysql
mysql-libs.x86_64       5.1.73-7.el6    @anaconda-CentOS-201605220104.x86_64/6.8
```

> 删除系统中已经存在的mysql安装包

```
[channelmonitor@manage ~]$ sudo yum -y remove mysql-libs.x86_64
```

> 解压 myslq5.5
>
> ```
> [channelmonitor@manage devsoft ]$ sudo mkdir /usr/local/mysql  
> [channelmonitor@manage devsoft ]$ sudo tar -xzvf mysql-5.5.57-linux-glibc2.12-x86_64.tar.gz -C /usr/local/  
> [channelmonitor@manage local ]$ sudo mv mysql-5.5.57-linux-glibc2.12-x86_64 mysql
> ```

### _添加mysql用户组和用户_

```
[root@manage ~]# groupadd -g 3000 mysql  
[root@manage ~]# useradd -u 4000 -g mysql mysql  
[root@manage ~]# passwd mysql
```

> 更改 /usr/local/mysql的用户和用户组

```
[root@manage local]# chown -R mysql /usr/local/mysql  
[root@manage local]# chgrp -R mysql /usr/local/mysql   
[root@manage local]# ll
```

### _设置sudo权限_

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
[root@manage ~]# echo 'mysql ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
// 内容含义：用户名 mysql 网络中主机=（全部用户）不需要密码：全部范围
```

> 恢复访问权限

```
[root@manage ~]# sudo chmod 400 /etc/sudoers
[root@manage ~]# ll /etc/sudoers
-r--------. 1 root root 3767 9月   7 10:03 /etc/sudoers
[root@manage ~]#
```

> 测试安装 libaio 包

```  
[root@manage local]# yum install libaio -y  
已加载插件：fastestmirror, refresh-packagekit, security  
设置安装进程  
Loading mirror speeds from cached hostfile包 libaio-0.3.107-10.el6.x86_64 已安装并且是最新版本  
无须任何处理  
[root@manage local ] #
```

### _mysql安装_

> 初始化mysql  

在mysql目录下执行【***scripts/mysql_install_db --user=mysql***】命令，完成初始化操作。

```
[root@manage mysql ] # scripts/mysql_install_db --user=mysql
```

### _mysql的启动与停止_

> 启动mysql

```
/usr/local/mysql/support-files/mysql.server start
```

> 停止 mysql

```
/usr/local/mysql/support-files/mysql.server stop
```

> 重启 mysql

```
/usr/local/mysql/support-files/mysql.server restart
```

### _mysql的修改密码_

> 修改密码  
[root@manage mysql ] # ./bin/mysqladmin -u root password '123456'

> 使用root用户登录

```  
[root@manage bin ] # pwd  
/usr/local/mysql/bin  
[root@manage bin ] # mysql -u root -p
```
> 输入密码【123456】
```
Enter password:  
Welcome to the MySQL monitor.  Commands end with ; or \g.
```

### _配置mysql的开机自启动_

> 生成mysql全局配置文件

```
[root@manage support-files ] # sudo cp my-medium.cnf /etc/my.cnf
```

> 生成开机自启动文件

```
[root@manage support-files ] # sudo cp mysql.server /etc/init.d/mysql
```

> 添加开机自启动项

[root@manage support-files ] # chkconfig mysql on

```
mysql              0:关闭    1:关闭    2:启用    3:启用    4:启用    5:启用    6:关闭
```
编辑自启动文件

> 修改 profile 文件

```
PATH=/usr/local/mysql/bin:$PATH
export PATH
```

# 配置防火墙

> 添加3306（mysql5.5）,8080（tomcat）

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
```

> 开启，停止防火墙  
> /etc/init.d/iptables stop
>
> 忽略大小写（表名）  
> 在\[mysqld\]下加入一行  
> \[root@manage \]\# vim /etc/my.cnf
>
> ```
> lower_case_table_names=1
> ```

# 安装遇到的问题总结

> 10065  
> 是否开启mysql远程访问权限  
> 10061  
> 看看mysql服务是否启动（ps -ef \| grep mysql\)
>
> 设置默认IP连接数量限制10，改500  
> set global max\_connect\_errors = 500;
>
> 另外一个程序锁定了 yum；等待它退出……

\[channelmonitor@manage \]$ sudo rm -f /var/run/yum.pid

