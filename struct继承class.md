之前有人说过boost库里使用了太多的语言级别的技巧，当时没什么感触

这两天看了下boost::asio模块的代码，还没完全搞明白，但是对boost使用了太多的技巧深以为然

今天又遇到一个：
```
class task_io_service_operation BOOST_ASIO_INHERIT_TRACKED_HANDLER
{
public:
........................
}

  typedef task_io_service_operation operation;


  // Operation object to represent the position of the task in the queue.
  struct task_operation : operation
  {
    task_operation() : operation(0) {}
  } task_operation_;
```
## 这个居然是struct继承class
的确没见过，见识少就是尴尬啊

