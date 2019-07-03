---
title: 运维 Java中jvm设置
date: 2017-04-15 22:08:55
tags:
  - Jvm
categories:
  - 运维
---
最近遇到java程序启动后，报错如下：
```python
Exception in thread "catalina-exec-1190" java.lang.OutOfMemoryError: PermGen space
Exception in thread "catalina-exec-1198" java.lang.OutOfMemoryError: PermGen space
Exception in thread "catalina-exec-102" java.lang.OutOfMemoryError: PermGen space
Exception in thread "catalina-exec-397" java.lang.OutOfMemoryError: PermGen space
Exception in thread "catalina-exec-709" java.lang.OutOfMemoryError: PermGen space
```
经查询发现是MaxPermSize设置太小，调整后恢复正常。之后查询java内存溢出的常见报错情况，整理如下：
常见的Java内存溢出有以下三种：
1. java.lang.OutOfMemoryError: Java heap space ----JVM Heap（堆）溢出
JVM在启动的时候会自动设置JVM Heap的值，其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)不可超过物理内存。
可以利用JVM提供的-Xmn -Xms -Xmx等选项可进行设置。Heap的大小是Young Generation 和Tenured Generaion 之和。
在JVM中如果98％的时间是用于GC，且可用的Heap size 不足2％的时候将抛出此异常信息。
`解决方法`：手动设置JVM Heap（堆）的大小

2. java.lang.OutOfMemoryError: PermGen space  ---- PermGen space溢出。
PermGen space的全称是Permanent Generation space，是指内存的永久保存区域。
为什么会内存溢出，这是由于这块内存主要是被JVM存放Class和Meta信息的，Class在被Load的时候被放入PermGen space区域，它和存放Instance的Heap区域不同,sun的 GC不会在主程序运行期对PermGen space进行清理，所以如果你的APP会载入很多CLASS的话，就很可能出现PermGen space溢出。
`解决方法`： 手动设置MaxPermSize大小

3. java.lang.StackOverflowError   ---- 栈溢出
栈溢出了，JVM依然是采用栈式的虚拟机，这个和C和Pascal都是一样的。函数的调用过程都体现在堆栈和退栈上了。
调用构造函数的 “层”太多了，以致于把栈区溢出了。
通常来讲，一般栈区远远小于堆区的，因为函数调用过程往往不会多于上千层，而即便每个函数调用需要 1K的空间(这个大约相当于在一个C函数内声明了256个int类型的变量)，那么栈区也不过是需要1MB的空间。通常栈的大小是1－2MB的。
通常递归也不要递归的层次过多，很容易溢出。 
`解决方法`：修改程序

其他，设置jvm内存使用方法：
linux下的tomcat：  
修改TOMCAT_HOME/bin/catalina.sh
位置cygwin=false前
```python
JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms4096m -Xmx4096m -XX:NewSize=512m -XX:MaxNewSize=512m -XX:PermSize=512m -XX:MaxPermSize=512m -XX:+DisableExplicitGC"  
```

jvm参数说明：
-server:一定要作为第一个参数，在多个CPU时性能佳
-Xms：java Heap初始大小。 默认是物理内存的1/64
-Xmx：java heap最大值。建议均设为物理内存的一半，不可超过物理内存。
-XX:PermSize:设定内存的永久保存区初始大小，缺省值为64M。
-XX:MaxPermSize:设定内存的永久保存区最大 大小，缺省值为64M。
-XX:SurvivorRatio=2  :生还者池的大小,默认是2，如果垃圾回收变成了瓶颈，您可以尝试定制生成池设置
-XX:NewSize: 新生成的池的初始大小。 缺省值为2M。
-XX:MaxNewSize: 新生成的池的最大大小。   缺省值为32M。
如果 JVM 的堆大小大于 1GB，则应该使用值：-XX:newSize=640m -XX:MaxNewSize=640m -XX:SurvivorRatio=16，或者将堆的总大小的 50% 到 60% 分配给新生成的池。调大新对象区，减少Full GC次数。
+XX:AggressiveHeap 会使得 Xms没有意义。这个参数让jvm忽略Xmx参数,疯狂地吃完一个G物理内存,再吃尽一个G的swap。
-Xss：每个线程的Stack大小，“-Xss 15120” 这使得JBoss每增加一个线程（thread)就会立即消耗15M内存，而最佳值应该是128K,默认值好像是512k.
-verbose:gc 现实垃圾收集信息
-Xloggc:gc.log 指定垃圾收集日志文件
-Xmn：young generation的heap大小，一般设置为Xmx的3、4分之一
-XX:+UseParNewGC ：缩短minor收集的时间
-XX:+UseConcMarkSweepGC ：缩短major收集的时间 此选项在Heap Size 比较大而且Major收集时间较长的情况下使用更合适。
-XX:userParNewGC 可用来设置并行收集【多CPU】
-XX:ParallelGCThreads 可用来增加并行度【多CPU】
-XX:UseParallelGC 设置后可以使用并行清除收集器【多CPU】

Tomcat并发优化
1.Tomcat连接相关参数
在Tomcat配置文件conf下面 server.xml 中的配置中和连接数相关的参数有：
minProcessors：最小空闲连接线程数，用于提高系统处理性能，默认值为10
maxProcessors：最大连接线程数，即：并发处理的最大请求数，默认值为75
acceptCount：允许的最大连接数，应大于等于maxProcessors，默认值为100
enableLookups：是否反查域名，取值为：true或false。为了提高处理能力，应设置为false
connectionTimeout：网络连接超时，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。
默认的tomcat 参数：
```python
<Connector port=“8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```
修改为：
```python
<Connector port=“8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
  maxThreads="600"
  minSpareThreads="100"
  maxSpareThreads="500"
  acceptCount="700"
  connectionTimeout="20000"
  redirectPort="8443" />
```
相关说明：
protocol="org.apache.coyote.http11.Http11NioProtocol" ///使用java的异步io护理技术,no blocking IO
maxThreads=“600" 表示最多同时处理600个连接 ///最大线程数
minSpareThreads=“100" 表示即使没有人使用也开这么多空线程等待  ///初始化时创建的线程数
maxSpareThreads=“500" 表示如果最多可以空500个线程，例如某时刻有505人访问，之后没有人访问了，则tomcat不会保留505个空线程，而是关闭505个空的。   ///一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。
acceptCount="700"//指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理

这里是http connector的优化，如果使用apache和tomcat做集群的负载均衡，并且使用ajp协议做apache和tomcat的协议转发，那么还需要优化ajp connector。
<Connector port="8009" protocol="AJP/1.3" maxThreads="600" minSpareThreads="100" maxSpareThreads="500" acceptCount="700" connectionTimeout="20000" redirectPort="8443" />







