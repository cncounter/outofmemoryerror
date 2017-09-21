# java.lang.OutOfMemoryError: Unable to create new native thread

# . lang。OutOfMemoryError:无法创建新的本地线程

Java applications are multi-threaded by nature. What this means is that the programs written in Java can do several things (seemingly) at once. For example – even on machines with just one processor – while you drag content from one window to another, the movie played in the background does not stop just because you carry out several operations at once.

Java应用程序本质上是多线程的。这意味着用Java编写的程序可以同时做几件事情(看似).例如,甚至在机器只有一个处理器,而您将内容从一个窗口拖到另一个地方,这部电影的背景不停止仅仅因为您执行一些操作。

A way to think about threads is to think of them as workers to whom you can submit tasks to carry out. If you had only one worker, he or she could only carry out one task at the time. But when you have a dozen workers at your disposal they can simultaneously fulfill several of your commands.

考虑线程是认为工人你可以提交任务来执行。如果你只有一个工人,他或她可能只执行一个任务.但是,当你在处理他们十几个工人可以同时满足几个你的命令。

Now, as with workers in physical world, threads within the JVM need some elbow room to carry out the work they are summoned to deal with. When there are more threads than there is room in memory we have built a foundation for a problem:

现在,与工人在物理世界中,JVM中的线程需要一些肘部空间进行他们召集来处理工作.当有更多的线程,在内存中有房间我们已经建立了一个基础的问题:


![java-lang-outofmemoryerror-unable-to-create-new-native-thread](05_01_unable-to-create-new-native-thread.png)



The message _java.lang.OutOfMemoryError: Unable to create new native thread_ means that the **Java application has hit the limit of how many Threads it can launch.**

_java.lang的消息。OutOfMemoryError:无法创建新的本地thread_意味着* * Java应用程序已达到的极限有多少线程启动。* *

## What is causing it?

## 是由什么原因导致的?

You have a chance to face the _java.lang.OutOfMemoryError: Unable to create new native thread_ whenever the JVM asks for a new thread from the OS. Whenever the underlying OS cannot allocate a new native thread, this OutOfMemoryError will be thrown. The exact limit for native threads is very platform-dependent thus we recommend to find out those limits by running a test similar to the below [example](#example). But, in general, the situation causing _java.lang.OutOfMemoryError: Unable to create new native thread_ goes through the following phases:

你有机会面对_java.lang。OutOfMemoryError:无法创建新的本地thread_每当JVM要求一个新线程的操作系统.当底层操作系统不能分配一个新的本地线程,这就会抛出OutOfMemoryError.原生线程非常平台相关的限制因此我们建议找出这些限制通过运行一个测试类似于下面(例子)(#).但总的来说,情况导致_java.lang。OutOfMemoryError:无法创建新的本地thread_经历以下阶段:

1.  A new Java thread is requested by an application running inside the JVM

1. 一个新的Java线程在JVM中运行一个应用程序请求的

2.  JVM native code proxies the request to create a new native thread to the OS

2. JVM本地代码代理请求创建一个新的本地线程的操作系统

3.  The OS tries to create a new native thread which requires memory to be allocated to the thread

3. 操作系统试图创建一个新的本地线程需要内存被分配给线程

4.  The OS will refuse native memory allocation either because the 32-bit Java process size has depleted its memory address space – e.g. (2-4) GB process size limit has been hit – or the virtual memory of the OS has been fully depleted

4. 操作系统会拒绝本机内存分配要么因为32位内存地址空间耗尽Java进程大小——如.(2 - 4)GB进程大小限制打击——或者操作系统的虚拟内存已经被完全耗尽

5.  The _java.lang.OutOfMemoryError: Unable to create new native thread_ error is thrown.

5. _java.lang。OutOfMemoryError:无法创建新的本地thread_抛出错误。

## Give me an example

## 给我一个例子

The following example creates and starts new threads in a loop. When running the code, operating system limits are reached fast and _java.lang.OutOfMemoryError: Unable to create new native thread_ message is displayed.

下面的示例创建并启动新线程在一个循环中。代码运行时,操作系统限制到达_java.lang快捷.OutOfMemoryError:无法创建新的本地thread_消息显示。

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

极限是平台相关的本机线程,例如测试在Windows、Linux和Mac OS X显示:

*   64-bit Mac OS X 10.9, Java 1.7.0_45 – JVM dies after #2031 threads have been created

* 64位Mac OS X 10.9中,Java JVM 1.7.0_45——# 2031线程创建之后死去

*   64-bit Ubuntu Linux, Java 1.7.0_45 – JVM dies after #31893 threads have been created

* 64位Ubuntu Linux,Java JVM 1.7.0_45——# 31893线程创建之后死去

*   64-bit Windows 7, Java 1.7.0_45 – due to a different thread model used by the OS, this error seems not to be thrown on this particular platform. On thread #250,000 the process was still alive, even though the swap file had grown to 10GB and the application was facing extreme performance issues.

* 64位Windows 7,Java 1.7.0_45——由于不同的线程模型所使用的操作系统,这个错误似乎不被扔在这个特殊的平台上.线程# 250000的进程还活着,即使交换文件已经10 gb和应用正面临极端性能问题。

So make sure you know your limits by invoking a small test and find out when the _java.lang.OutOfMemoryError: Unable to create new native thread_ will be triggered

所以确保你知道你的极限通过调用一个小测试,找出当_java.lang。OutOfMemoryError:无法创建新的本地thread_将被触发

## What is the solution?

## 解决方案是什么?

Occasionally you can bypass the _Unable to create new native thread_ issue by increasing the limits at the OS level. For example, if you have limited the number of processes that the JVM can spawn in user space you should check out and possibly increase the limit:

偶尔可以绕过_Unable创建新的本地thread_问题通过增加操作系统级别的限制.例如,如果您的流程的数量有限的JVM可以产卵在用户空间中你应该并可能增加限制:

```
[root@dev ~]# ulimit -a
core file size          (blocks, -c) 0
--- cut for brevity ---
max user processes              (-u) 1800

```



More often than not, the limits on new native threads hit by the OutOfMemoryError indicate a programming error. When your application spawns thousands of threads then chances are that something has gone terribly wrong – there are not many applications out there which would benefit from such a vast amount of threads.

往往限制新受OutOfMemoryError表明原生线程编程错误.当你的应用程序产生成千上万的线程然后很可能某方面出了大问题——没有很多应用程序将受益于这样一个巨大的数量的 线程。

One way to solve the problem is to start taking thread dumps to understand the situation. You usually end up spending days doing this. Our suggestion is to connect [Plumbr](http://plumbr.eu) to your application to find out what is causing the problem and how to cure it in just minutes.

解决这个问题的一个方法是开始服用线程转储了解情况。你通常花天这样做。我们的建议是连接[Plumbr](http://plumbr.欧盟)应用程序找出导致问题以及如何在几分钟治愈它。

