legacy MARCO遗留代码问题

今天在代码里发现了个很奇怪的宏
·
#define TEST_MARCO(ARG1, ARG2, ARG3)  PRODUCT_MARCO(ARG2, ARG3)
`
L2EmTimerGetExpired

在代码调用的时候传的参数非常恶心
`
TEST_MARCO(XXXX->YYYY, arg2, arg3);
`
其中XXXX->YYYY居然都是不存在的，全是灰的，未定义

由于第一个参数是没人用的，所以正常的编译运行是没有问题的
但是，对于阅读代码就是非常恶心的了
必须改掉

代码首先是给人看的
顺便给机器执行的，得出来正确的结果只是我们事先设计预期好了的

要不代码里到处是这种奇奇怪怪的问题
而且程序还能正常执行
哪天改了一点点就哪哪都不正常了
这种代码最终一定会烂掉的
