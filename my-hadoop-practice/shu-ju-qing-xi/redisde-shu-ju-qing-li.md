# [Redis的数据清理](http://lxw1234.com/archives/2017/07/874.htm)

我们数据平台中有使用Redis来给线上提供低延时（20毫秒以内）的高并发读写请求，其中最大的Redis使用了阿里云的Redis集群（256G），存储的记录超过10亿，Key的有效期设置为15天，每天写入的记录大概5000万左右，QPS大概在6万左右。由于过期Key的产生速度大于Redis自动清理的速度，因此在Redis中会有大量过期Key未被及时清理。

为什么有过期的Key未被清理呢？这个得先熟悉一下Redis的删除策略。

Redis常用的删除策略有以下三种：

* 被动删除（惰性删除）：当读/写一个已经过期的Key时，会触发惰性删除策略，直接删除掉这个Key;
* 主动删除（定期删除）：Redis会定期巡检，来清理过期Key；
* 当内存达到maxmemory配置时候，会触发Key的删除操作；

另外，还有一种基于触发器的删除策略，因为对Redis压力太大，一般没人使用。

这里先介绍后两种删除策略（网上有很多说明）。

参考：http://www.cnblogs.com/chenpingzhao/p/5022467.html

## 主动删除（定期删除）

在 Redis 中，常规操作由 redis.c/serverCron 实现，它主要执行以下操作：

* 更新服务器的各类统计信息，比如时间、内存占用、数据库占用情况等。
* 清理数据库中的过期键值对。
* 对不合理的数据库进行大小调整。
* 关闭和清理连接失效的客户端。
* 尝试进行 AOF 或 RDB 持久化操作。
* 如果服务器是主节点的话，对附属节点进行定期同步。
* 如果处于集群模式的话，对集群进行定期同步和连接测试。

Redis 将 serverCron 作为时间事件来运行，从而确保它每隔一段时间就会自动运行一次， 又因为 serverCron 需要在 Redis 服务器运行期间一直定期运行， 所以它是一个循环时间事件：serverCron 会一直定期执行，直到服务器关闭为止。

在 Redis 2.6 版本中， 程序规定 serverCron 每秒运行 10 次， 平均每 100 毫秒运行一次。 从 Redis 2.8 开始， 用户可以通过修改 hz选项来调整 serverCron 的每秒执行次数， 具体信息请参考 redis.conf 文件中关于 hz 选项的说明。

---

也叫定时删除，这里的“定期”指的是Redis定期触发的清理策略，由位于src/redis.c的activeExpireCycle\(void\)函数来完成。

serverCron是由redis的事件框架驱动的定位任务，这个定时任务中会调用activeExpireCycle函数，针对每个db在限制的时间REDIS\_EXPIRELOOKUPS\_TIME\_LIMIT内迟可能多的删除过期key，之所以要限制时间是为了防止过长时间 的阻塞影响redis的正常运行。这种主动删除策略弥补了被动删除策略在内存上的不友好。

因此，Redis会周期性的随机测试一批设置了过期时间的key并进行处理。测试到的已过期的key将被删除。典型的方式为,Redis每秒做10次如下的步骤：

* 随机测试100个设置了过期时间的key
* 删除所有发现的已过期的key
* 若删除的key超过25个则重复步骤1

这是一个基于概率的简单算法，基本的假设是抽出的样本能够代表整个key空间，redis持续清理过期的数据直至将要过期的key的百分比降到了25%以下。这也意味着在任何给定的时刻已经过期但仍占据着内存空间的key的量最多为每秒的写操作量除以4.

Redis-3.0.0中的默认值是10，代表每秒钟调用10次后台任务。

除了主动淘汰的频率外，Redis对每次淘汰任务执行的最大时长也有一个限定，这样保证了每次主动淘汰不会过多阻塞应用请求，以下是这个限定计算公式：

\#define ACTIVE\_EXPIRE\_CYCLE\_SLOW\_TIME\_PERC 25 /\* CPU max % for keys collection \*/

…

timelimit = 1000000\*ACTIVE\_EXPIRE\_CYCLE\_SLOW\_TIME\_PERC/server.hz/100;

hz调大将会提高Redis主动淘汰的频率，如果你的Redis存储中包含很多冷数据占用内存过大的话，可以考虑将这个值调大，但Redis作者建议这个值不要超过100。我们实际线上将这个值调大到100，观察到CPU会增加2%左右，但对冷数据的内存释放速度确实有明显的提高（通过观察keyspace个数和used\_memory大小）。

可以看出timelimit和server.hz是一个倒数的关系，也就是说hz配置越大，timelimit就越小。换句话说是每秒钟期望的主动淘汰频率越高，则每次淘汰最长占用时间就越短。这里每秒钟的最长淘汰占用时间是固定的250ms（1000000\*ACTIVE\_EXPIRE\_CYCLE\_SLOW\_TIME\_PERC/100），而淘汰频率和每次淘汰的最长时间是通过hz参数控制的。

从以上的分析看，当redis中的过期key比率没有超过25%之前，提高hz可以明显提高扫描key的最小个数。假设hz为10，则一秒内最少扫描200个key（一秒调用10次\*每次最少随机取出20个key），如果hz改为100，则一秒内最少扫描2000个key；另一方面，如果过期key比率超过25%，则扫描key的个数无上限，但是cpu时间每秒钟最多占用250ms。

当REDIS运行在主从模式时，只有主结点才会执行上述这两种过期删除策略，然后把删除操作”del key”同步到从结点。

## maxmemory

当前已用内存超过maxmemory限定时，触发**主动清理**策略，这些策略可以配置（参数maxmemory-policy），包括以下几个：



volatile-lru：从已设置过期时间的数据集（server.db\[i\].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db\[i\].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db\[i\].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db\[i\].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db\[i\].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据

当mem\_used内存已经超过maxmemory的设定，对于所有的读写请求，都会触发redis.c/freeMemoryIfNeeded\(void\)函数以清理超出的内存。注意这个清理过程是阻塞的，直到清理出足够的内存空间。所以如果在达到maxmemory并且调用方还在不断写入的情况下，可能会反复触发主动清理策略，导致请求会有一定的延迟。



清理时会根据用户配置的maxmemory-policy来做适当的清理（一般是LRU或TTL），这里的LRU或TTL策略并不是针对redis的所有key，而是以配置文件中的maxmemory-samples个key作为样本池进行抽样清理。

## 总结与备忘

如果Redis中每天过期大量Key（比如几千万），那么必须得考虑过期Key的清理：

增加Redis主动清理的频率（通过调大hz参数）；

手动清理过期Key，最简单的方法是进行scan操作，scan操作会触发第一种被动删除，scan操作时候别忘了加count；

dbsize命令返回的Key数量，包含了过期Key；

randomkey命令返回的Key，不包含过期Key；

scan命令返回的Key，包含过期Key；

info命令返回的\# Keyspace：

db6:keys=1034937352,expires=994731489,avg\_ttl=507838502

keys对应的Key数量等同于dbsize；

expires指的是设置了过期时间的Key数量；

avg\_ttl指设置了过期时间的Key的平均过期时间（单位：毫秒）；





如果觉得本博客对您有帮助，请[赞助作者](http://lxw1234.com/pay-blog)。

转载请注明：[lxw的大数据田地](http://lxw1234.com/)»[关于Redis的数据清理](http://lxw1234.com/archives/2017/07/874.htm)



