### 1. 构造器解析

只要学过Java的人都知道, JVM在创建一个对象的时候会通过构造函数来决定这个类要如何构造, 构造函数在Java中是非常特殊的一类"方法", 通过这篇文章我们来探索一下构造器的底层实现. 

还是以Tree.java为例, 分析一下Tree的字节码

```
public class Tree {

    public static int state = 1;
    int height;

    public Tree(){
        height = 0;
    }

    public Tree(int initialHeight){
        height = initialHeight;
    }

    public static void message(String message){
        return;
    }
}
```

我定义了一个静态字段外加一个静态方法, 之后可以顺带分析一下带static标识的字段和方法是如何构造的. 现在先来看Tree的两个构造器:

```
 public Dao.Tree();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iconst_0
         6: putfield      #2                  // Field height:I
         9: return
      LineNumberTable:
        line 8: 0
        line 9: 4
        line 10: 9

  public Dao.Tree(int);
    descriptor: (I)V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iload_1
         6: putfield      #2                  // Field height:I
         9: return
      LineNumberTable:
        line 12: 0
        line 13: 4
        line 14: 9

```

从字节码中我们看出, 无论是怎么样的构造器, 在一开始都会调用<init>,  在之后才是构造器自己的逻辑. 事实上, 对于一个类的任何构造器, 编译器都会修改构造函数的字节码指令, 将Java类成员变量的初始化指令<init>插入到构造方法中, 那么这个<init>是什么呢? <<揭秘Java虚拟机>>的作者认为<init>就是无参构造器, 简直就是在胡说八道, 我们不妨试一下把无参构造去掉看看会是是怎么样的情况:

```
public class Tree {

    public static int state = 1;
    int height;

    public Tree(int initialHeight){
        height = initialHeight;
    }

    public static void message(String message){
        return;
    }
}
```

构造器的字节码为:

```
public Dao.Tree(int);
    descriptor: (I)V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iload_1
         6: putfield      #2                  // Field height:I
         9: return
      LineNumberTable:
        line 8: 0
        line 9: 4
        line 10: 9

```

非常明显的, 这个类里面没有无参构造器, 但是却有<init>, 这个结果非常轻易的就打了脸. 这样看来<init>和无参构造器没有什么关系, 那么<init>里面到底干了点啥呢? 实际上, 在<init>中, JVM主要完成Java类成员变量的初始化逻辑, 同时会执行Java类中被{}包裹的块逻辑. 如果成员变量没有被赋值, 那么不会被初始化, 也就不会放在<init>里面.

我们不妨举个例子, 对于下面这个类:

```
public class Tree {
    int height = 3;

    public Tree(){
    }

	public Tree(String message){
    	System.out.println(message);
	}
}
```

它在JVM里面的表示应该就是:

```
public class Tree {
    int height;

    public Tree(){
    	height = 3;
    }

	public Tree(String message){
		height = 3;
    	System.out.println(message);
	}
}
```

这样就非常清楚了, <init>的作用就是把写在构造器外的字段赋值和{}块写到构造器里面, 让所有构造器有了共同的一段初始化, 减少了代码的重复性.

这里插几句, 关于构造器到底是个什么东西, <<Think in Java>>中认为构造器是一个静态方法的, 这个观点是非常有问题的. 首先, 在调用构造器的时候会将this引用传入, 字节码中对应的就是aload_0, 而静态方法因为是全局的, 是不可能传入this的, 这一点决定了他不可能是一个静态方法. 那么它有没有可能是一个私有的方法呢? 也不可能, 非常明显一点的是构造器本身是跟着这个类走的, 就连它是不是方法都不好说, 因为除了再new一个对象的时候会调用这个构造器, 其他时候是没有使用的(更正: 反射可以将构造器反射出来, 并且通过newInstance调用). 从这一点说的话构造器可以说是一种非常特殊的"方法".

---

### 2. 静态初始化解析

一旦理解了<init>的含义, <clinit>就会显得很简单, 因为它们俩的作用基本差不多, 只不过一个是普通成员的初始化, 一个静态初始化. 本文开头的Tree.java的静态相关的的字节码如下

```
 public static void message(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 17: 0

  static {};
    descriptor: ()V
    flags: (0x0008) ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: iconst_1
         1: putstatic     #3                  // Field state:I
         4: return
      LineNumberTable:
        line 5: 0

```

这里的static {}就是<clinit>方法, 作用是完成了静态字段的赋值, 和执行static代码段. 我们列一张表来对比一下<init>和<clinit>的区别:

|       | <init>          | <clinit>             |
| ----- | --------------- | -------------------- |
| 初始化内容 | 普通成员变量赋值以及{}代码块 | 静态变量赋值以及static {}代码块 |
| 调用时机  | 对象构建时           | 类加载时(初始化)            |
| 继承    | 可以继承            | 不能继承                 |

可以看出, <clinit>不仅加载时间比<init>早很多, 并且在类加载时就会被调用. 关于<clinit>的调用时机, 一个最明显的例子就是注册JDBC Driver的时候只需要加一句

Class.forName("com.mysql.jdbc.Driver");

Driver就自己加载了, 这是为什么呢? 其实非常简单, com.mysql.jdbc.Driver这个类一共就一个构造方法再加一个下面的静态代码段, 在forName的时候这段代码就自动被加载进JVM, 完成了driver的注册:

```
static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```

挺有意思的吧?

---

### 3. 构造器的继承解析

在构造器的继承中, <init>和<clinit>这两者到底扮演着什么样的角色是我们所关心的, 简单来说<clinit>不具有继承性, 原因很简单, 父类和子类是分别加载的, 在父类加载时就已经执行过<clinit>方法, 就无需在子类再加载一遍, 所以子类只需要完成自己的<clinit>就行.

而对于<init>来说情况就比较复杂, 当Java类显示继承父类时, 则Java编译器会让子类的各个构造函数调用父类的默认构造函数<init>(). 从而在子类实例化是完成父类成员变量的初始化逻辑.

举个例子, 彻底理清新建对象时候的初始化问题:

```
public class Father {

    {
        System.out.println("father.{}");
    }

    static {
        System.out.println("father.static{}");
    }

    public Father(){
        System.out.println("father.constructor()");
    }
}

public class Son extends Father{

    {
        System.out.println("Son.{}");
    }

    static {
        System.out.println("Son.static{}");
    }

    public Son(){
        System.out.println("Son.constructor");
    }

    public static void main(String[] args) {
        Son son = new Son();
    }
}
```

执行结果为:

```
father.static{}
Son.static{}
father.{}
father.constructor()
Son.{}
Son.constructor
```

从这个结果来看, 当新建一个对象时, 总是先加载父类再加载子类, 并且执行了他们的静态初始化过程, 也就是<clinit>, 接着再执行父类的默认无参构造函数(注意, 这里只能是无参构造器, 如果父类没有无参构造器, 编译器会报错), 最后才是子类的构造函数. 

从这个结果中我们还能看出, {}代码段是在构造器的输出之前执行的, 也间接证明了<init>被插入在了构造器的最前面.

