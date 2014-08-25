runtime.LockOSThread
================================================
函数用于将当前的goroutine与当前的的本地线程进行一对一的绑定。在使用本地调用的时候，这是个非常有用的机制。


对于一些使用thread local storage 的本地函数， 比如说errno在多线程环境，就使用了thread local storage来存放 errno值。

对于用到thread id的一些本地函数调用。

一些图形库函数的调用，必须在一个线程中完成所有调用。