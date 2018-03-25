从JVM看Java语言特性(六) 匿名内部类



匿名内部类是局部内部类的语法糖, (具体)











不能访问封装类的非final成员
this关键字变得很迷惑，如果一个匿名类有一个与其封装类相同的成员名称，内部类会覆盖外部的成员变量，在这种情况下，外部成员在匿名类内部是不可见的，甚至不能通过this来访问。

实例说明

    public void test() {  
        String variable = "Outer Method Variable";  
        new Thread(new Runnable() {  
            String variable = "Runnable Class Member";  
            public void run() {  
                String variable = "Run Method Variable";  
                System.out.println("->" + variable);  
                System.out.println("->" + this.variable);  
           }  
        }).start();  
    } 
输出
->Run Method Variable   
->Runnable Class Member
这个例子很好的说明了上面的两个问题，而Lambda表达式几乎解决上面的所有问题，我们讨论Lambda表达式之前，让我们来看看Functional Interfaces



函数式接口

​	就是一个检测接口是不是只有一个方法的东西, 方便使用lambda



lambda