# OutOfMemoryError系列（2）: GC overhead limit exceeded


Java runtime environment contains a built-in [Garbage Collection (GC)](https://plumbr.eu/handbook/what-is-garbage-collection) process. In many other programming languages, the developers need to manually allocate and free memory regions so that the freed memory can be reused.

Java运行时环境内置了 [垃圾收集(GC)](http://blog.csdn.net/renfufei/article/details/53432995) 模块. 上一代的很多编程语言中并没有自动内存回收机制, 需要程序员手工编写代码来进行内存分配和释放, 以重复利用堆内存。


Java applications on the other hand only need to allocate memory. Whenever a particular space in memory is no longer used, a separate process called [Garbage Collection](https://plumbr.eu/handbook/garbage-collection-in-jvm) clears the memory for them. How the GC detects that a particular part of memory is explained in more detail in the [Garbage Collection Handbook](https://plumbr.eu/java-garbage-collection-handbook), but you can trust the GC to do its job well.

在Java程序中, 只需要关心内存分配就行。如果某块内存不再使用, [垃圾收集(Garbage Collection)](http://blog.csdn.net/renfufei/article/details/54144385) 模块会自动执行清理。GC的详细原理请参考 [GC性能优化](http://blog.csdn.net/column/details/14851.html) 系列文章, 一般来说, JVM内置的垃圾收集算法就能够应对绝大多数的业务场景。


The _java.lang.OutOfMemoryError: GC overhead limit exceeded_ error is displayed when **your application has exhausted pretty much all the available memory and GC has repeatedly failed to clean it**.

_java.lang.OutOfMemoryError: GC overhead limit exceeded_ 这种情况发生的原因是, **程序基本上耗尽了所有的可用内存, GC也清理不了**。


## What is causing it?

## 原因分析


The _java.lang.OutOfMemoryError: GC overhead limit exceeded_ error is the JVM’s way of signalling that your application spends too much time doing garbage collection with too little result. By default the JVM is configured to throw this error if it spends more than **98% of the total time doing GC and when after the GC only less than 2% of the heap is recovered**.

JVM抛出 _java.lang.OutOfMemoryError: GC overhead limit exceeded_ 错误就是发出了这样的信号: 执行垃圾收集的时间比例太大, 有效的运算量太小. 默认情况下, 如果GC花费的时间超过 **98%**, 并且GC回收的内存少于 **2%**, JVM就会抛出这个错误。


![java.lang.OutOfMemoryError: GC overhead limit exceeded](02_01_OOM-example-schema3.png)


What would happen if this GC overhead limit would not exist? Note that the _java.lang.OutOfMemoryError: GC overhead limit exceeded_ error is only thrown when 2% of the memory is freed after several [GC cycles](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations). This means that the small amount of heap the GC is able to clean will likely be quickly filled again, forcing the GC to restart the cleaning process again. This forms a vicious cycle where the CPU is 100% busy with GC and no actual work can be done. End users of the application face extreme slowdowns – operations which normally complete in milliseconds take minutes to finish.

注意, _java.lang.OutOfMemoryError: GC overhead limit exceeded_ 错误只在连续多次 [GC](http://blog.csdn.net/renfufei/article/details/54885190) 都只回收了不到2%的极端情况下才会抛出。假如不抛出 `GC overhead limit` 错误会发生什么情况呢? 那就是GC清理的这么点内存很快会再次填满, 迫使GC再次执行. 这样就形成恶性循环, CPU使用率一直是100%, 而GC却没有任何成果. 系统用户就会看到系统卡死 - 以前只需要几毫秒的操作, 现在需要好几分钟才能完成。


So the “_java.lang.OutOfMemoryError: GC overhead limit exceeded_” message is a pretty nice example of a [fail fast](http://en.wikipedia.org/wiki/Fail-fast) principle in action.

这也是一个很好的 [快速失败原则](http://en.wikipedia.org/wiki/Fail-fast) 的案例。


## Give me an example

## 示例


In the following example we create a “_GC overhead limit exceeded_” error by initializing a Map and adding key-value pairs into the map in an unterminated loop:

以下代码在无限循环中往 Map 里添加数据。 这会导致 “_GC overhead limit exceeded_” 错误:


```
package com.cncounter.rtime;
import java.util.Map;
import java.util.Random;
public class TestWrapper {
    public static void main(String args[]) throws Exception {
        Map map = System.getProperties();
        Random r = new Random();
        while (true) {
            map.put(r.nextInt(), "value");
        }
    }
}
```

配置JVM参数: `-Xmx12m`。执行时产生的错误信息如下所示:

```
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.util.Hashtable.addEntry(Hashtable.java:435)
	at java.util.Hashtable.put(Hashtable.java:476)
	at com.cncounter.rtime.TestWrapper.main(TestWrapper.java:11)
```


As you might guess this cannot end well. And, indeed, when we launch the above program with:


你碰到的错误信息不一定就是这个。确实, 我们执行的JVM参数为:


```
java -Xmx12m -XX:+UseParallelGC TestWrapper
```

we soon face the _java.lang.OutOfMemoryError: GC overhead limit exceeded_ message. But the above example is tricky. When launched with different Java heap size or a different [GC algorithm](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations), my Mac OS X 10.9.2 with Hotspot 1.7.0_45 will choose to die differently. For example, when I run the program with smaller Java heap size like this:

很快就看到了 _java.lang.OutOfMemoryError: GC overhead limit exceeded_ 错误提示消息。但实际上这个示例是有些坑的. 因为配置不同的堆内存大小, 选用不同的[GC算法](http://blog.csdn.net/renfufei/article/details/54885190), 产生的错误信息也不相同。例如,当Java堆内存设置为10M时:


```
java -Xmx10m -XX:+UseParallelGC TestWrapper
```

DEBUG模式下错误信息如下所示:

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Hashtable.rehash(Hashtable.java:401)
	at java.util.Hashtable.addEntry(Hashtable.java:425)
	at java.util.Hashtable.put(Hashtable.java:476)
	at com.cncounter.rtime.TestWrapper.main(TestWrapper.java:11)
```

读者应该试着修改参数, 执行看看具体。错误提示以及堆栈信息可能不太一样。


the application will die with a more common _java.lang.OutOfMemoryError: Java heap space_ message that is thrown on Map resize. And when I run it with other [garbage collection algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) besides [ParallelGC](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations/parallel-gc), such as [-XX:+UseConcMarkSweepGC](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations/concurrent-mark-and-sweep) or [-XX:+UseG1GC](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations/g1), the error is caught by the default exception handler and is without stacktrace as the heap is exhausted to the extent where the [stacktrace cannot even be filled](https://plumbr.eu/blog/how-not-to-create-a-permgen-leak) on Exception creation.

这里在 Map 进行 `rehash` 时抛出了 _java.lang.OutOfMemoryError: Java heap space_ 错误消息. 如果使用其他 [垃圾收集算法](http://blog.csdn.net/renfufei/article/details/54885190), 比如 [-XX:+UseConcMarkSweepGC](http://blog.csdn.net/renfufei/article/details/54885190#t6), 或者 [-XX:+UseG1GC](http://blog.csdn.net/renfufei/article/details/54885190#t9), 错误将被默认的 exception handler 所捕获, 但是没有 stacktrace 信息, 因为在创建 Exception 时 [没办法填充stacktrace信息](https://plumbr.eu/blog/how-not-to-create-a-permgen-leak)。


例如配置:

```
-Xmx12m -XX:+UseG1GC
```

在Win7x64, Java8环境运行, 产生的错误信息为:

```
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "main"
```

建议读者修改内存配置, 以及垃圾收集器进行测试。



These variations are truly good examples that demonstrate that in resource-constrained situations you cannot predict the way your application is going to die so do not base your expectations on a specific sequence of actions to be completed.

这些真实的案例表明, 在资源受限的情况下, 无法准确预测程序会死于哪种具体的原因。所以在这类错误面前, 不能绑死某种特定的错误处理顺序。


######################
######################
######################

## What is the solution?

## 解决方案


As a tongue-in-cheek solution, if you just wished to get rid of the “_java.lang.OutOfMemoryError: GC overhead limit exceeded_” message, adding the following to your startup scripts would achieve just that:

有一种应付了事的解决方案, 就是不想抛出 “_java.lang.OutOfMemoryError: GC overhead limit exceeded_” 错误信息, 则添加下面启动参数:

```
// 不推荐
-XX:-UseGCOverheadLimit
```



I would **strongly suggest NOT to use this option** though – instead of fixing the problem you just postpone the inevitable: the application running out of memory and needing to be fixed. Specifying this option will just mask the original _java.lang.OutOfMemoryError: GC overhead limit exceeded_ error with a more familiar message _java.lang.OutOfMemoryError: Java heap space_.

我们强烈建议不要指定该选项: 因为这不能真正地解决问题，只能推迟一点 `out of memory` 错误发生的时间，到最后还得进行其他处理。指定这个选项, 会将原来的 _java.lang.OutOfMemoryError: GC overhead limit exceeded_ 错误掩盖，变成更常见的 _java.lang.OutOfMemoryError: Java heap space_ 错误消息。


On a more serious note – sometimes the GC overhead limit error is triggered because the amount of heap you have allocated to your JVM is just not enough to accommodate the needs of your applications running on that JVM. In that case, you should just allocate more heap – see at the end of this chapter for how to achieve that. 

需要注意: 有时候触发 GC overhead limit 错误的原因, 是因为分配给JVM的堆内存不足。这种情况下只需要增加堆内存大小即可。


In many cases however, providing more Java heap space will not solve the problem. For example, if your application contains a memory leak, adding more heap will just postpone the _java.lang.OutOfMemoryError: Java heap space_ error. Additionally, increasing the amount of Java heap space also tends to increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice/tuning-for-throughput) affecting your application’s [throughput or latency](https://plumbr.eu/handbook/gc-tuning/throughput-vs-latency-vs-capacity).

在大多数情况下, 增加堆内存并不能解决问题。例如程序中存在内存泄漏, 增加堆内存只能推迟产生 _java.lang.OutOfMemoryError: Java heap space_ 错误的时间。

当然, 增大堆内存, 还有可能会增加 [GC pauses](http://blog.csdn.net/renfufei/article/details/55102729#t6) 的时间, 从而影响程序的 [吞吐量或延迟](http://blog.csdn.net/renfufei/article/details/55102729#t7)。


If you wish to solve the underlying problem with the Java heap space instead of masking the symptoms, you need to figure out which part of your code is responsible for allocating the most memory. In other words, you need to answer these questions:

如果想从根本上解决问题, 则需要排查内存分配相关的代码. 简单来说, 需要回答以下问题:


1.  Which objects occupy large portions of heap

1. 哪类对象占用了最多内存？



2.  where these objects are being allocated in source code

2. 这些对象是在哪部分代码中分配的。



At this point, make sure to clear a couple of days in your calendar (or – see an automated way below the bullet list). Here is a rough process outline that will help you answer the above questions:

要搞清这一点, 可能需要好几天时间。下面是大致的流程:


*   Get clearance for acquiring a heap dump from your JVM-to-troubleshoot. “Dumps” are basically snapshots of heap contents that you can analyze, and contain everything that the application kept in memory at the time of the dump. Including passwords, credit card numbers etc.

* 获得在生产服务器上执行堆转储(heap dump)的权限。“转储”(Dump)是堆内存的快照, 可用于后续的内存分析. 这些快照中可能含有机密信息, 例如密码、信用卡账号等, 所以有时候, 由于企业的安全限制, 要获得生产环境的堆转储并不容易。


*   Instruct your JVM to dump the contents of its heap memory into a file. Be prepared to get a few dumps, as when taken at a wrong time, heap dumps contain a significant amount of  noise and can be practically useless. On the other hand, every heap dump “freezes” the JVM entirely, so don’t take too many of them or your end users start swearing.

* 在适当的时间执行堆转储。一般来说,内存分析需要比对多个堆转储文件, 假如获取的时机不对, 那就可能是一个“废”的快照. 另外, 每执行一次堆转储, 就会对JVM进行一次“冻结”, 所以生产环境中,不能执行太多的Dump操作,否则系统缓慢或者卡死,你的麻烦就大了。


*   Find a machine that can load the dump. When your JVM-to-troubleshoot uses for example 8GB of heap, you need a machine with more than 8GB to be able to analyze heap contents. Fire up dump analysis software (we recommend [Eclipse MAT](http://www.eclipse.org/mat/), but there are also equally good alternatives available).

* 用另一台机器来加载Dump文件。如果出问题的JVM内存是8GB, 那么分析 Heap Dump 的机器内存一般需要大于 8GB.  然后打开转储分析软件(我们推荐[Eclipse MAT](http://www.eclipse.org/mat/) , 当然你也可以使用其他工具)。


*   Detect the paths to GC roots of the biggest consumers of heap. We have covered this activity in a separate post [here](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste). Don’t worry, it will feel cumbersome at first, but you’ll get better after spending a few days digging.

* 检测快照中占用内存最大的 GC roots。详情请参考: [Solving OutOfMemoryError (part 6) – Dump is not a waste](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste)。 这对新手来说可能有点困难, 但这也会加深你对堆内存结构以及 navigation 机制的理解。


*   Next, you need to figure out where in your source code the potentially hazardous large amount of objects is being allocated. If you have good knowledge of your application’s source code you’ll hopefully be able to do this in a couple searches. When you have less luck, you will need some energy drinks to assist.

* 接下来, 找出可能会分配大量对象的代码. 如果对整个系统非常熟悉, 可能很快就能定位问题。运气不好的话，就只有加班加点来进行排查了。

Alternatively, we suggest [Plumbr, the only Java monitoring solution with automatic root cause detection](http://plumbr.eu). Among other performance problems it catches all _java.lang.OutOfMemoryError_s and automatically hands you the information about the most memory-hungry data structres. It takes care of gathering the necessary data behind the scenes – this includes the relevant data about heap usage (only the object layout graph, no actual data), and also some data that you can’t even find in a heap dump. It also does the necessary data processing for you – on the fly, as soon as the JVM encounters an _java.lang.OutOfMemoryError_. Here is an example _java.lang.OutOfMemoryError_ incident alert from Plumbr:

打个广告, 我们推荐 [Plumbr, the only Java monitoring solution with automatic root cause detection](http://plumbr.eu)。 Plumbr 能捕获所有的  _java.lang.OutOfMemoryError_ , 并找出其他的性能问题, 例如最消耗内存的数据结构等等。

Plumbr 在后台负责收集数据 —— 包括堆内存使用情况(只统计对象分布图, 不涉及实际数据),以及在堆转储中不容易发现的各种问题。 如果发生 _java.lang.OutOfMemoryError_ , 还能在不停机的情况下, 做必要的数据处理. 下面是Plumbr 对一个 _java.lang.OutOfMemoryError_ 的提醒:


![Plumbr OutOfMemoryError incident alert](01_02_outofmemoryerror-analyzed.png)


Without any additional tooling or analysis you can see:

强大吧, 不需要其他工具和分析, 就能直接看到:


*   Which objects are consuming the most memory (271 _com.example.map.impl.PartitionContainer_ instances consume 173MB out of 248MB total heap)

* 哪类对象占用了最多的内存(此处是 271 个 _com.example.map.impl.PartitionContainer_ 实例, 消耗了 173MB 内存, 而堆内存只有 248MB)


*   Where these objects were allocated (most of them allocated in the _MetricManagerImpl_ class, line 304)

* 这些对象在何处创建(大部分是在 _MetricManagerImpl_ 类中,第304行处)


*   What is currently referencing these objects (the full reference chain up to GC root)

* 当前是谁在引用这些对象(从 GC root 开始的完整引用链)


Equipped with this information you can zoom in to the underlying root cause and make sure the data structures are trimmed down to the levels where they would fit nicely into your memory pools.

得知这些信息, 就可以定位到问题的根源, 例如是当地精简数据结构/模型, 只占用必要的内存即可。


However, when your conclusion from memory analysis or from reading the Plumbr report are that memory use is legal and there is nothing to change in the source code, you need to allow your JVM more Java heap space to run properly. In this case, alter your JVM launch configuration and add (or increase the value if present) just one parameter in your startup scripts:

当然, 根据内存分析的结果, 以及Plumbr生成的报告, 如果发现对象占用的内存很合理, 也不需要修改源代码的话, 那就增大堆内存吧。在这种情况下,修改JVM启动参数, (按比例)增加下面的值:

```
java -Xmx1024m com.yourcompany.YourClass`
```



In the above example the Java process is given 1GB of heap. Modify the value as best fits to your JVM. However, if the result is that your JVM still dies with OutOfMemoryError, you might still not be able to avoid the manual or Plumbr-assisted analysis described above.

这里配置了最大堆内存为 `1GB`。请根据实际情况修改这个值. 如果 JVM 还是会抛出 OutOfMemoryError, 那么你可能还需要查询手册, 或者借助工具再次进行分析和诊断。


原文链接: <https://plumbr.eu/outofmemoryerror/gc-overhead-limit-exceeded>

