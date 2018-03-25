比较Java的内部类和JavaScript的闭包



根据维基百科的定义: In programming languages, a closure (also lexical closure or function closure) is a technique for implementing lexically scoped name binding in a language with first-class functions. Operationally, a closure is a record storing a function[a] together with an environment.[1] The environment is a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created.[b] A closure—unlike a plain function—allows the function to access those captured variables through the closure's copies of their values or references, even when the function is invoked outside their scope.

大意是: 在函数是一等公民的编程语言里, 闭包是一种实现了作用域命名绑定的一种机制. 实际操作中, 闭包记录了一个函数和它的运行环境. 运行环境是关联到函数内每一个自由变量的映射(这些参量被用在函数内, 但是被定义在一个闭合的作用域里), 当闭包被创建的时候, 参量的值或引用就已经被绑定.闭包和普通函数不同的地方在于如果一个函数返回另一个函数，而被返回函数又需要外层函数的变量时，不会立即释放这个变量，而是允许被返回的函数引用这些变量。支持这种机制的语言称为支持闭包机制，而这个内部函数连同其自由变量就形成了一个闭包。

非常明显的, 定义里面要求闭包只在函数式语言出现, 即函数是一等公民, 函数式语言的要求如下:

1. 可以将一个函数赋值给一个变量。
2. 函数可以作为参数传递给另一个函数。
3. 函数的返回值可以是一个函数。

其中, JavaScript符合全部的要求, 而Java一条都不符合, 因此讲道理Java应该没有闭包这个概念. 但是呢, 有好事者将方法体内的内部类也划归到闭包这个概念里, 这其实也未尝不可, 因为从长相上他们确实差不多, 函数套函数嘛, 但即便内部类确实可以调用外部作用域的成员, 它和JS的闭包还是有着本质的区别, 我们可以先看一下JS的闭包怎么实现的.



#### 1. JavaScript闭包

​	虽然并不是JS中的闭包才叫闭包, 但是网上一搜大把的文章全是在讲JS, 显然JS的闭包已经有点被人当做"正统"的感觉了









一个是链式作用域, 一个是 传入参数







####2. Java的闭包

从函数式的定义上讲, Java不是函数式的语言, 理论上没有闭包这个概念. 但是, 匿名类和方法体内的内部类是一个函数嵌套函数的形式, 从某种意义上而言也具备了闭包的特性. 维基百科中说"allows the function to access those captured variables through the closure's copies of their values or references", 实际是把Java闭包这种copies of values也算进来的.

引用Think in Java的话, 闭包(closure)是一个可调用的对象, 它记录了一些信息, 这些信息来自于创建它的作用域. 通过这个定义, 可以看出内部类是面对对象的闭包, 因为它不仅包含外围类对象(创建内部类的作用域)的信息, 还自动拥有一个指向外围类对象的引用, 在此作用域内, 内部类有权操作所有的成员, 包括private成员.



下面是一段经典的新建线程的Java代码:

```
new Thread(new Runnable() {
            @Override
            public void run() {
                //do something...
                
            }
        }).run();
```

读一下Thread类的源码就会发现, Thread对象初始化的时候会将传入的Runnable对象作为一个Field存入. 在需要调用的时候则会调用这个Runnable对象的run()方法. Java本身没办法直接传递函数, 因此在需要传递函数的时候需要将方法封装在一个接口对象里进行传递, 这个接口还有专门的名字, 叫作函数式接口(Functional Interfaces). 具体函数式接口的作用和Lambda表达式, 之后会专门写一篇博客来讲.

说白了, Java里面的闭包就是一个匿名内部类. 之所以说Java的闭包是一个"残疾"的, 是因为它不会像JS的闭包那样保存运行环境(其实就是将在function内部保存一个指向function外部的指针, 在外部作用域被销毁时不会清除这块内存),  [前一篇博客](https://www.jianshu.com/p/0afd2ef05872)曾经提到过Java内部类在初始化的时候会将外部类对象当做一个参数传入, 因此在匿名内部类里面也可以调用到外部的参量和方法. 匿名内部类是Java中为数不多的语法糖之一(当然现在是越来越多了), 匿名内部类和局部内部类本质上是一样的, 都是在方法体内的内部类. 但是值得注意的是, 由于外部方法的参量是被直接传入的,  因此, java的闭包只能调用方法内的其他参量, 而不能改变(也就是说参量必须是final或者effective final). 这一点看一下Java闭包的实现原理就知道了, 当然在这之前我们先看一下参量可能出现的几种位置:

```
public class Outer3 {

    int a = 1;

    private void running(){
        int a = 2;
        new Thread(new Runnable() {
            int a = 3;
            @Override
            public void run() {
                //do something...
                int a = 4;
                System.out.println(Outer3.this.a);
                System.out.println(this.a);
                System.out.println(a);
            }
        }).run();
    }
    
    public static void main(String[] args) {
        Outer3 outer3 = new Outer3();
        outer3.running();
    }
}

Output:
1
3
4
```

可以看到, 在方法体内的参量(a=2), 如果名字和局部参量(a=4)一样的话, 是没办法获取到的. 





总结一下, Java在处理闭包问题的时候比较偷懒, 还是在编译时用传入参数的机制来实现调用作用域外的参量. 而没有采用顺着作用域找引用的方式, 这和Java采取的值传递非常有关系. 因为在Java中基本类型不是引用, 很多时候只能复制一份来传递参数. 这就造成了在闭包里面无法采用引用传递的模式来传参, 从而导致作用域内的代码无法修改作用域外的代码. Sun估计也比较无奈, 没办法, 一开始机制设计的有问题, 这个问题解决不了. 当然还有一种办法, 就是将外部基本类型装箱成引用类型, 这样问题也能解决, 但Sun可能觉得问题搞得有点复杂, 干脆直接一刀切, 方法体内的闭包只能调用final的参量(目前外部参量只要是没有重新赋值都可以调用).





被{}括起来的都能叫闭包?



用成员内部类实现柯里化.



#### 闭包的意义

说了一堆概念, 还是不明白闭包到底是啥意思. 

闭包最重要的意义是实现惰性运算



Java闭包说实话并没有什么太大的用, 只是方便









