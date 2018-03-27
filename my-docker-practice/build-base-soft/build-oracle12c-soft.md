# 构建oracle12c的基础软件环境

在Ubuntu14.04为基础系统环境下，构建docker的oracle12c的基础软件环境，根据具体步骤构建出本文。

### _从docker的hub中搜索oracle镜像_

```
inpcs@inpcsPC:~$ sudo docker search oracle
```

> 执行后显示的内容如下：

```
NAME                                DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
wnameless/oracle-xe-11g             Oracle Express 11g R2 on Ubuntu 16.04 LTS       493                  [OK]
oraclelinux                         Oracle Linux is an open-source operating s...   369       [OK]       
frolvlad/alpine-oraclejdk8          The smallest Docker image with OracleJDK 8...   250                  [OK]
alexeiled/docker-oracle-xe-11g      This is a working (hopefully) Oracle XE 11...   209                  [OK]
sath89/oracle-12c                   Oracle Standard Edition 12c Release 1 with...   163                  [OK]
sath89/oracle-xe-11g                Oracle xe 11g with database files mount su...   111                  [OK]
isuper/java-oracle                  This repository contains all java releases...   56                   [OK]
jaspeen/oracle-11g                  Docker image for Oracle 11g database            45                   [OK]
oracle/openjdk                      Docker images containing OpenJDK Oracle Linux   22                   [OK]
ingensi/oracle-jdk                  Official Oracle JDK installed on centos.        21                   [OK]
oracle/glassfish                    GlassFish Java EE Application Server on Or...   21                   [OK]
airdock/oracle-jdk                  Docker Image for Oracle Java SDK (8 and 7)...   20                   [OK]
cogniteev/oracle-java               Oracle JDK 6, 7, 8, and 9 based on Ubuntu ...   19                   [OK]
n3ziniuka5/ubuntu-oracle-jdk        Ubuntu with Oracle JDK. Check tags for ver...   13                   [OK]
oracle/nosql                        Oracle NoSQL on a Docker Image with Oracle...   12                   [OK]
bofm/oracle12c                      Docker image for Oracle Database                11                   [OK]
andreptb/oracle-java                Debian Jessie based image with Oracle JDK ...   8                    [OK]
openweb/oracle-tomcat               A fork off of Official tomcat image with O...   6                    [OK]
flurdy/oracle-java7                 Base image containing Oracle's Java 7 JDK       4                    [OK]
davidcaste/debian-oracle-java       Oracle Java 8 (and 7) over Debian Jessie        3                    [OK]
teradatalabs/centos6-java8-oracle   Docker image of CentOS 6 with Oracle JDK 8...   2                    
publicisworldwide/oracle-core       This is the core image based on Oracle Lin...   1                    [OK]
sigma/nimbus-lock-oracle                                                            0                    [OK]
spansari/nodejs-oracledb            nodejs with oracledb installed globally on...   0                    
trollin/oraclelinux
```

### _选择12C的版本\[sath89/oracle-12c\]_

sudo docker pull sath89/oracle-12c

```
inpcs@inpcsPC:~$ sudo docker pull sath89/oracle-12c
Using default tag: latest
latest: Pulling from sath89/oracle-12c
Digest: sha256:6e9de6f1e5927e6012f1d824f998a20b7cb53f3042231869d5d6826fddb66282
Status: Image is up to date for sath89/oracle-12c:latest
```

# 查看Images文件

sudo docker images

```
inpcs@inpcsPC:~$ sudo docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
mongo                           latest              b39de1d79a53        3 weeks ago         359 MB
redis                           latest              d4f259423416        5 weeks ago         106 MB
sath89/oracle-12c               latest              b93c23bfc173        6 weeks ago         5.7 GB
mkoester/ubuntu12.04-mysql5.5   latest              7d4a3356364e        3 years ago         303 MB
inpcs@inpcsPC:~$
```

# 使用images 创建一个container，并运行其上的oracle数据库

sudo docker run -d -p 8080:8080 -p 1521:1521 -v /media/inpcs/ee43d9f8-d8ba-44d0-9a34-190a5a2d2a38/data/oracleData/data:/u01/app/oracle sath89/oracle-12c

```
inpcs@inpcsPC:~$ sudo docker run -d -p 8080:8080 -p 1521:1521 -v /media/inpcs/ee43d9f8-d8ba-44d0-9a34-190a5a2d2a38/data/oracleData/data:/u01/app/oracle sath89/oracle-12c
55f12846e98b2c7dde5a7f5a01342a100587c78f2815f47a4a8e80fd95c04964
inpcs@inpcsPC:~$
```

# 查看运行日志信息

sudo docker logs -f 55f12846e98b2c7dde5a7f5a01342a100587c78f2815f47a4a8e80fd95c04964

```
inpcs@inpcsPC:~$ sudo docker logs -f 55f12846e98b2c7dde5a7f5a01342a100587c78f2815f47a4a8e80fd95c04964
Database not initialized. Initializing database.
Starting tnslsnr
Copying database files
1% complete
3% complete
11% complete
18% complete
26% complete
37% complete
Creating and starting Oracle instance
40% complete
45% complete
50% complete
55% complete
56% complete
60% complete
62% complete
Completing Database Creation
66% complete
70% complete
73% complete
85% complete
96% complete
100% complete
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/xe/xe.log" for further details.
Configuring Apex console
Database initialized. Please visit http://#containeer:8080/em http://#containeer:8080/apex for extra configuration if needed
Starting web management console

PL/SQL procedure successfully completed.

Starting import from '/docker-entrypoint-initdb.d':
found file /docker-entrypoint-initdb.d//docker-entrypoint-initdb.d/*
[IMPORT] /entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*

Import finished

Database ready to use. Enjoy! ;)
```

# 查看运行容器信息

sudo docker ps

```
inpcs@inpcsPC:~$ sudo docker ps
CONTAINER ID        IMAGE                                  COMMAND              CREATED             STATUS              PORTS                                            NAMES
55f12846e98b        sath89/oracle-12c                      "/entrypoint.sh "    28 minutes ago      Up 28 minutes       0.0.0.0:1521->1521/tcp, 0.0.0.0:8080->8080/tcp   elated_cori
inpcs@inpcsPC:~$
```

# 进入oracle container容器内部

sudo docker exec -it 55f12846e98b /bin/bash

```
inpcs@inpcsPC:~$ sudo docker exec -it 55f12846e98b /bin/bash
root@55f12846e98b:/#
```

# 切换到oracle用户

su oracle

```
root@55f12846e98b:/# su oracle
oracle@55f12846e98b:/$
```

# 使用sqlplus

$ORACLE\_HOME/bin/sqlplus / as sysdba

```
oracle@55f12846e98b:/$ $ORACLE_HOME/bin/sqlplus / as sysdba

SQL*Plus: Release 12.1.0.2.0 Production on Wed Aug 30 07:24:17 2017

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Standard Edition Release 12.1.0.2.0 - 64bit Production

SQL>
```

# Oracle实例相关信息

| hostname | prot | sid | username | password |
| --- | :--- | ---: | ---: | ---: |
| localhost | 1521 | xe | system | oracle |

接下来就可以快速使用Oracle12C了

