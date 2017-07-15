# Java heap space - java.lang.OutOfMemoryError:

## 堆内存引起的OutOfMemoryError

Java applications are only allowed to use a limited amount of memory. This limit is specified during application startup. To make things more complex, Java memory is separated into two different regions. These regions are called Heap space and Permgen (for Permanent Generation):

Java程序的内存使用是受限制的, JVM启动时由参数决定了最大内存限制。JVM将程序内存分为两个部分: 堆内存(Heap space)和 永久代(Permanent Generation, 简称 Permgen):


![](01_01_java-heap-space.png)



The size of those regions is set during the Java Virtual Machine (JVM) launch and can be customized by specifying JVM parameters _-Xmx_ and _-XX:MaxPermSize_. If you do not explicitly set the sizes, platform-specific defaults will be used.

在 JVM 启动时, 由参数 `-Xmx` 和 `-XX:MaxPermSize` 决定这两部分的空间大小. 如果没有指定, 则会根据机器上的物理内存决定默认大小。


The _java.lang.OutOfMemoryError: Java heap space_ error will be triggered when the application **attempts to add more data into the heap space area, but there is not enough room for it**.

在新创建对象时, 如果堆内存不足, 就会引起 `java.lang.OutOfMemoryError: Java heap space` 错误。


Note that there might be plenty of physical memory available, but the _java.lang.OutOfMemoryError: Java heap space_ error is thrown whenever the JVM reaches the heap size limit.

请注意, 即便还有空闲的物理内存, 但只要JVM达到了最大堆内存限制, 就会产生 `java.lang.OutOfMemoryError: Java heap space` 错误。


## What is causing it?

## 原因分析


There most common reason for the _java.lang.OutOfMemoryError: Java heap space_ error is simple – you try to fit an XXL application into an S-sized Java heap space. That is – the application just requires more Java heap space than available to it to operate normally. Other causes for this OutOfMemoryError message are more complex and are caused by a programming error:

你会发现, 大多数时候产生 `java.lang.OutOfMemoryError: Java heap space` 的原因很简单 —— 这是要把 XXL 那么强壮的身材, 往 S 号的裤衩里塞呀!. 而 Java heap space 永远都像是那最小号的衣服。其实很容易解决对不对? 换成大号的裤衩就好了嘛。 增加堆内存空间, 程序就可以正常运行. 另外的原因可能就会比较复杂, 一般都是因为代码写的有问题:


*   **Spikes in usage/data volume**. The application was designed to handle a certain amount of users or a certain amount of data. When the number of users or the volume of data suddenly spikes and crosses that expected threshold, the operation which functioned normally before the spike ceases to operate and triggers the _java.lang.OutOfMemoryError: Java heap space_ error.

* 超乎预期的请求高峰。 应用系统的设计一般都是有 “容量” 定义的, 例如每天处理哪个量级的数据、服务多少数量的系统用户。 如果访问量突然飙升, 类似于时间坐标系中针尖形状的图谱, 超过预期的阈值, 那么在峰值来临时, 程序可能就会卡死、并触发 _java.lang.OutOfMemoryError: Java heap space_  错误。


*   **Memory leaks**. A particular type of programming error will lead your application to constantly consume more memory. Every time the leaking functionality of the application is used it leaves some objects behind into the Java heap space. Over time the leaked objects consume all of the available Java heap space and trigger the already familiar _java.lang.OutOfMemoryError: Java heap space_ error.

* **Memory leaks**. 内存泄漏也是经常出现的一种情形。程序代码中的某些错误, 导致系统消耗的内存越来越多. 每一次执行存在内存泄漏的方法/代码, 就会将更多的（垃圾）对象写入到Java堆内存中. 随着运行时间的推移, 泄漏的对象消耗了Java堆中的所有可用内存, 然后就造成了我们所熟知的 _java.lang.OutOfMemoryError: Java heap space_ 错误。


## Give me an example

## 具体示例


### Trivial example

### 一个非常简单的示例


The first example is truly simple – the following Java code tries to allocate an array of 2M integers. When you compile it and launch with 12MB of Java heap space (_java -Xmx12m OOM_), it fails with the _java.lang.OutOfMemoryError: Java heap space_ message. With 13MB Java heap space the program runs just fine.

下面是一个非常简单的示例 —— Java代码试图分配 200万个以上的整形数组. 执行时指定参数,例如 `java -Xmx12m OOM`, 指定最大内存为 12MB, 那么就会看到 `java.lang.OutOfMemoryError: Java heap space` 错误。而只要将参数修改一下, 变成 13MB, 这个错误就不会发生。

：


	class OOM {
	  static final int SIZE=2*1024*1024;
	  public static void main(String[] a) {
	    int[] i = new int[SIZE];
	   }
	} 



### Memory leak example

### 一个内存泄漏的示例


The second and a more realistic example is of a memory leak. In Java, when developers create and use new objects _e.g. new Integer(5)_, they don’t have to allocate memory themselves – this is being taken care of by the Java Virtual Machine (JVM). During the life of the application the JVM periodically checks which objects in memory are still being used and which are not. Unused objects can be discarded and the memory reclaimed and reused again. This process is called [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection). The corresponding module in JVM taking care of the collection is called the [Garbage Collector (GC)](https://plumbr.eu/handbook/garbage-collection-algorithms).

这个示例更真实一些。在Java中, 当开发人员创建一个新对象时, 例如 `Integer num = new Integer(5);` , 并不需要手工去分配内存。因为 JVM 封装并自动处理了内存分配. 在应用程序执行过程中, JVM 会在必要时检查内存中还有哪些对象仍在使用, 而那些不再使用的对象会被丢弃, 将其占用的内存进行回收和重用。这个过程称为 [垃圾收集](http://blog.csdn.net/renfufei/article/details/53432995). JVM中负责垃圾回收的模块叫做 [垃圾收集器(GC)](http://blog.csdn.net/renfufei/article/details/54407417)。


Java’s automatic memory management relies on [GC](https://plumbr.eu/java-garbage-collection-handbook) to periodically look for unused objects and remove them. Simplifying a bit we can say that a **memory leak in Java is a situation where some objects are no longer used by the application but [Garbage Collection](https://plumbr.eu/handbook/garbage-collection-in-jvm) fails to recognize it**. As a result these unused objects remain in Java heap space indefinitely. This pileup will eventually trigger the _java.lang.OutOfMemoryError: Java heap space_ error.

Java中的自动内存管理依赖于 [GC](http://blog.csdn.net/column/details/14851.html) 去定期扫描不使用的对象并将其删除. 简单来说, **Java中的内存泄漏就是这种情况, 那些应用程序不再使用的对象, 却没有被 [垃圾收集程序](http://blog.csdn.net/renfufei/article/details/54144385) 进行回收处理**. 导致那些垃圾对象还存活在堆内存中, 慢慢地堆积, 最后造成 _java.lang.OutOfMemoryError: Java heap space_ 错误。


It is fairly easy to construct a Java program that satisfies the definition of a memory leak:

很容易构建一个Java程序, 来模拟内存泄漏:


	class KeylessEntry {

	   static class Key {
	      Integer id;

	      Key(Integer id) {
		 this.id = id;
	      }

	      @Override
	      public int hashCode() {
		 return id.hashCode();
	      }
	   }

	   public static void main(String[] args) {
	      Map m = new HashMap();
	      while (true){
		 for (int i = 0; i < 10000; i++){
		    if (!m.containsKey(new Key(i))){
		       m.put(new Key(i), "Number:" + i);
		    }
		 }
	      }
	   }
	} 


When you execute the above code above you might expect it to run forever without any problems, assuming that the naive caching solution only expands the underlying Map to 10,000 elements, as beyond that all the keys will already be present in the HashMap. However, in reality the elements will keep being added as the Key class does not contain a proper _equals()_ implementation next to its _hashCode()_.

对于上面的代码, 你可能会觉得没什么问题, 这只会缓存最多 10000 个元素, 因为所有的 key 只有这么多个. 但事实是, `Key` 类只重写了 `hashCode()` 方法, 没有重写 `equals()` 方法, 最终会导致一直往 HashMap 这添加更多的 Key。


As a result, over time, with the leaking code constantly used, the “cached” results end up consuming a lot of Java heap space. And when the leaked memory fills all of the available memory in the heap region and [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection) is not able to clean it, the _java.lang.OutOfMemoryError:Java heap space_ is thrown.

因此, 随着运行时间的推移, “cached” 的对象会消耗越来越多的堆内存. 当泄漏的对象填满了所有可用的内存, [垃圾收集](https://plumbr.eu/handbook/what-is-garbage-collection) 也不能清除它们, 就时候就会抛出 _java.lang.OutOfMemoryError:Java heap space_ 错误。


The solution would be easy – add the implementation for the _equals()_ method similar to the one below and you will be good to go. But before you manage to find the cause, you will definitely have lose some precious brain cells.

解决办法很简单 —— 在类 `Key` 中实现正确的 `equals()` 方法即可：

	@Override
	public boolean equals(Object o) {
	   boolean response = false;
	   if (o instanceof Key) {
	      response = (((Key)o).id).equals(this.id);
	   }
	   return response;
	}

说实话, 在找到内存泄漏的真正原因之前, 你一般会死掉很多宝贵的脑细胞。


### 一个SpringMVC中的场景


译者曾经碰到过这样一种场景, 具体是在 SpringMVC的 Interceptor 实现类中, 在 before 方法里, 将 request 对象注入到一个静态的 ThreadLocal 里,
在 post 方法中将 ThreadLocal 中的 request 再给 remove。

但在实际使用过程中, 业务代码编写人员将一个很大的对象（例如占用内存200MB左右的List）设置为 request 的 Attributes， 传递给 JSP 中。

假设这时候发生了异常, 则 post 方法不会被执行。 而 Tomcat 中的线程调度,有可能会一直调度不到刚才那个抛出了异常的线程,其中没有 remove 掉相关的ThreadLocal值。 随着运行时间的推移, 然后就造成一直在执行老年代GC, 系统直接卡死。

后续的修正为: 使用 Filter, 在 try{} finally{} 语句块中执行 ThreadLocal 的释放。 教训是： 可以使用 ThreadLocal， 但必须有受自己控制的释放措施、一般就是 try-finally 形式。



## What is the solution?

## 解决方案


In some cases, the amount of heap you have allocated to your JVM is just not enough to accommodate the needs of your applications running on that JVM. In that case, you should just allocate more heap – see at the end of this chapter for how to achieve that. 

在某些情况下,你分配给JVM堆的数量是不够容纳JVM上运行的应用程序的需求.在这种情况下,您应该分配更多堆——看到这一章结束时如何实现这一点。


In many cases however, providing more Java heap space will not solve the problem. For example, if your application contains a memory leak, adding more heap will just postpone the _java.lang.OutOfMemoryError: Java heap space_ error. Additionally, increasing the amount of Java heap space also tends to increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice/tuning-for-throughput) affecting your application’s [throughput or latency](https://plumbr.eu/handbook/gc-tuning/throughput-vs-latency-vs-capacity).

然而,在许多情况下提供更多的Java堆空间不会解决这个问题。例如,如果您的应用程序包含一个内存泄漏,添加更多的堆只会推迟_java.lang.OutOfMemoryError:Java堆space_错误。此外,增加Java堆空间的长度也增加GC暂停(https://plumbr.欧盟/手册/ gc-tuning gc-tuning-in-practice / tuning-for-throughput)影响应用程序的吞吐量和延迟(https://plumbr.eu/handbook/gc-tuning/throughput-vs-latency-vs-capacity)。


If you wish to solve the underlying problem with the Java heap space instead of masking the symptoms, you need to figure out which part of your code is responsible for allocating the most memory. In other words, you need to answer these questions:

如果你想解决这个根本问题与Java堆空间而不是掩盖症状,你需要找出哪些部分的代码负责分配内存.换句话说,你需要回答这些问题:


1.  Which objects occupy large portions of heap

1. 哪些对象占据大部分堆吗



2.  where these objects are being allocated in source code

2. 这些对象被分配在源代码在哪里



At this point, make sure to clear a couple of days in your calendar (or – see an automated way below the bullet list). Here is a rough process outline that will help you answer the above questions:

在这一点上,一定要清楚几天在你的日历(或-见下面的一个自动化的方式子弹列表)。这是一个粗略的流程大纲将帮助你回答上面的问题:


*   Get security clearance in order to perform a heap dump from your JVM. “Dumps” are basically snapshots of heap contents that you can analyze. These snapshot can thus contain confidential information, such as passwords, credit card numbers etc, so acquiring such a dump might not even be possible for security reasons.

*获得安全间隙为了执行一个JVM堆转储。“转储”基本上是堆内容的快照,您可以分析.这些快照可以包含机密信息,如密码、信用卡号码等,所以获得这样一个转储甚至可能不可能出于安全原因。


*   Get the dump at the right moment. Be prepared to get a few dumps, as when taken at a wrong time, heap dumps contain a significant amount of  noise and can be practically useless. On the other hand, every heap dump “freezes” the JVM entirely, so don’t take too many of them or your end users start facing performance issues.

*获取转储在正确的时刻。准备几个转储,当在一个错误的时间,堆转储包含大量噪声,可以几乎毫无用处.另一方面,每个JVM堆转储“冻结”,所以不要拿太多的或最终用户开始面临的性能问题。


*   Find a machine that can load the dump. When your JVM-to-troubleshoot uses for example 8GB of heap, you need a machine with more than 8GB to be able to analyze heap contents. Fire up dump analysis software (we recommend [Eclipse MAT](http://www.eclipse.org/mat/), but there are also equally good alternatives available).

*找到一个机器,可以加载转储。当你JVM-to-troubleshoot使用例如8 gb堆,你需要一台机器有超过8 gb能够分析堆内容.打开转储分析软件(我们建议(Eclipse垫)(http://www.eclipse.org/mat/),但也有同样好的替代品可用)。


*   Detect the paths to GC roots of the biggest consumers of heap. We have covered this activity in a separate post [here](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste). It is especially tough for beginners, but the practice will make you understand the structure and navigation mechanics.

*检测路径GC根堆的最大的消费者。我们已经介绍了活动在一个单独的文章[这](https://plumbr.欧盟/博客/内存泄漏/ solving-outofmemoryerror-dump-is-not-a-waste)。这对于初学者尤其艰难,但这种做法会让你理解和导航结构力学。


*   Next, you need to figure out where in your source code the potentially hazardous large amount of objects is being allocated. If you have good knowledge of your application’s source code you’ll be able to do this in a couple searches.

*接下来,你需要找出有潜在危险的大量源代码的对象被分配.如果你有良好的知识的应用程序的源代码可以在几个搜索。


Alternatively, we suggest [Plumbr, the only Java monitoring solution with automatic root cause detection](http://plumbr.eu). Among other performance problems it catches all _java.lang.OutOfMemoryError_s and automatically hands you the information about the most memory-hungry data structres. 

另外,我们建议[Plumbr,唯一的Java监控解决方案与自动根源检测)(http://plumbr.eu)。它捕获所有_java.lang其他性能问题.OutOfMemoryError_s和自动给你最消耗内存的数据结构的信息。


Plumbr takes care of gathering the necessary data behind the scenes – this includes the relevant data about heap usage (only the object layout graph, no actual data), and also some data that you can’t even find in a heap dump. It also does the necessary data processing for you – on the fly, as soon as the JVM encounters an _java.lang.OutOfMemoryError_. Here is an example _java.lang.OutOfMemoryError_ incident alert from Plumbr:

Plumbr负责幕后收集必要的数据——这包括相关数据堆使用情况(只有对象布局图,没有实际数据),还有一些数据你甚至不能发现在一个堆转储。它还为你做必要的数据处理,在飞,一旦遇到_java.lang.OutOfMemoryError_ JVM.这里有一个例子_java.lang。从Plumbr OutOfMemoryError_事件提醒:


[![Plumbr OutOfMemoryError incident alert](https://plumbr.eu/wp-content/uploads/2015/08/outofmemoryerror-analyzed.png)](https://plumbr.eu/wp-content/uploads/2015/08/outofmemoryerror-analyzed.png)



Without any additional tooling or analysis you can see:

没有任何额外的工具或分析可以看到:


*   Which objects are consuming the most memory (271 _com.example.map.impl.PartitionContainer_ instances consume 173MB out of 248MB total heap)

*哪些对象(271 _com.example.map.impl消耗最记忆。PartitionContainer_实例使用173 mb的248 mb总堆)


*   Where these objects were allocated (most of them allocated in the _MetricManagerImpl_ class, line 304)

*这些对象被分配(其中大部分是分配在_MetricManagerImpl_类,第304行)


*   What is currently referencing these objects (the full reference chain up to GC root)

*目前引用这些对象(完整的引用链GC根)


Equipped with this information you can zoom in to the underlying root cause and make sure the data structures are trimmed down to the levels where they would fit nicely into your memory pools.

配备了这些信息,你就可以放大到底层的根源,确保数据结构修剪下来的水平,他们会很好地融入你的内存池。

However, when your conclusion from memory analysis or from reading the Plumbr report are that memory use is legal and there is nothing to change in the source code, you need to allow your JVM more Java heap space to run properly. In this case, alter your JVM launch configuration and add (or increase the value if present) the following:

然而,当你的结论从内存分析或阅读Plumbr报告内存使用是合法的,没有什么改变在源代码中,你需要让你的JVM更多的Java堆空间正常运行。在这种情况下,改变你的JVM启动配置和添加(如果存在)或增加价值如下:


	-Xmx1024m

The above configuration would give the application 1024MB of Java heap space. You can use g or G for GB, m or M for MB, k or K for KB. For example all of the following are equivalent to saying that the maximum Java heap space is 1GB:

这里配置Java堆空间为 1024MB。可以使用 g/G 表示 GB, m/M 代表 MB, k/K 表示 KB. 所以下面的这些形式都是等价的, 设置Java堆的最大空间是 1GB:


	java -Xmx1073741824 com.mycompany.MyClass
	java -Xmx1048576k com.mycompany.MyClass
	java -Xmx1024m com.mycompany.MyClass
	java -Xmx1g com.mycompany.MyClass 


原文链接: <https://plumbr.eu/outofmemoryerror/java-heap-space>


