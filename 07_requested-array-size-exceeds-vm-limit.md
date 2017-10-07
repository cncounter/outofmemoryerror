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

Java平台限制了数组的最大长度。各个版本的具体限制可能稍有不同, 但范围都在 `1 ~ 21亿` 之间。


![outofmemoryerror](./07_01_array-size-exceeds-vm-limit.png)



When you face the `java.lang.OutOfMemoryError: Requested array size exceeds VM limit`, this means that the application that crashes with the error is trying to allocate an array larger than the Java Virtual Machine can support.

如果程序抛出 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误, 就说明想要创建的数组长度超过限制。

## What is causing it?

## 原因分析

The error is thrown by the native code within the JVM. It happens before allocating memory for an array when the JVM performs a platform-specific check: whether the allocated data structure is addressable in this platform. This error is less common than you might initially think.

这个错误是由JVM中的本地代码抛出的. 在真正为数组分配内存之前, JVM会执行一项检查: 要分配的数据结构在该平台是否可以寻址(addressable). 当然, 这个错误比你所想的还要少见得多。

The reason you only seldom face this error is that Java arrays are indexed by int. The maximum positive int in Java is `2^31 – 1 = 2,147,483,647`. And the platform-specific limits can be really close to this number – for example on my 64bit MB Pro on Java 1.7 I can happily initialize arrays with up to 2,147,483,645 or `Integer.MAX_VALUE-2` elements.

一般很少看到这个错误, 因为Java使用 int 类型作为数组的下标(index, 索引)。在Java中, int类型的最大值为 `2^31 – 1 = 2,147,483,647`。大多数平台的限制都约等于这个值 —— 例如在 64位的 MB Pro 上, Java 1.7 平台可以分配长度为 `2,147,483,645`, 以及 `Integer.MAX_VALUE-2`) 的数组。

Increasing the length of the array by one to Integer.MAX_VALUE-1 results in the familiar `OutOfMemoryError`:

再增加一点点长度, 变成 `Integer.MAX_VALUE-1` 时, 就会抛出我们所熟知的 `OutOfMemoryError`:

```
`Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit`
```



But the limit might not be that high – on 32-bit Linux with OpenJDK 6, you will hit the “`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`” already when allocating an array with ~1.1 billion elements. To understand the limits of your specific environments run the small test program described in the next chapter.

在有的平台上, 这个最大限制可能还会更小一些, 例如在32位Linux, OpenJDK 6 上面, 数组长度大约在 11亿左右(约`2^30`) 就会抛出 “`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`“ 错误。要找出具体的限制值, 可以执行一个小小的测试用例, 具体示例参见下文。

## Give me an example

## 示例

When trying to recreate the `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` error, let’s look at the following code:

以下代码用来演示 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误:

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

其中,for循环迭代4次, 每次都去初始化一个 int 数组, 长度从 `Integer.MAX_VALUE-3` 开始递增, 到 `Integer.MAX_VALUE` 为止. 在 64位 Mac OS X 的 Hotspot 7 平台上, 执行这段代码会得到类似下面这样的结果:

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

请注意, 在后两次迭代抛出 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误之前, 先抛出了2次 `java.lang.OutOfMemoryError: Java heap space` 错误。 这是因为 `2^31-1` 个 int 数占用的内存超过了JVM默认的8GB堆内存。

This example also demonstrates why the error is so rare – in order to see the VM limit on array size being hit, you need to allocate an array with the size right in between the platform limit and Integer.MAX_INT. When our example is run on 64bit Mac OS X with Hotspot 7, there are only two such array lengths: Integer.MAX_INT-1 and Integer.MAX_INT.

此示例也展示了这个错误比较罕见的原因 —— 要取得JVM对数组大小的限制, 要分配长度差不多等于 `Integer.MAX_INT` 的数组. 这个示例运行在64位的Mac OS X, Hotspot 7平台时, 只有两个长度会抛出这个错误: `Integer.MAX_INT-1` 和 `Integer.MAX_INT`。

## What is the solution?

## 解决方案

The `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` can appear as a result of either of the following situations:

发生 `java.lang.OutOfMemoryError: Requested array size exceeds VM limit` 错误的原因可能是:

*   Your arrays grow too big and end up having a size between the platform limit and the `Integer.MAX_INT`

*   数组太大, 最终长度超过平台限制值, 但小于 `Integer.MAX_INT` 

*   You deliberately try to allocate arrays larger than 2^31-1 elements to experiment with the limits.

*   为了测试系统限制, 故意分配长度大于 `2^31-1` 的数组。

In the first case, check your code base to see whether you really need arrays that large. Maybe you could reduce the size of the arrays and be done with it. Or divide the array into smaller bulks and load the data you need to work with in batches fitting into your platform limit.

第一种情况, 需要检查业务代码, 确认是否真的需要那么大的数组。如果可以减小数组长度, 那就万事大吉. 如果不行，可能需要把数据拆分为多个块, 然后根据需要按批次加载。

In the second case – remember that Java arrays are indexed by int. So you cannot go beyond 2^31-1 elements in your arrays when using the standard data structures within the platform. In fact, in this case you are already blocked by the compiler announcing “`error: integer number too large`” during compilation.

如果是第二种情况, 请记住, Java 数组用 int 值作为索引。所以数组元素不能超过 ` 2^31-1 ` 个. 实际上, 代码在编译阶段就会报错,提示信息为 “`error: integer number too large`”。

But if you really work with truly large data sets, you need to rethink your options. You can load the data you need to work with in smaller batches and still use standard Java tools, or you might go beyond the standard utilities. One way to achieve this is to look into the `sun.misc.Unsafe` class. This allows you to allocate memory directly like you would in C.

如果确实需要处理超大数据集, 那就要考虑调整解决方案了. 例如拆分成多个小块,按批次加载; 或者放弃使用标准库,而是自己处理数据结构,比如使用 `sun.misc.Unsafe` 类, 通过Unsafe工具类可以像C语言一样直接分配内存。



原文链接: <https://plumbr.eu/outofmemoryerror/requested-array-size-exceeds-vm-limit>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

