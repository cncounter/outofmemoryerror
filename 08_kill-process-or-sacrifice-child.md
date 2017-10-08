# Out of memory: **Kill process or sacrifice child**

# OutOfMemoryError系列（8）: Kill process or sacrifice child

> 一言不合就杀进程。。。



这是本系列的第八篇文章, 相关文章列表:

- [OutOfMemoryError系列（1）: Java heap space](http://blog.csdn.net/renfufei/article/details/76350794)
- [OutOfMemoryError系列（2）: GC overhead limit exceeded](http://blog.csdn.net/renfufei/article/details/77585294)
- [OutOfMemoryError系列（3）: Permgen space](http://blog.csdn.net/renfufei/article/details/77994177)
- [OutOfMemoryError系列（4）: Metaspace](http://blog.csdn.net/renfufei/article/details/78061354)
- [OutOfMemoryError系列（5）: Unable to create new native thread](http://blog.csdn.net/renfufei/article/details/78088553)
- [OutOfMemoryError系列（6）: Out of swap space？](http://blog.csdn.net/renfufei/article/details/78136638)
- [OutOfMemoryError系列（7）: Requested array size exceeds VM limit](http://blog.csdn.net/renfufei/article/details/78170188)



In order to understand this error, we need to recoup the operating system basics. As you know, operating systems are built on the concept of processes. Those processes are shepherded by several kernel jobs, one of which, named “Out of memory killer” is of interest to us in this particular case.

为了理解这个错误,我们先回顾一下操作系统相关的基础知识。

我们知道, 操作系统(operating system)构建在进程(process)的基础上. 进程由内核作业(kernel jobs)进行调度和维护, 其中有一个内核作业称为 “Out of memory killer(OOM终结者)”, 与本节所讲的 OutOfMemoryError 有关。

This kernel job can annihilate your processes under extremely low memory conditions. When such a condition is detected, the Out of memory killer is activated and picks a process to kill. The target is picked using a set of heuristics scoring all processes and selecting the one with the worst score to kill. The `Out of memory: Kill process or sacrifice child` is thus different from other errors covered in our [OOM handbook](http://plumbr.eu/outofmemoryerror) as it is not triggered nor proxied by the JVM but is a safety net built into the operating system kernels.

`Out of memory killer` 在可用内存极低的情况下会杀死某些进程。只要达到触发条件就会激活, 选中某个进程并杀掉。 通常采用启发式算法, 对所有进程计算评分(heuristics scoring), 得分最低的进程将被 kill 掉。因此 `Out of memory: Kill process or sacrifice child` 和前面所讲的 [OutOfMemoryError](http://blog.csdn.net/renfufei/article/category/5884735) 都不同, 因为它既不由JVM触发,也不由JVM代理, 而是系统内核内置的一种安全保护措施。


![out of memory linux kernel](./08_01_out-of-memory-kill-process-or-sacrifice-child.png)



The `Out of memory: kill process or sacrifice child` error is generated when the available virtual memory (including swap) is consumed to the extent where the overall operating system stability is put to risk. In such case the Out of memory killer picks the rogue process and kills it.

如果可用内存(含swap)不足, 就有可能会影响系统稳定, 这时候 `Out of memory killer` 就会设法找出流氓进程并杀死他, 也就是引起 `Out of memory: kill process or sacrifice child` 错误。

## What is causing it?

## 原因分析

By default, Linux kernels allow processes to request more memory than currently available in the system. This makes all the sense in the world, considering that most of the processes never actually use all of the memory they allocate. The easiest comparison to this approach would be the broadband operators. They sell all the consumers a 100Mbit download promise, far exceeding the actual bandwidth present in their network. The bet is again on the fact that the users will not simultaneously all use their allocated download limit. Thus one 10Gbit link can successfully serve way more than the 100 users our simple math would permit.

默认情况下, Linux kernels(内核)允许进程申请的量超过系统可用内存. 这是因为,在大多数情况下, 很多进程申请了很多内存, 但实际使用的量并没有那么多. 
有个简单的类比, 宽带租赁的服务商, 可能他的总带宽只有 10Gbps, 但却卖出远远超过100份以上的 100Mbps 带宽, 原因是多数时候, 宽带用户之间是错峰的, 而且不可能每个用户都用满服务商所承诺的带宽。

A side effect of such an approach is visible in case some of your programs are on the path of depleting the system’s memory. This can lead to extremely low memory conditions, where no pages can be allocated to process. You might have faced such situation, where not even a root account cannot kill the offending task. To prevent such situations, the killer activates, and identifies the rogue process to be the killed.

这样的话,可能会有一个问题, 假若某些程序占用了大量的系统内存, 那么可用内存量就会极小, 导致没有内存页面(pages)可以分配给需要的进程。可能这时候会出现极端情况, 就是 root 用户也不能通过 kill 来杀掉流氓进程. 为了防止发生这种情况, 系统会自动激活 killer, 查找流氓进程并将其杀死。

You can read more about fine-tuning the behaviour of “`Out of memory killer`” in [this article from RedHat documentation](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html).

更多关于 ”`Out of memory killer`“ 的性能调优细节, 请参考: [RedHat 官方文档](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html).

Now that we have the context, how can you know what triggered the “killer” and woke you up at 5AM? One common trigger for the activation is hidden in the operating system configuration. When you check the configuration in `/proc/sys/vm/overcommit_memory`, you have the first hint – the value specified here indicates whether all malloc() calls are allowed to succeed. Note that the path to the parameter in the proc file system varies depending on the system affected by the change.

现在我们知道了为什么会发生这种问题, 那为什么是半夜5点钟触发 “killer” 发报警信息给你呢? 通常触发的原因在于操作系统配置. 例如, `/proc/sys/vm/overcommit_memory` 配置文件的值, 指定了是否允许所有的 `malloc()` 调用成功. 请注意, 在各操作系统中, 这个配置对应的 proc 文件路径可能不同。

Overcommitting configuration allows to allocate more and more memory for this rogue process which can eventually trigger the “`Out of memory killer`” to do exactly what it is meant to do.

过量使用(overcommitting)配置, 允许流氓进程申请越来越多的内存, 最终惹得 ”`Out of memory killer`“ 出来搞事情。

## Give me an example

## 示例

When you compile and launch the following Java code snippet on Linux (I used the latest stable Ubuntu version):

在Linux上(如最新稳定版的Ubuntu)编译并执行以下的示例代码:

```
package eu.plumbr.demo;

public class OOM {

public static void main(String[] args){
  java.util.List<int[]> l = new java.util.ArrayList();
  for (int i = 10000; i < 100000; i++) {
      try {
        l.add(new int[100_000_000]);
      } catch (Throwable t) {
        t.printStackTrace();
      }
    }
  }
}
```



then you will face an error similar to the following in the system logs ( `/var/log/kern.log` in our example):

将会在系统日志中(如 `/var/log/kern.log` 文件)看到一个错误, 类似这样:

```
Jun  4 07:41:59 plumbr kernel: 
	[70667120.897649]
	Out of memory: Kill process 29957 (java) score 366 or sacrifice child
Jun  4 07:41:59 plumbr kernel: 
	[70667120.897701]
	Killed process 29957 (java) total-vm:2532680kB, anon-rss:1416508kB, file-rss:0kB
```



Note that you might need to tweak the swapfile and heap sizes, in our testcase we used a 2g heap specified by `-Xmx2g` and had the following swap configuration:

> **提示**: 可能需要调整 swap 的大小并设置最大堆内存, 例如堆内存配置为 `-Xmx2g`,  swap 配置如下:

```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```



## What is the solution?

## 解决方案

There are several ways to handle such situation. The first and most straightforward way to overcome the issue is to migrate the system to an instance with more memory.

有多种处理办法。最简单的办法就是将系统迁移到内存更大的实例中。

Other possibilities would involve [fine-tuning the OOM killer](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html), scaling the load horizontally across several small instances or reducing the memory requirements of the application.

另外, 还可以通过 [OOM killer 调优](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html), 或者做负载均衡(水平扩展,集群), 或者降低应用对内存的需求。

One solution which we are not keen to recommend involves increasing swap space. When you recall that Java is a garbage collected language, then this solution already seems less lucrative. Modern GC algorithms are efficient when running in physical memory, but when dealing with swapped allocations the efficiency is hammered. Swapping can increase the length of GC pauses in several orders of magnitude, so you should think twice before jumping to this solution.

不太推荐的方案是加大交换空间/虚拟内存(swap space)。 试想一下, Java 包含了自动垃圾回收机制, 增加交换内存的代价会很高昂. 现代GC算法在处理物理内存时性能飞快, 但对交换内存来说,其效率就是硬伤了. 交换内存可能导致GC暂停的时间增长几个数量级, 因此在采用这个方案之前, 看看是否真的有这个必要。




原文链接: <https://plumbr.eu/outofmemoryerror/kill-process-or-sacrifice-child>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

