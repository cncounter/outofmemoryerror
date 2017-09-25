# java.lang.OutOfMemoryError: Out of swap space?

# OutOfMemoryError系列（6）: Out of swap space?

Java applications are given limited amount of memory during the startup. This limit is specified via the -Xmx and other similar startup parameters. In situations where the total memory requested by the JVM is larger than the available physical memory, operating system starts swapping out the content from memory to hard drive.

Java程序由JVM启动参数指定了最大内存限制。包括 `-Xmx` 和其他类似的启动参数. 如果JVM请求的内存总数大于可用的物理内存, 操作系统可能会使用虚拟内存。

![java.lang.outofmemoryerror swap](./06_01_outofmemoryerror-out-of-swap-space.png)



The _java.lang.OutOfMemoryError: Out of swap space?_ error indicates that the swap space is also exhausted and the new attempted allocation fails due to the lack of both physical memory and swap space.

_java.lang.OutOfMemoryError: Out of swap space?_ 表明, 交换空间(swap space,虚拟内存) 已经耗尽,由于物理内存和交换空间都不足导致内存分配失败, 。

## What is causing it?

## 原因分析

The _java.lang.OutOfmemoryError: Out of swap space?_ is thrown by JVM when an allocation request for bytes from the native heap fails and the native heap is close to exhaustion. The message indicates the size (in bytes) of the allocation which failed and the reason for the memory request.

 _java.lang.OutOfmemoryError: Out of swap space?_ 错误抛出时, JVM请求分配本机内存失败, 而且本机heap内存接近耗尽. 消息表明请求的内存分配失败,。

The problem occurs in situations where the Java processes have started swapping, which, recalling that [Java is a garbage collected language](https://plumbr.eu/handbook/garbage-collection-in-jvm) is already not a good situation. Modern [GC algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) do a good job, but when faced with latency issues caused by swapping, the [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) tend to increase to levels not tolerable by most applications.

问题发生在Java进程已经开始使用交换内存的情况下, 这对 [Java的垃圾收集](http://blog.csdn.net/renfufei/article/details/54144385) 来说是很难应付的场景。现代的[GC算法](http://blog.csdn.net/renfufei/article/details/54885190) 虽然很先进, 但内存不足导致虚拟内存交换，影响系统的延迟, [GC暂停](http://blog.csdn.net/renfufei/article/details/61924893) 会增长到令人不能容忍的地步。

_java.lang.OutOfMemoryError: Out of swap space?_ is often caused by operating system level issues, such as:

_java.lang.OutOfMemoryError: Out of swap space?_ 通常是操作系统层面的问题,例如:

*   The operating system is configured with insufficient swap space.

*   操作系统设置的交换空间不足。

*   Another process on the system is consuming all memory resources.

*   本机的另一个进程消耗了所有的内存资源。

It is also possible that the application fails due to a native leak, for example, if application or library code continuously allocates memory but does not release it to the operating system.

当然也有可能是因为应用程序的本地内存泄漏引起的,例如, 程序或库不断地申请内存,却不释放。

## What is the solution?

## 解决方案

To overcome this issue, you have several possibilities. First and often the easiest workaround is to increase swap space. The means for this are platform specific, for example in Linux you can achieve with the following example sequence of commands, which create and attach a new swapfile sized at 640MB:

为了克服这个问题,你有几个可能性。第一,通常最简单的方法是增加交换空间.意味着这个特定于平台的,例如在Linux中可以实现下面的示例命令序列,创建和附加新的swapfile在640 mb大小的:

```
swapoff -a
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```



Now, you should recall that due to garbage collection sweeping the memory content, swapping is undesirable for Java processes in general. Running [garbage collection algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) on swapped allocations can increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) by several orders of magnitude, so you should think twice before jumping to the easy solution bandwagon.

现在,你应该记得,由于垃圾收集全面的记忆内容,交换是不受欢迎的Java进程。运行垃圾收集算法(https://plumbr.欧盟/手册/ garbage-collection-algorithms-implementations)交换分配的长度可以增加GC暂停(https://plumbr.欧盟/手册/ gc-tuning / gc-tuning-in-practice)由几个数量级,所以你应该三思而后行跟风容易的解决方案。

If your application is deployed next to a “noisy neighbor” with whom the JVM needs to compete for resources, you should isolate the services to separate (virtual) machines.

如果您的应用程序部署在一个“吵闹的邻居”与JVM需要对资源的竞争,你应该隔离服务分离(虚拟)的机器。

And in many cases, your only truly viable alternative is to either upgrade the machine to contain more memory or optimize the application to reduce its memory footprint. When you turn to the optimization path, a good way to start is by using memory dump analyzers to detect large allocations in memory.

在许多情况下,你唯一真正可行的替代方法是升级机器包含更多的内存或减少内存占用优化应用程序.当你转向优化路径,一个好的方法是通过使用内存转储分析检测大型分配在内存中。



原文链接: <https://plumbr.eu/outofmemoryerror/out-of-swap-space>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

