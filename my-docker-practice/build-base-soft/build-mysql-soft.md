# 构建基于官方的mysql的镜像

#### 重点说明：

配置文件在这个地址下，并且默认没有开启log

```
/etc/mysql/mysql.conf.d/mysqld.cnf
```

参见mysqld.cnf文件内容

```
# Copyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
log-error	= /var/log/mysql/error.log
symbolic-links=0

```

#### 运行命令





docker run -p 8536:3306 --name mymysql -v $PWD/conf/my.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf -v $PWD/logs:/var/log/mysql -v $PWD/data:/var/lib/mysql -e MYSQL\_ROOT\_PASSWORD=zkfread19761129 -d mysql:5.6

命令说明：

* **-p 3306:3306**：将容器的 3306 端口映射到主机的 3306 端口。

* **-v -v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。

* **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。

* **-v $PWD/data:/var/lib/mysql**：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。

* **-e MYSQL\_ROOT\_PASSWORD=123456：**初始化 root 用户的密码



-v /soft/mysql/my.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf



