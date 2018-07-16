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

## 使用离线方式下载docker相关的离线安装rpm包

### 离线下载docker.io

```
## 离线方式下载docker-io的安装包
[root@localhost yumdownloadfile]# yumdownloader --resolve docker-io
##安装python2.7 所有的开发工具包
[root@localhost yumdownloadfile]# yumdownloader "@Development Tools" --resolve
## 安装python2.7必须工具包
[root@localhost yumdownloadfile]# yumdownloader --resolve zlib-devel bzip2-devel openssl-devel ncurses-devel
```

#### 执行docker-io包的顺序

```
Dependencies Resolved

=================================================================================================
 Package                   Arch              Version                       Repository       Size
=================================================================================================
Installing:
 docker-io                 x86_64            1.7.1-2.el6                   epel            4.6 M
Installing for dependencies:
 libcgroup                 x86_64            0.40.rc1-26.el6               base            131 k
 lua-alt-getopt            noarch            0.7.0-1.el6                   epel            6.9 k
 lua-filesystem            x86_64            1.4.2-1.el6                   epel             24 k
 lua-lxc                   x86_64            1.0.11-1.el6                  epel             16 k
 lxc                       x86_64            1.0.11-1.el6                  epel            124 k
 lxc-libs                  x86_64            1.0.11-1.el6                  epel            257 k

Transaction Summary
=================================================================================================
Install       7 Package(s)

Total download size: 5.1 M
Installed size: 20 M
Is this ok [y/N]: y
Downloading Packages:
(1/7): docker-io-1.7.1-2.el6.x86_64.rpm                                   | 4.6 MB     00:00     
(2/7): libcgroup-0.40.rc1-26.el6.x86_64.rpm                               | 131 kB     00:00     
(3/7): lua-alt-getopt-0.7.0-1.el6.noarch.rpm                              | 6.9 kB     00:00     
(4/7): lua-filesystem-1.4.2-1.el6.x86_64.rpm                              |  24 kB     00:00     
(5/7): lua-lxc-1.0.11-1.el6.x86_64.rpm                                    |  16 kB     00:00     
(6/7): lxc-1.0.11-1.el6.x86_64.rpm                                        | 124 kB     00:00     
(7/7): lxc-libs-1.0.11-1.el6.x86_64.rpm                                   | 257 kB     00:00     
-------------------------------------------------------------------------------------------------
Total                                                            2.1 MB/s | 5.1 MB     00:02     
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Importing GPG key 0x0608B895:
 Userid : EPEL (6) <epel@fedoraproject.org>
 Package: epel-release-6-8.noarch (@/epel-release-6-8.noarch)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
Is this ok [y/N]: y
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : lxc-libs-1.0.11-1.el6.x86_64                                                  1/7 
  Installing : lua-filesystem-1.4.2-1.el6.x86_64                                             2/7 
  Installing : lua-lxc-1.0.11-1.el6.x86_64                                                   3/7 
  Installing : libcgroup-0.40.rc1-26.el6.x86_64                                              4/7 
  Installing : lua-alt-getopt-0.7.0-1.el6.noarch                                             5/7 
  Installing : lxc-1.0.11-1.el6.x86_64                                                       6/7 
  Installing : docker-io-1.7.1-2.el6.x86_64                                                  7/7 
  Verifying  : lxc-1.0.11-1.el6.x86_64                                                       1/7 
  Verifying  : lua-lxc-1.0.11-1.el6.x86_64                                                   2/7 
  Verifying  : lxc-libs-1.0.11-1.el6.x86_64                                                  3/7 
  Verifying  : lua-alt-getopt-0.7.0-1.el6.noarch                                             4/7 
  Verifying  : docker-io-1.7.1-2.el6.x86_64                                                  5/7 
  Verifying  : libcgroup-0.40.rc1-26.el6.x86_64                                              6/7 
  Verifying  : lua-filesystem-1.4.2-1.el6.x86_64                                             7/7 

Installed:
  docker-io.x86_64 0:1.7.1-2.el6                                                                 

Dependency Installed:
  libcgroup.x86_64 0:0.40.rc1-26.el6              lua-alt-getopt.noarch 0:0.7.0-1.el6            
  lua-filesystem.x86_64 0:1.4.2-1.el6             lua-lxc.x86_64 0:1.0.11-1.el6                  
  lxc.x86_64 0:1.0.11-1.el6                       lxc-libs.x86_64 0:1.0.11-1.el6                 

Complete!

```

执行Development Tools工具包的顺序

```
[root@localhost ~]# yum groupinstall -y "Development tools"
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Group Process
Loading mirror speeds from cached hostfile
 * base: ftp.sjtu.edu.cn
 * epel: mirrors.aliyun.com
 * extras: mirrors.163.com
 * updates: ftp.sjtu.edu.cn
Package 1:make-3.81-23.el6.x86_64 already installed and latest version
Package 1:pkgconfig-0.23-9.1.el6.x86_64 already installed and latest version
Package elfutils-0.164-2.el6.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package autoconf.noarch 0:2.63-5.1.el6 will be installed
---> Package automake.noarch 0:1.11.1-4.el6 will be installed
---> Package binutils.x86_64 0:2.20.51.0.2-5.46.el6 will be updated
---> Package binutils.x86_64 0:2.20.51.0.2-5.48.el6 will be an update
---> Package bison.x86_64 0:2.4.1-5.el6 will be installed
---> Package byacc.x86_64 0:1.9.20070509-7.el6 will be installed
---> Package cscope.x86_64 0:15.6-7.el6 will be installed
---> Package ctags.x86_64 0:5.8-2.el6 will be installed
---> Package cvs.x86_64 0:1.11.23-16.el6 will be installed
---> Package diffstat.x86_64 0:1.51-2.el6 will be installed
---> Package doxygen.x86_64 1:1.6.1-6.el6 will be installed
---> Package flex.x86_64 0:2.5.35-9.el6 will be installed
---> Package gcc.x86_64 0:4.4.7-18.el6 will be updated
---> Package gcc.x86_64 0:4.4.7-23.el6 will be an update
--> Processing Dependency: libgomp = 4.4.7-23.el6 for package: gcc-4.4.7-23.el6.x86_64
--> Processing Dependency: cpp = 4.4.7-23.el6 for package: gcc-4.4.7-23.el6.x86_64
--> Processing Dependency: libgcc >= 4.4.7-23.el6 for package: gcc-4.4.7-23.el6.x86_64
---> Package gcc-c++.x86_64 0:4.4.7-23.el6 will be installed
--> Processing Dependency: libstdc++-devel = 4.4.7-23.el6 for package: gcc-c++-4.4.7-23.el6.x86_64
--> Processing Dependency: libstdc++ = 4.4.7-23.el6 for package: gcc-c++-4.4.7-23.el6.x86_64
---> Package gcc-gfortran.x86_64 0:4.4.7-23.el6 will be installed
--> Processing Dependency: libgfortran = 4.4.7-23.el6 for package: gcc-gfortran-4.4.7-23.el6.x86_64
--> Processing Dependency: libgfortran.so.3()(64bit) for package: gcc-gfortran-4.4.7-23.el6.x86_64
---> Package gettext.x86_64 0:0.17-18.el6 will be installed
---> Package git.x86_64 0:1.7.1-9.el6_9 will be installed
--> Processing Dependency: perl-Git = 1.7.1-9.el6_9 for package: git-1.7.1-9.el6_9.x86_64
--> Processing Dependency: perl(Git) for package: git-1.7.1-9.el6_9.x86_64
--> Processing Dependency: perl(Error) for package: git-1.7.1-9.el6_9.x86_64
---> Package indent.x86_64 0:2.2.10-7.el6 will be installed
---> Package intltool.noarch 0:0.41.0-1.1.el6 will be installed
--> Processing Dependency: perl(XML::Parser) for package: intltool-0.41.0-1.1.el6.noarch
--> Processing Dependency: gettext-devel for package: intltool-0.41.0-1.1.el6.noarch
---> Package libtool.x86_64 0:2.2.6-15.5.el6 will be installed
---> Package patch.x86_64 0:2.6-6.el6 will be updated
---> Package patch.x86_64 0:2.6-8.el6_9 will be an update
---> Package patchutils.x86_64 0:0.3.1-3.1.el6 will be installed
---> Package rcs.x86_64 0:5.7-37.el6 will be installed
---> Package redhat-rpm-config.noarch 0:9.0.3-51.el6.centos will be installed
---> Package rpm-build.x86_64 0:4.8.0-59.el6 will be installed
--> Processing Dependency: rpm = 4.8.0-59.el6 for package: rpm-build-4.8.0-59.el6.x86_64
--> Processing Dependency: /usr/bin/gdb-add-index for package: rpm-build-4.8.0-59.el6.x86_64
---> Package subversion.x86_64 0:1.6.11-15.el6_7 will be installed
--> Processing Dependency: perl(URI) >= 1.17 for package: subversion-1.6.11-15.el6_7.x86_64
---> Package swig.x86_64 0:1.3.40-6.el6 will be installed
---> Package systemtap.x86_64 0:2.9-9.el6 will be installed
--> Processing Dependency: systemtap-devel = 2.9-9.el6 for package: systemtap-2.9-9.el6.x86_64
--> Processing Dependency: systemtap-client = 2.9-9.el6 for package: systemtap-2.9-9.el6.x86_64
--> Running transaction check
---> Package cpp.x86_64 0:4.4.7-18.el6 will be updated
---> Package cpp.x86_64 0:4.4.7-23.el6 will be an update
---> Package gdb.x86_64 0:7.2-92.el6 will be installed
---> Package gettext-devel.x86_64 0:0.17-18.el6 will be installed
--> Processing Dependency: gettext-libs = 0.17-18.el6 for package: gettext-devel-0.17-18.el6.x86_64
--> Processing Dependency: libgettextpo.so.0()(64bit) for package: gettext-devel-0.17-18.el6.x86_64
--> Processing Dependency: libgcj_bc.so.1()(64bit) for package: gettext-devel-0.17-18.el6.x86_64
--> Processing Dependency: libasprintf.so.0()(64bit) for package: gettext-devel-0.17-18.el6.x86_64
---> Package libgcc.x86_64 0:4.4.7-18.el6 will be updated
---> Package libgcc.x86_64 0:4.4.7-23.el6 will be an update
---> Package libgfortran.x86_64 0:4.4.7-23.el6 will be installed
---> Package libgomp.x86_64 0:4.4.7-18.el6 will be updated
---> Package libgomp.x86_64 0:4.4.7-23.el6 will be an update
---> Package libstdc++.x86_64 0:4.4.7-18.el6 will be updated
---> Package libstdc++.x86_64 0:4.4.7-23.el6 will be an update
---> Package libstdc++-devel.x86_64 0:4.4.7-23.el6 will be installed
---> Package perl-Error.noarch 1:0.17015-4.el6 will be installed
---> Package perl-Git.noarch 0:1.7.1-9.el6_9 will be installed
---> Package perl-URI.noarch 0:1.40-2.el6 will be installed
---> Package perl-XML-Parser.x86_64 0:2.36-7.el6 will be installed
--> Processing Dependency: perl(LWP) for package: perl-XML-Parser-2.36-7.el6.x86_64
---> Package rpm.x86_64 0:4.8.0-55.el6 will be updated
--> Processing Dependency: rpm = 4.8.0-55.el6 for package: rpm-libs-4.8.0-55.el6.x86_64
--> Processing Dependency: rpm = 4.8.0-55.el6 for package: rpm-python-4.8.0-55.el6.x86_64
---> Package rpm.x86_64 0:4.8.0-59.el6 will be an update
---> Package systemtap-client.x86_64 0:2.9-9.el6 will be installed
--> Processing Dependency: systemtap-runtime = 2.9-9.el6 for package: systemtap-client-2.9-9.el6.x86_64
---> Package systemtap-devel.x86_64 0:2.9-9.el6 will be installed
--> Running transaction check
---> Package gettext-libs.x86_64 0:0.17-18.el6 will be installed
---> Package libgcj.x86_64 0:4.4.7-23.el6 will be installed
---> Package perl-libwww-perl.noarch 0:5.833-5.el6 will be installed
--> Processing Dependency: perl-HTML-Parser >= 3.33 for package: perl-libwww-perl-5.833-5.el6.noarch
--> Processing Dependency: perl(HTML::Entities) for package: perl-libwww-perl-5.833-5.el6.noarch
--> Processing Dependency: perl(Compress::Zlib) for package: perl-libwww-perl-5.833-5.el6.noarch
---> Package rpm-libs.x86_64 0:4.8.0-55.el6 will be updated
---> Package rpm-libs.x86_64 0:4.8.0-59.el6 will be an update
---> Package rpm-python.x86_64 0:4.8.0-55.el6 will be updated
---> Package rpm-python.x86_64 0:4.8.0-59.el6 will be an update
---> Package systemtap-runtime.x86_64 0:2.9-7.el6 will be updated
---> Package systemtap-runtime.x86_64 0:2.9-9.el6 will be an update
--> Running transaction check
---> Package perl-Compress-Zlib.x86_64 0:2.021-144.el6 will be installed
--> Processing Dependency: perl(IO::Uncompress::Gunzip) >= 2.021 for package: perl-Compress-Zlib-2.021-144.el6.x86_64
--> Processing Dependency: perl(IO::Compress::Gzip::Constants) >= 2.021 for package: perl-Compress-Zlib-2.021-144.el6.x86_64
--> Processing Dependency: perl(IO::Compress::Gzip) >= 2.021 for package: perl-Compress-Zlib-2.021-144.el6.x86_64
--> Processing Dependency: perl(IO::Compress::Base::Common) >= 2.021 for package: perl-Compress-Zlib-2.021-144.el6.x86_64
--> Processing Dependency: perl(Compress::Raw::Zlib) >= 2.021 for package: perl-Compress-Zlib-2.021-144.el6.x86_64
---> Package perl-HTML-Parser.x86_64 0:3.64-2.el6 will be installed
--> Processing Dependency: perl(HTML::Tagset) >= 3.03 for package: perl-HTML-Parser-3.64-2.el6.x86_64
--> Processing Dependency: perl(HTML::Tagset) for package: perl-HTML-Parser-3.64-2.el6.x86_64
--> Running transaction check
---> Package perl-Compress-Raw-Zlib.x86_64 1:2.021-144.el6 will be installed
---> Package perl-HTML-Tagset.noarch 0:3.20-4.el6 will be installed
---> Package perl-IO-Compress-Base.x86_64 0:2.021-144.el6 will be installed
---> Package perl-IO-Compress-Zlib.x86_64 0:2.021-144.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================
 Package                        Arch           Version                        Repository    Size
=================================================================================================
Installing:
 autoconf                       noarch         2.63-5.1.el6                   base         781 k
 automake                       noarch         1.11.1-4.el6                   base         550 k
 bison                          x86_64         2.4.1-5.el6                    base         637 k
 byacc                          x86_64         1.9.20070509-7.el6             base          48 k
 cscope                         x86_64         15.6-7.el6                     base         136 k
 ctags                          x86_64         5.8-2.el6                      base         147 k
 cvs                            x86_64         1.11.23-16.el6                 base         712 k
 diffstat                       x86_64         1.51-2.el6                     base          29 k
 doxygen                        x86_64         1:1.6.1-6.el6                  base         2.4 M
 flex                           x86_64         2.5.35-9.el6                   base         285 k
 gcc-c++                        x86_64         4.4.7-23.el6                   base         4.7 M
 gcc-gfortran                   x86_64         4.4.7-23.el6                   base         4.7 M
 gettext                        x86_64         0.17-18.el6                    base         1.8 M
 git                            x86_64         1.7.1-9.el6_9                  base         4.6 M
 indent                         x86_64         2.2.10-7.el6                   base         115 k
 intltool                       noarch         0.41.0-1.1.el6                 base          58 k
 libtool                        x86_64         2.2.6-15.5.el6                 base         564 k
 patchutils                     x86_64         0.3.1-3.1.el6                  base          95 k
 rcs                            x86_64         5.7-37.el6                     base         173 k
 redhat-rpm-config              noarch         9.0.3-51.el6.centos            base          60 k
 rpm-build                      x86_64         4.8.0-59.el6                   base         131 k
 subversion                     x86_64         1.6.11-15.el6_7                base         2.3 M
 swig                           x86_64         1.3.40-6.el6                   base         1.1 M
 systemtap                      x86_64         2.9-9.el6                      base          23 k
Updating:
 binutils                       x86_64         2.20.51.0.2-5.48.el6           base         2.8 M
 gcc                            x86_64         4.4.7-23.el6                   base          10 M
 patch                          x86_64         2.6-8.el6_9                    base          91 k
Installing for dependencies:
 gdb                            x86_64         7.2-92.el6                     base         2.3 M
 gettext-devel                  x86_64         0.17-18.el6                    base         155 k
 gettext-libs                   x86_64         0.17-18.el6                    base         112 k
 libgcj                         x86_64         4.4.7-23.el6                   base          19 M
 libgfortran                    x86_64         4.4.7-23.el6                   base         268 k
 libstdc++-devel                x86_64         4.4.7-23.el6                   base         1.6 M
 perl-Compress-Raw-Zlib         x86_64         1:2.021-144.el6                base          70 k
 perl-Compress-Zlib             x86_64         2.021-144.el6                  base          46 k
 perl-Error                     noarch         1:0.17015-4.el6                base          29 k
 perl-Git                       noarch         1.7.1-9.el6_9                  base          29 k
 perl-HTML-Parser               x86_64         3.64-2.el6                     base         109 k
 perl-HTML-Tagset               noarch         3.20-4.el6                     base          17 k
 perl-IO-Compress-Base          x86_64         2.021-144.el6                  base          70 k
 perl-IO-Compress-Zlib          x86_64         2.021-144.el6                  base         136 k
 perl-URI                       noarch         1.40-2.el6                     base         117 k
 perl-XML-Parser                x86_64         2.36-7.el6                     base         224 k
 perl-libwww-perl               noarch         5.833-5.el6                    base         390 k
 systemtap-client               x86_64         2.9-9.el6                      base         3.7 M
 systemtap-devel                x86_64         2.9-9.el6                      base         1.7 M
Updating for dependencies:
 cpp                            x86_64         4.4.7-23.el6                   base         3.7 M
 libgcc                         x86_64         4.4.7-23.el6                   base         104 k
 libgomp                        x86_64         4.4.7-23.el6                   base         135 k
 libstdc++                      x86_64         4.4.7-23.el6                   base         296 k
 rpm                            x86_64         4.8.0-59.el6                   base         906 k
 rpm-libs                       x86_64         4.8.0-59.el6                   base         318 k
 rpm-python                     x86_64         4.8.0-59.el6                   base          61 k
 systemtap-runtime              x86_64         2.9-9.el6                      base         206 k

Transaction Summary
=================================================================================================
Install      43 Package(s)
Upgrade      11 Package(s)

Total download size: 74 M
Downloading Packages:
(1/54): autoconf-2.63-5.1.el6.noarch.rpm                                  | 781 kB     00:00     
(2/54): automake-1.11.1-4.el6.noarch.rpm                                  | 550 kB     00:00     
(3/54): binutils-2.20.51.0.2-5.48.el6.x86_64.rpm                          | 2.8 MB     00:00     
(4/54): bison-2.4.1-5.el6.x86_64.rpm                                      | 637 kB     00:00     
(5/54): byacc-1.9.20070509-7.el6.x86_64.rpm                               |  48 kB     00:00     
(6/54): cpp-4.4.7-23.el6.x86_64.rpm                                       | 3.7 MB     00:00     
(7/54): cscope-15.6-7.el6.x86_64.rpm                                      | 136 kB     00:00     
(8/54): ctags-5.8-2.el6.x86_64.rpm                                        | 147 kB     00:00     
(9/54): cvs-1.11.23-16.el6.x86_64.rpm                                     | 712 kB     00:00     
(10/54): diffstat-1.51-2.el6.x86_64.rpm                                   |  29 kB     00:00     
(11/54): doxygen-1.6.1-6.el6.x86_64.rpm                                   | 2.4 MB     00:00     
(12/54): flex-2.5.35-9.el6.x86_64.rpm                                     | 285 kB     00:00     
(13/54): gcc-4.4.7-23.el6.x86_64.rpm                                      |  10 MB     00:01     
(14/54): gcc-c++-4.4.7-23.el6.x86_64.rpm                                  | 4.7 MB     00:00     
(15/54): gcc-gfortran-4.4.7-23.el6.x86_64.rpm                             | 4.7 MB     00:00     
(16/54): gdb-7.2-92.el6.x86_64.rpm                                        | 2.3 MB     00:00     
(17/54): gettext-0.17-18.el6.x86_64.rpm                                   | 1.8 MB     00:00     
(18/54): gettext-devel-0.17-18.el6.x86_64.rpm                             | 155 kB     00:00     
(19/54): gettext-libs-0.17-18.el6.x86_64.rpm                              | 112 kB     00:00     
(20/54): git-1.7.1-9.el6_9.x86_64.rpm                                     | 4.6 MB     00:00     
(21/54): indent-2.2.10-7.el6.x86_64.rpm                                   | 115 kB     00:00     
(22/54): intltool-0.41.0-1.1.el6.noarch.rpm                               |  58 kB     00:00     
(23/54): libgcc-4.4.7-23.el6.x86_64.rpm                                   | 104 kB     00:00     
(24/54): libgcj-4.4.7-23.el6.x86_64.rpm                                   |  19 MB     00:02     
(25/54): libgfortran-4.4.7-23.el6.x86_64.rpm                              | 268 kB     00:00     
(26/54): libgomp-4.4.7-23.el6.x86_64.rpm                                  | 135 kB     00:00     
(27/54): libstdc++-4.4.7-23.el6.x86_64.rpm                                | 296 kB     00:00     
(28/54): libstdc++-devel-4.4.7-23.el6.x86_64.rpm                          | 1.6 MB     00:00     
(29/54): libtool-2.2.6-15.5.el6.x86_64.rpm                                | 564 kB     00:00     
(30/54): patch-2.6-8.el6_9.x86_64.rpm                                     |  91 kB     00:00     
(31/54): patchutils-0.3.1-3.1.el6.x86_64.rpm                              |  95 kB     00:00     
(32/54): perl-Compress-Raw-Zlib-2.021-144.el6.x86_64.rpm                  |  70 kB     00:00     
(33/54): perl-Compress-Zlib-2.021-144.el6.x86_64.rpm                      |  46 kB     00:00     
(34/54): perl-Error-0.17015-4.el6.noarch.rpm                              |  29 kB     00:00     
(35/54): perl-Git-1.7.1-9.el6_9.noarch.rpm                                |  29 kB     00:00     
(36/54): perl-HTML-Parser-3.64-2.el6.x86_64.rpm                           | 109 kB     00:00     
(37/54): perl-HTML-Tagset-3.20-4.el6.noarch.rpm                           |  17 kB     00:00     
(38/54): perl-IO-Compress-Base-2.021-144.el6.x86_64.rpm                   |  70 kB     00:00     
(39/54): perl-IO-Compress-Zlib-2.021-144.el6.x86_64.rpm                   | 136 kB     00:00     
(40/54): perl-URI-1.40-2.el6.noarch.rpm                                   | 117 kB     00:01     
(41/54): perl-XML-Parser-2.36-7.el6.x86_64.rpm                            | 224 kB     00:01     
(42/54): perl-libwww-perl-5.833-5.el6.noarch.rpm                          | 390 kB     00:01     
(43/54): rcs-5.7-37.el6.x86_64.rpm                                        | 173 kB     00:00     
(44/54): redhat-rpm-config-9.0.3-51.el6.centos.noarch.rpm                 |  60 kB     00:00     
(45/54): rpm-4.8.0-59.el6.x86_64.rpm                                      | 906 kB     00:01     
(46/54): rpm-build-4.8.0-59.el6.x86_64.rpm                                | 131 kB     00:00     
(47/54): rpm-libs-4.8.0-59.el6.x86_64.rpm                                 | 318 kB     00:00     
(48/54): rpm-python-4.8.0-59.el6.x86_64.rpm                               |  61 kB     00:00     
(49/54): subversion-1.6.11-15.el6_7.x86_64.rpm                            | 2.3 MB     00:01     
(50/54): swig-1.3.40-6.el6.x86_64.rpm                                     | 1.1 MB     00:00     
(51/54): systemtap-2.9-9.el6.x86_64.rpm                                   |  23 kB     00:00     
(52/54): systemtap-client-2.9-9.el6.x86_64.rpm                            | 3.7 MB     00:01     
(53/54): systemtap-devel-2.9-9.el6.x86_64.rpm                             | 1.7 MB     00:00     
(54/54): systemtap-runtime-2.9-9.el6.x86_64.rpm                           | 206 kB     00:00     
-------------------------------------------------------------------------------------------------
Total                                                            2.3 MB/s |  74 MB     00:31     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : libgcc-4.4.7-23.el6.x86_64                                                   1/65 
  Updating   : libstdc++-4.4.7-23.el6.x86_64                                                2/65 
  Updating   : rpm-libs-4.8.0-59.el6.x86_64                                                 3/65 
  Updating   : rpm-4.8.0-59.el6.x86_64                                                      4/65 
  Installing : perl-URI-1.40-2.el6.noarch                                                   5/65 
  Updating   : patch-2.6-8.el6_9.x86_64                                                     6/65 
  Installing : 1:perl-Error-0.17015-4.el6.noarch                                            7/65 
  Updating   : libgomp-4.4.7-23.el6.x86_64                                                  8/65 
  Installing : 1:perl-Compress-Raw-Zlib-2.021-144.el6.x86_64                                9/65 
  Installing : autoconf-2.63-5.1.el6.noarch                                                10/65 
  Installing : automake-1.11.1-4.el6.noarch                                                11/65 
  Updating   : binutils-2.20.51.0.2-5.48.el6.x86_64                                        12/65 
  Installing : perl-IO-Compress-Base-2.021-144.el6.x86_64                                  13/65 
  Installing : perl-IO-Compress-Zlib-2.021-144.el6.x86_64                                  14/65 
  Installing : perl-Compress-Zlib-2.021-144.el6.x86_64                                     15/65 
  Installing : git-1.7.1-9.el6_9.x86_64                                                    16/65 
  Installing : perl-Git-1.7.1-9.el6_9.noarch                                               17/65 
  Updating   : systemtap-runtime-2.9-9.el6.x86_64                                          18/65 
  Installing : libstdc++-devel-4.4.7-23.el6.x86_64                                         19/65 
  Installing : gettext-libs-0.17-18.el6.x86_64                                             20/65 
  Installing : libgcj-4.4.7-23.el6.x86_64                                                  21/65 
  Updating   : cpp-4.4.7-23.el6.x86_64                                                     22/65 
  Updating   : gcc-4.4.7-23.el6.x86_64                                                     23/65 
  Installing : systemtap-devel-2.9-9.el6.x86_64                                            24/65 
  Installing : systemtap-client-2.9-9.el6.x86_64                                           25/65 
  Installing : cvs-1.11.23-16.el6.x86_64                                                   26/65 
  Installing : gettext-0.17-18.el6.x86_64                                                  27/65 
  Installing : gettext-devel-0.17-18.el6.x86_64                                            28/65 
  Installing : perl-HTML-Tagset-3.20-4.el6.noarch                                          29/65 
  Installing : perl-HTML-Parser-3.64-2.el6.x86_64                                          30/65 
  Installing : perl-libwww-perl-5.833-5.el6.noarch                                         31/65 
  Installing : perl-XML-Parser-2.36-7.el6.x86_64                                           32/65 
  Installing : libgfortran-4.4.7-23.el6.x86_64                                             33/65 
  Installing : redhat-rpm-config-9.0.3-51.el6.centos.noarch                                34/65 
  Installing : gdb-7.2-92.el6.x86_64                                                       35/65 
  Installing : rpm-build-4.8.0-59.el6.x86_64                                               36/65 
  Installing : gcc-gfortran-4.4.7-23.el6.x86_64                                            37/65 
  Installing : intltool-0.41.0-1.1.el6.noarch                                              38/65 
  Installing : systemtap-2.9-9.el6.x86_64                                                  39/65 
  Installing : gcc-c++-4.4.7-23.el6.x86_64                                                 40/65 
  Installing : libtool-2.2.6-15.5.el6.x86_64                                               41/65 
  Installing : subversion-1.6.11-15.el6_7.x86_64                                           42/65 
  Updating   : rpm-python-4.8.0-59.el6.x86_64                                              43/65 
  Installing : 1:doxygen-1.6.1-6.el6.x86_64                                                44/65 
  Installing : swig-1.3.40-6.el6.x86_64                                                    45/65 
  Installing : rcs-5.7-37.el6.x86_64                                                       46/65 
  Installing : indent-2.2.10-7.el6.x86_64                                                  47/65 
  Installing : flex-2.5.35-9.el6.x86_64                                                    48/65 
  Installing : patchutils-0.3.1-3.1.el6.x86_64                                             49/65 
  Installing : byacc-1.9.20070509-7.el6.x86_64                                             50/65 
  Installing : cscope-15.6-7.el6.x86_64                                                    51/65 
  Installing : ctags-5.8-2.el6.x86_64                                                      52/65 
  Installing : diffstat-1.51-2.el6.x86_64                                                  53/65 
  Installing : bison-2.4.1-5.el6.x86_64                                                    54/65 
  Cleanup    : gcc-4.4.7-18.el6.x86_64                                                     55/65 
  Cleanup    : rpm-python-4.8.0-55.el6.x86_64                                              56/65 
  Cleanup    : rpm-libs-4.8.0-55.el6.x86_64                                                57/65 
  Cleanup    : rpm-4.8.0-55.el6.x86_64                                                     58/65 
  Cleanup    : systemtap-runtime-2.9-7.el6.x86_64                                          59/65 
  Cleanup    : libstdc++-4.4.7-18.el6.x86_64                                               60/65 
  Cleanup    : libgcc-4.4.7-18.el6.x86_64                                                  61/65 
  Cleanup    : binutils-2.20.51.0.2-5.46.el6.x86_64                                        62/65 
  Cleanup    : cpp-4.4.7-18.el6.x86_64                                                     63/65 
  Cleanup    : libgomp-4.4.7-18.el6.x86_64                                                 64/65 
  Cleanup    : patch-2.6-6.el6.x86_64                                                      65/65 
  Verifying  : gdb-7.2-92.el6.x86_64                                                        1/65 
  Verifying  : perl-Compress-Zlib-2.021-144.el6.x86_64                                      2/65 
  Verifying  : bison-2.4.1-5.el6.x86_64                                                     3/65 
  Verifying  : gcc-4.4.7-23.el6.x86_64                                                      4/65 
  Verifying  : perl-IO-Compress-Zlib-2.021-144.el6.x86_64                                   5/65 
  Verifying  : perl-HTML-Parser-3.64-2.el6.x86_64                                           6/65 
  Verifying  : rpm-4.8.0-59.el6.x86_64                                                      7/65 
  Verifying  : diffstat-1.51-2.el6.x86_64                                                   8/65 
  Verifying  : perl-URI-1.40-2.el6.noarch                                                   9/65 
  Verifying  : gcc-gfortran-4.4.7-23.el6.x86_64                                            10/65 
  Verifying  : subversion-1.6.11-15.el6_7.x86_64                                           11/65 
  Verifying  : systemtap-runtime-2.9-9.el6.x86_64                                          12/65 
  Verifying  : libstdc++-devel-4.4.7-23.el6.x86_64                                         13/65 
  Verifying  : automake-1.11.1-4.el6.noarch                                                14/65 
  Verifying  : rpm-python-4.8.0-59.el6.x86_64                                              15/65 
  Verifying  : redhat-rpm-config-9.0.3-51.el6.centos.noarch                                16/65 
  Verifying  : ctags-5.8-2.el6.x86_64                                                      17/65 
  Verifying  : cscope-15.6-7.el6.x86_64                                                    18/65 
  Verifying  : perl-Git-1.7.1-9.el6_9.noarch                                               19/65 
  Verifying  : libstdc++-4.4.7-23.el6.x86_64                                               20/65 
  Verifying  : byacc-1.9.20070509-7.el6.x86_64                                             21/65 
  Verifying  : perl-IO-Compress-Base-2.021-144.el6.x86_64                                  22/65 
  Verifying  : binutils-2.20.51.0.2-5.48.el6.x86_64                                        23/65 
  Verifying  : systemtap-devel-2.9-9.el6.x86_64                                            24/65 
  Verifying  : systemtap-client-2.9-9.el6.x86_64                                           25/65 
  Verifying  : autoconf-2.63-5.1.el6.noarch                                                26/65 
  Verifying  : 1:doxygen-1.6.1-6.el6.x86_64                                                27/65 
  Verifying  : 1:perl-Compress-Raw-Zlib-2.021-144.el6.x86_64                               28/65 
  Verifying  : patchutils-0.3.1-3.1.el6.x86_64                                             29/65 
  Verifying  : libgomp-4.4.7-23.el6.x86_64                                                 30/65 
  Verifying  : libgfortran-4.4.7-23.el6.x86_64                                             31/65 
  Verifying  : gcc-c++-4.4.7-23.el6.x86_64                                                 32/65 
  Verifying  : 1:perl-Error-0.17015-4.el6.noarch                                           33/65 
  Verifying  : gettext-libs-0.17-18.el6.x86_64                                             34/65 
  Verifying  : flex-2.5.35-9.el6.x86_64                                                    35/65 
  Verifying  : rpm-libs-4.8.0-59.el6.x86_64                                                36/65 
  Verifying  : rpm-build-4.8.0-59.el6.x86_64                                               37/65 
  Verifying  : perl-HTML-Tagset-3.20-4.el6.noarch                                          38/65 
  Verifying  : indent-2.2.10-7.el6.x86_64                                                  39/65 
  Verifying  : rcs-5.7-37.el6.x86_64                                                       40/65 
  Verifying  : gettext-0.17-18.el6.x86_64                                                  41/65 
  Verifying  : perl-XML-Parser-2.36-7.el6.x86_64                                           42/65 
  Verifying  : perl-libwww-perl-5.833-5.el6.noarch                                         43/65 
  Verifying  : systemtap-2.9-9.el6.x86_64                                                  44/65 
  Verifying  : intltool-0.41.0-1.1.el6.noarch                                              45/65 
  Verifying  : swig-1.3.40-6.el6.x86_64                                                    46/65 
  Verifying  : gettext-devel-0.17-18.el6.x86_64                                            47/65 
  Verifying  : cvs-1.11.23-16.el6.x86_64                                                   48/65 
  Verifying  : patch-2.6-8.el6_9.x86_64                                                    49/65 
  Verifying  : cpp-4.4.7-23.el6.x86_64                                                     50/65 
  Verifying  : libtool-2.2.6-15.5.el6.x86_64                                               51/65 
  Verifying  : git-1.7.1-9.el6_9.x86_64                                                    52/65 
  Verifying  : libgcj-4.4.7-23.el6.x86_64                                                  53/65 
  Verifying  : libgcc-4.4.7-23.el6.x86_64                                                  54/65 
  Verifying  : rpm-4.8.0-55.el6.x86_64                                                     55/65 
  Verifying  : gcc-4.4.7-18.el6.x86_64                                                     56/65 
  Verifying  : cpp-4.4.7-18.el6.x86_64                                                     57/65 
  Verifying  : libstdc++-4.4.7-18.el6.x86_64                                               58/65 
  Verifying  : libgcc-4.4.7-18.el6.x86_64                                                  59/65 
  Verifying  : systemtap-runtime-2.9-7.el6.x86_64                                          60/65 
  Verifying  : rpm-libs-4.8.0-55.el6.x86_64                                                61/65 
  Verifying  : libgomp-4.4.7-18.el6.x86_64                                                 62/65 
  Verifying  : patch-2.6-6.el6.x86_64                                                      63/65 
  Verifying  : binutils-2.20.51.0.2-5.46.el6.x86_64                                        64/65 
  Verifying  : rpm-python-4.8.0-55.el6.x86_64                                              65/65 

Installed:
  autoconf.noarch 0:2.63-5.1.el6          automake.noarch 0:1.11.1-4.el6                        
  bison.x86_64 0:2.4.1-5.el6              byacc.x86_64 0:1.9.20070509-7.el6                     
  cscope.x86_64 0:15.6-7.el6              ctags.x86_64 0:5.8-2.el6                              
  cvs.x86_64 0:1.11.23-16.el6             diffstat.x86_64 0:1.51-2.el6                          
  doxygen.x86_64 1:1.6.1-6.el6            flex.x86_64 0:2.5.35-9.el6                            
  gcc-c++.x86_64 0:4.4.7-23.el6           gcc-gfortran.x86_64 0:4.4.7-23.el6                    
  gettext.x86_64 0:0.17-18.el6            git.x86_64 0:1.7.1-9.el6_9                            
  indent.x86_64 0:2.2.10-7.el6            intltool.noarch 0:0.41.0-1.1.el6                      
  libtool.x86_64 0:2.2.6-15.5.el6         patchutils.x86_64 0:0.3.1-3.1.el6                     
  rcs.x86_64 0:5.7-37.el6                 redhat-rpm-config.noarch 0:9.0.3-51.el6.centos        
  rpm-build.x86_64 0:4.8.0-59.el6         subversion.x86_64 0:1.6.11-15.el6_7                   
  swig.x86_64 0:1.3.40-6.el6              systemtap.x86_64 0:2.9-9.el6                          

Dependency Installed:
  gdb.x86_64 0:7.2-92.el6                         gettext-devel.x86_64 0:0.17-18.el6            
  gettext-libs.x86_64 0:0.17-18.el6               libgcj.x86_64 0:4.4.7-23.el6                  
  libgfortran.x86_64 0:4.4.7-23.el6               libstdc++-devel.x86_64 0:4.4.7-23.el6         
  perl-Compress-Raw-Zlib.x86_64 1:2.021-144.el6   perl-Compress-Zlib.x86_64 0:2.021-144.el6     
  perl-Error.noarch 1:0.17015-4.el6               perl-Git.noarch 0:1.7.1-9.el6_9               
  perl-HTML-Parser.x86_64 0:3.64-2.el6            perl-HTML-Tagset.noarch 0:3.20-4.el6          
  perl-IO-Compress-Base.x86_64 0:2.021-144.el6    perl-IO-Compress-Zlib.x86_64 0:2.021-144.el6  
  perl-URI.noarch 0:1.40-2.el6                    perl-XML-Parser.x86_64 0:2.36-7.el6           
  perl-libwww-perl.noarch 0:5.833-5.el6           systemtap-client.x86_64 0:2.9-9.el6           
  systemtap-devel.x86_64 0:2.9-9.el6             

Updated:
  binutils.x86_64 0:2.20.51.0.2-5.48.el6  gcc.x86_64 0:4.4.7-23.el6  patch.x86_64 0:2.6-8.el6_9 

Dependency Updated:
  cpp.x86_64 0:4.4.7-23.el6                     libgcc.x86_64 0:4.4.7-23.el6                     
  libgomp.x86_64 0:4.4.7-23.el6                 libstdc++.x86_64 0:4.4.7-23.el6                  
  rpm.x86_64 0:4.8.0-59.el6                     rpm-libs.x86_64 0:4.8.0-59.el6                   
  rpm-python.x86_64 0:4.8.0-59.el6              systemtap-runtime.x86_64 0:2.9-9.el6             

Complete!
[root@localhost ~]# 

```













