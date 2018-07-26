# 大规模集群下使用P2P技术分发大文件



一般情况下，在运维多台服务器的时候，使用Ansible来完成文件的分发和命令的执行。但如果运维的机器数量多，而且内网带宽有限的情况下，比如，需要向500台机器分发一个1G大小的升级包，这时候如果使用Ansible直接分发，那么肯定会引起带宽占满，导致SSH链接超时，Ansible执行卡死，分发任务执行失败。

那么对于这种大文件的大规模分发，能想到的好的解决方案就是利用P2P网络实现，节省带宽，提高效率。关于p2p网络，大家可以自行搜索，典型的代表就是BitTorrent（BT下载）。

这里介绍的P2P文件分发的软件是Twitter开源的Murder。早期Twitter每次版本更新，需要向上万台服务器分发代码，从中央服务器向那么多台服务器上分发文件，服务器越多，分发时间越长。后来Twitter研发了Murder，用来解决这个问题，以前需要40-60分钟完成的代码发布任务，使用Murder之后，只需要几十秒就可以完成。

![](http://7xipth.com1.z0.glb.clouddn.com/20180722-2.png)

murder

![](http://7xipth.com1.z0.glb.clouddn.com/20180722-3.png)

murder

![](http://7xipth.com1.z0.glb.clouddn.com/20180722-4.jpeg)

murder

## Murder的架构

![](http://7xipth.com1.z0.glb.clouddn.com/20180722-1.jpg)

murder

整个Murder由三个组件构成：

* Tracker

它是运行在一台服务器上的单个服务，用来根据哪些torrent文件正在被分发，以及管理和维护Peer节点的信息和状态。

* Seeder

存放需要向其他主机分发的文件的服务器。Seeder将要分发的真实文件制作成torrent，这个torrent文件很小，只存放关于真实文件的基本Hash信息，torrent文件需要先分发到要下载文件的服务器\(peer\)上。

* Peer

就是要接收文件的服务器，他们之间可以互相传输文件。

## Murder的使用示例

我这里有4台服务器，角色分别如下：

tracker  192.168.1.220

seeder  192.168.1.220

peers   192.168.1.222、192.168.1.228、192.168.1.229

Tracker和Seeder在同一服务器上。

### 1.    下载和部署Murder

从[https://github.com/lg/murder](https://github.com/lg/murder)下载zip包，并解压到/opt目录下；

在所有的服务器（Tracker、Seeder、Peers）上，将Murder部署在/opt目录下，部署后的目录为：/opt/murder-master

### 2.    启动Tracker服务

在Tracker服务器上执行：

_**cd /opt/murder-master/dist**_

_**nohup python murder\_tracker.py –port 8998 –dfile data –logfile urder\_tracker.log &**_



–port tracker监听的端口,默认是8998

–dfile  存储近期下载信息的文件

–logfile  tracker日志文件，默认是标准输出

### 3.    在Seeder服务器上准备好要分发的文件，并创建种子

**在Seeder服务器上，要分发的文件：**

_\[liuxiaowen@test04v ~/lxw\]$ du -h /home/liuxiaowen/lxw/test.tar.gz_

_3.1G    /home/liuxiaowen/lxw/test.tar.gz_

**制作种子：**

_cd /opt/murder-master/dist/_

_python murder\_make\_torrent.py /home/liuxiaowen/lxw/test.tar.gz 192.168.1.220:8998 /tmp/test.tar.gz.torrent_

其中，第一个参数为要分发的文件；第二个参数为Tracker服务器的地址和端口；第三个参数为torrent文件路径。

**将种子文件分发至Peers：**

用Ansible将很小的torrent文件，分发至所有peers节点，关于Ansible的使用，请自行搜索，你也可以使用SCP。

_ansible myhosts -m synchronize -a “src=/tmp/test.tar.gz.torrent dest=/tmp/”_

**启动Seeder服务：**

_cd /opt/murder-master/dist/_

_python murder\_client.py seed /tmp/test.tar.gz.torrent /home/liuxiaowen/lxw/test.tar.gz 192.168.1.220_

最后一个IP是Seeder服务器本机IP。

### 4.    在peer节点执行下载

单个节点下载：在peer节点上，

_cd /opt/murder-master/dist/_

_python murder\_client.py peer /tmp/test.tar.gz.torrent /tmp/test.tar.gz 192.168.1.220_

单台当然不能体现Murder的优势了，使用Ansible命令，在多台Peer节点上同时下载：

_ansible myhosts -m shell -a “python /opt/murder-master/dist/murder\_client.py peer /tmp/test.tar.gz.torrent /tmp/test.tar.gz 192.168.1.220″_



**PS：这种P2P分发，在大规模服务器环境下才能体现出优势，节点越多，分发任务消耗的整体时间越短。。**





如果觉得本博客对您有帮助，请[赞助作者](http://lxw1234.com/pay-blog)。

转载请注明：[lxw的大数据田地](http://lxw1234.com/)»[大规模集群下使用P2P技术软件（Murder）分发大文件](http://lxw1234.com/archives/2018/07/915.htm)

