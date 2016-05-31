# java.lang.OutOfMemoryError:
**Unable to create new native thread**

Java applications are multi-threaded by nature. What this means is that the programs written in Java can do several things (seemingly) at once. For example – even on machines with just one processor – while you drag content from one window to another, the movie played in the background does not stop just because you carry out several operations at once.

A way to think about threads is to think of them as workers to whom you can submit tasks to carry out. If you had only one worker, he or she could only carry out one task at the time. But when you have a dozen workers at your disposal they can simultaneously fulfill several of your commands.

Now, as with workers in physical world, threads within the JVM need some elbow room to carry out the work they are summoned to deal with. When there are more threads than there is room in memory we have built a foundation for a problem:



![java-lang-outofmemoryerror-unable-to-create-new-native-thread](https://plumbr.eu/wp-content/uploads/2014/04/java-lang-outofmemoryerror-unable-to-create-new-native-thread.png)



The message _java.lang.OutOfMemoryError: Unable to create new native thread_ means that the **Java application has hit the limit of how many Threads it can launch.**

## What is causing it?

You have a chance to face the _java.lang.OutOfMemoryError: Unable to create new native thread_ whenever the JVM asks for a new thread from the OS. Whenever the underlying OS cannot allocate a new native thread, this OutOfMemoryError will be thrown. The exact limit for native threads is very platform-dependent thus we recommend to find out those limits by running a test similar to the below [example](#example). But, in general, the situation causing _java.lang.OutOfMemoryError: Unable to create new native thread_ goes through the following phases:

1.  A new Java thread is requested by an application running inside the JVM

2.  JVM native code proxies the request to create a new native thread to the OS

3.  The OS tries to create a new native thread which requires memory to be allocated to the thread

4.  The OS will refuse native memory allocation either because the 32-bit Java process size has depleted its memory address space – e.g. (2-4) GB process size limit has been hit – or the virtual memory of the OS has been fully depleted

5.  The _java.lang.OutOfMemoryError: Unable to create new native thread_ error is thrown.


## Give me an example

The following example creates and starts new threads in a loop. When running the code, operating system limits are reached fast and _java.lang.OutOfMemoryError: Unable to create new native thread_ message is displayed.

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

*   64-bit Mac OS X 10.9, Java 1.7.0_45 – JVM dies after #2031 threads have been created

*   64-bit Ubuntu Linux, Java 1.7.0_45 – JVM dies after #31893 threads have been created

*   64-bit Windows 7, Java 1.7.0_45 – due to a different thread model used by the OS, this error seems not to be thrown on this particular platform. On thread #250,000 the process was still alive, even though the swap file had grown to 10GB and the application was facing extreme performance issues.

So make sure you know your limits by invoking a small test and find out when the _java.lang.OutOfMemoryError: Unable to create new native thread_ will be triggered

## What is the solution?

Occasionally you can bypass the _Unable to create new native thread_ issue by increasing the limits at the OS level. For example, if you have limited the number of processes that the JVM can spawn in user space you should check out and possibly increase the limit:

```
[root@dev ~]# ulimit -a
core file size          (blocks, -c) 0
--- cut for brevity ---
max user processes              (-u) 1800

```

More often than not, the limits on new native threads hit by the OutOfMemoryError indicate a programming error. When your application spawns thousands of threads then chances are that something has gone terribly wrong – there are not many applications out there which would benefit from such a vast amount of threads.

One way to solve the problem is to start taking thread dumps to understand the situation. You usually end up spending days doing this. Our suggestion is to connect [Plumbr](http://plumbr.eu) to your application to find out what is causing the problem and how to cure it in just minutes.