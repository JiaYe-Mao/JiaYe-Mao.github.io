从JVM看Java语言特性(四)  接口和抽象类

接口和抽象类都是上层抽象, 一个类可以实现多个接口却只能继承一个抽象类. 从上一篇文章中我们大致明白了继承和多态是如何实现的, 多态通过JVM在vtable放置不同的方法指针来决定到底是调用父类的方法还是子类的方法. 按照多态的这个思路的话, 接口应该就是一个指示器, 方便JVM来判断vtable里面需要哪些方法. 那么到底是不是这么回事儿我们目前还不知道, 这篇文章让我们探索一下接口和抽象类的一些细节.



#### 1. 接口与抽象类的实现

如果一个**接口**里面有一个getA()方法的话, 它的方法字节码是这样的:

```
 public abstract int getA();
    descriptor: ()I
    flags: (0x0401) ACC_PUBLIC, ACC_ABSTRACT

```



另一方面, 如果一个__抽象类__里面只有一个abstract getA()方法, 它的方法字节码是这样的:

```
public Dao.FInterface();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  abstract int getA();
    descriptor: ()I
    flags: (0x0400) ACC_ABSTRACT

```

可以看到, 抽象类有构造器, 如果里面没有抽象方法的话它和一般的__类__没什么区别. 但是如果抽象类里有抽象方法的话, 那么这些抽象方法和**接口**里的抽象方法也几乎没有区别, 唯一的区别是抽象类的抽象方法可以是加上访问修饰符, 如public或protected, 也可以不加修饰符(给抽象方法加private会报错, 因为这样的话这个抽象方法永远无法被实现, 进而就没办法生成这个类的对象, 没有意义), 而接口只能为public(不加修饰符也会默认为public).

这样来看, 抽象类的结构像是一个普通类加上接口的抽象方法. 

接口不能有构造器, 因为接口是一种规范, 类可以实现多个接口，若多个接口都有自己的构造器，则不好决定构造器链的调用次序.



#### 2. 接口和抽象类的意义

传统多继承的复杂度一直是逃避不开的问题, 如果继承的两个父类里面包含了相同名称的方法或参量, 那么 JVM无法判断到底需要执行哪一种, 冲突问题就会非常明显. 由此, Java采用了单继承来避免这个问题, 为了提高灵活程度, 又引入接口来实现轻耦合的多继承. 接口用不会引起冲突的方式解决了多重继承的问题

接口的目的是实现, 它和实现类的关系是"has a", 也就是说接口可以是实现类的一个组件, 一个部分. 

抽象类的目的则是继承, 它和实现类的关系是"is a", 其实完全可以不存在抽象类这个东西, 用一个普通类代替抽象类, 将所有抽象方法都改成空的实方法, 然后再让子类将其重写, 这也就成了变相的抽象类. 这样子想下去抽象类的唯一作用就是帮助程序员少犯错(抽象类无法实现, 只能多态, 减少程序员误操作的可能), 并且帮助程序员理顺继承关系(名字就叫抽象类, 一看就知道可能有多个类继承于它). 

在知乎上看到的一句话说的特别好[[传送门](https://www.zhihu.com/question/20149818/answer/153188511)] "抽象类主要是用来抽象类别, 接口主要是用来抽象方法功能. 当你关注事物的本质的时候, 用抽象类; 当你关注一种操作的时候用接口", 



#### 3. 抽象类与接口中的参数

抽象类由于是一个类, 并且有构造函数, 所以它和一个普通类对参数的处理是一样的, 具体可见我的[这篇文章](https://www.jianshu.com/p/5bdab05eaa38), 因此这里就不展开了.

但是对于接口来说, 没有构造器的话接口是如何初始化它的数据的? 这个问题值得探讨. 我们假设有下面这个接口:

```
public interface FInterface {

    int a = 1;

    public static int c = 2;

    public static final int d = 1;

    int getA();

}
```

它的方法字节码是:

```
public static final int a;
    descriptor: I
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 1

  public static final int b;
    descriptor: I
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 2

  public static final int c;
    descriptor: I
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 1

  public abstract int getA();
    descriptor: ()I
    flags: (0x0401) ACC_PUBLIC, ACC_ABSTRACT

```

经过测试, 发现无论有没有public static final, 生成的class文件里都会有这三个关键词. 也就意味着接口里面所有的参数都是常量, 在类加载过程中都已经被解析并且被放进了常量池里. 



#### 4. jdk8 和 jdk9 中接口的变化

1. jdk8 中接口允许有default方法

   default方法是指接口默认实现的方法, 主要是为了解决旧版本的兼容性问题, 以下是无default关键字和有default关键字的字节码对比:

   ```
    public abstract int getA();
       descriptor: ()I
       flags: (0x0401) ACC_PUBLIC, ACC_ABSTRACT
   ```

   ```
   public int getA();
       descriptor: ()I
       flags: (0x0001) ACC_PUBLIC
       Code:
         stack=1, locals=1, args_size=1
            0: iconst_1
            1: ireturn
         LineNumberTable:
           line 13: 0

   ```

   可以发现default方法就是一个普通类里面实际存在的方法. 这样Java就实现了方法的多继承, 虽然设计的初衷并不是让使用者去滥用default, 只是一种为了不破坏之前代码的一种妥协.

   那如果两个接口都有相同的方法不就冲突了吗? 其实可以使用接口名+.super.+方法名来区分不同的方法, 就像下面这样:

   ```
   A.super.getA();
   B.super.getA();
   ```

   这样就不会引起冲突了. 但是Java为了让程序员少犯错误, 除非实现类重写了这个方法, 不允许实现的两个接口里有相同的default方法.

   除此之外, jdk9支持接口存在私有方法, 由于私有方法不存在子类直接调用, 因此也不会有冲突的问题, 这里就不再展开了.

   ​

总结

​	为了写这一章, 本人查阅了大量资料, 包括了很多博客. 但是经过实践之后, 很多博客都有或多或少的错误, 比如有的说接口不能有静态方法(静态方法的解析是类加载的一部分, 当然可以有!), 我自己以后写博客的时候会尽量找官方或者权威的说法来论证, 不会轻易相信网上的东西.

​	总的来说, 接口和抽象类是越来越相似了, 两者的主要区别在于接口可以实现多个和接口没有构造器.