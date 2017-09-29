# java.lang.OutOfMemoryError: Out of swap space?

# OutOfMemoryError系列（6）: Out of swap space?


这是本系列的第六篇文章, 相关文章列表:

- [OutOfMemoryError系列（1）: Java heap space](http://blog.csdn.net/renfufei/article/details/76350794)
- [OutOfMemoryError系列（2）: GC overhead limit exceeded](http://blog.csdn.net/renfufei/article/details/77585294)
- [OutOfMemoryError系列（3）: Permgen space](http://blog.csdn.net/renfufei/article/details/77994177)
- [OutOfMemoryError系列（4）: Metaspace](http://blog.csdn.net/renfufei/article/details/78061354)
- [OutOfMemoryError系列（5）: Unable to create new native thread](http://blog.csdn.net/renfufei/article/details/78088553)


Java applications are given limited amount of memory during the startup. This limit is specified via the -Xmx and other similar startup parameters. In situations where the total memory requested by the JVM is larger than the available physical memory, operating system starts swapping out the content from memory to hard drive.

JVM启动参数指定了最大内存限制。如 `-Xmx` 以及相关的其他启动参数. 假若JVM使用的内存总量超过可用的物理内存, 操作系统就会用到虚拟内存。

![java.lang.outofmemoryerror swap](./06_01_outofmemoryerror-out-of-swap-space.png)



The _java.lang.OutOfMemoryError: Out of swap space?_ error indicates that the swap space is also exhausted and the new attempted allocation fails due to the lack of both physical memory and swap space.

错误信息 _java.lang.OutOfMemoryError: Out of swap space?_ 表明, 交换空间(swap space,虚拟内存) 不足,是由于物理内存和交换空间都不足所以导致内存分配失败。

## What is causing it?

## 原因分析

The _java.lang.OutOfmemoryError: Out of swap space?_ is thrown by JVM when an allocation request for bytes from the native heap fails and the native heap is close to exhaustion. The message indicates the size (in bytes) of the allocation which failed and the reason for the memory request.

如果 native heap 内存耗尽,  内存分配时, JVM 就会抛出 _java.lang.OutOfmemoryError: Out of swap space?_ 错误消息, 这个消息告诉用户, 请求分配内存的操作失败了。

The problem occurs in situations where the Java processes have started swapping, which, recalling that [Java is a garbage collected language](https://plumbr.eu/handbook/garbage-collection-in-jvm) is already not a good situation. Modern [GC algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) do a good job, but when faced with latency issues caused by swapping, the [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) tend to increase to levels not tolerable by most applications.

Java进程使用了虚拟内存才会发生这个错误。 对 [Java的垃圾收集](http://blog.csdn.net/renfufei/article/details/54144385) 来说这是很难应付的场景。即使现代的 [GC算法](http://blog.csdn.net/renfufei/article/details/54885190) 很先进, 但虚拟内存交换引发的系统延迟, 会让 [GC暂停时间](http://blog.csdn.net/renfufei/article/details/61924893) 膨胀到令人难以容忍的地步。

_java.lang.OutOfMemoryError: Out of swap space?_ is often caused by operating system level issues, such as:

通常是操作系统层面的原因导致 _java.lang.OutOfMemoryError: Out of swap space?_ 问题, 例如:

*   The operating system is configured with insufficient swap space.

*   操作系统的交换空间太小。

*   Another process on the system is consuming all memory resources.

*   机器上的某个进程耗光了所有的内存资源。

It is also possible that the application fails due to a native leak, for example, if application or library code continuously allocates memory but does not release it to the operating system.

当然也可能是应用程序的本地内存泄漏(native leak)引起的, 例如, 某个程序/库不断地申请本地内存,却不进行释放。

## What is the solution?

## 解决方案

To overcome this issue, you have several possibilities. First and often the easiest workaround is to increase swap space. The means for this are platform specific, for example in Linux you can achieve with the following example sequence of commands, which create and attach a new swapfile sized at 640MB:

这个问题有多种解决办法。

第一种, 也是最简单的方法, 增加虚拟内存(swap space) 的大小. 各操作系统的设置方法不太一样, 比如Linux,可以使用下面的命令设置:

```
swapoff -a
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```

其中创建了一个大小为 640MB 的 swapfile(交换文件) 并启用该文件。

Now, you should recall that due to garbage collection sweeping the memory content, swapping is undesirable for Java processes in general. Running [garbage collection algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) on swapped allocations can increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) by several orders of magnitude, so you should think twice before jumping to the easy solution bandwagon.

因为垃圾收集器需要清理整个内存空间, 所以虚拟内存对 Java GC 来说是难以忍受的。存在内存交换时, 执行 [垃圾收集](http://blog.csdn.net/renfufei/article/details/54885190) 的 [暂停时间](http://blog.csdn.net/renfufei/article/details/61924893) 会增加上百倍,甚至更多, 所以最好不要增加虚拟内存。

If your application is deployed next to a “noisy neighbor” with whom the JVM needs to compete for resources, you should isolate the services to separate (virtual) machines.

如果程序允许环境还受到 “坏邻居效应” 的干扰, 那么JVM还要和其他程序竞争计算资源, 提高性能的办法就是单独部署到专用的服务器/虚拟机中。

And in many cases, your only truly viable alternative is to either upgrade the machine to contain more memory or optimize the application to reduce its memory footprint. When you turn to the optimization path, a good way to start is by using memory dump analyzers to detect large allocations in memory.

大多数时候, 我们唯一能做的就是升级服务器配置, 增加物理机的内存。当然也可以进行程序优化, 降低内存空间的使用量, 通过堆转储分析器可以检测到哪些方法/代码分配了大量的内存。



原文链接: <https://plumbr.eu/outofmemoryerror/out-of-swap-space>

翻译日期: 2017年9月27日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

