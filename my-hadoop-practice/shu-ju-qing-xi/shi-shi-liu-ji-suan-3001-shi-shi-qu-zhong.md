## [实时流计算、Spark Streaming、Kafka、Redis、Exactly-once、实时去重](http://lxw1234.com/archives/2018/02/901.htm)

本文想记录和表达的东西挺多的，一时想不到什么好的标题，所以就用上面的关键字作为标题了。

在实时流式计算中，最重要的是在任何情况下，消息不重复、不丢失，即Exactly-once。本文以Kafka–&gt;Spark Streaming–&gt;Redis为例，一方面说明一下如何做到Exactly-once，另一方面说明一下我是如何计算实时去重指标的。

![](http://7xipth.com1.z0.glb.clouddn.com/20180222-1.jpg "spark streaming")

## 1. 关于数据源

数据源是文本格式的日志，由Nginx产生，存放于日志服务器上。在日志服务器上部署Flume Agent，使用TAILDIR Source和Kafka Sink，将日志采集到Kafka进行临时存储。日志格式如下：

2018-02-22T00:00:00+08:00\|~\|200\|~\|/test?**pcid**=DEIBAH&**siteid**=3

2018-02-22T00:00:00+08:00\|~\|200\|~\|/test?pcid=GLLIEG&siteid=3

2018-02-22T00:00:00+08:00\|~\|200\|~\|/test?pcid=HIJMEC&siteid=8

2018-02-22T00:00:00+08:00\|~\|200\|~\|/test?pcid=HMGBDE&siteid=3

2018-02-22T00:00:00+08:00\|~\|200\|~\|/test?pcid=HIJFLA&siteid=4

2018-02-22T00:00:01+08:00\|~\|200\|~\|/test?pcid=JCEBBC&siteid=9

2018-02-22T00:00:01+08:00\|~\|200\|~\|/test?pcid=KJLAKG&siteid=8

2018-02-22T00:00:01+08:00\|~\|200\|~\|/test?pcid=FHEIKI&siteid=3

2018-02-22T00:00:01+08:00\|~\|200\|~\|/test?pcid=IGIDLB&siteid=3

2018-02-22T00:00:01+08:00\|~\|200\|~\|/test?pcid=IIIJCD&siteid=5

日志是由测试程序模拟产生的，字段之间由\|~\|分隔。

## 2. 实时计算需求

分天、分小时PV；

分天、分小时、分网站\(siteid\)PV；

分天 UV；

## 3. Spark Streaming消费Kafka数据

http://spark.apache.org/docs/latest/streaming-kafka-0-10-integration.html

在Spark Streaming中消费Kafka数据，保证Exactly-once的核心有三点：

使用Direct方式连接Kafka；自己保存和维护Offset；更新Offset和计算在同一事务中完成；

后面的Spark Streaming程序（文章结尾），主要有以下步骤：

1. 启动后，先从Redis中获取上次保存的Offset，Redis中的key为”topic\_partition”，即每个分区维护一个Offset；
2. 使用获取到的Offset，创建DirectStream；
3. 在处理每批次的消息时，利用Redis的事务机制，确保在Redis中指标的计算和Offset的更新维护，在同一事务中完成。只有这两者同步，才能真正保证消息的Exactly-once。

```

```

在启动Spark Streaming程序时候，有个参数最好指定：

spark.streaming.kafka.maxRatePerPartition=20000（每秒钟从topic的每个partition最多消费的消息条数）

如果程序第一次运行，或者因为某种原因暂停了很久重新启动时候，会积累很多消息，如果这些消息同时被消费，很有可能会因为内存不够而挂掉，因此，需要根据实际的数据量大小，以及批次的间隔时间来设置该参数，以限定批次的消息量。

如果该参数设置20000，而批次间隔时间未10秒，那么每个批次最多从Kafka中消费20万消息。

## 4. Redis中的数据模型

* 分小时、分网站PV

普通K-V结构，计算时候使用incr命令递增，

Key为 “site\_pv\_网站ID\_小时”，

如：site\_pv\_9\_2018-02-21-00、site\_pv\_10\_2018-02-21-01

该数据模型用于计算分网站的按小时及按天PV。

* 分小时PV

普通K-V结构，计算时候使用incr命令递增，

Key为“pv\_小时”，如：pv\_2018-02-21-14、pv\_2018-02-22-03

该数据模型用于计算按小时及按天总PV。

* 分天UV

Set结构，计算时候使用sadd命令添加，

Key为”uv\_天”，如：uv\_2018-02-21、uv\_2018-02-20

该数据模型用户计算按天UV（获取时候使用SCARD命令获取Set元素个数）



注：这些Key对应的时间，均由实际消息中的第一个字段（时间）而定。

## 5. 故障恢复

如果Spark Streaming程序因为停电、网络等意外情况终止而需要恢复，则直接重启即可；

如果因为其他原因需要重新计算某一时间段的消息，可以先删除Redis中对应时间段内的Key，然后从原始日志中截取该时间段内的消息，当做新消息添加至Kafka，由Spark Streaming程序重新消费并进行计算；

## 6. 附程序

依赖jar包：

commons-pool2-2.3.jar

jedis-2.9.0.jar

kafka-clients-0.11.0.1.jar

spark-streaming-kafka-0-10\_2.11-2.2.1.jar

_**InternalRedisClient （Redis链接池）**_

```
package com.lxw1234.spark
 
import redis.clients.jedis.JedisPool
import org.apache.commons.pool2.impl.GenericObjectPoolConfig
 
/**
 * @author lxw1234
 */
/**
 * Internal Redis client for managing Redis connection {@link Jedis} based on {@link RedisPool}
 */
object InternalRedisClient extends Serializable {
  
  @transient private var pool: JedisPool = null
  
  def makePool(redisHost: String, redisPort: Int, redisTimeout: Int,
      maxTotal: Int, maxIdle: Int, minIdle: Int): Unit = {
    makePool(redisHost, redisPort, redisTimeout, maxTotal, maxIdle, minIdle, true, false, 10000)   
  }
  
  def makePool(redisHost: String, redisPort: Int, redisTimeout: Int,
      maxTotal: Int, maxIdle: Int, minIdle: Int, testOnBorrow: Boolean,
      testOnReturn: Boolean, maxWaitMillis: Long): Unit = {
    if(pool == null) {
         val poolConfig = new GenericObjectPoolConfig()
         poolConfig.setMaxTotal(maxTotal)
         poolConfig.setMaxIdle(maxIdle)
         poolConfig.setMinIdle(minIdle)
         poolConfig.setTestOnBorrow(testOnBorrow)
         poolConfig.setTestOnReturn(testOnReturn)
         poolConfig.setMaxWaitMillis(maxWaitMillis)
         pool = new JedisPool(poolConfig, redisHost, redisPort, redisTimeout)
         
         val hook = new Thread{
             override def run = pool.destroy()
         }
         sys.addShutdownHook(hook.run)
    }
  }
  
  def getPool: JedisPool = {
    assert(pool != null)
    pool
  }
}
```

_**TestSparkStreaming**_

```
package com.lxw1234.spark
 
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.TopicPartition
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.Seconds
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.kafka010.ConsumerStrategies
import org.apache.spark.streaming.kafka010.HasOffsetRanges
import org.apache.spark.streaming.kafka010.KafkaUtils
import org.apache.spark.streaming.kafka010.LocationStrategies
 
import redis.clients.jedis.Pipeline
 
 
/**
 * @author lxw1234
 * 获取topic最小的offset
 * ./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list datadev1:9092 --topic lxw1234 --time -2
 */
object TestSparkStreaming {
  
  def main(args : Array[String]) : Unit = {
    val brokers = "datadev1:9092"
    val topic = "lxw1234"
    val partition : Int = 0 //测试topic只有一个分区
    val start_offset : Long = 0l
    
    //Kafka参数
    val kafkaParams = Map[String, Object](
        "bootstrap.servers" -> brokers,
        "key.deserializer" -> classOf[StringDeserializer],
        "value.deserializer" -> classOf[StringDeserializer],
        "group.id" -> "exactly-once",
        "enable.auto.commit" -> (false: java.lang.Boolean),
        "auto.offset.reset" -> "none"
    )
    
    // Redis configurations
    val maxTotal = 10
    val maxIdle = 10
    val minIdle = 1
    val redisHost = "172.16.213.79"
    val redisPort = 6379
    val redisTimeout = 30000
    //默认db，用户存放Offset和pv数据
    val dbDefaultIndex = 8
    InternalRedisClient.makePool(redisHost, redisPort, redisTimeout, maxTotal, maxIdle, minIdle)
    
        
    val conf = new SparkConf().setAppName("TestSparkStreaming").setIfMissing("spark.master", "local[2]")
    val ssc = new StreamingContext(conf, Seconds(10))
    
    //从Redis获取上一次存的Offset
    val jedis = InternalRedisClient.getPool.getResource
    jedis.select(dbDefaultIndex)
    val topic_partition_key = topic + "_" + partition
    var lastOffset = 0l 
    val lastSavedOffset = jedis.get(topic_partition_key)
    
    if(null != lastSavedOffset) {
      try {
        lastOffset = lastSavedOffset.toLong
      } catch {
        case ex : Exception => println(ex.getMessage)
        println("get lastSavedOffset error, lastSavedOffset from redis [" + lastSavedOffset + "] ")
        System.exit(1)
      }
    }
    InternalRedisClient.getPool.returnResource(jedis)
    
    println("lastOffset from redis -> " + lastOffset)
    
    //设置每个分区起始的Offset
    val fromOffsets = Map{new TopicPartition(topic, partition) -> lastOffset}
    
    //使用Direct API 创建Stream
    val stream = KafkaUtils.createDirectStream[String, String](
      ssc,
      LocationStrategies.PreferConsistent,
      ConsumerStrategies.Assign[String, String](fromOffsets.keys.toList, kafkaParams, fromOffsets)
    )
    
    //开始处理批次消息
    stream.foreachRDD { 
      rdd => 
        val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
        
        val result = processLogs(rdd)
        println("=============== Total " + result.length + " events in this batch ..")
        
        val jedis = InternalRedisClient.getPool.getResource
        val p1 : Pipeline = jedis.pipelined();
        p1.select(dbDefaultIndex)
        p1.multi() //开启事务
        
        
        //逐条处理消息
        result.foreach { 
          record => 
              //增加小时总pv
              val pv_by_hour_key = "pv_" + record.hour
              p1.incr(pv_by_hour_key)
              
              //增加网站小时pv
              val site_pv_by_hour_key = "site_pv_" + record.site_id + "_" + record.hour
              p1.incr(site_pv_by_hour_key)
              
              //使用set保存当天的uv
              val uv_by_day_key = "uv_" + record.hour.substring(0, 10)
              p1.sadd(uv_by_day_key, record.user_id)
        }
        
        //更新Offset
        offsetRanges.foreach { offsetRange =>
          println("partition : " + offsetRange.partition + " fromOffset:  " + offsetRange.fromOffset + " untilOffset: " + offsetRange.untilOffset)
          val topic_partition_key = offsetRange.topic + "_" + offsetRange.partition
          p1.set(topic_partition_key, offsetRange.untilOffset + "")
        }
        
        p1.exec();//提交事务  
        p1.sync();//关闭pipeline 
        
        InternalRedisClient.getPool.returnResource(jedis)
 
    }
    
    case class MyRecord(hour: String, user_id: String, site_id: String)
    
    def processLogs(messages: RDD[ConsumerRecord[String, String]]) : Array[MyRecord] = {
      messages.map(_.value()).flatMap(parseLog).collect()
    }
    
    //解析每条日志，生成MyRecord
    def parseLog(line: String): Option[MyRecord] = {
      val ary : Array[String] = line.split("\\|~\\|", -1);
      try {
        val hour = ary(0).substring(0, 13).replace("T", "-")
        val uri = ary(2).split("[=|&]",-1)
        val user_id = uri(1)
        val site_id = uri(3)
        return Some(MyRecord(hour,user_id,site_id))
        
      } catch {
        case ex : Exception => println(ex.getMessage)
      }
      
      return None
      
    }
    
    
    
    
    ssc.start()
    ssc.awaitTermination()
  }
  
 
}
```



![](http://7xipth.com1.z0.glb.clouddn.com/20180222-2.jpg "spark streaming")



![](http://7xipth.com1.z0.glb.clouddn.com/20180222-3.jpg "spark streaming")

如果觉得本博客对您有帮助，请[赞助作者](http://lxw1234.com/pay-blog)。

转载请注明：[lxw的大数据田地](http://lxw1234.com/)»[实时流计算、Spark Streaming、Kafka、Redis、Exactly-once、实时去重](http://lxw1234.com/archives/2018/02/901.htm)

