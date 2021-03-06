---
layout:     post
title:      "Java集合"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-05 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - Java
---

* content
{:toc}

## 1.集合包

    集合包最常用的有Collection和Map两个接口的实现类，Colleciton用于存放多个单对象，Map用于存放Key-Value形式的键值对。

  Collection中最常用的又分为两种类型的接口：List和Set，两者最明显的差别为List支持放入重复的元素，而Set不支持。

List最常用的实现类有：ArrayList、LinkedList、Vector及Stack；Set接口常用的实现类有：HashSet、TreeSet。

### 1.1 ArrayList

  ArrayList基于数组方式实现，默认构造器通过调用ArrayList(int)来完成创建，传入的值为10，实例化了一个Object数组，并将此数组赋给了当前实例的elementData属性，此Object数组的大小即为传入的initialCapacity，因此调用空构造器的情况下会创建一个大小为10的Object数组。

插入对象：add(E)

    基于已有元素数量加1作为名叫minCapacity的变量，比较此值和Object数组的大小，若大于数组值，那么先将当前Object数组值赋给一个数组对象，接着产生一个鑫的数组容量值。此值的计算方法为当前数组值*1.5+1，如得出的容量值仍然小于minCapacity，那么就以minCapacity作为新的容量值，调用Arrays.copyOf来生成新的数组对象。

    还提供了add(int,E)这样的方法将元素直接插入指定的int位置上，将目前index及其后的数据都往后挪一位，然后才能将指定的index位置的赋值为传入的对象，这种方式要多付出一次复制数组的代价。还提供了addAll

 删除对象：remove(E)

   这里调用了faseRemove方法将index后的对象往前复制一位，并将数组中的最后一个元素的值设置为null，即释放了对此对象的引用。 还提供了remove(int)方法来删除指定位置的对象，remove(int)的实现比remove(E)多了一个数组范围的检测，但少了对象位置的查找，因此性能会更好。

获取单个对象：get(int)

遍历对象：iterator()

判断对象是否存在：contains(E)

 总结：

    1，ArrayList基于数组方式实现，无容量的限制；

    2，ArrayList在执行插入元素时可能要扩容，在删除元素时并不会减小数组的容量（如希望相应的缩小数组容量，可以调用ArrayList的trimToSize()），在查找元素时要遍历数组，对于非null的元素采取equals的方式寻找；

    3，ArrayList是非线程安全的。

### 1.2 LinkedList

    LinkedList基于双向链表机制，所谓双向链表机制，就是集合中的每个元素都知道其前一个元素及其后一个元素的位置。在LinkedList中，以一个内部的Entry类来代表集合中的元素，元素的值赋给element属性，Entry中的next属性指向元素的后一个元素，Entry中的previous属性指向元素的前一个元素，基于这样的机制可以快速实现集合中元素的移动。

总结：

    1，LinkedList基于双向链表机制实现；

    2，LinkedList在插入元素时，须创建一个新的Entry对象，并切换相应元素的前后元素的引用；在查找元素时，须遍历链表；在删除元素时，要遍历链表，找到要删除的元素，然后从链表上将此元素删除即可，此时原有的前后元素改变引用连在一起；

    3，LinkedList是非线程安全的。

### 1.3 Vector

    其add、remove、get(int)方法都加了synchronized关键字，默认创建一个大小为10的Object数组，并将capacityIncrement设置为0。容量扩充策略：如果capacityIncrement大于0，则将Object数组的大小扩大为现有size加上capacityIncrement的值；如果capacity等于或小于0，则将Object数组的大小扩大为现有size的两倍，这种容量的控制策略比ArrayList更为可控。

    Vector是基于Synchronized实现的线程安全的ArrayList，但在插入元素时容量扩充的机制和ArrayList稍有不同，并可通过传入capacityIncrement来控制容量的扩充。

### 1.4 Stack

    Stack继承于Vector，在其基础上实现了Stack所要求的后进先出(LIFO)的弹出与压入操作，其提供了push、pop、peek三个主要的方法：

    push操作通过调用Vector中的addElement来完成；

    pop操作通过调用peek来获取元素，并同时删除数组中的最后一个元素；

    peek操作通过获取当前Object数组的大小，并获取数组上的最后一个元素。

### 1.5 HashSet

    默认构造创建一个HashMap对象

add(E)：调用HashMap的put方法来完成此操作，将需要增加的元素作为Map中的key，value则传入一个之前已创建的Object对象。

remove(E)：调用HashMap的remove(E)方法完成此操作。

contains(E)：HashMap的containsKey

iterator()：调用HashMap的keySet的iterator方法。

HashSet不支持通过get(int)获取指定位置的元素，只能自行通过iterator方法来获取。

总结：

    1，HashSet基于HashMap实现，无容量限制；

    2，HashSet是非线程安全的。

### 1.6 TreeSet

    TreeSet和HashSet的主要不同在于TreeSet对于排序的支持，TreeSet基于TreeMap实现。

### 1.7 HashMap

    HashMap空构造，将loadFactor设为默认的0.75，threshold设置为12，并创建一个大小为16的Entry对象数组。

    基于数组+链表的结合体(链表散列)实现，将key-value看成一个整体，存放于Entity[]数组，put的时候根据key hash后的hashcode和数组length-1按位与的结果值判断放在数组的哪个位置，如果该数组位置上若已经存放其他元素，则在这个位置上的元素以链表的形式存放。如果该位置上没有元素则直接存放。

当系统决定存储HashMap中的key-value对时，完全没有考虑Entry中的value，仅仅只是根据key来计算并决定每个Entry的存储位置。我们完全可以把Map集合中的value当成key的附属，当系统决定了key的存储位置之后，value随之保存在那里即可。get取值也是根据key的hashCode确定在数组的位置，在根据key的equals确定在链表处的位置。
```
while (capacity < initialCapacity)
    capacity <<= 1;
```

以上代码保证了初始化时HashMap的容量总是2的n次方，即底层数组的长度总是为2的n次方。它通过h & (table.length -1) 来得到该对象的保存位，若length为奇数值，则与运算产生相同结果，便会形成链表，尽可能的少出现链表才能提升hashMap的效率，所以这是hashMap速度上的优化。

扩容resize():

当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容，而在HashMap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。

负载因子衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。

HashMap的实现中，通过threshold字段来判断HashMap的最大容量。threshold就是在此loadFactor和capacity对应下允许的最大元素数目，超过这个数目就重新resize，以降低实际的负载因子。默认的的负载因子0.75是对空间和时间效率的一个平衡选择。

initialCapacity*2，成倍扩大容量，HashMap(int initialCapacity, float loadFactor)：以指定初始容量、指定的负载因子创建一个 HashMap。不设定参数，则初始容量值为16，默认的负载因子为0.75，不宜过大也不宜过小，过大影响效率，过小浪费空间。扩容后需要重新计算每个元素在数组中的位置，是一个非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。

     HashTable数据结构的原理大致一样，区别在于put、get时加了同步关键字，而且HashTable不可存放null值。

在高并发时可以使用ConcurrentHashMap，其内部使用锁分段技术，维持这锁Segment的数组，在数组中又存放着Entity[]数组，内部hash算法将数据较均匀分布在不同锁中。

总结：

    1，HashMap采用数组方式存储key、value构成的Entry对象，无容量限制；

    2，HashMap基于key hash寻找Entry对象存放到数组的位置，对于hash冲突采用链表的方式解决；

    3，HashMap在插入元素时可能会扩大数组的容量，在扩大容量时须要重新计算hash，并复制对象到新的数组中；

    4，HashMap是非线程安全的。

详细说明：http://zhangshixi.iteye.com/blog/672697 

## 1.8 TreeMap

    TreeMap基于红黑树的实现，因此它要求一定要有key比较的方法，要么传入Comparator实现，要么key对象实现Comparable借口。在put操作时，基于红黑树的方式遍历，基于comparator来比较key应放在树的左边还是右边，如找到相等的key，则直接替换掉value。