## this指针和类的继承

boost代码里继承关系复杂，父类，子类里this指针不断出现
那问题就来了
this指针在子类和父类里是一样的吗？

教科书上只给了个非常简单的example，根本不够用

那只能自己动手了，留个记录：
```
#include <iostream>
#include <typeinfo>
using namespace std;

class test {
public:
        test() { cout << "Now is in test class" << endl;
                this->print();
        }
        virtual void print () {
                cout << "Now in test print function" << endl;
                cout << "this pointer:" << this << " and the typename:" << typeid(*this).name() << endl;
                cout << endl;
        }
};

class testA : public test {
public:
        testA() {cout << "Now is in testA class" << endl;
                this->print();
        }
        void print () {
                cout << "Now in testA print function" << endl;
                cout << "this pointer:" << this <<  " and the typename:" << typeid(*this).name() <<endl;
                cout << endl;
        }
};

int main ()
{
        testA a;
        cout << "&a is: " << &a << endl;
        //test& rt = a;
        //rt.print();
}
```
输出：
```
[root@localhost ~]# g++ -g -std=c++11 this.cpp -o this
[root@localhost ~]# ./this
Now is in test class
Now in test print function
this pointer:0x7fffe6bc8490 and the typename:4test

Now is in testA class
Now in testA print function
this pointer:0x7fffe6bc8490 and the typename:5testA

&a is: 0x7fffe6bc8490
三个地址是一样的，这个可以证明this指针实际就是指向的实例地址

this->print的调用结果不一样
在test的构造函数里调用的是test::print(), typeid 打出来的类型是class test,数字4是随机的
在testA的构造函数里调用的是testA::print(), typeid打出来的类型是class testA，数字5是随机的
那这个证明，this指针的地址虽然是一样的，但是在不同的类里面拿到的指针类型是不一样的
这样设计也是合理的，this指针本来就是设计用访问本class类的数据的
如果在父类里也是子类类型指针的话，那不就乱了，根本没办法保证访问到的函数一定是本class，所以默认拿到的就是当前class的指针类型是最合理的
[root@localhost ~]#
```

网上搜集的this指针的一些细节，记录：
4. 关于this指针的一个经典回答:

当你进入一个房子后， 　　
你可以看见桌子、椅子、地板等， 　　
但是房子你是看不到全貌了。 　　
对于一个类的实例来说， 　　
你可以看到它的成员函数、成员变量， 　　
但是实例本身呢？ 　　
this是一个指针，它时时刻刻指向你这个实例本身

5. 类的this指针有以下特点：

（1）this只能在成员函数中使用。
全局函数、静态函数都不能使用this.
实际上，成员函数默认第一个参数为T * const this。
如：

 class A

 {

  public:

     int func(int p)

     {

     }

 };

其中，func的原型在编译器看来应该是：
  int func(A * const this,int p);

（2）由此可见，this在成员函数的开始前构造，在成员函数的结束后清除。
这个生命周期同任何一个函数的参数是一样的，没有任何区别。
当调用一个类的成员函数时，编译器将类的指针作为函数的this参数传递进去。如：
A a;

a.func(10);

此处，编译器将会编译成：
A::func(&a,10);

看起来和静态函数没差别，对吗？不过，区别还是有的。
编译器通常会对this指针做一些优化，因此，this指针的传递效率比较高--如VC通常是通过ecx寄存器传递this参数的。

（3）几个this指针的易混问题。
A. this指针是什么时候创建的？
this在成员函数的开始执行前构造，在成员的执行结束后清除。
但是如果class或者struct里面没有方法的话，它们是没有构造函数的，只能当做C的struct使用。
采用 TYPE xx的方式定义的话，在栈里分配内存，这时候this指针的值就是这块内存的地址。
采用new的方式 创建对象的话，在堆里分配内存，new操作符通过eax返回分配的地址，然后设置给指针变量。
之后去调 用构造函数（如果有构造函数的话），这时将这个内存块的地址传给ecx，之后构造函数里面怎么处理请 看上面的回答。

B. this指针存放在何处？堆、栈、全局变量，还是其他？
this指针会因编译器不同而有不同的放置位置。
可能是栈，也可能是寄存器，甚至全局变量。
在汇编级 别里面，一个值只会以3种形式出现：立即数、寄存器值和内存变量值。
不是存放在寄存器就是存放在内存中，它们并不是和高级语言变量对应的。

C. this指针是如何传递类中的函数的？绑定？还是在函数参数的首参数就是this指针？那么，this指针 又是如何找到“类实例后函数的”？
大多数编译器通过ecx寄存器传递this指针。
事实上，这也是一个潜规则。
一般来说，不同编译器都会遵从一致的传参规则，否则不同编译器产生的obj就无法匹配了。
在call之前，编译器会把对应的对象地址放到eax中。
this是通过函数参数的首参来传递的。
this指针在调用之前生成，至于“类实例后函数”，没有这个说法。
类在实例化时，只分配类中的变量空间，并没有为函数分配空间。
自从类的函数定义完成后，它就在那儿，不会跑的。

D. this指针是如何访问类中的变量的？
如果不是类，而是结构体的话，那么，如何通过结构指针来访问结构中的变量呢？如果你明白这一点的话，就很容易理解这个问题了。

在C++中 ,类和结构是只有一个区别的：类的成员默认是private，而结构是public。
this是类的指针，如果换成结构，那this就是结构的指针了。

E. 我们只有获得一个对象后，才能通过对象使用this指针。如果我们知道一个对象this指针的位置，可以直接使用吗？
this指针只有在成员函数中才有定义。
因此，你获得一个对象后，也不能通过对象使用this指针。
所以，我们无法知道一个对象的this指针的位置（只有在成员函数里才有this指针的位置）。
当然，在成员函数里，你是可以知道this指针的位置的（可以通过&this获得），也可以直接使用它。

F. 每个类编译后，是否创建一个类中函数表保存函数指针，以便用来调用函数？
普通的类函数（不论是成员函数，还是静态函数）都不会创建一个函数表来保存函数指针。
只有虚函数才会被放到函数表中。
但是，即使是虚函数，如果编译器能明确知道调用的是哪个函数，编译器就不会通过函数表中的指针来间接调用，而是会直接调用该函数。
