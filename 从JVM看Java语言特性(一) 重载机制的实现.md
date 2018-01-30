大家应该都知道, 在C语言中是没有重载的, 因为在编译过程中编译器只检查函数名, 不管参数是否一致. 因此编译器无法区分两个同名函数的不同. 有时候在同一函数中需要不同入参会比较麻烦, 需要在方法结尾加上点标识来实现重载, 比如:

```
_syscall0(type,name)
_syscall1(type,name,type1,arg1)
_syscall2(type,name,type1,arg1,type2,arg2)
_syscall3(type,name,type1,arg1,type2,arg2,type3,arg3)
_syscall4(type,name,type1,arg1,type2,arg2,type3,arg3,type4,arg4)
_syscall5(type,name,type1,arg1,type2,arg2,type3,arg3,type4,arg4,type5,arg5)
_syscall6(type,name,type1,arg1,type2,arg2,type3,arg3,type4,arg4,type5,arg5,type6,arg6)
```

 

而在Java和C#中, 编译器允许在同一个类中的多个方法使用相同的方法名, 编译时通过不同的参数类型列表来区分. 例如在下面的例子中非常方便的重载了构造器, 来实现多种构造方法:

``` java
class Tree(){
	int height;

    public Tree(){
        height = 0;
    }

    public Tree(int initialHeight){
        height = initialHeight;
    }
}
```

这样的写法让developer不需要记一长串的方法名, 降低了编程的难度, 那么重载到底在JVM里面是如何实现的呢? 让我们继续深入~



 Java在编译时先将.java文件编译成.class文件, 在class字节码中, 方法签名主要由三部分组成:

1. 方法的标识(access_flag): public, private, static, final, synchronized, native等
2. 方法的名称(name)
3. 方法的描述(descriptor),  包括方法的返回值类型和入参信息

这三个部分的内容可以唯一确定一个方法, 那么java重载的秘密也就揭开了, jvm在运行期间不仅检查方法的名字, 同时还会区分入参和返回值.

我们可以实际看一下Tree.java 的字节码文件来验证一下, 用javap -verbose Tree.class 来解析Tree, 会发现这两个构造器的字节码是这样的:

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
        line 7: 0
        line 8: 4
        line 9: 9

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
        line 11: 0
        line 12: 4
        line 13: 9

```

可以看到对于第一个构造方法, 标识为public, 名字为Dao.Tree(自己加了一个叫Dao的package...), 方法描述叫作()V.

而对于第二个构造方法, 标识为public, 名字同样为Dao.Tree, 方法描述叫作(I)V.  这里返回值int之所以叫作I是因为JVM规范中规定了基本类型的简写, 这里的I就是int 的简写.



我们理解了方法重载的原理之后, 由此继续深入思考下去, 照这么理解的话方法的返回值也会影响重载, 理论上只要编译器可以确定一个方法的返回值是可以实现不同返回值的重载的. 但是要是出现如下情况的话编译器就会无法判断到底应该用哪个重载方法:

```
	private int getHeight(String message){
		System.out.println("return int!");
        return height;
    }

    private Integer getHeight(String message){
    	System.out.println("return Integer!");
        return height;
    }

    public static void main(String[] args) {
        Tree tree = new Tree();
        tree.getHeight("编译器报错");
    }
```

getHeight(String)这个方法中, 因为返回值没有赋值给任何变量, 因此编译器无法判断返回值, 此时编译器报错,  错误提示是该方法已经定义, 无法重复定义.

可以看到, 返回值是无法作为重载的区分的, 这不是因为JVM里面不能同时存在这两个方法, 而是因为在某些情况下无法判断返回值, 导致编译器干脆直接不允许这样写(编译器内心: 怪我喽). 这样的话想要实现返回值的重载就只能再写一个不同名的方法了.

除此之外参数的顺序也会决定函数的重载, 比如下面的两个方法:

```
	private int getHeight(String message, Integer a){
        System.out.println(message + height);
        return height;
    }

    private int getHeight(Integer a, String message){
        return height;
    }
```

字节码为:

```
public int getHeight(java.lang.String, java.lang.Integer);
    descriptor: (Ljava/lang/String;Ljava/lang/Integer;)I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=3
         0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: aload_1
         4: aload_0
         5: getfield      #2                  // Field height:I
         8: invokedynamic #4,  0              // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;I)Ljava/lang/String;
        13: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        16: aload_0
        17: getfield      #2                  // Field height:I
        20: ireturn
      LineNumberTable:
        line 16: 0
        line 17: 16

  public int getHeight(java.lang.Integer, java.lang.String);
    descriptor: (Ljava/lang/Integer;Ljava/lang/String;)I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=3, args_size=3
         0: aload_0
         1: getfield      #2                  // Field height:I
         4: ireturn
      LineNumberTable:
        line 21: 0
```

可以看到, 入参不同顺序也会生成两个不同的方法, 唯一的问题是仅仅是依靠参数的顺序来生成两个不同的函数非常容易造成误解, 一般要避免使用这样的写法.