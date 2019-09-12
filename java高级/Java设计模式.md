# Java设计模式

- **创建型模式(对象怎么来)：** 

  – 单例模式、工厂模式、抽象工厂模式、建造者模式、原型模式。 

- **结构型模式(对象和谁有关)**： 

  – 适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。 

- **行为型模式(对象与对象在干嘛)**： 

  – 模版方法模式、命令模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式、状态模式、策略模式、职责链模式、访问者模式。

- **设计模式的六大原则**
  - 开闭原则
    - **对扩展开放, 对修改关闭**; 在程序需要进行扩展的时候, 不能去修改原有的代码, 实现一个热插拔效果. 是为了使程序的扩展性好, 易于维护和升级. 想要到达这种效果, 需要使用接口和抽象类;
  - 里氏代换原则
    - 实现抽象的规范，实现子父类互相替换；
  - 依赖倒转原则
    - 这个原则是开闭原则的基础，具体内容：**针对接口编程，依赖于抽象而不依赖于具体的实现类**。
  - 接口隔离原则
    - 降低耦合度, 接口单独设计, 相互隔离
  - 迪米特法则
    - 又称不知道原则, 功能模块尽量独立；
  - 合成复用原则
    - 尽量使用合成/聚合的方式，而不是使用继承。



# 创建型模式

## 1.单例模式

- **核心作用**： 

  – **保证一个类只有一个实例，并且提供一个访问该实例的全局访问点。** 

- **常见应用场景**:

  – Windows的Task Manager（任务管理器）就是很典型的单例模式 

  – windows的Recycle Bin（回收站）也是典型的单例应用。在整个系统运行过程中，回收站一直维护着仅有的一个实例。 

  – 项目中，读取配置文件的类，一般也只有一个对象。没有必要每次使用配置文件数据，每次new一个对象去读取。 

  – 网站的计数器，一般也是采用单例模式实现，否则难以同步。 

  – 应用程序的日志应用，一般都何用单例模式实现，这一般是由于共享的日志文件一直处于打开状态，因为只能有一个实例去操作 ，否则内容不好追加。 

  – *数据库连接池的设计一般也是采用单例模式，因为数据库连接是一种数据库资源。*

  – 操作系统的文件系统，也是大的单例模式实现的具体例子，一个操作系统只能有一个文件系统。 

  – Application 也是单例的典型应用（Servlet编程中会涉及到） 

  – *在Spring中，每个Bean默认就是单例的*，这样做的优点是Spring容器可以管理 

  – 在servlet编程中，每个Servlet也是单例 

  – *在spring MVC框架/struts1框架中，控制器对象也是单例*

- **单例模式的优点**:

  – *由于单例模式只生成一个实例，**减少了系统性能开销**，当一个对象的产生需要 比较多的资源时，如读取配置、产生其他依赖对象时，则可以通过在应用启动 时直接产生一个单例对象，然后永久驻留内存的方式来解决* 

  – *单例模式可以在系统设置全局的访问点，**优化共享资源访问**，例如可以设计 一个单例类，负责所有数据表的映射处理* 

- **常见的五种单例模式实现方式**： 

  – 主要： 

  - 饿汉式（线程安全，调用效率高。 但是，不能延时加载。） 

  - **懒汉式（线程安全，调用效率不高。 但是，可以延时加载。）** 

  – 其他： 

  - 双重检测锁式（由于JVM底层内部模型原因，偶尔会出问题。不建议使用） 

  - **静态内部类式(线程安全，调用效率高。 但是，可以延时加载)** 
  - 枚举单例(线程安全，调用效率高，不能延时加载)

### 1.1 饿汉式:

> 饿汉式单例模式代码中，static变量会在类装载时初始化，此时也不会涉及多个线程对象访问该对象的问 
>
> 题。虚拟机保证只会装载一次该类，肯定不会发生并发访问的问题。因此，可以省略synchronized关键字。 

***问题：如果只是加载本类，而不是要调用getInstance()，甚至永远没有调用，则会造成资源浪费！***

```java
package cn.demo.singletion;

/**
 * 饿汉式单例模式
 * Create by Jjcc on 2019/6/27 22:31
 *
 * @author Jjcc
 */
public class SingletonDemo1 {

    /**
     * 设置成静态变量,类在加载时,在链接的准备阶段就分配内存并初始化默认值,
     * 初始化阶段,创建实例;并且类加载是线程安全的, 虚拟机只会加载一次该类
     */
    private static SingletonDemo1 sd = new SingletonDemo1();

    /**
     * 构造方法私有化,该类不能被创建实例对象
     */
    private SingletonDemo1(){}

    public static SingletonDemo1 getInstance() {
        return sd;
    }
}

class Test {
    public static void main(String[] args){
        SingletonDemo1 instance1 = SingletonDemo1.getInstance();
        SingletonDemo1 instance2 = SingletonDemo1.getInstance();
        System.out.println(instance1 == instance2);
    }
}
```

### 1.2 懒汉式

> 延迟加载， 懒加载！ 真正用的时候才加载！

***问题： 资源利用率高了。每次调用getInstance()方法都要同步检查，并发效率较低。***

```java
package cn.demo.singletion;

/**
 * 单例懒汉式
 * Create by Jjcc on 2019/6/27 23:05
 *
 * @author Jjcc
 */
public class SingletonDemo2 {

    private static SingletonDemo2 sd = null;

    /**
     * 构造方法私有化
     */
    private SingletonDemo2(){}

    /**
     * 线程安全的,效率较低;不调用该实例方法时,类不会初始化
     * @return
     */
    public static synchronized SingletonDemo2 getInstance() {
        if (sd == null) {
            sd = new SingletonDemo2();
        }

        return sd;
    }
}

```

### 1.3 双重检测锁

 这个模式将同步内容下方到if内部，提高了执行的效率 不必每次获取对象时都进行同步，只有第一次才同步 创建了以后就没必要了。 

***问题: 由于编译器优化原因和JVM底层内部模型原因,偶尔会出问题。不建议使用。*** 

```java
package cn.demo.singletion;

/**
 * 双重检测锁
 * Create by Jjcc on 2019/6/27 23:28
 *
 * @author Jjcc
 */
public class SingletonDemo3 {

    private static SingletonDemo3 sd = null;

    private SingletonDemo3() {}

    public static SingletonDemo3 getInstance() {
        if (sd == null) {
            //增加同步块
            synchronized (SingletonDemo3.class) {
                if (sd == null) {
                    sd = new SingletonDemo3();
                }
            }
        }
        return sd;
    }
}

```

#### 原子性

简单来说，原子操作（atomic）就是不可分割的操作，在计算机中，就是指不会因为线程调度被打断的操作。

比如，简单的赋值是一个原子操作：

```java
m = 6; // 这是个原子操作
```

假如m原先的值为0，那么对于这个操作，要么执行成功m变成了6，要么是没执行m还是0，而不会出现诸如m=3这种中间态——即使是在并发的线程中。

而，声明并赋值就不是一个原子操作：

```java
int n = 6; // 这不是一个原子操作
```

*对于这个语句，至少有两个操作：*

①声明一个变量n

②给n赋值为6

——**这样就会有一个中间状态：变量n已经被声明了但是还没有被赋值的状态。**

——这样，在多线程中，由于线程执行顺序的不确定性，如果两个线程都使用m，就可能会导致不稳定的结果出现。

*知识点：什么是指令重排？*

***简单来说，就是计算机为了提高执行效率，会做的一些优化，在不影响最终结果的情况下，可能会对一些语句的执行顺序进行调整。***

主要在于`singleton = new Singleton()`这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。

1.给 singleton 分配内存

2.调用 Singleton 的构造函数来初始化成员变量，形成实例

3.将singleton对象指向分配的内存空间（执行完这步 singleton才是非 null 了）

*但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后顺理成章地报错。*

#### 改良版(volatile)

给`private static Singleton sd = null`加上 `volatile`

```java
// Version 4 

public class Single4 {

    private static volatile Single4 instance;

    private Single4() {}

    public static Single4 getInstance() {

        if (instance == null) {

            synchronized (Single4.class) {

                if (instance == null) {

                    instance = new Single4();

                }

            }

        }
        return instance;
    }
}
```

volatile关键字的一个作用是禁止指令重排，把instance声明为volatile之后，对它的写操作就会有一个内存屏障（什么是内存屏障？），这样，**在它的赋值完成之前，就不用会调用读操作。**

**注意**：

volatile不是阻止的singleton = new Singleton()这句话内部[1-2-3]的指令重排，而是保证了在一个写操作（[1-2-3]）完成之前，不会调用读操作`（if (instance == null)）`。

### 1.4 静态内部类

– **外部类没有static属性，则不会像饿汉式那样立即加载对象。** 

– 只有真正调用getInstance(),才会加载静态内部类。加载类时是线程 安全的。 **instance是static final** 

**类型，保证了内存中只有这样一个实例存在**，而且只能被赋值一次，从而保证了线程安全性. 

– **兼备了并发高效调用和延迟加载的优势！**

```java
package cn.demo.singletions.review;

/**
 * 单例模式-静态内部类
 * 线程安全的；调用效率高；实现了延时加载；
 * Create by Jjcc on 2019/7/10 21:25
 *
 * @author Jjcc
 */
public class StaticInnerClassSingletonDemo1 {

    private StaticInnerClassSingletonDemo1() {
    }

    /**
     * 静态内部类
     */
    private static class StaticInnerClass {
        /**
         * final static修饰的，保证了内存中只有这样一个实例；
         * final修饰：值不会改变；
         * static修饰：实例只会初始化一次，ClassLoader机制一个类只会加载一次；
         */
        private final static StaticInnerClassSingletonDemo1 SICSD = new StaticInnerClassSingletonDemo1();

        /**
         * SICSD虽然使用了final修饰，但是常量池中只能引用到基本数据类型和String类型的字面量，所以该内部类还是会初始化
         * 初始化：执行类构造器：static变量与static域
         */
        static {
            System.out.println("ok");
        }
    }

    public static StaticInnerClassSingletonDemo1 getInstance() {
        return StaticInnerClass.SICSD;
    }
}
```

- 对于内部类SingletonHolder，它是一个饿汉式的单例实现，在SingletonHolder初始化的时候会由ClassLoader来保证同步，使INSTANCE是一个真·单例。
- 同时，由于SingletonHolder是一个内部类，只在外部类的Singleton的getInstance()中被使用，所以它被加载的时机也就是在getInstance()方法第一次被调用的时候。

它利用了ClassLoader来保证了同步，同时又能让开发者控制类加载的时机。从内部看是一个饿汉式的单例，但是从外部看来，又的确是懒汉式的实现。

### 1.5单例破解问题

- **反射可以破解上面几种单例(枚举方式除外)实现方式!**

  - 解决办法:

    -可以在构造方法中手动 抛出异常控制

    -饿汉式单例方式无法实现反破解

- **反序列化可以破解上面几种单例(枚举方式除外)实现方式!**

  - 解决办法:

    -**可以通过定义readResolve()防止获得不同对象。**

    -反序列化时，如果对象所在类定义了readResolve()，（实际是一种回调），定义返回哪个对象。 

**懒汉式单例模式，解决反射和反序列化漏洞:**

```java
package com.iter.devbox.singleton;
 
import java.io.ObjectStreamException;
import java.io.Serializable;
 
/**
 * 懒汉式（如何防止反射和反序列化漏洞）
 * @author Shearer
 *
 */
public class SingletonDemo6 implements Serializable{
	
	// 类初始化时，不初始化这个对象（延迟加载，真正用的时候再创建）
	private static SingletonDemo6 instance;
	
	private SingletonDemo6() {
		// 防止反射获取多个对象的漏洞
		if (null != instance) {
			throw new RuntimeException();
		}
	}
	
	// 方法同步，调用效率低
	public static synchronized SingletonDemo6 getInstance() {
		if (null == instance)
			instance = new SingletonDemo6();
		return instance;
	}
 
	// 防止反序列化获取多个对象的漏洞。
	// 无论是实现Serializable接口，或是Externalizable接口，当从I/O流中读取对象时，readResolve()方法都会被调用到。
	// 实际上就是用readResolve()中返回的对象直接替换在反序列化过程中创建的对象。
	private Object readResolve() throws ObjectStreamException {  
		return instance;
	}
}
 
 
package com.iter.devbox.singleton;
 
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
 
public class Client2 {
 
	public static void main(String[] args) throws Exception {
		SingletonDemo6 sc1 = SingletonDemo6.getInstance();
		SingletonDemo6 sc2 = SingletonDemo6.getInstance();
		System.out.println(sc1); // sc1，sc2是同一个对象
		System.out.println(sc2);
		
		// 通过反射的方式直接调用私有构造器（通过在构造器里抛出异常可以解决此漏洞）
/*		Class<SingletonDemo6> clazz = (Class<SingletonDemo6>) Class.forName("com.iter.devbox.singleton.SingletonDemo6");
		Constructor<SingletonDemo6> c = clazz.getDeclaredConstructor(null);
		c.setAccessible(true); // 跳过权限检查
		SingletonDemo6 sc3 = c.newInstance();
		SingletonDemo6 sc4 = c.newInstance();
		System.out.println(sc3);  // sc3，sc4不是同一个对象
		System.out.println(sc4);*/
		
		// 通过反序列化的方式构造多个对象（类需要实现Serializable接口）
		
		// 1. 把对象sc1写入硬盘文件
		FileOutputStream fos = new FileOutputStream("object.out");
		ObjectOutputStream oos = new ObjectOutputStream(fos);
		oos.writeObject(sc1);
		oos.close();
		fos.close();
		
		// 2. 把硬盘文件上的对象读出来
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream("object.out"));
		// 如果对象定义了readResolve()方法，readObject()会调用readResolve()方法。从而解决反序列化的漏洞
		SingletonDemo6 sc5 = (SingletonDemo6) ois.readObject();
		// 反序列化出来的对象，和原对象，不是同一个对象。如果对象定义了readResolve()方法，可以解决此问题。
		System.out.println(sc5); 
		ois.close();
	}
 
}
```

**静态内部类式单例模式（解决反射和反序列化漏洞）**

```java
package com.iter.devbox.singleton;
 
import java.io.ObjectStreamException;
import java.io.Serializable;
 
/**
 * 静态内部类实现方式（也是一种懒加载方式）
 * 这种方式：线程安全，调用效率高，并且实现了延迟加载
 * 解决反射和反序列化漏洞
 * @author Shearer
 *
 */
public class SingletonDemo7 implements Serializable{
	
	private static class SingletonClassInstance {
		private static final SingletonDemo7 instance = new SingletonDemo7();
	}
	
	// 方法没有同步，调用效率高
	public static SingletonDemo7 getInstance() {
		return SingletonClassInstance.instance;
	}
	
	// 防止反射获取多个对象的漏洞
	private SingletonDemo7() {
		if (null != SingletonClassInstance.instance)
			throw new RuntimeException();
	}
	
	// 防止反序列化获取多个对象的漏洞
	private Object readResolve() throws ObjectStreamException {  
		return SingletonClassInstance.instance;
	}
}
 
 
package com.iter.devbox.singleton;
 
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Constructor;
 
public class Client3 {
 
	public static void main(String[] args) throws Exception {
		SingletonDemo7 sc1 = SingletonDemo7.getInstance();
		SingletonDemo7 sc2 = SingletonDemo7.getInstance();
		System.out.println(sc1); // sc1，sc2是同一个对象
		System.out.println(sc2);
		
		// 通过反射的方式直接调用私有构造器（通过在构造器里抛出异常可以解决此漏洞）
		Class<SingletonDemo7> clazz = (Class<SingletonDemo7>) Class.forName("com.iter.devbox.singleton.SingletonDemo7");
		Constructor<SingletonDemo7> c = clazz.getDeclaredConstructor(null);
		c.setAccessible(true); // 跳过权限检查
		SingletonDemo7 sc3 = c.newInstance();
		SingletonDemo7 sc4 = c.newInstance();
		System.out.println("通过反射的方式获取的对象sc3：" + sc3);  // sc3，sc4不是同一个对象
		System.out.println("通过反射的方式获取的对象sc4：" + sc4);
		
		// 通过反序列化的方式构造多个对象（类需要实现Serializable接口）
		
		// 1. 把对象sc1写入硬盘文件
		FileOutputStream fos = new FileOutputStream("object.out");
		ObjectOutputStream oos = new ObjectOutputStream(fos);
		oos.writeObject(sc1);
		oos.close();
		fos.close();
		
		// 2. 把硬盘文件上的对象读出来
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream("object.out"));
		// 如果对象定义了readResolve()方法，readObject()会调用readResolve()方法。从而解决反序列化的漏洞
		SingletonDemo7 sc5 = (SingletonDemo7) ois.readObject();
		// 反序列化出来的对象，和原对象，不是同一个对象。如果对象定义了readResolve()方法，可以解决此问题。
		System.out.println("对象定义了readResolve()方法，通过反序列化得到的对象：" + sc5); 
		ois.close();
	}
 
}
```

### 1.6使用枚举方式

- 优点： 

  – 实现简单 

  – 枚举本身就是单例模式。由JVM从根本上提供保障！避免通过反射和反序列化的漏洞！ 

- 缺点： 

  – 无延迟加载

  – 与饿汉单例类似, 在加载时就已经创建了

```java
package com.bjsxt.singleton;

/**
 * 测试枚举式实现单例模式(没有延时加载)
 * @author 尚学堂高淇 www.sxt.cn
 *
 */
public enum SingletonDemo5 {
	
	//这个枚举元素，本身就是单例对象！
	INSTANCE;
	
	//添加自己需要的操作！
	public void singletonOperation(){
	}
}
```

### 总结

- 常见的五种单例模式实现方式 
  - 主要：  
    - 饿汉式（线程安全，调用效率高。 但是，不能延时加载。） 
    - 懒汉式（线程安全，调用效率不高。 但是，可以延时加载。）
  - 其它:
    - 双重检测锁式（由于JVM底层内部模型原因，偶尔会出问题。**不建议使用**） 
    - 静态内部类式(线程安全，调用效率高。 但是，可以延时加载) 
    - 枚举式(线程安全，调用效率高，不能延时加载。并且可以天然的防止反射和反序列化漏洞！) 

- 如何选用? 
  - – 单例对象 占用 资源 少，不需要 延时加载： 
    - 枚举式 好于 饿汉式 
  - 单例对象 占用 资源 大，需要 延时加载： 
    - 静态内部类式 好于 懒汉式



## 2.工厂模式

- 工厂模式
  - **实现了创建者和调用者的分离**
  - 详细分类
    - 简单工厂模式
    - 工厂模式
    - 抽象工厂模式
- 面向对象设计的基本原则
  - OPC(开闭原则, Open-Closed Principle)
    - **一个软件的实体应当对扩展开放, 对修改关闭**
  - DIP(依赖倒转原则, Dependence Inversion Principle)
    - 要针对接口编程, 不要针对实现编程
  - Lod(迪米特法则，Law of Demeter)
    - 只与你直接的朋友通信, 而避免和陌生人通信
- 核心本质
  - 实例化对象, 用工厂方法代替new操作
  - 将选择实现类, 创建对象统一管理和控制. 从而将调用者跟我们的实现类解耦
- 工厂模式
  - 简单工厂模式
    - *用来生产同一等级结构中的任意产品*(对于增加新的产品, 需要修改已有的代码)
  - 工厂模式
    - *用来生产同一等级结构中的固定产品*(支持增加任意产品)
  - 抽象工厂模式
    - *用来生产不同产品族的全部产品*(对于增加新的产品，无能为力；支持增加产品族)

### 2.1 简单工厂模式

- **特点**:

  简单工厂模式又 叫静态工厂方法模式（Static FactoryMethod Pattern）,是**通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。** 

- **缺点**:

  对于增加新产品无能为力！不修改代码的话，是无法扩展的。

  ![](.\img\SimpleFactoryDemo1.png)

```java
package cn.demo.factorys.simplefactory;

import com.sun.org.apache.xpath.internal.operations.And;

/**
 * 简单工厂模式
 * Create by Jjcc on 2019/6/30 22:24
 *
 * @author Jjcc
 */
public class SimpleFactoryDemo1 {
    public static void main(String[] args){
        SimpleFactory simpleFactory = new SimpleFactory();
        Phone phone = simpleFactory.getPhone("apple");
        phone.call();
    }
}

/**
 * 手机接口
 */
interface Phone {
    void call();
}

/**
 * 苹果手机
 */
class Apple implements Phone {
    @Override
    public void call() {
        System.out.println("苹果手机");
    }
}

/**
 * 安卓手机
 */
class Android implements Phone {
    @Override
    public void call() {
        System.out.println("安卓手机");
    }
}
/**
 * 工厂类,创建对象的类
 */
class SimpleFactory {
    public Phone getPhone(String phoneType) {
        Phone p = null;
        switch (phoneType) {
            case "apple":
                p = new Apple();
                break;
            case "android":
                p = new Android();
                break;
            default:
        }
        return p;
    }

}
```

### 2.2 工厂方法模式

**工厂方法 *定义一个用于创建对象的接口*，让子类决定实例化哪一个类，工厂方法使得一个类的实例化延迟到了子类 **
工厂方法在简单工厂的基础上再包了一层工厂，所有的工厂都是此工厂的子类。而产生对象的类型由子类工厂决定。

- **要点**

  为了避免简单工厂模式的缺点，不完全满足OCP。 

  工厂方法模式和简单工厂模式最大的不同在于，简单工厂模式只有一个（对于一个项目 

  或者一个独立模块而言）工厂类，而工厂方法模式有一组实现了相同接口的工厂类。

- **模式要素**

  - ***提供一个产品类的接口***。产品类均要实现这个接口(也可以是abstract类，即抽象产品)。
  - ***提供一个工厂类的接口***。工厂类均要实现这个接口(即抽象工厂)。
  - 由工厂实现类创建产品类的实例。工厂实现类应有一个方法，用来实例化产品类。

![](.\img\Client1.png)

```java
package cn.demo.factorys.factorymethod;

/**
 * Create by Jjcc on 2019/7/2 18:53
 *
 * @author Jjcc
 */
public class FactoryMethodDemo1 {
    public static void main(String[] args){
        Phone1 phone = new AppleFactory1().createPhone();
        phone.call();
    }

}

/**
 * 手机类
 */
interface Phone1 {
    void call();
}

/**
 * 安卓手机
 */
class Android1 implements Phone1 {
    @Override
    public void call() {
        System.out.println("Android手机");
    }
}

/**
 * 苹果手机
 */
class Apple1 implements Phone1 {
    @Override
    public void call() {
        System.out.println("Apple手机");
    }
}

/**
 * 创建手机类的工厂
 */
interface PhoneFactory1 {
    Phone1 createPhone();
}

/**
 * 安卓手机实例创建工厂
 */
class AndroidFactory1 implements PhoneFactory1 {
    @Override
    public Phone1 createPhone() {
        return new Android1();
    }
}

/**
 * 苹果手机实例创建工厂
 */
class AppleFactory1 implements PhoneFactory1 {
    @Override
    public Phone1 createPhone() {
        return new Apple1();
    }
}
```

### 2.3 抽象工厂模式

提供一个**创建一系列相关或相互依赖对象的接口，而无需指定他们具体的类。抽象工厂为不同产品族的对象创建提供接口。 **

- 抽象工厂模式 

  – 用来生产不同产品族的全部产品。（*对于增加新的产品，无能为力；支持增加产品族*） 

  – *抽象工厂模式是工厂方法模式的升级版本，在有多个业务品种、业务分类时，通过抽象工厂模式产生需要的对象是一种非常好的解决方式。*

在工厂方法模式中具体工厂负责生产具体的产品，每一个具体工厂对应一种具体产品，工厂方法具有唯一性，一般情况下，一个具体工厂中只有一个或者一组重载的工厂方法。但是有时候我们希望一个工厂可以提供多个产品对象，而不是单一的产品对象，如一个电器工厂，它可以生产电视机、电冰箱、空调等多种电器，而不是只生产某一种电器。

- 产品等级结构

  **产品等级结构即产品的继承结构**，如一个抽象类是电视机，其子类有海尔电视机、海信电视机、TCL电视机，则抽象电视机与具体品牌的电视机之间构成了一个产品等级结构，抽象电视机是父类，而具体品牌的电视机是其子类。

- 产品族

  在抽象工厂模式中，**产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品**，如海尔电器工厂生产的海尔电视机、海尔电冰箱，海尔电视机位于电视机产品等级结构中，海尔电冰箱位于电冰箱产品等级结构中，海尔电视机、海尔电冰箱构成了一个产品族。

![img](.\img\SouthEast)

在上图中，不同颜色的多个正方形、圆形和椭圆形分别构成了三个不同的产品等级结构，而相同颜色的正方形、圆形和椭圆形构成了一个产品族，每一个形状对象都位于某个产品族，并属于某个产品等级结构。图3中一共有五个产品族，分属于三个不同的产品等级结构。我们只要指明一个产品所处的产品族以及它所属的等级结构，就可以唯一确定这个产品。

![img](.\img\SouthEast1.png)

在上图中, 当系统所提供的工厂生产的具体产品并不是一个简单的对象，而是多个位于不同产品等级结构、属于不同类型的具体产品时就可以使用抽象工厂模式。**抽象工厂模式是所有形式的工厂模式中最为抽象和最具一般性的一种形式。**抽象工厂模式与工厂方法模式最大的区别在于，工厂方法模式针对的是一个产品等级结构，而抽象工厂模式需要面对多个产品等级结构，一个工厂等级结构可以负责多个不同产品等级结构中的产品对象的创建。**当一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象时，抽象工厂模式比工厂方法模式更为简单、更有效率。**

在上图中，**每一个具体工厂可以生产属于一个产品族的所有产品**，例如生产颜色相同的正方形、圆形和椭圆形，所生产的产品又位于不同的产品等级结构中。如果使用工厂方法模式，图4所示结构需要提供15个具体工厂，而使用抽象工厂模式只需要提供5个具体工厂，极大减少了系统中类的个数。

![](.\img\AbstractFactoryDemo2.png)

```java
package cn.demo.factorys.abstractfactory1.abstractfactory2;

/**
 * 抽象工厂模式;
 * 需要多个定义产品的接口,实现同一产品接口的多个实现类称之为产品等级结构;
 * 需要定义一个产品族工厂接口,工厂接口的实现类可以自由搭配产品等级结构中的产品
 * Create by Jjcc on 2019/7/2 22:50
 *
 * @author Jjcc
 */
public class AbstractFactoryDemo1 {
    public static void main(String[] args){
        ICarFactory benzCarFactory = new BenzCarFactory();
        IEngine engine = benzCarFactory.createEngine();
        ISeat seat = benzCarFactory.createSeat();
        ITyre tyre = benzCarFactory.createTyre();
        engine.brand();
        seat.brand();
        tyre.brand();

        System.out.println("------------------");

        ICarFactory audiCarFactory = new AudiCarFactory();
        IEngine engine1 = audiCarFactory.createEngine();
        ISeat seat1 = audiCarFactory.createSeat();
        ITyre tyre1 = audiCarFactory.createTyre();
        engine1.brand();
        seat1.brand();
        tyre1.brand();

    }
}

/**
 * 产品等级结构 - 引擎接口
 */
interface IEngine {
    /**
     * 品牌
     */
    void brand();
}

/**
 * 产品等级结构 - 具体产品实现类
 */
class BenzEngineImpl implements IEngine {
    @Override
    public void brand() {
        System.out.println("奔驰引擎!");
    }
}
class AudiEngineImpl implements IEngine {
    @Override
    public void brand() {
        System.out.println("奥迪引擎!");
    }
}

/**
 * 产品等级结构 - 轮胎接口
 */
interface ITyre {
    /**
     * 品牌
     */
    void brand();
}

/**
 * 产品等级结构 - 轮胎具体实现类
 */
class BenzTyreImpl implements ITyre {
    @Override
    public void brand() {
        System.out.println("奔驰轮胎!");
    }
}
class AudiTyreImpl implements ITyre {
    @Override
    public void brand() {
        System.out.println("奥迪轮胎!");
    }
}

/**
 * 产品等级结构 - 座椅接口
 */
interface ISeat {
    /**
     * 品牌
     */
    void brand();
}

/**
 * 产品等级结构 - 具体实现类
 */
class BenzSeatImpl implements ISeat {
    @Override
    public void brand() {
        System.out.println("奔驰座椅!");
    }
}
class AudiSeatImpl implements ISeat {
    @Override
    public void brand() {
        System.out.println("奥迪座椅!");
    }
}

/**
 * 抽象工厂类
 */
interface ICarFactory {

    /**
     * 选择引擎产品等级结构中哪一个产品
     * @return IEngine
     */
    IEngine createEngine();

    /**
     * 选择轮胎产品等级结构中哪一个产品
     * @return
     */
    ITyre createTyre();

    /**
     * 选择座椅产品等级结构中哪一个产品
     * @return
     */
    ISeat createSeat();
}

/**
 * 产品族;
 */
class BenzCarFactory implements ICarFactory {
    @Override
    public IEngine createEngine() {
        return new BenzEngineImpl();
    }

    @Override
    public ITyre createTyre() {
        return new BenzTyreImpl();
    }

    @Override
    public ISeat createSeat() {
        return new BenzSeatImpl();
    }
}

class AudiCarFactory implements ICarFactory {
    @Override
    public IEngine createEngine() {
        return new AudiEngineImpl();
    }

    @Override
    public ITyre createTyre() {
        return new AudiTyreImpl();
    }

    @Override
    public ISeat createSeat() {
        return new AudiSeatImpl();
    }
}
```

抽象工厂最大的好处就是便于交换产品系列，具体工厂在代码中一般只出现一次。这就使得改变应用的具体工厂很容易。 
第二个好处是他能让**具体的创建对象实例和客户端分离**，客户端是通过他们的抽象接口操作实例 
**抽象工厂不太易于拓展，如果需要自增功能，或者自增产品，则需要至少修改三个类，而且实例化的代码是写死在程序中的 ， 这样无法避免违背开放-关闭原则。** 

对于上述问题，可以通过配置文件，结合反射的方式来解决。

### 总结

- 简单工厂模式(静态工厂模式) 
  - 虽然某种程度不符合设计原则，但实际使用最多。 

- 工厂方法模式 
  - 不修改已有类的前提下，通过增加新的工厂类实现扩展。 

- 抽象工厂模式 
  - 不可以增加产品，可以增加产品族！ 



## 3. 原型设计模式

- 通过new产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。 
- 就是java中的克隆技术，以某个对象为原型，复制出新的对象。显然，新的对象具备原型对象的特点 
- 优势有：效率高(直接克隆，避免了重新执行构造过程步骤) 。 
- 克隆类似于new，但是不同于new。new创建新的对象属性采用的是默认值。克隆出的对象的属性值完全和原型对象相同。并且克隆出的新对象改变不会影响原型对象。然后,再修改克隆对象的值。

### 3.1 原型模式实现

任何类，要想支持克隆，必须实现一个接口 `Cloneable`，该接口中有`clone()`方法，可以在类中重写自定义的克隆方法。

***基本数据类型和String能够自动实现深度克隆（值的复制）***

- 浅克隆
  - 浅拷贝是指在拷贝对象时，对于基本数据类型的变量会重新复制一份，而对**于引用类型的变量只是对直接引用进行拷贝，没有对直接引用指向的对象进行拷贝。**
- 深克隆
  - 深拷贝是指在拷贝对象时,不仅把基本数据类型的变量会重新复制一份，同时会对引用指向的对象进行拷贝。
- 完全克隆
  - 在包括上面两者共同点的基础上把对象间接引用的对象也进行拷贝(*对象中的子对象*)。这是最彻底的一种拷贝。通常先使对象序列化，实现Serializable接口 然后将对象写进二进制流里 再从二进制流里读出新对象。

![1562571828972](.\img\sd2asd2k913.png)

```java
package cn.demo.factorys.prototype;


import java.io.*;
import java.util.StringTokenizer;

/**
 * 简单工厂模式与原型模式Demo;
 * Create by Jjcc on 2019/7/5 16:41
 *
 * @author Jjcc
 */
public class PrototypeDemo2 {
    public static void main(String[] args){
        /**
         * 类实现Cloneable接口并重写clone()方法实现克隆
         */
        IPerson yellow1 = PersonFactory.createInstance("yellow");
        IPerson yellow2 = PersonFactory.createInstance("yellow");
        System.out.println(yellow1 == yellow2);


        /**
         * 使用序列化和反序列化实现深复制
         */
        Car car = new Car();
        car.setType("11");
        car.setName("Benz");
        try (ByteArrayOutputStream arrayOutputStream = new ByteArrayOutputStream();
             ObjectOutputStream objectOutputStream = new ObjectOutputStream(arrayOutputStream);
             ) {
            objectOutputStream.writeObject(car);

            byte[] bytes = arrayOutputStream.toByteArray();

            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);

            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);

            Car car1 = (Car) objectInputStream.readObject();

            System.out.println(car1);

            objectInputStream.close();
            byteArrayInputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }
}

class Car implements Serializable {
    private String name;
    private String type;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }
    @Override
    public String toString() {
        return "Car{" +
                "name='" + name + '\'' +
                ", type='" + type + '\'' +
                '}';
    }
}

interface IPerson {
    /**
     * 肤色
     * @return
     */
    void color();
}

class YellowPersonImpl implements IPerson,Cloneable {
    public YellowPersonImpl() {
        System.out.println("执行构造方法!");
    }

    @Override
    public void color() {
        System.out.println("黄皮肤的");
    }
	
    /**
     * 重写clone()方法
     */
    @Override
    protected Object clone() {
        Object o = null;
        try {
            o = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return o;
    }
}

class WhitePersonImpl implements IPerson,Cloneable {

    @Override
    public void color() {
        System.out.println("白皮肤的");
    }
}

/**
 * 简单工厂模式与克隆模式
 */
class PersonFactory {
    static YellowPersonImpl ypi = null;

    public static IPerson createInstance(String type) {
        IPerson person = null;
        switch (type) {
            case "yellow" :
                if (ypi == null) {
                    ypi = new YellowPersonImpl();
                    person = ypi;
                } else {
                    YellowPersonImpl clone = (YellowPersonImpl) ypi.clone();
                    person = clone;
                }
                break;
            case "white":
                person = new WhitePersonImpl();
                break;
            default:
        }

        return person;
    }
}
```

### 3.2 clone的特点

- 克隆的对象与原对象不是同一个对象, 分别占用不用的内存空间 `ojb.clone() != obj;`
- 克隆的对象与原对象的类型一样 `obj.clone().getClass() == obj.getClass();`
- 克隆对象不会调用constructor方法;

### 总结

- 提高性能
  - **使用原型模式创建对象比直接new一个对象在性能上要好的多**，因为Object类的clone方法是一个本地方法，它**直接操作内存中的二进制流，特别是复制大对象时，性能的差别非常明显。**
- 简化对象的创建
  - 因为以上优点，所以在**需要重复地创建相似对象时可以考虑使用原型模式。比如需要在一个循环体内创建对象，假如对象创建过程比较复杂或者循环次数很多的话**，使用原型模式不但以使系统的整体性能提高很多而且可以简化创建过程
- 逃避构造函数的约束
  - **使用原型模式复制对象不会调用类的构造方法。**因为对象的复制是通过调用Object类的clone方法来完成的，它**直接在内存中复制数据**，因此不会调用到类的构造方法。



## 创建型模式总结

**创建型模式都是用来帮我们创建对象的;**

**创建型模式关注对象的创建过程。**

- 单例模式

  - 保证一个类只有一个实例,并提供一个访问该实例的全局访问点

- 工厂模式

  - 简单工厂模式
    - 用来生产同一等级结构中的任意产品;(对于增加新的产品,需要修改已有代码);
  - 工厂方法模式
    - 用来生产同一等级结构中的固定产品(支持增加任意产品);
  - 抽象工厂模式
    - 用来生产不同产品族的全部产品(对于增加新的产品,无能为力;支持增加产品族);

- 建造者模式

  - 分离了对象子组件的单独构造(由Builder来负责)和装配(由Director负责)。 从而可 

    以构造出复杂的对象。 

- 原型模式

  - 通过new产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式;



# 结构型模式

- **核心作用**

  **是从程序的结构上实现松耦合，从而可以扩大整体的类结构，用来解决更大的问题。** 

- **结构型模式关注对象和类的组织。**

- **结构型模式汇总**

  |   模式   | 介绍                                                         |
  | :------: | :----------------------------------------------------------- |
  | 适配模式 | 使原本由于接口不兼容不能一起工作的类可以一起工作;            |
  | 代理模式 | 为真实对象提供一个代理,从而控制对真实对象的访问;             |
  | 组合模式 | 将对象组合成树状结构以表示”部分和整体”层次结构，使得客户可以统一 的调用叶子对象和容器对象 |
  | 桥接模式 | 处理多层继承结构，处理多维度变化的场景，将各个维度设计成独立的继承结构，使各个维度可以独立的扩展在抽象层建立关联。 |
  | 装饰模式 | 动态地给一个对象添加额外的功能，比继承灵活                   |
  | 外观模式 | 为子系统提供统一的调用接口，使得子系统更加容易使用           |
  | 享元模式 | 运用共享技术有效的实现管理大量细粒度对象，节省内存，提高效率 |

## 1.适配器模式

适配器模式是一种**结构型**设计模式。适配器模式的思想是：**把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作**。

> 用电器来打个比喻：有一个电器的插头是三脚的，而现有的插座是两孔的，要使插头插上插座，我们需要一个插头转换器，这个转换器即是适配器。

适配器模式设计3种角色

- 源(Adaptee)

  需要被适配的对象或类型, 相当于插头;

- 适配器(Adapter)

  连接目标和源的中间对象, 相当于插头转换器

- 目标(Target)

  期待得到的目标, 相当于插座

**适配器模式包括3种形式：类适配器模式、对象适配器模式、接口适配器模式（或又称作缺省适配器模式）。** 

### 1.1 类适配器模式

从下面的结构图可以看出，`Adaptee`类并没有`method2()`方法，而客户端则期待这个方法。为使客户端能够使用Adaptee类，我们把`Adaptee`与`Target`衔接起来。`Adapter`与`Adaptee`是**继承关系**，这决定了这是一个类适配器模式。 

> Java 是单继承机制，所以类适配器需要继承 resource 类这一点算是一个缺点, 因为这要求 dst 必须是接口，有一定局限性;
>
> resource  类的方法在 Adapter 中都会暴露出来，也增加了使用的成本。

![è¿éåå¾çæè¿°](.\img\asdasd1231232)

```java
package cn.demo.adapter;

/**
 * adapter-类适配器
 * 适配器继承Adaptee（源类）并实现Target（目标接口）；
 * Create by Jjcc on 2019/7/8 21:36
 *
 * @author Jjcc
 */
public class AdapterDemo1 {

    public static void main(String[] args){
        Adapter adapter = new Adapter();
        test(adapter);
    }

    public static void test(ITarget adapter) {
        adapter.method1();
        adapter.method2();
    }
}

/**
 * 期待得到的目标
 */
interface ITarget {

    /**
     * 方法1
     */
    void method1();

    /**
     * 方法2
     */
    void method2();
}

/**
 * 需要被适配的对象或类型(源)
 */
class Adaptee {
    /**
     * 方法1
     * 如果子类中不存在该方法的实现[或者覆盖],在使用该类对象调用该方法的时候，就会使用从父类继承的方		 * 法。同时将这个从父类继承来的方法当作接口方法的实现，也就可以不再实现接口的方法体。
     * 接口的优先级别要高于父类。
     */
    public void method1(){
        System.out.println("method 1");
    }
}

/**
 * 适配器；
 * 如果实现类不重写接口中的方法，则把从父类中继承的方法当做接口方法的实现
 */
class Adapter extends Adaptee implements ITarget {
    @Override
    public void method2() {
        System.out.println("method 2");
    }
}
```

### 1.2 对象适配器模式

对象适配器模式是另外6种结构型设计模式的起源。 

![è¿éåå¾çæè¿°](.\img\sdfsdg31asdasd)

从下面的结构图可以看出，`Adaptee`类并没有`method2()`方法，而客户端则期待这个方法。与类适配器模式一样，为使客户端能够使用Adaptee类，我们把`Adaptee`与`Target`衔接起来。但**这里我们不继承`Adaptee`，而是把`Adaptee`封装进`Adapter`里。这里`Adaptee`与`Adapter`是组合关系。** 

![è¿éåå¾çæè¿°](.\img\asdasdsss21.png)

```java
package cn.demo.adapter;

/**
 * 对象适配器
 * Create by Jjcc on 2019/7/8 23:12
 *
 * @author Jjcc
 */
public class ObjectAdapterDemo1 {
    public static void main(String[] args){

        Adaptee2 adaptee2 = new Adaptee2();
        ITarget2 adapter2 = new Adapter2(adaptee2);

        test(adapter2);
    }

    public static void test(ITarget2 adapter2) {
        adapter2.method1();
        adapter2.mehtod2();
    }
}

/**
 * 目标
 */
interface ITarget2 {
    void method1();
    void mehtod2();
}

/**
 * 需要被适配的对象或类
 */
class Adaptee2 {
    public void method1() {
        System.out.println("method 1");
    }
}

/**
 * 适配器
 */
class Adapter2 implements ITarget2 {

    private Adaptee2 adaptee;

    @Override
    public void mehtod2() {
        System.out.println("method 2");
    }

    public Adapter2(Adaptee2 adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void method1() {
        adaptee.method1();
    }
}
```

**类适配器与对象适配器的区别**

- 类适配器使用的是继承的方式, 直接继承了`Adaptee`, 所以无法对`Adaptee`的子类进行适配;
- 对象适配器使用的是组合方式, 所以`Adaptee`及其子孙类都可以被适配.另外, 对象适配器对象对于增加一些新行为非常方便, 而且新增加的行为同时适用于所有的源;

基于组合/聚合优于继承的原则，使用对象适配器是更好的选择。但具体问题应该具体分析，某些情况可能使用类适配器会适合，最适合的才是最好的。

### 1.3 接口适配器模式

接口适配器模式（缺省适配模式）的思想是，为一个接口提供缺省实现，这样子类可以从这个缺省实现进行扩展，而不必从原有接口进行扩展。

> 有时我们写的一个接口中有多个抽象方法，当我们写该接口的实现类时，必须实现该接口的所有方法，这明显有时比较浪费，因为并不是所有的方法都是我们需要的，有时只需要某一些，此处为了解决这个问题，我们引入了接口的适配器模式，**借助于一个抽象类，该抽象类实现了该接口，实现了所有的方法，而我们不和原始的接口打交道，只和该抽象类取得联系**，所以我们**写一个类，继承该抽象类，重写我们需要的方法就行。**

**在任何时候，如果不准备实现一个接口里的所有方法时，就可以使用“缺省适配模式”制造一个抽象类，实现所有方法，这样，从这个抽象类再继承下去的子类就不必实现所有的方法，只要重写需要的方法就可以了。**

```java
/**
 * 定义端口接口，提供通信服务
 */
interface Port {
    /**
     * 远程SSH端口为22
     */
    void SSH();

    /**
     * 网络端口为80
     */
    void NET();

    /**
     * Tomcat容器端口为8080
     */
    void Tomcat();

    /**
     * MySQL数据库端口为3306
     */
    void MySQL();
}

/**
 * 定义抽象类实现端口接口，但是什么事情都不做
 */
abstract class Wrapper implements Port {
    @Override
    public void SSH() {

    }

    @Override
    public void NET() {

    }

    @Override
    public void Tomcat() {

    }

    @Override
    public void MySQL() {

    }
}

/**
 * 提供聊天服务
 * 需要网络功能
 * 需要什么方法就重写什么方法
 */
class Chat extends Wrapper {
    @Override
    public void NET() {
        System.out.println("Hello World...");
    }
}

/**
 * 网站服务器
 * 需要Tomcat容器，Mysql数据库，网络服务，远程服务
 */
class Server extends Wrapper {
    @Override
    public void SSH() {
        System.out.println("Connect success...");
    }

    @Override
    public void NET() {
        System.out.println("WWW...");
    }

    @Override
    public void Tomcat() {
        System.out.println("Tomcat is running...");
    }

    @Override
    public void MySQL() {
        System.out.println("MySQL is running...");
    }
}

public class AdapterPattern {

    private static Port chatPort = new Chat();
    private static Port serverPort = new Server();

    public static void main(String[] args) {
        // 聊天服务
        chatPort.NET();

        // 服务器
        serverPort.SSH();
        serverPort.NET();
        serverPort.Tomcat();
        serverPort.MySQL();
    }
}
```

```java
package cn.demo.adapter;

import javax.swing.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

/**
 * 接口类适配器
 * Create by Jjcc on 2019/7/9 21:26
 *
 * @author Jjcc
 */
public class InterfaceAdapterDemo1 {

    public static void main(String[] args){
        test(new BaseAdapter5() {
            @Override
            public void method2() {
                System.out.println("method 2");
            }
        });
    }

    public static void test(BaseAdapter5 ba) {
        ba.method1();
    }
}

/**
 * 被适配的类;源
 */
class Adaptee5 {
    public void method1() {
        System.out.println("method 1");
    }
}

/**
 * 接口；客户所期待的接口类型
 */
interface ITarget5 {
    void method1();
    void method2();
}

/**
 * 适配器；定义抽象类实现端口接口，但是什么事情都不做；
 * 这里抽象类BaseAdapter5继承了Adaptee5，Adaptee5的方法method1()相当于是重写了接口ITarget5的抽象方法method1()；
 */
abstract class BaseAdapter5 extends Adaptee5 implements ITarget5{
    @Override
    public void method2() {

    }
}

/**
 * 基类的子类；需要什么方法就重写什么方法
 */
class MethodOne extends BaseAdapter5 {
    @Override
    public void method2() {
        System.out.println("method 2");
    }
}
```

- 工作中的场景
  - 经常用来做旧系统升级和改造
- 学习中见过的场景
  - java.io.InputStreamReader(InputStream is);
  - java.io.OutputStreamWriter(OutputStream os);

### 总结

**适配器模式的优缺点：**

- 优点
  - 更好的复用性
    - 系统需要使用现有的类，而此类的接口不符合系统的需求。那么通过适配器模式就可以让这些功能得到更好的复用。
  - 更好的扩展性
    - 在实现适配器功能的时候，可以扩展自己源（Adaptee）的行为（增加方法），从而自然的扩展系统的功能。
- 缺点
  - 会导致系统紊乱
    - 滥用适配器，会让系统变得非常零乱。例如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。



## 2.桥接模式

**手机操作问题：**

现在对不同手机类型的不同品牌实现操作编程(比如:开机、关机、上网，打电话等)，如图:

![1567869340473](.\img\1567869340473.png)

**传统方案解决手机操作问题：**

传统方法对应的类图

![1567869389592](.\img\1567869389592.png)

**传统方案解决手机操作问题分析：**

1. 扩展性问题( 类爆炸)，如果我们再增加手机的样式(旋转式)，就需要增加各个品牌手机的类，同样如果我们增加一个手机品牌，也要在各个手机样式类下增加。
2. 违反了单一职责原则，当我们增加手机样式时，要同时增加所有品牌的手机，这样增加了代码维护成本.
3. 手机样式与品牌是双维度扩展的；**解决方案-使用 桥接模式**

### 2.1 基本介绍：

> 桥接模式(Bridge 模式)是指：**将实现与抽象放在两个不同的类层次中，使两个层次可以独立改变。**
>
> Bridge 模式基于类的最小设计原则，通过使用封装、聚合及继承等行为让不同的类承担不同的职责。它的**主要特点是把抽象(Abstraction)与行为实现(Implementation)分离开来，从而可以保持各部分的独立性以及应对他们的功能扩展；**

### 2.3 桥接模式-原理类图

<img src=".\img\s1aoksfpo2112.png" alt="1566446264299" style="zoom:125%;" />

**上图做了说明**

1. Client 类：桥接模式的调用者
2. 抽象类(Abstraction) :维护了 Implementor / 即它的实现类 ConcreteImplementorA.., 二者是聚合关系, Abstraction充当桥接类
3. RefinedAbstraction : 是 Abstraction 抽象类的子类
4. Implementor：行为实现类的接口
5. ConcreteImplementorA/B：行为的具体实现类
6. 从 UML 图：**这里的抽象类和接口是聚合的关系，其实是调用和被调用关系**

### 2.4 案例

![](.\img\BasePhone.png)

```java
package cn.demo.bridges.review;

/**
 *
 * 品牌接口
 * @author Jjcc
 * @version 1.0.0
 * @Description
 * @ClassName IBrand.java
 * @createTime 2019年08月22日 15:21:00
 */
public interface IBrand {
    
    /**
     * 开机
     * @title open
     * @description 
     * @author Jjcc 
     * @return void
     * @createTime 2019/8/22 15:28
     * @throws 
     */
    void open();
    
    /**
     * 关机
     * @title close
     * @description 
     * @author Jjcc 
     * @return void
     * @createTime 2019/8/22 15:29
     * @throws 
     */
    void close();
    
    /**
     * 打电话
     * @title call
     * @description 
     * @author Jjcc 
     * @return void
     * @createTime 2019/8/22 15:29
     * @throws 
     */
    void call();
    
}

class VivoImple implements IBrand {
    @Override
    public void open() {
        System.out.println("vivo手机开机！");
    }

    @Override
    public void close() {
        System.out.println("vivo手机关机！");
    }

    @Override
    public void call() {
        System.out.println("vivo手机打电话！");
    }
}

class XiaoMiImple implements IBrand {
    @Override
    public void open() {
        System.out.println("小米手机开机！");
    }

    @Override
    public void close() {
        System.out.println("小米手机关机！");
    }

    @Override
    public void call() {
        System.out.println("小米手机打电话！");
    }
}

```

```java
package cn.demo.bridges.review;

/**
 * 桥接类-手机类
 * @author Jjcc
 * @version 1.0.0
 * @Description
 * @ClassName BasePhone.java
 * @createTime 2019年08月22日 15:32:00
 */
public abstract class BasePhone {

    /**
     * 组合-品牌
     */
    private IBrand brand;

    public BasePhone(IBrand brand) {
        this.brand = brand;
    }

    protected void open() {
        this.brand.open();
    }

    protected void close() {
        this.brand.close();
    }

    protected void call() {
        this.brand.call();
    }

}


/**
 * 折叠手机类
 */
class FoldedPhone extends BasePhone {
    public FoldedPhone(IBrand brand) {
        super(brand);
    }

    @Override
    protected void open() {
        super.open();
        System.out.println(" 折叠样式手机 ");
    }

    @Override
    protected void close() {
        super.close();
        System.out.println(" 折叠样式手机 ");
    }

    @Override
    protected void call() {
        super.call();
        System.out.println(" 折叠样式手机 ");
    }
}

```

```java
package cn.demo.bridges.review;

/**
 * 桥接模式：将实现与抽象放在两个不同的类层次中，使两个层次可以保持各部分的独立性以及应对他们的功能扩展
 * @author Jjcc
 * @version 1.0.0
 * @Description
 * @ClassName Client.java
 * @createTime 2019年08月22日 15:21:00
 */
public class Client {

    public static void main(String[] args){
        FoldedPhone foldedPhone = new FoldedPhone(new XiaoMiImple());
        foldedPhone.call();
        foldedPhone.open();
        foldedPhone.close();

        System.out.println("-----------vivo手机-------------");

        FoldedPhone foldedPhone1 = new FoldedPhone(new VivoImple());
        foldedPhone1.call();
        foldedPhone1.open();
        foldedPhone1.close();
    }
}

```

### 总结

1. **实现了抽象和实现部分的分离，从而极大的提供了系统的扩展性，让抽象部分和实现部分独立开来，这有助于系统进行分层设计，从而产生更好的结构化系统；**
2. 对于系统的高层部分，只需要知道抽象部分和实现部分的接口就可以了，其他的部分由具体业务来完成。
3. **桥接模式替代多层继承方案**，可以减少 **子类的个数**，降低系统的管理和维护成本。
4. 桥接模式的引入增加了系统的理解和设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设
   计和编程
5. 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用.



## 3. 装饰模式

### 3.1 职责

> **动态的为一个对象增加新的功能。**
>
> 装饰模式是一种用于代替继承的技术，**无需通过继承增加子类就能扩展对象新的功能**，使用对象的关联关系代替继承关系，更加灵活，**同时避免类型体系的快速膨胀**；

### 3.2 实现细节

- 装饰模式就像打包一个快递
  - 主体：比如：陶瓷，衣服（Component）//被装饰者
  - 包装：比如：报纸填充，塑料泡沫，值班，模板（Decorator）
- Component抽象构建角色
  - 真实对象和装饰对象有相同的接口。这样，客户端对象就能够以与真实对象相同的方式同装饰对象交互；
- ConcreteComponent具体构建角色（真实对象）：
  - io流中的FileInputStream、FileOutputStream；
- Decorator装饰角色
  - 持有一个抽象构建的引用。装饰对象接受所有客户端的请求，并把这些请求转发给真实的对象。这样，就能在真实对象调用前后增加新的功能；
- ConcreteDecorator
  - 负责给构建对象增加新的功能；

![1566914448373](.\img\askdo2as1gd4.png)

### 3.3 案例

![1566916824863](.\img\affsi123zxd2.png)

```java
package cn.demo.decorators;

/**
 * Component抽象构建角色：真实对象和装饰对象有相同的接口或父类，这样，客户端对象就能够以与真实对象相同的方式同装饰对象交互；
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className ICar.java
 * @createTime 2019年08月27日 22:07:00
 */
public interface ICar {
    /**
     * 真实对象接口本身的方法
     * @title move
     * @description
     * @author Jjcc
     * @return void
     * @createTime 2019/8/27 22:11
     * @throws
     */
    void move();
}

/**
 * ConcreteComponent具体构建对象（真实对象)
 */
class CarImpl implements ICar {
    @Override
    public void move() {
        System.out.println("陆地上跑！");
    }

}

/**
 * Decorator装饰对象
 */
class DecoratorCar implements ICar {
    @Override
    public void move() {
        System.out.println("陆地上跑");
    }
}

/**
 * ConcreteComponent具体装饰对象
 */
class FlyCar extends DecoratorCar {

    private ICar car;

    /**
     * 通过构造方法，具体装饰对象与真实对象组合起来；
     * @title FlyCar
     * @description
     * @author Jjcc
     * @param car
     * @return
     * @createTime 2019/8/27 22:20
     * @throws
     */
    public FlyCar(ICar car) {
        this.car = car;
    }

    public void move() {
        car.move();
    }

    public void fly() {
        System.out.println("天上飞！");
    }
}

/**
 * ConcreateDecorator具体的装饰对象
 */
class WaterCar extends DecoratorCar {

    private ICar car;

    /**
     * 通过构造方法，具体装饰对象与真实对象组合起来；
     * @title WaterCar
     * @description
     * @author Jjcc
     * @param car
     * @return
     * @createTime 2019/8/27 22:25
     * @throws
     */
    public WaterCar(ICar car) {
        this.car = car;
    }

    public void move() {
        car.move();
    }

    public void swim(){
        System.out.println("水上游！");
    }
}
```

```java
package cn.demo.decorators;

/**
 * 装饰模式：不需要通过继承增加子类就能够扩展对象新的功能，使用关联关系代替继承关系；
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className Client.java
 * @createTime 2019年08月22日 22:26:00
 */
public class Client {
    public static void main(String[] args){
//        try {
//            FileInputStream fileInputStream = new FileInputStream("");
//
//            BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
//        } catch (Exception e) {
//            e.printStackTrace();
//        }

        //与BufferedInputStream(new FileInputStream(""));相同的使用方式
        FlyCar flyCar = new FlyCar(new CarImpl());
        flyCar.move();
        flyCar.fly();

        System.out.println("--------------------分割线---------------------");

        WaterCar waterCar = new WaterCar(new CarImpl());
        waterCar.move();
        waterCar.swim();
    }
}

```

### 3.4 JDK源码-装饰模式场景

**Java 的 IO 结构，FilterInputStream 就是一个装饰者**；

**BufferedInputStream，DataInputStream类就是具体的装饰对象；**

![1567000472950](.\img\dof02as2koasdf.png)

![1567000581421](.\img\asdko1asfo2-g.png)

![1567000597493](.\img\asdasd111d2.png)

```java
package com.atguigu.jdk;
import java.io.DataInputStream;
import java.io.FileInputStream;
import java.io.InputStream;

public class Decorator {
    public static void main(String[] args) throws Exception{
        // TODOAuto-generated method stub
        //说明
        //1. InputStream 是抽象类, 类似我们前面讲的 ICar
        //2. FileInputStream 是 InputStream 子类，类似我们前面的 CarImpl
        //3. FilterInputStream 是 InputStream 子类：类似我们前面 的 Decorator 修饰者
        //4. DataInputStream 是 FilterInputStream 子类，具体的修饰者，类似前面的FlyCar等
        //5. FilterInputStream 类 有 protected volatile InputStream in; 即含被装饰者
        //6. 分析得出在 jdk 的 io 体系中，就是使用装饰者模式
        DataInputStream dis = new DataInputStream(new FileInputStream("d:\\abc.txt"));
        System.out.println(dis.read());
        dis.close();
    }
}
```

### 3.5 总结

**总结：**

- 装饰模式（Decorator）也叫包装器模式（Wrapper）
- 装饰模式降低系统的耦合度，可以动态的增加或删除对象的职责，并使得**需要装饰的具体构建类和具体装饰类可以独立变化**，以便增加新的具体构建类和具体装饰类。

**优点：**

- 扩展对象功能，比继承灵活，不会导致类个数急剧增加
- 可以对一个对象进行多次装饰，创造出不同行为的组合，得到功能更加强大的对象
- 具体构建类和具体装饰类可以独立变化，用户可以根据需要自己增加新的具体构件子类和具体装饰子类。

**缺点：**

-  产生很多小对象。大量小对象占据内存，一定程度上影响性能。
- 装饰模式易于出错，调试排查比较麻烦。

**桥接模式与装饰模式的区别：**

- 两个模式都是为了解决过多子类对象问题。但他们的诱因不一样。桥接模式是对象自身现有机制沿着多个维度变化，是既有部分不稳定。装饰模式是为了增加新的功能；



## 4. 组合模式

### 4.1 基本介绍

> 组合模式（Composite Pattern）又叫部分整体模式，它创建了对象组的树形结构，将对象组合成树状结构以表示**“整体-部分”**的层次关系；
>
> 组合模式依据**树形结构来组合对象**，用来表示部分以及整体层次。
>
> 组合模式使得**用户对单个对象和组合对象的访问具有一致性**，即：组合能让客户以一致的方式处理个别对象以及组合对象

### 4.2 组合模式原理类图

![1567434148764](.\img\fs21ofasd211.png)

![1567434708911](.\img\so2sdo1dasd1p.png)

- Component（抽象构建角色）：这是组合中对象声明接口，在适当情况下，实现所有类共有的接口默认行为，用于访问和管理Component子部件，Component可以是抽象类或者接口。**定义了叶子和容器构件的共同点**；
- Leaf（叶子）：在组合中表示叶子节点，**叶子节点没有子节点**；
- Composite（容器构建角色）：非叶子节点，用于存储子部件，在Composite接口中实现子部件的相关操作，比如增加（add）,删除（delete）。**有容器特征，可以包含子节点**；

### 4.3 案例

**应用实例要求：**

编写程序展示一个学校院系结构：需求是这样，要在一个页面中展示出学校的院系组成，一个学校有多个学院，
一个学院有多个系。

![1567518556335](.\img\asjdi21224.png)

```java
package cn.demo.composite;

/**
 * 组合模式：又叫部分整体模式，它创建了对象的树形结构，将对象组合成树状结构以表示“整体-部分”的层次关系；
 * 组合模式使得用户对单个对象和组合对象的访问具有一致性，即；组合能让客户已一致的方式处理个别对象以及组合对象；
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className Client.java
 * @createTime 2019年09月02日 22:53:00
 */
public class Client {
    public static void main(String[] args) {
        //学校
        BaseOrganizationComponent university = new University("清华大学", "中国顶级大学");

        //学院
        BaseOrganizationComponent computerCollege = new College("计算机学院", " 计算机学院 ");
        BaseOrganizationComponent infoEngineerCollege = new College("信息工程学院", " 信息工程学院 ");

        university.add(computerCollege);
        university.add(infoEngineerCollege);

        //计算机学院专业
        computerCollege.add(new Department("通信工程", " 通信工程不好学 "));
        computerCollege.add(new Department("信息工程", " 信息工程好学 "));

        university.print();
    }
}

```

```java
package cn.demo.composite;

/**
 * 抽象构建角色，定义了叶子和容器的共同点；在适当情况下，实现所有类共有的接口默认行为；
 * @author Jjcc
 * @version 1.0.0
 * @description 抽象构建类；叶子和容器的共同接口
 * @className OrganizationComponent.java
 * @createTime 2019年09月02日 22:58:00
 */
public abstract class BaseOrganizationComponent {

    private String name;

    private String description;

    /**
     * 容器构建角色类独有的方法
     * @title add
     * @description 
     * @author Jjcc
     * @param baseOrganizationComponent
     * @return void
     * @createTime 2019/9/3 21:12
     * @throws
     */
    protected void add(BaseOrganizationComponent baseOrganizationComponent) {
        //默认实现
        throw new UnsupportedOperationException();
    }
	/**
     * 容器构建角色类独有的方法
     * @title remove
     * @description 
     * @author Jjcc
     * @param baseOrganizationComponent
     * @return void
     * @createTime 2019/9/3 21:12
     * @throws
     */
    protected void remove(BaseOrganizationComponent baseOrganizationComponent) {
        //默认实现
        throw new UnsupportedOperationException();
    }

    public BaseOrganizationComponent(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    /**
     * 叶子和容器共有的方法，定义为抽象方法，必须重写
     * @title print
     * @description
     * @author Jjcc
     * @return void
     * @createTime 2019/9/2 23:22
     * @throws
     */
    public abstract void print();
}
```

```java
package cn.demo.composite;

import java.util.ArrayList;
import java.util.List;

/**
 * 容器构建角色，就是Composite类，可以包含子节点；
 * @author Jjcc
 * @version 1.0.0
 * @description 学校类,可以管理学院类（College）
 * @className University.java
 * @createTime 2019年09月02日 23:23:00
 */
public class University extends BaseOrganizationComponent{

    List<BaseOrganizationComponent> baseOrganizationComponents = new ArrayList<>();

    public University(String name, String description) {
        super(name, description);
    }

    /**
     * 重写抽象构建角色类的add方法
     * @title add
     * @description
     * @author Jjcc
     * @return void
     * @createTime 2019/9/3 16:11
     * @throws
     */
    @Override
    protected void add(BaseOrganizationComponent baseOrganizationComponent) {
        baseOrganizationComponents.add(baseOrganizationComponent);
    }

    /**
     * 重写抽象构建角色类的remove方法
     * @title remove
     * @description
     * @author Jjcc
     * @return void
     * @createTime 2019/9/3 16:19
     * @throws
     */
    @Override
    protected void remove(BaseOrganizationComponent baseOrganizationComponent) {
        baseOrganizationComponents.remove(baseOrganizationComponent);
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public String getDescription() {
        return super.getDescription();
    }

    @Override
    public void print() {
        System.out.println("----------" + getName() + "----------");

        //遍历 organizationComponents
        for (BaseOrganizationComponent organizationComponent : baseOrganizationComponents) {
            organizationComponent.print();
        }
    }
}
```

```java
package cn.demo.composite;

import java.util.ArrayList;
import java.util.List;

/**
 * 容器构建角色，可以包含子节点；
 * @author Jjcc
 * @version 1.0.0
 * @description 学院类；可以管理系(专业)（Department）
 * @className College.java
 * @createTime 2019年09月03日 15:56:00
 */
public class College  extends BaseOrganizationComponent{

    List<BaseOrganizationComponent> baseOrganizationComponents = new ArrayList<>();

    public College(String name, String description) {
        super(name, description);
    }

    @Override
    protected void add(BaseOrganizationComponent baseOrganizationComponent) {
        baseOrganizationComponents.add(baseOrganizationComponent);
    }

    @Override
    protected void remove(BaseOrganizationComponent baseOrganizationComponent) {
        baseOrganizationComponents.remove(baseOrganizationComponent);
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public String getDescription() {
        return super.getDescription();
    }

    @Override
    public void print() {
        System.out.println("----------" + getName() + "----------");

        for (BaseOrganizationComponent baseOrganizationComponent : baseOrganizationComponents) {
            baseOrganizationComponent.print();
        }
    }
}
```

```java
package cn.demo.composite.review1;

/**
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className Department.java
 * @createTime 2019年09月03日 21:24:00
 */
public class Department extends BaseOrganizationComponent {


    public Department(String name, String description) {
        super(name, description);
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public void setName(String name) {
        super.setName(name);
    }

    @Override
    public void print() {
        System.out.println("----------------" + getName() + "----------------");
    }
}

```

### 4.4 JDK源码-组合模式场景

![1567525792137](.\img\123312123.png)

```java
package com.atguigu.jdk;

import java.util.HashMap;
import java.util.Map;

public class Composite {

	public static void main(String[] args) {
		
		
		//说明
		//1. Map 就是一个抽象的构建 (类似我们的Component)
		//2. HashMap是一个中间的构建(Composite), 实现/继承了相关方法
		//   put, putall 
		//3. Node 是 HashMap的静态内部类，类似Leaf叶子节点, 这里就没有put, putall
		//   static class Node<K,V> implements Map.Entry<K,V>
		
		Map<Integer,String> hashMap=new HashMap<Integer,String>();
		hashMap.put(0, "东游记");//直接存放叶子节点(Node)
		
		Map<Integer,String> map=new HashMap<Integer,String>();
		map.put(1, "西游记");
		map.put(2, "红楼梦"); //..
		hashMap.putAll(map);
		System.out.println(hashMap);

	}

}

```

### 4.5 总结

**总结：**

- 简化客户端操作，客户端只需要面对一致的对象而不用考虑整体部分或者节点叶子的问题。
- 具有较强的扩展性。当我们要更改组合对象时，我们只需要调用内部的层次接口，客户端不用做出任何改动。
- 方便创建出复杂的层次结构。客户端不用理会组合里面的组成细节，容易添加节点或者叶子从而创建出复杂的
  树形结构
- 需要遍历组织机构，或者**处理的对象具有树形结构时，非常适合使用组合模式。**
- 要求较高的抽象性，**如果节点和叶子有很多差异性的话，比如很多方法和属性都不一样，不适合使用组合模式。**



# 行为型模式

- **行为型模式**关注系统中对象之间的相互交互，研究系统在运行时对象之间的相互通信和协作，进一步明确对象的职责，共有11种模式。

| 模式       | 介绍                                                         |
| ---------- | :----------------------------------------------------------- |
| 观察者模式 | 当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。 |
| 策略模式   | 定义一系列算法，并将每个算法封装在一个类中。                 |
| 职责链模式 | 避免请求发送者和接受者耦合，让多个对象都有可能接收请求，让这些对象连成一条链，并且沿着这条链传递请求，直到有对象处理为止。 |
| 命令模式   | 将一个请求封装为一个对象，从而使得请求调用者和请求接收者解耦 |
| 解释器模式 | 描述如何为语言定义一个文法，如何解析                         |
| 迭代器模式 | 提供了一种方法来访问聚合对象                                 |
| 中介者模式 | 通过一个中介对象来封装一系列的对象交互，使得各对象不需要相互引用 |
| 备忘录模式 | 捕获一个对象的内部状态，并保存之；需要时，可以恢复到保存的状态 |
| 状态模式   | 允许一个对象在其内部状态改变时改变它的行为                   |
| 模板方法   | 定义一个操作的算法骨架，将某些易变的步骤延迟到子类中实现     |
| 访问者模式 | 表示一个作用于某对象结构中的各元素的操作，它使得用户可以在不改变各元素的类的前提下定义作用于这些元素的新操作 |



## 1. 观察者模式

### 1.1 基本介绍

**场景：**

- 聊天室程序的创建。服务器创建好后，A,B,C三个客户端连上来公开聊天。A向服务器发送数据，服务器端聊天数剧改变。我们希望将这些聊天数据分别发给其他在线的客户。也就是说，每个客户端需要更新服务器端得数据。
- 网站上，很多人订阅了”java主题”的新闻。当有这个主题新闻时，就会将这些新闻发给所有订阅的人。
- 大家一起玩CS游戏时，服务器需要将每个人的方位变化发给所有的客户。

> 上面这些场景，我们都可以使用观察者模式来处理。我们可以**把多个订阅者、客户称之为观察者**； 需要同步给多个订阅者的数据封装到对象中，称之为**目标**。

**核心：**

- 观察者模式主要用于1：N的通知。当一个对象（**目标对象Subject或Observable**）的状态变化时，他需要即时告知一系列对象（**观察者对象，Observer**）令他们做出响应。
- 通知观察者的方式
  - 推（push）
    - 每次都会把通知以广播方式发送给所有观察者，所有观察者只能被动接收。
  - 拉（pull）
    - 观察者只要知道有情况即可。至于什么时候获取内容，获取什么内容，都可以自主决定。

### 1.2 观察者模式原理

- 观察者模式类似于订购牛奶业务。
- 奶站/气象局：Subject（主题，目标。登记注册、移除和通知）。
- 用户/第三方网站：Observer（观察者对象，接收者）。

> 观察者对象：对象之间多对一的一种设计方案，被依赖的对象为Subject，依赖的对象为Observer，Subject通知Observer变化，比如这里的奶站是Subject，是1的一方，用户是Observer，是多的一方。

### 1.3 案例

**应用实例要求：**

编写一个当主题对象状态发生改变时，其相关所有依赖对象都发生改变；

![1567691597115](.\img\1567691597115.png)

```java
package cn.demo.observer.review1;

/**
 * 观察者模式：当一个（Subject或者Observable）对象的状态发生改变时，其相关依赖对象（Observer）都会收到通知并且被自动更新；
 * 观察者模式就是1：N的模式，主题对象就是那个1，N是订阅者，又称之为观察者对象；
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className Client.java
 * @createTime 2019年09月05日 20:48:00
 */
public class Client {
    public static void main(String[] args) {
		//主题类
        ObservableA observableA = new ObservableA();

        observableA.setObjState(222);

        ObserverImplA observerImplA1 = new ObserverImplA();
        ObserverImplA observerImplA2 = new ObserverImplA();
        ObserverImplA observerImplA3 = new ObserverImplA();

        observableA.add(observerImplA1);
        observableA.add(observerImplA2);
        observableA.add(observerImplA3);

        observableA.notifyAllObservers();

        System.out.println(observerImplA1.getMyState());
        System.out.println(observerImplA2.getMyState());
        System.out.println(observerImplA3.getMyState());

        System.out.println("!!!!!!!!!!!!!!!!!!!!!!!!!");
        observableA.setObjState(333);

        observableA.notifyAllObservers();

        System.out.println(observerImplA1.getMyState());
        System.out.println(observerImplA2.getMyState());
        System.out.println(observerImplA3.getMyState());
    }
}

```

```java
package cn.demo.observer.review1;

import java.util.ArrayList;
import java.util.List;

/**
 * 主题对象的父类
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className BaseObservable.java
 * @createTime 2019年09月05日 21:02:00
 */
public abstract class BaseObservable {

    /**
     * 观察者对象的集合；所有订阅了该主题的对象都会存入该集合中；
     * @title
     * @description
     * @author Jjcc
     * @return
     * @createTime 2019/9/5 21:08
     * @throws
     */
    private List<IObserver> list = new ArrayList<>();

    public void add(IObserver observer) {
        list.add(observer);
    }

    public void remove(IObserver observer) {
        list.remove(observer);
    }

    /**
     * 主题对象的通知方法，当主题对象发生改变时，通知所有的订阅者
     * @title notifyAllObservers
     * @description
     * @author Jjcc
     * @return void
     * @createTime 2019/9/5 21:25
     * @throws
     */
    public void notifyAllObservers() {
        list.forEach(observer ->
            observer.update(this)
        );
    }

}
```

```java
	package cn.demo.observer.review1;

/**
 * 主题类的具体构建角色
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className ObservableImplA.java
 * @createTime 2019年09月05日 21:28:00
 */
public class ObservableA extends BaseObservable {

    private int objState;

    public int getObjState() {
        return objState;
    }

    public void setObjState(int objState) {
        this.objState = objState;
    }


}
```

```java
package cn.demo.observer.review1;

/**
 * 观察者对象共有的接口
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className IObserver.java
 * @createTime 2019年09月05日 20:55:00
 */
public interface IObserver {

   /**
    * 观察者对象更新信息的方法
    * @title update
    * @description
    * @author Jjcc
    * @param baseObservable 封装信息的对象，参数类型为主题对象的父类
    * @return void
    * @createTime 2019/9/5 21:11
    * @throws
    */
    void update(BaseObservable baseObservable);
}

```

```java
package cn.demo.observer.review1;

/**
 * 观察者的具体构建类
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className ObserverImplA.java
 * @createTime 2019年09月05日 21:32:00
 */
public class ObserverImplA implements IObserver {

    private int myState;

    public int getMyState() {
        return myState;
    }

    public void setMyState(int myState) {
        this.myState = myState;
    }

    @Override
    public void update(BaseObservable baseObservable) {
        myState = ((ObservableA) baseObservable).getObjState();
    }
}

```

### 1.4 JDK源码-观察者模式场景

![1567694095784](.\img\1567694095784.png)

> JDK的`Observable`类和`Observer`类就是观察者模式的主题角色和观察者角色；

模式角色分析
Observable 的作用和地位等价于 我们前面讲过 Subject
Observable 是类，不是接口，类中已经实现了核心的方法 ,即管理 Observer 的方法 add.. delete .. notify...
Observer 的作用和地位等价于我们前面讲过的 Observer, 有 update
Observable 和 Observer 的使用方法和前面讲过的一样，只是 Observable 是类，通过继承来实现观察者模式

### 1.5 总结

**开发中场景的场景：**

- 聊天室程序，服务器转发给所有客户端；
- 网络游戏（多人联机对战）场景中，服务器将客户端的状态进行分发；
- 邮件订阅；
- Servlet中，监听器的实现；
- Android中，广播机制；
- 京东商城中，群发某商品打折信息



## 2. 策略模式

### 2.1 基本介绍

> 策略模式（Strategy Pattern）中，定义 **算法族（策略组）**，分别封装起来，让他们之间可以互相替换，此模式**让算法的变化独立于使用算法的客户**
>
> 策略模式对应于解决某一个问题的算法族（策略组），允许用户从该算法族中任意选择一个算法解决某一问题，同时可以方便的更换算法或者增加新的算法。并且由客户端决定调用哪个算法。
>
> 策略模式体现了几个设计原则，第一，把变化的代码从不变的代码中分离出来；第二，针对接口编程，而不是具体的实现类（定义了策略接口）；第三，多用了组合/聚合，少用策略（客户通过组合方式策略）；

### 2.2 实现原理类图

![1567954669139](.\img\1567954669139.png)







`context` 类有成员变量 strategy 或者其他的策略接口,至于需要使用到哪个策略，我们可以在构造器中指定；

> Context类：负责和具体的策略类交互；这样的话，具体的算法和直接的客户端调用分离了，使得算法可以独立于客户端独立的变化；
>
> 如果使用`spring`的依赖注入功能，还可以通过配置文件，动态的注入不同策略对象，动态的切换不同的算法.

### 2.3 案例

**场景：**
某个市场人员接到单后的报价策略(CRM系统中常见问题)。报价策略
很复杂，可以简单作如下分类：
普通客户小批量报价
普通客户大批量报价
老客户小批量报价
老客户大批量报价
具体选用哪个报价策略，这需要根据实际情况来确定。这时候，我们采用策略模式即可。

```java
package cn.demo.strategy;

/**
 * 策略模式：定义算法族（策略组），分别封装起来，让他们之间可以互相替换，此模式让算法的变化独立于使用算法的客户；
 *          允许用户从该算法族中选择某一个算法解决某一个问题，同时可以方便的更换算法或增加新的算法，并且由客户决定更换哪个算法；
 * 策略模式体现的几种设计原则：
 *      1.面向接口开发，而不是具体的实现类（策略模式中定义了策略接口）；
 *      2.使用组合/聚合代替了继承；
 *      3.把变化的代码从不变的代码中分离开来；
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className Client.java
 * @createTime 2019年09月08日 23:00:00
 */
public class Client {

    public static void main(String[] args) {
        Context context = new Context(new NewCustomerManyStrategyImpl());

        context.pringPrice(1111);
    }
}

```

```java
package cn.demo.strategy;

/**
 * Context：负责和具体的策略类交互，这样的话，具体的算法和直接的客户端调用分离了，使得算法可以独立于客户端独立的变化。
 *
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className Context.java
 * @createTime 2019年09月08日 23:16:00
 */
public class Context {

    /**
     * 算法族接口；使用组合方式
     */
    private IStrategy strategy;

    public Context(IStrategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(IStrategy strategy) {
        this.strategy = strategy;
    }

    public void pringPrice(double param) {
        System.out.println("您该报价："+strategy.getPrice(param));
    }


}
```

```java
package cn.demo.strategy;

/**
 * 抽象构建角色：算法族的接口
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className IStartegy.java
 * @createTime 2019年09月08日 23:10:00
 */
public interface IStrategy {

    /**
     * 算法方法；
     * @title getPrice
     * @description
     * @author Jjcc
     * @param standardPrice
     * @return double
     * @createTime 2019/9/8 23:11
     * @throws
     */
    double getPrice(double  standardPrice);
}
```

```java
package cn.demo.strategy;

/**
 * 算法族的某一个具体构建类
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className NewCustomerFewStrategyImpl.java
 * @createTime 2019年09月08日 23:13:00
 */
public class NewCustomerFewStrategyImpl implements IStrategy {
    @Override
    public double getPrice(double standardPrice) {
        System.out.println("不打折，原价");
        return standardPrice;
    }
}
```

```java
package cn.demo.strategy;

/**
 * 算法族抽象构建的某一个具体构建类
 * @author Jjcc
 * @version 1.0.0
 * @description
 * @className NewCustomerManyStrategyImpl.java
 * @createTime 2019年09月08日 23:15:00
 */
public class NewCustomerManyStrategyImpl implements IStrategy {
    @Override
    public double getPrice(double standardPrice) {
        System.out.println("打九折");
        return standardPrice * 0.9;
    }
}
```

> 在Spring项目中，使用策略模式。
>
> 定义一个算法族接口，实现不同的具体构建类，并用@Service("...")注册。
>
> 定义一个Context类，类中定义一个成员变量的集合；Map<String, 算法族接口类型>，并用@Autowired标记，Spring源码中@AutoWired注释大意为：当该注解标记于Map，List等集合时，会查找容器中所有注册的Bean并放入标记的集合中（通过集合的类型查找，Map的value为查找的类型；ps：如果是Map集合，查找到符合条件的Bean后，Bean的name是key，Bean本身是value）；
>
> 可以通过工厂模式，枚举，反射等获得客户所需要的算法所在的Bean；

### 2.4 JDK源码-策略模式场景



```java
package com.atguigu.jdk;

import java.util.Arrays;
import java.util.Comparator;


public class Strategy {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		//数组
		Integer[] data = { 9, 1, 2, 8, 4, 3 };
		// 实现降序排序，返回-1放左边，1放右边，0保持不变
		
		// 说明
		// 1. 实现了 Comparator 接口（策略接口） , 匿名类 对象 new Comparator<Integer>(){..}
		// 2. 对象 new Comparator<Integer>(){..} 就是实现了 策略接口 的对象
		// 3. public int compare(Integer o1, Integer o2){} 指定具体的处理方式
		Comparator<Integer> comparator = new Comparator<Integer>() {
			public int compare(Integer o1, Integer o2) {
				if (o1 > o2) {
					return -1;
				} else {
					return 1;
				}
			};
		};
		
		// 说明
		/*
		 * public static <T> void sort(T[] a, Comparator<? super T> c) {
		        if (c == null) {
		            sort(a); //默认方法
		        } else { 
		            if (LegacyMergeSort.userRequested)
		                legacyMergeSort(a, c); //使用策略对象c
		            else
		            	// 使用策略对象c
		                TimSort.sort(a, 0, a.length, c, null, 0, 0);
		        }
		    }
		 */
		//方式1 
		Arrays.sort(data, comparator);
		
		System.out.println(Arrays.toString(data)); // 降序排序

		
		//方式2- 同时lambda 表达式实现 策略模式
		Integer[] data2 = { 19, 11, 12, 18, 14, 13 };
		
		Arrays.sort(data2, (var1, var2) -> {
			if(var1.compareTo(var2) > 0) {
				return -1;
			} else {
				return 1;
			}
		});
		
		System.out.println("data2=" + Arrays.toString(data2));
		
	}

}

```





















