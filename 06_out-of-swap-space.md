# java.lang.OutOfMemoryError: **Out of swap space?**

Java applications are given limited amount of memory during the startup. This limit is specified via the -Xmx and other similar startup parameters. In situations where the total memory requested by the JVM is larger than the available physical memory, operating system starts swapping out the content from memory to hard drive.



![java.lang.outofmemoryerror swap](./06_01_outofmemoryerror-out-of-swap-space.png)



The _java.lang.OutOfMemoryError: Out of swap space?_ error indicates that the swap space is also exhausted and the new attempted allocation fails due to the lack of both physical memory and swap space.

## What is causing it?

The _java.lang.OutOfmemoryError: Out of swap space?_ is thrown by JVM when an allocation request for bytes from the native heap fails and the native heap is close to exhaustion. The message indicates the size (in bytes) of the allocation which failed and the reason for the memory request.

The problem occurs in situations where the Java processes have started swapping, which, recalling that [Java is a garbage collected language](https://plumbr.eu/handbook/garbage-collection-in-jvm) is already not a good situation. Modern [GC algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) do a good job, but when faced with latency issues caused by swapping, the [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) tend to increase to levels not tolerable by most applications.

_java.lang.OutOfMemoryError: Out of swap space?_ is often caused by operating system level issues, such as:

*   The operating system is configured with insufficient swap space.

*   Another process on the system is consuming all memory resources.

It is also possible that the application fails due to a native leak, for example, if application or library code continuously allocates memory but does not release it to the operating system.

## What is the solution?

To overcome this issue, you have several possibilities. First and often the easiest workaround is to increase swap space. The means for this are platform specific, for example in Linux you can achieve with the following example sequence of commands, which create and attach a new swapfile sized at 640MB:

```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```

Now, you should recall that due to garbage collection sweeping the memory content, swapping is undesirable for Java processes in general. Running [garbage collection algorithms](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) on swapped allocations can increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) by several orders of magnitude, so you should think twice before jumping to the easy solution bandwagon.

If your application is deployed next to a “noisy neighbor” with whom the JVM needs to compete for resources, you should isolate the services to separate (virtual) machines.

And in many cases, your only truly viable alternative is to either upgrade the machine to contain more memory or optimize the application to reduce its memory footprint. When you turn to the optimization path, a good way to start is by using memory dump analyzers to detect large allocations in memory.


原文链接: <https://plumbr.eu/outofmemoryerror/out-of-swap-space>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

