# JavaSE笔记

## 一, 数据类型和运算符

### 1. 基础类型

**Java中定义了3类8种基本数据类型**

- 数值型-   byte, short, int , long, float, double
- 字符型-   char
- 布尔型-   boolean 

**引用数据类型**

- 类(class)
- 接口(interface)
- 数组

引用数据类型的大小统一为4个字节, 在栈中记录的是其引用对象在堆中的地址



### 2. 整型变量

| **表**2-4整型数据类型 |                  |                                                           |
| --------------------- | ---------------- | --------------------------------------------------------- |
| **类型**              | **占用存储空间** | **表数范围**                                              |
| byte                  | 1字节            | -2的7次方 ~   2的7次方-1（-128~127）                      |
| short                 | 2字节            | -2的15次方 ~   2的15次方-1（-32768~32767）                |
| int                   | 4字节            | -2的31次方 ~   2的31次方-1 (-2147483648~2147483647)约21亿 |
| long                  | 8字节            | -2的63次方 ~   2的63次方-1                                |

Java语言的整型常数默认为int型，声明long型常量可以后加‘ l ’或‘ L ’ 。

```Java
long a = 55555555;  //编译成功，在int表示的范围内(21亿内)。
long b = 55555555555;//不加L编译错误，已经超过int表示的范围。
```



### 3. 浮点型变量/常量

**【示例2-11】使用科学记数法给浮点型变量赋值**

```Java
`double f = 314e2;  //314*10^2-->31400.0``double f2 = 314e-2; //314*10^(-2)-->3.14`
```

​        float类型的数值有一个后缀F或者f ，没有后缀F/f的浮点数值默认为double类型。也可以在浮点数值后添加后缀D或者d， 以明确其为double类型。

**【示例2-12】float类型赋值时需要添加后缀F/f**

```Java
`float  f = 3.14F;``double d1  = 3.14;``double d2 = 3.14D;`
```

**老鸟建议**

- 浮点类型float，double的数据不适合在不容许舍入误差的金融计算领域。如果需要进行不产生舍入误差的精确数字计算，需要使用BigDecimal类。

**【示例2-13】浮点数的比较一** 

```Java
`float f = 0.1f;``double d = 1.0/10;``System.out.println(f==d);//结果为false`
```

**【示例2-14】浮点数的比较二**

```Java
`float d1 = 423432423f;``float d2 = d1+1;``if(d1==d2){``   ``System.out.println("d1==d2");//输出结果为d1==d2``}else{``    ``System.out.println("d1!=d2");``}`
```

​       运行以上两个示例，发现示例2-13的结果是“false”，而示例2-14的输出结果是“d1==d2”。这是因为由于字长有限，浮点数能够精确表示的数是有限的，因而也是离散的。 浮点数一般都存在舍入误差，很多数字无法精确表示(例如0.1)，其结果只能是接近， 但不等于。二进制浮点数不能精确的表示0.1、0.01、0.001这样10的负次幂。并不是所有的小数都能可以精确的用二进制浮点数表示。

​        java.math包下面的两个有用的类：BigInteger和BigDecimal，这两个类可以处理任意长度的数值。BigInteger实现了任意精度的整数运算。BigDecimal实现了任意精度的浮点运算。

**浮点数使用总结**

- 默认是double类型
- 浮点数存在舍入误差，数字不能精确表示。如果需要进行不产生舍入误差的精确数字计算，需要使用**BigDecimal类。**
- 避免比较中使用浮点数，需要比较请使用BigDecimal类



### 4. 算数运算符

**二元运算符的运算规则：**

　　**整数运算：**

　　1. 如果两个操作数有一个为Long, 则结果也为long。

　　2. 没有long时，结果为int。即使操作数全为short，byte，结果也是int。

　　**浮点运算：**

　　3. 如果两个操作数有一个为double，则结果为double。

　　4. 只有两个操作数都是float，则结果才为float。



### 5. 逻辑运算符

| 运算符   | 说明      |                                             |
| -------- | --------- | ------------------------------------------- |
| 逻辑与   | &( 与)    | 两个操作数为true，结果才是true，否则是false |
| 逻辑或   | \|(或)    | 两个操作数有一个是true，结果就是true        |
| 短路与   | &&( 与)   | 只要有一个为false，则直接返回false          |
| 短路或   | \|\|(或)  | 只要有一个为true， 则直接返回true           |
| 逻辑非   | !（非）   | 取反：!false为true，!true为false            |
| 逻辑异或 | ^（异或） | 相同为false，不同为true                     |

​      短路与和短路或采用短路的方式。从左到右计算，如果只通过运算符左边的操作数就能够确定该逻辑表达式的值，则不会继续计算运算符右边的操作数，提高效率。

**【示例2-22】短路与和逻辑与**

```Java
//1>2的结果为false，那么整个表达式的结果即为false，将不再计算2>(3/0)
boolean c = 1>2 && 2>(3/0);
System.out.println(c);
//1>2的结果为false，那么整个表达式的结果即为false，还要计算2>(3/0)，0不能做除数，//会输出异常信息
boolean d = 1>2 & 2>(3/0);
System.out.println(d);
```



 ### 3. 强制类型转换

​	强制类型转换，又被称为造型，用于显式的转换一个数值的类型。在有可能丢失信息的情况下进行的转换是通过造型来完成的，但可能造成精度降低或溢出。

**【示例2-27】强制类型转换**

```Java
double x  = 3.14; 
int nx = (int)x;   //值为3
char c = 'a';
int d = c+1;
System.out.println(nx);
System.out.println(d);
System.out.println((char)d);
```

运行结果如图2-7所示。

![](.\img\1494906447385004 (1).png)

- 操作比较大的数时，要留意是否溢出，尤其是整数操作时。

**【示例2-29】常见问题一**

```Java
int money = 1000000000; //10亿
int years = 20;
//返回的total是负数，超过了int的范围
int total = money*years;
System.out.println("total="+total);
//返回的total仍然是负数。默认是int，因此结果会转成int值，再转成long。但是已经发生//了数据丢失
long total1 = money*years; 
System.out.println("total1="+total1);
//返回的total2正确:先将一个因子变成long，整个表达式发生提升。全部用long来计算。
long total2 = money*((long)years); 
System.out.println("total2="+total2);
```

**运行结果如图2-8所示。**

![](.\img\1494906673550331.png)



## 二, Java面向对象

　　面向过程(Procedure Oriented)和面向对象(Object Oriented,OO)都是对软件分析、设计和开发的一种思想,它指导着人们以不同的方式去分析、设计和开发软件。早期先有面向过程思想，随着软件规模的扩大，问题复杂性的提高，面向过程的弊端越来越明显的显示出来，出现了面向对象思想并成为目前主流的方式。两者都贯穿于软件分析、设计和开发各个阶段，对应面向对象就分别称为面向对象分析(OOA)、面向对象设计(OOD)和面向对象编程(OOP)。C语言是一种典型的面向过程语言，Java是一种典型的面向对象语言。

**建议**

　　1.面向对象具有三大特征：封装性、继承性和多态性，而面向过程没有继承性和多态性，并且面向过程的封装只是封装功能，而面向对象可以封装数据和功能。所以面向对象优势更明显。

　　2.一个经典的比喻：面向对象是盖浇饭、面向过程是蛋炒饭。盖浇饭的好处就是“菜”“饭”分离，从而提高了制作盖浇饭的灵活性。饭不满意就换饭，菜不满意换菜。用软件工程的专业术语就是“可维护性”比较好，“饭” 和“菜”的耦合度比较低。

**对象**:

　　1.对象说白了也是一种数据结构(对数据的管理模式)，将数据和数据的行为放到了一起。

　　2.在内存上，对象就是一个内存块，存放了相关的数据集合!

　　3.对象的本质就一种数据的组织方式!

### 1. 面向对象的内存分析

​	**Java1.8以前Java虚拟机的内存可以分为五个区域: 栈stack, 堆heap, 方法区method area, 本地方法栈, 程序计数器**

####  **栈的特点如下:**

1. 栈描述的是方法执行的内存模型. 每个方法被调用都会创建一个栈帧(存储局部变量, 操作数, 指向运行时常

   量池的引用, 方法返回地址等)	

2. JVM为每个线程创建一个栈, 用于存放该线程执行方法的信息(实际参数, 局部变量等)

3. 栈属于线程私有, 不能实现线程间的共享!

4. 栈的存储特性是 "先进后出, 后进先出"

5. 栈是由系统自动分配, 速度快!栈是一个连续的内存空间

#### **堆的特点:**

​	1. 堆用于存储创建好的对象和数组(数组也是对象)

​	2. JVM只有一个堆, 被所有线程共享

​	3. 堆是一个不连续性的内存空间, 分配灵活, 速度慢!

#### **方法区特点:**

​	方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被类加载器加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java堆区分开来。方法区又称为“永久代”（Permanent Generation）。从jdk1.7已经开始准备“去永久代”的规划，jdk1.7的HotSpot中，已经把原本放在方法区中的静态变量、字符串常量池等移到堆内存中。

​	**在jdk1.8中，永久代** **已经不存在，存储的类信息、编译后的代码数据,运行时常量池等已经移动到了元空间（MetaSpace）中，元空间并没有处于堆内存上，而是直接占用的本地内存（NativeMemory）。**

​	元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制

三种情况：
java7之前，方法区位于永久代(PermGen)，永久代和堆相互隔离，永久代的大小在启动JVM时可以设置一个固定值，不可变；
java7中，存储在永久代的部分数据就已经转移到Java Heap或者Native memory。但永久代仍存在于JDK 1.7中，并没有完全移除，譬如符号引用(Symbols)转移到了native memory；字符串常量池(interned strings)转移到了Java heap；类的静态变量(class statics)转移到了Java heap。
java8中，取消永久代，方法区存放于元空间(Metaspace)，元空间仍然与堆不相连，但与堆共享物理内存，逻辑上可认为在堆中.**Native memory：本地内存，也称为C-Heap，是供JVM自身进程使用的。当Java Heap空间不足时会触发GC，但Native memory空间不够却不会触发GC。**

**Java1.8之前: **

![](.\img\20180821135022994.png)

**Java1.8之后: **

![](.\img\20180312125453153.jpg)



#### **常量池:**

用**final修饰**的成员变量表示常量，值一旦给定就无法改变！

修饰方法：该方法不可被子类重写。但是可以被重载!

final修饰的变量有三种：静态变量、实例变量和局部变量，分别表示三种类型的常量。

Java中的常量池分为三种类型：

- 类文件中常量池（The Constant Pool）
- 运行时常量池（The Run-Time Constant Pool）
- String常量池

**类文件常量池 ---- 存在于Class文件中**

所处区域：class文件中

诞生时间：编译时

内容概要：符号引用和字面量 ( 符号引用包括：1.类的全限定名，2.字段名和描述，3.方法名和描述。)

class常量池是在编译的时候每个class都有的，在编译阶段，存放的是常量的符号引用。class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池(constant pool table)，用于存放编译器生成的各种字面量(Literal)和符号引用(Symbolic References)。 字面量就是我们所说的常量概念，如文本字符串、被声明为final的常量值等。 符号引用是一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可（它与直接引用区分一下，直接引用一般是指向方法区的本地指针，相对偏移量或是一个能间接定位到目标的句柄）。一般包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

**运行时常量池 ---- 存在于内存的元空间中**

诞生时间：JVM运行时

jvm在执行某个类的时候，必须经过加载、链接、初始化，而链接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。class常量池中存的是字面量和符号引用，也就是说他们存的并不是对象的实例，而是对象的符号引用值。而经过解析（resolve）之后，也就是把符号引用替换为直接引用，解析的过程会去查询全局字符串池，也就是我们上面所说的StringTable，以保证运行时常量池所引用的字符串与全局字符串池中所引用的是一致的。

举个实例来说明一下:

```Java
public class HelloWorld {
    public static void main(String []args) {
		String str1 = "abc"; 
		String str2 = new String("def"); 
		String str3 = "abc"; 
		String str4 = str2.intern(); 
		String str5 = "def"; 
		System.out.println(str1 == str3);//true 
		System.out.println(str2 == str4);//false 
		System.out.println(str4 == str5);//true
    }
}
```

回到上面的那个程序，现在就很容易解释整个程序的内存分配过程了，首先，在堆中会有一个”abc”实例，全局StringTable中存放着”abc”的一个引用值，然后在运行第二句的时候会生成两个实例，一个是”def”的实例对象，并且StringTable中存储一个”def”的引用值，还有一个是new出来的一个”def”的实例对象，与上面那个是不同的实例，当在解析str3的时候查找StringTable，里面有”abc”的全局驻留字符串引用，所以str3的引用地址与之前的那个已存在的相同，str4是在运行的时候调用intern()函数，返回StringTable中”def”的引用值，如果没有就将str2的引用值添加进去，在这里，StringTable中已经有了”def”的引用值了，所以返回上面在new str2的时候添加到StringTable中的 “def”引用值，最后str5在解析的时候就也是指向存在于StringTable中的”def”的引用值，那么这样一分析之后，下面三个打印的值就容易理解了。上面程序的首先经过编译之后，在该类的class常量池中存放一些符号引用，然后**类加载之后，将class常量池中存放的符号引用转存到运行时常量池中**，然后经过验证，准备阶段之后，在堆中生成驻留字符串的实例对象（也就是上例中str1所指向的”abc”实例对象），然后将这个对象的引用存到全局String Pool中，也就是StringTable中，最后在解析阶段，要把运行时常量池中的符号引用替换成直接引用，那么就直接查询StringTable，保证StringTable里的引用值与运行时常量池中的引用值一致，大概整个过程就是这样了。

**字符串常量池 ---- 存在于堆中**

全局字符串池里的内容是在类加载完成，经过验证，**准备阶段之后**在堆中生成字符串对象实例，然后将该字符串对象实例的引用值存到string pool中（记住：**string pool中存的是引用值而不是具体的实例对象，具体的实例对象是在堆中开辟的一块空间存放的。**）。 在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个哈希表，里面存的是驻留字符串(也就是我们常说的用双引号括起来的)的引用（而不是驻留字符串实例本身），也就是说在堆中的某些字符串实例被这个StringTable引用之后就等同被赋予了”驻留字符串”的身份。这个StringTable在每个HotSpot VM的实例只有一份，被所有的类共享。



![](.\img\20171203210517543.png)

由图可知：

- str1==str2 指向同一个堆对象，同时创建了一个常量池引用。
- str3 创建了3个堆对象(**错的,应该是1个堆对象**)，只创建了一个常量池引用。
- str4 创建了2个堆对象，其中有个对象的value引用另一个的value地址，并未创建常量池引用(**错的, 实际是创建了常量池引用)。**

**总结**

- 全局常量池在每个JVM中只有一份，存放的是字符串常量的引用值。
- class常量池是在编译的时候每个class都有的，在编译阶段，存放的是常量的符号引用和字面量。
- 运行时常量池是在类加载完成之后，将每个class常量池中的符号引用值转存到运行时常量池中，也就是说，每个class都有一个运行时常量池，类在解析之后，将符号引用替换成直接引用，与全局常量池中的引用值保持一致。
- 字面量方式声明，查找常量池有则返回引用。否则，堆里生成对象，同时在在常量池生成引用。如：String s = "xyz";
- 字面量相+，根据+的结果查找常量池有则返回引用，否则，堆里生成对象，同时在常量池生成引用。如：String s = "a"+"b"; 常量池查找“ab”。最多生成三个对象。
- 字符串相+，如果有一个不是字面量，则必在堆里生成一个新对象，常量池不生成引用。如：String s=s1+"a";
- String a = new String("a"); 创建了两个堆对象, 其中有个对象的value引用另一个的value地址



#### 基本数据类型的包装类:

包装类：java提供的为原始数据类型的封装类，如：int(基本数据类型)，Integer封装类。 
对象池：为了一定程度上减少频繁创建对象，将一些对象保存到一个”容器”中。

Byte,Short,Integer,Long,Character。这5种整型的包装类的对象池范围在-128~127之间(Character除外, 它的范围在0-127)，也就是说， 
超出这个范围的对象都会开辟自己的堆内存。 
Boolean也实现了对象池技术。Double,Float两种浮点数类型的包装类则没有实现。 

String也实现了常量池技术。



自动装箱拆箱技术 :

如：Integer i = 3;（这就是自动装箱） 
实际编译代码是：Integer i=Integer.valueOf(3); 编译器自动转换 
1 Integer i = 10; //装箱 
int t = i; //拆箱，实际上执行了 int t = i.intValue();

public class Main {
    public static void main(String[] args) {
    //自动装箱
    Integer total = 99;
    //自定拆箱
    int totalprim = total;
    }
}

```Java
public class Main {
    public static void main(String[] args) {
    //自动装箱
    Integer total = 99;
    //自定拆箱
    int totalprim = total;
    }
}
```

Integer total = 99; 
执行上面那句代码的时候，系统为我们执行了： 
Integer total = Integer.valueOf(99);

int totalprim = total; 
执行上面那句代码的时候，系统为我们执行了： 
int totalprim = total.intValue();

我们现在就以Integer为例，来分析一下它的源码： 
1、首先来看看Integer.valueOf函数

```Java
public static Integer valueOf(int i) { 
	return i >= 128 || i < -128 ? new Integer(i) : SMALL_VALUES[i + 128]; 
}
```

它会首先判断i的大小：如果i小于-128或者大于等于128，就创建一个Integer对象，否则执行SMALL_VALUES[i + 128]

下面看看SMALL_VALUES[i + 128]是什么东西： 
**private static final Integer[] SMALL_VALUES = new Integer[256];** 
它是一个静态的Integer数组对象，也就是说最终valueOf返回的都是一个Integer对象。 
即Byte,Short,Integer,Long,Boolean；Character为字符型：常量池数值范围为0~127 
这5种包装类默认创建了数值[-128，127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。 

两种浮点数类型的包装类Float,Double并没有实现常量池技术。



### 2. 垃圾回收原理和算法

- **内存管理**

  ​	Java的内存管理很大程度指的就是对象的管理, 其中包括对象空间的分配和释放

  ​	对象空间的分配: 使用 **new** 关键字创建对象即可

  ​	对象空间的释放: 将对象赋值 **null** 即可, 垃圾回收器将负责回收所有 **"不可达"** 对象的内存空间

- **垃圾回收过程**

  ​	任何垃圾回收算法一般要做两件基本事情:

  ​	1. 发现无用的对象

  ​	2. 回收无用对象占用的内存空间

  ​	垃圾回收机制保证可以将  "无用的对象"  进行回收. 无用的对象指的就是没有任何变量引用该对象, Java的垃圾回收器通过相关算法发现无用对象, 并进行清理和整理.

- **垃圾回收相关算法**

  ​	1. 引用计数法

  　　堆中每个对象都有一个引用计数。被引用一次，计数加1. 被引用变量值变为null，则计数减1，直到计数为0，则表示变成无用对象。优点是算法简单，缺点是“循环引用的无用对象”无法别识别。

**【示例4-7】循环引用示例**　　

```Java
public class Student {
    String name;
    Student friend;
     
    public static void main(String[] args) {
        Student s1 = new Student();
        Student s2 = new Student();
         
        s1.friend = s2;
        s2.friend = s1;        
        s1 = null;
        s2 = null;
    }
}
```

　　s1和s2互相引用对方，导致他们引用计数不为0，但是实际已经无用，但无法被识别。

​	2. 引用可达法(根搜索算法)

　　程序把所有的引用关系看作一张图，从一个节点GC ROOT开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点。



#### 分代垃圾回收机制

​	分代垃圾回收机制, 是基于这样一个事实: 不同的对象的生命周期是不一样的. 因此, 不同生命周期的对象可以采取不同的回收算法, 以便提高收效率.  分代垃圾回收采用分治的思想，进行代的划分，把不同生命周期的对象放在不同代上，不同代上采用最适合它的垃圾回收方式进行回收。

​	堆内存分为年轻代（Young Generation）和年老代（Old Generation）。年轻代又分为两种，一种是Eden区域，另外一种是两个大小对等的Survivor区域。

![img](.\img\345531-20151115204320728-1210139023-1551797567453.png)

​	**年轻代** 

　　所有新生成的对象首先都是放在Eden区。 年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象，对应的是Minor GC，每次 Minor GC 会清理年轻代的内存，算法采用效率较高的复制算法. 年轻代分三个区。一个Eden区，两个Survivor区(一般而言,可以配置多个)。大部分对象在Eden区中生成。

​	**年老代**

　　在年轻代中经历了N(默认15)次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。年老代对象越来越多，我们就需要启动Major GC和Full GC(全量回收)，来一次大扫除，全面清理年轻代区域和年老代区域。

​	**持久代**

　　用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响。



​	**年轻代的垃圾回收**

​	当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区，然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden区能尽快空闲出来。

​	首先，新对象的内存分配都是先在Eden区域中进行的，当Eden区域的空间不足于分配新对象时，就会触发年轻代上的垃圾回收，我们称之为"minor garbage collection".同时，每个对象都有一个“年龄”，这个年龄实际上指的就是该对象经历过的minor gc的次数。如图1所示，当对象刚分配到Eden区域时，对象的年龄为“0”，当minor gc被触发后，所有存活的对象（仍然可达对象）会被拷贝到其中一个Survivor区域，同时年龄增长为“1”。并清除整个Eden内存区域中的非可达对象。

​	当第二次minor gc被触发时（如图2所示），JVM会通过Mark算法找出所有在Eden内存区域和Survivor1内存区域存活的对象，并将他们拷贝到新的Survivor2内存区域(这也就是为什么需要两个大小一样的Survivor区域的原因)，同时对象的年龄加1. 最后，清除所有在Eden内存区域和Survivor1内存区域的非可达对象。

​	当对象的年龄足够大（这个年龄可以通过JVM参数进行指定，这里假定是2），当minor gc再次发生时，它会从Survivor内存区域中升级到年老代中，如图3所示。

![img](.\img\345531-20151115205122619-1825517208.png)

**年老代的垃圾回收**

​	当minor gc发生时，又有对象从Survivor区域升级到Tenured区域，但是Tenured区域已经没有空间容纳新的对象了，那么这个时候就会触发年老代上的垃圾回收，我们称之为"major garbage collection"。而在年老代上选择的垃圾回收算法则取决于JVM上采用的是什么垃圾回收器，通过的垃圾回收器有两种：Parallel Scavenge(PS) 和Concurrent Mark Sweep(CMS)。

​	**Minor GC:**

　　用于清理年轻代区域。Eden区满了就会触发一次Minor GC。清理无用对象，将有用对象复制到“Survivor1”、“Survivor2”区中(这两个区，大小空间也相同，同一时刻Survivor1和Survivor2只有一个在用，一个为空)

　　**Major GC：**

　　用于清理老年代区域。

　　**Full GC：**

　　用于清理年轻代、年老代区域。 成本较高，会对系统性能产生影响。



### 3. this关键字

​	构造方法是创建Java对象的重要途径，通过new关键字调用构造器时，构造器也确实返回该类的对象，但这个对象并不是完全由构造器负责创建。创建一个对象分为如下四步：

　　1. 分配对象空间，并将对象成员变量初始化为0或空

　　2. 执行属性值的显示初始化

　　3. 执行构造方法

　　4. 返回对象的地址给相关的变量

　　this的本质就是“创建好的对象的地址”! 由于在构造方法调用前，对象已经创建。因此，在构造方法中也可以使用this代表“当前对象” 。



### 4. static关键字

在类中，用static声明的成员变量为静态成员变量，也称为类变量。 静态变量的生命周期和类相同，在整个应用程序执行期间都有效。它有如下特点：

　　1. 为该类的公用变量，属于类，被该类的所有实例共享，在类被载入时被显式初始化。

　　2. 对于该类的所有对象来说，static成员变量只有一份。被该类的所有对象共享!!

　　3. 一般用“类名.类属性/方法”来调用。(也可以通过对象引用或类名(不需要实例化)访问静态成员。)

　　4. 在static方法中不可直接访问非static的成员(使用static修饰了的变量和方法在类载入JVM时就初始化了,而非静态变量和方法在此时并没有被初始化, 所以静态方法内找不到非静态变量和非静态方法)。

**核心要点：**

​            **static修饰的成员变量和方法，从属于类。**

​            **普通变量和方法从属于对象的。**



### 5. 参数传递机制

Java中，方法中所有参数都是“值传递”，也就是“传递的是值的副本”。 也就是说，我们得到的是“原参数的复印件，而不是原件”。因此，复印件改变不会影响原件。

**· 基本数据类型参数的传值**

　　传递的是值的副本。 副本改变不会影响原件。

**· 引用类型参数的传值**

　　传递的是值的副本。但是引用类型指的是“对象的地址”。因此，副本和原参数都指向了同一个“地址”，改变“副本指向地址对象的值，也意味着原参数指向对象的值也发生了改变”。



### 6. JDK中主要的包

| **表****4-3 JDK****中的主要包** |                                                              |
| ------------------------------- | ------------------------------------------------------------ |
| **Java****中的常用包**          | **说明**                                                     |
| java.lang                       | 包含一些Java语言的核心类，如String、Math、Integer、System和Thread，提供常用功能。 |
| java.awt                        | 包含了构成抽象窗口工具集（abstract window toolkits）的多个类，这些类被用来构建和管理应用程序的图形用户界面(GUI)。 |
| java.net                        | 包含执行与网络相关的操作的类。                               |
| java.io                         | 包含能提供多种输入/输出功能的类。                            |
| java.util                       | 包含一些实用工具类，如定义系统特性、使用与日期日历相关的函数。 |

​	静态导入(static import)是在JDK1.5新增加的功能，其作用是用于导入指定类的静态属性，这样我们可以直接使用静态属性。

```Java
package cn.sxt;
 //以下两种静态导入的方式二选一即可
import static java.lang.Math.*;//导入Math类的所有静态属性
import static java.lang.Math.PI;//导入Math类的PI属性
 
public class Test2{
    public static void main(String [] args){
        System.out.println(PI);  //不需要类名PI了
        System.out.println(random());
    }
}
```



### 7. 继承

 instanceof是二元运算符，左边是对象，右边是类；当对象是右面类或子类所创建对象时，返回true；否则，返回false。

1.父类也称作超类、基类、派生类等。

2.Java中只有单继承，没有像C++那样的多继承。多继承会引起混乱，使得继承链过于复杂，系统难于维护。

3.Java中类没有多继承，接口有多继承。

4.子类继承父类，可以得到父类的全部属性和方法 (除了父类的构造方法)，但不见得可以直接访问(比如，父类私有的属性和方法)。

5.如果定义一个类时，没有调用extends，则它的父类是：java.lang.Object。



### 8. 重写

 子类通过重写父类的方法，可以用自身的行为替换父类的行为。方法的重写是实现多态的必要条件。

**方法的重写需要符合下面的三个要点：**

​      1.“==”： 方法名、形参列表相同。

​      2.“≤”：返回值类型和声明异常类型，子类小于等于父类。

​      3.“≥”： 访问权限，子类大于等于父类。



###9. ==和equals方法

​	“==”代表比较双方是否相同。如果是基本类型则表示值相等，如果是引用类型则表示地址相等即是同一个对象。

​      Object类中定义有：public boolean equals(Object obj)方法，提供定义“对象内容相等”的逻辑。比如，我们在公安系统中认为id相同的人就是同一个人、学籍系统中认为学号相同的人就是同一个人。

​      Object 的 equals 方法默认就是比较两个对象的hashcode，是同一个对象的引用时返回 true 否则返回 false。但是，我们可以根据我们自己的要求重写equals方法。

**equals方法测试和自定义类重写equals方法:**

```Java
public class TestEquals { 
    public static void main(String[] args) {
        Person p1 = new Person(123,"高淇");
        Person p2 = new Person(123,"高小七");     
        System.out.println(p1==p2);     //false，不是同一个对象
        System.out.println(p1.equals(p2));  //true，id相同则认为两个对象内容相同
        String s1 = new String("尚学堂");
        String s2 = new String("尚学堂");
        System.out.println(s1==s2);         //false, 两个字符串不是同一个对象
        System.out.println(s1.equals(s2));  //true,  两个字符串内容相同
    }
}
class Person {
    int id;
    String name;
    public Person(int id,String name) {
        this.id=id;
        this.name=name;
    }
    public boolean equals(Object obj) {
        if(obj == null){
            return false;
        }else {
            if(obj instanceof Person) {
                Person c = (Person)obj;
                if(c.id==this.id) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

 	**JDK提供的一些类，如String、Date、包装类等，重写了Object的equals方法，调用这些类的equals方法， x.equals (y) ，当x和y所引用的对象是同一类对象且属性内容相等时（并不一定是相同对象），返回 true 否则返回 false。**



### 10. 继承树追溯

​       super是直接父类对象的引用。可以通过super来访问父类中被子类覆盖的方法或属性。

​      使用super调用普通方法，语句没有位置限制，可以在子类中随便调用。

​      若是构造方法的第一行代码没有显式的调用super(...)或者this(...);那么Java默认都会调用super(),含义是调用父类的无参数构造方法。这里的super()可以省略。

**·属性/方法查找顺序：(比如：查找变量h)**

​      \1. 查找当前类中有没有属性h

​      \2. 依次上溯每个父类，查看每个父类中是否有h，直到Object

​      \3. 如果没找到，则出现编译错误。

​      \4. 上面步骤，只要找到h变量，则这个过程终止。

**·构造方法调用顺序：**

​      构造方法第一句总是：super(…)来调用父类对应的构造方法。所以，流程就是：先向上追溯到Object，然后再依次向下执行类的初始化块和构造方法，直到当前子类为止。

​      注：静态初始化块调用顺序，与构造方法调用顺序一样，不再重复。

**super关键字的使用**

```Java
public class TestSuper01 { 
    public static void main(String[] args) {
        new ChildClass().f();
    }
}
class FatherClass {
    public int value;
    public void f(){
        value = 100;
        System.out.println ("FatherClass.value="+value);
    }
}
class ChildClass extends FatherClass {
    public int value;
    public void f() {
        super.f();  //调用父类对象的普通方法
        value = 200;
        System.out.println("ChildClass.value="+value);
        System.out.println(value);
        System.out.println(super.value); //调用父类对象的成员变量
    }
}
```

![å¾5-5 ç¤ºä¾5-7è¿è¡ææå¾.png](.\img\1495186136513650.png)



### 11. 封装

**编程中封装的具体优点：**

1. 提高代码的安全性。

2. 提高代码的复用性。

3. “高内聚”：封装细节，便于修改内部代码，提高可维护性。

4. “低耦合”：简化外部调用，便于调用者使用，便于扩展和协作。



​	Java是使用“访问控制符”来控制哪些细节需要封装，哪些细节需要暴露的。 Java中4种“访问控制符”分别为private、default、protected、public，它们说明了面向对象的封装性，所以我们要利用它们尽可能的让访问权限降到最低，从而提高安全性。

1. private 表示私有，只有自己类能访问

2. default表示没有修饰符修饰，只有同一个包的类能访问

3. protected表示可以被同一个包的类以及其他包中的子类访问

4. public表示可以被该项目的所有包中的所有类访问



### 12. 多态

​      多态指的是同一个方法调用，由于对象不同可能会有不同的行为。现实生活中，同一个方法，具体实现会完全不同。 比如：同样是调用人的“休息”方法，张三是睡觉，李四是旅游，高淇老师是敲代码，数学教授是做数学题; 同样是调用人“吃饭”的方法，中国人用筷子吃饭，英国人用刀叉吃饭，印度人用手吃饭。

​      多态的要点：

​		1. 多态是方法的多态，不是属性的多态(多态与属性无关)。

​		2. 多态的存在要有3个必要条件：继承，方法重写，父类引用指向子类对象。

​		3. 父类引用指向子类对象后，用该父类引用调用子类重写的方法，此时多态就出现了。

**多态和类型转换测试: **

```java
class Animal {
    public void shout() {
        System.out.println("叫了一声！");
    }
}
class Dog extends Animal {
    public void shout() {
        System.out.println("旺旺旺！");
    }
    public void seeDoor() {
        System.out.println("看门中....");
    }
}
class Cat extends Animal {
    public void shout() {
        System.out.println("喵喵喵喵！");
    }
}
public class TestPolym {
    public static void main(String[] args) {
        Animal a1 = new Cat(); // 向上可以自动转型
        //传的具体是哪一个类就调用哪一个类的方法。大大提高了程序的可扩展性。
        animalCry(a1);
        Animal a2 = new Dog();
        animalCry(a2);//a2为编译类型，Dog对象才是运行时类型。
         
        //编写程序时，如果想调用运行时类型的方法，只能进行强制类型转换。
        // 否则通不过编译器的检查。
        Dog dog = (Dog)a2;//向下需要强制类型转换
        dog.seeDoor();
    }
 
    // 有了多态，只需要让增加的这个类继承Animal类就可以了。
    static void animalCry(Animal a) {
        a.shout();
    }
 
    /* 如果没有多态，我们这里需要写很多重载的方法。
     * 每增加一种动物，就需要重载一种动物的喊叫方法。非常麻烦。
    static void animalCry(Dog d) {
        d.shout();
    }
    static void animalCry(Cat c) {
        c.shout();
    }*/
}
```

![å¾5-17 ç¤ºä¾5-12è¿è¡ææå¾.png](E:\软件开发资料\Java\Java资料\img\1495253451887350.png)



​	给大家展示了多态最为多见的一种用法，即父类引用做方法的形参，实参可以是任意的子类对象，可以通过不同的子类对象实现不同的行为方式。

​      由此，我们可以看出多态的主要优势是提高了代码的可扩展性，符合开闭原则。但是多态也有弊端，就是无法调用子类特有的功能，比如，我不能使用父类的引用变量调用Dog类特有的seeDoor()方法。这时，我们就需要进行类型的强制转换，我们称之为向下转型!



### 13. 抽象

**·抽象方法**

​      使用abstract修饰的方法，没有方法体，只有声明。定义的是一种“规范”，就是告诉子类必须要给抽象方法提供具体的实现。

**·抽象类**

​      包含抽象方法的类就是抽象类。通过abstract方法定义规范，然后要求子类必须定义具体实现。通过抽象类，我们就可以做到严格限制子类的设计，使子类之间更加通用。

**抽象类和抽象方法的基本用法**

```Java
//抽象类
abstract class Animal {
    abstract public void shout();  //抽象方法
}
class Dog extends Animal { 
    //子类必须实现父类的抽象方法，否则编译错误
    public void shout() {
        System.out.println("汪汪汪！");
    }
    public void seeDoor(){
        System.out.println("看门中....");
    }
}
//测试抽象类
public class TestAbstractClass {
    public static void main(String[] args) {
        Dog a = new Dog();
        a.shout();
        a.seeDoor();
    }
}
```

**抽象类的使用要点:**

1. 有抽象方法的类只能定义成抽象类
2. 抽象类不能实例化，即不能用new来实例化抽象类。
3. 抽象类可以包含属性、方法、构造方法。但是构造方法不能用来new实例，只能用来被子类调用。
4. 抽象类只能用来被继承。
5. 抽象方法必须被子类实现。
6. 抽象类的意义在于: 为子类提供统一的, 规范的模板.



### 14. 接口

 	接口就是比“抽象类”还“抽象”的“抽象类”，可以更加规范的对实现类进行约束。全面地专业地实现了：规范和具体实现的分离。

​      抽象类还提供某些具体实现，接口不提供任何实现，接口中所有方法都是抽象方法(java8开始,接口类开始提供普通的静态方法)。接口是完全面向规范的，规定了一批类具有的公共方法规范。

​      从接口的实现者角度看，接口定义了可以向外部提供的服务。

​      从接口的调用者角度看，接口定义了实现者能提供那些服务。

​      接口和实现类不是父子关系，是实现规则的关系。比如：我定义一个接口Runnable，Car实现它就能在地上跑，Train实现它也能在地上跑，飞机实现它也能在地上跑。就是说，如果它是交通工具，就一定能跑，但是一定要实现Runnable接口。

**区别: **

1. 普通类：具体实现

2. 抽象类：具体实现，规范(抽象方法)

3. 接口：规范!

**定义接口的详细说明：**

1. 访问修饰符：只能是public或默认。

2. 接口名：和类名采用相同命名机制。

3. extends：接口可以多继承。

4. 常量：接口中的属性只能是常量，总是：public static final 修饰。不写也是。

5. 方法：接口中的方法只能是：public abstract。 省略的话，也是public abstract。

**要点: **

1. 子类通过implements来实现接口中的规范。

2. 接口不能创建实例，但是可用于声明引用变量类型。

3. 一个类实现了接口，必须实现接口中所有的方法，并且这些方法只能是public的。

4. JDK1.7之前，接口中只能包含静态常量、抽象方法，不能有普通属性、构造方法、普通方法。

5. **JDK1.8后，接口中包含普通的静态方法, 默认修饰符(default)方法**



### 15. 内部类

​      一般情况，我们把类定义成独立的单元。有些情况下，我们**把一个类放在另一个类的内部定义，称为内部类(innerclasses)。**

​      内部类可以使用public、default、protected 、private以及static修饰。而外部顶级类(我们以前接触的类)只能使用public和default修饰。	

**注意**

​      **内部类只是一个编译时概念，一旦我们编译成功，就会成为完全不同的两个类。**对于一个名为Outer的外部类和其内部定义的名为Inner的内部类。编译完成后会出现Outer.class和Outer$Inner.class两个类的字节码文件。所以**内部类是相对独立的一种存在，其成员变量/方法名可以和外部类的相同。**

**内部类介绍:**

```Java
/**外部类Outer*/
class Outer {
    private int age = 10;
    public void show(){
        System.out.println(age);//10
    }
    /**内部类Inner*/
    public class Inner {
        //内部类中可以声明与外部类同名的属性与方法
        private int age = 20;
        public void show(){
            System.out.println(age);//20
        }
    }
}
```

**内部类的作用：**

1. 内部类提供了更好的封装。只能让外部类直接访问，不允许同一个包中的其他类直接访问。

2. **内部类可以直接访问外部类的私有属性，内部类被当成其外部类的成员。 但外部类不能访问内部类的内部属性。**

3. **接口只是解决了多重继承的部分问题，而内部类使得多重继承的解决方案变得更加完整。**

**内部类的使用场合：**

1. 由于内部类提供了更好的封装特性，并且可以很方便的访问外部类的属性。所以，在只为外部类提供服务的情况下可以优先考虑使用内部类。
2. **使用内部类间接实现多继承**：每个内部类都能独立地继承一个类或者实现某些接口，所以无论外部类是否已经继承了某个类或者实现了某些接口，对于内部类没有任何影响。



#### 15.1 内部类的分类

​	在Java中内部类主要分为成员内部类(非静态内部类、静态内部类)、匿名内部类、局部内部类。

##### 成员内部类

- **成员内部类(可以使用private、default、protected、public任意进行修饰。 类文件：外部类$内部类.class)**

**a) 非静态内部类(外部类里使用非静态内部类和平时使用其他类没什么不同)**

​      i. **非静态内部类必须寄存在一个外部类对象里**。因此，如果有一个非静态内部类对象那么一定存在对应的外部类对象。非静态内部类对象单独属于外部类的某个对象。

​      ii. **非静态内部类可以直接访问外部类的成员，但是外部类不能直接访问非静态内部类成员。**

​      iii. **非静态内部类不能有静态方法、静态属性和静态初始化块。**

​      iv. **外部类的静态方法、静态代码块不能访问非静态内部类，包括不能使用非静态内部类定义变量、创建实例。**

​      **v. 成员变量访问要点：**

​		1. 内部类里方法的局部变量：变量名。

​		2. 内部类属性：this.变量名。

​		3. 外部类属性：外部类名.this.变量名。

**成员变量的访问要点**

```Java
class Outer {
    private int age = 10;
    class Inner {
        int age = 20;
        public void show() {
            int age = 30;
            System.out.println("内部类方法里的局部变量age:" + age);// 30
            System.out.println("内部类的成员变量age:" + this.age);// 20
            System.out.println("外部类的成员变量age:" + Outer.this.age);// 10
        }
    }
}
```

​      **vi. 内部类的访问：**

​	1. 外部类中定义内部类：

```Java
new Inner()
```

​	2. 外部类以外的地方使用非静态内部类：

```java
 Outer.Inner  varname = new Outer().new Inner()。
```

**内部类的访问**

```Java
public class TestInnerClass {
    public static void main(String[] args) {
        //先创建外部类实例，然后使用该外部类实例创建内部类实例
        Outer.Inner inner = new Outer().new Inner();
        inner.show();
        Outer outer = new Outer();
        Outer.Inner inn = outer.new Inner();
        inn.show();
    }
}
```

![å¾5-25 ç¤ºä¾5-21è¿è¡ææå¾.png](.\img\1495262283489060.png)

**b) 静态内部类**

​      **i. 定义方式：**

```Java
static class ClassName {
//类体
}
```

​      **ii. 使用要点：**

​	1. 当一个静态内部类对象存在，并不一定存在对应的外部类对象。 因此，**静态内部类的实例方法不能直接访问外部类的实例方法。**

​	2. 静态内部类看做外部类的一个静态成员。 因此，外部类的方法中可以通过：“静态内部类.名字”的方式访问静态内部类的静态成员，通过 new 静态内部类()访问静态内部类的实例。

**静态内部类的访问**

```Java
class Outer{
    //相当于外部类的一个静态成员
    static class Inner{
    }
}
public class TestStaticInnerClass {
    public static void main(String[] args) {
        //通过 new 外部类名.内部类名() 来创建内部类对象
        Outer.Inner inner =new Outer.Inner();
    }
}
```

##### 匿名内部类

适合那种只需要使用一次的类。

```java
new 父类构造器(实参类表) 实现接口 () {
    //匿名内部类类体！
}
```

**匿名内部类的使用**

```Java
//匿名对象
new addWindowListener(new WindowAdapter(){
        @Override
        public void windowClosing(WindowEvent e) {
            System.exit(0);
        }
    }
).windowClosing(windowEvent e);  //匿名对象只能调用一次方发

addKeyListener add = new addKeyListener(new KeyAdapter(){
        @Override
        public void keyPressed(KeyEvent e) {
            myTank.keyPressed(e);
        }      
        @Override
        public void keyReleased(KeyEvent e) {
            myTank.keyReleased(e);
        }
    }
 	add.keyPressed(KeyEvent e);
    add.keyReleased(KeyEvent e);
);
```

**注意**

1. 匿名内部类没有访问修饰符。
2. 匿名内部类没有构造方法。因为它连名字都没有那又何来构造方法呢。
3. 匿名内部类在创建对象的时候, 只能使用唯一一次.
4. 匿名对象在调用方法的时候, 只能调用唯一一次, 如果希望同一个对象, 调用多次方法,name必须给对象
5.  匿名内部类是省略了 【实现类/子类名称】, 但是匿名对象是省略了【对象名称】

##### 局部内部类

还有一种内部类，它是定义在方法内部的，作用域只限于本方法，称为局部内部类。

​      局部内部类的的使用主要是用来解决比较复杂的问题，想创建一个类来辅助我们的解决方案，到那时又不希望这个类是公共可用的，所以就产生了局部内部类。局部内部类和成员内部类一样被编译，只是它的作用域发生了改变，它只能在该方法中被使用，出了该方法就会失效。

​      局部内部类在实际开发中应用很少。

**方法中的内部类**

```Java
public class Test2 {
    public void show() {
        //作用域仅限于该方法
        class Inner {
            public void fun() {
                System.out.println("helloworld");
            }
        }
        new Inner().fun();
    }
    public static void main(String[] args) {
        new Test2().show();
    }
}
```

![å¾5-26 ç¤ºä¾5-24è¿è¡ææå¾.png](.\img\1495262685367524.png)



### 16. String类和常量池

​	在Java的内存分析中，我们会经常听到关于“常量池”的描述，实际上常量池也分了以下三种：

**1. 全局字符串常量池(String Pool)**

​      全局字符串常量池中存放的内容是在类加载完成后存到String Pool中的，在每个VM中只有一份，存放的是字符串常量的引用值(在堆中生成字符串对象实例)。

**2. class文件常量池(Class Constant Pool)**

​      class常量池是在编译的时候每个class都有的，在编译阶段，存放的是常量(文本字符串、final常量等)和符号引用。

**3. 运行时常量池(Runtime Constant Pool)**

​      运行时常量池是在类加载完成之后，将每个class常量池中的符号引用值转存到运行时常量池中，也就是说，每个class都有一个运行时常量池，类在解析之后，将符号引用替换成直接引用，与全局常量池中的引用值保持一致。

**注意:**

String  a  =  new  String("abc");是创建了两个堆对象以及将文本字符串"abc"实例对象的引用存放到了String constant pool中;

```Java
	public static void main(String[] args) throws Exception {
		String hello="hello world";
		String xx=new String("hello world");
		String yy="hello world";
		
		//输出判断内存地址是否相等
		System.out.println("xx==hello : "+ (xx==hello));
		System.out.println("yy==hello : "+ (yy==hello)+"\n");
		
		//通过反射修改hello的value值
		Field hello_field=String.class.getDeclaredField("value");
		hello_field.setAccessible(true);
		char[] value=(char[])hello_field.get(hello);
		value[5]='_';
		
		//首先输出修改结果
		System.out.println("Hello: "+hello+"\n");
		
		//然后判断内存地址是否有变化
		System.out.println("xx==hello : "+ (xx==hello));
		System.out.println("yy==hello:"+(hello==yy));
		System.out.println("xx==yy:"+(xx==yy)+"\n"); 
		
		//最后输出所有值的结果
		System.out.println("xx: "+xx);
		System.out.println("yy: "+yy);
		System.out.println("Hello: "+hello);
	}
```

```Java
xx==hello : false
yy==hello : true
 
Hello: hello_world
 
xx==hello : false
yy==hello:true
xx==yy:false
 
xx: hello_world
yy: hello_world
Hello: hello_world
```

​	**根据结果可以判断，无论是String还是new String最终都指向了String constant pool中，只不过是String直接指向了Stringconstant pool中。而new String是在Heap中创建了一个指向String constant pool中的引用。通过反射修改hello的value值之后, new  String("abc")中的文本字符串的value值也发生了改变, 由此可知, new  String("abc")中的文本字符串也在String pool中检索是否有该对象实例的引用值**

​	**要测试两个字符串除了大小写区别外是否是相等的，需要使用equalsIgnoreCase方法。**



### 17. 数组

**数组的定义**

​	数组是相同类型数据的有序集合, 数组描述的是相同类型的若干个数据, 按照一定的先后次序排序组合而成. 其中, 每个数据称作一个元素, 每个元素可以通过一个索引(下标)来访问他们. 

数组的三个基本特点:

 	1. 长度是确定的, 数组一旦被创建, 它的大小就是不可以改变的
 	2. 其元素必须是相同各类型, 不允许出现混合类型
 	3. 数组类型可以是任何数据类型, 包括基本类型和引用类型

​      **数组变量属引用类型，数组也可以看成是对象，数组中的每个元素相当于该对象的成员变量。数组本身就是对象，Java中对象是在堆中的，因此数组无论保存原始类型还是其他对象类型，数组对象本身是在堆中存储的。**



1. **数组是相同类型数据的有序集合。**

2. 数组的四个基本特点：

​      -- 其长度是确定的

​      -- 其元素必须是相同类型

​      -- 可以存储基本数据类型和引用数据类型

​      -- 数组变量属于引用类型

3. 一维数组的声明方式

​      -- type[] arr_name; (推荐使用这种方式)

​      -- type arr_name[]。

4. **数组的初始化：静态初始化、动态初始化和默认初始化。**

5. 数组的长度:数组名.length，下标的合法区间[0,数组名.length-1]。

6. **数组拷贝:System类中的static void arraycopy(object src，int srcpos，object dest， int destpos，int length)方法。**

7. 数组操作的常用类java.util.Arrays类

​      -- 打印数组:Arrays.toString(数组名);

​      -- 数组排序:Arrays.sort(数组名);

​      -- 二分查找:Arrays.binarySearch(数组名,查找的元素)。

8. 二维数组的声明

```Java
-- type[][] arr_name = new type[length][];
-- type arr_name[][] = new type[length][length]。
```

9. **数组的优缺点:**

优点：
1、按照索引查询元素速度快
2、能存储大量数据
3、按照索引遍历数组方便

缺点：
1、根据内容查找元素速度慢
2、数组的大小一经确定不能改变。
3、数组只能存储一种类型的数据
4、增加、删除元素效率慢
5、未封装任何方法，所有操作都需要用户自己定义。



**注意事项**

1. 声明的时候并没有实例化任何对象，只有在实例化数组对象时，JVM才分配空间，这时才与长度有关。

2. 声明一个数组的时候并没有数组真正被创建。

3. 构造一个数组，必须指定长度。

![å¾7-1 åºæ¬ç±"åæ°ç"åå­åéå¾.png](.\img\1495418560857133.png)

​      **数组是引用类型，它的元素相当于类的实例变量，因此数组一经分配空间，其中的每个元素也被按照实例变量同样的方式被隐式初始化。**

```Java
int a2[] = new int[2]; // 默认值：0,0
boolean[] b = new boolean[2]; // 默认值：false,false
String[] s = new String[2]; // 默认值：null, null
```



#### 17.1 冒泡排序

**冒泡排序的基础算法: **

```Java
import java.util.Arrays;
public class Test {
    public static void main(String[] args) {
        int[] values = { 3, 1, 6, 2, 9, 0, 7, 4, 5, 8 };
        bubbleSort(values);
        System.out.println(Arrays.toString(values));
    }
 
    public static void bubbleSort(int[] values) {
        int temp;
        for (int i = 0; i < values.length; i++) {
            for (int j = 0; j < values.length - 1 - i; j++) {
                if (values[j] > values[j + 1]) {
                    temp = values[j];
                    values[j] = values[j + 1];
                    values[j + 1] = temp;
                }
            }
        }
    }
}
```



**冒泡排序的优化算法:**

```Java
import java.util.Arrays;
public class Test1 {
    public static void main(String[] args) {
        int[] values = { 3, 1, 6, 2, 9, 0, 7, 4, 5, 8 };
        bubbleSort(values);
        System.out.println(Arrays.toString(values));
    }
    public static void bubbleSort(int[] values) {
        int temp;
        int i;
        // 外层循环：n个元素排序，则至多需要n-1趟循环
        for (i = 0; i < values.length - 1; i++) {
            // 定义一个布尔类型的变量，标记数组是否已达到有序状态
            boolean flag = true;
    /*内层循环：每一趟循环都从数列的前两个元素开始进行比较，比较到无序数组的最后*/
            for (int j = 0; j < values.length - 1 - i; j++) {
                // 如果前一个元素大于后一个元素，则交换两元素的值；
                if (values[j] > values[j + 1]) {
                    temp = values[j];
                    values[j] = values[j + 1];
                    values[j + 1] = temp;
                       //本趟发生了交换，表明该数组在本趟处于无序状态，需要继续比较；
                    flag = false;
                }
            }
           //根据标记量的值判断数组是否有序，如果有序，则退出；无序，则继续循环。
            if (flag) {
                break;
            }
        }
    }
}
```



####17.2 双指正算法: 

Arrays.sort()排序方法就是使用的双指正算法

```Java
public static void sortMethodTwo(int[] arrs) {
        int count = 0;
        int sun = 0;
        for (int i = 0, j = i; i < arrs.length - 1; j = ++i) {
            int ai = arrs[i + 1];
            while (ai < arrs[j]) {
                count++;
                sun++;
                arrs[j + 1] = arrs[j];
                //j--: 每一趟循环都从数列的前两个元素开始进行比较之后, 在与前面的比较
                if (j-- == 0) {
                    break;
                }
            }
            arrs[j + 1] = ai;
            System.out.println("count: " + count);
            count = 0;
        }
        System.out.println("sun: " + sun);
    }
```

#### 17.3 二分查找法

二分法检索(binary search)又称折半检索，二分法检索的基本思想是设数组中的元素从小到大有序地存放在数组(array)中，首先将给定值key与数组中间位置上元素的关键码(key)比较，如果相等，则检索成功;

​      否则，若key小，则在数组前半部分中继续进行二分法检索；

​      若key大，则在数组后半部分中继续进行二分法检索。

​      这样，经过一次比较就缩小一半的检索区间，如此进行下去，直到检索成功或检索失败。

二分法检索是一种效率较高的检索方法。比如，我们要在数组[7, 8, 9, 10, 12, 20, 30, 40, 50, 80, 100]中查询到10元素，过程如下：

![å¾7-16 äºåæ³ç¤ºæå¾.png](.\img\1495424458323195.png)

**二分法查找**

```Java
import java.util.Arrays;
public class Test {
    public static void main(String[] args) {
        int[] arr = { 30,20,50,10,80,9,7,12,100,40,8};
        int searchWord = 20; // 所要查找的数
        Arrays.sort(arr); //二分法查找之前，一定要对数组元素排序
        System.out.println(Arrays.toString(arr));
        System.out.println(searchWord+"元素的索引："+binarySearch(arr,searchWord));
    }
 
    public static int binarySearch(int[] array, int value){
        int low = 0;
        int high = array.length - 1;
        while(low <= high){
            int middle = (low + high) >> 2;
            if(value == array[middle]){
                return middle;         //返回查询到的索引位置
            }
            if(value > array[middle]){
                low = middle + 1;
            }
            if(value < array[middle]){
                high = middle - 1;
            }
        }
        return -1;     //上面循环完毕，说明未找到，返回-1
    }
}
```

![å¾7-17 ç¤ºä¾7-22è¿è¡ææå¾.png](.\img\1495424527473230.png)



## 三, 常用类

### 1. 包装类型基本知识

Java是面向对象的语言，但并不是“纯面向对象”的，因为我们经常用到的基本数据类型就不是对象。但是我们在实际应用中经常需要将基本数据转化成对象，以便于操作。比如：将基本数据类型存储到Object[]数组或集合中的操作等等。

​      为了解决这个不足，Java在设计类时为每个基本数据类型设计了一个对应的类进行代表，这样八个和基本数据类型对应的类统称为包装类(Wrapper Class)。

​      包装类均位于java.lang包，八种包装类和基本数据类型的对应关系如表8-1所示：

![è¡¨8-1åºæ¬æ°æ®ç±"åå¯¹åºçåè£ç±".png](.\img\1495593568889579.png)

​      在这八个类名中，除了Integer和Character类以外，其它六个类的类名和基本数据类型一致，只是类名的第一个字母大写而已。

​      在这八个类中，除了Character和Boolean以外，其他的都是“数字型”，“数字型”都是java.lang.Number的子类。Number类是抽象类，因此它的抽象方法，所有子类都需要提供实现。Number类提供了抽象方法：intValue()、longValue()、floatValue()、doubleValue()，意味着所有的“数字型”包装类都可以互相转型。

![å¾8-1 Numberç±"çå­ç±".png](.\img\1495593704462305.png)

![å¾8-2 Numberç±"çæ½è±¡æ¹æ³.png](.\img\1495593719360536.png)

**包装类的使用**

```Java
public class Test {
    /** 测试Integer的用法，其他包装类与Integer类似 */
    void testInteger() {
        // 基本类型转化成Integer对象
        Integer int1 = new Integer(10);
        Integer int2 = Integer.valueOf(20); // 官方推荐这种写法
        // Integer对象转化成int
        int a = int1.intValue();
        // 字符串转化成Integer对象
        Integer int3 = Integer.parseInt("334");
        Integer int4 = new Integer("999");
        // Integer对象转化成字符串
        String str1 = int3.toString();
        // 一些常见int类型相关的常量
        System.out.println("int能表示的最大整数：" + Integer.MAX_VALUE); 
    }
    public static void main(String[] args) {
        Test test  = new Test();
        test.testInteger();
    }
}
```

**自动装箱：**

​      基本类型的数据处于需要对象的环境中时，会自动转为“对象”。

**自动拆箱：**

​      每当需要一个值时，对象会自动转成基本数据类型，没必要再去显式调用intValue()、doubleValue()等转型方法。**当包装类型遇上算术运算符也会自动拆箱**

​      如 Integer i = 5;int j = i; 这样的过程就是自动拆箱。

​      我们可以用一句话总结自动装箱/拆箱：

​      自动装箱过程是通过调用包装类的valueOf()方法实现的，而自动拆箱过程是通过调用包装类的 xxxValue()方法实现的(xxx代表对应的基本数据类型，如intValue()、doubleValue()等)。

​      自动装箱与拆箱的功能事实上是编译器来帮的忙，编译器在编译时依据您所编写的语法，决定是否进行装箱或拆箱动作



### 2. String类

​	String 类对象代表不可变的Unicode字符序列，因此我们可以将String对象称为“不可变对象”。 那什么叫做“不可变对象”呢?指的是对象内部的成员变量的值无法再改变。我们打开String类的源码

![å¾8-8 Stringç±"çé¨åæºç .png](.\img\1495606355697814.png)

 我们发现字符串内容全部存储到value[]数组中，而变量value是final类型的，也就是常量(即只能被赋值一次)。 这就是“不可变对象”的典型定义方式。

**String类常用的方法有:** 

1. String类的下述方法能创建并返回一个新的String对象: concat()、 replace()、substring()、 toLowerCase()、 toUpperCase()、trim()。
2. 提供查找功能的有关方法: endsWith()、 startsWith()、 indexOf()、lastIndexOf()。
3. 提供比较功能的方法: equals()、equalsIgnoreCase()、compareTo()。
4. 其它方法: charAt() 、length()。
5. **将字符串转为char[]数组.**



输出一个字符串中字符出现的次数, 并通过字符存在值从大到小输出

```Java
public static void main(String[] args){
        String str = "aaabcbgccc";
        char[] chars = str.toCharArray();
        System.out.println(Arrays.toString(chars));
        Map<Character, Integer> map = new HashMap<Character, Integer>(5);
        for (char aChar : chars) {
            if (map.containsKey(aChar)) {
                int num = map.get(aChar);
                map.put(aChar,num + 1);
            } else {
                map.put(aChar, 1);
            }
        }
        System.out.println("map: " + map);
        sortMap(map);

    }

    public static void sortMap(Map<Character, Integer> map) {
        List<Map.Entry<Character, Integer>> entryList = new ArrayList<Map.Entry<Character, Integer>>(
                map.entrySet());
        Collections.sort(entryList, new MapValueComparator());
        System.out.println(entryList);

    }

    static class MapValueComparator implements Comparator<Map.Entry<Character, Integer>> {
        @Override
        public int compare(Map.Entry<Character, Integer> me1, Map.Entry<Character, Integer> me2) {
            //此为map的value逆序排序
            return me2.getValue().compareTo(me1.getValue());
            //此为map的value顺序排序
            //return me2.getValue().compareTo(me1.getValue());
        }
    }
```





### 3. StringBullfer和StringBuilder

​	StringBuffer和StringBuilder非常类似，均代表可变的字符序列。 这两个类都是抽象类AbstractStringBuilder的子类，方法几乎一模一样。

```Java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char value[];
//以下代码省略
}
```

​     显然，内部也是一个字符数组，但这个字符数组没有用final修饰，随时可以修改。因此，StringBuilder和StringBuffer称之为“可变字符序列”。那两者有什么区别呢?

1. **StringBuffer JDK1.0版本提供的类，线程安全，做线程同步检查， 效率较低。**

2. **StringBuilder JDK1.5版本提供的类，线程不安全，不做线程同步检查，因此效率较高。 建议采用该类。**

**要点：**

1. String：不可变字符序列。

2. StringBuffer：可变字符序列，并且线程安全，但是效率低。

3. StringBuilder：可变字符序列，线程不安全，但是效率高(一般用它)。



### 4. 日期类型

**DateFormat类和SimpleDateFormat类**

**·DateFormat类的作用**

​     把时间对象转化成指定格式的字符串。反之，把指定格式的字符串转化成时间对象。

​     DateFormat是一个抽象类，一般使用它的的子类SimpleDateFormat类来实现。

**DateFormat类和SimpleDateFormat类的使用**

```Java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
public class TestDateFormat {
    public static void main(String[] args) throws ParseException {
        // new出SimpleDateFormat对象
        SimpleDateFormat s1 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        SimpleDateFormat s2 = new SimpleDateFormat("yyyy-MM-dd");
        // 将时间对象转换成字符串
        String daytime = s1.format(new Date());
        System.out.println(daytime);
        System.out.println(s2.format(new Date()));
        System.out.println(new SimpleDateFormat("hh:mm:ss").format(new Date()));
        // 将符合指定格式的字符串转成成时间对象.字符串格式需要和指定格式一致。
        String time = "2007-10-7";
        Date date = s2.parse(time);
        System.out.println("date1: " + date);
        time = "2007-10-7 20:15:30";
        date = s1.parse(time);
        System.out.println("date2: " + date);
    }
}
```

![å¾8-16 ç¤ºä¾8-15è¿è¡ææå¾.png](.\img\1495608746205492.png)



### 5. File类

​	java.io.File类：代表文件和目录。 在开发中，读取文件、生成文件、删除文件、修改文件的属性时经常会用到本类。

**File类的常见构造方法：public File(String pathname)**

​     以pathname为路径创建File对象，如果pathname是相对路径，则默认的当前路径在系统属性user.dir中存储

```Java
import java.io.File;
public class TestFile1 {
    public static void main(String[] args) throws Exception {
        System.out.println(System.getProperty("user.dir"));
        File f = new File("a.txt"); //相对路径：默认放到user.dir目录下面
        f.createNewFile();//创建文件
        File f2 = new File("d:/b.txt");//绝对路径
        f2.createNewFile();
    }
}
```

 	在eclipse项目开发中，user.dir就是本项目的目录。因此，执行完毕后，在本项目和D盘下都生成了新的文件

**通过File对象可以访问文件的属性：**

![è¡¨8-3 Fileç±"è®¿é®å±æ§çæ¹æ³åè¡¨.png](.\img\1495611382530451.png)

**通过File对象创建空文件或目录(在该对象所指的文件或目录不存在的情况下)**

![è¡¨8-4 Fileç±"åå"ºæä"¶æç®å½çæ¹æ³åè¡¨.png](.\img\1495611400819053.png)

**使用mkdir创建目录**

```Java
import java.io.File;
public class TestFile3 {
    public static void main(String[] args) throws Exception {
        File f = new File("d:/c.txt");
        f.createNewFile(); // 会在d盘下面生成c.txt文件
        f.delete(); // 将该文件或目录从硬盘上删除
        File f2 = new File("d:/电影/华语/大陆");
        boolean flag = f2.mkdir(); //目录结构中有一个不存在，则不会创建整个目录树
        System.out.println(flag);//创建失败
    }
}
```

**使用mkdirs创建目录**

```Java
import java.io.File;
public class TestFile4 {
    public static void main(String[] args) throws Exception {
        File f = new File("d:/c.txt");
        f.createNewFile(); // 会在d盘下面生成c.txt文件
        f.delete(); // 将该文件或目录从硬盘上删除
        File f2 = new File("d:/电影/华语/大陆");
        boolean flag = f2.mkdirs();//目录结构中有一个不存在也没关系；创建整个目录树
        System.out.println(flag);//创建成功
    }
}
```

**File类的综合应用**

```Java
import java.io.File;
import java.io.IOException;
public class TestFile5 {
    public static void main(String[] args) {
        //指定一个文件
        File file = new File("d:/sxt/b.txt");
        //判断该文件是否存在
        boolean flag= file.exists();
        //如果存在就删除，如果不存在就创建
        if(flag){
            //删除
            boolean flagd = file.delete();
            if(flagd){
                System.out.println("删除成功");
            }else{
                System.out.println("删除失败");
            }
        }else{
            //创建
            boolean flagn = true;
            try {
                //如果目录不存在，先创建目录
                File dir = file.getParentFile();
                dir.mkdirs();
                //创建文件
                flagn = file.createNewFile();
                System.out.println("创建成功");
            } catch (IOException e) {
                System.out.println("创建失败");
                e.printStackTrace();
            }          
        }
        //文件重命名(同学可以自己测试一下)
        //file.renameTo(new File("d:/readme.txt"));
    }
}
```

![å¾8-28 ç¤ºä¾8-26è¿è¡ææå¾.png](.\img\1495611676693262.png)



### 6. 枚举

​	JDK1.5引入了枚举类型。枚举类型的定义包括枚举声明和枚举体。格式如下：

```java
enum  枚举名 {
      枚举体（常量列表）
}
```

​	枚举体就是放置一些常量。

```java
enum Season {
    SPRING, SUMMER, AUTUMN, WINDER 
}
```

​     **所有的枚举类型隐性地继承自 java.lang.Enum。枚举实质上还是类!而每个被枚举的成员实质就是一个枚举类型的实例，他们默认都是public static final修饰的。可以直接通过枚举类型名使用它们。**

1. 当你需要定义一组常量时，可以使用枚举类型。

2. 尽量不要使用枚举的高级特性，事实上高级特性都可以使用普通类来实现，没有必要引入枚举，增加程序的复杂性!

**枚举的使用**

```Java
import java.util.Random;
public class TestEnum {
    public static void main(String[] args) {
        // 枚举遍历
        for (Week k : Week.values()) {
            System.out.println(k);
        }
        // switch语句中使用枚举
        int a = new Random().nextInt(4); // 生成0，1，2，3的随机数
        switch (Season.values()[a]) {
        case SPRING:
            System.out.println("春天");
            break;
        case SUMMER:
            System.out.println("夏天");
            break;
        case AUTUMN:
            System.out.println("秋天");
            break;
        case WINDTER:
            System.out.println("冬天");
            break;
        }
    }
}
/**季节*/
enum Season {
    SPRING, SUMMER, AUTUMN, WINDTER
}
/**星期*/
enum Week {
    星期一, 星期二, 星期三, 星期四, 星期五, 星期六, 星期日
}
```



## 四, 容器

​     数组的优势：是一种简单的线性序列，可以快速地访问数组元素，效率高。如果从效率和类型检查的角度讲，数组是最好的。

​      数组的劣势：不灵活。容量需要事先定义好，不能随着需求的变化而扩容。比如：我们在一个用户管理系统中，要把今天注册的所有用户取出来，那么这样的用户有多少个?我们在写程序时是无法确定的。因此，在这里就不能使用数组。

​      基于数组并不能满足我们对于“管理和组织数据的需求”，所以我们需要一种**更强大、更灵活、容量随时可扩的容器来装载我们的对象。** 这就是我们今天要学习的容器，也叫集合(Collection)。以下是容器的接口层次结构图：

![å¾9-1å®¹å¨çæ¥å£å±æ¬¡ç"æå¾.png](.\img\1495613220648265.png)

### 1. 泛型Generics

​      泛型是JDK1.5以后增加的，它可以帮助我们建立类型安全的集合。在使用了泛型的集合中，遍历时不必进行强制类型转换。JDK提供了支持泛型的编译器，将运行时的类型检查提前到了编译时执行，提高了代码可读性和安全性。

​      **泛型的本质就是“数据类型的参数化”。 我们可以把“泛型”理解为数据类型的一个占位符(形式参数)，即告诉编译器，在调用泛型时必须传入实际类型。**

​	我们可以在类的声明处增加泛型列表，如：<T,E,V>。

**泛型类的声明**

```Java
class MyCollection<E> {// E:表示泛型;
    Object[] objs = new Object[5];
 
    public E get(int index) {// E:表示泛型;
        return (E) objs[index];
    }
    public void set(E e, int index) {// E:表示泛型;
        objs[index] = e;
    }
}
```

​     泛型E像一个占位符一样表示“未知的某个数据类型”，我们在真正调用的时候传入这个“数据类型”。

**泛型类的应用**

```Java
public class TestGenerics {
    public static void main(String[] args) {
        // 这里的”String”就是实际传入的数据类型；
        MyCollection<String> mc = new MyCollection<String>();
        mc.set("aaa", 0);
        mc.set("bbb", 1);
        String str = mc.get(1); //加了泛型，直接返回String类型，不用强制转换;
        System.out.println(str);
    }
}
```



### 2. Collection接口

​	Collection 表示一组对象，它是集中、收集的意思。Collection接口的两个子接口是List、Set接口。

![è¡¨9-1 Collectionæ¥å£ä¸­å®ä¹çæ¹æ³.png](.\img\1495614959696503.png)

​	由于List、Set是Collection的子接口，意味着所有List、Set的实现类都有上面的方法。



### 3. List容器

 **List是有序、可重复的容器。**

**有序：**

​	List中每个元素都有索引标记。可以根据元素的索引标记(在List中的位置)访问元素，从而精确控制这些元素。

**可重复：**

​	List允许加入重复的元素。更确切地讲，List通常允许满足 e1.equals(e2) 的元素重复加入容器。

**List接口中定义的方法**

![è¡¨9-2 Listæ¥å£ä¸­å®ä¹çæ¹æ³.png](.\img\1495616109914665.png)

**list接口常用的实现类有3个：ArrayList、LinkedList和Vector。**

**两个List之间的元素处理**

```Java
public class TestList {
    public static void main(String[] args) {
        test02();
    }
    /**
     * 测试两个容器之间元素处理
     */
    public static void test02() {
        List<String> list = new ArrayList<String>();
        list.add("高淇");
        list.add("高小七");
        list.add("高小八");
 
        List<String> list2 = new ArrayList<String>();
        list2.add("高淇");
        list2.add("张三");
        list2.add("李四");
        System.out.println(list.containsAll(list2)); //false list是否包含list2中所有元素
        System.out.println(list);
        list.addAll(list2); //将list2中所有元素都添加到list中
        System.out.println(list);
        list.removeAll(list2); //从list中删除同时在list和list2中存在的元素
        System.out.println(list);
        list.retainAll(list2); //取list和list2的交集
        System.out.println(list);
    }
}
```

![å¾9-3 ç¤ºä¾9-4è¿è¡ææå¾.png](.\img\1495616180529639.png)

**List中操作索引的常用方法**

```Java
public class TestList {
    public static void main(String[] args) {
        test03();
    }
    /**
     * 测试List中关于索引操作的方法
     */
    public static void test03() {
        List<String> list = new ArrayList<String>();
        list.add("A");
        list.add("B");
        list.add("C");
        list.add("D");
        System.out.println(list); // [A, B, C, D]
        list.add(2, "高");
        System.out.println(list); // [A, B, 高, C, D]
        list.remove(2);
        System.out.println(list); // [A, B, C, D]
        list.set(2, "c");
        System.out.println(list); // [A, B, c, D]
        System.out.println(list.get(1)); // 返回：B
        list.add("B");
        System.out.println(list); // [A, B, c, D, B]
        System.out.println(list.indexOf("B")); // 1 从头到尾找到第一个"B"
        System.out.println(list.lastIndexOf("B")); // 4 从尾到头找到第一个"B"
    }
}
```

![å¾9-5 ç¤ºä¾9-6è¿è¡ææå¾.png](.\img\1495616328435709.png)



####3.1 ArrayList特点和底层实现

 	ArrayList底层是用数组实现的存储。 特点：查询效率高，增删效率低，线程不安全。我们一般使用它。查看源码：

![å¾9-6 ArrayListåºå±æºç (.\img\1495616601500147.png).png](https://www.sxt.cn/360shop/Public/admin/UEditor/20170524/1495616601500147.png)

​      我们可以看出ArrayList底层使用Object数组来存储元素数据。所有的方法，都围绕这个核心的Object数组来开展。

​      我们知道，数组长度是有限的，而ArrayList是可以存放任意数量的对象，长度不受限制，那么它是怎么实现的呢?**本质上就是通过定义新的更大的数组，将旧数组中的内容拷贝到新数组，来实现扩容。** **ArrayList的Object数组初始化长度为10**，如果我们存储满了这个数组，需要存储第11个对象，就会定义新的长度更大的数组，并将原数组内容和新的元素一起加入到新数组中

```Java
	/**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     * ArrayList的底层Object[]默认长度是10
     * minCapacity: add后的容量
     * MAX_ARRAY_SIZE: Integer最大长值-8
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```



#### 3.2 手写ArrayList

```Java
public class JavaTestUserDefinedArrayListThree<E> {
    //数组
    private Object[] elementData;

    //数组默认长度
    private static final int DEFAULT_CAPACITY = 10;

    //容器大小
    private int size;

    public JavaTestUserDefinedArrayListThree() {
        elementData = new Object[DEFAULT_CAPACITY];
    }

    public JavaTestUserDefinedArrayListThree(int initialCapacity) {
        if (0 < initialCapacity) {
            elementData = new Object[initialCapacity];
        } else if (0 == initialCapacity) {
            elementData = new Object[DEFAULT_CAPACITY];
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                    initialCapacity);
        }
    }

    public E get(int index) {
        rangeCheck(index);
        return (E) elementData[index];
    }

    public void add(E value) {
        if (size == elementData.length) {
            //扩容操作
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1);
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
        elementData[size++] = value;
    }

    public int size() {
        return size;
    }

    public E set(int index, E val) {
        rangeCheck(index);
        E oldValue = (E) elementData[index];
        elementData[index] = val;
        return oldValue;
    }

    public E remove(int index) {
        rangeCheck(index);
        E oldValue = (E) elementData[index];
        fastRemove(index);
        return oldValue;
    }

    public boolean remove(E value) {
        if (null == value) {
            for (int i = 0; i < elementData.length; i++) {
                if (null == elementData[i]) {
                    fastRemove(i);
                    return true;
                }
            }
        } else {
            for (int i = 0; i < elementData.length; i++) {
                if (value.equals(elementData[i])) {
                    fastRemove(i);
                    return true;
                }
            }
        }
        return false;
    }

    public void fastRemove(int index) {
        int numMoved = size - index - 1;
        if (0 < numMoved) {
            /**
             * System.arraycopy(Object src, int srcPos, Object dest, int destPos, int 				 * length)
             * src: 源数组
             * srcPos: 源数组中的起始位置
             * dest: 新数组
             * destPos: 目标数组的起始位置
             * length: 要复制数组元素的数量
             */
            System.arraycopy(elementData, index + 1, elementData, index, numMoved);
        }
        elementData[--size] = null;
    }

    public void rangeCheck(int index) {
        if (index >= size || 0 > index) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (int i = 0; i < size; i++) {
            sb.append(elementData[i] + ",");
        }
        if (2 > sb.length()) {
            sb.append(']');
        } else {
            sb.setCharAt(sb.length() - 1, ']');
        }
        return sb.toString();
    }

    public void clear() {
        for (int i = 0; i < size; i++) {
            elementData[i] = null;
        }

        size = 0;
    }

    public static void main(String[] args){
        JavaTestUserDefinedArrayListThree<String> jtud = new 		                                                                       JavaTestUserDefinedArrayListThree<>(0);
        for (int i = 0; i < 23; i++) {
            jtud.add(String.valueOf(i));
        }
        jtud.set(4, "2");
        System.out.println("jtud: " + jtud);
        jtud.remove("2");
        System.out.println("newJtud: " + jtud);
    }
}
```



#### 3.3 LinkedList特点和底层实现

​      LinkedList底层用双向链表实现的存储。特点：查询效率低，增删效率高，线程不安全。

​      双向链表也叫双链表，是链表的一种，它的每个数据节点中都有两个指针，分别指向前一个节点和后一个节点。 所以，从双向链表中的任意一个节点开始，都可以很方便地找到所有节点。

![å¾9-8 LinkedListçå­å¨ç"æå¾.png](.\img\1495616843888130.png)

##### 字段属性 

```Java
	//链表元素（节点）的个数
    transient int size = 0;

    /**
     *指向第一个节点的指针
     */
    transient Node<E> first;

    /**
     *指向最后一个节点的指针
     */
    transient Node<E> last;
```

　　注意这里出现了一个 Node 类，这是 LinkedList 类中的一个内部类，其中每一个元素就代表一个 Node 类对象，LinkedList 集合就是由许多个 Node 对象类似于手拉着手构成。

```Java
private static class Node<E> {
        E item;//实际存储的元素
        Node<E> next;//指向上一个节点的引用
        Node<E> prev;//指向下一个节点的引用

        //构造函数
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

![img](.\img\1120165-20180402091402743-458763981.png)

　　上图的 LinkedList 是有四个元素，也就是由 4 个 Node 对象组成，size=4，head 指向第一个elementA,tail指向最后一个节点elementD。

##### 构造函数 

```Java
public LinkedList() {
    }
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

　　LinkedList 有两个构造函数，第一个是默认的空的构造函数，第二个是将已有元素的集合Collection 的实例添加到 LinkedList 中，调用的是 addAll() 方法，这个方法下面我们会介绍。

　　注意：LinkedList 是没有初始化链表大小的构造函数，因为链表不像数组，一个定义好的数组是必须要有确定的大小，然后去分配内存空间，而链表不一样，它没有确定的大小，通过指针的移动来指向下一个内存地址的分配。

##### LinkedList部分底层

```Java
public class LinkedListDemoOne<E> {

    //链表元素（节点）的个数
    transient int size = 0;

    /**
     *指向第一个节点的指针
     */
    transient Node<E> first;

    /**
     *指向最后一个节点的指针
     */
    transient Node<E> last;

    public LinkedListDemoOne() { }

    public LinkedListDemoOne(Collection<? extends E> c) {
        this();
    }

    public void addFirst(E e) {
        linkFirst(e);
    }

    private void linkFirst(E e) {
        //将头结点赋值给f
        final Node<E> f = first;
        //将指定元素创建成一个新节点, 此节点的next为头结点
        final Node<E> newNode = new Node<>(null, e, last);
        //将新头结点设置为头结点, 原先的头结点f变为第二个节点
        first = newNode;
        if (null == f) {
            //如果头结点为null, 尾节点为指定元素
            last = newNode;
        } else {
            //如果头结点不为空, 将原先头结点的previous指向新节点
            f.previous = newNode;
        }
        size++;
    }

    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    public void addLast(E e) {
        linkLast(e);
    }

    public void add(int index, E e) {
        checkElementIndex(index);
        if (size == index)
            //如果索引值等于链表大小, 将节点插入到尾节点
            linkLast(e);
        else
            linkBefore(e, node(index));
    }

    void linkLast(E e) {
        //将尾节点赋值给l
        final Node<E> l = last;
        //将指定元素创建成一个新的节点, 此节点的first为尾节点
        final Node<E> newNode = new Node<E>(l, e, null);
        //将新节点设置为尾结点
        last = newNode;
        if (null == l) {
            //如果尾结点为null, 头节点为指定元素
            first = newNode;
        } else {
            //如果尾结点不为空, 将原先的尾结点的next指向新节点
            l.next = newNode;
        }
        size++;
    }

    /**
     *
     * @param elem :插入的元素
     * @param e :索引对应的节点
     */
    public void linkBefore(E elem, Node<E> e) {
        //将pervious设为索引对应节点的上一个节点
        Node<E> previous = e.previous;
        //创建新节点,上一个节点的引用为pervious,下一个节点的引用为e
        Node<E> newNode = new Node<>(previous, elem, e);
        //索引对应节点的 上一个引用节点 设为 新节点
        e.previous = newNode;
        if (null == previous) {
            //如果索引对应节点为null,新节点就是first
            first = newNode;
        } else {
            //如果索引对应节点不为null,索引对应节点的上一个引用节点的next为新节点
            previous.next = newNode;
        }
        size++;
    }

    public E getFirst() {
        if (null == first)
            throw new NoSuchElementException();
        return first.element;
    }

    public E getLast() {
        if (null == last)
            throw new NoSuchElementException();
        return last.element;
    }

    public E get(int index) {
        checkElementIndex(index);
        return node(index).element;
    }

    public void checkElementIndex(int index) {
        if (0 > index || index > size) {
            throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);
        }
    }

    public void checkPositionIndex(int index) {
        if (0 > index || index >= size) {
            throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);
        }
    }

    /**
     * 找到索引对应的节点
     */
    Node<E> node(int index) {
        if (index < (size >> 1)) {
            //如果插入的索引在前半部分
            //设x为头节点
            Node<E> x = this.first;
            /*
            * 从头结点开始循环,符合条件,拿到下一个节点的引用
            * index为0,不符合循环条件,不循环,直接 return first
            * */
            for (int i = 0; i < index; i++) {
                x = x.next;
            }
            return x;
        } else {
            //如果插入的索引在后半部分
            //设x为尾节点
            Node<E> x = this.last;
            /*
            * 从尾结点开始循环,符合条件,拿到上一个节点的引用
            * 当index是尾结点,index==size-1,不符合条件,直接 return last
            * */
            for (int i = size - 1; i > index; i--) {
                x = x.previous;
            }
            return x;
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        if (null != first) {
            sb.append(first.element + ",");
            Node<E> next = first.next;
            for (int i = 0; i < size - 1; i++) {
                E element = next.element;
                sb.append(element + ",");
                next = next.next;
            }
        }
        if (null != first)
            sb.setCharAt(sb.length() - 1, ']');
        else
            sb.append(']');
        return sb.toString();
    }

    public int size() {
        return size;
    }

    private static class Node<E> {
        //实际存储的元素
        E element;
        //指向上一个节点的引用
        Node<E> previous;
        //指向下一个节点的引用
        Node<E> next;

        Node(Node<E> previous, E element, Node<E> next) {
            this.previous = previous;
            this.element = element;
            this.next = next;
        }
    }

}
```

[资料]: https://www.cnblogs.com/ysocean/p/8657850.html	"JDK源码:LinkedList"

##### 遍历集合

**普通 for 循环**

```Java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add("B");
linkedList.add("C");
linkedList.add("D");
for(int i = 0 ; i < linkedList.size() ; i++){
    System.out.print(linkedList.get(i)+" ");//A B C D
}
```

代码很简单，我们就利用 LinkedList 的 get(int index) 方法，遍历出所有的元素。

但是需要注意的是， **get(int index) 方法每次都要遍历该索引之前的所有元素。这样如果集合元素很多，越查找到后面（当然此处的get方法进行了优化，查找前半部分从前面开始遍历，查找后半部分从后面开始遍历，但是需要的时间还是很多）花费的时间越多。**

**迭代器**

```Java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add("B");
linkedList.add("C");
linkedList.add("D");


Iterator<String> listIt = linkedList.listIterator();
while(listIt.hasNext()){
    System.out.print(listIt.next()+" ");//A B C D
}

//通过适配器模式实现的接口，作用是倒叙打印链表
Iterator<String> it = linkedList.descendingIterator();
while(it.hasNext()){
    System.out.print(it.next()+" ");//D C B A
}
```

　　在 LinkedList 集合中也有一个内部类 ListItr，方法实现大体上也差不多，**通过移动游标指向每一次要遍历的元素，不用在遍历某个元素之前都要从头开始。**其方法实现也比较简单：

```Java
public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
}

private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }
}
```

　　**这里需要重点注意的是 modCount 字段，前面我们在增加和删除元素的时候，都会进行自增操作 modCount，这是因为如果想一边迭代，一边用集合自带的方法进行删除或者新增操作，都会抛出异常。（使用迭代器的增删方法不会抛异常）**

**比如:**

```Java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add("B");
linkedList.add("C");
linkedList.add("D");


Iterator<String> listIt = linkedList.listIterator();
while(listIt.hasNext()){
    System.out.print(listIt.next()+" ");//A B C D
    //linkedList.remove();//此处会抛出异常
    listIt.remove();//这样可以进行删除操作
}
```

**迭代器的另一种形式就是使用 foreach 循环，底层实现也是使用的迭代器，这里我们就不做介绍了。**

**普通for循环：每次遍历一个索引的元素之前，都要访问之间所有的索引。**

**迭代器：每次访问一个元素后，都会用游标记录当前访问元素的位置，遍历一个元素，记录一个位置。**



#### 3.4 Vector向量

​      Vector底层是用数组实现的List，相关的方法都加了同步检查，因此“线程安全,效率低”。 比如，indexOf方法就增加了synchronized同步标记。

![å¾9-10 Vectorçåºå±æºç .png](.\img\1495617071989433.png)

**老鸟建议**

   如何选用ArrayList、LinkedList、Vector?

1. 需要线程安全时，用Vector。

2. 不存在线程安全问题时，并且查找较多用ArrayList(一般使用它)。

3. 不存在线程安全问题时，增加或删除元素较多用LinkedList。



### 4. Map接口

​      Map就是用来存储“键(key)-值(value) 对”的。 Map类中存储的“键值对”通过键来标识，所以“键对象”不能重复。

​      Map 接口的实现类有HashMap、TreeMap、HashTable、Properties等。

![è¡¨9-3 Mapæ¥å£ä¸­å¸¸ç¨çæ¹æ³.png](.\img\1495617463792119.png)

**哈希表**

　　Hash表也称为散列表，也有直接译作哈希表，Hash表是一种根据关键字值（key - value）而直接进行访问的数据结构。也就是说它通过把关键码值映射到表中的一个位置来访问记录，以此来加快查找的速度。在链表、数组等数据结构中，查找某个关键字，通常要遍历整个数据结构，也就是O(N)的时间级，但是对于哈希表来说，只是O(1)的时间级。

　　比如对于前面我们讲解的 [ArrayList](http://www.cnblogs.com/ysocean/p/8622264.html) 集合和 [LinkedList](http://www.cnblogs.com/ysocean/p/8657850.html) ，如果我们要查找这两个集合中的某个元素，通常是通过遍历整个集合，需要**O(N)**的时间级。

![img](.\img\1120165-20180403234003772-1775417648.png)

　　如果是哈希表，它是通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做**散列函数**，存放记录的数组叫做**散列表，**只需要**O(1)**的时间级。

![img](.\img\1120165-20180403235818954-222956412.png)

```tex
大家都用过汉语字典吧，汉语字典的优点是我们可以通过前面的拼音目录快速定位到所要查找的汉字。当给定我们某个汉字时，大脑会自动将汉字转换成拼音（如果我们认识，不认识可以通过偏旁部首），这个转换的过程我们可以看成是一个散列函数，之后在根据转换得到的拼音找到该字所在的页码，从而找到该汉字。
```

 　　汉语字典是哈希表的典型实现，但是我们仔细思考，会发现这样几个问题？

　　**①、为什么要有散列函数？**

　　**②、多个 key 通过散列函数会得到相同的值，这时候怎么办？**

　　对于第一个问题，**散列函数的存在能够帮助我们更快的确定key和value的映射关系**，试想一下，如果没有汉字和拼音的转换规则（或者汉字和偏旁部首的），给你一个汉字，你该如何从字典中找到该汉字？我想除了遍历整部字典，你没有什么更好的办法。

　　对于第二个问题，多个 key 通过散列函数得到相同的值，这其实也是哈希表最大的问题——**冲突**。比如同音字汉字，我们得到的拼音就会是相同的，那么我们该如何在字典中存放同音字汉字呢？有两种做法：

　　第一种是**开放地址法**，当我们遇到冲突了，这时候通过另一种函数再计算一遍，得到相应的映射关系。比如对于汉语字典，一个字 “余”，拼音是“yu”，我们将其放在页码为567(假设在该位置)，这时候又来了一个汉字“于”，拼音也是“yu”，那么这时候我们要是按照转换规则，也得将其放在页码为567的位置，但是我们发现这个页码已经被占用了，这时候怎么办？我们可以在通过另一种函数，得到的值加1。那么汉字"于"就会被放在576+1=577的位置。

　　第二种是**链地址法**，我们可以将字典的每一页都看成是一个子数组或者子链表，当遇到冲突了，直接往当前页码的子数组或者子链表里面填充即可。那么我们进行同音字查找的时候，可能需要遍历其子数组或者子链表。

![img](.\img\1120165-20180404082228573-2042730199.png)





#### 4.1 HashMap和HashTable

​      HashMap采用哈希算法实现，是Map接口最常用的实现类。 由于底层采用了哈希表存储数据，我们要求键不能重复，如果发生重复，新的键值对会替换旧的键值对。 HashMap在查找、删除、修改方面都有非常高的效率。

**Map接口中的常用方法**

```Java
public class TestMap {
    public static void main(String[] args) {
        Map<Integer, String> m1 = new HashMap<Integer, String>();
        Map<Integer, String> m2 = new HashMap<Integer, String>();
        m1.put(1, "one");
        m1.put(2, "two");
        m1.put(3, "three");
        m2.put(1, "一");
        m2.put(2, "二");
        System.out.println(m1.size());
        System.out.println(m1.containsKey(1));
        System.out.println(m2.containsValue("two"));
        m1.put(3, "third"); //键重复了，则会替换旧的键值对
        Map<Integer, String> m3 = new HashMap<Integer, String>();
        m3.putAll(m1);
        m3.putAll(m2);
        System.out.println("m1:" + m1);
        System.out.println("m2:" + m2);
        System.out.println("m3:" + m3);
    }
}
```

![å¾9-11 ç¤ºä¾9-7è¿è¡ææå¾.png](.\img\1495617739123587.png)

​     **HashTable类和HashMap用法几乎一样，底层实现几乎一样，只不过HashTable的方法添加了synchronized关键字确保线程同步检查，效率较低。**

**HashMap与HashTable的区别**

1. HashMap: 线程不安全，效率高。允许key或value为null。

2. HashTable: 线程安全，效率低。不允许key或value为null。



#### 4.2 HashMap底层实现详解

​      数据结构中由数组和链表来实现对数据的存储，他们各有特点。

​      (1) 数组：占用空间连续。 寻址容易，查询速度快。但是，增加和删除效率非常低。

​      (2) 链表：占用空间不连续。 寻址困难，查询速度慢。但是，增加和删除效率非常高。

​      **哈希表的本质就是“数组+单链表”。**

**▪ Hashmap基本结构讲解**

​      在JDK1.8之前，HashMap采用数组+链表实现，即使用链表处理冲突，**同一hash值的节点都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，HashMap采用数组+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树**，这样大大减少了查找时间。

​      下图中代表jdk1.8之前的hashmap结构，左边部分即代表哈希表，也称为哈希数组，数组的每个元素都是一个单链表的头节点，链表是用来解决冲突的，如果不同的key映射到了数组的同一位置处，就将其放入单链表中。

​	其中的Entry[] table 就是HashMap的核心数组结构，我们也称之为“位桶数组”。

![å¾9-13 HashMapåºå±æºç (.\img\1495619012687826.png).png](https://www.sxt.cn/360shop/Public/admin/UEditor/20170524/1495619012687826.png)

一个Entry对象存储了：

1. **key：键对象** 
2. **value：值对象**

2. **next:下一个节点**

3. **hash: 键对象的hash值**

​      显然每一个Entry对象就是一个单向链表结构

![å¾9-14 Entryå¯¹è±¡å­å¨ç"æå¾.png](.\img\1495619082593896.png)

![å¾9-15 Entryæ°ç"å­å¨ç"æå¾.png](.\img\1495619119905721.png)

​    jdk1.8之前的hashmap都采用上图的结构，都是基于一个数组和多个单链表，hash值冲突的时候，就将对应节点以链表的形式存储。如果在一个链表中查找其中一个节点时，将会花费O（n）的查找时间，会有很大的性能损失。到了jdk1.8，当同一个hash值的节点数不小于8时，不再采用单链表形式存储，而是采用红黑树

![img](.\img\1120165-20180404211600945-1711602320.png)



**▪ HashMap存储数据过程示意图**



![å¾9-16 HashMapå­å¨æ°æ®è¿ç¨ç¤ºæå¾.png](.\img\1495619181777762.png)

 	hashcode是一个整数，我们需要将它转化成[0, 数组长度-1]的范围。我们要求转化后的hash值尽量均匀地分布在[0,数组长度-1]这个区间，减少“hash冲突”

​	一种简单和常用的算法是(相除取余算法)：

​	 **hash值 = hashcode % 数组长度**

​           这种算法可以让hash值均匀的分布在[0,数组长度-1]的区间。 早期的HashTable就是采用这种算法。但是，这种算法由于使用了“除法”，效率低下。JDK后来改进了算法。首先约定数组长度必须为2的整数幂，这样采用位运算即可实现取余的效果：hash值 = hashcode&(数组长度-1)。

```Java
public class Test {
    public static void main(String[] args) {
        int h = 25860399;
        int length = 16;//length为2的整数次幂,则h&(length-1)相当于对length取模
        myHash(h, length);
    }
    /**
     * @param h  任意整数
     * @param length 长度必须为2的整数幂
     * @return
     */
    public static  int myHash(int h,int length){
        System.out.println(h&(length-1));
        //length为2的整数幂情况下，和取余的值一样
        System.out.println(h%length);//取余数
        return h&(length-1);
    }
}
```

​	**JDK对hashcode进行了两次散列处理(核心目标就是为了分布更散更均匀)**

![å¾9-17 hashç®æ³æºç .png](.\img\1495619395858263.png)

**总结如上过程：**

​      当添加一个元素(key-value)时，首先计算key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这时就添加到同一hash值的元素的后面，他们在数组的同一位置，就形成了链表，同一个链表上的Hash值是相同的，所以说数组存放的是链表。 JDK8中，当链表长度大于8时，链表就转换为红黑树，这样又大大提高了查找的效率。

**▪ 取数据过程get(key)**

​      我们需要通过key对象获得“键值对”对象，进而返回value对象。明白了存储数据过程，取数据就比较简单了，参见以下步骤：

​      (1) 获得key的hashcode，通过hash()散列算法得到hash值，进而定位到数组的位置。

​      (2) 在链表上挨个比较key对象。 调用equals()方法，将key对象和链表上所有节点的key对象进行比较，直到碰到返回true的节点对象为止。

​      (3) 返回equals()为true的节点对象的value对象。

​      明白了存取数据的过程，我们再来看一下hashcode()和equals方法的关系：

​      Java中规定，两个内容相同(equals()为true)的对象必须具有相等的hashCode。因为如果equals()为true而两个对象的hashcode不同;那在整个存储过程中就发生了悖论。

**▪ 扩容问题**

​      HashMap的位桶数组，初始大小为16。实际使用时，显然大小是可变的。如果位桶数组中的元素达到(0.75*数组 length)， 就重新调整数组大小变为原来2倍大小。

​      扩容很耗时。扩容的本质是定义新的更大的数组，并将旧数组内容挨个拷贝到新数组中。

**▪ JDK8将链表在大于8情况下变为红黑二叉树**

​      JDK8中，HashMap在存储一个元素时，当对应链表长度大于8时，链表就转换为红黑树，这样又大大提高了查找的效率。

##### HashMap字段属性

```Java
	//序列化和反序列化时，通过该字段进行版本一致性验证
    private static final long serialVersionUID = 362498820763181265L;
    //默认 HashMap 集合初始容量为16（必须是 2 的倍数）
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    //集合的最大容量，如果通过带参构造指定的最大容量超过此数，默认还是使用此数
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //当桶(bucket)上的结点数大于这个值时会转成红黑树(JDK1.8新增)
    static final int TREEIFY_THRESHOLD = 8;
    //当桶(bucket)上的节点数小于这个值时会转成链表(JDK1.8新增)
    static final int UNTREEIFY_THRESHOLD = 6;
    /**(JDK1.8新增)
     * 当集合中的容量大于这个值时，表中的桶才能进行树形化 ，否则桶内元素太多时会扩容，
     * 而不是树形化 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
	/**
     * 初始化使用，长度总是 2的幂
     */
    transient Node<K,V>[] table;

    /**
     * 保存缓存的entrySet（）
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * 此映射中包含的键值映射的数量。（集合存储键值对的数量）
     */
    transient int size;

    /**
     * 跟前面ArrayList和LinkedList集合中的字段modCount一样，记录集合被修改的次数
     * 主要用于迭代器中的快速失败
     */
    transient int modCount;

    /**
     * 调整大小的下一个大小值（容量*加载因子）。capacity * load factor
     */
    int threshold;

    /**
     * 散列表的加载因子。
     */
    final float loadFactor;
```

　　**后面三个字段是 JDK1.8 新增的，主要是用来进行红黑树和链表的互相转换。**

​	下面我们重点介绍上面几个字段：

　　**①、Node<K,V>[] table**

　　我们说 HashMap 是由数组+链表+红黑树组成，这里的数组就是 table 字段。后面对其进行初始化长度默认是 DEFAULT_INITIAL_CAPACITY= 16。而且 JDK 声明数组的长度总是 2的n次方(一定是合数)，为什么这里要求是合数，一般我们知道哈希算法为了避免冲突都要求长度是质数，这里要求是合数，下面在介绍 HashMap 的hashCode() 方法(散列函数)，我们再进行讲解。

　　②**、size**

　　集合中存放key-value 的实时对数。

　　**③、loadFactor**

　　装载因子，是用来衡量 HashMap 满的程度，计算HashMap的实时装载因子的方法为：size/capacity，而不是占用桶的数量去除以capacity。capacity 是桶的数量，也就是 table 的长度length。

　　默认的负载因子0.75 是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子loadFactor 的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子 loadFactor 的值，这个值可以大于1。

　　**④、threshold**

　　计算公式：capacity * loadFactor。这个值是当前已占用数组长度的最大值。超过这个数目就重新resize(扩容)，扩容后的 HashMap 容量是之前容量的两倍



##### 确定哈希桶数组索引位置

​	主要分为三步：

　　①、取 hashCode 值： key.hashCode()

　　②、高位参与运算：h>>>16

　　③、取模运算：(n-1) & hash

　　这里获取 hashCode() 方法的值是变量，但是我们知道，对于任意给定的对象，只要它的 hashCode() 返回值相同，那么程序调用 hash(Object key) 所计算得到的 hash码 值总是相同的。

　　为了让数组元素分布均匀，我们首先想到的是把获得的 hash码对数组长度取模运算( hash%length)，但是计算机都是二进制进行操作，取模运算相对开销还是很大的，那该如何优化呢？

　　HashMap 使用的方法很巧妙，它通过 hash & (table.length -1)来得到该对象的保存位，前面说过 HashMap 底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当 length 总是2的n次方时，hash & (length-1)运算等价于对 length 取模，也就是 hash%length，但是&比%具有更高的效率。比如 n % 32 = n & (32 -1)

　　**这也解释了为什么要保证数组的长度总是2的n次方。**

　　再就是在 JDK1.8 中还有个高位参与运算，hashCode() 得到的是一个32位 int 类型的值，通过hashCode()的高16位 **异或** 低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

![img](.\img\249993-20170725154200021-869603681.png)

##### 添加元素

```Java
public V put(K key, V value) {
    // 对key的hashCode()做hash 
    return putVal(hash(key), key, value, false, true);  
} 
```

![img](.\img\249993-20170725160254943-1515467235.png)

①.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；

③.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

⑤.遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

⑥.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤①：tab为空则创建 
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤②：计算index，并对null做处理  
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 步骤③：节点key存在，直接覆盖value 
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // 步骤④：判断该链为红黑树 
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 步骤⑤：该链为链表 
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 步骤⑥：超过最大容量 就扩容 
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

**HashMap的数据存储实现原理**

**流程：**

1.根据key计算得到key.hash = (h = k.hashCode()) ^ (h >>> 16)；

2.根据key.hash计算得到桶数组的索引index = key.hash & (table.length - 1)，这样就找到该key的存放位置了：

① 如果该位置没有数据，用该数据新生成一个节点保存新数据，返回null；

② 如果该位置有数据是一个红黑树，那么执行相应的插入 / 更新操作；

③ 如果该位置有数据是一个链表，分两种情况一是该链表没有这个节点，另一个是该链表上有这个节点，注意这里判断的依据是key.hash是否一样：

如果该链表没有这个节点，那么采用尾插法新增节点保存新数据，返回null；如果该链表已经有这个节点了，那么找到该节点并更新新数据，返回老数据。

注意：

HashMap的put会返回key的上一次保存的数据，比如：

HashMap<String, String> map = new HashMap<String, String>();
System.out.println(map.put("a", "A")); // 打印null
System.out.println(map.put("a", "AA")); // 打印A
System.out.println(map.put("a", "AB")); // 打印AA



##### 查找元素

```java
public V get(Object key) {
    Node<k,v> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

​	首先通过 key 找到计算索引，找到桶位置，先检查第一个节点，如果是则返回，如果不是，则遍历其后面的链表或者红黑树。其余情况全部返回 null。

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // table已经初始化，长度大于0，根据hash寻找table中的项也不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 桶中第一项(数组元素)相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个结点
        if ((e = first.next) != null) {
            // 为红黑树结点
            if (first instanceof TreeNode)
                // 在红黑树中查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 否则，在链表中查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

**判断是否存在给定的 key 或者 value**

```java
public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            //遍历桶
            for (int i = 0; i < tab.length; ++i) {
                //遍历桶中的每个节点元素
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```



##### 遍历元素

首先构造一个 HashMap 集合：

```Java
HashMap<String,Object> map = new HashMap<>();
map.put("A","1");
map.put("B","2");
map.put("C","3");
```

　　①、分别获取 key 集合和 value 集合。

```java
1 //1、分别获取key和value的集合
2 for(String key : map.keySet()){
3     System.out.println(key);
4 }
5 for(Object value : map.values()){
6     System.out.println(value);
7 }
```

　　②、获取 key 集合，然后遍历key集合，根据key分别得到相应value

```java
1 //2、获取key集合，然后遍历key，根据key得到 value
2 Set<String> keySet = map.keySet();
3 for(String str : keySet){
4     System.out.println(str+"-"+map.get(str));
5 }
```

　　③、得到 Entry 集合，然后遍历 Entry

```java
1 //3、得到 Entry 集合，然后遍历 Entry
2 Set<Map.Entry<String,Object>> entrySet = map.entrySet();
3 for(Map.Entry<String,Object> entry : entrySet){
4     System.out.println(entry.getKey()+"-"+entry.getValue());
5 }
```

　　④、迭代

```java
1 //4、迭代
2 Iterator<Map.Entry<String,Object>> iterator = map.entrySet().iterator();
3 while(iterator.hasNext()){
4     Map.Entry<String,Object> mapEntry = iterator.next();
5     System.out.println(mapEntry.getKey()+"-"+mapEntry.getValue());
6 }
```

　　基本上使用第三种方法是性能最好的，

　　第一种遍历方法在我们只需要 key 集合或者只需要 value 集合时使用；

　　第二种方法效率很低，不推荐使用；

　　第四种方法效率也挺好，关键是在遍历的过程中我们可以对集合中的元素进行删除。



#### 4.3 ThreeMap

​	TreeMap是红黑二叉树的典型实现。我们打开TreeMap的源码，发现里面有一行核心代码：

```Java
private transient Entry<K,V> root = null;
```

​      root用来存储整个树的根节点。我们继续跟踪Entry(是TreeMap的内部类)的代码：

![å¾9-23 Entryåºå±æºç .png](.\img\1495695046855639.png)

​      可以看到里面存储了本身数据、左节点、右节点、父节点、以及节点颜色。 TreeMap的put()/remove()方法大量使用了红黑树的理论。

​      TreeMap和HashMap实现了同样的接口Map，因此，用法对于调用者来说没有区别。HashMap效率高于TreeMap;在需要排序的Map时才选用TreeMap。



#### 4.4 LinkedHashMap

　　①、基于JDK1.8的HashMap是由数组+链表+红黑树组成，相对于早期版本的 JDK HashMap 实现，新增了红黑树作为底层数据结构，在数据量较大且哈希碰撞较多时，能够极大的增加检索的效率。

　　②、允许 key 和 value 都为 null。key 重复会被覆盖，value 允许重复。

　　③、非线程安全

　　④、无序（遍历HashMap得到元素的顺序不是按照插入的顺序）

　　HashMap 集合可以说是最重要的集合之一，LinkedHashMap其实也是继承 HashMap 集合来实现的，而且我们在介绍 HashMap 集合的 put 方法时，也指出了 put 方法中调用的部分方法在 HashMap 都是空实现，而在 LinkedHashMap 中进行了重写。



　　LinkedHashMap 是基于 HashMap 实现的一种集合，具有 HashMap 集合上面所说的所有特点，除了 HashMap 无序的特点，**LinkedHashMap 是有序的，因为 LinkedHashMap 在 HashMap 的基础上单独维护了一个具有所有数据的双向链表，该链表保证了元素迭代的顺序。**

　　所以我们可以直接这样说：**LinkedHashMap = HashMap + LinkedList。LinkedHashMap 就是在 HashMap 的基础上多维护了一个双向链表，用来保证元素迭代顺序。**

##### 字段属性

 　　①、Entry<K,V>

```Java
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

![img](.\img\1120165-20181119213656738-1642597597.png)

　　LinkedHashMap 中 Entry 相对于 HashMap 多出的 before 和 after 便是用来维护 LinkedHashMap  插入 Entry 的先后顺序的。

　②、其它属性

```Java
//用来指向双向链表的头节点
transient LinkedHashMap.Entry<K,V> head;
//用来指向双向链表的尾节点
transient LinkedHashMap.Entry<K,V> tail;
//用来指定LinkedHashMap的迭代顺序
//true 表示按照访问顺序，会把访问过的元素放在链表后面，放置顺序是访问的顺序
//false 表示按照插入顺序遍历
final boolean accessOrder;
```

　　注意：这里有五个属性别搞混淆的，对于 Node  next 属性，是用来维护整个集合中 Entry 的顺序。对于 Entry before，Entry after ，以及 Entry head，Entry tail，这四个属性都是用来维护保证集合顺序的链表，其中前两个before和after表示某个节点的上一个节点和下一个节点，这是一个双向链表。后两个属性 head 和 tail 分别表示这个链表的头节点和尾节点。

![img](.\img\1120165-20181120211140408-453616443.png)

　**去掉红色和蓝色的虚线指针，其实就是一个HashMap。**



### 5. Set接口

​      Set接口继承自Collection，Set接口中没有新增方法，方法和Collection保持完全一致。我们在前面通过List学习的方法，在Set中仍然适用。因此，学习Set的使用将没有任何难度。

​      **Set容器特点：无序、不可重复。**无序指Set中的元素没有索引，我们只能遍历查找;不可重复指不允许加入重复的元素。更确切地讲，新元素如果和Set中某个元素通过equals()方法对比为true，则不能加入;甚至，Set中也只能放入一个null元素，不能多个。

​      Set常用的实现类有：HashSet、TreeSet等，我们一般使用HashSet。

#### 5.1 HashSet底层实现

​      HashSet是采用哈希算法实现，**底层实际是用HashMap实现的**(HashSet本质就是一个简化版的HashMap)，因此，查询效率和增删效率都比较高。



##### 字段属性

![å¾9-25 HashSetåºå±æºç .png](.\img\1495697834473119.png)

```Java
1 //HashSet集合中的内容是通过 HashMap 数据结构来存储的
2 private transient HashMap<E,Object> map;
3 //向HashSet中添加数据，数据在上面的 map 结构是作为 key 存在的，而value统一都是 PRESENT
4 private static final Object PRESENT = new Object();
```

​	我们发现里面有个map属性，这就是HashSet的核心秘密。我们再看add()方法，发现增加一个元素说白了就是在map中增加一个键值对，键对象就是这个元素，值对象是名为PRESENT的Object对象。说白了，就是“往set中加入元素，本质就是把这个元素作为key加入到了内部的map中”。

​      由于map中key都是不可重复的，因此，Set天然具有“不可重复”的特性。

##### 构造函数

　　①、无参构造

```Java
1     public HashSet() {
2         map = new HashMap<>();
3     }
```

　　直接 new 一个 HashMap 对象出来，采用无参的 HashMap 构造函数，具有默认初始容量（16）和加载因子（0.75）。

　　②、指定初始容量

```Java
1     public HashSet(int initialCapacity) {
2         map = new HashMap<>(initialCapacity);
3     }
```

　　③、指定初始容量和加载因子

```Java
1     public HashSet(int initialCapacity, float loadFactor) {
2         map = new HashMap<>(initialCapacity, loadFactor);
3     }
```

　　④、构造包含指定集合中的元素

```Java
1     public HashSet(Collection<? extends E> c) {
2         map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
3         addAll(c);
4     }
```

　　集合容量很好理解，这里我介绍一下什么是加载因子。在 HashMap 中，能够存储元素的数量就是：总的容量*加载因子 ，新增一个元素时，如果HashMap集合中的元素大于前面公式计算的结果了，那么就必须要进行扩容操作，从时间和空间考虑，加载因子一般都选默认的0.75。

##### 遍历元素

```Java
HashSet<Integer> set = new HashSet<>();
set.add(1);
set.add(2);
//增强for循环
for(Integer i : set){
    System.out.println(i);
}
//普通for循环
Iterator<Integer> iterator = set.iterator();
while (iterator.hasNext()){
    System.out.println(iterator.next());
}
```



#### 5.2 TreeSet底层实现

​      TreeSet底层实际是用TreeMap实现的，内部维持了一个简化版的TreeMap，通过key来存储Set的元素。 TreeSet内部需要对存储的元素进行排序，因此，我们对应的类需要实现Comparable接口。这样，才能根据compareTo()方法比较对象之间的大小，才能进行内部排序。

**TreeSet和Comparable接口的使用**

```Java
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1001, "高淇", 18);
        User u2 = new User(2001, "高希希", 5);
        Set<User> set = new TreeSet<User>();
        set.add(u1);
        set.add(u2);
    }
}
 
class User implements Comparable<User> {
    int id;
    String uname;
    int age;
 
    public User(int id, String uname, int age) {
        this.id = id;
        this.uname = uname;
        this.age = age;
    }
    /**
     * 返回0 表示 this == obj 返回正数表示 this > obj 返回负数表示 this < obj
     */
    @Override
    public int compareTo(User o) {
        if (this.id > o.id) {
            return 1;
        } else if (this.id < o.id) {
            return -1;
        } else {
            return 0;
        }
    }
}
```

**使用TreeSet要点：**

​      (1) 由于是二叉树，需要对元素做内部排序。 如果要放入TreeSet中的类没有实现Comparable接口，则会抛出异常：java.lang.ClassCastException。

​      (2) TreeSet中不能放入null元素。



**Map与Set总结:**

​	**Map栽树, Set乘凉**, Set的所有子类(HashSet, TreeSet, LinkedHashSet)都是基于Map的子类实现的, 如HashSet基于HashMap, TreeSet基于TreeMap, LinkedHashSet基于LinkedHashMap



### 6. Iterator

​      **如果遇到遍历容器时，判断删除元素的情况，使用迭代器遍历!**

**迭代器遍历List**

```Java
public class Test {
    public static void main(String[] args) {
        List<String> aList = new ArrayList<String>();
        for (int i = 0; i < 5; i++) {
            aList.add("a" + i);
        }
        System.out.println(aList);
        for (Iterator<String> iter = aList.iterator(); iter.hasNext();) {
            String temp = iter.next();
            System.out.print(temp + "\t");
            if (temp.endsWith("3")) {// 删除3结尾的字符串
                iter.remove();
            }
        }
        System.out.println();
        System.out.println(aList);
    }
}
```

![å¾9-27ç¤ºä¾9-11è¿è¡ææå¾.png](.\img\1495698648334893.png)

**迭代器遍历Set**

```java
public class Test {
    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();
        for (int i = 0; i < 5; i++) {
            set.add("a" + i);
        }
        System.out.println(set);
        for (Iterator<String> iter = set.iterator(); iter.hasNext();) {
            String temp = iter.next();
            System.out.print(temp + "\t");
        }
        System.out.println();
        System.out.println(set);
    }
}
```

![å¾9-28ç¤ºä¾9-12è¿è¡ææå¾.png](.\img\1495698749486697.png)

**迭代器遍历Map**

```Java
public class Test {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("A", "高淇");
        map.put("B", "高小七");
        Set<Entry<String, String>> ss = map.entrySet();
        for (Iterator<Entry<String, String>> iterator = ss.iterator(); iterator.hasNext();) {
            Entry<String, String> e = iterator.next();
            System.out.println(e.getKey() + "--" + e.getValue());
        }
    }
}
```

![å¾9-29ç¤ºä¾9-13è¿è¡ææå¾.png](.\img\1495698803361096.png)



### 7. Collections工具类

类 java.util.Collections 提供了对Set、List、Map进行排序、填充、查找元素的辅助方法。

1. void sort(List) //对List容器内的元素排序，排序的规则是按照升序进行排序。

2. void shuffle(List) //对List容器内的元素进行随机排列。

3. void reverse(List) //对List容器内的元素进行逆续排列 。

4. void fill(List, Object) //用一个特定的对象重写整个List容器。

5. int binarySearch(List, Object)//对于顺序的List容器，采用折半查找的方法查找特定对象。

**Collections工具类的常用方法**

```Java
public class Test {
    public static void main(String[] args) {
        List<String> aList = new ArrayList<String>();
        for (int i = 0; i < 5; i++){
            aList.add("a" + i);
        }
        System.out.println(aList);
        Collections.shuffle(aList); // 随机排列
        System.out.println(aList);
        Collections.reverse(aList); // 逆续
        System.out.println(aList);
        Collections.sort(aList); // 按照递增的方式排序, 自定义的类使用: Comparable接口。
        System.out.println(aList);
        System.out.println(Collections.binarySearch(aList, "a2")); 
        Collections.fill(aList, "hello");
        System.out.println(aList);
    }
}
```

![å¾9-31ç¤ºä¾9-23è¿è¡ææå¾.png](.\img\1495699582304352.png)



###8. 容器总结

1. Collection 表示一组对象，它是集中、收集的意思，就是把一些数据收集起来。

2. Collection接口的两个子接口：

​      1) List中的元素有顺序，可重复。常用的实现类有ArrayList、LinkedList和 vector。

​        Ø ArrayList特点：查询效率高，增删效率低，线程不安全。

​        Ø LinkedList特点：查询效率低，增删效率高，线程不安全。

​        Ø vector特点：线程安全,效率低,其它特征类似于ArrayList。

​      2) Set中的元素没有顺序，不可重复。常用的实现类有HashSet和TreeSet。

​        Ø HashSet特点：采用哈希算法实现,查询效率和增删效率都比较高。

​        Ø TreeSet特点：内部需要对存储的元素进行排序。因此，我们对应的类需要实现Comparable接口。这样，才能根据compareTo()方法比较对象之间的大小，才能进行内部排序。

3. 实现Map接口的类用来存储键(key)-值(value) 对。Map 接口的实现类有HashMap和TreeMap等。Map类中存储的键-值对通过键来标识，所以键值不能重复。

4. Iterator对象称作迭代器，用以方便的实现对容器内元素的遍历操作。

5. 类 java.util.Collections 提供了对Set、List、Map操作的工具方法。

6. 如下情况，可能需要我们重写equals/hashCode方法：

​      1) 要将我们自定义的对象放入HashSet中处理。

​      2) 要将我们自定义的对象作为HashMap的key处理。

​      3) 放入Collection容器中的自定义对象后，可能会调用remove、contains等方法时。

7. JDK1.5以后增加了泛型。泛型的好处：

​      1) 向集合添加数据时保证数据安全。

​      2) 遍历集合元素时不需要强制转换。



## 五, IO流

### 1. 概述

#### 1.1 流的概念

​      流是一个抽象、动态的概念，是一连串连续动态的数据集合。

​      对于输入流而言，数据源就像水箱，流(stream)就像水管中流动着的水流，程序就是我们最终的用户。我们通过流(A Stream)将数据源(Source)中的数据(information)输送到程序(Program)中。

​      对于输出流而言，目标数据源就是目的地(dest)，我们通过流(A Stream)将程序(Program)中的数据(information)输送到目的数据源(dest)中。

![å¾10-2 æµä¸æºæ°æ®æºåç®æ æ°æ®æºä¹é´çå³ç³".png](.\img\1495701094668697.png)

 **输入/输出流的划分是相对程序而言的，并不是相对数据源。**

**IO流体系中大量使用了装饰器模式，让流具有更强的功能、更强的灵活性。**

#### 1.2 IO的分类

根据数据的流向分为：**输入流**和**输出流**。

- **输入流** ：把数据从`其他设备`上读取到`内存`中的流。 
- **输出流** ：把数据从`内存` 中写出到`其他设备`上的流。

格局数据的类型分为：**字节流**和**字符流**。

- **字节流** ：以字节为单位，读写数据的流。

- **字符流** ：以字符为单位，读写数据的流。

- 一个字符是两个字节, 一个字节是8个二进制位

  ![1554816070311](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1554816070311.png)

![1554731027950](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1554731027950.png)

#### 1.3 顶级父类

|            |         **输入流**         |           输出流            |
| :--------: | :------------------------: | :-------------------------: |
| **字节流** | 字节输入流 **InputStream** | 字节输出流 **OutputStream** |
| **字符流** |   字符输入流 **Reader**    |    字符输出流 **Writer**    |



### 2. 字节流

#### 2.1 字节输出流【OutputStream】

`java.io.OutputStream `抽象类是表示字节输出流的所有类的超类，将指定的字节信息写出到目的地。它定义了字节输出流的基本共性功能方法。

- `public void close()` ：关闭此输出流并释放与此流相关联的任何系统资源。  
- `public void flush() ` ：刷新此输出流并强制任何缓冲的输出字节被写出。  
- `public void write(byte[] b)`：将 b.length字节从指定的字节数组写入此输出流。  
- `public void write(byte[] b, int off, int len)` ：从指定的字节数组写入 len字节，从偏移量 off开始输出到此输出流。  
- `public abstract void write(int b)` ：将指定的字节输出流。

> 小贴士：
>
> close方法，当完成流的操作时，必须调用此方法，释放系统资源。

#### 2.2 FileOutputStream类

`OutputStream`有很多子类，我们从最简单的一个子类开始。

`java.io.FileOutputStream `类是文件输出流，用于将数据写出到文件。

##### 构造方法

- `public FileOutputStream(File file)`：创建文件输出流以写入由指定的 File对象表示的文件。 
- `public FileOutputStream(String name)`： 创建文件输出流以指定的名称写入文件。  

当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有这个文件，会创建该文件。如果有这个文件，会清空这个文件的数据。

- 构造举例，代码如下：

```java
public class FileOutputStreamConstructor throws IOException {
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileOutputStream fos = new FileOutputStream(file);
      
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("b.txt");
    }
}
```

##### 写出字节数据

1. **写出字节**：`write(int b)` 方法，每次可以写出一个字节数据，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");     
      	// 写出数据
      	fos.write(97); // 写出第1个字节
      	fos.write(98); // 写出第2个字节
      	fos.write(99); // 写出第3个字节
      	// 关闭资源
        fos.close();
    }
}
输出结果：
abc
```

> 小贴士：
>
> 1. 虽然参数为int类型四个字节，但是只会保留一个字节的信息写出。
> 2. 流操作完毕后，必须释放系统资源，调用close方法，千万记得。

2. **写出字节数组**：`write(byte[] b)`，每次可以写出数组中的数据，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");     
      	// 字符串转换为字节数组
      	byte[] b = "黑马程序员".getBytes();
      	// 写出字节数组数据
      	fos.write(b);
      	// 关闭资源
        fos.close();
    }
}
输出结果：
黑马程序员
```

3. **写出指定长度字节数组**：`write(byte[] b, int off, int len)` ,每次写出从off索引开始，len个字节，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");     
      	// 字符串转换为字节数组
      	byte[] b = "abcde".getBytes();
		// 写出从索引2开始，2个字节。索引2是c，两个字节，也就是cd。
        fos.write(b,2,2);
      	// 关闭资源
        fos.close();
    }
}
输出结果：
cd
```

数据追加续写

经过以上的演示，每次程序运行，创建输出流对象，都会清空目标文件中的数据。如何保留目标文件中数据，还能继续添加新数据呢？

- `public FileOutputStream(File file, boolean append)`： 创建文件输出流以写入由指定的 File对象表示的文件。  
- `public FileOutputStream(String name, boolean append)`： 创建文件输出流以指定的名称写入文件。  

这两个构造方法，参数中都需要传入一个boolean类型的值，`true` 表示追加数据，`false` 表示清空原有数据。这样创建的输出流对象，就可以指定是否追加续写了，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt"，true);     
      	// 字符串转换为字节数组
      	byte[] b = "abcde".getBytes();
		// 写出从索引2开始，2个字节。索引2是c，两个字节，也就是cd。
        fos.write(b);
      	// 关闭资源
        fos.close();
    }
}
文件操作前：cd
文件操作后：cdabcde
```

##### 写出换行

Windows系统里，换行符号是`\r\n` 。把

以指定是否追加续写了，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");  
      	// 定义字节数组
      	byte[] words = {97,98,99,100,101};
      	// 遍历数组
        for (int i = 0; i < words.length; i++) {
          	// 写出一个字节
            fos.write(words[i]);
          	// 写出一个换行, 换行符号转成数组写出
            fos.write("\r\n".getBytes());
        }
      	// 关闭资源
        fos.close();
    }
}

输出结果：
a
b
c
d
e
```

> - 回车符`\r`和换行符`\n` ：
>   - 回车符：回到一行的开头（return）。
>   - 换行符：下一行（newline）。
> - 系统中的换行：
>   - Windows系统里，每行结尾是 `回车+换行` ，即`\r\n`；
>   - Unix系统里，每行结尾只有 `换行` ，即`\n`；
>   - Mac系统里，每行结尾是 `回车` ，即`\r`。从 Mac OS X开始与Linux统一。

#### 2.3 字节输入流【InputStream】

`java.io.InputStream `抽象类是表示字节输入流的所有类的超类，可以读取字节信息到内存中。它定义了字节输入流的基本共性功能方法。

- `public void close()` ：关闭此输入流并释放与此流相关联的任何系统资源。    
- `public abstract int read()`： 从输入流读取数据的下一个字节。 
- `public int read(byte[] b)`： 从输入流中读取一些字节数，并将它们存储到字节数组 b中 。

> 小贴士：
>
> close方法，当完成流的操作时，必须调用此方法，释放系统资源。

#### 2.4 FileInputStream流

`java.io.FileInputStream `类是文件输入流，从文件中读取字节。

构造方法

- `FileInputStream(File file)`： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的 File对象 file命名。 
- `FileInputStream(String name)`： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名 name命名。  

当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有该文件,会抛出`FileNotFoundException` 。

- 构造举例，代码如下：

```java
public class FileInputStreamConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileInputStream fos = new FileInputStream(file);
      
        // 使用文件名称创建流对象
        FileInputStream fos = new FileInputStream("b.txt");
    }
}
```

##### 读取字节数据

1. **读取字节**：`read`方法，每次可以读取一个字节的数据，提升为int类型，读取到文件末尾，返回`-1`，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象
       	FileInputStream fis = new FileInputStream("read.txt");
      	// 读取数据，返回一个字节
        int read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
        read = fis.read();
        System.out.println((char) read);
      	// 读取到末尾,返回-1
       	read = fis.read();
        System.out.println( read);
		// 关闭资源
        fis.close();
    }
}
输出结果：
a
b
c
d
e
-1
```

循环改进读取方式，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象
       	FileInputStream fis = new FileInputStream("read.txt");
      	// 定义变量，保存数据
        int b ；
        // 循环读取
        while ((b = fis.read())!=-1) {
            System.out.println((char)b);
        }
		// 关闭资源
        fis.close();
    }
}
输出结果：
a
b
c
d
e
```

> 小贴士：
>
> 1. 虽然读取了一个字节，但是会自动提升为int类型。
> 2. 流操作完毕后，必须释放系统资源，调用close方法，千万记得。

2. **使用字节数组读取**：`read(byte[] b)`，每次读取b的长度个字节到数组中，返回读取到的有效字节个数，读取到末尾时，返回`-1` ，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象.
       	FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
      	// 定义变量，作为有效个数
        int len ；
        // 定义字节数组，作为装字节数据的容器   
        byte[] b = new byte[2];
        // 循环读取
        while (( len= fis.read(b))!=-1) {
           	// 每次读取后,把数组变成字符串打印
            System.out.println(new String(b));
        }
		// 关闭资源
        fis.close();
    }
}

输出结果：
ab
cd
ed
```

错误数据`d`，是由于最后一次读取时，只读取一个字节`e`，数组中，上次读取的数据没有被完全替换，所以要通过`len` ，获取有效的字节，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象.
       	FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
      	// 定义变量，作为有效个数
        int len ；
        // 定义字节数组，作为装字节数据的容器   
        byte[] b = new byte[2];
        // 循环读取
        while (( len= fis.read(b))!=-1) {
           	// 每次读取后,把数组的有效字节部分，变成字符串打印
            System.out.println(new String(b，0，len));//  len 每次读取的有效字节个数
        }
		// 关闭资源
        fis.close();
    }
}

输出结果：
ab
cd
e
```

> 小贴士：
>
> 使用数组读取，每次读取多个字节，减少了系统间的IO操作次数，从而提高了读写的效率，建议开发中使用。



### 3. 字符流

当使用字节流读取文本文件时，可能会有一个小问题。就是遇到中文字符时，可能不会显示完整的字符，那是因为一个中文字符可能占用多个字节存储。所以Java提供一些字符流类，以字符为单位读写数据，专门用于处理文本文件。

#### 3.1 字符输入流【Reader】

 `java.io.Reader`抽象类是表示用于读取字符流的所有类的超类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法。

- `public void close()` ：关闭此流并释放与此流相关联的任何系统资源。    
- `public int read()`： 从输入流读取一个字符。 
- `public int read(char[] cbuf)`： 从输入流中读取一些字符，并将它们存储到字符数组 cbuf中 。

#### 3.2  FileReader类  

`java.io.FileReader `类是读取字符文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区。

> 小贴士：
>
> 1. 字符编码：字节与字符的对应规则。Windows系统的中文编码默认是GBK编码表。
>
> idea中UTF-8
>
> 2. 字节缓冲区：一个字节数组，用来临时存储字节数据。

##### 构造方法

- `FileReader(File file)`： 创建一个新的 FileReader ，给定要读取的File对象。   
- `FileReader(String fileName)`： 创建一个新的 FileReader ，给定要读取的文件的名称。  

当你创建一个流对象时，必须传入一个文件路径。类似于FileInputStream 。

- 构造举例，代码如下：

```java
public class FileReaderConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileReader fr = new FileReader(file);
      
        // 使用文件名称创建流对象
        FileReader fr = new FileReader("b.txt");
    }
}
```

##### 读取字符数据

1. **读取字符**：`read`方法，每次可以读取一个字符的数据，提升为int类型，读取到文件末尾，返回`-1`，循环读取，代码使用演示：

```java
public class FRRead {
    public static void main(String[] args) throws IOException {
      	// 使用文件名称创建流对象
       	FileReader fr = new FileReader("read.txt");
      	// 定义变量，保存数据
        int b ；
        // 循环读取
        while ((b = fr.read())!=-1) {
            System.out.println((char)b);
        }
		// 关闭资源
        fr.close();
    }
}
输出结果：
黑
马
程
序
员
```

> 小贴士：虽然读取了一个字符，但是会自动提升为int类型。

2. **使用字符数组读取**：`read(char[] cbuf)`，每次读取b的长度个字符到数组中，返回读取到的有效字符个数，读取到末尾时，返回`-1` ，代码使用演示：

```java
public class FRRead {
    public static void main(String[] args) throws IOException {
      	// 使用文件名称创建流对象
       	FileReader fr = new FileReader("read.txt");
      	// 定义变量，保存有效字符个数
        int len ；
        // 定义字符数组，作为装字符数据的容器
         char[] cbuf = new char[2];
        // 循环读取
        while ((len = fr.read(cbuf))!=-1) {
            System.out.println(new String(cbuf));
        }
		// 关闭资源
        fr.close();
    }
}
输出结果：
黑马
程序
员序
```

获取有效的字符改进，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException {
      	// 使用文件名称创建流对象
       	FileReader fr = new FileReader("read.txt");
      	// 定义变量，保存有效字符个数
        int len ；
        // 定义字符数组，作为装字符数据的容器
        char[] cbuf = new char[2];
        // 循环读取
        while ((len = fr.read(cbuf))!=-1) {
            System.out.println(new String(cbuf,0,len));
        }
    	// 关闭资源
        fr.close();
    }
}

输出结果：
黑马
程序
员
```

#### 3.3 字符输出流【Writer】

`java.io.Writer `抽象类是表示用于写出字符流的所有类的超类，将指定的字符信息写出到目的地。它定义了字节输出流的基本共性功能方法。

- `void write(int c)` 写入单个字符。
- `void write(char[] cbuf) `写入字符数组。 
- `abstract  void write(char[] cbuf, int off, int len) `写入字符数组的某一部分,off数组的开始索引,len写的字符个数。 
- `void write(String str) `写入字符串。 
- `void write(String str, int off, int len)` 写入字符串的某一部分,off字符串的开始索引,len写的字符个数。
- `void flush() `刷新该流的缓冲。  
- `void close()` 关闭此流，但要先刷新它。 

#### 3.4 FileWriter类

`java.io.FileWriter `类是写出字符到文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区。

##### 构造方法

- `FileWriter(File file)`： 创建一个新的 FileWriter，给定要读取的File对象。   
- `FileWriter(String fileName)`： 创建一个新的 FileWriter，给定要读取的文件的名称。  

当你创建一个流对象时，必须传入一个文件路径，类似于FileOutputStream。

- 构造举例，代码如下：

```java
public class FileWriterConstructor {
    public static void main(String[] args) throws IOException {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileWriter fw = new FileWriter(file);
      
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("b.txt");
    }
}
```

##### 基本写出数据

**写出字符**：`write(int b)` 方法，每次可以写出一个字符数据，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	// 写出数据
      	fw.write(97); // 写出第1个字符
      	fw.write('b'); // 写出第2个字符
      	fw.write('C'); // 写出第3个字符
      	fw.write(30000); // 写出第4个字符，中文编码表中30000对应一个汉字。
      
      	/*
        【注意】关闭资源时,与FileOutputStream不同。
      	 如果不关闭,数据只是保存到缓冲区，并未保存到文件。
        */
        // fw.close();
    }
}
输出结果：
abC田
```

> 小贴士：
>
> 1. 虽然参数为int类型四个字节，但是只会保留一个字符的信息写出。
> 2. 未调用close方法，数据只是保存到了缓冲区，并未写出到文件中。

##### 关闭和刷新

因为内置缓冲区的原因，如果不关闭输出流，无法写出字符到文件中。但是关闭的流对象，是无法继续写出数据的。如果我们既想写出数据，又想继续使用流，就需要`flush` 方法了。

- `flush` ：刷新缓冲区，流对象可以继续使用。
- `close `:先刷新缓冲区，然后通知系统释放资源。流对象不可以再被使用了。

代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");
        // 写出数据，通过flush
        fw.write('刷'); // 写出第1个字符
        fw.flush();
        fw.write('新'); // 继续写出第2个字符，写出成功
        fw.flush();
      
      	// 写出数据，通过close
        fw.write('关'); // 写出第1个字符
        fw.close();
        fw.write('闭'); // 继续写出第2个字符,【报错】java.io.IOException: Stream closed
        fw.close();
    }
}
```

> 小贴士：即便是flush方法写出了数据，操作的最后还是要调用close方法，释放系统资源。

##### 写出其他数据

1. **写出字符数组** ：`write(char[] cbuf)` 和 `write(char[] cbuf, int off, int len)` ，每次可以写出字符数组中的数据，用法类似FileOutputStream，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	// 字符串转换为字节数组
      	char[] chars = "黑马程序员".toCharArray();
      
      	// 写出字符数组
      	fw.write(chars); // 黑马程序员
        
		// 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
        fw.write(b,2,2); // 程序
      
      	// 关闭资源
        fos.close();
    }
}
```

2. **写出字符串**：`write(String str)` 和 `write(String str, int off, int len)` ，每次可以写出字符串中的数据，更为方便，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	// 字符串
      	String msg = "黑马程序员";
      
      	// 写出字符数组
      	fw.write(msg); //黑马程序员
      
		// 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
        fw.write(msg,2,2);	// 程序
      	
        // 关闭资源
        fos.close();
    }
}
```

3. **续写和换行**：操作类似于FileOutputStream。

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象，可以续写数据
        FileWriter fw = new FileWriter("fw.txt"，true);     
      	// 写出字符串
        fw.write("黑马");
      	// 写出换行
      	fw.write("\r\n");
      	// 写出字符串
  		fw.write("程序员");
      	// 关闭资源
        fw.close();
    }
}
输出结果:
黑马
程序员
```

> 小贴士：字符流，只能操作文本文件，不能操作图片，视频等非文本文件。
>
> 当我们单纯读或者写文本文件时  使用字符流 其他情况使用字节流

### 4, IO异常的处理

#### JDK7前处理

之前的入门练习，我们一直把异常抛出，而实际开发中并不能这样处理，建议使用`try...catch...finally` 代码块，处理异常部分，代码使用演示：

```java  
public class HandleException1 {
    public static void main(String[] args) {
      	// 声明变量
        FileWriter fw = null;
        try {
            //创建流对象
            fw = new FileWriter("fw.txt");
            // 写出数据
            fw.write("黑马程序员"); //黑马程序员
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fw != null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### JDK7的处理(扩展知识点了解内容)

还可以使用JDK7优化后的`try-with-resource` 语句，该语句确保了每个资源在语句结束时关闭。**所谓的资源（resource）是指在程序完成后，必须关闭的对象。**

格式：

```java
try (创建流对象语句，如果多个,使用';'隔开) {
	// 读写数据
} catch (IOException e) {
	e.printStackTrace();
}
```

代码使用演示：

```java
public class HandleException2 {
    public static void main(String[] args) {
      	// 创建流对象
        try ( FileWriter fw = new FileWriter("fw.txt"); ) {
            // 写出数据
            fw.write("黑马程序员"); //黑马程序员
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### JDK9的改进(扩展知识点了解内容)

JDK9中`try-with-resource` 的改进，对于**引入对象**的方式，支持的更加简洁。被引入的对象，同样可以自动关闭，无需手动close，我们来了解一下格式。

改进前格式：

```java
// 被final修饰的对象
final Resource resource1 = new Resource("resource1");
// 普通对象
Resource resource2 = new Resource("resource2");
// 引入方式：创建新的变量保存
try (Resource r1 = resource1;
     Resource r2 = resource2) {
     // 使用对象
}
```

改进后格式：

```java
// 被final修饰的对象
final Resource resource1 = new Resource("resource1");
// 普通对象
Resource resource2 = new Resource("resource2");

// 引入方式：直接引入
try (resource1; resource2) {
     // 使用对象
}
```

改进后，代码使用演示：

```java
public class TryDemo {
    public static void main(String[] args) throws IOException {
       	// 创建流对象
        final  FileReader fr  = new FileReader("in.txt");
        FileWriter fw = new FileWriter("out.txt");
       	// 引入到try中
        try (fr; fw) {
          	// 定义变量
            int b;
          	// 读取数据
          	while ((b = fr.read())!=-1) {
            	// 写出数据
            	fw.write(b);
          	}
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



### 5. 属性集

#### 5.1 概述

`java.util.Properties ` 继承于` Hashtable` ，来表示一个持久的属性集。它使用键值结构存储数据，每个键及其对应值都是一个字符串。该类也被许多Java类使用，比如获取系统属性时，`System.getProperties` 方法就是返回一个`Properties`对象。

#### 5.2 Properties类

##### 构造方法

- `public Properties()` :创建一个空的属性列表。

##### 基本的存储方法

- `public Object setProperty(String key, String value)` ： 保存一对属性。  
- `public String getProperty(String key) ` ：使用此属性列表中指定的键搜索属性值。
- `public Set<String> stringPropertyNames() ` ：所有键的名称的集合。

```java
public class ProDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 创建属性集对象
        Properties properties = new Properties();
        // 添加键值对元素
        properties.setProperty("filename", "a.txt");
        properties.setProperty("length", "209385038");
        properties.setProperty("location", "D:\\a.txt");
        // 打印属性集对象
        System.out.println(properties);
        // 通过键,获取属性值
        System.out.println(properties.getProperty("filename"));
        System.out.println(properties.getProperty("length"));
        System.out.println(properties.getProperty("location"));

        // 遍历属性集,获取所有键的集合
        Set<String> strings = properties.stringPropertyNames();
        // 打印键值对
        for (String key : strings ) {
          	System.out.println(key+" -- "+properties.getProperty(key));
        }
    }
}
输出结果：
{filename=a.txt, length=209385038, location=D:\a.txt}
a.txt
209385038
D:\a.txt
filename -- a.txt
length -- 209385038
location -- D:\a.txt
```

##### 与流相关的方法

- `public void load(InputStream inStream)`： 从字节输入流中读取键值对。 

参数中使用了字节输入流，通过流对象，可以关联到某文件上，这样就能够加载文本中的数据了。文本数据格式:

```
filename=a.txt
length=209385038
location=D:\a.txt
```

加载代码演示：

```java
public class ProDemo2 {
    public static void main(String[] args) throws FileNotFoundException {
        // 创建属性集对象
        Properties pro = new Properties();
        // 加载文本中信息到属性集
        pro.load(new FileInputStream("read.txt"));
        // 遍历集合并打印
        Set<String> strings = pro.stringPropertyNames();
        for (String key : strings ) {
          	System.out.println(key+" -- "+pro.getProperty(key));
        }
     }
}
输出结果：
filename -- a.txt
length -- 209385038
location -- D:\a.txt
```

> 小贴士：文本中的数据，必须是键值对形式，可以使用空格、等号、冒号等符号分隔。



### 6. 缓冲流

缓冲流,也叫高效流，是对4个基本的`FileXxx` 流的增强，所以也是4个流，按照数据类型分类：

- **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream` 
- **字符缓冲流**：`BufferedReader`，`BufferedWriter`

缓冲流的基本原理，**是在创建流对象时，会创建一个内置的默认大小的缓冲区数组**，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

![1555220411740](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1555220411740.png)

#### 6.1 字节缓冲流

##### 构造方法

- `public BufferedInputStream(InputStream in)` ：创建一个 新的缓冲输入流。 
- `public BufferedOutputStream(OutputStream out)`： 创建一个新的缓冲输出流。

构造举例，代码如下：

```java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
```

##### 效率测试

查询API，缓冲流读写方法与基本的流是一致的，我们通过复制大文件（375MB），测试它的效率。

1. 基本流，代码如下：

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
      	long start = System.currentTimeMillis();
		// 创建流对象
        try (
        	FileInputStream fis = new FileInputStream("jdk9.exe");
        	FileOutputStream fos = new FileOutputStream("copy.exe")
        ){
        	// 读写数据
            int b;
            while ((b = fis.read()) != -1) {
                fos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("普通流复制时间:"+(end - start)+" 毫秒");
    }
}

十几分钟过去了...
```

2. 缓冲流，代码如下：

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
      	long start = System.currentTimeMillis();
		// 创建流对象
        try (
        	BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
	     BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
        ){
        // 读写数据
            int b;
            while ((b = bis.read()) != -1) {
                bos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流复制时间:"+(end - start)+" 毫秒");
    }
}

缓冲流复制时间:8016 毫秒
```

如何更快呢？

使用数组的方式，代码如下：

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
      	// 记录开始时间
        long start = System.currentTimeMillis();
		// 创建流对象
        try (
			BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
		 BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
        ){
          	// 读写数据
            int len;
            byte[] bytes = new byte[8*1024];
            while ((len = bis.read(bytes)) != -1) {
                bos.write(bytes, 0 , len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流使用数组复制时间:"+(end - start)+" 毫秒");
    }
}
缓冲流使用数组复制时间:666 毫秒
```

#### 6.2 字符缓冲流

##### 构造方法

- `public BufferedReader(Reader in)` ：创建一个 新的缓冲输入流。 
- `public BufferedWriter(Writer out)`： 创建一个新的缓冲输出流。

构造举例，代码如下：

```java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("br.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
```

##### 特有方法

字符缓冲流的基本方法与普通字符流调用方式一致，不再阐述，我们来看它们具备的特有方法。

- BufferedReader：`public String readLine()`: 读一行文字。 
- BufferedWriter：`public void newLine()`: 写一行行分隔符,由系统属性定义符号。 

`readLine`方法演示，代码如下：

```java
public class BufferedReaderDemo {
    public static void main(String[] args) throws IOException {
      	 // 创建流对象
        BufferedReader br = new BufferedReader(new FileReader("in.txt"));
		// 定义字符串,保存读取的一行文字
        String line  = null;
      	// 循环读取,读取到最后返回null
        while ((line = br.readLine())!=null) {
            System.out.print(line);
            System.out.println("------");
        }
		// 释放资源
        br.close();
    }
}
```

`newLine`方法演示，代码如下：

```java
public class BufferedWriterDemo throws IOException {
    public static void main(String[] args) throws IOException  {
      	// 创建流对象
		BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));
      	// 写出数据
        bw.write("黑马");
      	// 写出换行
        bw.newLine();
        bw.write("程序");
        bw.newLine();
        bw.write("员");
        bw.newLine();
		// 释放资源
        bw.close();
    }
}
输出效果:
黑马
程序
员
```

### 7. 转换流

#### 字符编码

计算机中储存的信息都是用二进制数表示的，而我们在屏幕上看到的数字、英文、标点符号、汉字等字符是二进制数转换之后的结果。按照某种规则，将字符存储到计算机中，称为**编码** 。反之，将存储在计算机中的二进制数按照某种规则解析显示出来，称为**解码** 。比如说，按照A规则存储，同样按照A规则解析，那么就能显示正确的文本符号。反之，按照A规则存储，再按照B规则解析，就会导致乱码现象。

编码:字符(能看懂的)--字节(看不懂的)

解码:字节(看不懂的)-->字符(能看懂的)

- **字符编码`Character Encoding`** : 就是一套自然语言的字符与二进制数之间的对应规则。

  编码表:生活中文字和计算机中二进制的对应规则

#### 字符集

- **字符集 `Charset`**：也叫编码表。是一个系统支持的所有字符的集合，包括各国家文字、标点符号、图形符号、数字等。

计算机要准确的存储和识别各种字符集符号，需要进行字符编码，一套字符集必然至少有一套字符编码。常见字符集有ASCII字符集、GBK字符集、Unicode字符集等。

![1555227973688](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1555227973688.png)

可见，当指定了**编码**，它所对应的**字符集**自然就指定了，所以**编码**才是我们最终要关心的。

- **ASCII字符集** ：
  - ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于拉丁字母的一套电脑编码系统，用于显示现代英语，主要包括控制字符（回车键、退格、换行键等）和可显示字符（英文大小写字符、阿拉伯数字和西文符号）。
  - 基本的ASCII字符集，使用7位（bits）表示一个字符，共128字符。ASCII的扩展字符集使用8位（bits）表示一个字符，共256字符，方便支持欧洲常用字符。
- **ISO-8859-1字符集**：
  - 拉丁码表，别名Latin-1，用于显示欧洲使用的语言，包括荷兰、丹麦、德语、意大利语、西班牙语等。
  - ISO-8859-1使用单字节编码，兼容ASCII编码。
- **GBxxx字符集**：
  - GB就是国标的意思，是为了显示中文而设计的一套字符集。
  - **GB2312**：简体中文码表。一个小于127的字符的意义与原来相同。但两个大于127的字符连在一起时，就表示一个汉字，这样大约可以组合了包含7000多个简体汉字，此外数学符号、罗马希腊的字母、日文的假名们都编进去了，连在ASCII里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的"全角"字符，而原来在127号以下的那些就叫"半角"字符了。
  - **GBK**：最常用的中文码表。是在GB2312标准基础上的扩展规范，使用了双字节编码方案，共收录了21003个汉字，完全兼容GB2312标准，同时支持繁体汉字以及日韩汉字等。
  - **GB18030**：最新的中文码表。收录汉字70244个，采用多字节编码，每个字可以由1个、2个或4个字节组成。支持中国国内少数民族的文字，同时支持繁体汉字以及日韩汉字等。
- **Unicode字符集** ：
  - Unicode编码系统为表达任意语言的任意字符而设计，是业界的一种标准，也称为统一码、标准万国码。
  - 它最多使用4个字节的数字来表达每个字母、符号，或者文字。有三种编码方案，UTF-8、UTF-16和UTF-32。最为常用的UTF-8编码。
  - UTF-8编码，可以用来表示Unicode标准中任何字符，它是电子邮件、网页及其他存储或传送文字的应用中，优先采用的编码。互联网工程工作小组（IETF）要求所有互联网协议都必须支持UTF-8编码。所以，我们开发Web应用，也要使用UTF-8编码。它使用一至四个字节为每个字符编码，编码规则：
    1. 128个US-ASCII字符，只需一个字节编码。
    2. 拉丁文等字符，需要二个字节编码。 
    3. 大部分常用字（含中文），使用三个字节编码。
    4. 其他极少使用的Unicode辅助字符，使用四字节编码。

####  编码引出的问题

在IDEA中，使用`FileReader` 读取项目中的文本文件。由于IDEA的设置，都是默认的`UTF-8`编码，所以没有任何问题。但是，当读取Windows系统中创建的文本文件时，由于Windows系统的默认是GBK编码，就会出现乱码。

```java
public class ReaderDemo {
    public static void main(String[] args) throws IOException {
        FileReader fileReader = new FileReader("E:\\File_GBK.txt");
        int read;
        while ((read = fileReader.read()) != -1) {
            System.out.print((char)read);
        }
        fileReader.close();
    }
}
输出结果：
���
```

那么如何读取GBK编码的文件呢？ 

#### InputStreamReader类  

转换流`java.io.InputStreamReader`，是Reader的子类，是从字节流到字符流的桥梁。它读取字节，并使用指定的字符集将其解码为字符。它的字符集可以由名称指定，也可以接受平台的默认字符集。 

##### 构造方法

- `InputStreamReader(InputStream in)`: 创建一个使用默认字符集的字符流。 
- `InputStreamReader(InputStream in, String charsetName)`: 创建一个指定字符集的字符流。

构造举例，代码如下： 

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```

##### 指定编码读取

```java
public class ReaderDemo2 {
    public static void main(String[] args) throws IOException {
      	// 定义文件路径,文件为gbk编码
        String FileName = "E:\\file_gbk.txt";
      	// 创建流对象,默认UTF8编码
        InputStreamReader isr = new InputStreamReader(new FileInputStream(FileName));
      	// 创建流对象,指定GBK编码
        InputStreamReader isr2 = new InputStreamReader(new FileInputStream(FileName) , "GBK");
		// 定义变量,保存字符
        int read;
      	// 使用默认编码字符流读取,乱码
        while ((read = isr.read()) != -1) {
            System.out.print((char)read); // ��Һ�
        }
        isr.close();
      
      	// 使用指定编码字符流读取,正常解析
        while ((read = isr2.read()) != -1) {
            System.out.print((char)read);// 大家好
        }
        isr2.close();
    }
}
```

#### OutputStreamWriter类

转换流`java.io.OutputStreamWriter` ，是Writer的子类，是从字符流到字节流的桥梁。使用指定的字符集将字符编码为字节。它的字符集可以由名称指定，也可以接受平台的默认字符集。 

##### 构造方法

- `OutputStreamWriter(OutputStream in)`: 创建一个使用默认字符集的字符流。 
- `OutputStreamWriter(OutputStream in, String charsetName)`: 创建一个指定字符集的字符流。

构造举例，代码如下： 

```java
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("out.txt"));
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```

##### 指定编码写出

```java
public class OutputDemo {
    public static void main(String[] args) throws IOException {
      	// 定义文件路径
        String FileName = "E:\\out.txt";
      	// 创建流对象,默认UTF8编码
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(FileName));
        // 写出数据
      	osw.write("你好"); // 保存为6个字节
        osw.close();
      	
		// 定义文件路径
		String FileName2 = "E:\\out2.txt";
     	// 创建流对象,指定GBK编码
        OutputStreamWriter osw2 = new OutputStreamWriter(new FileOutputStream(FileName2),"GBK");
        // 写出数据
      	osw2.write("你好");// 保存为4个字节
        osw2.close();
    }
}
```

##### 转换流理解图解

**转换流是字节与字符间的桥梁！**

![1555228004103](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1555228004103.png)

### 8. 序列化

Java 提供了一种对象**序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该`对象的数据`、`对象的类型`和`对象中存储的属性`等信息。字节序列写出到文件之后，相当于文件中**持久保存**了一个对象的信息。 

反之，该字节序列还可以从文件中读取回来，重构对象，对它进行**反序列化**。`对象的数据`、`对象的类型`和`对象中存储的数据`信息，都可以用来在内存中创建对象。

**注意**

- static属性不参与序列化。
- 对象中的某些属性如果不想被序列化，不能使用static，而是使用transient(瞬态的)修饰。
- 为了防止读和写的序列化ID不一致，一般指定一个固定的序列化ID。

![1555340293628](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1555340293628.png)

#### 8.1 ObjectOutputStream

`java.io.ObjectOutputStream ` 类，将Java对象的原始数据类型写出到文件,实现对象的持久存储。

##### 构造方法

- `public ObjectOutputStream(OutputStream out) `： 创建一个指定OutputStream的ObjectOutputStream。

构造举例，代码如下：  

```java
FileOutputStream fileOut = new FileOutputStream("employee.txt");
ObjectOutputStream out = new ObjectOutputStream(fileOut);
```

##### 序列化操作

1. 一个对象要想序列化，必须满足两个条件:

- 该类必须实现`java.io.Serializable ` 接口，`Serializable` 是一个标记接口，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出`NotSerializableException` 。
- 该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用`transient` 关键字修饰。

```java
public class Employee implements java.io.Serializable {
    public String name;
    public String address;
    public transient int age; // transient瞬态修饰成员,不会被序列化
    public void addressCheck() {
      	System.out.println("Address  check : " + name + " -- " + address);
    }
}
```

2.写出对象方法

- `public final void writeObject (Object obj)` : 将指定的对象写出。

```java
public class SerializeDemo{
   	public static void main(String [] args)   {
    	Employee e = new Employee();
    	e.name = "zhangsan";
    	e.address = "beiqinglu";
    	e.age = 20; 
    	try {
      		// 创建序列化流对象
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.txt"));
        	// 写出对象
        	out.writeObject(e);
        	// 释放资源
        	out.close();
        	fileOut.close();
        	System.out.println("Serialized data is saved"); // 姓名，地址被序列化，年龄没有被序列化。
        } catch(IOException i)   {
            i.printStackTrace();
        }
   	}
}
输出结果：
Serialized data is saved
```

#### 8.2 ObjectInputStream

ObjectInputStream反序列化流，将之前使用ObjectOutputStream序列化的原始数据恢复为对象。 

##### 构造方法

- `public ObjectInputStream(InputStream in) `： 创建一个指定InputStream的ObjectInputStream。

##### 反序列化操作1

如果能找到一个对象的class文件，我们可以进行反序列化操作，调用`ObjectInputStream`读取对象的方法：

- `public final Object readObject ()` : 读取一个对象。

```java
public class DeserializeDemo {
   public static void main(String [] args)   {
        Employee e = null;
        try {		
             // 创建反序列化流
             FileInputStream fileIn = new FileInputStream("employee.txt");
             ObjectInputStream in = new ObjectInputStream(fileIn);
             // 读取一个对象
             e = (Employee) in.readObject();
             // 释放资源
             in.close();
             fileIn.close();
        }catch(IOException i) {
             // 捕获其他异常
             i.printStackTrace();
             return;
        }catch(ClassNotFoundException c)  {
        	// 捕获类找不到异常
             System.out.println("Employee class not found");
             c.printStackTrace();
             return;
        }
        // 无异常,直接打印输出
        System.out.println("Name: " + e.name);	// zhangsan
        System.out.println("Address: " + e.address); // beiqinglu
        System.out.println("age: " + e.age); // 0
    }
}
```

**对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个 `ClassNotFoundException` 异常。**  

##### **反序列化操作2**

**另外，当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个`InvalidClassException`异常。**发生这个异常的原因如下：

- 该类的序列版本号与从流中读取的类描述符的版本号不匹配 
- 该类包含未知数据类型 
- 该类没有可访问的无参数构造方法 

`Serializable` 接口给需要序列化的类，提供了一个序列版本号。`serialVersionUID` 该版本号的目的在于验证序列化的对象和对应类是否版本匹配。

```java
public class Employee implements java.io.Serializable {
     // 加入序列版本号
     private static final long serialVersionUID = 1L;
     public String name;
     public String address;
     // 添加新的属性 ,重新编译, 可以反序列化,该属性赋为默认值.
     public int eid; 

     public void addressCheck() {
         System.out.println("Address  check : " + name + " -- " + address);
     }
}
```



## 六. 多线程 

### 1. 并发与并行

- **并发**：指两个或多个事件在**同一个时间段内**发生。**交替执行**
- **并行**：指两个或多个事件在**同一时刻**发生（同时发生）。**同时执行**

![1555423726186](.\img\6546546546158.png)

<img src=".\img\1555423786648.png" alt="1555423786648" style="zoom:150%;" />

在操作系统中，安装了多个程序，并发指的是在一段时间内宏观上有多个程序同时运行，这在单 CPU 系统中，每一时刻只能有一道程序执行，即微观上这些程序是分时的交替运行，只不过是给人的感觉是同时运行，那是因为分时交替运行的时间是非常短的。

而在多个 CPU 系统中，则这些可以并发执行的程序便可以分配到多个处理器上（CPU），实现多任务并行执行，即利用每个处理器来处理一个可以并发执行的程序，这样多个程序便可以同时执行。目前电脑市场上说的多核 CPU，便是多核处理器，核 越多，并行处理的程序越多，能大大的提高电脑运行的效率。

**CPU的核与线程:**

- 核: 处理点
- 线程: 处理器能够并行(同时执行)执行的任务数
- 四核八线程: 有四个处理点, 能够并行执行八个任务数, 当超过八个任务数时, 会进行并发的处理(交替执行)

> 注意：单核处理器的计算机肯定是不能并行的处理多个任务的，只能是多个任务在单个CPU上并发运行。同理,线程也是一样的，从宏观角度上理解线程是并行运行的，但是从微观角度上分析却是串行运行的，即一个线程一个线程的去运行，当系统只有一个CPU时，线程会以某种顺序执行多个线程，我们把这种情况称之为线程调度。

### 2. 线程与进程

- **进程**：是指一个内存中运行的应用程序，每个进程都有一个独立的内存空间，一个应用程序可以同时运行多个进程；进程也是程序的一次执行过程，是系统运行程序的基本单位；系统运行一个程序即是一个进程从创建、运行到消亡的过程。

- **线程**：线程是进程中的一个执行单元，负责当前进程中程序的执行，一个进程中至少有一个线程。一个进程中是可以有多个线程的，这个应用程序也可以称之为多线程程序。 

  简而言之：一个程序运行后至少有一个进程，一个进程中可以包含多个线程

**进程**

![1555480283297](.\img\1555480283297.png)

**线程**

![1555480581119](.\img\1555480581119.png)

**线程调度:**

- 分时调度

  所有线程轮流使用 CPU 的使用权，平均分配每个线程占用 CPU 的时间。

- 抢占式调度

  优先让优先级高的线程使用 CPU，如果线程的优先级相同，那么会随机选择一个(线程随机性)，Java使用的为抢占式调度。

  - 设置线程的优先级

  <img src=".\img\1555509031282.png" alt="1555509031282" style="zoom:150%;" />

  - 抢占式调度详解

    大部分操作系统都支持多进程并发运行，现在的操作系统几乎都支持同时运行多个程序。比如：现在我们上课一边使用编辑器，一边使用录屏软件，同时还开着画图板，dos窗口等软件。此时，这些程序是在同时运行，”感觉这些软件好像在同一时刻运行着“。

    实际上，CPU(中央处理器)使用抢占式调度模式在多个线程间进行着高速的切换。对于CPU的一个核而言，某个时刻，只能执行一个线程，而 CPU的在多个线程间切换速度相对我们的感觉要快，看上去就是在同一时刻运行。
    **其实，多线程程序并不能提高程序的运行速度，但能够提高程序运行效率，让CPU的使用率更高。**

### 3. 守护线程

- 线程分为用户线程核守护线程;
- **Java虚拟机必须确保用户线程执行完毕;**
- **Java虚拟机不用等待守护线程执行完毕;**
- 如后台记录操作日志, 监控内存使用等

**设置守护线程方法**

- setDaemon(Boolean true)
  - 可以将指定的线程设置成后台线程, **守护线程**;
  - 创建用户线程的线程结束时, 后台线程也随之消亡;
  - 只能在线程启动之前把他为后台线程



### 4. 主线程

- 主线程: 执行主方法(main)的线程
- 单线程程序: java程序中只有一个线程, 执行从main方法开始, 从上到下依次执行

**JVM执行main方法, main方法会进入到栈内存, JVM会找操作系统开辟一条main方法通向cpu的执行路径, cpu就可以通过这个路径来执行main方法, 这个路径就叫做main(主)线程**

![1555515527209](.\img\1555515527209.png)

### 5. 线程

#### 1. 多线程原理

程序启动运行main时候，java虚拟机启动一个进程，主线程main在main()调用时候被创建。随着调用mt的对象的start方法，另外一个新的线程也启动了，这样，整个应用就在多线程下运行

**多线程执行时，在栈内存中，其实每一个执行线程都有一片自己所属的栈内存空间。进行方法的压栈和弹栈。**

<img src=".\img\1555593287390.png" alt="1555593287390" style="zoom:150%;" />

**当执行线程的任务结束了，线程自动在栈内存中释放了。但是当所有的执行线程都结束了，那么进程就结束了。**

<img src=".\img\1555912467358.png" alt="1555912467358" style="zoom:150%;" />

- **新建状态:**

  使用 **new** 关键字和 **Thread** 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 **start()** 这个线程。

- **就绪状态:**

  当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。

- **运行状态:**

  如果就绪状态的线程获取 CPU 资源，就可以执行 **run()**，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。

- **阻塞状态:**

  如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：
  -  等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。
  -  同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。
  - 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

* **死亡状态:**

  一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

**注意：**

​	start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的。

​	从程序运行的结果可以发现，多线程程序是乱序执行。因此，只有乱序执行的代码才有必要设计为多线程。

Thread.sleep()方法调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留出一定时间给其他线程执行的机会。

​	实际上所有的多线程代码执行顺序都是不确定的，每次执行的结果都是随机的。

#### 2. Thread类

##### 构造方法：

- public Thread() :分配一个新的线程对象。
- public Thread(String name) :分配一个指定名字的新的线程对象。
- public Thread(Runnable target) :分配一个带有指定目标新的线程对象
- public Thread(Runnable target,String name) :分配一个带有指定目标新的线程对象并指定名字。

##### 常用方法:

- getName()
  - 获取当前线程名称。
- start()
  - 导致此线程开始执行; Java虚拟机调用此线程的run方法
- run()
  - 此线程要执行的任务在此处定义代码。
- currentThread() 
  - 返回对当前正在执行的线程对象的引用。

- sleep()
  - 使线程停止运行一段时间, 将处于**阻塞状态, 不会释放同步锁**
  - 如果调用了sleep方法之后, 没有其他等待执行的线程, 这个时候当前线程不会马上恢复执行
- join()
  - 当某个程序执行流中调用其他线程的 join() 方法时，**调用线程将被阻塞**，直到 join() 方法加入的 join 线程执行完为止
- yield()
  - 让当前正在执行线程暂停, 不是阻塞线程, 而是将线程转入**就绪状态**
  - 调用了yield方法之后, 如果没有其它等待执行的线程, **此时当前线程就会马上恢复执行**
- wait()
  - 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。	
- notify()
  - 唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。**被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；**例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，**唤醒在此对象监视器上等待的所有线程。**
- setDaemon()
  - 可以将指定的线程设置成后台线程, **守护线程**;
  - 创建用户线程的线程结束时, 后台线程也随之消亡;
  - 只能在线程启动之前把他为后台线程
- setPriority(int newPriority) ；getPriority()
  - 线程的优先级代表的是**概率**
  - 范围从1到10, 默认为5
- stop() 停止使用
  - 不推荐使用

**设置线程名字:**

- 设置线程名称有两种方式, 一种是开启线程(执行run方法)前, 使用`setName(String name)`, 一种是通过构造方法来设置, 在继承Thread类的类中的构造方法中`super(String name) = Thread(String name)`

#### 3. 创建线程一Thread

Java使用`java.lang.Thread`类代表**线程**，所有的线程对象都必须是Thread类或其子类的实例。每个线程的作用是完成一定的任务，实际上就是执行一段程序流即一段顺序执行的代码。Java使用线程执行体来代表这段程序流。Java中通过继承Thread类来**创建**并**启动多线程**的步骤如下：

1. 定义Thread类的子类，并重写该类的run()方法，该run()方法的方法体就代表了线程需要完成的任务,因此把run()方法称为线程执行体。
2. 创建Thread子类的实例，即创建了线程对象
3. 调用线程对象的start()方法来启动该线程
   1. void start() 使该线程开始执行; Java虚拟机调用该线程的`run`方法, 结果是两个线程并发执行; 当前线程(main线程)和另一个线程(创建的新线程, 执行其run方法).
   2. 多次启动一个线程是非法的. 特别是当线程已经结束后, 不能重新启动

**Java程序属于抢占式调度, 哪个线程的优先级高, 哪个线程优先执行(增加概率); 相同优先级, 随机一个执行**

代码如下：

测试类：

```java
public class Demo01 {
	public static void main(String[] args) {
		//创建自定义线程对象
		MyThread mt = new MyThread("新的线程！");
		//开启新线程
		mt.start();
		//在主方法中执行for循环
		for (int i = 0; i < 10; i++) {
			System.out.println("main线程！"+i);
		}
	}
}
```

自定义线程类：

```java
public class MyThread extends Thread {
	//定义指定线程名称的构造方法
	public MyThread(String name) {
		//调用父类的String参数的构造方法，指定线程的名称
		super(name);
	}
	/**
	 * 重写run方法，完成该线程执行的逻辑
	 */
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(getName()+"：正在执行！"+i);
		}
	}
}
```

<img src=".\img\1555592299397.png" alt="1555592299397" style="zoom:250%;" />

#### 4. 创建线程二Runnable

采用 `java.lang.Runnable` 也是非常常见的一种，我们只需要重写run方法即可。

**步骤如下:**

1. 定义Runnable接口的实现类, 并重写该接口的run()方法, 该run()方法的方法体同样是该线程的线程执行体.
2. 创建Runnable实现类的实例, 并以此实例作为Thread的target来创建Thread对象, 该Thread对象才是真正的线程对象.
3. 调用线程对象的start()方法来启动线程.

**实现Runnable接口创建对线程程序的好处;**

1. **避免了单继承的局限性**
2. 增加了程序的扩展性, 降低了程序的耦合度(解耦)
   1. 实现Runnable接口的方式, **把设置线程任务和开启新线程进行了分离(解耦)**
   2. 实现类中,  重写了run方法: 用来设置线程任务
   3. 创建Thread对象, 调用start方法: 用来开启新线程

```Java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
        	System.out.println(Thread.currentThread().getName()+" "+i);
    	}
    }
}
public class Demo {
    public static void main(String[] args) {
        //创建自定义类对象 线程任务对象
        MyRunnable mr = new MyRunnable();
        //创建线程对象
        //Thread t = new Thread(mr);
        Thread t = new Thread(mr, "小强");
        t.start();
        for (int i = 0; i < 20; i++) {
            System.out.println("旺财 " + i);
        }
     }
}
```

实际上所有的多线程代码都是通过运行Thread的start()方法来运行的。因此，不管是继承Thread类还是实现Runnable接口来实现多线程，最终还是通过Thread的对象的API来控制线程的，熟悉Thread类的API是进行多线程编程的基础。

> tips:Runnable对象仅仅作为Thread对象的target，Runnable实现类里包含的run()方法仅作为线程执行体。而实际的线程对象依然是Thread实例，只是该Thread线程负责执行其target的run()方法。

#### 5. Thread和Runnable的区别

如果一个类继承Thread, 则不适合资源共享. 但是如果实现了Runnable接口的话, 则很容易的实现资源共享.

**实现Runnable接口比继承Thread类所具有的优势：**

1. 适合**多个相同的程序代码的线程去共享同一个资源。**
2. 可以避免java中的单继承的局限性。
3.  增加程序的健壮性，实现解耦操作，代码可以被多个线程共享，代码和线程独立。
4. **线程池只能放入实现Runable或Callable类线程，不能直接放入继承Thread的类。**

> 扩充：在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用
> java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM其实在就是在操作系统中启动了一个进
> 程。

#### 6. 匿名内部类方式实现线程

使用线程的内匿名内部类方式，可以方便的实现每个线程执行不同的线程任务操作。
使用匿名内部类的方式实现Runnable接口，重新Runnable接口中的run方法：

```Java
public static void main(String[] args) {
    	//匿名内部类
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println(Thread.currentThread().getName() + "---->" + i);
                }
            }
        };
        new Thread(runnable).start();
		//匿名对象
        new Thread() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println(Thread.currentThread().getName() + "---->" + i);
                }
            }
        }.start();

        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName() + "---->" + i);
        }
    }
```

### 6. 线程安全

当我们使用多个线程访问同一资源的时候，且多个线程中对资源有写的操作，就容易出现线程安全问题。
要解决上述多线程并发访问一个资源的安全性问题:也就是解决重复票与不存在票问题，Java中提供了同步机制
(synchronized)来解决。

为了保证每个线程都能正常执行原子操作,Java引入了线程同步机制。

1. 同步代码块
2. 同步方法
3. 锁机制

> 线程安全问题都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写
> 操作，一般来说，这个全局变量是线程安全的；**若有多个线程同时执行写操作，一般都需要考虑线程同步，**
> **否则的话就可能影响线程安全。**

> synchronized关键字可以作为函数的修饰符，也可作为函数内的语句，也就是平时说的同步方法和同步语句块。如果再细的分类，**synchronized可作用于instance变量、object reference（对象引用）、static函数和class literals(类名称字面常量)身上。**

> synchronized就是针对内存区块申请内存锁，**this关键字代表类的一个对象，所以其内存锁是针对相同对象的互斥操作，**而static成员属于类专有，其内存空间为该类所有成员共有，**这就导致synchronized()对static成员加锁，相当于对类加锁，也就是在该类的所有成员间实现互斥，在同一时间只有一个线程可访问该类的实例。**

- **synchronized关键字的作用域有二种：**
  - 是某个对象实例内，synchronized aMethod(){}可以防止多个线程同时访问这个对象的synchronized方法（**如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法**）。这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同类的另一个对象实例中的synchronized方法；
  - 是某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static 方法。**它可以对类的所有对象实例起作用。**

- 除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。用法是: synchronized(this){/*区块*/}，它的作用域是当前对象；

- synchronized关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法；

#### 1. 同步代码块

- 同步代码块： synchronized 关键字可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。

```Java
synchronized(同步锁){
	需要同步操作的代码
}
```

**同步锁: **

对象的同步锁只是一个概念, 可以想象为在对象上标记一个锁.

1. 锁对象 可以是任意对象
2. 多个线程对象 要使用同一把锁.

> 注意:在任何时候,最多允许一个线程拥有同步锁,谁拿到锁就进入代码块,其他的线程只能在外等着(BLOCKED)。

```Java
public class SynchronizedObjectOne implements Runnable {

    private int ticket = 100;
	//注：零长度的byte数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的byte[]对象只需3条操作码，而Object lock = new Object()则需要7行操作码。
    private byte[] lock = new byte[0];  // 特殊的instance变量
    
    @Override
    public void run() {
        while (true) {
            //同步代码块方式
            synchronized (this) {
                if (0 < ticket) {
                    try {
                        //睡眠线程
                        Thread.sleep(50L);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "--->" + ticket--);
                } else {
                    return;
                }

            }
        }
    }
}
```

- **同步代码块的锁可以是一个创建的对象, this, static 对象, .class(字节码对象)**
- 创建的对象可以是类的成员变量(创建一个实例只执行一次), 方法的实际参数(传入的对象是), run()中创建的对象(run()方法每个线程都会执行, 执行一次创建一个对象); **对象相同, 使用的是同一个锁, 对象不同,即锁不一样;**
- this指的是调用这个方法的对象(创建的实例); **不同实例开启的线程, 锁是不一样的, 不受同步机制的控制**
- static成员属于类专有(类的实例对象共享)，其内存空间为该类所有成员共有，这就导致synchronized()对static成员加锁，相当于对类加锁; **对于类锁, 不同实例开启的线程在同一时间只有一个线程可访问该类的实例。**
- **字节码对象是某个类的范围，防止不同对象实例的线程同时访问这个类中的某个方法。它可以对类的所有对象实例起作用; 一个对象实例的线程拿到锁后, 其它对象实例的线程只能等待解锁**



#### 2. 同步方法

同步方法:使用synchronized修饰的方法,就叫做同步方法,保证A线程执行该方法的时候,其他线程只能在方法外
等着。

> 同步锁是谁?
> 对于非static方法,同步锁就是this。
> 对于static方法,我们使用当前方法所在类的字节码对象(类名.class)。

```Java
package cn.galc.test;

public class TestSync implements Runnable {
    Timer timer = new Timer();

    public static void main(String args[]) {
        TestSync test = new TestSync();
        Thread t1 = new Thread(test);
        Thread t2 = new Thread(test);
        t1.setName("t1");// 设置t1线程的名字
        t2.setName("t2");// 设置t2线程的名字
        t1.start();
        t2.start();
    }

    public void run() {
        timer.add(Thread.currentThread().getName());
    }
}

class Timer {
    private static int num = 0;

    public /* static **/ synchronized void add(String name) {
        // 在声明方法时加入synchronized时表示在执行这个方法的过程之中当前对象被锁定
        num++;
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(name + "：你是第" + num + "个使用timer的线程");    
    }
}
```

- 同步方法的锁只有两个, 普通的同步方法的同步锁是**调用方法的实例对象(this)**, 静态的同步方法的同步锁是**类的字节码对象(.class)**
- **如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法**
- **是某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static 方法。它可以对类的所有对象实例起作用。**

#### 3. Lock类

**`Lock`实现提供比使用`synchronized`方法和语句可以获得的更广泛的锁定操作。**

主要目的是和synchronized一样， 两者都是为了解决同步问题，处理资源争端而产生的技术。功能类似但有一些区别。

**区别如下:**

```
  lock更灵活，可以自由定义多把锁的枷锁解锁顺序（synchronized要按照先加的后解顺序）
  提供多种加锁方案，lock 阻塞式, trylock 无阻塞式, lockInterruptily 可打断式， 还有trylock的带   超时时间版本。
  本质上和监视器锁（即synchronized是一样的）
  能力越大，责任越大，必须控制好加锁和解锁，否则会导致灾难。
  和Condition类的结合。
```

- Lock是显式锁（手动开启和关闭锁），又叫重复锁，synchronized是隐式锁，出了作用域自动释放；
- Lock只有代码块锁，synchronized有方法锁和代码块锁；
- **使用Lock锁，JVM将花费较少的时间来调度线程**，性能更好。并且具有更好的扩展性；

**构造方法如下：**

`new ReentrantLock()`：源码中是执行的`new NonfairSync()`。**同步对象，用于非公平锁，即线程执行是无序的;**

`new ReentrantLock(Boolean fair)`：**参数为true，执行的是`new FairSync()`。为公平锁同步对象，线程执行是有序的；参数为false，执行`new NonfairSync()`**

**Lock锁也称为同步锁, 加锁与释放锁方法如下:**

- `public void lock()` :加同步锁
- `public void unlock()` :释放同步锁

```java
public class Ticket implements Runnable{
    private int ticket = 100;
    Lock lock = new ReentrantLock();
    /** 执行卖票操作*/
    @Override
    public void run() {
        //每个窗口卖票的操作
        //窗口 永远开启
        while(true){
            lock.lock();
            try {
                if(ticket>0){//有票 可以卖
                    //出票操作
                    //使用sleep模拟一下出票时间
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        // TODO Auto‐generated catch block
                        e.printStackTrace();
                    }
                    //获取当前线程对象的名字
                    String name =Thread.currentThread().getName();
                    System.out.println(name+"正在卖:"+ticket‐‐);
                }
            } finally {
               lock.unlock(); 
            }     
        }
    }
}
```

### 7. 线程状态

![img](.\img\70)

- **新建**（new）：线程对象被创建后就进入了新建状态。如：Thread thread = new Thread();
- **就绪状态**（Runnable）：也被称为“可执行状态”。线程对象被创建后，其他线程调用了该对象的start()方法，从而启动该线程。如：thread.start(); 处于就绪状态的线程随时可能被CPU调度执行。
- **运行状态**（Running）：线程获取CPU权限进行执行。需要注意的是，线程只能从就绪状态进入到运行状态。
- **阻塞状态**（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权限，暂时停止运行。直到线程进入就绪状态，才有机会进入运行状态。阻塞的三种情况：
  - ***等待阻塞***：运行的线程执行wait()方法，JVM会把该线程放入等待池中。(wait会释放持有的锁)
  - ***同步阻塞***：线程在获取synchronized同步锁失败（因为锁被其他线程占用），它会进入同步阻塞状态。
  - ***其他阻塞***：通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或超时、或者I/O处理完毕时，线程重新转入就绪状态。**（注意,sleep是不会释放持有的锁）**
- **死亡状态**（Dead）：线程执行完了或因异常退出了run()方法，该线程结束生命周期。



#### 阻塞状态

<img src=".\img\1556113314786.png" style="zoom:150%;" />

##### 计时等待(其他阻塞): 

**线程睡眠：Thread.sleep(long millis)方法**，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

**线程加入：join()方法**，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

![](.\img\1556113512777.png)

##### 无限等待(等待阻塞): 

**线程等待：Object类中的wait()方法**，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

![](.\img\1556113602119.png)

##### 锁阻塞(同步阻塞):

**线程在获取synchronized同步锁失败（因为锁被其他线程占用），它会进入同步阻塞状态。**

<img src=".\img\1556115653109.png" alt="1556115653109" style="zoom:150%;" />



**线程让步：Thread.yield() 方法**，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。让当前正在执行线程暂停, 不是阻塞线程, 而是将线程转入**就绪状态**

**线程唤醒：Object类中的notify()方法**，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。

### 8.死锁

- **不同的线程分别占用对方需要的同步资源不放弃，都在等对方放弃自己需要的同步资源，就形成了线程的死锁；**
- **出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于堵塞状态，无法继续；**

![1569747524875](.\img\1569747524875.png)

**解决办法：**

- 专门的算法，原则
- 尽量减**少同步资源的定义**
- 尽量**避免嵌套同步**

### 9. 等待唤醒机制

#### 1.线程间通信

**概念：**多个线程在处理同一个资源，但是处理的动作（线程的任务）却不相同。

比如：线程A用来生成包子的，线程B用来吃包子的，包子可以理解为同一资源，线程A与线程B处理的动作，一个是生产，一个是消费，那么线程A与线程B之间就存在线程通信问题。

<img src=".\img\1556543411245.png" alt="1556543411245" style="zoom:150%;" />

**为什么要处理线程间通信：**

多个线程并发执行时, 在默认情况下CPU是随机切换线程的，当我们需要多个线程来共同完成一件任务，并且我们希望他们有规律的执行, 那么多线程之间需要一些协调通信，以此来帮我们达到多线程共同操作一份数据。

**如何保证线程间通信有效利用资源：**

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作。 就是多个线程在操作同一份数据时， 避免对同一共享变量的争夺。也就是我们需要通过一定的手段使各个线程能有效的利用资源。而这种手段即—— **等待唤醒机制。**

#### 2. 等待唤醒机制

**什么是等待唤醒机制**

这是多个线程间的一种**协作**机制。谈到线程我们经常想到的是线程间的**竞争（race）**，比如去争夺锁，但这并不是故事的全部，线程间也会有协作机制。就好比在公司里你和你的同事们，你们可能存在在晋升时的竞争，但更多时候你们更多是一起合作以完成某些任务。

就是在一个线程进行了规定操作后，就进入等待状态（**wait()**）， 等待其他线程执行完他们的指定代码过后 再将其唤醒（**notify()**）;在有多个线程进行等待时， 如果需要，可以使用 notifyAll()来唤醒所有的等待线程。

wait/notify 就是线程间的一种协作机制。

**等待唤醒中的方法**

等待唤醒机制就是用于解决线程间通信的问题的，使用到的3个方法的含义如下：

1. wait：线程不再活动，不再参与调度，进入 wait set 中，因此不会浪费 CPU 资源，也不会去竞争锁了，这时的线程状态即是 WAITING。它还要等着别的线程执行一个**特别的动作**，也即是“**通知（notify）**”在这个对象上等待的线程从wait set 中释放出来，重新进入到调度队列（ready queue）中
2. notify：则选取所通知对象的 wait set 中的一个线程释放；例如，餐馆有空位置后，等候就餐最久的顾客最先入座。
3. notifyAll：则释放所通知对象的 wait set 上的全部线程。

> 注意：
>
> 哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不持有锁，所以她需要再次尝试去获取锁（很可能面临其它线程的竞争），成功后才能在当初调用 wait 方法之后的地方恢复执行。
>
> 总结如下：
>
> - 如果能获取锁，线程就从 WAITING 状态变成 RUNNABLE 状态；
> - 否则，从 wait set 出来，又进入 entry set，线程就从 WAITING 状态又变成 BLOCKED 状态

**调用wait和notify方法需要注意的细节**

1. wait方法与notify方法必须要由同一个锁对象调用。因为：对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
2. wait方法与notify方法是属于Object类的方法的。因为：锁对象可以是任意对象，而任意对象的所属类都是继承了Object类的。
3. wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为：必须要通过锁对象调用这2个方法。

#### 3. 生产者与消费者问题

等待唤醒机制其实就是经典的“生产者与消费者”的问题。

就拿生产包子消费包子来说等待唤醒机制如何有效利用资源：

```java
包子铺线程生产包子，吃货线程消费包子。当包子没有时（包子状态为false），吃货线程等待，包子铺线程生产包子（即包子状态为true），并通知吃货线程（解除吃货的等待状态）,因为已经有包子了，那么包子铺线程进入等待状态。接下来，吃货线程能否进一步执行则取决于锁的获取情况。如果吃货获取到锁，那么就执行吃包子动作，包子吃完（包子状态为false），并通知包子铺线程（解除包子铺的等待状态）,吃货线程进入等待。包子铺线程能否进一步执行则取决于锁的获取情况。
```

**代码演示：**

包子资源类：

```java
public class BaoZi {
     String  pier ;
     String  xianer ;
     boolean  flag = false ;//包子资源 是否存在  包子资源状态
}
```

吃货线程类：

```java
public class ChiHuo extends Thread{
    private BaoZi bz;

    public ChiHuo(String name,BaoZi bz){
        super(name);
        this.bz = bz;
    }
    @Override
    public void run() {
        while(true){
            synchronized (bz){
                if(bz.flag == false){//没包子
                    try {
                        bz.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("吃货正在吃"+bz.pier+bz.xianer+"包子");
                bz.flag = false;
                bz.notify();
            }
        }
    }
}
```

包子铺线程类：

```java
public class BaoZiPu extends Thread {

    private BaoZi bz;

    public BaoZiPu(String name,BaoZi bz){
        super(name);
        this.bz = bz;
    }

    @Override
    public void run() {
        int count = 0;
        //造包子
        while(true){
            //同步
            synchronized (bz){
                if(bz.flag == true){//包子资源  存在
                    try {

                        bz.wait();

                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                // 没有包子  造包子
                System.out.println("包子铺开始做包子");
                if(count%2 == 0){
                    // 冰皮  五仁
                    bz.pier = "冰皮";
                    bz.xianer = "五仁";
                }else{
                    // 薄皮  牛肉大葱
                    bz.pier = "薄皮";
                    bz.xianer = "牛肉大葱";
                }
                count++;

                bz.flag=true;
                System.out.println("包子造好了："+bz.pier+bz.xianer);
                System.out.println("吃货来吃吧");
                //唤醒等待线程 （吃货）
                bz.notify();
            }
        }
    }
}
```

测试类：

```java
public class Demo {
    public static void main(String[] args) {
        //等待唤醒案例
        BaoZi bz = new BaoZi();

        ChiHuo ch = new ChiHuo("吃货",bz);
        BaoZiPu bzp = new BaoZiPu("包子铺",bz);

        ch.start();
        bzp.start();
    }
}
```

执行效果：

```java
包子铺开始做包子
包子造好了：冰皮五仁
吃货来吃吧
吃货正在吃冰皮五仁包子
包子铺开始做包子
包子造好了：薄皮牛肉大葱
吃货来吃吧
吃货正在吃薄皮牛肉大葱包子
包子铺开始做包子
包子造好了：冰皮五仁
吃货来吃吧
吃货正在吃冰皮五仁包子
```

### 10. 线程池

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

通过线程池可以使得线程可以复用，就是执行完一个任务，并不被销毁,可以继续执行其他的任务.

#### 1. 线程池的概念

**线程池：**其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。

<img src=".\img\1556544232259.png" alt="1556544232259" style="zoom:150%;" />

合理利用线程池能够带来三个好处：

1. **降低资源消耗。**减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. **提高响应速度。**当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. **提高线程的可管理性。**可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

#### 2. 线程池的使用

Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工程类来创建线程池对象。

Executors类中有个创建线程池的方法如下：

- `public static ExecutorService newFixedThreadPool(int nThreads)`：返回线程池对象。(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)

获取到了一个线程池ExecutorService 对象，那么怎么使用呢，在这里定义了一个使用线程池对象的方法如下：

- `public Future<?> submit(Runnable task)`:获取线程池中的某一个线程对象，并执行

  > Future接口：用来记录线程任务执行完毕后产生的结果。线程池创建与使用。

使用线程池中线程对象的步骤：

1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

Runnable实现类代码：

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("教练来了： " + Thread.currentThread().getName());
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
```

线程池测试类：

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.submit(r);
        // 再获取个线程对象，调用MyRunnable中的run()
        service.submit(r);
        service.submit(r);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}
```

### 11. volatile

`volatile`保证线程间变量的**同步, 可见性**, 简单的说就是当线程A对变量X进行了修改后, 在现场A后面执行的其它线程能看到变量X的变动, 更详细的说是要符合以下两个规则:

- 线程对变量进行修改后, 要立刻回写到主内存.
- 线程对变量读取的时候, 要从主内存中读, 而不是缓存

![1556978009056](.\img\1556978009056.png)

![1556978500027](.\img\1556978500027.png)

原子性: 事务中各项操作，要么全做要么全不做，任何一项操作的失败都会导致整个事务的失败；

```Java
public class VolatileDemoOne {
    private volatile static int num = 0;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (0 == num) {

            }
        }).start();
		
        /**
        *静态变量不使用volatile修饰,while无线循环一直占用CPU资源,无法执行更改的数据会写到主内存中
        */
        Thread.sleep(1000);
        num = 1;
    }
}
```

### 12. CountDownLatch

CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程执行完后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有框架服务之后执行。

CountDownLatch是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得减1。当计数器到达0时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务。

- countDown() 当前线程调此方法，则计数减一(建议放在 finally里执行) 
- await()， 调用此方法会一直阻塞当前线程，直到计时器的值为0

```java
public class Test1 {
 
    public static ExecutorService executorService = Executors.newCachedThreadPool();
    private static CountDownLatch cdl = new CountDownLatch(10);
    private static final Random random = new Random();
 
    public void test() {
        for (int i = 0; i < 10; i++) executorService.execute(new ThreadTest());
    }
 
    public static void main(String[] args) {
        new Test1().test();
 
        //插入数据完成后  执行修改操作
        try {
            cdl.await();
        } catch (InterruptedException e) {
        }
        System.out.println("它们已经插完啦..............................");
        executorService.shutdown();
 
    }
 
    class ThreadTest implements Runnable {
 
        public void run() {
            //执行插入数据操作  每次插入一条
            // 模拟耗时
            int time = random.nextInt(10000);
            try {
                Thread.sleep(time);
            } catch (InterruptedException e) {
            }
            System.out.println(Thread.currentThread().getName() + "执行完了，耗时：" + time / 1000 + "秒");
            cdl.countDown();
        }
    }
}
```



## 七. 网络编程

### 1. 网路编程入门

#### 1.1 软件结构

- **C/S结构** ：全称为Client/Server结构，是指客户端和服务器结构。常见程序有ＱＱ、迅雷等软件。
- **B/S结构** ：全称为Browser/Server结构，是指浏览器和服务器结构。常见浏览器有谷歌、火狐等。

两种架构各有优势，但是无论哪种架构，都离不开网络的支持。**网络编程**，就是在一定的协议下，实现两台计算机的通信的程序。

#### 1.2. 网络通信协议

- **网络通信协议：**通过计算机网络可以使多台计算机实现连接，位于同一个网络中的计算机在进行连接和通信时需要遵守一定的规则，这就好比在道路中行驶的汽车一定要遵守交通规则一样。在计算机网络中，这些连接和通信的规则被称为网络通信协议，它对数据的传输格式、传输速率、传输步骤等做了统一规定，通信双方必须同时遵守才能完成数据交换。

- **TCP/IP协议：** 传输控制协议/因特网互联协议( Transmission Control Protocol/Internet Protocol)，是Internet最基本、最广泛的协议。它定义了计算机如何连入因特网，以及数据如何在它们之间传输的标准。它的内部包含一系列的用于处理数据通信的协议，并采用了4层的分层模型，每一层都呼叫它的下一层所提供的协议来完成自己的需求。

![1557061181233](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557061181233.png)

上图中，TCP/IP协议中的四层分别是应用层、传输层、网络层和链路层，每层分别负责不同的通信功能。
链路层：链路层是用于定义物理传输通道，通常是对某些网络连接设备的驱动协议，例如针对光纤、网线提供的驱动。
网络层：网络层是整个TCP/IP协议的核心，它主要用于将传输的数据进行分组，将分组数据发送到目标计算机或者网络。
运输层：主要使网络程序进行通信，在进行网络通信时，可以采用TCP协议，也可以采用UDP协议。
应用层：主要负责应用程序的协议，例如HTTP协议、FTP协议等。

#### 1.3 协议分类

`java.net` 包中提供了两种常见的网络协议的支持：

- **UDP**：用户数据报协议(User Datagram Protocol)。**UDP是无连接通信协议，即在数据传输时，数据的发送端和接收端不建立逻辑连接。**简单来说，当一台计算机向另外一台计算机发送数据时，发送端不会确认接收端是否存在，就会发出数据，同样接收端在收到数据时，也不会向发送端反馈是否收到数据。

  由于**使用UDP协议消耗资源小，通信效率高**，所以通常都会用于音频、视频和普通数据的传输例如视频会议都使用UDP协议，因为这种情况即使偶尔丢失一两个数据包，也不会对接收结果产生太大影响。

  但是在使用UDP协议传送数据时，由于UDP的面向无连接性，不能保证数据的完整性，因此在传输重要数据时不建议使用UDP协议。UDP的交换过程如下图所示。

![1557061479384](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557061479384.png)

特点:数据被限制在64kb以内，超出这个范围就不能发送了。

数据报(Datagram):网络传输的基本单位 

URG：紧急指针（urgent pointer）有效。
ACK：确认序号有效。
PSH：接收方应该尽快将这个报文交给应用层。
RST：重置连接。
SYN：发起一个新连接。
FIN：释放一个连接。

- **TCP**：传输控制协议 (Transmission Control Protocol)。TCP协议是**面向连接的通信协议，即传输数据之前，在发送端和接收端建立逻辑连接**，然后再传输数据，它提供了两台计算机之间可靠无差错的数据传输。

> 在TCP连接中必须要明确客户端与服务器端，由客户端向服务端发出连接请求，每次连接的创建都需要经过“三次握手”。

- 三次握手：所谓三次握手（Three-Way Handshake）即建立TCP连接，就是指建立一个TCP连接时，需要客户端和服务端总共发送3个包以确认连接的建立。在socket编程中，这一过程由客户端执行connect来触发
  - 第一次握手: Client将标志位SYN置为1，随机产生一个值seq=J，并将该数据包发送给Server，Client进入SYN_SENT状态，等待Server确认。
  - 第二次握手: Server收到数据包后由标志位SYN=1知道Client请求建立连接，Server将标志位SYN和ACK都置为1，ack=J+1，随机产生一个值seq=K，并将该数据包发送给Client以确认连接请求，Server进入SYN_RCVD状态。
  - 第三次握手: Client收到确认后，检查ack是否为J+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=K+1，并将该数据包发送给Server，Server检查ack是否为K+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了。

![img](.\img\22312037_1365405910EROI.png)

![1557061989481](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557061989481.png)

​    完成三次握手，连接建立后，客户端和服务器就可以开始进行数据传输了。由于这种面向连接的特性，TCP协议可以保证传输数据的安全，所以应用十分广泛，例如下载文件、浏览网页等。

- 四次握手:   由于TCP连接时全双工的，因此，每个方向都必须要单独进行关闭，这一原则是当一方完成数据发送任务后，发送一个FIN来终止这一方向的连接，收到一个FIN只是意味着这一方向上没有数据流动了，即不会再收到数据了，但是在这个TCP连接上仍然能够发送数据，直到这一方向也发送了FIN。首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭，上图描述的即是如此。
  - 第一次挥手：Client发送一个FIN，**用来关闭Client到Server的数据传送**，Client进入FIN_WAIT_1状态。
  - 第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。
  - 第三次挥手：Server发送一个FIN，**用来关闭Server到Client的数据传送**，Server进入LAST_ACK状态。
  - 第四次挥手：Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。

![img](.\img\22312037_1365503104wDR0.png)

为什么建立连接是三次握手，而关闭连接却是四次挥手呢？

​        这是因为服务端在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。而关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。

#### 1.4 网络编程三要素

##### 协议

- **协议：**计算机网络通信必须遵守的规则，已经介绍过了，不再赘述。

##### IP地址

- **IP地址：指互联网协议地址（Internet Protocol Address）**，俗称IP。IP地址用来给一个网络中的计算机设备做唯一的编号。假如我们把“个人电脑”比作“一台电话”的话，那么“IP地址”就相当于“电话号码”。

**IP地址分类**

- IPv4：是一个32位的二进制数，通常被分为4个字节，表示成`a.b.c.d` 的形式，例如`192.168.65.100` 。其中a、b、c、d都是0~255之间的十进制整数，那么最多可以表示42亿个。

- IPv6：由于互联网的蓬勃发展，IP地址的需求量愈来愈大，但是网络地址资源有限，使得IP的分配越发紧张。

  为了扩大地址空间，拟通过IPv6重新定义地址空间，采用128位地址长度，每16个字节一组，分成8组十六进制数，表示成`ABCD:EF01:2345:6789:ABCD:EF01:2345:6789`，号称可以为全世界的每一粒沙子编上一个网址，这样就解决了网络地址资源数量不够的问题。

**常用命令**

- 查看本机IP地址，在控制台输入：

```java
ipconfig
```

- 检查网络是否连通，在控制台输入：

```java
ping 空格 IP地址
ping 220.181.57.216
```

**特殊的IP地址**

- 本机IP地址：`127.0.0.1`、`localhost` 。

##### 端口号

网络的通信，本质上是两个进程（应用程序）的通信。每台计算机都有很多的进程，那么在网络通信时，如何区分这些进程呢？

如果说**IP地址**可以唯一标识网络中的设备，那么**端口号**就可以唯一标识设备中的进程（应用程序）了。

- **端口号：用两个字节表示的整数，它的取值范围是0~65535**。其中，0~1023之间的端口号用于一些知名的网络服务和应用，普通的应用程序需要使用1024以上的端口号。如果端口号被另外一个服务或应用所占用，会导致当前程序启动失败。

利用`协议`+`IP地址`+`端口号` 三元组合，就可以标识网络中的进程了，那么进程间的通信就可以利用这个标识与其它进程进行交互。

### 2. TCP通信程序

TCP通信能实现两台计算机之间的数据交互，通信的两端，要严格区分为客户端（Client）与服务端（Server）。

**两端通信时步骤：**

1. 服务端程序，需要事先启动，等待客户端的连接。
2. 客户端主动连接服务器端，连接成功才能通信。服务端不可以主动连接客户端。
3. 客户端和服务端就会建立一个逻辑连接, 而这个连接中包含一个对象, 这个对象就是IO流
4. 通信的数据不仅仅是字符, 所以IO流对象是字节**流对象**

**服务器端必须明确两件事:**

1. 多个客户端同时和服务器进行交互, 服务器必须明确和哪个客户端进行的交互, 在服务端有一个方法, 叫`accept()`获取到请求的客户端对象
2. 多个客户端同时和服务端进行交互, 就需要使用多个IO流对象

> - 服务器是没有IO流的, 但是服务器可以获取到请求的客户端对象Socket, 使用每个客户端Socket中提供的IO流和客户进行交互(**服务器使用客户端的流和客户端交互**)
>
> - 服务器使用客户端的字节输入流读取客户端发送的数据
>
> - 服务器使用客户端的字节输出流给客户端回写数据

**在Java中，提供了两个类用于实现TCP通信程序：**

1. 客户端：`java.net.Socket` 类表示。创建`Socket`对象，向服务端发出连接请求，服务端响应请求，两者建立连接开始通信。
2. 服务端：`java.net.ServerSocket` 类表示。创建`ServerSocket`对象，相当于开启一个服务，并等待客户端的连接。

#### 1.  Socket类

`Socket` 类：该类实现客户端套接字，套接字指的是两台设备之间通讯的端点。

##### 构造方法

- `public Socket(String host, int port)` :创建套接字对象并将其连接到指定主机上的指定端口号。如果指定的host是null ，则相当于指定地址为回送地址。  

  > 小贴士：回送地址(127.x.x.x) 是本机回送地址（Loopback Address），主要用于网络软件测试以及本地机进程间通信，无论什么程序，一旦使用回送地址发送数据，立即返回，不进行任何网络传输。

构造举例，代码如下：

```java
Socket client = new Socket("127.0.0.1", 6666);
```

##### 成员方法

- `public InputStream getInputStream()` ： 返回此套接字的输入流。
  - 如果此Scoket具有相关联的通道，则生成的InputStream 的所有操作也关联该通道。
  - 关闭生成的InputStream也将关闭相关的Socket。
- `public OutputStream getOutputStream()` ： 返回此套接字的输出流。
  - 如果此Scoket具有相关联的通道，则生成的OutputStream 的所有操作也关联该通道。
  - 关闭生成的OutputStream也将关闭相关的Socket。
- `public void close()` ：关闭此套接字。
  - 一旦一个socket被关闭，它不可再使用。
  - 关闭此socket也将关闭相关的InputStream和OutputStream 。 
- `public void shutdownOutput()` ： **禁用此套接字的输出流。**   
  - 禁用此套接字的输出流，对于 TCP 套接字，任何以前写入的数据都将被发送，并且后跟 TCP 的正常连接终止序列（即-1），之后，从另一端TCP套接字的输入流中读取数据时，如果到达输入流末尾而不再有数据可用，则返回 `-1`。
- `public void shutdownInput()` ： **禁用此套接字的输入流。**   
  -  禁用此套接字的输入流，发送到套接字的输入流端的任何数据都将被确认然后被静默丢弃。任何想从该套接字的输入流读取数据的操作都将返回-1；

**实现步骤:**
        1.创建一个客户端对象Socket,构造方法绑定服务器的IP地址和端口号
        2.使用Socket对象中的方法getOutputStream()获取网络字节输出流OutputStream对象
        3.使用网络字节输出流OutputStream对象中的方法write,给服务器发送数据
        4.使用Socket对象中的方法getInputStream()获取网络字节输入流InputStream对象
        5.使用网络字节输入流InputStream对象中的方法read,读取服务器回写的数据
        6.释放资源(Socket)

**注意:**
        1.客户端和服务器端进行交互,必须使用Socket中提供的网络流,不能使用自己创建的流对象
        2.当我们创建客户端对象Socket的时候,就会去请求服务器和服务器经过3次握手建立连接通路
            这时如果服务器没有启动,那么就会抛出异常ConnectException: Connection refused: connect
            如果服务器已经启动,那么就可以进行交互了

```Java
public static void main(String[] args) throws IOException {
        //1.创建一个客户端对象Socket,构造方法绑定服务器的IP地址和端口号
        Socket socket = new Socket("127.0.0.1",8888);
        //2.使用Socket对象中的方法getOutputStream()获取网络字节输出流OutputStream对象
        OutputStream os = socket.getOutputStream();
        //3.使用网络字节输出流OutputStream对象中的方法write,给服务器发送数据
        os.write("你好服务器".getBytes());

        //4.使用Socket对象中的方法getInputStream()获取网络字节输入流InputStream对象
        InputStream is = socket.getInputStream();

        //5.使用网络字节输入流InputStream对象中的方法read,读取服务器回写的数据
        byte[] bytes = new byte[1024];
        int len = is.read(bytes);
        System.out.println(new String(bytes,0,len));

        //6.释放资源(Socket,IO流)
    	is.close();
    	os.close();
        socket.close();
}
```

#### 2. ServerSocket类

`ServerSocket`类：这个类实现了服务器套接字，该对象等待通过网络的请求。

##### 构造方法

- `public ServerSocket(int port)` ：使用该构造方法在创建ServerSocket对象时，就可以将其绑定到一个指定的端口号上，参数port就是端口号。

构造举例，代码如下：

```java
ServerSocket server = new ServerSocket(6666);
```

##### 成员方法

- `public Socket accept()` ：侦听并接受连接，返回一个新的Socket对象(**请求服务器端的客户端对象**)，**用于和客户端实现通信。该方法会一直阻塞直到建立连接。** 

**服务器的实现步骤:**
        1.创建服务器ServerSocket对象和系统要指定的端口号
        2.使用ServerSocket对象中的方法accept,获取到请求的客户端对象Socket
        3.使用Socket对象中的方法getInputStream()获取网络字节输入流InputStream对象
        4.使用网络字节输入流InputStream对象中的方法read,读取客户端发送的数据
        5.使用Socket对象中的方法getOutputStream()获取网络字节输出流OutputStream对象
        6.使用网络字节输出流OutputStream对象中的方法write,给客户端回写数据
        7.释放资源(Socket,ServerSocket)

```Java
public static void main(String[] args) throws IOException {
        //1.创建服务器ServerSocket对象和系统要指定的端口号
        ServerSocket server = new ServerSocket(8888);
        //2.使用ServerSocket对象中的方法accept,获取到请求的客户端对象Socket
        Socket socket = server.accept();
        //3.使用Socket对象中的方法getInputStream()获取网络字节输入流InputStream对象
        InputStream is = socket.getInputStream();
        //4.使用网络字节输入流InputStream对象中的方法read,读取客户端发送的数据
        byte[] bytes = new byte[1024];
        int len = is.read(bytes);
        System.out.println(new String(bytes,0,len));
        //5.使用Socket对象中的方法getOutputStream()获取网络字节输出流OutputStream对象
        OutputStream os = socket.getOutputStream();
        //6.使用网络字节输出流OutputStream对象中的方法write,给客户端回写数据
        os.write("收到谢谢".getBytes());
        //7.释放资源(Socket,ServerSocket, IO流)
    	os.close();
    	is.close();
        socket.close();
        server.close();
}
```

#### 3. TCP通信分析图解

1. 【服务端】启动,创建ServerSocket对象，等待连接。
2. 【客户端】启动,创建Socket对象，请求连接。
3. 【服务端】接收连接,调用accept方法，并返回一个Socket对象。
4. 【客户端】Socket对象，获取OutputStream，向服务端写出数据。
5. 【服务端】Scoket对象，获取InputStream，读取客户端发送的数据。

> 到此，客户端向服务端发送数据成功。

![1557152847708](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557152847708.png)

> 自此，服务端向客户端回写数据。

6. 【服务端】Socket对象，获取OutputStream，向客户端回写数据。
7. 【客户端】Scoket对象，获取InputStream，解析回写数据。
8. 【客户端】释放资源，断开连接。

#### 4. shutdownInput()与shutdownOutput()

1. 在客户端或者服务端通过socket.shutdownOutput()都是单向关闭的，即关闭客户端的输出流并不会关闭服务端的输出流，所以是一种单方向的关闭流；
2. 通过socket.shutdownOutput()关闭输出流，但socket仍然是连接状态，连接并未关闭
3. **如果直接关闭输入或者输出流，即：in.close()或者out.close()，会直接关闭Socket**

### 5. 网络编程案例

#### 1. 文件上传

1. 【客户端】输入流，从硬盘读取文件数据到程序中。
2. 【客户端】输出流，写出文件数据到服务端。
3. 【服务端】输入流，读取文件数据到服务端程序。
4. 【服务端】输出流，写出文件数据到服务器硬盘中。
5. 【服务端】获取输出流，回写数据。
6. 【客户端】获取输入流，解析回写数据。

![1557294559540](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557294559540.png)

##### 客户端: 

```Java
package cn.demo.netsocket.netsockettwo;

import java.io.*;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Create by Jjcc on 2019/5/12 20:32
 *
 * @author Jjcc
 */
public class FileUploadClient {

    public static void main(String[] args) throws IOException {
        //创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        System.getProperty("user.dir");
		//创建Runnable实现类实例对象
        ThreadTaskClient threadTaskClient = new ThreadTaskClient();

        //开启线程
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
        executorService.submit(threadTaskClient);
    }
}

class ThreadTaskClient implements Runnable {
    @Override
    public void run() {
        /**
         * 创建客户端对象Socket,构造方法绑定IP, 端口
         * 创建缓存文件字节输入流
         * 创建缓存网络字节输入流
         * 创建缓存网络字节输出流
         */
        try (Socket socket = new Socket("localhost", 11111);
             OutputStream bufferedOutputStream = new BufferedOutputStream(socket.getOutputStream());
             InputStream bufferedInputStream = new BufferedInputStream(socket.getInputStream());
             InputStream bufferedFileInputStream = new BufferedInputStream(new FileInputStream("biglan.jpg"));
            ) {

            byte[] bytes = new byte[1024 * 8];
            int len1;
            //通过文件字节缓存输入流读取文件到内存中
            while ((len1 = bufferedFileInputStream.read(bytes)) != -1) {
                //将内存中的数据写入到网络字节缓存输出流中
                bufferedOutputStream.write(bytes, 0, len1);
                //把缓冲区中的缓存刷新到其他设备中
                bufferedOutputStream.flush();
            }
            //关闭socket对象的输出流, 让服务端知道文件传输完毕
            socket.shutdownOutput();

            System.out.println("文件上传完毕----------");

            //获取服务端回写的数据
            byte[] bytes2 = new byte[1024  * 8];
            int len2 = bufferedInputStream.read(bytes2);
            System.out.println(new String(bytes2, 0, len2));

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

##### 服务端: 

```Java
package cn.demo.netsocket.netsockettwo;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.UUID;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Create by Jjcc on 2019/5/12 20:33
 *
 * @author Jjcc
 */
public class FileUploadServer {

    public static void main(String[] args) throws IOException {
        //创建服务器端对象ServerSocket, 构造方法绑定port
        ServerSocket serverSocket = new ServerSocket(11111);
        //创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        while (true) {
            //通过accept()方法拿到客户端对象Socket
            Socket accept = serverSocket.accept();

            //创建Runnable实现类实例对象
            //开启线程
            executorService.submit(new ThreadTasksServer(accept));
        }

    }
}

class ThreadTasksServer implements Runnable {

    private Socket socket;

    public ThreadTasksServer(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        //创建文件对象,使用UUID为文件名
        String uuid = UUID.randomUUID().toString().replace("-", "");
        String fileName = "E:\\软件开发资料\\files\\";
        File file = new File(fileName);
        //文件地址是否存在, 不存在则创建地址
        if (!file.exists()) {
            file.mkdirs();
        }

        /**
         * 创建文件字节缓存输出流
         * 创建网络字节缓存输入流
         * 创建网络字节缓存输出流
         * 在try () 中创建流对象等, 不需要手动close()
         */
        try (InputStream bufferedInputStream = new BufferedInputStream(socket.getInputStream());
             OutputStream bufferedOutputStream = new BufferedOutputStream(socket.getOutputStream());
             OutputStream bufferedFileOutputStream = new BufferedOutputStream(new FileOutputStream(fileName + uuid + ".jpg"));
             ) {

            byte[] bytes1 = new byte[1024 * 8];
            int len1;
            while ((len1 = bufferedInputStream.read(bytes1)) != -1) {
                bufferedFileOutputStream.write(bytes1, 0 , len1);
                bufferedFileOutputStream.flush();
            }
            System.out.println("文件保存完毕----------");

            //服务端回写信息给客户端
            bufferedOutputStream.write("文件上传成功!!!!!!!".getBytes());
            bufferedOutputStream.flush();

            socket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 2. 模拟B/S服务器

##### 服务器程序中字节输入流可以读取到浏览器发来的请求信息

![1557757944159](E:\软件开发资料\Java\Java资料\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557757944159.png)

GET/web/index.html HTTP/1.1是浏览器的请求消息。/web/index.html为浏览器想要请求的服务器端的资源,使用字符串切割方式获取到请求的资源。

```Java
//转换流,读取浏览器请求第一行
BufferedReader readWb = new BufferedReader(new InputStreamReader(socket.getInputStream()));
String requst = readWb.readLine();
//取出请求资源的路径
String[] strArr = requst.split(" ");
//去掉web前面的/
String path = strArr[1].substring(1);
System.out.println(path);
```

##### B/S架构图解

![1557758139672](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1557758139672.png)

> 小贴士：不同的浏览器，内核不一样，解析效果有可能不一样。

发现浏览器中出现很多的叉子,说明浏览器没有读取到图片信息导致。

浏览器工作原理是遇到图片会开启一个线程进行单独的访问,因此在服务器端加入线程技术。

```Java
package cn.demo.netsocket.bsfremework;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Create by Jjcc on 2019/5/13 21:53
 *
 * @author Jjcc
 */
public class TCPServerTwo {

    public static void main(String[] args) throws IOException {
        //创建服务端对象ServerSocket,构造方法绑定端口号
        ServerSocket serverSocket = new ServerSocket(11111);
        System.getProperty("user.dir");
        //创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        while (true) {
            //通过accpet()方法拿到客户端对象Socket
            Socket accept = serverSocket.accept();
            //创建Runnable实现类的实例对象
            ThreadTask threadTask = new ThreadTask(accept);
            //将Runnable实现类的实例对象放进线程池, 开启线程
            executorService.submit(threadTask);
        }
    }
}

class ThreadTask implements Runnable {
    
    private Socket socket;

    public ThreadTask(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        /**
         * 创建缓存字符输入流
         * 创建缓存字节输出流
         */
        try (BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             OutputStream bufferedOutputStream = new BufferedOutputStream(socket.getOutputStream());) {

            //获取web请求第一行信息
            String readLine = bufferedReader.readLine();
            if (null != readLine) {
                //获取访问地址
                String[] split = readLine.split(" ");
                String resourcePath = split[1].substring(1);
                System.out.println("resourcePath: " + resourcePath);
                //缓存文件字节输入流
                InputStream bufferedFileInputStream = new BufferedInputStream(new FileInputStream(resourcePath));

                // 写入HTTP协议响应头,固定写法
                bufferedOutputStream.write("HTTP/1.1 200 OK\r\n".getBytes());
                bufferedOutputStream.write("Content-Type:text/html\r\n".getBytes());
                // 必须要写入空行,否则浏览器不解析
                bufferedOutputStream.write("\r\n".getBytes());

                byte[] bytes = new byte[1024 * 8];
                int len;
                //通过缓存文件字节输入流读取硬盘中的文件到内存中
                while ((len = bufferedFileInputStream.read(bytes)) != -1) {
                    //服务端通过客户端的网络字节输出流回写数据给浏览器
                    bufferedOutputStream.write(bytes, 0 , len);
                    //把缓冲区中的缓存刷新到浏览器
                    bufferedOutputStream.flush();
                }
                //关闭缓存文件字节输入流
                bufferedFileInputStream.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        
    }
}
```



## 八. 函数式编程

### 1. 函数式接口

函数式接口在Java中是指：**有且仅有一个抽象方法的接口。**

函数式接口，即适用于函数式编程场景的接口。而Java中的函数式编程体现就是Lambda，所以函数式接口就是可
以适用于Lambda使用的接口。只有确保接口中有且仅有一个抽象方法，Java中的Lambda才能顺利地进行推导。

**lambad与匿名内部类的区别: **使用匿名内部类,在编译后会生成一个类文件, 而lambda不会创建新的类文件; 既使用lambda不会创建多余的类文件, 减少内存消耗

#### 1. 格式

只要确保接口中有且仅有一个抽象方法即可(**jdk1.8开始,接口可以有静态方法, default修饰的方法**)：

```Java
修饰符 interface 接口名称 {
public abstract 返回值类型 方法名称(可选参数信息);
    // 其他非抽象方法内容
}
```

#### 2. @FunctionalInterface

与 `@Override` 注解的作用类似，Java 8中专门为函数式接口引入了一个新的注解： @FunctionalInterface 。该注
解可用于一个接口的定义上：

```Java
@FunctionalInterface
public interface MyFunctionalInterface {
	void myMethod();    
}
```

一旦使用该注解来定义接口，**编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错**。需要注意的是，即使不使用该注解，**只要满足函数式接口的定义，这仍然是一个函数式接口，使用起来都一样。**

#### 3. 自定义函数式接口

```Java
public class Demo09FunctionalInterface {   
    // 使用自定义的函数式接口作为方法参数    
    private static void doSomething(MyFunctionalInterface inter) {    
    	inter.myMethod(); // 调用自定义的函数式接口方法        
    }    
   
    public static void main(String[] args) {    
    	// 调用使用函数式接口的方法        
    	doSomething(() ‐> System.out.println("Lambda执行啦！"));   
        
        //使用函数式接口带参数的方法
        //doSomething(123, (i) -> {
            //System.out.println("Hello World!!!" + i);
        //});
    }    
}
```

### 2. 函数式编程

#### 1. Lambda的延迟执行

有些场景的代码执行后，结果不一定会被使用，从而造成性能浪费。而Lambda表达式是延迟执行的，这正好可以
作为解决方案，提升性能。

```Java
public class Demo01Logger {
    private static void log(int level, String msg) {
        if (level == 1) {
        	System.out.println(msg);
        }
    }
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        String msgC = "Java";
        log(1, msgA + msgB + msgC);
    }
}
```

这段代码存在问题：无论级别是否满足要求，作为 log 方法的第二个参数，三个字符串一定会首先被拼接并传入方法内，然后才会进行级别判断。如果级别不符合要求，那么字符串的拼接操作就白做了，存在性能浪费。

> 备注：SLF4J是应用非常广泛的日志框架，它在记录日志时为了解决这种性能浪费的问题，并不推荐首先进行
> 字符串的拼接，而是将字符串的若干部分作为可变参数传入方法中，仅在日志级别满足要求的情况下才会进
> 行字符串拼接。例如： LOGGER.debug("变量{}的取值为{}。", "os", "macOS") ，其中的大括号 {} 为占位
> 符。如果满足日志级别要求，则会将“os”和“macOS”两个字符串依次拼接到大括号的位置；否则不会进行字
> 符串拼接。这也是一种可行解决方案，但Lambda可以做到更好。

##### **体验Lambda的更优写法:**

使用Lambda必然需要一个函数式接口：

```Java
@FunctionalInterface
public interface MessageBuilder {
	String buildMessage();
}
```

然后对 log 方法进行改造：

```Java
public class Demo02LoggerLambda {
    private static void log(int level, MessageBuilder builder) {
        if (level == 1) {
        System.out.println(builder.buildMessage());
    }
    
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        String msgC = "Java";
        log(1, () ‐> msgA + msgB + msgC );
    }
}
```

这样一来，**只有当级别满足要求的时候，才会进行三个字符串的拼接；否则三个字符串将不会进行拼接。**

> 扩展：实际上使用内部类也可以达到同样的效果，只是将代码操作延迟到了另外一个对象当中通过调用方法
> 来完成。而是否调用其所在方法是在条件判断之后才执行的。

### 2. 使用Lambda作为参数和返回值

如果抛开实现原理不说，Java中的Lambda表达式可以被当作是匿名内部类的替代品。如果方法的参数是一个函数式接口类型，那么就可以使用Lambda表达式进行替代。使用Lambda表达式作为方法参数，其实就是使用函数式接口作为方法参数。

例如 java.lang.Runnable 接口就是一个函数式接口，假设有一个 startThread 方法使用该接口作为参数，那么就可以使用Lambda进行传参。这种情况其实和 Thread 类的构造方法参数为 Runnable 没有本质区别。

类似地，**如果一个方法的返回值类型是一个函数式接口，那么就可以直接返回一个Lambda表达式。**当需要通过一个方法来获取一个 java.util.Comparator 接口类型的对象作为排序器时,就可以调该方法获取。

```Java
import java.util.Arrays;
import java.util.Comparator;
public class Demo06Comparator {
    private static Comparator<String> newComparator() {
    	return (a, b) ‐> b.length() ‐ a.length();
    }
    public static void main(String[] args) {
    	String[] array = { "abc", "ab", "abcd" };
    	System.out.println(Arrays.toString(array));
    	Arrays.sort(array, newComparator());
    	System.out.println(Arrays.toString(array));
    }
}
```

其中直接return一个Lambda表达式即可.

### 3. 常用函数式接口

#### 1. Supplier

`java.util.function.Supplier<T>` 接口仅包含一个无参的方法： `T get()` 。**用来获取一个泛型参数指定类型的对象数据。**由于这是一个函数式接口，这也就意味着对应的Lambda表达式需要“对外提供”一个符合泛型类型的对象数据。

**生产型接口**: 指定接口泛型的类型, 就可以使用`get()`方法生产什么类型的数据

```Java
package cn.demo.functionalinterface;

import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.function.Supplier;

/**
 * Create by Jjcc on 2019/5/16 21:07
 *
 * @author Jjcc
 */
public class SupplierFunctionDemoOne {

    public static void main(String[] args){
        Integer[] arr = {2,3,4,52,333,23};

        Supplier supplier = supplierMethodOne(arr);
        Object o = supplier.get();
        System.out.println(o);

        int i = SupplierMethodTwo(() -> {
            Arrays.sort(arr, (a, b) -> {
                return b - a;
            });

            return arr[0];
        });
        System.out.println(i);
    }

    /**
     * 生产型接口: 指定接口泛型的类型,就可以使用get()方法生产什么类型的数据
     * 使用Lambda作为返回值
     * @param
     * @return
     */
    public static Supplier<Integer> supplierMethodOne(Integer[] arr) {
        return () -> {
            int max = arr[0];
            for (int i : arr) {
                if (max < i) {
                    max = i;
                }
            }
            return max;
        };
    }

    /**
     * 使用Lambda作为参数
     * @param supplier
     * @return
     */
    public static int SupplierMethodTwo(Supplier<Integer> supplier) {
        return supplier.get();
    }
}
```

#### 2. Consumer

`java.util.function.Consumer<T>` 接口则正好与Supplier接口相反，它不是生产一个数据，而是消费一个数据，其数据类型由泛型决定。

**消费型接口: **指定接口泛型的类型, 就可以使用`accept()`方法消费什么类型的数据

##### 抽象方法:  void accep(T t)

`Consumer` 接口中包含抽象方法 `void accept(T t)` ，意为消费一个指定泛型的数据。

```Java
package cn.demo.functionalinterface;

import java.util.function.Consumer;

/**
 * Create by Jjcc on 2019/5/16 21:55
 *
 * @author Jjcc
 */
public class ConsumerFunctionDemoOne {

    public static void main(String[] args){
        ConsumerMethodOne("Hello World!!!", (str) -> {
            System.out.println(str);
        });

        Consumer stringConsumer = ConsumerMethodTwo();
        stringConsumer.accept("Thanks!!!");
    }

    /**
     * 消费型接口: 泛型是什么类型, 就可以使用accept方法消费什么类型的数据
     * Lambda作为参数
     * @param s
     * @param consumer
     */
    public static void ConsumerMethodOne(String s, Consumer<String> consumer) {
        consumer.accept(s);
    }

    /**
     * Lambda作为返回值
     * @return
     */
    public static Consumer ConsumerMethodTwo() {
        return (a) -> {
            System.out.println(a + ".......");
        };
    }
}
```

**更好的写法是使用方法引用。**

##### 默认方法: andThen(Consumer<T> c1, Consumer<T> c2)

**andThen:** 需要两个Consumer接口,可以把两个Consumer组合在一起, 在对数据消费

如果一个方法的参数和返回值全都是 Consumer 类型，那么就可以实现效果：消费数据的时候，首先做一个操作，然后再做一个操作，实现组合。而这个方法就是 Consumer 接口中的default方法 andThen 。下面是JDK的源代码：

```Java
default Consumer<T> andThen(Consumer<? super T> after) {
	Objects.requireNonNull(after);
	return (T t) ‐> { accept(t); after.accept(t); };
}
```

> 备注： java.util.Objects 的 requireNonNull 静态方法将会在参数为null时主动抛出NullPointerException 异常。这省去了重复编写if语句和抛出空指针异常的麻烦。
>

要想实现组合，需要两个或多个Lambda表达式即可，而 andThen 的语义正是“一步接一步”操作。例如两个步骤组合的情况：

```Java
package cn.demo.functionalinterface;

import java.util.function.Consumer;

/**
 * Create by Jjcc on 2019/5/16 22:46
 *
 * @author Jjcc
 */
public class ConsumerAndThenDemoOne {
    public static void main(String[] args){

        //andThen: 需要两个Consumer接口,可以把两个Consumer组合在一起,在对数据消费
        ConsumerMethodOne().andThen(ConsumerMethodTwo()).accept("Hello!!!");

        ConsumerMethodThree("Hello", (a) -> {
                                System.out.println(a.toUpperCase());
                            },
                            a -> {
                                System.out.println(a.toLowerCase());
                            }
                            );
    }

    /**
     * Lambda作为返回值
     * @return
     */
    public static Consumer<String> ConsumerMethodOne() {
        return (a) -> {
            System.out.println(a.toUpperCase());
        };
    }
    public static Consumer<String> ConsumerMethodTwo() {
        return (a) -> {
            System.out.println(a.toLowerCase());
        };
    }
    
	/**
     * Lambda作为参数
     * @param str
     * @param c1
     * @param c2
     */
    public static void ConsumerMethodThree(String str, Consumer<String> c1, Consumer<String> c2) {
        c1.andThen(c2).accept(str);
    }


}
```

**结果: **

```
HELLO!!!
hello!!!
HELLO
hello
```

```Java
package cn.demo.functionalinterface;

import java.util.Arrays;
import java.util.function.Consumer;

/**
 * Create by Jjcc on 2019/5/18 15:31
 *
 * @author Jjcc
 */
public class ConsumerFunctionDemoTwo {

    public static void main(String[] args){
        String[] array = { "迪丽热巴,女", "古力娜扎,女", "马尔扎哈,男" };

        consumerAndThenMethod(array, consumerMethodOne(), consumerMethodTwo());

        consumerAndThenMethod(array,
                (i) -> {
                    System.out.print ("姓名: " + i.split(",")[0]);
                },
                (i) -> {
                    System.out.println (". 性别: " + i.split(",")[1]);
                });
    }

    public static Consumer<String> consumerMethodOne() {
        return (str) -> {
            String[] split = str.split(",");
            System.out.print("姓名: " + split[0]);
        };
    }

    public static Consumer<String> consumerMethodTwo() {
        return (str) -> {
            String[] split = str.split(",");
            System.out.println(". 性别: " + split[1]);
        };
    }

    public static void consumerAndThenMethod(String[] array, Consumer<String> c1, Consumer<String> c2) {
        Arrays.stream(array).forEach( (i) ->{
            c1.andThen(c2).accept(i);
        });

    }
}

```

```
姓名: 迪丽热巴. 性别: 女
姓名: 古力娜扎. 性别: 女
姓名: 马尔扎哈. 性别: 男

姓名: 迪丽热巴. 性别: 女
姓名: 古力娜扎. 性别: 女
姓名: 马尔扎哈. 性别: 男
```

### 3. Predicate

有时候我们需要对某种类型的数据进行判断，从而得到一个boolean值结果。这时可以使用
java.util.function.Predicate<T> 接口

#### 抽象方法：test

Predicate 接口中包含一个抽象方法： boolean test(T t) 。用于条件判断的场景：

```java
import java.util.function.Predicate;
public class Demo15PredicateTest {
    private static void method(Predicate<String> predicate) {
        boolean veryLong = predicate.test("HelloWorld");
        System.out.println("字符串很长吗：" + veryLong);
    }
    public static void main(String[] args) {
        method(s ‐> s.length() > 5);
    }
}
```

条件判断的标准是传入的Lambda表达式逻辑，只要字符串长度大于5则认为很长。

#### 默认方法：and

既然是条件判断，就会存在与、或、非三种常见的逻辑关系。其中将两个 `Predicate` 条件使用“与”逻辑连接起来实
现“并且”的效果时，可以使用default方法 `and` 。其JDK源码为：

```java
default Predicate<T> and(Predicate<? super T> other) {
	Objects.requireNonNull(other);
	return (t) ‐> test(t) && other.test(t);
}
```

如果要判断一个字符串既要包含大写“H”，又要包含大写“W”，那么：

```Java
import java.util.function.Predicate;
public class Demo16PredicateAnd {
    private static void method(Predicate<String> one, Predicate<String> two) {
    	boolean isValid = one.and(two).test("Helloworld");
    	System.out.println("字符串符合要求吗：" + isValid);
    }
    public static void main(String[] args) {
    	method(s ‐> s.contains("H"), s ‐> s.contains("W"));
    }
}
```

#### 默认方法：or

与 `and` 的“与”类似，默认方法 `or` 实现逻辑关系中的“或”。JDK源码为

```java
default Predicate<T> or(Predicate<? super T> other) {
	Objects.requireNonNull(other);
	return (t) ‐> test(t) || other.test(t);
}
```

如果希望实现逻辑“字符串包含大写H或者包含大写W”，那么代码只需要将“and”修改为“or”名称即可，其他都不
变：

```java
import java.util.function.Predicate;
public class Demo16PredicateAnd {
	private static void method(Predicate<String> one, Predicate<String> two) {
		boolean isValid = one.or(two).test("Helloworld");
		System.out.println("字符串符合要求吗：" + isValid);
	}
    public static void main(String[] args) {
    	method(s ‐> s.contains("H"), s ‐> s.contains("W"));
    }
}
```

#### 默认方法：negate

“与”、“或”已经了解了，剩下的“非”（取反）也会简单。默认方法 `negate` 的JDK源代码为：

```java
default Predicate<T> negate() {
	return (t) ‐> !test(t);
}
```

从实现中很容易看出，它是执行了test方法之后，对结果boolean值进行“!”取反而已。一定要在 test 方法调用之前调用 negate 方法，正如 and 和 or 方法一样：

```java
import java.util.function.Predicate;
public class Demo17PredicateNegate {
    private static void method(Predicate<String> predicate) {
    	boolean veryLong = predicate.negate().test("HelloWorld");
    	System.out.println("字符串很长吗：" + veryLong);
    }
    public static void main(String[] args) {
    	method(s ‐> s.length() < 5);
    }
}
```



```java
package cn.demo.functionalinterface;

import java.util.function.Predicate;

/**
 * Create by Jjcc on 2019/5/18 16:10
 *
 * @author Jjcc
 * Predicate: 对某种数据类型的数据进行判断, 返回一个boolean值
 */
public class PredicateFunctionDemoOne {
    public static void main(String[] args){
        String str = "ss";
        predicateTestMethodOne(str, (string) ->
          string.length() > 5
        );

        predicateAndMethodOne(str, (a) -> a.length() > 15, (a) -> a.contains("a"));

        predicateOrMethodOne(str, (a) -> a.contains("b"), (a) -> a.contains("as"));

        predicateNegateMethodOne(str, (a) -> a.length() > 5);
    }

    /**
     * test(): 用来对指定数据类型的数据进行判断的方法
     * @param str
     * @param predicate
     */
    public static void predicateTestMethodOne(String str, Predicate<String> predicate) {
        System.out.println("字符串长度是否大于5: " + predicate.test(str));
    }

    /**
     * and(): 将两个 Predicate 条件使用“与”逻辑连接起来实现“并且”的效果
     * @param str
     * @param predicate1
     * @param predicate2
     */
    public static void predicateAndMethodOne(String str, Predicate<String> predicate1, Predicate<String> predicate2) {
        System.out.println("AND()结果: " + predicate1.and(predicate2).test(str));
    }

    /**
     * or(): 将两个 Predicate 条件使用 "或"逻辑连接起来实现"或者"的效果
     * @param str
     * @param predicate1
     * @param predicate2
     */
    public static void predicateOrMethodOne(String str, Predicate<String> predicate1, Predicate<String> predicate2) {
        System.out.println("OR()结果: " + predicate1.or(predicate2).test(str));
    }

    /**
     * negate(): 执行了test方法之后，对结果boolean值进行“!”取反而已
     * @param str
     * @param predicate1
     */
    public static void predicateNegateMethodOne(String str, Predicate<String> predicate1) {
        System.out.println("Negate: " + predicate1.negate().test(str));
    }
}

```

### 4. Function

`java.util.function.Function<T, R>` 用来根据一个类型的数据得到另一个类型的数据,前者称为apply()参数类型,后者称apply()为返回类型.

#### 抽象方法：apply

Function 接口中最主要的抽象方法为： R apply(T t) ，根据类型T的参数获取类型R的结果。
使用的场景例如：将 String 类型转换为 Integer 类型。

```java
import java.util.function.Function;
public class Demo11FunctionApply {
    private static void method(Function<String, Integer> function) {
    	int num = function.apply("10");
    	System.out.println(num + 20);
    }
    public static void main(String[] args) {
    	method(s ‐> Integer.parseInt(s));
    }
}
```

**最好是通过方法引用的写法。**

#### 默认方法：andThen

`Function` 接口中有一个默认的 andThen 方法，用来进行组合操作。JDK源代码如：

```Java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
	Objects.requireNonNull(after);
	return (T t) ‐> after.apply(apply(t));
}	
```

该方法同样用于“先做什么，再做什么”的场景，和 Consumer 中的 andThen 差不多：

```java
import java.util.function.Function;
    public class Demo12FunctionAndThen {
    private static void method(Function<String, Integer> one, Function<Integer, Integer> two) {
    	int num = one.andThen(two).apply("10");
    	System.out.println(num + 20);
    }
    public static void main(String[] args) {
    	method(str‐>Integer.parseInt(str)+10, i ‐> i *= 10);
    }
}
```

> 请注意，Function的前置条件泛型和后置条件泛型可以相同。

```java
package cn.demo.functionalinterface;

import java.util.function.BiFunction;
import java.util.function.Function;

/**
 * Create by Jjcc on 2019/5/19 16:46
 *
 * @author Jjcc
 * Function<T, R>: 用来根据一个类型的数据得到另一个类型的数据,前者称为apply()参数类型,后者称apply()为返回类型
 */
public class FunctionInteFunctionDemoOne {
    public static void main(String[] args){
        int a = functionMethodOne("10", i -> Integer.parseInt(i));
        System.out.println(a*10);

        functionAndthenMethodOne("10", i ->  Integer.parseInt(i) + 10, i ->  i *10);

        functionComposeMethodOne("10", i -> String.valueOf(i), i -> Integer.parseInt(i) * 10);
    }

    /**
     * apple(T t): 根据类型T的参数获取类型R的结果
     * @param str
     * @param function
     * @return
     */
    public static Integer functionMethodOne(String str, Function<String, Integer> function) {
        return function.apply(str);
    }

    /**
     * andThen(Function<T t, R r> f): 该方法需要两个function接口,调用方法的接口的抽象方法第一个执行,
     * 参数接口的抽象方法第二个执行,且第一个执行的apply()方法的返回值为第二个apply()方法的参数
     * @param str
     * @param function1
     * @param function2
     */
    public static void functionAndthenMethodOne(String str, Function<String, Integer> function1, Function<Integer, Integer> function2) {
        System.out.println(function1.andThen(function2).apply(str));
    }

    /**
     * compose(Function<T t, R r> f): 与andThen执行次序相反
     * @param str
     * @param function1
     * @param function2
     */
    public static void functionComposeMethodOne(String str, Function<Integer, String> function1, Function<String, Integer> function2) {
        System.out.println(function1.compose(function2).apply(str));
    }
}
```

## 九. Stream流

当需要对多个元素进行操作（特别是多步操作）的时候，考虑到性能及便利性，我们应该首先拼好一个“模型”步骤
方案，然后再按照方案去执行它

![1558276512362](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1558276512362.png)

这张图中展示了过滤、映射、跳过、计数等多步操作，这是一种集合元素的处理方案，而方案就是一种“函数模型”。图中的每一个方框都是一个“流”，调用指定的方法，可以从一个流模型转换为另一个流模型。而最右侧的数字3是最终结果

这里的 `filter` 、 `map` 、 `skip` 都是在对函数模型进行操作，集合元素并没有真正被处理。**只有当终结方法 count执行的时候，整个模型才会按照指定策略执行操作。而这得益于Lambda的延迟执行特性。**

> 备注：“Stream流”其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其本身并不存储任何元素（或其地址值）。

**Stream（流）是一个来自数据源的元素队列**

- 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算
- **数据源** 流的来源。 可以是集合，数组 等。

**和以前的Collection操作不同， Stream操作还有两个基础的特征：**

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent
  style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代**： 以前对集合遍历都是通过Iterator或者增强for的方式, 显式的在集合外部进行迭代， 这叫做外部迭
  代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。

当使用一个流的时候，通常包括三个基本步骤：获取一个数据源（source）→ 数据转换→执行操作获取想要的结
果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以
像链条一样排列，变成一个管道。

### 1. 获取流

`java.util.stream.Stream<T>` 是Java 8新加入的最常用的流接口。（这并不是一个函数式接口。）获取一个流非常简单，有以下几种常用的方式：

- 所有的 Collection 集合都可以通过 stream 默认方法获取流；
- Stream 接口的静态方法 `of` 可以获取数组对应的流。

#### 根据Collection获取流

首先， `java.util.Collection` 接口中加入了default方法 stream 用来获取流，所以其所有实现类均可获取流。

```java
import java.util.*;
import java.util.stream.Stream;
public class Demo04GetStream {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        // ...
        Stream<String> stream1 = list.stream();
        Set<String> set = new HashSet<>();
        // ...
        Stream<String> stream2 = set.stream();
        Vector<String> vector = new Vector<>();
        // ...
        Stream<String> stream3 = vector.stream();
    }
}
```

#### 根据Map获取流

`java.util.Map` 接口不是 Collection 的子接口，且其K-V数据结构不符合流元素的单一特征，所以获取对应的流需要分key、value或entry等情况：

```java
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Stream;
public class Demo05GetStream {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        // ...
        Stream<String> keyStream = map.keySet().stream();
        Stream<String> valueStream = map.values().stream();
        Stream<Map.Entry<String, String>> entryStream = map.entrySet().stream();
    }
}
```

#### 根据数组获取流

如果使用的不是集合或映射而是数组，由于数组对象不可能添加默认方法，所以 Stream 接口中提供了静态方法
of ，使用很简单：

```java
import java.util.stream.Stream;
public class Demo06GetStream {
    public static void main(String[] args) {
    	String[] array = { "张无忌", "张翠山", "张三丰", "张一元" };
    	Stream<String> stream = Stream.of(array);
    }
}
```

> 备注： `of` 方法的参数其实是一个可变参数，所以支持数组。
>

### 2. 常用方法	

流模型的操作很丰富，这里介绍一些常用的API。这些方法可以被分成两种：

- **延迟方法**：返回值类型仍然是 Stream 接口自身类型的方法，因此支持链式调用。（除了终结方法外，其余方
  法均为延迟方法。）
- **终结方法**：返回值类型不再是 Stream 接口自身类型的方法，因此**不再支持类似 StringBuilder 那样的链式调**
  **用**。本小节中，终结方法包括 count 和 forEach 方法。

#### 逐一处理：forEach

虽然方法名字叫 forEach ，但是与for循环中的“for-each”昵称不同。

```java
void forEach(Consumer<? super T> action);
```

该方法接收一个 `Consumer` 接口函数，会将每一个流元素交给该函数进行处理。

#### 过滤：filter

可以通过 `filter` 方法将一个流转换成另一个子集流。方法签名：

```java
Stream<T> filter(Predicate<? super T> predicate);
```

该接口接收一个 `Predicate` 函数式接口参数（可以是一个Lambda或方法引用）作为筛选条件。

该方法将会产生一个boolean值结果，代表指定的条件是否满足。如果结果为true，那么Stream流的 filter 方法将会留用元素；如果结果为false，那么 `filter` 方法将会舍弃元素

#### 映射：map

如果需要将流中的元素映射到另一个流中，可以使用 map 方法。方法签名：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

该接口需要一个 `Function` 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

这可以将一种T类型转换成为R类型，而这种转换的动作，就称为“**映射**”。

#### 统计个数：count

正如旧集合 Collection 当中的 size 方法一样，流提供 count 方法来数一数其中的元素个数：

```java
long count();
```

该方法返回一个long值代表元素个数（不再像旧集合那样是int值）。

```java
import java.util.stream.Stream;
public class Demo09StreamCount {
    public static void main(String[] args) {
    	Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若");
    	Stream<String> result = original.filter(s ‐> s.startsWith("张"));
    	System.out.println(result.count()); // 2
    }
}
```

#### 取用前几个：limit

`limit` 方法可以对流进行截取，只取用前n个。方法签名：

```java
Stream<T> limit(long maxSize);
```

参数是一个long型，**如果集合当前长度大于参数则进行截取；否则不进行操作。**

#### 跳过前几个：skip

如果希望跳过前几个元素，可以使用 `skip` 方法获取一个截取之后的新流：

```java
Stream<T> skip(long n);
```

如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。

#### 组合：concat

如果有两个流，希望合并成为一个流，那么可以使用 `Stream` 接口的静态方法 `concat` ：

```
static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
```

> 备注：这是一个静态方法，与 java.lang.String 当中的 `concat` 方法是不同的。

**该方法的基本使用代码如：**

```java
import java.util.stream.Stream;
public class Demo12StreamConcat {
    public static void main(String[] args) {
    	Stream<String> streamA = Stream.of("张无忌");
    	Stream<String> streamB = Stream.of("张翠山");
    	Stream<String> result = Stream.concat(streamA, streamB);
    }
}
```

#### Demo:

```Java
package cn.demo.stream;

import java.util.ArrayList;
import java.util.stream.Stream;

/**
 * Create by Jjcc on 2019/5/20 22:09
 *
 * @author Jjcc
 * 1. 第一个队伍只要名字为3个字的成员姓名；存储到一个新集合中。
 * 2. 第一个队伍筛选之后只要前3个人；存储到一个新集合中。
 * 3. 第二个队伍只要姓张的成员姓名；存储到一个新集合中。
 * 4. 第二个队伍筛选之后不要前2个人；存储到一个新集合中。
 * 5. 将两个队伍合并为一个队伍；存储到一个新集合中。
 * 6. 根据姓名创建 Person 对象；存储到一个新集合中。
 * 7. 打印整个队伍的Person对象信息。
 */
public class StreamDemoTwo {
    public static void main(String[] args) {
        //第一支队伍
        ArrayList<String> one = new ArrayList<>();
        one.add("迪丽热巴");
        one.add("宋远桥");
        one.add("苏星河");
        one.add("石破天");
        one.add("石中玉");
        one.add("老子");
        one.add("庄子");
        one.add("洪七公");

        //第二支队伍
        ArrayList<String> two = new ArrayList<>();
        two.add("古力娜扎");
        two.add("张无忌");
        two.add("赵丽颖");
        two.add("张三丰");
        two.add("尼古拉斯赵四");
        two.add("张天爱");
        two.add("张二狗");

        // 第一个队伍只要名字为3个字的成员姓名；
        // 第一个队伍筛选之后只要前3个人；
        Stream<String> streamOne = one.stream().filter(s -> s.length() == 3).limit(3);
        // 第二个队伍只要姓张的成员姓名；
        // 第二个队伍筛选之后不要前2个人；
        Stream<String> streamTwo = two.stream().filter(s -> s.startsWith("张")).skip(2);
        // 将两个队伍合并为一个队伍；
        // 根据姓名创建Person对象；
        // 打印整个队伍的Person对象信息。
        Stream.concat(streamOne, streamTwo).map(Person::new).forEach(System.out::println);

    }
}

class Person {
    private String name;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "'}";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```



## 十. 方法引用

### 1.方法引用符

双冒号 `::` 为引用运算符，而它所在的表达式被称为方法引用。**如果Lambda要表达的函数方案已经存在于某个方法的实现中，那么则可以通过双冒号来引用该方法作为Lambda的替代者。**

#### 1. 语义分析

例如上例中， `System.out` 对象中有一个重载的 `println(String)` 方法恰好就是我们所需要的。那么对于printString 方法的函数式接口参数，对比下面两种写法，完全等效：

- Lambda表达式写法： `s -> System.out.println(s);`
- 方法引用写法： `System.out::printl`

第一种语义是指：拿到参数之后经Lambda之手，继而传递给 `System.out.println` 方法去处理。

第二种等效写法的语义是指：直接让 `System.out` 中的 `println` 方法来取代Lambda。两种写法的执行效果完全一样，而第二种方法引用的写法复用了已有方案，更加简洁。

**注:Lambda 中 传递的参数 一定是方法引用中 的那个方法可以接收的类型,否则会抛出异常**

#### 2. 推导与省略

如果使用Lambda，那么根据“**可推导就是可省略**”的原则，无需指定参数类型，也无需指定的重载形式——它们都将被自动推导。而如果使用方法引用，也是同样可以根据上下文进行推导。

函数式接口是Lambda的基础，而方法引用是Lambda的孪生兄弟。

下面这段代码将会调用 `println` 方法的不同重载形式，将函数式接口改为int类型的参数：

```java
@FunctionalInterface
public interface PrintableInteger {
	void print(int str);
}
```

由于上下文变了之后可以自动推导出唯一对应的匹配重载，所以方法引用没有任何变化：

```java
public class Demo03PrintOverload {
    private static void printInteger(PrintableInteger data) {
        data.print(1024);
    }
    public static void main(String[] args) {
    	printInteger(System.out::println);
    }
}
```

这次方法引用将会自动匹配到 `println(int)` 的重载形式。

### 2. 通过对象名引用成员方法

这是最常见的一种用法，与上例相同。如果一个类中已经存在了一个成员方法：

```java
public class MethodRefObject {
    public void printUpperCase(String str) {
    	System.out.println(str.toUpperCase());
    }
}
```

函数式接口仍然定义为：

```java
@FunctionalInterface
public interface Printable {
	void print(String str);
}
```

那么当需要使用这个 `printUpperCase` 成员方法来替代 `Printable` 接口的Lambda的时候，已经具有了
MethodRefObject 类的对象实例，则可以通过对象名引用成员方法，代码为：

```java
public class Demo04MethodRef {
    private static void printString(Printable lambda) {
        lambda.print("Hello");
    }
    public static void main(String[] args) {
    	MethodRefObject obj = new MethodRefObject();
    	printString(obj::printUpperCase);
    }
}
```

### 3. 通过类名称引用静态方法

由于在 `java.lang.Math` 类中已经存在了静态方法 `abs` ，所以当我们需要通过Lambda来调用该方法时，有两种写法。首先是函数式接口：

```java
@FunctionalInterface
public interface Calcable {
	int calc(int num);
}
```

第一种写法是使用Lambda表达式：

```java
public class Demo05Lambda {
    private static void method(int num, Calcable lambda) {
        System.out.println(lambda.calc(num));
    }
    public static void main(String[] args) {
        method(‐10, n ‐> Math.abs(n));
    }
}
```

但是使用方法引用的更好写法是：

```java
public class Demo06MethodRef {
    private static void method(int num, Calcable lambda) {
    	System.out.println(lambda.calc(num));
    }
    public static void main(String[] args) {
    	method(‐10, Math::abs);
    }
}
```

在这个例子中，下面两种写法是等效的：

- Lambda表达式： `n -> Math.abs(n)`
- 方法引用： `Math::abs`

### 4. 通过super引用成员方法

如果存在继承关系，当Lambda中需要出现super调用时，也可以使用方法引用进行替代。首先是函数式接口：

```java
@FunctionalInterface
public interface Greetable {
	void greet();
}
```

然后是父类 `Human` 的内容：

```java
public class Human {
    public void sayHello() {
    	System.out.println("Hello!");
    }
}
```

使用方法引用来调用父类中的 sayHello 方法，例如一个子类 Woman ：

```java
public class Man extends Human {
    @Override
    public void sayHello() {
    	System.out.println("大家好,我是Man!");
    }
    //定义方法method,参数传递Greetable接口
    public void method(Greetable g){
    	g.greet();
    }
    public void show(){
    	method(super::sayHello);
    }
}
```

在这个例子中，下面两种写法是等效的：

- Lambda表达式： `() -> super.sayHello()`
- 方法引用： `super::sayHello`

### 5. 通过this引用成员方法

this代表当前对象，如果需要引用的方法就是当前类中的成员方法，那么可以使用“`this::成员方法`”的格式来使用方法引用。首先是简单的函数式接口：

```java
@FunctionalInterface
public interface Richable {
	void buy();
}
```

下面是一个丈夫 Husband 类：

```
public class Husband {
    private void marry(Richable lambda) {
    	lambda.buy();
    }
    public void beHappy() {
    	marry(() ‐> System.out.println("买套房子"));
    }
}
```

开心方法 `beHappy` 调用了结婚方法 `marry` ，后者的参数为函数式接口 `Richable` ，所以需要一个Lambda表达式。但是如果这个Lambda表达式的内容已经在本类当中存在了，则可以对 `Husband` 丈夫类进行修改：

```java
public class Husband {
    private void buyHouse() {
    	System.out.println("买套房子");
    }
    private void marry(Richable lambda) {
    	lambda.buy();
    }
    public void beHappy() {
    	marry(() ‐> this.buyHouse());
    }
}
```

方法引用写法:

```java
public class Husband {
    private void buyHouse() {
        System.out.println("买套房子");
    }
    private void marry(Richable lambda) {
        lambda.buy();
    }
    public void beHappy() {
        marry(this::buyHouse);
    }
}
```

在这个例子中，下面两种写法是等效的：

- Lambda表达式： `() -> this.buyHouse()`
- 方法引用： `this::buyHouse`

### 6. 类的构造器引用

由于构造器的名称与类名完全一样，并不固定。所以构造器引用使用 类名称::new 的格式表示。首先是一个简单
的 Person 类：

```java
public class Person {
	private String name;
    public Person(String name) {
    	this.name = name;
    }
    public String getName() {
    	return name;
    }
    public void setName(String name) {
    	this.name = name;
    }
}
```

然后是用来创建 `Person` 对象的函数式接口：

```java
public interface PersonBuilder {
	Person buildPerson(String name);
}
```

构造器引用写法:

```java
public class Demo10ConstructorRef {
    public static void printName(String name, PersonBuilder builder) {
    	System.out.println(builder.buildPerson(name).getName());
    }
    public static void main(String[] args) {
    	printName("赵丽颖", Person::new);
    }
}
```

下面两种写法是等效的：

- Lambda表达式： `name -> new Person(name)`
- 方法引用： `Person::new`

### 7. 数组的构造器引用

数组也是 Object 的子类对象，所以同样具有构造器，只是语法稍有不同。如果对应到Lambda的使用场景中时，需要一个函数式接口：

```java
@FunctionalInterface
public interface ArrayBuilder {
	int[] buildArray(int length);
}
```

在应用该接口的时候，可以通过Lambda表达式：

```java
public class Demo11ArrayInitRef {
    private static int[] initArray(int length, ArrayBuilder builder) {
    	return builder.buildArray(length);
    }
    public static void main(String[] args) {
    	int[] array = initArray(10, length ‐> new int[length]);
    }
}
```

使用数组的构造器引用：

```java
public class Demo12ArrayInitRef {
    private static int[] initArray(int length, ArrayBuilder builder) {
    	return builder.buildArray(length);
    }
    public static void main(String[] args) {
    	int[] array = initArray(10, int[]::new);
    }
}
```

在这个例子中，下面两种写法是等效的：

- Lambda表达式： `length -> new int[length]`
- 方法引用： `int[]::new`



## 十一. 反射

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

要想解剖一个类,必须先要获取到该类的字节码文件对象。而解剖使用的就是Class类中的方法.所以先要获取到每一个字节码文件对应的Class类型的对象.

**反射就是把java类中的各种成分映射成一个个的Java对象**

例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。（其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）

![1558855774009](.\img\%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1558855774009.png)

* 框架：半成品软件。可以在框架的基础上进行软件开发，简化编码
* 反射：**将类的各个组成部分封装为其他对象，这就是反射机制**
  * 好处：
    1. 可以在程序运行过程中，操作这些对象。
    2. 可以解耦，提高程序的可扩展性。

### 1. 获取Class对象的三种方式

* 获取Class对象的方式：
  1. `Class.forName("全类名")`：将字节码文件加载进内存，返回Class对象
    * 多用于配置文件，将类名定义在配置文件中。读取文件，加载类
  2. `类名.class`：通过类名的属性class获取
    * 多用于参数的传递
  3. `对象.getClass()`：getClass()方法在Object类中定义着。
    * 多用于对象的获取字节码的方式

  * 结论：
    **同一个字节码文件(`*.class`)在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个。**

```java
package fanshe;
/**
 * 获取Class对象的三种方式
 * 1 Object ——> getClass();
 * 2 任何数据类型（包括基本数据类型）都有一个“静态”的class属性
 * 3 通过Class类的静态方法：forName（String  className）(常用)
 *
 */
public class Fanshe {
	public static void main(String[] args) {
		//第一种方式获取Class对象  
		Student stu1 = new Student();//这一new 产生一个Student对象，一个Class对象。
		Class stuClass = stu1.getClass();//获取Class对象
		System.out.println(stuClass.getName());
		
		//第二种方式获取Class对象
		Class stuClass2 = Student.class;
		System.out.println(stuClass == stuClass2);//判断第一种方式获取的Class对象和第二种方式获取的是否是同一个
		
		//第三种方式获取Class对象
		try {
			Class stuClass3 = Class.forName("fanshe.Student");//注意此字符串必须是真实路径，就是带包名的类路径，包名.类名
			System.out.println(stuClass3 == stuClass2);//判断三种方式是否获取的是同一个Class对象
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
		
	}
}
```

> 三种方式常用第三种，第一种对象都有了还要反射干什么。第二种需要导入类的包，依赖太强，不导包就抛编译错误。**一般都第三种，一个字符串可以传入也可写在配置文件中等多种方法。**

### 2. 通过反射获取构造方法并使用

#### 1.获取构造方法

  1).批量的方法：
`public Constructor[] getConstructors()`：**所有"公有的"构造方法**
`public Constructor[] getDeclaredConstructors()`：获取所有的构造方法(包括私有、受保护、默认、公有)
     
  2).获取单个的方法，并调用：
`public Constructor getConstructor(Class... parameterTypes)`：获取**单个的"公有的"构造方法**
`public Constructor getDeclaredConstructor(Class... parameterTypes)`：获取"某个构造方法"可以是私有的，或受保护、默认、公有；

#### 2. 调用构造方法

`Constructor --> newInstance(Object... initargs)`

`newInstance`是 `Constructor`类的方法（管理构造函数的类）
api的解释为：
`newInstance(Object... initargs)`
           **使用此 Constructor 对象表示的构造方法来创建该构造方法的声明类的新实例，并用指定的初始化参数初始化该实例。**

它的返回值是T类型，所以newInstance是创建了一个构造方法的声明类的新实例对象。并为之调用

```java
package fanshe;
 
import java.lang.reflect.Constructor;
 
 
/*
 * 通过Class对象可以获取某个类中的：构造方法、成员变量、成员方法；并访问成员；
 * 
 * 1.获取构造方法：
 * 		1).批量的方法：
 * 			public Constructor[] getConstructors()：所有"公有的"构造方法
            public Constructor[] getDeclaredConstructors()：获取所有的构造方法(包括私有、受保护、默认、公有)
     
 * 		2).获取单个的方法，并调用：
 * 			public Constructor getConstructor(Class... parameterTypes):获取单个的"公有的"构造方法：
 * 			public Constructor getDeclaredConstructor(Class... parameterTypes):获取"某个构造方法"可以是私有的，或受保护、默认、公有；
 * 		
 * 			调用构造方法：
 * 			Constructor-->newInstance(Object... initargs)
 */
public class Constructors {
 
	public static void main(String[] args) throws Exception {
		//1.加载Class对象
		Class clazz = Class.forName("fanshe.Student");
		
		
		//2.获取所有公有构造方法
		System.out.println("**********************所有公有构造方法*********************************");
		Constructor[] conArray = clazz.getConstructors();
		for(Constructor c : conArray){
			System.out.println(c);
		}
		
		
		System.out.println("************所有的构造方法(包括：私有、受保护、默认、公有)***************");
		conArray = clazz.getDeclaredConstructors();
		for(Constructor c : conArray){
			System.out.println(c);
		}
		
		System.out.println("*****************获取公有、无参的构造方法*******************************");
		Constructor con = clazz.getConstructor(null);
		//1>、因为是无参的构造方法所以类型是一个null,不写也可以：这里需要的是一个参数的类型，切记是类型
		//2>、返回的是描述这个无参构造函数的类对象。
	
		System.out.println("con = " + con);
		//调用构造方法
		Object obj = con.newInstance();
	//	System.out.println("obj = " + obj);
	//	Student stu = (Student)obj;
		
		System.out.println("******************获取私有构造方法，并调用*******************************");
		con = clazz.getDeclaredConstructor(char.class);
		System.out.println(con);
		//调用构造方法
		con.setAccessible(true);//暴力访问(忽略掉访问修饰符)
		obj = con.newInstance('男');
	} 
	
}  
```

### 3. 获取成员变量并调用

#### 1. 获取字段

- 批量获取字段的方法
  - `getDeclaredFields()`: 返回的数组 Field对象反映此表示的类或接口声明的**所有字段** 类对象。
  - `getFields()` : 返回包含一个数组 Field对象反射由此表示的类或接口的所有可访问的**公共字段** 类对象。
- 获取单个字段的方法
  - `getDeclaredField(String name)`: 返回一个 Field对象，它反映此表示的类或接口的指定已声明字段 类对象(可以是任何修饰)。
  - `getField(String name)`: 返回一个 Field对象，它反映此表示的类或接口的**指定公共成员字段** 类对象。

#### 2. Field主要方法

- 设置值

  - `void set(Object obj, Object value)`  

    - 参数说明：

      1.obj: 要设置的字段所在的对象；

      2.value: 要为字段设置的值；

- 获取值

  - `get(Object obj)` 

- 忽略访问权限修饰符的安全检查

  - `setAccesible(true)`: 暴力反射，解除私有限定(**禁止安全检查, 可以提高反射的运行的速度**)

```java
package fanshe.field;
import java.lang.reflect.Field;
/*
 * 获取成员变量并调用：
 * 
 * 1.批量的
 * 		1).Field[] getFields():获取所有的"公有字段"
 * 		2).Field[] getDeclaredFields():获取所有字段，包括：私有、受保护、默认、公有；
 * 2.获取单个的：
 * 		1).public Field getField(String fieldName):获取某个"公有的"字段；
 * 		2).public Field getDeclaredField(String fieldName):获取某个字段(可以是私有的)
 * 
 * 	 设置字段的值：
 * 		Field --> public void set(Object obj,Object value):
 * 					参数说明：
 * 					1.obj:要设置的字段所在的对象；
 * 					2.value:要为字段设置的值；
 * 
 */
public class Fields {
 
		public static void main(String[] args) throws Exception {
			//1.获取Class对象
			Class stuClass = Class.forName("fanshe.field.Student");
			//2.获取字段
			System.out.println("************获取所有公有的字段********************");
			Field[] fieldArray = stuClass.getFields();
			for(Field f : fieldArray){
				System.out.println(f);
			}
			System.out.println("************获取所有的字段(包括私有、受保护、默认的)********************");
			fieldArray = stuClass.getDeclaredFields();
			for(Field f : fieldArray){
				System.out.println(f);
			}
			System.out.println("*************获取公有字段**并调用***********************************");
			Field f = stuClass.getField("name");
			System.out.println(f);
			//获取一个对象
			Object obj = stuClass.getConstructor().newInstance();//产生Student对象--》Student stu = new Student();
			//为字段设置值
			f.set(obj, "刘德华");//为Student对象中的name属性赋值--》stu.name = "刘德华"
			//验证
			Student stu = (Student)obj;
			System.out.println("验证姓名：" + stu.name);
			
			
			System.out.println("**************获取私有字段****并调用********************************");
			f = stuClass.getDeclaredField("phoneNum");
			System.out.println(f);
			f.setAccessible(true);//暴力反射，解除私有限定
			f.set(obj, "18888889999");
			System.out.println("验证电话：" + stu);
			
		}
	}
```

### 4. 获取成员方法并调用

#### 1. 获取方法

- 批量获取方法的方法
  - `getDeclaredMethods()`: 获取所有的成员方法，包括私有的(不包括继承的)
  - `getMethods()` : 取所有"**公有方法**"；（包含了父类的方法也包含Object类）
- 获取单个方法的方法
  - `getDeclaredMethod(String name, Class<?>... parameterTypes)`: 取所有"**公有方法**"
  - `getMethod(String name, Class<?>... parameterTypes)`: 获取所有的成员方法，包括私有的
  - 参数: 
    - name: 方法名
    - parameterType: 形参的参数类型(int.class, String,class)

#### 2. Method主要方法

- 调用方法

  - `Object invoke(Object obj,Object... args):`  

    - 参数说明：

      1.obj: 要调用方法的对象；

      2.args: 调用方式时所传递的实参；

- 获取方法名

  - `String getName()` 

- 忽略访问权限修饰符的安全检查

  - `setAccessible(true)`: 解除私有限定

```java
package fanshe.method;
 
import java.lang.reflect.Method;
 
/*
 * 获取成员方法并调用：
 * 
 * 1.批量的：
 * 		public Method[] getMethods():获取所有"公有方法"；（包含了父类的方法也包含Object类）
 * 		public Method[] getDeclaredMethods():获取所有的成员方法，包括私有的(不包括继承的)
 * 2.获取单个的：
 * 		public Method getMethod(String name,Class<?>... parameterTypes):
 * 					参数：
 * 						name : 方法名；
 * 						Class ... : 形参的Class类型对象
 * 		public Method getDeclaredMethod(String name,Class<?>... parameterTypes)
 * 
 * 	 调用方法：
 * 		Method --> public Object invoke(Object obj,Object... args):
 * 					参数说明：
 * 					obj : 要调用方法的对象；
 * 					args:调用方式时所传递的实参；
):
 */
public class MethodClass {
 
	public static void main(String[] args) throws Exception {
		//1.获取Class对象
		Class stuClass = Class.forName("fanshe.method.Student");
		//2.获取所有公有方法
		System.out.println("***************获取所有的”公有“方法*******************");
		stuClass.getMethods();
		Method[] methodArray = stuClass.getMethods();
		for(Method m : methodArray){
			System.out.println(m);
		}
		System.out.println("***************获取所有的方法，包括私有的*******************");
		methodArray = stuClass.getDeclaredMethods();
		for(Method m : methodArray){
			System.out.println(m);
		}
		System.out.println("***************获取公有的show1()方法*******************");
		Method m = stuClass.getMethod("show1", String.class);
		System.out.println(m);
		//实例化一个Student对象
		Object obj = stuClass.getConstructor().newInstance();
		m.invoke(obj, "刘德华");
		
		System.out.println("***************获取私有的show4()方法******************");
		m = stuClass.getDeclaredMethod("show4", int.class);
		System.out.println(m);
		m.setAccessible(true);//解除私有限定
		Object result = m.invoke(obj, 20);//需要两个参数，一个是要调用的对象（获取有反射），一个是实参
		System.out.println("返回值：" + result);
		
	}
}
```

### 5. 注解Annotation

- Annotation是从JDK5.0开始引入的新技术。 

- Annotation的作用： 
  - 不是程序本身，可以对程序作出解释。(这一点，跟注释没什么区别) 
  - 可以被其他程序(比如：编译器等)读取。(注解信息处理流程，是注解和注释的重大区别.如果没有注解信息处理流程，则注解毫无意义) 

- Annotation的格式： 
  - 注解是以“@注释名”在代码中存在的，还可以添加一些参数值，例如：@SuppressWarnings(value="unchecked")。 

- Annotation在哪里使用? 
  - 可以附加在package, class, method, field等上面，相当于给它们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问。

#### 1. 开发中常见的注解

- @Override：用于标识该方法继承自超类, 当父类的方法被删除或修改了，编译器会提示错误信息(我们最经常看到的toString()方法上总能看到这货)
- @Deprecated：表示该类或者该方法已经不推荐使用，已经过期了，如果用户还是要使用，会生成编译的警告
- @SuppressWarnings：用于忽略的编译器警告信息
- Junit测试：@Test
- Spring的一些注解：@Controller、@RequestMapping、@RequestParam、@ResponseBody、@Service、@Component、@Repository、@Resource、@Autowire
- Java验证的注解：@NotNull、@Email

#### 2. 注解数据类型

注解是写在.java文件中，使用@interface作为关键字, 所以注解也是Java的一种数据类型，从广泛的定义来说，Class、Interface、Enum、Annotation都属于Class类型。

**格式: **

```java
public @interface 注解名称{
    String[] value() default {};
}
```

- `public @interface 注解名{定义体}`
- 其中的每一个方法实际上是声明了一个配置参数。
- 方法的名称就是参数的名称
- 返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum, 可以通过`default`来声明参数的默认值。
  - 基本数据类型
  - String
  - Annotation
  - Enum
  - 以上类型的注解
- 如果只有一个参数成员，一般参数名为`value`
- **注解元素必须要有值。**我们定义注解元素时，经常使用空字符串、0作为默认值。也经常使用负数(比如：-1)表示不存在的含义 
- 注解本质上就是一个接口，该接口默认继承`Annotation`接口
  - `public interface MyAnno extends java.lang.annotation.Annotation {}`

#### 3. 元注解

在创建注解的时候，需要使用一些注解来描述自己创建的注解，就是写在`@interface`上面的那些注解，这些注解被称为元注解，如在`Override`中看到的`@Target`、`@Retention`等。

##### @Documented

用于标记在生成javadoc时是否将注解包含进去，可以看到这个注解和@Override一样，注解中空空如也，什么东西都没有

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {

}
```

@Target

**用于定义注解可以在什么地方使用，默认可以在任何地方使用，也可以指定使用的范围**，开发中将注解用在类上(如`@Controller`)、字段上(如`@Autowire`)、方法上(如`@RequestMapping`)、方法的参数上(如`@RequestParam`)等比较常见。

- TYPE: 类, 接口, Enume声明
- FIELD: 字段(类的成员变量)声明
- METHOD: 方法声明
- PARAMETER: 参数声明
- CONSTRUCTOR: 构造方法声明
- LOCAL_VARIABLE: 局部变量声明
- ANNOTATION_TYPE: 注释类型声明
- PACKAGE: 包声明

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /** Type parameter declaration */
    TYPE_PARAMETER,

    /** Use of a type */
    TYPE_USE
}
```

##### @Inherited

允许子类继承父类中的注解，可以通过反射获取到父类的注解(**描述注解是否被子类继承**)

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {

}
```

##### @Constraint

用于验证属性是否合法

```java
@Documented
@Target({ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraint {
    Class<? extends ConstraintValidator<?, ?>>[] validatedBy();
}
```

##### @Retention

注解的声明周期，**用于定义注解的存活阶段，可以存活在源码级别、编译级别(字节码级别)、运行时级别**

- SOURCE: 源码级别，注解只存在源码中，一般用于和编译器交互，用于检测代码。如@Override, @SuppressWarings。
- CLASS: 字节码级别，注解存在于源码和字节码文件中，主要用于编译时生成额外的文件，如XML，Java文件等，但运行时无法获得。 如mybatis生成实体和映射文件，这个级别需要添加JVM加载时候的代理（javaagent），使用代理来动态修改字节码文件。
- RUNTION: 运行时级别，注解存在于源码、字节码、java虚拟机中，主要用于运行时，可以使用反射获取相关的信息。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

#### 4. 注解的使用场景

可以通过注解的声明周期来分析注解的使用场景：

- SOURCE源码级别：给编译器使用，如@Override、@Deprecated 等， 这部分开发者应该使用的场景不多
- CLASS：字节码级别，这部分也很少见到
- RUNTIME：运行时级别，这个是最多的，几乎开发者使用到的注解都是运行时级别，运行时注解常用的有以下几种情况 
  - **注解中没有任何属性的，空的注解，这部分注解通常起到一个标注的作用**，如@Test、@Before、@After，通过获取这些标记注解在逻辑上做一些特殊的处理
  - **可以使用约束注解@Constraint来对属性值进行校验**，如@Email, @NotNull等
  - **可以通过在注解中使用属性来配置一些参数，然后可以使用反射获取这些参数，这些注解没有其他特殊的功能，只是简单的代替xml配置的方式来配置一些参数。**使用注解来配置参数这在Spring boot中得到了热捧，如@Configuration

**关于配置方式xml vs annotation, 一般使用xml配置一些和业务关系不太紧密的配置，使用注解配置一些和业务密切相关的参数。**

#### 5. 注解和反射基本API

```java
// 获取某个类型的注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass);
// 获取所有注解(包括父类中被Inherited修饰的注解)
public Annotation[] getAnnotations(); 
// 获取声明的注解(但是不包括父类中被Inherited修饰的注解)
public Annotation[] getDeclaredAnnotations();
// 判断某个对象上是否被某个注解进行标注
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass);
```

#### 6. Demo

```java
package cn.demo.reflects;

/**
 * Create by Jjcc on 2019/5/26 16:25
 *
 * @author Jjcc
 */
@Table("tb_person")
public class Person {

    @TabPrimaryKey
    @TabField(name = "id", type = "int", length = 11, isNotNull = false, autoIncrement = true)
    private String id;

    @TabPrimaryKey
    @TabField(name = "name", type = "varchar", length = 20, isNotNull = true, autoIncrement = false, defaultVal = "DEFAULT NULL")
    private String name;

    @TabField(name = "sex", type = "varchar", length = 20, isNotNull = true, autoIncrement = false, defaultVal = "DEFAULT NULL")
    private String sex;


    //无参构造方法
    public Person() {

    }

    //有参构造方法
    public Person(String name, String sex, String id) {
        this.name = name;
        this.sex = sex;
        this.id = id;
    }

    private Person(String name) {
        this.name = name;
    }

    public Person(String name, String id) {
        this.name = name;
        this.id = id;
    }


    //无参无返回
    private void a() {
        System.out.println("a");
    }

    //有参无返回
    private void b(String b) {
        System.out.println(b);
    }

    //无参有返回
    private String c() {
        return "c";
    }

    //有参有返回
    private String d(String d) {
        return d;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", id='" + id + '\'' +
                '}';
    }
}
```

```java
package cn.demo.reflects;

import java.lang.annotation.*;

/**
 * Create by Jjcc on 2019/5/30 22:45
 *
 * @author Jjcc
 */

@Documented
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    String value();

}
```

```java
package cn.demo.reflects;

import java.lang.annotation.*;

/**
 * Create by Jjcc on 2019/5/30 22:52
 *
 * @author Jjcc
 */
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TabPrimaryKey {

}
```

```java
package cn.demo.reflects;

import java.lang.annotation.*;

/**
 * Create by Jjcc on 2019/5/30 23:37
 *
 * @author Jjcc
 */
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TabField {
    //字段名
    String name();

    //字段类型
    String type();

    //字段长度
    int length();

    //是否为空, true为允许为空, false为不允许为空
    boolean isNotNull() default true;

    //是否自增, true为自增, false为不自增
    boolean autoIncrement() default false;

    //默认值
    String defaultVal() default "";
}
```

```java
package cn.demo.reflects;


import java.lang.reflect.Field;
import java.util.Arrays;

/**
 * Create by Jjcc on 2019/5/30 22:42
 *
 * @author Jjcc
 */
public class MyAnnoDemoOne {
    public static void main(String[] args){
        try {
            Class aClass = Class.forName("cn.demo.reflects.Person");
            StringBuilder sb = new StringBuilder();
            sb.append("CREATE TABLE ");

            //判断是否有@Table注解
            if (aClass.isAnnotationPresent(Table.class)) {
                //获取表名
                Table tableName = (Table) aClass.getAnnotation(Table.class);
                sb.append(tableName.value() + "(");

                //获取类的所有成员变量
                Field[] fields = aClass.getDeclaredFields();

                StringBuilder sb2 = new StringBuilder();
                sb2.append("PRIMARY KEY (");
                Arrays.stream(fields).forEach(t -> {
                    if (t.isAnnotationPresent(TabField.class)) {
                        TabField anno = t.getAnnotation(TabField.class);
                        sb.append(anno.name() + " " + anno.type() + "(" + anno.length() + ") ");

                        if (!anno.isNotNull())
                            sb.append("NOT NULL ");

                        if (anno.autoIncrement())
                            sb.append("AUTO_INCREMENT ");

                        sb.append(anno.defaultVal() + ",");

                        if (t.isAnnotationPresent(TabPrimaryKey.class))
                            sb2.append(anno.name() + ",");
                    }
                });
                sb2.setCharAt(sb2.length() - 1, ')');
                sb.append(sb2);

                sb.append(");");

                System.out.println(sb.toString());
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

}
```

**结果:**

```
CREATE TABLE tb_person(
id int(11) NOT NULL AUTO_INCREMENT ,
name varchar(20) DEFAULT NULL,
sex varchar(20) DEFAULT NULL,
PRIMARY KEY (id,name)
);

```



## 十二. Java字节码操作

**Java动态性的两种常见实现方式**

- 字节码操作
- 反射

**运行时操作字节码可以让我们实现如下功能: **

- 动态生成新的类
- 动态改变某个类的结构(添加/删除/修改; 新的属性/方法)

**优势:**

- 比反射开销小, 性能高
- Javassist性能高于反射, 低于ASM

**常见的字节码操作类库**

**BCEL**

这是Apache Software Fundation的jakarta项目的一部分。BCEL是javaclassworking广泛使用的一种跨级啊，它可以让你深入JVM汇编语言进行类的操作的细节。BCEL与javassist所强调的是源码级别的工作。

**ASM**

是一个轻量级java字节码操作框架，直接涉及到JVM底层的操作和指令。

**CGLIB**

是一个强大的，高性能，高质量的Code生成类库，基于ASM实现。

**JAVAssist**

是一个开源的分析、编辑和创建java字节码的类库，性能较ASM差，跟cglib查不到，但是使用简单

### 1. Javassist使用方法

通过使用Javassist对字节码操作为JBoss实现动态AOP框架。javassist是jboss的一个子项目，其主要的优点，在于简单，而且快速。直接使用java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类。

`Javassist`中最为重要的是`ClassPool`，`CtClass` ，`CtMethod` 以及 `CtField`这几个类。

`ClassPool`：一个基于HashMap实现的CtClass对象容器，其中键是类名称，值是表示该类的CtClass对象。默认的ClassPool使用与底层JVM相同的类路径，因此在某些情况下，可能需要向ClassPool添加类路径或类字节。

**CtClass**：表示一个类，这些CtClass对象可以从ClassPool获得。

**CtMethods**：表示类中的方法。

**CtFields** ：表示类中的字段。

#### 1. 创建一个类

```java
package cn.demo.bytecode;

import javassist.*;
import javassist.bytecode.AccessFlag;

import java.io.IOException;

/**
 * Create by Jjcc on 2019/6/2 21:51
 *
 * @author Jjcc
 */
public class BytecodeDemoOne {

    public static void main(String[] args){
        ClassPool pool = ClassPool.getDefault();
        //创建类
        CtClass cc = pool.makeClass("cn.demo.bytecode.Emp");
        //创建的类实现Cloneable接口
        cc.setInterfaces(new CtClass[]{pool.makeInterface("java.lang.Cloneable")});

        try {
            //两种方式创建成员变量
            //使用静态方法创建类的字段
            CtField private_int_id = CtField.make("private int id;", cc);
            cc.addField(private_int_id);
            //使用构造方法创建类的字段
            CtField name = new CtField(pool.get("java.lang.String"), "name", cc);
            name.setModifiers(AccessFlag.PRIVATE);
            cc.addField(name);

            //两种方式创建构造方法
            //使用静态方法创建类的无参构造方法
            CtConstructor ctConstructor1 = CtNewConstructor.make("public Emp(){}", cc);
            cc.addConstructor(ctConstructor1);
            //使用构造方法创建类的有参构造方法
            CtConstructor ctConstructor2 = new CtConstructor(new CtClass[]{CtClass.intType, pool.get("java.lang.String")}, cc);
            //构造方法的方法体
            ctConstructor2.setBody("{this.id=id; this.name=name;}");
            cc.addConstructor(ctConstructor2);

            //两种方式创建成员方法
            //使用静态方法的方式创建类的方法
            CtMethod setId = CtNewMethod.make("public void setId(int id){this.id = id;}", cc);
            CtMethod getId = CtNewMethod.make("public int getId(){return this.id;}", cc);
            cc.addMethod(setId);
            cc.addMethod(getId);
            //使用构造方法的方式创建类的成员方法
            CtMethod setName = new CtMethod(CtClass.voidType, "setName", new CtClass[]{pool.get("java.lang.String")}, cc);
            CtMethod getName = new CtMethod(pool.get("java.lang.String"), "getName", new CtClass[]{}, cc);
            //方法体
            setName.setBody("{this.name = name;}");
            getName.setBody("{return this.name;}");
            cc.addMethod(setName);
            cc.addMethod(getName);

            //将生成的.class文件保存到磁盘
            cc.writeFile("E:\\软件开发资料\\Java\\MyJava");


        } catch (CannotCompileException e) {
            e.printStackTrace();
        } catch (NotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

```

**结果: **

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package cn.demo.bytecode;

public class Emp implements Cloneable {
    private int id;
    private String name;

    public Emp() {
    }

    public Emp(int var1, String var2) {
        this.id = this.id;
        this.name = this.name;
    }

    public void setId(int var1) {
        this.id = var1;
    }

    public int getId() {
        return this.id;
    }

    public void setName(String var1) {
        this.name = this.name;
    }

    public String getName() {
        return this.name;
    }
}

```



## 十三. 新时间日期API

> Java 8 吸收了 Joda-Time 的精华，以一个新的开始为 Java 创建优秀的 API。新的 java.time 中**包含了所有关于本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）和持续时间（Duration）的类**。历史悠久的 Date 类新增了 toInstant() 方法，用于把 Date 转换成新的表示形式。这些新增的本地化时间日期 API 大大简化了日期时间和本地化的管理。
>

- **`java.time` – 包含值对象的基础包**
- `java.time.chrono` – 提供对不同的日历系统的访问
- **`java.time.format` – 格式化和解析时间和日期**
- **`java.time.temporal` – 包括底层框架和扩展特性**
- `java.time.zone` – 包含时区支持的类

### 1. LocalDate、LocalTime、LocalDateTime

**LocalDate、LocalTime、LocalDateTime** 类是其中较重要的几个类，它们的实例是不可变的对象，分别表示使用 `ISO-8601`日历（公立）系统的日期、时间、日期和时间。它们提供了简单的本地日期或时间，并不包含当前的时间信息，也不包含与时区相关的信息。

|                           **方法**                           | **描述**                                                     |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
|              **now**()  /  **now(ZoneId zone)**              | 静态方法，根据当前时间创建对象/指定时区的对象                |
|                           **of()**                           | 静态方法，根据指定日期/时间创建对象                          |
|              **getDayOfMonth()/getDayOfYear()**              | 获得月份天数(1-31) /获得年份天数(1-366)                      |
|                      **getDayOfWeek()**                      | 获得星期几(返回一个 DayOfWeek 枚举值)                        |
|                        **getMonth()**                        | 获得月份, 返回一个 Month 枚举值                              |
|             **getMonthValue() /** **getYear()**              | 获得月份(1-12) /获得年份                                     |
|            **getHour()/getMinute()/getSecond()**             | 获得当前对象对应的小时、分钟、秒                             |
| **withDayOfMonth()/withDayOfYear()/withMonth()/withYear()**  | 将月份天数、年份天数、月份、年份修改为指定的值并返回新的对象 |
| **plusDays(), plusWeeks(),** **plusMonths(), plusYears(),plusHours()** | 向当前对象添加几天、几周、几个月、几年、几小时               |
| **minusMonths() / minusWeeks()/minusDays()/minusYears()/minusHours()** | 从当前对象减去几月、几周、几天、几年、几小时                 |

**以`LocalDateTime`为例，其它2个类使用方法一样**

```java
// 1. LocalDate / LocalTime / LocalDateTime
	//替换原有的Calendar
	@Test
	public void test1() {
		// 实例化
		// now():当前的时间
		LocalDate localDate = LocalDate.now();

		LocalTime localTime = LocalTime.now();

		LocalDateTime localDateTime = LocalDateTime.now();

		System.out.println(localDate);
		System.out.println(localTime);
		System.out.println(localDateTime);

		// of()：获取指定的时间
		LocalDate localDate2 = LocalDate.of(2017, 8, 15);
		System.out.println(localDate2);

		LocalDateTime localDateTime2 = LocalDateTime.of(2017, 8, 15, 11, 11, 23);
		System.out.println(localDateTime2);
		System.out.println();
		
		// getXxx():
		System.out.println(localDateTime.getDayOfYear());
		System.out.println(localDateTime.getDayOfMonth());
		System.out.println(localDateTime.getDayOfWeek());
		System.out.println(localDateTime.getMonth());
		System.out.println(localDateTime.getMonthValue());
		System.out.println(localDateTime.getHour());
		System.out.println(localDateTime.getMinute());
		System.out.println();
		
		// withXxx():体现了不可变性
		LocalDateTime localDateTime3 = localDateTime.withDayOfMonth(20);
		System.out.println(localDateTime);
		System.out.println(localDateTime3);

		LocalDateTime localDateTime4 = localDateTime.withHour(12);
		System.out.println(localDateTime4);

		// plus()
		// minus()
		LocalDateTime localDateTime5 = localDateTime.plusDays(3);
		System.out.println(localDateTime5);

		LocalDateTime localDateTime6 = localDateTime.minusMinutes(20);
		System.out.println(localDateTime6);

	}

	// 2. Instant:时间点
	//替换原有的Date
	@Test
	public void test2() {
		// now():得到Instant的实例
		Instant instant = Instant.now();// 表示自1970年1月1日0时0分0秒（UTC）开始的秒数
		System.out.println(instant);

		// atOffset():得到带偏移量的日期时间
		OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
		System.out.println(offsetDateTime);

		// 得到时间戳
		long milli = instant.toEpochMilli();
		System.out.println(milli);// 
		

		// 根据毫秒数，得到时间点的对象
		Instant instant2 = Instant.ofEpochMilli(milli);
		System.out.println(instant2);
		
		
		Date date = new Date();//被76行替代
		Date date1 = new Date(3252345324534L);//被89行替代
	}

	// 3.DateTimeFormatter:日期时间的格式化工具
	//替换原有的SimpleDateFormat
	//格式化：日期 ---> 字符串
	//解析：字符串 ---> 日期
	@Test
	public void test3() {
		//方式一：预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE
		DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
		LocalDateTime localDateTime = LocalDateTime.now();
		String formatDateTime = dateTimeFormatter.format(localDateTime);
		System.out.println(formatDateTime);

		//方式二：本地化相关的格式。如：ofLocalizedDateTime()
		// FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT :适用于LocalDateTime
		LocalDateTime localDateTime1 = LocalDateTime.now();
		DateTimeFormatter dateTimeFormatter1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT);
		// 格式化：DateTime ---->文本
		String formatDateTime1 = dateTimeFormatter1.format(localDateTime1);
		System.out.println(formatDateTime1);
		
		// 本地化相关的格式。如：ofLocalizedDate()
		// FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT : 适用于LocalDate
		LocalDate localDate = LocalDate.now();
		DateTimeFormatter format = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);
		String formatDateTime2 = format.format(localDate);
		System.out.println(formatDateTime2);
		
		// 解析:文本 --->DateTime
		TemporalAccessor temporalAccessor = dateTimeFormatter1.parse("17-11-5 上午1:03");
		System.out.println(temporalAccessor);
//
//		//方式三：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss E”)
		DateTimeFormatter dateTimeFormatter2 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
		// 格式化：DateTime ---->文本
		String dateTimeStr = dateTimeFormatter2.format(localDateTime);
		System.out.println(dateTimeStr);// 2017-08-15 05:07:33
		// 解析:文本 --->DateTime
		TemporalAccessor temporalAccessor1 = dateTimeFormatter2.parse("2017-08-15 05:07:33");
		System.out.println(temporalAccessor1);

	
```
























