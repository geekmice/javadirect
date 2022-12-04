## String是基本数据类型，有什么特性（重点）

不是，

String类是final类，也即意味着String类不能被继承，并且它的成员方法都默认为final方法。在Java中，被final修饰的类是不允许被继承的，并且该类中的成员方法都默认为final方法

String类其实是通过char数组来保存字符串的

String对象一旦被创建就是固定不变的了，对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象

使用字符串常量池**。**每当我们创建字符串常量时，JVM会首先检查字符串常量池，如果该字符串已经存在常量池中，那么就直接返回常量池中的实例引用。如果字符串不存在常量池中，就会实例化该字符串并且将其放到常量池中。由于String字符串的不可变性我们可以十分肯定常量池中一定不存在两个相同的字符串

## java基础类型(重点)

数值型：byte（位）、short（短整数）、int（整数）、long（长整数）、

浮点型：float（单精度）、double（双精度）、

字符型：char（字符）

布尔型：boolean（布尔值）

## java 中==和 equals 和 hashCode 的区别(1) ##

1、==若是基本数据类型比较，是比较值，若是引用类型，则比较的是他们在内存中的存 放地址。

2、equals 是 Object 类中的方法，Object 类的 equals 方法用于判断对象的内存地址引用 是不是同一个地址（是不是同一个对象）。若是类中覆盖了 equals 方法，就要根据具体代 码来确定，一般覆盖后都是通过对象的内容是否相等来判断对象是否相等。

3、hashCode()计算出对象实例的哈希码，在对象进行散列时作为 key 存入。之所以有 hashCode 方法，因为在批量的对象比较中，hashCode 比较要比 equals 快；核心思想就是将集合分成若干个存储区域（可以看成一个个桶），每个对象可以计算出一个哈希码，可以根据哈希码分组，每组分别对应某个存储区域，这样一个对象根据它的哈希码就可以分到不同的存储区域（不同的区域）

所有对于需要大量并且快速的对比的话如果都用equals()去做显然效率太低，所以解决方式是，每当需要对比的时候，首先用hashCode()去对比，如果hashCode()不一样，则表示这两个对象肯定不相等（也就是不必再用equals()去再对比了）,如果hashCode()相同，此时再对比他们的equals()，如果equals()也相同，则表示这两个对象是真的相同了，这样既能大大提高了效率也保证了对比的绝对正确性！
	因为在散列表中，hashCode()相等即两个键值对的哈希值相等，然而哈希值相等，并不一定能得出键值对相等。



## int 与 integer 的区别(1) ##

1. Ingeter是int的包装类，int的初值为0，Ingeter的初值为null。

2. Integer变量必须实例化后才能使用；int变量不需要；

3. Integer实际是对象的引用，指向此new的Integer对象；int是直接存储数据值 ；

4. Integer是int的包装类；int是基本数据类型；

5. 关于int,Integer之间进行比较说明
   1. 无论如何，Integer与new Integer不会相等。不会经历拆箱过程，new出来的对象存放在堆，而非newInteger常量则在常量池（在方法区），他们的内存地址不一样，所以为false。

   2. 两个都是非new出来的Integer，如果数在-128到127之间，则是true,否则为false。因为java在编Integer i2 = 128的时候,被翻译成：Integer i2 = Integer.valueOf(128);而valueOf()函数会对-12到127之间的数进行缓存。

   3. 两个都是new出来的,都为false。还是内存地址不一样。

   4. int和Integer(无论new否)比，都为true，因为会把Integer自动拆箱为int再去比。

6. 装箱 就是自动将基本数据类型转换为包装器类型；拆箱 就是自动将包装器类型转换为基本数据类型,从反编译的结果可以看到，在装箱的时候，调用了 Integer 的 valueOf(int i) 方法；而在拆箱的时候，则调用了 Integer 的 intValue() 方法

7. 案例说明

```java
public class IntAndIntegerTest{
	public static void main(String[] args){
/* 		  new出来的对象存放在堆，而非new的Integer常量则在常量池（在方法区），
		  他们的内存地址不一样，所以为false
*/		
		Integer i = 100 ;
		Integer i1 = new Integer(100) ;
		System.out.println("i == i1 ?" + (i==i1)) ; // false
		System.out.println("*********************") ;
/* 
	两个都是非new出来的Integer，如果数在-128到127之间，则是true,否则为false。
	因为java在编译Integer i2 = 128的时候,被翻译成：Integer i2 = Integer.valueOf(128);而valueOf()函数会对-128到127之间的数进行缓存。
*/
		Integer i3 = 127 ;
		Integer i2 = 127 ;
		Integer i4 = 129 ;
		Integer i5 = 129 ;
		System.out.println("i3 == i2 ? "+(i3==i2)) ;// true
		System.out.println("i5 == i4 ? "+(i5==i4)) ;// false
		System.out.println("*********************") ;
		Integer i6 = new Integer(129) ;
		Integer i7 = new Integer(129) ;
		System.out.println("i7 == i6 ? "+(i7==i6)) ; 
		System.out.println("*********************") ;
/* 
	int和Integer(无论new否)比，都为true，因为会把Integer自动拆箱为int再去比。
*/
		int i8 = 100 ;
		Integer i9 = 100 ;
		Integer i10 = new Integer(100) ;
		System.out.println("i8 == i9 ? "+(i8==i9)) ; 
		System.out.println("i8 == i10 ? "+(i8==i10)) ; 		
	}
}
```

8.关于Integer.valueOf()方法说明

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

IntegerCache 的源码如下

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

通过源码可以看到，在 Interger 的 valueOf 方法中，如果数值在 [-128, 127] 范围的时候，就会去 IntegerCache.cache 这个数组寻找有没有存在的 Integer 对象的引用，如果有，则直接返回对象的引用，如果没有(超过了范围)，就新建一个 Integer 对象。这个是 cache 数组中已经存在的对象，只是一部分，数组下标范围为 [0，255]，对应数值范围是 [-128，127]

9.经典面试题
面试题1

```java
Integer a = 100;
Integer b = 100;
Integer a1 = 200;
Integer b1 = 200;
Integer a2 = new Integer(100);

a == b;
a1 == b1;
a == a2;
```

分析：

> a == b 原因

> a 和 b 先各自调用 Integer 的 valueOf() 方法进行装箱，因为数值 a 和 b 的数值是100的，在 [-128，127] 这个范围之间，所以执行 Integer a = 100 这行代码的时候，在装箱的时候，会先初始化一个长为 255 的 的数组，里面的值的范围是 -128~127，变量 a 的值直接从 cache 对应的位置拿，之后执行到 Integer b = 100 的时候，便去这个已经实例化好了的数组中拿对应位置的值，该位置和变量 a 在数组中的位置相同，最终，这两个变量拿到的是同一个数组的同一个位置的变量，此时肯定是相同的

> a1 == b1 原因

> a 和 b 的数值为200时，在调用 valueOf() 方法进行装箱的时候，因为值不在 [-128，127] 间，因此没有实例化 cache 数组和从数组中拿值的过程，因此只能各自实例化一个 Integer 的对象，此时这两个对象的地址是不同，因此不同
> a == a2 原因

> a 和 a2 比较的时候，由于 a 调用 valueOf() 进行自动装箱，而 a2 已经是 Integer 对象，相当于两个不同对象之间的比较，因为地址不同，所以肯定不等

面试题2

```java
Double i1 = 100.0;
Double i2 = 100.0;
Double i3 = 200.0;
Double i4 = 200.0;

i1 == i2
i3 == i4
```

分析：
结果：false、false。

为什么呢？由于 Double 在进行装箱的时候也会调用 valueOf() 这个方法，因此看一下源代码

```java
public static Double valueOf(double d) {
    return new Double(d);
}

public Double(double value) {
    this.value = value;
}
```

可以看到，Double 并没有 Integer 的缓存机制，而是直接返回了一个新的 Double 对象，因此以上的四个引用都是指向不同的对象的，所以不同

>PS
>为什么 Double 类的 valueOf() 方法会采用与 Integer 类的 valueOf() 方法不同的实现呢？

很简单：在某个范围内的整型数值的个数是有限的，而浮点数却不是。

我又查看了其他几种数据类型的 valueOf 源码

其中，Double 和 Float 没有缓存机制，都是直接返回新的对象；Integer、Short、Byte、Character 有缓存机制

面试题3

```java
Boolean b1 = false;
Boolean b2 = false;
Boolean b3 = true;
Boolean b4 = true;

b1 == b2
b3 == b4
b1 == b3

```

>分析：
> 结果：true、true、false

为什么会这样呢？看一下 Boolean 的 valueOf() 的源代码

public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
其中的 TRUE 和 FALSE，代表两个静态成员属性

public static final Boolean TRUE = new Boolean(true);

public static final Boolean FALSE = new Boolean(false);

因此可以知道了，每次传入的 true 或 false，都是指向同一个 Boolean 对象，因此他们的引用肯定相同了

面试题4

```java
Integer a = 10;
Integer b = 20;
Integer c = 30;
Integer d = 30;
Integer e = 300;
Integer f = 300;
Integer g = a + b;

c == d;
e == f;
c == g;
c == (a + b);
c.equals(a + b);

```

分析：

> 结果：true、false、true、true、true

先注意这句话：当 "=="运算符的两个操作数都是 包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）

前两个没什么好说的

第三个：由于运用了算术运算符(+)，因此右边的算式在进行计算之前会先调用 intVal() 方法进行拆箱，在进行相加，然后得到的结果30之后，由于前面是 Integer g，因此还会再调用 valueOf() 方法进行装箱。由于此时 cache() 数组中已经有了 30 了，因此直接指向缓存池中的 30 这个 Integer 对象。此时 c == g 比较的还是对象的地址是否相同


第四个：由于右边是 a+b，包含算术运算，因此会调用 intVal() 方法，将左边的 c 进行拆箱，之后又分别对 a 和 b 进行拆箱，即一共调用了三次拆箱过程，最后比较的是数值大小，而不是地址


第五个：通过 equals 的源码看到，传入的参数需要是一个对象，这就意味着，和 equals 比较的时候，一定是装完箱之后的结果
a + b 的时候，a 和 b 各自执行 intVal() 方法进行拆箱，完成相加的计算之后，再对计算出的结果执行 valueOf() 方法进行装箱，此时再传入 equals 方法进行比较。equals 比较两个对象的时候，直接比较两个值拆箱之后的结果，即比较的是数值，本例中两个数值相同，所以输出为 true

面试题5

```java
Integer a = 10;
Integer b = 20;
Long g = 30L;
Long h = 20L;

g == (a + b);
g.equals(a + b);
g.equals(a + h);

```

>结果：true、false、true

和上面一样，同样是使用拆箱，第一题比较的拆完箱之后的值，但是需要注意的是，Integer 类型调用 Integer.valueOf 进行拆箱，而 Long 类型调用 Long.valueOf 进行拆箱

最后一个结果是 true，因为 a + h 会使小的精度转为大的精度，最终的 30 是 Long 类型的，因此结果是 true

## 谈谈对 java 多态的理解 ##

​	多态是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编 译时不确定，在运行期间才确定，一个引用变量到底会指向哪个类的实例。这样就可以不用 修改源程序，就可以让引用变量绑定到各种不同的类实现上。

如何实现？

继承、重定、向上转型，在多态中需要将子类的引用赋值给父类对象，只有这样该引用才能 够具备调用父类方法和子类的方法。



## String、StringBuffer、StringBuilder 区别(重点) ##

都是操作字符串

​	String 类中使用字符数组保存字符串，因有 final 修饰符，String 对象是不 可变的，每次对 String 操作都会生成新的 String 对象，这样效率低，且浪费内存空间。但 线程安全。

​	StringBuilder 和 StringBuffer 也是使用字符数组保存字符，但这两种对象都是可变的，即 对字符串进行 append 操作，不会产生新的对象。它们的区别是：StringBuffer 对方法加 了同步锁，是线程安全的，StringBuilder 非线程安全。

如果要操作少量的数据用 = String

单线程操作字符串缓冲区 下操作大量数据 = StringBuilder

多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

## 抽象类和接口区别(1) ##

1）抽象类可以提供成员方法实现的细节，而接口只能存在抽象方法； 

2）抽象类的成员变量可以是各种类型，而接口中成员变量只能是 public static final 类型； 

3）接口中不能含有静态方法及静态代码块，而抽象类可以有静态方法和静态代码块； 

4）一个类只能继承一个抽象类，用 extends 来继承，却可以实现多个接口，用 implements 来实现接口。



## 抽象类和接口意义 ##

接口

1）有利于代码的规范，对于大型项目，对一些接口进行定义，可以给开发人员一个清晰的 指示，防止开发人员随意命名和代码混乱，影响开发效率。 

2）有利于代码维护和扩展，当前类不能满足要求时，不需要重新设计类，只需要重新写了 个类实现对应的方法。 

3）解耦作用，全局变量的定义，当发生需求变化时，只需改变接口中的值即可。

抽象类

抽象类是用来提供子类的通用性，用来创建继承层级里子类的模板，减少代码编写，有利于 代码规范化。



## 抽象类和接口的应用场景 ##

抽象类的应用场景：1）规范了一组公共的方法，与状态无关，可以共享的，无需子类分别 实现；而另一些方法却需要各个子类根据自己特定状态来实现特定功能；

接口的应用场景： 2）定义一组接口，但不强迫每个实现类都必须实现所有的方法，可用抽象类定义一组方法 体可以是空方法体，由子类选择自己感兴趣的方法来覆盖； 

限定参数类型的上界，参数类型必须是 T 或 T 的子类型，但对于 List，不能通过 add()来加入元素，因为不知道是 T 的哪一种子类； 限定参数类型的下界，参数类型必须是 T 或 T 的父类型，不能能过 get()获取 元素，因为不知道哪个超类；

泛型上限：？extends 类型

泛型下限：？super 类型

泛型擦除：

JVM在实现泛型的时候是使用擦除功能来实现的，也就是JVM本身是不支持泛型的，比如：
List<Integer> integer = new ArrayList<Integer>();
这样的一段代码，针对调用者而言语法结构发生了改变，但是实际上针对JVM而言，它会去解析泛型语法，让后使它变成了原始版本：
List integer = new ArrayList();
这种情况就称为泛型的“擦除”，因为JVM本身不支持泛型，所以在实现的时候如同楼上的说的，这确实是Java泛型的悲哀，它没有真正使用泛型的标准原理实现JVM对泛型的编译，只是玩了个小小的花招
主要原因是JVM在推出泛型的时候实际上并没有对JDK的编译器进行改写，所以使用了擦除功能，也就是说，针对JVM本身而言：
上边两段代码都是一模一样的，没有区别，因为使用的“擦除”功能，在“擦除”过程里面，它会针对Java泛型的新语法进行类型检测操作，JVM本身不支持泛型，在编译器进行泛型代码的编译的时候，其实是使用了“擦除”功能，就是JVM在编译带泛型的代码的时候，实际上对带泛型的代码进行了类型检查，然后“擦除”泛型代码中的类型支持，转换为普通类型进行编译。这里有一个新概念成为“外露”类型——单独出现而不是位于某个类型中的类型参数如（List<T>中的T）针对T类型而言，T的上界就是Object。这一项技术的功能极其强大，我们可以使几乎所有泛型类型的精度增强，但是与JVM兼容。



## final，finally，finalize 的区别(1) ##

final: 修饰类，此类不能继承；修饰方法，此方法不能重写；修饰变量，此变量不可修改即常量。

finally: 异常处理关键字try...catch...finally 一直会执行的代码

finalize: Object类中的一个方法，垃圾回收的方法



## 说说你对 Java 反射的理解 ##

在运行状态中，对任意一个类，都能知道这个类的所有属性和方法，

对任意一个对象，都能 调用它的任意一个方法和属性。

反射的作用：开发过程中，经常会遇到某个类的某个成员变量、方法或属性是私有的，或只 对系统应用开放，这里就可以利用 java 的反射机制通过反射来获取所需的私有成员或是方 法

1) 获取类的 Class 对象实例 Class clz = Class.forName("com.zhenai.api.Apple"); 

2) 根 据 Class 对 象 实 例 获 取 Constructor 对 象 Constructor appConstructor = clz.getConstructor(); 3) 使 用 Constructor 对 象 的 newInstance 方 法 获 取 反 射 类 对 象 Object appleObj = appConstructor.newInstance(); 

3) 获取方法的 Method 对象 Method setPriceMethod = clz.getMethod("setPrice", int.class);

4) 通过 getFields()可以获取 Class 类的属性，但无法获取私有属性，而 getDeclaredFields()可 以获取到包括私有属性在内的所有属性。带有 Declared 修饰的方法可以反射到私有的方法， 没有 Declared 修饰的只能用来反射公有的方法，其他如 Annotation\Field\Constructor 也是如 此



## String 为什么要设计成不可变的？ ##

1）字符串常量池需要 String 不可变。因为 String 设计成不可变，当创建一个 String 对象时， 若此字符串值已经存在于常量池中，则不会创建一个新的对象，而是引用已经存在的对象。 如果字符串变量允许必变，会导致各种逻辑错误，如改变一个对象会影响到另一个独立对象。 

2）String 对象可以缓存 hashCode。字符串的不可变性保证了 hash 码的唯一性，因此可以缓 存 String 的 hashCode，这样不用每次去重新计算哈希码。在进行字符串比较时，可以直接 比较 hashCode，提高了比较性能； 

3）安全性。String 被许多 java 类用来当作参数，如 url 地址，文件 path 路径，反射机制所 需的 Strign 参数等，若 String 可变，将会引起各种安全隐患。

## 在使用 HashMap 的时候，用 String 做 key 有什么好处？

HashMap 内部实现是通过 key 的 hashcode 来确定 value 的存储位置，因为字符串是不可变的，所以当创建字符串时，它的 hashcode 被缓存下来，不需要再次计算，所以相比于其他对象更快。

## BIO,NIO,AIO 有什么区别（重点）

同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO

BIO：Block IO 同步阻塞式 IO，就是我们平常使用的传统 IO，它的特点是模式简单使用方便，并发处理能力低。
NIO：Non IO 同步非阻塞 IO，是传统 IO 的升级，客户端和服务器端通过 Channel（通道）通讯，实现了多路复用。
AIO：Asynchronous IO 是 NIO 的升级，也叫 NIO2，实现了异步非堵塞 IO ，异步 IO 的操作基于事件和回调机制

BIO：同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的
NIO (New I/O): NIO是一种同步非阻塞的I/O模型，在Java 1.4 中引入了NIO框架，对应 java.nio 包，提供了 Channel , Selector，Buffer等抽象。NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发
AIO (Asynchronous I/O): AIO 也就是 NIO 2。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异步IO的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。查阅网上相关资料，我发现就目前来说 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了

## io流分类（重点）

从大的方面来分可以分为字节流和字符流。

字符流提供了提供了reader和writer;字节流提供了outputstream 和inputstream.

## 反射（重点）

对于任意一个类，都能够知道这个类的所有属性和方法

对任意一个对象，都能 调用它的任意一个方法和属性

**应用场景**

1.jdbc链接数据库时候使用反射  Class.forName();

2.Spring框架也用到很多反射机制，最经典的就是xml的配置模式。Spring 通过 XML 配置模式装载 Bean 的过程：

1) 将程序内所有 XML 或 Properties 配置文件加载入内存中;
2) Java类里面解析xml或properties里面的内容，得到对应实体类的字节码字符串以及相关的属性信息; 
3) 使用反射机制，根据这个字符串获得某个类的Class实例; 
4) 动态配置实例的属性

**获取反射对象**

```java
//方式一(通过建立对象)
Student stu = new Student();
Class classobj1 = stu.getClass();
System.out.println(classobj1.getName());
//方式二（所在通过路径-相对路径）
Class classobj2 = Class.forName("fanshe.Student");
System.out.println(classobj2.getName());
//方式三（通过类名）
Class classobj3 = Student.class;
System.out.println(classobj3.getName());
```



[创建线程四种方案](https://www.cnblogs.com/li666/p/11139968.html)
[bio,nio,aio](https://blog.csdn.net/skiof007/article/details/52873421)
[SpringBoot多线程定时任务](https://blog.csdn.net/qq_42920045/article/details/104948423?ops_request_misc=&request_id=&biz_id=&utm_medium=distribute.pc_search_result.none-task-code-2~all~es_rank~default-5-104948423-0.pc_search_all_es&utm_term=%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%BA%94%E7%94%A8+%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)
