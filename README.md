# java.lang.OutOfMemoryError 实例详解 
### The 8 symptoms that surface them 
### 内存溢出的8种主要类型



The many thousands of java.lang.OutOfMemoryErrors that I’ve met during my career all bear one of the below eight symptoms. This handbook explains what causes a particular error to be thrown, offers code examples that can cause such errors, and gives you solution guidelines for a fix. The content is all based on my own experience.

笔者在工作中碰到过成千上万的 `java.lang.OutOfMemoryError`, 都可以归结为以下八种症状。
本手册阐述了各种内存溢出错误的形成原因,并提供了可测试这种错误的示例代码,以及解决方案。 内容都来源于笔者的一线开发和实践经验。



目录:



1. [java.lang.OutOfMemoryError:**Java heap space**](https://plumbr.eu/outofmemoryerror/java-heap-space)
2. [java.lang.OutOfMemoryError:**GC overhead limit exceeded**](https://plumbr.eu/outofmemoryerror/gc-overhead-limit-exceeded)
3. [java.lang.OutOfMemoryError:**Permgen space**](https://plumbr.eu/outofmemoryerror/permgen-space)
4. [java.lang.OutOfMemoryError:**Metaspace**](https://plumbr.eu/outofmemoryerror/metaspace)
5. [java.lang.OutOfMemoryError:**Unable to create new native thread**](https://plumbr.eu/outofmemoryerror/unable-to-create-new-native-thread)
6. [java.lang.OutOfMemoryError:**Out of swap space?** ](https://plumbr.eu/outofmemoryerror/out-of-swap-space)
7. [java.lang.OutOfMemoryError:**Requested array size exceeds VM limit** Details](https://plumbr.eu/outofmemoryerror/requested-array-size-exceeds-vm-limit)
8. [Out of memory:**Kill process or sacrifice child**](https://plumbr.eu/outofmemoryerror/kill-process-or-sacrifice-child)













原文作者: **Nikita Salnikov-Tarnovski**  -- Plumbr Co-Founder and VP of Engineering




原文链接： https://plumbr.eu/outofmemoryerror
