# java.lang.OutOfMemoryError: Requested array size exceeds VM limit

# OutOfMemoryError系列（7）: Requested array size exceeds VM limit



这是本系列的第七篇文章, 相关文章列表:

- [OutOfMemoryError系列（1）: Java heap space](http://blog.csdn.net/renfufei/article/details/76350794)
- [OutOfMemoryError系列（2）: GC overhead limit exceeded](http://blog.csdn.net/renfufei/article/details/77585294)
- [OutOfMemoryError系列（3）: Permgen space](http://blog.csdn.net/renfufei/article/details/77994177)
- [OutOfMemoryError系列（4）: Metaspace](http://blog.csdn.net/renfufei/article/details/78061354)
- [OutOfMemoryError系列（5）: Unable to create new native thread](http://blog.csdn.net/renfufei/article/details/78088553)
- [OutOfMemoryError系列（6）: Out of swap space？](http://blog.csdn.net/renfufei/article/details/78136638)



Java has got a limit on the maximum array size your program can allocate. The exact limit is platform-specific but is generally somewhere between 1 and 2.1 billion elements.

Java限制了数组的最大尺寸。各个平台的具体限制可以不同, 但范围都在 `1 ~ 21亿` 之间。


![outofmemoryerror](./07_01_array-size-exceeds-vm-limit.png)



When you face the `java.lang.OutOfMemoryError: Requested array size exceeds VM limit`, this means that the application that crashes with the error is trying to allocate an array larger than the Java Virtual Machine can support.

如果程序抛出 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误, 就说明试图分配的数组大小超过 JVM 的限制。

## What is causing it?

## 原因分析

The error is thrown by the native code within the JVM. It happens before allocating memory for an array when the JVM performs a platform-specific check: whether the allocated data structure is addressable in this platform. This error is less common than you might initially think.

该错误是由JVM中的本地代码所抛出的. 它发生在为数组真正分配内存之前, JVM会执行一项检查: 所分配的数据结构在该平台是否可寻址(addressable). 这个错误可能比你所想的还要少见。

The reason you only seldom face this error is that Java arrays are indexed by int. The maximum positive int in Java is `2^31 – 1 = 2,147,483,647`. And the platform-specific limits can be really close to this number – for example on my 64bit MB Pro on Java 1.7 I can happily initialize arrays with up to 2,147,483,645 or `Integer.MAX_VALUE-2` elements.

我们很少面对这个错误, 是因为Java使用 int 来作为数组的下标(index, 索引)。Java中int类型的最大值为 `2^31 – 1 = 2,147,483,647`。特定平台的限制一般都约等于这个数字 —— 例如 64位的 MB Pro 机器上, Java 1.7 可以很愉快地分大小为 `2,147,483,645`, 或者大小为 `Integer.MAX_VALUE-2`) 的数组。

Increasing the length of the array by one to Integer.MAX_VALUE-1 results in the familiar `OutOfMemoryError`:

将这个数字增加 1, 即 `Integer.MAX_VALUE-1`, 结果就是抛出很脸熟的 `OutOfMemoryError`:

```
`Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit`
```



But the limit might not be that high – on 32-bit Linux with OpenJDK 6, you will hit the “`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`” already when allocating an array with ~1.1 billion elements. To understand the limits of your specific environments run the small test program described in the next chapter.

但最大值限制可能还会小一些, 在32位Linux的 OpenJDK 6上面, 数组长度大约在 11亿(约`2^30`) 时,就会抛出 “`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`“ 错误。要找出具体的限制, 执行一个小小的测试用例即可,请参考下文。

## Give me an example

## 示例

When trying to recreate the `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` error, let’s look at the following code:

下面的示例用来演示 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误:

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

这个例子循环4次, 每次都尝试初始化一个 int 数组, 长度从 `Integer.MAX_VALUE-3` 开始, 到 `Integer.MAX_VALUE` 为止. 如果在 64-bit Mac OS X with Hotspot 7 平台上执行, 应该会得到类似下面这样的输出:

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

请注意, 在后面两次循环中, 在抛出 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误之前, 先抛出了我们熟悉的 `java.lang.OutOfMemoryError: Java heap space` 错误。 这是因为 `2^31-1` 个 int 型数据所占用的内存超过了JVM默认使用的8GB内存。

This example also demonstrates why the error is so rare – in order to see the VM limit on array size being hit, you need to allocate an array with the size right in between the platform limit and Integer.MAX_INT. When our example is run on 64bit Mac OS X with Hotspot 7, there are only two such array lengths: Integer.MAX_INT-1 and Integer.MAX_INT.

此示例还演示了为什么这个错误比较罕见 —— 为了看到VM对数组大小的限制, 需要分配一个差不多等于 `Integer.MAX_INT` 的数字. 我们的示例运行在64位的Mac OS X, Hotspot 7平台上, 只有两个长度会导致这个错误: `Integer.MAX_INT-1` 和 `Integer.MAX_INT`。

## What is the solution?

## 解决方案

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

