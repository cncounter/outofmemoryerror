# java.lang.OutOfMemoryError:

**Java heap space**

Java applications are only allowed to use a limited amount of memory. This limit is specified during application startup. To make things more complex, Java memory is separated into two different regions. These regions are called Heap space and Permgen (for Permanent Generation):

Java程序可以使用的内存是有限的。而且只能在程序启动时指定最大内存。更复杂的是, Java内存被分成两个区: 称为堆空间(Heap space)和 永久代(Permanent Generation, 简称 Permgen):


![](01_01_java-heap-space.png)



The size of those regions is set during the Java Virtual Machine (JVM) launch and can be customized by specifying JVM parameters _-Xmx_ and _-XX:MaxPermSize_. If you do not explicitly set the sizes, platform-specific defaults will be used.

这些地区的大小被设置在Java虚拟机(JVM)发射,可以定制通过指定JVM参数_-Xmx_ _-XX:MaxPermSize_.如果不显式地设置的大小,将使用特定于平台的违约。


The _java.lang.OutOfMemoryError: Java heap space_ error will be triggered when the application **attempts to add more data into the heap space area, but there is not enough room for it**.

_java.lang。OutOfMemoryError:Java堆space_错误时将触发应用程序* *试图将更多的数据添加到堆空间区域,但没有足够的房间* *。


Note that there might be plenty of physical memory available, but the _java.lang.OutOfMemoryError: Java heap space_ error is thrown whenever the JVM reaches the heap size limit.

请注意,可能会有大量可用的物理内存,但_java.lang。抛出OutOfMemoryError:Java堆space_错误当JVM堆大小限制。


## What is causing it?

## 是由什么原因导致的?


There most common reason for the _java.lang.OutOfMemoryError: Java heap space_ error is simple – you try to fit an XXL application into an S-sized Java heap space. That is – the application just requires more Java heap space than available to it to operate normally. Other causes for this OutOfMemoryError message are more complex and are caused by a programming error:

_java.lang的最常见原因。OutOfMemoryError:Java堆space_错误很简单——你想XXL)应用程序适合一个S-sized Java堆空间.这是Java堆——应用程序只需要更多的空间比可以正常运行.其他原因OutOfMemoryError消息更复杂的,是由一个编程错误:


*   **Spikes in usage/data volume**. The application was designed to handle a certain amount of users or a certain amount of data. When the number of users or the volume of data suddenly spikes and crosses that expected threshold, the operation which functioned normally before the spike ceases to operate and triggers the _java.lang.OutOfMemoryError: Java heap space_ error.

* * *的使用峰值/数据量* *。应用程序被设计用来处理一定数量的用户或一定数量的数据.当用户的数量或突然飙升和十字架,预期的数据量阈值,峰值前的操作通常运行停止操作并触发_java.lang.OutOfMemoryError:Java堆space_错误。


*   **Memory leaks**. A particular type of programming error will lead your application to constantly consume more memory. Every time the leaking functionality of the application is used it leaves some objects behind into the Java heap space. Over time the leaked objects consume all of the available Java heap space and trigger the already familiar _java.lang.OutOfMemoryError: Java heap space_ error.

* * * * *内存泄漏。一个特定类型的编程错误将导致应用程序不断消耗更多的内存.每一次泄漏的功能应用程序使用它的叶子到Java堆空间背后的一些对象.随着时间的推移,泄漏对象消耗所有可用的Java堆空间和触发_java.lang已经熟悉。OutOfMemoryError:Java堆space_错误。


## Give me an example

## 给我一个例子


### Trivial example

### 小例子


The first example is truly simple – the following Java code tries to allocate an array of 2M integers. When you compile it and launch with 12MB of Java heap space (_java -Xmx12m OOM_), it fails with the _java.lang.OutOfMemoryError: Java heap space_ message. With 13MB Java heap space the program runs just fine.

第一个例子是真正简单——下面的Java代码试图分配一个2米的整数数组.当你编译并启动12 mb的Java堆空间(_java -Xmx12m OOM_),它与_java.lang失败。OutOfMemoryError:Java堆space_消息.与13 mb的Java堆空间程序运行正常。

	class OOM {
	  static final int SIZE=2*1024*1024;
	  public static void main(String[] a) {
	    int[] i = new int[SIZE];
	   }
	} 



### Memory leak example

### 内存泄漏示例


The second and a more realistic example is of a memory leak. In Java, when developers create and use new objects _e.g. new Integer(5)_, they don’t have to allocate memory themselves – this is being taken care of by the Java Virtual Machine (JVM). During the life of the application the JVM periodically checks which objects in memory are still being used and which are not. Unused objects can be discarded and the memory reclaimed and reused again. This process is called [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection). The corresponding module in JVM taking care of the collection is called the [Garbage Collector (GC)](https://plumbr.eu/handbook/garbage-collection-algorithms).

第二和一个更实际的例子是内存泄漏的。在Java中,当开发人员创建和使用_e.g新对象.New Integer (5) _, they don 't have to the allocate more memory - this is being seems care of by the Java Virtual Machine (JVM).在应用程序的生命JVM定期检查哪些对象在内存中仍在使用,哪些不是.未使用的对象可以被丢弃,内存回收和重用。这个过程称为(垃圾收集)(https://plumbr.eu/handbook/what-is-garbage-collection).在JVM的照顾收集相应的模块称为[垃圾收集器(GC)](https://plumbr.eu/handbook/garbage-collection-algorithms)。


Java’s automatic memory management relies on [GC](https://plumbr.eu/java-garbage-collection-handbook) to periodically look for unused objects and remove them. Simplifying a bit we can say that a **memory leak in Java is a situation where some objects are no longer used by the application but [Garbage Collection](https://plumbr.eu/handbook/garbage-collection-in-jvm) fails to recognize it**. As a result these unused objects remain in Java heap space indefinitely. This pileup will eventually trigger the _java.lang.OutOfMemoryError: Java heap space_ error.

Java的自动内存管理依赖(GC)(https://plumbr.eu/java-garbage-collection-handbook)定期寻找未使用的对象和删除它们.简化一点我们可以说,* *在Java内存泄漏是一种情况,一些对象不再使用的应用程序,但垃圾收集(https://plumbr.Eu/faced/garbage collection - - - the JVM) fails to recognize it in * *. As a result these unused objects remain in Java heap space indefinitely. This pileup will eventually trigger the _java. Lang.space_ heap OutOfMemoryError错误:Java。


It is fairly easy to construct a Java program that satisfies the definition of a memory leak:

很容易构建一个Java程序,满足内存泄漏的定义:


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
	      while (true)
		 for (int i = 0; i < 10000; i++)
		    if (!m.containsKey(new Key(i)))
		       m.put(new Key(i), "Number:" + i);
	   }
	} 


When you execute the above code above you might expect it to run forever without any problems, assuming that the naive caching solution only expands the underlying Map to 10,000 elements, as beyond that all the keys will already be present in the HashMap. However, in reality the elements will keep being added as the Key class does not contain a proper _equals()_ implementation next to its _hashCode()_.

当您执行上面上面的代码,你可能期望它永远运行没有任何问题,假设只天真的缓存解决方案扩展底层映射到10000个元素,除此之外所有的钥匙已经出现在HashMap.然而,在现实的元素将被作为关键类不包含一个适当的添加_equals()_实现旁边_hashCode()_。


As a result, over time, with the leaking code constantly used, the “cached” results end up consuming a lot of Java heap space. And when the leaked memory fills all of the available memory in the heap region and [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection) is not able to clean it, the _java.lang.OutOfMemoryError:Java heap space_ is thrown.

因此,作为对time with the code,leaking不断使用,“end up consuming cached”结果为Java of lot heap空间.当泄漏内存填满所有可用的内存堆中地区和垃圾收集(https://plumbr.eu/handbook/what-is-garbage-collection)是不可以清除它,_java.lang.OutOfMemoryError:Java heap space_是必须的。


The solution would be easy – add the implementation for the _equals()_ method similar to the one below and you will be good to go. But before you manage to find the cause, you will definitely have lose some precious brain cells.

解决方案很容易——添加实现_equals()_方法类似于下面的一个,你就会好了.但在你设法找到原因之前,你一定会失去一些宝贵的脑细胞。

	@Override
	public boolean equals(Object o) {
	   boolean response = false;
	   if (o instanceof Key) {
	      response = (((Key)o).id).equals(this.id);
	   }
	   return response;
	}



## What is the solution?

## 解决方案是什么?


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

上面的配置将使应用程序1024 mb的Java堆空间。您可以使用g或g GB,m和m MB,k和k对KB.例如以下均相当于说Java堆的最大空间是1 gb:


	java -Xmx1073741824 com.mycompany.MyClass
	java -Xmx1048576k com.mycompany.MyClass
	java -Xmx1024m com.mycompany.MyClass
	java -Xmx1g com.mycompany.MyClass 

