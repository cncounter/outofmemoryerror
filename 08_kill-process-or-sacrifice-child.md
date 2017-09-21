# Out of memory: **Kill process or sacrifice child**

In order to understand this error, we need to recoup the operating system basics. As you know, operating systems are built on the concept of processes. Those processes are shepherded by several kernel jobs, one of which, named “Out of memory killer” is of interest to us in this particular case.

This kernel job can annihilate your processes under extremely low memory conditions. When such a condition is detected, the Out of memory killer is activated and picks a process to kill. The target is picked using a set of heuristics scoring all processes and selecting the one with the worst score to kill. The _Out of memory: Kill process or sacrifice child_ is thus different from other errors covered in our [OOM handbook](http://plumbr.eu/outofmemoryerror) as it is not triggered nor proxied by the JVM but is a safety net built into the operating system kernels.



![out of memory linux kernel](./08_01_out-of-memory-kill-process-or-sacrifice-child.png)



The _Out of memory: kill process or sacrifice child_ error is generated when the available virtual memory (including swap) is consumed to the extent where the overall operating system stability is put to risk. In such case the Out of memory killer picks the rogue process and kills it.

## What is causing it?

By default, Linux kernels allow processes to request more memory than currently available in the system. This makes all the sense in the world, considering that most of the processes never actually use all of the memory they allocate. The easiest comparison to this approach would be the broadband operators. They sell all the consumers a 100Mbit download promise, far exceeding the actual bandwidth present in their network. The bet is again on the fact that the users will not simultaneously all use their allocated download limit. Thus one 10Gbit link can successfully serve way more than the 100 users our simple math would permit.

A side effect of such an approach is visible in case some of your programs are on the path of depleting the system’s memory. This can lead to extremely low memory conditions, where no pages can be allocated to process. You might have faced such situation, where not even a root account cannot kill the offending task. To prevent such situations, the killer activates, and identifies the rogue process to be the killed.

You can read more about fine-tuning the behaviour of “_Out of memory killer_” in [this article from RedHat documentation](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html).

Now that we have the context, how can you know what triggered the “killer” and woke you up at 5AM? One common trigger for the activation is hidden in the operating system configuration. When you check the configuration in _/proc/sys/vm/overcommit_memory_, you have the first hint – the value specified here indicates whether all malloc() calls are allowed to succeed. Note that the path to the parameter in the proc file system varies depending on the system affected by the change.

Overcommitting configuration allows to allocate more and more memory for this rogue process which can eventually trigger the “_Out of memory killer_” to do exactly what it is meant to do.

## Give me an example

When you compile and launch the following Java code snippet on Linux (I used the latest stable Ubuntu version):

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

then you will face an error similar to the following in the system logs ( _/var/log/kern.log_ in our example):

```
Jun  4 07:41:59 plumbr kernel: [70667120.897649] Out of memory: Kill process 29957 (java) score 366 or sacrifice child
Jun  4 07:41:59 plumbr kernel: [70667120.897701] Killed process 29957 (java) total-vm:2532680kB, anon-rss:1416508kB, file-rss:0kB
```

Note that you might need to tweak the swapfile and heap sizes, in our testcase we used a 2g heap specified by _-Xmx2g_ and had the following swap configuration:

```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```

## What is the solution?

There are several ways to handle such situation. The first and most straightforward way to overcome the issue is to migrate the system to an instance with more memory.

Other possibilities would involve [fine-tuning the OOM killer](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html), scaling the load horizontally across several small instances or reducing the memory requirements of the application.

One solution which we are not keen to recommend involves increasing swap space. When you recall that Java is a garbage collected language, then this solution already seems less lucrative. Modern GC algorithms are efficient when running in physical memory, but when dealing with swapped allocations the efficiency is hammered. Swapping can increase the length of GC pauses in several orders of magnitude, so you should think twice before jumping to this solution.



原文链接: <https://plumbr.eu/outofmemoryerror/unable-to-create-new-native-thread>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

