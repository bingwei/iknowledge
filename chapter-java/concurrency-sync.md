# Java Concurrency - 同步机制

+ 现有的 _Synchronizer (同步器)_
  + Blocking queue
    + `BlockingQueue` 及其实现类
  + Semaphore
    + `Semaphore`
  + Latch / Barrier / Phaser
    + `CountDownLatch`
    + `CyclicBarrier`
    + `Phaser`
+ 搭建自己的同步器
  + Condition queue: wait / notify 原语
  + `Condition`
  + AQS 框架

## 现成的 Synchronizer

### Blocking queue

Blocking queue 适合用作数据共享的通道，如典型的生产者-消费者模型：

```Java
class ProducerConsumer {

    public static void main(String[] args) {
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);
        new Thread(producer).start();
        new Thread(consumer).start();
    }

}

class Producer implements Runnable {

    private final BlockingQueue<String> queue;

    public Producer(BlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 100; i++) {
                TimeUnit.MILLISECONDS.sleep(500);
                String product = "Product-" + i;
                System.out.println("Produce: " + product);
                queue.put(product);
            }
            queue.put("exit");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

class Consumer implements Runnable {

    private final BlockingQueue<String> queue;

    public Consumer(BlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            while (true) {
                String product = queue.take();
                if (product.equals("exit")) {
                    return;
                }
                TimeUnit.SECONDS.sleep(1);
                System.out.println("Consume: " + product);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Semaphore (信号量): `Semaphore`

(计数)信号量通常用来控制可以访问某个资源的线程个数。

+ 有一个初始的 count (_permit_ 的数量)
+ `acquire` 获取 permit
  + 如果 permit 已用完，阻塞，直到有可用的 permit
+ `release` 释放 permit

只有一个 permit 的信号量（叫做 _binary semaphore_）相当于一个**互斥锁**。这个锁：不可重入；可以传给另外的线程，让另外的线程释放。

`Semaphore` 可以设置为公平的或不公平的。

### Latch: `CountDownLatch`

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

### Barrier: `CyclicBarrier`

Barrier 本意为“栅栏”，它设置一个“阻拦点”，让所有的线程彼此等待。`CyclicBarrier` 可以重复使用 (cyclic 的意思)。

Barrier 类似 latch，主要的不同点在于：

+ `CountDownLatch` 用于让一个线程等待其他的线程，而 `CyclicBarrier` 用于让线程**互相等待**
+ `CountDownLatch` 只能使用一次，而 `CyclicBarrier` 可以重复使用

### Phaser: `Phaser`

`Phaser` 的功能涵盖了 `CountDownLatch` 和 `CyclicBarrier`，而且更加强大和灵活。

+ 注册
  + Phaser 中参与的线程 (party) 的数量是可变的
  + `register()`
  + `bulkRegister()`
  + `arriveAndDeregister()`
+ 同步
  + `arriveAndAwaitAdvance()` 类似 `CyclicBarrier.await()`
  + `awaitAdvance()` 类似 `CountDownLatch.await()`
  + `arrive()` 和 `arriveAndDeregister()` 不会阻塞
+ 终止
  + 重写 `onArrive()`，返回值可以控制是否终止
  + `forceTermination()` 可强制 phaser 终止
+ 层叠 (tiering)
  + 可以将 phaser 组织成一个树形结构
  + `new Phaser(phaser)`
+ 监控
  + `getRegisteredParties()`
  + `getArrivedParties()`
  + `getUnarrivedParties()`

## 自定义 Synchronizer

### Condition queue (wait / notify)

任何一个 Java 对象都可以作为 lock object (放在 `synchronized` 关键字后)，也都可以作为 condition queue object (调用 `wait`, `notify`, `notifyAll`)。 调用 `wait()` 时会释放锁，而 `wait()` 返回时又会重新获取锁。

+ Condition predicate 相关的状态变量必须 guarded by 一个锁 (lock object)
+ lock object 和 condition queue object 必须是同一个对象
  + **In order to call any of the condition queue methods on object X, you must hold the lock on X**.
+ 必须在上锁状态中调用 `wait()`, `notify()`, `notifyAll()`
+ 必须在一个循环中调用 `wait()`，循环的表达式是 condition predicate
  + 也即在 `wait()` 调用的前后都检查 condition predicate
  + `wait()` 返回仅仅代表有线程调用了 `notifyAll()`，此时 condition predicate 不一定满足
+ 通常情况下，要使用 `notifyAll()`

_Effective Java_ #69 中强调，**既然正确地使用 wait / notify 比较困难，就应该用更高级的并发工具代替**。现在应该优先使用各种 synchronizer，在非常必要的情况下再考虑 wait / notify。

参考：

+ _Java Concurrency in Practice_, 14.2
+ _Effective Java_, #69

### `java.util.concurent.locks.Condition`

`Condition` 之于 wait / notify 正如 `ReentrantLock` 之于 `synchronized`。`Condition` 的功能与 wait / notify 类似，对应关系：

+ lock object -- `Lock`
+ condition queue object -- `Condition`
+ `Object.wait()` -- `Condition.await()`
+ `Object.notify()` -- `Condition.signal()`
+ `Object.notifyAll()` -- `Condition.signalAll()`

`Condition` 相比 condition queue 的优点：

+ `Lock.newCondition()` 的创建方式保证了 `Lock` 和 `Condition` 的对应关系
  + 原先，需要自己保证 lock object 和 condition queue object 对应
+ 一个 `Lock` 可以对应多个 `Condition`
  + 原先，只能有一个对应的 condition queue object，然后设置多个 condition predicate
  + 多个 `Condition` 可以将 condition predicate 分离
  + 更多情况下，可以使用 `signal()`，不必使用 `signalAll()`
+ 可以设置 `Condition` 的公平性 (继承 `Lock` 的公平性)
+ 可以控制 `Condition` 的可见性
+ `await()` 可以设置 interruptible 或 uninterruptible
+ Deadline-based waiting

## AQS 框架

`AbstractQueuedSynchronizer` (AQS) 是一个框架，用来构建 lock 和 synchronizer。