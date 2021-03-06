# JVM

1、**运行时数据区中包含哪些区域？哪些线程共享？哪些线程独享？**【⭐⭐⭐⭐⭐】

堆、方法区、直接内存、虚拟机栈、本地方法栈、程序计数器

* 共享：堆、方法区、直接内存
* 独享：虚拟机栈、本地方法栈、程序计数器



2、**说一下方法区和永久代的关系**。【⭐⭐⭐】

《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。**方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 

也就是说，永久代是 HotSpot 的概念，**方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现** 



3、**讲一下 Java 创建一个对象的过程**。【⭐⭐⭐⭐】

> 1）类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中找到这个类的符号引用，并且检查这个类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

> 2）分配内存

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定

分配内存方式：**指针碰撞** 和 **空闲列表**。**选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

* **指针碰撞**：用过的放到一遍，没用过的内存放一边，中间由一个指针控制，分配内存时往空闲区域移动指针即可
    * **GC**：Serial、ParNew
* **空闲列表**：虚拟机会维护一个列表，列表中会记录哪些是可用的，分配内存时选择一块足够大的空间，之后更新列表
    * **GC**：CMS

> 3）初始化零值

内存分配完成后，虚拟机需要将对象属性都初始化为零值（不包括对象头），**这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值** 

> 4）设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 

> 5）执行 init() 方法

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。

执行 new 指令之后会执行 `<init>` 方法，**把对象按照程序员的意愿进行初始化**，这样一个真正可用的对象才算完全产生出来



4、**对象的访问定位的两种方式（句柄和直接指针两种方式）**。【⭐⭐⭐⭐⭐】

> **句柄** 

如果使用句柄的话，Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了**对象实例数据**与**类型数据**各自的具体地址信息；

> 直接指针

如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，**而 reference 中存储的直接就是对象的地址**。



5、**你了解分代理论吗？讲一下 Minor GC、还有 Full GC**。【⭐⭐⭐⭐⭐】

目前主流的垃圾收集器都会采用分代回收算法，因此需要**将堆内存分为新生代和老年代**，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

* **Minor GC**：只对新生代进行垃圾收集，新生代的对象经过一定次数的 GC 还没有被收集的就放到老年代

* **Full GC**：收集整个 Java 堆和方法区



6、**Java 用什么方法确定哪些对象该被清理？ 讲一下可达性分析算法的流程。**【⭐⭐⭐⭐】

> 1）引用计数法

给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加 1；当引用失效，计数器就减 1；任何时候计数器为 0 的对象就是不可能再被使用的。

**这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题**。

如下面代码所示：除了对象 objA 和 objB 相互引用着对方之外，这两个对象之间再无任何引用。但是他们因为互相引用对方，导致它们的引用计数器都不为 0，于是引用计数算法无法通知 GC 回收器回收他们。

```java
public class ReferenceCountingGc {
    Object instance = null;
	public static void main(String[] args) {
		ReferenceCountingGc objA = new ReferenceCountingGc();
		ReferenceCountingGc objB = new ReferenceCountingGc();
		objA.instance = objB;
		objB.instance = objA;
		objA = null;
		objB = null;
	}
}
```

> 2）可达性分析算法

这个算法的基本思想就是通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，**当 GC Roots 无法到达一个对象时，则证明此对象是不可用的**。

可作为 **GC Roots** 的对象包括下面几种：

- 虚拟机栈(栈帧中的本地变量表)中引用的对象
- 本地方法栈(Native 方法)中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 所有被同步锁持有的对象



7、**JDK 中有几种引用类型？分别的特点是什么？**【⭐⭐】

> **强引用** 

以前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，那就类似于**必不可少的生活用品**，垃圾回收器绝不会回收它。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

> **软引用** 

如果一个对象只具有软引用，那就类似于**可有可无的生活用品**。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。

只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA 虚拟机就会把这个软引用加入到与之关联的引用队列中。

> **弱引用** 

如果一个对象只具有弱引用，那就类似于**可有可无的生活用品**。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。

在垃圾回收器线程扫描它所管辖的内存区域的过程中，**一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存**。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

> **虚引用**：虚引用主要用来跟踪对象被垃圾回收的活动

顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。**如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收**。

**虚引用与软引用和弱引用的一个区别在于：** 虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。**程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收**。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。



8、**如何回收方法区？**【⭐⭐⭐】

方法区的垃圾收集主要回收两部分内容：**废弃的常量**和**不再使用的类型** 

> 收集废弃常量

假如在字符串常量池中存在字符串 "abc"，如果当前没有任何 String 对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池了。

> 收集不再使用的类型

判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面 3 个条件才能算是 **“无用的类”** ：

* 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。

* 加载该类的 `ClassLoader` 已经被回收。

* 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

虚拟机可以对满足上述 3 个条件的无用类进行回收，**这里说的仅仅是“可以”**，而并不是和对象一样不使用了就会必然被回收。



9、**标记清除、标记复制、标记整理分别是怎样清理垃圾的？各有什么优缺点？**【⭐⭐⭐⭐⭐】

> **标记-清除** 

该算法分为“标记”和“清除”阶段：首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象。它是最基础的收集算法，后续的算法都是对其不足进行改进得到。这种垃圾收集算法会带来两个明显的问题：

1. **效率问题** 
2. **空间问题（标记清除后会产生大量不连续的碎片）** 

> **标记-复制：解决效率问题** 

它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

> **标记-整理** 

根据老年代的特点提出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让**所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。** 



10、**JVM 中的安全点和安全区各代表什么？写屏障你了解吗？**【⭐⭐⭐⭐】

> 安全点

程序执行时并非在所有地方都能停顿下来开始 GC，只有在特定的位置才能停顿下来开始 GC，**这些位置称为"安全点（ Safe point）"** 

**Safe point 的选择很重要，如果太少可能导致 GC 等待的时间太长，如果太频繁可能导致运行时的性能问题**。大部分指令的执行时间都非常短暂，**==通常会根据 "是否具有让程序长时间执行的特征" 为标准==**。比如：选择些执行时间较长的指令作为 Safe Point，如方法调用、循环跳转和异常跳转等

> 安全区域（ Safe Region）

Safe point 机制保证了程序执行时，在不太长的时间内就会遇到可进入的 Safe point。但是，程序 “不执行” 的时候呢？例如**线程处于 Sleep 状态或 Blocked 状态，这时候线程无法响应 JVM 的中断请求**，“走” 到安全点去中断挂起，JVM 也不太可能等待线程被唤醒。**对于这种情况，就需要==安全区域（ Safe Region）==来解决** 

**安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始 GC 都是安全的**。我们也可以把 Safe Region 看做是被扩展了的 Safe point

> 写屏障

**对一个对象引用进行写操作（即引用赋值）之前或之后附加执行的逻辑**，相当于为引用赋值挂上的一小段钩子代码

“HotSpot通过写屏障来维护卡表”，写屏障就是在将引用赋值写入内存之前，先做一步mark card——即**将出现跨代引用的内存块对应的卡页置为dirty** 



11、**并发标记要解决什么问题？并发标记带来了什么问题？如何解决并发扫描时对象消失问题？**【⭐⭐⭐⭐】相关阅读：[面试官:你说你熟悉 jvm?那你讲一下并发的可达性分析](https://juejin.cn/post/6844904070788939790) 。

> 并发标记要解决什么问题？

初始标记阶段中，程序中所有的工作线程都将会因为 `"Stop-the-World"` 机制而出现短暂的暂停，**这个阶段的主要任务仅仅只是标 GC Roots 能直接关联到的对象。一旦标记完成之后就会恢复之前被暂停的所有应用线程**。由于直接关联对象比较小，所以这里的速度非常快

**从 GC Roots 的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程**，可以与垃圾收集线程一起并发运行，**并发标记要解决的就是要消减这一部分的停顿时间**。那就是让垃圾回收器和用户线程同时运行，并发工作。

> 并发标记带来了什么问题？

**并发标记除了会产生浮动垃圾，还会出现"对象消失"的问题**。

**浮动垃圾**：在并发标记阶段，由于用户线程与垃圾回收线程同时执行，用户线程可能中途更改了对象的引用关系，把原本消亡的对象错误的标记为存活，这不是好事，但是其实是可以容忍的，只是产生了一点逃过本次回收的浮动垃圾，下次清理就可以。

**对象消失**：在并发标记阶段，由于用户线程与垃圾回收线程同时执行，用户线程可能中途更改了对象的引用关系，导致存活对象被当成死亡对象被清除

> 如何解决并发扫描时对象消失问题？

**黑色对象**：存活对象

**灰色对象**：被垃圾收集器访问过，但至少还指向一个其他对象

**白色对象**：未被垃圾收集器访问过的对象

有一个大佬叫`Wilson`，他在1994年在理论上证明了，**当且仅当以下两个条件同时满足时**，会产生"对象消失"的问题，原来应该是黑色的对象被误标为了白色：

* 条件一：赋值器插入了一条或者多条从黑色对象到白色对象的新引用。

* 条件二：赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。

由于两个条件之间是当且仅当的关系。所以，我们要解决并发标记时对象消失的问题，只需要破坏两个条件中的任意一个就行。

于是产生了两种解决方案：**增量更新（Incremental Update）和原始快照（Snapshot At The Beginning，SATB）。**

在HotSpot虚拟机中，**CMS是基于增量更新来做并发标记的，G1则采用的是原始快照的方式**。

**增量更新**：增量更新要破坏的是第一个条件（赋值器插入了一条或者多条从黑色对象到白色对象的新引用），当黑色对象插入新的指向白色对象的引用关系时，就**将这个新插入的引用记录下来**，等并发扫描结束之后，再**将这些记录过的黑色对象为根，重新扫描一次**。

**原始快照**：原始快照要破坏的是第二个条件（赋值器删除了全部从灰色对象到该白色对象的直接或间接引用），当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再**将这些记录过的灰色对象为根，重新扫描一次**。

**简单来说就是将更新关系记录下来，重新再扫描一遍**。



12、**对于 JVM 的垃圾收集器你有什么了解的？**【⭐⭐⭐⭐】有时候面试官会问出这种十分开放性的问题，你需要脑子里过一下你对这个大问题下的哪些知识熟悉哪些不熟悉，不熟悉的点一下就过，熟悉的展开讲。在准备校招时，我的一个是阿里 P7 的学姐，给我做过一次模拟面试，问出这个问题时让我有点懵，那么多东西我不知道从哪开始回答呀，就答得很凌乱。模拟面试完我问她这种问题应该从哪开始回答？ 她说她因为不知道我的掌握情况，所以就先问一个大问题，根据我的回答再追问，以后遇到这种问题主要从自己熟悉得方面切入就可以了。后来的面试还真遇到过好几次这种情况，我就答，垃圾收集器的种类有以下几种 Serial，ParNew...现在用的多的还是 CMS 和 G1，CMS 的垃圾收集流程是 xxx，G1 的垃圾收集流程是 xxx，他们特点是...就这样把话题引到 CMS 和 G1 了，只 CMS 和 G1 这部分和面试官讨论十几分钟完全没问题。



13、**新生代垃圾收集器有哪些？老年代垃圾收集器有哪些？哪些是单线程垃圾收集器，哪些是多线程垃圾收集器？各有什么特点？各基于哪一种垃圾收集算法？**【⭐⭐⭐⭐】

|              | 分类       | 作用位置     | 使用算法        | 特点         | 适用场景                         |
| ------------ | ---------- | ------------ | --------------- | ------------ | -------------------------------- |
| Serial       | 串行       | 新生代       | 复制            | 响应速度优先 | 适用于单CPU下的Client模式        |
| ParNew       | 并行       | 新生代       | 复制            | 响应速度优先 | 多CPU下Server模式，与CMS配合使用 |
| Parallel     | 并行       | 新生代       | 复制            | 吞吐量优先   | 适用于后台运算不需要过多交互场景 |
| Serial Old   | 串行       | 老年代       | 标记-压缩       | 响应速度优先 | 适用于单CPU下Client模式          |
| Parallel Old | 并行       | 老年代       | 标记-压缩       | 吞吐量优先   | 适用于后台运算不需要过多交互场景 |
| CMS          | 并发       | 老年代       | 标记-清除       | 响应速度优先 | 适用于互联网或 B/S 业务          |
| G1           | 并发、并行 | 新生、老年代 | 标记-压缩、复制 | 响应速度优先 | 面向服务端应用                   |



14、**讲一下 CMS 垃圾收集器的四个步骤。CMS 有什么缺点？**【⭐⭐⭐⭐】

> 初始标记阶段（ Initial-Mark），**==该阶段具有 STW 机制==** 

在这个阶段中，程序中所有的工作线程都将会因为 "Stop-the-World" 机制而出现短暂的暂停，**这个阶段的主要任务仅仅只是标 GC Roots 能直接关联到的对象。一旦标记完成之后就会恢复之前被暂停的所有应用线程**。由于直接关联对象比较小，所以这里的速度非常快

> 并发标记阶段（ Concurrent-Mark），**==并发执行==** 

**从 GC Roots 的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程**，可以与垃圾收集线程一起并发运行

> 重新标记阶段（ Remark），**==该阶段具有 STW 机制==** 

由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了**修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录**，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短

> 并发清除阶段（ Concurrent- Sweep），**==并发执行==** 

**此阶段清理删除掉标记阶段判断的已经死亡的对象，释放内存空间**。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

> 弊端

* 采用**标记-清除算法**，会产生内存碎片

* `CMS` 收集器无法处理浮动垃圾；浮动垃圾是在并发标记阶段产生的，**是将死亡对象错误标记为存活对象** 

* 并发标记阶段会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低



15、**G1 垃圾收集器的步骤。有什么缺点？**【⭐⭐⭐⭐】

`young gc -> young gc + concurrent mark -> Mixed GC` 

> 当年轻代的 Eden 区用尽时开始年轻代回收过程

**G1 的年轻代收集阶段是一个并行的独占式收集器。在年轻代回收期，G1 GC 暂停所有应用程序线程，启动多线程执行年轻代回收**。然后**从年轻代区间移动存活对象到 Survivor 区间或者老年区间，也有可能是两个区间都会涉及** 



> 老年代并发标记过程（Concurrent  Marking）

1、初始标记阶段：标记从根节点直接可达的对象。这个阶段是 STW 的，并且会触发一次年轻代 GC

2、根区域扫描：G1 GC 扫描 Survivor 区直接可达的老年代区域对象，并标记被引用的对象。这一过程必须在 Young GC 之前完成

3、并发标记：在整个堆中进行并发标记（和应用程序并发执行），此过程可能被 young GC 中断。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那这个区域会被立即回收。同时，并发标记过程中，会计算每个区域的对象活性（区域中存活对象的比例）

4、再次标记：**由于应用程序持续进行，需要修正上一次的标记结果**。是 STW 的。G1 中采用了比 CMS 更快的初始快照算法：snapshot-at-the-beginning（SATB）

5、独占清理：计算各个区域的存活对象和 GC 回收比例，并进行排序识别可以混合回收的区域，为下阶段做铺垫，是 STW 的 

6、并发清理阶段：识别并清理完全空闲的区域

> 混合回收过程

当越来越多的对象晋升到老年代 `Old Region` 时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即 Mixed GC，该算法并不是一个 Old GC，除了回收**整个 Young Region，还会回收一部分的 Old Region**。

这里需要注意：**是一部分老年代，而不是全部老年代**。可以选择哪些 Old Region 进行收集，从而可以对垃圾回收的耗时时间进行控制。也要注意的是 **Mixed GC 并不是 Full GC** 

**并发标记结束以后，老年代中百分百为垃圾的内存分段被回收了，部分为垃圾的内存分段被计算了出来**。默认情况下，这些老年代的内存分段会分 8 次（可以通过 -XX:G1MixedGCCountTarget 设置）被回收

由于老年代中的内存分段默认分 8 次回收，G1 会优先回收垃圾多的内存分段。**垃圾占内存分段比例越高的，越会被先回收**。并且有一个阈值会决定内存分段是否被回收， **-XX:G1MixedGCLiveThresholdPercent**，默认为 65%，意思是垃圾占内存分段比例要达到 65% 才会被回收。

混合回收并不一定要进行 8 次。有一个阈值 -XX:G1HeapWastePercent，默认值为10%，**意思是允许整个堆内存中有10% 的空间被浪费，意味着如果发现可以回收的垃圾占堆内存的比例低于10%，则不再进行混合回收。因为 GC 会花费很多的时间但是回收到的内存却很少** 

G1 的初衷就是要避免 Full GC 的出现。但是如果上述方式不能正常工作，G1 会停止应用程序的执行（stop-The- World），使用单线程的内存回收算法进行垃圾回收，性能会非常差，应用程序停顿时间会很长

要避免 Full GC 的发生，一旦发生需要进行调整。什么时候会发生 Full GC 呢？比如堆内存太小，当 G1 在复制存活对象的时候没有空的内存分段可用则会回退到 Full GC，这种情况可以通过增大内存解决

> 缺点

在用户程序运行过程中， G1 无论是为了垃圾收集产生的**内存占用**还是程序运行时的**额外执行负载**都要比 CMS 要高

从经验上来说，在小内存应用上 CMS 的表现大概率会优于 G1，而 G1 在大内存应用上则发挥其优势。平衡点在6-8GB 之间



16、**讲一下内存分配策略？**【⭐⭐⭐⭐】

1）优先在 **Eden** 区为对象分配内存，如果Eden区不足分配对象，会做一个 **Minor GC**，回收内存，尝试分配对象，如果依然不足分配，才分配到老年代

2）大对象直接进入老年代。**大对象是指需要大量连续内存空间的Java对象**，最典型的大对象就是那种很长的字符串及数组，虚拟机提供了一个 -XX:PretenureSizeThreshold 参数，令大于这个设置值的对象直接在老年代中分配。

这样做的目的是避免在Eden区及两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。**PretenureSizeThreshold 参数只对Serial和ParNew两款收集器有效**。

3）长期存活的对象将进入老年代。在触发了Minor GC后，存活对象被存入 Survivor 区在经历了多次 Minor GC 之后，如果仍然存活的话，则该对象被晋升到老年代。

4）Minor GC 后 Survivor 空间不足就直接放入老年代



17、**虚拟机基础故障处理工具有哪些？**【⭐⭐⭐】

JProfiler、jvisualvm

jinfo命令实时查看和调整虚拟机各项参数



18、什么是字节码？类文件结构的组成了解吗？【⭐⭐⭐⭐】

> **字节码** 

源代码经过编译器编译之后使会生成一个字节码文件，**字节码是一种二进制的类文件**，它的内容是 JVM 的指令，而不像 **C、C++ 经由编译器直接生成==机器码==** 

> **类文件组成** 

1、magic number：魔数

2、Class 文件的版本号

3、常量池、访问标识符、类索引

4、方法表、字段表、属性表等



19、**类的生命周期？类加载的过程了解么？加载这一步主要做了什么事情？初始化阶段中哪几种情况必须对类初始化？** 

> **声命周期** 

加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）。**其中验证、准备、解析三个部分又统称为连接（Linking）**。

> **类加载过程** 

1）类加载过程的第一步，主要完成下面 3 件事情：

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口

2）验证：它的目的是保让加载的字节码是合法、合理并且符合规范的

格式检查、语义检查、字节码验证、符号引用验证

3）准备（Preparation）阶段：在这个阶段，**虚拟机就会为这个类分配相应的内存室间，并设置默认初始值** 

> 注意

* 这里不包含基本数据类型的字段用 `static final` 修饰的情况，**因为 final 在==编译时==就会分配了**，准备阶段会显式赋值

* 注意这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到 Java 堆中

* 在这个阶段并不会像初始化阶段中那样会有初始化或者代码被执行

4）解析（Resolution）阶段：**将类、接口、字段、方法的符号引用转为直接引用**。解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，也就是**得到类或者字段、方法在内存中的指针或者偏移量**。

5）初始化：为类的静态变量赋予正确的**初始值**。初始化阶段是执行初始化方法 `<clinit> ()`方法的过程，是类加载的最后一步，这一步 JVM 才开始真正执行类中定义的 Java 程序代码(字节码)。说明： `<clinit> ()`方法是编译之后自动生成的。

**对于初始化阶段，虚拟机严格规范了有且只有 5 种情况下，必须对类进行初始化(只有主动去使用类才会初始化类)：** 

1. 当遇到 `new` 、 `getstatic`、`putstatic` 或 `invokestatic` 这 4 条直接码指令时，比如 `new` 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。

- 当 jvm 执行 `new` 指令时会初始化类。即当程序创建一个类的实例对象。
- 当 jvm 执行 `getstatic` 指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。
- 当 jvm 执行 `putstatic` 指令时会初始化类。即程序给类的静态变量赋值。
- 当 jvm 执行 `invokestatic` 指令时会初始化类。即程序调用类的静态方法。

2. 使用 `java.lang.reflect` 包的方法对类进行反射调用时如 `Class.forname("...")`, `newInstance()` 等等。如果类没初始化，需要触发其初始化。

3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。

4. 当虚拟机启动时，用户需要定义一个要执行的主类 (包含 `main` 方法的那个类)，虚拟机会先初始化这个类。

5. `MethodHandle` 和 `VarHandle` 可以看作是轻量级的反射调用机制，而要想使用这 2 个调用， 就必须先使用 `findStaticVarHandle` 来初始化要调用的类。

6. 当一个接口中定义了 JDK8 新加入的默认方法（被 default 关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。



20、讲一下双亲委派模型。【⭐⭐⭐⭐⭐】

如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把这个请求任务委托给父加载器去完成，依次递归，如果父加载器可以完成类加载任务，就成功返回。只有父类加载器无法完成此加载仼务时，才自己加载

> 优势

1、避免类的重复加载，确保一个类的全局唯一性

​    Java 类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以**免类的重复加载**，当父亲已经加载了该类时，就没有必要子 ClassLoader 再加载一次

2、保护程序安全，防止核心 API 被随意篡改

​    JDK 为核心类库提供了一层保护机制。**不管是自定义的类加载器，还是系统类加载器或扩展类加载器，最终都必须调用java.lang.ClassLoader.defineClass（String，byte[]，int，int，ProtectionDomain）方法，而该方法会执行 preDefineClass（）接口，该接口中提供了对 JDK 核心类库的保护** 

> 弊端

检查类是否加载的委托过程是单向的，这个方式虽然从结构上说比较清晰，使各个 ClassLoader 的职责非常明确，但是同时会带来一个问题，即**顶层的 ClassLoader 无法访问底层的 ClassLoader 所加载的类** 



