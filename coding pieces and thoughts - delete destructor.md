
最近阅读代码发现了一个非常有意思的class， 一个class居然delete了构造函数和析构函数  
抽象如下：
```
#include <iostream>

using namespace std;

namespace {
        int* data = nullptr;
}

class testA {
public:
        static void allocate();
        static int* get();
private:
        testA() = delete;
        testA(const testA&) = delete;
        testA& operator=(const testA& ) = delete;
        ~testA() = delete;
};

void testA::allocate()
{
        data = new int(7);
}

int* testA::get()
{
        cout << "The ptr is %p.\n" << data << endl;
}

int main()
{
        testA::allocate();
        testA::get();
        testA a;
//      testA a;
//      a.allocate();
//      a.get();
}
```
看这个文件的实现，其实就是一个单例  
为了实现共用一块公共的share_data_ptr， 把所有构造函数都delete，只留了静态的allocate函数，而且allocate函数里面还有assert，
甚至delete析构函数， 问题是这个class delete了所有构造函数，根本不会有实例可以构造出来：  
```
ubuntu@ubuntu:~/test/c++$ g++ -g -std=c++11 delete.cpp -o delete
delete.cpp: In function ‘int main()’:
delete.cpp:14:2: error: ‘testA::testA()’ is private
  testA() = delete;
  ^
delete.cpp:34:8: error: within this context
  testA a;
        ^
delete.cpp:34:8: error: use of deleted function ‘testA::testA()’
delete.cpp:14:2: note: declared here
  testA() = delete;
  ^
delete.cpp:17:2: error: ‘testA::~testA()’ is private
  ~testA() = delete;
  ^
delete.cpp:34:8: error: within this context
  testA a;
        ^
delete.cpp:34:8: error: use of deleted function ‘testA::~testA()’
delete.cpp:17:2: note: declared here
  ~testA() = delete;
  ^
ubuntu@ubuntu:~/test/c++$
```
那析构函数根本就不会走到，
当然，这块内存是在内存池里分配的，所以可以理解设计者不想这块内存被释放了  
但是这个实现的确异常丑陋,其实就是2个static的函数和一个匿名空间的指针,
的确有点丑,看来也的确不能怪老外总是觉得我们这边的代码不漂亮,老代码本来就是类C风格,然后呢5G里面又没人有勇气优化,直接抄过来...  

这个肯定有办法可以实现的更优雅一点,找找
