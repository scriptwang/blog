
ConcurrentHashMap，正如它的名字一样，CoccurentHashMap是支持并发的HashMap（Concurrent adj. 并发的，一致的），建议先了解HashMap之后再了解它，ConcurrentHashMap也分JDK1.7版和JDK1.8版，两个版本的实现截然不同

# JDK1.7
- Segment+数组+链表，采用分段锁实现并发（每个Segement一把锁）
- 并发量理论上等于Segement的数量
- 用**volatile**修饰实际保存的值和指针保证内存可见行，保证多线程之间单次读取或写入的原子性

 

## 成员变量
```java
/**
 * Segment 数组，存放数据时首先需要定位到具体的 Segment 中。
 */
final Segment<K,V>[] segments;
transient Set<K> keySet;
transient Set<Map.Entry<K,V>> entrySet;

```
## Segment的定义
Segement是ReentrantLock（重入锁）的子类
```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
       private static final long serialVersionUID = 2249069246763182397L;
       
       // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
       transient volatile HashEntry<K,V>[] table;
       transient int count;
       transient int modCount;//结构化修改数量
       transient int threshold;//扩容阈值
       final float loadFactor;//负载因子
}

```

## put
```java
//根据hash找到对应的Segement
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    
    //找到该Segement后，调用该Segement的put方法    
    return s.put(key, hash, value, false);
}



//将元素put到Segement
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    //尝试获取锁，如果tryLock()返回false说明有其他线程竞争
    //则调用scanAndLockForPut自旋获取锁，并定位当前分段table中存放的位置
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
        
        
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        //遍历该table
        for (HashEntry<K,V> e = first;;) {
            //该条HashEntry不为空，则说明是一个链表，循环该链表
            if (e != null) {
                K k;
                //如果在该链表中找到一样的key，则覆盖旧值并返回旧值
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            
            //该HashEntry为空，说明是首次放入元素    
            } else {
            
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                //判断是否需要扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        //最后释放锁
        unlock();
    }
    return oldValue;
}



//scanAndLockForPut的源码：
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    //定位HashEntry数组位置，获取第一个节点
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1;//扫描次数
    
    while (!tryLock()) {//尝试自旋获取锁（所谓自旋就是不停的循环去获取锁）
        HashEntry<K,V> f;
        if (retries < 0) {
            if (e == null) {
                if (node == null) // 构造新节点
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) { //如果尝试达到了最大尝试次数，则改为获取阻塞锁
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // 首节点有变动，更新first，重新扫描
            retries = -1;
        }
    }
    return node;
}

```

## get
get方法逻辑清晰，获取数据效率高，因为整个过程都不需要加锁，而且获取的元素是通过volatile修饰的（HashEntry的value是volatile的），保证了内存的可见性，获取的都是最新的值
```java
public V get(Object key) {
    //定义元素所在的Segement
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    //定义Segement所在的table
    HashEntry<K,V>[] tab;
    
    //通过hash定位到具体的Segement，下面给s赋值就是找到具体的Segement
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        
        //在当前table中寻找该元素，(tab.length - 1) & h)表示通过hash确定元素在桶中的位置
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}

```

# JDK1.8
- 抛弃了1.7 Segement的分段锁，采用CAS和synchronized保证并发的安全性
- 数据结构采用了数组+链表的形式，和HashMap类似
- tabAt,casTabAt,setTabAt操作都是volatile的，保证线程之前的可见性
- 写入时只要hash不冲突，就不会产生并发
- 链表元素个数大于8个转化成红黑树


## 成员变量

```java
//volatile确保了table对所有线程的可见性
transient volatile Node<K,V>[] table;
```

Node
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;   //该key的hash值，final修饰，不可变
    final K key;      //key的实际值，final修饰，不可变
    volatile V val;   //真正的值，volatile修饰，保证多线程之间的内存可见行
    volatile Node<K,V> next; //链表，volatile修饰，保证多线程之间的内存可见行
    ....
}
```
可以看到，这两个重要的成员都被 volatile 进行了修饰，使得他们在并发的情况下数据的变动对于其他线程是可以立即
感知的

- 首先第一个 table 用 volatile 修饰，确保了在扩容时，扩容后的结果对其他线程是立即可见的
- 其次， Node 节点中的 val 以及 next 熟悉被 volatile 修饰，确保下面操作在多线程之间的数据可见性
    - 对 ConcurrentHashMap 进行 put 一份数据<K, V>
    - 线程A对 Map 进行 remove 操作，线程B对 Map 进行 get 操作
    - 如果A 先于 B，由于 volatile 的特新，B无法 get 到被删除的数据

## put
```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    //循环操作直到成功
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //判断是否需要初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        
        //如果table[i]槽位上为空，则采用cas自旋锁（自旋其实就是死循环）保证插入成功
        //此处可能有有疑惑：如果两个线程同时进入了casTabAt方法，其中一个修改table[i]的值，则另外一个cat操作一直得不到预期值null，岂不是死循环？？
        //其实，这里存在着java内存模型happens-before的应用（通常情况下，写操作必须要happens-before读操作）
        //tabAt是volatile读，casTabAt具有volatile读写相同的内存语义
        //因此casTabAt所做的操作都能立即反馈会tabAt方法并且
        //jdk在这里运用了for + cas更新 实现无锁的更新操作——乐观锁，相比悲观锁性能有了一定的提升
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))
                break;// no lock when adding to empty bin
        }
        //达到扩容条件了，就准备扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //上面的条件都不满足，则锁定当前node写入数据
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    //hash大于0说明是链表
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果找到key相同的则替换掉旧值
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            //都没找到则在链表尾部添加
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) { //如果是红黑树，则按照红黑树的方式添加
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                //链表长度超过阈值8则转换成红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

## get

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        //table[(n - 1) & h]槽位上的hash值刚好等于传进来的hash值，则直接返回
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
            
        // 如果计算出来的hash值小于0，则表示该槽位挂着的是一棵红黑树，采用二叉查找树的形式搜索  
        } else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        
        // 此处则确定是链表的形式，直接遍历查找比对即可    
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```


# HashMap面试题
常见的关于HashMap和ConcurrentHashMap的面试题
## 谈谈你理解的 HashMap，讲讲其中的 get put 过程
HashMap 这样的 Key Value 在软件开发中是非常经典的结构，常用于在**内存**中存放数据，底层是数组加链表的形式
- 通过元素hashCode确定元素在数据中的位置，数组的长度总是2的幂次方（为了效率），当容量到达阈值(负载因子和数组长度之积)时会引发自动扩容，非常消耗性能
- 当hash冲突时通过拉链法解决冲突，即该数组纵向延伸成为一个链表，最坏的情况退化成单一链表，时间复杂度为O(N)
- 非线程安全
- put过程
    1. 判断数组是否初始化
    2. key如果是空，则放一个空值
    3. 计算hash得到在数组中的位置，如果该位置上有值则遍历链表是否能找到相同的key，找到则替换原来的值
    4. 如果桶是空的，或者没找到相同的key，则在当前位置上创建一个新的Entry放入（如果超过了阈值就扩容）
- get过程
    1. 如果size为0，直接返回null
    2. 计算key的hash找到在数组中的位置
    3. 遍历当前位置的链表（如果只有一个Entry那就是单个元素的链表）寻找相同的key，找到则返回
    4. 都没找到返回null

## JDK1.8 做了什么优化？
主要是优化了查询的效率
- 链表长度超过8则转换成红黑树，增加查找的效率，时间复杂度为O(logN)
- 扩容时会均匀分配元素，而JDK1.7会原封不动的拷贝过来
- 多线程会引发链表死循环的问题已解决 

## 是线程安全的嘛？
不是，HashMap是非线程安全的

## 不安全会导致哪些问题？
多个线程写HashMap时可能会导致数据不一致，特别是在JDK1.7中多线程**扩容时**还会引发死循环


## 如何解决？有没有线程安全的并发容器？
需要线程安全的HashMap可以用```Collections.synchronizedMap(new HashMap)```来增加同步锁，但是这样效率比较低，所以增加了线程安全的ConcurrentHashMap


## ConcurrentHashMap 是如何实现的？ 1.7、1.8 实现有何不同？为什么这么做？
- JDK 1.7
    1. Segment+数组+链表，采用分段锁实现并发（每个Segement一把锁）
    2. 并发量理论上等于Segement的数量
    3. 用**volatile**修饰实际保存的值和指针保证内存可见行，保证多线程之间单次读取或写入的原子性
- JDK 1.8（提高性能）
    1. 抛弃了1.7 Segement的分段锁，采用CAS和synchronized保证并发的安全性
    2. 数据结构采用了数组 + 链表的形式，和HashMap类似
    3. tabAt,casTabAt,setTabAt操作都是volatile的，保证线程之前的可见性
    4. 写入时只要hash不冲突，就不会产生并发
    5. 链表元素个数大于8个转化成红黑树


