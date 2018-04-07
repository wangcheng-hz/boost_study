## C++ Type Traits

今天看basic_io_object.hpp的代码，发现一个很奇怪的用法：
```
#if !defined(BOOST_ASIO_HAS_MOVE)
template <typename IoObjectService>
#else
template <typename IoObjectService,
    bool Movable = detail::service_has_move<IoObjectService>::value>
#endif
class basic_io_object

........

#if defined(BOOST_ASIO_HAS_MOVE)

template <typename IoObjectService>
class basic_io_object<IoObjectService, true>
```

简单的分析下这2个basic_io_object的定义，在BOOST_ASIO_HAS_MOVE为Yes的时候似乎有重复定义了

i) BOOST_ASIO_HAS_MOVE = Yes
	template <typename IoObjectService,
		bool Movable = detail::service_has_move<IoObjectService>::value>
	class basic_io_object {}

	template <typename IoObjectService>  //为什么没重名冲突？
	class basic_io_object<IoObjectService, true>
	
ii) BOOST_ASIO_HAS_MOVE = No
	template <typename IoObjectService>
	class basic_io_object {}
	
这个代码肯定是可以工作的
写段小代码简单验证下：
```
#include <iostream>
/*
class test {
public:
        int a;
};
*/

template <typename Type, typename T>
class test {};

template <typename Type>
class test<Type, int> {};

template <typename IoObjectService,
        bool Movable = false>
class basic_io_object {};

template <typename IoObjectService>
class basic_io_object<IoObjectService, true> {};

int main()
{
//      test t1;
        test<int, char> t2;   //对于class test,只能有一个，而且实例化的时候必须有2个template args，即使有模板类的偏特化
        test<int, int> t3;  //  

        basic_io_object<int> io;  // 因为template args就一个，还有个是参数，所以可以basic_io_object<int>去实例化，同理也就解释了boost代码为什么会能工作
}

'''

## 对主版本模板类、全特化类、偏特化类的调用优先级从高到低进行排序是：全特化类>偏特化类>主版本模板类。这样的优先级顺序对性能也是最好的。
