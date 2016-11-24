# Java Virtual Machine Tuning
1. HotSpot虚拟器有两种可供选择，如果追求更快的启动速度和更小的内存占用，则选32位的client虚拟器；如果追求更高的吞吐量，则选32位或64位的server虚拟器；
2. Client Runtime是快速启动，更小的内存占用以及快速代码（机器码）生成的JIT编译器;
3. Server Runtime开放编译优化选项，便于生成高性能代码；
4. Tiered Runtime综合Client Runtime和Server Runtime优点，目前还在开发中。它拥有较快的启动时间以及较高的代码编译性能。在Java 6u25，Java 7及后续更新的版本中，选项 -server -XX：+TieredCompilation可开启这种运行环境。
5. JVM的Throughput垃圾器采用选项-XX:+UseParallelOldGC或+XX:+UseParallelGC开启。两者不同为：-XX:+UseParallelOldGC触发young代和old代均采用多线程进行垃圾回收，即Minor GC和Full GC都是多线程的；而+XX:+UseParallelGC仅仅是young代的GC是多线程的，old代的GC仍为单线程。

# GC "三原则"
1. Minor GC回收原则：在minor垃圾回收器中，最大量的对象被回收。目的是减少Full GC的数量和频率。
2. GC最大内存原则：将尽可能多的内存分配给垃圾回收器。所以更大的Java堆空间，GC器和应用在吞吐量和延迟上的性能更好。
3. 2/3 GC优化原则：优化JVM垃圾回收器的3个指标(吞吐量、延迟和内存占用)中的2个。

# 打印信息选项

-XX:+PrintGCTimeStamps
-XX:+PrintGCDateStamps

-XX:+PrintGCDetails
-Xloggc:<filename>

-XX:+PrintGCApplicationStoppedTime  打印Java程序进入安全点操作的时间
-XX:+PrintGCApplicationConcurrentTime 打印当前时间
-XX:+PrintSafepointStatistics 可以区别垃圾回收的安全点及其他的安全点
这两个选项可以看出java进入安全点操作用了多长时间。当在应用响应时间超过预期时，是安全点操作引起的还是应用或操作系统其他操作引起的。

注：安全点（safepoint）操作  会让JVM进入一种所有应用程序的线程都被阻塞以及阻止任何正在执行的本地程序把结果返回给Java代码的状态。当需要进行优化虚拟机内部操作的时候，安全点（Safepoint操作会被执行以使得所有线程都进入阻塞状态避免影响Java堆，垃圾回收是一种安全点操作。

# 堆大小调优经验
* 年轻代大小通过观察GC打印的信息，观察MinorGC的时间消耗和频率来调节的；
* MinorGC的时间消耗和年轻代理存活的对象数量相关；
* 规律：年轻代空间越小，MinorGC所用时间越短；如果MinorGC的时间消耗不是约束条件，减少年轻代大小会导致MinorGC变得更加频繁，较小的空间意味着用完空间也会用更少的时间；反之亦然，增加年轻代空间会降低MinorGC的频率；
* 经验1：当观察垃圾回收数据的时候，发现MinorGC的时间太长了，则减少年轻代空间大小；如果发现MinorGC太频繁了，则增加年轻代空间大小。如果发现FullGC的频率过高，则增加年老代空间，如果FullGC的耗时太长，则减少年老代空间大小。
* 经验2：每次只调优一个变量，调优年轻代空间大小时保持年老代空间大小不变；同样，调优年老代大小时，也要保持年轻代空间大小不变，这样才能保证调优高效；注意因为调节年轻代和年老代大小时，要保持另一个空间大小不变，因而需要同时修改java堆分配空间的大小；
* 经验3：年老代空间不小于活动对象所占空间的1.5倍；年轻代空间大小不小于java堆空间的10%； 提高Java堆大小时，堆空间不超过系统物理内存空间的90%，过高的堆空间会导致系统使用交换区；


