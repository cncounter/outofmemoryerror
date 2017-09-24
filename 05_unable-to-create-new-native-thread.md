# java.lang.OutOfMemoryError: Unable to create new native thread

# OutOfMemoryError系列（5）: Unable to create new native thread

Java applications are multi-threaded by nature. What this means is that the programs written in Java can do several things (seemingly) at once. For example – even on machines with just one processor – while you drag content from one window to another, the movie played in the background does not stop just because you carry out several operations at once.

Java应用程序本质上是多线程的。用Java编写的程序可以同时执行多件事情. 就像是在播放视频的时候, 还可以将窗口中的内容拖放到另一个窗口中, 视频却不会因为这些操作而暂停, 即使物理机上只有一个CPU。

A way to think about threads is to think of them as workers to whom you can submit tasks to carry out. If you had only one worker, he or she could only carry out one task at the time. But when you have a dozen workers at your disposal they can simultaneously fulfill several of your commands.

线程(thread)可以看作是干活的工人(workers)。 只有一个工人的话, 在同一时间,只能执行一项任务. 但如果有很多工人, 就可以同时执行多项任务。

Now, as with workers in physical world, threads within the JVM need some elbow room to carry out the work they are summoned to deal with. When there are more threads than there is room in memory we have built a foundation for a problem:

和现实世界类似, JVM中的线程也需要有相应的空间来执行自己的任务. 如果线程数量太多, 就会引入新的问题:


![java-lang-outofmemoryerror-unable-to-create-new-native-thread](05_01_unable-to-create-new-native-thread.png)



The message _java.lang.OutOfMemoryError: Unable to create new native thread_ means that the **Java application has hit the limit of how many Threads it can launch.**

_java.lang.OutOfMemoryError: Unable to create new native thread_ 错误表达的意思是: **程序创建的线程已达到上限**

## What is causing it?

## 原因分析

You have a chance to face the _java.lang.OutOfMemoryError: Unable to create new native thread_ whenever the JVM asks for a new thread from the OS. Whenever the underlying OS cannot allocate a new native thread, this OutOfMemoryError will be thrown. The exact limit for native threads is very platform-dependent thus we recommend to find out those limits by running a test similar to the below [example](#example). But, in general, the situation causing _java.lang.OutOfMemoryError: Unable to create new native thread_ goes through the following phases:

只要JVM向底层操作系统请求创建新的 native thread(原生线程), 就有可能会碰到 _java.lang.OutOfMemoryError: Unable to create new native thread_ .如果底层操作系统不能创建一个新的 native thread, 就会抛出OutOfMemoryError. 原生线程的数量限制由具体环境决定, 我们建议通过一些测试用例找出这种限制, 请参考下文的示例. 但总的来说, 导致 _java.lang.OutOfMemoryError: Unable to create new native thread_ 的情景大致需要经历这些阶段:

1.  A new Java thread is requested by an application running inside the JVM

2.  JVM native code proxies the request to create a new native thread to the OS

3.  The OS tries to create a new native thread which requires memory to be allocated to the thread

4.  The OS will refuse native memory allocation either because the 32-bit Java process size has depleted its memory address space – e.g. (2-4) GB process size limit has been hit – or the virtual memory of the OS has been fully depleted

5.  The _java.lang.OutOfMemoryError: Unable to create new native thread_ error is thrown.

6.  JVM中的程序请求创建一个新的Java线程;

7.  JVM本地代码(native code)代理该请求, 创建一个操作系统级别的 native thread(原生线程);

8.  操作系统尝试创建一个新的native thread, 需要同时分配一些内存给该线程;

9.  如果操作系统的虚拟内存已耗尽, 或者是32位进程的地址空间耗尽(约2-4GB), 就会拒绝本地内存分配;

10.  抛出 _java.lang.OutOfMemoryError: Unable to create new native thread_  错误。

## Give me an example

## 示例

The following example creates and starts new threads in a loop. When running the code, operating system limits are reached fast and _java.lang.OutOfMemoryError: Unable to create new native thread_ message is displayed.

下面的示例在一个死循环中创建并启动新线程。代码运行后, 很快就会达到操作系统限制, 报出 _java.lang.OutOfMemoryError: Unable to create new native thread_ 错误。

```
 while(true){
    new Thread(new Runnable(){
        public void run() {
            try {
                Thread.sleep(10000000);
            } catch(InterruptedException e) { }        
        }    
    }).start();
} 

```



The exact native thread limit is platform-dependent, for example tests on Windows, Linux and Mac OS X reveal that:

原生线程的数量由具体环境决定, 例如在 Windows, Linux 和 Mac OS X 系统上:

*   64-bit Mac OS X 10.9, Java 1.7.0_45 – JVM dies after #2031 threads have been created

*   64-bit Mac OS X 10.9, Java 1.7.0_45 – JVM 在创建 #2031 号线程之后挂掉

*   64-bit Ubuntu Linux, Java 1.7.0_45 – JVM dies after #31893 threads have been created

*   64-bit Ubuntu Linux, Java 1.7.0_45 – JVM 在创建 #31893 号线程之后挂掉

*   64-bit Windows 7, Java 1.7.0_45 – due to a different thread model used by the OS, this error seems not to be thrown on this particular platform. On thread #250,000 the process was still alive, even though the swap file had grown to 10GB and the application was facing extreme performance issues.

*   64-bit Windows 7, Java 1.7.0_45 – 由于操作系统使用了不同的线程模型, 这个错误信息似乎不会出现. 创建 #250,000 号线程之后进程依然存在, 虚拟内存(swap file) 已达到 10GB, 此时系统极其缓慢。

So make sure you know your limits by invoking a small test and find out when the _java.lang.OutOfMemoryError: Unable to create new native thread_ will be triggered

所以要知道系统的极限, 只需要执行一个小小的测试就够了,找出 _java.lang.OutOfMemoryError: Unable to create new native thread_ 触发时创建的线程数即可。

## What is the solution?

## 解决方案

Occasionally you can bypass the _Unable to create new native thread_ issue by increasing the limits at the OS level. For example, if you have limited the number of processes that the JVM can spawn in user space you should check out and possibly increase the limit:

有些情况下, 可以通过修改系统级别的限制来避开 _Unable to create new native thread_ 问题. 如果限制了 user space 中JVM可以使用的文件数量、那么可以放开一些限制, 例如:

```
[root@dev ~]# ulimit -a
core file size          (blocks, -c) 0
...... 省略部分内容 ......
max user processes              (-u) 1800

```


More often than not, the limits on new native threads hit by the OutOfMemoryError indicate a programming error. When your application spawns thousands of threads then chances are that something has gone terribly wrong – there are not many applications out there which would benefit from such a vast amount of threads.

更多情况是, 创建 native 线程时触发OutOfMemoryError, 表明编程有BUG. 如果应用程序创建了成千上万的线程, 很可能是某些地方出了大问题 —— 没有几个程序可以 Hold 住上万个线程。

One way to solve the problem is to start taking thread dumps to understand the situation. You usually end up spending days doing this. Our suggestion is to connect [Plumbr](http://plumbr.eu) to your application to find out what is causing the problem and how to cure it in just minutes.

一种解决办法是执行线程转储(thread dump) 来分析具体情景。 通常需要花费好几个工作日。 当然, 我们推荐使用 [Plumbr](http://plumbr.eu) 来找出问题的原因, 分分钟帮你搞定。


原文链接: <https://plumbr.eu/outofmemoryerror/unable-to-create-new-native-thread>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

