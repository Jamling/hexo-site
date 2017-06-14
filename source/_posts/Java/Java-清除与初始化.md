---
title: 清除与初始化
date: 2007-10-23 20:13
category: [Java]
tags: []
toc: false
---
构造函数构建函数(Constructor)属于一种较特殊的方法类型,因为它没有返回值.这与void返回值存在着明显的区别。对于void返回值，尽管方法本身不会自动返回什么，但仍然可以让它返回另一些东西。构建器则不同，它不仅什么也不会自动返回，而且根本不能有任何选择.若创建一个没有构件器的类,则编译器会自动创建一个默认构件器.
<!-- more -->
gc()与finalize()gc只能清除在堆上分配的内存(纯java语言的所有对象都在堆上使用new分配内存),而不能清除栈上分配的内存（当使用JNI技术时,可能会在栈上分配内存,例如java调用c程序，而该c程序使用malloc分配内存时）.因此,如果某些对象被分配了栈上的内存区域,那gc就管不着了,对这样的对象进行内存回收就要靠finalize().

举个例子来说,当java 调用非java方法时（这种方法可能是c或是c++的）,在非java代码内部也许调用了c的malloc()函数来分配内存，而且除非调用那个了free() 否则不会释放内存(因为free()是c的函数),这个时候要进行释放内存的工作,gc是不起作用的,因而需要在finalize()内部的一个固有方法调用它(free()).

finalize的工作原理应该是这样的：一旦垃圾收集器准备好释放对象占用的存储空间，它首先调用finalize()，而且只有在下一次垃圾收集过程中，才会真正回收对象的内存.所以如果使用finalize()，就可以在垃圾收集期间进行一些重要的清除或清扫工作.

finalize()在什么时候被调用?

1. 所有对象被Garbage Collection时自动调用,比如运行System.gc()的时候.
2. 程序退出时为每个对象调用一次finalize方法。
3. 显式的调用finalize方法

除此以外,正常情况下,当某个对象被系统收集为无用信息的时候,finalize()将被自动调用,但是jvm不保证finalize()一定被调用,也就是说,finalize()的调用是不确定的,这也就是为什么sun不提倡使用finalize()的原因.成员初始化对于类成员，你可以不初始化就直接使用它，JVM会自动对类成员进行初始化，比如，int类型初始化为0，char类型初始化为’’；在书者，作者指的就是类成员的初始化。那么成员函数的成员初始化呢？这就需要自己手动进行了，如果没有初始化而直接使用的话，编译器会给出一个错误：当前变量可能还没有初始化。
如下所示：
```java
int i;
String s;
System.out.println(this.i);//can not be i:The local variable i may not have been initialized
```

这使我想到了C/C++，在C/C++中是这样介绍的，如果局部变量没有初始化的话，那么编译器将自动赋给它一个随机值。所以在C/C++中，如果没有给局部变量赋初值（初始化）的话，将是一件“危险”的事，而Java则聪明了许多，没有初始化的局部变量，如果直接使用的话，编译不让你通过，以减少程序出现BUG的机率。

构造函数的初始化没看懂，比仙剑的迷宫还难走。搞了半天，没搞懂，惭愧！以下是原文。

static 初始化只有在必要的时候才会进行，...它们才会得到初始化在这以后，static 对象都不会重新初始化。初始化顺序首先是 static（如果它们尚未由前一次对象创建过程初始化）接着是非 static对象。大家可从输出结果中找到相应的证据。
原来是没有看完，根据以上原则，其它一些原则请参考原文。原文中的原例做一下面的分析。

```java
package ch2;
public class Test {
    int i;
    public void menberInit() {
       int i;
       String s;
       System.out.println(this.i);// can not be i:The local variable i may not
       // have been initialized
    }
    /**
     * @param args
     */
    public static void main(String[] args) {
       // Test t = new Test();
       // t.menberInit();
       /*static init
       System.out.println("Creating new Cupboard() in main");
       new Cupboard();// 13
       System.out.println("Creating new Cupboard() in main");
       new Cupboard();// 14
       t2.f2(1);
       t3.f3(1);
       */
       Mugs x = new Mugs();
    }
    //static Table t2 = new Table();// 1,初始化静态成员t2.
    //static Cupboard t3 = new Cupboard();// 8
}
class Bowl {
    Bowl(int marker) {// 3,没有静态成员/方法,执行构造函数//5
       System.out.println("Bowl(" + marker + ")");
    }
    void f(int marker) {
       System.out.println("f(" + marker + ")");
    }
}
class Table {
    static Bowl b1 = new Bowl(1);// 2,初始化静态成员b1
    Table() {// 6
       System.out.println("Table()");
       b2.f(1);// 7
    }
    void f2(int marker) {
       System.out.println("f2(" + marker + ")");
    }
    static Bowl b2 = new Bowl(2);// 4,初始化静态成员b2
}
class Cupboard {
    Bowl b3 = new Bowl(3);// 11最后才是非静态
    static Bowl b4 = new Bowl(4);// 9,先静态
    Cupboard() {// 12
       System.out.println("Cupboard()");
       b4.f(2);
    }
    void f3(int marker) {
       System.out.println("f3(" + marker + ")");
    }
    static Bowl b5 = new Bowl(5);// 10,第二个静态
}
/**
* non-static init
*/
class Mug {
    Mug(int marker) {
       System.out.println("Mug(" + marker + ")");
    }
    void f(int marker) {
       System.out.println("f(" + marker + ")");
    }
}
class Mugs {
    Mug c1;
    Mug c2;
    {
       c1 = new Mug(1);
       c2 = new Mug(2);
       System.out.println("c1 & c2 initialized");
    }
    Mugs() {
       System.out.println("Mugs()");
    }
}
```

请注意程序中注释中的序号，我喜欢用机器的方式思考方式来解这种题。
1. 发现静态成员t1，它新建一个Table对象。
在Table对象中，发现静态变量b1，新建一个Bowl对象，初始化Bowl，执行构造
在Table对象中，发现静态变量b2，新建一个Bowl对象，初始化Bowl，执行构造
Table.class没有成员/静态方法了，进行构造方法。
2. 发现静态成员t2，它新建一个Cupbord对象。
在Cupboard.class，先初始化静态成员b4,b5,然后才是非静态成员b3,最后执行其构造。
3. 程序继续往下执行，new Cupbord(), new Cupbord()，注意到这两次初始化，对已经初始化的静态b4,b5是不会进行第二次初始化的，而b3则每new 一次，则初始化一次。
4. 继续执行。
非静态实例的初始化很容易理解。
下面是例子运行的结果：
>
Mug(1)
Mug(2)
c1 & c2 initialized
Mugs()
>
注：如果没有把两条static 实例注释掉的话，将有助于理解上面的静态实例初始化。
