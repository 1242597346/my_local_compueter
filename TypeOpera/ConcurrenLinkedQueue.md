# ConcurrenLinkedQueue:

## 	1.concurrentlinkedqueue的简单了解和用途

​			继承的是AbstractQueue接口，实现的是Queue接口。

​			使用的是单向链表的形式来实现队列  

![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\concurrentlinkedqueue_headFile.png)

​			![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\concurrentlinkedqueue_jicheng.png)

## 2.ConcurrentLinkedQueue的源码解读

定义一个Node方法体，使用volatile对元素item和next域进行修饰，这样就可以修改到主内存，全部线程可见。

![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\ConcurrentLinkedQueue_functionBody.png)

从上图可以看到，在Node构造函数中，使用了unsafe类，分别设置、比较并替换了item和next域的值。但是对next域使用了unsafe.putOrderedObject方法，导致对next域的修改并不会对其他线程立即可见。使用了unsafe类的cas算法保证了出入队列的一致性。CAS其实是一条CPU原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子的更新某个位置的值，__即cas是基于硬件平台的，JVM封装了汇编调用，AtomicInteger类则使用了这些封装后的接口(atomic底层提供的是volatile和cas来实现的对数据的更改__，volatile实现了数据修改的主内存可见，禁止重排序，cas比较并替换实现了数据更新的原子性)。

**而java的原子类是通过Unsafe类实现的，所以简单了解一下unsafe类。主要提供的是执行级别较低、且不安全操作的方法(java没有为我们提供unsafe类的对外API),例如：访问和管理系统内存资源。**

![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\java-thread-x-atomicinteger-unsafe.png)

**ConcurrentLinkedQueue中头尾节点的定义：**concurrentlinkedqueue持有head和tail头尾指针管理队列，用来存放队首和队尾的节点信息。

![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\concurrentlinkedqueue_head.png)

![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\concurrentlinkedqueue_tail.png)

head和tail除了使用volatile进行原子修饰以外，还使用了transient进行修饰。transient的作用主要是使其修饰的变量可以不被序列化(transient只能修饰变量，同时其所在的类需要继承serializable)。

**类的构造函数，对head和tail进行初始化，指向一个空的域。**

![](D:\Users\Sangfor\Desktop\TypeOpera\pictures\concurrentlinkedqueue_initial_head_tail.png)

