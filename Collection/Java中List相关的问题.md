### 																					Java中List相关的问题

**[极度推荐的集合源码阅读系列](https://www.cnblogs.com/tong-yuan/p/10810042.html)**

#### 1.ArrayList 和LinkedList有什么区别？

+ 两者都实现了List; ArrayList 实现了RandoemAccess接口，底层是基于动态数组(Object[] elementData)，初始容量为10，每次扩容会增大到之前的1.5倍(newCapacity = oldCapacity + (oldCapacity >> 1));LinkedList实现的是Queue和Deque，底层是基于双向链表(Node<E> first,Node<E>last),LinkedList既可以做List,也可以作为双端队列，还可以作为栈。

+ 对于随机访问get和set ,Arraylist要优于LinkedList,原因是LinkedList要移动指针。

+ 元素的插入删除的效率，要视插入删除的位置分析，各有优劣：

  - **列表首部**操作元素，LinkedList > ArrayList，原因是ArrayList要移动每个元素的索引和数组扩容（删除操作除外），而LinkedList则直接增加元素，修改first即可(源码调用linkFirst(E e))。
  - **列表中间**操作元素，ArrayList 性能耗损在移动位置之后的元素索引，LinkedList耗损在for循环检索该位置的元素。位置靠前 :LinkedList > ArrayList。
  - **列表末尾**操作元素，性能相差不大。


![LinkedList和ArrayList时间复杂度总结](/Users/xujie/Desktop/屏幕快照 2019-08-12 03.01.23.png)