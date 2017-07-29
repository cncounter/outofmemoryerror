# Java heap space - java.lang.OutOfMemoryError:

## OutOfMemoryError系列（1）: Java heap space

Java applications are only allowed to use a limited amount of memory. This limit is specified during application startup. To make things more complex, Java memory is separated into two different regions. These regions are called Heap space and Permgen (for Permanent Generation):

每个Java程序都只能使用一定量的内存, 这种限制是由JVM的启动参数决定的。而更复杂的情况在于, Java程序的内存分为两部分: 堆内存(Heap space)和 永久代(Permanent Generation, 简称 Permgen):


![](01_01_java-heap-space.png)


The size of those regions is set during the Java Virtual Machine (JVM) launch and can be customized by specifying JVM parameters _-Xmx_ and _-XX:MaxPermSize_. If you do not explicitly set the sizes, platform-specific defaults will be used.

这两个区域的最大内存大小, 由JVM启动参数 `-Xmx` 和 `-XX:MaxPermSize` 指定. 如果没有明确指定, 则根据操作系统类型和物理内存的大小来确定。

The _java.lang.OutOfMemoryError: Java heap space_ error will be triggered when the application **attempts to add more data into the heap space area, but there is not enough room for it**.

假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发`java.lang.OutOfMemoryError: Java heap space` 错误。


Note that there might be plenty of physical memory available, but the _java.lang.OutOfMemoryError: Java heap space_ error is thrown whenever the JVM reaches the heap size limit.

不管机器上还没有空闲的物理内存, 只要堆内存使用量达到最大内存限制,就会抛出 `java.lang.OutOfMemoryError: Java heap space` 错误。

## What is causing it?

## 原因分析


There most common reason for the _java.lang.OutOfMemoryError: Java heap space_ error is simple – you try to fit an XXL application into an S-sized Java heap space. That is – the application just requires more Java heap space than available to it to operate normally. Other causes for this OutOfMemoryError message are more complex and are caused by a programming error:

产生 `java.lang.OutOfMemoryError: Java heap space` 错误的原因, 很多时候, 就类似于将 XXL 号的对象,往 S 号的 Java heap space 里面塞。其实清楚了原因, 就很容易解决对不对?  只要增加堆内存的大小, 程序就能正常运行. 另外还有一些比较复杂的情况, 主要是由代码问题导致的:


*   **Spikes in usage/data volume**. The application was designed to handle a certain amount of users or a certain amount of data. When the number of users or the volume of data suddenly spikes and crosses that expected threshold, the operation which functioned normally before the spike ceases to operate and triggers the _java.lang.OutOfMemoryError: Java heap space_ error.

*   **超出预期的访问量/数据量**。 应用系统设计时,一般是有 “容量” 定义的, 部署这么多机器, 用来处理一定量的数据/业务。 如果访问量突然飙升, 超过预期的阈值, 类似于时间坐标系中针尖形状的图谱, 那么在峰值所在的时间段, 程序很可能就会卡死、并触发 _java.lang.OutOfMemoryError: Java heap space_  错误。


*   **Memory leaks**. A particular type of programming error will lead your application to constantly consume more memory. Every time the leaking functionality of the application is used it leaves some objects behind into the Java heap space. Over time the leaked objects consume all of the available Java heap space and trigger the already familiar _java.lang.OutOfMemoryError: Java heap space_ error.

*   **内存泄露(Memory leak)**. 这也是一种经常出现的情形。由于代码中的某些错误, 导致系统占用的内存越来越多. 如果某个方法/某段代码存在内存泄漏的, 每执行一次, 就会（有更多的垃圾对象）占用更多的内存. 随着运行时间的推移, 泄漏的对象耗光了堆中的所有内存, 那么 _java.lang.OutOfMemoryError: Java heap space_ 错误就爆发了。


## Give me an example

## 具体示例


### Trivial example

### 一个非常简单的示例


The first example is truly simple – the following Java code tries to allocate an array of 2M integers. When you compile it and launch with 12MB of Java heap space (_java -Xmx12m OOM_), it fails with the _java.lang.OutOfMemoryError: Java heap space_ message. With 13MB Java heap space the program runs just fine.

以下代码非常简单, 程序试图分配容量为 2M 的 int 数组. 如果指定启动参数 `-Xmx12m`, 那么就会发生 `java.lang.OutOfMemoryError: Java heap space` 错误。而只要将参数稍微修改一下, 变成 `-Xmx13m`, 错误就不再发生。

```
public class OOM {
    static final int SIZE=2*1024*1024;
    public static void main(String[] a) {
        int[] i = new int[SIZE];
    }
}
```


### Memory leak example

### 内存泄漏示例


The second and a more realistic example is of a memory leak. In Java, when developers create and use new objects _e.g. new Integer(5)_, they don’t have to allocate memory themselves – this is being taken care of by the Java Virtual Machine (JVM). During the life of the application the JVM periodically checks which objects in memory are still being used and which are not. Unused objects can be discarded and the memory reclaimed and reused again. This process is called [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection). The corresponding module in JVM taking care of the collection is called the [Garbage Collector (GC)](https://plumbr.eu/handbook/garbage-collection-algorithms).

这个示例更真实一些。在Java中, 创建一个新对象时, 例如 `Integer num = new Integer(5);` , 并不需要手动分配内存。因为 JVM 自动封装并处理了内存分配. 在程序执行过程中, JVM 会在必要时检查内存中还有哪些对象仍在使用, 而不再使用的那些对象则会被丢弃, 并将其占用的内存回收和重用。这个过程称为 [垃圾收集](http://blog.csdn.net/renfufei/article/details/53432995). JVM中负责垃圾回收的模块叫做 [垃圾收集器(GC)](http://blog.csdn.net/renfufei/article/details/54407417)。


Java’s automatic memory management relies on [GC](https://plumbr.eu/java-garbage-collection-handbook) to periodically look for unused objects and remove them. Simplifying a bit we can say that a **memory leak in Java is a situation where some objects are no longer used by the application but [Garbage Collection](https://plumbr.eu/handbook/garbage-collection-in-jvm) fails to recognize it**. As a result these unused objects remain in Java heap space indefinitely. This pileup will eventually trigger the _java.lang.OutOfMemoryError: Java heap space_ error.

Java的自动内存管理依赖 [GC](http://blog.csdn.net/column/details/14851.html), GC会一遍又一遍地扫描内存区域, 将不使用的对象删除. 简单来说, **Java中的内存泄漏, 就是那些逻辑上不再使用的对象, 却没有被 [垃圾收集程序](http://blog.csdn.net/renfufei/article/details/54144385) 给干掉**. 从而导致垃圾对象继续占用堆内存中, 逐渐堆积, 最后造成 _java.lang.OutOfMemoryError: Java heap space_ 错误。


It is fairly easy to construct a Java program that satisfies the definition of a memory leak:

很容易写个BUG程序, 来模拟内存泄漏:


```
import java.util.*;

public class KeylessEntry {

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
        while (true){
        for (int i = 0; i < 10000; i++){
           if (!m.containsKey(new Key(i))){
               m.put(new Key(i), "Number:" + i);
           }
        }
        System.out.println("m.size()=" + m.size());
        }
    }
}
```


When you execute the above code above you might expect it to run forever without any problems, assuming that the naive caching solution only expands the underlying Map to 10,000 elements, as beyond that all the keys will already be present in the HashMap. However, in reality the elements will keep being added as the Key class does not contain a proper _equals()_ implementation next to its _hashCode()_.

粗略一看, 可能觉得没什么问题, 因为这最多缓存 10000 个元素嘛! 但仔细审查就会发现, `Key` 这个类只重写了 `hashCode()` 方法, 却没有重写 `equals()` 方法, 于是就会一直往 HashMap 中添加更多的 Key。

> 请参考: [Java中hashCode与equals方法的约定及重写原则](http://blog.csdn.net/renfufei/article/details/14163329)


As a result, over time, with the leaking code constantly used, the “cached” results end up consuming a lot of Java heap space. And when the leaked memory fills all of the available memory in the heap region and [Garbage Collection](https://plumbr.eu/handbook/what-is-garbage-collection) is not able to clean it, the _java.lang.OutOfMemoryError:Java heap space_ is thrown.

随着时间推移, “cached” 的对象会越来越多. 当泄漏的对象占满了所有的堆内存, [GC](http://blog.csdn.net/renfufei/article/details/53432995) 又清理不了, 就会抛出 _java.lang.OutOfMemoryError:Java heap space_ 错误。


The solution would be easy – add the implementation for the _equals()_ method similar to the one below and you will be good to go. But before you manage to find the cause, you will definitely have lose some precious brain cells.

解决办法很简单, 在 `Key` 类中恰当地实现 `equals()` 方法即可：

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

说实话, 在寻找真正的内存泄漏原因时, 你可能会死掉很多很多的脑细胞。


### 一个SpringMVC中的场景


译者曾经碰到过这样一种场景: 

为了轻易地兼容从 Struts2 迁移到 SpringMVC 的代码, 在 Controller 中直接获取 request.

所以在 `ControllerBase` 类中通过 `ThreadLocal` 缓存了当前线程所持有的 request 对象:

```
public abstract class ControllerBase {

    private static ThreadLocal<HttpServletRequest> requestThreadLocal = new ThreadLocal<HttpServletRequest>();

    public static HttpServletRequest getRequest(){
        return requestThreadLocal.get();
    }
    public static void setRequest(HttpServletRequest request){
        if(null == request){
        requestThreadLocal.remove();
        return;
        }
        requestThreadLocal.set(request);
    }
}
```

然后在 SpringMVC的拦截器(Interceptor)实现类中, 在 `preHandle` 方法里, 将 request 对象保存到 ThreadLocal 中:


```
/**
 * 登录拦截器
 */
public class LoginCheckInterceptor implements HandlerInterceptor {
    private List<String> excludeList = new ArrayList<String>();
    public void setExcludeList(List<String> excludeList) {
        this.excludeList = excludeList;
    }
    
    private boolean validURI(HttpServletRequest request){
        // 如果在排除列表中
        String uri = request.getRequestURI();
        Iterator<String> iterator = excludeList.iterator();
        while (iterator.hasNext()) {
        String exURI = iterator.next();
        if(null != exURI && uri.contains(exURI)){
            return true;
        }
        }
        // 可以进行登录和权限之类的判断
        LoginUser user = ControllerBase.getLoginUser(request);
        if(null != user){
        return true;
        }
        // 未登录,不允许
        return false;
    }

    private void initRequestThreadLocal(HttpServletRequest request){
        ControllerBase.setRequest(request);
        request.setAttribute("basePath", ControllerBase.basePathLessSlash(request));
    }
    private void removeRequestThreadLocal(){
        ControllerBase.setRequest(null);
    }

    @Override
    public boolean preHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler) throws Exception {
        initRequestThreadLocal(request);
        // 如果不允许操作,则返回false即可
        if (false == validURI(request)) {
        // 此处抛出异常,允许进行异常统一处理
        throw new NeedLoginException();
        }
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler, ModelAndView modelAndView)
        throws Exception {
        removeRequestThreadLocal();
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request,
        HttpServletResponse response, Object handler, Exception ex)
        throws Exception {
        removeRequestThreadLocal();
    }
}
```

在 `postHandle` 和 `afterCompletion` 方法中, 清理 ThreadLocal 中的 request 对象。


但在实际使用过程中, 业务开发人员将一个很大的对象（如占用内存200MB左右的List）设置为 request 的 Attributes， 传递到 JSP 中。

JSP代码中可能发生了异常, 则SpringMVC的`postHandle` 和 `afterCompletion` 方法不会被执行。 

Tomcat 中的线程调度, 可能会一直调度不到那个抛出了异常的线程, 于是 ThreadLocal 一直 hold 住 request。 随着运行时间的推移,把可用内存占满, 一直在执行 Full GC, 系统直接卡死。

后续的修正: 通过 Filter, 在 finally 语句块中清理 ThreadLocal。

```
@WebFilter(value="/*", asyncSupported=true)
public class ClearRequestCacheFilter implements Filter{

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException,
            ServletException {
        clearControllerBaseThreadLocal();
        try {
            chain.doFilter(request, response);
        } finally {
            clearControllerBaseThreadLocal();
        }
    }

    private void clearControllerBaseThreadLocal() {
        ControllerBase.setRequest(null);
    }
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}
    @Override
    public void destroy() {}
}
```

教训是：可以使用 ThreadLocal, 但必须有受控制的释放措施、一般就是 `try-finally` 的代码形式。


> **说明:** SpringMVC 的 Controller 中, 其实可以通过 `@Autowired` 注入 request, 实际注入的是一个 `HttpServletRequestWrapper` 对象, 执行时也是通过 ThreadLocal 机制调用当前的 request。
>
> 常规方式: 直接在controller方法中接收 request 参数即可。



## What is the solution?

## 解决方案


In some cases, the amount of heap you have allocated to your JVM is just not enough to accommodate the needs of your applications running on that JVM. In that case, you should just allocate more heap – see at the end of this chapter for how to achieve that. 

如果设置的最大内存不满足程序的正常运行, 只需要增大堆内存即可, 配置参数可以参考下文。


In many cases however, providing more Java heap space will not solve the problem. For example, if your application contains a memory leak, adding more heap will just postpone the _java.lang.OutOfMemoryError: Java heap space_ error. Additionally, increasing the amount of Java heap space also tends to increase the length of [GC pauses](https://plumbr.eu/handbook/gc-tuning/gc-tuning-in-practice/tuning-for-throughput) affecting your application’s [throughput or latency](https://plumbr.eu/handbook/gc-tuning/throughput-vs-latency-vs-capacity).

但很多情况下, 增加堆内存空间并不能解决问题。比如存在内存泄漏, 增加堆内存只会推迟 _java.lang.OutOfMemoryError: Java heap space_ 错误的触发时间。

当然, 增大堆内存, 可能会增加 [GC pauses](http://blog.csdn.net/renfufei/article/details/55102729#t6) 的时间, 从而影响程序的 [吞吐量或延迟](http://blog.csdn.net/renfufei/article/details/55102729#t7)。


If you wish to solve the underlying problem with the Java heap space instead of masking the symptoms, you need to figure out which part of your code is responsible for allocating the most memory. In other words, you need to answer these questions:

如果想从根本上解决问题, 则需要排查分配内存的代码. 简单来说, 需要解决这些问题:


1.  Which objects occupy large portions of heap

1. 哪类对象占用了最多内存？


2.  where these objects are being allocated in source code

2. 这些对象是在哪部分代码中分配的。



At this point, make sure to clear a couple of days in your calendar (or – see an automated way below the bullet list). Here is a rough process outline that will help you answer the above questions:

要搞清这一点, 可能需要好几天时间。下面是大致的流程:


* Get security clearance in order to perform a heap dump from your JVM. “Dumps” are basically snapshots of heap contents that you can analyze. These snapshot can thus contain confidential information, such as passwords, credit card numbers etc, so acquiring such a dump might not even be possible for security reasons.

* 获得在生产服务器上执行堆转储(heap dump)的权限。“转储”(Dump)是堆内存的快照, 稍后可以用于内存分析. 这些快照中可能含有机密信息, 例如密码、信用卡账号等, 所以有时候, 由于企业的安全限制, 要获得生产环境的堆转储并不容易。


*   Get the dump at the right moment. Be prepared to get a few dumps, as when taken at a wrong time, heap dumps contain a significant amount of  noise and can be practically useless. On the other hand, every heap dump “freezes” the JVM entirely, so don’t take too many of them or your end users start facing performance issues.

* 在适当的时间执行堆转储。一般来说,内存分析需要比对多个堆转储文件, 假如获取的时机不对, 那就可能是一个“废”的快照. 另外, 每次执行堆转储, 都会对JVM进行“冻结”, 所以生产环境中,也不能执行太多的Dump操作,否则系统缓慢或者卡死,你的麻烦就大了。


*   Find a machine that can load the dump. When your JVM-to-troubleshoot uses for example 8GB of heap, you need a machine with more than 8GB to be able to analyze heap contents. Fire up dump analysis software (we recommend [Eclipse MAT](http://www.eclipse.org/mat/), but there are also equally good alternatives available).

* 用另一台机器来加载Dump文件。一般来说, 如果出问题的JVM内存是8GB, 那么分析 Heap Dump 的机器内存需要大于 8GB.  打开转储分析软件(我们推荐[Eclipse MAT](http://www.eclipse.org/mat/) , 当然你也可以使用其他工具)。


*   Detect the paths to GC roots of the biggest consumers of heap. We have covered this activity in a separate post [here](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste). It is especially tough for beginners, but the practice will make you understand the structure and navigation mechanics.

* 检测快照中占用内存最大的 GC roots。详情请参考: [Solving OutOfMemoryError (part 6) – Dump is not a waste](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste)。 这对新手来说可能有点困难, 但这也会加深你对堆内存结构以及navigation机制的理解。


*   Next, you need to figure out where in your source code the potentially hazardous large amount of objects is being allocated. If you have good knowledge of your application’s source code you’ll be able to do this in a couple searches.

* 接下来, 找出可能会分配大量对象的代码. 如果对整个系统非常熟悉, 可能很快就能定位了。


Alternatively, we suggest [Plumbr, the only Java monitoring solution with automatic root cause detection](http://plumbr.eu). Among other performance problems it catches all _java.lang.OutOfMemoryError_s and automatically hands you the information about the most memory-hungry data structres. 

打个广告, 我们推荐 [Plumbr, the only Java monitoring solution with automatic root cause detection](http://plumbr.eu)。 Plumbr 能捕获所有的  _java.lang.OutOfMemoryError_ , 并找出其他的性能问题, 例如最消耗内存的数据结构等等。


Plumbr takes care of gathering the necessary data behind the scenes – this includes the relevant data about heap usage (only the object layout graph, no actual data), and also some data that you can’t even find in a heap dump. It also does the necessary data processing for you – on the fly, as soon as the JVM encounters an _java.lang.OutOfMemoryError_. Here is an example _java.lang.OutOfMemoryError_ incident alert from Plumbr:

Plumbr 在后台负责收集数据 —— 包括堆内存使用情况(只统计对象分布图, 不涉及实际数据),以及在堆转储中不容易发现的各种问题。 如果发生 _java.lang.OutOfMemoryError_ , 还能在不停机的情况下, 做必要的数据处理. 下面是Plumbr 对一个 _java.lang.OutOfMemoryError_ 的提醒:


![Plumbr OutOfMemoryError incident alert](01_02_outofmemoryerror-analyzed.png)


Without any additional tooling or analysis you can see:

强大吧, 不需要其他工具和分析, 就能直接看到:


*   Which objects are consuming the most memory (271 _com.example.map.impl.PartitionContainer_ instances consume 173MB out of 248MB total heap)

* 哪类对象占用了最多的内存(此处是 271 个 _com.example.map.impl.PartitionContainer_ 实例, 消耗了 172MB 内存, 而堆内存只有 248MB)


*   Where these objects were allocated (most of them allocated in the _MetricManagerImpl_ class, line 304)

* 这些对象在何处创建(大部分是在 _MetricManagerImpl_ 类中,第304行处)


*   What is currently referencing these objects (the full reference chain up to GC root)

* 当前是谁在引用这些对象(从 GC root 开始的完整引用链)


Equipped with this information you can zoom in to the underlying root cause and make sure the data structures are trimmed down to the levels where they would fit nicely into your memory pools.

得知这些信息, 就可以定位到问题的根源, 例如是当地精简数据结构/模型, 只占用必要的内存即可。

However, when your conclusion from memory analysis or from reading the Plumbr report are that memory use is legal and there is nothing to change in the source code, you need to allow your JVM more Java heap space to run properly. In this case, alter your JVM launch configuration and add (or increase the value if present) the following:

当然, 根据内存分析的结果, 以及Plumbr生成的报告, 如果发现对象占用的内存很合理, 也不需要修改源代码的话, 那就增大堆内存吧。在这种情况下,修改JVM启动参数, (按比例)增加下面的值:


    -Xmx1024m

The above configuration would give the application 1024MB of Java heap space. You can use g or G for GB, m or M for MB, k or K for KB. For example all of the following are equivalent to saying that the maximum Java heap space is 1GB:

这里配置Java堆内存最大为 `1024MB`。可以使用 `g/G` 表示 GB, `m/M` 代表 MB, `k/K` 表示 KB. 

下面的这些形式都是等价的, 设置Java堆的最大空间为 1GB:

    # 等价形式: 最大1GB内存
    java -Xmx1073741824 com.mycompany.MyClass
    java -Xmx1048576k com.mycompany.MyClass
    java -Xmx1024m com.mycompany.MyClass
    java -Xmx1g com.mycompany.MyClass 


原文链接: <https://plumbr.eu/outofmemoryerror/java-heap-space>

翻译日期: 2017年7月29日

翻译人员: [铁锚: http://blog.csdn.net/renfufei](http://blog.csdn.net/renfufei)

