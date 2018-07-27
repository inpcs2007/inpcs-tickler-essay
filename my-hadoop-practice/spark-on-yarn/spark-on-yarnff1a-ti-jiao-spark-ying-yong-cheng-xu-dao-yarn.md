# [Spark On Yarn：提交Spark应用程序到Yarn](http://lxw1234.com/archives/2015/07/416.htm)

Spark On Yarn模式配置非常简单，只需要下载编译好的Spark安装包，在一台带有Hadoop Yarn客户端的机器上解压，简单配置之后即可使用。

要把Spark应用程序提交到Yarn运行，首先需要配置HADOOP\_CONF\_DIR或者YARN\_CONF\_DIR，让Spark知道Yarn的配置信息，比如：ResourceManager的地址。可以配置在spark-env.sh中，也可以在提交Spark应用之前export：

export HADOOP\_CONF\_DIR=/etc/hadoop/conf

## yarn-cluster模式提交Spark应用程序



```
./spark-submit \
--class com.lxw1234.test.WordCount \
--master yarn-cluster \
--executor-memory 4G \
--num-executors 10 \
/home/lxw1234/spark-wordcount.jar \
/logs/2015-07-14/ /tmp/lxw1234/output/
```

## yarn-client模式提交Spark应用程序

```
./spark-submit \
--class com.lxw1234.test.WordCount \
--master yarn-client \
--executor-memory 4G \
--num-executors 10 \
/home/lxw1234/spark-wordcount.jar \
/logs/2015-07-14/ /tmp/lxw1234/output/
```

## Yarn Cluster模式和Yarn Client模式的主要区别

yarn-cluster模式中，应用程序\(包括SparkContext\)都是作为Yarn框架所需要的

ApplicationMaster,在Yarn ResourceManager为其分配的一个随机节点上运行；

而在yarn-client模式中，SparkContext运行在本地，该模式适用于应用程序本身需要在本地进行交互的场合。



Spark Standalone模式下提交Spark应用程序，可参考：

[http://lxw1234.com/archives/2015/05/215.htm](http://lxw1234.com/archives/2015/05/215.htm)

以下是一些Spark On Yarn相关的配置参数：

* **spark.yarn.am.memory**

默认值：512M

在yarn-client模式下，申请Yarn App Master所用的内存。

* **spark.driver.memory**

默认值：512M

在yarn-cluster模式下，申请Yarn App Master（包括Driver）所用的内存。

* **spark.yarn.am.cores**

默认值：1

在yarn-client模式下，申请Yarn App Master所用的CPU核数

* **spark.driver.cores**

默认值：1

在yarn-cluster模式下，申请Yarn App Master（包括Driver）所用的CPU核数。

* **spark.yarn.am.waitTime**

默认值：100s

在yarn-cluster模式下，Yarn App Master等待SparkContext初始化完成的时间；

在yarn-client模式下，Yarn App Master等待SparkContext链接它的时间；

* **spark.yarn.submit.file.replication**

默认值：HDFS副本数

Spark应用程序的依赖文件上传到HDFS时，在HDFS中的副本数，这些文件包括Spark的Jar包、应用程序的Jar包、其他作为DistributeCache使用的文件等。通常，如果你的集群节点数越多，相应地就需要设置越多的拷贝数以加快这些文件的分发。

* **spark.yarn.preserve.staging.files**

默认值：false

在应用程序结束后是否保留上述上传的文件。

* **spark.yarn.scheduler.heartbeat.interval-ms**

默认值：5000

Spark Application Master向Yarn ResourceManager发送心跳的时间间隔，单位毫秒。

* **spark.yarn.max.executor.failures**

默认值：numExecutors \* 2 \(最小为3\)

最多允许失败的Executor数量。

* **spark.yarn.historyServer.address**

默认值：none

Spark运行历史Server的地址，主机:host，如：lxw1234.com:18080，注意不能包含http://

默认不配置，必须开启Spark的historyServer之后才能配置。该地址用于Yarn ResourceManager在Spark应用程序结束时候，将该application的运行URL从ResourceManager的UI指向Spark historyServer UI。

* **spark.executor.instances**

默认值：2

Executor实例的数量，不能与spark.dynamicAllocation.enabled同时使用。

* **spark.yarn.queue**

默认值：default

指定提交到Yarn的资源池

* **spark.yarn.jar**

Spark应用程序使用的Jar包位置，比如：hdfs://cdh5/lxw1234.com/



参考更多大数据Hadoop、Spark、Hive相关：[lxw的大数据田地](http://lxw1234.com/)



另外，在提交Spark应用程序到Yarn时候，可以使用—files指定应用程序所需要的文件；

使用—jars 和 –archives添加应用程序所依赖的第三方jar包等。





如果觉得本博客对您有帮助，请[赞助作者](http://lxw1234.com/pay-blog)。

转载请注明：[lxw的大数据田地](http://lxw1234.com/)»[Spark On Yarn：提交Spark应用程序到Yarn](http://lxw1234.com/archives/2015/07/416.htm)



