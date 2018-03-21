比较Java的内部类和JavaScript的闭包



引用Think in Java的话, 闭包(closure)是一个可调用的对象, 它记录了一些信息, 这些信息来自于创建它的作用域. 通过这个定义, 可以看出内部类是面对对象的闭包, 因为它不仅包含外围类对象(创建内部类的作用域)的信息, 还自动拥有一个指向外围类对象的引用, 在此作用域内, 内部类有权操作所有的成员, 包括private成员.

根据维基百科的定义: In programming languages, a closure (also lexical closure or function closure) is a technique for implementing lexically scoped name binding in a language with first-class functions. Operationally, a closure is a record storing a function[a] together with an environment.[1] The environment is a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created.[b] A closure—unlike a plain function—allows the function to access those captured variables through the closure's copies of their values or references, even when the function is invoked outside their scope.



闭包是指有权访问另一个函数作用域中的变量的函数, 访问上层函数的作用域的内层函数就是闭包

 如果一个函数返回另一个函数，而被返回函数又需要外层函数的变量时，不会立即释放这个变量，而是允许被返回的函数引用这些变量。支持这种机制的语言称为支持闭包机制，而这个内部函数连同其自由变量就形成了一个闭包。

 闭包是一类特殊的函数。如果一个函数定义在另一个函数的作用域中，并且函数中引用了外部函数的局部变量，那么这个函数就是一个闭包。



即便内部类确实可以调用外部作用域的成员, 它和JS的闭包还是有着本质的区别, 我们可以先看一下JS的闭包怎么实现的.



#### 数学上的闭包





#### 1. JavaScript闭包

​	虽然并不是JS中的闭包才叫闭包, 但是网上一搜大把的文章全是在讲JS, 显然JS的闭包已经有点被人当做"正统"的感觉了









一个是链式作用域, 一个是 传入参数







2. Java的闭包

   Java闭包的工作原理



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

说白了, Java里面的闭包就是一个匿名内部类. 之所以说Java的闭包是一个"残疾"的, 是因为它不会像JS的闭包那样保存运行环境(其实就是将在function内部保存一个指向function外部的指针, 在外部作用域被销毁时不会清除这块内存),  [前一篇博客](https://www.jianshu.com/p/0afd2ef05872)曾经提到过Java内部类在初始化的时候会将外部类对象当做一个参数传入, 因此在匿名内部类里面也可以调用到外部类的参量和方法. 但是值得注意的是, 外部类的参量









被{}括起来的都能叫闭包?

















