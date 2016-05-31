
# java.lang.OutOfMemoryError:
**Java heap space**

Java applications are only allowed to use a limited amount of memory. This limit is specified during application startup. To make things more complex, Java memory is separated into two different regions. These regions are called Heap space and Permgen (for Permanent Generation):



![OutOfMemoryError: Java heap space](https://plumbr.eu/wp-content/uploads/2014/04/java-lang-outofmemoryerror-java-heap-space.png)



The size of those regions is set during the Java Virtual Machine (JVM) launch and can be customized by specifying JVM parameters _-Xmx_ and _-XX:MaxPermSize_. If you do not explicitly set the sizes, platform-specific defaults will be used.

The _java.lang.OutOfMemoryError: Java heap space_ error will be triggered when the application **attempts to add more data into the heap space area, but there is not enough room for it**.

Note that there might be plenty of physical memory available, but the _java.lang.OutOfMemoryError: Java heap space_ error is thrown whenever the JVM reaches the heap size limit.

## What is causing it?

There most common reason for the _java.lang.OutOfMemoryError: Java heap space_ error is simple – you try to fit an XXL application into an S-sized Java heap space. That is – the application just requires more Java heap space than available to it to operate normally. Other causes for this OutOfMemoryError message are more complex and are caused by a programming error:

*   **Spikes in usage/data volume**. The application was designed to handle a certain amount of users or a certain amount of data. When the number of users or the volume of data suddenly spikes and crosses that expected threshold, the operation which functioned normally before the spike ceases to operate and triggers the _java.lang.OutOfMemoryError: Java heap space_ error.

*   **Memory leaks**. A particular type of programming error will lead your application to constantly consume more memory. Every time the leaking functionality of the application is used it leaves some objects behind into the Java heap space. Over time the leaked objects consume all of the available Java heap space and trigger the already familiar _java.lang.OutOfMemoryError: Java heap space_ error.



## Give me an example

### Trivial example

The first example is truly simple – the following Java code tries to allocate an array of 2M integers. When you compile it and launch with 12MB of Java heap space (_java -Xmx12m OOM_), it fails with the _java.lang.OutOfMemoryError: Java heap space_ message. With 13MB Java heap space the program runs just fine.

```
class OOM {
  static final int SIZE=2*1024*1024;
  public static void main(String[] a) {
    int[] i = new int[SIZE];
   }
} 

```

### Memory leak example

The second and a more realistic example is of a memory leak. In Java, when developers create and use new objects _e.g. new Integer(5)_, they don’t have to allocate memory themselves – this is being taken care of by the Java Virtual Machine (JVM). During the life of the application the JVM periodically checks which objects in memory are still being used and which are not. Unused objects can be discarded and the memory reclaimed and reused again. This process is called [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection). The corresponding module in JVM taking care of the collection is called the [Garbage Collector (GC)](https://plumbr.eu/handbook/garbage-collection-algorithms).

Java’s automatic memory management relies on [GC](https://plumbr.eu/java-garbage-collection-handbook) to periodically look for unused objects and remove them. Simplifying a bit we can say that a **memory leak in Java is a situation where some objects are no longer used by the application but [Garbage Collection](https://plumbr.eu/handbook/garbage-collection-in-jvm) fails to recognize it**. As a result these unused objects remain in Java heap space indefinitely. This pileup will eventually trigger the _java.lang.OutOfMemoryError: Java heap space_ error.

It is fairly easy to construct a Java program that satisfies the definition of a memory leak:

```
class KeylessEntry {

   static class Key {
      Integer id;

      Key(Integer id) {
         this.id = id;
      }

      @Override
      public int hashCode() {
         return id.hashCode();
      }
   }

   public static void main(String[] args) {
      Map m = new HashMap();
      while (true)
         for (int i = 0; i < 10000; i++)
            if (!m.containsKey(new Key(i)))
               m.put(new Key(i), "Number:" + i);
   }
} 
```

When you execute the above code above you might expect it to run forever without any problems, assuming that the naive caching solution only expands the underlying Map to 10,000 elements, as beyond that all the keys will already be present in the HashMap. However, in reality the elements will keep being added as the Key class does not contain a proper _equals()_ implementation next to its _hashCode()_.

As a result, over time, with the leaking code constantly used, the “cached” results end up consuming a lot of Java heap space. And when the leaked memory fills all of the available memory in the heap region and [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection) is not able to clean it, the _java.lang.OutOfMemoryError:Java heap space_ is thrown.

The solution would be easy – add the implementation for the _equals()_ method similar to the one below and you will be good to go. But before you manage to find the cause, you will definitely have lose some precious brain cells.

```
@Override
public boolean equals(Object o) {
   boolean response = false;
   if (o instanceof Key) {
      response = (((Key)o).id).equals(this.id);
   }
   return response;
}
```


## What is the solution?

In some cases, the amount of heap you have allocated to your JVM is just not enough to accommodate the needs of your applications running on that JVM. In that case, you should just allocate more heap – see at the end of this chapter for how to achieve that. 

In many cases however, providing more Java heap space will not solve the problem. For example, if your application contains a memory leak, adding more heap will just postpone the _java.lang.OutOfMemoryError: Java heap space_ error. Additionally, increasing the amount of Java heap space also tends to increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice/tuning-for-throughput) affecting your application’s [throughput or latency](https://plumbr.eu/handbook/gc-tuning/throughput-vs-latency-vs-capacity).

If you wish to solve the underlying problem with the Java heap space instead of masking the symptoms, you need to figure out which part of your code is responsible for allocating the most memory. In other words, you need to answer these questions:

1.  Which objects occupy large portions of heap

2.  where these objects are being allocated in source code

At this point, make sure to clear a couple of days in your calendar (or – see an automated way below the bullet list). Here is a rough process outline that will help you answer the above questions:

*   Get security clearance in order to perform a heap dump from your JVM. “Dumps” are basically snapshots of heap contents that you can analyze. These snapshot can thus contain confidential information, such as passwords, credit card numbers etc, so acquiring such a dump might not even be possible for security reasons.

*   Get the dump at the right moment. Be prepared to get a few dumps, as when taken at a wrong time, heap dumps contain a significant amount of  noise and can be practically useless. On the other hand, every heap dump “freezes” the JVM entirely, so don’t take too many of them or your end users start facing performance issues.

*   Find a machine that can load the dump. When your JVM-to-troubleshoot uses for example 8GB of heap, you need a machine with more than 8GB to be able to analyze heap contents. Fire up dump analysis software (we recommend [Eclipse MAT](http://www.eclipse.org/mat/), but there are also equally good alternatives available).

*   Detect the paths to GC roots of the biggest consumers of heap. We have covered this activity in a separate post [here](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste). It is especially tough for beginners, but the practice will make you understand the structure and navigation mechanics.

*   Next, you need to figure out where in your source code the potentially hazardous large amount of objects is being allocated. If you have good knowledge of your application’s source code you’ll be able to do this in a couple searches.

Alternatively, we suggest [Plumbr, the only Java monitoring solution with automatic root cause detection](http://plumbr.eu). Among other performance problems it catches all _java.lang.OutOfMemoryError_s and automatically hands you the information about the most memory-hungry data structres. 

Plumbr takes care of gathering the necessary data behind the scenes – this includes the relevant data about heap usage (only the object layout graph, no actual data), and also some data that you can’t even find in a heap dump. It also does the necessary data processing for you – on the fly, as soon as the JVM encounters an _java.lang.OutOfMemoryError_. Here is an example _java.lang.OutOfMemoryError_ incident alert from Plumbr:



[![Plumbr OutOfMemoryError incident alert](https://plumbr.eu/wp-content/uploads/2015/08/outofmemoryerror-analyzed.png)](https://plumbr.eu/wp-content/uploads/2015/08/outofmemoryerror-analyzed.png)



Without any additional tooling or analysis you can see:

*   Which objects are consuming the most memory (271 _com.example.map.impl.PartitionContainer_ instances consume 173MB out of 248MB total heap)

*   Where these objects were allocated (most of them allocated in the _MetricManagerImpl_ class, line 304)

*   What is currently referencing these objects (the full reference chain up to GC root)

Equipped with this information you can zoom in to the underlying root cause and make sure the data structures are trimmed down to the levels where they would fit nicely into your memory pools.


However, when your conclusion from memory analysis or from reading the Plumbr report are that memory use is legal and there is nothing to change in the source code, you need to allow your JVM more Java heap space to run properly. In this case, alter your JVM launch configuration and add (or increase the value if present) the following:

`-Xmx1024m`

The above configuration would give the application 1024MB of Java heap space. You can use g or G for GB, m or M for MB, k or K for KB. For example all of the following are equivalent to saying that the maximum Java heap space is 1GB:

```
 java -Xmx1073741824 com.mycompany.MyClass
    java -Xmx1048576k com.mycompany.MyClass
    java -Xmx1024m com.mycompany.MyClass
    java -Xmx1g com.mycompany.MyClass 
```