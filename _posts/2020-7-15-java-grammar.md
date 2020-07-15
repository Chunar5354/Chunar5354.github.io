---
layout: article
tags: Java
title: Java学习笔记-语法篇
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

根据《Java核心技术卷1》学习Java语法的记录

<!--more-->

# 安装

## Windows

Windows系统安装JDK可以直接在[官方网站](https://www.oracle.com/java/technologies/javase-jdk14-downloads.html)下载`.exe`的安装包，
然后根据[官方文档](https://docs.oracle.com/en/java/javase/14/install/installation-jdk-microsoft-windows-platforms.html#GUID-DAF345BA-B3E7-4CF2-B87A-B6662D691840)的提示配置环境变量即可

## Linux

Linux系统同样需要下载对应版本的安装包，不过在使用wget方法下载是需要注意，Oracle上的资源在下载时需要进行验证，如果直接wget下载的话，只能下载到几KB的错误文件

即使带上下面的cookies参数：

```
wget --no-check-certificate -c --header "Cookie: oraclelicense=accept-securebackup-cookie"
```

看似能够下载完整的文件，但在解压时还是会出现问题

所以最好是下载到自己的电脑上，再传给linux机器

然后可以解压安装包到指定的文件

```
# sudo mkdir /usr/local/jdk

# tar -zxvf  jdk-14.0.1_linux-x64_bin.tar.gz -C /usr/local/jdk
```

然后需要添加环境变量

打开`~/.bash_profile`文件，添加这样一句

```
export JAVA_HOME=/usr/local/jdk/jdk-14.0.1
```

并且将`/usr/local/jdk/jdk-14.0.1/bin`添加到PATH变量中

这样环境变量就配置好了，可以在命令行输入

```
# java --version
```

来进行验证


# 语法知识

Java是一门静态语言，而且同于一般的编译语言或解释型语言。它首先将源代码编译成`字节码`，再依赖各种不同平台上的`虚拟机`来解释执行字节码，从而具有`一次编写，到处运行`的跨平台特性。

编译器对`文件`操作，Java解释器加载`类`

## 类

一个类的基本组成：

- 实例域
- 构造器
- 方法

方法包含隐式参数`this`，类似Python的`self`，但不用在定义方法的时候显式给出


## 静态

### 静态域

在实例域定义的时候加上`static`

```java
private static String name="";
```

每个类`只有一个`，为所有该类的实例对象所共用，类似于Python的类变量。属于类而不属于任何对象实例

### 静态常量

在定义的时候加上`final`，常量不可被修改，所以将其作为公有（public）也没有影响

```java
public static final int n=1234;
```

### 静态方法

静态方法是一种`不能向对象`实施操作的方法，它没有隐式参数

静态方法`不能访问实例域`，但可以访问自身的`静态域`

最好使用`类名`，而不是对象来调用静态方法


## 方法参数

Java的参数调用总是`按值调用`，方法不能修改所传入的变量的内容

如果传入的参数是一个对象的引用，那么方法可以修改被引用的对象的状态，但不能够修改参数的值

```java
harry = new Employee();
doSomething(harry);
```

如上面的例子，harry是Employee类的一个对象引用，当通过doSomething()方法来处理harry时，harry所指向的对象状态可能会改变，但harry自身的值不变


## 重载

多个方法有相同的名字，不同的参数

编译器会根据方法`给出的参数类型`和`特定方法调用所使用的值类型`进行匹配来挑选出应该使用哪个方法

方法名+参数类型：方法的`签名`


## 多态

Java中对象变量是`多态`的，一个类变量可以引用该类本身的对象，也可以引用该类子类的对象

- 静态绑定：调用private、static、final方法或构造器的时候，编译器能够准确地知道应该调用哪个方法

- 动态绑定：调用public方法时，调用哪个方法依赖于方法的`签名`（方法名+参数列表）

    动态绑定时，`虚拟机`事先为每个类创建了一个`方法表`，其中列出了所有方法的签名和调用方法，其搜索是自下而上的（从子类到超类）

## final

不允许被扩展和继承的类称为final类

如果在类中的方法声明为final，那么这个方法不能被子类覆盖

## 强制类型转换

使用圆括号将转换后的类型括起来，比如：

```java
double x = 1.234;
int nx = (int) x;
```

## 抽象

抽象类中的抽象方法起着`占位`的作用，可以不为其填写任何内容，而在子类中完善它


## 4种访问修饰符

- 1.private 仅对本类可见
- 2.public 对所有类可见
- 3.protected 对本包和所有子类可见
- 4.默认，不需要修饰符 对本包可见

## Object超类

- equals方法

- hashcode方法

- toString方法

## ArrayList数组列表

类型化`<>`

泛型

## 接收多个参数

`...`省略号

## 枚举类

```java
public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE};
```

实际上Size是一个类，它继承了`Enum`类，具有四个实例，尽量不要构建新对象

## 反射

能够分析类能力的程序称为反射

## 接口

描述类具有什么功能，而不给出每个功能的具体实现

接口不能含有`实例域`

接口不是类，不能使用`new`来实例化一个接口，不能构造接口对象

但可以声明接口变量，而且接口变量必须引用`实现了接口的类对象`

```java
Comparable x;  // Comparable是一个接口
```

而且接口变量必须引用实现了该接口类的类对象

```java
x = new Employee();

public class Employee implements Comparable<T> {

}
```

与类继承相似，接口也可以通过`extends`来进行扩展

每个类智能继承于`一个超类`，但可以实现`多个接口`


## lambda表达式

参数，箭头`->`以及表达式

```java
(int a, int b) -> {
    if (a > b) return 1;
    else if (a < b) return -1;
    else return 0;
}
```

一个lambda表达式包含

- 1.一个代码块
- 2.参数
- 3.自由变量的值（值非参数而且`不在lambda表达式中定义`的变量），自由变量必须是`不可变的`

## 函数式接口

只有一个抽象方法的接口

当需要函数式接口的对象时，可以提供一个lambda表达式

## 方法引用

有时候有一些现成的方法已经实现了想要传递到其他代码的某的动作，此时可以通过方法引用的方式来实现

比如：

```java
Timer t = new Timer(10000, event -> System.out.println(event));
```

将在触发定时器的时候打印事件对象，这行代码等价于：

```java
Timer t = new Timer(10000, System.out::println);
```

其中`::`操作符就是方法引用

方法引用不能独立存在，一般是作为其他方法的参数，它最终会转换为`函数式接口的实例`


## 内部类

内部类是定义在另一个类中的类，它有几个性质：

- 1.内部类的方法可以访问定义它的类中的数据，包括私有数据
- 2.内部类可以对同一个包中其他的类隐藏起来
- 3.只有内部类可以是`私有类`，其他的类必须是公有或包可见（默认）

### 局部内部类

如果某个内部类只使用了一次，可以将其定义在`方法`内

### 匿名内部类

入宫某个内部类只生成一个对象，可以不为这个类命名


## 代理

使用代理可以在运行时创建一个实现了一组给定接口的新类

代理类是在`程序运行中`被创建的，一旦创建，它就变成了常规类

关于代理的实现方法，可以看一个示例程序

```java
package proxy;

import java.lang.reflect.*;
import java.util.*;


public class ProxyTest {
    public static void main(String[] args) {
        Object[] elements = new Object[1000];

        for (int i=0; i<elements.length; i++) {
            Integer value = i + 1;
            InvocationHandler handler = new TraceHandler(value);
            // 在这里使用了代理，首先为Comparable添加了代理，然后在InvocationHandler接口中重载invoke方法，将proxy作为第一个参数传递
            // 此后每当调用被handler代理的方法（比如compareTo），就会执行invoke中的内容
            Object proxy = Proxy.newProxyInstance(null, new Class[] { Comparable.class }, handler);
            elements[i] = proxy;
        }

        Integer key = new Random().nextInt(elements.length) + 1;

        int result = Arrays.binarySearch(elements, key);

        if (result > 0) System.out.println(elements[result]);
    }
}

class TraceHandler implements InvocationHandler {
    private Object target;

    public TraceHandler(Object t) {
        target = t;
    }

    // 这里的第一个参数proxy基本用不到，它可以任意命名，和上面的proxy变量没有关系
    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable {
        // System.out.println("-- " + proxy);
        System.out.print(target);
        System.out.print("." + m.getName() + "(");
        if (args != null) {
            for (int i=0; i<args.length; i++) {
                System.out.print(args[i]);
                if (i < args.length - 1) System.out.print(", ");
            }
        }
        System.out.println(")");

        return m.invoke(target, args);
    }
}
```


## 异常

Java中所有的异常对象都派生于`Throwable`类的实例

异常的层次结构可以分为：

- Throwable
    - Error
    Error描述了Java运行时系统的内部错误和资源耗尽错误，应用程序一般不会抛出这种异常
    - Exception
        - RuntimeException
        通常是用户编写程序时的错误，包括`错误的类型转换`，`数组访问越界`，`访问null指针`等
        - 其他异常
        如`试图在文件尾部读取数据`，`打开一个不存在的文件`等

其中Error和RuntimeException类派生出的异常称为`非受查`异常，其他的异常称为`受查`异常

一个方法必须声明所有可能的受查异常

- 带资源的try语句

```java
try (Recourse res = ...)
{
    work with res
}
```

就是在try关键词后面加上参数，通常是文件或数据库之类需要使用后关闭的资源，通过这样带资源的try语句可以`自动关闭资源`，而不需人为地加上fially语句


## 断言 assert

断言机制允许在`测试期间`向代码中插入一些检查语句，当代码发布时，插入的检测语句将`自动被移走`

默认情况下啊断言被禁用，需要在运行程序时加上`enableassertions`或`ea`参数

```
# java -ea MyApp
```

而禁用某个特定类或包的断言需要加上`disableassertions`或`da`参数

```
# java -ea:... -da:MyClass MyApp
```

启用或禁用断言是`类加载器`的功能，因此`不需要`重新编译程序


## 泛型

泛型就是在定义类或方法的时候传入一个类型参数，可以使得该泛型类或泛型方法可以被`不同类型的对象`重用

### 泛型类

下面是一个定义简单泛型类的例子

```java
public class Pair<T> {
    private T first;
    private T second;

    public Pair() {
        first = null;
        second = null;
    }

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public T getSecond() {
        return second;
    }
}
```

注意类名后面的`<T>`，这种用尖括号括起来的参数就是一个类型参数，可以在创建Pair对象的时候指定任意的类型，比如：

```java
Pair<String> p = new Pair<>("abc", "123");
```

这里在前面声明类型的时候指定了String，后面的尖括号可以不用加类型，编译器能够自行推导出该用什么类型

### 泛型方法

在定义泛型方法的时候，需要将类型变量放在返回类型的前面，如

```java
public static <T> Pair<T> minmax(T[] a) {
}
```

同时可以对类型变量加以约束，比如规定该类型必须要实现某些方法，如

```java
public static <T extends Comparable> Pair<T> minmax(T[] a) {
}
```

此时minmax()方法只能被实现了Comparable接口的类所调用，注意这里使用的是`extends`修饰符

### 泛型代码和虚拟机

虚拟机中没有泛型类和泛型方法，他们的类型变量都被`擦除`了

桥方法可以保持多态

### 一些注意事项

- 不能创建参数化类型的数组，如

```java
Pair<String>[] table = new Pair<String>[10];
```

- 不能实例化类型变量，如 `new T()`，`new T[]` 或 `T.class`

- 不能构造泛型数组

- 不能在`静态`域或方法中引用类型变量

- 不能抛出或捕获`泛型类对象`，泛型类不能扩展`Throwable`，但可以使用`类型变量`


## 集合类

可以看作集合是实现了一系列接口功能（如Queue，Collection，Stack等）的类

### Iterator接口

任何集合都可以使用`for...each`循环，它能够实现`Iterator`接口

Java迭代器查找一个元素`唯一`的办法是调用next方法，可以认为Java迭代器`位于两个元素之间`，调用next时，迭代器`越过下一个元素`，并返回刚刚越过的那个元素的引用

而remove方法会删除`上次调用next方法`返回的元素

所以想要删除一个元素，必须先调用next，再调用remove


### 散列集（P380/363）

Java中散列码是根据`对象的实例域`计算出的一个整数

Java中散列表用`链表数组`实现，每个列表称为`桶(bucket)`

想要查找表中某一个对象的位置时，将散列码与桶的个数取余，得到的数值就是该元素对应的列表位置，但有可能这个列表已经被填充，这种现象称为`散列冲突`，
此时就将该元素插入到该列表位置对应的`链表尾`（这是Java散列表所采用的`外链法`，在Python中，解决散列冲突的方法是`开放定址法`）

如果散列表太慢，就需要`再散列`，即创建一个桶数更多的表，丢弃原来的表。而何时需要再散列由`装填因子`决定

### 优先级队列 PriorityQueue

通过`堆`实现，元素可以按照任意的顺序插入，但总是按照排序的顺序进行检索，调用remove方法总是会得到`最小`的元素

### 映射 map

Java中有两种基本的映射：`HashMap`和`TreeMap`


## 视图

视图返回一个实现了原集合（或映射）接口的类对象，而且视图的方法对`原集合（或映射）`进行操作

### 同步视图

用于多线程，对于同步视图来说，在另一个线程调用另一个方法之前，当前的方法必须彻底完成


## 多线程

通过`Thread`来使用多线程

一种常用的简单方法：

- 1.将任务代码移动到实现了`Runnable`接口的类的run方法中，由于Runnable只有一个方法，属于函数式接口，可以通过lambda表达式来简单的创建一个实例

```java
Runnable r = () -> { task code };
```

- 2.再由Runnable实例创建一个Thread对象

```java
Thread t = new Thread(r);
```

- 3.启动线程

```java
t.start();
```

### 线程状态

线程可以有以下6种状态：

- 1.New（新建），此时程序还没有运行线程中的代码

- 2.Runnable（可运行），调用了start方法之后，线程处于Runnable状态

可运行状态的线程可能正在运行也可能没有运行

- 3.Blocked（被阻塞）

当一个线程试图获取一个`内部的对象锁`，而该锁被`其他线程所持有`，则该线程进入阻塞状态

当`所有其他线程`释放该锁，并且线程调度器`允许本线程持有它`的时候，该线程编程非阻塞状态

- 4.Waiting（等待）

当线程等待另一个线程通知调度器一个条件时，它自己进入等待状态

- 5.Timed Waiting（计时等待）

计时等待状态将保持到超时期满或者接收到适当的通知

- 6.Terminated（被终止）

线程可能有两种原因被终止：

1.因为`run方法正常退出`而自然死亡

2.因为一个`没有被捕获的异常`终止了run方法而意外死亡


### 线程锁

Java有两中机制防止代码块受并发访问的干扰

- 1.ReentrantLock

用ReentrantLock保护代码块的基本结构如下：

```java
private Lock myLock = new ReentrantLock();
myLock.lock();
try {
    do_something;
}
finally {
    myLock.unLock();
}
```

*(注意，使用锁的时候不能使用带资源的try语句)*

它的原理是：一旦一个线程封锁了锁对象，其它任何线程都`无法通过lock语句`，知道第一个线程释放锁对象

锁是`可重入`的，因为线程可以重复地获得已经持有的锁。锁保持一个`持有计数`来跟踪对`同一个lock`的嵌套调用

- 2.synchronized

Java中的每一个对象都有一个`内部锁`，如果一个方法用synchronized关键字声明，对象的锁将保护`整个方法`

以下两种方式是等价的

```java
public synchronized void method() {
    method body
}
```

```java
public void method() {
    this.intrinsicLock.lock();
    try {
        method body
    }
    finally {
        this.intrinsicLock.unlock();
    }
}
```

内部锁的`wait`方法等价于条件对象的await，`notifyAll`等价于singalAll

如下两个代码是等价的：

```java
    public synchronized void method() throws InterruptedException{
        while (!(ok for proceed))
            wait();
            
        method body
        
        notifyAll();
    }
```

```java
    private Lock theLock = new ReentrantLock();
    private Condition theCondition = theLock.newCondition();
    
    public void method() throws InterruptedException{
        theLock.lock();
        try {
            while (!(ok for proceed))
                theCondition.await();
                
            method body
            
            theCondition.signalAll();
        }
        finally {
            theLock.unlock();
        }
    }
```

### 公平锁与非公平锁

公平锁与非公平锁的区分是针对`锁的获取`而言的

公平锁的获取顺序按照`请求的时间顺序`，它每次都从同步队列的第一个节点获取到锁，而非公平锁则不一定，有可能刚刚释放锁的线程能够马上再次获取到锁

- 优缺点

公平锁为了严格按照时间顺序，需要频繁的上下文切换，增加系统开销

而非公平锁有可能导致某些线程永远无法获取到锁，造成`饥饿`现相

ReentrantLock默认使用的是`非公平锁`


### 条件对象

使用条件对象来管理`已经获得了一个锁`但是不能做有用工作的线程

通过`await`方法来实现


```java
// 在当前线程中
theCondition.await();
```

这样当前线程就被阻塞了，并`放弃了锁`，然后当前线程进入`等待集`，当锁可用时，该线程并不能马上解除阻塞，直到另一个线程调用`同一个条件上`的`singalAll`方法为止

```java
// 在另一个线程中
theCondition.singalAll();
```

这一调用将激活`所有因为这一条件而等待的线程`，它们从等待集中移出时再次变成可运行的，一旦锁成为可用的，他们中的某个将`从await调用返回`，`获得该锁`并从被阻塞的地方继续运行

通常，对await的调用应该放在一个循环体中

```java
while (!(ok to proceed))
    condition.await();
```

### 一些机制（P674/657）

线程是一个很复杂的概念，为了完善它，Java给出了一系列机制

- `监视器` 一种只包含私有域的类，可以确保一个线程在对对象操作时，没有其他线程能访问该域

- `Volatile域` 为实例域的同步访问提供了一种`免锁`机制

- `final变量` 如果使用final定义一个可变的对象，其他线程只能在`完成构造`之后才能看到

- `原子性` 一个操作的原子性指的是在整个操作完成之前，不会被`中断`

- `死锁` 有可能在某一个时间点，每一个线程的阻塞条件都不能被满足，这样就进入了死锁状态，因为没有内部的机制来打破死锁，所以必须在设计程序的时候小心

- `线程局部变量` 可以通过`ThreadLocal`或`ThreadLocalRandom.current()`为每一个线程构造一个实例

- `锁测试` 线程在调用lock方法来获得另一个线程所持有的锁的时候，很可能发生阻塞，这是可以使用`tryLock`方法来申请一个锁，在成功获得锁之返回true，
否则立即返回false，而且线程可以立即离开去做其他事情

- `超时` 可以在使用await方法的时候提供一个超时参数

```java
myCondition.await(100, TimeUnitMILLISECONDS)
```

这样在该线程被singalAll激活、被中断或者超时时限已达到时，await方法将会返回

- `读/写锁` 如果很多线程从一个数据结构读取数据而很少线程修改其中数据的话，使用读锁是非常有用的

### 阻塞队列

对于一些多线程问题，可以使用一个或多个队列以优雅安全的方式将其形式化，生产者线程向队列插入元素，消费者线程读取它们

当试图向队列添加元素而元素已满，或者想从队列移出元素而队列为空的时候，`阻塞队列`导致线程阻塞，而且队列会自动`平衡负载`

关于阻塞队列，请看一个很实用的例程，该程序能够在指定的目录中搜索全部包含指定关键词的文件，并打印其位置

```java
package blockingQueue;

import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class BlockingQueueTest {
    private static final int FILE_QUEUE_SIZE = 10;
    private static final int SEARCH_THREADS = 100;
    // 文件处理
    private static final File DUMMY = new File("");
    // 阻塞队列
    private static BlockingQueue<File> queue = new ArrayBlockingQueue<>(FILE_QUEUE_SIZE);

    public static void main(String[] args) {
        try (Scanner in = new Scanner(System.in)) {
            System.out.print("Enter base directory (e.g. /opt/jdk1.8.0/src): ");
            String directory = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String keyword = in.nextLine();

            // 遍历所有文件的线程
            Runnable enumerator = () -> {
                try {
                    enumerate(new File(directory));
                    // 将一个空文件名放到队列尾，用来标志一个阻塞队列的结束
                    queue.put(DUMMY);
                }
                catch (InterruptedException e) {

                }
            };
            new Thread(enumerator).start();
  
            for (int i = 1; i <= SEARCH_THREADS; i++) {
                // 在文件中查找关键词的线程
                Runnable searcher = () -> {
                    try {
                        boolean done = false;
                        while (!done) {
                            File file = queue.take();
                            if (file == DUMMY) {
                                queue.put(file); // 当前队列全部搜索完之后，将DUMMY再次放入队列中，这样可以结束这一线程
                                done = true;
                            }
                            else search(file, keyword);
                        }
                    }
                    catch (IOException e) {
                        e.printStackTrace();
                    }
                    catch (InterruptedException e) {

                    }
                };
                new Thread(searcher).start();
            }
        }
    }

    public static void enumerate(File directory) throws InterruptedException {
        File[] files = directory.listFiles();
        // 找到所有子目录中的文件
        for (File file: files) {
            if (file.isDirectory()) enumerate(file);
            else queue.put(file);
        }
    }

    public static void search(File file, String keyword) throws IOException {
        try (Scanner in = new Scanner(file, "UTF-8")) {
            int lineNumber = 0;
            // 在文件的每一行查找关键词
            while (in.hasNextLine()) {
                lineNumber++;
                String line = in.nextLine();
                if (line.contains(keyword)) {
                    System.out.printf("%s:%d:%s%n", file.getPath(), lineNumber, line);
                }
            }
        }
    }
}
```

### 线程安全的集合

Java中有一些线程安全的集合，在并发访问时，即使不人为地加锁，也可以进行安全的访问

这些集合包括`ConcurrentHashMap`、`ConcurrentSkipListMap`、`ConcurrentSkipListSet`和`ConcurrentLinkedQueue`

### Callable与Future

`Callable`与Runnable类似，只不过Callable会返回一个结果，结果的类型由类型参数决定

Callable接口是只有一个方法call：

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

`Future`可以保存异步计算的结果，可以通过Future中的get方法获取这个结果

`FutureTask`包装器可以将Callable转换成Future和Runnable，同时实现二者的接口

下面是一个查找文件中关键词程序的修改版，使用了Callable和Future来实现

```java
package future;

import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class FutureTest {
    public static void main(String[] args) {
        try (Scanner in = new Scanner(System.in)) {
            System.out.print("Enter base directory (e.g. /opt/jdk1.8.0/src): ");
            String directory = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String keyword = in.nextLine();

            MatchCounter counter = new MatchCounter(new File(directory), keyword);
            // FutureTask是一个包装器，可以将Callable转换成Future和Runnable，同时使用二者的方法
            FutureTask<Integer> task = new FutureTask<>(counter);
            Thread t = new Thread(task);
            t.start();
            try {
                System.out.println(task.get() + " matching files");
            }
            catch (ExecutionException e) {
                e.printStackTrace();
            }
            catch (InterruptedException e) {

            }
        }
    }
}

class MatchCounter implements Callable<Integer> {
    private File directory;
    private String keyword;

    public MatchCounter(File directory, String keyword) {
        this.directory = directory;
        this.keyword = keyword;
    }

    // 通过重写Callable接口中的call方法，Callable与Runnable类似，当执行多线程时会自动调用call方法中的内容
    public Integer call() {
        int count = 0;
        try {
            File[] files = directory.listFiles();
            List<Future<Integer>> results = new ArrayList<>();

            for (File file: files) {
                if (file.isDirectory()) {
                    MatchCounter counter = new MatchCounter(file, keyword);
                    FutureTask<Integer> task = new FutureTask<>(counter);
                    results.add(task);
                    Thread t = new Thread(task);
                    t.start();
                }
                else {
                    if (search(file)) count++;
                }
            }

            for (Future<Integer> result: results) {
                try {
                    // get是Future对象的一个方法，由于在这里通过FutureTask将MatchCounter对象进行了包装，
                    // 所以get方法其实是递归调用counter.call，得到的是当前子目录中的count
                    int res = result.get();
                    System.out.println(res);
                    count += res;
                }
                catch (ExecutionException e) {
                    e.printStackTrace();
                }
            }
        }
        catch (InterruptedException e) {

        }
        return count;
    }

    public boolean search(File file) {
        try {
            try (Scanner in = new Scanner(file, "UTF-8")) {
                boolean found = false;
                while (!found && in.hasNextLine()) {
                    String line = in.nextLine();
                    if (line.contains(keyword)) found = true;
                }
                return found;
            }
        }
        catch (IOException e) {
            return false;
        }
    }
}
```


### 线程池

构建一个新线程是有代价的，因为涉及到与操作系统的交互，如果程序中创建了大量的`生命周期很短`的线程，应该使用`线程池`

一个线程池中包含许多准备运行的`空闲线程`，将Runnable对象交给线程池，将会有一个线程调用其中的run方法，当run方法退出时，线程`不会死亡`，而是在池中准备为下一个请求提供服务

使用线程池可以`减少并发线程的数目`

可以通过`执行器`（Executer）类来构建线程池

下面是使用线程池实现搜索文件中的关键词实例

```java
package threadPool;

import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class ThreadPoolTest {
    public static void main(String[] args) throws Exception {
        try (Scanner in = new Scanner(System.in)) {
            System.out.print("Enter base directory (e.g. /opt/jdk1.8.0/src): ");
            String directory = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String keyword = in.nextLine();

            // 创建一个线程池
            ExecutorService pool = Executors.newCachedThreadPool();

            MatchCounter counter = new MatchCounter(new File(directory), keyword, pool);
            // 将一个Callable对象交给线程池去执行
            Future<Integer> result = pool.submit(counter);

            try {
                System.out.println(result.get() + " matching files.");
            }
            catch (ExecutionException e) {
                e.printStackTrace();
            }
            catch (InterruptedException e) {

            }
            // 关闭线程池，这个操作会先完成已经提交的任务而不再接收新任务
            pool.shutdown();

            int largestPoolSize = ((ThreadPoolExecutor) pool).getLargestPoolSize();
            System.out.println("largest pool size = " + largestPoolSize);
        }
    }
}

class MatchCounter implements Callable<Integer> {
    private File directory;
    private String keyword;
    private ExecutorService pool;
    private int count;

    public MatchCounter(File directory, String keyword, ExecutorService pool) {
        this.directory = directory;
        this.keyword = keyword;
        this.pool = pool;
    }

    public Integer call() {
        count = 0;
        try {
            File[] files = directory.listFiles();
            List<Future<Integer>> results = new ArrayList<>();

            for (File file: files) {
                if (file.isDirectory()) {
                    MatchCounter counter = new MatchCounter(file, keyword, pool);
                    Future<Integer> result = pool.submit(counter);
                    results.add(result);
                }
                else {
                    if (search(file)) count++;
                }
            }

            for (Future<Integer> result: results) {
                try {
                    count += result.get();
                }
                catch (ExecutionException e) {
                    e.printStackTrace();
                }
            }
        }
        catch (InterruptedException e) {

        }
        return count;
    }

    public boolean search(File file) {
        try {
            try (Scanner in = new Scanner(file, "UTF-8")) {
                boolean found = false;
                while (!found && in.hasNextLine()) {
                    String line = in.nextLine();
                    if (line.contains(keyword)) found = true;
                }
                return found;
            }
        }
        catch (IOException e) {
            return false;
        }
    }
}
```

