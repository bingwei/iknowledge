# Java Concurrency - 同步模式

_Synchronizer (同步器)_：

+ Blocking queue
  + `BlockingQueue` 及其实现类
+ Semaphore
  + `Semaphore`
+ Barrier
  + `CyclicBarrier`
+ Latch
  + `CountDownLatch`

## Latch

### `CountDownLatch` 

+ 有一个初始的 count
+ `countDown()` 方法减少 count
+ `await()` 方法：阻塞，直到 count 变为零

`CountDownLatch` 有多种用法：

+ count 为 1 时
  + 可以看作“开关”或“大门”（latch 的本意即“门闩”）
  + 可以有多个线程等待此 latch，“大门”一开都开始跑
+ count 为 N 时
  + 等待 N 个线程完成工作
  + 等待一个工作完成 N 次

```Java
CountDownLatch startGate = new CountDownLatch(1);
CountDownLatch endGate = new CountDownLatch(N);

for (int i = 0; i < N; i++) {
    new Thread(() -> {
        try {
            startGate.await();
            try {
                work();
            } finally {
                endGate.countDown();
            }
        } catch (InterruptedException ignored) {
        }
    }).start();
}
System.out.println(System.nanoTime());
startGate.countDown(); // 同时开始执行所有线程
endGate.await(); // 等待所有线程结束
System.out.println(System.nanoTime());
```

### 其他 latch

`Future` (实现类 `FutureTask`) 也能实现和 latch 类似的效果。例如，等待某个资源初始化之后再进行计算：

```Java
public class ComputeWorker {

    private final FutureTask<Resource> futureTask = new FutureTask<>(() -> {
        return loadResource();
    });

    public void initialize(){
        new Thread(futureTask).start();
    }

    public ComputingResult compute() throws InterruptedException {
        try {
            Resource resource = futureTask.get();
            return compute(resource);
        } catch (ExecutionException e) {
            // ...
        }
    }
}
```

## Semaphore (信号量)

(计数)信号量通常用来控制可以访问某个资源的线程个数。

+ 有一个初始的 count (_permit_ 的数量)
+ `acquire` 获取 permit
  + 如果 permit 已用完，阻塞，直到有可用的 permit
+ `release` 释放 permit

只有一个 permit 的信号量（叫做 _binary semaphore_）相当于一个**互斥锁**。这个锁：不可重入；可以传给另外的线程，让另外的线程释放。

`Semaphore` 可以设置为公平的或不公平的。

## Barrier

设置一个 common barrier point，让所有的线程彼此等待。`CyclicBarrier` 可以重复使用。

Barrier 类似 latch。