# Java类加载器

每个编写的”.java”拓展名类文件都存储着需要执行的程序逻辑，这些”.java”文件经过Java编译器编译成拓展名为”.class”的文件.
”.class”文件中保存着Java代码经转换后的虚拟机指令，当需要使用某个类时，虚拟机将会加载它的”.class”文件，并创建对应的class对象

**将class文件加载到虚拟机的内存，这个过程称为类加载**

![](.\img\12331212.png)

## 一.类加载过程

### 1.加载

> 加载指的是将class文件读取到内存,并在堆中创建一个java.lang.Class对象, 也就是说, 当程序使用任何类时, 系统都会为之建立一个java.lang.Class对象, 作为元空间类数据的访问入口.

类的加载由类加载器完成，类加载器通常由JVM提供，这些类加载器也是前面所有程序运行的基础，JVM提供的这些类加载器通常被称为系统类加载器。除此之外，开发者可以通过继承ClassLoader基类来创建自己的类加载器。

- 从本地文件系统加载class文件，这是前面绝大部分示例程序的类加载方式。

- 从JAR包加载class文件，这种方式也是很常见的，前面介绍JDBC编程时用到的数据库驱动类就放在JAR文件中JVM可以从JAR文件中直接加载该class文件。

- 通过网络加载class文件。

- 把一个Java源文件动态编译，并执行加载。

***类加载器通常无须等到“首次使用”该类时才加载该类，Java虚拟机规范允许系统预先加载某些类。***

### 2.链接

*当类加载后, 系统为之生成一个对应的class对象, 接着会进入链接状态, 链接阶段负责把类的二进制文件合并到JRE中. 类链接又可分为如下三个阶段.*

#### 1.验证

> 目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。。验证阶段是Java非常重要的一个阶段，它会直接的保证应用是否会被恶意入侵的一道重要的防线，越是严谨的验证机制越安全。**验证的目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。**主要包括四种验证: 文件格式验证，元数据验证，字节码验证，符号引用验证。

- 文件格式验证

主要验证字节流是否符合Class文件格式规范，并且能被当前的虚拟机加载处理。例如：主，次版本号是否在当前虚拟机处理的范围之内。常量池中是否有不被支持的常量类型。指向常量的中的索引值是否存在不存在的常量或不符合类型的常量。

- 元数据验证

对字节码描述的信息进行语义的分析，分析是否符合java的语言语法的规范。

- 字节码验证

最重要的验证环节，分析数据流和控制，确定语义是合法的，符合逻辑的。主要的针对元数据验证后对方法体的验证。保证类方法在运行时不会有危害出现。

- 符号引用验证

要是针对符号引用转换为直接引用的时候，是会延伸到第三解析阶段，主要去确定访问类型等涉及到引用的情况，主要是要保证引用一定会被访问到，不会出现类等无法访问的问题。

#### 2.准备

为类变量(即static修饰的字段变量)分配内存并且设置该类变量的初始值即0(*如static int i=5;这里只将i初始化为0，*
*至于5的值将在初始化时赋值*)，**这里不包含用final修饰的static，因为final修饰的字段在编译的时候就把结果放入到了Class常量池中，在准备阶段就赋值了；**

**public static final int value=123，即当类字段的字段标注为final之后，value的值在准备阶段初始化为123而非0.**

**注意这里不会为实例变量分配初始化，类变量会分配在元空间中，而实例变量是会随着对象一起分配到Java堆中。**

**在实际的程序中，只有同时被final和static修饰的字段才有ConstantValue属性，且限于基本类型和String。编译时Javac将会为该常量生成ConstantValue属性，在类加载的准备阶段虚拟机便会根据ConstantValue为常量设置相应的值，如果该变量没有被final修饰，或者并*非基本类型及字符串，则选择在类构造器中进行初始化*。**

**分配内存，并为类变量设置初始化默认值 （元空间中）**

- public static int v=1;
- 在准备阶段中，v会被设置为0
- 在初始化的中才会被设置为1
- **对于static final类型，在准备阶段就会被赋上正确的值**
- public static final int v =1;

#### 3.解析

将类的二进制数据中的符号引用替换成直接引用。

**符号引用: **

符号引用是以一组符号来描述所引用的目标，符号可以是任何的字面形式的字面量，只要不会出现冲突能够定位到就行。布局和内存无关。

**直接引用: **

是指向目标的指针，偏移量或者能够直接定位的句柄。该引用是和内存中的布局有关的，并且一定加载进来的。

### 3.初始化

**类加载最后阶段，若该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化成员变量(如前面只初始化了默认值的static变量将会在这个阶段赋值，成员变量也将被初始化)**。

类`<clint>()`方法是由编译器自动收集类中**所有类变量的赋值动作和静态语句块(static块)中的语句合并产生的**.

- 执行类构造器
  - static变量 赋值语句
  - static{}语句
- 子类的调用前保证父类的被调用
- 是线程安全的

## 二.类加载时机

- 类的主动引用(一定会发生类的初始化)
  - new一个对象
  - 调用类的静态成员(除了final修饰的静态变量)和静态方法
  - 使用反射`Class.forName("com.jjcc.demo")`
  - 初始化一个类的子类或实现类, 则一定会先初始化他的基类和接口
  - JVM启动时标明的启动类, 即文件名和类名相同的那个类(`main(String[] arge){}`)

- 类的被动引用(不会发生类的初始化)

  - **当访问一个静态域时, 只有真正声明这个域的类才会初始化, 通过子类引用父类的静态变量, 不会导致子类初始化**
  - 通过数组定义类引用, 不会触发此类的初始化
  - **引用常量不会触发此类的初始化**(*常量在编译阶段就将值存入类的常量池中了*,**即类常量池, 在初始化后又转存到了运行时常量池中；在链接的准备阶段就会赋值**)

  

## 三.类加载器

类加载器负责加载所有的类，其为所有被载入内存中的类生成一个java.lang.Class实例对象。一旦一个类被加载如JVM中，同一个类就不会被再次载入了。正如一个对象有一个唯一的标识一样，一个载入JVM的类也有一个唯一的标识。

### 1.启动（Bootstrap）类加载器

启动类加载器主要加载的是JVM自身需要的类, 这个类加载使用C++语言实现的, 是虚拟机的一部分, 它负责将`<JAVA_HOME>/lib`路径下的核心类库或`-Xbootclasspath`参数指定的路径下的jar包加载到内存中; **出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类**

### 2.扩展(Extension)类加载器

扩展类加载器是指Sun公司(已被Oracle收购)实现的`sun.misc.Launcher$ExtClassLoader`类，由Java语言实现的，是Launcher的静态内部类，它负责加载JRE的扩展目录，lib/ext或者由java.ext.dirs系统属性指定的目录中的JAR包的类。由Java语言实现，父类加载器为null。

### 3.系统(System)类加载器

它负责加载系统类路径`java -classpath`或`-D java.class.path` 指定路径下的类库，也就是我们经常用到的classpath路径，开发者可以直接使用系统类加载器，一般情况下该类加载是程序中默认的类加载器，通过`ClassLoader#getSystemClassLoader()`方法可以获取到该类加载器。如果没有特别指定，则用户自定义的类加载器都以此类加载器作为父加载器。由Java语言实现，父类加载器为ExtClassLoader。

**类加载器加载Class大致要经过如下8个步骤：**

1. 检测此Class是否载入过，即在缓冲区中是否有此Class，如果有直接进入第8步，否则进入第2步。
2. 如果没有父类加载器，则要么Parent是根类加载器，要么本身就是根类加载器，则跳到第4步，如果父类加载器存在，则进入第3步。
3. 请求使用父类加载器去载入目标类，如果载入成功则跳至第8步，否则接着执行第5步。
4. 请求使用根类加载器去载入目标类，如果载入成功则跳至第8步，否则跳至第7步。
5. 当前类加载器尝试寻找Class文件，如果找到则执行第6步，如果找不到则执行第7步。
6. 从文件中载入Class，成功后跳至第8步。
7. 抛出ClassNotFountException异常。
8. 返回对应的java.lang.Class对象。

### 4. 类加载机制

#### 1.全盘负责

所谓全盘负责，就是当一个类加载器负责加载某个Class时，该Class所依赖和引用其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。

#### 2.缓存机制

缓存机制将会保证所有加载过的Class都会被缓存，**当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象**，存入缓冲区中。这就是为很么修改了Class后，必须重新启动JVM，程序所做的修改才会生效的原因。

#### 3.双亲委派

双亲委派模式要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器，请注意双亲委派模式中的父子关系并非通常所说的类继承关系，而是采用组合关系来复用父类加载器的相关代码.

![image](.\img\asdasdas.png)

![image](.\img\asdas213122312.jpg)

双亲委派机制，其工作原理的是，**如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归**，**请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式**，*即每个儿子都很懒，每次有活就丢给父亲去干，直到父亲说这件事我也干不了时，儿子自己才想办法去完成。*

##### 双亲委派模式优势

采用双亲委派模式的是好处是**Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关系可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。**

其次是考虑到安全因素，**java核心api中定义类型不会被随意替换**，假设通过网络传递一个名为`java.lang.Integer`的类，*通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的`java.lang.Integer`，而直接返回已加载过的`Integer.class`，这样便可以防止核心API库被随意篡改。*

可能你会想，如果我们在`classpath`路径下自定义一个名为`java.lang.SingleInterge`类(该类是胡编的)呢？该类并不存在`java.lang`中，经过双亲委托模式，传递到启动类加载器中，由于父类加载器路径下并没有该类，所以不会加载，将反向委托给子类加载器加载，最终会通过系统类加载器加载该类。但是这样做是不允许，因为`java.lang`是核心API包，需要访问权限，强制加载将会报出如下异常

```java
java.lang.SecurityException: Prohibited package name: java.lang
```



### 三.自定义类加载器

> 实现自定义类加载器需要继承`ClassLoader`或者`URLClassLoader`，继承`ClassLoader`则需要自己重写`findClass()`方法并编写加载逻辑，
> 继承`URLClassLoader`则可以省去编写`findClass()`方法以及class文件加载转换成字节码流的代码。那么编写自定义类加载器的意义何在呢？

- **当class文件不在ClassPath路径下，默认系统类加载器无法找到该class文件，**
  **在这种情况下我们需要实现一个自定义的ClassLoader来加载特定路径下的class文件生成class对象。**
- 当一个class文件是通过网络传输并且可能会进行相应的加密操作时，需要先对class文件进行相应的解密后再加载到JVM内存中，
  这种情况下也需要编写自定义的ClassLoader并实现相应的逻辑。
- 当需要实现热部署功能时(**一个class文件通过不同的类加载器产生不同class对象**从而实现热部署功能)，
  需要实现自定义ClassLoader的逻辑。
- ***注意：被两个类加载器加载的同一个类，JVM不认为是相同的类。***

**方法:**

- `getParent()`: 返回该类加载器的父类加载器。
- `loadClass(String name)`: 加载名称为 name的类，返回的结果是 java.lang.Class类的实例。此方法负责加载指定名字的类，***首先会从已加载的类中去寻找***，如果没有找到；从`parentClassLoader[ExtClassLoader]`中加载；如果没有加载到，则从Bootstrap ClassLoader中尝试加载(findBootstrapClassOrNull方法), 如果还是加载失败，则自己加载。如果还不能加载，则抛出异常ClassNotFoundException。如果要改变类的加载顺序可以覆盖此方法；
- `findClass(String name)`: 查找名称为 name的类，返回的结果是 java.lang.Class类的实例。
- `findLoadedClass(String name)`: 查找名称为 name的***已经被加载过的类***，返回的结果是 java.lang.Class类的实例。
- `defineClass(String name, byte[] b, int off, int len)`: 把字节数组 b中的内容转换成 Java 类，返回的结果是
  java.lang.Class类的实例。这个方法被声明为 final的。*需要注意的是，如果直接调用defineClass()方法生成类的Class对象，这个类的Class对象并没有解析(也可以理解为链接阶段，毕竟解析是链接的最后一步)，其解析操作需要等待初始化阶段进行。*
- `resolveClass(Class c)`: 使用该方法可以使用类的Class对象创建完成也同时被解析。前面我们说链接阶段主要是对字节码进行验证，为类变量分配内存并设置初始值同时将字节码文件中的符号引用转换为直接引用。
- 表示类名称的 name参数的值是类的名称。需要注意的是内部类的表示，如 `com.example.Sample$1`和
  `com.example.Sample$Inner`等表示方式。

#### 1.自定义File类文件加载器

**继承ClassLoader类: **

```java
package cn.demo.classloaderx;

import java.io.*;

/**
 * Create by Jjcc on 2019/6/26 22:02
 *
 * @author Jjcc
 */
public class ClassLoaderDemoTwo {

    public static void main(String[] args){
        FileClassLoader fileClassLoader = new FileClassLoader("D:\\JavaSE\\JavaDemoOne\\out\\production\\JavaDemoOne");

        try {
            //loadClass()方法会不会自己加载类, 而是先将请求委托给其父类加载器加载
            Class<?> class1 = fileClassLoader.loadClass("cn.demo.collection.ArrayListContains");
            //摆托双亲委机制, 直接执行类加载
            Class<?> class2 = fileClassLoader.findClass("cn.demo.collection.ArrayListContains");

            System.out.println(class1.hashCode());
            System.out.println(class2.hashCode());

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    public static class FileClassLoader extends ClassLoader {
        //Class文件地址
        private String rootDir;

        public FileClassLoader(String rootDir) {
            this.rootDir = rootDir;
        }

        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            //该类是否已被加载, 如果已经被加载则返回该类
            Class c = this.findLoadedClass(name);
            if (c != null) {
                return c;
            } else {
                //双亲委机制, 类加载器接收到加载请求,不会先自己执行加载, 而是把加载请求委托给父类加载器执行;
                ClassLoader parent = this.getParent();
                c = parent.loadClass(name);

                if (c != null) {
                    return c;
                } else {
                    try {
                        byte[] classData = getClassData(name);
                        //通过class文件的转换的字节数组创建Class对象
                        c = defineClass(name, classData, 0, classData.length);

                        if (c != null) {
                            return c;
                        } else {
                            throw new ClassNotFoundException("没有找到该类文件!");
                        }
                    } catch (FileNotFoundException e) {
                        e.printStackTrace();
                    }
                }
            }

            return c;
        }

        /**
         * 获取类文件的字节码流
         * @param classPath
         * @return
         * @throws FileNotFoundException
         */
        public byte[] getClassData(String classPath) throws FileNotFoundException {
            String path = rootDir + classPath.replace(".", "/") + ".class";

            File file = new File(path);

            if (!file.exists())
                throw new FileNotFoundException("文件不存在!");

            try (InputStream in = new BufferedInputStream(new FileInputStream(path));
                 ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ) {
                byte[] bytes = new byte[1024];
                int temp = 0;
                while ((temp = in.read(bytes)) != -1) {
                    baos.write(bytes, 0, temp);
                    baos.flush();
                }

                baos.toByteArray();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return  null;
        }
    }
}
```

**继承URLClassLoader:**

需要重写构造器外无需编写findClass()方法及其class文件的字节流转换逻辑。

```java
/**
 * Created by zejian on 2017/6/21.
 * Blog : http://blog.csdn.net/javazejian [原文地址,请尊重原创]
 */
public class FileUrlClassLoader extends URLClassLoader {

    public FileUrlClassLoader(URL[] urls, ClassLoader parent) {
        super(urls, parent);
    }

    public FileUrlClassLoader(URL[] urls) {
        super(urls);
    }

    public FileUrlClassLoader(URL[] urls, ClassLoader parent, URLStreamHandlerFactory factory) {
        super(urls, parent, factory);
    }


    public static void main(String[] args) throws ClassNotFoundException, MalformedURLException {
        String rootDir="/Users/zejian/Downloads/Java8_Action/src/main/java/";
        //创建自定义文件类加载器
        File file = new File(rootDir);
        //File to URI
        URI uri=file.toURI();
        URL[] urls={uri.toURL()};

        FileUrlClassLoader loader = new FileUrlClassLoader(urls);

        try {
            //加载指定的class文件
            Class<?> object1=loader.loadClass("com.zejian.classloader.DemoObj");
            System.out.println(object1.newInstance().toString());

            //输出结果:I am DemoObj
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 2.网络类加载器

从网络直接获取到字节流再转车字节数组然后利用defineClass方法创建class对象，
如果继承URLClassLoader类则和前面文件路径的实现是类似的，无需担心路径是filePath还是Url，
因为URLClassLoader内的URLClassPath对象会根据传递过来的URL数组中的路径判断是文件还是jar包，
然后根据不同的路径创建FileLoader或者JarLoader或默认类Loader去读取对于的路径或者url下的class文件。

```java
/**
 * Created by zejian on 2017/6/21.
 * Blog : http://blog.csdn.net/javazejian [原文地址,请尊重原创]
 */
public class NetClassLoader extends ClassLoader {

    private String url;//class文件的URL

    public NetClassLoader(String url) {
        this.url = url;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassDataFromNet(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    /**
     * 从网络获取class文件
     * @param className
     * @return
     */
    private byte[] getClassDataFromNet(String className) {
        String path = classNameToPath(className);
        try {
            URL url = new URL(path);
            InputStream ins = url.openStream();
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead = 0;
            // 读取类文件的字节
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            //这里省略解密的过程.......
            return baos.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        // 得到类文件的URL
        return url + "/" + className.replace('.', '/') + ".class";
    }

}
```

#### 3. 加密解密类加载器

**加密工具类:**

```java
package com.bjsxt.test;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * 加密工具类
 * @author 尚学堂高淇 www.sxt.cn
 *
 */
public class EncrptUtil {
	
	public static void main(String[] args) {
		encrpt("d:/myjava/HelloWorld.class", "d:/myjava/temp/HelloWorld.class");
	}
	
	public static void encrpt(String src, String dest){
		FileInputStream fis = null;
		FileOutputStream fos = null;
		
		try {
			fis = new FileInputStream(src);
			fos = new FileOutputStream(dest);
			
			int temp = -1;
			while((temp=fis.read())!=-1){
				fos.write(temp^0xff);  //取反操作
			}
		} catch (Exception e) {
			e.printStackTrace();
		}finally{
			try {
				if(fis!=null){
					fis.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
			try {
				if(fos!=null){
					fos.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		
	}
}
```

**加载文件系统中加密后的class字节码的类加载器:**

```java
package com.bjsxt.test;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * 加载文件系统中加密后的class字节码的类加载器
 * @author 尚学堂高淇 www.sxt.cn
 *
 */
public class DecrptClassLoader  extends ClassLoader {
	
	//com.bjsxt.test.User   --> d:/myjava/  com/bjsxt/test/User.class      
	private String rootDir;
	
	public DecrptClassLoader(String rootDir){
		this.rootDir = rootDir;
	}
	
	@Override
	protected Class<?> findClass(String name) throws ClassNotFoundException {
		
		Class<?> c = findLoadedClass(name);
		
		//应该要先查询有没有加载过这个类。如果已经加载，则直接返回加载好的类。如果没有，则加载新的类。
		if(c!=null){
			return c;
		}else{
			ClassLoader parent = this.getParent();
			try {
				c = parent.loadClass(name);	   //委派给父类加载
			} catch (Exception e) {
//				e.printStackTrace();
			}
			
			if(c!=null){
				return c;
			}else{
				byte[] classData = getClassData(name);
				if(classData==null){
					throw new ClassNotFoundException();
				}else{
					c = defineClass(name, classData, 0,classData.length);
				}
			}
			
		}
		return c;
	}
	
	private byte[] getClassData(String classname){   //com.bjsxt.test.User   d:/myjava/  com/bjsxt/test/User.class
		String path = rootDir +"/"+ classname.replace('.', '/')+".class";
		
//		IOUtils,可以使用它将流中的数据转成字节数组
		InputStream is = null;
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
		try{
			is  = new FileInputStream(path);
			
			int temp = -1;
			while((temp=is.read())!=-1){
				baos.write(temp^0xff);  //取反操作,相当于解密
			}
			
			return baos.toByteArray();
		}catch(Exception e){
			e.printStackTrace();
			return null;
		}finally{
			try {
				if(is!=null){
					is.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
			try {
				if(baos!=null){
					baos.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		
	}
}
```

#### 4.线程类加载机器

在Java应用中存在着很多服务提供者接口（Service Provider Interface，SPI），这些接口允许第三方为它们提供实现，
如常见的 SPI 有 JDBC、JNDI等，这些 SPI 的接口属于 Java 核心库，一般存在rt.jar包中，由Bootstrap类加载器加载，
而 SPI 的第三方实现代码则是作为Java应用所依赖的 jar 包被存放在classpath路径下，
由于SPI接口中的代码经常需要加载具体的第三方实现类并调用其相关方法，但SPI的核心接口类是由引导类加载器来加载的，
而Bootstrap类加载器无法直接加载SPI的实现类，同时由于双亲委派模式的存在，Bootstrap类加载器也无法反向委托AppClassLoader加载器SPI的实现类。
在这种情况下，我们就需要一种特殊的类加载器来加载第三方的类库，而线程上下文类加载器就是很好的选择。
线程上下文类加载器（contextClassLoader）是从 JDK 1.2 开始引入的，我们可以通过java.lang.Thread类中的`getContextClassLoader()`和
`setContextClassLoader(ClassLoader cl)`方法来获取和设置线程的上下文类加载器。如果没有手动设置上下文类加载器，
线程将继承其父线程的上下文类加载器，初始线程的上下文类加载器是系统类加载器（AppClassLoader）,
在线程中运行的代码可以通过此类加载器来加载类和资源

![image](.\img\asd123asd1.png)

从图可知rt.jar核心包是有Bootstrap类加载器加载的，其内包含SPI核心接口类，由于SPI中的类经常需要调用外部实现类的方法，
而jdbc.jar包含外部实现类(jdbc.jar存在于classpath路径)无法通过Bootstrap类加载器加载，
因此只能委派线程上下文类加载器把jdbc.jar中的实现类加载到内存以便SPI相关类使用。
**显然这种线程上下文类加载器的加载方式破坏了“双亲委派模型”，它在执行过程中抛弃双亲委派加载链模式，使程序可以逆向使用类加载器，**

我们知道线程上下文类加载器默认情况下就是AppClassLoader，
那为什么不直接通过`getSystemClassLoader()`获取类加载器来加载classpath路径下的类的呢？其实是可行的，
但这种***直接使用`getSystemClassLoader()`方法获取AppClassLoader加载类有一个缺点，那就是代码部署到不同服务时会出现问题，***
如把代码部署到Java Web应用服务或者EJB之类的服务将会出问题，因为这些服务使用的线程上下文类加载器并非AppClassLoader，
而是Java Web应用服自家的类加载器，类加载器不同。，所以我们应用该少用getSystemClassLoader()。
总之不同的服务使用的可能默认ClassLoader是不同的，但使用线程上下文类加载器总能获取到与当前程序执行相同的ClassLoader，
从而避免不必要的问题。















