# 八、垃圾回收

关于垃圾收集有三个经典问题：
1、哪些内存需要回收？
2、什么时候回收？
3、如何回收？

## 1、垃圾回收概述

> 什么是垃圾？

垃圾是指在运行程序中**没有任何指针指向的对象**，这个对象就是需要被回收的垃圾

如果不及时对内存中的垃圾进行清理，那么，这些垃圾对象所占的内存空间会一直保留到应用程序结束，被保留的空间无法被其他对象使用。甚至可能导致内存溢出



> 为什么需要 **GC**

对于高级语言来说，一个基本认知是**如果不进行垃圾回收，内存迟早都会被消耗完**，因为不断地分配内存空间而不进行回收，就好像不停地生产生活垃圾而从来不打扫一样

除了释放没用的对象，**垃圾回收也可以清除内存里的记录碎片**。碎片整理将所占用的堆内存移到堆的一端，以便 Java 将整理出的内存分配给新的对象

随着应用程序所应付的业务越来越庞大、复杂，用户越来越多，**没有 GC 就不能保证应用程序的正常进行**。而经常造成 **STW** 的 **GC** 又跟不上实际的需求，所以才会不断地尝试对 **GC** 进行优化



### 1.1 早期垃圾回收

在早期的 C/C++时代，垃圾回收基本上是手工进行的。开发人员可以使用 new 关键字进行内存申请，并使用 delete关键字进行内存释放。比如以下代码

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708075039.png)

这种方式可以灵活控制内存释放的时间，但是**会给开发人员带来频繁申请和释放内存的管理负担**。倘若有一处内存区间由于程序员编码的问题忘记被回收，那么就会产生**内存泄漏**，垃圾对象永远无法被清除，随着系统运行时间的不断増长，垃圾对象所耗内存可能持续上升，直到出现内存溢岀并造成应用程序崩溃



### 1.2 Java 垃圾回收机制

> 自动内存管理

无需开发人员手动参与内存的分配与回收，这样降低内存泄漏和内存溢出的风险

自动内存管理机制，将程序员从繁重的内存管理中释放出来，可以更专心地专注于业务开发



> 担忧

对于Java开发人员而言，自动内存管理就像是一个黑匣子，如果过度依赖于 “自动”，那么这将会是一场灾难，最严重的就会弱化 Java 开发人员在程序出现内存溢出时定位问题和解决问题的能力

此时，了解 JVM 的自动内存分配和内存回收原理就显得非常重要，只有在真正了解 JVM 是如何管理内存后，我们才能够在遇见 OutOfMemoryError 时，快速地根据错误异常日志定位问题和解决问题

当需要排査各种内存溢出、内存泄漏问题时，当垃圾收集成为系统达到更高并发量的瓶颈时，我们就必须对这些“自动化” 的技术实施必要的监控和调节



## 2、垃圾回收算法

标记阶段：引用计数算法、可达性分析算法

清除阶段：标记-清除算法、复制算法、标记压缩算法



### 2.1 标记阶段-引用计数算法

> 对象存活判断

在堆里存放着几乎所有的 Java 对象实例，**在 GC 执行垃圾回收之前，首先需要区分出内存中哪些是存活对象，哪些是已经死亡的对象**。只有被标记为己经死亡的对象，GC 才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为**垃圾标记阶段** 



判断对象存活一般有两种方式：引计数算法、可达性分析算法

> 如何标记一个死亡对象

简单来说，**当一个对象已经不再被任何的存活对象继续引用时**，就可以宣判为已经死亡



> 引用计数算法

对每个对象保存一个**整型的引用计数器属性**，用于记录对象被引用的情况

对于一个对象 A，只要有任何一个对象引用了 A，则 A 的引用计数器就加 1，当引用失效时，引用计数器就减 1，只要对象 A 的引用计数器的值为 0，即表示对象 A 不可能再被使用，可进行回收

* 优点

实现简单，垃圾对象便于辨识，判定效率高，回收没有延迟性

* 缺点

它需要单独的字段存储计数器，这样的做法增加了存储空间的开销

每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销

**引用计数器有一个严重的问题，即无法处理循环引用的情况**。这是一条致命缺陷，导致在 Java 的垃圾回收器没有使用这类算法

>循环引用

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708085846.png)

> 举例

```java
public class RefCount {
private byte[] bytes = new byte[1024 * 1024 * 5];

Object reference = null;

public static void main(String[] args) {
RefCount refCount1 = new RefCount();
RefCount refCount2 = new RefCount();

refCount1.reference = refCount2;
refCount2.reference = refCount1;

refCount1 = null;
refCount2 = null;

/**
 * 显示调用 GC，对比没有 GC 的情况
 * 如果回收了，则说明没有使用引用计数算法
 */
System.gc();
}
}
```

* 没有回收时

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708090934.png)

* 有回收时

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708091042.png)

**说明没有使用引用计数算法** 



> 小结

引用计数算法，是很多语言的资源回收选择，例如因人工智能而更加火热的 **Python，它更是同时支持引用计数和垃圾收集机制** 

具体哪种最优是要看场景的，业界有大规模实践中仅保留引用计数机制，以提高吞吐量的尝试

> Python如何解决循环引用？

手动解除：很好理解，就是在合适的时机，解除引用关系

使用弱引用 **weakref**，**weakref** 是 **Python** 提供的标准库，旨在解决循环引用



### 2.2 标记阶段-可达性分析算法

也可以叫做==**根搜索算法、追踪性垃圾收集**== 

相对于引用计数算法而言，可达性分析算法不仅同样具备实现简单和执行高效等特点，更重要的是**该算法可以有效地解决在引用计数算法中==循环引用==的问题，防止内存泄漏的发生** 

相较于引用计数算法，这里的可达性分析就是 Java、c# 选择的。这种类型的垃圾收集通常也叫作追**踪性垃圾收集** 



所谓 **"GC Roots" 根集合就是一组必须活跃的引用** 

基本思路：

可达性分析算法是**以根对象集合（ GC Roots）为起始点**，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达

使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为**引用链** 

如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象 

在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708092116.png)



> 在 Java 语言中， GC Roots 包括以下几类元素：

1、虚拟机栈中引用的对象
比如：各个线程被调用的方法中使用到的参数、局部变量等

2、本地方法栈内 **JNI**（通常说的本地方法）引用的对象

3、方法区中**类静态属性**引用的对象
比如：Java 类的引用类型静态变量

4、方法区中**常量**引用的对象
比如：字符串常量池（ String Table）里的引用

5、所有被同步锁 **synchronized** 持有的对象

6、Java 虚拟机内部的引用
基本数据类型对应的 Class 对象，一些常驻的异常对象（如：NullPointerException、OutOfMemoryError），系统类加载器

7、反映 Java 虛拟机内部情况的 JMXBean、JVMTI 中注册的回调、本地代码缓存等



* 小技巧

由于 **Root** 采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个 **Root** 



* 注意

如果要使用可达性分析算法来判断内存是否可回收，那么**分析工作必须在个能保障一致性的快照中进行**。这点不满足的话分析结果的准确性就无法保证

这点也是导致 **GC 进行时必须  "Stop The World"** 的一个重要原因，即使是号称（几乎）不会发生停顿的 CMS 收集器中，枚举根节点时也是必须要停顿的



> HotSpot 获取根节点的方法

我们以可达性分析算法中从 GC Roots 集合找引用链这个操作作为介绍虚拟机高效实现的第一个例子

**固定可作为 GC Roots 的节点主要在==全局性的引用==（例如常量或类静态属性）与==执行上下文==（例如栈帧中的局部变量表）中**，尽管目标明确，但査找过程要做到高效并非一件容易的事情，现在 Java 应用越做越庞大，光是方法区的大小就常有数百上千兆，里面的类、常量等更是恒河沙数，**若要逐个检查以这里为起源的引用肯定得消耗不少时间** 

其实并不需要一个不漏地检查完所有执行上下文和全局的引用位置，虚拟机应当是有办法直接得到哪些地方存放着对象引用的

在 **HotSpot** 的解决方案里，是**==使用一组称为 Oop Map 的数据结构==**来达到这个目的。一旦类加载动作完成的时候， HotSpot 就会把对象内什么偏移量上是什么类型的数据计算出来，在即时编译过程中，也会在特定的位置记录下栈里和寄存器里哪些位置是引用，这样收集器在扫描时就可以直接得知这些信息了，并不需要真正一个不漏地从方法区等 GC Roots 开始查找



### 2.3 对象的 finalization 机制

Java 语言提供了对象终止（ finalization）机制来**允许开发人员提供对象被销毁之前的自定义处理逻辑** 

当垃圾回收器发现没有引用指间一个对象，即：**垃圾回收此对象之前，总会先调用这个对象的 finalize() 方法** 

**finalize()** 方法允许在子类中被重写，用于在对象被回收时进行资源释放。**通常在这个方法中进行一些资源释放和清理的工作**，比如关闭文件、套接字和数据库连接等

> **==永远不要主动调用某个对象的 finalize() 方法，应该交给垃圾回收机制调用==** 

1、在 **finalize()** 时可能会导致对象复活

2、**finalize()** 方法的执行时间是没有保障的，它完全由 GC 线程决定，极端情况下，若不发生 GC，则 **finalize()** 方法将没有执行机会

3、一个糟糕的 **finalize()** 会严重影响 GC 的性能



> 由于 **finalize()** 方法的存在，虚拟机中的对象一般处于三种可能的状态

如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说此对象需要被回收。但事实上，也并非是 “非死不可” 的，这时候它们暂时处于 “缓刑” 阶段。**一个无法触及的对象有可能在某一个条件下 “复活” 自己，如果这样，那么对它的回收就是不合理的**，为此，定义虚拟机中的对象可能的三种状态：

1、可触及的：从根节点开始，可以到达这个对象

2、可复活的：对象的所有引用都被释放，但是对象有可能在 **finalize()** 中复活

3、不可触及的：对象的 **finalize()** 被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，因为 **==finalize() 只会被调用一次==** 

以上 3 种状态中，是由于 **finalize()** 方法的存在，进行的区分。只有在对象不可触及时才可以被回收



> 对象回收具体过程

1、如果对象 **objA** 到 **GC Roots** 没有引用链，则进行**第一次标记** 

2、进行筛选，判断此对象是否有必要执行 **finalize()** 方法

如果对象 **objA** 没有重写 **finalize()** 方法或者 **finalize()** 方法已经被虚拟机调用过，则虚拟机视为 “没有必要执行”，**objA** 被判定为不可触及的

如果对象 **objA** 重写了 **finalize()** 方法，且还未执行过，那么 **objA** 会被插入到 **F-Queue** 队列中，由一个虚拟机自动创建的、低优先级的 **Finalizer** 线程触发其 **finalize()** 方法执行

**finalize()** 方法是对象逃脱死亡的最后机会，稍后 **GC** 会对 **F-Queue** 队列中的对象进行第二次标记。如果 **objA** 在**finalize()** 方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，**objA** 会被移出 “即将回收” 集合。之后，对象会再次出现没有引用存在的情况。在这个情况下，**finalize()** 方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，**==一个对象的 finalize 方法只会被调用一次==** 

要尽量避免使用该方法，因为它并不能等同于 C 和 C++ 语言中的析构函数，而是 Java 刚诞生时为了使传统C、C++ 程序员更容易接受 Java 所做出的一项妥协。**它的运行代价高昂，不确定性大，无法保证各个对象的调用顺序，如今已被官方明确声明为不推荐使用的语法**。有些教材中描述它适合做 “关闭外部资源” 之类的清理性工作，这完全是对finalize() 方法用途的一种自我安慰。finalize() 能做的所有工作，使用 try-finally 或者其他方式都可以做得更好、更及时，所以建议大家完全可以忘掉 Java 语言里面的这个方法

> 代码复活演示

```java
public class ReLive {
public static ReLive obj;  // 类变量

@Override
protected void finalize() throws Throwable {
super.finalize();
System.out.println("调用当前类的 finalize 方法....");
obj = this;
}

public static void main(String[] args) {

try {
obj = new ReLive();

// 第一次拯救自己
obj = null;
System.gc();
System.out.println("第一次 GC");
// 因为 Finalizer 线程优先级很低，等待一会儿，让它执行
Thread.sleep(2000);
if (obj == null) {
System.out.println("obj is dead");
} else {
System.out.println("obj is alive");
}

obj = null;
System.gc();
System.out.println("第二次 GC");
Thread.sleep(2000);
if (obj == null) {
System.out.println("obj is dead");
} else {
System.out.println("obj is alive");
}


} catch (InterruptedException e) {
e.printStackTrace();
}
}
}
```

* 执行结果

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708111406.png)



### 2.4 清除阶段: 标记-清除算法

当成功区分出内存中存活对象和死亡对象后，**GC** 接下来的任务就是执行拉圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存

目前在 **JVM** 中比较常见的三种垃圾收集算法是 **标记一清除算法、复制算法、标记-压缩算法** 

标记-清除算法（**Mark- Sweep**）是一种非常基础和常见的垃圾收集算法，该算法被 **J.McCarthy** 等人在1960年提出并应用于 **Lisp** 语言



> 执行过程

当堆中的有效内存空间（ available memory）被耗尽的时候，就会停止整个程序（也被称为 Stop The World）然后进行两项工作，第一项则是**标记**，第二项则是**清除** 

标记：**Collector** 从引用根节点开始遍历，**标记所有被引用的对象。一般是==在对象的 Header 中记录为可达对象==** 
​清除：**Collector** 对堆内存从头到尾进行线性的遍历，如果发现某个对象在其 **Header** 中没有标记为可达对象，则将其回收

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200713162800.png)



* 缺点

效率不算高

在进行GC的时候，需要停止整个应用程序，导致用户体验差

这种方式**清理出来的空闲内存是不连续的，会产生内存碎片**，需要维护一个空闲列表



> 何为清除？

这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放（覆盖）



### 2.5 清除阶段：复制算法

**为了解决标记-清除算法在垃圾收集==效率==方面的缺陷**，**M.L.Minsky** 于 1963 年发表了著名的论文，“使用双存储区的Lisp 语言垃圾收集器 CA LISP Garbage Collector Algorithm Using Serial Secondary Storage”。**M.L.Minsky 在该论文中描述的算法被人们称为复制（ Copying）算法**，它也被 M.L.Minsky 本人成功地引入到了 Lisp 语言的一个实现版本中

> 算法思想

**将活着的内存空间分为两块，每次只使用其中一块**，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块，交换两个内存的角色，最后完成垃圾回收

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708143022.png)

* 优点

1、没有标记和清除过程，实现简单，运行高效

2、复制过去以后保证空间的连续性，不会出现 “碎片” 问题

* 缺点

1、此算法的缺点也是很明显的，就是需要两倍的内存空间

2、对于 G1 这种分拆成为大量 region 的 GC，复制而不是移动，意味着 GC 需要维护 region 之间对象引用关系，不管是内存占用或者时间开销也不小

* 特别的

如果系统中的垃圾对象很多，复制算法不是很理想，因为**复制算法需要复制的存活对象数量并不会太多**，或者说非常低才行



### 2.6 清除阶段：标记-压缩算法

> 也叫做**标记-整理算法** 

**复制算法的高效性是建立在==存活对象少、垃圾对象多==的前提下的**。这种情况在新生代经常发生，但是在老年代，更常见的情况是大部分对象都是存活对象。如果依然使用复制算法由于存活对象较多，复制的成本也将很高。因此，**基于老年代垃圾回收的特性，需要使用其他的算法 ** 

标记一清除算法的确可以应用在老年代中，但是该算法不仅执行效率低下，而且在执行完存回收后迅会产生内存碎片，所以 JVM 的设计者需要在此基础之上进行改进。**标记压缩（Mark- Compact）算法**由此诞生

> 执行过程

第一阶段和标记清除算法一样从根节点开始标记所有被引用对象

第二阶段将所有的存活对象压缩到内存的一端，按顺序排放

之后，清理边界外所有的空间



![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708144518.png)



* 优点

1、消除了标记-清除算法当中，内存区域分散的缺点，我们需要给新对象分配内存时，JVM 只需要持有一个内存的起始地址即可

2、消除了复制算法当中，内存减半的高额代价

* 缺点

1、从效率上来说，标记-整理算法要低于复制算法

2、移动对象的同时，如果对象被其他对象引用，则还需要调整引用的地址

3、移动过程中，需要全程暂停用户应用程序。即：STW



### 2.7 清除阶段三种算法比较

|              | Mark-Sweep     | Mark-Compact | Copying               |
| ------------ | -------------- | ------------ | --------------------- |
| 速度         | 中等           | 最慢         | 最快                  |
| 空间开销     | 少（但有碎片） | 少（无碎片） | 通常需要存活对象的2倍 |
| 是否移动对象 | 否             | 是           | 是                    |



效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存

而为了尽量兼顾上面提到的三个指标，**标记-整理算法相对来说更平滑一些，但是效率上不尽如人意**，它比复制算法多了一个标记的阶段，比标记-清除多了一个整理内存的阶段



### 2.8 分代收集算法

分代收集算法，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，**不同生命周期的对象可以采取不同的收集方式，以便提高回收效率**。一般是把 Java 堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率

在 Java 程序运行的过程中，会产生大量的对象，其中有些对象是与业务信息相关，比如 Http 请求中的 Session 对象、线程、Socket 连接，这类对象跟业务直接挂钩，因此生命周期比较长。但是还有一些对象，主要是程序运行过程中生成的临时变量，这些对象生命周期会比较短，比如：String 对象，由于其不变类的特性，系统会产生大量的这些对象，有些对象甚至只用一次即可回收

**==目前几乎所有的 GC 都是采用分代收集算法执行垃圾回收的==** 

在 HotSpot 中，基于分代的概念，GC 所使用的内存回收算法必须结合年轻代和老年代各自的特点

* 年轻代
    * 年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁

**这种情况使用==复制算法==的回收整理，速度是最快的**。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过 **Hotspot** 中的两个 **survivor** 的设计得到缓解

* 老年代
    * 老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁

这种情况存在大量存活率高的对象，复制算法明显变得不合适。**一般是由标记-清除或者是标记-清除与标记整理的混合实现** 
​**Mark** 阶段的开销与存活对象的数量成正比
​**Sweep** 阶段的开销与所管理区域的大小成正相关
​**Compact** 阶段的开销与存活对象的数据成正比

以 **Hotspot** 中的 **CMS** 回收器为例，**CMS** 是基于 **Mark- Sweep** 实现的，对于对象的回收效率很高。对于碎片问题，**CMS** 采用基于 **Mark-Compact** 算法的 **Serial Old** 回收器作为补偿措施：当内存回收不佳，将采用 **Serial Old** 执行 **Full GC**以达到对老年代内存的整理

分代的思想被现有的虚拟机广泛使用。几乎所有的垃圾回收器都区分新生代和老年代



### 2.9 增量收集算法

上述现有的算法，在垃圾回收过程中，应用软件将处于一种 Stop the World 的状态。在 Stop the World 状态下，应用程序所有的线程都会挂起，暂停切正常的工作，等待垃圾回收的完成。**如果垃圾回收时间过长，应用程序会被挂起很久，将严重影响用户体验或者系统的稳定性**。为了解决这个问题，对实时垃圾收集算法的研究直接导致了增量收集（ Incremental Collecting）算法的诞生

> 算法思想

如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就**可以让垃圾收集线程和应用程序线程交替执行**。每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成

总的来说，增量收集算法的基础仍是传统的标记-清除和复制算法。增量收集算法通过**对线程间冲突的妥善处理，允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作** 



* 缺点

使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，**因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成==系统吞吐量的下降==** 



### 2.10 分区算法

一般来说，在相同条件下，堆空间越大，一次 GC 时所需要的时间就越长，有关 GC 产生的停顿也越长。**为了更好地控制 GC 产生的停顿时间，将一块大的内存区域分割成多个小块**，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次 GC 所产生的停顿

**分代算法**将按照对象的生命周期长短划分成两个部分，**分区算法将==整个堆空间==**划分成连续的不同小区间

每一个小区间都独立使用，独立回收。这种算法的**好处是可以控制一次回收多少个小区间** 



## 3、垃圾回收相关概念

### 3.1 System.gc() 的理解

在默认情况下，通过 System.gc()（或者 **Runtime.getRuntime().gc()**）的调用，会**显式触发 Full GC**，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存

然而 **System.gc()** 调用附带一个免责声明，**无法保证对垃圾收集器的调用** 

**JVM** 实现者可以通过 **System.gc()** 调用来决定 **JVM** 的 **GC** 行为。而一般情况下，垃圾回收应该是自动进行的，无须手动触发，否则就太过于麻烦了。在些特殊情况下，如我们正在编写一个性能基准，我们可以在运行之间调用 **System.gc()** 

```java
public class SystemGCTest {
public static void main(String[] args) {
new SystemGCTest();
System.gc();   // 底层 Runtime.getRuntime().gc();

// 强制调用失去引用对象的 finalize 方法
System.runFinalization();
}

@Override
protected void finalize() throws Throwable {
super.finalize();
System.out.println("SystemGCTest 重写了 finalize 方法");
}
}
```



### 3.2 内存溢出

内存溢出相对于内存泄漏来说，尽管更容易被理解，但是同样的，内存溢出也是引发程序崩溃的罪魁祸首之一

由于 GC 一直在发展，所有一般情况下，除非应用程序占用的内存增长速度非常快，造成垃圾回收已经跟不上内存消耗的速度，否则不太容易出现 OOM 的情况

大多数情况下，GC 会进行各种年龄段的垃圾回收，实在不行了就放大招来一次独占式的 **Full GC** 操作，这时候会回收大量的内存，供应用程序继续使用

**javadoc** 中对 **OutOfMemoryError** 的解释是，**没有空闲内存，并且垃圾收集器也无法提供更多内存** 



> **==没有空闲内存==**的情况：说明 Java 虛拟机的堆内存不够，原因有二

1、Java 虚拟机的堆内存设置不够

比如：可能存在内存泄漏问题，也很有可能就是堆的大小不合理，比如我们要处理比较可观的数据量，但是没有显式指定 **Java** 堆大小或者指定数值偏小。我们可以通过参数 **-Xms、-Xmx** 来调整

2、代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）

对于老版本的 **Oracle JDK**，因为永久代的大小是有限的，并且 **JVM** 对永久代垃圾回收（如，常量池回收、卸载不再需要的类型）非常不积极，所以当我们不断添加新类型的时候，永久代出现 **OutOfMemoryError** 也非常多见，尤其是在运行时存在大量动态类型生成的场合；**类似 intern 字符串缓存占用太多空间，也会导致 OOM 问题**。对应的异常信息，标记出来和永久代相关：**"java.lang.OutOfMemoryError ：PermGen space"**

随着元数据区的引入，方法区内存已经不再那么窘迫，所以相应的 OOM 有所改观，出现  OOM，异常信息则变成了 **"java.lang.OutOfMemoryErro：:Metaspace"**。直接内存不足，也会导致 OOM



> 垃圾收集器无法提供更多的内存空间

在抛出 **OutOfMemoryError** 之前，通常垃圾收集器会被触发，尽其所能的释放出空间

当然，也不是在任何情况下垃圾收集器都会被触发的，比如，我们去分配一个超大对象，类似一个超大数组超过堆的最大值，**JVM** 可以判断出垃圾收集并不能解决这个问题，所以直接抛出 **OutOfMemoryError** 



### 3.3 内存泄漏

严格来说，**==只有对象不会再被程序用到了，但是 GC 又不能回收他们的情况，才叫内存泄漏==** 

但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致 OOM，也可以叫做宽泛意义上的 "内存泄漏"

尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现 **OutOfMemoryError** 异常导致程序崩溃

> 注意，这里的存储空间并不是指物理内存，而是指虚拟内存大小，这个虚拟内存大小取决于磁盘交换区设定的大小



举例

1、单例模式：单例的生命周期和应用程序是一样长的，所以单例程序中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生

2、一些提供 **close** 的资源未关闭导致内存泄漏，数据库连接（dataSource.getConnection()，网络连接（ socket）和 io 连接**必须手动 close，否则是不能被回收的** 

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200708162835.png)



### 3.4 Stop the World

**Stop the World**，简称 **STW**，指的是 **GC** 事件发生过程中，会产生应用程序的停顿。停顿产生时整个应用程序线程都会被暂停，没有任何响应有点像卡死的感觉，这个停顿称为 **STW** 

可达性分析算法中枚举根节点（ GC Roots）会导致所有 Java 执行线程停顿

* 分析工作必须在一个能确保一致性的快照中进行
* 一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上
* 如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证



被 STW 中断的应用程序线程会在完成 GC 之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，所以我们需要减少 STW 的发生

STW 事件和采用哪款 GC 无关，所有的 GC 都有这个事件

STW 是 JVM 在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉

开发中不要用 **System.gc()，会导致 Stop the World 的发生** 

```java
public class StopTheWorldTest {
public static class WorkThread extends Thread {
List<byte[]> list = new ArrayList<>();

@Override
public void run() {
try {
while (true) {
for (int i = 0; i < 1000; i++) {
byte[] buffer = new byte[1024];
list.add(buffer);
}

if (list.size() > 10000) {
list.clear();
System.gc();
}
}
} catch (Exception e) {
e.printStackTrace();
}
}
}

public static class PrintThread extends Thread {
public final long startTime = System.currentTimeMillis();

@Override
public void run() {
try {
while (true) {
long time = System.currentTimeMillis() - startTime;
System.out.println(time / 1000 + "." + time % 1000);
Thread.sleep(1000);
}
} catch (Exception e) {
e.printStackTrace();
}
}
}

public static void main(String[] args) {
WorkThread workThread = new WorkThread();
PrintThread printThread = new PrintThread();

workThread.start();
printThread.start();
}
}
```



### 3.5 垃圾回收的并行与并发

并发和并行，在谈论垃**圾收集器的上下文语境中**，它们可以解释如下：

并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态

串行（ Serial）：相较于并行的概念，单线程执行，如果内存不够，则程序暂停，启动 JVM 垃圾回收器进行垃圾回收。回收完成，再启动程序线程

并发 （Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），垃圾回收线程在执行时不会停顿用户程序的运行。用户程序在继续运行，而垃圾收集程序线程运行于另一个CPU上；如：CMS、G1



### 3.6 安全点和安全区域

> 安全点

程序执行时并非在所有地方都能停顿下来开始 GC，只有在特定的位置才能停顿下来开始 GC，**这些位置称为"安全点（ Safe point）"** 

**Safe point 的选择很重要，如果太少可能导致 GC 等待的时间太长，如果太频繁可能导致运行时的性能问题**。大部分指令的执行时间都非常短暂，**==通常会根据 "是否具有让程序长时间执行的特征" 为标准==**。比如：选择些执行时间较长的指令作为 Safe Point，如方法调用、循环跳转和异常跳转等

> 如何在 GC 发生时，检査所有线程都跑到最近的安全点停顿下来呢？

**抢先式中断**（目前没有虚拟机采用了）
首先中断所有线程。如果还有线程不在安全点，就恢复线程，让线程跑到安全点

**主动式中断**
设置一个中断标志，各个线程运行到 Safe point 的时候主动轮询这个标志，如果中断标志为真，则将自己进行中断挂起



> 安全区域（ Safe Region）

Safe point 机制保证了程序执行时，在不太长的时间内就会遇到可进入的 Safe point。但是，程序 “不执行” 的时候呢？例如**线程处于 Sleep 状态或 Blocked 状态，这时候线程无法响应 JVM 的中断请求**，“走” 到安全点去中断挂起，JVM 也不太可能等待线程被唤醒。**对于这种情况，就需要==安全区域（ Safe Region）==来解决** 

**安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始 GC 都是安全的**。我们也可以把 Safe Region 看做是被扩展了的 Safe point



> 实际执行

1、当线程运行到 Safe Region 的代码时，首先标识已经进入了 Safe Region，如果这段时间内发生 GC，JVM 会忽略标识为 Safe Region 状态的线程

2、当线程即将离开 Safe Region 时，会检查 JVM 是否已经完成 GC，如果完成了，则继续运行，否则线程必须等待直到收到可以安全离开 Safe Region 的信号为止



### 3.7 再谈引用

我们希望能描述这样一类对象：当内存空间还足够时，则能保留在内存中；如果内存空间在进行垃圾收集后还是很紧张，则可以抛弃这些对象

> 【既偏门又非常高频的面试题】强引用、软引用、弱引用、虚引用有什么区别？具体使用场景是什么？

在 **JDK1.2** 之后，Java 对引用的概念进行了扩充，将引用分为**强引用**（Strong Reference）、**软引用**（ Soft Reference）、**弱引用**（ Weak Reference）、**虚引用**（ Phantom reference）4种，**这 4 种引用强度依次逐渐减弱** 

除强引用外，其他 3 种引用均可以在 **java.lang.ref** 包中找到它们的身影。如下图，显示了这3种引用类型对应的类，开发人员可以在应用程序中直接使用它们

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200709102138.png)

> 强引用（ StrongReference）

最传统的 “引用” 的定义，是指在程序代码之中普遍存在的引用赋值，即类似 “Object obj= new Object（）” 这种引用关系。**无论任何情况下只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象** 

> 软引用（ SoftReference）

**在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收**。如果这次回收后还没有足够的内存，才会抛出内存溢出异常

> 弱引用（ WeakReference）

**被弱引用关联的对象只能生存到下一次垃圾收集之前**。当下一次垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象

> 虚引用（ PhantomReference）

一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。**为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知** 



#### 3.7.1 强引用

在 Java 程序中，最常见的引用类型是强引用（普通系统99%以上都是强引用），也就是我们最常见的普通对象引用，也是默认的引用类型

当在 Java 语言中使用 new 操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用

只要强引用的对象是可触及的，垃圾收集器就永远不会回收掉被引用的对象

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略

相对的，**软引用、弱引用和虚引用的对象是软可触及、弱可触及和虚可触及的**，在定条件下，都是可以被回收的。所以，**强引用是造成 Java 内存泄漏的主要原因之一** 



#### 3.7.2 软引用

**软引用是用来描述一些==还有用，但非必需==的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收**，如果这次回收还没有足够的内存，才会抛出内存溢出异常

**软引用通常用来实现内存敏感的==缓存==**。比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存

垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列

类似弱引用，只不过 Java 虚拟机会尽量让软引用的存活时间长一些，迫不得已才清理



在 JDK1.2 版之后提供了 **java.lang.ref.SoftReference** 类来实现软引用

```java
Object object = new Object();
SoftReference<Object> softReference = new SoftReference<>(object);
object = null;  // 销毁强引用
```

```java
public class SoftRefTest {

public static class User {
private int id;
private String name;

public User(int id, String name) {
this.id = id;
this.name = name;
}

@Override
public String toString() {
return "Usesr{" +
"id=" + id +
", name='" + name + '\'' +
'}';
}
}

public static void main(String[] args) {
// -Xms10m -Xmx10m
SoftReference<User> softRef = new SoftReference<>(new User(1, "徐凤年"));

System.gc();
System.out.println("After GC " + softRef.get());

try {
byte[] bytes = new byte[1024 * 1024 * 20]; // 制造资源紧张
} catch (Throwable e) {
e.printStackTrace();
} finally {
System.out.println(softRef.get()); // 若资源紧张时会回收软引用，则此处为 null
}
}
}
```

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200709105755.png)



#### 3.7.3 弱引用

弱引用也是用来描述那些非必需对象，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。**在系统 GC 时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象** 

但是，**由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象**。在这种情况下，弱引用对象可以存在较长的时间

弱引用和软引用一样，**在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收** 

**软引用、弱引用都非常适合来保存那些可有可无的缓存数据**。如果这么做，系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用



```java
public static class User {
private int id;
private String name;

public User(int id, String name) {
this.id = id;
this.name = name;
}

@Override
public String toString() {
return "Usesr{" +
"id=" + id +
", name='" + name + '\'' +
'}';
}
}

public static void main(String[] args) {
WeakReference<User> weakRef=new WeakReference<>(new User(1,"徐凤年"));

System.out.println(weakRef.get());
System.gc();
System.out.println("After GC");
System.out.println(weakRef.get());
}
```

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200709110737.png)



#### 3.7.4 虚引用

也称为 “幽灵引用” 或者 “幻影引用”，是所有引用类型中最弱的一个

一个对象是否有虚引用的存在，完全不会决定对象的生命周期。**如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回** 

它不能单独使用，也无法通过虚引用来获取被引用的对象。**当试图通过虚引用的 get() 方法取得对象时，总是 null ** 

**==为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程==**。比如：能在这个对象被收集器回收时收到一个系统通知

**虚引用必须和引用队列一起使用。虚引用在创建时必须提供一个==引用队列==作为参数**。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况



```java
public class PhantomRefTest {
public static PhantomRefTest obj;
static ReferenceQueue<PhantomRefTest> phantomQueue = null;

public static class CheckQueue extends Thread {
@Override
public void run() {
while (true) {
if (phantomQueue != null) {
PhantomReference<PhantomRefTest> objt = null;
try {
objt = (PhantomReference<PhantomRefTest>) phantomQueue.remove();
} catch (Exception e) {
e.printStackTrace();
}
if (objt != null) {
System.out.println("追踪垃圾回收过程：PhantomRefTest 被GC了");
}
}
}
}
}

@Override
protected void finalize() throws Throwable {
System.out.println("调用当前类的 finalize 方法");
obj = this;
}

public static void main(String[] args) {
Thread t = new CheckQueue();
// 设置为守护线程，当程序中没有非守护线程时，守护线程也将结束
t.setDaemon(true);
t.start();

phantomQueue = new ReferenceQueue<>();
obj = new PhantomRefTest();

PhantomReference<PhantomRefTest> phantomRef = new PhantomReference<>(obj, phantomQueue);

try {
// 不可获得虚引用中的对象
System.out.println(phantomRef.get());

// 去除强引用
obj = null;
// 第一次 GC，由于对象可复活，此次GC无法回收
System.out.println("第一次 GC");
System.gc();
Thread.sleep(1000);

if (obj == null) {
System.out.println("obj is null");
} else {
System.out.println("obj is alive");
}

// 第二次 GC
System.out.println("第二次 GC");
obj = null;
System.gc();

Thread.sleep(1000);

if (obj == null) {
System.out.println("obj is null");
} else {
System.out.println(" obj is alive");
}

} catch (Exception e) {
e.printStackTrace();
}
}
}
```

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200709113228.png)



## 4、垃圾回收器



### 4.1 垃圾回收器的性能指标

**吞吐量**：**运行用户代码的时间**占**总运行时间**（程序的运行时间+内存回收的时间）的比例

垃圾收集开销：吞吐量的补数，**圾收集所用时间**与**总运行时间**的比例

**暂停时间**：执行垃圾收集时，程序的工作线程被暂停的时间

收集频率：相对于应用程序的执行，收集操作发生的频率

**内存占用**：Java 堆区所占的内存大小

快速：一个对象从诞生到被回收所经历的时间



在设计（或使用）GC 算法时，我们必须确定我们的目标：一个GC算法只可能针对两个目标之一（即只专注于较大吞吐量或最小暂停时间），或尝试找到一个二者的折衷



现在标准：**==在最大吞吐量优先的情况下，降低停顿时间==** 



### 4.2 不同的垃圾回收器

有了虚拟机，就一定需要收集垃圾的机制，这就是 Garbage Collection（垃圾收集算法），对应的产品我们称为 Garbage Collector（垃圾收集器）



1999 年随 JDK 1.3.1 一起发布的是串行方式的 Serial GC，它是第一款 GC。ParNew 垃圾收集器是 Serial 收集器的多线程版本

2002 年 2 月 26 日，Parallel GC 和 CMS（Concurrent Mark Sweep）GC 跟随 JDK 1.4.2 一起发布

Parallel GC 在 JDK6 之后成为 HotSpot 默认GC

2012年，在 JDK1.7u4 版本中，G1 可用

2017年，JDK9 中 G1 变成默认的垃圾收集器，以替代 CMS

2018年3月，JDK10 中 G1 垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟

2018年9月，JDK11发布。引入 Epsilon 垃圾回收器，又被称为 No-Op（无操作）回收器。同时，引入 ZGC：可伸缩的低延迟垃圾回收器（ Experimental）

2019年3月，JDK12 发布。增强 G1，自动返回未用堆内存给操作系统。同时，引入 Shenandoah GC：低停顿时间的 GC（ Experimental）

2019年9月，JDK13 发布。增强 ZGC，自动返回未用堆内存给操作系统

2020年3月，JDK14 发布。删除 CMS 垃圾回收器。扩展 ZGC 在 macOS 和 Windows 上的应用



> 经典的垃圾回收器

串行：**Serial、 Serial old** 

并行：**ParNew、 Parallel Scavenge、Parallel old** 

并发：**CMS、G1** 

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200709154929.png)



> 垃圾回收器的组合

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200709155040.png)

黑色实线是 JDK 8 以前的组合

其中 Serial Old 作为 CMS 出现 "Concurrent Mode Failure" 失败的后备预案

红色虚线连接的由于维护和兼容性测试的成本，在 JDK 8 时将 **==Serial + CMS==** 和 **==ParNew + Serial Old==** 这两个组合声明为废弃，并在 JDK9 中完全取消了些组合的支持

绿色虚线 连接的在 JDK14 中，**弃用 ==Parallel Scavenge + Serial Old== Gc组合** 

青色虚线所表示的在 JDK14 中已经删除了 CMS 垃圾回收器



> 查看默认的垃圾回收器

1、**-XX:+PrintCommandLineFlags**：查看命令行相关参数（包含使用的垃圾收集器）

2、使用命令行指令：**jnfo  -flag   相关垃圾回收器参数   进程 ID** 



### 4.3 串行：Serial GC

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710092346.png)



Serial 收集器是最基本、历史最悠久的垃圾收集器了，**JDK1.3 之前回收==新生代==唯一的选择** 

Serial 收集器作为 Hotspot 中 **==Client 模式==**下的默认新生代垃圾收集器

Serial 收集器采用**==复制算法==**、串行回收和 "Stop-the-World" 机制的方式执行内存回收

除了年轻代之外， **Serial 收集器还提供用于执行==老年代==垃圾收集的 Serial Old 收集器**， Serial Old 收集器同样也采用了串行回收和 "Stop the World" 机制，只不过**内存回收算法使用的是==标记-压缩算法==** 

**Serial Old 是运行在 ==Client 模式==下默认的老年代的垃圾回收器** 



Serial Old 在 **==Server 模式下==**主要有两个用途：

1. 与新生代的 Parallel  Scavenge 配合使用 
2. 作为老年代 CMS 收集器失败时的后备垃圾收集方案



* 优势

简单而高效（与其他收集器的单线程比），对于限定单个 CPU 的环境来说， Serial 收集器由于**没有线程交互的开销**，专心做垃圾收集自然可以获得最高的单线程收集效率，**运行在 ==Client 模式==下的虚拟机是个不错的选择** 

在用户的桌面应用场景中，可用内存一般不大（几十MB至一两百MB），可以在较短时间内完成垃圾收集（几十ms至一百多ms），只要不频繁发生，使用串行回收器是可以接受的



在 Hotspot 虚拟机中，使用 **-XX:+UseSerialGC** 参数可以指定年轻代和老年代都使用串行收集器，**等价于新生代用 ==Serial GC==，且老年代用 ==Serial Old GC==** 



* 小结

这种垃圾收集器了解即可，现在已经不用串行的了。而且在限定单核 CPU 才可以用，现在都不是单核的了

对于交互较强的应用而言，这种垃圾收集器是不能接受的。一般在 Java web 应用程序中是不会采用串行垃圾收集器的



### 4.4 并行：ParNew GC

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710093938.png)



**Par 是 Parallel 的缩写，New 表示只能处理新生代**。如果说 **Serial GC** 是年轻代中的**单线程**垃圾收集器，那么 **ParNew** 收集器则是 **Serial** 收集器的**多线程并行版本** 

**ParNew** 收集器除了采用并行回收的方式执行内存回收外，与 **Serial** 之间几乎没有任何区别。 **ParNew 收集器在年轻代中同样也是采用==复制算法==、"Stop-the-World" 机制** 

ParNew 是很多 JVM 运行在 **==Server 模式下新生代==**的默认垃圾收集器



在程序中，开发人员可以通过选项 **-XX:+UseParNewGC 手动指定使用 ParNew 收集器执行内存回收任务**。它表示年轻代使用并行收集器，不影响老年代

**-XX:ParallelGCThreads 限制线程数量**，默认开启和 CPU 数据相同的线程数



### 4.5 吞吐量优先：Parallel Scavenge GC

> 也可以直接就叫做 Parallel GC

HotSpot 的年轻代中除了拥有 ParNew 收集器是基于并行回收的以外， Parallel  Scavenge 收集器同样也釆用了**复制算法、==并行回收==和 "stop the world" 机制** 

和 ParNew 收集器不同，**Parallel Scavenge 收集器的目标则是达到一个==可控制的吞吐量==**（ Throughput），它也被称为吞吐量优先的垃圾收集器

**==自适应调节策略==**也是 Parallel  Scavenge 与 ParNew 一个重要区别



高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，**主要适合在后台运算而==不需要太多交互==的任务。因此，常见在服务器环境中使用**。例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序



**Parallel 收集器在 JDK 1.6 时提供了用于执行老年代垃圾收集的 ==Parallel Old== 收集器**，用来代替老年代的 Serial Old 收集器，**Parallel Old 收集器采用了==标记-压缩算法==，但同样也是==基于并行==回收和 "Stop-the-World" 机制** 



在程序吞吐量优先的应用场景中，Parallel 收集器和 Parallel Old 收集器的组合，在 Server 模式下的内存回收性能很不错，在 Java8 中，默认是此垃圾收集器



### 4.6 低延时：CMS GC

**==JDK 9 及以后该收集器不建议使用，JDK 14 时被删除了==** 

> 老年代垃圾回收器

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710104933.png)



在 JDK 1.5 时期， Hotspot 推出了一款在**强交互应用**中几乎可认为有划时代意义的垃圾收集器：CMS（ Concurrent-Mark-Sweep）收集器，**这款收集器是 Hotspot 虚拟机中第一款真正意义上的==并发==收集器，它第一次实现了让==垃圾收集线程与用户线程同时工作==** 

CMS 收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间。停顿时间越短（低延迟）就越适合与用户交互的程序，良好的响应速度能提升用户体验

**CMS 的垃圾收集算法采用==标记-清除算法==，并且也会 "Stop-the- World"** 



**不幸的是，CMS 作为老年代的收集器，却无法与 JDK 1.4.0 中已经存在的新生代收集器 Parallel  Scavenge 配合工作**，所以在 JDK 1.5 中使用 CMS 来收集老年代的时候，新生代只能选择 ParNew 或者 Serial 收集器中的一个



**CMS** 整个过程比之前的收集器要复杂，整个过程分为 4 个主要阶段，**初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段** 

> 初始标记阶段（ Initial-Mark），**==该阶段具有 STW 机制==** 

在这个阶段中，程序中所有的工作线程都将会因为 "Stop-the-World" 机制而出现短暂的暂停，**这个阶段的主要任务仅仅只是标 GC Roots 能直接关联到的对象。一旦标记完成之后就会恢复之前被暂停的所有应用线程**。由于直接关联对象比较小，所以这里的速度非常快

> 并发标记阶段（ Concurrent-Mark），**==并发执行==** 

**从 GC Roots 的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程**，可以与垃圾收集线程一起并发运行

> 重新标记阶段（ Remark），**==该阶段具有 STW 机制==** 

由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了**修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录**，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短

> 并发清除阶段（ Concurrent- Sweep），**==并发执行==** 

**此阶段清理删除掉标记阶段判断的已经死亡的对象，释放内存空间**。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的



* 分析

尽管 CMS 收集器采用的是并发回收（非独占式），但是在其**初始化标记和再次标记这两个阶段中仍然需要执行 "Stop-the-Wor1d" 机制**暂停程序中的工作线程，不过暂停时间并不会太长，因此可以说明目前所有的垃圾收集器都做不到完全不需要 "Stop the World"

**由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的** 

另外，由于在垃圾收集阶段用户线程没有中断，所以在 CMS 回收过程中，还应该确保应用程序用户线程有足够的内存可用。因此，CMS 收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是当**堆内存使用率达到某一阈值时，便开始进行回收，以确保应用程序在 CMS 工作过程中依然有足够的空间支持应用程序运行**。要是 CMS 运行期间预留的内存无法满足程序需要，就会出现一次 “Concurrent Mode Failure” 失败，这时虚拟机将启动后备预案：临时启用 Serial Old 收集器来重新进行老年代垃圾收集，这样停顿时间就很长了

**CMS 收集器的垃圾收集算法采用的是标记清除算法**，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，**不可避免地将会产生一些内存碎片**。那么 CMS 在为新对象分配内存空间时，将无法使用指针碰撞（ Bump the Pointer）技术，而只能够选择空闲列表（ Free list）执行内存分配



> 为什么不使用标记压缩算法呢？

**因为当并发清除的时候，用 Compact 整理内存的话，原来的用户线程使用的内存就用不了了**，要保证用户线程能继续执行，前提的它运行的资源不受影响。Mark Compact 更适合 "Stop the world" 这种场景下使用



* 弊端

1、会产生内存碎片，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发  Full GC

2、CMS 收集器对 CPU 资源非常敏感。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低

3、CMS 收集器无法处理浮动垃圾。可能出现 “Concurrent mode failure" 失败而导致另一次 Full GC 的产生。在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，那么在并发标记阶段如果产生新的垃圾对象，CMS 将无法对这些垃圾对象进行标记，最终会导致这些新产生的垃圾对象没有被及时回收，从而只能在下一次执行 GC 时释放这些之前未被回收的内存空间



### 4.7 G1(Garbage First) GC

> 既然我们已经有了前面几个强大的 GC，为什么还要发布 Garbage First（G1）GC？

原因就在于应用程序所应对的业务越来越庞大、复杂，用户越来越多，没有 GC 就不能保证应用程序正常进行，而经常造成 STW 的 GC 又跟不上实际的需求，所以才会不断地尝试对 GC 进行优化。**G1（ Garbage-First）垃圾回收器是在Java7 update4 之后引入的一个新的垃圾回收器，==是当今收集器技术发展的最前沿成果之一==** 

与此同时，为了适应现在不断扩大的内存和不断增加的处理器数量，进一步降低暂停时间（ Pause Time），同时兼顾良好的吞吐量

**官方给 G1 设定的目标是==在延迟可控的情况下获得尽可能高的吞吐量==，所以才担当起 "全功能收集器" 的重任与期望** 



> 为什么名字叫做 Garbage First（G1）呢？

因为 G1 是一个并行回收器，**它把堆内存分割为很多不相关的区域（ Region）（物理上不连续的）**。使用不同的 Region 来表示 Eden、幸存者0区，幸存者1区，老年代等

G1 GC 有计划地避兔在整个 Java 堆中进行全区域的垃圾收集。**G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），==在后台维护一个优先列表==，每次根据允许的收集时间，优先回收价值最大的 Region** 

由于这种方式的侧重点在于回收垃圾最大量的区间（ Region），所以我们给 G1一个名字：垃圾优先（ Garbage First）



> 适用场景

G1 是一款面向服务端应用的垃圾收集器，**主要针对配备多核 CPU 及大容量内存的机器**，以极高概率满足 GC 停顿时间的同时，还兼具高吞吐量的性能特征 

在 JDK1.7 版本正式启用，移除了 Experimental 的标识，**是 JDK9 以后的默认垃圾回收器**，取代了CMS 回收器以及Parallel + Parallel Old 组合，oracle 官方称为 "全功能的垃圾收集器"



* 优势

> 并行与并发

并行性：G1 在回收期间，可以有多个 GC 线程同时工作，有效利用多核计算能力。此时用户线程 STW

并发性：G1 拥有与应用程序交替执行的能力，**部分工作可以和应用程序同时执行**，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况

> 分代收集

从分代上看，G1 依然属于分代型垃圾回收器，它会区分年轻代和老年代，年轻代依然有 Eden 区和 Survivor 区。但从堆的结构上看，**它不要求整个 Eden 区、年轻代、老年代都是连续的，也不再坚持固定大小和固定数量** 

将堆空间分为若干个区域（ Region），**这些区域中包含了==逻辑上==的年轻代和老年代** 

和之前的各类回收器不同，**它同时兼顾年轻代和老年代**。对比其他回收器，或者工作在年轻代，或者工作在老年代

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710115521.png)

> 空间整合

G1 将内存划分为一个个的 Region。内存的回收是以 region 作为基本单位的。**Region 之间是==复制算法==，但整体上实际可看作是==标记-压缩（Mark- Compact）算法==**，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次 GC。尤其是当 Java 堆非常大的时候，G1 的优势更加明显

> 可预测停顿时间模型

这是 G1 相对于 CMS 的另一大优势，G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒

由于分区的原因，G1 可以只选取部分区域进行内存回收，这样缩小了回收的范围，因此对于全局停顿情况的发生也能得到较好的控制

**G1 跟踪各个 Region 里面的==垃圾堆积的价值大小==（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region**。保证了 G1 收集器在有限的时间内可以获取尽可能高的收集效率

相比于 CMS GC，G1 未必能做到 CMS 在最好情况下的延时停顿，但是最差情况要好很多



* 缺点

相较于 CMS，G1 还不具备全方位、压倒性优势。比如在用户程序运行过程中， G1 无论是为了垃圾收集产生的**内存占用**还是程序运行时的**额外执行负载**都要比 CMS 要高

从经验上来说，在小内存应用上 CMS 的表现大概率会优于 G1，而 G1 在大内存应用上则发挥其优势。平衡点在6-8GB 之间



> **Region** 介绍：化整为零

使用 G1 收集器时，它将整个 Java 堆划分成约 2048 个大小相同的独立 Region 块，每个 Region 块大小根据堆空间的实际大小而定，整体被控制在 1MB 到 32MB 之间，且为 2 的 N 次幂，即 1MB，2MB，4MB，8MB，16MB，32MB。可以通过 -XX:G1HeapRegionS1ze 设定。**所有的 Region 大小相同，且在 JVM 生命周期内不会被改变** 

虽然还保留有新生代和老年代的概念，但**新生代和老年代不再是物理隔离的了，它们都是一部分 Region（不需要连续）的集合**。通过 Region 的动态分配方式实现逻辑上的连续

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710145524.png)

一个 Region 有可能属于 Eden, Survivor 或者 old 内存区域。但是一个 Region 只可能属于一个角色。图中的 E 表示该 Region 属于 Eden 内存区域，S 表示属于 Survivor 内存区域，o 表示属于 old 内存区域。图中空白的表示未使用的内存空间

G1 垃圾收集器还增加了一种新的内存区域，叫做 Humongous 内存区域，如图中的 H 块。**主要用于存储大对象，如果超过 0.5 个 Region，就放到 H** 

> 设置 H 块的原因

**对于堆中的大对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象，就会对垃圾收集器造成负面影响**。为了解决这个问题，G1 划分了一个 Humongous 区它用来专门存放大对象。如果一个 H 区装不下一个大对象，那么 G1 会寻找连续的 H 区来存储。**为了能找到连续的 H 区，有时候不得不启动 Full GC**，G1 的大多数行为都把 H 区作为老年代的一部分来看待



> 记忆集：Remembered  Set

记忆集是一种用于记录**从==非收集区域==指向==收集区域==的指针集合**的抽象数据结构

* 问题：一个对象被不同区域所引用

一个 Region 不可能是孤立的，一个 Region 中的对象可能被其他任意 Region 中对象所引用，要判断对象存活时，是否需要扫描整个 Java 堆才能保证准确？

在其他的分代收集器，也存在这样的问题（而G1更突出），回收新生代也不得不同时扫描老年代？这样的话会降低Minor GC 的效率

* 解决办法
    * **无论 G1 还是其他分代收集器，JVM 都是使用 Remembered Set 来避免全局扫描** 

**==每个 Region 都有一个对应的 Remembered Set==** 

每次 Reference 类型数据写操作时，都会产生一个 Write Barrier 暂时中断操作，然后**检查将要写入的引用指向的对象是否和该 Reference 类型数据在不同的 Region（其他收集器：检查老年代对象是否引用了新生代对象）**，如果不同，通过 CardTable 把相关引用信息记录到引用指向对象的所在 Region 对应的 Remembered Set 中，这样，当进行垃圾收集时，在 GC 根节点的枚举范围加入 Remembered Set，就可以保证不进行全局扫描，也不会有遗漏

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710171751.png)



> G1 垃圾回收过程

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710151040.png)

当年轻代的 Eden 区用尽时开始年轻代回收过程；**G1 的年轻代收集阶段是一个并行的独占式收集器。在年轻代回收期，G1 GC 暂停所有应用程序线程，启动多线程执行年轻代回收**。然后**从年轻代区间移动存活对象到 Survivor 区间或者老年区间，也有可能是两个区间都会涉及** 

当堆内存使用达到一定值（默认45%）时，开始老年代并发标记过程

标记完成马上开始**混合回收**过程。对于一个混合回收期，G1 GC 从老年区间移动存活对象到空闲区间，这些空闲区间也就成为了老年代的一部分。和年轻代不同，老年代的 G1 回收器和其他 GC 不同，**G1 的老年代回收器不需要整个老年代被回收，一次只需要扫描/回收小部分老年代的 Region 就可以了**。同时，这个老年代 Region 是和年轻代一起被回收的

举个例子：一个 Web 服务器，Java 进程最大堆内存为 4G，每分钟响应 1500 个请求，每 45 秒钟会新分配大约 2G的内存。G1 会每 45 秒钟进行一次年轻代回收，每 31 个小时整个堆的使用率会达到 45%，会开始老年代并发标记过程，标记完成后开始四到五次的混合回收



* 年轻代 GC（Young GC）

JVM 启动时，G1 先准备好 Eden 区，程序在运行过程中不断创建对象到 Eden 区，当 Eden 空间耗尽时，G1 会启动一次年轻代垃圾回收过程

**年轻代垃圾回收只会回收 Eden 区和 Survivor 区** 

YGC 时，首先 G1 停止应用程序的执行（Sop-The-Wor1d），G1 创建回收集（Collection set），**回收集是指需要被回收的内存分段的集合**，年轻代回收过程的回收集包含年轻代 Eden 区和 Survivor 区所有的内存分段

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710214811.png)



* 老年代并发标记过程（Concurrent  Marking）

1、初始标记阶段：标记从根节点直接可达的对象。这个阶段是 STW 的，并且会触发一次年轻代 GC

2、根区域扫描：G1 GC 扫描 Survivor 区直接可达的老年代区域对象，并标记被引用的对象。这一过程必须在 Young GC 之前完成

3、并发标记：在整个堆中进行并发标记（和应用程序并发执行），此过程可能被 young GC 中断。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那这个区域会被立即回收。同时，并发标记过程中，会计算每个区域的对象活性（区域中存活对象的比例）

4、再次标记：**由于应用程序持续进行，需要修正上一次的标记结果**。是 STW 的。G1 中采用了比 CMS 更快的初始快照算法：snapshot-at-the-beginning（SATB）

5、独占清理：计算各个区域的存活对象和 GC 回收比例，并进行排序识别可以混合回收的区域，为下阶段做铺垫，是 STW 的 

6、并发清理阶段：识别并清理完全空闲的区域



* 混合回收（Mixed  GC）

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200710220318.png)

当越来越多的对象晋升到老年代 Old Region 时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即 Mixed GC，该算法并不是一个 Old GC，除了回收**整个 Young Region，还会回收一部分的 Old Region**。这里需要注意：**是一部分老年代，而不是全部老年代**。可以选择哪些 Old Region 进行收集，从而可以对垃圾回收的耗时时间进行控制。也要注意的是 **Mixed GC 并不是 Full GC** 

**并发标记结束以后，老年代中百分百为垃圾的内存分段被回收了，部分为垃圾的内存分段被计算了出来**。默认情况下，这些老年代的内存分段会分 8 次（可以通过 -XX:G1MixedGCCountTarget 设置）被回收

混合回收的回收集包括八分之一的老年代内存分段，Eden 区内存分段， Survivor 区内存分段。**混合回收的算法和年轻代回收的算法完全一样，只是回收集多了老年代的内存分段**。具体过程请参考上面的年轻代回收过程

由于老年代中的内存分段默认分 8 次回收，G1 会优先回收垃圾多的内存分段。**垃圾占内存分段比例越高的，越会被先回收**。并且有一个阈值会决定内存分段是否被回收， **-XX:G1MixedGCLiveThresholdPercent**，默认为 65%，意思是垃圾占内存分段比例要达到 65% 才会被回收。如果垃圾占比太低，意味着存活的对象占比高，在复制的时候会花费更多的时间

混合回收并不一定要进行 8 次。有一个阈值 -XX:G1HeapWastePercent，默认值为10%，**意思是允许整个堆内存中有10% 的空间被浪费，意味着如果发现可以回收的垃圾占堆内存的比例低于10%，则不再进行混合回收。因为 GC 会花费很多的时间但是回收到的内存却很少** 



* 如果需要，单线程、独占式、高强度的 Full GC 还是继续存在的。它针对 GC 的评估失败提供了一种失败保护机制，即强力回收

G1 的初衷就是要避免 Full GC 的出现。但是如果上述方式不能正常工作，G1 会停止应用程序的执行（stop-The- World），使用单线程的内存回收算法进行垃圾回收，性能会非常差，应用程序停顿时间会很长

要避免 Full GC 的发生，一旦发生需要进行调整。什么时候会发生 Full GC 呢？比如堆内存太小，当 G1 在复制存活对象的时候没有空的内存分段可用则会回退到 Full GC，这种情况可以通过增大内存解决



### 4.8 七款垃圾回收器小结

|              | 分类       | 作用位置     | 使用算法        | 特点         | 适用场景                         |
| ------------ | ---------- | ------------ | --------------- | ------------ | -------------------------------- |
| Serial       | 串行       | 新生代       | 复制            | 响应速度优先 | 适用于单CPU下的Client模式        |
| ParNew       | 并行       | 新生代       | 复制            | 响应速度优先 | 多CPU下Server模式，与CMS配合使用 |
| Parallel     | 并行       | 新生代       | 复制            | 吞吐量优先   | 适用于后台运算不需要过多交互场景 |
| Serial Old   | 串行       | 老年代       | 标记-压缩       | 响应速度优先 | 适用于单CPU下Client模式          |
| Parallel Old | 并行       | 老年代       | 标记-压缩       | 吞吐量优先   | 适用于后台运算不需要过多交互场景 |
| CMS          | 并发       | 老年代       | 标记-清除       | 响应速度优先 | 适用于互联网或 B/S 业务          |
| G1           | 并发、并行 | 新生、老年代 | 标记-压缩、复制 | 响应速度优先 | 面向服务端应用                   |



## 5、GC日志分析

参数：-XX:+PrintGC，该参数只会显示总的 GC 堆的变化

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200711104717.png)

解析：

> GC、Full GC：是 GC 的类型，GC 只在新生代上进行，Full GC 包括永生代，新生代，老年代
>
> Allocation Failure：GC发生的原因
>
> 15267K->14054K：堆在 GC 前的大小和 GC 后的大小
>
> 58880K：堆的总大小
>
> 0.2960286 secs：GC持续的时间



详细参数解析

![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200711110329.png)



![](C:/Users/tfc/Desktop/笔记/MarkDown/JVM/images/TIM截图20200711110406.png)



> 将日志保存到文件

-Xloggc:路径：-Xloggc:/path/tfc/a.log



# 九、HotSpot JVM 常用参数

## Serial GC

* 指定新生代和老年代使用 **Serial** 串行收集器，**等价于新生代用 ==Serial GC==，且老年代用 ==Serial Old GC==** 

>  -XX:+UseSerialGC 



## ParNew

* 指定新生代使用 **ParNew** 收集器，不影响老年代

> -XX:+UseParNewGC
>
> -XX:ParallelGCThreads：限制线程数量，默认开启和 CPU 数据相同的线程数
>
> **==JDK 9 及以后该收集器不建议使用==** 



## Parallel GC

* 指定新生代使用 Parallel GC，老年代就会默认使用 Parallel Old GC，反之亦然

> -XX:+UseParallelGC   -XX:UseParallelOldGC
>
> 这两个参数在 JDK 8 中默认开启，设置任何一个，另一个也随之开启
>
> -XX:ParallelGCThreads：设置新生代并行收集线程数，一般最好与 CPU 数量相等
>  在默认情况下，当 CPU 数量小于 8 个，**ParallelGCThreads** 的值等于 CPU 数量
>  当 CPU 数量大于 8 个，**ParallelGCThreads** 的值等于 **==3+( 5* CPU_Count ) / 8==** 



* 设置垃圾收集器最大停顿时间，单位是毫秒

对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器端适合Parallel，进行控制。**==该参数谨慎使用==** 

> -XX:MaxGCPauseMillis



* 垃圾收集时间占总时间的比例，用于衡量吞吐量的大小

取值范围（0,100），**默认值 99**，也就是垃圾回收时间不超过 1%

与 -XX:MaxGCPauseMillis 参数有一定矛盾性。暂停时间越长， Radio 参数就容易超过设定的比例

> -XX:GCTimeRadio



* 设置 Parallel Scavenger GC 的自适应调节策略

在这种模式下，年轻代的大小、Eden 和 Survivor 的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点

在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（ GCTimeRatio）和停顿时间（ MaxGCPauseMillis），让虚拟机自己完成调优工作

> -XX:UseAdaptiveSizePolicy



## CMS GC

* 手动指定老年代垃圾回收器使用 CMS
    * 开启该参数后会自动将 -XX:+UseParNewGC 打开( 新生代垃圾回收器 )

> -XX:+UseConcMarkSweepGC



* 设置堆内存使用率的阈值，旦达到该阈值，便开始进行回收

JDK 5 及以前版本的默认值为 68，即当老年代的空间使用率达到 68%时，会执行 CMS 回收。JDK6 及以上版本默认值为 92%

如果内存增长缓慢，则可以设置一个稍大的值，大的阈值可以有效降低 CMS 的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁触发老年代串行收集器。因此通过该选项便可以有效降低 Full GC 的执行次数

> -XX:CMSInitiatingOccupanyFraction



**用于指定在执行完 Full GC 后对内存空间进行压缩整理，以此避免内存碎片的产生**。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了

> -XX:+UseCMSCompactAtFullCollection



设置在执行多少次Fu11Gc后对内存空间进行压缩整理

> -XX:CMSFullGCsBeforeCompaction



设置 CMS 的线程数量。CMS 默认启动的线程数是（ParallelGcThreads+3）/ 4，**ParallelGCThreads 是年轻代并行收集器的线程数**。当 CPU 资源比较紧张时，受到CMS收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟

> -XX:ParallelCMSThreads



## G1 GC

* JDK 9之前手动指定使用 G1 收集器执行内存回收任务

> -XX:+UseG1GC



设置每个 Region 的大小。值是2的次幂，范围是 1MB 到 32MB 之间，目标是根据最小的Java 堆大小划分出约2048个区域。默认是堆内存的 1/2000

> -XX:G1HeapRegionSize



设置期望达到的最大 **GC** 停顿时间指标（ **JVM** 会尽力实现，但不保证达到）。默认值是 **200ms** 

> -XX:MaxGCPauseMillis



设置 STW 工作时 GC 线程数的值，最多设置为 8

> -XX:ParallelGCThread



设置**并发标记**的线程数。将 n 设置为**并行**垃圾回收线程数（ParallelGCThreads）的 1/4 左右

> -XX:ConcGCThreads



设置触发并发 GC 周期的 Java 堆占用率阈值。超过此值，就触发 GC，默认值是 45

> -XX:InitiatingHeapOccupancyPercent