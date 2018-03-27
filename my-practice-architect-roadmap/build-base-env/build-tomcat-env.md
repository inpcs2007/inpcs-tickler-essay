# 构建tomcat的基础环境

在Centos6为基础系统环境构建tomcat的基础环境，包含自动重启等功能，根据具体步骤构建出本文。

### 解压缩tomcat7压缩文件gz包
> 解压缩 tomcat7 到app目录下 

[channelmonitor@manage devsoft]$ tar -xzvf apache-tomcat-7.0.81.tar.gz -C ~/app/

### 配置 Tomcat7 

> 编辑 etc/profile 文件

export CATALINA_HOME=/home/channelmonitor/app/apache-tomcat-7.0.81
export CATALINE_BASH=/home/channelmonitor/app/apache-tomcat-7.0.81

### 配置开机自启动

> 生成开机自启动文件

```
[channelmonitor@manage bin]$ sudo cp /home/channelmonitor/app/apache-tomcat-7.0.81/bin/catalina.sh /etc/init.d/tomcat 
```

> 编辑自启动文件

添加内容
```
#chkconfig: 2345 80 90    
#description:tomcat auto start    
#processname: tomcat
```

> 为tomcat文件添加操作权限

进入到/etc/rc.d/init.d/目录，并使用 ***ll*** 命令查看tomcat文件权限 
如果没有权限则用命令 ***chmod +x tomcat*** 为tomcat添加权限

> 添加到开机启动服务列表

```
chkconfig --add tomcat
```

> 查看自动启动列表

最后检查下tomcat有没有被成功添加到开机启动服务列表中 
命令：chkconfig --list 
```
[channelmonitor@manage init.d]$ sudo chkconfig --add tomcat7
[channelmonitor@manage init.d]$ sudo chkconfig --list
tomcat7        	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
```
* 备注：
- 等级0表示：表示关机
- 等级1表示：单用户模式
- 等级2表示：无网络连接的多用户命令行模式
- 等级3表示：有网络连接的多用户命令行模式
- 等级4表示：不可用
- 等级5表示：带图形界面的多用户模式
- 等级6表示：重新启动
