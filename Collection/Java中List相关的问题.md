### 																																							Java中List相关的问题

**[极度推荐的集合源码阅读系列](https://www.cnblogs.com/tong-yuan/p/10810042.html)**



#### 1.ArrayList 和LinkedList有什么区别？

+ 两者都实现了List; ArrayList 实现了RandoemAccess接口，底层是基于动态数组(Object[] elementData)，初始容量为10，每次扩容会增大到之前的1.5倍(newCapacity = oldCapacity + (oldCapacity >> 1));LinkedList实现的是Queue和Deque，底层是基于双向链表(Node<E> first,Node<E>last),LinkedList既可以做List,也可以作为双端队列，还可以作为栈。

+ 对于随机访问get和set ,Arraylist要优于LinkedList,原因是LinkedList要移动指针。

+ 元素的插入删除的效率，要视插入删除的位置分析，各有优劣：

  - **列表首部**操作元素，LinkedList > ArrayList，原因是ArrayList要移动每个元素的索引和数组扩容（删除操作除外），而LinkedList则直接增加元素，修改first即可(源码调用linkFirst(E e))。
  - **列表中间**操作元素，ArrayList 性能耗损在移动位置之后的元素索引，LinkedList耗损在for循环检索该位置的元素。位置靠前 :LinkedList > ArrayList。
  - **列表末尾**操作元素，性能相差不大。



#### 2.ArrayList是如何实现扩容的？

  + **add(E e):**   从尾部进行插入

    ````java
    public boolean add(E e) {
            modCount++;                //操作数+1
            add(e, elementData, size); //默认从数组的实际大小的后面进行插入
            return true;
        }
    private void add(E e, Object[] elementData, int s) {
            if (s == elementData.length) //如果size ==length 说明数据的容量已经满了
                elementData = grow();		// 则需要扩容
            elementData[s] = e;       	//将数据插入到最后
            size = s + 1;								
        }
     private Object[] grow() {
            return grow(size + 1);
        }
    private Object[] grow(int minCapacity) {
      			//创立数组，数组的length是从newCapacity(minCapacity)获取新长度，将原数组的数据放入到新数组中
            return elementData = Arrays.copyOf(elementData,
                                               newCapacity(minCapacity));
      	/*
      		1.newCapacity(minCapacity)返回的是扩容后需要的length长度
      		2.Arrays.copyof(array,length)底层调用的是System.arraycopy(src(原数组),srcPos,dest(目标数组）,destPos,Math.min(src.length,dest.length))
      	*/
        }
    
    private int newCapacity(int minCapacity) {
            // overflow-conscious code
            int oldCapacity = elementData.length;
      			//新容量为旧容量的1.5倍 (oldCapacity >>1) =oldCapacity/2
            int newCapacity = oldCapacity + (oldCapacity >> 1);
      			//如果新容量还是比需要的容量更小，以需要的容量为准
            if (newCapacity - minCapacity <= 0) {
                if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                    return Math.max(DEFAULT_CAPACITY, minCapacity);
                if (minCapacity < 0) // overflow
                    throw new OutOfMemoryError();
                return minCapacity;
            }
      			//进行超大容量处理  MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
            return (newCapacity - MAX_ARRAY_SIZE <= 0)
                ? newCapacity
                : hugeCapacity(minCapacity);
        }
    		//最大分配 Integer.MAX_VALUE
    private static int hugeCapacity(int minCapacity) {
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return (minCapacity > MAX_ARRAY_SIZE)
                ? Integer.MAX_VALUE
                : MAX_ARRAY_SIZE;
        }
    // PS:MAX_ARRAY_SIZE 为什么是 Integer.MAX_VALUE - 8呢？
    //数组对象有一个额外的元数据，用于表述数组的大小，因为虚拟机需要保存数组的header words 在内存中，额外的8个字节用来描述数组的信息。如果全部分配可能会导致OOM
    ````

+  **add(int index,E element):**    从指定的位置进行插入

+  ````java
   public void add(int index, E element) {
           rangeCheckForAdd(index);   //判断index 是否越界
     																//index > size || index < 0
           modCount++;
           final int s;
           Object[] elementData;
    				//size == length ? 扩容：不变
           if ((s = size) == (elementData = this.elementData).length)
               elementData = grow();  //扩容
           System.arraycopy(elementData, index,
                            elementData, index + 1,
                            s - index);
     			/*
     				System.arraycopy用法：
     				从索引index后的所有数据，都向后移动一位
     				eg:  1 2 3 4 5  //index =2
     				-->  1 2 3 3 4 5 
     			*/
           elementData[index] = element;  //替换下标为index的值
           size = s + 1;
       }
   ````



#### 3.ArrayList插入、删除、查询元素的时间复杂度各是多少？

+ add(E element) 最快O(1),最慢O(n)因为数组必须要扩容和移动
+ add(int index,E element) O(n)
+ remove(int index)  O(n)
+ get(int index) O(1)



#### 4.怎么求两个集合的并集、交集、差集？

+ **addAll(Collection<? extents E> c )**  : 求两个集合的并集：

  - 拷贝c中的元素到数组a中；

  - 检查是否需要扩容；

  - 把数组a中的元素拷贝到elementData的尾部；

+ ````java
  public boolean addAll(Collection<? extends E> c) {
          Object[] a = c.toArray();  //将集合c转换为数组
          modCount++;
          int numNew = a.length;   //获取要合并集合的长度
          if (numNew == 0)
              return false;
          Object[] elementData;
          final int s;
    /* 
    			这一步的意思是检查是否需要扩容：
    			  首先获取原数组还剩多少个空余位置（element.length -size）
    			  其次判断要合并的集合的长度是否大于空余位置。大于，则说明需要扩容
    */
          if (numNew > (elementData = this.elementData).length - (s = size))
              elementData = grow(s + numNew);
   				//将c中的元素全部拷贝到数组的最后
          System.arraycopy(a, 0, elementData, s, numNew);
    			//size增大c的大小
          size = s + numNew;
          return true;
      }
  ````

+ **retailAll(Collection<?> c)** :求两个集合的交集：

  - 遍历elementData数组；

  - 如果元素在c中，则把这个元素添加到elementData数组的w位置并将w位置往后移一位；

    （3）遍历完之后，w之前的元素都是两者共有的，w之后（包含）的元素不是两者共有的；

    （4）将w之后（包含）的元素置为null，方便GC回收；

    ````java
     public boolean retainAll(Collection<?> c) {
       			//当complement =true 时，表示删除不包含c中的元素。
            return batchRemove(c, true, 0, size);  
        }
    		/*
    			使用两个指针r,w,同时遍历c中不包含的元素。
    			先找到第一个不包含的c集合的数据，下标赋值给r,w=r++;
    			读指针每次自增1，写指针放入数据的时候才加1
    			这样不需要额外的空间，只需要在原有的数组上操作就可以了
    		*/
        boolean batchRemove(Collection<?> c, boolean complement,
                            final int from, final int end) {
          	//集合c是否为空
            Objects.requireNonNull(c);
            final Object[] es = elementData;
            int r;
            // Optimize for initial run of survivors
           	//这个for循环的意义时找到第一个不包含c集合的数据，并存入到r指针中。
            for (r = from;; r++) {
                if (r == end)
                    return false;
                if (c.contains(es[r]) != complement)
                    break;
            }
           //w指针从r指针开始
            int w = r++;
            try {
              	// 这个for循环的意义是，从r开始，依次遍历整个数组中的值
              	//如果c集合包含了该元素，则把该元素放入写指针的位置上
                for (Object e; r < end; r++)
                    if (c.contains(e = es[r]) == complement)
                        es[w++] = e;
              //c.contains()可能抛出异常
            } catch (Throwable ex) {
                // Preserve behavioral compatibility with AbstractCollection,
                // even if c.contains() throws.
              	//把未读的元素都拷贝到写指针之后
                System.arraycopy(es, r, es, w, end - r);
                w += end - r;
                throw ex;
            } finally {
                modCount += end - w;
              //将w-end 节点的数据清空
                shiftTailOverGap(es, w, end);
            }
            return true;
        }
    ````

+ **removeAll(Collection<?> c)**  求两个集合的差集：

  ````java
  public boolean removeAll(Collection<?> c) {
      // 同样调用批量删除方法，这时complement传入false，表示删除包含在c中的元素
       return batchRemove(c, false, 0, size);
  }
  ````



#### 5.ArrayList是怎么实现序列化和反序列化的?

+ **writeObject():**

````java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out element count, and any hidden stuff
  			//防止序列化途中，数组有修改
        int expectedModCount = modCount;
 				//先序列化默认的数据（非 static 非 transient）
        s.defaultWriteObject();

        // Write out size as capacity for behavioral compatibility with clone()
  			//写出元素个数
        s.writeInt(size);

        // Write out all elements in the proper order.
  			//写出元素
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
				
  			//请看第七个问题
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

````

+ **readObject()**:

````java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
    // 声明为空数组
    elementData = EMPTY_ELEMENTDATA;

    // 读入非transient非static属性（会读取size属性）
    s.defaultReadObject();

    // 读入元素个数，没什么用，只是因为写出的时候写了size属性，读的时候也要按顺序来读
    s.readInt();

    if (size > 0) {
        // 计算容量
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        // 检查是否需要扩容
        ensureCapacityInternal(size);
        
        Object[] a = elementData;
        // 依次读取元素到数组中
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
````



#### 6.集合的方法toArray()有什么问题?

+ toArray()返回的类型不一定是Object[] 其类型取决于其返回的实际类型。当有子类override父类并改变了返回的类型时，最终返回为子类重写的类型,当转换成Object[]时会抛出*ArrayStoreException*

  ````java
  public static void main(String[] args) {
          List<String> strList = new MyList();
          // 打印结果为class [Ljava.lang.String;
          System.out.println(strList.toArray().getClass());
      }
  }
  class MyList extends ArrayList<String>{
      public String[] toArray() {
          return new String[]{"1", "2", "3"};
      }
  }
  ````



#### 7.什么是fail-fast？

[参考这里：](https://blog.csdn.net/ch717828/article/details/46892051)

+ **说明**：Fail-fast是一种错误检测机制，当一个或多个线程遍历集合时，某几个线程对集合进行结构上的改变（删除，修改，新增)，那么此时程序就会抛出ConcurrentModificationException 异常。

+ **实现**：

  迭代器在遍历过程中是直接访问内部数据的，因此内部的数据在遍历的过程中无法被修改。为了保证不被修改，迭代器内部维护了一个标记 “**expectedModCount**” ，当集合结构改变，标记"**modCount**"会被修改，而迭代器每次的hasNext()和next()方法都会检查`modCount == expectedModCount`，当不等于时，抛出ConcurrentModificationException。

  以下是源码：

  ````java
  //删除不必要的代码
  private class Itr implements Iterator<E> {
         	//首先获取到遍历之前的modCount。
          int expectedModCount = modCount;
          public E next() {
            //在遍历的时候检查modCount == expectedModCount
            //不等于就抛出  ConcurrentModificationException
              checkForComodification();
              try {
                 	//省略不重要的代码
              } catch (IndexOutOfBoundsException e) {
                  checkForComodification();
                  throw new NoSuchElementException();
              }
          }
          public void remove() {
              if (lastRet < 0)
                  throw new IllegalStateException();
              checkForComodification();
              try {
                //省略不重要的代码
             //在单线程情况下，remove()是不会抛出异常，
             //因为它会让expectModcount 和modCount相等。
                expectedModCount = modCount;
              } catch (IndexOutOfBoundsException e) {
                  throw new ConcurrentModificationException();
              }
          }
  ````

  

  checkForComodification()的实现：

  ````java
  final void checkForComodification() {
              if (modCount != expectedModCount)
                  throw new ConcurrentModificationException();
          }
      }
  ````



#### 8.LinkedList是单链表还是双链表实现的?

+ LinkedList是双链表实现的,源码定义了First 和 last两个节点分别代表了头位置 和 尾位置。

  ````java
  /**
       * Pointer to first node.
       */
      transient Node<E> first;
      /**
       * Pointer to last node.
       */
      transient Node<E> last;
  ````



#### 9.LinkedList除了作为List还有什么用处？

+ 作为双端队列或者栈来使用



#### 10.LinkedList插入、删除、查询元素的时间复杂度各是多少？

+ get(int index) **O(n):**平均是n/4
+ add(E element) :**O(1)**
+ add(int index,E element)**:O(n)**平均是**O(n/4)** 当index=0 或index =last 是O(1)
+ remove(int index):**O(n)**



#### 11.什么是随机访问？

+ 可以通过下标直接定位到某一个元素存放的位置。



#### 12.哪些集合支持随机访问？他们都有哪些共性？

+ 实现了RandomAccess和List两个接口的都支持随机访问。



#### 13.CopyOnWriteArrayList是怎么保证并发安全的?

+ CopyOnWriteArrayList 在写操作的时候，会将list数组**复制一份**作为缓存，然后对该缓存中的数组进行操作，此时若有其他线程读的话，那么该线程读取的还是原先没有修改过的数组，若有新的线程写的话，那么该线程会被阻塞锁在外面。操作完毕后再将list数组地址引用指向修改后的新数组地址。
+ 读操作是无锁的。
+ CopyOnWriteArrayList 现在用Synchronized 来替换之前的ReentrantLock。



#### 14.CopyOnWriteArrayList的实现采用了什么思想？

+ 采用了**读写分离**的思想，容器允许并发读，读操作无锁，性能较高。写操作，首先将当前容器复制一份，然后在新副本上执行写操作，结束后再将原容器的引用指向新容器。



#### 15.CopyOnWriteArrayList是不是强一致性的？

+  ==强一致性==：任意时刻，所有节点中的数据是一样的。同一时间点，在节点A中获取到key1的值与在节点B中获取到key1的值应该都是一样的。任何一次读都能读到某个数据的最近一次写的数据，系统中的所有进程，看到的操作顺序，都和全局时钟下的顺序一致。

+  CopyOnWriteArrayList**只保证最终一致性，不保证实时一致性**；



#### 16.CopyOnWriteArrayList适用于什么样的场景？

+ 适用于**读多写少**的并发场景



#### 17.CopyOnWriteArrayList插入、删除、查询元素的时间复杂度各是多少？

+  Read：O(1)
+ Write：O(n)
+ Remove：O(n)



#### 18.CopyOnWriteArrayList为什么没有size属性？

+ ​	因为每次修改都是拷贝一份整哈存储目标个数的数组，所以不需要size属性了，数组的长度就是集合的大小，而不像ArrayList的长度是要大于集合的大小的。
+ e g: add(E e)操作，先拷贝一份n+1个数组，再把新元素放到新数组的最后一位，这时新数组的长度为len+1了，也就是集合的size了。



#### 19.比较古老的集合Vector和Stack有什么缺陷？

+ Vector对每个独立操作都实现了**同步**，即某一时刻只有一个线程能够写Vector,避免多线程同时写而引起的不一致性，但是，加锁的操作需要很大的花费，很多时候是得不偿失的。这就导致了性能非常低。
+ Stack继承了Vector,性能不是很好。
+ Vector包含了大量集合处理的方法。Stack复用了Vector方法来实现push和pop。但是，只是为了复用简单的方法而继承Vector。使得Stack在基于数组实现上效率受影响。
+ Stack可以复用vector大量方法，使得在设计上不严谨。





####参考如下：

> https://www.cnblogs.com/leefreeman/archive/2013/05/16/3082400.html
>
> https://www.cnblogs.com/tong-yuan/p/LinkedList.html
>
> [https://cnblogs.com/tong-yuan/p/10638902.html](https://www.cnblogs.com/tong-yuan/p/10638902.html)



