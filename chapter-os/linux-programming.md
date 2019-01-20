# Unix/Linux 环境编程

## 非本地跳转 (nonlocal jump): `setjmp` / `longjmp`

C++ / Java 中的异常可以看作是 `setjmp` / `longjmp` 的结构化版本，`setjmp` 相当于捕获异常的 `catch`，而 `longjmp` 相当于抛出异常的 `throw`，`longjmp` 指定的返回值相当于异常的类型。

+ `setjmp` 在 `env` 缓冲区中保存当前的**调用环境**，`setjmp` 自己会返回 0
+ `longjmp` 会跳转到最近一次调用的 `setjmp` 处，恢复调用环境，并让 `setjmp` 返回指定的返回值
+ 也就是说，`setjmp` 函数**只调用一次，但是返回多次**。

参考： CSAPP 8.6