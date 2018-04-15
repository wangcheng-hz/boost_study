NRVO 直接返回std::vector<typename T>

今天看代码发现了个很奇怪的事情，
一段代码最后直接返回了整个临时的std::vector<typename T>,
这个非常颠覆我三观

从我开始接触代码来，一直的认知就是要避免返回临时的栈变量
避免返回大块的复杂数据结构，返回指针或者要求INOUT/OUT参数

这块C++代码让我非常震惊，这是python吗？什么情况

然后我翻了下我们的编程手册，嗯，有描述：
CppCoreGuidelines.md
F.20: For "out" output values, prefer return values to output parameters

我去，居然就是鼓励直接返回的，什么情况？

必须搞清楚，google一把，嗯，自从google滚出中国后才发现baidu真是烂，尤其是搜技术文档

关于RVO/NRVO， Stack Overflow上也一堆人掐架，反正就是各种不信任，因为没保证说编译器一定支持

先不管，先写个demo感受下：
```
#include <iostream>
using namespace std;


class X {
    public:
        X(int i = 1) {
            x_ = i;
            cout << "Constructor" << x_ << endl;
        }
        X(const X& h) {
            cout << "Contsructor " << x_ << endl;
        }
    ~X() {
            cout << "Deconstruct fn" << x_ << endl;
    }
    private:
        int x_;

};

X test() {
    X a;
    X x(7);
    cout << &a << endl;
    return a;
}

int main() {
    X b = test();
    cout << "In main fn" << endl;
    cout << &b << endl;
    return 0;
}
```

编译一执行：
```
ubuntu@ubuntu:~$ ./nrvo
Constructor1
Constructor7
0x7ffed6d1a7a0
Deconstruct fn7
In main fn
0x7ffed6d1a7a0
Deconstruct fn1    //这家伙居然在main里面才析构，的确有点颠覆三观，X a;是X::test()里的栈变量啊
ubuntu@ubuntu:~$ 
```

看来NRVO是真是存在的，即便我不用c++11标准编译，也是一样的
但是如果没标准的话，那还真不能随便用，更何况即使有标准某些编译器也未必支持

查下C++11的标准文档：
chapter 12.8
    Copying and moving class objects
```
When certain criteria are met, an implementation is allowed to omit the copy/move construction of a class
object, even if the constructor selected for the copy/move operation and/or the destructor for the object
have side effects. In such cases, the implementation treats the source and target of the omitted copy/move
operation as simply two different ways of referring to the same object, and the destruction of that object
occurs at the later of the times when the two objects would have been destroyed without the optimization.126
This elision of copy/move operations, called copy elision, is permitted in the following circumstances (which
may be combined to eliminate multiple copies):
— in a return statement in a function with a class return type, when the expression is the name of a
non-volatile automatic object (other than a function or catch-clause parameter) with the same cvunqualified
type as the function return type, the copy/move operation can be omitted by constructing
the automatic object directly into the function’s return value
```
的确有相关的描述，看来C++11的确是支持的

附link：
https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/RVO_V_S_std_move?lang=en
C++11 standard：
http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3690.pdf
    
    