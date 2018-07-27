# [在Yarn上运行spark-shell和spark-sql命令行](http://lxw1234.com/archives/2015/08/448.htm)

前面的文章《[Spark On Yarn：提交Spark应用程序到Yarn](http://lxw1234.com/archives/2015/07/416.htm)》介绍了将Spark应用程序提交到Yarn上运行。有时候在做开发测试的时候，需要使用spark-shell和spark-sql命令行，除了Local和Spark standalone模式，spark-shell和spark-sql也可以运行在yarn上，这里就简单介绍一下使用方法。

## spark-shell On Yarn

如果你已经有一个正常运行的Hadoop Yarn环境，那么只需要下载相应版本的Spark，解压之后做为Spark客户端即可。

需要配置Yarn的配置文件目录，export HADOOP\_CONF\_DIR=/etc/hadoop/conf   这个可以配置在spark-env.sh中。

运行命令：

```
cd $SPARK_HOME/bin
./spark-shell \
--master yarn-client \
--executor-memory 1G \
--num-executors 10
```

注意，这里的–master必须使用yarn-client模式，如果指定yarn-cluster，则会报错：

Error: Cluster deploy mode is not applicable to Spark shells.

因为spark-shell作为一个与用户交互的命令行，必须将Driver运行在本地，而不是yarn上。

其中的参数与提交Spark应用程序到yarn上用法一样。

启动之后，在命令行看上去和standalone模式下的无异：

![](http://7xipth.com1.z0.glb.clouddn.com/0811-1.jpg "spark-shell on yarn")

在ResourceManager的WEB页面上，看到了该应用程序（spark-shell是被当做一个长服务的应用程序运行在yarn上）：

![](http://7xipth.com1.z0.glb.clouddn.com/0811-2.jpg "spark-shell on yarn")

点击ApplicationMaster的UI，进入到了Spark应用程序监控的WEB页面：

![](http://7xipth.com1.z0.glb.clouddn.com/0811-3.jpg "spark-shell on yarn")



## spark-sql On Yarn

spark-sql命令行运行在yarn上，原理和spark-shell on yarn一样。只不过需要将Hive使用的相关包都加到Spark环境变量。

1. 将hive-site.xml拷贝到$SPARK\_HOME/conf

2.export HIVE\_HOME=/usr/local/apache-hive-0.13.1-bin 添加到spark-env.sh

3.将以下jar包添加到Spark环境变量：

datanucleus-api-jdo-3.2.6.jar、datanucleus-core-3.2.10.jar、datanucleus-rdbms-3.2.9.jar、mysql-connector-java-5.1.15-bin.jar

可以在spark-env.sh中直接添加到SPARK\_CLASSPATH变量中。



运行命令：

```
cd $SPARK_HOME/bin
./spark-sql \
--master yarn-client \
--executor-memory 1G \
--num-executors 10
```

即可在yarn上运行spark-sql命令行。  
![](http://7xipth.com1.z0.glb.clouddn.com/0811-5.jpg "spark-sql on yarn")

在ResourceManager上的显示以及点击ApplicationMaster进去Spark的WEB UI，与spark-shell无异。

![](http://7xipth.com1.z0.glb.clouddn.com/0811-4.jpg "spark-sql on yarn")



这样，只要之前有使用Hadoop Yarn，那么就不需要搭建standalone的Spark集群，也能发挥Spark的强大威力了。



其他相关阅读：

## [Spark On Yarn：提交Spark应用程序到Yarn](http://lxw1234.com/archives/2015/07/416.htm)

## [Spark1.3.1安装配置运行](http://lxw1234.com/archives/2015/06/281.htm)

## [Spark1.4.0-SparkSQL与Hive整合-支持窗口分析函数](http://lxw1234.com/archives/2015/06/294.htm)

## [Spark算子系列文章](http://lxw1234.com/archives/2015/07/363.htm)



如果觉得本博客对您有帮助，请[赞助作者](http://lxw1234.com/pay-blog)。

转载请注明：[lxw的大数据田地](http://lxw1234.com/)»[在Yarn上运行spark-shell和spark-sql命令行](http://lxw1234.com/archives/2015/08/448.htm)



