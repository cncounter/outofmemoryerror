# Out of memory: **Kill process or sacrifice child**

# OutOfMemoryError系列（8）: Kill process or sacrifice child

In order to understand this error, we need to recoup the operating system basics. As you know, operating systems are built on the concept of processes. Those processes are shepherded by several kernel jobs, one of which, named “Out of memory killer” is of interest to us in this particular case.

为了理解这个错误,我们需要回顾一下操作系统基础知识。如你所知,操作系统(operating system)有一个概念叫做进程(process). 这些进程由几个系统内核任务(kernel jobs)进行守护,其中一个叫做 “Out of memory killer(内存不足杀手)”, 正对应本节所讲的 OutOfMemoryError。

This kernel job can annihilate your processes under extremely low memory conditions. When such a condition is detected, the Out of memory killer is activated and picks a process to kill. The target is picked using a set of heuristics scoring all processes and selecting the one with the worst score to kill. The `Out of memory: Kill process or sacrifice child` is thus different from other errors covered in our [OOM handbook](http://plumbr.eu/outofmemoryerror) as it is not triggered nor proxied by the JVM but is a safety net built into the operating system kernels.

这个kernel job在内存极低的情况下可以杀死你的过程。只要满足其执行条件, `Out of memory killer` 就会被激活, 选中一个进程来杀死. 其通过启发式算法,计算所有进程的得分(heuristics scoring), 得分最低的进程将被杀死。因此 `Out of memory: Kill process or sacrifice child` 和其他的 [OOM handbook](http://plumbr.eu/outofmemoryerror) 不同, 因为既不由JVM触发,也不由JVM代理, 而是内置于系统内核的一种安全措施。


![out of memory linux kernel](./08_01_out-of-memory-kill-process-or-sacrifice-child.png)



The `Out of memory: kill process or sacrifice child` error is generated when the available virtual memory (including swap) is consumed to the extent where the overall operating system stability is put to risk. In such case the Out of memory killer picks the rogue process and kills it.

如果内存(包括swap)不足, 有可能会影响系统稳定, 这时候 `Out of memory killer` 就会想办法找出流氓进程, 并杀死他, 这就会引起 `Out of memory: kill process or sacrifice child` 错误。

## What is causing it?

## 原因分析

By default, Linux kernels allow processes to request more memory than currently available in the system. This makes all the sense in the world, considering that most of the processes never actually use all of the memory they allocate. The easiest comparison to this approach would be the broadband operators. They sell all the consumers a 100Mbit download promise, far exceeding the actual bandwidth present in their network. The bet is again on the fact that the users will not simultaneously all use their allocated download limit. Thus one 10Gbit link can successfully serve way more than the 100 users our simple math would permit.

默认情况下, Linux内核允许进程请求的内存超过当前系统的可用内存. 这是因为在实际场景中, 大多数进程所使用的内存, 并没有他们所申请的那么多. 
一个简单的类比是宽带租赁。运营商卖出 100Mb 带宽给你, 这只占运营商出口总带宽的一小部分. 但事实上, 宽带用户不可能同时用到这么大的量. 因此有 10Gbit 的带宽,就可以卖给超过100个用户100Mbps 的带宽。

A side effect of such an approach is visible in case some of your programs are on the path of depleting the system’s memory. This can lead to extremely low memory conditions, where no pages can be allocated to process. You might have faced such situation, where not even a root account cannot kill the offending task. To prevent such situations, the killer activates, and identifies the rogue process to be the killed.

可以预见, 这种方式有一个副作用, 如果某些程序消耗了大量的系统内存, 就会导致可用内存量极低, 没有内存页面(pages)可以分配给进程。这时候可能会出现一种极端情况, root 用户也没办法 kill 掉这个流氓程序. 为了阻止这种情况, killer会自动激活, 找到流氓进程并将其杀死。

You can read more about fine-tuning the behaviour of “`Out of memory killer`” in [this article from RedHat documentation](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html).

关于 ”`Out of memory killer`“ 性能调优的更多细节, 请参考: [this article from RedHat documentation](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html).

Now that we have the context, how can you know what triggered the “killer” and woke you up at 5AM? One common trigger for the activation is hidden in the operating system configuration. When you check the configuration in `/proc/sys/vm/overcommit_memory`, you have the first hint – the value specified here indicates whether all malloc() calls are allowed to succeed. Note that the path to the parameter in the proc file system varies depending on the system affected by the change.

现在我们知道了 “killer” 是怎么回事, 那么你知道是什么在早上5点钟触发了“killer”吗? 一个常见的触发器隐藏在操作系统配置文件中. 打开 `/proc/sys/vm/overcommit_memory` 文件, 其中的值指定了是否允许所有malloc()调用成功. 注意, proc文件的路径各个操作系统不太一样。

Overcommitting configuration allows to allocate more and more memory for this rogue process which can eventually trigger the “`Out of memory killer`” to do exactly what it is meant to do.

超量申请(Overcommitting)配置允许流氓进程分配更多的内存, 最终可能触发”`Out of memory killer`“。

## Give me an example

## 示例

When you compile and launch the following Java code snippet on Linux (I used the latest stable Ubuntu version):

在Linux上(如最新稳定版的Ubuntu)编译并执行如下代码:

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

然后在系统日志中(如 `/var/log/kern.log` 文件)会看到一个错误, 类似下面这样:

```
Jun  4 07:41:59 plumbr kernel: 
	[70667120.897649]
	Out of memory: Kill process 29957 (java) score 366 or sacrifice child
Jun  4 07:41:59 plumbr kernel: 
	[70667120.897701]
	Killed process 29957 (java) total-vm:2532680kB, anon-rss:1416508kB, file-rss:0kB
```



Note that you might need to tweak the swapfile and heap sizes, in our testcase we used a 2g heap specified by `-Xmx2g` and had the following swap configuration:

> **提示**: 你可能需要调整 swap 的大小以及堆内存大小, 例如我们使用的堆内存配置为 `-Xmx2g`,  swap 配置如下:

```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```



## What is the solution?

## 解决方案

There are several ways to handle such situation. The first and most straightforward way to overcome the issue is to migrate the system to an instance with more memory.

处理这种情况有许多方法。第一种,也是最直接的方法，就是将系统迁移到一个内存更大的实例。

Other possibilities would involve [fine-tuning the OOM killer](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html), scaling the load horizontally across several small instances or reducing the memory requirements of the application.

另外,还可以执行 [fine-tuning the OOM killer](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html), 将应用程序做负载均衡(水平扩展,集群),或者减小应用程序的内存需求。

One solution which we are not keen to recommend involves increasing swap space. When you recall that Java is a garbage collected language, then this solution already seems less lucrative. Modern GC algorithms are efficient when running in physical memory, but when dealing with swapped allocations the efficiency is hammered. Swapping can increase the length of GC pauses in several orders of magnitude, so you should think twice before jumping to this solution.

有一种不太推荐的解决方案是增加交换空间。回想一下,Java是一种包含自动垃圾收集的语言,所以增加交换内存并不合算. 现代的GC算法在处理物理内存时是非常高效的, 但在处理交换空间时效率就是硬伤了. 交换内存可能导致GC暂停的时间增长多个数量级, 因此在考虑这个方案之前,请三思而后行。




原文链接: <https://plumbr.eu/outofmemoryerror/unable-to-create-new-native-thread>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

