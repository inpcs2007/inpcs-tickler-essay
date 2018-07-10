#### log4j 2.3的线程切换 {#log4j-2.3的线程切换}

Log4j 2使用了新一代的基于LMAX Disruptor的无锁异步日志系统。在多线程的程序中，异步日志系统吞吐量比Log4j 1.x和logback高10倍，而时间延迟更低。

听起来很美，是不是？  
发现这个问题的原因是我们线上出了一次故障，当一次网络异常，导致上游不断重试并发请求量特别高时，cpu利用率跑到了4000%，服务完全恢复不过来了，一开始以为是GC的原因导致服务出问题了。后来在线下进行复现，查看GC发现没啥异常，但是发现了如下的数据

dstat  
![](https://images2015.cnblogs.com/blog/1081851/201704/1081851-20170423111415413-790504823.png)

usr 占用不高，sys 占用超高，同时csw\(context switch\) 达到1200w，一次csw 大约耗费1000ns，已经逼近cpu的极限。

> jstack -l 43911&gt; 43911.log  
> grep tid 43911.log \| wc -l  
> 12312  
> grep RUNNABLE 43911.log \| wc -l  
> 53

总线程12312，处于runnable的只有53个，看看这些线程在干什么

> "pool-6-thread-14380" prio=10 tid=0x00007f7087594000 nid=0x53e1 runnable \[0x00007f6b67bfc000\]  
> java.lang.Thread.State: TIMED\_WAITING \(parking\)  
> at sun.misc.Unsafe.park\(Native Method\)  
> at java.util.concurrent.locks.LockSupport.parkNanos\(LockSupport.java:349\)  
> at com.lmax.disruptor.MultiProducerSequencer.next\(MultiProducerSequencer.java:136\)  
> at com.lmax.disruptor.MultiProducerSequencer.next\(MultiProducerSequencer.java:105\)  
> at com.lmax.disruptor.RingBuffer.publishEvent\(RingBuffer.java:502\)  
> at org.apache.logging.log4j.core.async.AsyncLoggerConfigHelper.callAppendersFromAnotherThread\(AsyncLoggerConfigHelper.java:342\)  
> at org.apache.logging.log4j.core.async.AsyncLoggerConfig.callAppenders\(AsyncLoggerConfig.java:114\)  
> ......

> grep "LockSupport.java:349" 43911.log \| wc -l  
> 11536

线程都跑在LockSupport.java:349，  
unsafe.park\(false, 1\);  
1 nano = 10^-9s, 推测大量线程频繁的短时间sleep造成了大量的线程切换，做个实验：

```
public
static
void
contextSwitchTest
(
int
 threadCount)
throws
 Exception 
{
  ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
  
for
 (
int
 i = 
0
; i 
<
 threadCount; i++) {
    executorService.execute(
new
 Runnable() {
      
@Override
public
void
run
()
{
        
while
 (
true
) {
          LockSupport.parkNanos(
1
);
        }
      }
    });
  }
  executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
}
```

在一台搭载了`两个E5-2670 v3 @ 2.30GHz`的机器上测试这段代码，在threadCount达到600后，测试跑起来后立即发现上下文切换到百万级别，推测被印证。

原因找到了，接下来看看出问题是log生产速度怎么样：通过不断地`ls -al error.log/business.log`，发现Log的生成速度才几MB/s,远没有达到磁盘的极限200M/s，再做个测试：

```
private
static
void
loggerSpeedTest
(
int
 threadCount)
throws
 Exception 
{
  ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
  
for
 (
int
 i = 
0
; i 
<
 threadCount; i++) {
    executorService.execute(
new
 Runnable() {
      
@Override
public
void
run
()
{
        
while
 (
true
) {
          LOGGER.error(
"test log4j2 logging speed"
, 
new
 UnsupportedOperationException());
        }
      }
    });
  }
  executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
}
```

线程少的时候速度还行，线程一多很慢，问题找到了，但什么造成的这个问题呢，顺着stacktrace挖一挖：

AsyncLoggerConfig.callAppenders\(\)

```
@
Override

protected
void
callAppenders
(
final LogEvent 
event
) 
{
    
// populate lazily initialized fields
event
.getSource();
    
event
.getThreadName();

    
// pass on the event to a separate thread
if
 (!helper.callAppendersFromAnotherThread(
event
)) {
        super.callAppenders(
event
);
    }
}
```

AsyncLoggerConfigHelper.callAppendersFromAnotherThread\(\)

```
public
 boolean 
callAppendersFromAnotherThread
(
final LogEvent 
event
) 
{
    
// TODO refactor to reduce size to 
<
= 35 bytecodes to allow JVM to inline it

    final Disruptor
<
Log4jEventWrapper
>
 temp = disruptor;
    
if
 (temp == 
null
) { 
// LOG4J2-639

        LOGGER.fatal(
"Ignoring log event after log4j was shut down"
);
        
return
true
;
    }

    
// LOG4J2-471: prevent deadlock when RingBuffer is full and object
// being logged calls Logger.log() from its toString() method
if
 (isAppenderThread.
get
() == Boolean.TRUE 
//
&
&
 temp.getRingBuffer().remainingCapacity() == 
0
) {

        
// bypass RingBuffer and invoke Appender directly
return
false
;
    }
    
// LOG4J2-639: catch NPE if disruptor field was set to null after our check above
try
 {
        LogEvent logEvent = 
event
;
        
if
 (
event
 instanceof RingBufferLogEvent) {
            logEvent = ((RingBufferLogEvent) 
event
).createMemento();
        }
        logEvent.getMessage().getFormattedMessage(); 
// LOG4J2-763: ask message to freeze parameters
// Note: do NOT use the temp variable above!
// That could result in adding a log event to the disruptor after it was shut down,
// which could cause the publishEvent method to hang and never return.

        disruptor.getRingBuffer().publishEvent(translator, logEvent, asyncLoggerConfig);
    } 
catch
 (final NullPointerException npe) {
        LOGGER.fatal(
"Ignoring log event after log4j was shut down."
);
    }
    
return
true
;
}
```

RingBuffer.publishEvent\(\)

```
@Override
public
<
A, B
>
void
publishEvent
(EventTranslatorTwoArg
<
E, A, B
>
 translator, A arg0, B arg1)
{
    
final
long
 sequence = sequencer.next();
    translateAndPublish(translator, sequence, arg0, arg1);
}
```

MultiProducerSequencer.next\(\)

```
@
Override

public
long
next
(
int
 n
)
{
    
if
 (n 
<
1
){
        
throw
new
 IllegalArgumentException(
"n must be 
>
 0"
);
    }

    
long
 current;
    
long
 next;

    
do
{
        current = cursor.
get
();
        next = current + n;

        
long
 wrapPoint = next - bufferSize;
        
long
 cachedGatingSequence = gatingSequenceCache.
get
();

        
if
 (wrapPoint 
>
 cachedGatingSequence || cachedGatingSequence 
>
 current){
            
long
 gatingSequence = Util.getMinimumSequence(gatingSequences, current);

            
if
 (wrapPoint 
>
 gatingSequence){
                LockSupport.parkNanos(
1
); 
// TODO, should we spin based on the wait strategy?
continue
;
            }

            gatingSequenceCache.
set
(gatingSequence);
        }
else
if
 (
cursor.compareAndSet(current, next
))
{
            
break
;
        }
    }
while
 (
true
);

    
return
 next;
}
```

整个流程下来就是说在消费速度跟不上生产速度的时候，生产线程做了无限重试，重试间隔为1 nano，这个1 nano 会导致线程被频繁休眠／唤醒，造成kernal cpu 利用率高，context switch已经达到了cpu的极限，进而导致写log的线程的线程获取cpu时间少，吞吐量下降。

查了下log4j的bug，发现：[https://github.com/apache/logging-log4j2/commit/7a7f5e4ed1ce8a27357a12a06d32ca2b04e5eb56](https://github.com/apache/logging-log4j2/commit/7a7f5e4ed1ce8a27357a12a06d32ca2b04e5eb56)

if this fails because the queue is full, then fall back to asking AsyncEventRouter what to do with the event，

把log4j2版本切到2.7， 跑一下上面的测试，发现性能上来了， context swtich也有了数量级的下降，看看怎么办到的：

AsyncLoggerConfig.callAppenders\(\)

```
@
Override

protected
void
callAppenders
(
final LogEvent 
event
) 
{
    populateLazilyInitializedFields(
event
);
    
if
 (!
delegate
.tryEnqueue(
event
, 
this
)) {
        final EventRoute eventRoute = 
delegate
.getEventRoute(
event
.getLevel());
        eventRoute.logMessage(
this
, 
event
);
    }
}
```

AsyncLoggerConfigDisruptor.getEventRoute\(\)

```
@Override
public
 EventRoute 
getEventRoute
(
final
 Level logLevel)
{
    
final
int
 remainingCapacity = remainingDisruptorCapacity();
    
if
 (remainingCapacity 
<
0
) {
        
return
 EventRoute.DISCARD;
    }
    
return
 asyncQueueFullPolicy.getRoute(backgroundThreadId, logLevel);
}
```

DefaultAsyncQueueFullPolicy.getRoute\(\)

```
@Override
public
 EventRoute 
getRoute
(
final
long
 backgroundThreadId, 
final
 Level level)
{

    
// LOG4J2-1518: prevent deadlock when RingBuffer is full and object being logged calls
// Logger.log in application thread
// See also LOG4J2-471: prevent deadlock when RingBuffer is full and object
// being logged calls Logger.log() from its toString() method in background thread
return
 EventRoute.SYNCHRONOUS;
}
```

没有了park，被block的线程没有一直被调度，context switch减少了，kernel cpu下降了，真正获取到lock的线程获得了更多的cpu用来干活了。

### log4j的性能 {#log4j的性能}

一想到性能，有哪些是对java程序影响大的呢？ 我们直观地会认为反射、编解码，这些东西对性能影响比较大。  
使用JProfiler进行分析后，一些结果却让人吃惊不小，最耗CPU的居然是以下函数  
InetSocketAddress.getHostName\(\)  
Log.info  
String.format  
String.replace  
Gson.fromJson  
把log关闭后，InetSocketAddress.getHostName\(\)的居然占到了惊人的27%！  
![](https://images2018.cnblogs.com/blog/1081851/201803/1081851-20180319214709040-1108477150.png)

当然，性能并不是最重要的指标，相比于日志能带来的定位线上问题的好处，这一点性能的损耗其实不值一提，毕竟业务开发中，效率和稳定性才是最重要的。

