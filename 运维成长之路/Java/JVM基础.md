# 1. JDK、JRE、JVM 的区别
![[Pasted image 20221129135819.png]]

JDK 包括了 Java 运行时的环境（JRE）、解释器（Java）、编译器（javac）、Java 归档（jar）、文档生成器（Javadoc）等工具

jdk-8u202-linux-x64.tar.gz
版本，8u后面的数字越大，版本越新
[Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/#java17)

# 2. 系统性能指标

系统资源：
CPU  内存   IO（存储+网络）

性能维度：
延迟     响应时间
吞吐量     TPS  QPS
系统容量


性能指标
	业务需求指标：吞吐量、响应时间、并发数、业务成功率
	资源约束指标：CPU、内存、IO

我们可采用的手段和方式包括：

-   使用 JDWP 或开发工具做本地/远程调试
-   系统和 JVM 的状态监控，收集分析指标
-   性能分析: CPU 使用情况/内存分配分析
-   内存分析: Dump 分析/GC 日志分析
-   调整 JVM 启动参数，GC 策略等等

等有瓶颈了，再考虑优化、否则就是过度优化。


# 3. Java类加载器

Java源码——编译成字节码class文件——JVM 运行
class文件可以打包成jar文件
javac 编译源码
javap 反编译源码


JVM 的启动参数, 从形式上可以简单分为：

-   以`-`开头为标准参数，所有的 JVM 都要实现这些参数，并且向后兼容。
-   以`-X`开头为非标准参数， 基本都是传给 JVM 的，默认 JVM 实现这些参数的功能，但是并不保证所有 JVM 实现都满足，且不保证向后兼容。
-   以`-XX:`开头为非稳定参数, 专门用于控制 JVM 的行为，跟具体的 JVM 实现有关，随时可能会在下个版本取消。
-   `-XX:+-Flags` 形式, `+-` 是对布尔值进行开关。
-   `-XX:key=value` 形式, 指定某个选项的值。

查看默认的所有系统属性，可以使用命令：
$ java -XshowSettings:properties -version

查看默认 VM 设置：
java -XshowSettings:vm -version


## JVM内存设置
JVM 总内存=堆+栈+非堆+堆外内存。
-   `-Xmx`, 指定最大堆内存。 如 `-Xmx4g`. 这只是限制了 Heap 部分的最大值为 4g。这个内存不包括栈内存，也不包括堆外使用的内存。
-   `-Xms`, 指定堆内存空间的初始大小。 如 `-Xms4g`。 而且指定的内存大小，并不是操作系统实际分配的初始值，而是 GC 先规划好，用到才分配。 专用服务器上需要保持 `-Xms`和`-Xmx`一致，否则应用刚启动可能就有好几个 FullGC。当两者配置不一致时，堆内存扩容可能会导致性能抖动。
-   `-Xmn`, 等价于 `-XX:NewSize`，使用 G1 垃圾收集器 **不应该** 设置该选项，在其他的某些业务场景下可以设置。官方建议设置为 `-Xmx` 的 `1/2 ~ 1/4`。
-   `-XX:MaxPermSize=size`, 这是 JDK1.7 之前使用的。Java8 默认允许的 Meta 空间无限大，此参数无效。
-   `-XX:MaxMetaspaceSize=size`, Java8 默认不限制 Meta 空间, 一般不允许设置该选项。
-   `XX:MaxDirectMemorySize=size`，系统可以使用的最大堆外内存，这个参数跟`-Dsun.nio.MaxDirectMemorySize`效果相同。
-   `-Xss`, 设置每个线程栈的字节数。 例如 `-Xss1m` 指定线程栈为 1MB，与`-XX:ThreadStackSize=1m`等价


### 最佳实践

#### 配置多少 xmx 合适

从上面的分析可以看到，系统有大量的地方使用堆外内存，远比我们常说的 xmx 和 xms 包括的范围要广。所以我们需要在设置内存的时候留有余地。

实际上，我个人比较推荐配置系统或容器里可用内存的 70-80% 最好。比如说系统有 8G 物理内存，系统自己可能会用掉一点，大概还有 7.5G 可以用，那么建议配置

> -Xmx6g 说明：xmx : 7.5G*0.8 = 6G，如果知道系统里有明确使用堆外内存的地方，还需要进一步降低这个值。



## 命令行诊断分析工具

jps 获取jvm进程id
jstat -gcutil -t  jvmid

Timestamp         S0     S1     E      O      M     CCS    YGC     YGCT     FGC    FGCT     CGC    CGCT       GCT   
       694645.4   0.00   9.30  21.07  55.41  99.30  96.69    788     3.318     0     0.000    10     0.058     3.376

显示的百分比

### 查看gc
jstat -gc -t 864 1s 3 
jstat -gc -t -h 10 864 1s 15
1s 每1秒输出一次 3次
-h 10 10行输出一次表头
Timestamp           S0C         S1C         S0U         S1U          EC           EU           OC           OU          MC         MU       CCSC      CCSU     YGC     YGCT     FGC    FGCT     CGC    CGCT       GCT   
       694846.4         0.0      1024.0         0.0        95.2     325632.0     141312.0     190464.0     105539.0   109568.0   108803.4   12864.0   12437.8    788     3.318     0     0.000    10     0.058     3.376
       694847.4         0.0      1024.0         0.0        95.2     325632.0     141312.0     190464.0     105539.0   109568.0   108803.4   12864.0   12437.8    788     3.318     0     0.000    10     0.058     3.376
       694848.4         0.0      1024.0         0.0        95.2     325632.0     142336.0     190464.0     105539.0   109568.0   108803.4   12864.0   12437.8    788     3.318     0     0.000    10     0.058     3.376
-   OU：Old 区，老年代的使用量，单位 kB。 （需要关注）
-   YGCT：年轻代 GC 消耗的总时间。 （重点关注）
-   FGCT：Full GC 消耗的时间。 （重点关注）

### 查看堆内存
JDK 8用这个
jmap -heap jvmid  
JDK 9以上用
jhsdb jmap --heap --pid 82754
问题：
	docker 环境下，centos7版本无法使用jhsdb，报错
	ERROR: ptrace(PTRACE_ATTACH, ..) failed for 1: Operation not permitted
	Error attaching to process: Can't attach to the process: ptrace(PTRACE_ATTACH, ..) failed for 1: Operation not permitted
	8版本正常



jinfo 用来查看具体生效的配置信息以及系统属性
jinfo -flags pid
VM Flags:
-XX:CICompilerCount=2 -XX:ConcGCThreads=1 -XX:G1ConcRefinementThreads=2 -XX:G1HeapRegionSize=1048576 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=62914560 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=979369984 -XX:MaxNewSize=587202560 -XX:MinHeapDeltaBytes=1048576 -XX:NonNMethodCodeHeapSize=5825164 -XX:NonProfiledCodeHeapSize=122916538 -XX:ProfiledCodeHeapSize=122916538 -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseG1GC 

jcmd id VM.command_line 查看启动参数
![[Pasted image 20221129153352.png]]

JMX 监控

[性能测试中服务器关键性能指标浅析 - 简书 (jianshu.com)](https://www.jianshu.com/p/62cf2690e6eb)

参考
[note-submit (lianglianglee.com)](https://learn.lianglianglee.com/)

java -D参数，设置系统属性
-XX 设置jvm启动参数

### 内存溢出与内存泄漏
内存溢出（out of memory）：申请内存，JVM没有足够的内存空间

内存泄漏（Memory Leak）：申请了内存，没有释放

绝大部分情况下，并不需要特意去调优JVM，因为那是最后一步的优化手段。即使真的需要，到时候再研究也不迟，因为时间是宝贵的，在解决自己程序的性能问题之前，不必在意JVM的性能。