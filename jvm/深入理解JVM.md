### 虚拟机
所谓虚拟机，就是一台虚拟的计算机。它是一款软件，用来执行一系列虚拟计算机指令。

### java虚拟机结构
![JVM基本结构](https://github.com/southCountry/omar-blog/raw/master/images/flink/jvm-module.png)

- 类加载子系统负责从文件系统或者网络中加载Class的信息，加载的类信息会存放于方法区。同时，方法区还会存放运行时常量池信息，包括字符串字面量和数字常量。
- Java堆再虚拟机启动时建立，几乎所有对象实例都存放于堆中。堆是所有线程共享的。
- 直接内存是在java堆外直接向系统申请的内存空间。访问速度更快但是内存分配速度更慢。
- 垃圾回收器可以对方法区、堆和直接方法区内存进行回收。
- 每个Java虚拟机线程都有一个私有的Java栈。Java栈在线程被创建的时候创建。
- 本地方法栈用于线程对本地方法的调用。
- PC寄存器是线程私有的，指向当前正在被执行的指令。
- 执行引擎负责虚拟机字节码（class文件）的执行。现代虚拟机为了提高执行效率，会使用编译技术，将方法编译成机器码之后执行：JIT即时编译技术。

### Java堆

### Java栈
![栈帧和函数调用](https://github.com/southCountry/omar-blog/raw/master/images/flink/java-zhan.png)

#### 栈上分配
对于线程私有的对象可以分配在栈上，不是分配在堆上。

### 方法区
jdk1.8之前，方法区可以理解为永久代。jdk1.8使用元数据区替代方法区。

## jvm参数配置

## GC
### GC算法
#### 引用计数法
- 无法处理循环引用
- 引用计数器在引用产生和消除时，需要维护计数，对系统性能有较大影响

#### 标记清除法
现代垃圾回收基础。

问题：会产生空间碎片

#### 复制算法
将原有空间分为两块，每次只使用一块。

#### 标记压缩
在标记清除动作之后，执行一次内存压缩。

#### 分代算法
- 新生代：90%的对象会很快被回收
- 老年代：多次GC后仍然能够存活的对象

为了支持高频率的新生代GC，虚拟机可能会使用卡表的数据结构。卡表为一个比特位的集合，每一个比特位可以用来标识老年代的某一区域中的所有对象是否持有新生代对象的引用。只有卡表的标记位为1时，才需要扫描给定区域的老年代对象。
![卡表逻辑结构](https://github.com/southCountry/omar-blog/raw/master/images/flink/cart-table.png)

#### 分区算法
分代算法按照对象的生命周期长短划分为两个部分，分区算法将整个堆划分为连续的小区间。每一个区间独立使用，独立回收。
![分区算法示意图](https://github.com/southCountry/omar-blog/raw/master/images/flink/partition-mem.png)

#### STW
垃圾回收，无论新生代还是老年代都会使应用程序产生停顿。

## 垃圾收集器
### 串行回收器
使用单线程执行GC

#### 新生代串行回收器
#### 老年代串行回收器
- 单线程
- 独占（与并发相对应）

### 并行回收器

#### 新生代ParNew回收器
串行回收器的多线程化

#### 新生代ParallelGC回收器
也是使用复制算法，也是多线程、独占式收集器。

关注系统吞吐量，提供参数控制系统的吞吐量。

#### 老年代ParallelOldGC
使用标记压缩算法的多线程并发收集器。

#### CMS收集器
关注系统停顿时间，使用标记清除算法的多线程并行回收的垃圾收集器。

- 初始标记：STW:标记根对象
- 并发标记：标记所有对象
- 预清理：清理前准备以及控制停顿时间
- 重新标记：STW：修正并发标记数据
- 并发清理：清理垃圾
- 并发重置

#### G1收集器
垃圾收集过程如下

1. 新生代GC：Eden区满了，则启动新生代GC。
2. 并发标记周期
- 初始标记：标记从根节点直接可达的对象。伴随一次新生代GC，会STW
- 根区域扫描：扫描由survivor区直接可达的老年代趋于，并标记这些直接可达的对象。与应用程序并行执行。
- 并发标记：扫描并查找整个堆的存活对象，并做好标记
- 重新标记：会STW
- 独占清理：会STW。计算各个区域的存活对象和GC回收比例并进行排序。
- 并发清理：和应用程序并行执行
3. 混合回收
并发标记虽然会清理部分垃圾，但是回收比例较低，在混合回收阶段专门针对垃圾较多的区域进行收集。
4. 必要时执行full gc
- G1并发标记时，由于老年代被快速填充，G1会终止并发标记转入full gc
- 如果GC时发生空间不足或者新生代Gc时，survivor区和老年代无法容纳存活对象时，触发full gc

#### 对象分配简要流程
![对象分配简要流程](https://github.com/southCountry/omar-blog/raw/master/images/flink/object-allocation.png)

## String在虚拟机中的实现
1. 不变性：对象创建之后就不再变化
2. 针对常量池的优化：String.intern()返回字符串在常量池中的引用
3. 类的final定义，有助于虚拟机寻找机会内联所有final方法

**jdk1.6中，string的substring方法可能会引起内存泄露**

## 锁
### 偏向锁
核心思想：如果程序没有竞争，则取消之前已经取得锁的线程同步操作。即如果线程已经取得锁，再次请求该锁时，无需进行同步操作。如果之间有其他线程请求了锁，则锁退出偏向锁模式。

在锁竞争强烈场景下，建议关闭。

### 轻量级锁
如果偏向锁失败，虚拟机会让线程申请轻量级锁，其内部使用一个称为BasicObjectLock的对象实现。

首先BasicLock通过set_displaced_header()方法备份原对象的mark Word。然后，使用cas操作，尝试将BasicLock的地址复制到对象头的mark word。如果复制成功，则枷锁成功，失败则膨胀微重量级锁。
![轻量级锁示意图](https://github.com/southCountry/omar-blog/raw/master/images/flink/light-lock.png)

### 锁膨胀
1. 废弃前面BasicLock备份的对象头信息
2. 通过inflate方法进行锁膨胀，目的是获得对象的ObjectMonitor
2. 使用enter方法尝试进入该锁。在enter方法调用中，线程很可能会在操作系统中被挂起，如此，线程间切换和调度成本会较高。

### 自旋锁
使在线程没有取得锁的时候，不被挂起，执行一个空循环。若干循环之后，再次尝试获取锁，如果不能获得锁，则挂起。

### 锁优化思路
1. 减少锁持有时间
2. 减少锁粒度，分段锁ConcurrentHashMap

### 锁分离
不同操作使用不同的锁

### 无锁 cas操作

### LongAdder
![原子类优化思路](https://github.com/southCountry/omar-blog/raw/master/images/flink/longAdder.png)

## 类加载系统
![class文件装载过程](https://github.com/southCountry/omar-blog/raw/master/images/flink/classload-system.png)

### 加载类
1. 通过类的全名，获得类的二进制数据流
2. 解析类的二进制数据流为方法区内的数据结构
3. 创建java.lang.Class类的实例，标识该类型

### 连接
#### 验证类
![验证类流程](https://github.com/southCountry/omar-blog/raw/master/images/flink/validate-class.png)

#### 准备
分配内存空间，并设置初始值

#### 解析
将类、接口、字段和方法的符号引用转为直接引用。

### 初始化
执行类的静态代码

## ClassLoader
![classloader层次结构](https://github.com/southCountry/omar-blog/raw/master/images/flink/classloader-layer.png)

### spi打破双亲委派模型
![上下文加载器](https://github.com/southCountry/omar-blog/raw/master/images/flink/spi.png)

#### 热替换
![热替换基本思路](https://github.com/southCountry/omar-blog/raw/master/images/flink/hot-replace.png)
