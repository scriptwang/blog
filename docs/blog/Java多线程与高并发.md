# CAS
Compare And Swap (Compare And Exchange) / 自旋 / 自旋锁 / 无锁 

因为经常配合循环操作，直到完成为止，所以泛指一类操作

cas(v, a, b) ，变量v，期待值a, 修改值b

ABA问题，你的女朋友在离开你的这段儿时间经历了别的人，自旋就是你空转等待，一直等到她接纳你为止

解决办法（版本号 AtomicStampedReference），基础类型简单值不需要版本号

# Unsafe

参考：[jdk1.8 Unsafe类初探](https://blog.csdn.net/a7980718/java/article/details/83279728)

AtomicInteger:

```java
public final int incrementAndGet() {
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return next;
        }
    }

public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

Unsafe:

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

运用：也可以参考[TestUnsafe](./参考代码/TestUnsafe.java)

```java
package com.mashibing.jol;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

public class T02_TestUnsafe {

    int i = 0;
    private static T02_TestUnsafe t = new T02_TestUnsafe();

    public static void main(String[] args) throws Exception {
        //Unsafe unsafe = Unsafe.getUnsafe();

        /**
         * 为什么不直接通过Unsafe unsafe = Unsafe.getUnsafe();拿到unsafe实例 ?
         * 因为直接拿要抛异常，不信你可以试试看，原因在于直接拿Unsafe会检查当前类加载器是不是Bootstrap加载器
         * 如果不是就抛异常，当前类加载器是AppClassLoader，当然不是Bootstrap ClassLoader，
         * 也就是说，Unsafe只允许JVM的某些系统来拿，但是你非要用，也可以自己通过下面的骚操作拿
         * 参考：https://blog.csdn.net/a7980718/article/details/83279728
         */
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);

        Field f = T02_TestUnsafe.class.getDeclaredField("i");
        long offset = unsafe.objectFieldOffset(f);
        System.out.println(offset);

        boolean success = unsafe.compareAndSwapInt(t, offset, 0, 1);
        System.out.println(success);
        System.out.println(t.i);
        //unsafe.compareAndSwapInt()
    }
}
```

jdk8u: unsafe.cpp:

cmpxchg = compare and exchange

```c++
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```

jdk8u: atomic_linux_x86.inline.hpp

is_MP = Multi Processor  

```c++
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```

jdk8u: os.hpp is_MP()

```c++
  static inline bool is_MP() {
    // During bootstrap if _processor_count is not yet initialized
    // we claim to be MP as that is safest. If any platform has a
    // stub generator that might be triggered in this phase and for
    // which being declared MP when in fact not, is a problem - then
    // the bootstrap routine for the stub generator needs to check
    // the processor count directly and leave the bootstrap routine
    // in place until called after initialization has ocurred.
    return (_processor_count != 1) || AssumeMP;
  }
```

jdk8u: atomic_linux_x86.inline.hpp

```c++
// Adding a lock prefix to an instruction on MP machine
#define LOCK_IF_MP(mp) "cmp $0, " #mp "; je 1f; lock; 1: "
```

```LOCK_IF_MP```表示程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在**多处理器**上运行，就为cmpxchg指令加上lock前缀（```lock cmpxchg```）。反之，如果程序是在单处理器上运行，就省略`lock`前缀（单处理器自身会维护单处理器内的顺序一致性，不需要`lock`前缀提供的内存屏障效果）。

硬件：

`lock`指令在执行后面指令的时候锁定一个北桥信号

`lock`指令的硬件实现参考[【CPU】Lock前缀](./参考文章/【CPU】Lock前缀.md)



# 工具：JOL = Java Object Layout

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
    <dependency>
        <groupId>org.openjdk.jol</groupId>
        <artifactId>jol-core</artifactId>
        <version>0.9</version>
    </dependency>
</dependencies>
```

打印对象布局
```java
Object o = new Object();
System.out.println(ClassLayout.parseInstance(o).toPrintable());
```

 

jdk8u: markOop.hpp

```java
// Bit-format of an object header (most significant first, big endian layout below):
//
//  32 bits:
//  --------
//             hash:25 ------------>| age:4    biased_lock:1 lock:2 (normal object)
//             JavaThread*:23 epoch:2 age:4    biased_lock:1 lock:2 (biased object)
//             size:32 ------------------------------------------>| (CMS free block)
//             PromotedObject*:29 ---------->| promo_bits:3 ----->| (CMS promoted object)
//
//  64 bits:
//  --------
//  unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object)
//  JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object)
//  PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object)
//  size:64 ----------------------------------------------------->| (CMS free block)
//
//  unused:25 hash:31 -->| cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && normal object)
//  JavaThread*:54 epoch:2 cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && biased object)
//  narrowOop:32 unused:24 cms_free:1 unused:4 promo_bits:3 ----->| (COOPs && CMS promoted object)
//  unused:21 size:35 -->| cms_free:1 unused:7 ------------------>| (COOPs && CMS free block)
```

# Java对象在内存中的布局
java命令行默认参数查看
```
java -XX:+PrintCommandLineFlags -version


-XX:InitialHeapSize=259117760(起始堆大小)
-XX:MaxHeapSize=4145884160(最大堆大小) 
-XX:+PrintCommandLineFlags(打印默认参数) 
-XX:+UseCompressedClassPointers(启用压缩指针，64位指针长度默认为8个字节，开启了压缩指针，指针就默认是4个字节) 
-XX:+UseCompressedOops(普通对象指针，对象里面的指针，默认是压缩的，也是4个字节)
-XX:-UseLargePagesIndividualAllocation 
-XX:+UseParallelGC
java version "1.8.0_73"
Java(TM) SE Runtime Environment (build 1.8.0_73-b02)
Java HotSpot(TM) 64-Bit Server VM (build 25.73-b02, mixed mode)
```

- 64bit系统中的情况：

开启压缩时
new Object()占 8 + 4 + 0 + 4 = 16字节

不开压缩时
new Object()占 8 + 8 + 0 + 0 = 16字节

```
          ---------> markword       (8个字节)对象头，记录关于锁的信息        
          |--------> class pointer  (4个字节或者8个字节) 类型指针，记录此对象是哪个类实例化的
new Object()         
          |--------> instance data  (视具体对象成员变量而言) 实例数据，存储成员变量
          ---------> padding        (视具体情况而言)对齐，当对象占用字节数不能被8整除时，对齐到能被8整除，达到快速读取的目的
```

- 32bit系统中的情况：

  32bit系统中markword占用32bit，也就是4字节，class pointer也是4字节，所以

  new Object()占 8 + 4 + 0 + 0 = 8字节

# synchronized的横切面详解

1. 升级过程

   **new - 偏向锁 - 轻量级锁 （无锁，自旋锁，自适应自旋）- 重量级锁**

2. synchronized原理

   **自旋锁的底层是CAS**

   **重量级锁的底层是mutex（操作系统互斥量）**

3. 汇编实现

4. vs reentrantLock的区别

## java源码层级

synchronized(o) 

## 字节码层级

monitorenter

moniterexit

## JVM层级（Hotspot）

```java
package com.mashibing.insidesync;

import org.openjdk.jol.info.ClassLayout;

public class T01_Sync1 {
  

    public static void main(String[] args) {
        Object o = new Object();

        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}
```

```java
com.mashibing.insidesync.T01_Sync1$Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4   (object header)  05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4   (object header)  00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4   (object header)  49 ce 00 20 (01001001 11001110 00000000 00100000) (536923721)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

```java
com.mashibing.insidesync.T02_Sync2$Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4   (object header)  05 90 2e 1e (00000101 10010000 00101110 00011110) (506368005)
      4     4   (object header)  1b 02 00 00 (00011011 00000010 00000000 00000000) (539)
      8     4   (object header)  49 ce 00 20 (01001001 11001110 00000000 00100000) (536923721)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes tota
```

InterpreterRuntime:: monitorenter方法

```c++
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  if (PrintBiasedLockingStatistics) {
    Atomic::inc(BiasedLocking::slow_path_entry_count_addr());
  }
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (UseBiasedLocking) {
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
  assert(Universe::heap()->is_in_reserved_or_null(elem->obj()),
         "must be NULL or an object");
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END
```

synchronizer.cpp

revoke_and_rebias

```c++
void ObjectSynchronizer::fast_enter(Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS) {
 if (UseBiasedLocking) {
    if (!SafepointSynchronize::is_at_safepoint()) {
      BiasedLocking::Condition cond = BiasedLocking::revoke_and_rebias(obj, attempt_rebias, THREAD);
      if (cond == BiasedLocking::BIAS_REVOKED_AND_REBIASED) {
        return;
      }
    } else {
      assert(!attempt_rebias, "can not rebias toward VM thread");
      BiasedLocking::revoke_at_safepoint(obj);
    }
    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
 }

 slow_enter (obj, lock, THREAD) ;
}
```

```c++
void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
  assert(!mark->has_bias_pattern(), "should not see bias pattern here");

  if (mark->is_neutral()) {
    // Anticipate successful CAS -- the ST of the displaced mark must
    // be visible <= the ST performed by the CAS.
    lock->set_displaced_header(mark);
    if (mark == (markOop) Atomic::cmpxchg_ptr(lock, obj()->mark_addr(), mark)) {
      TEVENT (slow_enter: release stacklock) ;
      return ;
    }
    // Fall through to inflate() ...
  } else
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
    assert(lock != mark->locker(), "must not re-lock the same lock");
    assert(lock != (BasicLock*)obj->mark(), "don't relock with same BasicLock");
    lock->set_displaced_header(NULL);
    return;
  }

#if 0
  // The following optimization isn't particularly useful.
  if (mark->has_monitor() && mark->monitor()->is_entered(THREAD)) {
    lock->set_displaced_header (NULL) ;
    return ;
  }
#endif

  // The object header will never be displaced to this lock,
  // so it does not matter what the value is, except that it
  // must be non-zero to avoid looking like a re-entrant lock,
  // and must not look locked either.
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
}
```

inflate方法：膨胀为重量级锁



# 锁升级过程



## JDK8 markword实现表



锁升级的过程：

**new - 偏向锁 - 轻量级锁 （无锁，自旋锁，自适应自旋）- 重量级锁**

synchronized优化的过程和markword息息相关，用markword中**低**的三位代表锁状态 其中1位是偏向锁位 两位是普通锁位

![markword](./file/markword.jpg)

分代年龄：一个对象经过一次GC之后，它的年龄会+1，当年龄到达一定程度后，它就会从年轻代升级到老年代，分代年龄用4个bit表示，所以最大就是0~15

上面是64bit系统下的markword，占8字节，下面是32bit系统下的，占4字节

![markword](./file/markword32.jpg)

### **new/无锁态**

```java
Object o = new Object();
System.out.println(ClassLayout.parseInstance(o).toPrintable());
```


001 无锁态 

```java
00000001 00000000 00000000 00000000
00000000 00000000 00000000 00000000
```

little endian 转换 big endian

```java
00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
```

- 调用hashCode之后(001 + hashcode)

```java
Object o = new Object();
O.hashCode();
System.out.println(ClassLayout.parseInstance(o).toPrintable());
```

```java
00000001 10101101 00110100 00110110
01011001 00000000 00000000 00000000
```

little endian 转换 big endian 

```java
00000000 00000000 00000000 01011001 00110110 00110100 10101101 00000001
```

### 偏向锁

偏向锁 - markword 上记录当前线程指针，下次同一个线程加锁的时候，不需要争用，只需要判断线程指针是否同一个，所以，偏向锁，偏向加锁的第一个线程 。hashCode备份在线程栈上 线程销毁，锁降级为无锁。

- 偏向锁延迟：默认情况偏向锁有个时延，默认是4秒，**why? 因为JVM虚拟机自己有一些默认启动的线程，里面有好多sync代码，这些sync代码启动时就知道肯定会有竞争，如果使用偏向锁，就会造成偏向锁不断的进行锁撤销和锁升级的操作，效率较低。**

```shell
-XX:BiasedLockingStartupDelay=0 # 可以通过这个参数关闭偏向锁延迟
```

- 匿名偏向锁：如果设定上述参数，new Object () - > 101 偏向锁 ->线程ID为0 -> Anonymous BiasedLock (匿名偏向锁)，打开偏向锁，new出来的对象，默认就是一个可偏向匿名对象101

- 效率：偏向锁由于有锁撤销的过程revoke，会消耗系统资源，所以，在**锁争用特别激烈的时候，用偏向锁未必效率高**。还不如直接使用轻量级锁。

### 轻量锁/自旋锁/无锁

有争用 - 锁升级为轻量级锁 - 每个线程有自己的LockRecord在自己的线程栈上，用CAS去争用markword的LR的指针，指针指向哪个线程的LR，哪个线程就拥有锁。

- 适应性自旋：自旋锁在 JDK1.4.2 中引入，使用 -XX:+UseSpinning 来开启。JDK 6 中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。

  自适应自旋锁意味着自旋的时间（次数）不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

- 如果太多线程自旋 CPU消耗过大，不如升级为重量级锁，进入等待队列（不消耗CPU）

### 重量锁

竞争加剧：有线程超过10次自旋，( -XX:PreBlockSpin可设置自旋次数，一般不建议设置)， 或者自旋线程数超过CPU核数的一半， **1.7及之后，加入自适应自旋 Adapative Self Spinning** ，自旋参数被取消， JVM自己控制

升级重量级锁：-> 向操作系统申请资源，linux mutex (互斥的数据结构), CPU从3级-0级**系统调用**，线程挂起，进入**等待队列**，等待操作系统的调度，然后再**映射回用户空间**

## synchronized最底层实现

```java

public class T {
    static volatile int i = 0;
    
    public static void n() { i++; }
    
    public static synchronized void m() {}
    
    publics static void main(String[] args) {
        for(int j=0; j<1000000; j++) {
            m();
            n();
        }
    }
}

```

java -XX:+UnlockDiagonositicVMOptions -XX:+PrintAssembly T

C1 Compile Level 1 (一级优化)

C2 Compile Level 2 (二级优化)

找到m() n()方法的汇编码，会看到 **lock cmpxchg** .....指令

## synchronized vs Lock (CAS)

```
 在高争用 高耗时的环境下synchronized效率更高
 在低争用 低耗时的环境下CAS效率更高
 synchronized到重量级之后是等待队列（不消耗CPU）
 CAS（等待期间消耗CPU）
 
 一切以实测为准
```



# 锁消除 lock eliminate

```java
public void add(String str1,String str2){
         StringBuffer sb = new StringBuffer();
         sb.append(str1).append(str2);
}
```

我们都知道 StringBuffer 是线程安全的，因为它的关键方法都是被 synchronized 修饰过的，但我们看上面这段代码，我们会发现，sb 这个引用只会在 add 方法中使用，**不可能被其它线程引用（因为是局部变量，栈私有）**，因此 sb 是不可能共享的资源，JVM 会自动消除 StringBuffer 对象内部的锁。

# 锁粗化 lock coarsening

```java
public String test(String str){
       
       int i = 0;
       StringBuffer sb = new StringBuffer():
       while(i < 100){
           sb.append(str);
           i++;
       }
       return sb.toString():
}
```

JVM 会检测到这样一连串的操作都对同一个对象加锁（while 循环内 100 次执行 append，没有锁粗化的就要进行 100  次加锁/解锁），此时 JVM 就会将加锁的范围粗化到这一连串的操作的外部（比如 while 循环体外），使得这一连串操作只需要加一次锁即可。

# 锁降级（不重要）

https://www.zhihu.com/question/63859501

其实，只被VMThread访问，降级也就没啥意义了。所以可以简单认为锁降级不存在！

# 超线程

一个ALU + 两组Registers + PC

所谓四核八线程，线程是CPU执行的	基本单位，进程是CPU分配资源的基本单位

# 参考资料

http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html

# volatile的用途

## 1.计算机基本知识
- 计算机组成
![](./file/计算机组成.jpg)
- 存储器的层次结构
![](./file/存储器的层次结构.jpg)
- 多核CPU
![](./file/多核CPU.jpg)


## 2.线程可见性

也叫保证CPU之间的可见性 

```java
package com.mashibing.testvolatile;

public class T01_ThreadVisibility {
    //去掉volatile，System.out.println("end")这句话永远也不会执行
    private static volatile boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(()-> {
            while (flag) {
                //do sth
            }
            System.out.println("end");
        }, "server").start();


        Thread.sleep(1000);

        flag = false;
    }
}
```

1. 如果flag不加volatile：**线程将优先读取CPU寄存器(缓存)中的变量，而不是直接去主内存中读取**

线程"server"首先将flag读取到自己的**线程本地内存**，发现是true，执行while循环，此时主线程将flag读取到自己的**线程本地内存**，将flag该成false，此时线程"server"依旧是读取自己的**线程本地内存**flag在那执行while循环

2. 如果加上volatile：**线程每次都将直接去主内存进行操作**（单次读，单次写是原子性的）

线程"server"首先将flag读取到自己的**线程本地内存**，发现是true，执行while循环，此时主线程将flag读取到自己的**线程本地内存**，将flag该成false，并且立即更新到**主内存**中，线程"server"从**主内存**中读取flag为false，于是停止while循环





## 3.防止指令重排序



### CPU的基础知识

- 缓存行对齐

  ![](./file/缓存行对齐.jpg)

- 缓存行**64个字节**是CPU同步的基本单位，缓存行隔离会比伪共享效率要高，java 消息队列框架Disruptor就用到了缓存行对齐（全世界最快的单机mq）

  ```java
  package com.mashibing.juc.c_028_FalseSharing;
  
  public class T02_CacheLinePadding {
      
      private static class Padding {
          //加上下面这一行方可对齐缓存行，7个long类型56字节，机上T的x一共64字节，刚好占一个缓存行
          //不加下面这段话执行需要250ms左右，加上下面这句话执行60ms左右
          public volatile long p1, p2, p3, p4, p5, p6, p7;
      }
  
      private static class T extends Padding {
          public volatile long x = 0L;
      }
  
      public static T[] arr = new T[2];
  
      static {
          //此处CPU会一次性读取进行处理
          arr[0] = new T();
          arr[1] = new T();
      }
  
      public static void main(String[] args) throws Exception {
          Thread t1 = new Thread(()->{
              for (long i = 0; i < 1000_0000L; i++) {
                  arr[0].x = i;
              }
          });
  
          Thread t2 = new Thread(()->{
              for (long i = 0; i < 1000_0000L; i++) {
                  arr[1].x = i;
              }
          });
  
          final long start = System.nanoTime();
          t1.start();
          t2.start();
          t1.join();
          t2.join();
          System.out.println((System.nanoTime() - start)/100_0000);
      }
  }
  
  ```

  **缓存行没对齐**：不加```public volatile long p1, p2, p3, p4, p5, p6, p7;```缓存行没有对齐，那么第一个线程t1不断的改arr[0]，每改一次要通知其他CPU arr[0]已经更改，让线程t2去**主内存**里面取值（因为加了volatile），线程t2不断改arr[1]，要通知其他CPU，让线程t1去主内存取值，来回通知很耗时。

  **缓存对齐了**：加上```public volatile long p1, p2, p3, p4, p5, p6, p7;```缓存行对齐了，那么第一个线程t1不断的改arr[0]，**不用通知其他CPU**，线程t2不断改arr[1]，**也不用通知其他CPU**，这样就提高效率了

  

  

- MESI

  CPU核之间的**缓存一致性协议**（ Modified/Exclusive/Shared/Invalid），有些无法被缓存的数据或者跨越多个缓存行的数据依然必须使用总线锁

- 伪共享

- 合并写
  CPU内部的4个字节的Buffer

  ```java
  package com.mashibing.juc.c_029_WriteCombining;
  
  public final class WriteCombining {
  
      private static final int ITERATIONS = Integer.MAX_VALUE;
      private static final int ITEMS = 1 << 24;
      private static final int MASK = ITEMS - 1;
  
      private static final byte[] arrayA = new byte[ITEMS];
      private static final byte[] arrayB = new byte[ITEMS];
      private static final byte[] arrayC = new byte[ITEMS];
      private static final byte[] arrayD = new byte[ITEMS];
      private static final byte[] arrayE = new byte[ITEMS];
      private static final byte[] arrayF = new byte[ITEMS];
  
      public static void main(final String[] args) {
  
          for (int i = 1; i <= 3; i++) {
              System.out.println(i + " SingleLoop duration (ns) = " + runCaseOne());
              System.out.println(i + " SplitLoop  duration (ns) = " + runCaseTwo());
          }
      }
  
      public static long runCaseOne() {
          long start = System.nanoTime();
          int i = ITERATIONS;
  
          while (--i != 0) {
              int slot = i & MASK;
              byte b = (byte) i;
              arrayA[slot] = b;
              arrayB[slot] = b;
              arrayC[slot] = b;
              arrayD[slot] = b;
              arrayE[slot] = b;
              arrayF[slot] = b;
          }
          return System.nanoTime() - start;
      }
  
      public static long runCaseTwo() {
          long start = System.nanoTime();
          int i = ITERATIONS;
          while (--i != 0) {
              int slot = i & MASK;
              byte b = (byte) i;
              arrayA[slot] = b;
              arrayB[slot] = b;
              arrayC[slot] = b;
          }
          i = ITERATIONS;
          while (--i != 0) {
              int slot = i & MASK;
              byte b = (byte) i;
              arrayD[slot] = b;
              arrayE[slot] = b;
              arrayF[slot] = b;
          }
          return System.nanoTime() - start;
      }
  }
  
  ```

  

- 指令重排序（乱序执行）
![](./file/乱序执行.jpg)

  ```java
  package com.mashibing.jvm.c3_jmm;
  
  public class T04_Disorder {
      private static int x = 0, y = 0;
      private static int a = 0, b =0;
  
      public static void main(String[] args) throws InterruptedException {
          int i = 0;
          for(;;) {
              i++;
              x = 0; y = 0;
              a = 0; b = 0;
              Thread one = new Thread(new Runnable() {
                  public void run() {
                      //由于线程one先启动，下面这句话让它等一等线程two. 读着可根据自己电脑的实际性能适当调整等待时间.
                      //shortWait(100000);
                      a = 1;
                      x = b;
                  }
              });
  
              Thread other = new Thread(new Runnable() {
                  public void run() {
                      b = 1;
                      y = a;
                  }
              });
              one.start();other.start();
              one.join();other.join();
              String result = "第" + i + "次 (" + x + "," + y + "）";
              if(x == 0 && y == 0) {
                  System.err.println(result);
                  break;
              } else {
                  //System.out.println(result);
              }
          }
      }
  
  
      public static void shortWait(long interval){
          long start = System.nanoTime();
          long end;
          do{
              end = System.nanoTime();
          }while(start + interval >= end);
      }
  }
  ```

  

### 内存屏障

- JSR内存屏障规范（JVM的要求）

  1. **LoadLoad屏障**：对于这样的语句Load1;LoadLoad;Load2，在Load2及后续操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕，也就是说先执行读取操作Load1，在执行Load2，不能换顺序

  2. **StoreStore屏障**：Store1;StoreStore;Store2，在Store2及后续写入操作执行前，保证Store1写入操作对其他处理器可见，就是先执行写操作Store1再执行写操作Store2

  3. **LoadStore屏障**：Load1;LoadStore;Store2，在Store2及后续写入操作被执行前，保证Load1要读取的数据被读取完毕。

  4. **StoreLoad屏障**：Store1;StoreLoad;Load2，在Load2及后续所有读取操作被执行前，保证Store1写入操作对其他处理器可见。

- JVM层面

  StoreStoreBarrier                                           LoadLoadBarrier

  volatile 写操作                                                 volatile 读操作

  StoreLoadBarrier                                            LoadStoreBarrier



### 系统底层如何实现数据一致性

1. MESI如果能解决，就使用MESI
2. 如果不能，就锁总线

### 系统底层如何保证有序性

1. 内存屏障sfence mfence lfence等系统原语（注意JVM没有用这些原语，而是自己实现的内存屏障）
2. 锁总线（最终的办法，但是效率不如MESI高）

### volatile如何解决指令重排序

1: volatile i     （源码层级）

2: ACC_VOLATILE  (字节码层级)

3: JVM的内存屏障 (JVM层级)

​	屏障两边的指令不可以重排！保证有序性！

4:hotspot实现 （lock指令，锁总线）

bytecodeinterpreter.cpp

```c++
int field_offset = cache->f2_as_index();
if (cache->is_volatile()) {//如果该cache是volatile
    if (support_IRIW_for_not_multiple_copy_atomic_cpu) {
        OrderAccess::fence();
    } 
```

orderaccess_linux_x86.inline.hpp

lock指令，JVM底层并没有用内存屏障的系统原语，而是用的lock指令，lock addl，锁定空，也就是锁总线的意思,下面源码注释也说了：always use locked addl since mfence is sometimes expensive。

```c++
inline void OrderAccess::fence() {
  if (os::is_MP()) {
    // always use locked addl since mfence is sometimes expensive
#ifdef AMD64
    __asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");
#else
    __asm__ volatile ("lock; addl $0,0(%%esp)" : : : "cc", "memory");
#endif
  }
}
```

### DCL单例需不需要加volatile？

DCL 就是 Double Check Lock (双重检查锁定)

要加

```java
public class Single {
    private static volatile Single INSTANCE;//必须加volatile
    private Single(){}//不允许new
    
    public static Single getInstance(){
        //省略业务代码
        
        if (INSTANCE == null){
            synchronized (Single.class){
                if (INSTANCE == null){// Double Check Lock
                    INSTANCE = new Single();
                }
            }
        }
        return INSTANCE;
    }
}

```

new 一个对象的过程其实分为了很多步骤

![](./file/对象创建过程.jpg)

Java new一个对象由多条汇编指令构成：

0 new 按照这个对象的大小分配一块内存，此时成员变量的值都是默认值（半初始化状态）

3 dup 复制一份引用，因为下面那条指令会消耗一个引用

4 invokespeicial <T.<init>> 此时才调用对象初始化方法，给成员变量赋值

7 astore_1 栈上引用指向堆上内存

8 return

如果不加```volatile```，在多线程环境下，如果线程A在new对象的时候，如果发生了指令重排序，先执行astore_1再执行invokespecial产生了一个**半初始化状态**的对象，此时如果线程B进入getInstance方法，判断INSTANCE不为null将直接拿到半初始化状态对象进行执行，从而产生混乱！



# Java中的引用类型——强软弱虚

## 强引用

```java
//s是强引用
String s = "aaaa";
```



最普通的引用，引用不存在了才能被GC回收



## 软引用

```java
SoftReference<byte[]> m = new SoftReference<>(new byte[1024*1024*10]);                
//能拿到
m.get()
```

特点：

- 堆内存不够用的时候自动清除软引用，被GC回收

使用场景：

- **缓存**

## 弱引用

```
WeakReference<String> m = new WeakReference<>("String");
//能拿到
m.get()
```

弱引用是被问得最多的

特点：

- **一次性**的，经过GC，直接被回收

场景：

- ThreadLocal用到了



## ThreadLocal

![](./file/ThreadLocal.jpg)

```java

//ThreadLocal set方法
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
    	map.set(this, value);
    else
        createMap(t, value);
}

//ThreadLocalMap 中的Entry是继承自WeakReference
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
    	super(k);//调用父类的构造方法，产生一个弱引用
        value = v;
    }
}
```



**线程用弱引用避免ThreadLocal产生内存泄漏**：当调用ThreadLocal.set方法时，ThreadLocal会把自己当作键，被set的值作为值放到当前线程自己的ThreadLocalMap中，ThreadLocalMap.Entry继承自弱引用，这样ThreadLocalMap中的键（也就是ThreadLocal）是一个弱引用，便于当ThreadLocal没有强引用指向时随时被回收，避免**内存泄漏**，并且，当不再用ThreadLocal时还应该调用其**remove**方法避免ThreadLocalMap.Entry.value强引用存在无法回收产生的内存泄漏



## 虚引用

``` java
//首先有一个队列
static final ReferenceQueue QUEUE = new ReferenceQueue<>();
//将虚引用放到队列里面，JVM监控此队列，当里面有东西了开始回收堆外内存
PhantomReference<String> p = new PhantomReference<>("String",QUEUE);
//p.get() 永远为true
p.get()
```

特点：

- get永远get不到，随时被回收

使用场景：

  - 管理堆外内存（背景：一些使用场景要将数据在堆外和堆内内存进行拷贝，不如直接将数据放到堆外这样效率高，NIO用到了，Netty里面Zero copy也用到了）

    

