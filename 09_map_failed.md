# java.lang.OutOfMemoryError
** Map failed**


以前在写各种处理Java OOM总结的ppt的时候，列到的OOM是以下这几种：

1.GC overhead limit exceeded

2.Java Heap Space

3.Unable to create new native thread

4.PermGen Space

5.Direct buffer memory

6.request {} bytes for {}. Out of swap space?


一直自认为不会有超过这个范围的OOM类型出现，没想到最近看到了一个新的OOM的类型，而这次OOM引发了一次严重的故障，整个排查过程是内部一个同事排查的，文章也是他写的，感谢他的文章，让我也学习到了之前遗漏的一个OOM相关的知识点。

故障现象为

应用日志中发现了大量的OOM异常：

	Caused by: java.lang.OutOfMemoryError: Map failed

跟踪堆栈找到抛出异常的地方是在 FileChannle#map，这个方法是创建一个内存映射文件，应用为了降低堆内存的使用，同时提高写入的效率，将一个文件分成多段，内存映射多个MappedByteBuffer进行读写操作；

跟踪fileChannle.map的方法发现最终调用的是FileChannelImpl.c里的方法；

继续跟踪这段代码，发现里面调用的mmap64这个系统函数，当mmap64返回的错误码是ENOMEM时，会向上抛出OOME，进一步查阅了GNU的手册，可以发现抛出ENOMEM错误码的解释：

1. 内存不足；

2. 地址空间不足。


而从当时的现场信息来看，这两点都不成立，当时没有新的思路，于是就先按照FileChannleImpl.unmap方法中的主动释放占用内存的方法改了下代码，改了后应用就一切正常了。

当天这个机器的JVM还crash了一次，crash日志中heap占用和物理内存都是非常正常，但日志中有个现象比较诡异： Dynamic libraries:这部分信息非常多，统计以后发现有65532条。

翻阅资料，发现这个数据来自 /proc/{pid}/maps, 这个文件展示了进程的虚拟地址空间的使用情况，这时突然想到ENOMEM中有说到进程的地址空间不足导致的，但是最后的7fff005aa000还远不到上限，而且计算虚拟内存占用也就几个G的空间。

这时想到前面提到65532这个数据，联想到了file-max，但是一查看是4889494,顺势猜想虚拟内存映射是不是也有打开上限？ 不出所料果然是有限制的。

max_map_count这个参数就是允许一个进程在VMAs(虚拟内存区域)拥有最大数量，VMA是一个连续的虚拟地址空间，当进程创建一个内存映像文件时VMA的地址空间就会增加，当达到max_map_count了就是返回out of memory errors。

这个数据通过下面的命令可以查看：

	cat /proc/sys/vm/max_map_count 

发现应用所在的机器这个数值果然是65536，而且测试修改max_map_count后filechannel#map的个数的上限也随之变化。 所以可以确定程序OOM是由于达到了这个系统的上限，也就是ENOMEM错误码中所指的out of process address。

确定了异常的触发原因，再排查引发的原因就比较容易了，再看下FileChannleImp#map的代码，发现在map第一次出现OOM时，会显式的调用System.gc去回收，但不幸的是应用启动参数上是有-XX:+DisableExplicitGC的，所以就导致了map失败，但如果在代码里主动clean是ok的现象。

总结来说，这个异常出现的原因是：

数据量增长，导致map file个数增长，应用启动参数上有-XX:+DisableExplicitGC，导致了在map file个数到达了max_map_count后直接OOM了（这也是因为heap比较大，所以full gc触发的频率低，这个问题就特别容易暴露）。

从这个问题来看，启动参数上加-XX:+DisableExplicitGC确实还是要小心，不仅map file这里是在OOM后靠显式的去执行System.gc来回收，Direct ByteBuffer其实也是这样，而这两个场景都有可能因为java本身full gc执行不频繁，导致达到了限制条件（例如map file个数达到max_map_count，而Direct ByteBuffer达到MaxDirectMemorySize），所以在CMS GC的场景下，看来还是去掉这个参数，改为加上-XX:+ExplicitGCInvokesConcurrent这个参数更稳妥一点。



题图来源于：http://goo.gl/Xv5j3t

微信公众号：hellojavacases

原文链接: http://mp.weixin.qq.com/s?__biz=MjM5MzYzMzkyMQ==&mid=200069481&idx=1&sn=52e17d327c7ea5033665459f194c7b1b&scene=4

