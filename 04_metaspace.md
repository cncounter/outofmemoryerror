# java.lang.OutOfMemoryError: **Metaspace**

# OutOfMemoryError系列（4）: Metaspace

Java applications are allowed to use only a limited amount of memory. The exact amount of memory your particular application can use is specified during application startup. To make things more complex, Java memory is separated into different regions, as seen in the following figure:

JVM限制了Java程序的最大内存, 修改/指定启动参数可以改变这种限制。Java将堆内存划分为多个部分, 如下图所示:


![metaspace error](04_01_OOM-example-metaspace.png)



The size of all those regions, including the metaspace area, can be specified during the JVM launch. If you do not determine the sizes yourself, platform-specific defaults will be used.

【Java8及以上】这些内存池的最大值, 由 `-Xmx` 和 `-XX:MaxMetaspaceSize` 等JVM启动参数指定. 如果没有明确指定, 则根据平台类型(OS版本+JVM版本)和物理内存的大小来确定。

The _java.lang.OutOfMemoryError: Metaspace_ message indicates that the Metaspace area in memory is exhausted.

_java.lang.OutOfMemoryError: Metaspace_ 错误所表达的信息是: **元数据区(Metaspace) 已被用满**

## What is causing it?

## 原因分析

If you are not a newcomer to the Java landscape, you might be familiar with another concept in Java memory management called PermGen. Starting from Java 8, the memory model in Java was significantly changed. A new memory area called Metaspace was introduced and Permgen was removed. This change was made due to variety of reasons, including but not limited to:

如果你是Java老司机, 应该对 PermGen 比较熟悉. 但从Java 8开始,内存模型发生重大改变, 不再使用Permgen, 而是引入一个新的空间: Metaspace. 这种改变基于多方面的考虑, 部分原因列举如下:

*   The required size of permgen was hard to predict. It resulted in either under-provisioning triggering [java.lang.OutOfMemoryError: Permgen size](http://www.plumbr.eu/outofmemoryerror/permgen-space) errors or over-provisioning resulting in wasted resources.

*   Permgen空间的具体多大很难预测。指定小了会造成 [java.lang.OutOfMemoryError: Permgen size](http://blog.csdn.net/renfufei/article/details/77994177) 错误, 设置多了又造成浪费。

*   [GC performance](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice) improvements, enabling concurrent class data de-allocation without [GC pauses](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations) and specific iterators on metadata

*   为了 [GC 性能](http://blog.csdn.net/renfufei/article/details/61924893) 的提升, 使得垃圾收集过程中的并发阶段不再 [停顿](http://blog.csdn.net/renfufei/article/details/54885190), 另外对 metadata 进行特定的遍历(specific iterators)。

*   Support for further optimizations such as [G1](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations/g1) concurrent class unloading.

*   对 [G1垃圾收集器](http://blog.csdn.net/renfufei/article/details/54885190#t9) 的并发 class unloading 进行深度优化。

So if you were familiar with PermGen then all you need to know as background is that – whatever was in PermGen before Java 8 (name and fields of the class, methods of a class with the bytecode of the methods, constant pool, JIT optimizations etc) – is now located in Metaspace.

在Java8中,将之前 PermGen 中的所有内容, 都移到了 Metaspace 空间。例如: class 名称, 字段, 方法, 字节码, 常量池, JIT优化代码, 等等。

As you can see, Metaspace size requirements depend both upon the number of classes loaded as well as the size of such class declarations. So it is easy to see the **main cause for the _java.lang.OutOfMemoryError: Metaspace_ is: either too many classes or too big classes being loaded to the Metaspace.**

Metaspace 的使用量与JVM加载到内存中的 class 数量/大小有关。可以说, _java.lang.OutOfMemoryError: Metaspace_ 错误的主要原因, 是加载到内存中的 class 数量太多或者体积太大。

## Give me an example

## 示例

As we explained in the previous chapter, Metaspace usage is strongly correlated with the number of classes loaded into the JVM. The following code serves as the most straightforward example:

和 [上一章的PermGen](http://blog.csdn.net/renfufei/article/details/77994177) 类似, Metaspace 空间的使用量, 与JVM加载的 class 数量有很大关系。下面是一个简单的示例:

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

可以看到, 使用 [javassist](http://www.csg.ci.i.u-tokyo.ac.jp/~chiba/javassist/) 工具库生成 class 那是非常简单。在 for 循环中, 动态生成很多class, 最终将这些class加载到 Metaspace 中。

The code will keep generating new classes and loading their definitions to Metaspace until the space is fully utilized and the _java.lang.OutOfMemoryError: Metaspace_ is thrown. When launched with _-XX:MaxMetaspaceSize=64m_ then on Mac OS X my Java 1.8.0_05 dies at around 70,000 classes loaded.

执行这段代码, 随着生成的class越来越多, 最后将会占满 Metaspace 空间, 抛出 _java.lang.OutOfMemoryError: Metaspace_. 在Mac OS X上, Java 1.8.0_05 环境下, 如果设置了启动参数 _-XX:MaxMetaspaceSize=64m_, 大约加载 70000 个class后JVM就会挂掉。

## What is the solution?

## 解决方案

The first solution when facing the OutOfMemoryError due to Metaspace should be obvious. If the application exhausts the Metaspace area in the memory you should increase the size of Metaspace. Alter your application launch configuration and increase the following:

如果抛出与 Metaspace 有关的 OutOfMemoryError , 第一解决方案是增加 Metaspace 的大小. 使用下面这样的启动参数:

```
-XX:MaxMetaspaceSize=512m
```


The above configuration example tells the JVM that Metaspace is allowed to grow up to 512 MB before it can start complaining in the form of _OutOfMemoryError_.

这里将 Metaspace 的最大值设置为 512MB, 如果没有用完, 就不会抛出 _OutOfMemoryError_。

Another solution is even simpler at first sight. You can remove the limit on Metaspace size altogether by deleting this parameter. But pay attention to the fact that by doing so you can introduce heavy swapping and/or reach native allocation failures instead.

有一种看起来很简单的方案, 是直接去掉 Metaspace 的大小限制。 但需要注意, 不限制Metaspace内存的大小, 假若物理内存不足, 有可能会引起内存交换(swapping), 严重拖累系统性能。 此外,还可能造成native内存分配失败等问题。

> 在现代应用集群中,宁可让应用节点挂掉, 也不希望其响应缓慢。

Before calling it a night though, be warned – more often than not it can happen that by using the above recommended “quick fixes” you end up masking the symptoms by hiding the _java.lang.OutOfMemoryError: Metaspace_ and not tackling the underlying problem. If your application leaks memory or just loads something unreasonable into Metaspace the above solution will not actually improve anything, it will just postpone the problem.

如果不想收到报警, 可以像鸵鸟一样, 把 _java.lang.OutOfMemoryError: Metaspace_ 错误信息隐藏起来。 但这不能真正解决问题, 只会推迟问题爆发的时间。 如果确实存在内存泄露, 请参考前面的文章, 认真寻找解决方案。


原文链接: <https://plumbr.eu/outofmemoryerror/permgen-space>

翻译日期: 2017年9月19日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

