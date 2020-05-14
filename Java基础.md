# 一、集合框架

<img src=".\images\JcQf7KG2rUN5MnsOX9VDgUfRn82x9QTB.png" alt="Java集合框架" style="zoom: 67%;" />

## 1. List

- 有序
- 元素可重复
- 可通过索引操作元素

|            | ArrayList | LinkedList | Vector                                 |
| ---------- | --------- | ---------- | -------------------------------------- |
| 底层实现   | 数组      | 双向链表   | 数组                                   |
| 线程安全性 | 不安全    | 不安全     | 线程安全性使用synchronized实现，效率低 |
| 查询元素   | O(1)      | O(n)       | O(1)                                   |
| 增删元素   | O(n)      | O(1)       | O(n)                                   |

## 2. Set

- 元素唯一

|                | HashSet | TreeSet                      |
| -------------- | ------- | ---------------------------- |
| 存储结构       | HashMap | 红黑树                       |
| 线程安全性     | 不安全  | 不安全                       |
| 顺序性         | 无序    | 有序（自然排序、自定义排序） |
| 查询、增删元素 | O(1)    | O(logn)                      |

## 3. HashMap

#### 存储结构

- JDK 1.7及之前采用数组 + 链表

- JDK 1.8开始采用数组 + 链表 + 红黑树
当一个桶存储的链表长度大于等于8时会将链表转成红黑树，提高访问效率

<img src=".\images\mVgnaq09AhehvZZPBmDdeC6Tj2EtUByL.png" style="zoom: 80%;" />

#### Hash方法实现

根据key计算元素所在桶的下标

<img src="E:\Files\Notes\images\qGmXOyQEYbJtQQBe4QcyrABe1drbmxKN.png" alt="image-20200430174139116" style="zoom:67%;" />

#### put() 逻辑

- 如果没有初始化，则进行初始化
- 对key求Hash值，然后计算数组下标
- 如果没有发生碰撞，直接放入桶中；如果发生碰撞，以链表的形式附加到后面
- 如果节点已经存在，替换原值
- 如果链表长度大于等于8，将链表转为红黑树；如果链表长度小于6，将红黑树转为链表
- 如果桶满，扩容2倍后重排

## 4. ConcurrentHashMap

#### 线程安全的原理

- 分段锁
 JDK 1.7的 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶，多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高。

- CAS + synchronized
  JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized. 

#### put() 逻辑

- 如果没有初始化，则进行初始化
- 对key求Hash值，然后计算数组下标
- **如果检测到内部正在扩容，则协助它一起扩容**
- **如果头结点不存在，使用CAS添加头结点，失败则循环重试**
- **如果头结点存在，则尝试获取头结点的同步锁，再进行操作**
- 如果链表长度大于等于8，将链表转为红黑树；如果链表长度小于6，将红黑树转为链表
- 如果桶满，扩容2倍后重排

TODO HashMap、ConcurrentHashMap、Hashtable区别

各个集合的扩容方式

# 二、Java虚拟机

## 1. 虚拟机组成

<img src=".\images\Ii4qkrhaWR494I8Lv2VawfLFx7Emb7FY.png" alt="image-20200429084341992" style="zoom:67%;" />

- Class Loader：依据特定格式，加载字节码文件到内存
- Execution Engine：对命令进行解析
- Native Interface：调用不同语言的原生库
- Runtime Data Area：JVM内存空间结构模型

## 2. 类加载器

### 类加载器分类

- 启动类加载器 (Bootstrap ClassLoader): C++编写，加载核心库java.*
- 扩展类加载器 (Extension ClassLoader): Java编写，加载扩展库javax.*
- 应用程序类加载器 (Application ClassLoader): Java编写，加载程序所在目录的文件
- 自定义类加载器(User Define ClassLoader): Java编写，定制化加载

### 自定义类加载器

- 自定义类加载器继承自 java.lang.ClassLoader，用于加载文件系统上的类
- 首先根据类的全名在文件系统上查找类的字节代码文件`findClass()`，读取该文件内容
- 最后通过 `defineClass()`方法来把这些字节代码转换成 java.lang.Class 类的实例

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}

protected final Class<?> defineClass(byte[] b, int off, int len) throws ClassFormatException {
    return defineClass(null, b, off, len, null);
}
```

- 自定义类加载器需要去重写 findClass() 方法，示例如下：

```Java
import java.io.*;

public class MyClassLoader extends ClassLoader {
    private String path;
    private String classLoaderName;

    public MyClassLoader(String path, String classLoaderName) {
        this.path = path;
        this.classLoaderName = classLoaderName;
    }

    // 自定义类加载器需要去重写该方法
    @Override
    public Class findClass(String name) {
        byte[] b = loadClassData(name);
        return defineClass(name, b, 0, b.length);
    }

    // 从指定目录加载class文件
    private byte[] loadClassData(String name) {
        name = path + name + ".class";
        InputStream in = null;
        ByteArrayOutputStream out = null;
        try{
            in = new FileInputStream(new File(name));
            out = new ByteArrayOutputStream();
            int i = 0;
            while ((i = in.read()) != -1) {
                out.write(i);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        finally {
            try {
                out.close();
                in.close();
            }
            catch (Exception e) {
                e.printStackTrace();
            }
        }
        return out.toByteArray();
    }
}
```

### 类加载器的双亲委派机制

作用：防止内存中出现多份同样的字节码

<img src=".\images\ipoMGRGwKp2YkPKf1XOkaWe67FtdLzeS.png" alt="image-20200429091933878" style="zoom:67%;" />

### Java反射

Java反射机制是在运行状态中：

- 对于任意一个类，都能知道这个类的所有属性和方法
- 对于任意一个对象，都能够调用它的任意方法和属性

### 类从编译到执行的过程

以源文件Demo.java为例

- 编译器将Demo.java源文件编译为Demo.class字节码文件
- 类加载器将字节码文件转换成JVM中的Class\<Demo\>对象
- JVM利用Class\<Demo\>对象实例化为Demo对象

### 类的装载过程

- 加载：通过类加载器加载Class字节码文件，生成Class对象
- 链接
  - 校验：检查加载的Class的正确性和安全性
  - 准备：为类变量分配存储空间，设置类变量初始值
  - 解析：JVM将常量池内的符号引用转换为直接引用
- 初始化：执行类变量赋值和静态代码块

### loadClass和forName的区别

- loadClass得到的Class还没有链接
- forName得到的Class已经完成初始化

## 3. 内存模型

<img src=".\images\yGMYvQcfleLmfXhfYaNGbrVOCrilifW8.png" alt="image-20200429090355706" style="zoom:67%;" />

### 线程私有部分

#### 程序计数器

- 当前线程所执行的字节码行号指示器
- 改变计数器的值来选取下一条需要执行的字节码指令
- 与线程是一对一的关系，不会发生内存泄露
- 对Java方法计数，如果是Native方法则计数器值为Undefined

#### 虚拟机栈

<img src=".\images\JLHt4Y5rXANWvVdp08X32Hdb0RSRQnwK.png" alt="image-20200429101429629" style="zoom:67%;" />

#### 本地方法栈

与虚拟机栈类似，主要作用于标注了native的方法

### 线程共享部分

#### 堆

所有对象都在这里分配内存，是垃圾收集的主要区域

#### 方法区

- 方法区是一个 JVM 规范，用于存放已被加载的类信息、常量、静态变量等数据，永久代与元空间都是其一种实现方式
- JDK 1.8之前，方法区位于永久代，使用虚拟机内存
- JDK 1.8开始，方法区被移至元空间，使用本地内存

##### 运行时常量池

运行时常量池是方法区的一部分，Class 文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域。

### 堆和栈的区别

- 管理方式：栈自动释放，堆需要GC
- 空间大小：栈比堆小
- 碎片：栈产生的碎片小于堆
- 分配方式：栈支持静态和动态分配，堆只支持动态分配
- 效率：栈的效率比堆高

## 4. 垃圾回收机制

### 判断对象是否为垃圾的算法

#### 引用计数算法

- 每个对象实例都有一个引用计数器，被引用则+1，完成引用则-1
- 通过判断对象的引用数量来决定对象是否可以被回收，引用计数为 0 的对象可被回收
- 无法检测出循环引用，会导致内存泄露

#### 可达性算法

以 GC Root 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。

可以作为GC Root的对象有：

- 虚拟机栈中局部变量表中引用的对象
- 本地方法栈中 JNI 中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象
- 活跃线程引用的对象

### 引用类型

#### 强引用 (Strong Reference)

- 最普遍的引用 `Object obj = new Object()`
- 抛出`OutOfMemoryError`终止程序也不会回收具有强引用的对象
- 通过将对象设置为null来弱化引用，使其被回收

#### 软引用 (Soft Reference)

- 对象处在有用但非必须的状态
- 只有当内存空间不足时，垃圾收集器才会回收该引用的对象的内存
- 可以用来实现高速缓存

```java
String s = new String("abc");									// 强引用
SoftReference<String> softRef = new SoftReference<String>(s);	// 弱引用
s = null;  														// 使对象只被软引用关联
```

#### 弱引用 (Weak Reference)

- 非必须的对象

- 垃圾回收时一定会被回收

- 适用于引用偶尔被使用且不影响垃圾收集的对象

```java
String s = new String("abc");									
WeakReference<String> weakRef = new WeakReference<String>(s);
```


#### 虚引用 (Phantom Reference)

- 不会决定对象的声明周期，也无法通过虚引用得到一个对象
- 为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知
- 必须和引用队列`ReferenceQueue`联合使用

```java
String s = new String("abc");									
ReferenceQueue queue = new ReferenceQueue();
PhantomReference phantomRef = new PhantomReference(s, queue);
```

#### 四种引用的比较

| 引用类型 | 被垃圾回收时间 | 用途           | 生存时间          |
| -------- | -------------- | -------------- | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态 | JVM停止运行时终止 |
| 软引用   | 内存不足时     | 对象缓存       | 内存不足时终止    |
| 弱引用   | 垃圾回收时     | 对象缓存       | GC运行后终止      |
| 虚引用   | 未知           | 标记、哨兵     | 未知              |



### 垃圾收集算法

#### 标记 - 清除算法

- 标记阶段：从GC Root开始扫描，对存活对象进行标记
- 清除阶段：对堆内存从头到尾进行线性遍历，进行对象回收并取消标志位
- 缺点：会产生大量不连续的内存碎片

<img src=".\images\8WJsmQH91WOQJNkdyAaCE7f2q3umimrP.png" alt="image-20200428160003661" style="zoom: 67%;" />

#### 标记 - 整理算法

- 标记阶段：从GC Root开始扫描，对存活对象进行标记
- 清除阶段：移动所有存活对象，且按照内存次序依次排列，然后将末端内存地址以后的内存全部回收
- 优点：不会产生内存碎片
- 缺点：需要移动大量对象，处理效率比较低

<img src=".\images\iKg0KJfZhp3uVxzguu5cbbctw5bfb4ly1.png" alt="image-20200428160745553" style="zoom: 58%;" />

#### 复制算法

- 将内存分为对象面和空闲面，对象在对象面创建
- 执行垃圾回收时，存活的对象被从对象面复制到空闲面，将对象面所有对象清除

<img src=".\images\RJj7EMASAKzdczlL2TGgMFeF9wtydbYb.png" alt="image-20200428160440452" style="zoom: 50%;" />

#### 分代收集算法

按照对象生命周期的不同划分区域以采用不同的垃圾回收算法，可以提高JVM的垃圾回收效率。

一般将堆分为新生代和老年代(大小为1 : 2), 新生代分成一个Eden区和两个Survivor区(大小为8 : 1 : 1).

- 新生代：复制算法
- 老年代：标记 - 清除算法、标记 - 整理 算法

<img src="E:\Files\Notes\images\6CBfqz37rKKTbJV90InwTcGInZEhNbrM.png" alt="image-20200428225101495" style="zoom: 67%;" />

##### Minor GC 和 Full GC

- Minor GC：回收新生代，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快
- Full GC：回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多

##### 对象如何晋升到老年代

- 经历一定次数Minor GC仍然存活的对象
- Survivor区中存放不下的对象
- 新生成的大对象 `-XX:PretenureSizeThreshold`

##### 触发Full GC的条件

- 老年代空间不足
- JDK 1.7 及以前的永久代空间不足
- 调用System.gc()
- CMS GC时出现promotion failed, concurrent mode failure
- Minor GC晋升到老年代的平均大小大于老年代的剩余空间

##### Stop-the-world

- JVM由于要执行GC而暂停了应用程序的执行
- 任何一种GC算法中都会发生
- 多数GC优化通过减少Stop-the-world发生的时间来提高程序性能

##### Safepoint

- 分析过程中对象引用关系不会变化的点
- 产生Safepoint的地方：方法调用、循环跳转、异常跳转等等

### 垃圾收集器

以下7个垃圾收集器，连线表示垃圾收集器可以配合使用

<img src=".\images\D2Y8t3VGm1Pt2kCQGJhXnYjPU8Y5gDlT.png" alt="image-20200428164842364" style="zoom:67%;" />

#### JVM运行模式

- Client模式：启动快，稳定后程序运行速度慢
- Server模式：启动时间长，优化多，稳定后程序运行速度快

#### 新生代垃圾收集器

##### Serial收集器

- Client模式下默认的年轻代收集器
- 单线程收集，进行垃圾收集时，必须暂停所有工作线程
- 采用复制算法

<img src=".\images\U6j0Y4FvOfiw5vYeoT4wHhRYnbqieejs.png" alt="image-20200428171025186" style="zoom:60%;" />

##### ParNew 收集器

- 多线程收集，其余行为特点和Serial收集器一样
- 单核执行效率不如Serial，在多核下执行才有优势
- 采用复制算法

<img src=".\images\YRnPpOQ0yLloaSrYmlhlXevWlIMrBQ8L.png" alt="image-20200428171131256" style="zoom: 67%;" />

##### Parallel Scavenge收集器

- Server模式下默认的年轻代收集器
- 更加关注系统的吞吐量 ，吞吐量 = 运行用户代码的时间 / (运行用户代码的时间 + 垃圾收集时间)
- 采用复制算法

#### 老年代垃圾收集器

##### Serial Old收集器

- Client模式下默认的老年代收集器
- 采用标记 - 整理算法

<img src=".\images\Jk0lmnLtIzwg6t50xh5zAePmwLdK4448.png" alt="image-20200428172047980" style="zoom:67%;" />

##### Parallel Old收集器

- 多线程收集，吞吐量优先
- 采用标记 - 整理算法

<img src=".\images\u5yZJUdh9zgXU4O5gMj9tS4N5WTIfCgW.png" alt="image-20200428172340469" style="zoom:67%;" />

##### CMS收集器

采用标记 - 清除算法，分为以下几个阶段：

- 初始标记：仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿
- 并发标记：进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿
- 并发清理：不需要停顿

<img src=".\images\IHyBKuX6bpJdTHIsOhadLYc8xQ1Jb3fV.png" alt="image-20200428172854177" style="zoom:75%;" />

#### G1收集器

- 使用复制、标记 - 整理算法
- 将整个Java堆内存划分成多个大小相等的Region，年轻代和老年代不再物理隔离，且运行期间不会产生内存空间碎片
- 可预测的停顿，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒

<img src=".\images\pSOUAbg3MjZrJxee2djHi9n80Pp5fLQq.png" alt="image-20200428173400027" style="zoom:50%;" />

## 5. 虚拟机性能调优

TODO

# 三、多线程与并发

## 1. 进程与线程

- 进程是资源分配的基本单位，有独立的地址空间，进程切换开销大
- 线程是CPU调度的基本单位，线程属于某个进程，共享其资源，线程切换开销小

### Java进程与线程的关系

- Java采用单线程编程模型，程序会自动创建主线程，主线程可以创建子线程
- 每个进程对应一个JVM实例，属于同一个进程的多个线程共享JVM里的堆

## 2. 线程状态

- 新建 NEW
创建后尚未启动

- 可运行 RUNABLE
包含 RUNNING 和 READY

- 无限期等待 WAITING
不会被分配CPU执行时间，等待其它线程显式唤醒
- 限期等待 TIMED_WAITING
在一定时间之后会被系统自动唤醒
- 阻塞 BLOCKED
等待获取排它锁
- 死亡 TERMINATED
可以是线程结束任务之后自己结束，或者产生了异常而结束

### 线程状态转换

#### 锁池
假设线程A已经拥有了某个对象的锁，另外有两个线程B、C想要进入这个对象的某个synchronized方法(或者块)
由于线程B、C在进入该对象的synchronized方法(或者块)前必须先获得锁，而该对象的锁当前正被线程A占用，此时线程B、C就会被阻塞，进入该对象的`锁池`去等待锁的释放。

#### 等待池
假设线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁，同时线程A就会进入该对象的等待池中，进入等待池的线程不会去竞争该对象的锁。

<img src=".\images\NABk0PXwdsvpbSd6FjCeOVApuVL3KH2R.png" alt="image-20200429170526372" style="zoom:67%;" />

## 3. 线程机制

### sleep()和wait()区别

| sleep()            | wait()                                       |
| ------------------ | -------------------------------------------- |
| Thread类的方法     | Object类的方法                               |
| 可以在任何地方使用 | 只能在synchronized方法或synchronized块中使用 |
| 只会让出CPU        | 不仅会让出CPU，还会释放已经占有的同步资源锁  |

### notify()和notifyAll()的区别

- notifyAll()会让**所有**处于等待池的线程进入锁池去竞争获取锁的机会
- notify()只会随机选取**一个**处于等待池的线程进入锁池去竞争获取锁的机会

### yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，但是线程调度器**可能忽略这个建议**。

### interrupt()

调用interrupt()通知线程应该中断：
- 如果线程处于阻塞状态，那么线程将立即退出阻塞状态，并抛出InterruptedException
- 如果线程处于正常活动状态，那么会将该线程的中断标志设置为true，被设置中断标志的线程将继续运行，不受影响

## 4. 使用线程

### 创建线程

- 实现 Runnable 接口

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
    
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start();
    }
}
```

- 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 0;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> future = new FutureTask<>(new MyCallable());
        Thread t = new Thread(future);
        t.start();
        System.out.println(future.get());
    }
}
```

- 继承 Thread 类

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }

    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();
    }
}

```

### 给线程传递参数
- 构造函数传参
- 成员变量传参
- 回调函数传参

### 获取线程的返回值
- 主线程等待法
- 使用Thread类的join()阻塞当前线程以等待子线程处理完毕
- 通过FutureTask或线程池获取

## 5. 线程池

### 线程池分类

Executors工具类提供了一系列静态工厂方法用来创建不同的线程池满足不同场景的需求：

- newFixedThreadPool(int n)
指定工作线程数量的线程池

- newCachedThreadPool()
处理大量短时间工作任务的线程池

- newSingleThreadExecutor()
创建唯一的工作线程来执行任务，如果线程异常结束，会有另一个线程取代它

```java
public class Main{
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(new MyRunnable());		// 提交工作任务
        executorService.shutdown();						// 关闭线程池
    }
}
```

### ThreadPoolExecutor

#### 构造函数的核心参数

| 参数            | 含义             |
| --------------- | ---------------- |
| corePoolSize    | 核心线程池大小   |
| maximumPoolSize | 最大线程池大小   |
| keepAliveTime   | 线程最大空闲时间 |
| workQueue       | 线程等待队列     |
| handler         | 拒绝策略         |

#### 饱和策略

| 策略名称            | 含义                                       |
| ------------------- | ------------------------------------------ |
| AbortPolicy         | 默认策略，直接抛出异常                     |
| CallerRunsPolicy    | 用调用者所在的线程来执行任务               |
| DiscardOldestPolicy | 丢弃工作队列中最靠前的任务，并执行当前任务 |
| DiscardPolicy       | 直接丢弃任务                               |

#### 工作流程

向线程池提交新任务后：

- 运行线程数 < corePoolSize，创建新线程来处理任务
- corePoolSize <= 运行线程数 < maximumPoolSize，只有当workQueue满时才创建新线程去处理任务
- 运行线程数 >= maximumPoolSize，如果workQueue满，通过handler指定的策略来处理任务

<img src=".\images\KiL5kMCT79IcLF4nK6lel1XzobZzNxSq.png" alt="image-20200429175403923" style="zoom: 67%;" />

### 线程池状态

- RUNNING
能接受新提交的任务

- SHUTDOWN
不再接受新提交的任务，但可以处理存量任务

- STOP
不再接受新提交任务，也不处理存量任务

- TIDYING
所有任务都已经终止

- TERMINATED
terminated()方法执行后进入该状态

#### 线程池状态转换

<img src=".\images\5YimYEiVDoEDgTtbGp2mQrylusfs21vi.png" alt="image-20200429182652823" style="zoom: 67%;" />

### submit()与execute()区别

```java
// java.util.concurrent.ExecutorService
void execute(Runnable command);
<T> Future<T> submit(Callable<T> task);
```

- 接收的参数不一样
- submit()有返回值，execute()没有返回值
- submit()可以通过Future.get()捕获抛出的异常

### 如何确定线程池大小

- CPU密集型：线程数 = CPU核数 + 1
- I/O密集型：线程数 = CPU核数 x ( 1 + 平均等待时间 / 平均工作时间)

## 6. 线程安全

### 引发线程安全问题的原因

- 存在共享数据
- 有多个线程共同操作共享数据

### 互斥锁

同一时刻有且只能有一个线程在操作共享数据，其他线程必须等到该线程处理完数据之后才能对共享数据进行操作。

#### synchronized

##### 基本使用

获取对象锁

- 同步代码块 `synchronized(this)` `synchronized(实例对象)` 锁是小括号中的实例对象
- 同步非静态方法`synchronized method` 锁是当前对象的实例对象

获取类锁

- 同步代码块 `synchronized (类.class)` 锁是小括号中的类对象（Class对象）
- 同步静态方法 `synchronized static method` 锁是当前对象的类对象（Class对象）

##### 实现原理

TODO

##### 锁优化技术

- 偏向锁

  很多情况下，锁不存在多线程竞争，总是由同一线程多次获得。

  如果一个线程获得了锁，那么锁就进入偏向模式；当该线程再次请求锁时，只需要去检查：

  - Mark Word的锁标记位是否为偏向锁
  - 当前线程ID是否等于Mark Word的Thread ID


- 自旋锁与自适应自旋锁
很多情况下，共享数据的锁定持续时间很短，切换线程开销大，可以让线程不让出CPU，执行忙循环等待锁的释放。
自适应自旋锁的自旋次数由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

- 锁消除
JIT编译时，对运行上下文进行扫描，去除不可能存在竞争的锁。

- 锁粗化
通过扩大加锁范围，避免反复加锁。

##### 锁的升级原理

锁共有4种状态，级别从低到高依次为：无状态锁、偏向锁、轻量级锁和重量级锁状态。这几个状态会随着竞争情况逐渐升级，锁可以升级但不能降级。

<img src=".\images\GV1lOu6DWu91WttgcaJwCiGRU4JDx6S9.png" style="zoom:67%;" />

##### 各种锁的比较

| 锁       | 优点                         | 缺点                                                         | 使用场景                                 |
| -------- | ---------------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| 偏向锁   | 和执行非同步方法速度几乎一样 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗               | 只有一个线程访问同步块或同步方法         |
| 轻量级锁 | 竞争线程不会阻塞，响应速度快 | 若线程长时间抢不到锁，自旋会消耗CPU性能                      | 线程交替执行同步块或同步方法             |
| 重量级锁 | 竞争线程不会自旋，不消耗CPU  | 线程阻塞，响应时间缓慢，多线程环境下，频繁获取和释放锁会带来巨大的性能消耗 | 追求吞吐量，同步块或同步方法执行时间较长 |

#### ReentrantLock

- 位于java.util.concurrent.locks，基于AQS实现
- 可以实现比synchronized更细粒度的控制，比如设置fairness和condition
- 调用lock()之后，必须调用unlock()释放锁

#### synchronized与ReentrantLock区别

| synchronized  |                   ReentrantLock                    |
| :-----------: | :------------------------------------------------: |
|  Java关键字   |                         类                         |
| 操作Mark Word |              调用Unsafe类的park()方法              |
|       /       |      可以对获取锁的等待时间进行设置，避免死锁      |
|       /       | 可以设置锁的公平性，获取各种锁的信息，实现多路通知 |

### 乐观锁 - CAS

CAS(Compare and Swap) 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。
适用于计数器、序列发生器等场景，J.U.C的atomic包提供了常用的原子性数据类型和更新操作工具。

#### 优点
- 简单高效

#### 缺点
- 若循环时间长，则开销很大
- 只能保证一个共享变量的原子操作
- ABA问题（可以使用AtomicStampedReference 来解决）

### ThreadLocal

TODO

## 7. Java内存模型

[扩展阅读：Java内存模型原理](https://zhuanlan.zhihu.com/p/51613784)

Java内存模型(Java Memory Model)本身是一个抽象的概念，描述的是一组规则或规范，通过这组规范定义了程序中各个变量的访问方式。

### 主内存和工作内存

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。
线程只能直接操作工作内存中的变量，主内存共享的方式是线程各拷贝一份数据到工作内存，操作完成后刷新回主内存。

- 主内存
存储Java实例变量，包括成员变量、类信息、常量、静态变量等。
属于数据共享的区域，多线程并发操作时会引发线程安全问题。
- 工作内存
存储当前方法的所有本地变量信息、字节码行号指示器、Native方法信息，对其他线程不可见。
属于线程私有区域，不存在线程安全问题。

<img src=".\images\n4U6CFdhEhBMyQcKo0nFuMgUun6smXAx.png" alt="image-20200430090343052" style="zoom:50%;" />


### 内存模型三大特性

- 原子性：一个操作或者多个操作要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
- 可见性：当一个线程修改了共享变量的值，其它线程能够立即得知这个修改
- 有序性：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序

### Happens-Before原则

用于描述 2 个操作的内存可见性，如果操作 A happens-before 操作 B，那么 A 的结果对 B 可见。

- **单一线程原则**
在一个线程内，在程序前面的操作先行发生于后面的操作。
- **锁定规则**
一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。
- **volatile 变量规则**
对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。
- **传递规则**
如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。
- **线程启动规则**
Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。
- **线程加入规则**
Thread 对象的结束先行发生于 join() 方法返回。
- **线程中断规则**
对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。
- **对象终结规则**
一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### volatile变量的特殊规则

volatile是JVM提供的轻量级同步机制。

#### 保证可见性

volatile保证了不同线程对该变量操作的内存可见性：
- 线程对变量进行修改之后，要立刻回写到主内存。
- 线程对变量读取的时候，要从主内存中读，而不是从线程的工作内存。

#### 禁止进行指令重排序

- 当程序执行到 volatile 变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行。
- 在进行指令优化时，不能将在对 volatile 变量访问的语句放在其后面执行，也不能把 volatile 变量后面的语句放到其前面执行。

#### volatile与synchronized的区别

| volatile                                 | synchronized                     |
| ---------------------------------------- | -------------------------------- |
| 仅能使用在变量上                         | 可以使用在变量、方法、类层次上   |
| 仅能实现变量的修改可见性，不能保证原子性 | 可以保证变量修改的可见性和原子性 |
| 不会造成线程阻塞                         | 可能会造成线程阻塞               |
| 标记的变量不可以被编译器优化             | 标记的变量可以被编译器优化       |

## 8. J.U.C的其他组件

### J.U.C包的分类

- 线程执行器 executor
- 锁 locks
- 原子变量类 atomic
- 并发工具类 tools
- 并发集合 collections

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200430224046076.png" alt="image-20200430224046076" style="zoom:150%;" />

### BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

- **FIFO 队列** ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
- **优先级队列** ：PriorityBlockingQueue

提供了阻塞的 take() 和 put() 方法：如果队列为空 take() 将阻塞，直到队列中有内容；如果队列为满 put() 将阻塞，直到队列有空闲位置。

<img src=".\images\YMwUBLdokmhpI82S.png" alt="image-20200501101242485" style="zoom: 50%;" />

```java
public class ProducerAndConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    // 生产者
    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    // 消费者
    private static class Consumer extends Thread {
        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }
        for (int i = 0; i < 5; i++) {
            Consumer consumer = new Consumer();
            consumer.start();
        }
        for (int i = 0; i < 3; i++) {
            Producer producer = new Producer();
            producer.start();
        }
    }
}
```



### ForkJoin

主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

对于一个比较大的任务，可以把这个任务分割为若干互不依赖的子任务，将这些子任务分别放到不同的**双端队列**里，并为每个队列创建一个单独的线程来执行队列里的任务。

<img src=".\images\Zy94pODIollq5xNb.png" alt="image-20200514212400365" style="zoom:60%;" />

- 工作窃取算法 (work-stealing)

工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。

例如，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争，但是如果队列中只有一个任务时还是会发生竞争。

<img src="E:\Files\Notes\images\AIgKjTz0dalK3s1v.png" alt="image-20200514205601038" style="zoom: 67%;" />

### 并发工具类

#### CountDownLatch

用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

<img src=".\images\4ItnsnogW1g1BqfX.png" alt="image-20200501100919333" style="zoom:67%;" />

```java
public class CountdownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        // 创建计数器
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        // 创建10个线程，提交线程池处理
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                // 计数器-1
                countDownLatch.countDown();
            });
        }
        // 调用await()的主线程进行等待，直到计数器为0才继续执行
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```



#### CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达栅栏时，这些线程才会继续执行。

所有线程都达到栅栏时，可以触发执行另一个预先设置的线程barrierAction.

<img src=".\images\tiwBdOgmthf3fu40.png" alt="image-20200501100457535" style="zoom: 67%;" />

```java
public class CyclicBarrierExample {
    public static void main(String[] args) {
        final int totalThread = 10;
        // 创建循环屏障
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        // 创建10个线程，提交线程池处理
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    // 当前线程进行等待
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                // 所有线程到达栅栏处才继续执行
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```



#### Semaphore

Semaphore可以控制某个资源可被同时访问的线程个数。

<img src=".\images\ReCHWcalcxD8PG2Y.png" alt="image-20200501101429151" style="zoom: 50%;" />

```java
public class SemaphoreExample {
	// 模拟对某个服务的并发请求
    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        // 每次只能有 3 个客户端同时访问
        Semaphore semaphore = new Semaphore(clientCount);
        // 请求总数为10
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    // 获取信号量 -1
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放信号量 +1
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

#### Exchanger

当两个线程到达同步点时，相互交换数据。

<img src=".\images\Qk0QzBwbCrqjdC2m.png" alt="image-20200501104142581" style="zoom: 50%;" />

```java
public class ExchangerExample {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Exchanger<String> exchanger = new Exchanger<>();

        // 线程1
        executorService.execute(() -> {
            try {
                // 交换给线程2的消息
                String message = "This is a message from Thread-1."
                String res = exchanger.exchange(message);
                System.out.println("Thread-1: " + res);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 线程2
        executorService.execute(() -> {
            try {
                // 线程2休眠3秒
                Thread.currentThread().sleep(3000);
                // 交换给线程1的消息
                String message = "This is a message from Thread-2."
                String res = exchanger.exchange(message);
                System.out.println("Thread-2: " + res);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}
```



## 9. 线程同步

控制多个线程按一定顺序执行。

### 多线程轮流打印信息

建立三个线程，A线程打印10次A，B线程打印10次B，C线程打印10次C，要求线程同时运行，交替打印10次ABC

#### synchronized方案

- 对于同一个对象锁而言，同一时刻只可能有一个线程拿到了这个锁，此时其他线程都会被阻塞，直到这个线程执行完同步代码块并释放这个锁后，其他线程才能拿到这个锁。
- state变量用于控制获得锁的线程能否进行打印。

```java
public class MutiThread2 {
    // 对象锁，确保只有一个线程执行同步代码块
    private static Object lock = new Object();
    // 用于判断当前线程能否打印
    private static int state = 0;

    public static void main(String[] args) {
        new Thread(() -> {
            for (int i = 0; i < 10;) {
                synchronized (lock) {
                    while (state % 3 == 0) {
                        System.out.print("A ");
                        state++;
                        i++;
                    }
                }
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10;) {
                synchronized (lock) {
                    while (state % 3 == 1) {
                        System.out.print("B ");
                        state++;
                        i++;
                    }
                }
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10;) {
                synchronized (lock) {
                    while (state % 3 == 2) {
                        System.out.print("C ");
                        state++;
                        i++;
                    }
                }
            }
        }).start();
    }
}
```

#### Semaphore方案

```java
public class MultiThread {
    // A线程先开始打印，初始为1
    private static Semaphore A = new Semaphore(1);
    // B、C信号量等到A打印之后才开始打印, 初始为0
    private static Semaphore B = new Semaphore(0);
    private static Semaphore C = new Semaphore(0);

    public static void main(String[] args) {
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    // A获取信号执行，A信号量减1，此时A为0无法继续打印
                    A.acquire();
                    System.out.print("A ");
                    // B释放信号，B信号量加1（初始为0），此时可以获取B信号量进行打印
                    B.release();
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    B.acquire();
                    System.out.print("B ");
                    C.release();
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    C.acquire();
                    System.out.print("C ");
                    A.release();
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

# 四、异常

## 1.异常分类

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： **Error** 和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

- RuntimeException：不可预知的，程序应该尽量避免
- 非RuntimeException：可预知的，由编译器检查的异常

<img src="E:\Files\Notes\images\px4VINsj0HaJMCCS2pH8pcCpbceGdx1E.png" alt="img" style="zoom: 40%;" />

## 2.常见异常

### Error

- OutOfMemoryError 内存溢出错误
- StackOverflowError 栈溢出错误

### RuntimeException

- NullPointerException 空指针引用异常
- IndexOutOfBoundsException 下标越界异常
- IllegalArgumentException 传递非法参数异常

### 非RuntimeException

- IOException IO操作异常
- ClassNotFoundException 找不到指定Class异常

# 五、 I/O

## 1. Java I/O 分类

Java 的 I/O 大概可以分成以下几类：

- 磁盘操作：File
- 字节操作：InputStream 和 OutputStream
- 字符操作：Reader 和 Writer
- 对象操作：Serializable
- 网络操作：Socket
- 新的输入/输出：NIO

## 2. 序列化

TODO


## 3.  I/O 模型

### BIO

BIO (Blocking I/O)，即同步阻塞I/O，InputStream、OutputStream、Reader 和 Writer采用BIO模型。

<img src=".\images\BehexAiyYIgwwFRs.png" alt="image-20200502213736783" style="zoom:50%;" />

### NIO

新的输入/输出 (NIO) 库是在 JDK 1.4 中引入的，弥补了原来的 I/O 的不足，提供了高速的、**面向块**、**多路复用**、**同步非阻塞**的I/O操作。

<img src=".\images\2HEPAHVMyjGn79UL.png" alt="image-20200502213511378" style="zoom:50%;" />

NIO由通道(Channel)、缓冲区(Buffer)和选择器(Selector)构成。

#### 通道 (Channel)

通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。

通道包括以下类型：

- FileChannel：从文件中读写数据
- DatagramChannel：通过 UDP 读写网络中数据
- SocketChannel：通过 TCP 读写网络中数据
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel

<img src=".\images\X9YOyic3NBH7OXDQ.png" alt="image-20200503144746966" style="zoom:50%;" />

#### 缓冲区 (Buffer)

发送给一个通道的所有数据都必须首先放到缓冲区中；同样地，从通道中读取的任何数据都要先读到缓冲区中。

缓冲区实质上是一个数组，提供了对数据的结构化访问，还可以跟踪系统的读/写进程。

缓冲区包括以下类型：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

#### 选择器 (Selector)

NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

<img src=".\images\oG3blPzZsiDOrkxM.png" alt="image-20200503142732684" style="zoom:50%;"/>

### AIO

基于事件和回调机制。

<img src=".\images\SYd2N3h4gTYSsu2u.png" alt="image-20200502214046197" style="zoom:50%;" />

### BIO、NIO、AIO区别

|                              | BIO        | NIO          | AIO          |
| ---------------------------- | ---------- | ------------ | ------------ |
| 是否阻塞与同步               | 阻塞、同步 | 非阻塞、同步 | 非阻塞、异步 |
| 服务线程数 (服务端 : 客户端) | 1 : 1      | 1 : N        | 0 : N        |
| 复杂度                       | 简单       | 较复杂       | 复杂         |
| 吞吐量                       | 低         | 高           | 高           |

## 4. I/O多路复用

使用一个或少量的线程来处理多个网络I/O，select/poll/epoll 都是 I/O 多路复用的具体实现。

<img src="E:\Files\Notes\images\R1uUjv97iePe05RE.png" alt="image-20200502213921595" style="zoom:50%;" />

|        | 一个进程支持的最大连接数               | 消息传递方式                                   | 效率                                                         | 应用场景                                                     |
| ------ | -------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| select | 由FD_SETSIZE宏确定，32位机器上是32*32  | 内核需要将数据传递到用户空间                   | 每次调用会对连接进行线性遍历，FD剧增会造成`线性下降`的性能问题 | 可移植性更好，实时性好                                       |
| poll   | 无限制                                 | 同上                                           | 同上                                                         | 如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select |
| epoll  | 连接数有限，但1G内存可以支持10万个连接 | 通过内核和用户空间共享一块内存来实现，性能较高 | 只有活跃可用的FD才会调用callback函数，效率高                 | 只需要运行在 Linux 平台上，有大量的描述符需要同时轮询，并且这些连接最好是长连接 |