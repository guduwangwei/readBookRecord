#### 类加载器

类加载器是从来加载java文件到虚拟机的。Java虚拟机使用java文件方式： 

.java文件经过java编译器过后被转换成Java字节码(.class文件) 类加载器负责读取java字节码，并转换成java.lang.Class类的一个实例。每个这样的实例用来表示一个java类。通过此实例的newInstance()方法可以创建该类的一个对象

`java.lang.ClassLoader`

### `ava.lang.ClassLoader`类介绍

`java.lang.ClassLoader`类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 `java.lang.Class`类的一个实例。除此之外，`ClassLoader`还负责加载 Java 应用所需的资源，如图像文件和配置文件等。不过本文只讨论其加载类的功能。为了完成加载类的这个职责，`ClassLoader`提供了一系列的方法，比较重要的方法如 [表 1](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/#minor1.1)所示。关于这些方法的细节会在下面进行介绍。

| 方法                                                   | 说明                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `getParent()`                                          | 返回该类加载器的父类加载器。                                 |
| `loadClass(String name)`                               | 加载名称为 `name`的类，返回的结果是 `java.lang.Class`类的实例。 |
| `findClass(String name)`                               | 查找名称为 `name`的类，返回的结果是 `java.lang.Class`类的实例。 |
| `findLoadedClass(String name)`                         | 查找名称为 `name`的已经被加载过的类，返回的结果是 `java.lang.Class`类的实例。 |
| `defineClass(String name, byte[] b, int off, int len)` | 把字节数组 `b`中的内容转换成 Java 类，返回的结果是 `java.lang.Class`类的实例。这个方法被声明为 `final`的。 |
| `resolveClass(Class<?> c)`                             | 链接指定的 Java 类。                                         |



#### 类加载器的树状组织结构

- 引导类加载器（bootstrap class loader）：它用来加载 Java 的核心库，是用原生代码来实现的，并不继承自 `java.lang.ClassLoader`。
- 扩展类加载器（extensions class loader）：它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。
- 系统类加载器（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 `ClassLoader.getSystemClassLoader()`来获取它。

除了系统提供的类加载器以外，开发人员可以通过继承 `java.lang.ClassLoader`类的方式实现自己的类加载器，以满足一些特殊的需求

![](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/image001.jpg)



````java
public class ClassLoaderTree {
    public  static void main(String [] args){
        ClassLoader loader =
                 ClassLoaderTree.class.getClassLoader();
        while (loader != null){
            System.out.println(loader.toString());
            loader = loader.getParent();
        }
    }
}
//输出 (有的jdk实现会引导类加载器返回null)
//sun.misc.Launcher$AppClassLoader@18b4aac2
//sun.misc.Launcher$ExtClassLoader@610455d6
````



#### 类加载器的代理模式

当加载一个类的字节码的时候， 首先会给父加载器去加载，以此类推。

Java虚拟机 怎么判断两个Java是否相同？ 

* 类的全名相同
* 是同一个类加载器加载的



### 线程上下文类加载器

线程上下文类加载器（context class loader）是从 JDK 1.2 开始引入的。类 `java.lang.Thread`中的方法 `getContextClassLoader()`和 `setContextClassLoader(ClassLoader cl)`用来获取和设置线程的上下文类加载器。如果没有通过 `setContextClassLoader(ClassLoader cl)`方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

### Class.forName

`Class.forName`是一个静态方法，同样可以用来加载类。该方法有两种形式：`Class.forName(String name, boolean initialize, ClassLoader loader)`和 `Class.forName(String className)`。第一种形式的参数 `name`表示的是类的全名；`initialize`表示是否初始化类；`loader`表示加载时使用的类加载器。第二种形式则相当于设置了参数 `initialize`的值为 `true`，`loader`的值为当前类的类加载器。`Class.forName`的一个很常见的用法是在加载数据库驱动的时候。如 `Class.forName("org.apache.derby.jdbc.EmbeddedDriver").newInstance()`用来加载 Apache Derby 数据库的驱动。

-----

###Android 类加载器





