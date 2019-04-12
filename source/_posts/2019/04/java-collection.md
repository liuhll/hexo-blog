---
title: java基础知识之集合
date: 2019-04-12 15:03:39
categories: "java"
tags:
- java基础知识
---
# 集合与数组
**数组**:（可以存储基本数据类型）是用来存现对象的一种容器，但是数组的长度固定，不适合在对象数量未知的情况下使用。

**集合**:（只能存储对象，对象类型可以不一样）的长度可变，可在多数情况下使用。

# 集合中接口和类的关系

## 层次图
### 简单版
![1](1.png)

### 完整版
![2](2.png)

## 对比
- `Collection`接口是集合类的根接口，Java中没有提供这个接口的直接的实现类。但是却让其被继承产生了两个接口，就是`Set`和`List`。
  - `Set`中不能包含重复的元素。
  - `List`是一个有序的集合，可以包含重复的元素，提供了按索引访问的方式。

- `Map`是`Java.util`包中的另一个接口，它和`Collection`接口没有关系，是相互独立的，但是都属于集合类的一部分。`Map`包含了`key-value`对。`Map`不能包含重复的`key`，但是可以包含相同的`value`。

- `Iterator`所有的集合类，都实现了`Iterator`接口，这是一个用于遍历集合中元素的接口，主要包含以下三种方法：
  - 1. `hasNext()`是否还有下一个元素。
  - 2. `next()`返回下一个元素。
  - 3. `remove()`删除当前元素。

| 接口 |	子接口 |	是否有序  | 	是否允许元素重复 |
|:-----|:---------|:------------|:--------------------|
| Collection |  |  |　
| List  |　　ArrayList |	 否  |	是 |
| 　　　 |　　LinkedList |	否 |	是 |
|　　　　|　　Vector |	否 |	是 |
| Set |	AbstractSet |	否 |	否 |
|　　 |	HashSet |	否 |	否 |
|　　 |	TreeSet |	是（用二叉排序树） |	否 |
| Map |	 AbstractMap |	否 |	使用key-value来映射和存储数据，key必须唯一，value可以重复 |
| 　　 |	HashMap	 |	否 |  |
|　　 |	TreeMap  |	是（用二叉排序树） |	使用key-value来映射和存储数据，key必须唯一，value可以重复 |

# Collection

## List
`List`里存放的对象是有序的，同时也是可以重复的，`List`关注的是索引，拥有一系列和索引相关的方法，查询速度快。因为往`list`集合里插入或删除数据时，会伴随着后面数据的移动，所有插入删除数据速度慢。

### ArrayList
`ArrayList`是基于数组的，在初始化`ArrayList`时，会构建空数组（`Object[] elementData={}`）。`ArrayList`是一个无序的，它是按照添加的先后顺序排列，当然，他也提供了`sort`方法，如果需要对`ArrayList`进行排序，只需要调用这个方法，提供`Comparator`比较器即可。
#### add操作：
- 如果是第一次添加元素，数组的长度被扩容到默认的`capacity`，也就是`10`.
- 当发觉同时添加一个或者是多个元素，数组长度不够时，就扩容，这里有两种情况：
   - 只添加一个元素，例如：原来数组的`capacity`为10，`size`已经为10，不能再添加了。需要扩容，新的capacity=old capacity+old capacity>>1=10+10/2=15.即新的容量为15。
   - 当同时添加多个元素时，原来数组的`capacity`为10，`size`为10，当同时添加6个元素时。它需要的min capacity为16，而按照capacity=old capacity+old capacity>>1=10+10/2=15。new capacity小于min capacity，则取min capacity。

- 对于添加，如果不指定下标，就直接添加到数组后面，不涉及元素的移动，如果要添加到某个特定的位置，那需要将这个位置开始的元素往后挪一个位置，然后再对这个位置设置。

#### Remove操作：
`Remove`提供两种，按照下标和value。

- `remove(int index)`：首先需要检查Index是否在合理的范围内。其次再调用`System.arraycopy`将`index`之后的元素向前移动。
- `remove(Object o)`：首先遍历数组，获取第一个相同的元素，获取该元素的下标。其次再调用`System.arraycopy`将`index`之后的元素向前移动。

#### Get操作：
这个比较简单，直接对数组进行操作即可。

### LinkedList
`LinkedList`是基于链表的，它是一个双向链表，每个节点维护了一个`prev`和`next`指针。同时对于这个链表，维护了`first`和`last`指针，`first`指向第一个元素，`last`指向最后一个元素。`LinkedList`是一个无序的链表，按照插入的先后顺序排序，不提供`sort`方法对内部元素排序。

#### Add元素：
- `LinkedList`提供了几个添加元素的方法：`addFirst`、`addLast`、`addAll`、`add`等，时间复杂度为`O(1)`。

#### Remove元素：
- `LinkedList`提供了几个移除元素的方法：`removeFirst`、`removeLast`、`removeFirstOccurrence`、`remove`等，时间复杂度为`O(1)`。

#### Get元素：
- 根据给定的下标`index`，判断它`first`节点、`last`直接距离，如果`index<size（数组元素个数)/2`,就从`first`开始。如果大于，就从`last`开始。这个和我们平常思维不太一样，也许按照我们的习惯，从`first`开始。这也算是一点小心的优化吧。

### 遍历操作
在类集中提供了以下四种的常见输出方式：
1. `Iterator`：迭代输出，是使用最多的输出方式。
2. `ListIterator`：是Iterator的子接口，专门用于输出List中的内容。
3. `foreach`输出：JDK1.5之后提供的新功能，可以输出数组或集合。
4. `for`循环

**代码操作**

for的形式：
```java
for（int i=0;i<arr.size();i++）{...}
```

foreach的形式： 
```java
for（int　i：arr）{...}
```
iterator的形式：
```java
Iterator it = arr.iterator();
while(it.hasNext()){ object o =it.next(); ...}
```

## Set
`Set`里存放的对象是无序，不能重复的，集合中的对象不按特定的方式排序，只是简单地把对象加入集合中。

### HashSet
- `HashSet`是基于`HashMap`来实现的，操作很简单，更像是对`HashMap`做了一次“封装”，而且只使用了`HashMap`的`key`来实现各种特性，而`HashMap`的`value`始终都是`PRESENT`。
- `HashSet`不允许重复（`HashMap`的`key`不允许重复，如果出现重复就覆盖），允许`null`值，非线程安全。

#### 构造方法
- `HashSet()` 
  - 构造一个新的空 `set`，其底层 `HashMap` 实例的默认初始容量是 16，加载因子是 0.75
- `HashSet(Collection<? extends E> c)`
  - 构造一个包含指定 `collection` 中的元素的新 set
- `HashSet(int initialCapacity)` 
  - 构造一个新的空 `set` ,其底层 `HashMap` 实例具有指定的初始容量和默认的加载因子（0.75）。
- `HashSet(int initialCapacity, float loadFactor)`
  - 构造一个新的空 `set`，其底层 `HashMap` 实例具有指定的初始容量和指定的加载因子。

#### 方法
- `boolean add(E e)` 
　　如果此 set 中尚未包含指定元素，则添加指定元素。
- `void clear()`
　　从此 set 中移除所有元素。
- `Object clone()` 
　　返回此 HashSet 实例的浅表副本：并没有复制这些元素本身。
- `boolean contains(Object o)` 
　　如果此 set 包含指定元素，则返回 true。
- `boolean isEmpty()`
　　如果此 set 不包含任何元素，则返回 true。
- `Iterator iterator()` 
　　返回对此 set 中元素进行迭代的迭代器。
- `boolean remove(Object o)` 
　　如果指定元素存在于此 set 中，则将其移除。
- `int size()`
　　返回此 set 中的元素的数量（set 的容量）。

### TreeSet
基于 `TreeMap` 的 `NavigableSet` 实现。使用元素的自然顺序对元素进行排序，或者根据创建 `set` 时提供的 `Comparator`进行排序，具体取决于使用的构造方法。

### set的遍历
同 `list`

## Queue
队列是一种特殊的线性表，它只允许在表的前端进行删除操作，而在表的后端进行插入操作。
`LinkedList`类实现了`Queue`接口，因此我们可以把`LinkedList`当成`Queue`来用。

# Map
`Map`集合中存储的是键值对，键不能重复，值可以重复。根据键得到值，对`map`集合遍历时先得到键的`set`集合，对`set`集合进行遍历，得到相应的值。
## HashMap
　　数组方式存储key/value，线程非安全，允许null作为key和value，key不可以重复，value允许重复，不保证元素迭代顺序是按照插入时的顺序，key的hash值是先计算key的hashcode值，然后再进行计算，每次容量扩容会重新计算所以key的hash值，会消耗资源，要求key必须重写equals和hashcode方法

　　默认初始容量16，加载因子0.75，扩容为旧容量乘2，查找元素快，如果key一样则比较value，如果value不一样，则按照链表结构存储value，就是一个key后面有多个value；

### 方法
1. 添加：
   - `V put(K key, V value)` （可以相同的key值，但是添加的value值会覆盖前面的，返回值是前一个，如果没有就返回null）
   - `putAll(Map<? extends K,? extends V> m)` 从指定映射中将所有映射关系复制到此映射中（可选操作）。

2. 删除
   - `remove()` 删除关联对象，指定key对象
   - `clear()` 清空集合对象

3. 获取
   - `value get(key)` 可以用于判断键是否存在的情况。当指定的键不存在的时候，返回的是null。

4. 判断：
   - `boolean isEmpty()` 长度为0返回true否则false
   - `boolean containsKey(Object key)` 判断集合中是否包含指定的key
   - `boolean containsValue(Object value)` 判断集合中是否包含指定的value

5. 长度：
  - `Int size()`

## Hashtable
`Hashtable`与`HashMap`类似，是`HashMap`的线程安全版，它支持线程的同步，即任一时刻只有一个线程能写`Hashtable`，因此也导致了`Hashtale`在写入时会比较慢，它继承自`Dictionary`类，不同的是它不允许记录的键或者值为null，同时效率较低。
## LinkedHashMap
`LinkedHashMap`保存了记录的插入顺序，在用`Iteraor`遍历`LinkedHashMap`时，先得到的记录肯定是先插入的，在遍历的时候会比`HashMap`慢，有`HashMap`的全部特性。

## TreeMap
基于红黑二叉树的`NavigableMap`的实现，线程非安全，不允许null，key不可以重复，value允许重复，存入`TreeMap`的元素应当实现`Comparable`接口或者实现`Comparator`接口，会按照排序后的顺序迭代元素，两个相比较的key不得抛出`classCastException`。主要用于存入元素的时候对元素进行自动排序，迭代输出的时候就按排序顺序输出

## Map遍历
### 1. KeySet()
将`Map`中所有的键存入到set集合中。因为set具备迭代器。所有可以迭代方式取出所有的键，再根据get方法。获取每一个键对应的值。 
`keySet()`:迭代后只能通过get()取key 。取到的结果会乱序，是因为取得数据行主键的时候，使用了`HashMap.keySet()`方法，而这个方法返回的`Set`结果，里面的数据是乱序排放的。
```java
Map map = new HashMap();
map.put("key1","lisi1");
map.put("key2","lisi2");
map.put("key3","lisi3");
map.put("key4","lisi4");  
//先获取map集合的所有键的set集合，keyset（）
Iterator it = map.keySet().iterator();
//获取迭代器
while(it.hasNext()){
    Object key = it.next();
    System.out.println(map.get(key));
}
```

### 2. values() 获取所有的值
```java
Collection<String> vs = map.values();
Iterator<String> it = vs.iterator();
while (it.hasNext()) {
    String value = it.next();
    System.out.println(" value=" + value);
}
```

### 3. entrySet()
`Set<Map.Entry<K,V>> entrySet()` //返回此映射中包含的映射关系的 `Set` 视图。（一个关系就是一个键-值对），就是把(key-value)作为一个整体一对一对地存放到`Set`集合当中的。`Map.Entry`表示映射关系。`entrySet()`：迭代后可以`e.getKey()`，`e.getValue()`两种方法来取key和value。返回的是`Entry`接口。
```java
// 返回的Map.Entry对象的Set集合 Map.Entry包含了key和value对象
       Set<Map.Entry<Integer, String>> es = map.entrySet();
       Iterator<Map.Entry<Integer, String>> it = es.iterator();
       while (it.hasNext()) {    
            // 返回的是封装了key和value对象的Map.Entry对象
            Map.Entry<Integer, String> en = it.next();

            // 获取Map.Entry对象中封装的key和value对象
            Integer key = en.getKey();
            String value = en.getValue();
            System.out.println("key=" + key + " value=" + value);
        }
```
> **Notes**
> - 推荐使用第三种方式，即`entrySet()`方法，效率较高。
> - 对于`keySet`其实是遍历了2次，一次是转为`iterator`，一次就是从`HashMap`中取出`key`所对于的`value`。而`entryset`只是遍历了第一次，它把`key`和`value`都放到了`entry`中，所以快了。两种遍历的遍历时间相差还是很明显的。

# 总结
## Vector和ArrayList
1. `Vector`是线程同步的，所以它也是线程安全的，而`Arraylist`是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用`Arraylist`效率比较高。
2. 如果集合中的元素的数目大于目前集合数组的长度时，`Vector`增长率为目前数组长度的100%，而`Arraylist`增长率为目前数组长度的50%。如果在集合中使用数据量比较大的数据，用`Vector`有一定的优势。
3. 如果查找一个指定位置的数据，`Vector`和`Arraylist`使用的时间是相同的，如果频繁的访问数据，这个时候使用`Vector`和`Arraylist`都可以。而如果移动一个指定位置会导致后面的元素都发生移动，这个时候就应该考虑到使用`Linklist`,因为它移动一个指定位置的数据时其它元素不移动。

4. `ArrayList` 和`Vector`是采用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，都允许直接序号索引元素，但是插入数据要涉及到数组元素移动等内存操作，所以索引数据快，插入数据慢，`Vector`由于使用了`synchronized`方法（线程安全）所以性能上比`ArrayList`要差，`LinkedList`使用双向链表实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项的前后项即可，所以插入数度较快。

## Arraylist和Linkedlist
1. `ArrayList`是实现了基于动态数组的数据结构，`LinkedList`基于链表的数据结构。
2. 对于随机访问`get`和`set`，`ArrayList`觉得优于`LinkedList`，因为`LinkedList`要移动指针。
3. 对于新增和删除操作`add`和`remove`，`LinedList`比较占优势，因为`ArrayList`要移动数据。 这一点要看实际情况的。若只对单条数据插入或删除，`ArrayList`的速度反而优于`LinkedList`。但若是批量随机的插入删除数据，`LinkedList`的速度大大优于`ArrayList`。 因为`ArrayList`每插入一条数据，要移动插入点及之后的所有数据。

## HashMap与TreeMap
1. `HashMap`通过`hashcode`对其内容进行快速查找，而`TreeMap`中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用`TreeMap`（`HashMap`中元素的排列顺序是不固定的）。
2. 在`Map` 中插入、删除和定位元素，`HashMap`是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么`TreeMap`会更好。使用`HashMap`要求添加的键类明确定义了`hashCode()`和 `equals()`的实现。
3. 两个map中的元素一样，但顺序不一样，导致`hashCode()`不一样。

**测试：**

- 在`HashMap`中，同样的值的`map`,顺序不同，`equals`时，false;
- 而在`TreeMap`中，同样的值的`map`,顺序不同,`equals`时，true，说明，`TreeMap`在`equals()`时是整理了顺序了的。

## HashTable与HashMap
1. 同步性:`Hashtable`是线程安全的，也就是说是同步的，而`HashMap`是线程序不安全的，不是同步的。
2. `HashMap`允许存在一个为null的key，多个为null的value。
3. `Hashtable`的key和value都不允许为null。