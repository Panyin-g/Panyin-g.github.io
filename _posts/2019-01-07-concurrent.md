---
layout:     post
title:      "Java并发包"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-05 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - Java
---

* content
{:toc}

# 并发包

 jdk5.0一很重要的特性就是增加了并发包java.util.concurrent.*，在说具体的实现类或接口之前，这里先简要说下Java内存模型、volatile变量及AbstractQueuedSynchronizer(以下简称AQS同步器)，这些都是并发包众多实现的基础。

## Java内存模型

    描述了线程内存与主存见的通讯关系。定义了线程内的内存改变将怎样传递到其他线程的规则，同样也定义了线程内存与主存进行同步的细节，也描述了哪些操作属于原子操作及操作间的顺序。

代码顺序规则：

    一个线程内的每个动作happens-before同一个线程内在代码顺序上在其后的所有动作.

volatile变量规则：

    对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入.

传递性：

    如果A happens-before B, B happens-before C, 那么A happens-before C.    

 

## volatile

当我们声明共享变量为volatile后，对这个变量的读/写将会很特别。理解volatile特性的一个好方法是：把对volatile变量的单个读/写，看成是使用同一个监视器锁对这些单个读/写操作做了同步。

监视器锁的happens-before规则保证释放监视器和获取监视器的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

 

简而言之，volatile变量自身具有下列特性：

可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

 

volatile写的内存语义如下：

当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。

 

volatile读的内存语义如下：

当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

 

下面对volatile写和volatile读的内存语义做个总结：

线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程发出了（其对共享变量所在修改的）消息。

线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改的）消息。

线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。

锁释放-获取与volatile的读写具有相同的内存语义，

锁释放的内存语义如下：

    当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。

锁获取的内存语义如下：

    当线程获取锁时，JMM会把该线程对应的本地内存置为无效，从而使得被监视器保护的临界区代码必须要从主内存中读取共享变量。

下面对锁释放和锁获取的内存语义做个总结：

线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的）消息。

线程B获取一个锁，实质上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。

线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息。

## AbstractQueuedSynchronizer (AQS)

    AQS使用一个整型的volatile变量（命名为state）来维护同步状态，这是接下来实现大部分同步需求的基础。提供了一个基于FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架。使用的方法是继承，子类通过继承同步器并需要实现它的方法来管理其状态，管理的方式就是通过类似acquire和release的方式来操纵状态。然而多线程环境中对状态的操纵必须确保原子性，因此子类对于状态的把握，需要使用这个同步器提供的以下三个方法对状态进行操作：

java.util.concurrent.locks.AbstractQueuedSynchronizer.getState()

java.util.concurrent.locks.AbstractQueuedSynchronizer.setState(int)

java.util.concurrent.locks.AbstractQueuedSynchronizer.compareAndSetState(int, int)

子类推荐被定义为自定义同步装置的内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干acquire之类的方法来供使用。该同步器即可以作为排他模式也可以作为共享模式，当它被定义为一个排他模式时，其他线程对其的获取就被阻止，而共享模式对于多个线程获取都可以成功。

    同步器是实现锁的关键，利用同步器将锁的语义实现，然后在锁的实现中聚合同步器。可以这样理解：锁的API是面向使用者的，它定义了与锁交互的公共行为，而每个锁需要完成特定的操作也是透过这些行为来完成的（比如：可以允许两个线程进行加锁，排除两个以上的线程），但是实现是依托给同步器来完成；同步器面向的是线程访问和资源控制，它定义了线程对资源是否能够获取以及线程的排队等操作。锁和同步器很好的隔离了二者所需要关注的领域，严格意义上讲，同步器可以适用于除了锁以外的其他同步设施上（包括锁）。
同步器的开始提到了其实现依赖于一个FIFO队列，那么队列中的元素Node就是保存着线程引用和线程状态的容器，每个线程对同步器的访问，都可以看做是队列中的一个节点。

获取锁，acquire(int arg)的主要逻辑包括：

1. 尝试获取（调用tryAcquire更改状态，需要保证原子性）；

    在tryAcquire方法中适用了同步器提供的对state操作的方法，利用compareAndSet保证只有一个线程能够对状态进行成功修改，而没有成功修改的线程将进入sync队列排队。

2. 如果获取不到，将当前线程构造成节点Node并加入sync队列；

    进入队列的每个线程都是一个节点Node，从而形成了一个双向队列，类似CLH队列，这样做的目的是线程间的通信会被限制在较小规模（也就是两个节点左右）。

3. 再次尝试获取，如果没有获取到那么将当前线程从线程调度器上摘下，进入等待状态。

释放锁，release(int arg)的主要逻辑包括：

1. 尝试释放状态；

    tryRelease能够保证原子化的将状态设置回去，当然需要使用compareAndSet来保证。如果释放状态成功之后，就会进入后继节点的唤醒过程。

2. 唤醒当前节点的后继节点所包含的线程。

    通过LockSupport的unpark方法将休眠中的线程唤醒，让其继续acquire状态。

回顾整个资源的获取和释放过程：

在获取时，维护了一个sync队列，每个节点都是一个线程在进行自旋，而依据就是自己是否是首节点的后继并且能够获取资源；

在释放时，仅仅需要将资源还回去，然后通知一下后继节点并将其唤醒。

这里需要注意，队列的维护（首节点的更换）是依靠消费者（获取时）来完成的，也就是说在满足了自旋退出的条件时的一刻，这个节点就会被设置成为首节点。

## 2.1 ConcurrentHashMap

    ConcurrentHashMap是线程安全的HashMap的实现，默认构造同样有initialCapacity和loadFactor属性，不过还多了一个concurrencyLevel属性，三属性默认值分别为16、0.75及16。其内部使用锁分段技术，维持这锁Segment的数组，在Segment数组中又存放着Entity[]数组，内部hash算法将数据较均匀分布在不同锁中。

put操作：并没有在此方法上加上synchronized，首先对key.hashcode进行hash操作，得到key的hash值。hash操作的算法和map也不同，根据此hash值计算并获取其对应的数组中的Segment对象(继承自ReentrantLock)，接着调用此Segment对象的put方法来完成当前操作。

ConcurrentHashMap基于concurrencyLevel划分出了多个Segment来对key-value进行存储，从而避免每次put操作都得锁住整个数组。在默认的情况下，最佳情况下可允许16个线程并发无阻塞的操作集合对象，尽可能地减少并发时的阻塞现象。

get(key)

    首先对key.hashCode进行hash操作，基于其值找到对应的Segment对象，调用其get方法完成当前操作。而Segment的get操作首先通过hash值和对象数组大小减1的值进行按位与操作来获取数组上对应位置的HashEntry。在这个步骤中，可能会因为对象数组大小的改变，以及数组上对应位置的HashEntry产生不一致性，那么ConcurrentHashMap是如何保证的？

    对象数组大小的改变只有在put操作时有可能发生，由于HashEntry对象数组对应的变量是volatile类型的，因此可以保证如HashEntry对象数组大小发生改变，读操作可看到最新的对象数组大小。

    在获取到了HashEntry对象后，怎么能保证它及其next属性构成的链表上的对象不会改变呢？这点ConcurrentHashMap采用了一个简单的方式，即HashEntry对象中的hash、key、next属性都是final的，这也就意味着没办法插入一个HashEntry对象到基于next属性构成的链表中间或末尾。这样就可以保证当获取到HashEntry对象后，其基于next属性构建的链表是不会发生变化的。

    ConcurrentHashMap默认情况下采用将数据分为16个段进行存储，并且16个段分别持有各自不同的锁Segment，锁仅用于put和remove等改变集合对象的操作，基于volatile及HashEntry链表的不变性实现了读取的不加锁。这些方式使得ConcurrentHashMap能够保持极好的并发支持，尤其是对于读远比插入和删除频繁的Map而言，而它采用的这些方法也可谓是对于Java内存模型、并发机制深刻掌握的体现。

 

## 2.2 ReentrantLock

    在并发包的开始部分介绍了volatile特性及AQS同步器，而这两部分正是ReentrantLock实现的基础。通过上面AQS的介绍及原理分析，可知道是以volatile维持的int类型的state值，来判断线程是执行还是在syn队列中等待。

ReentrantLock的实现不仅可以替代隐式的synchronized关键字，而且能够提供超过关键字本身的多种功能。

    这里提到一个锁获取的公平性问题，如果在绝对时间上，先对锁进行获取的请求一定被先满足，那么这个锁是公平的，反之，是不公平的，也就是说等待时间最长的线程最有机会获取锁，也可以说锁的获取是有序的。ReentrantLock这个锁提供了一个构造函数，能够控制这个锁是否是公平的。

    对于公平和非公平的定义是通过对同步器AbstractQueuedSynchronizer的扩展加以实现的，也就是tryAcquire的实现上做了语义的控制。

    公平和非公平性的更多原理分析见于http://ifeve.com/reentrantlock-and-fairness/ 

 

## 2.3 Condition

    Condition是并发包中提供的一个接口，典型的实现有ReentrantLock，ReentrantLock提供了一个mewCondition的方法，以便用户在同一个锁的情况下可以根据不同的情况执行等待或唤醒动作。典型的用法可参考ArrayBlockingQueue的实现，下面来看ReentrantLock中

newCondition的实现。

ReentrantLock.newCondition()

    创建一个AbstractQueuedSynchronizer的内部类ConditionObject的对象实例。

ReentrantLock.newCondition().await()

    将当前线程加入此condition的等待队列中，并将线程置为等待状态。

ReentrantLock.newCondition().signal()

    从此condition的等待队列中获取一个等待节点，并将节点上的线程唤醒，如果要唤醒全部等待节点的线程，则调用signalAll方法。

 

## 2.4 CopyOnWriteArrayList

    CopyOnWriteArrayList是一个线程安全、并且在读操作时无锁的ArrayList，其具体实现方法如下。

CopyOnWriteArrayList()

    和ArrayList不同，此步的做法为创建一个大小为0的数组。

add(E)

    add方法并没有加上synchronized关键字，它通过使用ReentrantLock来保证线程安全。此处和ArrayList的不同是每次都会创建一个新的Object数组，此数组的大小为当前数组大小加1，将之前数组中的内容复制到新的数组中，并将

新增加的对象放入数组末尾，最后做引用切换将新创建的数组对象赋值给全局的数组对象。

remove(E)

    和add方法一样，此方法也通过ReentrantLock来保证其线程安全，但它和ArrayList删除元素采用的方式并不一样。

    首先创建一个比当前数组小1的数组，遍历新数组，如找到equals或均为null的元素，则将之后的元素全部赋值给新的数组对象，并做引用切换，返回true；如未找到，则将当前的元素赋值给新的数组对象，最后特殊处理数组中的最后

一个元素，如最后一个元素等于要删除的元素，即将当前数组对象赋值为新创建的数组对象，完成删除操作，如最后一个元素也不等于要删除的元素，那么返回false。

    此方法和ArrayList除了锁不同外，最大的不同在于其复制过程并没有调用System的arrayCopy来完成，理论上来说会导致性能有一定下降。

get(int)    

    此方法非常简单，直接获取当前数组对应位置的元素，这种方法是没有加锁保护的，因此可能会出现读到脏数据的现象。但相对而言，性能会非常高，对于写少读多且脏数据影响不大的场景而言是不错的选择。

iterator()

    调用iterator方法后创建一个新的COWIterator对象实例，并保存了一个当前数组的快照，在调用next遍历时则仅对此快照数组进行遍历，因此遍历此list时不会抛出ConcurrentModificatiedException。

    与ArrayList的性能对比，在读多写少的并发场景中，较之ArrayList是更好的选择，单线程以及多线程下增加元素及删除元素的性能不比ArrayList好

 

## 2.5 CopyOnWriteArraySet

    CopyOnWriteArraySet基于CopyOnWriteArrayList实现，其唯一的不同是在add时调用的是CopyOnWriteArrayList的addIfAbsent方法。保证了无重复元素，但在add时每次都要进行数组的遍历，因此性能会略低于上个。

 

## 2.6 ArrayBlockingQueue

## 2.7 ThreadPoolExecutor

与每次需要时都创建线程相比，线程池可以降低创建线程的开销，在线程执行结束后进行的是回收操作，提高对线程的复用。Java中主要使用的线程池是ThreadPoolExecutor，此外还有定时的线程池ScheduledThreadPoolExecutor。

Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。

比较重要的几个类：

ExecutorService	真正的线程池接口
ScheduledExecutorService	和Time/TimeTask类似，解决需要任务重复执行的问题
ThreadPoolExecutor	ExecutorService的默认实现
SchedulesThreadPoolExecutor	继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现
要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在Executors类里面提供了一些静态工厂，生成一些常用的线程池。

### 1. newSingleThreadExecutor

创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

### 2.newFixedThreadPool

创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

### 3. newCachedThreadPool

创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，

那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

### 4.newScheduledThreadPool

创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

PS：但需要注意使用，newSingleThreadExecutor和newFixedThreadPool将超过处理的线程放在队列中，但工作线程较多时，会引起过多内存被占用，而后两者返回的线程池是没有线程上线的，所以在使用时需要当心，创建过多的线程容易引起服务器的宕机。

使用ThreadPoolExecutor自定义线程池，具体使用时需根据系统及JVM的配置设置适当的参数，下面是一示例：
```
1 int corePoolSize = Runtime.getRuntime().availableProcessors();
2 threadsPool = new ThreadPoolExecutor(corePoolSize, corePoolSize, 10l, TimeUnit.SECONDS,
3                new LinkedBlockingQueue<Runnable>(2000));
```
## 2.8 Future和FutureTask

Future是一个接口，FutureTask是一个具体实现类。这里先通过两个场景看看其处理方式及优点。

场景1,

现在通过调用一个方法从远程获取一些计算结果，假设有这样一个方法：

1 HashMap data = getDataFromRemote();
如果是最传统的同步方式的使用，我们将一直等待getDataFromRemote()的返回，然后才能继续后面的工作。这个函数是从远程获取数据的计算结果的，如果需要的时间很长，并且后面的那部分代码与这些数据没有关系的话，阻塞在这里等待结果就会比较浪费时间。如何改进呢？

能够想到的办法就是调用函数后马上返回，然后继续向下执行，等需要用数据时再来用或者再来等待这个数据。具体实现有两种方式：一个是用Future，另一个使用回调。

Future的用法
```
1 Future<HashMap> future = getDataFromRemote2();
2 //do something
3 HashMap data = future.get();
```
可以看到，我们调用的方法返回一个Future对象，然后接着进行自己的处理，后面通过future.get()来获取真正的返回值。也即，在调用了getDataFromRemote2后，就已经启动了对远程计算结果的获取，同时自己的线程还在继续处理，直到需要时再获取数据。来看一下getDataFromRemote2的实现：
```
privete Future<HashMap> getDataFromRemote2(){
    return threadPool.submit(new Callable<HashMap>(){
        public HashMap call() throws Exception{
            return getDataFromRemote();
        }
    });
}
```
可以看到，在getDataFromRemote2中还是使用了getDataFromRemote来完成具体操作，并且用到了线程池：把任务加入到线程池中，把Future对象返回出去。我们调用了getDataFromRemote2的线程，然后返回来继续下面的执行，而背后是另外的线程在进行远程调用及等待的工作。get方法也可设置超时时间参数，而不是一直等下去。

场景2，

key-value的形式存储连接，若key存在则获取，若不存在这个key，则创建新连接并存储。

传统的方式会使用HashMap来存储并判断key是否存在而实现连接的管理。而这在高并发的时候会出现多次创建连接的现象。那么新的处理方式又是怎样呢？

通过ConcurrentHashMap及FutureTask实现高并发情况的正确性，ConcurrentHashMap的分段锁存储满足数据的安全性又不影响性能，FutureTask的run方法调用Sync.innerRun方法只会执行Runnable的run方法一次(即使是高并发情况)。

## 2.9 并发容器

在JDK中，有一些线程不安全的容器，也有一些线程安全的容器。并发容器是线程安全容器的一种，但是并发容器强调的是容器的并发性，也就是说不仅追求线程安全，还要考虑并发性，提升在容器并发环境下的性能。

加锁互斥的方式确实能够方便地完成线程安全，不过代价是降低了并发性，或者说是串行了。而并发容器的思路是尽量不用锁，比较有代表性的是以CopyOnWrite和Concurrent开头的几个容器。CopyOnWrite容器的思路是在更改容器的时候，把容器写一份进行修改，保证正在读的线程不受影响，这种方式用在读多写少的场景中会非常好，因为实质上是在写的时候重建了一次容器。而以Concurrent开头的容器的具体实现方式则不完全相同，总体来说是尽量保证读不加锁，并且修改时不影响读，所以达到比使用读写锁更高的并发性能。比如上面所说的ConcurrentHashMap，其他的并发容器的具体实现，可直接分析JDK中的源码。