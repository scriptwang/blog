
Map 这样的 Key Value 在软件开发中是非常经典的结构，常用于在内存中存放数据。


# 总结

## JDK1.7

- 数组加链表，"拉链法"解决hash冲突
- 底层数组长度总是为**2的幂次方**。这是因为在此条件下hash & (length - 1) == hash % length，**而且&比%的效率更高**，（hash % length总是小于length的，因此可以用来计算元素在桶中的位置）
    - 默认长度16，会动态增长为32，64，128...，就算初始化的时候指定为11，其实底层的数组长度还是16
- 负载因子默认是0.75，是可以修改的，扩容阈值=数组长度*负载因子
    - 假设数组长度为16，则扩容阈值为16*0.75=12，当实际所放元素大于12时，则触发扩容操作
- 自动扩容，非常消耗性能
- 当hash严重冲突时，链表会越来越长严重影响效率，时间复杂度最长为O(N)
- 线程不安全（需要线程安全的请使用ConcurrentHashMap），多线程会引发链表死循环

## JDK1.8后的优化
- 当链表长度超过8的时候则直接转换成红黑树，查询效率为O(logN)
- 扩容时会均匀分配元素，而JDK1.7会原封不动的拷贝过来
- 多线程会引发链表死循环的问题已解决


# 测试
```java
//指定初始容量为11，但是底层数组的长度还是会初始化为16，具体看tableSizeFor方法
Map<Integer,String> map = new HashMap(11);


//初始化map
map = new HashMap<>();//初始化容量为16的HashMap
map.put(1,"A");//索引位置 1 % 16 = 1；放入第一个元素的时候会初始化map


//元素放在不同的桶中
map = new HashMap<>();//初始化容量为16的HashMap
map.put(1,"A");//索引位置 1 % 16 = 1；放入第一个元素的时候会初始化map
map.put(2,"B");//索引位置 2 % 16 = 2


//hash碰撞，产生链表
map = new HashMap<>();//初始化容量为16的HashMap
map.put(1,"A");//索引位置   1 % 16 = 1；放入第一个元素的时候会初始化map
map.put(17,"B");//索引位置 17 % 16 = 1，索引相同，hash碰撞，产生链表


//hash碰撞，产生红黑树
map = new HashMap<>();//初始化容量为16的HashMap
map.put(1,"A");   //索引位置   1 % 16 = 1；放入第一个元素的时候会初始化map
map.put(17,"B");  //索引位置  17 % 16 = 1，hash碰撞，链表长度2
map.put(33,"D");  //索引位置  33 % 16 = 1，hash碰撞，链表长度3
map.put(49,"E");  //索引位置  49 % 16 = 1，hash碰撞，链表长度4
map.put(65,"F");  //索引位置  65 % 16 = 1，hash碰撞，链表长度5
map.put(81,"G");  //索引位置  81 % 16 = 1，hash碰撞，链表长度6
map.put(97,"H");  //索引位置  97 % 16 = 1，hash碰撞，链表长度7
map.put(113,"I"); //索引位置 113 % 16 = 1，hash碰撞，链表长度8
map.put(129,"J"); //索引位置 129 % 16 = 1，hash碰撞，链表长度9，此时产生红黑树，调用treeifyBin(tab, hash)

```



# JDK1.8
## 成员变量
```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
- DEFAULT_INITIAL_CAPACITY表示初始化容量大小为2^4 = 16，可以在初始化的时候指定

```java
/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;
```
- 最大容量为2^30

```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
- 负载因子默认为0.75

```java
/**
 * The bin count threshold for using a tree rather than list for a
 * bin.  Bins are converted to trees when adding an element to a
 * bin with at least this many nodes. The value must be greater
 * than 2 and should be at least 8 to mesh with assumptions in
 * tree removal about conversion back to plain bins upon
 * shrinkage.
 */
static final int TREEIFY_THRESHOLD = 8;

```
- 当链表长度大于8的时候，链表将转换成红黑树

```java
/**
 * The table, initialized on first use, and resized as
 * necessary. When allocated, length is always a power of two.
 * (We also tolerate length zero in some operations to allow
 * bootstrapping mechanics that are currently not needed.)
 */
transient Node<K,V>[] table;

```
- table是真正存放数据的数组

```java
/**
* The number of key-value mappings contained in this map.
*/
transient int size;

```
- size表示当前map实际存放元素数量的大小

```java
/**
 * The number of times this HashMap has been structurally modified
 * Structural modifications are those that change the number of mappings in
 * the HashMap or otherwise modify its internal structure (e.g.,
 * rehash).  This field is used to make iterators on Collection-views of
 * the HashMap fail-fast.  (See ConcurrentModificationException).
 */
transient int modCount;
```
- 结构化修改次数的大小

```java
/**
 * The next size value at which to resize (capacity * load factor).
 *
 * @serial
 */
int threshold;

```
- 需要扩容的阈值，当size和threshold相等时会触发扩容操作

```java
/**
* The load factor for the hash table.
*
* @serial
*/
final float loadFactor;

```
- 负载因子，默认为DEFAULT_LOAD_FACTOR，可构造map的时候传入

## 容量capacity(C)，负载因子loadFactor(L)，扩容阈值threshold(T)和实际存放元素大小size(S)的关系
- 注意capacity变量不是成员变量，而是实际存放数据数组的长度，可以理解成table.length
- T = C * L
- 当S>T时会触发扩容操作，此时C会变成原来的2倍（当数组长度C 是2的 n 次时， hash&(length-1) == hash%length，因为&操作比%操作效率高，所以数组长度C总是2的n次方，目的是为了提升效率。）
举例:
默认大小C为16，此时T=16 * 0.75 = 12，当S=13时，触发扩容，C=C*2=16*2=32，T=32 * 0.75=24


## put方法
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    //定义tab p n i          
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    //第一次往map里面put时候会调用扩容resize方法初始化table
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;//初始化table并且将长度赋值给n
    
    //i = (n - 1) & hash 会得到当前元素放置在数组中的位置，
    //和hash % n的值相等（前提是table.length为2的幂次方），但是&的操作效率更高
    //如果该位置上面没有元素则直接新建一个元素
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
      
    //下面的操作在该元素位置上有值的时候进行操作    
    else {
        Node<K,V> e; K k;
        //如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 key、key 的 hashcode 与写入的 key 是否相等，相等就赋值给 e
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
            
        //如果当前数据为红黑树，则按照红黑树的方式写入数据    
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //循环链表
            for (int binCount = 0; ; ++binCount) {
                //循环到链表的最后一个
                if ((e = p.next) == null) {
                    //新建节点
                    p.next = newNode(hash, key, value, null);
                    //如果大于预设阈值，则转换成红黑数，接着退出循环
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                
                //如果在循环中找到了键相同的，则直接退出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        //如果找到了键相同的节点，则替换掉相应的值，并返回该值（不算结构化修改）
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //结构化修改加1
    ++modCount;
    
    //如果实际所装的元素大于了阈值，则触发扩容操作
    if (++size > threshold)
        resize();
        
    afterNodeInsertion(evict);
    return null;
}

```


## get方法
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    
    //first = tab[(n - 1) & hash] 通过(n - 1) & hash定位到该键所在桶的索引
    //如果桶为null则直接返回null 
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        
        //如果桶的第一个匹配则直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        
        //如果桶的第一个节点不匹配    
        if ((e = first.next) != null) {
            //如果节点是红黑树，则按照红黑树的方式查找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            
            //如果是链表，则按照链表的方式循环找    
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    //没找到或各种意外情况下都返回null
    return null;
}
```


# JDK1.7
JDK1.7中的put和get方法就简单很多，也没有红黑树那些转换
## put
```java
public V put(K key, V value) {
    //没有初始化的时候先初始化
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    //空key的时候
    if (key == null)
        return putForNullKey(value);
    
    //计算hash    
    int hash = hash(key);
    //计算出该hash在当前桶中的定位
    int i = indexFor(hash, table.length);
    
    //如果桶是一个链表，则循环链表
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        //如果在当前链表中找到键相同的，则替换掉原来的值
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            //返回原来的值
            return oldValue;
        }
    }
    //如果桶是空的或者没在链表中找到一样的键则新增加一个Entry
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}


void addEntry(int hash, K key, V value, int bucketIndex) {
    //如果实际的容量达到了阈值，则需要扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
        //扩容为原来的两倍
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        //在计算出该key的索引，即在桶中的位置
        bucketIndex = indexFor(hash, table.length);
    }
    //创建Entry
    createEntry(hash, key, value, bucketIndex);
}


void createEntry(int hash, K key, V value, int bucketIndex) {
    //把当前位置上的元素拿出来
    Entry<K,V> e = table[bucketIndex];
    //新建Entry，如果e不是null的话，则形成链表结构
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

## get
```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);
    return null == entry ? null : entry.getValue();
}


final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }
    //计算hashCode
    int hash = (key == null) ? 0 : hash(key);
    
    //indexFor(hash, table.length) 计算出在桶中的位置
    //如果是链表则循环找到键相同的Entry
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    
    //如果啥也没找到，则直接返回null
    return null;
}

```