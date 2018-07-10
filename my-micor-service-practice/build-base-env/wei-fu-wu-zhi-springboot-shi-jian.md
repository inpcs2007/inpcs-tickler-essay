# SpringBoot中集成Log4j2日志框架

### 什么是Log4j

Log4j是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

### 什么是Log4j2



### Log4j2的调用

引入的包

```
import org.apache.logging.log4j.Level;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
```

静态变量

```
private static Logger logger = LogManager.getLogger(Test.class.getName());
```

### 参考资料：

https://www.cnblogs.com/lzb1096101803/p/5796849.html



