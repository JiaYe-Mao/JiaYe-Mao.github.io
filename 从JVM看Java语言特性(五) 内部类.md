从JVM看Java语言特性(五) 成员内部类



#### 1. 成员内部类

总的来说, 内部类其实就是在一个类中定义一个类, 举个简单的例子:

```
public class Outer {

    int a;

    class Inner {
        private int getB(){
            return a;
        }
    }

    public static void main(String[] args) {
        Inner inner = new Outer().new Inner();
        System.out.println(inner);
    }
}
```

对于上面这个类来说, Inner类被嵌在了Outer类的里面, 那么成员内部类的意义在哪里呢? 在大多数时候成员内部类的作用不大, 容易被其他方法所取代, 那么我们就以ArrayList里面的iterator()方法来举例子吧:

```
public class ArrayList<E> {         //这里还继承和实现了很多类和接口, 这里就简化一下, 看得清楚一点

	//something else...
	
	public Iterator<E> iterator() {
        return new Itr();
    }
    
    private class Itr implements Iterator<E> {
    	public boolean hasNext() {//...}
        
        public E next() {//...}
        
        public void remove() {//...}    
    }
}

//调用方法:
//new ArrayList().iterator();
```

在ArrayList里面, iterator()方法返回的是一个内部类对象, 因为这个内部类实现了Iterator接口, 因此对外隐藏了内部类细节, 让外部使用者以为自己拿到的直接就是Iterator接口对象. 但是实际上, 像下面这种写法依然正确, 并且这种写法还减少了内部类这个概念, 看上去简单了不少:

```
public class ArrayList<E> implements Iterator<E> {

	//something else...
	
	public Iterator<E> iterator() {
        return this;
	}

	public boolean hasNext() {//...}
        
     public E next() {//...}
        
     public void remove() {//...} 

}

//调用方法:
//new ArrayList().iterator();
```

可以看到, 在这样的情况下, 使用成员内部类和不使用成员内部类可以达到一样的效果. 那么是不是不用内部类会更好呢? 并不是, 这里之所以会觉得这两种写法没什么差别, 是因为这里的代码很简单, 如果我们把ArrayList的代码完整的写出来, 整个代码会非常的长. 由于实现了多个接口, 在return不同的接口对象的时候会显得非常混乱, 试想下面这个代码写出来会是什么样子:

```
public class Outer3 implements Interface1, Interface2, Interface3...
{
    public Iterface1 iterface1(){
        return this;
    }

    public Iterface2 iterface2(){
        return this;
    }
    
    public Iterface3 iterface3(){
        return this;
    }      
    
    //一堆乱七八糟的代码, 分不清哪个是哪个接口的
    //...
}
```

这个类实现了多个接口, 如果代码一多很容易就乱了, 由于所有的interface()方法都是return this, 会导致很难看懂代码想要表达的意思. 而如果使用内部类, 就可以帮助我们将实现不同接口的代码进行分类, 让代码更加优雅清晰.

```
public class Outer3 
{
    public Iterface1 iterface1(){
        return new Ite1();
    }

    public Iterface2 iterface2(){
        return new Ite2();
    }

    public Iterface3 iterface3(){
        return new Ite3();
    }

    private class Ite1 implements Interface1{
		//..
    }

    private class Ite2 implements Interface2{
		//..
    }

    private class Ite3 implements Interface3{
		//..
    }

}
```

上面的代码中将不同接口相关的代码放在一起, 就使得代码一下子就变得清晰了许多. 



#### 2. 成员内部类的机制

还是以一开始那一段代码为例:

```
public class Outer {

    int a;

    class Inner {
        private int getB(){
            return a;
        }
    }

    public static void main(String[] args) {
        Inner inner = new Outer().new Inner();
        System.out.println(inner);
    }
}
```

可以发现内部类可以调用外部类的变量和方法, 然而将java文件编译成字节码, 发现字节码Outer.class中并没有和内部类有关的部分, 仅仅在最后加了一句说明:

```
InnerClasses:
  #9= #2 of #3;                           // Inner=class Dao/Outer$Inner of class Dao/Outer
```

这是在Class文件里有记录的很少量的反射信息，也是VM层面上唯一能表现类的嵌套关系的东西, 该信息可以通过Java的反射API暴露出来（例如 java.lang.Class.getEnclosingClass() ). 

那为什么内部类可以调用外部类的参量呢? 答案就在下面的代码里, 编译器会在内部类的每一个构造器中都添加一个外部类对象, 并且JVM会在创建内部类实例时把调用的外部类对象一并传入, 造成外部类把内部类包裹起来形成闭包的"假象".

```
class Outer$Inner {
    Outer$Inner(Outer var1) {
        this.this$0 = var1;
    }

    private int getB() {
        return this.this$0.a;
    }
}
```

可以看出, 内部类和外部类几乎完全没有联系, 两者都是顶层类.

其实是Java语言编译器（例如javac、ECJ）在将Java源码编译到Class文件的过程中，将内部类做了“解糖”，给其添加一些必要的转换之后将其提升为跟顶层类一样的形式，然后后面就不再有内部类与否的区别了。于是JVM就不用关心什么是内部类了。



####3. 成员内部类的静态字段问题

​	java语法规定, 成员内部类里不能存在静态字段和方法.

​	 就本人的看法而言, JVM完全可以实现静态字段和静态方法, 因为一个类里面能否有静态纯粹取决于在类解析阶段是否有<clinit>方法, 关于这一点大量博客的观点都有问题, 被它们带偏了就完了, 例如很多文章都将静态变量的加载和创建实例建立联系, 认为在new一个实例前不能调用静态方法, 这明显是种谬误, 因为此二者并没有什么关系.

​	<以下纯属猜测>至于编译器为什么不允许在内部类里实现静态字段, 我认为这和Java规范有关, java不希望使用者在外部类的一个"成员"中加入静态字段, 让语义上出现歧义, 在使用上完全可以吧内部类的静态字段放在外部类里, 达到一样的效果.



#### 4. 值得注意的地方

​	我们已经知道了内部类调用外部对象的原理, 也就是将外部类的对象引用复制一份作为参数传入内部类里去, 那是不是意味着在内部类里面修改外部类的值会失败呢? 其实是不会的, 我们可以看一下下面这个例子:

```
public class Outer {

    int a = 1;

    class Inner {
        private int getB(){
            a = 2;
            return a;
        }
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        Inner inner = outer.new Inner();
        System.out.println("a has been changed in Inner class! a = " + inner.getB());
        System.out.println("a in outer class: a = " + outer.a);
    }
}

// output:
// a has been changed in Inner class! a = 2
// a in outer class: a = 2
```

​	可以看到,  在内部类里修改的a值在外部类里也变了, 这是为什么呢? 这里涉及到对值传递和引用传递的理解, 虽然内部类在初始化的时候确实是传入的一个外部类对象引用的复制, 但是这个引用指向的内存是和原来的对象是同一块, 不管用哪个引用来修改对象, 只要不是将这个引用指向了其他对象, 都可以修改到同一个对象. 以这段代码为例, 内部类中的赋值语句a=2, 其实就是outer.a = 2, outer是外部类的对象复制过来的一份引用, 但是也能对这个对象进行操作.

​	所以, 在内部类里修改外部类对象的参量是可以修改成功的, 原因就是传入的引用指向的内存空间就是真实的外部类对象的内存空间.  这一点容易和匿名类调用外部类的变量必须是final混淆起来.

​	如果将我们内部类推广下去, 嵌套更多层的内部类来看一下结果, 

```
public class Outer {

    int a = 1;

    class Inner {

        private int innerGetA(){
            a = 2;
            return a;
        }

        class SecondInner{
            private int doubleInnerGetA(){
                a = 3;
                return a;
            }
        }
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        Inner inner = outer.new Inner();
        Inner.IInner iInner = inner.new IInner();
        System.out.println("a has been changed in Inner class! a = " + inner.getB());
        System.out.println("a in outer class: a = " + outer.a);
        System.out.println(iInner.getC());
        System.out.println(outer.a);
    }
}

// output:
// a has been changed in Inner class! a = 2
// a in outer class: a = 2
// a has been changed in SecondInner class! a = 3
// a in outer class: a = 3
```

可以看出内部类是可以一层一层嵌套下去的, 而且外部类的参数传递会继续进行下去. 如果我们将内部类推广下去也是一样的结果, (有种数学归纳法的味道..) , 



#### 5. 典型实现

​	之前写Android的时候用到的OnClickListener, 各大教程提供了OnClickListener的N种写法, 它们会说用内部类是这样写, 用匿名类是这么写, 而用外部类是这么写. 本身这些说法没有问题, 但是这些教程往往流于表面, 只告诉你有多种写法, 忽视了它的本质, 造成的后果就是很多人直接把listener的写法给背了下来, 完全不知道listener是怎么回事儿. 

​	OnClickListener的本质就是一个回调函数, Java没法像JS一样直接传递一个函数, 只能把回调方法放在一个类里面传递. Android教程里面所说的N种写法, 其实就是写一个类的N种写法. 当然也可以用Lambda来写, 更加简洁.



####6. 成员内部类的意义

​	除了我开头所讲的内部类的作用之外, 内部类还有很多其他的意义, 具体可以参考[这篇回答](https://www.zhihu.com/question/21373020/answer/131513780), 讲的比较清楚, 概括一下就是说在普通的情形下, 完全可以不用内部类, 直接继承一个接口写方法也可以达到一样的结果, 但是在特定情形下内部类就能发挥作用:

  1.  外部类不止有一种接口实现方法, 就可以写多个内部类分别继承, 使用时根据不同的内部类对象来区分实现.
  2.  多重继承, 这一点也被Think in Java认为是使用外部类的主要原因, 需要多重继承了怎么办? 那就在里面写个内部类继承吧. 这样做的非常轻松的就能实现多重继承, 但是如果要进一步再写一个子类就没法继承这两个类了, 内部类终究只是缺失多重继承的一种弥补.



 








