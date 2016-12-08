#1 集合体系
- 单列集合Collection
	- **List** 集合有序、可重复
		- **ArrayList** 最常用底层是数组，查询快，增删慢
		- **LinkedList** 底层是链表，查询慢、增删快
	- **Set** 集合无序、不可重复
		- **HashSet** 底层的实现实际上基于HashMap，集合相当于HashMap的key的集合，哈希表保证唯一
			- LinkedHashSet 底层的实现基于LinkedHashMap，集合了哈希表和链表结构的优点，哈希表保证唯一，链表保证有序
		- **TreeSet** 一组有次序的集合，如果没有指定排序规则Comparator，则会按照自然排序。（自然排序即e1.compareTo(e2) == 0作为比较），底层基于TreeMap
- 键值对集合Map
	- **HashMap** 用的比较多，基于哈希表的Map接口实现
		- **LinkedHashMap** 底层是链表和哈希表
	- **TreeMap** 底层是二叉树
	- **HashTable** 线程安全

**小结**

- 哈希表是元素为链表的数组
- LindkXX 开头的底层数据机构是链表，查询慢、增删快
- ArrayXX 开头的底层数据结构是数组，查询快，增删慢
- HashXX 开头的底层数据结构是哈希表，哈希表用来保证唯一性
- TreeXX 开头的底层数据结构是二叉树（红黑树），可以用来排序，自然排序和比较器排序，比较器排序用的比较多。





## 面试题
### 1.ArrayList底层是如何实现的？

ArrayList介绍：

- ArrayList是List集合的实现类
- 可以添加任何元素，包括null
- 它的size, isEmpty, get, set, iterator,add这些方法的时间复杂度是O(1),如果add n个数据则时间复杂度是O(n)
- ArrayList不是synchronized的

ArrayList底层用到了对象数组Object[]，当新建一个ArrayList集合的时候，EmptyArray.OBJECT是一个静态的长度为0的Object[]数组。

有3个成员变量

	// 每次增加的最小长度
    private static final int MIN_CAPACITY_INCREMENT = 12;
  	 //集合的长度
    int size;
   	//集合实际上有对象数组来表示
    transient Object[] array;

    public ArrayList() {
        array = EmptyArray.OBJECT;
    }

    public ArrayList(int capacity) {
        if (capacity < 0) {
            throw new IllegalArgumentException("capacity < 0: " + capacity);
        }
        array = (capacity == 0 ? EmptyArray.OBJECT : new Object[capacity]);
    }

	 public ArrayList(Collection<? extends E> collection) {
        if (collection == null) {
            throw new NullPointerException("collection == null");
        }

        Object[] a = collection.toArray();
        if (a.getClass() != Object[].class) {
            Object[] newArray = new Object[a.length];
			//复制集合
            System.arraycopy(a, 0, newArray, 0, a.length);
            a = newArray;
        }
        array = a;
        size = a.length;
    }

底层的添加、移除方法都是用到了System.arrayCopy方法，是一个native方法，效率肯定高。

	public static native void arraycopy(Object src, int srcPos,
        Object dst, int dstPos, int length);
其中参数的意思是：
- src 需要复制数据的源数组
- srcPos 源数组中需要被复制的数据的起始索引
- dst 把数据复制到的目标数组
- dstPos 目标数组的复制数据的起始索引
- length 有多少个元素被复制


添加数据，如果当前集合已经满了，那么就增加数组的长度，这个长度根据当前数组长度是否小于6，小于的话就增加12个长度，否则就增加6个长度。

	 @Override public boolean add(E object) {
        Object[] a = array;
        int s = size;
        if (s == a.length) {
            Object[] newArray = new Object[s +
                    (s < (MIN_CAPACITY_INCREMENT / 2) ?
                     MIN_CAPACITY_INCREMENT : s >> 1)];
            System.arraycopy(a, 0, newArray, 0, s);
            array = a = newArray;
        }
        a[s] = object;
        size = s + 1;
        modCount++;
        return true;
    }


###2 HashMap底层的实现原理，键值对是如何存储，如何通过键取值？

可参考[ java面试题-HashMap原理 ](http://blog.chinaunix.net/uid-11775320-id-3143919.html)

HashMap是基于哈希表的Map接口的非同步实现，键和值都允许所有的元素，包括null。集合中的顺序和添加的顺序不一定一样，并且不保真每次运行都一样。如果需要一样的话使用LinkedHashMap。
![](http://upload-images.jianshu.io/upload_images/1594931-ca3e730b2760ed97.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**数据结构**

在java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

**HashMap的存取**
HashMap能够通过key，快速的找到value。也能够通过put(key,value)去存储键值对。key是唯一的，同样的key，后面put的value会覆盖掉前面对应的value。

![](http://blog.chinaunix.net/attachment/201203/22/11775320_1332399870gIGw.jpg)
1.put(key,value)的时候首先会计算key对应的hashCode。

2.然后会有下面的运算,获得hashCode相关的一个hash值。为什么要经过这样的运算呢？这就是HashMap的高明之处。先看个例子，一个十进制数32768(二进制1000 0000 0000 0000)，经过上述公式运算之后的结果是35080(二进制1000 1001 0000 1000)。看出来了吗？或许这样还看不出什么，再举个数字61440(二进制1111 0000 0000 0000)，运算结果是65263(二进制1111 1110 1110 1111)，现在应该很明显了，它的目的是让“1”变的均匀一点，散列的本意就是要尽量均匀分布。那这样有什么意义呢？看第3步

 	private static int secondaryHash(int h) {
        // Spread bits to regularize both segment and index locations,
        // using variant of single-word Wang/Jenkins hash.
        h += (h <<  15) ^ 0xffffcd7d;
        h ^= (h >>> 10);
        h += (h <<   3);
        h ^= (h >>>  6);
        h += (h <<   2) + (h << 14);
        return h ^ (h >>> 16);
    }

3.得到hash之后会进行一个与预算，这样的结果就是得到一个小于等于tab.length - 1的索引index，而这个index指的就是插入数组中的位置。第二步的意义就希望得到一个比较均匀的index。HashMap的键可以为null，它的值是放在数组的第一个位置。

	HashMapEntry<K, V>[] tab = table;
        int index = hash & (tab.length - 1);



4、 我们用table[index]表示已经找到的元素需要存储的位置。先判断该位置上有没有元素（这个元素是HashMap内部定义的一个类Entity， 基本结构它包含三个类，key，value和指向下一个Entity的next）,没有的话就创建一个Entity对象，在 table[index]位置上插入，这样插入结束；如果有的话，通过链表的遍历方式去逐个遍历，看看有没有已经存在的key，有的话用新的value替 换老的value；如果没有，则在table[index]插入该Entity，把原来在table[index]位置上的Entity赋值给新的 Entity的next，这样插入结束。 

5 扩展问题
要同时复写equals方法和hashCode方法。
按照散列函数的定义，如果两个对象相同，即obj1.equals(obj2)=true，则它们的hashCode必须相同，但如果两个对象不同，则它们的hashCode不一定不同。
如果两个不同对象的hashCode相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的hashCode()方法，能加快哈希表的操作。
如果相同的对象有不同的hashCode，对哈希表的操作会出现意想不到的结果（期待的get方法返回null），
再回头看看前面提到的为什么覆盖了equals方法之后一定要覆盖hashCode方法，很简单，比如，String a = new String(“abc”);String b = new String(“abc”);如果不覆盖hashCode的话，那么a和b的hashCode就会不同，把这两个类当做key存到HashMap中的话就 会出现问题，就会和key的唯一性相矛盾。

添加元素，实际上也是先看看当前要添加的这这个key的一个与hashcode相关的值以及这个值本身是否与已经存在的值相同，相同的话就不添加，返回原来的Value，否则就添加到集合中。

  	 @Override public V put(K key, V value) {
        if (key == null) {
            return putValueForNullKey(value);
        }

		//计算与hashCode相关的一个值
        int hash = Collections.secondaryHash(key);
        HashMapEntry<K, V>[] tab = table;
		//查一下这个值是否已经存在，查出对应的index。
        int index = hash & (tab.length - 1);
		
		//循环判断，如果从tab[index]中拿到的值不为空
        for (HashMapEntry<K, V> e = tab[index]; e != null; e = e.next) {
			//循环比较这个key对应的hash和已经存储的相比，如果hash相等并且键相同的话，就添加了这个元素，并且返回了原来的Value。
            if (e.hash == hash && key.equals(e.key)) {
                preModify(e);
                V oldValue = e.value;
                e.value = value;
                return oldValue;
            }
        }

        // No entry for (non-null) key is present; create one
        modCount++;
        if (size++ > threshold) {
            tab = doubleCapacity();
            index = hash & (tab.length - 1);
        }
		//添加这个元素
        addNewEntry(key, value, hash, index);
        return null;
    }


获取元素，与添加的时候类似，也是看看这个键的hash值与键值是否存在，如果存在的话就返回对应的value，否则就返回null。

 	public V get(Object key) {
        if (key == null) {
            HashMapEntry<K, V> e = entryForNullKey;
            return e == null ? null : e.value;
        }

        int hash = Collections.secondaryHash(key);
        HashMapEntry<K, V>[] tab = table;
        for (HashMapEntry<K, V> e = tab[hash & (tab.length - 1)];
                e != null; e = e.next) {
            K eKey = e.key;
            if (eKey == key || (e.hash == hash && key.equals(eKey))) {
                return e.value;
            }
        }
        return null;
    }

###3 Android中的SparseArray
可参考[ Android性能优化之谈谈SparseArray,SparseBooleanArray和SparseIntArray](http://blog.csdn.net/stzy00/article/details/45035301)

SparseArray是android里为<Interger,Object> 这样的Hashmap而专门写的类,目的是提高效率，其核心是折半查找函数（binarySearch）。在Android中，当我们需要定义
HashMap <Integer, E> hashMap = new HashMap <Integer, E> ();
时，我们可以使用如下的方式来取得更好的性能.
SparseArray <E> sparseArray = new SparseArray <E> ();
