# 			                                   List Queue(Deque) Map Set 

# 			        				                   的使用及部分源码的分析



## 脉络：

结合自己看过的jdk源码，和网上关于集合源码的分析并结合平时在项目中对其的使用，介绍大部分常用的集合及其源码

## List:

主要介绍以下四种集合类

+ **`Arraylist` ★** 
+ **`LinkedList` ★** (放到Queue章节)
+ **`Vector`**  
+ **`CopyOnWriteArrayList`**

### ArrayList：

#### 概要介绍：

​	ArrayList是一个数组队列可以动态增长和缩减的索引序列 

+ 实现了[List](https://stackoverflow.com/questions/2165204/why-does-linkedhashsete-extend-hashsete-and-implement-sete) 接口 并继承 AbstractList  类，提供基础的添加、删除、遍历等操作
+ 实现 RandomAcess 提供随机访问的能力
+ 实现 Cloneable 提供克隆的能力
+ 实现 Serializable 提供被序列化的能力

![image-20201217150830665](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201217151802014-762388560.png)

#### 源码解析：

##### 属性值：

````java
/* 真正存放元素的地方 ，使用transient修饰，为了不序列化这个字段 */
transient Object[] elementData; // non-private to simplify nested class access
````
````java
/* 真正存储元素个数的大小，并不是elementData数组的长度 */
private int size; 
````
````java
/* 缺省容量，如果构造器中传入的容量大小是0，那么下一次添加元素扩容时，容量就会使用这个参数进行扩容 */
private static final int DEFAULT_CAPACITY = 10;
````

````java
/* 空数组，通过构造器 new ArrayList<>(0) 创建时，使用该数组 */
private static final Object[] EMPTY_ELEMENTDATA = {};
````

````java
/* 空数组，通过构造器 new ArrayList<>() 创建时，使用该数组 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
````


````java
/* 版本号 */
private static final long serialVersionUID = 8683452581122892189L;
````

##### 构造器：

​	共有三种构造器：

+ 无参构造器
+ 有参构造器(设置初始容量)
+ 有参构造器(导入集合)

![image-20201217154557199](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201217154600520-785346400.png)

###### 无参构造器：

````java
public ArrayList() {
    // 如果未传初始容量，则使用属性数组 DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    // 使用该数组时，添加第一个元素的时候会扩容到默认大小10
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
````

###### 有参构造器(设置初始容量)：

````java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) { 
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
````

###### 有参构造器(导入集合)：

````java
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray(); //集合转数组
        if ((size = elementData.length) != 0) {
            // 如果失败，检查c.toArray()是否是Object[]类型，不是则重新拷贝为Object[].class
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
           //参考 http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6260652
        } else {
            // 集合为空，那么使用空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
````

##### 核心方法：

+ add:

  - `add(E e)`

  - `add(int index,E element)`

  - `addAll(Collection<? extends E> c)`

  - `addAll(int index, Collection<? extends E> c)`

![image-20201217161022167](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201217161025578-1367285096.png)

+ remove:

  - remove(int index)
  - remove(Object o)
  - removeAll(Collection<?> c) 
  - retainAll(Collection<?>  c）

  ![image-20201218134117598](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201218134122179-352208490.png)

+ iterator:

  - Itr
  - ListItr
  - ArrayListSpliterator

+ get() set() indexOf(Object) contains(Object) 等常用的方法不予介绍

###### add(E e): 默认在末尾添加元素

````java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 判断是否需要扩容
        elementData[size++] = e; // 元素插入到最后一位
        return true;
    }
````

````java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
                        ||
                        \/
private static int calculateCapacity(Object[] elementData, int minCapacity) {
     //若为空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，就初始化为默认大小10
     if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
      }
      return minCapacity;
}
````

````java
private void ensureExplicitCapacity(int minCapacity) {
       //这里为了实现 fail-fast 机制
        modCount++;

        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }           
                    ||
                    \/
private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        //扩容大小：newCapacity = oldCapacity + oldCapacity * 0.5
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 使用减号为了防止数值溢出为负
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
                    ||
                    \/
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // 如果溢出了
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
````

*快速失败：*

​	**说明：**系统运行中，如果有错误发生，那么系统立即结束，这种设计就是快速失败，java 集合中有错误存在是在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了增加、删除、修改操作，则会抛出ConcurrentModificationException

​	**实现： **迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 `expectedModCount`变量保存 `modCount` 值。集合在被遍历期间如果内容发生变化，就会改变`modCount`的值。每当迭代器使用 hashNext() 或 next()遍历下一个元素时，都会检测modCount变量是否为`expectedmodCount`值，是的话就继续遍历；否则抛出ConcurrentModificationException异常，并终止遍历

​	**补充：**快速失败不能保证每次都会抛出 ConcurrentModificationException异常，因为异常的抛出条件是 `modCount != expectedModCount`,假如集合结构进行修改时 modCount 刚好又设置为expectModCount 那么异常就不会抛出了。所以，不能依赖 fail-fast 机制进行并发编程的操作

*[使用减号为了防止数值溢出为负：](https://stackoverflow.com/questions/33147339/difference-between-if-a-b-0-and-if-a-b)* 

````java
public static void main(String[] args) {
        int oldCapacity = Integer.MAX_VALUE - 16;
        System.out.println(oldCapacity);
        int newCapacity = oldCapacity + oldCapacity >> 1;
        System.out.println(newCapacity);
        /*
        * 输出：
        *  2147483631
        *  -17
        * */
    }
````

###### add(int index,E e): 在特定位置添加元素

````java
public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // 与上面的逻辑一致
    	
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }                   
                       ||
                       \/
// System提供了一个native 静态方法，arraycopy()可以使用它来实现数组之间的复制
// 由于System.arrayCopy是对内存直接进行复制，减少了for循环下需要的寻址时间，在大数据量的前提下提高了性能
public static native void arraycopy(Object src,  //原数组
                                    int  srcPos, //源数组需要复制的起始位置
                                    Object dest, //目标数组
                                    int destPos, //目标数组放置数据的起始位置
                                    int length); // 要复制的长度

````

*System.arrayCopy 分析:*

+ `System.arraycopy`为 JVM 内部固有方法，它通过手工编写汇编或其他优化方法来进行 Java 数组拷贝，这种方式比起直接在 Java 上进行 for 循环或 clone 是更加高效的。数组越大体现地越明显。

+ openjdk1.8-src/hotspot/src/share/vm/prims/**jvm**.cpp

  ````c++
  JVM_ENTRY(void, JVM_ArrayCopy(JNIEnv *env, jclass ignored, jobject src, jint src_pos,
                                 jobject dst, jint dst_pos, jint length))
    JVMWrapper("JVM_ArrayCopy");
    if (src == NULL || dst == NULL) {
      THROW(vmSymbols::java_lang_NullPointerException());
    }
    arrayOop s = arrayOop(JNIHandles::resolve_non_null(src));
    arrayOop d = arrayOop(JNIHandles::resolve_non_null(dst));
    assert(s->is_oop(), "JVM_ArrayCopy: src not an oop");
    assert(d->is_oop(), "JVM_ArrayCopy: dst not an oop");
  
    //上面的代码进行的是参数校验
     
    // 真正进行复制的函数 ：copy_array
    Klass::cast(s->klass())->copy_array(s, src_pos, d, dst_pos, length, thread);
  JVM_END
  ````

+ openjdk1.8-src/hotspot/src/share/vm/utilities/**typeArrayKlass**.cpp

  ````c++
  void typeArrayKlass::copy_array(arrayOop s, int src_pos, arrayOop d, int dst_pos, int length, TRAPS) {
    /* 
    	忽略参数校验的代码
     	......
    */
   
    int l2es = log2_element_size();
    int ihs = array_header_in_bytes() / wordSize;
    char* src = (char*) ((oop*)s + ihs) + ((size_t)src_pos << l2es);
    char* dst = (char*) ((oop*)d + ihs) + ((size_t)dst_pos << l2es);
    Copy::conjoint_memory_atomic(src, dst, (size_t)length << l2es);
  }
  
  ````

+ openjdk1.8-src/hotspot/src/share/vm/utilities/**copy**.cpp

  ````c++
  void Copy::conjoint_memory_atomic(void* from, void* to, size_t size) {
    /* 
    	忽略参数校验的代码
     	......
    */
    if (bits % sizeof(jlong) == 0) {
      Copy::conjoint_jlongs_atomic((jlong*) src, (jlong*) dst, size / sizeof(jlong));
    } else if (bits % sizeof(jint) == 0) {
      Copy::conjoint_jints_atomic((jint*) src, (jint*) dst, size / sizeof(jint));
    } else if (bits % sizeof(jshort) == 0) {
      Copy::conjoint_jshorts_atomic((jshort*) src, (jshort*) dst, size / sizeof(jshort));
    } else {
      // Not aligned, so no need to be atomic.
      Copy::conjoint_jbytes((void*) src, (void*) dst, size);
    }
  }
  
  /*
  	对 int 基本数据类型的数据进行复制
  	因为数组在内存中是顺序排列的，所以可以直接通过移动from 和to 两个指针来获取数组中的数据
  	主要逻辑是分成两种情况复制：向前复制和向后复制。
  	并且是通过指针遍历数组来赋值，这里进行的是值拷贝，也称为的“深拷贝”(java中对于基本数据类型的复制，没有深浅拷贝的区别)。
  */
  void _Copy_conjoint_jints_atomic(jint* from, jint* to, size_t count) {
    if (from > to) {
      jint *end = from + count;
      while (from < end)
        *(to++) = *(from++);
    }
    else if (from < to) {
      jint *end = from;
      from += count - 1;
      to   += count - 1;
      while (from >= end)
        *(to--) = *(from--);
    }
  }
  
  /*
    将*(to--) = *(from--) 换成了 os::atomic_copy64(from--,to--);
    通过内嵌汇编的方式复制 是为了适应64位的数据
  */
  void _Copy_conjoint_jlongs_atomic(jlong* from, jlong* to, size_t count) {
      if (from > to) {
        jlong *end = from + count;
        while (from < end)
          os::atomic_copy64(from++, to++);
      }
      else if (from < to) {
        jlong *end = from;
        from += count - 1;
        to   += count - 1;
        while (from >= end)
          os::atomic_copy64(from--, to--);
      }
    }
  ````

  ````c
  //atomic_copy64 内嵌汇编
    static void atomic_copy64(volatile void *src, volatile void *dst) {
  #if defined(PPC32)
      double tmp;
      asm volatile ("lfd  %0, 0(%1)\n"
                    "stfd %0, 0(%2)\n"
                    : "=f"(tmp)
                    : "b"(src), "b"(dst));
  #elif defined(S390) && !defined(_LP64)
      double tmp;
      asm volatile ("ld  %0, 0(%1)\n"
                    "std %0, 0(%2)\n"
                    : "=r"(tmp)
                    : "a"(src), "a"(dst));
  #else
      *(jlong *) dst = *(jlong *) src;
  #endif
    }
  ````
  
  ````assembly
        MOV     SI,1000H
        MOV     DI,2000H
        MOV     CX,1
  NEXT: LODSB   // 表示把DS：SI指向的存储单元装入al中
        STOSB	  // 表示把AL中的数据装入 ES:DI 指向的内存地址中
        DEC     CX
        JNZ     NEXT
  ````



###### addAll(Collection<? extends E> c) ：

````java
public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
}
````

###### addAll(int index,Collection<? extends E> c)：

````java
public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

````

###### removeAll & retainAll（Collection<?> c) ：

*分析：* `removeAll` 和`retainAll`都是调用内部方法 `batchRemove`

````java
 public boolean removeAll(Collection<?> c) {
     Objects.requireNonNull(c);
     return batchRemove(c, false);
 }

 public boolean retainAll(Collection<?> c) {
     Objects.requireNonNull(c);
     return batchRemove(c, false);
 }
````

`batchRemove` 使用的是 双指针 方法进行遍历并替换

````java
private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            if (r != size) {  // 进入这部分表示有报错未全部遍历完
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {  //  GC 回收
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
}
````



图解双指针：

![image-20201218152606833](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201218152610257-1986960750.png)

###### 迭代器：

运用for-each 遍历 list：

````java
 public static void main(String[] args) {
        List<Integer> list = new ArrayList<Integer>(){{
            add(1);add(2);add(3);
            add(4);add(5);add(6);
        }};
        for (Integer integer : list) 
            System.out.print(integer);
    }
````

反编译结果：

````java
 public static void main(String[] args) {
        List<Integer> list = new ArrayList<Integer>() {
            {
                this.add(1);
                this.add(2);
                this.add(3);
                this.add(4);
                this.add(5);
                this.add(6);
            }
        };
        Iterator var2 = list.iterator();
        while(var2.hasNext()) {
            Integer integer = (Integer)var2.next();
            System.out.print(integer);
        }
    }
````

**迭代器：** 使用for-each 本质上就是使用迭代器方式进行遍历。迭代器是一种*设计模式*，它是一个对象，它可以遍历并选择序列中的对象，而我们不需要了解该序列的底层结构。迭代器通常被称为“轻量级”对象，因为创建它的代价小。

Iter内的方法：

+ hasNext():检查序列中是否还有元素
+ next()获取序列中的下一个元素
+ remove() 将迭代器中返回的元素进行删z除
+ forEachRemaining(Consumer<?> consumer) 遍历剩余的元素

Itr实现图解：

![image-20201218154945527](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201218154948975-1167863866.png)

实现代码(只保留核心代码)：

````java
 private class Itr implements Iterator<E> {
    int cursor;       
    int lastRet = -1; 
    int expectedModCount = modCount;
    Itr() {}
     
	public boolean hasNext() {
            return cursor != size;
    }
     
    public E next() {
          int i = cursor;
          cursor = i + 1;
          return (E) elementData[lastRet = i];
     }
     
     public void remove() {
    	ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
     }
     
     public void forEachRemaining(Consumer<? super E> consumer) {
        int i = cursor;
        if (i >= size) {
           return;
        }
        while (i != size && modCount == expectedModCount) {
           consumer.accept((E) elementData[i++]);
        }
        cursor = i;
        lastRet = i - 1;
     }
}     
````



`forEachRemaining`(consumer)的使用：

````java
public static void main(String[] args) {
        List<Integer> list = new ArrayList<Integer>(){{
            add(1);add(2);add(3);
            add(4);add(5);add(6);
        }};
        int i = 0;
        Iterator<Integer> iterator = list.iterator();
        while (iterator.hasNext()) {
            if(i == 5)
                break;
            else System.out.printf("%d \t",iterator.next());
            i++;
        }
        System.out.println("\n forEachRemaining遍历 剩余的结果值");
        iterator.forEachRemaining(System.out::print);
        /*
        * 输出：
        *    1  2 	3 	4 	5
        *    forEachRemaining遍历 剩余的结果值
        *    6
        * */
    }
````

`ArrayLislt` 还有另一中迭代器 `ListItr` ，它继承了`Itr`，并可以向前遍历，还有`add`和`set`方法对数据进行操作

在1.8中，`ArrayList`  内部实现了一个 `Spliterator`，专门为了多线程并行遍历元素，这个迭代器的主要功能就是把集合分成了好几段，每个线程执行自己那段，所以是线程安全的。

测试代码：

````java

class MySpliterator<T> extends Thread {
    private Spliterator<T> list;

    MySpliterator(Spliterator<T> list) {
        this.list = list;
    }

    @Override
    public void run() {
        Spliterator<T> list2 = list;
        list2.forEachRemaining(t -> System.out.print(t + " "));
    }

}

public class Test {

    public static void main(String[] args) throws InterruptedException {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            list.add(i + 1);
        }

        Spliterator<Integer> s1 = list.spliterator();
        Spliterator<Integer> s2 = s1.trySplit();
        Spliterator<Integer> s3 = s1.trySplit();
        Spliterator<Integer> s4 = s2.trySplit();
        MySpliterator<Integer> t01 = new MySpliterator<>(s1);
        MySpliterator<Integer> t02 = new MySpliterator<>(s2);
        MySpliterator<Integer> t03 = new MySpliterator<>(s3);
        MySpliterator<Integer> t04 = new MySpliterator<>(s4);

        /*
        * 打印代码省略
        * */
        
/*
        输出：
                16 17 18 19 20
                -----------------------
                6 7 8 9 10
                -----------------------
                11 12 13 14 15
                -----------------------
                1 2 3 4 5
*/
    }
}
````



<img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201218182459608-2126191299.png" alt="image-20201218182455913" style="zoom:80%;" />

### Vector：

#### 概要介绍：

​	`Vector` 与 `ArrayList` 类似，内部同样维护了一个数组，但`Vector`里面的方法大都用`synchronized` 修饰，所以是线程安全的，且它的扩容因子可以自定义，若无设置扩容因子，默认为2倍增长。

​	`Vector` 很多方法都是直接使用`AbstractCollection` 和 `AbstractList`的默认实现，所以性能不是特别好

​	`Vector`与`Collections.synchronizedList`生成的线程安全`List`相比：

+ `SynchronizedList`有很好的扩展和兼容功能, 可以将所有的`List`子类转成线程安全的类
+ `SynchronizedList`可以指定锁对象
+ `Vector`迭代器加锁机制，不需要手动同步处理(`SynchronizedList`在遍历的时候要手动进行同步处理)

![image-20201219103138261](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219103142581-1475709977.png)

#### 方法分析：

````java
private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
    	// 扩容。新的容量=当前容量+当前容量/2.即将当前容量增加一倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
````

#### Stack：

 `stack` 类是一个遗留类，继承了 `Vector`，用数组实现了栈，本身实现方式存在缺陷。

`push(E item):`

     ````java
public E push(E item) {
        addElement(item);
        return item;
}
     ````

`pop()：`

````java
public synchronized E pop() {
        E       obj;
        int     len = size();
        obj = peek();
        removeElementAt(len - 1);
        return obj;
}
````

`peek()：`

````java

public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
}
````

#### 图解`push()`和 `pop()`

<img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219105209040-1766657873.png" alt="image-20201219105205534" style="zoom:80%;" />



### CopyOnWriteArrayList：

#### 概要介绍：

##### 实现思想：

​	**写入时复制思想（CopyOnWrite）**：指：如果有多个调用者同时要求相同的资源，他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源内容时，系统才会真正复制一份专用副本给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的。此做法主要的优点是如果调用者没有修改资源，就不会有副本被创建，所以多个调用者只是读取操作时可以共享同一份资源。

  <img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219122628957-849674129.png" alt="image-20201219122625439" />

`CopyOnWriteArrayList`是`ArrayList`的线程安全版本，内部是通过数组实现，实现线程安全的原理称为**读写分离**：每次对数组的修改都完全拷贝一份新的数组来修改，修改完了再替换掉老数组，这样保证了只阻塞写操作，不阻塞读操作。

![image-20201219110450757](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219110454291-1724943025.png)



### 源码分析：

##### 属性值：

````java
/* 可重入锁，当修改list集合时加锁 */
final transient ReentrantLock lock = new ReentrantLock();

/* 存储元素的地方 */
private transient volatile Object[] array;
````

`CopyOnWriteArrayList`没有`size`属性，是因为每次修改都是一份正好存储目标个数元素的数组，数组的长度就是集合的大小，所以就不需要`size`属性指定集合的大小了

`CopyOnWriteArrayList` 的`add`和`remove` 方法和`ArrayList` 类似，不过多了一步加锁的操作，这里不再赘述，

里面的`set`方法比较有意思

````java
public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
    	 //修改时，先进行加锁
        lock.lock();
        try {
            //复制一份list,并得到对应下标的值
            Object[] elements = getArray();
            E oldValue = get(elements, index);
			      //如果需要重新写入
            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                //就算不写入，这里也调用一次setArray,使 volatile 写语义生效
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
````



###### 解释：

​	volatile写的内存语义：

+ happen-before 规则： volatile变量规则：对一个volatile域的**写**，happens- before 于任意后续对这个volatile域的**读**

+ 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。实现方式是在写操作的前加上 StoreStore 屏障， 在写操作的后面加上StoreLoad屏障
+ 当有线程进行读该volatile变量时，JMM会使本地内存无效，强制从主内存中读取该变量的值



<img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219151544519-200093659.png" alt="image-20201219151540881" style="zoom:110%;" />





**setArray(elements) 不是为了保证CopyOnWriteArrayList 本身的可见性**

**而是为了保证外部的非volatile属性遵循happen-before原则**

代码说明：

````java
class Test {
    int a = 100;
   /* volatile */ boolean flag;
 
    public void writer() {
        a = 200;                   //1
        flag = true;               //2
    }
 
    public void reader() {
        if (flag) {               //3
           int i =  a;            //4
        }
    }
}
/*
假如有两个线程持有同一份 Test 对象，其中线程1执行 writer() 方法，线程2执行reader()方法，如果 flag 没有 volatile 修饰，那么图中语句 1 和 2 可能会发生指令重排， flag = true 先设置了，然后跳到线程二执行reader()方法，线程2的if 条件成立，i 读到的是还没有设置的值100 ，而不是 200，因为指令重排线程1还没有设置 a = 200 。这就造成了不一致问题，那如果用 volatile 修饰 flag ，那么就不会发生 语句 1 和 2 的指令重排，解决了这个问题
*/
````



## Queue(Deque):

主要介绍以下四种集合类

+ **`LinkedList` ★**
+ **`ArrayDeque`**  
+ **`PriorityQueue`**
+ **`LinkedBlockingQueue`**



![20190517160435818](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219154643850-1760412467.png)



### 数据结构介绍：

#### Queue:

​	队列(queue)是一种只允许在一端进行插入，在另一端进行删除的线性表结构。允许插入的一端成为队尾(rear),允许删除的一端称为队头(front).

#### Deque：

​	双端队列则是两端都可以进行插入和删除。**只在一端**进行插入和删除操作，就可以作为栈来使用



<img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219160254021-1413767255.png" alt="image-20201219160250508" style="zoom:100%;" />

### LinkedList：

##### 概要介绍：

LinkedList是一个以双向链表实现的List，它除了作为List使用，还可以作为队列或者栈来使用

![image-20201219162231906](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219162235519-72512651.png)



##### 属性值：

````java
// 元素个数
transient int size = 0;
// 链表首节点
transient Node<E> first;
// 链表尾节点
transient Node<E> last;
````

##### 核心方法：

+ `add:`

  - `linkFirst(E e)`

  + `linkLast(E e)`
  + `linkBefore(E e,Node<E> succ)`

+ `remove:`
  - `unlinkFirst(Node<E> f)`
  - `unlinkLast(Node<E> f)`
  - `unlink(Node<E> x)`



###### `linkFirst(E e)`：在头节点上插入元素(`linkLast(E e)`同理)

````java
 private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
}

void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
}
````

###### 图解linkFirst(E e):

**![image-20201219163617527](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219163620980-1626267481.png)**



 `linkBefore(E e,Node<E> succ)`:在指定位置上插入元素

````java
void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
}
````

###### 图解`linkBefore`(E e,Node<E> succ):

![image-20201219165808294](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201219165811750-467537024.png)



`remove` 方法和`add`方法思想是一样的，这里不再赘述。



LinkedList 作为栈来使用：

````java
public void push(E e) {
    addFirst(e);
}

public E pop() {
    return removeFirst();
}
````



### ArrayDeque：

#### 概要介绍：

​	`	ArrayDeque`是基于数组实现到的一个无界双端队列，容量可扩展，不允许null元素，因为移除位的元素将使用null填充。队列的容量是数组的长度，并且数组长度始终是2的次幂。使用`ArrayDeque`实现栈或者队列比使用`Stack`、`LinkedList`效率高。

​	为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即循环数组，也就是说数组的任何一点都可能被看作起点或者终点。`ArrayDeque`是非线程安全的不允许放入`null`元素,是因为要区分`tail`和 `head`的指针，每次都将`tail`指向null,`tail`指向null起到哨兵的作用，那么就不允许队列中再有null值。

   **`head`指向首端第一个有效元素，`tail`指向尾端第一个可以插入元素的空位**。因为是循环数组，所以`head`不一定总等于0，`tail`不一定总是比`head`大，所以再查找位置的时候，需要取模运算。

<img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221094725999-926019059.png" alt="image-20201219222754887" style="zoom: 40%;" />

#### 源码分析：

##### 属性值：

````java
// 底层元素存储， 默认长度为8
transient Object[] elements;
// 队列头部元素
transient int head;
// 队列尾部元素
transient int tail;
// 默认最小容量，如果传递的容量小于8，则使用8作为传递容量，最后实际容量被修改为16
private static final int MIN_INITIAL_CAPACITY = 8;
````

##### 核心方法：

+ ###### addFirst():(只介绍一个方法，因为其余方法同理)

  ````java
  public void addFirst(E e) {
          if (e == null)
              throw new NullPointerException();
    			//需要判断下标是否越界，所以需要与长度进行模运算
          elements[head = (head - 1) & (elements.length - 1)] = e;
          if (head == tail)
              doubleCapacity();
  }
  ````

  ###### 说明：
  
  `	addFirst(E e)`的作用是在队列的首端插入元素，也就是在`head`的前面插入元素，在空间足够且下标没有越界的情况下，只需要将`elements[--head] = e`即可。
  
  但还需要考虑两种情况：1.**是否需要扩容**  2.**是否越界**。如果`head`为`0`之后接着调用`addFirst()`，虽然空余空间还够用，但`head`为`-1`，下标越界了。所以需要采用 `head = (head - 1) & (elements.length - 1)`来获取越界后的值。
  
  其中 doubleCapacity()方法表示扩容，每次扩容都是翻倍，扩容后里面的参数如下，先复制head右边的元素，再复制左边的元素
  
  ![doubleCapacity](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221094732515-1185028829.png)

+ ###### calculatesize(int numElements): 找到大于给定numElements 的最小2的幂倍数

````java
private static int calculateSize(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++;
            if (initialCapacity < 0)   
                initialCapacity >>>= 1;
        }
        return initialCapacity;
    }
````



### PriorityQueue:

#### 概要介绍：

​	优先队列不是按照普通对象先进先出原FIFO则进行数据操作，其中的元素有优先级属性，优先级高的元素先出队。PriorityQueue实现的队列，是基于最小堆原理实现。

​	最小堆是一个完全二叉树，有一个特性是根节点必定是最小节点，子女节点一定大于其父节点。还有一个特性是叶子节点数量=全部非叶子节点数量+1

​    在 PriorityQueue队列中，基于数组保存了完全二叉树。所以在已知任意一个节点在数组中的位置，就可以通过一个公式推算出其左子树和右子树的下标。已知节点下标是i，那么他的左子树是2*i+1，右子树是2*i+2。

#### 核心方法：

+ ###### add(E e)(只介绍一个方法，因为其余方法同理)

  ````java
   public boolean add(E e) {
          return offer(e);
  }
  
  /*
  新加入的元素x可能会破坏小顶堆的性质，因此需要进行调整。调整的过程为：从k指定的位置开始，将x逐层与当前点的parent进行比较并交换，直到满足x >= queue[parent]为止。这里的比较可以是元素的自然顺序，也可以是依靠指定比较器的顺序。
  */
  public boolean offer(E e) {
          int i = size;
          if (i >= queue.length)
            	//  oldCapacity + ((oldCapacity < 64) ?
             //                        (oldCapacity + 2) :
            //                         (oldCapacity >> 1));
              grow(i + 1);
          size = i + 1;
          if (i == 0)
              queue[0] = e;
          else
              siftUp(i, e);
          return true;
  }
  
  /*
  入队是元素首先进入队列尾，然后和自己的父节点比较，像冒泡一样将该节点冒到合适的位置，即比自己的父亲节点大，比自己的儿子节点小。
  */
  private void siftUpComparable(int k, E x) {
          Comparable<? super E> key = (Comparable<? super E>) x;
          while (k > 0) {
              int parent = (k - 1) >>> 1;
              Object e = queue[parent];
              if (key.compareTo((E) e) >= 0)
                  break;
              queue[k] = e;
              k = parent;
          }
          queue[k] = key;
      }
  
  ````

### LinkedBlockingQueue：

#### 概要介绍：

​	`LinkedBlockingQueue`是一个单向链表实现的阻塞队列，先进先出的顺序。支持多线程并发操作。内部由**单链表**实现，只能从head取元素，从tail添加元素。添加元素和获取元素都有独立的锁，也就是说`LinkedBlockingQueue`是**读写锁分离**，读写操作可以并行执行。LinkedBlockingQueue采用可重入锁ReentrantLock来保证在并发情况下的线程安全

#### 源码分析：

##### 属性值：

````java
/ 容量
private final int capacity;

// 元素数量
private final AtomicInteger count = new AtomicInteger();

// 链表头
transient Node<E> head;

// 链表尾
private transient Node<E> last;

// take锁
private final ReentrantLock takeLock = new ReentrantLock();

// notEmpty条件
// 当队列空的时候，take锁会阻塞在notEmpty条件上，等待其它线程唤醒
private final Condition notEmpty = takeLock.newCondition();

// 锁
private final ReentrantLock putLock = new ReentrantLock();

// notFull条件
// 当队列满的时候，put锁会会阻塞在notFull上，等待其它线程唤醒
private final Condition notFull = putLock.newCondition();
````



#### 核心方法：

+ ###### put(E e): (其他方法同理)

+ ###### take(E e):  (其他方法同理)

##### put(E e):

````java
public void put(E e) throws InterruptedException {
    // 不允许出现null元素
    if (e == null) throw new NullPointerException();
    int c = -1;
    // 新建一个节点
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    // 使用put锁
    putLock.lockInterruptibly();
    try {
        // 如果队列满了，就阻塞在notFull条件上
        while (count.get() == capacity) {
            notFull.await();
        }
        // 否则才入对
        enqueue(node);
        // 队列长度加1
        c = count.getAndIncrement();
        //如果还有剩余空间可以存放元素，就唤醒其他需要入队列的线程 
       //c + 1 表示的是入队列之后队列元素的总和
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    // 如果原队列长度为0，现在加了一个元素后立即唤醒阻塞在notEmpty条件
    if (c == 0)
        signalNotEmpty();
}
````

##### take():

````java
 public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
     	//使用take锁
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lockInterruptibly();
        try {
          	//如果队列为空，则阻塞在notEmpty条件上
            while (count.get() == 0) {
                notEmpty.await();
            }
            // 否则取队列一个元素
            x = dequeue();
            c = count.getAndDecrement();
          //如果取完以后，还有元素在队列中，则再唤醒另一个线程
            if (c > 1)
                notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
   
        //如果原队列已经满了，现在取了一个元素后需要唤醒阻塞在notFull条件的线程
        if (c == capacity)
            signalNotFull();
        return x;
}

````



## Map:

主要介绍以下四种集合类

+ **`HashMap`** ★ (红黑树分析放在TreeMap)
+ **`LinkedHashMap` **
+ **`TreeMap`**  

### HashMap：

#### 概要介绍：

+ HashMap的实现采用了（数组 + 链表 + 红黑树）的复杂结构，数组的一个元素又称作桶。

+ 在添加元素时，会根据hash值算出元素在数组中的位置，如果该位置没有元素，则直接把元素放置在此处，如果该位置有元素了，则把元素以链表的形式放置在链表的尾部。

+ 当一个链表的元素个数达到一定的数量(**8)**且数组的长度达到一定的长度(**64**)后，则把链表转化为红黑树，从而提高效率。
+ 当一个红黑树的元素小于一定数量(**6**)时,红黑树将会退化成链表

+ 数组的查询效率为O(1)，链表的查询效率是O(k)，红黑树的查询效率是O(log k)，k为桶中的元素个数，所以当元素数量非常多的时候，转化为红黑树能极大地提高效率
+ hashmap底层结构：

![image-20201221130834929](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221130838662-1932498381.png)

#### 源码分析：

##### 查找：

说明：HashMap的查找获取key的hash值，然后定位键值对所在的桶的位置，然后按照节点的类型，对链表或红黑树进行查找

````java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            //用位运算代替取余运算
            (first = tab[(n - 1) & hash]) != null) {
            //每次都先判断桶中第一个元素是否满足条件
            if (first.hash == hash && 
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                //红黑树查找
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //链表查找
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
    	//查找失败，返回 null
        return null;
}

````

流程图：



![image-20201221141220235](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221141223858-1485067996.png)

##### 插入：

说明：先定位要插入的键值的桶位置，桶为空直接插入，不为空则判断桶节点类型，按照红黑树或链表方式进行插入，若是链表插入，则要根据链表数量判断是否需要扩容

````java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // map为空时初始化桶
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 对应的桶下标为空时
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 查找桶的第一个链表是否符合条件
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
            
        // 调用红黑树的插入方法
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 对链表进行遍历，并统计链表长度
            for (int binCount = 0; ; ++binCount) {
                // 链表中不包含要插入的键值对节点时，则将该节点接在链表的最后
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 链表太长，需要进行树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        treeifyBin(tab, hash);
                    break;
                }
                
                // 链表方式查找到了对应的键值对
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // 如果不为空，表示map中已经有对应的键值对
        if (e != null) { 
            V oldValue = e.value;
            //onlyIfAbsent 是否要进行覆盖
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
````

总结：

+ 当桶数组 table 为空时，通过扩容的方式初始化 table
+ 查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
+ 如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
+ 判断键值对数量是否大于阈值，大于的话则进行扩容操作

##### 扩容方法：

说明： 

+ 在 HashMap 中，桶数组的长度均是2的幂，阈值大小为桶数组长度与负载因子的乘积。当 HashMap 中的键值对数量超过阈值时，进行扩容。 按当前桶数组长度的2倍进行扩容，阈值也变为原来的2倍（如果计算过程中，阈值溢出归零，则按阈值公式重新计算）。扩容之后，要重新计算键值对的位置，并把它们移动到合适的位置上去。

````java
final Node<K,V>[] resize() {
    /*
    	省略计算扩容阈值和门槛的代码
    */
    if (oldTab != null) {
        // 如果旧的桶数组不为空，则遍历桶数组，并将键值对映射到新的桶数组中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 拆分成两颗红黑树
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //下面代码表示拆分成两个链表
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    //高低位链表
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
````

##### 删除：

HashMap 的删除操作三个步骤完成操作。第一步是定位桶位置，第二步遍历链表并找到键值相等的节点，第三步删除节点

````java
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
    	// 定位桶位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            //判断第一个桶节点是否满足条件
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    //红黑树获取
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    //链表获取
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    //红黑树删除
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                //链表删除
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
````



### LinkedHashMap：

`LinkedHashMap` 继承了 HashMap，它保留插入的顺序，如果需要输出的顺序和输入时的相同，那么就选用 `LinkedHashMap`。原因是增加了**双向链表**结构用于存储所有元素的顺序，添加删除元素的时候同时需要维护HashMap的存储以及双向链表的存储，所以性能不如HashMap

![image-20201221151302137](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221151305809-2089926443.png)

`LinkedHashMap`定义了三个钩子：

+ `afterNodeInsertion`：用来回调移除最早放入Map的对象
+ `afterNodeAccess：put()已经存在的元素或get()时被调用，如果accessOrder为true，调用这个方法把访问到的节点移动到双向链表的末尾
  - accessOrder为false，则可以按**插入元素**的顺序 遍历元素；为true，表示按照**访问顺序**遍历元素
+ `afterNodeRemoval：`在节点被删除后调用该方法维护双向链表

````java
void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }


    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }


    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
````



#### TreeMap：

##### 概要介绍：

​	`TreeMap`是通过红黑树来实现，所以要介绍`TreeMap`本质就是介绍红黑树

##### 红黑树的性质：

普通的二叉查找树在极端情况下可退化成链表，此时的增删查效率都会比较低下。为了避免这种情况，就出现了一些自平衡的查找树，比如红黑树。红黑树通过定义一些性质，将任意节点的左右子树高度差控制在规定范围内，以达到平衡状态。红黑树有以下5种性质：

+ **节点是红色或黑色**
+ **根是黑色**
+ **所有叶子都是黑色（NIL节点）**
+ **从每个叶子到根的所有路径上不能有两个连续的红色节点**
+ **从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点（简称黑高）**

前三种性质，可以避免**二叉查找树退化成单链表**的情况。后面两种性质，限制节点到其每个叶子节点路径长度的过长的问题 ，保证任意节点到其每个叶子节点路径最长不会超过最短路径的2倍。

##### 红黑树示例：

![image-20201221160049877](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221160053701-1048101438.png)

##### 红黑树操作：

###### 左旋与右旋：

+ **左旋：** 将某个节点**A**旋转为其自身右孩子的左孩子，并将其右孩子的左孩子旋转为A的右孩子

图解：

<img src="https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221164709240-646866611.png" alt="image-20201221164705647" style="zoom:80%;" />

````java
 private void rotateLeft(Entry<K,V> p) {
        if (p != null) {
            Entry<K,V> r = p.right;
            p.right = r.left;
            if (r.left != null)
                r.left.parent = p;
            r.parent = p.parent;
            if (p.parent == null)
                root = r;
            else if (p.parent.left == p)
                p.parent.left = r;
            else
                p.parent.right = r;
            r.left = p;
            p.parent = r;
        }
    }
````



+ **右旋：** 将某个节点**A**旋转为其自身左孩子的右孩子，并将其左孩子的右孩子旋转为A的左孩子

图解：

![image-20201221165745815](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221165749449-810778683.png)

````java
private void rotateRight(Entry<K,V> p) {
        if (p != null) {
            Entry<K,V> l = p.left;
            p.left = l.right;
            if (l.right != null) l.right.parent = p;
            l.parent = p.parent;
            if (p.parent == null)
                root = l;
            else if (p.parent.right == p)
                p.parent.right = l;
            else p.parent.left = l;
            l.right = p;
            p.parent = l;
        }
    }
````

###### 插入：

说明：

红黑树的插入过程和二叉搜索树插入过程基本相同，不同点在于，红黑树插入新节点后，还需要进行调整，以便满足性质**4**和**5**。每次新插入的节点颜色默认为**red**，这样只需要考虑出现两个连续的红色节点的情况，通过变色和旋转即可完成调整

插入的情况共有五种：

+ ***情况一***：红黑树为空，插入节点即为根节点：

![image-20201221171038184](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221171041804-623178358.png)

+ ***情况二***：父亲节点为黑色，直接插入即可：

  ![image-20201221171324711](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221171328320-329094452.png)

+ ***情况三***：父亲节点为红色，且叔叔节点也是红色，此时不满足**性质4:从每个叶子到根的所有路径上不能有两个连续的红色节点)**,需要重新调整，先将父亲节点和叔叔节点染成黑色，再将父亲节点的父结点染成红色。若向上递归时发现它与它的父亲节点形成连续的红色节点，还需向上递归调整。若根节点为红色，直接染成黑色就行

  ![image-20201221181710608](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221181714661-1246262033.png)

+ ***情况四***：若插入节点的父结点是红色，叔叔节点是黑色，且插入节点是父结点的左孩子，父结点是其自身父结点的左孩子，那么需要对其进行右旋，调整位置，并互换父结点和自身父结点的颜色

![image-20201221184517172](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221184520857-2035775942.png)

+ ***情况五***：若插入节点的父结点为红色，叔叔节点为黑色，且插入节点是父结点的右孩子，父结点是其自身父结点的左孩子，那么需要对其进行左旋，然后按照情况四来处理

  ![image-20201221191712262](https://img2020.cnblogs.com/blog/1625166/202012/1625166-20201221191715880-672531051.png)

如果父结点是祖父节点的右孩子，那么***情况 4***和***情况 5***旋转刚好好相反，这里不再累述：

总结：

根据不同的情况有以下几种处理方式：

1. 插入的元素如果是根节点，则直接涂成黑色即可，不用平衡；
2. 插入的元素的父节点如果为黑色，不需要平衡；
3. 插入的元素的父节点如果为红色，则违背了特性4，需要平衡，平衡时又分成下面三种情况：

**（如果父节点是祖父节点的左节点）**

| 情况                                                         | 处理                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1）父节点为红色，叔叔节点也为红色                            | （1）将父节点设为黑色<br> （2）将叔叔节点设为黑色 <br>（3）将祖父节点设为红色 <br>（4）将祖父节点设为新的当前节点，进入下一次循环判断 |
| 2）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的右节点 | （1）将父节点作为新的当前节点；<br> （2）以新当节点为支点进行左旋，进入情况4）； |
| 3）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的左节点 | （1）将父节点设为黑色<br> （2）将祖父节点设为红色；<br>（3）以祖父节点为支点进行右旋，进入下一次循环判断 |

**（如果父节点是祖父节点的右节点，则正好与上面反过来）**

| 情况                                                         | 策略                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1）父节点为红色，叔叔节点也为红色                            | （1）将父节点设为黑色 <br>（2）将叔叔节点设为黑色 <br/>（3）将祖父节点设为红色<br/> （4）将祖父节点设为新的当前节点，进入下一次循环判断 |
| 2）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的左节点 | （1）将父节点作为新的当前节点<br/> （2）以新当节点为支点进行右旋 |
| 3）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的右节点 | （1）将父节点设为黑色<br/> （2）将祖父节点设为红色<br/> （3）以祖父节点为支点进行左旋，进入下一次循环判断 |

````java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        // 如果没有根节点，直接插入到根节点
        compare(key, key); 
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    
    int cmp;
    
    Entry<K,V> parent;
    
    Comparator<? super K> cpr = comparator;
    if (cpr != null) 
        // 按照指定的比较器查找
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        // 默认的比较器查找
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
               t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 如果为空，新建一个节点，插入到树中
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
         parent.left = e;
    else
       parent.right = e;

    // 插入之后再平衡
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
````

再平衡方法：

````java
/**
 * 插入再平衡
 *（1）每个节点或者是黑色，或者是红色。
 *（2）根节点是黑色。
 *（3）每个叶子节点（NIL）是黑色。（叶子节点，是指为空(NIL或NULL)的节点）
 *（4）如果一个节点是红色的，则它的子节点必须是黑色的。
 *（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。
 */
private void fixAfterInsertion(Entry<K,V> x) {
    // 插入的节点为红节点，x为当前节点
    x.color = RED;

    // 只有当插入节点不是根节点且其父节点为红色时才需要平衡（违背了特性4）
    while (x != null && x != root && x.parent.color == RED) {
        // 如果父节点是祖父节点的左节点
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            // y为叔叔节点
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                // 情况3）如果叔叔节点为红色
                // （1）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （2）将叔叔节点设为黑色
                setColor(y, BLACK);
                // （3）将祖父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // （4）将祖父节点设为新的当前节点
                x = parentOf(parentOf(x));
            } else {
                // 如果叔叔节点为黑色
                // 情况5）如果当前节点为其父节点的右节点
                if (x == rightOf(parentOf(x))) {
                    // （1）将父节点设为当前节点
                    x = parentOf(x);
                    // （2）以新当前节点左旋
                    rotateLeft(x);
                }
                // 如果当前节点为其父节点的右节点（则左旋之后新当前节点正好为其父节点的左节点了）
                //那么就要跳到情况4来处理了
                // （1）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （2）将祖父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // （3）以祖父节点为支点进行右旋
                rotateRight(parentOf(parentOf(x)));
            }
        } else {  //对称
           Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
               setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 根节点为black
    root.color = BLACK;
}
````



###### 删除：

红黑树删除特别复杂，操作首先是采用二叉树的删除规则，然后根据被删除节点的颜色判断是否需要做再平衡操作。

**二叉树的删除规则**：

+ 如果删除的位置有两个叶子节点，则从其右子树中取最小的元素放到删除的位置，然后把删除位置移到替代元素的位置，进入下一步
+ 如果删除的位置只有一个叶子节点（或经过第一步转换后的删除位置），则把那个叶子节点作为替代元素，放到删除的位置，然后把这个叶子节点删除
+ 如果删除的位置没有叶子节点，则直接把这个删除位置的元素删除即可

红黑树删除后再调整, 我自己写可能讲不好，所以给出以下两个链接作为参考：

> [图解红黑树](https://juejin.cn/post/6844903511893737486)
>
> [红黑树详细分析](http://www.tianxiaobo.com/2018/01/11/红黑树详细分析/)

假设，顶替删除节点的那个后来节点，继承了被删除的黑色节点的那层黑色，也就是说顶替的节点具有双重颜色，如果原来是黑色，那么现在就是黑+黑，如果原来是红色，现在就是红+黑。因为有了这层额外的黑色，所以性质5还是能保持的，现在只需要恢复它的性质即可

+ *x节点时黑+黑，且x的兄弟节点是红色（x的父节点和兄弟节点的子节点都是黑色）*
+ *x节点时黑+黑，x的兄弟节点时黑色（x的兄弟节点的两个孩子都是黑色）*
+ *x节点是黑+黑节点，x的兄弟节点时黑色（x的兄弟节点的左儿子是红，右儿子是黑）*
+ *x节点是黑+黑节点，x的兄弟节点时黑色（x的兄弟节点的右儿子是红，左儿子随意）*


````java
private void fixAfterDeletion(Entry<K,V> x) {
    // 只有当前节点不是根节点且当前节点是黑色时才进入循环
    while (x != root && colorOf(x) == BLACK) {
        if (x == leftOf(parentOf(x))) {
            // 如果当前节点是其父节点的左子节点
            // sib是当前节点的兄弟节点
            Entry<K,V> sib = rightOf(parentOf(x));

            // 情况1）如果兄弟节点是红色
            if (colorOf(sib) == RED) {
                // （1）将兄弟节点设为黑色
                setColor(sib, BLACK);
                // （2）将父节点设为红色
                setColor(parentOf(x), RED);
                // （3）以父节点为支点进行左旋
                rotateLeft(parentOf(x));
                // （4）重新设置x的兄弟节点，进入下一步
                sib = rightOf(parentOf(x));
            }

            if (colorOf(leftOf(sib))  == BLACK &&
                    colorOf(rightOf(sib)) == BLACK) {
                // 情况2）如果兄弟节点的两个子节点都是黑色
                // （1）将兄弟节点设置为红色
                setColor(sib, RED);
                // （2）将x的父节点作为新的当前节点，进入下一次循环
                x = parentOf(x);
            } else {
                if (colorOf(rightOf(sib)) == BLACK) {
                    // 情况3）如果兄弟节点的右子节点为黑色
                    // （1）将兄弟节点的左子节点设为黑色
                    setColor(leftOf(sib), BLACK);
                    // （2）将兄弟节点设为红色
                    setColor(sib, RED);
                    // （3）以兄弟节点为支点进行右旋
                    rotateRight(sib);
                    // （4）重新设置x的兄弟节点
                    sib = rightOf(parentOf(x));
                }
                // 情况4）
                // （1）将兄弟节点的颜色设为父节点的颜色
                setColor(sib, colorOf(parentOf(x)));
                // （2）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （3）将兄弟节点的右子节点设为黑色
                setColor(rightOf(sib), BLACK);
                // （4）以父节点为支点进行左旋
                rotateLeft(parentOf(x));
                // （5）将root作为新的当前节点（退出循环）
                x = root;
            }
        } else { // symmetric
            // 如果当前节点是其父节点的右子节点
            // sib是当前节点的兄弟节点
            Entry<K,V> sib = leftOf(parentOf(x));

            // 情况1）如果兄弟节点是红色
            if (colorOf(sib) == RED) {
                // （1）将兄弟节点设为黑色
                setColor(sib, BLACK);
                // （2）将父节点设为红色
                setColor(parentOf(x), RED);
                // （3）以父节点为支点进行右旋
                rotateRight(parentOf(x));
                // （4）重新设置x的兄弟节点
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                    colorOf(leftOf(sib)) == BLACK) {
                // 情况2）如果兄弟节点的两个子节点都是黑色
                // （1）将兄弟节点设置为红色
                setColor(sib, RED);
                // （2）将x的父节点作为新的当前节点，进入下一次循环
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    // 情况3）如果兄弟节点的左子节点为黑色
                    // （1）将兄弟节点的右子节点设为黑色
                    setColor(rightOf(sib), BLACK);
                    // （2）将兄弟节点设为红色
                    setColor(sib, RED);
                    // （3）以兄弟节点为支点进行左旋
                    rotateLeft(sib);
                    // （4）重新设置x的兄弟节点
                    sib = leftOf(parentOf(x));
                }
                // 情况4）
                // （1）将兄弟节点的颜色设为父节点的颜色
                setColor(sib, colorOf(parentOf(x)));
                // （2）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （3）将兄弟节点的左子节点设为黑色
                setColor(leftOf(sib), BLACK);
                // （4）以父节点为支点进行右旋
                rotateRight(parentOf(x));
                // （5）将root作为新的当前节点（退出循环）
                x = root;
            }
        }
    }
    // 退出条件为多出来的黑色向上传递到了根节点或者红节点
    // 则将x设为黑色即可满足红黑树规则
    setColor(x, BLACK);
}
````



## Set:

#### 概要介绍：

`HashSet`，`TreeSet`和`LinkedHashSet` 都是运用对应的`Map`来确保元素不重复，以`HashMap<k,v>`为例，相当于只用了里面的key,而value都是内部调用一个Object

HashMap 为例

````java
// PRESENT 就是所有键值对的值
private static final Object PRESENT = new Object();

//添加
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
//查询
public boolean contains(Object o) {
    return map.containsKey(o);
}
//若为null,则说明没有元素，不是null那么value 肯定等于 PRESNET
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
````

#### BitSet:



