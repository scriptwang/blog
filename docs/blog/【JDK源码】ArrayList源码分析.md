ArrayList是应用最广泛的线性表之一

特点：
- 线程非安全（安全版本的ArrayList是Vector）
- 容量无限（最大为Integer.MAX_VALUE），动态扩容
- 逻辑上内存分配是连续的，有数据下标，取数据时间复杂度为O(1)，添加数据有可能会导致扩容，时间复杂度<=O(n)
底层采用的是System.arraycopy的本地方法


# 成员变量
```java
//默认容量为10个
private static final int DEFAULT_CAPACITY = 10;
//空对象
private static final Object[] EMPTY_ELEMENTDATA = {};
//空对象
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//底层数据容器是Object数组
transient Object[] elementData;
//容器大小
private int size;
```
# 构造函数
```java
//给定一个初始容量，如果知道数组大概长度的情况下可用此构造函数，
//避免ArrayList频繁的扩容带来的性能损耗
public ArrayList(int initialCapacity) {
	if (initialCapacity > 0) {
		this.elementData = new Object[initialCapacity];
	} else if (initialCapacity == 0) {
		this.elementData = EMPTY_ELEMENTDATA;
	} else {
		throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
	}
}
//默认无参构造函数
public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
//用另外的容器来初始化
public ArrayList(Collection<? extends E> c) {
	elementData = c.toArray();
	if ((size = elementData.length) != 0) {
	    // c.toArray might (incorrectly) not return Object[] (see 6260652)
	    //这里需要注意一下：c.toArray有可能返回的不是Object[]，此时用Arrays.copyOf将数组中的元素再拷贝一遍
		//比如Mylist extends ArrayList，然后重写toArray返回非Object[]
		//具体原因看这里https://blog.csdn.net/GuLu_GuLu_jp/article/details/51457492
		if (elementData.getClass() != Object[].class)
			elementData = Arrays.copyOf(elementData, size, Object[].class);
	} else {
		// 如果给一个空容器则用空对象初始化
		this.elementData = EMPTY_ELEMENTDATA;
	}
}
```
# 添加元素
## 尾部追加
```public boolean add(E e)```其实就类似于append，在尾部追加
```java
//add 方法看似简单，只有三句话，其实每次都会去检查数组是否需要扩容
public boolean add(E e) {
	//确定是否需要扩容，因为新的元素进来了，所以需要size + 1
	//相当于确定在满足需求的容量下是否需要扩容
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	//赋值元素
	elementData[size++] = e;
	return true;
}

private void ensureCapacityInternal(int minCapacity) {
	//如果是第一次添加元素，设置minCapacity为默认的容量10（DEFAULT_CAPACITY）
	//minCapacity（大于默认容量10时候）可以理解为要装下所有元素（包括即将要装进来的）所需要的容量
	if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
		minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
	}
	ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
	modCount++;
	//如果装下所有元素需要的容量大于了底层数组的长度
	//也就是说底层数据装不下新元素了，那么就需要扩容
	if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}

private void grow(int minCapacity) {
	int oldCapacity = elementData.length;
	//oldCapacity >> 1 表示oldCapacity右移一位相当于oldCapacity除以2^1
	//所以每次扩容都是在原来容量的1.5倍
	int newCapacity = oldCapacity + (oldCapacity >> 1); 
	//首次扩容时，会走这个条件，相当于扩容到10
	if (newCapacity - minCapacity < 0)
		newCapacity = minCapacity;
	//判断是否超过最大限制
	//MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8，为什么要减8呢，可以看看这里
	//https://stackoverflow.com/questions/35756277/why-the-maximum-array-size-of-arraylist-is-integer-max-value-8
	if (newCapacity - MAX_ARRAY_SIZE > 0)
		newCapacity = hugeCapacity(minCapacity);
	//扩容操作，底层调用的是System.arraycopy，native方法，内存操作，效率高
	//此时会new一个新的数组，并将原来数组里面的元素挨个拷贝过去
	//所以说如果经常扩容会影响效率
	elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
	//这里有个疑惑的地方，从逻辑来看minCapacity不可能会小余0
	//而且jdk源码注释这里是overflow，这里要怎么理解？
	if (minCapacity < 0) // overflow
		throw new OutOfMemoryError();
	return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```
## 某位置上添加
```public void add(int index, E element)```频繁的调用不是明智之举，因为每往数组中间添加一个元素，都会把数组掰成两半，然后在中间放入元素，会影响该元素后的所有元素（内存表现为往右边移动一个位置）
```java
public void add(int index, E element) {
	//检查索引是否超过范围
	rangeCheckForAdd(index);
	//扩容操作，和上面一样
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	//核心代码，将index位置后面的数据挨个"顺移"，空出索引位置再将元素赋值给当前索引
	//频繁的"顺移"不是明智之举，这也是ArrayList"写慢"的原因
	System.arraycopy(elementData, index, elementData, index + 1,size - index);
	elementData[index] = element;
	size++;
}
//检查索引范围，没什么好说的
private void rangeCheckForAdd(int index) {
	if (index > size || index < 0)
		throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```


#令人疑惑的minCapacity < 0
逻辑上来看minCapacity是不可能小余0的，但是忽略了一种情况，ArrayList的size是用int类型来表示，如果把两个很大的ArrayList通过addAll合并成一个，size超出了int的范围，则有可能导致minCapacity小余0，抛出outOfMemory错误
参考：https://stackoverflow.com/questions/8557832/cant-understand-the-possibility-of-overflow-when-resizing-arraylist-in-java
```java
private static int hugeCapacity(int minCapacity) {
	//令人疑惑的代码
	if (minCapacity < 0) // overflow
		throw new OutOfMemoryError();
	return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}

//看一下addAll代码你就明白了
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
	//新容器元素的数量
    int numNew = a.length;
	//当size和numNew的和超过了int表示的范围但是又强转int所以会出现负数
	//可以试一下：System.out.println(Integer.MAX_VALUE + 1);
    ensureCapacityInternal(size + numNew);
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```
# 包含元素的方法contains
```java
public boolean contains(Object o) {
	return indexOf(o) >= 0;
}
public int indexOf(Object o) {
    //判断元素是否为null,分别进行处理
	if (o == null) {
		for (int i = 0; i < size; i++)
			if (elementData[i]==null)
				return i;
	} else {
		for (int i = 0; i < size; i++)
			if (o.equals(elementData[i]))
				return i;
	}
	return -1;
}
```
# 删除元素的方法
有两种，一种是移除指定位置上的元素，一种是移除指定内容（Object）元素
```java
//移除指定位置(索引)上的元素
public E remove(int index) {
	//检查索引是否越界
	rangeCheck(index);
	//更改"被修改"次数
	modCount++;
	//拿出数据用于返回
	E oldValue = elementData(index);
	//该位置后面还有多少个元素（将要被移动的元素）
	int numMoved = size - index - 1;
	if (numMoved > 0)
		//从被移除元素的下一个位置开始，将后面剩下的所有元素往前挪一个位置，
		System.arraycopy(elementData, index+1, elementData, index,numMoved);//五个参数分别是src srcPos dest destPos length
	//将最后一个位置上的元素置空	
	elementData[--size] = null; // clear to let GC do its work
	return oldValue;
}
//移除指定元素
public boolean remove(Object o) {
    //null和非null分开处理
	if (o == null) {
	    //找到最近的一个元素开始移除
		for (int index = 0; index < size; index++)
			if (elementData[index] == null) {
				fastRemove(index);
				return true;
			}
	} else {
		for (int index = 0; index < size; index++)
			if (o.equals(elementData[index])) {
				fastRemove(index);
				return true;
			}
	}
	return false;
}
//fastRemove和remove相比只是少了边界检查rangeCheck
private void fastRemove(int index) {
	modCount++;
	int numMoved = size - index - 1;
	if (numMoved > 0)
		System.arraycopy(elementData, index+1, elementData, index,numMoved);
	elementData[--size] = null; // clear to let GC do its work
}
```
# 关于modCount
很多地方都看到对modCount的修改，比如扩容操作ensureExplicitCapacity，或者移除元素remove的时候，modCount意味结构性修改，比如修改列表的长度，这个用来检查列表是否被修改过，因为ArrayList是非线程安全的，所以通过modCount来确定列表是否被修改，如果被意外的修改，则会抛出ConcurrentModificationException