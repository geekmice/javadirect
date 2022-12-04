
## HashMap ##

![jdk1.8中HashMap数据结构](http://oss.geekmice.cn/blog/20191214222452844.png)

### 数据结构（重点）

HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的即哈希表和红黑树

首先有一个每个元素都是链表（可能表述不准确）的数组，当添加一个元素（key-value）时，就首先计算元素key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这时就添加到同一hash值的元素的后面，他们在数组的同一位置，但是形成了链表，同一各链表上的Hash值是相同的，所以说数组存放的是链表。而当链表长度太长时，链表就转换为红黑树，这样大大提高了查找的效率

当链表数组的容量超过初始容量的0.75时，再散列将链表数组扩大2倍，把原链表数组的搬移到新的数组中

### 常用变量

在 HashMap源码中，比较重要的常用变量，主要有以下这些。还有两个内部类来表示普通链表的节点和红黑树节点

```java

//默认的初始化容量为16，必须是2的n次幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//最大容量为 2^30
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认的加载因子0.75，乘以数组容量得到的值，用来表示元素个数达到多少时，需要扩容。
//为什么设置 0.75 这个值呢，简单来说就是时间和空间的权衡。
//若小于0.75如0.5，则数组长度达到一半大小就需要扩容，空间使用率大大降低，
//若大于0.75如0.8，则会增大hash冲突的概率，影响查询效率。
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//刚才提到了当链表长度过长时，会有一个阈值，超过这个阈值8就会转化为红黑树
static final int TREEIFY_THRESHOLD = 8;

//当红黑树上的元素个数，减少到6个时，就退化为链表
static final int UNTREEIFY_THRESHOLD = 6;

//链表转化为红黑树，除了有阈值的限制，还有另外一个限制，需要数组容量至少达到64，才会树化。
//这是为了避免，数组扩容和树化阈值之间的冲突。
static final int MIN_TREEIFY_CAPACITY = 64;

//存放所有Node节点的数组
transient Node<K,V>[] table;

//存放所有的键值对
transient Set<Map.Entry<K,V>> entrySet;

//map中的实际键值对个数，即数组中元素个数
transient int size;

//每次结构改变时，都会自增，fail-fast机制，这是一种错误检测机制。
//当迭代集合的时候，如果结构发生改变，则会发生 fail-fast，抛出异常。
transient int modCount;

//数组扩容阈值
int threshold;

//加载因子
final float loadFactor;					

//普通单向链表节点类
static class Node<K,V> implements Map.Entry<K,V> {
	//key的hash值，put和get的时候都需要用到它来确定元素在数组中的位置
	final int hash;
	final K key;
	V value;
	//指向单链表的下一个节点
	Node<K,V> next;

	Node(int hash, K key, V value, Node<K,V> next) {
		this.hash = hash;
		this.key = key;
		this.value = value;
		this.next = next;
	}
}

//转化为红黑树的节点类
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
	//当前节点的父节点
	TreeNode<K,V> parent;  
	//左孩子节点
	TreeNode<K,V> left;
	//右孩子节点
	TreeNode<K,V> right;
	//指向前一个节点
	TreeNode<K,V> prev;    // needed to unlink next upon deletion
	//当前节点是红色或者黑色的标识
	boolean red;
	TreeNode(int hash, K key, V val, Node<K,V> next) {
		super(hash, key, val, next);
	}
}	

```

### put流程（重点）

![putVal方法执行流程图](http://oss.geekmice.cn/blog/20191214222552803.png)

1. 判断数组是否为空，为空进行初始化;
2. 不为空，计算 k 的 hash 值，通过`(n - 1) & hash`计算应当存放在数组中的下标 index;
3. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
4. 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；
5. 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)
6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于<font color=green size=5>8</font>并且数组长度大于<font color=green size=5>64</font>， 大于的话链表转换为红黑树；判断红黑树节点个数小于<font color=green size=5>6</font>时，转换链表
7. 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。

### resize()扩容机制

在上边 put 方法中，我们会发现，当数组为空的时候，会调用 resize 方法，当数组的 size 大于阈值的时候，也会调用 resize方法。 那么看下 resize 方法都做了哪些事情吧。

```java
final Node<K,V>[] resize() {
	//旧数组
	Node<K,V>[] oldTab = table;
	//旧数组的容量
	int oldCap = (oldTab == null) ? 0 : oldTab.length;
	//旧数组的扩容阈值，注意看，这里取的是当前对象的 threshold 值，下边的第2种情况会用到。
	int oldThr = threshold;
	//初始化新数组的容量和阈值，分三种情况讨论。
	int newCap, newThr = 0;
	//1.当旧数组的容量大于0时，说明在这之前肯定调用过 resize扩容过一次，才会导致旧容量不为0。
	//为什么这样说呢，之前我在 tableSizeFor 卖了个关子，需要注意的是，它返回的值是赋给了 threshold 而不是 capacity。
	//我们在这之前，压根就没有在任何地方看到过，它给 capacity 赋初始值。
	if (oldCap > 0) {
		//容量达到了最大值
		if (oldCap >= MAXIMUM_CAPACITY) {
			threshold = Integer.MAX_VALUE;
			return oldTab;
		}
		//新数组的容量和阈值都扩大原来的2倍
		else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
				 oldCap >= DEFAULT_INITIAL_CAPACITY)
			newThr = oldThr << 1; // double threshold
	}
	//2.到这里，说明 oldCap <= 0，并且 oldThr(threshold) > 0，这就是 map 初始化的时候，第一次调用 resize的情况
	//而 oldThr的值等于 threshold，此时的 threshold 是通过 tableSizeFor 方法得到的一个2的n次幂的值(我们以16为例)。
	//因此，需要把 oldThr 的值，也就是 threshold ，赋值给新数组的容量 newCap，以保证数组的容量是2的n次幂。
	//所以我们可以得出结论，当map第一次 put 元素的时候，就会走到这个分支，把数组的容量设置为正确的值（2的n次幂)
	//但是，此时 threshold 的值也是2的n次幂，这不对啊，它应该是数组的容量乘以加载因子才对。别着急，这个会在③处理。
	else if (oldThr > 0) // initial capacity was placed in threshold
		newCap = oldThr;
	//3.到这里，说明 oldCap 和 oldThr 都是小于等于0的。也说明我们的map是通过默认无参构造来创建的，
	//于是，数组的容量和阈值都取默认值就可以了，即 16 和 12。
	else {               // zero initial threshold signifies using defaults
		newCap = DEFAULT_INITIAL_CAPACITY;
		newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
	}
	//③ 这里就是处理第2种情况，因为只有这种情况 newThr 才为0，
	//因此计算 newThr（用 newCap即16 乘以加载因子 0.75，得到 12） ，并把它赋值给 threshold
	if (newThr == 0) {
		float ft = (float)newCap * loadFactor;
		newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
				  (int)ft : Integer.MAX_VALUE);
	}
	//赋予 threshold 正确的值，表示数组下次需要扩容的阈值（此时就把原来的 16 修正为了 12）。
	threshold = newThr;
	@SuppressWarnings({"rawtypes","unchecked"})
		//我们可以发现，在构造函数时，并没有创建数组，在第一次调用put方法，导致resize的时候，才会把数组创建出来。这是为了延迟加载，提高效率。
		Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
	table = newTab;
	//如果原来的数组不为空，那么我们就需要把原来数组中的元素重新分配到新的数组中
	//如果是第2种情况，由于是第一次调用resize，此时数组肯定是空的，因此也就不需要重新分配元素。
	if (oldTab != null) {
		//遍历旧数组
		for (int j = 0; j < oldCap; ++j) {
			Node<K,V> e;
			//取到当前下标的第一个元素，如果存在，则分三种情况重新分配位置
			if ((e = oldTab[j]) != null) {
				oldTab[j] = null;
				//1.如果当前元素的下一个元素为空，则说明此处只有一个元素
				//则直接用它的hash()值和新数组的容量取模就可以了，得到新的下标位置。
				if (e.next == null)
					newTab[e.hash & (newCap - 1)] = e;
				//2.如果是红黑树结构，则拆分红黑树，必要时有可能退化为链表
				else if (e instanceof TreeNode)
					((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
				//3.到这里说明，这是一个长度大于 1 的普通链表，则需要计算并
				//判断当前位置的链表是否需要移动到新的位置
				else { // preserve order
					// loHead 和 loTail 分别代表链表旧位置的头尾节点
					Node<K,V> loHead = null, loTail = null;
					// hiHead 和 hiTail 分别代表链表移动到新位置的头尾节点
					Node<K,V> hiHead = null, hiTail = null;
					Node<K,V> next;
					do {
						next = e.next;
						//如果当前元素的hash值和oldCap做与运算为0，则原位置不变
						if ((e.hash & oldCap) == 0) {
							if (loTail == null)
								loHead = e;
							else
								loTail.next = e;
							loTail = e;
						}
						//否则，需要移动到新的位置
						else {
							if (hiTail == null)
								hiHead = e;
							else
								hiTail.next = e;
							hiTail = e;
						}
					} while ((e = next) != null);
					//原位置不变的一条链表，数组下标不变
					if (loTail != null) {
						loTail.next = null;
						newTab[j] = loHead;
					}
					//移动到新位置的一条链表，数组下标为原下标加上旧数组的容量
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
```

### HashMap怎么设定初始容量大小的吗？

一般如果`new HashMap()` 不传值，默认大小是16，负载因子是0.75， 如果自己传入初始大小k，初始化大小为 大于k的 2的整数次方，例如如果传10，大小为16。

### HashMap的哈希函数怎么设计的吗？

hash函数是先拿到 key 的hashcode，是一个32位的int值，然后让hashcode的高16位和低16位进行异或操作

### 那你知道为什么这么设计吗？

1. 一定要尽可能**降低hash碰撞**，越分散越好；

2. 算法一定要尽可能高效，因为这是**高频操作**, 因此采用**位运算**；


### 为什么采用hashcode的高16位和低16位异或能降低hash碰撞？hash函数能不能直接用key的hashcode？

 	因为key.hashCode()函数调用的是key键值类型自带的哈希函数，返回int型散列值。int值范围为**-2147483648~2147483647**，前后加起来大概40亿的映射空间。只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。你想，如果HashMap数组的初始大小才16，用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标。

### 1.8对hash函数做了优化，1.8还有别的优化吗？

1. 数组+链表改成了数组+链表或红黑树；
2. 链表的插入方式从头插法改成了尾插法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后；
3. 扩容的时候1.7需要对原数组中的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小；
4. 在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容；

### 为什么要做这几点优化

1. 防止发生hash冲突，链表长度过长，将时间复杂度由`O(n)`降为`O(logn)`;

2. 因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环；


### 那HashMap是线程安全的吗？

​	不是，在多线程环境下，1.7 会产生死循环、数据丢失、数据覆盖的问题，1.8 中会有数据覆盖的问题，以1.8为例，当A线程判断index位置为空后正好挂起，B线程开始往index位置的写入节点数据，这时A线程恢复现场，执行赋值操作，就把A线程的数据给覆盖了；还有++size这个地方也会造成多线程同时扩容等问题。

### 那你平常怎么解决这个线程不安全的问题？

Java中有HashTable、Collections.synchronizedMap、以及ConcurrentHashMap可以实现线程安全的Map

### 那有没有有序的Map？

​	LinkedHashMap 和 TreeMap

### TreeMap怎么实现有序的？LinkedHashMap怎么实现有序的？

​	TreeMap是按照Key的自然顺序或者Comprator的顺序进行排序，内部是通过红黑树来实现。所以要么key所属的类实现Comparable接口，或者自定义一个实现了Comparator接口的比较器，传给TreeMap用于key的比较。

​	LinkedHashMap内部维护了一个单链表，有头尾节点，同时LinkedHashMap节点Entry内部除了继承HashMap的Node属性，还有before 和 after用于标识前置节点和后置节点。可以实现按插入的顺序或访问顺序排序。

### hashmap为什么是线程不安全？

HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的。

### 为什么初始化容量为16

HashMap作为一种数据结构，元素在put的过程中需要进行hash运算，目的是计算出该元素存放在hashMap中的具体位置。hash运算的过程其实就是对目标元素的Key进行hashcode，再对Map的容量进行取模，而JDK 的工程师为了提升取模的效率，使用位运算代替了取模运算，这就要求Map的容量一定得是2的幂。而作为默认容量，太大和太小都不合适，所以16就作为一个比较合适的经验值被采用了。为了保证任何情况下Map的容量都是2的幂，HashMap在两个地方都做了限制。首先是，如果用户制定了初始容量，那么HashMap会计算出比该数大的第一个2的幂作为初始容量。另外，在扩容的时候，也是进行成倍的扩容，即4变成8，8变成16

初始长度为16。当长度为2^n时，在计算index时length-1的二进制全是1，既能加快运算的速度，也可以保证均匀分布

### 为什么负载因子是0.75

负载因子是和扩容机制有关的，意思是如果当前容器的容量，达到了我们设定的最大值，就要开始执行扩容操作

比如说当前的容器容量是16，负载因子是0.75,16\*0.75=12，也就是说，当容量达到了12的时候就会进行扩容操作

他的作用很简单，相当于是一个扩容机制的阈值。当超过了这个阈值，就会触发扩容机制

负载因子的作用肯定也是节省时间和空间。

```
当负载因子是1.0的时候，也就意味着，只有当数组的8个值（这个图表示了8个）全部填充了，才会发生扩容。这就带来了很大的问题，因为Hash冲突时避免不了的。当负载因子是1.0的时候，意味着会出现大量的Hash的冲突，底层的红黑树变得异常复杂。对于查询效率极其不利。这种情况就是牺牲了时间来保证空间的利用率。

因此一句话总结就是负载因子过大，虽然空间利用率上去了，但是时间效率降低了。
```

```
负载因子是0.5的时候，这也就意味着，当数组中的元素达到了一半就开始扩容，既然填充的元素少了，Hash冲突也会减少，那么底层的链表长度或者是红黑树的高度就会降低。查询效率就会增加。

但是，兄弟们，这时候空间利用率就会大大的降低，原本存储1M的数据，现在就意味着需要2M的空间。

一句话总结就是负载因子太小，虽然时间效率提升了，但是空间利用率降低了。
```

```
大致意思就是说负载因子是0.75的时候，空间利用率比较高，而且避免了相当多的Hash冲突，使得底层的链表或者是红黑树的高度比较低，提升了空间效率。
```

### HashMap允许空键空值么

HashMap最多只允许一个键为Null(多条会覆盖)，但允许多个值为Null

### 影响HashMap性能的重要参数

初始容量：创建哈希表(数组)时桶的数量，默认为 16

负载因子：哈希表在其容量自动增加之前可以达到多满的一种尺度，默认为 0.75

### HashMap的工作原理

HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象

### HashMap 的底层数组长度为何总是2的n次方

　　HashMap根据用户传入的初始化容量，利用无符号右移和按位或运算等方式计算出第一个大于该数的2的幂。使数据分布均匀，减少碰撞；当length为2的n次方时，h&(length - 1) 就相当于对length取模，而且在速度、效率上比直接取模要快得多。

### 为什么1.8改用红黑树

　　比如某些人通过找到你的hash碰撞值，来让你的HashMap不断地产生碰撞，那么相同key位置的链表就会不断增长，当你需要对这个HashMap的相应位置进行查询的时候，就会去循环遍历这个超级大的链表，性能及其地下。java8使用红黑树来替代超过8个节点数的链表后，查询方式性能得到了很好的提升，从原来的是O(n)到O(logn)。

### 1.8中的扩容为什么逻辑判断更简单

　　元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

　　因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图

　　这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。这一块就是JDK1.8新增的优化点。有一点注意区别，JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，JDK1.8不会倒置。

### hashmap重复数据校验

```java
class SpringbootMybatisCrudApplicationTests {

    @Test
    public static void contextLoads() {

        Person p1 = new Person("xiaoer", 1);
        Person p2 = new Person("san", 4);

        Map<Person, String> maps = new HashMap<Person, String>();
        maps.put(p1, "1111");
        maps.put(p2, "2222");
        System.out.println(maps);

        maps.put(p2, "333");
        System.out.println(maps);
        System.out.println(p1.hashCode());
        p1.setAge(5);
        System.out.println(maps);


        maps.put(p1, "1111");

        System.out.println(p1.hashCode());


        System.out.println(p1.hashCode());
        System.out.println(maps);
        System.out.println(maps.get(p1));
    }

    public static void main(String[] args) {
        contextLoads();
    }

}

```

在数组中存的是对象的引用，不是对象。

```
我对上面的程序debug了一遍，发现原来是因为在p1刚放进去的时候是由Person的name和age的hashcolde散列码共同构成的，但是，但是，但是，重要说3遍！由于放进去的只是对象的引用，所以当下面的p1.setAge(5);  时候所以数组中的值发生变化，也说明他本身的hashcode即hash发生变化，但是，但是，但是，重要说3遍！尽管现在他的散列码发生变化，然而他已经在hashmap数组中了，他位置没有发生变化，即意味着他现在变化后的hash的位置空着，所以对hashmap添加的时候就又添加进去，方在现在的位置上.
```

### 何时链表转换为红黑树

HashMap在jdk1.8之后引入了红黑树的概念，表示若桶中链表元素超过8时，会自动转化成红黑树；若桶中元素小于等于6时，树结构还原成链表形式。

原因：

红黑树的平均查找长度是log(n)，长度为8，查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，这才有转换成树的必要；链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

还有选择6和8的原因是：

中间有个差值7可以防止链表和树之间频繁的转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低

### HashMap 和 ConcurrentHashMap 的区别

 ConcurrentHashMap对整个桶数组进行了分割分段(Segment)，然后在每一个分段上都用lock锁进行保护，相对于HashTable的synchronized锁的粒度更精细了一些，并发性能更好，而HashMap没有锁机制，不是线程安全的。（JDK1.8之后ConcurrentHashMap启用了一种全新的方式实现,利用CAS算法。）
HashMap的键值对允许有null，但是ConCurrentHashMap都不允许

### 平时开发中遍历HashMap方案有哪些

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for(Map.Entry<Integer, Integer> entry : map.entrySet()){
	System.out.println("key = " + entry.getKey() + ", value = " + entry.getValue())
}
```

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
 
//iterating over keys only
for (Integer key : map.keySet()) {
	System.out.println("Key = " + key);
}
 
//iterating over values only
for (Integer value : map.values()) {
	System.out.println("Value = " + value);
}
```



## ArrayList ##

### ArrayList理解，谈谈你的理解（重点）

ArrayList是基于数组实现的,是一个动态数组,get和set效率高；其容量能自动增长,内存连续。

### 说说ArrayList为什么增删慢

ArrayList本质是数组的操作

增删慢：
1.每当插入或删除操作时 对应的需要向前或向后的移动元素
2.当插入元素时 需要判定是否需要扩容操作
扩容操作：创建一个新数组 增加length 再将元素放入进去
较为繁琐

查询快：
数组的访问 实际上是对地址的访问 效率是挺高的
列如 new int arr[5];
arr数组的地址假设为0x1000
arr[0] ~ arr[5] 地址可看作为 0x1000 + i * 4
首地址 + 下标 * 引用数据类型的字节大小

### ArrayList的底层实现是数组，但是数组的⼤⼩是定⻓的，如果我们不断的往⾥⾯添加数据的话，不会有问题吗？

通过⽆参构造⽅法的⽅式`ArrayList()`初始化，则赋值底层数`Object[] elementData`为⼀个默认空数组 `Object[]DEFAULTCAPACITY_EMPTY_ELEMENTDATA ={}`所以数组容量为0，只有真正对数据进⾏添加add时，才分配默认`DEFAULT_CAPACITY = 10`的初始容量。 `private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};`

### ArrayList常用方法

1、add(Object element)： 向列表的尾部添加指定的元素。
2、size()： 返回列表中的元素个数。
3、get(int index)： 返回列表中指定位置的元素，index从0开始。
4、add(int index, Object element)： 在列表的指定位置插入指定元素。
5、set(int i, Object element)： 将索引i位置元素替换为元素element并返回被替换的元素。
6、clear()： 从列表中移除所有元素。
7、isEmpty()： 判断列表是否包含元素，不包含元素则返回 true，否则返回false。
8、contains(Object o)： 如果列表包含指定的元素，则返回 true。
9、remove(int index)： 移除列表中指定位置的元素，并返回被删元素。
10、remove(Object o)： 移除集合中第一次出现的指定元素，移除成功返回true，否则返回false。
11、iterator()： 返回按适当顺序在列表的元素上进行迭代的迭代器。

### ArrayList⽤来做队列合适么

队列一般是FIFO的，如果用ArrayList做队列，就需要在数组尾部追加数据，数组头部删除数组，反过来也可以。但是无论如何总会有一个操作会涉及到数组的数据搬迁，这个是比较耗费性能的。

这个回答是错误的！

ArrayList固然不适合做队列，但是数组是非常合适的。比如ArrayBlockingQueue内部实现就是一个环形队列，它是一个定长队列，内部是用一个定长数组来实现的。另外著名的Disruptor开源Library也是用环形数组来实现的超高性能队列，具体原理不做解释，比较复杂。简单点说就是使用两个偏移量来标记数组的读位置和写位置，如果超过长度就折回到数组开头，前提是它们是定长数组。

### 可以手写一个ArrayList的简单实现

#### 确定数据结构

我们知道，ArrayList中无论什么数据都能放，是不是意味着它是一个Object类型，既然是数组，那么是不是Object[]数组类型的？所以我们定义的数据结构如下：

```
private Object[] elementData;  
private int size;
12
```

设置自定义的MyArrayList的长度为10

```
public MyArrayList(){
         this(10);
     }
 
     public MyArrayList(int initialCapacity){
         if(initialCapacity<0){
            try {
                 throw new Exception();
             } catch (Exception e) {
                e.printStackTrace();
            }
        }
        elementData = new Object[initialCapacity];
    }
1234567891011121314
```

有了存放数据的位置，接下来，我们想想如何将数据放入数组？

#### 添加数据

```
public void add(Object obj){
        elementData[size++]=obj;
}
123
```

每添加一个元素，size就会自增1，我们定义的数组长度为10，当我们添加到11个元素的时候，显然没有地方存放新添加的数据，这个时候我们需要对数组进行扩容处理

对上面代码做如下修改：

```
 public void add(Object obj){
         if(size==elementData.length){
            //创建一个新的数组，并且这个数组的长度是原数组长度的2倍
             Object[] newArray = new Object[size*2];
            //使用底层拷贝，将原数组的内容拷贝到新数组
             System.arraycopy(elementData, 0, newArray, 0, elementData.length);
            //并将新数组赋值给原数组的引用
             elementData = newArray;
         }
        //新来的元素，直接赋值
        elementData[size++]=obj;
}
123456789101112
```

#### 查询数据

接着我们看一下删除的操作。ArrayList支持两种删除方式：

- 按照下标删除
- 按照元素删除，这会删除ArrayList中与指定要删除的元素匹配的第一个元素

对于ArrayList来说，这两种删除的方法差不多，都是调用的下面一段代码：

```
 public void remove(int index){
         //删除指定位置的对象
         //a b d e
         int numMoved = size - index - 1;
         if (numMoved > 0){
             System.arraycopy(elementData, index+1, elementData, index,
                     numMoved);
         }
         elementData[--size] = null; // Let gc do its work
}
12345678910
```

其实做的事情就是两件:

- 把指定元素后面位置的所有元素，利用System.arraycopy方法整体向前移动一个位置
- 最后一个位置的元素指定为null，这样让gc可以去回收它

#### 指定位置添加数据

把从指定位置开始的所有元素利用System,arraycopy方法做一个整体的复制，向后移动一个位置（当然先要用ensureCapacity方法进行判断，加了一个元素之后数组会不会不够大），然后指定位置的元素设置为需要插入的元素，完成了一次插入的操作。

```
public void add(int index,Object obj){
        ensureCapacity();  //数组扩容
        System.arraycopy(elementData, index, elementData, index + 1,
                 size - index);
        elementData[index] = obj;
        size++;
}
1234567
```

#### 总结一下

从上面的几个过程总结一下ArrayList的特点：

- ArrayList底层以数组实现，是一种随机访问模式，通过下标索引定位数据，所以查找非常快
- ArrayList在顺序添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已(这里指的末尾添加数据)
- 当删除元素的时候，涉及到一次元素复制移位，如果要复制的元素很多，那么就会比较耗费性能
- 当插入元素的时候，涉及到一次元素复制移位，如果要复制的元素很多，那么就会比较耗费性能

因此，ArrayList比较适合顺序添加、随机访问的场景。

### 你什么时候要选择 ArrayList，什么时候选择 LinkedList

1、如果应用程序对数据有较多的随机访问，ArrayList对象要优于LinkedList对象；
2、如果应用程序有更多的插入或者删除操作，较少的随机访问，LinkedList对象要优于ArrayList对象；
3、不过ArrayList的插入，删除操作也不一定比LinkedList慢，如果在List靠近末尾的地方插入，那么ArrayList只需要移动较少的数据，而LinkedList则需要一直查找到列表尾部，反而耗费较多时间，这时ArrayList就比LinkedList要快。

### ArrayList中elementData为什么被transient修饰

 ArrayList在序列化的时候会调用writeObject，直接将size和element写入ObjectOutputStream；反序列化时调用readObject，从ObjectInputStream获取size和element，再恢复到elementData。
   为什么不直接用elementData来序列化，而采用上诉的方式来实现序列化呢？原因在于elementData是一个缓存数组，它通常会预留一些容量，等容量不足时再扩充容量，那么有些空间可能就没有实际存储元素，采用上诉的方式来实现序列化时，就可以保证只序列化实际存储的那些元素，而不是整个数组，从而节省空间和时间。

### ArrayList添加第一个元素过程

1、当通过 ArrayList() 构造一个空集合，初始长度是为0的，第 1 次添加元素，会创建一个长度为10的数组，并将该元素赋值到数组的第一个位置。
2、第 2 次添加元素，集合不为空，而且由于集合的长度size+1是小于数组的长度10，所以直接添加元素到数组的第二个位置，不用扩容。
3、第 11 次添加元素，此时 size+1 = 11，而数组长度是10，这时候创建一个长度为10+10*0.5 = 15 的数组（扩容1.5倍），然后将原数组元素引用拷贝到新数组。并将第 11 次添加的元素赋值到新数组下标为10的位置。
4、第 Integer.MAX_VALUE - 8 = 2147483639，然后 2147483639%1.5=1431655759（这个数是要进行扩容） 次添加元素，为了防止溢出，此时会直接创建一个 1431655759+1 大小的数组，这样一直，每次添加一个元素，都只扩大一个范围。
5、第 Integer.MAX_VALUE - 7 次添加元素时，创建一个大小为 Integer.MAX_VALUE 的数组，在进行元素添加。
6、第 Integer.MAX_VALUE + 1 次添加元素时，抛出 OutOfMemoryError 异常

### ArrayList 底层实现就是数组，访问速度本身就很快，为何还要实现 RandomAccess 

RandomAccess是一个空的接口, 空接口一般只是作为一个标识, 如Serializable接口. JDK文档说明RandomAccess是一个标记接口(Marker interface), 被用于List接口的实现类, 表明这个实现类支持快速随机访问功能(如ArrayList). 当程序在遍历这中List的实现类时, 可以根据这个标识来选择更高效的遍历方式

### ArrayList 中的 elementData 为什么是 Object 而不是泛型 E 

Java 中泛型运用的目的就是实现对象的重用，泛型T和Object类其实在编写时没有太大区别,只是JVM中没有T这个概念，T只是存在于编写时,进入虚拟机运行时,虚拟机会对泛型标志进行擦除，也就是替换T会限定类型替换（根据运行时类型）,如果没有限定就会用Object替换。同时Object可以new Object()，就是说可以实例化，而T则不能实例化。在反射方面来说，从运行时,返回一个T的实例时,不需要经过强制转换,然后Object则需要经过转换才能得到。

### ArrayList 的插入删除一定慢么？ 

取决于你删除的元素离数组末端有多远，ArrayList拿来作为堆栈来用还是挺合适的，push和pop操作完全不涉及数据移动操作

### ArrayList默认容量

ArrayList默认构造的容量为10，没错。ArrayList的底层是由一个Object[]数组构成的，而这个baiObject[]数组，默认的长度是10,所以有的文章会说ArrayList长度容量为10。然而你所指的size()方法，只的是“逻辑”长度。
所谓“逻辑”长度，是指内存已存在的“实际元素的长度”,而“空元素不被计算”,即：当你利用add()方法，向ArrayList内添加一个“元素”时，逻辑长度就增加1位。 而剩下的9个空元素不被计算。

​	ArrayList相当于在没指定initialCapacity时就是会使用延迟分配对象数组空间，当第一次插入元素时才分配10（默认）个对象空间。假如有20个数据需要添加，那么会分别在第一次的时候，将ArrayList的容量变为10 (如下图一)；之后扩容会按照1.5倍增长。也就是当添加第11个数据的时候，Arraylist继续扩容变为10*1.5=15

1、ArrayList构造函数

```java
    /**
     * E : Default initial capacity.
     * C : 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;
    
    /**
     * The size of the ArrayList (the number of elements it contains).
     * 数组元素个数
     */
    private int size;
    
    /**
     * E : Shared empty array instance used for empty instances.
     * C : 用于存放空数据的空数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**inflte : 膨胀，充气
     * Shared empty array instance used for default sized empty instances. 
     * 默认用来存放空数据的空数组
     * We distinguish this from EMPTY_ELEMENTDATA to know how much to inflate 
     * 用来区分用于存放空数据的空数组
     * when first element is added .
     * 当第一次添加数据的时候
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * arraylist中数据存储在array缓存中
     * The capacity of the ArrayList is the length of this array buffer. 
     * arraylist容量就是数组内存的长度
     * Any empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     * 任何一个空的arraylist都是用 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * 第一次添加数据时进行扩容
     */
    transient Object[] elementData; // non-private to simplify nested class access 
	// 非私有化简化嵌套类


    /**
     *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {//初始容量大于0
            //创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//初始容量等于0
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {//初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }


   /**
    *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
    *如果指定的集合为null，throws NullPointerException。
    */
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

```
以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10
```

2、分析 ArrayList 扩容机制

无参构造函数创建的 ArrayList 为例分析

add方法

```java
	/**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
   		// 添加元素之前，先调用ensureCapacityInternal方法
        ensureCapacityInternal(size + 1) ;  // Increments modCount!!
        // 这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e ;
        return true ;
    }
```

ensureCapacityInternal()方法

```java
//得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 获取默认的容量和传入参数的较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
当要add 进第1个元素时，minCapacity为1，在Math.max()方法比较后，minCapacity 为10。
```

ensureExplicitCapacity()方法

```java
//判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }
```

```
当我们要 add 进第1个元素到 ArrayList 时，elementData.length 为0 （因为还是一个空的 list），因为执行了 ensureCapacityInternal() 方法 ，所以 minCapacity 此时为10。此时，minCapacity - elementData.length > 0 成立，所以会进入 grow(minCapacity) 方法。

当add第2个元素时，minCapacity 为2，此时e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，minCapacity - elementData.length > 0 不成立，所以不会进入 （执行）grow(minCapacity) 方法。

添加第3、4···到第10个元素时，依然不会执行grow方法，数组容量都为10；直到添加第11个元素，minCapacity(为11)比elementData.length（为10）要大。进入grow方法进行扩容。
```

4、grow方法

```java
/**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
       //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

```
int newCapacity = oldCapacity + (oldCapacity >> 1),所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity为偶数就是1.5倍，否则是1.5倍左右）！ 奇偶不同，比如 ：10+10/2 = 15, 33+33/2=49。如果是奇数的话会丢掉小数.

　　我们再来通过例子探究一下grow() 方法 ：

当add第1个元素时，oldCapacity 为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断不会成立，即newCapacity 不比 MAX_ARRAY_SIZE大，则不会进入 hugeCapacity 方法。数组容量为10，add方法中 return true,size增为1。
当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true,size增为11。
以此类推······
　　这里补充一点比较重要，但是容易被忽视掉的知识点：

　　①java 中的 length 属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.

　　②java 中的 length() 方法是针对字符串说的,如果想看这个字符串的长度则用到 length() 这个方法.

　　③java 中的 size() 方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!
```

5、hugeCapacity()方法

从上面 grow() 方法源码我们知道： 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) hugeCapacity() 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 Integer.MAX_VALUE - 8。

```java
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //对minCapacity和MAX_ARRAY_SIZE进行比较
        //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
        //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

6、Arrays.copyOf()与System.arraycopy()方法

```java
	/**
     * 在此列表中的指定位置插入指定的元素。
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()方法实现数组自己复制自己
        //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }
// 测试
public class ArraycopyTest {

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        int[] a = new int[10];
        a[0] = 0;
        a[1] = 1;
        a[2] = 2;
        a[3] = 3;
        System.arraycopy(a, 2, a, 3, 3);
        a[2]=99;
        for (int i = 0; i < a.length; i++) {
            System.out.println(a[i]);
        }
    }
}

// 结果
0 1 99 2 3 0 0 0 0 0

```

```java
/**
     以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 返回的数组的运行时类型是指定数组的运行时类型。
     */
    public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
        return Arrays.copyOf(elementData, size);
    }
    
 // 测试
public class ArrayscopyOfTest {

    public static void main(String[] args) {
        int[] a = new int[3];
        a[0] = 0;
        a[1] = 1;
        a[2] = 2;
        int[] b = Arrays.copyOf(a, 10);
        System.out.println("b.length"+b.length);
    }
}
// 结果
10
```

　　联系： 看两者源代码可以发现 copyOf() 内部实际调用了 System.arraycopy() 方法 

　　区别： arraycopy() 需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 copyOf() 是系统自动在内部新建一个数组，并返回该数组。

### 遍历一个 List 有哪些不同的方式 每种方法的实现原理是什么 Java 中 List 遍历的最佳实践是什么

for 循环遍历，基于计数器。在集合外部维护一个计数器，然后依次读取每一个位置的元素，当读取到最后一个元素后停止。

迭代器遍历，Iterator。Iterator 是面向对象的一个设计模式，目的是屏蔽不同数据集合的特点，统一遍历集合的接口。Java 在 Collections 中支持了 Iterator 模式。

foreach 循环遍历。foreach 内部也是采用了 Iterator 的方式实现，使用时不需要显式声明 Iterator 或计数器。优点是代码简洁，不易出错；缺点是只能做简单的遍历，不能在遍历过程中操作数据集合，例如删除、替换。

### 说一下 ArrayList 的优缺点

ArrayList的优点如下：

ArrayList 底层以数组实现，是一种随机访问模式。ArrayList 实现了 RandomAccess 接口，因此查找的时候非常快。
ArrayList 在顺序添加一个元素的时候非常方便。
ArrayList 的缺点如下：

删除元素的时候，需要做一次元素复制操作。如果要复制的元素很多，那么就会比较耗费性能。
插入元素的时候，也需要做一次元素复制操作，缺点同上。
ArrayList 比较适合顺序添加、随机访问的场景。

### 如何实现数组和 List 之间的转换

 数组转 List：使用 Arrays. asList(array) 进行转换。
List 转数组：使用 List 自带的 toArray() 方法。
代码示例：

// list to array
List<String> list = new ArrayList<String>();
list.add("123");
list.add("456");
list.toArray();

// array to list
String[] array = new String[]{"123","456"};
Arrays.asList(array);

### arraylist与linkedlist区别

LinkList是一个双链表,在添加和删除元素时具有比ArrayList更好的性能.但在get与set方面弱于ArrayList.当然,这些对比都是指数据量很大或者操作很频繁.

可以看作是能够自动增长容量的数组,连续

- 新增数据空间判断
- 新增数据的时候需要判断当前是否有空闲空间存储
- 扩容需要申请新的连续空间
- 把老的数组复制过去
- 新加的内容
- 回收老的数组空间

### 多线程场景下如何使用 ArrayList

ArrayList 不是线程安全的，如果遇到多线程场景，可以通过 Collections 的 synchronizedList 方法将其转换成线程安全的容器后再使用。例如像下面这样：

```java
List<String> synchronizedList = Collections.synchronizedList(list);
synchronizedList.add("aaa");
synchronizedList.add("bbb");

for (int i = 0; i < synchronizedList.size(); i++) {
    System.out.println(synchronizedList.get(i));
}
```

### 为什么 ArrayList 的 elementData 加上 transient 修饰？

transient 的作用是说不希望 elementData 数组被序列化，重写了 writeObject 实现

```java
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
    *// Write out element count, and any hidden stuff*
        int expectedModCount = modCount;
    s.defaultWriteObject();
    *// Write out array length*
        s.writeInt(elementData.length);
    *// Write out all elements in the proper order.*
        for (int i=0; i<size; i++)
            s.writeObject(elementData[i]);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
}
```

每次序列化时，先调用 defaultWriteObject() 方法序列化 ArrayList 中的非 transient 元素，然后遍历 elementData，只序列化已存入的元素，这样既加快了序列化的速度，又减小了序列化之后的文件大小

### ArrayList是线程安全的么？

不是，可以用Vector,Collections.synchronizedList(),原理都是給方法套个synchronized，CopyOnWriteArrayList

## HashMap和List遍历方法总结及如何遍历删除元素

### list

for循环遍历

```java
    List<String> list= new ArrayList<String>();
    famous.add("zs");
    famous.add("ls");
    famous.add("ww");
    famous.add("dz");
```

> 这是一种很常见的遍历方式，但是使用这种遍历删除元素会出现问题，原因在于删除某个元素后，list的大小发生了变化，而你的索引也在变化，所以会导致你在遍历的时候漏掉某些元素。比如当你删除第一个元素后，继续根据索引访问第二个元素后，因为删除的原因，后面的元素都往前移动了以为，所以实际访问的是第三个元素。因此，这种遍历方式可以用在读取元素，而不适合删除元素。

增强for循环遍历

```java
for(String x:list){
    if(x.equals("ls"))
    list.remove(x);
}
```

但是使用这种遍历删除元素也会出现问题，运行时会报ConcurrentModificationException异常

iterator遍历

```java
Iterator<String> it = list.iterator();
while(it.hasNext()){
   String x = it.next(); 
   if(x.equals("del")){
        it.remove();
   }
}
```

一个使用list.remove(),一个使用it.remove()。

### hashmap

```java
	private static HashMap<Integer, String> map = new HashMap<Integer, String>();;
 
	public static void main(String[] args) {
		
		 for(int i = 0; i < 10; i++){  
	            map.put(i, "value" + i);  
	        }  
	  
	
	}
```

for循环遍历

```java
        for(Map.Entry<Integer, String> entry : map.entrySet()){  
             Integer key = entry.getKey();  
             if(key % 2 == 0){  
                 System.out.println("To delete key " + key);  
                 map.remove(key);  
                 System.out.println("The key " + + key + " was deleted");  
             }  
```

这种遍历删除依旧会报ConcurrentModificationException异常，

增强for循环遍历

```java
       Set<Integer> keySet = map.keySet();
	     for(Integer key : keySet){
	            if(key % 2 == 0){
	                System.out.println("To delete key " + key);
	                keySet.remove(key);
	                System.out.println("The key " + + key + " was deleted");
	            }
	     }
```

这种遍历删除依旧会报ConcurrentModificationException异常，

iterator遍历

```java
   Iterator<Map.Entry<Integer, String>> it = map.entrySet().iterator();
        while(it.hasNext()){
            Map.Entry<Integer, String> entry = it.next();
            Integer key = entry.getKey();
            if(key % 2 == 0){
           	 System.out.println("To delete key " + key);
           	 it.remove();    
           	 System.out.println("The key " + + key + " was deleted");
 
            }
        }
```



[HashMap 链表和红黑树的转换 ](https://blog.csdn.net/weixin_37264997/article/details/106074846?utm_source=app&app_version=4.20.0&code=app_1562916241&uLinkId=usr1mkqgl919blen)
[ArrayList常见面试题 ](https://blog.csdn.net/zhouhengzhe/article/details/108319369)

[HashMap和List遍历方法总结及如何遍历删除元素 ](https://blog.csdn.net/demohui/article/details/77748809)

[面试官再问你 HashMap 底层原理，就把这篇文章甩给他看](https://blog.csdn.net/qq_26542493/article/details/105482732?utm_source=app&app_version=5.3.0&code=app_1562916241&uLinkId=usr1mkqgl919blen)
