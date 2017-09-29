# java.lang.OutOfMemoryError: Requested array size exceeds VM limit

# . lang。VM OutOfMemoryError:要求数组大小超过限制

Java has got a limit on the maximum array size your program can allocate. The exact limit is platform-specific but is generally somewhere between 1 and 2.1 billion elements.

Java有一个限制最大数组大小您的程序可以分配。确切的限制是特定于平台的,但通常是介于1和21亿个元素。


![outofmemoryerror](./07_01_array-size-exceeds-vm-limit.png)



When you face the `java.lang.OutOfMemoryError: Requested array size exceeds VM limit`, this means that the application that crashes with the error is trying to allocate an array larger than the Java Virtual Machine can support.

当你面对`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`,这意味着应用程序崩溃的错误正试图分配一个数组比Java虚拟机可以支持。

## What is causing it?

## 是由什么原因导致的?

The error is thrown by the native code within the JVM. It happens before allocating memory for an array when the JVM performs a platform-specific check: whether the allocated data structure is addressable in this platform. This error is less common than you might initially think.

本机代码抛出的错误是在JVM中.它发生之前为数组分配内存时,JVM执行一个特定于平台的检查:是否分配数据结构是这个平台的可寻址.这个错误是比最初你可能会觉得不太常见。

The reason you only seldom face this error is that Java arrays are indexed by int. The maximum positive int in Java is `2^31 – 1 = 2,147,483,647`. And the platform-specific limits can be really close to this number – for example on my 64bit MB Pro on Java 1.7 I can happily initialize arrays with up to 2,147,483,645 or `Integer.MAX_VALUE-2` elements.

你只有很少面对这个错误的原因是Java int数组索引。最大的正用Java int`2^31 – 1 = 2,147,483,647`。和特定于平台的限制可以非常接近这个数字——例如在我64位MB Pro在Java 1.7我可以愉快地使用2147483645或初始化数组`Integer.MAX_VALUE-2`元素。

Increasing the length of the array by one to Integer.MAX_VALUE-1 results in the familiar `OutOfMemoryError`:

增加一个整数数组的长度。在熟悉的MAX_VALUE-1结果`OutOfMemoryError`:

```
`Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit`
```



But the limit might not be that high – on 32-bit Linux with OpenJDK 6, you will hit the “`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`” already when allocating an array with ~1.1 billion elements. To understand the limits of your specific environments run the small test program described in the next chapter.

但限制可能不会有那么的高,在32位Linux OpenJDK 6,你会遭遇“`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`“已经分配一个数组时~ 11亿元素。理解您的特定环境中运行的极限小的下一章中描述的测试程序。

## Give me an example

## 给我一个例子

When trying to recreate the `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` error, let’s look at the following code:

当试图重现`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`错误,让我们来看看下面的代码:

```
 for (int i = 3; i >= 0; i--) {
  try {
    int[] arr = new int[Integer.MAX_VALUE-i];
    System.out.format("Successfully initialized an array with %,d elements.\n", Integer.MAX_VALUE-i);
  } catch (Throwable t) {
    t.printStackTrace();
  }
} 
```



The example iterates four times and initializes an array of long primitives on each turn. The size of the array this program is trying to initialize grows by one with every iteration and finally reaches Integer.MAX_VALUE. Now, when launching the code snippet on 64-bit Mac OS X with Hotspot 7, you should get the output similar to the following:

迭代的例子并初始化一个数组的四倍长原语在每个转弯.数组的大小这个程序正在初始化一个每次迭代生长,并最终达到Integer.MAX_VALUE.现在,当启动代码片段在64位Mac OS X热点7,应该会得到类似于下面的输出:

```
java.lang.OutOfMemoryError: Java heap space
  at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
java.lang.OutOfMemoryError: Java heap space
  at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
  at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
  at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
```



Note that before facing `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` on the last two attempts, the allocations failed with a lot more familiar `java.lang.OutOfMemoryError: Java heap space` message. It happens because the 2^31-1 int primitives you are trying to make room for require 8G of memory which is less than the defaults used by the JVM.

请注意,在面临`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`在过去的两次,分配失败的更熟悉`java.lang.OutOfMemoryError: Java heap space`消息。这是因为2 ^还有int原语你们房间需要8 g的内存小于默认使用的JVM。

This example also demonstrates why the error is so rare – in order to see the VM limit on array size being hit, you need to allocate an array with the size right in between the platform limit and Integer.MAX_INT. When our example is run on 64bit Mac OS X with Hotspot 7, there are only two such array lengths: Integer.MAX_INT-1 and Integer.MAX_INT.

这个示例还演示了为什么错误是如此罕见的——为了看到VM限制数组大小被打击,你需要分配一个数组的大小在平台限制和Integer.MAX_INT之间.当我们的例子是运行在64位Mac OS X与热点7中,只有两个这样的数组长度:整数。MAX_INT-1 Integer.MAX_INT。

## What is the solution?

## 解决方案是什么?

The `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` can appear as a result of either of the following situations:

的`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`可以导致出现以下情况:

*   Your arrays grow too big and end up having a size between the platform limit and the `Integer.MAX_INT`

*你的数组增长太大,最终有一个平台限制和之间的大小`Integer.MAX_INT`

*   You deliberately try to allocate arrays larger than 2^31-1 elements to experiment with the limits.

*你故意试图分配数组大于2 ^还有元素实验的局限性。

In the first case, check your code base to see whether you really need arrays that large. Maybe you could reduce the size of the arrays and be done with it. Or divide the array into smaller bulks and load the data you need to work with in batches fitting into your platform limit.

在第一种情况下,检查您的代码库,看你是否真的需要大的数组。也许你可以减少数组的大小,就万事大吉了.或者把数组分成较小的膨胀和加载数据需要处理批量限制适合你的平台。

In the second case – remember that Java arrays are indexed by int. So you cannot go beyond 2^31-1 elements in your arrays when using the standard data structures within the platform. In fact, in this case you are already blocked by the compiler announcing “`error: integer number too large`” during compilation.

在第二种情况下,记住Java int数组索引。所以你不能超出2 ^还有元素在数组在使用标准的数据结构内的平台.事实上,在这种情况下,你已经被编译器宣布“`error: integer number too large`“在编译。

But if you really work with truly large data sets, you need to rethink your options. You can load the data you need to work with in smaller batches and still use standard Java tools, or you might go beyond the standard utilities. One way to achieve this is to look into the `sun.misc.Unsafe` class. This allows you to allocate memory directly like you would in C.

但如果你真正地处理大型数据集,你需要考虑你的选择.你可以载数据需要与你在batches and still使用《标准工具、但据You爪哇超越了《标准》(公用事业。实现这种之一纳入look to`sun.misc.Unsafe`类。这允许您直接像在C语言中分配内存。



原文链接: <https://plumbr.eu/outofmemoryerror/requested-array-size-exceeds-vm-limit>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

