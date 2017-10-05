# Out of memory: **Kill process or sacrifice child**

# OutOfMemoryError系列（8）: Kill process or sacrifice child

In order to understand this error, we need to recoup the operating system basics. As you know, operating systems are built on the concept of processes. Those processes are shepherded by several kernel jobs, one of which, named “Out of memory killer” is of interest to us in this particular case.

为了理解这个错误,我们需要回顾一下操作系统基础知识。如你所知,操作系统(operating system)有一个概念叫做进程(process). 这些进程由几个系统内核任务(kernel jobs)进行守护,其中一个叫做 “Out of memory killer(内存不足杀手)”, 正对应本节所讲的 OutOfMemoryError。

This kernel job can annihilate your processes under extremely low memory conditions. When such a condition is detected, the Out of memory killer is activated and picks a process to kill. The target is picked using a set of heuristics scoring all processes and selecting the one with the worst score to kill. The `Out of memory: Kill process or sacrifice child` is thus different from other errors covered in our [OOM handbook](http://plumbr.eu/outofmemoryerror) as it is not triggered nor proxied by the JVM but is a safety net built into the operating system kernels.

这个内核工作在极低内存条件下可以消灭你的过程。当这样的条件被检测到,记忆的杀手被激活和选择过程.目标是选择使用一组启发式得分最差的所有过程和选择一个分数。的`Out of memory: Kill process or sacrifice child`因此不同于其他错误覆盖在我们的[伯父手册](http://plumbr.欧盟/ outofmemoryerror)JVM不触发代理也不但是一个安全网构建到操作系统内核。


![out of memory linux kernel](./08_01_out-of-memory-kill-process-or-sacrifice-child.png)



The `Out of memory: kill process or sacrifice child` error is generated when the available virtual memory (including swap) is consumed to the extent where the overall operating system stability is put to risk. In such case the Out of memory killer picks the rogue process and kills it.

的`Out of memory: kill process or sacrifice child`错误时生成可用虚拟内存(包括交换)消耗的程度整体操作系统稳定风险.在这种情况下,内存不足杀手挑选流氓过程并杀死它。

## What is causing it?

## 原因分析

By default, Linux kernels allow processes to request more memory than currently available in the system. This makes all the sense in the world, considering that most of the processes never actually use all of the memory they allocate. The easiest comparison to this approach would be the broadband operators. They sell all the consumers a 100Mbit download promise, far exceeding the actual bandwidth present in their network. The bet is again on the fact that the users will not simultaneously all use their allocated download limit. Thus one 10Gbit link can successfully serve way more than the 100 users our simple math would permit.

默认情况下,Linux内核允许进程请求内存比当前可用的系统.这个世界上所有的意义,考虑到大部分的过程从来没有真正使用所有的内存分配.这种方法是最简单的比较宽带运营商。他们卖100 mbit下载的所有消费者承诺,远远超过实际的带宽呈现在他们的网络.赌局的这一事实再次分配的用户将不同时使用他们的下载限制.因此一个10 gbit链接可以成功地服务于超过100个用户我们的简单数学将允许。

A side effect of such an approach is visible in case some of your programs are on the path of depleting the system’s memory. This can lead to extremely low memory conditions, where no pages can be allocated to process. You might have faced such situation, where not even a root account cannot kill the offending task. To prevent such situations, the killer activates, and identifies the rogue process to be the killed.

这种方法的一个副作用是可见的,以防你的一些项目的道路上消耗系统的内存.这可能导致极低内存条件,没有页面可以分配给进程。你可能会面临这样的情况,甚至连根帐户不能杀死冒犯任务.为了防止这种情况,凶手激活,标识流氓死亡的过程。

You can read more about fine-tuning the behaviour of “`Out of memory killer`” in [this article from RedHat documentation](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html).

你可以阅读更多关于微调的行为”`Out of memory killer`“在(本文从RedHat文档)(https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html)。

Now that we have the context, how can you know what triggered the “killer” and woke you up at 5AM? One common trigger for the activation is hidden in the operating system configuration. When you check the configuration in `/proc/sys/vm/overcommit_memory`, you have the first hint – the value specified here indicates whether all malloc() calls are allowed to succeed. Note that the path to the parameter in the proc file system varies depending on the system affected by the change.

现在我们有了上下文,你怎么能知道什么引发了“杀手”,在早晨5点叫醒你吗?一个常见的触发激活是隐藏在操作系统配置.当你检查中的配置`/proc/sys/vm/overcommit_memory`,你有第一个暗示——指定的值指示是否允许所有malloc()调用成功.注意,路径参数proc文件系统在系统变化的影响而异。

Overcommitting configuration allows to allocate more and more memory for this rogue process which can eventually trigger the “`Out of memory killer`” to do exactly what it is meant to do.

简述配置允许分配更多的内存这个流氓的过程最终可以触发”`Out of memory killer`“它到底是什么要做。

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

然后你将面临一个错误类似于系统日志(以下`/var/log/kern.log`在我们的示例中):

```
Jun  4 07:41:59 plumbr kernel: 
	[70667120.897649]
	Out of memory: Kill process 29957 (java) score 366 or sacrifice child
Jun  4 07:41:59 plumbr kernel: 
	[70667120.897701]
	Killed process 29957 (java) total-vm:2532680kB, anon-rss:1416508kB, file-rss:0kB
```



Note that you might need to tweak the swapfile and heap sizes, in our testcase we used a 2g heap specified by `-Xmx2g` and had the following swap configuration:

请注意,您可能需要调整swapfile和堆大小,在我们的testcase我们使用2 g堆规定`-Xmx2g`和交换有以下配置:

```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```



## What is the solution?

## 解决方案

There are several ways to handle such situation. The first and most straightforward way to overcome the issue is to migrate the system to an instance with more memory.

有几种方法来处理这种情况。第一个也是最直接的方法来克服这个问题是系统迁移到一个实例和更多的内存。

Other possibilities would involve [fine-tuning the OOM killer](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html), scaling the load horizontally across several small instances or reducing the memory requirements of the application.

其他可能需要微调伯父杀手(https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html),扩展负载水平跨几个小实例或减少应用程序的内存需求。

One solution which we are not keen to recommend involves increasing swap space. When you recall that Java is a garbage collected language, then this solution already seems less lucrative. Modern GC algorithms are efficient when running in physical memory, but when dealing with swapped allocations the efficiency is hammered. Swapping can increase the length of GC pauses in several orders of magnitude, so you should think twice before jumping to this solution.

一个解决方案,我们并不急于推荐包括增加交换空间。当你回想一下,Java是一种垃圾收集语言,然后这个解决方案似乎已不赚钱的.现代GC算法运行在物理内存时是有效的,但在处理交换配置效率是重创.交换可以增加GC暂停的长度在几个数量级,因此这个解决方案之前,你应该三思而行。




原文链接: <https://plumbr.eu/outofmemoryerror/unable-to-create-new-native-thread>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

