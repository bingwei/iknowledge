# Python Concurrency

## Python 的并发模块

+ `multiprocessing` 多进程
+ `threading` 多线程
+ `thread` 多线程低级模块

### Threading 模块

参考 Java 的线程模型设计。

> The design of this module is loosely based on Java’s threading model. However, where Java makes locks and condition variables basic behavior of every object, they are separate objects in Python. Python’s Thread class supports a subset of the behavior of Java’s Thread class; currently, there are no priorities, no thread groups, and threads cannot be destroyed, stopped, suspended, resumed, or interrupted. The static methods of Java’s Thread class, when implemented, are mapped to module-level functions.

## Python 的多线程究竟有什么性能问题

Python 虽然支持多线程，但是由于 GIL 的限制，在任意时间只有一个 Python 解释器在解释字节码。

GIL (Global Interpreter Lock，全局解释器锁)。任何 Python 线程执行前，必须先获得 GIL 锁，然后，每执行 100 条字节码，解释器就自动释放 GIL 锁，让别的线程有机会执行。这个 GIL 全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在 Python 中只能交替执行，即使 100 个线程跑在 100 核 CPU 上，也只能用到 1 个核。

GIL 是 Python 解释器设计的历史遗留问题，通常我们用的解释器是官方实现的 CPython，要真正利用多核，除非重写一个不带 GIL 的解释器。所以，Python 多线程不利于执行 CPU 密集型的任务。