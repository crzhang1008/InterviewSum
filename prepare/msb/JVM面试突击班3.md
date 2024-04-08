# JVM面试突击班3                                         主讲人：严镇涛

## 你接触过哪些垃圾收集器，聊一下

> **如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\30.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/6f9efd6ca5b246629b25dbfec3d4a16c.png)

* Serial

**Serial收集器是最基本、发展历史最悠久的收集器，曾经（在JDK1.3.1之前）是虚拟机新生代收集的唯一选择。**

**它是一种单线程收集器，不仅仅意味着它只会使用一个CPU或者一条收集线程去完成垃圾收集工作，更重要的是其在进行垃圾收集的时候需要暂停其他线程。**

```
优点：简单高效，拥有很高的单线程收集效率
缺点：收集过程需要暂停所有线程
算法：复制算法
适用范围：新生代
应用：Client模式下的默认新生代收集器
```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\31.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/e61e1fe960534b26b848708a753e05d6.png)

* Serial Old

Serial Old收集器是Serial收集器的老年代版本，也是一个单线程收集器，不同的是采用"**标记-整理算法**"，运行过程和Serial收集器一样。

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\32.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/fb0e283d58494cc9993355a1fa6ddbee.png)

早期的垃圾收集器为什么要设计成单线程？ 单核CPU  嵌入式  STW

* ParNew

**可以把这个收集器理解为Serial收集器的多线程版本。**

```
优点：在多CPU时，比Serial效率高。
缺点：收集过程暂停所有应用程序线程，单CPU时比Serial效率差。
算法：复制算法
适用范围：新生代
应用：运行在Server模式下的虚拟机中首选的新生代收集器
```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\33.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/6ff517bede0043a2b37ff35c2b85f45e.png)

* Parallel Scavenge           STW    单位时间内接受请求并响应的数量  能不能控制

**Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集器，看上去和ParNew一样，但是Parallel Scanvenge更关注系统的吞吐量**。

> **吞吐量=运行用户代码的时间/(运行用户代码的时间+垃圾收集时间)**
>
> **比如虚拟机总共运行了100分钟，垃圾收集时间用了1分钟，吞吐量=(100-1)/100=99%。**
>
> **若吞吐量越大，意味着垃圾收集的时间越短，则用户代码可以充分利用CPU资源，尽快完成程序的运算任务。**

```
-XX:MaxGCPauseMillis控制最大的垃圾收集停顿时间，
-XX:GCRatio直接设置吞吐量的大小。
```

* Parallel Old

**Parallel Old收集器是Parallel Scavenge收集器的老年代版本，使用多线程和标记-整理算法**进行垃圾回收，也是更加关注系统的**吞吐量**。

吞吐量                           停顿时间

95%-96%以上                             young  GC        full  GC   不能小于一天/次

业务代码  + 垃圾收集线程      并发类垃圾收集器

* CMS

> `官网`： [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector)
>
> **CMS(Concurrent Mark Sweep)收集器是一种以获取** `最短回收停顿时间`为目标的收集器。
>
> **采用的是"标记-清除算法",整个过程分为4步**

我们的CMS用什么方式尽可能的节省垃圾收集的时间

```
(1)初始标记 CMS initial mark     标记GC Roots直接关联对象，不用Tracing，速度很快    不耗时       STW
(2)并发标记 CMS concurrent mark  进行GC Roots Tracing         耗时    并发
(3)重新标记 CMS remark           修改并发标记因用户程序变动的内容     不耗时       STW
(4)并发清除 CMS concurrent sweep 清除不可达对象回收空间，同时有新垃圾产生，留着下次清理称为浮动垃圾
```

> **由于整个过程中，并发标记和并发清除，收集器线程可以与用户线程一起工作，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发地执行的。**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\34.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/51a12718671b4d368a4721cb7cff4b5c.png)

```
优点：并发收集、低停顿
缺点：产生大量空间碎片、并发阶段会降低吞吐量

```

## 什么是记忆集？

当我们进行young gc时，我们的**gc roots除了常见的栈引用、静态变量、常量、锁对象、class对象**这些常见的之外，如果 **老年代有对象引用了我们的新生代对象** ，那么老年代的对象也应该加入gc roots的范围中，但是如果每次进行young gc我们都需要扫描一次老年代的话，那我们进行垃圾回收的代价实在是太大了，因此我们引入了一种叫做记忆集的抽象数据结构来记录这种引用关系。

记忆集是一种用于记录从非收集区域指向收集区域的指针集合的数据结构。

```
如果我们不考虑效率和成本问题，我们可以用一个数组存储所有有指针指向新生代的老年代对象。但是如果这样的话我们维护成本就很好，打个比方，假如所有的老年代对象都有指针指向了新生代，那么我们需要维护整个老年代大小的记忆集，毫无疑问这种方法是不可取的。因此我们引入了卡表的数据结构
```

### 卡表

记忆集是我们针对于跨代引用问题提出的思想，而卡表则是针对于该种思想的具体实现。（可以理解为记忆集是结构，卡表是实现类）

[1字节，00001000，1字节，1字节]

```
在hotspot虚拟机中，卡表是一个字节数组，数组的每一项对应着内存中的某一块连续地址的区域，如果该区域中有引用指向了待回收区域的对象，卡表数组对应的元素将被置为1，没有则置为0；
```

(1)  卡表是使用一个字节数组实现:CARD_TABLE[],每个元素对应着其标识的内存区域一块特定大小的内存块,称为"卡页"。hotSpot使用的卡页是2^9大小,即512字节

(2)  一个卡页中可包含多个对象,只要有一个对象的字段存在跨代指针,其对应的卡表的元素标识就变成1,表示该元素变脏,否则为0。GC时,只要筛选本收集区的卡表中变脏的元素加入GC Roots里。

卡表的使用图例

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1652270310036/e428c317c70e4f01945955cd7f93a4d9.png)

并发标记的时候，A对象发生了所在的引用发生了变化，所以A对象所在的块被标记为脏卡

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1652270310036/ca90c6933c49450eba6bbb085a90bada.png)

继续往下到了重新标记阶段，修改对象的引用，同时清除脏卡标记。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1652270310036/007dd60e3a58435a9880aede2b95b47e.png)

**卡表其他作用：**

老年代识别新生代的时候

对应的card table被标识为相应的值（card table中是一个byte，有八位，约定好每一位的含义就可区分哪个是引用新生代，哪个是并发标记阶段修改过的）

* G1(Garbage-First)     JDK8推荐使用的   JDK9默认的   JDK7的最后的维护版
  要比CMS的停顿时间短    你想多短就多短             优先回收垃圾价值高的区域
  它可以某种程度上解决空间碎片的问题

> `官网`： [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection)
>
> **使用G1收集器时，Java堆的内存布局与就与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region）2048个，虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分Region（不需要连续）的集合。 **
>
> **每个Region大小都是一样的，可以是1M到32M之间的数值，但是必须保证是2的n次幂**
>
> **如果对象太大，一个Region放不下[超过Region大小的50%]，那么就会直接放到H中**
>
> **设置Region大小：-XX:G1HeapRegionSize=**<N>**M**
>
> **所谓Garbage-Frist，其实就是优先回收垃圾最多的Region区域**
>
> ```
> （1）分代收集（仍然保留了分代的概念）
> （2）空间整合（整体上属于“标记-整理”算法，不会导致空间碎片）
> （3）可预测的停顿（比CMS更先进的地方在于能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒）
> ```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\35.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/943614eae58f4649b9210c32fb2f8f98.png)

**工作过程可以分为如下几步**

```
初始标记（Initial Marking）      标记以下GC Roots能够关联的对象，并且修改TAMS的值，需要暂停用户线程
并发标记（Concurrent Marking）   从GC Roots进行可达性分析，找出存活的对象，与用户线程并发执行
最终标记（Final Marking）        修正在并发标记阶段因为用户程序的并发执行导致变动的数据，需暂停用户线程
筛选回收（Live Data Counting and Evacuation） 对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间制定回收计划  
```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\36.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/a04fb27a0ef64f8ca25a96eb95d44d71.png)

* ZGC

> `官网`： [https://docs.oracle.com/en/java/javase/11/gctuning/z-garbage-collector1.html#GUID-A5A42691-095E-47BA-B6DC-FB4E5FAA43D0](https://docs.oracle.com/en/java/javase/11/gctuning/z-garbage-collector1.html#GUID-A5A42691-095E-47BA-B6DC-FB4E5FAA43D0)
>
> **JDK11新引入的ZGC收集器，不管是物理上还是逻辑上，ZGC中已经不存在新老年代的概念了**
>
> **会分为一个个page，当进行GC操作时会对page进行压缩，因此没有碎片问题**
>
> **只能在64位的linux上使用，目前用得还比较少**

**（1）可以达到10ms以内的停顿时间要求**

**（2）支持TB级别的内存**

**（3）堆内存变大后停顿时间还是在10ms以内**

## JVM常用参数有哪些？

## JVM参数

### 3.1.1 标准参数

```
-version
-help
-server
-cp
```

![](images/37.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1677915268044/aad767aa05404131b5617b415764f6ec.png)

### 3.1.2 -X参数

> 非标准参数，也就是在JDK各个版本中可能会变动

```
-Xint     解释执行
-Xcomp    第一次使用就编译成本地代码
-Xmixed   混合模式，JVM自己来决定
```

![](images/38.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1677915268044/d65086d93881426483bda40f576cd219.png)

### 3.1.3 -XX参数

> 使用得最多的参数类型
>
> 非标准化参数，相对不稳定，主要用于JVM调优和Debug

```
a.Boolean类型
格式：-XX:[+-]<name>            +或-表示启用或者禁用name属性
比如：-XX:+UseConcMarkSweepGC   表示启用CMS类型的垃圾回收器
	 -XX:+UseG1GC              表示启用G1类型的垃圾回收器
b.非Boolean类型
格式：-XX<name>=<value>表示name属性的值是value
比如：-XX:MaxGCPauseMillis=500   
```

### 3.1.4 其他参数

```
-Xms1000M等价于-XX:InitialHeapSize=1000M
-Xmx1000M等价于-XX:MaxHeapSize=1000M
-Xss100等价于-XX:ThreadStackSize=100
```

> 所以这块也相当于是-XX类型的参数

### 3.1.5 查看参数

> java -XX:+PrintFlagsFinal -version > flags.txt

![](images/39.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1677915268044/62dcc3fda07d4e71b5c0a63ab80a410c.png)

![](images/40.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1677915268044/2c05bca3c4f344c497074dd768d1efef.png)

> 值得注意的是"="表示默认值，":="表示被用户或JVM修改后的值
> 要想查看某个进程具体参数的值，可以使用jinfo，这块后面聊
> 一般要设置参数，可以先查看一下当前参数是什么，然后进行修改

### 3.1.6 设置参数的常见方式

* 开发工具中设置比如IDEA，eclipse
* 运行jar包的时候:java  -XX:+UseG1GC xxx.jar
* web容器比如tomcat，可以在脚本中的进行设置
* 通过jinfo实时调整某个java进程的参数(参数只有被标记为manageable的flags可以被实时修改)

### 3.1.7 实践和单位换算

```
1Byte(字节)=8bit(位)
1KB=1024Byte(字节)
1MB=1024KB
1GB=1024MB
1TB=1024GB
```

```
(1)设置堆内存大小和参数打印
-Xmx100M -Xms100M -XX:+PrintFlagsFinal
(2)查询+PrintFlagsFinal的值
:=true
(3)查询堆内存大小MaxHeapSize
:= 104857600
(4)换算
104857600(Byte)/1024=102400(KB)
102400(KB)/1024=100(MB)
(5)结论
104857600是字节单位
```

### 3.1.8 常用参数含义

| 参数                                                                                |                                                 含义                                                 |                                                                    说明                                                                    |
| :---------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------: |
| -XX:CICompilerCount=3                                                               |                                            最大并行编译数                                            |                               如果设置大于1，虽然编译速度会提高，但是同样影响系统稳定性，会增加JVM崩溃的可能                               |
| -XX:InitialHeapSize=100M                                                            |                                             初始化堆大小                                             |                                                                简写-Xms100M                                                                |
| -XX:MaxHeapSize=100M                                                                |                                              最大堆大小                                              |                                                                简写-Xms100M                                                                |
| -XX:NewSize=20M                                                                     |                                           设置年轻代的大小                                           |                                                                                                                                            |
| -XX:MaxNewSize=50M                                                                  |                                            年轻代最大大小                                            |                                                                                                                                            |
| -XX:OldSize=50M                                                                     |                                            设置老年代大小                                            |                                                                                                                                            |
| -XX:MetaspaceSize=50M                                                               |                                            设置方法区大小                                            |                                                                                                                                            |
| -XX:MaxMetaspaceSize=50M                                                            |                                            方法区最大大小                                            |                                                                                                                                            |
| -XX:+UseParallelGC                                                                  |                                           使用UseParallelGC                                           |                                                             新生代，吞吐量优先                                                             |
| -XX:+UseParallelOldGC                                                               |                                         使用UseParallelOldGC                                         |                                                             老年代，吞吐量优先                                                             |
| -XX:+UseConcMarkSweepGC                                                             |                                                使用CMS                                                |                                                            老年代，停顿时间优先                                                            |
| -XX:+UseG1GC                                                                        |                                               使用G1GC                                               |                                                        新生代，老年代，停顿时间优先                                                        |
| -XX:NewRatio                                                                        |                                            新老生代的比值                                            |                                   比如-XX:Ratio=4，则表示新生代:老年代=1:4，也就是新生代占整个堆内存的1/5                                   |
| -XX:SurvivorRatio                                                                   |                                         两个S区和Eden区的比值                                         |                               比如-XX:SurvivorRatio=8，也就是(S0+S1):Eden=2:8，也就是一个S占整个新生代的1/10                               |
| -XX:+HeapDumpOnOutOfMemoryError                                                     |                                          启动堆内存溢出打印                                          |                                             当JVM堆内存发生溢出时，也就是OOM，自动生成dump文件                                             |
| -XX:HeapDumpPath=heap.hprof                                                         |                                        指定堆内存溢出打印目录                                        |                                                    表示在当前目录生成一个heap.hprof文件                                                    |
| -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:g1-gc.log |                                             打印出GC日志                                             |                                                  可以使用不同的垃圾收集器，对比查看GC情况                                                  |
| -Xss128k                                                                            |                                        设置每个线程的堆栈大小                                        |                                                            经验值是3000-5000最佳                                                            |
| -XX:MaxTenuringThreshold=6                                                          |                                        提升年老代的最大临界值                                        |                                                                 默认值为 15                                                                 |
| -XX:InitiatingHeapOccupancyPercent                                                  |                                    启动并发GC周期时堆内存使用占比                                    |     G1之类的垃圾收集器用它来触发并发GC周期,基于整个堆的使用率,而不只是某一代内存的使用比. 值为 0 则表示”一直执行GC循环”. 默认值为 45.     |
| -XX:G1HeapWastePercent                                                              |                                        允许的浪费堆空间的占比                                        |                                       默认是10%，如果并发标记可回收的空间小于10%,则不会触发MixedGC。                                       |
| -XX:MaxGCPauseMillis=200ms                                                          |                                            G1最大停顿时间                                            | 暂停时间不能太小，太小的话就会导致出现G1跟不上垃圾产生的速度。最终退化成Full GC。所以对这个参数的调优是一个持续的过程，逐步调整到最佳状态。 |
| -XX:ConcGCThreads=n                                                                 |                                     并发垃圾收集器使用的线程数量                                     |                                                       默认值随JVM运行的平台不同而不同                                                       |
| -XX:G1MixedGCLiveThresholdPercent=65                                                |                            混合垃圾回收周期中要包括的旧区域设置占用率阈值                            |                                                              默认占用率为 65%                                                              |
| -XX:G1MixedGCCountTarget=8                                                          | 设置标记周期完成后，对存活数据上限为 G1MixedGCLIveThresholdPercent 的旧区域执行混合垃圾回收的目标次数 |                                         默认8次混合垃圾回收，混合回收的目标是要控制在此目标次数以内                                         |
| -XX:G1OldCSetRegionThresholdPercent=1                                               |                               描述Mixed GC时，Old Region被加入到CSet中                               |                                                默认情况下，G1只把10%的Old Region加入到CSet中                                                |
|                                                                                     |                                                                                                      |                                                                                                                                            |

## JVM常用命令有哪些

### jps

> 查看java进程

```
The jps command lists the instrumented Java HotSpot VMs on the target system. The command is limited to reporting information on JVMs for which it has the access permissions.
```

![](images/41.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/fcaee39e479c4d078f16ecb325619c53.png)

### jinfo

> （1）实时查看和调整JVM配置参数

```
The jinfo command prints Java configuration information for a specified Java process or core file or a remote debug server. The configuration information includes Java system properties and Java Virtual Machine (JVM) command-line flags.
```

> （2）查看用法
>
> jinfo -flag name PID     查看某个java进程的name属性的值

```
jinfo -flag MaxHeapSize PID 
jinfo -flag UseG1GC PID
```

![](images/42.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/4be28005e8a244f7b357d277a9244dbe.png)

> （3）修改
>
> **参数只有被标记为manageable的flags可以被实时修改**

```
jinfo -flag [+|-] PID
jinfo -flag <name>=<value> PID
```

> （4）查看曾经赋过值的一些参数

```
jinfo -flags PID
```

![](images/43.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/01493e1542304c32af3dd5e5dfdcaf39.png)

### jstat

> （1）查看虚拟机性能统计信息

```
The jstat command displays performance statistics for an instrumented Java HotSpot VM. The target JVM is identified by its virtual machine identifier, or vmid option.
```

> （2）查看类装载信息

```
jstat -class PID 1000 10   查看某个java进程的类装载信息，每1000毫秒输出一次，共输出10次
```

![](images/44.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/f672e595715a452bb7a3abd3301c74d0.png)

> （3）查看垃圾收集信息

```
jstat -gc PID 1000 10
```

![](images/45.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/06ce7f95993346b283c7f2f86fe1516f.png)

### jstack

> （1）查看线程堆栈信息

```
The jstack command prints Java stack traces of Java threads for a specified Java process, core file, or remote debug server.
```

> （2）用法

```
jstack PID
```

![](images/46.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/f3a09e00012043baafea183e17291d48.png)

> (4)排查死锁案例

* DeadLockDemo

```java
//运行主类
public class DeadLockDemo
{
    public static void main(String[] args)
    {
        DeadLock d1=new DeadLock(true);
        DeadLock d2=new DeadLock(false);
        Thread t1=new Thread(d1);
        Thread t2=new Thread(d2);
        t1.start();
        t2.start();
    }
}
//定义锁对象
class MyLock{
    public static Object obj1=new Object();
    public static Object obj2=new Object();
}
//死锁代码
class DeadLock implements Runnable{
    private boolean flag;
    DeadLock(boolean flag){
        this.flag=flag;
    }
    public void run() {
        if(flag) {
            while(true) {
                synchronized(MyLock.obj1) {
                    System.out.println(Thread.currentThread().getName()+"----if获得obj1锁");
                    synchronized(MyLock.obj2) {
                        System.out.println(Thread.currentThread().getName()+"----if获得obj2锁");
                    }
                }
            }
        }
        else {
            while(true){
                synchronized(MyLock.obj2) {
                    System.out.println(Thread.currentThread().getName()+"----否则获得obj2锁");
                    synchronized(MyLock.obj1) {
                        System.out.println(Thread.currentThread().getName()+"----否则获得obj1锁");

                    }
                }
            }
        }
    }
}
```

* 运行结果

![](images/47.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/518fbc7a34494f1e91d914f5b8e2de8f.png)

* jstack分析

![](images/48.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/b2a12fc243fa43248a12df586ee1df5a.png)

> 把打印信息拉到最后可以发现

![](images/49.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/9e2c9900320244d4a8ed33f3e64a117a.png)

### jmap

> （1）生成堆转储快照

```
The jmap command prints shared object memory maps or heap memory details of a specified process, core file, or remote debug server.
```

> （2）打印出堆内存相关信息

```
jmap -heap PID
```

```
jinfo -flag UsePSAdaptiveSurvivorSizePolicy 35352
-XX:SurvivorRatio=8
```

![](images/50.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/ac32aa9fcc89458eb5cac86209bdfa0a.png)

> （3）dump出堆内存相关信息

```
jmap -dump:format=b,file=heap.hprof PID
```

![](images/51.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655274390025/837bbf874f234b6f9a2f432c31eedbc6.png)

> （4）要是在发生堆内存溢出的时候，能自动dump出该文件就好了

一般在开发中，JVM参数可以加上下面两句，这样内存溢出时，会自动dump出该文件

-XX:+HeapDumpOnOutOfMemoryError     -XX:HeapDumpPath=heap.hprof

```
设置堆内存大小: -Xms20M -Xmx20M
启动，然后访问localhost:9090/heap，使得堆内存溢出
```

## 你会估算GC频率吗?

正常情况我们应该根据我们的系统来进行一个内存的估算，这个我们可以在测试环境进行测试，最开始可以将内存设置的大一些，比如4G这样，当然这也可以根据业务系统估算来的。
比如从数据库获取一条数据占用128个字节，需要获取1000条数据，那么一次读取到内存的大小就是(（128 B/1024 Kb/1024M）* 1000 = 0.122M，那么我们程序可能需要并发读取，比如每秒读取100次，那么内存占用就是0.122100 = 12.2M，如果堆内存设置1个G，那么年轻代大小大约就是333M，那么333M*80%/12.2M =21.84s，也就是说我们的程序几乎每分钟进行两到三次youngGC。这样可以让我们对系统有一个大致的估算。

## 内存优化

### 4.1.1 内存分配

> 正常情况下不需要设置，那如果是促销或者秒杀的场景呢？
>
> 每台机器配置2c4G，以每秒3000笔订单为例，整个过程持续60秒

![](images/59.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1655967490073/aeab3eedd1e64d5cb02d7743f2f0252e.png)

### 4.1.2 内存溢出(OOM)

> 一般会有两个原因：
>
> （1）大并发情况下
>
> （2）内存泄露导致内存溢出

#### 4.1.2.1 大并发[秒杀]

浏览器缓存、本地缓存、验证码

CDN静态资源服务器

集群+负载均衡

动静态资源分离、限流[基于令牌桶、漏桶算法]

应用级别缓存、接口防刷限流、队列、Tomcat性能优化

异步消息中间件

Redis热点数据对象缓存

分布式锁、数据库锁

5分钟之内没有支付，取消订单、恢复库存等

#### 4.1.2.2 内存泄露导致内存溢出

> ThreadLocal引起的内存泄露，最终导致内存溢出
>
> ```Java
> public class TLController {
>  @RequestMapping(value = "/tl")
>  public String tl(HttpServletRequest request) {
>      ThreadLocal<Byte[]> tl = new ThreadLocal<Byte[]>();
>      // 1MB
>      tl.set(new Byte[1024*1024]);
>      return "ok";
>  }
> }
> ```

（1）上传到阿里云服务器

jvm-case-0.0.1-SNAPSHOT.jar

（2）启动

```
java -jar -Xms1000M -Xmx1000M -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=jvm.hprof  jvm-case-0.0.1-SNAPSHOT.jar
```

（3）使用jmeter模拟10000次并发

39.100.39.63:8080/tl

（4）top命令查看

```
top
top -Hp PID
```

（5）jstack查看线程情况，发现没有死锁或者IO阻塞的情况

```
jstack PID
java -jar arthas.jar   --->   thread
```

（6）查看堆内存的使用，发现堆内存的使用率已经高达88.95%

```
jmap -heap PID
java -jar arthas.jar   --->   dashboard
```

（7）此时可以大体判断出来，发生了内存泄露从而导致的内存溢出，那怎么排查呢？

```
jmap -histo:live PID | more
获取到jvm.hprof文件，上传到指定的工具分析，比如heaphero.io
```

GC的垃圾回收条件模式图：

https://www.processon.com/view/link/62bc50e47d9c08073522779c
