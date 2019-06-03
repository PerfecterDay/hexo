---
title: java基础-集合
date: 2019-06-01  20:30:34
tags: java
category: java
---
Java 集合大致可分为 Set 、List 、 Queue 和 Map 四种体系，其中
+ Set 代表 **无序的、不可重复的集合**；
+ List 代表有序、可重复的集合
+ Map 代表具有映射关系的集合
+ Queue 代表队列集合的实现，是 Java 5 新增的

## 概述
Java 集合主要由两个接口派生而出： `Collection` 和 `Map` ，它们是整个集合框架的根接口。
![Collection继承树](/pics/java-collection.jpg)
![Map体系的继承树](/pics/java-map.jpg)
图中以灰色背景覆盖的，分别是 `HashSet` 、 `TreeSet` 、 `ArrayList` 、 `ArrayDeque` 、 `LinkedList` 和 `HashMap` 、 `TreeMap` 等常用实现类。

## Collection
### Collection的常用方法
+ boolean add(Object o)：添加对象到集合，添加成功返回 true
+ boolean addAll(Collection<? extends E> c):把集合c里所有的元素都添加到集合里，如果集合被改变了则返回 true
+ void clear()：清空集合里的元素，将集合长度变为0
+ boolean contains(Object o)：查找集合中是否有指定的对象
+ boolean containsAll(Collection c)：查找集合中是否有集合c中的所有元素
+ boolean isEmpty()：判断集合是否为空
+ Iterator<E> iterator()：返回一个 Iterator 迭代器用于便利元素
+ boolean remove(Object o)：删除指定的对象
+ boolean removeAll(Collection c)：从集合中删除c集合中也有的元素，如果有一个或多个元素被删除，返回 true
+ void retainAll(Collection c)：从集合中删除集合c中不包含的元素
+ int size()：返回当前集合中元素的数量
+ Object[] toArray()：把集合转换成一个数组，所有元素成为数组的元素。
+  <T> T[] toArray(T[] a)：把集合转换成一个**指定类型**数组，所有元素成为数组的元素。

### 遍历操作 Collection 的三种方法
#### 使用 Iterable 接口
使用 `Iterable` 的 `void forEach(Consumer<? super T> action)` 方法， `Iterable` 是 `Collection` 的父接口，因此可以直接调用此方法。

#### 使用 Iterator 迭代器遍历元素
`Iterator` 必须依附于某个 `Collection` 对象,`Iterator` 接口定义了如下4个方法：
+ boolean hasNext()：如果被迭代的集合元素没有遍历完则返回 true
+ E next()：返回集合里的下一个元素
+ void remove()：删除集合中上一次 next 方法返回的元素
+ void forEachRemaining(Consumer<? super E> action)：Java 8 新增方法，可以使用 Lambda 表达式来遍历集合

当使用 `Iterator` 遍历集合元素时，需要注意：
`Collection` 集合里的元素不能被改变，只有通过 `Iterator` 的 `remove()` 方法删除集合元素才可以；否则将会引发 `ConcurrentModificationException` 异常。

#### 使用 foreach 循环遍历集合元素
```
List list = new ArrayList():
list.add("adass"):
for (Object o : list) {
    .....
}
```
同样，使用此方法遍历元素时，`Collection` 集合里的元素也不能被改变，否则将会引发 `ConcurrentModificationException` 异常。

### Java 8 新增的 Predicate操作集合

### Java 8 新增的 Stream 操作集合

## Set
Set 不允许包含相同的元素，如果试图把两个相同的元素加入到同一个 Set 集合，则添加操作失败， add() 方法返回 false，且新元素也不会被加入。但是，对不同的实现类对相同元素的定义略有不同。

### HashSet类
HashSet 使用哈希算法来存储集合元素，因此具有良好的存储和查找性能，具有以下特点：
+ 不能保证元素的排列顺序，顺序可能与添加顺序不同，而且可能发生变化
+ 不是线程安全的
+ 元素值可以为 null

当向 Hashset 中存入一个元素时， HashSet 首先会调用该对象的 hashCode() 方法计算 hashCode 值，然后根据该 hashCode 值决定对象的存储位置。如果两个元素的 equals() 方法返回 true，但是 hashCode() 方法返回值不相等，HashSet 会将它们视为不同的元素存储在不同的位置。

也就是说， HashSet 判断两个元素相同的标准是： equals() 方法返回 true 且 hashCode() 方法返回值相等。

#### equals() 与 hashCode() 方法
如果需要重写类的 equals() 方法，则也应该重写其 hashCode() 方法。规则是：如果两个对象通过 equals() 方法返回 true，则 hashCode() 方法返回值也应该相等。看以下反例：
##### equals()返回true，hashCode()返回值不等
这将导致两个元素存储到不同的位置，从而使两个相同的（equals() 为true）的元素都可以保存到同一个 Set，与Set集合的规则冲突了。
##### equals()返回false, hashCode()返回相等
这将导致不同的对象将存储在同一个位置（hashCode()相等），实际上 HashSet 以链表保存这些元素，这会导致性能的下降。

### LinkedHashSet
LinkedHashSet 是 HashSet 的子类，它也是根据元素的 hashCode()值来决定元素的存储位置，但是它同时使用链表维护元素的次序，使得元素看起来是以插入的顺序保存的的。也就是说，当遍历 LinkedHashSet 集合里的元素时，会按照元素添加的顺序来访问集合里的元素。

LinkedHashSet 需要维护元素的插入顺序，因此性能略低于 HashSet 的性能，但是在迭代访问集合中的全部元素时将有很好的性能，因为它以链表来维护内部顺序。

### TreeSet
TreeSet 是 SortedSet 接口的实现类，它可以确保集合元素处于排序状态。TreeSet 提供了如下额外方法：
+ Comparator<? super E> comparator()：如果 TreeSet 使用了定制排序，则返回定制的 Comparator, 如果 TreeSet 使用了自然排序，则返回 null
+ E first()：返回集合中第一个元素
+ E last()：返回集合中最后一个元素
+ E lower(E e)：返回集合中位于指定元素e的前一个元素，e不需要是 TreeSet中的元素
+ E higher(E e)：返回集合中位于指定元素e的后一个元素，e不需要是 TreeSet中的元素
+ SortedSet<E> subSet(E fromElement, E toElement)：返回子集合Set，范围是[fromElement,toElement)
+ SortedSet<E> headSet(E toElement)：返回小于 toElement 元素组成的子集合
+ SortedSet<E> tailSet(E fromElement)：返回大于或等于 fromElement 的元素组成的子集合

TreeSet 底层使用红黑树结构来存储集合元素。
#### TreeSet 排序规则
TreeSet 支持两种排序方法：自然排序和定制排序，默认使用自然排序。

1. 自然排序

TreeSet 会调用集合元素的 `int compareTo(T o)` 方法来比较元素之间的大小关系，然后将集合元素按升序排列。因此待插入的元素必须实现 `Comparable` 接口，否则会引发
`ClassCastException` 。

当调用 obj1.compareTo(obj2) 时，如果返回值大于0，代表 obj1 大于 obj2，等于0代表两者相等，小于0代表 obj1 小于 obj2 。

在这种情况下，对于 TreeSet 集合而言，判断两个对象是否相等的唯一标准是：两个对象通过 compareTo() 方法比较是否返回0。即使 equals() 方法返回true，而 compareTo() 方法返回不为0， TreeSet 也将两者视为不相元素。因此，如果要把对象插入到 TreeSet，且重写了 equals 方法，则应该保证 equals() 相等的两个对象在调用 compareTo() 方法时返回0。

2. 定制排序

自然排序默认使用升序排列，如果想要实现如降序排列等其它特性，则可以通过 Comparator 接口的帮助来实现。该接口里包含一个 int compare(T o1, T o2) 方法用于比较 o1 和 o2 的大小：
+ 如果该方法返回正整数，则表明 o1 大于 o2；
+ 如果该方法返回负整数，则表明 o1 小于 o2；
+ 如果该方法返回0，则表明 o1 等于 o2

如果需要实现定制排序，则要在创建 TreeSet 集合时，提供一个 Comparator 对象与该 TreeSet 集合关联，由该 Comparator 负责元素的排序逻辑。

在这种情况下，对于 TreeSet 集合而言，判断两个对象是否相等的唯一标准是：两个对象通过 compare() 方法比较是否返回0。即使 equals() 方法返回true，而 compare() 方法返回不为0， TreeSet 也将两者视为不相元素。因此，如果要把对象插入到 TreeSet，且重写了 equals 方法，则应该保证 equals() 相等的两个对象在调用 compare() 方法时返回0。

### EnumSet
### 各个 Set 实现类的性能分析
HashSet 和 TreeSet 是 Set 的两个典型实现， HashSet 总是比 TreeSet 的性能好，因为 TreeSet 需要额外的红黑树来维护集合元素次序，只有当需要保持排序的 Set 时，才应该考虑使用 TreeSet。

LinkedHashSet 的插入、删除操作要比 HashSet 略慢，这是由于要维护链表所带来的额外开销所致，但是由于有了链表，在遍历集合元素时会更快。

EnumSet 是所有 Set 中性能最好的，但它只能保存同一个枚举类的枚举值作为集合元素。

## List 集合
List 集合代表一个元素有序、可重复的集合，集合中每个元素都有其对应的顺序索引。

### List 接口
List 作为 Collection 接口的子接口，当然可以使用 Collection 接口里的全部方法。而且因为 List 是有序集合，因此 List 里增加了一些根据索引来操作集合的方法：
+ void add(int index, E element)：将 element 添加到 index 处
+ boolean addAll(Collection<? extends E> c)：将集合c所包含的所有元素都插入到 List 集合的 index 处
+ E get(int index):返回集合 index 处的元素
+ int indexOf(Object o)：返回对象o在List集合中第一次出现的位置索引
+ int lastIndexOf(Object o)：返回对象o在List集合中最后次出现的位置索引
+ E remove(int index)：删除并返回 index 索引处的元素
+ E set(int index, E element)：将 index 索引处的元素替换成 element 对象，并返回被替换的旧元素
+ List<E> subList(int fromIndex, int toIndex)：返回从索引 [fromIndex,toIndex)处所有元素组成的子集合。
+ void replaceAll(UnaryOperator<E> operator)：根据 operator 指定的计算规则重新设置List集合所有元素
+ void sort(Comparator<? super E> c)：根据 Comparator 参数对 List 排序。

List 判断两个对象相等的依据是：只要equals() 方法返回 true ，两者就视为相等。

### ListIterator 接口
List除了可以使用 iterator() 来获取 Iterator 迭代器以外，还提供了一个 ListIterator listIterator() 方法获取一个 ListIterator 接口，ListIterator 继承了 Iterator ，并增加了如下方法：
+ boolean hasPrevious()：返回该迭代器关联的集合是否还有上一个元素
+ E previous()：返回上一个元素
+ void add(E e)：在指定位置插入一个元素

### ArrayList 和 Vector
ArrayList 和 Vector 都是基于数组实现的 List 类，底层封装了一个动态的、允许再分配的 Object[]数组（或泛型数组）。

如果创建 ArrayList 或者 Vector 时，没有指定数组长度，则默认大小是10，如果List长度增长到数组最大长度时，再添加元素就要重新分配数组并将原数组中的元素拷贝到新数组中，会有很大开销。所以，如果开始就知道List的大小，可以在创建时就指定，这样能避免数组重分配，提高性能。

此外，两者还提供了如下方法来重新分配数组：
+ void ensureCapacity(int minCapacity)：增加数组长度，数组长度+= minCapacity
+ void trimToSize()：调整数组长度为当前元素个数，释放空间

两者有一个显著的区别就是：ArrayList 是线程不安全的，而 Vector 是线程安全的。即使如此，也不推荐使用 Vector 类，如果想要线程安全可以使用 Collections 工具类，它可以将 ArrayList 变成线程安全的类。

### Arrays.ArrayList
Arrays 工具类的 asList(Object... os) 方法可以把一个数组或指定个数的对象转换成一个 List 集合，这个 List 实际上是 Arrays.ArrayList 的实例。

Arrays.ArrayList 是一个固定长度的 List 集合，程序只能遍历访问该集合里的元素，不可增加、删除其中的元素。任何添加、删除操作将在运行时引发 UnsupportedOperationException 异常。

## Queue 和 Deque   
Queue 用于模拟队列这种数据结构，队列通常是先进先出（FIFO）的容器。队列的头部保存存放时间最长的元素，队尾保存存放时间最短的元素。新入元素放到队尾，访问元素从队列头部出队。通常，队列不允许随机访问队列中的元素。

Queue 提供了如下常用方法:
+ boolean add(E e)：将指定元素添加到队列的尾部
+ E element()：获取队列头部的元素，但是不删除该元素（不出队）
+ boolean offer(E e)：将制定元素加入到队尾（入队），此方法通常比 add 更好
+ E peek()：获取队列头部的元素，但是不删除该元素，如果队列为空，返回null
+ E poll()：获取队列头部的元素，并删除该元素（出队），如果队列为空返回 null
+ E remove()：获取队列的头部元素，并删除该元素。

Queue 还有一个 Deque 子接口， Deque 代表一个 *双端队列* ，双端队列可以同时从两端来添加、删除元素。因此 Deque 的实现类既可以当成队列使用，也可以当成栈使用。JDK为 Deque 提供了 ArrayDeque 和 LinkedList 两个实现类。

### PriorityQueue 实现类
PriorityQueue 保存队列的顺序不是按元素加入的顺序，而是按照队列元素的大小进行重新排序的。出队时，取的不是最先进入队列的元素，而是队列中最小的元素（队尾到队头降序排列）。

因为涉及到排序，因此也有自然排序和定制排序两种方式，与前文所述相同。

### Deque 接口与 ArrayDeque 实现类
+ void addFirst(E e):将指定元素插入双端队列的队头
+ void addLast(E e):将指定元素插入到双端队列的队尾
+ boolean offerFirst(E e):将指定元素插入该双端队列的队头
+ boolean offerLast(E e):将指定元素插入该双端队列的队尾
+ E removeFirst():获取并删除队头元素
+ E removeLast():获取并删除队尾元素
+ E pollFirst():获取并删除队头元素，如果队列为空，返回 null
+ E pollLast():获取并删除队尾元素，如果队列为空，返回 nul
+ E getFirst():获取但并不删除队头元素
+ E getLast():获取但并不删除队尾元素
+ E peekFirst():获取但并不删除队头元素，如果队列为空，返回 nul
+ E peekLast():获取但并不删除队尾元素，如果队列为空，返回 nul
+ boolean removeFirstOccurrence(Object o):删除第一次出现的元素o
+ boolean removeLastOccurrence(Object o):删除最后一次出现的元素o
+ void push(E e):将元素e入栈
+ E pop():pop处栈顶元素
+ public int size():获取队列大小
+ Iterator<E> iterator():获取队列迭代器

Deque 不仅可以当成双端队列来使用，而且可以被当成栈来使用，因为含有 push() 、 pop() 方法。

Deque 接口有一个典型实现类 ArrayDeque，基于数组实现的双端队列，可以在初始化时指定数组大小，如果不指定，则默认大小是 16。

### LinkedList 实现类
LinkedList 既实现了 List 接口，同时也实现了 Deque 接口。因此，其功能非常强大，可以用来做 List , 也可以作为队列、栈、双端队列等数据结构使用。

### 各种线性表的性能分析
一般来说，由于数组以一块连续内存来保存数据，所以数组的随机访问性能最好，所有内部以数组作为底层实现的集合类在随机访问时都有较好的性能；内部以链表实现的集合在执行插入、删除操作的集合有较好的性能。但总体上说，ArrayList 的性能要优于 LinkedList，大部分时候应该考虑使用 ArrayList。

关于 List 集合的使用有如下建议：
1. 如果需要遍历 List 集合元素：对于数组类（ArrayList、ArrayDeque、Vector等）List应该使用随机访问形式(get(index))来遍历；对于链表类（LinkedList)，则应该采用迭代器（Iterator)来遍历集合元素
2. 如果需要经常执行插入、删除操作来改变包含大量数据的List集合，应该优先考虑链表类的LinkedList。使用ArrayList可能需要经常重新分配内存数组
3. 如果有多个线程需要同时访问List集合，应该使用 Collections 工具类将集合包装成线程安全的类。

## HashMap 和 Hashtable 的区别
1. Hashtable 是一个线程安全的 Map 实现，但 HashMap不是线程安全的，所以 HashMap 性能会更好点
2. Hashtable 不允许使用 null 作为 key 和 value。如果试图把 null 放入 Hashtable 将会引发 NPE，但 HashMap 可以使用 null 作为 key 或 value

与 HashSet 类似， HashMap 、 Hashtable 判断两个 key 相等的标准也是： 两个 key 通过 equals() 方法返回 true ，且 hashCode() 返回值也相同。

另外， HashMap 、 Hashtable 中还有一个 containsValue() 方法，用于判断是否包含指定 value。此时，判断是否与 value 相等的标准更简单：只需要 equals 方法返回 true 即可。
