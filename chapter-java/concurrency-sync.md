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

### 重要概念

+ 条件变量 (_condition queue_, _condition variable_)
  + 条件变量是一种线程同步的原语，可以让线程一直等待 (_wait_)，直到其他线程通知 (_signal_) 它某个条件变为真
  + 条件变量实际上是一个线程队列，等待的线程进入队列，队列中的线程叫做 _wait set_
  + 互斥锁与条件变量是**一对多**的关系
+ 条件变量中的条件 (_condition predicate_, _predicate_, _assertion_)
  + 是线程执行的某个**状态**
  + 可以看作是**状态变量**的表达式
+ 条件变量的三个操作
  + **wait**
  + **signal / notify**
  + **broadcast / notifyAll**

_Effective Java_ #69 中强调，**既然正确地使用 wait / notify 比较困难，就应该用更高级的并发工具代替**。现在应该优先使用各种 synchronizer，在必要的情况下再考虑 wait / notify。

参考：

+ _Java Concurrency in Practice_, 14.2
+ [Condition variables - Wikipedia](https://en.wikipedia.org/wiki/Monitor_(synchronization)#Condition_variables)
+ _Effective Java_, #69

### `Object.wait()` / `Object.notify()`

+ 锁对象 (lock object)
  + 使用 `synchronized` 的对象
  + 可以是任何 Java 对象
+ 条件变量
  + 调用 `wait()`, `notify()`, `notifyAll()` 的对象
  + 可以是任何 Java 对象
+ Condition predicate
  + 例如：生产者-消费者模型中的 **not empty**, **not full**

调用 `wait()` 时会释放锁，并阻塞当前线程，让其他线程可以获取锁； `wait()` 返回时又会重新获取锁。

复杂的使用约束：

+ Lock object 和条件变量必须是同一个对象
  + 最大的缺点，这里未遵守一对多关系，导致不能创建多个条件变量
+ Condition predicate 相关的状态变量必须 guarded by 一个锁 (lock object)
  + 也即检查 condition predicate 时，lock object 必须已上锁
+ 调用条件变量的 `wait()`, `notify()`, `notifyAll()` 时，条件变量对象必须已上锁
  + 为了保证状态一致性
+ 必须在一个循环中单独调用 `wait()`，循环的表达式是 condition predicate
  + 也即在 `wait()` 调用的前后都检查 condition predicate
  + 当有多个 condition predicate 时，`wait()` 返回可能是其他的 condition predicate 满足了，当前 condition predicate 不一定满足
  + 即使没有 notify，`wait()` 也可能 "spuriously" 返回
+ 调用 `notify()` / `notifyAll()` 之后要尽快释放锁
  + 否则，被唤醒的线程将无法继续运行
+ 通常情况下，要使用 `notifyAll()` 而不是 `notify()`
  + `notify()` 会（随机）唤醒条件变量队列中的一个线程
  + `notifyAll()` 会唤醒条件变量队列中的所有线程
  + 当一个条件变量对应多个 condition predicate 时，必须使用 `notifyAll()` 保证目标线程被唤醒
  + 当想一次唤醒多个线程时（例如 latch 的 starting gate 例子），当然必须使用 `notifyAll()`

![Three-way relationship: Object](concurrency/three-way-relationship-object.jpg)

使用模式：

```Java
Object lock = new Object();
// Lock 需要 guard condition predicate 和 wait()
synchronized (lock) {
    // 需要在循环中调用 wait()
    while (!conditionPredicate()) {
        lock.wait(); // 需要在 lock 对象上调用 wait()
    }
    // object is now in desired state
}
```

示例代码：生产者-消费者队列

```Java
class BoundedBuffer<E> {

    private final E[] items = (E[]) new Object[10];
    private int head = 0;
    private int tail = 0;
    private int N = 0;

    private boolean isFull() { return N == items.length; }

    private boolean isEmpty() { return N == 0; }

    private void addElement(E e) {
        items[tail] = e;
        if (++tail == items.length) tail = 0;
        ++N;
    }

    private E removeElement() {
        E e = items[head];
        if (++head == items.length) head = 0;
        --N;
        return e;
    }

    // Lock == this
    // Condition variable == this
    // Condition predicates == {!isFull(), !isEmpty()}

    public synchronized void put(E e) throws InterruptedException {
        while (isFull()) {
            this.wait();
        }
        addElement(e);
        this.notifyAll();
    }

    public synchronized E take() throws InterruptedException {
        while (isEmpty()) {
            this.wait();
        }
        E e = removeElement();
        this.notifyAll();
        return e;
    }
}
```

### `java.util.concurent.locks.Condition`

`Condition` 之于 `Object.wait()` / `Object.notify()` 正如 `ReentrantLock` 之于 `synchronized`。`Condition` 的功能与 `Object.wait()` / `Object.notify()` 类似，对应关系：

+ lock object -- `Lock`
+ condition variable object -- `Condition`
+ `Object.wait()` -- `Condition.await()`
+ `Object.notify()` -- `Condition.signal()`
+ `Object.notifyAll()` -- `Condition.signalAll()`

`Condition` 相比 `Object` 的优点：

+ 显式声明 lock object 和 condition variable object
+ `Condition` 天生就是和 `Lock` 绑定的
  + 只能通过 `Lock.newCondition()` 的方式创建
  + `Object` 需要自己保证 lock object 和 condition variable object 对应
+ 一个 `Lock` 可以对应多个 `Condition`
  + 这样使得 `Condition` 可以和 condition predicate 一一对应 (best practice)
  + 这时，使用 `signal()` 就可以，不必使用 `signalAll()`
+ 可以设置 `Condition` 的公平性 (继承 `Lock` 的公平性)
+ `await()` 可以设置 interruptible 或 uninterruptible

![Three-way relationship: Condition](concurrency/three-way-relationship-condition.jpg)

使用模式：

```Java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
// Lock 需要 guard condition predicate 和 await()
lock.lock();
try {
    // 需要在循环中调用 wait()
    while (!conditionPredicate()) {
        lock.await(); // 需要在 lock 对象上调用 await()
    }
    // object is now in desired state
} finally {
    lock.unlock();
}
```

示例代码：生产者-消费者队列

```Java
class BoundedBuffer<E> {

    private final Lock lock = new ReentrantLock();
    private final Condition notFull  = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    private final E[] items = (E[]) new Object[10];
    private int head = 0;
    private int tail = 0;
    private int N = 0;

    private boolean isFull() { return N == items.length; }

    private boolean isEmpty() { return N == 0; }

    private void addElement(E e) {
        items[tail] = e;
        if (++tail == items.length) tail = 0;
        ++N;
    }

    private E removeElement() {
        E e = items[head];
        if (++head == items.length) head = 0;
        --N;
        return e;
    }

    public void put(E e) throws InterruptedException {
        lock.lock();
        try {
            while (isFull()) {
                notFull.await();
            }
            addElement(e);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public E take() throws InterruptedException {
        lock.lock();
        try {
            while (isEmpty()) {
                notEmpty.await();
            }
            E e = removeElement();
            notFull.signal();
            return e;
        } finally {
            lock.unlock();
        }
    }
}

```

## AQS 框架

`AbstractQueuedSynchronizer` (AQS) 是一个框架，用来构建 lock 和 synchronizer。

### AQS 中的概念

#### 状态

AQS 使用一个 int 类型的成员变量 state 来表示同步状态，当 state > 0 时表示已经获取了锁，当 state == 0 时表示释放了锁。

+ `getState()` —— 返回同步状态的当前值
+ `setState(int newState)` —— 设置当前同步状态
+ `compareAndSetState(int expect, int update)` —— 使用 CAS 设置当前状态，该方法能够保证状态设置的原子性

#### Acquire / release

基本操作是 acquire 和 release。有两种基本模式

+ Exclusive mode 独占式
  + 只有一个线程可以 acquire 成功
  + 如 `ReentrantLock`
  + 实现 `tryAcquire`, `tryRelease`, `isHeldExclusively`
+ Shared mode 共享式
  + 有多个线程可以 acquire 成功
  + 如 `Semaphore`
  + 实现 `tryAcquireShared`, `tryReleaseShared`

一般一个实现类只会使用其中一种模式，但也有例外，如 `ReadWriteLock`

#### CLH 队列

参考 `AbstractQueuedSynchronizer.Node` 的注释

#### Condition object

参考 `AbstractQueuedSynchronizer.ConditionObject` 的注释

### 使用 AQS 的例子

用到 AQS 框架的 JUC 类：

+ `ReentrantLock`
+ `Semaphore`
+ `ReentrantReadWriteLock`
+ `CountDownLatch`
+ `SynchronousQueue`
+ `FutureTask`

AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。

`ReentrantLock` 示例代码：

```Java
protected boolean tryAcquire(int ignored) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, 1)) {
            owner = current;
            return true;
        }
    } else if (current == owner) {
        setState(c+1);
        return true;
    }
    return false;
}
```

`Semaphore` 示例代码：

```Java
protected int tryAcquireShared(int acquires) {
    while (true) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0
                || compareAndSetState(available, remaining))
            return remaining;
    }
}

protected boolean tryReleaseShared(int releases) {
    while (true) {
        int p = getState();
        if (compareAndSetState(p, p + releases))
            return true;
    }
}
```

### 参考资料

+ _Java Concurrency in Action_, Section 14.5, 14.6
+ `AbstractQueuedSynchronizer` 源码