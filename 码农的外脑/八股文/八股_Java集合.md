
Collection 接口

- *List* 接口
	- *ArrayList*
		- 线程不安全
		- 允许存放 null
	- *Vector*
		- 线程安全
	- *LinkedList*
		- 线程不安全
- *Set* 接口
	- *HashSet*
		- LinkedHashSet
	- *TreeSet*
- *Queue* 接口
	- LinkedList
	- ArrayDeque

Map 接口

- HashMap
	- LinkedHashMap
- HashTable
- TreeMap

## List, Set, Queue, Map 四者的区别？

- `List` : 存储的元素是**有序**的、**可重复**的。
- `Set` : 存储的元素**不可重复**的。
- `Queue` : 按特定的排队规则来确定先后顺序，存储的元素是**有序**的、**可重复**的
- `Map` : 使用键值对（key-value）存储，
	- key 是**无序**的、**不可重复**的，
	- value 是**无序**的、**可重复**的，

## 底层数据结构

List

- `ArrayList`：`Object[]` 数组
- `Vector`：`Object[]` 数组
- `LinkedList`：双向链表

Set

- `HashSet`（无序）：基于 `HashMap` 实现的
	- `LinkedHashSet`：内部是通过 `LinkedHashMap` 来实现的。
- `TreeSet`（有序）: TreeMap 实现

Queue

- `PriorityQueue`：`Object[]` 数组来实现**小顶堆**
- `DelayQueue`：`PriorityQueue`
- `ArrayDeque`：可扩容动态双向数组

Map

- `HashMap`：数组+**链表/红黑树**。当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。
	- `LinkedHashMap`：`LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和**链表或红黑树**组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条**双向链表**，使得上面的结构可以**保持键值对的插入顺序**。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- `Hashtable`：数组+链表
- `TreeMap`（有序）：红黑树（自平衡的排序二叉树）

## List

### ArrayList 和 Array（数组）的区别？

- `ArrayList` 会根据实际存储的元素**动态地扩容或缩容**，而 `Array` 被创建之后就不能改变它的长度了
	- `ArrayList` 创建时不需要指定大小，而 `Array` 需要
- `ArrayList` 允许你使用**泛型**来确保类型安全
- 存储内容：
	- `ArrayList` 中只能存储**对象**。对于基本类型数据，需要使用其对应的包装类（如 Integer、Double 等）
	- `Array` 可以直接存储**基本类型数据**，也可以存储**对象**
- `ArrayList` 支持插入、删除、遍历等常见操作，并且提供了丰富的 API 操作方法，比如 `add()`、`remove()` 等。`Array` 只是一个固定长度的数组，只能按照下标访问其中的元素，不具备动态添加、删除元素的能力。

### Vector 和 Stack 的区别?（了解即可）

- `Vector` 和 `Stack` 两者都是**线程安全**的，都是使用 `synchronized` 关键字进行同步处理。
- `Stack` 继承自 `Vector`，是一个后进先出的栈，而 `Vector` 是一个列表。

随着 Java 并发编程的发展，`Vector` 和 `Stack` 已经被淘汰，推荐使用**并发集合类**（例如 `ConcurrentHashMap`、`CopyOnWriteArrayList` 等）或者手动实现线程安全的方法来提供安全的多线程操作支持。

### ArrayList 插入和删除元素的时间复杂度？

对于插入：

- *头部插入*：由于需要将所有元素都依次向后移动一个位置，因此时间复杂度是 `O(n)`
- *尾部插入*：
	- 当 `ArrayList` 的容量未达到极限时，`O(1)`
	- 当需要扩容时，则需要执行一次 `O(n)` 的操作将原数组复制到新的更大的数组中，然后再执行 `O(1)` 的操作添加元素
- *指定位置插入*：需要将目标位置之后的所有元素都向后移动一个位置，然后再把新元素放入指定位置。这个过程需要移动平均 n/2 个元素，因此时间复杂度为 `O(n)`

对于删除：

- *头部删除*：需要将所有元素依次向前移动一个位置，`O(n)`
- *尾部删除*：当删除的元素位于列表末尾时，`O(1)`
- *指定位置删除*：需要将目标元素之后的所有元素向前移动一个位置以填补被删除的空白位置，因此需要移动平均 n/2 个元素，时间复杂度为 `O(n)`

### LinkedList 插入和删除元素的时间复杂度？

- *头部插入/删除*：只需要修改头结点的指针即可完成插入/删除操作，`O(1)`
- *尾部插入/删除*：只需要修改尾结点的指针即可完成插入/删除操作，`O(1)`
- *指定位置插入/删除*：需要先移动到指定位置，再修改指定节点的指针完成插入/删除，因此需要移动平均 n/2 个元素，时间复杂度为 `O(n)`

### LinkedList 为什么不能实现 RandomAccess 接口？

#todo

`RandomAccess` 是一个标记接口，用来表明实现该接口的类支持随机访问（即可以通过索引快速访问元素）。由于 `LinkedList` 底层数据结构是链表，内存地址不连续，只能通过指针来定位，不支持随机快速访问，所以不能实现 `RandomAccess` 接口

### ArrayList 与 LinkedList 区别?

- 是否保证线程安全
- 底层数据结构
- 插入和删除是否受元素位置的影响
	- ArrayList 只有尾部插入才是 `O(1)`，其他位置 `O(n)`
	- LinkedList 只有指定位置插入 `O(n)`，其他位置 `O(1)`
- 是否支持快速随机访问
- 内存空间占用
	- ArrayList 的空间浪费主要体现在在 list 列表的**结尾**会预留一定的容量空间，
	- LinkedList 的空间花费则体现在它的**每一个元素**都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）

> 我们在项目中一般是不会使用到 `LinkedList` 的，需要用到 `LinkedList` 的场景几乎都可以使用 `ArrayList` 来代替，并且，性能通常会更好！就连 `LinkedList` 的作者约书亚 · 布洛克（Josh Bloch）自己都说从来不会使用 `LinkedList`

### ArrayList 扩容机制

ArrayList 维护了一个 `transient Object[] elementData` 数组（transient 表示该属性不会被序列化）

- 新增元素后会检查是否会超过数组的容量，如果超过，则进行扩容

- 创建 ArrayList 对象时，
	- 如果使用无参构造，给 elementData 分配一个空数组，初始容量为 0，第一次添加则扩容 elementData 为 10，如需再次扩容，则 elementData 容量为原先的 1.5 倍
	- 如果使用的是指定大小的构造器，则初始容量为指定大小，之后扩容为原先的 1.5 倍
- 将老数组的元素复制到新数组中，扩容完成


## Set

### 无序性和不可重复性的含义是什么

- 无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的**哈希值**决定的。
- 不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法

### HashSet、LinkedHashSet 和 TreeSet 三者的异同

- 都不是线程安全的
- 底层数据结构不同
	- `HashSet` 的底层数据结构是**哈希表**（基于 `HashMap` 实现）
	- `LinkedHashSet` 的底层数据结构是**链表和哈希表**，元素的插入和取出顺序满足 **FIFO**
	- `TreeSet` 底层数据结构是**红黑树**，元素是**有序**的，排序的方式有自然排序和**定制排序**
- 应用场景不同
	- `HashSet` 用于不需要保证元素插入和取出顺序的场景，
	- `LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景，
	- `TreeSet` 用于支持对元素自定义排序规则的场景。

### HashSet 如何检查重复?

在 JDK1.8 中，`HashSet` 的 `add()` 方法只是简单的调用了 `HashMap` 的 `put()` 方法，并且判断了一下返回值以确保是否有重复元素

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当 set 中没有包含 add 的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}
```

而在 `HashMap` 的 `putVal()` 方法中也能看到如下说明：

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

也就是说，在 JDK1.8 中，实际上无论 `HashSet` 中是否已经存在了某元素，`HashSet` 都会**直接插入（覆盖）**，只是会在 `add()` 方法的==返回值处告诉我们插入前是否存在相同元素==

## Queue

#todo 

## Map

### HashMap 和 Hashtable 的区别

- *线程是否安全*： HashMap 是非线程安全的，`Hashtable` 线程安全
- *对 Null key 和 Null value 的支持*： 
	- HashMap 可以存储 null 的 key 和 value，但 ==null 作为键只能有一个==，null 作为值可以有多个；
	- Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`
- *初始容量和扩容机制不同*
- *底层数据结构*
	- HashMap 是 哈希表（数组+链表/红黑树）
	- HashTable 是 哈希表（数组+链表）

### HashMap 和 TreeMap 区别

- 相同点：`TreeMap` 和 `HashMap` 都继承自 `AbstractMap`

- 不同点：
	- `TreeMap` 它还实现了 **NavigableMap 接口** 和 **SortedMap 接口**
	- 底层数据结构

实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力。`NavigableMap` 接口提供了丰富的方法来探索和操作键值对:

1. *定向搜索*：`ceilingEntry()`, `floorEntry()`, `higherEntry()` 和 `lowerEntry()` 等方法可以用于定位大于、小于、大于等于、小于等于给定键的最接近的键值对。
2. *子集操作*：`subMap()`, `headMap()` 和 `tailMap()` 方法可以高效地创建原集合的子集视图，而无需复制整个集合。
3. *逆序视图*：`descendingMap()` 方法返回一个逆序的 `NavigableMap` 视图，使得可以反向迭代整个 `TreeMap`。
4. *边界操作*：`firstEntry()`, `lastEntry()`, `pollFirstEntry()` 和 `pollLastEntry()` 等方法可以方便地访问和移除元素。

这些方法都是基于**红黑树**的属性实现的，红黑树保持平衡状态，从而保证了搜索操作的时间复杂度为 O(log n)，这让 `TreeMap` 成为了处理**有序集合搜索**问题的强大工具

实现 `SortedMap` 接口让 `TreeMap` 有了对集合中的元素根据键**排序**的能力。默认是按 key 的升序排序，不过我们也可以指定**排序**的比较器（构造器传入 comparator）

### HashMap 底层原理

#### 元素添加和去重

- 添加一个元素时会先得到 hash 值（hashcode 方法）
- 通过 hash 值 和 数组当前容量 n 计算出元素在数组中存放的索引位置：`(n - 1) & hash`
- 看这个位置是否已经存放有元素
    - 如果没有，直接加入；
    - 如果有，调用 `equals()` 和链表上的节点逐个比较，相同直接**覆盖**，不同添加到最后

#### 初始容量及扩容机制

`HashMap` 默认的初始化大小为 16，加载因子 `loadFactor = 0.75`，即容量超过 75% 就会扩容，临界值 `threshold=16 * 0.75=12`

之后每次扩充，容量变为原来的 2 倍，临界值也为原来的 2 倍。

创建时如果给定了容量初始值，`HashMap` 会将其扩充为 **2 的幂次方**大小

---

JDK1.8 之后在解决**哈希冲突**时有了较大的变化，当链表长度大于**阈值（默认为 8）**

- 如果当前数组的长度**小于 64**，那么会选择先进行**数组扩容**，
- 否则将链表转化为**红黑树**，以减少搜索时间。

#### HashMap 的长度为什么是 2 的幂次方？

#todo 

### HashMap 多线程操作导致死循环问题

JDK1.7 及之前版本的 `HashMap` 在多线程环境下扩容操作可能存在死循环问题，这是由于当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，头插法可能会导致链表中的节点指向错误的位置，从而形成一个**环形链表**，进而使得查询元素的操作陷入死循环无法结束。

为了解决这个问题，JDK1.8 版本的 HashMap 采用了==尾插法而不是头插法来避免链表倒置==，使得插入的节点永远都是放在链表的末尾，避免了链表中的环形结构。但是还是不建议在多线程下使用 `HashMap`，因为多线程下使用 `HashMap` 还是会存在数据覆盖的问题。并发环境下，推荐使用 `ConcurrentHashMap`

### 常见遍历方式

#todo

### ConcurrentHashMap

#todo

