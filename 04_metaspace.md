# java.lang.OutOfMemoryError:
**Metaspace**

Java applications are allowed to use only a limited amount of memory. The exact amount of memory your particular application can use is specified during application startup. To make things more complex, Java memory is separated into different regions, as seen in the following figure:



![metaspace error](https://plumbr.eu/wp-content/uploads/2014/05/OOM-example-metaspace.png)



The size of all those regions, including the metaspace area, can be specified during the JVM launch. If you do not determine the sizes yourself, platform-specific defaults will be used.

The _java.lang.OutOfMemoryError: Metaspace_ message indicates that the Metaspace area in memory is exhausted.

## What is causing it?

If you are not a newcomer to the Java landscape, you might be familiar with another concept in Java memory management called PermGen. Starting from Java 8, the memory model in Java was significantly changed. A new memory area called Metaspace was introduced and Permgen was removed. This change was made due to variety of reasons, including but not limited to:

*   The required size of permgen was hard to predict. It resulted in either under-provisioning triggering [java.lang.OutOfMemoryError: Permgen size](http://www.plumbr.eu/outofmemoryerror/permgen-space) errors or over-provisioning resulting in wasted resources.

*   [GC performance](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) improvements, enabling concurrent class data de-allocation without [GC pauses](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) and specific iterators on metadata

*   Support for further optimizations such as [G1](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations/g1) concurrent class unloading.

So if you were familiar with PermGen then all you need to know as background is that – whatever was in PermGen before Java 8 (name and fields of the class, methods of a class with the bytecode of the methods, constant pool, JIT optimizations etc) – is now located in Metaspace.

As you can see, Metaspace size requirements depend both upon the number of classes loaded as well as the size of such class declarations. So it is easy to see the **main cause for the _java.lang.OutOfMemoryError: Metaspace_ is: either too many classes or too big classes being loaded to the Metaspace.**

## Give me an example

As we explained in the previous chapter, Metaspace usage is strongly correlated with the number of classes loaded into the JVM. The following code serves as the most straightforward example:

```
public class Metaspace {
  static javassist.ClassPool cp = javassist.ClassPool.getDefault();

  public static void main(String[] args) throws Exception{
    for (int i = 0; ; i++) { 
      Class c = cp.makeClass("eu.plumbr.demo.Generated" + i).toClass();
    }
  }
}
```

In this example the source code iterates over a loop and generates classes at the runtime. All those generated class definitions end up consuming Metaspace. Class generation complexity is taken care of by the [javassist](http://www.csg.ci.i.u-tokyo.ac.jp/~chiba/javassist/) library.

The code will keep generating new classes and loading their definitions to Metaspace until the space is fully utilized and the _java.lang.OutOfMemoryError: Metaspace_ is thrown. When launched with _-XX:MaxMetaspaceSize=64m_ then on Mac OS X my Java 1.8.0_05 dies at around 70,000 classes loaded.


## What is the solution?

The first solution when facing the OutOfMemoryError due to Metaspace should be obvious. If the application exhausts the Metaspace area in the memory you should increase the size of Metaspace. Alter your application launch configuration and increase the following:

`-XX:MaxMetaspaceSize=512m`

The above configuration example tells the JVM that Metaspace is allowed to grow up to 512 MB before it can start complaining in the form of _OutOfMemoryError_.

Another solution is even simpler at first sight. You can remove the limit on Metaspace size altogether by deleting this parameter. But pay attention to the fact that by doing so you can introduce heavy swapping and/or reach native allocation failures instead.

Before calling it a night though, be warned – more often than not it can happen that by using the above recommended “quick fixes” you end up masking the symptoms by hiding the _java.lang.OutOfMemoryError: Metaspace_ and not tackling the underlying problem. If your application leaks memory or just loads something unreasonable into Metaspace the above solution will not actually improve anything, it will just postpone the problem.
