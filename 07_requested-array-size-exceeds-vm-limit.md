# java.lang.OutOfMemoryError: **Requested array size exceeds VM limit**

Java has got a limit on the maximum array size your program can allocate. The exact limit is platform-specific but is generally somewhere between 1 and 2.1 billion elements.



![outofmemoryerror](./07_01_array-size-exceeds-vm-limit.png)



When you face the _java.lang.OutOfMemoryError: Requested array size exceeds VM limit_, this means that the application that crashes with the error is trying to allocate an array larger than the Java Virtual Machine can support.

## What is causing it?

The error is thrown by the native code within the JVM. It happens before allocating memory for an array when the JVM performs a platform-specific check: whether the allocated data structure is addressable in this platform. This error is less common than you might initially think.

The reason you only seldom face this error is that Java arrays are indexed by int. The maximum positive int in Java is `2^31 – 1 = 2,147,483,647`. And the platform-specific limits can be really close to this number – for example on my 64bit MB Pro on Java 1.7 I can happily initialize arrays with up to 2,147,483,645 or _Integer.MAX_VALUE-2_ elements.

Increasing the length of the array by one to Integer.MAX_VALUE-1 results in the familiar _OutOfMemoryError_:

```
`Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit`
```

But the limit might not be that high – on 32-bit Linux with OpenJDK 6, you will hit the “_java.lang.OutOfMemoryError: Requested array size exceeds VM limit_” already when allocating an array with ~1.1 billion elements. To understand the limits of your specific environments run the small test program described in the next chapter.


## Give me an example

When trying to recreate the _java.lang.OutOfMemoryError: Requested array size exceeds VM limit_ error, let’s look at the following code:

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

Note that before facing _java.lang.OutOfMemoryError: Requested array size exceeds VM limit_ on the last two attempts, the allocations failed with a lot more familiar _java.lang.OutOfMemoryError: Java heap space_ message. It happens because the 2^31-1 int primitives you are trying to make room for require 8G of memory which is less than the defaults used by the JVM.

This example also demonstrates why the error is so rare – in order to see the VM limit on array size being hit, you need to allocate an array with the size right in between the platform limit and Integer.MAX_INT. When our example is run on 64bit Mac OS X with Hotspot 7, there are only two such array lengths: Integer.MAX_INT-1 and Integer.MAX_INT.

## What is the solution?

The _java.lang.OutOfMemoryError: Requested array size exceeds VM limit_ can appear as a result of either of the following situations:

*   Your arrays grow too big and end up having a size between the platform limit and the _Integer.MAX_INT_

*   You deliberately try to allocate arrays larger than 2^31-1 elements to experiment with the limits.

In the first case, check your code base to see whether you really need arrays that large. Maybe you could reduce the size of the arrays and be done with it. Or divide the array into smaller bulks and load the data you need to work with in batches fitting into your platform limit.

In the second case – remember that Java arrays are indexed by int. So you cannot go beyond 2^31-1 elements in your arrays when using the standard data structures within the platform. In fact, in this case you are already blocked by the compiler announcing “_error: integer number too large_” during compilation.

But if you really work with truly large data sets, you need to rethink your options. You can load the data you need to work with in smaller batches and still use standard Java tools, or you might go beyond the standard utilities. One way to achieve this is to look into the _sun.misc.Unsafe_ class. This allows you to allocate memory directly like you would in C.


原文链接: <https://plumbr.eu/outofmemoryerror/requested-array-size-exceeds-vm-limit>

翻译日期: 2017年9月21日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

