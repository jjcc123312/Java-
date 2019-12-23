# Java并发

参考资料：

- http://www.iocoder.cn/JUC/good-collection/
- https://segmentfault.com/blog/ressmix_multithread?page=2
- https://www.jianshu.com/p/79d9baaa396e

# 一、Java中的锁

 在 `java.util.concurrent.locks` 包下，实现了 Java Lock 相关的工具类。整体类图如下： 

 <img src=".\img\locks-01.png" alt="类图" style="zoom:150%;" /> 

## 1、AQS简介

### 1.1、简介

AQS，`AbstractQueuedSynchronizer`，即**队列同步器**。它是构建锁或者其它同步组件的基础架构（如ReentrantLock，ReentrantReadWriteLock， Semaphore 等）。

它是 J.U.C 并发包中的核心基础组件。 

### 1.2、同步状态

AQS的主要使用方式是继承，子类通过继承同步器，并实现他的抽象方法来管理同步状态。

**AQS使用一个`int`类型的成员变量`state`来表示同步状态。**

- 当`state > 0`时，表示已获取到了锁；
- 当`state = 0`时，表示释放了锁；

它提供了三个方法，来对同步状态 `state` 进行操作，并且 AQS 可以确保对 `state` 的操作是**安全**的： 

- `#getState()`
- `#setState(int newState)`
- `#compareAndSetState(int expect, int update)`

### 1.3、同步队列

AQS通过内置的`FIFO`同步队列来完成资源获取线程的排队工作；

- 如果当前线程获得同步状态（锁）失败时，AQS则**会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列**，同时会阻塞当前线程。
- 当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态（锁）。

## 2、Lock

> **公平锁/非公平锁**

**公平锁是指多线程按照申请锁的顺序来获取锁，非公平锁指多个线程获取锁的顺序不是按照申请锁的顺序**，有可能造成优先级反转或者饥饿现象。

**非公平锁的优点在于比公平锁吞吐量大**，ReentrantLock默认非公平锁，可以通过构造函数选择公平锁，Synchronized是非公平锁。

**公平与非公平获取锁的区别** ：

公平性与否是针对获取锁而言的，如果一个锁是公平的，那么**锁的获取顺序就应该符合请求的绝对时间顺序**，也就是 **FIFO**。 

对于上非公平锁实现的`nonfairTryAcquire(int acquires)`，只要 **CAS** 设置同步状态成功，即获取到锁，而公平锁则不同，如下： 

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

相比非公平锁的实现，公平锁的实现在获取锁的时候多了一个`!hasQueuedPredecessors()`判断： 

即加入了 **同步队列中当前节点是否有前驱节点的判断** ，如果该方法返回 `true`，则表示有线程比当前线程更早地请求获取锁，因此 **需要等待前驱线程获取并释放锁之后才能继续获取锁**。 

> **可重入锁**

ReentrantLock与synchronized都是可重入的。

就是支持重进入的锁，它表示该锁能够支持 **一个线程对资源的重复加锁**。除此之外，该锁的还支持获取锁时的公平和非公平性选择。 

重进入是指**任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞**。

现在有方法 m1 和 m2，两个方法均使用了同一把锁对方法进行同步控制，同时方法 m1 会调用 m2。**线程 t 进入方法 m1 成功获得了锁，此时线程 t 要在没有释放锁的情况下，调用 m2 方法。由于 m1 和 m2 使用的是同一把可重入锁，所以线程 t 可以进入方法 m2，并再次获得锁，而不会被阻塞住**。示例代码大致如下： 

```java
void m1() {
    lock.lock();
    try {
        // 调用 m2，因为可重入，所以并不会被阻塞
        m2();
    } finally {
        lock.unlock()
    }
}

void m2() {
    lock.lock();
    try {
        // do something
    } finally {
        lock.unlock()
    }
}
```

假如 lock 是不可重入锁，那么上面的示例代码必然会引起死锁情况的发生。这里请大家思考一个问题，ReentrantLock 的可重入特性是怎样实现的呢？简单说一下，ReentrantLock 内部是通过 AQS 实现同步控制的，AQS 有一个变量 `state` 用于记录同步状态。初始情况下，state = 0，表示 ReentrantLock 目前处于解锁状态。如果有线程调用 lock 方法进行加锁，state 就由0变为1，如果该线程再次调用 lock 方法加锁，就让其自增，即 state++。线程每调用一次 unlock 方法释放锁，会让 state--。通过查询 state 的数值，即可知道 ReentrantLock 被重入的次数了。这就是可重复特性的大致实现流程。 

> **独享锁/共享锁**

**独享锁是指一个锁只能一个线程独有，共享锁指一个锁可被多个线程共享**； JUC中`ReentrantLock`与`CyclicBarrier`为独占锁，`CountDownLatch`与`Semaphore`为共享锁，ReentrantReadWriteLock中`writeLock`为独占锁，`ReadLock`为共享锁。 

独占锁与共享锁的区别：

- 独占功能
  当锁被头节点获取后，只有头节点获取锁，其余节点的线程继续沉睡，等待锁被释放后，才会唤醒下一个节点的线程。
- 共享功能
  只要头节点获取锁成功，就在唤醒自身节点对应的线程的同时，继续唤醒AQS队列中的下一个节点的线程，每个节点在唤醒自身的同时还会唤醒下一个节点对应的线程，以实现共享状态的“向后传播”，从而实现共享功能。

> **互斥锁/读写锁**

独享锁，共享锁是一种广义的说法，互斥锁/读写锁是其具体实现。

> **乐观锁/悲观锁**

 乐观锁与悲观锁是看待同步的角度不同，**乐观锁认为对于同一个数据的修改操作，是不会有竞争的，会尝试更新，如果失败，不断重试。悲观锁与此相反，直接获取锁，之后再操作，最后释放锁**。 

> **分段锁**

 分段锁是一种设计思想，通**过将一个整体分割成小块，在每个小块上加锁，提高并发**。 

> **偏向锁/轻量级锁/重量级锁**

这三种为锁的状态，是JVM对于Synchronized的优化。

**偏向锁是指同一段代码一直被同一个线程访问，那么这个线程会自动获取锁**，降低获取锁的代价。

**轻量级锁指在偏向锁的时候，被另一个线程访问，通过CAS尝试获取锁**。

重量级锁就是**在轻量级锁的前提下，其中一个线程自旋次数过多，但是锁还没获取到，就膨胀为重量级锁**。 

> 自旋锁

**典型的CAS思想，不会立即阻塞，会不断重试获取锁**，减少了线程上下文切换，但是增加了CPU消耗。 

### 2.1、ReentrantLock

#### 2.1.1、简介

ReentrantLock，可重入锁，**是一种递归无阻塞的同步机制**。它等同于`Synchronized`的使用，但是ReentrantLock提供了比`synchronized`更强大的锁机制，可以减少死锁发生的概率。

> 虽然它缺少了（通过`synchronized`块或者方法所提供的）隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的 **可操作性**、**可中断的获取锁** 以及 **超时获取锁** 等多种`synchronized`关键字所不具备的同步特性。
>
> 使用`synchronized`关键字将会 **隐式** 地获取锁，但是它将锁的获取和释放固化了，也就是先获取再释放。 

ReentrantLock 还提供了**公平锁**和**非公平锁**的选择，**通过构造方法接受一个可选的 `fair` 参数（默认非公平锁）：当设置为 true 时，表示公平锁；否则为非公平锁**。 

**公平锁与非公平锁的区别在于，公平锁的锁获取是有顺序的。但是公平锁的效率往往没有非公平锁的效率高，在许多线程访问的情况下，公平锁表现出较低的吞吐量。** 

 ReentrantLock 整体结构如下图： 

 ![201702010001](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\201812081321001-1574580030822-1574581143927.png) 

- ReentrantLock 实现 Lock 接口，基于内部的 Sync 实现。
- Sync 实现 AQS ，提供了 FairSync 和 NonFairSync 两种实现。

#### 2.1.2、构造声明

 ReentrantLock类提供了两类构造器：  ![ReentrantLock构造声明](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\1460000015804923) 

**通过构造方法接受一个可选的 `fair` 参数（默认非公平锁）：当设置为 true 时，表示公平锁；否则为非公平锁**。 默认为**非公平策略**。 

> **公平策略：**在多个线程争用锁的情况下，**公平策略倾向于将访问权授予等待时间最长的线程**。也就是说，相当于有一个线程等待队列，**先进入等待队列的线程后续会先获得锁**，这样按照“先来后到”的原则，对于每一个等待线程都是公平的。

> **非公平策略：**在多个线程争用锁的情况下，能够最终获得锁的线程是随机的（由底层OS调度）。

**使用方式**

```csharp
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...
    public void m() {
        lock.lock(); // block until condition holds
        try {
            // ... method body
        } finally {
            lock.unlock();
        }
    }
}
```

注意 ： 

> 1.在`finally`块中释放锁，目的是保证在获取到锁之后，最终能够被释放。
>
> 2.不要将获取锁的过程写在`try`块中，因为如果在获取锁（自定义锁的实现）时发生了异常，异常抛出的同时，也会导致锁无故释放。 

#### 2.1.3、 `Lock`接口提供的`synchronized`关键字所不具备的主要特性 

-  **尝试非阻塞性获取锁**： 当前线程尝试获取锁，如果此时没有其他线程占用此锁，则成功获取到锁。
  -  **在非公平的状态下，新来的线程在入队之前会尝试抢一次锁**，如果失败了就会乖乖进入队列，一旦进入队列是不能再次出来抢的，只能等待队列一个一个地执行完毕。**所谓不公平是指新来的线程会不会在入队之前尝试「野蛮」地抢锁**，公平的时候是不会，但是非公平的时候是会的。 
-  **能被中断的获取锁**： 当获取到锁的线程被中断时，中断异常会抛出并且会释放锁。
-  **超时获取锁**： 在指定时间内获取锁，如果超过时间还没获取，则返回false。

#### 2.1.4、`Lock` 相关的API 

 `java.util.concurrent.locks.Lock` 接口，定义方法如下： 

```csharp
void lock();
void lockInterruptibly() throws InterruptedException;boolean tryLock();
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
void unlock();Condition newCondition();`
```

`void lock();`：获取锁，获取之后返回

`void lockInterruptibly() throws InterruptedException;`：可中断的获取锁，和`lock()`方法不同之处在于该方法会响应中断，即在锁的获取中可以中断当前线程。

`boolean tryLock();`：尝试非阻塞的获取锁，调用该方法后立即返回，如果能够获取则返回true，否则返回flase。

`boolean tryLock(long time, TimeUnit unit) throws InterruptedException;`： 超时获取锁。 超时时间结束，未获得锁，返回`false`。**当前线程在以下3种情况下会返回：①当前线程在超时时间内获得了锁②当前线程在超时时间内被中断③超时时间结束，返回flase;**

`void unlock();`：释放锁

 `Condition newCondition();`：获取等待通知组件，改组件和锁绑定，当前线程获取到锁才能调用`wait()`方法，调用之后则会释放锁。  

### 2.2、ReentrantReadWriteLock

#### 2.2.1、简介

重入锁`ReentrantLock`是排他锁，**排他锁在同一时刻仅有一个线程可以访问**，但是在大多数场景下，**大部分时间都是提供读服务，而写服务占有的时间较少**。然而，读服务不存在数据竞争问题，如果一个线程在读时禁止其它线程读势必会导致性能降低。所以就提供了读写锁。**读写锁** 能够提供比 **排它锁** 更好的 **并发性** 和 **吞吐量**。 

读写锁维护着**一对**锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提高；

- 在同一时间，可以允许多个读线程同时访问。
- 但是，**在写线程时，所有读线程和写线程都会被阻塞**。

读写锁的**主要特性**： 

-  **公平性选择** ：支持公平和非公平的方式获取锁，吞吐量非公平优于公平。
-  **重进入** ： **读锁在获取锁之后可以再获取读锁；写锁在获取锁之后可以再获取读锁和写锁**。 读写锁最多支持 65535 个递归写入锁和 65535 个递归读取锁。 
-  **锁降级** ：先获取**写锁**，然后获取**读锁**，最后释放**写锁**，这样写锁就降级成了**读锁**。但是，读锁不能升级到写锁。简言之，就是：**写锁可以降级成读锁，读锁不能升级成写锁。** 

#### 2.2.2、ReadWriteLock

`java.util.concurrent.locks.ReadWriteLock` ，读写锁接口。定义方法如下：

```java
Lock readLock();

Lock writeLock();
```

`ReadWriteLock`仅定义了获取读锁和写锁的两个方法，即`readLock()`方法和`writeLock()`方法，而其实现类`ReentrantReadWriteLock`，除了接口方法之外，还提供了一些便于外界监控其内部工作状态的方法，

#### 2.2.3、ReentrantReadWriteLock

`java.util.concurrent.locks.ReentrantReadWriteLock` ，实现 ReadWriteLock 接口，**可重入**的**读写锁**实现类。在它内部，维护了**一对**相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 Writer 线程，读取锁可以由多个 Reader 线程同时保持。也就说说，**写锁是独占的，读锁是共享的**。 

相关API：

-  `getReadLockCount()`：返回当前读锁获取的次数
-  `getReadHoldCount()`：返回当前线程获取读锁的次数
-  `isWriteLocked()`：判断写锁是否被获取
-  `getWriteHoldCount()`：返回当前写锁被获取的次数

**内部类：**

**Sync**

ReentrantReadWriteLock的同步实现，**继承自AbstractQueuedSynchronizer，内部重写了tryRelease，tryAcquire，tryReleaseShared，tryAcquireShared，属于核心。**

**FairSync和NonfairSync**

公平模式与非公平模式同步器，重写了Sync类中的writerShouldBlock和readerShouldBlock方法，用于处理当前线程在获取读锁和写锁阻塞时的策略。

**ReadLock**

读锁的实现，重写了接口Lock中共享锁的一些方法，**内部维护的同步器与写锁是同一个同步器。不支持Condition条件队列。**

**WriteLock**

写锁也是实现了Lock接口，但是实现了独占锁的一些方法，**内部维护的同步器和读锁的是同一个，支持Condition条件队列。**

#### 2.2.4、使用示例

```java
public class RWHashMap {

    private static Map<String, Object> map = new HashMap<>(8);

    private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private static Lock readLock = readWriteLock.readLock();

    private static Lock writeLock = readWriteLock.writeLock();

    /**
     * 获取一个key对应的value
     * @title get
     * @author Jjcc
     * @param key key值
     * @return java.lang.Object
     * @createTime 2019/11/23 11:08
     */
    public static Object get(String key) {
        readLock.lock();
        try {
            return map.get(key);
        } finally {
            readLock.unlock();
        }
    }

    /**
     * 设置key对应的value，并返回旧的value
     * @title put
     * @author Jjcc
     * @param key key值
     * @param value value值
     * @return java.lang.Object
     * @createTime 2019/11/23 11:10
     */
    public static Object put(String key, Object value) {
        writeLock.lock();
        try {
            //这里睡眠4秒，用以验证写锁被占有，读锁无法使用；
            Thread.sleep(4000);
            return map.put(key, value);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            writeLock.unlock();
        }
        return null;
    }

    /**
     * 清空map所有内容
     * @title clear
     * @author Jjcc
     * @return void
     * @createTime 2019/11/23 11:12
     */
    public static void clear() {
        writeLock.lock();
        try {
            map.clear();
        } finally {
            writeLock.unlock();
        }
    }

}
```

```java
public static void main(String[] args){

        new Thread(() ->{
            RWHashMap.put("k1", "value1");
        }).start();

        new Thread(() -> {
            Object k1 = RWHashMap.get("k1");

            System.out.println("thread2: " + k1);
        }).start();

        System.out.println("asdas: " + RWHashMap.get("k1"));
    }
------->console：
thread2: value1
asdas: value1
```

#### 2.2.5、读写锁的实现分析

-  **读写状态的设计** 

  读写锁将变量切分成了两个部分，高16位表示读，低16位表示写，如下图：![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\asdasd123123123121ff-1574581143928)

  当前同步状态表示一个线程已经获取了写锁，且重进入了两次，同时也连续获取了两次读锁。
  读写锁是如何迅速确定读和写各自的状态呢？答案是通过位运算。 

  假设当前同步状态值为`S`，写状态等于`S&0x0000FFFF`（将高16位全部抹去），读状态等于`S>>16`（无符号补0右移16位）。  

  当写状态增加`1`时，等于`S+1`，当读状态增加`1`时，等于`S+(1<<16)`，也就是`S+0x00010000`。
  根据状态的划分能得出一个推论：`S != 0`时，当写状态`S&0x0000FFFF = 0`时，则读状态`S>>16 > 0`，即读锁已被获取。

-  **写锁的获取与释放** 

  写锁是一个支持重进入的排它锁。如果当前线程已经获取了写锁，则增加写状态。**如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程（无法重入），则当前线程进入等待状态。** 

  `ReentrantReadWriteLoc`的`tryAcquire`方法如下： 

  ```java
  protected final boolean tryAcquire(int acquires) {
      Thread current = Thread.currentThread();
      //当前锁个数
      int c = getState();
      //写锁
      int w = exclusiveCount(c);
      if (c != 0) {
          //c != 0 && w == 0 表示存在读锁
          //当前线程不是已经获取写锁的线程
          if (w == 0 || current != getExclusiveOwnerThread())
              return false;
          //超出最大范围
          if (w + exclusiveCount(acquires) > MAX_COUNT)
              throw new Error("Maximum lock count exceeded");
          setState(c + acquires);
          return true;
      }
      // 是否需要阻塞
      if (writerShouldBlock() ||
              !compareAndSetState(c, c + acquires))
          return false;
      //设置获取锁的线程为当前线程
      setExclusiveOwnerThread(current);
      return true;
  }
  ```

  该方法除了重入条件（当前线程为获取了写锁的线程）之外，增加了一个 **读锁是否存在** 的判断。**如果存在读锁，则写锁不能被获取。**调用 `#writerShouldBlock()` **抽象**方法，若返回 true ，则获取写锁**失败**。 

  写锁的释放与`ReentrantLock`的释放过程基本类似，**每次释放均减少写状态，当写状态为`0`时表示写锁已被释放**，从而等待的读写线程能够继续访问读写锁，同时前次写线程的修改对后续读写线程可见。 

- **读锁的获取与释放** 

  读锁是一个支持重进入的 **共享锁**，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为`0`）时，读锁总会被成功地获取，而所做的也只是（线程安全的）增加读状态。 

  **如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。** 

  `ReentrantReadWriteLock`的`tryAcquireShared`方法:

  ```java
  protected final int tryAcquireShared(int unused) {
      //当前线程
      Thread current = Thread.currentThread();
      int c = getState();
      //exclusiveCount(c)计算写锁
      //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
      //存在锁降级问题，后续阐述
      if (exclusiveCount(c) != 0 &&
              getExclusiveOwnerThread() != current)
          return -1;
      //读锁
      int r = sharedCount(c);
  
      /*
       * readerShouldBlock():读锁是否需要等待（公平锁原则）
       * r < MAX_COUNT：持有线程小于最大数（65535）
       * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
       */
      if (!readerShouldBlock() &&
              r < MAX_COUNT &&
              compareAndSetState(c, c + SHARED_UNIT)) { //修改高16位的状态，所以要加上2^16
          /*
           * holdCount部分后面讲解
           */
          if (r == 0) {
              firstReader = current;
              firstReaderHoldCount = 1;
          } else if (firstReader == current) {
              firstReaderHoldCount++;
          } else {
              HoldCounter rh = cachedHoldCounter;
              if (rh == null || rh.tid != getThreadId(current))
                  cachedHoldCounter = rh = readHolds.get();
              else if (rh.count == 0)
                  readHolds.set(rh);
              rh.count++;
          }
          return 1;
      }
      return fullTryAcquireShared(current);
  }
  ```

  **如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态**。
  **如果当前线程获取了写锁或者写锁未被获取，则当前线程增加读状态，成功获取读锁**。

  读锁的每次释放均减少读状态，减少的值是`1<<16`。

- **降级锁**

   锁降级指的是 **写锁降级成为读锁**。 

  如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁**降级是指把持住（当前拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁的过程。**

  在获取读锁的方法 `#tryAcquireShared(int unused)` 中，有一段代码就是来判断读锁降级的：  

  ```java
  `int c = getState();
  //exclusiveCount(c)计算写锁
//如果存在写锁，且锁的持有者不是当前线程，直接返回-1
  //存在锁降级问题，后续阐述
if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)    
      return -1;		//读锁int r = sharedCount(c);`
  ```
  
  以下是是锁降级的示例： 
  
  ```java
  //当数据发生变更后，update变量（布尔类型且volatile修饰）被设置为false
  public void processData() {
      readLock.lock();
      if (!update) {
          // 必须先释放读锁
          readLock.unlock();
          // 锁降级从写锁获取到开始
          writeLock.lock();
          try {
              if (!update) {
                  // 准备数据的流程（略）
                  update = true;
              }
              readLock.lock();
          } finally {
              writeLock.unlock();
          }
          // 锁降级完成，写锁降级为读锁
      }
      try {
          // 使用数据的流程（略）
    } finally {
          readLock.unlock();
    }
  }
  ```
```
  
  上述示例中，当数据发生变更后，`布尔`类型且`volatile`修饰`update`变量被设置为`false`，此时所有访问`processData()`方法的线程都能够感知到变化，但只有一个线程能够获取到写锁，其他线程会被阻塞在读锁和写锁的`lock()`方法上。**当前线程获取写锁完成数据准备之后，再获取读锁，随后释放写锁，完成锁降级。**
  
**锁降级中读锁的获取是否必要呢？** 
  
  答案是必要的。主要是为了 **保证数据的可见性**。
  如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程（记作`线程T`）获取了写锁并修改了数据，那么 **当前线程无法感知`线程T`的数据更新**。
  如果当前线程获取读锁，即遵循锁降级的步骤，则`线程T`将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁进行数据更新。
  
  `RentrantReadWriteLock`不支持锁升级。目的也是保证数据可见性，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见（**修改与获取数据不能同时发生**）的。 

### 2.3、LockSupport工具

#### 2.3.1、简介

LockSupport类，是JUC包中的一个工具类，是用来创建锁和其他同步类的基本线程阻塞原语。 

当需要阻塞或唤醒一个线程的时候，都会使用`LockSupport`工具类来完成相应工作。

LockSupport类的核心方法其实就两个：`park()`和`unark()`，其中**`park()`方法用来阻塞当前调用线程，`unpark()`方法用于唤醒指定线程。**
这其实和Object类的wait()和signial()方法有些类似，但是**LockSupport的这两种方法从语意上讲比Object类的方法更清晰，而且可以针对指定线程进行阻塞和唤醒。** 

> LockSupport类使用了一种名为Permit（许可）的概念来做到阻塞和唤醒线程的功能，可以把许可看成是一种(0,1)信号量（Semaphore），但与 Semaphore 不同的是，许可的累加上限是1。
> **初始时，permit为0，当调用`unpark()`方法时，线程的permit加1，当调用`park()`方法时，如果permit为0，则调用线程进入阻塞状态。** 

#### 2.3.2、相关API

`park()`：阻塞当前线程，只有调用 `unpark(Thread thread)`或者被中断之后才能从`park()`返回。

`parkNanos(long nanos)`：再`park()`的基础上增加了超时返回。

`parkUntil(long deadline)`：阻塞线程直到 `deadline` 对应的时间点。

`park(Object blocker)`：`Java 6`时增加，`blocker`为当前线程在等待的对象。

`parkNanos(Object blocker, long nanos)`：`Java 6`时增加，`blocker`为当前线程在等待的对象。

`parkUntil(Object blocker, long deadline)`：`Java 6`时增加，`blocker`为当前线程在等待的对象。

`unpark(Thread thread)`：唤醒处于阻塞状态的线程 `thread`。

**park(Object blocker)方法的blocker参数，主要是用来标识当前线程在等待的对象，该对象主要用于问题排查和系统监控**。 park方法和unpark(Thread thread)都是成对出现的，**同时unpark必须要在park执行之后执行**，当然并不是说没有不调用unpark线程就会一直阻塞，park有一个方法，它带了时间戳（parkNanos(long nanos)：为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用）。 

#### 2.3.3、使用示例

*假设现在需要实现一种FIFO类型的独占锁，可以把这种锁看成是ReentrantLock的公平锁简单版本，且是不可重入的，就是说当一个线程获得锁后，其它等待线程以FIFO的调度方式等待获取锁。* 

​```java
public class FIFOMutex {
    private final AtomicBoolean locked = new AtomicBoolean(false);
    private final Queue<Thread> waiters = new ConcurrentLinkedQueue<Thread>();
 
    public void lock() {
        Thread current = Thread.currentThread();
        //需要将等待线程放入线程等待队列中
        waiters.add(current);
 
        // 如果当前线程不在队首，或锁已被占用，则当前线程阻塞
        // NOTE：这个判断的意图其实就是：锁必须由队首元素拿到
        while (waiters.peek() != current || !locked.compareAndSet(false, true)) {
            LockSupport.park(this);
        }
        waiters.remove(); // 删除队首元素
    }
 
    public void unlock() {
        locked.set(false);
        LockSupport.unpark(waiters.peek());
    }
}
```

测试用例： 

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        FIFOMutex mutex = new FIFOMutex();
        MyThread a1 = new MyThread("a1", mutex);
        MyThread a2 = new MyThread("a2", mutex);
        MyThread a3 = new MyThread("a3", mutex);
 
        a1.start();
        a2.start();
        a3.start();
 
        a1.join();
        a2.join();
        a3.join();
 
        assert MyThread.count == 300;
        System.out.print("Finished");
    }
}
 
class MyThread extends Thread {
    private String name;
    private FIFOMutex mutex;
    public static int count;
 
    public MyThread(String name, FIFOMutex mutex) {
        this.name = name;
        this.mutex = mutex;
    }
 
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            mutex.lock();
            count++;
            System.out.println("name:" + name + "  count:" + count);
            mutex.unlock();
        }
    }
}
```

上述FIFOMutex 类的实现中，当判断锁已被占用时，会调用`LockSupport.park(this)`方法，将当前调用线程阻塞；当使用完锁时，会调用`LockSupport.unpark(waiters.peek())`方法将等待队列中的队首线程唤醒。 

通过LockSupport的这两个方法，可以很方便的阻塞和唤醒线程。但是LockSupport的使用过程中还需要注意以下几点： 

1. **`park`方法的调用一般要方法一个循环判断体里面。**

   如上述示例中的：

   ```java
   while (waiters.peek() != current || !locked.compareAndSet(false, true)) {
       LockSupport.park(this);
   }
   ```

   之所以这样做，**是为了防止线程被唤醒后，不进行判断而意外继续向下执行**，这其实是一种[Guarded Suspension](https://segmentfault.com/a/1190000015558585)的多线程设计模式。

2. `park`方法是会响应中断的，但是不会抛出异常。(也就是说如果当前调用线程被中断，则会立即返回但不会抛出中断异常)

3. park的重载方法`park(Object blocker)`，会传入一个blocker对象，所谓Blocker对象，其实就是当前线程调用时所在调用对象（如上述示例中的FIFOMutex对象）。该对象一般供监视、诊断工具确定线程受阻塞的原因时使用。

### 2.4、Condition

#### 2.4.1、简介

任意一个java对象，都拥有一组监视器方法，定义在 `java.lang.Object` ，主要包括`wait()`、`wait(long timeout)`、`notify()`以及`notifyAll()`方法 。这些方法与`synchronized`同步关键字配合，可以实现**等待/通知模式**。

`Condition`接口也提供了类似`Object`的监视器方法，与`Lock`配合可以实现 **等待/通知** 模式，但是这两者在使用方式以及功能特性上还是有差别的。 

以下是Object的监视器方法与Condition接口的对比：

|                        对比项                        |     Object      |                          Condition                           |
| :--------------------------------------------------: | :-------------: | :----------------------------------------------------------: |
|                       前置条件                       |  获取对象的锁   | 调用`Lock.lock()`获取锁；调用`Lock.newCondition()`获取condition对象 |
|                       调用方式                       | `object.wait()` |                     `condition.await()`                      |
|                     等待队列个数                     |      一个       |                             多个                             |
|             当前线程释放锁并进入等待状态             |      支持       |                             支持                             |
| 当前线程释放锁并进入等待状态，在等待状态中不响应中断 |     不支持      |                             支持                             |
|           当前线程释放锁并进入超时等待状态           |      支持       |                             支持                             |
|       当前线程释放锁并进入等待状态到将来某时间       |     不支持      |                             支持                             |
|                唤醒等待队列的一个线程                |      支持       |                             支持                             |
|                唤醒等待队列的全部线程                |      支持       |                             支持                             |

#### 2.4.2、Condition接口与示例

`Condition`定义了等待/通知两种类型的方法，当前线程调用这些方法时，需要提前获取到`Condition`对象关联的锁。
`Condition`对象是由调用`Lock`对象的`newCondition()`方法创建出来的，换句话说，`Condition`是依赖`Lock`对象的。

`Condition`的使用方式比较简单，需要注意在调用方法前获取锁，如下：

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
public void conditionWait() throws InterruptedException {
    lock.lock();
    try {
        condition.await();
    } finally {
        lock.unlock();
    }
}
public void conditionSignal() throws InterruptedException {
    lock.lock();
    try {
        condition.signal();
    } finally {
        lock.unlock();
    }
}
```

`Condition` 接口方法介绍：

-  `void await() throws InterruptedException` ： 当前线程进入等待状态直到被通知或中断
-  `void awaitUninterruptibly()` ：当前线程进入等待状态直到被通知，对**中断不敏感**
-  `long awaitNanos(long var1) throws InterruptedException` ：当前线程进入等待状态直到被通知、中断或超时
-  `boolean await(long var1, TimeUnit var3) throws InterruptedException` ：当前线程进入等待状态直到被通知、中断或超时
-  `boolean awaitUntil(Date var1) throws InterruptedException` ：当前线程进入等待状态直到被通知、中断或到某一时间
-  `void signal()` ：唤醒`Condition`上一个在等待的线程
-  `void signalAll()` ：唤醒`Condition`上全部在等待的线程

**获取一个Condition必须通过Lock的newCondition()方法。**

通过下面这个有界队列的示例我们来深入了解下 `Condition` 的使用方式:

```java
public class BoundedQueue<T> {
    private Object[] items;
    // 添加的下标，删除的下标和数组当前数量
    private int addIndex, removeIndex, count;
    private Lock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();

    public BoundedQueue(int size) {
        items = new Object[size];
    }

    // 添加一个元素，如果数组满，则添加线程进入等待状态，直到有"空位"
    public void add(T t) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length){
                notFull.await();
            }
            items[addIndex] = t;
            if (++addIndex == items.length)
                addIndex = 0;
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    // 由头部删除一个元素，如果数组空，则删除线程进入等待状态，直到有新添加元素
    @SuppressWarnings("unchecked")
    public T remove() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            Object x = items[removeIndex];
            if (++removeIndex == items.length)
                removeIndex = 0;
            --count;
            notFull.signal();
            return (T) x;
        } finally {
            lock.unlock();
        }
    }
}
```

上述代码`add` 和 `remove` 方法 都需要先获取锁保证数据的可见性和排它性。
当储存数组满了的时候时候调用`notFull.await()`，线程即释放锁并进入等待队列。
当储存数组未满时，则添加到数组，并通知 `notEmpty` 中等待的线程。
方法中使用`while`循环是为了防止过早或者意外的通知。

#### 2.4.3、Condition的实现分析

主要包括 等待队列、等待和通知。

- **等待队列**

  等待队列是一个`FIFO`的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在`Condition`对象上等待的线程，如果一个线程调用了`Condition.await()`方法，**那么该线程将会释放锁、构造成节点加入等待队列（AQS队列）并进入等待状态**。同步队列和等待队列中节点类型都是同步器的静态内部类`AbstractQueuedSynchronizer.Node`。

  线程调用`Condition.await()`，即**以当前线程构造节点，并加入等待队列的尾部**。

  ```java
  public class ConditionObject implements Condition, java.io.Serializable {    
      /** First node of condition queue. */    
     private transient Node firstWaiter; // 头节点  
      
      /** Last node of condition queue. */    
      private transient Node lastWaiter; // 尾节点 
      
      public ConditionObject() {    }    
      // ... 省略内部代码
  }
  ```

  从上面代码可以看出，ConditionObject 拥有首节点（`firstWaiter`），尾节点（`lastWaiter`）。当前线程调用 `#await()`方法时，将会以当前线程构造成一个节点（Node），并将节点加入到该队列的尾部。结构如下：![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\asd3s3das54dasd)

  如下图所示，`Condition`的实现是同步器的内部类，因此每个`Condition`实例都能够访问同步器提供的方法，相当于每个`Condition`都拥有所属同步器的引用。  

   ![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\1k421j31451)

- **等待**

  调用`Condition`的`await()`等方法，会使当前线程进入等待队列并释放锁，同时线程状态变为等待状态。当从`await()`方法返回时，当前线程一定获取了`Condition`相关联的锁。 

  ```java
  public final void await() throws InterruptedException {
      // 当前线程中断
      if (Thread.interrupted())
          throw new InterruptedException();
      //当前线程加入等待队列
      Node node = addConditionWaiter();
      //释放锁
      long savedState = fullyRelease(node);
      int interruptMode = 0;
      /**
       * 检测此节点的线程是否在同步队上，如果不在，则说明该线程还不具备竞争锁的资格，则继续等待
       * 直到检测到此节点在同步队列上
       */
      while (!isOnSyncQueue(node)) {
          //线程挂起
          LockSupport.park(this);
          //如果已经中断了，则退出
          if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
              break;
      }
      //竞争同步状态
      if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
          interruptMode = REINTERRUPT;
      // 清理下条件队列中的不是在等待条件的节点
      if (node.nextWaiter != null) // clean up if cancelled
          unlinkCancelledWaiters();
      if (interruptMode != 0)
          reportInterruptAfterWait(interruptMode);
  }
  ```

  - 首先，将当前线程新建一个节点同时加入到条件队列中。
  - 然后，释放当前线程持有的同步状态。
  - 之后，则是**不断**检测该节点代表的线程，出现在 CLH 同步队列中（收到 signal 信号之后，就会在 AQS 队列中检测到），如果不存在则一直**挂起**。
  - 最后，**重新**参与竞争，获取到同步状态。
  - **注意**： 如果不是通过其他线程调用`Condition.signal()`方法唤醒，而是对等待线程进行中断，则会抛出`InterruptedException`。 

   ![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\32322fd1dwsfsd)

  

- **通知**

  调用`Condition`的`signal()`方法，将会唤醒在等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移到同步队列中。 

  ```java
  public final void signal() {    
      //检测当前线程是否为拥有锁的独    
    if (!isHeldExclusively())        
          throw new IllegalMonitorStateException();    
      //头节点，唤醒条件队列中的第一个节点    
    Node first = firstWaiter;    
      if (first != null)        
        doSignal(first);    //唤醒
  }
  ```

  调用该方法的前置条件是**当前线程必须获取了锁**，可以看到`signal()`方法进行了`isHeldExclusively()`检查，也就是当前线程必须是获取了锁的线程。
接着获取等待队列的首节点，将其移动到同步队列并使用`LockSupport`唤醒节点中的线程。
  

 ![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\1234sdf21rwsdfsd)

  通过调用同步器的`enq(Node node)`方法，Condition队列中的头节点线程安全地移动到同步队列。
  当节点移动到同步队列后，当前线程再使用`LockSupport`唤醒该节点的线程。  

  被唤醒后的线程，将从`await()`方法中的`while`循环中退出`isOnSyncQueue(Node node)`方法返回`true`，节点已经在同步队列中，进而调用同步器的`acquireQueued()`方法加入到获取同步状态的竞争中。

  成功获取同步状态之后，被唤醒的线程将从先前调用的`await()`方法返回，此时该线程已经成功地获取了锁。 

  `Condition`的`signalAll()`方法，相当于对等待队列中的每个节点均执行一次signal()方法，效果就是将等待队列中所有节点全部移动到同步队列中，并唤醒每个节点的线程。



# 二、并发容器

## 1、ConcurrentHashMap

### 1.1、简介

`ConcurrentHashMap`是在JDK1.5时，J.U.C引入的一个同步集合工具类，顾名思义，这是一个线程安全的HashMap。不同版本的ConcurrentHashMap，内部实现机制千差万别，本节所有的讨论基于JDK1.8。

**ConcurrentHashMap**的类继承关系并不复杂：

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\as1d55as142334)

可以看到ConcurrentHashMap继承了**AbstractMap**，这是一个`java.util`包下的抽象类，提供Map接口的骨干实现，以最大限度地减少实现Map这类数据结构时所需的工作量，一般来讲，如果需要重复造轮子——自己来实现一个Map，那一般就是继承`AbstractMap`。

`ConcurrentHashMap` 是在 `HashMap` 的线程安全的版本，不允许 **空键空值**。

和`HashMap`类似，`ConcurrentHashMap`使用了一个`table`来存储`Node`，`ConcurrentHashMap`同样使用记录的`key`的`hashCode`来寻找记录的存储`index`，而处理哈希冲突的方式与`HashMap`也是类似的，冲突的记录将被存储在同一个位置上，形成一条链表，当链表的长度大于`8`的时候会将链表转化为一棵 **红黑树**，从而将查找的复杂度从`O(N)`降到了`O(lgN)`。

 ![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\18458df8as142)

### 1.2、ConcurrentHashMap基本结构

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\145df4g58f5qqas)

#### 1.2.1、基本结构

**ConcurrentHashMap**内部维护了一个`Node`类型的数组，也就是`table`；

`transient volatile Node[] table;`

数组的每一个位置`table[i]`代表了一个桶，当插入键值对时，会根据键的hash值映射到不同的桶位置，table一共可以包含**4种不同类型**的桶：**Node**、**TreeBin**、**ForwardingNode**、**ReservationNode**。上图中，不同的桶用不同颜色表示。可以看到，有的桶链接着**链表**，有的桶链接着**树**，这也是JDK1.8中ConcurrentHashMap的特殊之处 。

需要注意的是：**TreeBin**所链接的是一颗红黑树，红黑树的结点用**TreeNode**表示，所以ConcurrentHashMap中实际上一共有**五种不同类型**的Node结点。 

*之所以用**TreeBin**而不是直接用**TreeNode**，是因为红黑树的操作比较复杂，包括构建、左旋、右旋、删除，平衡等操作，用一个代理结点TreeBin来包含这些复杂操作，其实是一种“职责分离”的思想。另外TreeBin中也包含了一些加/解锁的操作。*   

> 在**JDK1.8之前，ConcurrentHashMap采用了分段锁的设计思路，以减少热点域的冲突。JDK1.8时不再延续， 而是利用 CAS + Synchronized 来保证并发更新的安全**，转而直接对每个桶加锁，并用“红黑树”链接冲突结点。关于红黑树和一般HashMap的实现思路。

#### 1.2.2、节点定义

- **Node节点**

  Node结点的定义非常简单，也是其它四种类型结点的父类。

  > 默认链接到`table[i]`——桶上的结点就是Node结点。
  > 当出现hash冲突时，Node结点会首先以**链表**的形式链接到table上，当结点数量超过一定数目时，链表会转化为红黑树。因为链表查找的平均时间复杂度为`O(n)`，而红黑树是一种平衡二叉树，其平均时间复杂度为`O(logn)`。 

- **TreeNode节点**

  `TreeNode`就是红黑树的结点，TreeNode不会直接链接到`table[i]`——桶上面，而是由`TreeBin`链接，`TreeBin`会指向红黑树的根结点。 

- **TreeBin节点**

  **TreeBin**相当于`TreeNode`的代理结点。`TreeBin`会直接链接到`table[i]`——桶上面，该结点提供了一系列红黑树相关的操作，以及加锁、解锁操作。 

-  **ForwardingNode结点** 

  **ForwardingNode结点仅仅在扩容时才会使用**

  ForwardingNode是一种临时结点，在扩容进行中才会出现，hash值固定为-1，且不存储实际数据。
  如果旧table数组的一个hash桶中全部的结点都迁移到了新table中，则在这个桶中放置一个ForwardingNode。
  **读操作碰到ForwardingNode时，将操作转发到扩容后的新table数组上去执行；写操作碰见它时，则尝试帮助扩容。**

-  **ReservationNode结点** 

  保留结点，ConcurrentHashMap中的一些特殊方法会专门用到该类结点。
  hash值固定为-3， 不保存实际数据
  只在`computeIfAbsent`和`compute`这两个函数式API中充当占位符加锁使用

### 1.3、ConcurrentHashMap的构造

#### 1.3.1、构造器定义

**ConcurrentHashMap**提供了**五个构造器**，这五个构造器内部最多也只是计算了下table的初始容量大小，并没有进行实际的创建table数组的工作： 

> ConcurrentHashMap，采用了一种**“懒加载”**的模式，只有到**首次插入键值对**的时候，才会真正的去初始化table数组。 

**空构造器**

```java
public ConcurrentHashMap() {
}
```

**指定table初始容量的构造器**

```java
/**
 * 指定table初始容量的构造器.
 * tableSizeFor会返回大于入参（initialCapacity + (initialCapacity >>> 1) + 1）的最小2次幂值
 */
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();

    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));

    this.sizeCtl = cap;
}
```

**指定table初始容量、负载因子、并发级别的构造器**

注意：**concurrencyLevel只是为了兼容JDK1.8以前的版本，并不是实际的并发级别，loadFactor也不是实际的负载因子**

```java
/**
 * 指定table初始容量、负载因子、并发级别的构造器.
 * <p>
 * 注意：concurrencyLevel只是为了兼容JDK1.8以前的版本，并不是实际的并发级别，loadFactor也不是实际的负载因子
 * 这两个都失去了原有的意义，仅仅对初始容量有一定的控制作用
 */
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();

    if (initialCapacity < concurrencyLevel)
        initialCapacity = concurrencyLevel;

    long size = (long) (1.0 + (long) initialCapacity / loadFactor);
    int cap = (size >= (long) MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int) size);
    this.sizeCtl = cap;
}
```

#### 1.3.2、常量/字段定义

##### 1.3.2.1、常量

```java
// 最大容量：2^30=1073741824
private static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认初始值，必须是2的幕数
private static final int DEFAULT_CAPACITY = 16;

//能转化为数组的最大长度 231 -1 - 8.
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//默认的并发等级
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

//负载因子，为了兼容JDK1.8以前的版本而保留。
//JDK1.8中的ConcurrentHashMap的负载因子恒定为0.75，即使构造方法中指定了负载因子也无用
private static final float LOAD_FACTOR = 0.75f;

// 链表转红黑树阀值,> 8 链表转换为红黑树
static final int TREEIFY_THRESHOLD = 8;

//树转链表阀值，小于等于6（tranfer时，lc、hc=0两个计数器分别++记录原bin、新binTreeNode数量，<=UNTREEIFY_THRESHOLD 则untreeify(lo)）
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 在链表转变成树之前，还会有一次判断：
 * 即只有键值对数量大于MIN_TREEIFY_CAPACITY，才会发生转换。
 * 这是为了避免在Table建立初期，多个键值对恰好被放入了同一个链表中而导致不必要的转化。
 */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
 * 在树转变成链表之前，还会有一次判断：
 * 即只有键值对数量小于MIN_TRANSFER_STRIDE，才会发生转换.
 */
private static final int MIN_TRANSFER_STRIDE = 16;

/**
 * 用于在扩容时生成唯一的随机数.
 */
private static int RESIZE_STAMP_BITS = 16;

/**
 * 可同时进行扩容操作的最大线程数.
 */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

// 32-16=16，sizeCtl中记录size大小的偏移量
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

// 标识ForwardingNode结点（在扩容时才会出现，不存储实际数据）的hash值
static final int MOVED     = -1;

// 红黑树根节点的hash值
static final int TREEBIN   = -2;

// ReservationNode的hash值
static final int RESERVED  = -3;

// CPU核心数，扩容时使用
static final int NCPU = Runtime.getRuntime().availableProcessors();
```

##### 1.3.2.2、字段

```java
/**
 * Node数组，标识整个Map，首次插入元素时创建，大小总是2的幂次.
 */
transient volatile Node<K, V>[] table;

/**
 * 扩容后的新Node数组，只有在扩容时才非空，数组为table的两倍；
 */
private transient volatile Node<K, V>[] nextTable;

/**
 * 控制table的初始化和扩容.
 *  0 : 初始默认值
 * -1 : 有线程正在进行table的初始化
 * >0 : table初始化时使用的容量，或初始化/扩容完成后的threshold
 * -(1 + nThreads) : 记录正在执行扩容任务的线程数
 */
private transient volatile int sizeCtl;

/**
 * 扩容时需要用到的一个下标变量.
 */
private transient volatile int transferIndex;

/**
 * 计数基值,当没有线程竞争时，计数将加到该变量上。类似于LongAdder的base变量
 */
private transient volatile long baseCount;

/**
 * 计数数组，出现并发冲突时使用。类似于LongAdder的cells数组
 */
private transient volatile CounterCell[] counterCells;

/**
 * 自旋标识位，用于CounterCell[]扩容时使用。类似于LongAdder的cellsBusy变量
 */
private transient volatile int cellsBusy;


// 视图相关字段
private transient KeySetView<K, V> keySet;
private transient ValuesView<K, V> values;
private transient EntrySetView<K, V> entrySet;
```

### 1.4、ConcurrentHashMap的相关函数

-  `spread(int h)`：散列计算， 作用同`HashMap`的 `hash`方法 。
-  `tableSizeFor(int c)`：根据传入的值计算`ConcurrentHashMap`的容量
-  `size()`：计算`ConcurrentHashMap`的大小
-  `initTable()`：初始化`table` 
-  `get(Object key)`：取数据
-  `put(K key, V value)`：存数据
-  `remove(Object key)`：移除数据

#### 1.4.1、put(key, value)操作

 put方法内部调用了`putVal()`这个私有方法。

 <img src=".\img\456sdfsd7sdf4" alt="img" style="zoom:150%;" />

首先根据**key**计算**hash**值，然后通过**hash**值与**table**容量进行运算，计算得到key所映射的索引——也就是对应到**table**中桶的位置。

这里需要注意的是计算索引的方式：`i = (n - 1) & hash`

`n - 1 == table.length - 1`，`table.length` 的大小必须为**2的幂次**的原因就在这里。

 当`table.length`为2的幂次时，`(table.length-1)`的二进制形式的特点是**除最高位外全部是1**，配合这种索引计算方式可以**实现key在table中的均匀分布，减少hash冲突**——出现hash冲突时，结点就需要以链表或红黑树的形式链接到table[i]，这样无论是插入还是查找都需要额外的时间。 

##### 1.4.1.1、首次初始化table —— 懒加载

之前讲构造器的时候说了，**ConcurrentHashMap**在构造的时候并不会初始化table数组，首次初始化就在这里通过**initTable**方法完成。

-  `(1.0)` 当`sizeCtl < 0`时，则会让出CPU的时间，等待下次重试。
-  `(2.0)` 当`sizeCtl >= 0`时，则通过 `CAS` 原子操作将 `sizeCtl` 设置为 `-1`。
-  `(3.0)` **设置成功之后，如果table为空，则初始化对应长度的 `Node`数组。并将 当前容量能保存的最大数量(`n * LOAD_FACTOR`)赋值给`sizeCtl`**

```java
/**
 * -1：正在初始化 ,其他小于0的数则表示正在调整大小
 * 初始化table, 使用sizeCtl作为初始化容量.
 */
private final Node<K, V>[] initTable() {
    Node<K, V>[] tab;
    int sc;
    while ((tab = table) == null || tab.length == 0) {  //自旋直到初始化成功
        if ((sc = sizeCtl) < 0)         // sizeCtl<0 说明table已经正在初始化/扩容
            Thread.yield();
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {  // 将sizeCtl更新成-1,表示正在初始化中
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);     // n - (n >>> 2) = n - n/4 = 0.75n, 前面说了loadFactor已在JDK1.8废弃
                }
            } finally {
                sizeCtl = sc;               // 设置threshold = 0.75 * table.length
            }
            break;
        }
    }
    return tab;
}
```

##### 1.4.1.2、table[i]对应的桶为空

最简单的情况，直接CAS操作占用桶`table[i]`即可。

##### 1.4.1.3、发现**ForwardingNode**结点，说明此时table正在扩容，则尝试协助进行数据迁移

**ForwardingNode**结点是ConcurrentHashMap中的五类结点之一，相当于一个占位结点，表示当前table正在进行扩容，当前线程可以尝试协助数据迁移。 

##### 1.4.1.4、出现hash冲突

当两个不同key映射到同一个`table[i]`桶中时，就会出现这种情况：

- 当table[i]的结点类型为Node——链表结点时，就会将新结点以**“尾插法”**的形式插入链表的尾部。

- 当table[i]的结点类型为TreeBin——红黑树代理结点时，就会将新结点通过红黑树的插入方式插入。

  **注意**： 涉及将链表转换为红黑树 —— **treeifyBin** ，**但实际情况并非立即就会转换，当table的容量小于64(静态常量：`MIN_TREEIFY_CAPACITY`)时，出于性能考虑，只是对table数组扩容1倍**——**tryPresize()**

#### 1.4.2、get(Object key)

和[HashMap](https://www.jianshu.com/p/d4fee00fe2f8) 类似，当前的索引处时要获取的值则返回当前值， 否则遍历链表。

 ![img](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\445df887w25a)

### 1.5、小结

**为什么要用ConcurrentHashMap**

原因有三：并发编程中`HashMap`会导致死循环；`HashTable`效率又非常低；`ConcurrentHashMap`的锁分段技术(JDK1.7)，JDK1.8开始使用`CAS + Synchronized + 底层使用数组加链表加红黑树`可有效提升并发访问率。

- **在并发编程使用`HashMap`会导致死循环。**
   在多线程环境下，使用`HashMap`进行`put`操作会引起 **死循环**，导致CPU利用率接近`100%`，所以在并发情况下不能使用`HashMap`。
   **是因为多线程会导致`HashMap`的`Entry`链表形成 环形数据结构，一旦形成环形数据结构，`Entry`的`next`节点永远不为空，就会产生死循环获取`Entry`。**
   以下代码就会导致死循环(`java se 5`)：

  ```java
  public static void main(String[] args) throws InterruptedException {
      long time = System.currentTimeMillis();
      HashMap<String, String> map = new HashMap<>(2);
      Thread t = new Thread(() -> {
          for (int i = 0; i < 100000; i++) {
              new Thread(() -> {
                  for (int j = 0; j < 1000; j++) {
                      String s = UUID.randomUUID().toString();
                      map.put(s, s);
                  }
              }, "ftf" + i).start();
          }
      }, "ftf");
      t.start();
      t.join();
      System.out.println(System.currentTimeMillis() - time);
  }
  ```

-  **线程安全的`HashTable`效率非常低。** 

  `HashTable`容器使用`synchronized`来保证线程安全，但在线程竞争激烈的情况下`HashTable`的效率非常低下。因为**当一个线程访问`HashTable`的同步方法，其他线程也访问`HashTable`的同步方法时，会进入阻塞或轮询状态**。如`线程1`使用`put`进行元素添加，`线程2`不但不能使用`put`方法添加元素，也不能使用`get`方法来获取元素，所以竞争越激烈效率越低。



## 2、ConcurrentSkipListMap

### 2.1、ConcurrentSkipListMap简介

ConcurrentSkipListMap的类继承图： 

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\4sd58f8sd4f8s)

我们知道，一般的Map都是无序的，也就是只能通过键的hash值进行定位。JDK为了实现有序的Map，提供了一个**SortedMap**接口，SortedMap提供了一些根据键范围进行查找的功能，比如返回整个Map中 key最小/大的键、返回某个范围内的子Map视图等等。

为了进一步对有序Map进行增强，JDK又引入了**NavigableMap**接口，该接口进一步扩展了SortedMap的功能，提供了根据指定Key返回最接近项、按升序/降序返回所有键的视图等功能。

同时，也提供了一个基于NavigableMap的实现类——**TreeMap**，TreeMap底层基于红黑树设计，是一种有序的Map。

我们在Java世界里看到了两种实现key-value的数据结构：`Hash`、`TreeMap`，这两种数据结构各自都有着优缺点。

1. Hash表：插入、查找最快，为O(1)；如使用链表实现则可实现无锁；数据有序化需要显式的排序操作。
2. 红黑树：插入、查找为O(logn)，但常数项较小；无锁实现的复杂性很高，一般需要加锁；数据天然有序。

然而，这次介绍第三种实现key-value的数据结构：SkipList。SkipList有着不低于红黑树的效率，但是其原理和实现的复杂度要比红黑树简单多了。

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\ssdsd54821sad42) 

J.U.C提供了基于**ConcurrentNavigableMap**接口的一个实现——`ConcurrentSkipListMap`。ConcurrentSkipListMap可以看成是并发版本的`TreeMap`，但是和`TreeMap`不同是，`ConcurrentSkipListMap`并不是基于红黑树实现的，其底层是一种类似**跳表（Skip List）**的结构。  

### 2.2、SkipList跳跃列表

#### 2.2.1、简介

**Skip List**（以下简称跳表），是一种类似链表的数据结构，其查询/插入/删除的时间复杂度都是`O(logn)`。 

我们知道，通常意义上的链表是不能支持随机访问的（通过索引快速定位），其查找的时间复杂度是`O(n)`，而数组这一可支持随机访问的数据结构，虽然查找很快，但是插入/删除元素却需要移动插入点后的所有元素，时间复杂度为`O(n)`。 

为了解决这一问题，引入了树结构，树的增删改查效率比较平均，一棵平衡二叉树（AVL）的增删改查效率一般为`O(logn)`，比如工业上常用红黑树作为AVL的一种实现。

但是，AVL的实现一般都比较复杂，插入/删除元素可能涉及对整个树结构的修改，特别是并发环境下，通常需要全局锁来保证AVL的线程安全，于是又出现了一种类似链表的数据结构——**跳表**。

跳表，它是一种可以替代平衡树的数据结构，**其数据元素默认按照key值升序，天然有序**。Skip list让已排序的数据分布在多层链表中，以0-1随机数决定一个数据的向上攀升与否，通过“空间来换取时间”的一个算法，在每个节点中增加了向前的指针**，在插入、删除、查找时可以忽略一些不可能涉及到的结点，从而提高了效率**。 

**使用CAS+自旋保证线程安全的；**

####  2.2.2、SkipList的特性

1. 由很多层结构组成，level( 层数 )是通过一定的概率随机产生的，有一个最大层数**MAX_LEVEL**限制 
2. **每一层都是一个有序的链表，默认是升序**，也可以根据创建映射时所提供的Comparator进行排序，具体取决于使用的构造方法
3. 最底层(Level 1)的链表包含所有元素
4. 如果一个元素出现在`Level i` 的链表中，则它在`Level i` 之下的链表也都会出现
5. **每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素**<img src=".\img\2018120824003.png" alt="1499590828559201707090003" style="zoom:150%;" />

#### 2.2.3、SkipList的查找

**传统的单链表：** 

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\asd44a4s21asd)

上图的单链表中（省去了结点之间的链接），当想查找7、15、46这三个元素时，必须从头指针head开始，遍历整个单链表，其查找复杂度很低，为`O(n)`。  

 **Skip List**的数据结构：

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\dsfjsio2a0129kds)

上图是Skip List一种可能的结构，它分了2层，假设我们要查找**“15”**这个元素，那么整个步骤如下：

1. 从头指针**head**开始，找到第一个结点的最上层，发现其指向的下个结点值为8，小于15，则直接从1结点跳到8结点。
2. 8结点最上层指向的下一结点值为18，大于15，则从8结点的下一层开始查找。
3. 从8结点的最下层一直向后查找，依次经过10、13，最后找到15结点。

 上述整个**查找路径如下图标黄**部分所示：

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\sdf211a2s552)

 同理，如果要查找**“46”**这个元素，则整个查找路径如下图标黄部分所示：

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\sd4f56sd4f6sd12)

#### 2.2.4、SkipList的插入

SkipList的插入操作主要包括：

1. 查找合适的位置。这里需要明确一点就是在确认新节点要占据的层次K时，采用丢硬币的方式，完全随机。如果占据的层次K大于链表的层次，则重新申请新的层，否则插入指定层次
2. 申请新的节点
3. 调整指针

假定我们要插入的元素为23，经过查找可以确认她是位于25后，9、16、21前。当然需要考虑申请的层次K。

**如果层次K > 3，需要申请新层次** 

 <img src=".\img\2018120824005.png" alt="201707090005" style="zoom:150%;" />

**如果层次 K = 2**，直接在Level 2 层插入即可 

 ![201707090006](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\2018120824006.png)

#### 2.2.5、SkipList的删除

删除节点和插入节点思路基本一致：找到节点，删除节点，调整指针。

比如删除节点9，如下： ![201707090007](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\2018120824007.png) 

###  2.3、ConcurrentSkipListMap内部结构

`ConcurrentSkipListMap`其内部采用`SkipLis`数据结构实现。为了实现`SkipList`，`ConcurrentSkipListMap`**提供了三个内部类来构建这样的链表结构：`Node`、`Index`、`HeadIndex`**。其中`Node`表示最底层的单链表有序节点、`Index`表示为基于`Node`的索引层，`HeadIndex`用来维护索引层次。到这里我们可以这样说**ConcurrentSkipListMap是通过HeadIndex维护索引层次，通过Index从最上层开始往下层查找，一步一步缩小查询范围，最后到达最底层Node时，就只需要比较很小一部分数据了**。在JDK中的关系如下图： <img src=".\img\2018120824008.png" alt="img" style="zoom:150%;" />

```java
public class ConcurrentSkipListMap2<K, V> extends AbstractMap<K, V>
    implements ConcurrentNavigableMap<K, V>, Cloneable, Serializable {
    /**
     * 最底层链表的头指针BASE_HEADER
     */
    private static final Object BASE_HEADER = new Object();

    /**
     * 最上层链表的头指针head
     */
    private transient volatile HeadIndex<K, V> head;

    /* ---------------- 普通结点Node定义 -------------- */
    static final class Node<K, V> {
        final K key;
        volatile Object value;
        volatile Node<K, V> next;

        // ...
    }

    /* ---------------- 索引结点Index定义 -------------- */
    static class Index<K, V> {
        final Node<K, V> node;      // node指向最底层链表的Node结点
        final Index<K, V> down;     // down指向下层Index结点
        volatile Index<K, V> right; // right指向右边的Index结点

        // ...
    }

    /* ---------------- 头索引结点HeadIndex -------------- */
    static final class HeadIndex<K, V> extends Index<K, V> {
        final int level;    // 层级

        // ...
    }
}
```

####  2.3.1、节点定义

<img src=".\img\1898124460-5b85fa780663c_articlex" alt="img" style="width:700px;height:410px;"/>

- **普通节点：Node**

  普通节点`Node`，也就是`ConcurrentSkipListMap`最底层链表中的节点，保存这实际的键值对， 如果单独看底层链，其实就是一个按照Key有序排列的单链表，key-value和一个指向下一个节点的next。 

  ```java
  static final class Node<K,V> {
      final K key;
      volatile Object value;
      volatile ConcurrentSkipListMap.Node<K, V> next;
  	
       /**
       * 正常结点.
       */
      Node(K key, Object value, Node<K, V> next) {
          this.key = key;
          this.value = value;
          this.next = next;
      }
  
      /**
       * 标记结点.
       */
      Node(Node<K, V> next) {
          this.key = null;
          this.value = this;
          this.next = next;
      }
      
      /** 省略些许代码 */
  }
  ```

- **索引节点：Index**

  Index节点是除底层链外，**其余各层链表中的非头结点(** 见示意图中的蓝色节点)。每个Index节点包含3个指针：`down，node，right`。 **`down`和`right`指针分别指向下层结点和后继结点，`node`指针指向其最底部的`node`结点。** 

  ```java
  static class Index<K, V> {
      final Node<K, V> node;      // node指向最底层链表的Node结点
      final Index<K, V> down;     // down指向下层Index结点
      volatile Index<K, V> right; // right指向右边的Index结点
  
      Index(Node<K, V> node, Index<K, V> down, Index<K, V> right) {
          this.node = node;
          this.down = down;
          this.right = right;
      }
  }
  ```

- **头索引节点：HeadIndex**

  **HeadIndex**结点是各层链表的头结点，**它是Index类的子类，唯一的区别是增加了一个`level`字段，用于表示当前链表的级别**，越往上层，level值越大。 

  ```java
  static final class HeadIndex<K, V> extends Index<K, V> {
      final int level;    //索引层，从1开始，Node单链表层为0
  
      HeadIndex(Node<K, V> node, Index<K, V> down, Index<K, V> right, int level) {
          super(node, down, right);
          this.level = level;
      }
  }
  ```

#### 2.3.2、构造器定义和初始化

**注：ConcurrentSkipListMap会基于比较器——Comparator ，来进行键Key的比较，如果构造时未指定Comparator ，那么就会按照Key的自然顺序进行比较，所谓Key的自然顺序是指key实现Comparable接口。** 

`ConcurrentSkipListMap`提供了4个构造函数，每个构造器都会调用`#initialize()`方法进行初始化。

```java
private void initialize() {
    keySet = null;
    entrySet = null;
    values = null;
    descendingMap = null;
    head = new HeadIndex<K, V>(new Node<K, V>(null, BASE_HEADER, null),null, null, 1);
}
```

**initialize**方法将一些字段置初始化null，然后将head指针指向新创建的**HeadIndex**结点。初始化完成后，ConcurrentSkipListMap的结构如下： 

 ![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\Java并发.assets\sdf4w45a54d21)

其中，**head**和**BASE_HEADER**都是ConcurrentSkipListMap的字段：

```java
/**
 * 最底层链表的头指针BASE_HEADER
 */
private static final Object BASE_HEADER = new Object();

/**
 * 最上层链表的头指针head
 */
private transient volatile HeadIndex<K, V> head;
```

### 2.4、ConcurrentSkipListMap的核心操作

#### 2.4.1、put操作

`CoucurrentSkipListMap`提供了put()方法用于将指定值与此映射中的指定键关联。 要注意的是`ConcurrentSkipListMap`在插入键值对时，**Key和Value都不能为null**： 

```java
/**
 * 插入键值对.
 *
 * @param key   键
 * @param value 值
 * @return 如果key存在，返回旧value值；否则返回null
 */
public V put(K key, V value) {
    if (value == null)          // ConcurrentSkipListMap的Value不能为null
        throw new NullPointerException();
    return doPut(key, value, false);
}
```

 doPut() 方法：

```java
`private V doPut(K key, V value, boolean onlyIfAbsent) {    Node z;             // added node    if (key == null)        throw new NullPointerException();    // 比较器    Comparator cmp = comparator;    outer: for (;;) {        for (Node b = findPredecessor(key, cmp), n = b.next; ; ) {        /** 省略代码 */`
```

`#doPut()`方法有三个参数，除了key，value外还有一个boolean类型的onlyIfAbsent，该参数作用与如果存在当前key时，该做何动作。当onlyIfAbsent为false时，替换value，为true时，则返回该value。用代码解释为： 

```java
 if (!map.containsKey(key))
    return map.put(key, value);
else
     return map.get(key);
```

内部的`findPredecessor`方法，`findPredecessor`用于查找**“小于且最接近给定key”的Node结点，并且这个Node结点必须有上层结点**： 

```java
/**
 * 返回“小于且最接近给定key”的数据结点.
 * 如果不存在这样的数据结点，则返回底层链表的头结点.
 *
 * @param key 待查找的键
 */
private Node<K, V> findPredecessor(Object key, Comparator<? super K> cmp) {
    if (key == null)
        throw new NullPointerException();

    /**
     * 从最上层开始，往右下方向查找
   */
    for (; ; ) {
        for (Index<K, V> q = head, r = q.right, d; ; ) {    // 从最顶层的head结点开始查找
            if (r != null) {             // 存在右结点
                Node<K, V> n = r.node;
                K k = n.key;
                if (n.value == null) {   // 处理结点”懒删除“的情况
                    if (!q.unlink(r))
                        break;
                    r = q.right;
                    continue;
                }
                if (cpr(cmp, key, k) > 0) { // key大于k,继续向右查找
                    q = r;
                    r = r.right;
                    continue;
                }
            }

            //已经到了level1的层
            if ((d = q.down) == null)   // 不存在下结点，说明q已经是level1链表中的结点了
                return q.node;          // 直接返回对应的Node结点
       // 转到下一层，继续查找(level-1层)
            q = d;
            r = d.right;
        }
    }
}
```

 <img src=".\img\sadf1s54df54sd" alt="clipboard.png" style="width:725px;height:400px;" />

上图中，假设要查找的Key为72，则步骤如下：

1. 从最上方head指向的结点开始，比较①号标红的Index结点的key值，发现3小于72，则继续向右；
2. 比较②号标红的Index结点的key值，发现62小于72，则继续向右
3. 由于此时右边是null，则转而向下，一直到⑥号标红结点；
4. 由于⑥号标红结点的down字段为空（不能再往下了，已经是level1最低层了），则直接返回它的node字段指向的结点，即⑧号结点。

> **注意：如果我们要查找key为59的Node结点，返回的不是Key为45的结点，而是key为23的结点。**

**总结：**

1. 首先通过findPredecessor()方法找到前辈节点Node
2. 根据返回的前辈节点以及key-value，新建Node节点，同时通过CAS设置next
3. 设置节点Node，再设置索引节点。采取抛硬币方式决定层次，如果所决定的层次大于现存的最大层次，则新增一层，然后新建一个Item链表。
4. 最后，将新建的Item链表插入到SkipList结构中。

#### 2.4.2、remove操作

**ConcurrentSkipListMap**在删除键值对时，不会立即执行删除，而是通过引入**“标记结点”**，以**“懒删除”**的方式进行，以提高并发效率。 

```java
public V remove(Object key) {
    return doRemove(key, null);
}

public boolean remove(Object key, Object value) {
    if (key == null)
        throw new NullPointerException();
    return value != null && doRemove(key, value) != null;
}
```

直接调用doRemove()方法，这里remove有两个参数，一个是key，另外一个是value，所以doRemove方法即提供remove key，也提供同时满足key-value。 

```java
final V doRemove(Object key, Object value) {
       if (key == null)
           throw new NullPointerException();
       Comparator<? super K> cmp = comparator;
       outer: for (;;) {
           for (ConcurrentSkipListMap.Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
               Object v; int c;
               if (n == null)
                   break outer;
               ConcurrentSkipListMap.Node<K,V> f = n.next;

               // 不一致读，重新开始
               if (n != b.next)                    // inconsistent read
                   break;

               // n节点已删除
               if ((v = n.value) == null) {        // n is deleted
                   n.helpDelete(b, f);
                   break;
               }

               // b节点已删除
               if (b.value == null || v == n)      // b is deleted
                   break;

               if ((c = cpr(cmp, key, n.key)) < 0)
                   break outer;

               // 右移
               if (c > 0) {
                   b = n;
                   n = f;
                   continue;
               }

               /*
                * 找到节点
                */

               // value != null 表示需要同时校验key-value值
               if (value != null && !value.equals(v))
                   break outer;

               // CAS替换value
               if (!n.casValue(v, null))
                   break;
               if (!n.appendMarker(f) || !b.casNext(n, f))
                   findNode(key);                  // retry via findNode
               else {
                   // 清理节点
                   findPredecessor(key, cmp);      // clean index

                   // head.right == null表示该层已经没有节点，删掉该层
                   if (head.right == null)
                       tryReduceLevel();
               }
               @SuppressWarnings("unchecked") V vv = (V)v;
               return vv;
           }
       }
       return null;
   }
```

调用findPredecessor()方法找到前辈节点，然后通过右移，然后比较，找到后利用CAS把value替换为null，然后判断该节点是不是这层唯一的index，如果是的话，调用tryReduceLevel()方法把这层干掉，完成删除。

其实从这里可以看出，**remove方法仅仅是把Node的value设置null，并没有真正删除该节点Node**，其实从上面的put操作、get操作我们可以看出，他们在寻找节点的时候都会判断节点的value是否为null，如果为null，则调用unLink()方法取消关联关系，如下：

```java
`if (n.value == null) {    if (!q.unlink(r))        break;           // restart    r = q.right;         // reread r    continue;}`
```

#### 2.4.3、get操作



## 3、ConcurrentSkipListSet

**ConcurrentSkipListSet**的实现非常简单，其内部引用了一个`ConcurrentSkipListMap`对象，所有API方法均委托`ConcurrentSkipListMap`对象完成 。

重点看下`add`方法：

```java
public boolean add(E e) {
    return m.putIfAbsent(e, Boolean.TRUE) == null;
}
```

我们知道**ConcurrentSkipListMap**对键值对的要求是均不能为null，所以ConcurrentSkipListSet在插入元素的时候，用一个`Boolean.TRUE`对象（相当于一个值为true的Boolean型对象）作为value，同时`putIfAbsent`可以保证不会存在相同的Key。

所以，最终跳表中的所有Node结点的Key均不会相同，且值都是`Boolean.True`。



## 4、CopyOnWriteArrayList和CopyOnWriteArraySet

`CopyOnWriteArrayList`与`CopyOnWriteArraySet`运用了一种**“写时复制”**的思想。通俗的理解就是当我们需要修改（增/删/改）列表中的元素时，不直接进行修改，而是先将列表Copy，然后在新的副本上进行修改，修改完成之后，再将引用从原列表指向新列表。

这样做的好处是**读/写是不会冲突**的，可以并发进行，读操作还是在原列表，写操作在新列表。仅仅当有多个线程同时进行写操作时，才会进行同步。

**CopyOnWriteArrayList总结：**

**CopyOnWriteArrayList**的思想和实现整体上还是比较简单，它适用于处理**“读多写少”**的并发场景。

**1. 内存的使用**
由于CopyOnWriteArrayList使用了“写时复制”，所以在进行写操作的时候，内存里会同时存在两个array数组，如果数组内存占用的太大，那么可能会造成频繁GC,所以CopyOnWriteArrayList并不适合大数据量的场景。

**2. 数据一致性**
**CopyOnWriteArrayList只能保证数据的最终一致性，不能保证数据的实时一致性——读操作读到的数据只是一份快照**。所以如果希望写入的数据可以立刻被读到，那CopyOnWriteArrayList并不适合。

**CopyOnWriteArraySet总结：**

**CopyOnWriteArraySet**也是基于“写时复制”的思想，它的特性也和**CopyOnWriteArrayList**是类似的，归结起来，有以下几点：

1. 适合“读多写少”且数据量不大的场景。
2. 线程安全
3. 内存的使用较多
4. 迭代是对快照进行的，不会抛出`ConcurrentModificationException`，且迭代过程中不支持修改操作。



## 5、ConcurrentLinkedQueue

### 5.1、简介

在并发编程中，有时候需要使用线程安全的队列。如果要实现一个线程安全的队列有两种方式：

- **使用阻塞算法**：使用阻塞算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现。
- **使用非阻塞算法**：非阻塞的实现方式则可以使用**循环CAS（CAS+自旋）**的方式来实现。

`ConcurrentLinkedQueue`是一个**基于链接节点的无界线的程安全队列，非阻塞的**，它采用`FIFO`的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部；当我们获取一个元素时，它会返回队列头部的元素。它采用了“`wait-free`”算法（即CAS算法）来实现，该算法在`Michael&Scott`算法上进行了一些修改。

**CoucurrentLinkedQueue规定了如下几个不变性：**

1. 在入队的最后一个元素的next为null
2. 队列中所有未删除的节点的item都不能为null且都能从head节点遍历到
3. 对于要删除的节点，不是直接将其设置为null，而是先将其item域设置为null（迭代器会跳过item为null的节点）
4. **允许head和tail更新滞后**。这是什么意思呢？**意思就说是head、tail不总是指向第一个元素和最后一个元素**。

**head的不变性和可变性：**

- 不变性
  1. 所有未删除的节点都可以通过head节点遍历到
  2. head不能为null
  3. head节点的next不能指向自身
- 可变性
  1. head的item可能为null，也可能不为null
     2.允许tail滞后head，也就是说调用succc()方法，从head不可达tail

**tail的不变性和可变性：**

- 不变性
  1. tail不能为null
- 可变性
  1. tail的item可能为null，也可能不为null
  2. tail节点的next域可以指向自身
  3. 允许tail滞后head，也就是说调用succc()方法，从head不可达tail


### 5.2、ConcurrentLinkedQueue原理

#### 5.2.1、队列结构

**ConcurrentLinkedQueue**的内部结构： 

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
    implements Queue<E>, java.io.Serializable {
 
    /**
     * 队列头指针
     */
    private transient volatile Node<E> head;
 
    /**
     * 队列尾指针.
     */
    private transient volatile Node<E> tail;
 
    // Unsafe mechanics
     
    private static final sun.misc.Unsafe UNSAFE;
    private static final long headOffset;
    private static final long tailOffset;
     
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentLinkedQueue.class;
            headOffset = UNSAFE.objectFieldOffset (k.getDeclaredField("head"));
            tailOffset = UNSAFE.objectFieldOffset (k.getDeclaredField("tail"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
 
    /**
     * 队列结点定义
     */
    private static class Node<E> {
        volatile E item;        // 元素值
        volatile Node<E> next;  // 后驱指针
 
        Node(E item) {
            UNSAFE.putObject(this, itemOffset, item);
        }
 
        boolean casItem(E cmp, E val) {
            return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
        }
 
        void lazySetNext(Node<E> val) {
            UNSAFE.putOrderedObject(this, nextOffset, val);
        }
 
        boolean casNext(Node<E> cmp, Node<E> val) {
            return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
        }
 
        // Unsafe mechanics
 
        private static final sun.misc.Unsafe UNSAFE;
        private static final long itemOffset;
        private static final long nextOffset;
 
        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = Node.class;
                itemOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("item"));
                nextOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
 
    //...
}
```

可以看到，**ConcurrentLinkedQueue**内部就是一个简单的单链表结构，每入队一个元素就是插入一个Node类型的结点。字段`head`指向队列头，`tail`指向队列尾，通过`Unsafe`来CAS操作字段值以及Node对象的字段值。 

 ![clipboard.png](.\img\asd211sdf3245f)

#### 5.2.2、构造器定义

ConcurrentLinkedQueue包含两种构造器： 

```java
/**
 * 构建一个空队列（head,tail均指向一个占位结点）.
 */
public ConcurrentLinkedQueue() {
    head = tail = new Node<E>(null);
}

/**
 * 根据已有集合,构造队列
 */
public ConcurrentLinkedQueue(Collection<? extends E> c) {
    Node<E> h = null, t = null;
    for (E e : c) {
        checkNotNull(e);
        Node<E> newNode = new Node<E>(e);
        if (h == null)
            h = t = newNode;
        else {
            t.lazySetNext(newNode);
            t = newNode;
        }
    }
    if (h == null)
        h = t = new Node<E>(null);
    head = h;
    tail = t;
}
```

我们重点看下空构造器，通过空构造器建立的ConcurrentLinkedQueue对象，其`head`和`tail`指针并非指向`null`，而是指向一个item值为null的`Node`结点——哨兵结点，如下图： 

 ![clipboard.png](.\img\asdasd1231g321)

#### 5.2.3、入队操作

入队列就是将入队节点添加到队列的尾部。

对于修改`head`中的`netx`节点以及修改`tail`的指向都是由`CAS方法`完成。

 ![img](E:\软件开发资料\Java\Java资料\java高级\img\asdasd12312s) 

- `添加元素1`：**这里测试发现插入第一个元素后，执行完`if (p.casNext(null, newNode))`后`head`节点的引用指向元素1，`tail`节点的指针指向不变，但成员变量next指向了自己（如图中所示，head指向了元素1，tail还是指向的初始化节点，只不过tail的next指向了自己）。**
- `添加元素2`：队列首先执行`p = (t != (t = tail)) ? t : head;`设置`for (Node<E> t = tail, p = t;;)`中`p`变量指向`head`节点；再次循环，设置**元素1**节点的`next`节点为**元素2**节点，然后执行`casTail(t, newNode)`更新`tail`节点指向`元素2`节点。
-  `添加元素3`：设置**元素2**节点的`next`节点为**元素3**节点（这时`tail`节点的`next`也指向了**元素3**）。
-  `添加元素4`：经过判断进入`else`执行`p = (p != t && t != (t = tail)) ? t : q;`，变量`p`指向`tail`节点的`next`节点；再次循环，执行完`if (p.casNext(null, newNode))`后`head`节点的引用指向**元素4**，然后执行`casTail(t, newNode)`更新`tail`节点指向**元素4**节点。
-  **注意：**`casTail(t, newNode)`的作用就是重置tail队尾指针的指向，可以看到，`ConcurrentLinkedQueue` 其实是以每次跳2个结点的方式移动指针，这主要考虑到并发环境以这种hop跳的方式可以提升效率。 

```java
/**
 * 入队一个元素.
 *
 * @throws NullPointerException 元素不能为null
 */
public boolean add(E e) {
    return offer(e);
}

/**
 * 在队尾入队元素e, 直到成功
 */
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);
    for (Node<E> t = tail, p = t; ; ) {     // 自旋, 直到插入结点成功
        Node<E> q = p.next;
        if (q == null) {                    // CASE1: 正常情况下, 新结点直接插入到队尾
            if (p.casNext(null, newNode)) {
                // CAS竞争插入成功
                if (p != t)                // CAS竞争失败的线程会在下一次自旋中进入该逻辑
                    casTail(t, newNode);   // 重新设置队尾指针tail
                return true;
            }
            // CAS竞争插入失败,则进入下一次自旋
 
        } else if (p == q)       
        // CASE2: 发生了出队操作
       	// p == q 代表着该节点已经被删除了，或者再插入第二个元素时，tail仍是指向初始节点
       	// 由于多线程的原因，我们offer()的时候也会poll()，如果offer()的时候正好该节点已经poll()了
 		// 那么在poll()方法中的updateHead()方法会将head指向当前的q，而把p.next指向自己，即：p.next == p
        // 这样就会导致tail节点滞后head（tail位于head的前面），则需要重新设置p
            p = (t != (t = tail)) ? t : head;
        else
            // 将p重新指向队尾结点
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

#### 5.2.4、出队列

出队列的就是从队列里返回一个节点元素，并清空该节点对元素的引用。

  ![img](.\img\asd123s1s245f)

```java
/**
 * 在队首出队元素, 直到成功
 */
public E poll() {
    restartFromHead:
    for (; ; ) {
        for (Node<E> h = head, p = h, q; ; ) {
            E item = p.item;
 
            if (item != null && p.casItem(item, null)) {    // CASE2: 队首是非哨兵结点(item!=null)
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            } else if ((q = p.next) == null) {      // CASE1: 队首是一个哨兵结点(item==null)
                updateHead(h, p);
                return null;
            } else if (p == q)
                continue restartFromHead;
            else
                p = q;
        }
    }
}
```

 首先获取`head`节点的元素`item`，然后判断是否为空？

- 如果为空，表示另外一个线程已经进行了一次出队操作将该节点的元素取走。
- 如果不为空，则使用`CAS`的方式将头节点的引用设置成`null`，如果CAS成功，则直接返回头节点的元素`item`，如果不成功，表示另外一个线程已经进行了一次出队操作更新了`head`节点，导致元素发生了变化，需要重新获取头节点。

### 5.3、总结

**ConcurrentLinkedQueue**使用了**自旋+CAS**的非阻塞算法来保证线程并发访问时的数据一致性。由于队列本身是一种链表结构，所以虽然算法看起来很简单，但其实需要考虑各种并发的情况，实现复杂度较高，并且ConcurrentLinkedQueue不具备实时的数据一致性，实际运用中，队列一般在生产者-消费者的场景下使用得较多，所以ConcurrentLinkedQueue的使用场景并不如阻塞队列那么多。

另外，关于ConcurrentLinkedQueue还有以下需要注意的几点：

1. ConcurrentLinkedQueue的迭代器是弱一致性的，这在并发容器中是比较普遍的现象，主要是指在一个线程在遍历队列结点而另一个线程尝试对某个队列结点进行修改的话不会抛出`ConcurrentModificationException`，这也就造成在遍历某个尚未被修改的结点时，在next方法返回时可以看到该结点的修改，但在遍历后再对该结点修改时就看不到这种变化。
2. `size`方法需要遍历链表，所以在并发情况下，其结果不一定是准确的，只能供参考。



## 6、ConcurrentLinkDeque

`Queue`是一种具有**FIFO**特点的数据结构，**元素只能在队首进行“出队”操作，在队尾进行“入队”操作。** 

而**Deque**（double-ended queue）是一种双端队列，也就是说可以在任意一端进行“入队”，也可以在任意一端进行“出队”： 

![clipboard.png](.\img\asdasdas11)

 Deque的数据结构示意图如下：

 ![clipboard.png](.\img\asdas1asd244)

我们再来看下JDK中**Queue**和**Deque**这两种数据结构的接口定义，看看Deque和Queue相比有哪些增强： 

### 6.1、Queue接口定义

Queue的接口非常简单，一共只有三种类型的操作：入队、出队、读取。

可以划分如下： 

| 操作类型 | 抛出异常  | 返回特殊值 |
| -------- | --------- | ---------- |
| 入队     | add(e)    | offer(e)   |
| 出队     | remove()  | poll()     |
| 读取     | element() | peek()     |

每种操作类型，**都给出了两种方法，区别就是其中一种操作在队列的状态不满足某些要求时，会抛出异常；另一种，则直接返回特殊值（如null）。** 

### 6.2、Deque接口定义

Queue接口的所有方法Deque都具备，只不过队首/队尾都可以进行“出队”和“入队”操作：

| 操作类型 | 抛出异常      | 返回特殊值    |
| -------- | ------------- | ------------- |
| 队首入队 | addFirst(e)   | offerFirst(e) |
| 队首出队 | removeFirst() | pollFirst()   |
| 队首读取 | getFirst()    | peekFirst()   |
| 队尾入队 | addLast(e)    | offerLast(e)  |
| 队尾出队 | removeLast()  | pollLast()    |
| 队尾读取 | getLast()     | peekLast()    |

除此之外，Deque还可以当作“栈”来使用，我们知道“栈”是一种具有**“LIFO”**特点的数据结构，Deque提供了`push`、`pop`、`peek`这三个栈方法，一般实现这三个方法时，可以利用已有方法，即有如下映射关系： 

| 栈方法 | Deque方法     |
| ------ | ------------- |
| push   | addFirst(e)   |
| pop    | removeFirst() |
| peek   | peekFirst()   |

### 6.3、ConcurrentLinkedDeque简介

`ConcurrentLinkedDeque`是JDK1.7时，J.U.C包引入的一个集合工具类。**在JDK1.7之前，除了Stack类外，并没有其它适合并发环境的“栈”数据结构。ConcurrentLinkedDeque作为双端队列，可以当作“栈”来使用，并且高效地支持并发环境。** 

`ConcurrentLinkedDeque`和`ConcurrentLinkedQueue`一样，采用了无锁算法，底层基于**自旋+CAS**的方式实现。  ![clipboard.png](.\img\fffa123asd12s)

### 6.4、ConcurrentLinkedDeque原理

#### 6.4.1、队列结构

ConcurrentLinkedDeque的内部结构： 

```java
public class ConcurrentLinkedDeque<E> extends AbstractCollection<E>
    implements Deque<E>, java.io.Serializable {

    /**
     * 头指针
     */
    private transient volatile Node<E> head;

    /**
     * 尾指针
     */
    private transient volatile Node<E> tail;
	
    private static final Node<Object> PREV_TERMINATOR, NEXT_TERMINATOR;
    
    // Unsafe mechanics
    //...
    
    /**
     * 双链表结点定义
     */
    static final class Node<E> {
        volatile Node<E> prev;  // 前驱指针
        volatile E item;        // 结点值
        volatile Node<E> next;  // 后驱指针

        Node() {
        }

        Node(E item) {
            UNSAFE.putObject(this, itemOffset, item);
        }
        
        //...
    }
    
    // ...
}
```

可以看到，`ConcurrentLinkedDeque`的内部和`ConcurrentLinkedQueue`类似，不过它是一个双链表结构，每入队一个元素就是插入一个`Node`类型的结点。字段`head`指向队列头，`tail`指向队列尾，通过`Unsafe`来`CAS`操作字段值以及Node对象的字段值。 

 ![clipboard.png](.\img\asf5123as12aws)

#### 6.4.2、构造器定义

`ConcurrentLinkedDeque`包含两种构造器：

```java
/**
 * 空构造器.
 */
public ConcurrentLinkedDeque() {
    head = tail = new Node<E>(null);
}

/**
 * 从已有集合，构造队列
 */
public ConcurrentLinkedDeque(Collection<? extends E> c) {
    Node<E> h = null, t = null;
    for (E e : c) {
        checkNotNull(e);
        Node<E> newNode = new Node<E>(e);
        if (h == null)
            h = t = newNode;
        else {  // 在队尾插入元素
            t.lazySetNext(newNode);
            newNode.lazySetPrev(t);
            t = newNode;
        }
    }
    initHeadTail(h, t);
}
```

####  6.4.3、入队操作

####  6.4.3、出队操作

### 6.5、总结

**ConcurrentLinkedDeque**使用了**自旋+CAS**的非阻塞算法来保证线程并发访问时的数据一致性。由于队列本身是一种双链表结构，所以虽然算法看起来很简单，但其实需要考虑各种并发的情况，实现复杂度较高，并且ConcurrentLinkedDeque不具备实时的数据一致性，实际运用中，如果需要一种线程安全的栈结构，可以使用ConcurrentLinkedDeque。 

另外，关于ConcurrentLinkedDeque还有以下需要注意的几点：

1. ConcurrentLinkedDeque的迭代器是弱一致性的，这在并发容器中是比较普遍的现象，主要是指在一个线程在遍历队列结点而另一个线程尝试对某个队列结点进行修改的话不会抛出ConcurrentModificationException，这也就造成在遍历某个尚未被修改的结点时，在next方法返回时可以看到该结点的修改，但在遍历后再对该结点修改时就看不到这种变化。
2. size方法需要遍历链表，所以在并发情况下，其结果不一定是准确的，只能供参考。




## 7、BlockingQueue

`ConcurrentLinkedQueue`和`ConcurrentLinkedDeque`是以非阻塞算法实现的高性能队列，其使用场景一般是在并发环境下，需要“队列”/“栈”这类数据结构时才会使用；而“阻塞队列”通常利用了“锁”来实现，也就是会阻塞调用线程，其使用场景一般是在**“生产者-消费者”**模式中，用于线程之间的数据交换或系统解耦。 

引入**阻塞队列**最大的好处就是解耦，在软件工程中 ，“高内聚，低耦合”是进行模块设计的准则之一 ，这样“生产者”和“消费者”其实是互不影响的，将来任意一方需要升级时，可以保证系统的平滑过度；

 ![clipboard.png](.\img\sdfsdi12as1sg)

###  7.1、BlockingQueue简介

`BlockingQueue`是在JDK1.5时，随着J.U.C引入的一个接口：

 ![clipboard.png](.\img\s21fs123asas)

阻塞队列（`BlockingQueue`）是一个支持以下两个附加操作的队列：

- **支持阻塞的插入方法：**当队列满时，队列会阻塞插入元素的线程，直到队列不满。
- **支持阻塞的移除方法：**在队列为空时，队列会阻塞获取元素的线程，直到队列变为非空。 

**阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。**

在阻塞队列不可用时，这两个附加操作提供了以下4种处理方式：

| 方法/处理方式 |  抛出异常   | 返回特殊值 | 一直阻塞 |        超时退出        |
| :-----------: | :---------: | :--------: | :------: | :--------------------: |
|   插入方法    |  `add(e)`   | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
|   移除方法    | `remove()`  |  `poll()`  | `take()` |   `poll(time, unit)`   |
|   检查方法    | `element()` |  `peek()`  |    /     |           /            |

- **抛出异常：**队列满时，再添加元素，会抛出 `IllegalStateException("Queue full")`异常 ，当队列为空时，从队列里获取元素会抛出 `NoSuchElementException`异常。 
- **返回特殊值：**往队列里插入元素时，返回`true`表示插入成功。从队列里移除元素，即取出元素，如果没有则返回`null`。
- **一直阻塞：**当阻塞队列满时，如果生产者线程往队列里`put`元素，队列会一直阻塞生产者线程，直到队列可用或者响应中断退出。当队列空时，如果消费者线程从队列里`take`元素，队列会阻塞消费者线程，直到队列不为空。
- **超时退出：**当阻塞队列满时，如果生产者线程往队列里插入`offer(e, time, unit)`元素，队列会阻塞生产者线程一段时间，如果超过了指定的时间`time`，生产者线程就会退出。当阻塞队列为空时，如果消费者线程从阻塞队列里获取`poll(time, unit)`元素，队列会阻塞消费者线程一段时间，如果超过了指定的时间`time`，消费者线程就会退出。
- **put(e)**和**take()**方法会一直阻塞调用线程，直到线程被中断或队列状态可用；
  **offer(e, time, unit)**和**poll(time, unit)**方法会限时阻塞调用线程，直到超时或线程被中断或队列状态可用。 

> **如果是无界阻塞队列，队列不可能会出现满的情况，所以使用`put`或`offer`方法永远不会被阻塞，而且使用`offer`方法时，该方法永远返回`true`。 **

```java
public interface BlockingQueue<E> extends Queue<E> {
    /**
     * 插入元素e至队尾, 如果队列已满, 则阻塞调用线程直到队列有空闲空间.
     */
    void put(E e) throws InterruptedException;

    /**
     * 插入元素e至队列, 如果队列已满, 则限时阻塞调用线程，直到队列有空闲空间或超时.
     */
    boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    /**
     * 从队首删除元素，如果队列为空, 则阻塞调用线程直到队列中有元素.
     */
    E take() throws InterruptedException;

    /**
     * 从队首删除元素，如果队列为空, 则限时阻塞调用线程，直到队列中有元素或超时.
     */
    E poll(long timeout, TimeUnit unit) throws InterruptedException;

    // ...
}
```

除此之外，BlockingQueue还具有以下特点：

- **BlockingQueue队列中不能包含null元素；**
- BlockingQueue接口的实现类都必须是线程安全的，实现类一般通过“锁”保证线程安全；
- BlockingQueue 可以是限定容量的。`#remainingCapacity()`方法用于返回剩余可用容量，对于没有容量限制的BlockingQueue实现，该方法总是返回`Integer.MAX_VALUE` 。



## 8、ArrayBlockingQueue

### 8.1、简介

`ArrayBlockingQueue`是在JDK1.5时，随着J.U.C包引入的一种阻塞队列，它实现了**BlockingQueue**接口，底层基于**数组**实现： 

![clipboard.png](.\img\aasas433fsd2112d123)

`ArrayBlockingQueue`是 一个**由数组实现的有界阻塞队列**，底层采用`ReentrantLock`锁保证并发安全。该队列采用 FIFO 的原则对元素进行排序添加的。 在初始构造的时候需要指定队列的容量。具有如下特点：

1.  队列的容器一旦在构造方法时指定，后续不能改变。
2. 插入元素时，在队尾进行；删除元素时，在队首进行。
3. 队列满时，调用特定方法(`put`，`offer(e, time, unit)`)插入元素会阻塞线程；删除元素(`take`, `poll(time, unit`))时也会阻塞线程。
4. 支持公平/非公平锁策略，默认为非公平锁策略。
5.   内部使用可重入锁 `ReentrantLock` + `Condition` 来完成多线程环境的并发操作。 

> *这里的公平策略，是指当线程从阻塞到唤醒后，以最初请求的顺序（FIFO）来添加或删除元素；非公平策略指线程被唤醒后，谁先抢占到锁，谁就能往队列中添加/删除，是随机的。* 

### 8.2、构造方法

`java.util.concurrent.ArrayBlockingQueue` 的构造方法，代码如下：

```java
/**
 * 指定队列初始容量和公平/非公平策略的构造器.
 */
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();

    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);     // 利用独占锁的策略
   notEmpty = lock.newCondition();
    notFull = lock.newCondition();
}
```

从构造器也可以看出，`ArrayBlockingQueue`在构造时就指定了内部数组的大小，并通过`ReentrantLock`来保证并发环境下的线程安全。

`ArrayBlockingQueue`的公平/非公平策略其实就是内部`ReentrantLock`对象的策略，此外构造时还创建了两个`Condition`对象。在队列满时，插入线程需要在`notFull`上等待；当队列空时，删除线程会在`notEmpty`上等待：

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {

    /**
     * 内部数组
     */
    final Object[] items;

    /**
     * 下一个待删除位置的索引: take, poll, peek, remove方法使用
     */
    int takeIndex;

    /**
     * 下一个待插入位置的索引: put, offer, add方法使用
     */
    int putIndex;

    /**
     * 队列中的元素个数
     */
    int count;

    /**
     * 全局锁
     */
    final ReentrantLock lock;

    /**
     * 非空条件队列：当队列空时，线程在该队列等待获取
     */
    private final Condition notEmpty;

    /**
     * 非满条件队列：当队列满时，线程在该队列等待插入
     */
    private final Condition notFull;

    //...
}
```

`ArrayBlockingQueue` 内部使用可重入锁 `ReentrantLock` + `Condition` 来完成多线程环境的并发操作。

- `items`变量，一个定长数组，维护 `ArrayBlockingQueue` 的元素。
  - `takeIndex` 变量，下一个待删除位置的索引: `take`, `poll`, `peek`, `remove`方法使用。
  - `putIndex` 变量，`int` ，下一个待插入位置的索引: `put`, `offer`, `add`方法使用。
  - `count` 变量，元素个数。
- `lock`变量，`ReentrantLock` ，`ArrayBlockingQueue` 出列入列都必须获取该锁，两个步骤共用一个锁。
  - `notEmpty` 变量，非空，即**出列**条件队列。
  - `notFull` 变量，未满，即**入列**条件队列。

### 8.3、入队

`ArrayBlockingQueue` 提供了诸多方法，可以将元素加入队列尾部。

- `#add(E e)` 方法：将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true ，如果此队列已满，则抛出 `IllegalStateException` 异常。
- `#offer(E e)` 方法：将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true ，如果此队列已满，则返回 false 。
- `#offer(E e, long timeout, TimeUnit unit)` 方法：将指定的元素插入此队列的尾部，如果该队列已满，则在到达指定的等待时间之前等待可用的空间。
- `#put(E e)` 方法：将指定的元素插入此队列的尾部，如果该队列已满，则等待可用的空间。

#### 8.3.1、add

```java
// ArrayBlockingQueue.java
@Override
public boolean add(E e) {
    return super.add(e);
}

// AbstractQueue.java
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}

```

- `#add(E e)`方法，调用 `#offer(E e)` 方法，如果返回false，则直接抛出 `IllegalStateException` 异常。

#### 8.3.2、offer

```java
@Override
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            enqueue(e);
            return true;
        }
    } finally {
        lock.unlock();
    }
}
```

- 首先，检查是否为 `null` 。
- 然后，获取 `Lock` 锁。获取锁成功后，如果队列已满则，直接返回 `false` 。
- 最后，调用 `#enqueue(E e)` 方法，**它为入列的核心方法**，所有入列的方法最终都将调用该方法，在队列尾部插入元素。

#### 8.3.3、enqueue

```java
private void enqueue(E x) {
    // 添加元素
    final Object[] items = this.items;
    items[putIndex] = x;
    // 队列已满,则重置索引为0
    if (++putIndex == items.length)
        putIndex = 0;
    // 总数+1
    count++;
    // 唤醒一个notEmpty上的等待线程(可以来队列取元素了)
    notEmpty.signal();
}
```

该方法就是在 `putIndex`（对尾）位置处，添加元素，最后调用 `notEmpty` 的 `#signal()` 方法，**通知阻塞在出列的线程**（如果队列为空，则进行出列操作是会阻塞）。

#### 8.3.4、可超时的offer

```java
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    // 获得锁；可中断的获取锁，和lock()方法不同之处在于该方法会响应中断，即在锁的获取中可以中断当前线	 //程。
    lock.lockInterruptibly();
    try {
        // <1> 若队列已满，循环等待被通知，再次检查队列是否非空
        while (count == items.length) {
            // 可等待的时间小于等于零，直接返回失败
            if (nanos <= 0)
                return false;
            // 等待，直到超时
            nanos = notFull.awaitNanos(nanos); // 返回的为剩余可等待时间，相当于每次等待，都会扣除相应已经等待的时间。
        }
        // 入队
        enqueue(e);
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}

```

- 相比 `#offer(E e)` 方法，增加了 `<1>` 处：
  - 若队列已满，调用 `notFull` 的 `#awaitNanos(long nanos)` 方法，等待被通知（元素出列时，会调用 `notFull` 的 `#signal()` 方法，进行通知阻塞等待的入列线程）**或者超时**。
  - 被通知后，再次检查队列是否非空。若非空，继续向下执行，否则继续等待被通知。

#### 8.3.5、put

```java
/**
 * 在队尾插入指定元素，如果队列已满，则阻塞线程.
 */
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();   // 加锁
    try {
        while (count == items.length)   // 队列已满。这里必须用while，防止虚假唤醒
            notFull.await();            // 在notFull队列上等待
        enqueue(e);                     // 队列未满, 直接入队
    } finally {
        lock.unlock();
    }
}
```

这里需要注意一点，队列已满的时候，是通过while循环判断的,这其实是多线程设计模式中的[Guarded Suspension模式](https://segmentfault.com/a/1190000015558585)（**当现在并不适合马上执行某个操作时，就要求想要执行该操作的线程等待**）：

```java
while (count == items.length)   // 队列已满。这里必须用while，防止虚假唤醒
    notFull.await();            // 在notFull队列上等待
```

**之所以这样做，是防止线程被意外唤醒，不经再次判断就直接调用enqueue**方法。

- 相比 `#offer(E e)` 方法，增加了 `<1>` 处：
  - 若队列已满，调用 `notFull` 的 `#await()` 方法，等待被通知（元素出列时，会调用 `notFull` 的 `#await()` 方法，进行通知阻塞等待的入列线程）。
  - 被通知后，再次检查队列是否非空。若非空，继续向下执行，否则继续等待被通知。

### 8.4、出队

`ArrayBlockingQueue` 提供的出队方法如下：

- `#poll()` 方法：获取并移除此队列的头，如果此队列为空，则返回 `null` 。
- `#poll(long timeout, TimeUnit unit)` 方法：获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
- `#take()` 方法：获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
- `#remove(Object o)` 方法：从此队列中移除指定元素的单个实例（如果存在）。如果队列为空，则抛出异常`NoSuchElementException`异常

#### 8.4.1、remove

`#remove`分为有参和无参两种方法。如果队列为空，无参方法`remove()`会抛出`NoSuchElementException`异常，有参方法`remove(Object o)`则会返回`flase`。

有参方法，返回`true`或者`flase`;

```java
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    // 获得锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            // 循环向下查找，若匹配，则进行移除。
            do {
                if (o.equals(items[i])) {
                    removeAt(i);
                    return true;
                }
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        // 释放锁
        lock.unlock();
    }
}

void removeAt(final int removeIndex) {
    // assert lock.getHoldCount() == 1;
    // assert items[removeIndex] != null;
    // assert removeIndex >= 0 && removeIndex < items.length;
    final Object[] items = this.items;
    // 移除的为队头，直接移除即可
    if (removeIndex == takeIndex) {
        // removing front item; just advance
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
    // 移除非队头，移除的同时，需要向前复制，填补这个空缺。
    } else {
        // an "interior" remove

        // slide over all others up through putIndex.
        final int putIndex = this.putIndex;
        for (int i = removeIndex;;) {
            int next = i + 1;
            if (next == items.length)
                next = 0;
            if (next != putIndex) {
                items[i] = items[next];
                i = next;
            } else {
                items[i] = null;
                this.putIndex = i;
                break;
            }
        }
        count--;
        if (itrs != null)
            itrs.removedAt(removeIndex);
    }
    // 通知
    notFull.signal();
}
```

无参方法返回队首元素：

```java
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}
```

#### 8.4.2、poll

```java
public E poll() {
    // 获得锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 获得头元素
        return (count == 0) ? null : dequeue();
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

- 如果队列为空，则返回 `null`，否则，调用 `#dequeue()` 方法，获取列头元素。

#### 8.4.3、dequeue

```java
 private E dequeue() {
    final Object[] items = this.items;
    // 去除队首元素
    E x = (E) items[takeIndex];
    items[takeIndex] = null; // 置空
    // 到达队尾，回归队头
    if (++takeIndex == items.length)
        takeIndex = 0;
    // 总数 - 1
    count--;
    // 维护下迭代器
    if (itrs != null)
        itrs.elementDequeued();
    // 通知阻塞在入列的线程
    notFull.signal();
    return x;
}

```

- 该方法主要是从列头（`takeIndex` 位置）取出元素，同时如果迭代器 `itrs` 不为 `null` ，则需要维护下该迭代器。最后，调用 `notFull` 的 `#signal()` 方法，唤醒阻塞在入列线程。

#### 8.4.4、take

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    // 获得锁
    lock.lockInterruptibly();
    try {
        // <1> 若队列已空，循环等待被通知，再次检查队列是否非空
        while (count == 0)
            notEmpty.await();
        // 出列
        return dequeue();
    } finally {
        // 解锁
        lock.unlock();
    }
}

```

-  相比 `#poll()` 方法，增加了 `<1>` 处：
  - 若队列已空，调用 `notEmpty` 的 `#await()` 方法，等待被通知（**元素入列时，会调用 `notEmpty` 的 `#signal()` 方法，进行通知阻塞等待的出列线程**）。
  - 被通知后，再次检查队列是否为空。若非空，继续向下执行，否则继续等待被通知。

#### 8.4.5、可超时的poll

```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    // 获得锁
    lock.lockInterruptibly();
    try {
        // <1> 若队列已空，循环等待被通知，再次检查队列是否非空
        while (count == 0) {
            // 可等待的时间小于等于零，直接返回 null
            if (nanos <= 0)
                return null;
            // 等待，直到超时
            nanos = notEmpty.awaitNanos(nanos); // 返回的为剩余可等待时间，相当于每次等待，都会扣除相应已经等待的时间。
        }
        // 出队
        return dequeue();
    } finally {
        // 解锁
        lock.unlock();
    }
}
```

- 相比 `#poll()` 方法，增加了 `<1>` 处：
  - 若队列已空，调用 `notEmpty` 的 `#await()` 方法，等待被通知（元素入列时，会调用 `notEmpty` 的 `#signal()` 方法，进行通知阻塞等待的出列线程）。
  - 被通知后，再次检查队列是否为空。若非空，继续向下执行，否则继续等待被通知。

### 8.5、环形对类

从上面的入队/出队操作，可以看出，`ArrayBlockingQueue`的内部数组其实是一种环形结构。

*假设`ArrayBlockingQueue`的容量大小为6，我们来看下整个入队过程：*

**①初始时**

![clipboard.png](.\img\ashfu2asdioias)

**②插入元素“9”**

![clipboard.png](.\img\fjsi2as89a0sfaksj12)

**③插入元素“2”、“10”、“25”、“93”**

![clipboard.png](.\img\aisogfoai124safsa2)

**④插入元素“90”**

注意，此时再插入一个元素“90”，则putIndex变成6，等于队列容量6，由于是循环队列，所以会将tableIndex重置为0：

![clipboard.png](.\img\asfjaoisjfoi1)

这是队列已经满了（count==6），如果再有线程尝试插入元素，并不会覆盖原有值，而是被阻塞。

------

我们再来看下出队过程：

**①出队元素“9”**

![clipboard.png](.\img\aisohfdoiasdao)

**②出队元素“2”、“10”、“25”、“93”**

![clipboard.png](.\img\asiohfaoisaoi1)

**③出队元素“90”**

注意，此时再出队一个元素“90”，则tabeIndex变成6，等于队列容量6，由于是循环队列，所以会将tableIndex重置为0：

![clipboard.png](.\img\aoihfgoiasfodias)

这是队列已经空了（count==0），如果再有线程尝试出队元素，则会被阻塞。

### 8.6、构造方法加锁问题

```java
public ArrayBlockingQueue(int capacity, boolean fair,
                          Collection<? extends E> c) {
    this(capacity, fair);

    final ReentrantLock lock = this.lock;
    // Lock only for visibility, not mutual exclusion
    // 锁的目的不是为了互斥，而是为了保证数据可见性
    lock.lock(); 
    try {
        int i = 0;
        try {
            for (E e : c) {
                checkNotNull(e);
                items[i++] = e;
            }
        } catch (ArrayIndexOutOfBoundsException ex) {
            throw new IllegalArgumentException();
        }
        count = i;
        putIndex = (i == capacity) ? 0 : i;
    } finally {
        lock.unlock();
    }
}
```

**缓存一致性**。为了解决 CPU 处理速度以及读写主存速度不一致的问题，引入了 CPU 高速缓存。虽然解决了速度问题，但是也带来了缓存一致性的问题。在不加锁的前提下，线程 A 在构造函数中 `items` 进行操作，线程 B 通过入队、出队的方式对 `items` 进行操作，这个过程对 `items` 的操作结果有可能只存在各自线程的缓存中**，并没有写入主存，这样肯定会造成数据不一致的情况。**

> 数组`item`是使用final修饰的，不存在指令重排问题。

链接：http://www.iocoder.cn/JUC/sike/ArrayBlockingQueue-construction-lock/有详细描述。

### 8.7、总结

`ArrayBlockingQueue`利用了`ReentrantLock`来保证线程的安全性，针对队列的修改都需要加全局锁。在一般的应用场景下已经足够。对于超高并发的环境，**由于生产者-消息者共用一把锁，可能出现性能瓶颈**。

另外，由于`ArrayBlockingQueue`是有界的，且在初始时指定队列大小，所以如果初始时需要限定消息队列的大小，则`ArrayBlockingQueue` 比较合适。后续，我们会介绍另一种基于单链表实现的阻塞队列——**LinkedBlockingQueue**，该队列的最大特点是使用了“两把锁”，以提升吞吐量。



## 9、LinkedBlockingQueue

### 9.1、简介

`LinkedBlockingQueue`是在JDK1.5时，随着J.U.C包引入的一种阻塞队列，它实现了`BlockingQueue`接口，底层基于**单链表**实现：

![clipboard.png](.\img\fsd244a4sdf12gy4)

`LinkedBlockingQueue`是一种**近似有界阻塞队列**，为什么说近似？因为`LinkedBlockingQueue`既可以在初始构造时就指定队列的容量，也可以不指定，如果不指定，那么它的容量大小默认为`Integer.MAX_VALUE`。

`LinkedBlockingQueue`除了底层数据结构（单链表）与`ArrayBlockingQueue`不同外，另外一个特点就是：
它维护了两把锁——`takeLock`和`putLock`。
`takeLock`用于控制出队的并发，`putLock`用于入队的并发。这也就意味着，同一时刻，只能只有一个线程能执行入队/出队操作，其余入队/出队线程会被阻塞；但是，**入队和出队之间可以并发执行，即同一时刻，可以同时有一个线程进行入队，另一个线程进行出队**，这样就可以提升吞吐量。

由于接口和`ArrayBlockingQueue`完全一样，所以`LinkedBlockingQueue`会阻塞线程的方法也一共有4个：`put(E e)`、`offer(e, time, unit)`和`take()`、`poll(time, unit)`。

> 在ArrayBlockingQueue章节中，我们说过，ArrayBlockingQueue维护了一把全局锁，无论是出队还是入队，都共用这把锁，这就导致任一时间点只有一个线程能够执行。那么对于“生产者-消费者”模式来说，意味着生产者和消费者不能并发执行。

### 9.2、构造

**LinkedBlockingQueue**提供了三种构造器：

```java
/**
 * 默认构造器.
 * 队列容量为Integer.MAX_VALUE.	
 */
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}

/**
 * 显示指定队列容量的构造器
 */
public LinkedBlockingQueue(int capacity) {
    // 初始化链长度不能小于等于0
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    // 设置头尾结点，元素为null
    last = head = new Node<E>(null);
}

/**
 * 从已有集合构造队列.
 * 队列容量为Integer.MAX_VALUE
 */
public LinkedBlockingQueue(Collection<? extends E> c) {
    this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();                     // 这里加锁仅仅是为了保证可见性
    try {
        int n = 0;
        for (E e : c) {
            if (e == null)              // 队列不能包含null元素
                throw new NullPointerException();
            if (n == capacity)          // 队列已满
                throw new IllegalStateException("Queue full");
            enqueue(new Node<E>(e));    // 队尾插入元素
            ++n;
        }
        count.set(n);                   // 设置元素个数
    } finally {
        putLock.unlock();
    }
}
```

可以看到，如果不指定容量，那么它的容量大小默认为`Integer.MAX_VALUE`。另外，**LinkedBlockingQueue**使用了一个原子变量`AtomicInteger`记录队列中元素的个数，以保证入队/出队并发修改元素时的数据一致性。

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    
    //队列容量大小，如果不指定，为Integer.MAX_VALUE
    private final int capacity;

    //队列中元素个数：(与ArrayBlockingQueue的不同)
    //出队和入队是两把锁
    private final AtomicInteger count = new AtomicInteger(0);

    //队列--头结点
    private transient Node<E> head;

    //队列--尾结点
    private transient Node<E> last;

    //与ArrayBlockingQueue的不同,两把锁
    //出队锁
    private final ReentrantLock takeLock = new ReentrantLock();

    //队列空时，出队线程在该条件队列等待
    private final Condition notEmpty = takeLock.newCondition();

    //入队锁
    private final ReentrantLock putLock = new ReentrantLock();

    //队列满时，入队线程在该条件队列等待
    private final Condition notFull = putLock.newCondition();
    
    /**
     * 链表结点定义
     */
    static class Node<E> {
        // 队列元素
        E item;

        Node<E> next;   // 后驱指针
		
        // 构造构造
        Node(E x) {
            item = x;
        }
    }

    //...
}
```

构造完成后，`LinkedBlockingQueue`的初始结构如下：

![clipboard.png](.\img\f47754s54d234)

插入部分元素后的`LinkedBlockingQueue`结构：

![clipboard.png](.\img\f5as8w5af5w5asd)

### 9.3、入队

#### 9.3.1、put

```java
/**
 * 在队尾插入指定的元素.
 * 如果队列已满，则阻塞线程.
 */
public void put(E e) throws InterruptedException {
    //不能插入为null元素：
    if (e == null) throw new NullPointerException();
    int c = -1;
    //创建元素结点：
    Node<E> node = new Node(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    //加锁，保证数据的一致性：
    putLock.lockInterruptibly();
    try {
        //当队列元素个数==链表长度
        while (count.get() == capacity) {
            //插入线程等待：
            notFull.await();
        }
        //插入元素：
        enqueue(node);
        //队列元素增加：count+1,但返回+1前的count值：
        c = count.getAndIncrement();
        // 容量还没满，唤醒生产者线程
        // (例如链表长度为5，此时第五个元素已经插入，c=4，+1=5，所以超过了队列容量，则不会再唤醒生产者线程)
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        //释放锁：
        putLock.unlock();
    }
    //当c=0时，即意味着之前的队列是空队列,消费者线程都处于等待状态，需要被唤醒进行消费
    if (c == 0)
        //唤醒消费者线程：
        signalNotEmpty();
}
```

插入元素时，首先需要获得**“入队锁”**，如果队列满了，则当前线程需要在**notFull**条件队列等待；否则，将新元素链接到队列尾部。

这里需要注意的是两个地方：

**①每入队一个元素后，如果队列还没满，则需要唤醒其它可能正在等待的“入队线程”：**

```java
/**
 * c+1 表示的元素个数.
 * 如果，则唤醒一个“入队线程”
 */
c = count.getAndIncrement();        // c表示入队前的队列元素个数
if (c + 1 < capacity)               // 入队后队列未满, 则唤醒一个“入队线程”
    notFull.signal();
```

**② 每入队一个元素，都要判断下队列是否空了，如果空了，说明可能存在正在等待的“出队线程”，需要唤醒它：**

```java
if (c == 0)                             // 队列为空, 则唤醒一个“出队线程”
    signalNotEmpty();
```

> *这里为什么不像`ArrayBlockingQueue`那样，入队完成后，直接唤醒一个在`notEmpty`上等待的出队线程？*

因为`ArrayBlockingQueue`中，入队/出队用的是同一把锁，两者不会并发执行，所以每入队一个元素（拿到锁），就可以通知可能正在等待的“出队线程”。（同一个锁的两个条件队列：**notEmpty**、**notFull**）

ArrayBlockingQueue中的**enqueue**方法：

```java
private void enqueue(E x) {
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)     // 队列已满,则重置索引为0
        putIndex = 0;
    count++;                            // 元素个数+1
    notEmpty.signal();                  // 唤醒一个notEmpty上的等待线程(可以来队列取元素了)
}
```

而`LinkedBlockingQueue`中，入队/出队用的是两把锁，入队/出队是会并发执行的。入队锁对应的是**notFull**条件队列，出队锁对应的是**notEmpty**条件队列，所以每入队一个元素，应当立即去唤醒可能阻塞的其它入队线程。当队列为空时，说明后面再来“出队线程”，一定都会阻塞，所以此时可以去唤醒一个出队线程，以提升性能。

> 试想以下，如果去掉上面的①和②，当入队线程拿到“入队锁”，入队元素后，直接尝试唤醒出队线程，会要求去拿出队锁，这样持有锁A的同时，再去尝试获取锁B，很可能引起死锁，就算通过打破死锁的条件避免死锁，**每次操作同时获取两把锁也会降低性能。**

#### 9.3.2、offer

`offer(E e)`是非阻塞式插入，队列中的元素与链表长度相同时，直接返回`false`,不会阻塞线程。

```java
//向队列尾部插入元素：返回true/false
public boolean offer(E e) {
    //插入元素不能为空
    if (e == null) throw new NullPointerException();
    final AtomicInteger count = this.count;
    //如果队列元素==链表长度，则直接返回false
    if (count.get() == capacity)
        return false;
    int c = -1;
    //创建元素结点对象：
    Node<E> node = new Node(e);
    final ReentrantLock putLock = this.putLock;
    //加锁，保证数据一致性
    putLock.lock();
    try {
        //队列元素个数 小于 链表长度
        if (count.get() < capacity) {
            //向队列中插入元素：
            enqueue(node);
            //增加队列元素个数：
            c = count.getAndIncrement();
            //容量还没满，唤醒生产者线程：
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        //释放锁：
        putLock.unlock();
    }
    //此时，代表队列中还有一条数据，可以进行消费，唤醒消费者线程
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}
```

### 9.4、出队

#### 9.4.1、take

删除元素的逻辑和插入元素类似。删除元素时，首先需要获得**“出队锁”**，如果队列为空，则当前线程需要在**notEmpty**条件队列等待；否则，从队首出队一个元素：

```java
/**
 * 从队首出队一个元素
 */
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;   // 获取“出队锁”
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {                  // 队列为空, 则阻塞线程
            notEmpty.await();
        }											
        x = dequeue();								// 从队列头部获取元素；
        c = count.getAndDecrement();                // c表示出队前的元素个数
        if (c > 1)                                  // 出队前队列非空, 则唤醒一个出队线程
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)                              // 队列初始为满，则唤醒一个入队线程
        signalNotFull();
    return x;
}
```

```java
/**
 * 队首出队一个元素.
 */
private E dequeue() {
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h;         // help GC
    head = first;
    E x = first.item;
    first.item = null;
    return x;
}
```

上面需要的注意点和插入元素一样：
**①每出队一个元素前，如果队列非空，则需要唤醒其它可能正在等待的“出队线程”：**

```java
c = count.getAndDecrement();                // c表示出队前的元素个数
if (c > 1)                                  // 出队前队列非空, 则唤醒一个出队线程
    notEmpty.signal();
```

**② 每出队一个元素，都要判断下队列是否满，如果是满的，说明可能存在正在等待的“入队线程”，需要唤醒它：**

```java
if (c == capacity)                              // 队列初始为满，则唤醒一个入队线程
    signalNotFull();
```

#### 9.4.2、poll

```java
//获取头部元素，并返回。队列为空，则直接返回null
public E poll() {
    final AtomicInteger count = this.count;
    //如果队列中还没有元素，则直接返回 null
    if (count.get() == 0)
        return null;
    E x = null;
    int c = -1;
    final ReentrantLock takeLock = this.takeLock;
    //加锁，保证数据的安全
    takeLock.lock();
    try {
        //此时在判断，队列元素是否大于0
        if (count.get() > 0) {
            //移除队头元素
            x = dequeue();
            //减少队列元素个数
            c = count.getAndDecrement();
            //此时队列中，还有1个元素，唤醒消费者线程继续执行
            if (c > 1)
                notEmpty.signal();
        }
    } finally {
        //释放锁：
        takeLock.unlock();
    }
    //队列中出现了空余元素，唤醒生产者进行生产。
    // (链表长度为5，队列在执行take前有5个元素，执行到此处时候有4个元素了，但是c的值还是5，所以会进入到if中来)
    if (c == capacity)
        signalNotFull();
    return x;
}
```

### 9.5、总结

归纳一下，**LinkedBlockingQueue**和**ArrayBlockingQueue**比较主要有以下区别：

1. 队列大小不同。`ArrayBlockingQueue`初始构造时必须指定大小，而**`LinkedBlockingQueue`构造时既可以指定大小，也可以不指定（不指定时默认为`Integer.MAX_VALUE`，近似于无界）；**
2. 底层数据结构不同。`ArrayBlockingQueue`底层采用**数组**作为数据存储容器，而`LinkedBlockingQueue`底层采用**单向链表**作为数据存储容器；
3. 两者的加锁机制不同。`ArrayBlockingQueue`使用一把**全局锁**，即入队和出队使用同一个`ReentrantLock`锁；而`LinkedBlockingQueue`进行了**锁分离**，入队使用一个`ReentrantLock`锁（`putLock`），出队使用另一个`ReentrantLock`锁（`takeLock`）；
4. `LinkedBlockingQueue`不能指定公平/非公平策略（默认都是非公平），而`ArrayBlockingQueue`可以指定策略。

**生产者消费者实现：**

```java
public class LinkedBlockingQueueTest {

    static class Apple{

        String colour;

        public Apple(String colour){
            this.colour = colour;
        }

        public String getColour() {
            return colour;
        }

        public void setColour(String colour) {
            this.colour = colour;
        }
    }

    //生产者
    static class Producer implements Runnable{

        LinkedBlockingQueue<Apple> queueProducer ;

        Apple apple;

        public Producer( LinkedBlockingQueue<Apple> queueProducer,Apple apple){
            this.queueProducer = queueProducer;
            this.apple = apple;
        }

        public void run() {
            try {
                System.out.println("生产"+apple.getColour()+"的苹果");
                queueProducer.put(apple);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }


    //消费者
    static class Consumer implements Runnable{

        LinkedBlockingQueue<Apple> queueConsumer ;

        public Consumer(LinkedBlockingQueue<Apple> queueConsumer){
            this.queueConsumer = queueConsumer;
        }

        public void run() {
            try {
                Apple apple = queueConsumer.take();
                System.out.println("消费"+apple.getColour()+"的苹果");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        LinkedBlockingQueue<Apple> queue = new LinkedBlockingQueue<Apple>();

        Apple appleRed = new Apple("红色");
        Apple appleGreen = new Apple("绿色");

        Producer producer1 = new Producer(queue,appleRed);

        Producer producer2 = new Producer(queue,appleGreen);

        Consumer consumer = new Consumer(queue);

        producer1.run();
        producer2.run();
        consumer.run();

        Thread.sleep(10000);
    }
}
```



## 10、PriorityBlockingQueue

### 10.1、简介

`PriorityBlockingQueue`，是在JDK1.5时，随着J.U.C包引入的一种阻塞队列，它实现了**BlockingQueue**接口，底层基于**堆**实现：

![clipboard.png](E:\软件开发资料\Java\Java资料\java高级\img\asdioasd12)

`PriorityBlockingQueue`是一种**无界阻塞队列**，在构造的时候可以指定队列的初始容量。具有如下特点：

1. `PriorityBlockingQueue`与之前介绍的阻塞队列最大的不同之处就是：它是一种**优先级队列**，也就是说元**素并不是以`FIFO`的方式出/入队，而是以按照权重大小的顺序出队；**
2. `PriorityBlockingQueue`是近似无界队列，它不像`ArrayBlockingQueue`那样构造时必须指定最大容量，也不像`LinkedBlockingQueue`默认最大容量为`Integer.MAX_VALUE`，而是**在初始化时可以不设置容量，而是在自动扩容期间，限制最大容量为`Integer.MAX_VALUE - 8`；**
3. 由于`PriorityBlockingQueue`是按照元素的权重进入排序，所以队列中的元素必须是可以比较的，也就是说元素必须实现`Comparable`接口；
4. 由于`PriorityBlockingQueue`无界队列，所以插入元素永远不会阻塞线程；
5. `PriorityBlockingQueue`底层是一种**基于数组实现的堆结构**。
6. 不能存储`Null`值。
7. `PriorityBlockingQueue`不能保证同优先级元素的顺序。
8. `PriorityBlockingQueue`底层采用二叉堆来实现的。

> **注意**：*堆分为“大顶堆”和“小顶堆”，`PriorityBlockingQueue`会依据元素的比较方式选择构建大顶堆或小顶堆。比如：如果元素是`Integer`这种引用类型，那么默认就是“小顶堆”，也就是每次出队都会是当前队列最小的元素。*

### 10.2、二叉树

二叉堆是一种特殊的堆，就结构性而言就是完全二叉树或者是近似完全二叉树，满足树结构性和堆序性。树机构特性就是完全二叉树应该有的结构，堆序性则是：**父节点的键值总是保持固定的序关系于任何一个子节点的键值，且每个节点的左子树和右子树都是一个二叉堆**。它有两种表现形式：最大堆、最小堆。

最大堆：**父节点的键值总是大于或等于任何一个子节点的键值**（下右图）

最小堆：**父节点的键值总是小于或等于任何一个子节点的键值**（下走图）

![201703270001](.\img\2018120827001.png)

二叉堆一般用数组表示，如果父节点的节点位置在`n`处，那么其左孩子节点为：`2 * n + 1` ，其右孩子节点为`2 * (n + 1)`，其父节点为`（n - 1） / 2` 处。上左图的数组表现形式为：

![201703270002_2](.\img\2018120827002.png)

二叉堆的基本结构了解了，下面来看看二叉堆的添加和删除节点。二叉堆的添加和删除相对于二叉树来说会简单很多。

#### 添加元素

首先将要添加的元素N插添加到堆的末尾位置（在二叉堆中我们称之为空穴）。如果元素N放入空穴中而不破坏堆的序（其值大于跟父节点值（最大堆是小于父节点）），那么插入完成。否则，我们则将该元素N的节点与其父节点进行交换，然后与其新父节点进行比较直到它的父节点不在比它小（最大堆是大）或者到达根节点。

假如有如下一个二叉堆

[![201703270003_3](.\img\2018120827003.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120827003.png)

这是一个最小堆，其父节点总是小于等于任一一个子节点。现在我们添加一个元素2。

第一步：在末尾添加一个元素2，如下：

第二步：元素2比其父节点6小，进行替换，如下：

[![201703270005_3](.\img\2018120827005.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120827005.png)
第三步：继续与其父节点5比较，小于，替换：

[![201703270006_3](.\img\2018120827006.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120827006.png)

第四步：继续比较其跟节点1，发现跟节点比自己小，则完成，到这里元素2插入完毕。所以整个添加元素过程可以概括为：在元素末尾插入元素，然后不断比较替换直到不能移动为止。

复杂度：Ο(logn)

#### 删除元素

删除元素与增加元素一样，需要维护整个二叉堆的序。删除位置1的元素（数组下标0），则把最后一个元素空出来移到最前边，然后和它的两个子节点比较，如果两个子节点中较小的节点小于该节点，就将他们交换，知道两个子节点都比该元素大为止。

就上面二叉堆而言，删除的元素为元素1。

第一步：删掉元素1，元素6空出来，如下：

第二步：与其两个子节点（元素2、元素3）比较，都小，将其中较小的元素（元素2）放入到该空穴中：

[![201703270008](.\img\2018120827008.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120827008.png)
第三步：继续比较两个子节点（元素5、元素7），还是都小，则将较小的元素（元素5）放入到该空穴中：[![201703270004_4](.\img\2018120827004.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120827004.png)[![201703270007_4](.\img\2018120827007.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120827007.png)

第四步：比较其子节点（元素8），比该节点小，则元素6放入该空穴位置不会影响二叉堆的树结构，放入：

[![201703270010_3](.\img\20181208270010.png)](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/20181208270010.png)
到这里整个删除操作就已经完成了。

### 10.3、构造

PriorityBlockingQueue提供了四种构造器：

```java
/**
 * 默认构造器.
 * 默认初始容量11, 以元素自然顺序比较(元素必须实现Comparable接口)
 */
public PriorityBlockingQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

/**
 * 指定初始容量的构造器.
 * 以元素自然顺序比较(元素必须实现Comparable接口)
 */
public PriorityBlockingQueue(int initialCapacity) {
    this(initialCapacity, null);
}

/**
 * 指定初始容量和比较器的构造器.
 */
public PriorityBlockingQueue(int initialCapacity,
                             Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    this.comparator = comparator;
    this.queue = new Object[initialCapacity];
}

/**
 * 从已有集合构造队列.
 * 如果已经集合是SortedSet或者PriorityBlockingQueue, 则保持原来的元素顺序
 */
public PriorityBlockingQueue(Collection<? extends E> c) {
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    boolean heapify = true;     // true if not known to be in heap order
    boolean screen = true;      // true if must screen for nulls
 
    if (c instanceof SortedSet<?>) {                        // 如果是有序集合
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        heapify = false;
    } else if (c instanceof PriorityBlockingQueue<?>) {     // 如果是优先级队列
        PriorityBlockingQueue<? extends E> pq = (PriorityBlockingQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        screen = false;
        if (pq.getClass() == PriorityBlockingQueue.class)   // exact match
            heapify = false;
    }
 
    Object[] a = c.toArray();
    int n = a.length;
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, n, Object[].class);
    if (screen && (n == 1 || this.comparator != null)) {    // 校验是否存在null元素
        for (int i = 0; i < n; ++i)
            if (a[i] == null)
                throw new NullPointerException();
    }
    this.queue = a;
    this.size = n;
    if (heapify)    // 堆排序
        heapify();
}
```

重点是第三种构造器，可以看到，`PriorityBlockingQueue`内部也是利用了**`ReentrantLock`**来保证并发访问时的线程安全。
`PriorityBlockingQueue`如果不指定容量，默认容量为11，内部数组`queue`其实是一种二叉树。

需要注意的是，`PriorityBlockingQueue`只有一个条件等待队列——`notEmpty`，因为构造时不会限制最大容量且会自动扩容，所以插入元素并不会阻塞，仅当队列为空时，才可能阻塞“出队”线程。

```java
	// 默认容量
    private static final int DEFAULT_INITIAL_CAPACITY = 11;

    // 最大容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    // 二叉堆数组
    private transient Object[] queue;

    // 队列元素的个数
    private transient int size;

    // 比较器，如果为空，则为自然顺序
    private transient Comparator<? super E> comparator;

    // 内部锁
    private final ReentrantLock lock;

	// 当队列为空时，出队线程在该条件队列上等待.
    private final Condition notEmpty;

    //
    private transient volatile int allocationSpinLock;

    // 优先队列：主要用于序列化，这是为了兼容之前的版本。只有在序列化和反序列化才非空
    private PriorityQueue<E> q;

```

### 10.4、入队

#### 10.4.1、插入元素—put

**PriorityBlockingQueue**插入元素不会阻塞线程，`put(E e)`方法内部其实是调用了`offer(E e)`方法：
首先获取全局锁（**对于队列的修改都要获取这把锁**），然后判断下队列是否已经满了，**如果满了就先进行一次内部数组的扩容**：

```java
/**
 * 向队列中插入指定元素.
 * 由于队列是无界的，所以不会阻塞线程.
 */
public void put(E e) {
    offer(e);   // never need to block
}
 
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
 
    final ReentrantLock lock = this.lock;   // 加锁
    lock.lock();
 
    int n, cap;
    Object[] array;
    while ((n = size) >= (cap = (array = queue).length))    // 队列已满, 则进行扩容
        tryGrow(array, cap);
 
    try {
        Comparator<? super E> cmp = comparator;
        if (cmp == null)    // 比较器为空, 则按照元素的自然顺序进行堆调整
            siftUpComparable(n, e, array);
        else                // 比较器非空, 则按照比较器进行堆调整
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;       // 队列元素总数+1
        notEmpty.signal();  // 唤醒一个可能正在等待的"出队线程"
    } finally {
        lock.unlock();
    }
    return true;
}
```

上面最关键的是**siftUpComparable**和**siftUpUsingComparator**方法，这两个方法内部几乎一样，只不过前者是一个根据元素的自然顺序比较，后者则根据外部比较器比较，我们重点看下siftUpComparable方法：

```java
/**
 * 将元素x插入到array[k]的位置.
 * 然后按照元素的自然顺序进行堆调整——"上浮"，以维持"堆"有序.
 * 最终的结果是一个"小顶堆".
 */
private static <T> void siftUpComparable(int k, T x, Object[] array) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;     // 相当于(k-1)除2, 就是求k结点的父结点索引parent
        Object e = array[parent];
        if (key.compareTo((T) e) >= 0)  // 如果插入的结点值大于父结点, 则退出
            break;
 
        // 否则，交换父结点和当前结点的值
        array[k] = e;
        k = parent;
    }
    array[k] = key;
}
```

**siftUpComparable**方法的作用其实就是**堆的“上浮调整”**，可以把堆可以想象成一棵完全二叉树，每次插入元素都链接到二叉树的**最右下方**，然后**将插入的元素与其父结点比较，如果父结点大，则交换元素，直到没有父结点比插入的结点大为止。这样就保证了堆顶（二叉树的根结点）一定是最小的元素**。（注：以上仅针对“小顶堆”）

#### 10.4.2、堆的“上浮”调整

我们通过示例来理解下入队的整个过程：***假设初始构造的队列大小为6，依次插入9、2、93、10、25、90\***。

**①初始队列情况**

![clipboard.png](.\img\fsdf32q231ref)

------

**②插入元素9（索引0处）**

![clipboard.png](.\img\15sdf5d5fwe3)

将上述数组想象成一棵**完全二叉树**，其实就是下面的结构：
![clipboard.png](.\img\545sdf32445)

------

**③插入元素2（索引1处）**

![clipboard.png](.\img\s5d6f1456sd4f6sd)

对应的二叉树：
![clipboard.png](.\img\65d4g6d5fgdfg1)

由于结点2的父结点为9，所以要进行“上浮调整”，最终队列结构如下：
![clipboard.png](.\img\df4352221fg)

![clipboard.png](.\img\455sdfd623445fdhgfgh)

------

**④插入元素93（索引2处）**

![clipboard.png](.\img\4s6d4f324123234)

![clipboard.png](.\img\fg545465232sdf4fgh)

------

**⑤插入元素10（索引3处）**

![clipboard.png](.\img\sdf54235614234hh)

![clipboard.png](.\img\sd56fg4d23r1dfga35)

------

**⑥插入元素25（索引4处）**

![clipboard.png](.\img\sd54f52434defgj)

![clipboard.png](.\img\454dgh1dfh32158)

------

**⑦插入元素90（索引5处）**

![clipboard.png](.\img\465as4d65as123)

![clipboard.png](.\img\42ph3544235sdgdfg)

此时，堆不满足有序条件，因为“90”的父结点“93”大于它，所以需要“上浮调整”：

![clipboard.png](.\img\sdf32542sdsgh)

![clipboard.png](.\img\sdf1423412)

最终，堆的结构如上，可以看到，经过调整后，堆顶元素一定是最小的。

#### 10.4.3、扩容

在入队过程中，如果队列内部的`queue`数组已经满了，就需要进行扩容：

```java
public boolean offer(E e) {
 
    // ...
    
    while ((n = size) >= (cap = (array = queue).length))    // 队列已满, 则进行扩容
        tryGrow(array, cap);
 
    // ...
}

private void tryGrow(Object[] array, int oldCap) {
    lock.unlock();  // 扩容和入队/出队可以同时进行, 所以先释放全局锁
    Object[] newArray = null;
    // CAS 占用
    if (allocationSpinLock == 0 &&
            UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset,
                    0, 1)) {    // allocationSpinLock置1表示正在扩容
        try {
            // 计算新的数组大小
            int newCap = oldCap + ((oldCap < 64) ?
                    (oldCap + 2) :
                    (oldCap >> 1));
            if (newCap - MAX_ARRAY_SIZE > 0) {    // 溢出判断
                int minCap = oldCap + 1;
                if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
                    throw new OutOfMemoryError();
                newCap = MAX_ARRAY_SIZE;          // 最大容量，Integer.MAX_VALUE-8;
            }
            if (newCap > oldCap && queue == array)
                newArray = new Object[newCap];  // 分配新数组
        } finally {
            allocationSpinLock = 0;		// 扩容后allocationSpinLock = 0 代表释放了自旋锁
        }
    }
  	// 到这里如果是本线程扩容newArray肯定是不为null，为null就是其他线程在处理扩容，那就让给别的线程处理
    if (newArray == null) 
        Thread.yield();
    
    lock.lock();            // 获取全局锁(因为要修改内部数组queue)
    if (newArray != null && queue == array) {
        queue = newArray;   // 指向新的内部数组
        //数组复制方法
        System.arraycopy(array, 0, newArray, 0, oldCap);
    }
}
```

由于调用**tryGrow**的方法一定会先获取全局锁，所以先释放锁，因为可能有线程正在出队，扩容/出队是可以并发执行的（扩容的前半部分只是新建一个内部数组，不会对出队产生影响）。扩容后的内部数组大小一般为原来的2倍。

上述需要注意的是`allocationSpinLock`字段，该字段通过CAS操作，置1表示有线程正在进行扩容。

### 10.5、出队

#### 10.5.1、删除元素—take

删除元素（出队）的整个过程比较简单，也是先获取全局锁，然后判断队列状态，如果是空，则阻塞线程，否则调用`dequeue`方法出队：

```java
/**
 * 出队一个元素.
 * 如果队列为空, 则阻塞线程.
 */
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();   // 获取全局锁
    E result;
    try {
        while ((result = dequeue()) == null)    // 队列为空
            notEmpty.await();                   // 线程在noEmpty条件队列等待
    } finally {
        lock.unlock();
    }
    return result;
}
 
private E dequeue() {
    int n = size - 1;   // n表示出队后的剩余元素个数
    if (n < 0)          // 队列为空, 则返回null
        return null;
    else {
        Object[] array = queue;
        E result = (E) array[0];    // array[0]是堆顶结点, 每次出队都删除堆顶结点
        E x = (E) array[n];         // array[n]是堆的最后一个结点, 也就是二叉树的最右下结点
        array[n] = null;
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftDownComparable(0, x, array, n);
        else
            siftDownUsingComparator(0, x, array, n, cmp);
        size = n;
        return result;
    }
}
```

从**dequeue**方法可以看出，每次出队的元素都是“堆顶结点”，对于“小顶堆”就是队列中的最小值，对于“大顶堆”就是队列中的最大值。

**siftDownComparable**：

```java
*
 * @param k     待删除的位置
 * @param x     待比较的健
 * @param array 堆数组
 * @param n     堆的大小
 */
private static <T> void siftDownComparable(int k, T x, Object[] array, int n) {
    if (n > 0) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        int half = n >>> 1;           // 相当于n除2, 即找到索引n对应结点的父结点
        while (k < half) {
            /**
             * 下述代码中:
             * c保存k的左右子结点中的较小结点值 
             * child保存较小结点对应的索引
             */
            int child = (k << 1) + 1; // k的左子结点
            Object c = array[child];
 
            int right = child + 1;    // k的右子结点
            if (right < n && ((Comparable<? super T>) c).compareTo((T) array[right]) > 0)
                c = array[child = right];
            
            if (key.compareTo((T) c) <= 0)
                break;
            array[k] = c;
            k = child;
        }
        array[k] = key;
    }
}
```

上述代码其实是经典的**堆“下沉”**操作，对堆中某个顶点下沉，步骤如下：

1. 找到该顶点的左右子结点中较小的那个；
2. 与当前结点交换；
3. 重复前2步直到当前结点没有左右子结点或比左右子结点都小。

#### 10.5.2、堆的“下沉”调整

来看个示例，假设堆的初始结构如下，现在出队一个元素（索引0位置的元素2）。

**①初始状态**

![clipboard.png](.\img\88sd12ret4354der32g)

对应二叉树结构：

![clipboard.png](.\img\d4b564df6g4df)

------

**②将顶点与最后一个结点调换**

即将顶点“2”与最后一个结点“93”交换，然后将索引5为止置null。

![clipboard.png](.\img\564fd56g4dfg44)

> **注意： **为了提升效率（比如siftDownComparable的源码所示）并不一定要真正交换，可以用一个变量保存索引5处的结点值，在整个下沉操作完成后再替换。但是为了理解这一过程，示例图中全是以交换进行的。

------

**③下沉索引0处结点**

比较元素“93”和左右子结点中的最小者，发现“93”大于“9”，违反了“小顶堆”的规则，所以交换“93”和“9”，这一过程称为**siftdown（下沉）**：

![clipboard.png](.\img\498s4sd5f46sd5f)

------

**④继续下沉索引1处结点**

比较元素“93”和左右子结点中的最小者，发现“93”大于“10”，违反了“小顶堆”的规则，所以交换“93”和“10”：

![clipboard.png](.\img\rthrt654654)

------

**⑤比较结束**

由于“93”已经没有左右子结点了，所以下沉结束，可以看到，此时堆恢复了有序状态，最终队列结构如下：

![clipboard.png](.\img\s4df654346523)

### 10.6、总结

**PriorityBlockingQueue**属于比较特殊的阻塞队列，适用于有元素优先级要求的场景。它的内部和`ArrayBlockingQueue`一样，使用一个了全局独占锁来控制同时只有一个线程可以进行入队和出队，另外由于该队列是无界队列，所以入队线程并不会阻塞。

`PriorityBlockingQueue`始终保证出队的元素是优先级最高的元素，并且可以定制优先级的规则，内部通过使用**堆（数组形式）**来维护元素顺序，它的内部数组是可扩容的，扩容和出/入队可以并发进行。



## 11、DelayQueue

### 11.1、简介

`DelayQueue`是JDK1.5时，随着J.U.C包一起引入的一种阻塞队列，它实现了BlockingQueue接口，底层基于已有的**PriorityBlockingQueue**实现：

![clipboard.png](.\img\asdasooi12j3io)

`DelayQueue`也是一种比较特殊的阻塞队列，从类声明也可以看出，DelayQueue中的所有元素必须实现`Delayed`接口：

```java
/**
 * 一种混合风格的接口，用来标记那些应该在给定延迟时间之后执行的对象。
 * <p>
 * 此接口的实现必须定义一个 compareTo 方法，该方法提供与此接口的 getDelay 方法一致的排序。
 */
public interface Delayed extends Comparable<Delayed> {

    /**
     * 返回与此对象相关的剩余有效时间，以给定的时间单位表示.
     */
    long getDelay(TimeUnit unit);
}
```

可以看到，`Delayed`接口除了自身的`getDelay`方法外，还实现了**Comparable**接口。**`getDelay`方法用于返回对象的剩余有效时间，实现Comparable接口则是为了能够根据剩余时间比较两个对象，以便排序**。

也就是说，如果一个类实现了Delayed接口，当创建该类的对象并添加到`DelayQueue`中后，**只有当该对象的getDalay方法返回的剩余时间≤0时才会出队**。

另外，由于`DelayQueue`内部委托了`PriorityBlockingQueue`对象来实现所有方法，所以能以堆的结构维护元素顺序。**元素是否在堆顶，依据的是比较器的具体实现，跟 `getDelayed()` 的剩余时间没关系，可能根节点的剩余时间还未`<=0`，其子节点已经`<=0`了，这就导致出队线程会一直等待根节点剩余时间`<=0`；**

**DelayQueue的特点简要概括如下**：

1. `DelayQueue`是一个**使用优先级队列实现的无界阻塞队列**。
2. `DelayQueue`是一个支持 **延时获取元素** 的 **无界** 阻塞队列
3. 队列中的**元素必须实现Delayed接口，元素过期后才会从队列中取走**；
4. 定义了一个可重入锁`ReentrantLock`
5. 用于阻塞和通知的`Condition`对象：`available`
6. 根据Delay时间排序的优先级队列：`PriorityQueue`
7. 用于优化阻塞通知的线程元素`leader`

### 11.2、总结

**DelayQueue**是阻塞队列中非常有用的一种队列，经常被用于缓存或定时任务等的设计。

*考虑一种使用场景：*

> 异步通知的重试，在很多系统中，当用户完成服务调用后，系统有时需要将结果异步通知到用户的某个URI。由于网络等原因，很多时候会通知失败，这个时候就需要一种重试机制。

这时可以用`DelayQueue`保存通知失败的请求，失效时间可以根据已通知的次数来设定（比如：2s、5s、10s、20s），这样每次从队列中take获取的就是剩余时间最短的请求，如果已重复通知次数超过一定阈值，则可以把消息抛弃。

可以将`DelayQueue`运用在以下应用场景:

- **缓存系统的设计**：可以用`DelayQueue`保存缓存元素的有效期，使用一个线程循环查询`DelayQueue`，一旦能从`DelayQueue`中获取元素时，表示缓存有效期到了。
- **定时任务调度**：使用`DelayQueue`保存当天将会执行的任务和执行时间，一旦从`DelayQueue`中获取到任务就开始执行，比如`TimerQueue`就是使用`DelayQueue`实现的。

https://segmentfault.com/a/1190000016388106#item-4

http://www.iocoder.cn/JUC/sike/DelayQueue/



## 12、SynchronousQueue

### 12.1、简介

`SynchronousQueue`是JDK1.5时，随着J.U.C包一起引入的一种阻塞队列，它实现了`BlockingQueue`接口，底层基于**栈**和**队列**实现：

![clipboard.png](.\img\2770671798-5b97b70bb6e76_articlex.png)

没有看错，`SynchronousQueue`的底层实现包含两种数据结构——**栈**和**队列**。**这是一种非常特殊的阻塞队列，其互斥锁只用于执行构造方法时的数据可见性，在入队与出队等操作中是采用`TransferQueue`的无锁算法实现的并发安全**；它的特点简要概括如下：

1. `SynchronousQueue`没有容量。与其他`BlockingQueue`不同，`SynchronousQueue`是一个不存储元素的`BlockingQueue`。**每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然**。
2. 因为没有容量，所以对应 peek, contains, clear, isEmpty … 等方法其实是无效的。例如clear是不执行任何操作的，contains始终返回false,peek始终返回null。
3. `SynchronousQueue`支持公平/非公平策略。其中非公平模式，基于内部数据结构——“栈”来实现，公平模式，基于内部数据结构——“队列”来实现；
4. SynchronousQueue基于一种名为“[Dual stack and Dual queue](http://www.cs.rochester.edu/research/synchronization/pseudocode/duals.html)”的无锁算法实现。

SynchronousQueue非常适合做交换工作，生产者的线程和消费者的线程同步以传递某些信息、事件或者任务。

### 12.2、总结

`**TransferQueue`主要用于线程之间的数据交换，由于采用无锁算法，其性能一般比单纯的其它阻塞队列要高**。它的最大特点是不存储实际元素，而是在内部通过栈或队列结构保存阻塞线程。

https://segmentfault.com/a/1190000016359551#item-2-6

http://www.iocoder.cn/JUC/sike/SynchronousQueue/



## 13、LinkedBlockingDeque

### 13.1、简介

`LinkedBlockingDeque`和[ConcurrentLinkedDeque](https://segmentfault.com/a/1190000016284649)类似，都是一种**双端队列**的结构，只不过LinkedBlockingDeque同时也是一种阻塞队列，它是在JDK1.5时随着J.U.C包引入的，实现了`BlockingDueue`接口，底层基于**双链表**实现：

![clipboard.png](.\img\f2392hjf21)

> **注意：**LinkedBlockingDeque底层利用ReentrantLock实现同步，并不像ConcurrentLinkedDeque那样采用无锁算法（CAS+自旋）。

另外，LinkedBlockingDeque是一种**近似有界阻塞队列**，为什么说近似？因为LinkedBlockingDeque既可以在初始构造时就指定队列的容量，也可以不指定，如果不指定，那么它的容量大小默认为`Integer.MAX_VALUE`。

#### BlockingDeque接口

目前阻塞队列都是实现了[BlockingQueue](https://segmentfault.com/a/1190000016296278)接口。和普通双端队列接口——[Deque](https://segmentfault.com/a/1190000016284649)一样，J.U.C中也有一种阻塞的双端队列接口——`BlockingDeque`。BlockingDeque是JDK1.6时，J.U.C包新增的一个接口：
![clipboard.png](.\img\5656sds6512gt)

***BlockingDeque的类继承关系图：\***
![clipboard.png](.\img\654s56df4s56d4f65s)

我们知道，`BlockingQueue`中阻塞方法一共有4个：`put(e)`、`take()`；`offer(e, time, unit)`、`poll(time, unit)`，忽略限时等待的阻塞方法，一共就两个：

队尾入队：`put(e)`
队首出队：`take()`

BlockingDeque相对于BlockingQueue，最大的特点就是增加了在**队首入队**/**队尾出队**的阻塞方法。下面是两个接口的比较：

| 阻塞方法 | BlockingQueue | BlockingDeque |
| -------- | ------------- | ------------- |
| 队首入队 | /             | putFirst(e)   |
| 队首出队 | take()        | takeFirst()   |
| 队尾入队 | put(e)        | putLast(e)    |
| 队尾出队 | /             | takeLast()    |

### 13.2、构造

**LinkedBlockingDeque**一共三种构造器，不指定容量时，默认为`Integer.MAX_VALUE`：

```java
/**
 * 默认构造器.
 */
public LinkedBlockingDeque() {
    this(Integer.MAX_VALUE);
}

/**
 * 指定容量的构造器.
 */
public LinkedBlockingDeque(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
}

/**
 * 从已有集合构造队列.
 */
public LinkedBlockingDeque(Collection<? extends E> c) {
    this(Integer.MAX_VALUE);
    final ReentrantLock lock = this.lock;
    lock.lock(); // Never contended, but necessary for visibility
    try {
        for (E e : c) {
            if (e == null)
                throw new NullPointerException();
            if (!linkLast(new Node<E>(e)))
                throw new IllegalStateException("Deque full");
        }
    } finally {
        lock.unlock();
    }
}
```

#### 13.2.1、内部结构

`LinkedBlockingDeque`内部是**双链表**的结构，结点`Node`的定义如下：

```java
/**
 * 双链表结点定义
 */
static final class Node<E> {
    /**
     * 结点值, null表示该结点已被移除.
     */
    E item;

    /**
     * 前驱结点指针.
     */
    Node<E> prev;

    /**
     * 后驱结点指针.
     */
    Node<E> next;

    Node(E x) {
        item = x;
    }
}
```

字段`first`指向队首结点，字段last指向队尾结点。另外LinkedBlockingDeque利用**ReentrantLock**来保证线程安全，所有对队列的修改操作都需要先获取这把全局锁：

```java
public class LinkedBlockingDeque<E> extends AbstractQueue<E>
    implements BlockingDeque<E>, java.io.Serializable {

    /**
     * 队首结点指针.
     */
    transient Node<E> first;

    /**
     * 队尾结点指针.
     */
    transient Node<E> last;

    /**
     * 大小，双向链表中当前节点个数
     */
    private transient int count;

    /**
     * 容量，在创建LinkedBlockingDeque时指定的
     */
    private final int capacity;

    /**
     * 全局锁
     */
    final ReentrantLock lock = new ReentrantLock();

    /**
     * 出队线程条件队列（队列为空时，出队线程在此等待）
     */
    private final Condition notEmpty = lock.newCondition();

    /**
     * 入队线程条件队列（队列为满时，入队线程在此等待）
     */
    private final Condition notFull = lock.newCondition();

    //...
}
```

通过上面的`Lock`可以看出，`LinkedBlockingDeque`底层实现机制与`LinkedBlockingQueue`一样，依然是通过互斥锁`ReentrantLock` 来实现，`notEmpty` 、`notFull` 两个`Condition`做协调生产者、消费者问题。

### 13.3、入队

#### 13.3.1、队尾入队——put

```java
/**
 * 在队尾入队元素e.
 * 如果队列已满, 则阻塞线程.
 */
public void put(E e) throws InterruptedException {
    putLast(e);
}

public void putLast(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();    // 队列不能包含null元素
    Node<E> node = new Node<E>(e);                      // 创建入队结点
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        while (!linkLast(node))                         // 队列已满, 则阻塞线程
            notFull.await();
    } finally {
        lock.unlock();
    }
}


```

put方法内部调用了`putLast`方法，这是`Deque`接口独有的方法。上述入队操作的关键是**`linkLast`**方法：

```java
/**
 * 将结点node链接到队尾, 如果失败, 则返回false.
 */
private boolean linkLast(Node<E> node) {
    // assert lock.isHeldByCurrentThread();
    if (count >= capacity)  // 队列已满, 直接返回
        return false;

   	// 将Node的前驱指向原本的last
    Node<E> l = last;
    node.prev = l;
    last = node;
    // 首节点为null，则设置node为first
    if (first == null)
        first = node;
    else
        // 非null，说明之前的last有值，就将之前的last的next指向node
        l.next = node;

    ++count;            // 队列元素加1
    notEmpty.signal();  // 唤醒一个等待的出队线程
    return true;
}
```

**linkLast**方法在队尾插入一个结点，插入失败（队列已满的情况）则返回false。插入成功，则唤醒一个正在等待的出队线程：

**初始：**
![clipboard.png](.\img\56gh4j56gh46j5gh465)

**队尾插入结点node：**
![clipboard.png](.\img\156gh14j65gh14j65gh)

#### 13.3.2、队首入队——putFirst

队首入队就是双链表的**“头插法”**插入一个结点，如果队列已满，则阻塞调用线程：

```java
/**
 * 在队首入队元素e.
 * 如果队列已满, 则阻塞线程.
 */
public void putFirst(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        while (!linkFirst(node))        // 队列已满, 则阻塞线程
            notFull.await();
    } finally {
        lock.unlock();
    }
}
```

先获取锁，然后调用`linkFirst`方法入列，最后释放锁。如果队列是满的则在`notFull`上面等待。`linkFirst`设置`Node`为队头：

```java
/**
 * 在队首插入一个结点, 插入失败则返回null.
 */
private boolean linkFirst(Node<E> node) {
    // assert lock.isHeldByCurrentThread();
    if (count >= capacity)      // 队列已满
        return false;

    // 首节点
    Node<E> f = first;
    // 新节点的next指向原first
    node.next = f;
    // 设置node为新的first
    first = node;
    
    // 没有尾节点，设置node为尾节点
    if (last == null)
        last = node;
    // 有尾节点，那就将之前first的pre指向新增node
    else
        f.prev = node;
    ++count;
    // 唤醒notEmpty
    notEmpty.signal();
    return true;
}
```

***初始：\***
![clipboard.png](.\img\sdjfoisdjfiosdoi)

***队首插入结点node：\***
![clipboard.png](.\img\6sdifis534dfjds231)

### 13.4、出队

#### 13.4.1、队首出队——take

队首出队的逻辑很简单，如果队列为空，则阻塞调用线程：

```java
/**
 * 从队首出队一个元素, 如果队列为空, 则阻塞线程.
 */
public E take() throws InterruptedException {
    return takeFirst();
}

public E takeFirst() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E x;
        while ((x = unlinkFirst()) == null)     // 队列为空, 则阻塞线程
            notEmpty.await();
        return x;
    } finally {
        lock.unlock();
    }
}
```

实际的出队由**unlinkFirst**方法执行：

```java
/**
 * 从队首删除一个元素, 失败则返回null.
 */
private E unlinkFirst() {
    // assert lock.isHeldByCurrentThread();
    Node<E> f = first;
    if (f == null)          // 队列为空
        return null;

    // 以下是双链表的头部删除过程
    Node<E> n = f.next;
    E item = f.item;
    f.item = null;
    f.next = f;             // help GC
    first = n;
    if (n == null)
        last = null;
    else
        n.prev = null;

    --count;                // 队列元素个数减1
    notFull.signal();       // 唤醒一个等待的入队线程
    return item;
}
```

***初始：\***
![clipboard.png](.\img\456a4sd56as)

***删除队首结点：\***
![clipboard.png](.\img\4654asd21hg5848)

#### 13.4.2、队尾出队——takeLast

队尾出队的逻辑很简单，如果队列为空，则阻塞调用线程：

```java
/**
 * 从队尾出队一个元素, 如果队列为空, 则阻塞线程.
 */
public E takeLast() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E x;
        while ((x = unlinkLast()) == null)  // 队列为空, 阻塞线程
            notEmpty.await();
        return x;
    } finally {
        lock.unlock();
    }
}
```

实际的出队由**unlinkLast**方法执行：

```java
/**
 * 删除队尾元素, 如果失败, 则返回null.
 */
private E unlinkLast() {
    // assert lock.isHeldByCurrentThread();
    Node<E> l = last;
    if (l == null)          // 队列为空
        return null;

    // 以下为双链表的尾部删除过程
    Node<E> p = l.prev;
    E item = l.item;
    l.item = null;
    l.prev = l;             // help GC
    last = p;
    if (p == null)
        first = null;
    else
        p.next = null;

    --count;                // 队列元素个数减1
    notFull.signal();       // 唤醒一个等待的入队线程
    return item;
}
```

***初始：\***
![clipboard.png](.\img\546sdfs2sd5)

***删除队尾结点：\***
![clipboard.png](.\img\615426asd54652)

### 13.5、总结

LinkedBlockingDeque作为一种阻塞双端队列，**提供了队尾删除元素和队首插入元素的阻塞方法。该类在构造时一般需要指定容量，如果不指定，则最大容量为`Integer.MAX_VALUE`**。另外，由于内部通过ReentrantLock来保证线程安全，所以LinkedBlockingDeque的整体实现时比较简单的。

另外，双端队列相比普通队列，**主要是多了【队尾出队元素】/【队首入队元素】的功能**。
阻塞队列我们知道一般用于“生产者-消费者”模式，而双端阻塞队列在“生产者-消费者”就可以利用“双端”的特性，从队尾出队元素。

考虑下面这样一种场景：有多个消费者，每个消费者有自己的一个消息队列，生产者不断的生产数据扔到队列中，消费者消费数据有快又慢。为了提升效率，速度快的消费者可以从其它消费者队列的**队尾**出队元素放到自己的消息队列中，由于是从其它队列的队尾出队，这样可以减少并发冲突（其它消费者从队首出队元素），又能提升整个系统的吞吐量。这其实是一种“**工作窃取算法**”的思路。



## 14、LinkedTransferQueue

### 14.1、简介

前面提到的各种`BlockingQueue`对读或者写都是锁上整个队列，在并发量大的时候，各种锁是比较耗资源和耗时间的，而前面的`SynchronousQueue`虽然不会锁住整个队列，但它是一个没有容量的“队列”，那么有没有这样一种队列，它即可以像其他的`BlockingQueue`一样有容量又可以像`SynchronousQueue`一样不会锁住整个队列呢？有！答案就是`LinkedTransferQueue`。

`LinkedTransferQueue`采用一种**预占模式**。什么意思呢？有就直接拿走，没有就占着这个位置直到拿到或者超时或者中断。**即消费者线程到队列中取元素时，如果发现队列为空，则会生成一个null节点，然后park住等待生产者。后面如果生产者线程入队时发现有一个null元素节点，这时生产者就不会入列了，直接将元素填充到该节点上，唤醒该节点的线程，被唤醒的消费者线程拿东西走人**。

`LinkedTransferQueue`是在JDK1.7时，J.U.C包新增的一种比较特殊的阻塞队列，它除了具备阻塞队列的常用功能外，还有一个比较特殊的`transfer`方法。

![clipboard.png](.\img\45f213h5621fg)

我们知道，在普通阻塞队列中，当队列为空时，消费者线程（调用**take**或**poll**方法的线程）一般会阻塞等待生产者线程往队列中存入元素。而**LinkedTransferQueue**的**transfer**方法则比较特殊：

1. 当有消费者线程阻塞等待时，调用`transfer`方法的生产者线程不会将元素存入队列，而是直接将元素传递给消费者；
2. 如果调用`transfer`方法的生产者线程发现没有正在等待的消费者线程，则会将元素入队，然后会阻塞等待，直到有一个消费者线程来获取该元素。



# 三、原子操作类

## 1、简介

当程序更新一个变量时，如果多个线程同时更新这变量，可能得到期望之外的值。

比如变量 `i = 1`，`A` 线程更新 `i+1`，`B` 线程也更新`i+1`，经过两个线程操作之后可能 `i` 不等于 `3`，而是等于 `2` 。

因为 `A` 和 `B` 线程在更新变量 `i` 的时候拿到的 `i` 都是 `1`，这就是 **线程不安全的更新操作**，通常我们会使用 `synchronized` 来解决这个问题，`synchronized` 会保证多线程不会同时更新变量 `i`。

而 Java 从 `JDK 1.5` 开始提供了 `java.util.concurrent.atomic` 包（以下简称`Atomic包`），这个包中的 **原子操作类** 提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。

因为变量的类型有很多种，所以在 `Atomic` 包里一共提供了 `12个` 类，属于以下 `4` 种类型的原子更新方式：

- 原子更新基本类型。
  - `AtomicBoolean`：原子更新布尔类型。
  - `AtomicInteger`：原子更新整型。
  - `AtomicLong`：原子更新长整型。
- 原子更新数组。
  - `AtomicIntegerArray`：原子更新整型数组里的元素。
  - `AtomicLongArray`：原子更新长整型数组里的元素。
  - `AtomicReferenceArray`：原子更新引用类型数组里的元素。
- 原子更新引用。
  - `AtomicReference`：原子更新对象引用。
  - `AtomicMarkableReference`：原子更新带有标记位的对象引用。
  - `AtomicStampedReference`：原子更新带有版本号的对象引用。
- 原子更新属性（字段）。
  - `AtomicIntegerFieldUpdater`：原子更新`volatile`修饰的整型的字段的更新器。
  - `AtomicLongFieldUpdater`：原子更新`volatile`修饰的长整型字段的更新器。
  - `AtomicReferenceFieldUpdater`：原子更新`volatile`修饰的引用类型里的字段的更新器。

`Atomic` 包里的类基本都是使用 `Unsafe` 实现的包装类。

## 2、原子更新基本类型

### 2.1、AtomicInteger

#### 2.1.1、创建AtomicInteger对象

先来看下AtomicInteger对象的创建。

AtomicInteger提供了两个构造器，使用默认构造器时，内部int类型的value值为0：
`AtomicInteger atomicInt = new AtomicInteger();`

AtomicInteger类的内部并不复杂，所有的操作都针对内部的int值——value，并通过`Unsafe`类来实现线程安全的`CAS`操作：
![clipboard.png](.\img\s56df45s6d4f)

#### 2.1.2、AtomicInteger的使用

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        AtomicInteger ai = new AtomicInteger();
		Accumlator accumlator = new Accumlator(ai);
        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Thread t = new Thread(accumlator, "thread-" + i);
            list.add(t);
            t.start();
        }

        for (Thread t : list) {
            t.join();
        }

        System.out.println(ai.get());
    }

    static class Accumlator implements Runnable {
        private AtomicInteger ai;

        Accumlator(AtomicInteger ai) {
            this.ai = ai;
        }

        @Override
        public void run() {
            for (int i = 0, len = 1000; i < len; i++) {
                ai.incrementAndGet();
            }
        }
    }
}
```

上述代码使用了AtomicInteger的**incrementAndGet**方法，以原子的操作对int值进行自增，该段程序执行的最终结果为10000（10个线程，每个线程对AtomicInteger增加1000），如果不使用AtomicInteger，使用原始的int或Integer，最终结果值可能会小于10000（并发时读到了过时的数据或存在值覆盖的问题）。

我们来看下**incrementAndGet**内部：
![clipboard.png](.\img\56as4fsdf8s4d892)

内部调用了Unsafe类的**getAndAddInt**方法，以原子方式将value值增加1，然后返回增加前的原始值。

**注意，上述是JDK1.8的实现，在JDK1.8之前，上述方法采用了自旋+CAS操作的方式：**

```java
public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return current;
    }
}
```

#### 2.2.3、AtomicInteger的特殊方法说明

AtomicInteger中有一个比较特殊的方法——**lazySet**：
![clipboard.png](.\img\asd5ioasj45d2oiasas)

**`lazySet`**方法是**set**方法的不可见版本。什么意思呢？

我们知道通过`volatile`修饰的变量，可以保证在多处理器环境下的“可见性”。也就是说当一个线程修改一个共享变量时，其它线程能立即读到这个修改的值。`volatile`的实现最终是加了内存屏障：

1. **保证写volatile变量会强制把CPU写缓存区的数据刷新到内存**
2. **读volatile变量时，使缓存失效，强制从内存中读取最新的值**
3. **由于内存屏障的存在，volatile变量还能阻止重排序**

**lazySet**内部调用了Unsafe类的**putOrderedInt**方法，通过该方法对共享变量值的改变，不一定能被其他线程立即看到。也就是说以普通变量的操作方式来写变量。

为什么会有这种奇怪方法？什么情况下需要使用lazySet呢？

考虑下面这样一个场景：

```java
private AtomicInteger ai = new AtomicInteger();
lock.lock();
try
{
    // ai.set(1);
}
finally
{
    lock.unlock();
}
```

由于锁的存在：

- **lock()**方法获取锁时，和`volatile`变量的读操作一样，会强制使CPU缓存失效，强制从内存读取变量。
- **unlock()**方法释放锁时，和`volatile`变量的写操作一样，会强制刷新CPU写缓冲区，把缓存数据写到主内存

所以，上述`ai.set(1)`可以用`ai.lazySet(1)`方法替换：

由锁来保证共享变量的可见性，以设置普通变量的方式来修改共享变量，减少不必要的内存屏障，从而提高程序执行的效率。

#### 2.2.4、接口声明

| 方法声明                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| int `accumulateAndGet`(int x, IntBinaryOperator accumulatorFunction) | 使用IntBinaryOperator 对当前值和x进行计算，并更新当前值，返回计算后的新值 |
| int `addAndGet`(int delta)                                   | 以原子方式将给定值与当前值相加，返回相加后的新值             |
| boolean `compareAndSet`(int expect, int update)              | 如果当前值 == expect，则以原子方式将该值设置为给定的更新值（update） |
| int `decrementAndGet`()                                      | 以原子方式将当前值减 1，返回新值                             |
| int `get`()                                                  | 获取当前值                                                   |
| int `getAndAccumulate`(int x, IntBinaryOperator accumulatorFunction) | 使用IntBinaryOperator 对当前值和x进行计算，并更新当前值，返回计算前的旧值 |
| int `getAndAdd`(int delta)                                   | 以原子方式将给定值与当前值相加，返回旧值                     |
| int `getAndDecrement`()                                      | 以原子方式将当前值减 1，返回旧值                             |
| int `getAndIncrement`()                                      | 以原子方式将当前值加 1，返回旧值                             |
| int `getAndSet`(int newValue)                                | 以原子方式设置为给定值，并返回旧值                           |
| int `getAndUpdate`(IntUnaryOperator updateFunction)          | 使用IntBinaryOperator 对当前值进行计算，并更新当前值，返回计算前的旧值 |
| int `incrementAndGet`()                                      | 以原子方式将当前值加 1，返回新值                             |
| void `lazySet`(int newValue)                                 | 设置为给定值，但不保证值的改变被其他线程立即看到             |
| void `set`(int newValue)                                     | 设置为给定值                                                 |
| int `updateAndGet`(IntUnaryOperator updateFunction)          | 使用IntBinaryOperator 对当前值进行计算，并更新当前值，返回计算后的新值 |
| boolean `weakCompareAndSet`(int expect, int update)          | weakCompareAndSet无法保证除操作目标外的其他变量的执行顺序( 编译器和处理器为了优化程序性能而对指令序列进行重新排序 )，同时也无法保证这些变量的可见性。 |

##### AtomicInteger的常用方法

- `int addAndGet(int delta)`：以原子方式将输入的数值与实例中的值（`AtomicInteger` 里的 `value`）相加，并返回结果。
- `boolean compareAndSet(int expect，int update)`：如果输入的数值等于预期值`expect`，则以原子方式将该值设置为输入的值。
- `int getAndIncrement()`：以原子方式将当前值加1，注意，这里返回的是自增前的值。
- `void lazySet(int newValue)`：最终会设置成 `newValue`，使用 `lazySet` 设置值后，可导致其他线程在之后的一小段时间内还是可以读到旧的值。
- `int getAndSet(int newValue)`：以原子方式设置为 `newValue` 的值，并返回旧值。

##### 示例代码

```java
public static void main(String[] args) {
    AtomicInteger ai = new AtomicInteger(1);

    System.out.println("ai.get() = " + ai.get());

    System.out.println("ai.addAndGet(5) = " + ai.addAndGet(5));
    System.out.println("ai.get() = " + ai.get());

    System.out.println("ai.compareAndSet(ai.get(), 10) = " + ai.compareAndSet(ai.get(), 10));
    System.out.println("ai.get() = " + ai.get());

    System.out.println("ai.getAndIncrement() = " + ai.getAndIncrement());
    System.out.println("ai.get() = " + ai.get());

    ai.lazySet(8);
    System.out.println("ai.lazySet(8)");
    System.out.println("ai.get() = " + ai.get());

    System.out.println("ai.getAndSet(5) = " + ai.getAndSet(5));
    System.out.println("ai.get() = " + ai.get());
}
```

输出：

```java
ai.get() = 1
ai.addAndGet(5) = 6
ai.get() = 6
ai.compareAndSet(ai.get(), 10) = true
ai.get() = 10
ai.getAndIncrement() = 10
ai.get() = 11
ai.lazySet(8)
ai.get() = 8
ai.getAndSet(5) = 8
ai.get() = 5
```

`Atomic` 包提供了 `3` 种基本类型的原子更新，但是 `Java` 的基本类型里还有 `char`、`float` 和 `double` 等。
 那么问题来了，如何原子的更新其他的基本类型呢？
 `Atomic`包里的类基本都是使用 `Unsafe` 实现的，让我们一起看一下Unsafe的源码：

```java
/**
 * 如果当前数值是expected，则原子的将Java变量更新成x
 *
 * @return 如果更新成功则返回true
 */
public final native boolean compareAndSwapObject(Object o, long offset,
                                                 Object expected, Object x);

public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected, int x);

public final native boolean compareAndSwapLong(Object o, long offset,
                                               long expected, long x);
```

看 `AtomicBoolean` 源码，发现它是先把 `Boolean` 转换成 `整型`，再使用 `compareAndSwapInt` 进行 [CAS](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FCAS%2F7371138)，所以原子更新 `char`、`float` 和 `double` 变量也可以用类似的思路来实现。

### 2.2、其它原子类

- `AtomicBoolean`：原子更新布尔类型。
- `AtomicInteger`：原子更新整型。
- `AtomicLong`：原子更新长整型。

以上3个类提供的方法几乎一模一样，仅以 `AtomicInteger` 为例进行讲解。



## 3、原子更新引用

### 3.1、简介

`AtomicReference`：原子更新对象引用。

`AtomicMarkableReference`：原子更新带有标记位的对象引用。

`AtomicStampedReference`：原子更新带有版本号的对象引用。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用 [CAS](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FCAS%2F7371138) 进行原子更新时可能出现的 [ABA问题](https://www.jianshu.com/p/72d02353dc7e)。

### 3.2、AtomicReference

#### 3.2.1、简介

AtomicReference，顾名思义，就是以原子方式更新对象引用。

可以看到，AtomicReference持有一个对象的引用——**value**，并通过`Unsafe`类来操作该引用:

![clipboard.png](.\img\56hg54asd646sg4g5gfas64)

> ***为什么需要AtomicReference？难道多个线程同时对一个引用变量赋值也会出现并发问题？***
> 引用变量的赋值本身没有并发问题，也就是说对于引用变量var ，类似下面的赋值操作本身就是原子操作:
> `Foo var = ... ;`
> **AtomicReference的引入是为了可以用一种类似乐观锁的方式操作共享资源，在某些情景下以提升性能。**

我们知道，当多个线程同时访问共享资源时，一般需要以加锁的方式控制并发：

```java
volatile Foo sharedValue = value;
Lock lock = new ReentrantLock();

lock.lock();
try{
    // 操作共享资源sharedValue
}
finally{
    lock.unlock();
}
```

上述访问方式其实是一种对共享资源加**悲观锁**的访问方式。

而`AtomicReference`提供了**以无锁方式访问共享资源**的能力，看看如何通过`AtomicReference`保证线程安全，来看个具体的例子：

```java
public class AtomicRefTest {
    public static void main(String[] args) throws InterruptedException {
        AtomicReference<Integer> ref = new AtomicReference<>(new Integer(1000));

        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            Thread t = new Thread(new Task(ref), "Thread-" + i);
            list.add(t);
            t.start();
        }

        for (Thread t : list) {
            t.join();
        }

        System.out.println(ref.get());    // 打印2000
    }

}

class Task implements Runnable {
    private AtomicReference<Integer> ref;

    Task(AtomicReference<Integer> ref) {
        this.ref = ref;
    }
    
    @Override
    public void run() {
        for (; ; ) {    //自旋操作
            Integer oldV = ref.get();   
            if (ref.compareAndSet(oldV, oldV + 1))  // CAS操作 
                break;
        }
    }
}
```

> 上述示例，最终打印“2000”。

该示例并没有使用锁，而是使用**自旋+CAS**的无锁操作保证共享变量的线程安全。1000个线程，每个线程对金额增加1，最终结果为2000，如果线程不安全，最终结果应该会小于2000。

通过示例，可以总结出`AtomicReference`的一般使用模式如下：

```java
AtomicReference<Object> ref = new AtomicReference<>(new Object());
Object oldCache = ref.get();

// 对缓存oldCache做一些操作
Object newCache  =  someFunctionOfOld(oldCache); 

// 如果期间没有其它线程改变了缓存值，则更新
boolean success = ref.compareAndSet(oldCache , newCache);
```

上面的代码模板就是AtomicReference的常见使用方式，看下**compareAndSet**方法：

![clipboard.png](.\img\4564fsd89f4s89sd2)

该方法会将入参的**expect**变量所指向的对象和AtomicReference中的引用对象进行比较，如果两者指向同一个对象，则将AtomicReference中的引用对象重新置为**update**，修改成功返回true，失败则返回false。也就是说，***AtomicReference其实是比较对象的引用***。

#### 3.2.2、接口声明

##### AtomicReference的常用方法

| 方法返回类型 | 方法和方法描述                                               |
| ------------ | ------------------------------------------------------------ |
| `V`          | `accumulateAndGet(V x, BinaryOperator accumulatorFunction)`  使用将给定函数应用于当前值和给定值的结果原子更新当前值，返回更新后的值。 |
| `boolean`    | `compareAndSet(V expect,  V update)`  如果当前值 `==`为预期值，则将值设置为给定的更新值。 |
| `V`          | `get()`  获取当前值。                                        |
| `V`          | `getAndAccumulate(V x, BinaryOperator accumulatorFunction)`  使用给定函数应用给当前值和给定值的结果原子更新当前值，返回上一个值。 |
| `V`          | `getAndSet(V newValue)`  将原子设置为给定值并返回旧值。      |
| `V`          | `getAndUpdate(UnaryOperator updateFunction)`  用应用给定函数的结果原子更新当前值，返回上一个值。 |
| `void`       | `lazySet(V newValue)`  最终设定为给定值。                    |
| `void`       | `set(V newValue)`  设置为给定值。                            |
| `String`     | `toString()`  返回当前值的String表示形式。                   |
| `V`          | `updateAndGet(UnaryOperator updateFunction)`  使用给定函数的结果原子更新当前值，返回更新的值。 |
| `boolean`    | `weakCompareAndSet(V expect,  V update)`  如果当前值为 `==` ，则将原值设置为给定的更新值。 |

### 3.3、CAS操作可能存在的问题

> CAS操作可能存在**ABA**的问题，就是说：
> **假如一个值原来是A，变成了B，又变成了A，那么CAS检查时会发现它的值没有发生变化，但是实际上却变化了。**

一般来讲这并不是什么问题，比如数值运算，线程其实根本不关心变量中途如何变化，只要最终的状态和预期值一样即可。

但是，**有些操作会依赖于对象的变化过程，此时的解决思路一般就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A - 2B - 3A。**

### 3.4、AtomicStampedReference

`AtomicStampedReference`就是上面所说的加了版本号的`AtomicReference`。

#### 3.4.1、AtomicStampedReference原理

`AtomicStampedReference`只有一个构造器：

![clipboard.png](.\img\5as4f68sd4fd)

可以看到，除了传入一个初始的引用变量**`initialRef`**外，还有一个**initialStamp**变量，**initialStamp**其实就是版本号（或者说时间戳），用来唯一标识引用变量。

在构造器内部，实例化了一个**Pair**对象，**Pair**对象记录了对象引用和时间戳信息，采用int作为时间戳，实际使用的时候，要保证时间戳唯一（一般做成自增的），如果时间戳如果重复，还会出现**ABA**的问题。

`AtomicStampedReference`的所有方法，其实就是`Unsafe`类针对这个**Pair**对象的操作。
和`AtomicReference`相比，`AtomicStampedReference`中的每个引用变量都带上了`pair.stamp`这个版本号，这样就可以解决`CAS`中的**ABA**问题了。

#### 3.4.2、使用示例

```java
AtomicStampedReference<Foo>  asr = new AtomicStampedReference<>(null,0);  // 创建AtomicStampedReference对象，持有Foo对象的引用，初始为null，版本为0

int[] stamp=new  int[1];
Foo  oldRef = asr.get(stamp);   // 调用get方法获取引用对象和对应的版本号
int oldStamp=stamp[0];          // stamp[0]保存版本号

asr.compareAndSet(oldRef, null, oldStamp, oldStamp + 1)   //尝试以CAS方式更新引用对象，并将版本号+1
```

上述模板就是`AtomicStampedReference`的一般使用方式，注意下***compareAndSet***方法：

![clipboard.png](.\img\s5d6f465sd65sf)

我们知道，`AtomicStampedReference`内部保存了一个`pair`对象，该方法的逻辑如下：

1. 如果AtomicStampedReference内部pair的引用变量、时间戳 与 入参**expectedReference**、**expectedStamp**都一样，说明期间没有其它线程修改过AtomicStampedReference，可以进行修改。此时，会创建一个新的Pair对象（casPair方法，因为Pair是Immutable类）。

但这里有段优化逻辑，就是如果 `newReference == current.reference && newStamp == current.stamp`，说明用户修改的新值和`AtomicStampedReference`中目前持有的值完全一致，那么其实不需要修改，直接返回true即可。

#### 3.4.3、接口声明

##### 常用方法

| 方法返回类型 | 方法与方法描述                                               |
| ------------ | ------------------------------------------------------------ |
| `boolean`    | `attemptStamp(V expectedReference,  int newStamp)`  如果当前参考是预期参考的==，则 `==`地将该值的值设置为给定的更新值。 |
| `boolean`    | `compareAndSet(V expectedReference,  V newReference,  int expectedStamp, int newStamp)`  以原子方式设置该引用和邮票给定的更新值的值，如果当前的参考是  `==`至预期的参考，并且当前标志等于预期标志。 |
| `V`          | `get(int[] stampHolder)`  返回引用和戳记的当前值。           |
| `V`          | `getReference()`  返回引用的当前值。                         |
| `int`        | `getStamp()`  返回邮票的当前值。                             |
| `void`       | `set(V newReference,  int newStamp)`  无条件地设置引用和戳记的值。 |
| `boolean`    | `weakCompareAndSet(V expectedReference,  V newReference,  int expectedStamp, int newStamp)`  以原子方式设置该引用和邮票给定的更新值的值，如果当前的参考是  `==`至预期的参考，并且当前标志等于预期标志。 |

### 3.5、AtomicMarkableReference

我们在讲**ABA**问题的时候，引入了AtomicStampedReference。

AtomicStampedReference可以给引用加上版本号，追踪引用的整个变化过程，如：
A -> B -> C -> D - > A，通过AtomicStampedReference，我们可以知道，引用变量中途被更改了3次。

但是，有时候，我们并不关心引用变量更改了几次，只是单纯的关心**是否更改过**，所以就有了**AtomicMarkableReference**：

![clipboard.png](.\img\asd4f68sd47f89sd89f)

可以看到，`AtomicMarkableReference`的唯一区别就是**不再用int标识引用，而是使用boolean变量——表示引用变量是否被更改过**。

从语义上讲，`AtomicMarkableReference`对于那些不关心引用变化过程，只关心引用变量是否变化过的应用会更加友好。

#### 3.5.1、接口声明

##### 常用方法

| 方法返回类型 | 方法和方法描述                                               |
| ------------ | ------------------------------------------------------------ |
| `boolean`    | `attemptMark(V expectedReference,  boolean newMark)`  以原子方式设置标志给定的更新值的值，如果当前引用 `==`预期引用。 |
| `boolean`    | `compareAndSet(V expectedReference,  V newReference,  boolean expectedMark, boolean newMark)`  以原子方式设置该引用和标记给定的更新值的值，如果当前的参考是  `==`至预期的参考和当前标记等于预期标记。 |
| `V`          | `get(boolean[] markHolder)`  返回引用和标记的当前值。        |
| `V`          | `getReference()`  返回引用的当前值。                         |
| `boolean`    | `isMarked()`  返回标记的当前值。                             |
| `void`       | `set(V newReference,  boolean newMark)`  无条件地设置引用和标记的值。 |
| `boolean`    | `weakCompareAndSet(V expectedReference,  V newReference,  boolean expectedMark, boolean newMark)`  以原子方式设置该引用和标记给定的更新值的值，如果当前的参考是  `==`至预期的参考和当前标记等于预期标记。 |



## 4、原子更新数组

### 4.1、Atomic数组简介

Atomic数组，顾名思义，就是能以原子的方式，操作数组中的元素。

JDK提供了三种类型的原子数组：`AtomicIntegerArray`、`AtomicLongArray`、`AtomicReferenceArray`。

这三种类型大同小异，AtomicIntegerArray对应AtomicInteger，AtomicLongArray对应AtomicLong，AtomicReferenceArray对应AtomicReference。

其实阅读源码也可以发现，**这些数组原子类与对应的普通原子类相比，只是多了通过索引找到内存中元素地址的操作而已。**

> 注意：**原子数组并不是说可以让线程以原子方式一次性地操作数组中所有元素的数组。而是指对于数组中的每个元素，可以以原子方式进行操作。**

说得简单点，**原子数组类型**其实可以看成**原子类型组成的数组**。

比如：

```Java
AtomicIntegerArray  array = new AtomicIntegerArray(10);
array.getAndIncrement(0);   // 将第0个元素原子地增加1
```

等同于

```java
AtomicInteger[]  array = new AtomicInteger[10];
array[0].getAndIncrement();  // 将第0个元素原子地增加1
```

常用方法如下：

- `int addAndGet(int i，int delta)`：以原子方式将输入值与数组中索引i的元素相加。
- `boolean compareAndSet(int i，int expect，int update)`：如果当前值等于预期值，则以原子方式将数组位置i的元素设置成update值。

示例代码：

```java
public static void main(String[] args) {

    int[] value = new int[]{1, 2};

    AtomicIntegerArray ai = new AtomicIntegerArray(value);

    System.out.println("ai.getAndSet(0, 3)");
    ai.getAndSet(0, 3);
    System.out.println("ai.get(0) = " + ai.get(0));
    System.out.println("value[0] = " + value[0]);

    ai.compareAndSet(1, 2, 5);
    System.out.println("ai.compareAndSet(1, 2, 5)");
    System.out.println("ai.get(1) = " + ai.get(1));
}
```

输出结果：

```java
ai.getAndSet(0, 3)
ai.get(0) = 3
value[0] = 1
ai.compareAndSet(1,2,5)
ai.get(1) = 5
```

> 需要注意的是，数组`value`通过构造方法传递进去，然后`AtomicIntegerArray`会将当前数组复制一份，所以当`AtomicIntegerArray`对内部的数组元素进行 **修改** 时，**不会影响传入的数组**。



## 5、原子更新字段

### 5.1、什么是FieldUpdater

在`java.util.concurrent.atomic`包中，由三个比较特殊的原子类：`AtomicIntegerFieldUpdater`、`AtomicLongFieldUpdater`、`AtomicReferenceFieldUpdater`。

- `AtomicIntegerFieldUpdater`：原子更新`volatile`修饰的整型的字段的更新器。
- `AtomicLongFieldUpdater`：原子更新`volatile`修饰的长整型字段的更新器。
- `AtomicReferenceFieldUpdater`：原子更新`volatile`修饰的引用类型里的字段的更新器。

**要想原子地更新字段类需要两步**：

- 因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法`newUpdater()`创建一个更新器，并且需要设置想要更新的类和属性。
- **更新类的字段（属性）必须使用`public volatile`修饰符。**

以上3个类提供的方法几乎一样，所以仅以 `AstomicIntegerFieldUpdater` 为例进行讲解。

> 假设有一个公司账户Account，100个人同时往里面存钱1块钱，那么正常情况下，最终账户的总金额应该是100。

先来看下线程不安全的方式：

**账户类：**

```java
class Account {
    private volatile int money;

    Account(int initial) {
        this.money = initial;
    }

    public void increMoney() {
        money++;
    }

    public int getMoney() {
        return money;
    }

    @Override
    public String toString() {
        return "Account{" +
            "money=" + money +
            '}';
    }
}
```

**调用类：**

```java
public class FieldUpdaterTest {
    public static void main(String[] args) throws InterruptedException {
        Account account = new Account(0);  // 初始金额0

        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            Thread t = new Thread(new Task(account));
            list.add(t);
            t.start();
        }

        for (Thread t : list) {            // 等待所有线程执行完成
            t.join();
        }

        System.out.println(account.toString());
    }


    private static class Task implements Runnable {
        private Account account;

        Task(Account account) {
            this.account = account;
        }

        @Override
        public void run() {
            account.increMoney();         // 增加账户金额
        }
    }
}
```

上述未对**Account**做并发控制，**最终账户金额很可能小于100**。
按照之前学习的atomic框架，可以将**Account**类的**int**类型字段改为**AtomicInteger**，或者在**Task**任务类中，将所有涉及到共享变量的地方都加锁访问。

**那么，还有没有其它解决方式？**

本章开头我们讲到，**AtomicXXXFieldUpdater**可以**以一种线程安全的方式操作非线程安全对象的某些字段**。
这里，**Account**就是非线程安全对象，**money**就是需要操作的字段。

我们来对上述代码进行改造：
**账户类Account改造：**

```java
class Account {
    private volatile int money;
    private static final AtomicIntegerFieldUpdater<Account> updater = AtomicIntegerFieldUpdater.newUpdater(Account.class, "money");  // 引入AtomicIntegerFieldUpdater

    Account(int initial) {
        this.money = initial;
    }

    public void increMoney() {
        updater.incrementAndGet(this);    // 通过AtomicIntegerFieldUpdater操作字段
    }

    public int getMoney() {
        return money;
    }

    @Override
    public String toString() {
        return "Account{" +
            "money=" + money +
            '}';
    }
}
```

**调用方，并未做任何改变：**

```java
public class FieldUpdaterTest {
    public static void main(String[] args) throws InterruptedException {
        Account account = new Account(0);

        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            Thread t = new Thread(new Task(account));
            list.add(t);
            t.start();
        }

        for (Thread t : list) {
            t.join();
        }

        System.out.println(account.toString());
    }

    private static class Task implements Runnable {
        private Account account;

        Task(Account account) {
            this.account = account;
        }

        @Override
        public void run() {
            account.increMoney();
        }
    }
}
```

上述代码，无论执行多少次，最终结果都是“100”，因为这回是线程安全的。

对比下改造，可以发现，**AtomicIntegerFiledUpdater**的引入，使得我们可以在***不修改用户代码（调用方）的情况下，就能实现并发安全性\***。

唯一的改变之处就是Account内部的改造：
<img src=".\img\saoi218u1gasd25t6" alt="clipboard.png" style="zoom:150%;" />

这也是**AtomicXXXFieldUpdater**引入的一个重要原因，单纯从功能上来讲，能用**AtomicXXXFieldUpdater**实现的并发控制，同步器和其它原子类都能实现，但是使用**AtomicXXXFieldUpdater**，符合面向对象设计的一个基本原则——开闭原则，尤其是对一些遗留代码的改造上。

另外，**使用AtomicXXXFieldUpdater，不需要进行任何同步处理，单纯的使用CAS+自旋操作就可以实现同步的效果。这也是整个atomic包的设计理念之一**。

### 5.2、AtomicIntegerFieldUpdater原理

`AtomicIntegerFieldUpdater`、`AtomicLongFieldUpdater`、`AtomicReferenceFieldUpdater`这三个类大同小异，**AtomicIntegerFieldUpdater**只能处理**int**原始类型的字段，**AtomicLongFieldUpdater**只能处理long原始类型的字段，**AtomicReferenceFieldUpdater**可以处理所有引用类型的字段。

本节以**AtomicReferenceFieldUpdater**为例，介绍下`FiledUpdater`的基本原理。

#### 5.2.1、AtomicReferenceFieldUpdater对象的创建

`AtomicReferenceFieldUpdater`本身是一个抽象类，没有公开的构造器，只能通过静态方法**newUpdater**创建一个`AtomicReferenceFieldUpdater`子类对象：

![image-20191205213409648](.\img\image-20191205213409648.png)

newUpdater的三个入参含义如下：

| 入参名称  | 含义           |
| --------- | -------------- |
| tclass    | 目标对象的类型 |
| fieldName | 目标字段名     |

**AtomicReferenceFieldUpdaterImpl**是`AtomicReferenceFieldUpdater`的一个内部类，并继承了`AtomicReferenceFieldUpdater`。`AtomicReferenceFieldUpdater`的API，基本都是委托`AtomicReferenceFieldUpdaterImpl` 来实现的。

来看下**AtomicReferenceFieldUpdaterImpl** 对象的构造，其实就是一系列的权限检查：
<img src=".\img\fiaosjfi2190ufzsakfa" alt="clipboard.png" style="zoom:150%;" />

通过源码，可以看到**AtomicReferenceFieldUpdater**的使用必须满足以下条件：

1. `AtomicReferenceFieldUpdater`只能修改对于它可见的字段，也就是说对于目标类的某个字段field，如果修饰符是`private`，但是`AtomicReferenceFieldUpdater`所在的使用类不能看到field，那就会报错；
2. 目标类的操作字段，必须用`volatile`修饰；
3. 目标类的操作字段，不能是`static`的；
4. `AtomicReferenceFieldUpdater`只适用于引用类型的字段；

#### 5.2.2、常用方法

| 方法返回类型                        | 方法描述                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| `int`                               | `accumulateAndGet(T obj,  int x, IntBinaryOperator accumulatorFunction)`  原子更新由此更新程序管理的给定对象的字段，并将给定函数应用于当前值和给定值，返回更新后的值。 |
| `int`                               | `addAndGet(T obj,  int delta)`  将给定值原子地添加到由此更新程序管理的给定对象的字段的当前值。 |
| `abstract boolean`                  | `compareAndSet(T obj,  int expect, int update)`  如果当前值 `==`为预期值，则将由此更新程序管理的给定对象的字段原子设置为给定的更新值。 |
| `int`                               | `decrementAndGet(T obj)`  由此更新程序管理的给定对象的字段的当前值原子减1。 |
| `abstract int`                      | `get(T obj)`  获取由此更新程序管理的给定对象的字段中保留的当前值。 |
| `int`                               | `getAndAccumulate(T obj,  int x, IntBinaryOperator accumulatorFunction)`  原子更新由此更新程序管理的给定对象的字段，并将给定函数应用于当前值和给定值，返回上一个值。 |
| `int`                               | `getAndAdd(T obj,  int delta)`  将给定值原子地添加到由此更新程序管理的给定对象的字段的当前值。 |
| `int`                               | `getAndDecrement(T obj)`  由此更新程序管理的给定对象的字段的当前值原子减1。 |
| `int`                               | `getAndIncrement(T obj)`  由此更新程序管理的给定对象的字段的当前值以原子方式递增1。 |
| `int`                               | `getAndSet(T obj,  int newValue)`  将由此更新程序管理的给定对象的字段原子设置为给定值，并返回旧值。 |
| `int`                               | `getAndUpdate(T obj,  IntUnaryOperator updateFunction)`  使用应用给定函数的结果原子更新由此更新程序管理的给定对象的字段，返回上一个值。 |
| `int`                               | `incrementAndGet(T obj)`  由此更新程序管理的给定对象的字段的当前值以原子方式递增1。 |
| `abstract void`                     | `lazySet(T obj,  int newValue)`  最终将由此更新程序管理的给定对象的字段设置为给定的更新值。 |
| `static  AtomicIntegerFieldUpdater` | `newUpdater(类 tclass, String fieldName)`  创建并返回具有给定字段的对象的更新程序。 |
| `abstract void`                     | `set(T obj,  int newValue)`  将由此更新程序管理的给定对象的字段设置为给定的更新值。 |
| `int`                               | `updateAndGet(T obj,  IntUnaryOperator updateFunction)`  原子更新由此更新程序管理的给定对象的字段与应用给定函数的结果，返回更新的值。 |
| `abstract boolean`                  | `weakCompareAndSet(T obj,  int expect, int update)`  如果当前值 `==`为预期值，则将由此更新程序管理的给定对象的字段原子设置为给定的更新值。 |



## 6、LongAdder

### 6.1、简介

JDK1.8时，`java.util.concurrent.atomic`包中提供了一个新的原子类：`LongAdder`。
根据Oracle官方文档的介绍，LongAdder在高并发的场景下会比它的前辈————AtomicLong 具有更好的性能，代价是消耗更多的内存空间：

那么，问题来了：

> **为什么要引入`LongAdder`？ `AtomicLong`在高并发的场景下有什么问题吗？ 如果低并发环境下，`LongAdder`和`AtomicLong`性能差不多，那`LongAdder`是否就可以替代`AtomicLong`了？**

### 为什么要引入LongAdder？

我们知道，**AtomicLong**是利用了底层的CAS操作来提供并发性的，比如**addAndGet**方法：

![clipboard.png](.\img\aijsgf9asjf2109409asf)

上述方法调用了**Unsafe**类的**getAndAddLong**方法，该方法是个**native**方法，它的逻辑是采用自旋的方式不断更新目标值，直到更新成功。

在并发量较低的环境下，线程冲突的概率比较小，自旋的次数不会很多。但是，高并发环境下，N个线程同时进行自旋操作，会出现大量失败并不断自旋的情况，此时**AtomicLong**的自旋会成为瓶颈。

这就是**LongAdder**引入的初衷——解决高并发环境下**AtomicLong**的自旋瓶颈问题。

### LongAdder快在哪里？

既然说到**LongAdder**可以显著提升高并发环境下的性能，那么它是如何做到的？这里先简单的说下**LongAdder**的思路，第二部分会详述**LongAdder**的原理。

我们知道，**AtomicLong**中有个内部变量**value**保存着实际的long值，所有的操作都是针对该变量进行。也就是说，高并发环境下，value变量其实是一个热点，也就是N个线程竞争一个热点。

**LongAdder**的基本思路就是***分散热点\***，将value值分散到一个数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样热点就被分散了，冲突的概率就小很多。如果要获取真正的long值，只要将各个槽中的变量值累加返回。

这种做法有没有似曾相识的感觉？没错，[ConcurrentHashMap](https://segmentfault.com/a/1190000015865714#)中的“分段锁”(`JDK1.7`)其实就是类似的思路。

### LongAdder能否替代AtomicLong？

**LongAdder**提供的API和**AtomicLong**比较接近，两者都能以原子的方式对long型变量进行增减。

但是**AtomicLong**提供的功能其实更丰富，尤其是**addAndGet**、**decrementAndGet**、**compareAndSet**这些方法。

**addAndGet**、**decrementAndGet**除了单纯的做自增自减外，还可以立即获取增减后的值，而**LongAdder**则需要做同步控制才能精确获取增减后的值。如果业务需求需要精确的控制计数，做计数比较，**AtomicLong**也更合适。

另外，从空间方面考虑，**LongAdder**其实是一种“空间换时间”的思想，从这一点来讲**AtomicLong**更适合。当然。

总之，**低并发、一般的业务场景下`AtomicLong`是足够了。如果并发量很多，存在大量写多读少的情况，那`LongAdder`可能更合适**。



# 四、并发工具类

## 1、等待多线程完成的CountDownLatch

### 1.1、简介

`CountDownLatch`是一个辅助同步器类，用来做计数使用，它的作用有点类似于生活中的倒数计数器。先设定一个计数初始值，当计数降到0时，将会触发一些事件，**如火箭的倒数计时**。

初始计数值在构造`CountDownLatch`对象时传入，每调用一次 `countDown()` 方法，计数值就会减1。

线程可以调用`CountDownLatch`的**`await`**方法进入阻塞，当计数值降到0时，所有之前调用**`await`**阻塞的线程都会释放。

> **注意：**`CountDownLatch`的初始计数值一旦降到0，无法重置。如果需要重置，可以考虑使用`CyclicBarrier`。

![2017021200001](.\img\2018120818001.png)

`CountDownLatch` 是通过一个计数器来实现的，当我们在 `new` 一个 `CountDownLatch` 对象的时候，需要带入该计数器值，该值就表示了线程的数量。

- 每当一个线程完成自己的任务后，计数器的值就会减 1 。
- 当计数器的值变为0时，就表示所有的线程均已经完成了任务，然后就可以恢复等待的线程继续执行了。

虽然，`CountDownLatch` 与 `CyclicBarrier` 有那么点相似，但是他们还是存在一些**区别**的：

1. `CountDownLatch` 的作用是允许 1 或 N 个线程等待其他线程完成执行；而 `CyclicBarrier` 则是允许 N 个线程相互等待。
2. `CountDownLatch` 的计数器无法被重置；`CyclicBarrier` 的计数器可以被重置后使用，因此它被称为是循环的 `barrier` 。

### 1.2、使用示例

假如有这样一个需求：我们需要解析一个Excel里多个sheet的数据，此时可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成sheet的解析操作，最简单的做法是使用`join()`方法。

[关于join的介绍](https://www.jianshu.com/p/595be9eab056)

`join` 用于让当前执行线程等待join线程执行结束。其实现原理是不停检查join线程是否存活，
 如果 `join` 线程存活则让当前线程永远等待。其中，`wait(0)`表示永远等待下去。代码片段如下：

```java
while (isAlive()) {
    wait(0);
}
```

直到 `join` 线程中止后，线程的 `this.notifyAll()` 方法会被调用

在JDK 1.5之后的并发包中提供的 `CountDownLatch` 也可以实现 `join` 的功能，并且比`join`的功能更多，示例如下：

```java
public static class CountDownLatchTest {
    static CountDownLatch c = new CountDownLatch(3);

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1 + " -- " + System.currentTimeMillis());
                c.countDown();
                System.out.println(2 + " -- " + System.currentTimeMillis());
                c.countDown();
                try {
                    Thread.sleep(1000);
                    System.out.println(4 + " -- " + System.currentTimeMillis());
                    c.countDown();
                    Thread.sleep(1000);
                    System.out.println(5 + " -- " + System.currentTimeMillis());
                    c.countDown();
                    System.out.println(6 + " -- " + System.currentTimeMillis());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        c.await();
        System.out.println("3" + " -- " + System.currentTimeMillis());
    }
}
```

**输出结果**

```
1 -- 1558966154351
2 -- 1558966154351
4 -- 1558966155352
3 -- 1558966155352
5 -- 1558966156353
6 -- 1558966156353
```

`CountDownLatch` 的构造函数接收一个`int` 类型的参数作为计数器，如果你想等待`N`个点完成，这里就传入`N`。
 当我们调用 `CountDownLatch` 的 `countDown` 方法时，`N` 就会 `-1`，`CountDownLatch` 的 `await` 方法会阻塞当前线程，`直到 N 变成零`。
 由于 `countDown` 方法可以用在任何地方，所以这里说的 `N` 个点，可以是 **N个线程**，也可以是 **1个线程里的N个执行步骤**。
 用在多个线程时，只需要把这个 `CountDownLatch` 的引用传递到线程里即可。
 如果有某个解析 `sheet` 的线程处理得比较慢，我们不可能让主线程一直等待，所以可以使用另外一个带指定时间的 `await` 方法—— `await(long time，TimeUnit unit)`，这个方法等待特定时间后，就会不再阻塞当前线程。
 `join` 也有类似的方法。

> 计数器必须大于等于0，只是等于0时候，计数器就是零，调用`await`方法时不会阻塞当前线程。
>  `CountDownLatch`不可能重新初始化或者修改`CountDownLatch`对象的内部计数器的值。
>  一个线程调用`countDown`方法`happen-before`另外一个线程调用`await`方法。

#### 1.2.1、作为一个开关/入口

将初始计数值为1的 `CountDownLatch` 作为一个的开关或入口：
在调用 **`countDown()`** 的线程打开入口前，所有调用 **await** 的线程都一直在入口处等待。

```java
public class Driver {
    private static final int N = 10;
 
    public static void main() throws InterruptedException {
        CountDownLatch switcher = new CountDownLatch(1);
 
        for (int i = 0; i < N; ++i) {
            new Thread(new Worker(switcher)).start();
        }
 
        doSomething();
        switcher.countDown();       // 主线程开启开关
 
    }
 
    public static void doSomething() {
    }
}
 
class Worker implements Runnable {
    private final CountDownLatch startSignal;
 
    Worker(CountDownLatch startSignal) {
        this.startSignal = startSignal;
    }
 
    public void run() {
        try {
            startSignal.await();    //所有执行线程在此处等待开关开启
            doWork();
        } catch (InterruptedException ex) {
        }
    }
    void doWork() { ...}
}
```

#### 1.2.2、作为一个完成信号

将初始计数值为N的 CountDownLatch作为一个完成信号点：使某个线程在其它N个线程完成某项操作之前一直等待。

```java
/**
 * 示例：老板进入会议室等待 5 个人全部到达会议室才会开会。
 * @author Jjcc
 * @version 1.0.0
 * @className CountDownLatchDemo1.java
 * @createTime 2019年12月08日 15:21:00
 */
public class CountDownLatchDemo1 {

    public static void main(String[] args) {
        int countDown = 5;
        CountDownLatch countDownLatch = new CountDownLatch(countDown);

        BossThread bossThread = new BossThread(countDownLatch);

        EmpleoyeeThread empleoyeeThread = new EmpleoyeeThread(countDownLatch);

        new Thread(bossThread).start();


        for (int i = 0; i < countDown; i++) {
            new Thread(empleoyeeThread).start();
        }

    }
}

/**
 *  Boss线程，等待员工到达开会
 * @title
 * @author Jjcc
 * @createTime 2019/12/8 15:30
 */
class BossThread implements Runnable {

    private CountDownLatch countDownLatch;

    BossThread(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }

    @Override
    public void run() {
        System.out.println("BOSS在办公室等待，总共有" + countDownLatch.getCount() + "个人开会...");
        try {
            countDownLatch.await(10, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("所有人都已经到齐了，开会吧...");
    }
}

/**
 * 员工到达会议室线程
 * @title
 * @author Jjcc
 * @createTime 2019/12/8 15:31
 */
class EmpleoyeeThread implements Runnable {

    private CountDownLatch countDownLatch;

    EmpleoyeeThread(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "，到达会议室....");
        //员工到达会议室 count - 1
        countDownLatch.countDown();
    }
}
```

**运行接口：**

![2017021200002](.\img\2018120818003.png)

### 1.3、接口声明

| 方法返回类型 | 方法描述                                                     |
| ------------ | ------------------------------------------------------------ |
| `void`       | `await()`  导致当前线程等到锁存器计数到零，除非线程是 [interrupted](../../../java/lang/Thread.html#interrupt--) 。 |
| `boolean`    | `await(long timeout,  TimeUnit unit)`  使当前线程等待直到锁存器计数到零为止，除非线程为 [interrupted](../../../java/lang/Thread.html#interrupt--)或指定的等待时间过去。 |
| `void`       | `countDown()`  减少锁存器的计数，如果计数达到零，释放所有等待的线程。 |
| `long`       | `getCount()`  返回当前计数。                                 |
| `String`     | `toString()`  返回一个标识此锁存器的字符串及其状态。         |

### 1.4、总结

`CountDownLatch` **内部通过共享锁实现**。

- 在创建 CountDownLatch 实例时，需要传递一个int型的参数：`count`，该参数为计数器的初始值，也可以理解为该共享锁可以获取的总次数。
- 当某个线程调用 `#await()` 方法，程序首先判断 `count` 的值是否为 0 ，如果不为 0 的话，则会一直等待直到为 0 为止。
- 当其他线程调用 `#countDown()` 方法时，则执行释放共享锁状态，使 `count` 值 - 1。
- 当在创建 CountDownLatch 时初始化的 `count` 参数，必须要有 `count` 线程调用`#countDown()` 方法，才会使计数器 `count` 等于 0 ，锁才会释放，前面等待的线程才会继续运行。
- 注意 CountDownLatch **不能回滚重置**。



## 2、同步屏障CyclicBarrier

### 2.1、简介

`CyclicBarrier`是一个辅助同步器类，在JDK1.5时随着J.U.C一起引入。

这个类的功能和我们之前介绍的[CountDownLatch](https://segmentfault.com/a/1190000015886497)有些类似。我们知道，`CountDownLatch`是一个倒数计数器，在计数器不为0时，所有调用`await`的线程都会等待，当计数器降为0，线程才会继续执行，且计数器一旦变为0，就不能再重置了。

`CyclicBarrier`可以认为是一个栅栏，栅栏的作用是什么？就是阻挡前行。

顾名思义，CyclicBarrier是一个可以循环使用的栅栏，它做的事情就是：
**让线程到达栅栏时被阻塞(调用await方法)，直到到达栅栏的线程数满足指定数量要求时，栅栏才会打开放行。**

> 这其实有点像军训报数，报数总人数满足教官认为的总数时，教官才会安排后面的训练。

可以看下面这个图来理解下:
一共4个线程A、B、C、D，它们到达栅栏的顺序可能各不相同。当A、B、C到达栅栏后，由于没有满足总数【4】的要求，所以会一直等待，当线程D到达后，栅栏才会放行。
![clipboard.png](.\img\864783998-5b66b8afe90dd_articlex.png)

从`CyclicBarrier`的构造器，我们也可以看出关于这个类的一些端倪，`CyclicBarrier`有两个构造器：

**构造器一：**
![clipboard.png](.\img\1539768829-5b66b8d16f1a5_articlex.png)
这个构造器的参数`parties`就是之前说的需要满足的计数总数。

**构造器二：**
![clipboard.png](.\img\3985229087-5b66b906283db_articlex.png)
这个构造器稍微特殊一些，除了指定了计数总数外，传入了一个`Runnable`任务。

Runnable任务其实就是当最后一个线程到达栅栏时，后续立即要执行的任务。

> 比如，军训报数完毕后，总人数满足了要求，教官就会开始命令大家执行下一个任务，这个【下一个任务】就是这里的Runnable。

### 2.2、CyclicBarrier原理

#### 2.2.1、CyclicBarrier的构造

**CyclicBarrier**有两个构造器：

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

构造器内部的各个字段含义如下：

| 字段名         | 作用                         |
| -------------- | ---------------------------- |
| parties        | 栅栏开启需要的到达线程总数   |
| count          | 剩余未到达的线程总数         |
| barrierCommand | 最后一个线程到达后执行的任务 |

- `CyclicBarrier(int parties)`：创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
- `CyclicBarrier(int parties, Runnable barrierAction)` ：创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 `barrier` 的线程执行。

#### 2.2.2、CyclicBarrier的内部结构

`java.util.concurrent.CyclicBarrier` 的结构如下：

![CyclicBarrier 结构图](.\img\2018120817001.png)

通过上图，我们可以看到 CyclicBarrier 的内部是使用重入锁 `ReentrantLock` 和 `Condition` 。

```java
public class CyclicBarrier {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition trip = lock.newCondition();

    // 栅栏开启需要的到达线程总数
    private final int parties;

    // 最后一个线程到达后执行的任务
    private final Runnable barrierCommand;

    // 剩余未到达的线程总数
    private int count;

    // 当前轮次的运行状态
    private Generation generation = new Generation();

     private static class Generation {
        boolean broken = false;
    }
    
    // ...
}
```

需要注意的是`generation`这个字段：

```java
/**
 * 由于CyclicBarrier是可以循环复用的
 * 所以用一个内部类Generation表示每一轮的CyclicBarrier的运行状况
 */
private static class Generation {
	boolean broken = false;
}

/**
 * 唤醒所有等待线程并开启下一轮
 */
private void nextGeneration() {
    // 唤醒trip条件队列中所有等待的线程
    trip.signalAll();
    // 重置未到达的线程数
    count = parties;
    // 创建一个新的Generation对象，表示下一轮的开始
    generation = new Generation();
}
```

我们知道，**`CyclicBarrier` 是可以循环复用的，所以`CyclicBarrier` 的每一轮任务都需要对应一个`generation` 对象。**
**`generation` 对象内部有个`broken`字段，用来标识当前轮次的`CyclicBarrier` 是否已经损坏**。
**`nextGeneration`**方法用来创建一个新的`generation` 对象，并唤醒所有等待线程，重置内部参数。

#### 2.2.3、CyclicBarrier的核心方法

##### await

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);//不超时等待
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
           BrokenBarrierException,
           TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}
```

可以看到，无论有没有超时功能，内部都是调了**dowait**这个方法。

##### dowait

`#dowait(boolean timed, long nanos)` 方法，代码如下：

```java
private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
        TimeoutException {
    // 获取锁
    final ReentrantLock lock = this.lock;
    // 加锁，控制对共享资源的访问。
    lock.lock();
    try {
        //分代
        final Generation g = generation;

        //当前generation“已损坏”，抛出BrokenBarrierException异常
        //抛出该异常一般都是某个线程在等待某个处于“断开”状态的CyclicBarrie
        if (g.broken)
            //当某个线程试图等待处于断开状态的 barrier 时，或者 barrier 进入断开状态而线程处于等待状态时，抛出该异常
            throw new BrokenBarrierException();

        //如果线程中断，终止CyclicBarrier
        if (Thread.interrupted()) {
            // 破坏栅栏，唤醒所有已经等待的线程。
            breakBarrier();
            throw new InterruptedException();
        }

        //进来一个线程 count - 1，index标识当前线程。
        int index = --count;
        //count == 0 表示所有线程均已到位，触发Runnable任务
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                //触发任务
                if (command != null)
                    command.run();
                ranAction = true;
                //唤醒所有等待线程，并更新generation
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction) // 未执行，说明 barrierCommand 执行报错，或者线程打断等等情况。
                    breakBarrier();
            }
        }
		
        // 自旋，防止线程被意外唤醒
        for (;;) {
            try {
                //如果不是超时等待，则调用Condition.await()方法等待
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    //超时等待，调用Condition.awaitNanos()方法等待
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                // 等待过程中被中断
                if (g == generation && ! g.broken) {
                    // 说明当前这轮还没有结束，没有其它线程执行过breakBarrier方法。
                    breakBarrier();
                    throw ie;
                } else {
                    // 说明当前这轮已经结束，或有其它线程执行过breakBarrier方法。
                    // 自我中断
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            //generation已经更新，返回index
            if (g != generation)
                return index;

            //“超时等待”，并且时间已到,终止CyclicBarrier，并抛出异常
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        //释放锁
        lock.unlock();
    }
}

private void breakBarrier() {
    // 表示栅栏已经被破坏
    generation.broken = true;
    //重置未到达的线程数
    count = parties;
    //唤醒条件队列中所有等待的线程
    trip.signalAll();
}
```

**dowait**方法并不复杂，一共有3部分：

1. 判断栅栏是否已经损坏或当前线程已经被中断，如果是会分别抛出异常；
2. 如果当前线程是最后一个到达的线程，会尝试执行最终任务（如果构造**`CyclicBarrier`**对象时有传入**`Runnable`**的话），执行成功即返回，失败会破坏栅栏；
3. 对于不是最后一个到达的线程，会在**`Condition`**队列上等待，为了防止被意外唤醒，这里用了一个自旋操作。

如果该线程不是到达的最后一个线程，则他会一直处于等待状态，除非发生以下情况：

1. 最后一个线程到达，即 `index == 0` 。
2. 超出了指定时间（超时等待）。
3. 其他的某个线程中断当前线程。
4. 其他的某个线程中断另一个等待的线程。
5. 其他的某个线程在等待 barrier 超时。
6. 其他的某个线程在此 barrier 调用 `#reset()` 方法。`#reset()` 方法，用于将屏障重置为初始状态。

在 `#dowait(boolean timed, long nanos)` 方法的源代码中，我们总是可以看到抛出 **BrokenBarrierException** 异常，那么什么时候抛出异常呢？例如：

- 如果一个线程处于等待状态时，如果其他线程调用 `#reset()` 方法。
- 调用的 `barrier` 原本就是被损坏的，则抛出 **BrokenBarrierException** 异常。
- 任何线程在等待时被中断了，则其他所有线程都将抛出 **BrokenBarrierException** 异常，并将 `barrier` 置于**损坏**状态。详细解析。

##### Generation

Generation 是 CyclicBarrier 内部静态类，描述了 CyclicBarrier 的更新换代。**在CyclicBarrier中，同一批线程属于同一代**。当有 `parties` 个线程全部到达 barrier 时，`generation` 就会被更新换代。其中 `broken` 属性，标识该当前 CyclicBarrier 是否已经处于中断状态。代码如下：

```java
private static class Generation {
    boolean broken = false;
}
```

##### breakBarrier

当 barrier 损坏了，或者有一个线程中断了，则通过 `#breakBarrier()` 方法，来终止所有的线程。代码如下：

```java
private void breakBarrier() {
    // 表示栅栏已经被破坏
    generation.broken = true;
    //重置未到达的线程数
    count = parties;
    //唤醒条件队列中所有等待的线程
    trip.signalAll();
}
```

在 `breakBarrier()` 方法中，中除了将 `broken`设置为 true ，还会调用 `#signalAll()` 方法，将在 CyclicBarrier 处于**等待状态的线程**全部唤醒。

##### nextGeneration

当所有线程都已经到达 barrier 处（`index == 0`），则会通过 `nextGeneration()` 方法，进行更新换代操作。在这个步骤中，做了三件事：

1. 唤醒所有线程。
2. 重置 `count` 。
3. 重置 `generation` 。

代码如下：

```java
private void nextGeneration() {
    trip.signalAll();
    count = parties;
    generation = new Generation();
}
```

##### reset

`#reset()` 方法，重置 barrier 到初始化状态。代码如下：

```java
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        breakBarrier();   // break the current generation
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

通过组合 `#breakBarrier()` 和 `#nextGeneration()` 方法来实现；先破坏栅栏，然后开始下一轮（新建一个generation对象）。

##### getNumberWaiting

`#getNumberWaiting()` 方法，获得等待的线程数。代码如下：

```java
public int getNumberWaiting() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return parties - count;
    } finally {
        lock.unlock();
    }
}
```

##### isBroken

`#isBroken()` 方法，判断 CyclicBarrier 是否处于中断。代码如下：

```java
public boolean isBroken() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return generation.broken;
    } finally {
        lock.unlock();
    }
}
```

### 2.3、CyclicBarrier示例

#### 2.3.1、示例一

**假设现在有这样一个场景：**
5个运动员准备跑步比赛，运动员在赛跑前会准备一段时间，当裁判发现所有运动员准备完毕后，就举起发令枪，比赛开始。
这里的起跑线就是屏障，运动员必须在起跑线等待其他运动员准备完毕。

```java
public class CyclicBarrierDemo1 {
    public static void main(String[] args) {
        int n = 5;

        CyclicBarrier cyclicBarrier = new CyclicBarrier(n, () ->
                System.out.println("****** 所有运动员已准备完毕，发令枪：跑！******")
        );

        for (int i = 0; i < n; i++) {
            Thread thread = new Thread(new PrepareWork(cyclicBarrier), "运动员[" + i + "]");

            if (3 == i) {
                thread.interrupt();
            }

            thread.start();
        }
    }

    private static class PrepareWork implements Runnable {

        private CyclicBarrier cyclicBarrier;

        public PrepareWork(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(3000);

                System.out.println(Thread.currentThread().getName() + ": 准备完成");
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

执行上面的程序，可能的输出结果如下：

```
运动员[3]: 准备完成
运动员[1]: 准备完成
运动员[0]: 准备完成
运动员[2]: 准备完成
运动员[4]: 准备完成
****** 所有运动员已准备完毕，发令枪：跑！******
```

#### 2.3.2、示例二

`CyclicBarrier` 可以用于多线程计算数据，最后合并计算结果的场景。
例如，用一个Excel保存了用户所有银行流水，每个Sheet保存一个账户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用 `CyclicBarrier(int parties, Runnable barrierAction)` 中的 `barrierAction` 用这些线程的计算结果，计算出整个Excel的日均银行流水，代码如下：

```java
public class CyclicBarrierDemo2 {

    private int n = 4;

    /**
     * 设置屏障的等待线程数为4个，到达4个之后执行指定的方法
     */
    private CyclicBarrier cyclicBarrier = new CyclicBarrier(n, new MyRunnable());

    /**
     * 创建一个线程池，设置最多4个核心线程池。
     */
    private Executor executor = Executors.newFixedThreadPool(n);

    /**
     * 保存每个sheet计算出的银流结果
     */
    private ConcurrentHashMap<String, Integer> concurrentHashMap = new ConcurrentHashMap<>(8);


    public static void main(String[] args){
        CyclicBarrierDemo2 cyclicBarrierDemo2 = new CyclicBarrierDemo2();
        cyclicBarrierDemo2.count();
    }


     private void count() {
        for (int i = 0; i < n; i++) {
            executor.execute(() ->{
                try {
                    // 计算当前sheet的银流数据，计算代码省略
                    concurrentHashMap.put(Thread.currentThread().getName(), 2);
                    // 银流计算完成，插入一个屏障
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
    }

    class MyRunnable implements Runnable {
        @Override
        public void run() {
            int result = 0;

            Set<Map.Entry<String, Integer>> entries = concurrentHashMap.entrySet();
            // 汇总每个sheet计算出的结果
            for (Map.Entry<String, Integer> entry : entries) {
                Integer value = entry.getValue();
                result += value;
            }

            concurrentHashMap.put("result", result);
            System.out.println("result: " + result);
        }
    }
}
```

使用线程池创建4个线程，分别计算每个sheet里的数据，每个sheet计算结果是`1`，再由`BankWaterService` 线程汇总4个sheet计算出的结果输出结果：

```
result: 8
```

#### 2.3.3、CyclicBarrier对异常的处理

我们知道，线程在阻塞过程中，可能被中断，那么既然**CyclicBarrier**放行的条件是等待的线程数达到指定数目，万一线程被中断导致最终的等待线程数达不到栅栏的要求怎么办？

CyclicBarrier一定有考虑到这种异常情况，不然其它所有等待线程都会无限制地等待下去。
那么CyclicBarrier是如何处理的呢？

我们看下CyclicBarrier的`await()`方法：

```java
public int await() throws InterruptedException, BrokenBarrierException {
    //...
}
```

可以看到，这个方法除了抛出**`InterruptedException`**异常外，还会抛出`BrokenBarrierException`。

**`BrokenBarrierException`**表示当前的**`CyclicBarrier`**已经损坏了，可能等不到所有线程都到达栅栏了，所以已经在等待的线程也没必要再等了，可以散伙了。

另外，**只要正在Barrier上等待的任一线程抛出了异常，那么`Barrier`就会认为肯定是凑不齐所有线程了，就会将栅栏置为损坏（Broken）状态，并传播BrokenBarrierException给其它所有正在等待（await）的线程**。

我们来对上面的例子做个改造，模拟下异常情况：

```java
public class CyclicBarrierTest {
    public static void main(String[] args) throws InterruptedException {

        int N = 5;  // 运动员数
        CyclicBarrier cb = new CyclicBarrier(N, new Runnable() {
            @Override
            public void run() {
                System.out.println("****** 所有运动员已准备完毕，发令枪：跑！******");
            }
        });

        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < N; i++) {
            Thread t = new Thread(new PrepareWork(cb), "运动员[" + i + "]");
            list.add(t);
            t.start();
            if (i == 3) {
                t.interrupt();  // 运动员[3]置中断标志位
            }
        }

        Thread.sleep(2000);
        System.out.println("Barrier是否损坏：" + cb.isBroken());
    }


    private static class PrepareWork implements Runnable {
        private CyclicBarrier cb;

        PrepareWork(CyclicBarrier cb) {
            this.cb = cb;
        }

        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + ": 准备完成");
                cb.await();
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName() + ": 被中断");
            } catch (BrokenBarrierException e) {
                System.out.println(Thread.currentThread().getName() + ": 抛出BrokenBarrierException");
            }
        }
    }
}
```

可能的输出结果：

```java
运动员[0]: 准备完成
运动员[2]: 准备完成
运动员[1]: 准备完成
运动员[3]: 准备完成
运动员[3]: 被中断
运动员[4]: 准备完成
运动员[4]: 抛出BrokenBarrierException
运动员[0]: 抛出BrokenBarrierException
运动员[1]: 抛出BrokenBarrierException
运动员[2]: 抛出BrokenBarrierException
Barrier是否损坏：true
```

这段代码，模拟了中断线程3的情况，从输出可以看到，线程0、1、2首先到达Brrier等待。
然后线程3到达，由于之前设置了中断标志位，所以线程3抛出中断异常，导致Barrier损坏，此时所有已经在栅栏等待的线程（0、1、2）都会抛出**BrokenBarrierException**异常。
此时，即使再有其它线程到达栅栏（线程4），都会抛出**BrokenBarrierException**异常。

> **注意：**使用`CyclicBarrier`时，对异常的处理一定要小心，比如线程在到达栅栏前就抛出异常，此时如果没有重试机制，其它已经到达栅栏的线程会一直等待（因为还没有满足总数），最终导致程序无法继续向下执行。

## 3、CyclicBarrier和CountDownLatch的区别

`CountDownLatch`的计数器只能使用一次，而`CyclicBarrier`的计数器可以使用`reset()`方法重置。
 所以`CyclicBarrier`能处理更为复杂的业务场景。
 例如，如果计算发生错误，可以重置计数器，并让线程重新执行一次。

`CyclicBarrier`还提供其他有用的方法，比如：

- `getNumberWaiting`方法可以获得`CyclicBarrier`阻塞的线程数量。
- `isBroken()`方法用来了解阻塞的线程是否被中断。



## 4、控制并发线程数的SemaPhore

### 4.1、简介

`Semaphore`，又名信号量，这个类的作用有点类似于“许可证”。有时，我们**因为一些原因需要控制同时访问共享资源的最大线程数量，比如出于系统性能的考虑需要限流，或者共享资源是稀缺资源，我们需要有一种办法能够协调各个线程，以保证合理的使用公共资源**，和 `CountDownLatch` 一样，其本质上是一个“**共享锁**”。

`Semaphore`维护了一个许可集，其实就是一定数量的“许可证”。
**当有线程想要访问共享资源时，需要先获取(`acquire`)的许可；如果许可不够了，线程需要一直等待，直到许可可用。当线程使用完共享资源后，可以归还(`release`)许可，以供其它需要的线程使用**。但是，不使用实际的许可对象，`Semaphore` 只对可用许可的号码进行计数，并采取相应的行动。

另外，`Semaphore`支持公平/非公平策略，这和`ReentrantLock`类似。

**`Semaphore`经常用于限制获取某种资源的线程数量。对于共享资源访问需要由锁来控制，`Semaphore`仅仅是保证了线程由权限使用共享资源，至于使用过程中是否由并发问题，需要通过锁来保证。**

**总结一下，`许可数 ≤ 0`代表共享资源不可用。`许可数 ＞ 0`，代表共享资源可用，且多个线程可以同时访问共享资源。**

**这是不是和CountDownLatch有点像？**

| 同步器         | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| CountDownLatch | 同步状态`State > 0`表示资源不可用，所有线程需要等待；`State == 0`表示资源可用，所有线程可以同时访问 |
| Semaphore      | `剩余许可数 < 0`表示资源不可用，所有线程需要等待； `许可剩余数 ≥ 0`表示资源可用，所有线程可以同时访问 |

一个停车场的简单例子来阐述 `Semaphore` ：

- 为了简单起见我们假设停车场仅有 5 个停车位。一开始停车场没有车辆所有车位全部空着，然后先后到来三辆车，停车场车位够，安排进去停车，然后又来三辆，这个时候由于只有两个停车位，所有只能停两辆，其余一辆必须在外面候着，直到停车场有空车位。当然，以后每来一辆都需要在外面候着。当停车场有车开出去，里面有空位了，则安排一辆车进去（至于是哪辆，要看选择的机制是公平还是非公平）。
- 从程序角度看，停车场就相当于信号量 Semaphore ，其中许可数为 5 ，车辆就相对线程。当来一辆车时，许可数就会减 1 。当停车场没有车位了（许可数 == 0 ），其他来的车辆需要在外面等候着。如果有一辆车开出停车场，许可数 + 1，然后放进来一辆车。
- **信号量 Semaphore 是一个非负整数（ `>=1` ）。当一个线程想要访问某个共享资源时，它必须要先获取 Semaphore。当 Semaphore > 0 时，获取该资源并使 Semaphore – 1 。如果S emaphore 值 = 0，则表示全部的共享资源已经被其他线程全部占用，线程必须要等待其他线程释放资源。当线程释放资源时，Semaphore 则 +1** 。

### 4.2、Semaphore原理

#### 4.2.1、Semaphore的内部结构

`java.util.concurrent.Semaphore` 结构如下图：

![Semaphore 结构](.\img\2018120819001.png)

从上图可以看出，`Semaphore` 内部包含公平锁（`FairSync`）和非公平锁（`NonfairSync`），继承内部类 `Sync` ，其中 `Sync` 继承 `AQS`（再一次阐述 AQS 的重要性）。

![clipboard.png](.\img\3580212759-5b69631a40274_articlex.png)

可以看到，`Semaphore`果然是通过内部类实现了AQS框架提供的接口，而且基本结构几乎和`ReentrantLock`完全一样，通过内部类分别实现了公平/非公平策略。

#### 4.2.2、Semaphore对象的构造

`Semaphore` 提供了两个构造函数。

1. `Semaphore(int permits)` ：创建具有给定的许可数和**非公平**的公平设置的 Semaphore 。
2. `Semaphore(int permits, boolean fair)` ：创建具有给定的许可数和给定的公平设置的 Semaphore 。

实现如下：

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

-  Semaphore 默认选择**非公平锁**。
- 当信号量 `Semaphore = 1` 时，它可以当作互斥锁使用。其中 0、1 就相当于它的状态：1）当 `=1` 时表示，其他线程可以获取；2）当 `=0` 时，排他，即其他线程必须要等待。

### 4.3、Semaphore的公平策略分析

我们还是通过示例来分析：

> 假设现在一共3个线程：**ThreadA**、**ThreadB**、**ThreadC**。一个许可数为2的公平策略的**Semaphore**。线程的调用顺序如下：

```java
Semaphore sm = new Semaphore (2, true);

// ThreadA: sm.acquire()

// ThreadB: sm.acquire(2)

// ThreadC: sm.acquire()

// ThreadA: sm.release()

// ThreadB: sm.release(2)
```

#### 4.3.1、创建公平策略的Semaphore对象

```java
Semaphore sm = new Semaphore(2, true);
```

可以看到，内部创建了一个**FairSync**对象，并传入许可数**permits**：

```java
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

static final class FairSync extends Sync {
    private static final long serialVersionUID = 2014338818796000944L;

    FairSync(int permits) {
        super(permits);
    }

    // FairSync.java
    @Override
    protected int tryAcquireShared(int acquires) {
        for (;;) {
            //判断该线程是否位于CLH队列的列头，从而实现公平锁
            if (hasQueuedPredecessors())
                return -1;
            //获取当前的信号量许可数
            int available = getState();

            //设置“获得acquires个信号量许可之后，剩余的信号量许可数”
            int remaining = available - acquires;

            //CAS设置信号量
            if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                return remaining;
        }
    }
}
```

通过 `#hasQueuedPredecessors()` 方法，判断该线程是否位于 CLH 队列的**列头**，从而实现公平锁。

**Sync**是**`Semaphore`**的一个内部抽象类，公平策略的**`FairSync`**和非公平策略的**`NonFairSync`**都继承该类。
可以看到，构造器传入的`permits`值就是同步状态的值，这也体现了我们在AQS系列中说过的：
AQS框架的设计思想就是分离构建同步器时的一系列关注点，它的所有操作都围绕着资源——*同步状态（`synchronization state`）*来展开，并将资源的定义和访问留给用户解决：

​	![clipboard.png](.\img\1827204768-5b6a48a34f6fd_articlex.png)

#### 4.3.2、ThreadA调用acquire方法

Semaphore 提供了 `#acquire()` 方法，来获取一个许可。

```java
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

内部调用 AQS 的 `#acquireSharedInterruptibly(int arg)` 方法，该方法以**共享模式**获取同步状态。代码如下：

```java
// AbstractQueuedSynchronizer.java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())				//响应线程中断
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)			//尝试获取锁，小于0表示获取失败
        doAcquireSharedInterruptibly(arg);	//加入等待队列
}
```

关键来看下**Semaphore**是如何实现**`tryAcquireShared`**方法的：

```java
/**
 * FairSync.java
 * 尝试获取指定数量的许可证，如果许可数不够，返回负数。
 * 如果许可数足够，则CAS更新同步状态State
 * acquires：需要的许可数。
 */
@Override
protected int tryAcquireShared(int acquires) {
    for (;;) {
        // 判断该线程是否位于CLH队列的列头，从而实现公平锁
        if (hasQueuedPredecessors())
            return -1;
        // 获取当前的信号量许可
        int available = getState();

        // 设置“获得acquires个信号量许可之后，剩余的信号量许可数”
        int remaining = available - acquires;

        // 剩余许可数remaing >= 0 才会更新State。
        // compareAndSetState方法用于更新State。
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

> 对于**`Semaphore`**来说，线程是可以一次性尝试获取多个许可的，此时只要剩余的许可数量够，最终会通过自旋操作更新成功。如果剩余许可数量不够，会返回一个负数，表示获取失败。
>

显然，**ThreadA**获取许可成功。此时，同步状态值`State == 1`，等待队列的结构如下：
![clipboard.png](.\img\1504063925-5b5d59707cbb0_articlex.png)

#### 4.3.3、ThreadB调用acquire(2)方法

这里调用`#acquire(2)`方法可知，线程绝对会进入等待队列。

带入参的**aquire**方法内部和无参的一样，都是调用了AQS的`acquireSharedInterruptibly`方法：

```java
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}

// AbstractQueuedSynchronizer.java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())				//响应线程中断
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)			//尝试获取锁，小于0表示获取失败
        doAcquireSharedInterruptibly(arg);	//加入等待队列
}
```

此时，ThreadB一样进入**tryAcquireShared**方法。不同的是，此时剩余许可数不足，因为ThreadB一次性获取2个许可，**tryAcquireShared**方法返回一个负数，表示获取失败：
`remaining = available - acquires = 1- 2 = -1;`

在 `#acquireSharedInterruptibly(int arg)` 方法中，会调用 `#tryAcquireShared(int arg)` 方法。而 `#tryAcquireShared(int arg)` 方法，由子类来实现。对于 Semaphore 而言，如果我们选择非公平模式，则调用 NonfairSync 的`#tryAcquireShared(int arg)` 方法，否则调用 FairSync 的 `#tryAcquireShared(int arg)` 方法。若 `#tryAcquireShared(int arg)` 方法返回 `< 0` 时，则会**阻塞**等待，从而实现 Semaphore 信号量不足时的阻塞。

另外，这也是为什么 `Semaphore` 在使用 AQS 时，**`state` 代表的是，剩余可获取的许可数，而不是已经使用的许可数。我们假设 `state` 代表的是已经使用的许可数，那么 `#tryAcquireShared(int arg)` 返回的结果 `= 原始许可数 - state` ，这个操作在并发情况下，会存在线程不安全的问题。所以，`state` 代表的是，剩余可获取的许可数，而不是已经使用的许可数**。

`ThreadB`会调用**`doAcquireSharedInterruptibly`**方法：

```java
// AQS.java
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);	//包装成共享节点，插入等待队列。
    boolean failed = true;
    try {
        for (;;) {								//自旋阻塞线程或尝试获取锁。
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);	//尝试获取锁
                if (r >= 0) {					//大于等于0表示获取锁
                    //将当前节点设置为AQS队列中的第一个节点，这是AQS的规则
                    //队列的头结点表示正在获取锁的节点。
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            /**
             * 对于 Semaphore 而言，如果 tryAcquireShared 返回小于 0 时，则会阻塞等待。
             */
            if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

上述方法首先通过**addWaiter**方法将ThreadB包装成一个共享结点，加入等待队列：

![clipboard.png](.\img\3006167883-5b5d5e653c0ca_articlex.png)

然后会进入自旋操作，先尝试获取一次资源，显然此时是获取失败的，然后判断是否要进入阻塞（**shouldParkAfterFailedAcquire**）：

<img src=".\img\2046238435-5b6b9605ec1f1_articlex.png" alt="clipboard.png" style="zoom:150%;" />

上述方法会先将前驱结点的状态置为**SIGNAL**，表示ThreadB需要阻塞，但在阻塞之前需要将前驱置为**SIGNAL**，以便将来可以唤醒ThreadB。

最终ThreadB会在**parkAndCheckInterrupt**中进入阻塞：
![clipboard.png](.\img\537149828-5b6b96eee8cef_articlex.png)

此时，同步状态值依然是`State == 1`，等待队列的结构如下：
![clipboard.png](.\img\2730043254-5b5d5e1c974b4_articlex.png)

#### 4.3.4、ThreadC调用acquire()方法

流程和步骤3完全相同，ThreadC被包装成结点加入等待队列后：
![clipboard.png](.\img\734986101-5b6be07bb96f3_articlex.png)

同步状态：`State == 1`

#### 4.3.5、ThreadA调用release()方法

`Semaphore`的`release`方法调用了`AQS`的`releaseShared`方法，

```java
public void release() {
    sync.releaseShared(1);
}

// AQS.java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {		//尝试一次释放锁
        doReleaseShared();
        return true;
    }
    return false;
}
```

`releaseShared(int arg)` 方法，会调用 Semaphore 内部类 Sync 的 `#tryReleaseShared(int arg)` 方法，释放同步状态。

```java
// Sync.java
protected final boolean tryReleaseShared(int releases) {
    for (;;) {  //自旋操作直到成功。
        int current = getState();
        //信号量的许可数 = 当前信号许可数 + 待释放的信号许可数
        int next = current + releases;
        if (next < current) 		// 同步状态值State超过int类型上限，溢出。
            throw new Error("Maximum permit count exceeded");
        //设置可获取的信号许可数为next
        if (compareAndSetState(current, next))		//CAS更新State值。
            return true;
    }
}
```

如该方法返回 **true** 时，代表释放同步状态成功，从而在 `#releaseShared(int args)` 方法中，调用 `#doReleaseShared()` 方法，**可唤醒阻塞等待 Semaphore 的许可的线程**。

更新完成后，`State == 2`,ThreadA会进入**doReleaseShared**方法，先将头结点状态置为0，表示即将唤醒后继结点：

![clipboard.png](.\img\1055457123-5b6be0c92cf4e_articlex.png)

此时，等待队列结构：

![clipboard.png](.\img\4026445270-5b6be17b8af23_articlex.png)

然后调用**unparkSuccessor**方法唤醒后继结点：

![clipboard.png](.\img\1479815106-5b6be189e4156_articlex.png)

此时，`ThreadB`被唤醒，会从原阻塞处继续向下执行：

![clipboard.png](.\img\112477466-5b6be19801e96_articlex.png)

此时，同步状态：`State == 2`

#### 4.3.6、ThreadB从原阻塞处继续执行

ThreadB被唤醒后，从下面开始继续往下执行，进入下一次自旋：
![clipboard.png](.\img\684856548-5b6be1b47b346_articlex-1576026767338.png)

在下一次自旋中，ThreadB调用**tryAcquireShared**方法成功获取到共享资源（State修改为0），**setHeadAndPropagate**方法把ThreadB变为头结点，
并根据传播状态判断是否要唤醒并释放后继结点：

# ![clipboard.png](.\img\1487534961-5b6be1cb9aaf8_articlex.png)

![clipboard.png](.\img\2391920370-5b6be1d90370f_articlex.png)

同步状态：`State == 0`

ThreadB会调用**doReleaseShared**方法，继续尝试唤醒后继的共享结点（也就是ThreadC），这个过程和ThreadB被唤醒完全一样：
![clipboard.png](.\img\1055457123-5b6be0c92cf4e_articlex-1576026734689.png)

![clipboard.png](.\img\1516025533-5b6be1f7574ad_articlex-1576026725685.png)

同步状态：`State == 0`

#### 4.3.7、ThreadC从原阻塞处继续执行

由于目前共享资源仍为0，所以ThreadC被唤醒后，在经过尝试获取资源失败后，又进入了阻塞：
![clipboard.png](.\img\684856548-5b6be1b47b346_articlex-1576026714851.png)

![clipboard.png](.\img\996885789-5b6be21846264_articlex.png)

#### 4.3.8、ThreadA调用release(2)方法

内部和无参的release方法一样：
![clipboard.png](.\img\952531955-5b6be24227f91_articlex.png)

更新完成后，`State == 2`,ThreadA会进入**doReleaseShared**方法，唤醒后继结点：
![clipboard.png](.\img\1055457123-5b6be0c92cf4e_articlex-1576026688636.png)

此时，等待队列结构：
![clipboard.png](.\img\1516025533-5b6be1f7574ad_articlex.png)

同步状态：`State == 2`

#### 4.3.9、ThreadC从原阻塞处继续执行

由于目前共享资源为2，所以ThreadC被唤醒后，获取资源成功：

![clipboard.png](.\img\684856548-5b6be1b47b346_articlex.png)

最终同步队列的结构如下：
![clipboard.png](.\img\3862758858-5b5db9eb27387_articlex.png)

同步状态：`State == 0`

### 4.4、Semaphore的非公平策略

**非公平**情况的 `NonfairSync` 的方法实现，代码如下：

```java
// NonfairSync.java
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

// Sync.java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

可以看到，**非公平策略不会去查看等待队列的队首是否有其它线程正在等待，而是直接尝试修改State值**。

### 4.5、其它方法

- `Semaphore(int permits)`: 创建一个指定数量的许可的信号量
- `Semaphore(int permits, boolean fair)` 创建一个指定数量的许可,并保证每个线程都是公平的,当`fair`为`true`时,信号量会安装先进先出的原则来获取许可.
- `acquire()` 在当前信号量中获取一个许可.当前线程会一直阻塞直到有一个可用的许可,或被其他线程中断.
- `acquireUninterruptibly()`: 在当前信号量中获取一个许可.当前线程会一直阻塞直到有一个可用的许可.
- `tryAcquire()` 在当前信号量尝试获取一个许可,如果有可用,则获取到这个许可,并立即返回`true`,否则立即返回`false`
- `tryAcquire(long timeout, TimeUnit unit)` 在当前信号量获取一个许可,当前线程会一直阻塞直到有一个可用的许可.或指定时间超时了,或被其他线程中断.
- `release()` 释放一个许可,把让它返回到这个信号量中.
- `acquire(int permits)` 请求指定数量的许可,如果有足够的许可可用,那么当前线程会立刻返回,如果许可不足,则当前会一直等待,直到被其他线程中断,或获取到足够的许可.
- `acquireUninterruptibly(int permits)` 请求指定数量的许可,如果有足够的许可可用,那么当前线程会立刻返回,如果许可不足,则当前会一直等待,直到获取到足够的许可.
- `tryAcquire(int permits)` 在当前信号量尝试获取指定数量的许可,如果有可用,则获取到这个许可,并立即返回`true`,否则立即返回`false`
- `tryAcquire(int permits, long timeout, TimeUnit unit)` 在指定的超时时间,当前信号量尝试获取指定数量的许可,如果有可用,则获取到这个许可,并立即返回`true`,否则立即返回`false`
- `release(int permits)` 释放指定数量的许可
- `availablePermits()` 返回当前信号量还有几个可用的许可
- `drainPermits()` 请求并立即返回当前信号量可用的全部许可
- `reducePermits(int reduction)` 根据指定的缩减量减小可用许可的数目。此方法在使用信号量来跟踪那些变为不可用资源的子类中很有用。此方法不同于 acquire，在许可变为可用的过程中，它不会阻塞等待。
- `isFair()` 返回当前的信号量时候是公平的
- `hasQueuedThreads()` 查询是否有线程正在等待获取。注意，因为同时可能发生取消，所以返回 true 并不保证有其他线程等待获取许可。此方法主要用于监视系统状态。
- `getQueueLength()` 返回正在等待获取的线程的估计数目。该值仅是估计的数字，因为在此方法遍历内部数据结构的同时，线程的数目可能动态地变化。此方法用于监视系统状态，不用于同步控制。
- `getQueuedThreads()` 返回一个 collection，包含可能等待获取的线程。因为在构造此结果的同时实际的线程 set 可能动态地变化，所以返回的 collection 仅是尽力的估计值。所返回 collection 中的元素没有特定的顺序。此方法用于加快子类的构造速度，提供更多的监视设施。

### 4.6、Semaphore示例

场景说明：

- 模拟学校食堂的窗口打饭过程
- 学校食堂有2个打饭窗口
- 学校中午有20个学生 按次序 排队打饭
- 每个人打饭时耗费时间不一样
- 有的学生耐心很好，他们会一直等待直到打到饭
- 有的学生耐心不好，他们等待时间超过了心里预期，就不再排队，而是回宿舍吃泡面了
- 有的学生耐心很好，但是突然接到通知，说是全班聚餐，所以也不用再排队，而是去吃大餐了

重点分析：

- 食堂有2个打饭窗口：需要定义一个permits=2的Semaphore对象。
- 学生 按次序 排队打饭：此Semaphore对象是公平的。
- 有20个学生：定义20个学生线程。
- 打到饭的学生：调用了acquireUninterruptibly()方法，无法被中断
- 吃泡面的学生：调用了tryAcquire(timeout,TimeUnit)方法，并且等待时间超时了
- 吃大餐的学生：调用了acquire()方法，并且被中断了

**实例代码：**

主线程类：

编写食堂打饭过程。

```java
public class SemaphoreDemo1 {

    private static final int THREAD_COUNT = 20;

    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);

    /**
     * Semaphore指定两个许可证，并且是公平的。
     */
    private static Semaphore semaphore = new Semaphore(2, true);

    public static void main(String[] args) throws InterruptedException {

        ArrayBlockingQueue<Future> queue = new ArrayBlockingQueue<>(100);

        for (int i = 0; i < THREAD_COUNT; i++) {
            if (i < 10) {
                //前10个同学都在耐心的等待打饭
                Student student = new Student("打饭学生" + i, semaphore, 0);

                QueueGetFood queueGetFood = new QueueGetFood(student);

                threadPool.submit(queueGetFood);
            } else if (i < 15) {
                //这5个学生没有耐心打饭，只会等1000毫秒
                //前10个同学都在耐心的等待打饭
                Student student = new Student("泡面学生" + i, semaphore, 1);

                QueueGetFood queueGetFood = new QueueGetFood(student);

                threadPool.submit(queueGetFood);
            } else {
                //聚餐学生
                Student student = new Student("取餐学生" + i, semaphore, 2);

                QueueGetFood queueGetFood = new QueueGetFood(student);

                Future<?> submit = threadPool.submit(queueGetFood);

                try {
                    queue.put(submit);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

        Thread.sleep(5000);

        for (Future future : queue) {
            // 尝试取消执行此任务
            // mayInterruptIfRunning：true - 如果执行该任务的线程应该被中断; 否则，正在进行的任务被允许完成
            future.cancel(true);
        }
    }
}
```

学生信息类：

```java
public class Student {

    /**
     * 学生姓名
     */
    private String name;

    /**
     * 打饭许可
     */
    private Semaphore semaphore;

    /**
     * 打饭方式：
     * 0 一直等待直到打到饭
     * 1 等了一会不耐烦了，回宿舍吃泡面了
     * 2 打饭中途被其他同学叫走了，不再等待
     */
    private int type;

    public Student(String name, Semaphore semaphore, int type) {
        this.name = name;
        this.semaphore = semaphore;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Semaphore getSemaphore() {
        return semaphore;
    }

    public void setSemaphore(Semaphore semaphore) {
        this.semaphore = semaphore;
    }

    public int getType() {
        return type;
    }

    public void setType(int type) {
        this.type = type;
    }
}
```

打饭方式策略：

```java
public class QueueGetFood implements Runnable {

    private Student student;

    private Random random = new Random();

    QueueGetFood(Student student) {
        this.student = student;
    }

    @Override
    public void run() {
        //根据打饭情形分别进行不同的处理
        switch (student.getType()) {
            case 0:
                //这个学生很有耐心，它会一直排队直到打到饭
                caseOne();
                break;
            case 1:
                //这个学生没有耐心，等了1000毫秒没打到饭，就回宿舍泡面了
                caseTwo();
                break;
            case 2:
                //打饭中途被其他同学叫走了，不再等待
                caseThree();
                break;
            default:
                break;
        }
    }

    private void caseOne() {
        /*
         * 排队
         * #acquireUninterruptibly()方法会一直等待许可证,直到当前线程获取了n个可用的许可证，则会停止等待。
         * 线程中断不会停止等待
         */
        student.getSemaphore().acquireUninterruptibly();
        //进行打饭
        try {
            Thread.sleep(random.nextInt(3000 - 1000 + 1) + 1000);

            System.out.println(student.getName() + "终于打到饭了。。。");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            student.getSemaphore().release();
        }
    }

    private void caseTwo() {
        try {
            /*
             * 排队，如果等待超时，则不再等待，回宿舍回家吃泡面。
             * #tryAcquire(int acquire, TimeUnit unit)方法：
             *  当前线程获取了可用的许可证，则会停止等待，继续执行，并返回true。
             *  当前线程等待时间timeout超时，则会停止等待，继续执行，并返回false。
             *  当前线程在timeout时间内被中断，则会抛出InterruptedException一次，并停止等待，继续执行。
             */
            if (student.getSemaphore().tryAcquire(random.nextInt(5000 - 3000 + 1) + 6000, TimeUnit.MILLISECONDS)) {
                //进行打饭
                try {
                    Thread.sleep(random.nextInt(8000 - 6000 + 1) + 1000);

                    System.out.println(student.getName() + "终于打到饭了。。。");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    student.getSemaphore().release();
                }
            } else {
                System.out.println(student.getName() + "回宿舍吃泡面。。。");
            }

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void caseThree() {
        try {
            //排队
            /*
             * #acquire()方法：
             *  当前线程获取到了许可证，则会停止等待。
             *  当前线程被中断连接，则会抛出InterruptedException异常，并停止等待，继续执行
             */
            student.getSemaphore().acquire();

            try {
                Thread.sleep(random.nextInt(3000 - 1000 + 1) + 1000);
            } catch (InterruptedException e) {
                //e.printStackTrace();
            }
            System.out.println(student.getName() + "终于打到饭了。。。");
        } catch (InterruptedException e) {
            System.out.println(student.getName() + "全班聚餐不打饭了。。。");
        } finally {
            student.getSemaphore().release();
        }
    }
}
```

运行结果：

```
打饭学生1终于打到饭了。。。
打饭学生0终于打到饭了。。。
打饭学生4终于打到饭了。。。
打饭学生2终于打到饭了。。。
取餐学生16全班聚餐不打饭了。。。
取餐学生17全班聚餐不打饭了。。。
取餐学生15全班聚餐不打饭了。。。
取餐学生19全班聚餐不打饭了。。。
取餐学生18全班聚餐不打饭了。。。
打饭学生3终于打到饭了。。。
泡面学生11终于打到饭了。。。
泡面学生12回宿舍吃泡面。。。
泡面学生14回宿舍吃泡面。。。
泡面学生10终于打到饭了。。。
打饭学生5终于打到饭了。。。
打饭学生9终于打到饭了。。。
打饭学生7终于打到饭了。。。
泡面学生13终于打到饭了。。。
打饭学生8终于打到饭了。。。
打饭学生6终于打到饭了。。。
```



## 5、Exchange

### 5.1、Exchange简介

`Exchanger`——交换器，是JDK1.5时引入的一个同步器，从字面上就可以看出，这个类的主要作用是交换数据。

Exchanger有点类似于`CyclicBarrier`，我们知道`CyclicBarrier`是一个栅栏，到达栅栏的线程需要等待其它(一定数量)的线程到达后，才能通过栅栏。

Exchanger可以看成是一个双向栅栏，如下图：

![clipboard.png](.\img\3655810457-5b7034f01b70a_articlex.png)

*`Thread1`线程到达栅栏后，会首先观察有没其它线程已经到达栅栏，如果没有就会等待，如果已经有其它线程（`Thread2`）已经到达了，就会以成对的方式交换各自携带的信息，因此Exchanger非常适合用于两个线程之间的数据交换。*

可以对元素进行配对和交换的线程的同步点。每个线程将条目上的某个方法呈现给 `exchange` 方法，与伙伴线程进行匹配，并且在返回时接收其伙伴的对象。Exchanger 可能被视为 `SynchronousQueue` 的双向形式。Exchanger 可能在应用程序（**比如遗传算法和管道设计**）中很有用。

**`Exchanger`，它允许在并发任务之间交换数据。具体来说，`Exchanger`类允许在两个线程之间定义同步点。当两个线程都到达同步点时，他们交换数据结构，因此第一个线程的数据结构进入到第二个线程中，第二个线程的数据结构进入到第一个线程中。**

### 5.2、Exchange示例

> **示例：**假设现在有1个生产者，1个消费者，如果要实现生产者-消费者模式，一般的思路是利用队列作为一个消息队列，生产者不断生产消息，然后入队；消费者不断从消息队列中取消息进行消费。如果队列满了，生产者等待，如果队列空了，消费者等待。

利用**Exchanger**实现生产者-消息者模式：

**生产者：**

```java
public class Producer implements Runnable {
    private final Exchanger<Message> exchanger;

    public Producer(Exchanger<Message> exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        Message message = new Message(null);
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(1000);

                message.setV(String.valueOf(i));
                System.out.println(Thread.currentThread().getName() + ": 生产了数据[" + i + "]");

                message = exchanger.exchange(message);

                System.out.println(Thread.currentThread().getName() + ": 交换得到数据[" + String.valueOf(message.getV()) + "]");

            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}
```

**消费者：**

```java
public class Consumer implements Runnable {
    private final Exchanger<Message> exchanger;

    public Consumer(Exchanger<Message> exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        Message msg = new Message(null);
        while (true) {
            try {
                Thread.sleep(1000);
                msg = exchanger.exchange(msg);
                System.out.println(Thread.currentThread().getName() + ": 消费了数据[" + msg.getV() + "]");
                msg.setV(null);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

}
```

**Main：**

```java
public class Main {
    public static void main(String[] args) {
        Exchanger<Message> exchanger = new Exchanger<>();
        Thread t1 = new Thread(new Consumer(exchanger), "消费者-t1");
        Thread t2 = new Thread(new Producer(exchanger), "生产者-t2");

        t1.start();
        t2.start();
    }
}
```

**输出：**

```
生产者-t2: 生产了数据[0]
生产者-t2: 交换得到数据[null]
消费者-t1: 消费了数据[0]
生产者-t2: 生产了数据[1]
消费者-t1: 消费了数据[1]
生产者-t2: 交换得到数据[null]
生产者-t2: 生产了数据[2]
消费者-t1: 消费了数据[2]
生产者-t2: 交换得到数据[null]
```

上述示例中，生产者生产了3个数据：0、1、2。通过**Exchanger**与消费者进行交换。可以看到，消费者消费完后会将空的**Message**交换给生产者。

在`Exchanger`中，如果一个线程已经到达了`exchanger`节点时，对于它的伙伴节点的情况有三种：

1. 如果它的伙伴节点在该线程到达之前已经调用了`exchanger`方法，则它会唤醒它的伙伴然后进行数据交换，得到各自数据返回。
2. 如果它的伙伴节点还没有到达交换点，则该线程将会被挂起，等待它的伙伴节点到达被唤醒，完成数据交换。
3. 如果当前线程被中断了则抛出异常，或者等待超时了，则抛出超时异常。

### 5.3、Exchange原理

#### 5.3.1、Exchange的构造

构造时，内部创建了一个**Participant**对象，**Participant**是Exchanger的一个内部类，本质就是一个[ThreadLocal](https://segmentfault.com/a/1190000015558915)，用来保存线程本地变量**Node**：

```java
public Exchanger() {
	participant = new Participant();
}

private final Participant participant;
static final class Participant extends ThreadLocal<Node> {
    public Node initialValue() { return new Node(); }
}
```

我们可以把**Node**对象理解成每个线程自身携带的交换数据。

#### 5.3.2、Exchange的成员变量

```java
private final Participant participant;
private volatile Node[] arena;
private volatile Node slot;
```

`participant`的作用是为每个线程保留唯一的一个`Node`节点。

`slot`为单个槽，`arena`为数组槽。他们都是`Node`类型。在这里可能会感觉到疑惑，`slot`作为`Exchanger`交换数据的场景，应该只需要一个就可以了啊？为何还多了一个`Participant` 和数组类型的`arena`呢？一个slot交换场所原则上来说应该是可以的，但实际情况却不是如此，**多个参与者使用同一个交换场所时，会存在严重伸缩性问题。既然单个交换场所存在问题，那么我们就安排多个，也就是数组arena**。通过数组`arena`来安排不同的线程使用不同的`slot`来降低竞争问题，并且可以保证最终一定会成对交换数据。但是Exchanger不是一来就会生成`arena`数组来降低竞争，只有当产生竞争是才会生成`arena`数组。那么**怎么将`Node`与当前线程绑定呢？`Participant` ，`Participant` 的作用就是为每个线程保留唯一的一个`Node`节点，它继承`ThreadLocal`，同时在`Node`节点中记录在`arena`中的下标`index`。**

Node节点定义如下：

```java
@sun.misc.Contended static final class Node {
    int index;              // Arena index
    int bound;              // Last recorded value of Exchanger.bound
    int collides;           // Number of CAS failures at current bound
    int hash;               // Pseudo-random for spins
    Object item;            // This thread's current item
    volatile Object match;  // Item provided by releasing thread
    volatile Thread parked; // Set to this thread when parked, else null
}
```

- index：arena的下标；
- bound：上一次记录的Exchanger.bound；
- collides：在当前bound下CAS失败的次数；
- hash：伪随机数，用于自旋；
- item：当前线程携带的数据
- match：配对线程携带的数据（后到达的线程会将自身携带的值设置到配对线程的该字段上）
- parked：此节点上的阻塞线程（先到达并阻塞的线程会设置该值为自身）

在`Node`定义中有两个变量值得思考：`bound`以及`collides`。前面提到了数组`area`是为了避免竞争而产生的，如果系统不存在竞争问题，那么完全没有必要开辟一个高效的arena来徒增系统的复杂性。首先通过单个`slot`的`exchanger`来交换数据，当探测到竞争时将安排不同的位置的`slot`来保存线程`Node`，并且可以确保没有`slot`会在同一个缓存行上。如何来判断会有竞争呢？CAS替换`slot`失败，如果失败，则通过记录冲突次数来扩展`arena`的尺寸，我们在记录冲突的过程中会跟踪“bound”的值，以及会重新计算冲突次数在`bound`的值被改变时。

#### 5.3.3、Exchanger的单槽位交换

**Exchanger**有两种数据交换的方式，当并发量低的时候，内部采用“**单槽位交换**”；并发量高的时候会采用“**多槽位交换**”。

我们先来看下**exchange**方法：

**exchange(V x)**：等待另一个线程到达此交换点（除非当前线程被中断），然后将给定的对象传送给该线程，并接收该线程的对象。方法定义如下：

```java
public V exchange(V x) throws InterruptedException {
    Object v;
    Object item = (x == null) ? NULL_ITEM : x; // translate null args
    
    /**
     * 这里的判断用来决定数据交换方式。
     * 1.单槽交换：当多槽交换数组为空时（arena==null），进行单槽交换（slotExchange）
     * 2.多槽交换：当多槽交换数组不为空时（arena != null）,或单槽交换失败，进行多槽交换			          *（arenaExchange）
     */
    if ((arena != null ||
         (v = slotExchange(item, false, 0L)) == null) &&
        ((Thread.interrupted() || // disambiguates null return
          (v = arenaExchange(item, false, 0L)) == null)))
        throw new InterruptedException();
    return (v == NULL_ITEM) ? null : (V)v;
}
```

可以看到`exchange`其实就是一个用于判断数据交换方式的方法，它的内部会根据**Exchanger**的某些字段状态来判断当前应该采用**单槽交换**（**`slotExchange`**）还是**多槽交换**（**`arenaExchange`**），整个判断的流程图如下：

![clipboard.png](.\img\2842338411-5b72b95ec64d6_articlex.png)

**`Exchanger`**`的`**arena**字段是一个**`Node`**类型的数组，代表了一个槽数组，只在多槽交换时会用到。此外，`Exchanger`还有一个**`slot`**字段，表示单槽交换结点，只在单槽交换时使用。

> **slot**字段最终会指向首个到达的线程的自身Node结点，表示线程占用了槽位。

![clipboard.png](.\img\2457671550-5b70371025bc3_articlex.png)

单槽交换示意图：

![clipboard.png](.\img\4176966548-5b72be9945e51_articlex.png)

我们来看下**Exchanger**具体是如何实现单槽交换的，单槽交换方法**slotExchange**并不复杂，**slotExchange**的入参**item**表示当前线程携带的数据，返回值正常情况下为配对线程携带的数据：

```java
/**
 * 单槽交换
 *
 * @param item 待交换的数据
 * @return 其它配对线程的数据; 如果多槽交换被激活或被中断返回null, 如果超时返回TIMED_OUT(一个Obejct对象)
 */
private final Object slotExchange(Object item, boolean timed, long ns) {
    Node p = participant.get();         // 当前线程携带的交换结点
    Thread t = Thread.currentThread();
    if (t.isInterrupted())              // 线程的中断状态检查
        return null;

    for (Node q; ; ) {
        if ((q = slot) != null) {       // slot != null, 说明已经有线程先到并占用了slot
            if (U.compareAndSwapObject(this, SLOT, q, null)) {
                Object v = q.item;      // 获取交换值
                q.match = item;         // 设置交换值
                Thread w = q.parked;
                if (w != null)          // 唤醒在此槽位等待的线程
                    U.unpark(w);
                return v;               // 交换成功, 返回结果
            }
            // CPU核数数多于1个, 且bound为0时创建arena数组，并将bound设置为SEQ大小
            if (NCPU > 1 && bound == 0 && U.compareAndSwapInt(this, BOUND, 0, SEQ))
                arena = new Node[(FULL + 2) << ASHIFT];
        } else if (arena != null)       // slot == null && arena != null
            // 单槽交换中途出现了初始化arena的操作，需要重新直接路由到多槽交换(arenaExchange)
            return null;
        else {                          // 当前线程先到, 则占用此slot
            p.item = item;
            if (U.compareAndSwapObject(this, SLOT, null, p))    // 将slot槽占用
                break;
            p.item = null;              // CAS操作失败, 继续下一次自旋
        }
    }

    // 执行到这, 说明当前线程先到达, 且已经占用了slot槽, 需要等待配对线程到达
    int h = p.hash;
    long end = timed ? System.nanoTime() + ns : 0L;
    int spins = (NCPU > 1) ? SPINS : 1;             // 自旋次数, 与CPU核数有关
    Object v;
    while ((v = p.match) == null) {                 // p.match == null表示配对的线程还未到达
        if (spins > 0) {                            // 优化操作:自旋过程中随机释放CPU
            h ^= h << 1;
            h ^= h >>> 3;
            h ^= h << 10;
            if (h == 0)
                h = SPINS | (int) t.getId();
            else if (h < 0 && (--spins & ((SPINS >>> 1) - 1)) == 0)
                Thread.yield();
        } else if (slot != p)                       // 优化操作:配对线程已经到达, 但是还未完全准备好, 所以需要再自旋等待一会儿
            spins = SPINS;
        else if (!t.isInterrupted() && arena == null &&
                (!timed || (ns = end - System.nanoTime()) > 0L)) {  //已经自旋很久了, 还是等不到配对, 此时才阻塞当前线程
            U.putObject(t, BLOCKER, this);
            p.parked = t;
            if (slot == p)
                U.park(false, ns);               // 阻塞当前线程
            p.parked = null;
            U.putObject(t, BLOCKER, null);
        } else if (U.compareAndSwapObject(this, SLOT, p, null)) {   // 超时或其他（取消）, 给其他线程腾出slot
            v = timed && ns <= 0L && !t.isInterrupted() ? TIMED_OUT : null;
            break;
        }
    }
    U.putOrderedObject(p, MATCH, null);
    p.item = null;
    p.hash = h;
    return v;
}
```

上述代码的整个流程大致如下：

<img src=".\img\3728110420-5b72c7a7becd1_articlex.png" alt="clipboard.png" style="zoom:170%;" />

**首先到达的线程：**

1. 如果当前线程是首个到达的线程，会将**slot**字段指向自身的**Node**结点，表示槽位被占用；
2. 然后，线程会自旋一段时间，如果经过一段时间的自旋还是等不到配对线程到达，就会进入阻塞。（**这里之所以不直接阻塞，而是自旋，是出于线程上下文切换开销的考虑，属于一种优化手段**）

**稍后到达的配对线程：**
如果当前线程（配对线程）不是首个到达的线程，则到达时槽（**slot**）已经被占用，此时**slot**指向首个到达线程自身的**Node**结点。配对线程会将**slot**置空，并取Node中的**item**作为交换得到的数据返回，另外，配对线程会把自身携带的数据存入**Node**的**match**字段中，并唤醒`Node.parked`所指向的线程（也就是先到达的线程）。

**首先到达的线程被唤醒：**
线程被唤醒后，由于**match**不为空（存放了配对线程携带过来的数据），所以会退出自旋，然后将**match**对应的值返回。

这样，线程A和线程B就实现了数据交换，**整个过程都没有用到同步操作**。

#### 5.3.4、Exchanger的多槽位交换

**Exchanger**最复杂的地方就是它的**多槽位交换（arenaExchange）**，我们先看下，什么时候会触发多槽位交换？
我们之前说了，并发量大的时候会触发多槽交换，这个说法并不准确。

**单槽交换（slotExchange）**中有这样一段代码：
![clipboard.png](.\img\3022563490-5b7038234c3c2_articlex.png)

也就是说，如果在单槽交换中，同时出现了**多个配对线程竞争修改slot槽位，导致某个线程CAS修改slot失败时，就会初始化arena多槽数组，后续所有的交换都会走arenaExchange**：

```java
/**
 * 多槽交换
 *
 * @param item 待交换的数据
 * @return 其它配对线程的数据; 如果被中断返回null, 如果超时返回TIMED_OUT(一个Obejct对象)
 */
private final Object arenaExchange(Object item, boolean timed, long ns) {
    Node[] a = arena;
    Node p = participant.get();                     // 当前线程携带的交换结点
    for (int i = p.index; ; ) {                     // 当前线程的arena索引
        int b, m, c;
        long j;

        // 从arena数组中选出偏移地址为(i << ASHIFT) + ABASE的元素, 即真正可用的Node
        Node q = (Node) U.getObjectVolatile(a, j = (i << ASHIFT) + ABASE);

        if (q != null && U.compareAndSwapObject(a, j, q, null)) {   // CASE1: 槽不为空，说明已经有线程到达并在等待了
            Object v = q.item;                     // 获取已经到达的线程所携带的值
            q.match = item;                        // 把当前线程携带的值交换给已经到达的线程
            Thread w = q.parked;                   // q.parked指向已经到达的线程
            if (w != null)
                U.unpark(w);                       // 唤醒已经到达的线程
            return v;
        } else if (i <= (m = (b = bound) & MMASK) && q == null) {       // CASE2: 有效槽位位置且槽位为空
            p.item = item;
            if (U.compareAndSwapObject(a, j, null, p)) {            // 占用该槽位, 成功
                long end = (timed && m == 0) ? System.nanoTime() + ns : 0L;
                Thread t = Thread.currentThread();
                for (int h = p.hash, spins = SPINS; ; ) {               // 自旋等待一段时间,看看有没其它配对线程到达该槽位
                    Object v = p.match;
                    if (v != null) {                                    // 有配对线程到达了该槽位
                        U.putOrderedObject(p, MATCH, null);
                        p.item = null;
                        p.hash = h;
                        return v;   // 返回配对线程交换过来的值
                    } else if (spins > 0) {
                        h ^= h << 1;
                        h ^= h >>> 3;
                        h ^= h << 10;
                        if (h == 0)                // initialize hash
                            h = SPINS | (int) t.getId();
                        else if (h < 0 &&          // approx 50% true
                                (--spins & ((SPINS >>> 1) - 1)) == 0)
                            Thread.yield();        // 每一次等待有两次让出CPU的时机
                    } else if (U.getObjectVolatile(a, j) != p)       // 优化操作:配对线程已经到达, 但是还未完全准备好, 所以需要再自旋等待一会儿
                        spins = SPINS;
                    else if (!t.isInterrupted() && m == 0 &&
                            (!timed || (ns = end - System.nanoTime()) > 0L)) {      // 等不到配对线程了, 阻塞当前线程
                        U.putObject(t, BLOCKER, this);
                        p.parked = t;                           // 在结点引用当前线程，以便配对线程到达后唤醒我
                        if (U.getObjectVolatile(a, j) == p)
                            U.park(false, ns);
                        p.parked = null;
                        U.putObject(t, BLOCKER, null);
                    } else if (U.getObjectVolatile(a, j) == p &&
                            U.compareAndSwapObject(a, j, p, null)) {    // 尝试缩减arena槽数组的大小
                        if (m != 0)                // try to shrink
                            U.compareAndSwapInt(this, BOUND, b, b + SEQ - 1);
                        p.item = null;
                        p.hash = h;
                        i = p.index >>>= 1;        // descend
                        if (Thread.interrupted())
                            return null;
                        if (timed && m == 0 && ns <= 0L)
                            return TIMED_OUT;
                        break;                     // expired; restart
                    }
                }
            } else                                 // 占用槽位失败
                p.item = null;
        } else {                                   // CASE3: 无效槽位位置, 需要扩容
            if (p.bound != b) {
                p.bound = b;
                p.collides = 0;
                i = (i != m || m == 0) ? m : m - 1;
            } else if ((c = p.collides) < m || m == FULL ||
                    !U.compareAndSwapInt(this, BOUND, b, b + SEQ + 1)) {
                p.collides = c + 1;
                i = (i == 0) ? m : i - 1;          // cyclically traverse
            } else
                i = m + 1;                         // grow
            p.index = i;
        }
    }
}

/**
 * 单槽交换
 *
 * @param item 待交换的数据
 * @return 其它配对线程的数据; 如果多槽交换被激活或被中断返回null, 如果超时返回TIMED_OUT(一个Obejct对象)
 */
private final Object slotExchange(Object item, boolean timed, long ns) {
    Node p = participant.get();         // 当前线程携带的交换结点
    Thread t = Thread.currentThread();
    if (t.isInterrupted())              // 线程的中断状态检查
        return null;

    for (Node q; ; ) {
        if ((q = slot) != null) {       // slot != null, 说明已经有线程先到并占用了slot
            if (U.compareAndSwapObject(this, SLOT, q, null)) {
                Object v = q.item;      // 获取交换值
                q.match = item;         // 设置交换值
                Thread w = q.parked;
                if (w != null)          // 唤醒在此槽位等待的线程
                    U.unpark(w);
                return v;               // 交换成功, 返回结果
            }
            // CPU核数数多于1个, 且bound为0时创建arena数组，并将bound设置为SEQ大小
            if (NCPU > 1 && bound == 0 && U.compareAndSwapInt(this, BOUND, 0, SEQ))
                arena = new Node[(FULL + 2) << ASHIFT];
        } else if (arena != null)       // slot == null && arena != null
            // 单槽交换中途出现了初始化arena的操作，需要重新直接路由到多槽交换(arenaExchange)
            return null;
        else {                          // 当前线程先到, 则占用此slot
            p.item = item;
            if (U.compareAndSwapObject(this, SLOT, null, p))    // 将slot槽占用
                break;
            p.item = null;              // CAS操作失败, 继续下一次自旋
        }
    }

    // 执行到这, 说明当前线程先到达, 且已经占用了slot槽, 需要等待配对线程到达
    int h = p.hash;
    long end = timed ? System.nanoTime() + ns : 0L;
    int spins = (NCPU > 1) ? SPINS : 1;             // 自旋次数, 与CPU核数有关
    Object v;
    while ((v = p.match) == null) {                 // p.match == null表示配对的线程还未到达
        if (spins > 0) {                            // 优化操作:自旋过程中随机释放CPU
            h ^= h << 1;
            h ^= h >>> 3;
            h ^= h << 10;
            if (h == 0)
                h = SPINS | (int) t.getId();
            else if (h < 0 && (--spins & ((SPINS >>> 1) - 1)) == 0)
                Thread.yield();
        } else if (slot != p)                       // 优化操作:配对线程已经到达, 但是还未完全准备好, 所以需要再自旋等待一会儿
            spins = SPINS;
        else if (!t.isInterrupted() && arena == null &&
                (!timed || (ns = end - System.nanoTime()) > 0L)) {  //已经自旋很久了, 还是等不到配对, 此时才阻塞当前线程
            U.putObject(t, BLOCKER, this);
            p.parked = t;
            if (slot == p)
                U.park(false, ns);               // 阻塞当前线程
            p.parked = null;
            U.putObject(t, BLOCKER, null);
        } else if (U.compareAndSwapObject(this, SLOT, p, null)) {   // 超时或其他（取消）, 给其他线程腾出slot
            v = timed && ns <= 0L && !t.isInterrupted() ? TIMED_OUT : null;
            break;
        }
    }
    U.putOrderedObject(p, MATCH, null);
    p.item = null;
    p.hash = h;
    return v;
}
```

多槽交换方法**arenaExchange**的整体流程和**slotExchange**类似，**主要区别在于它会根据当前线程的数据携带结点Node中的index字段计算出命中的槽位**。

如果槽位被占用，说明已经有线程先到了，之后的处理和**slotExchange**一样；

如果槽位有效且为null，说明当前线程是先到的，就占用槽位，然后按照：`spin->yield->block`这种锁升级的顺序进行优化的等待，等不到配对线程就会进入阻塞。

其次，在定位**arena**数组的有效槽位时，需要考虑缓存行的影响。由于高速缓存与内存之间是以缓存行为单位交换数据的，根据局部性原理，相邻地址空间的数据会被加载到高速缓存的同一个数据块上（缓存行），而数组是连续的（逻辑，涉及到虚拟内存）内存地址空间，因此，多个**slot**会被加载到同一个缓存行上，当一个**slot**改变时，会导致这个**slot**所在的缓存行上所有的数据（包括其他的**slot**）无效，需要从内存重新加载，影响性能。

> 需要注意的是，由于不同的JDK版本，同步工具类内部的实现细节千差万别，所以最关键的还是理解它的设计思想。**Exchanger**的设计思想和**[LongAdder](https://segmentfault.com/a/1190000015865714)**有些类似，都是通过`无锁+分散热点`的方式提升性能

### 5.4、Exchange接口声明

| 方法返回类型 | 方法描述                                                     |
| ------------ | ------------------------------------------------------------ |
| `V`          | `exchange(V x)`  等待另一个线程到达此交换点（除非当前线程为 [interrupted](../../../java/lang/Thread.html#interrupt--)  ），然后将给定对象传输给它，接收其对象作为回报。 |
| `V`          | `exchange(V x, long timeout, TimeUnit unit)`  等待另一个线程到达此交换点（除非当前线程为 [interrupted](../../../java/lang/Thread.html#interrupt--)或指定的等待时间已过），然后将给定对象传输给它，接收其对象作为回报。 |

### 5.5、白话简概

其实就是”我”和”你”(可能有多个”我”，多个”你”)在一个叫Slot的地方做交易(一手交钱，一手交货)，过程分以下步骤：

1. 我先到一个叫做Slot的交易场所交易，发现你已经到了，那我就尝试喊你交易，如果你回应了我，决定和我交易那么进入第2步；如果别人抢先一步把你喊走了，那我就进入第5步。
2. 我拿出钱交给你，你可能会接收我的钱，然后把货给我，交易结束；也可能嫌我掏钱太慢(超时)或者接个电话(中断)，TM的不卖了，走了，那我只能再找别人买货了(从头开始)。
3. 我到交易地点的时候，你不在，那我先尝试把这个交易点给占了(一屁股做凳子上…)，如果我成功抢占了单间(交易点)，那就坐这儿等着你拿货来交易，进入第4步；如果被别人抢座了，那我只能在找别的地方儿了，进入第5步。
4. 你拿着货来了，喊我交易，然后完成交易；也可能我等了好长时间你都没来，我不等了，继续找别人交易去，走的时候我看了一眼，一共没多少人，弄了这么多单间(交易地点Slot)，太TM浪费了，我喊来交易地点管理员：一共也没几个人，搞这么多单间儿干毛，给哥撤一个！。然后再找别人买货(从头开始)；或者我老大给我打了个电话，不让我买货了(中断)。
5. 我跑去喊管理员，尼玛，就一个坑交易个毛啊，然后管理在一个更加开阔的地方开辟了好多个单间，然后我就挨个来看每个单间是否有人。如果有人我就问他是否可以交易，如果回应了我，那我就进入第2步。如果我没有人，那我就占着这个单间等其他人来交易，进入第4步。
6. 如果我尝试了几次都没有成功，我就会认为，是不是我TM选的这个单间风水不好？不行，得换个地儿继续(从头开始)；如果我尝试了多次发现还没有成功，怒了，把管理员喊来：给哥再开一个单间(Slot)，加一个凳子，这么多人就这么几个破凳子够谁用！



## 6、Phaser

### 6.1、简介

`Phaser`是**JDK1.7**开始引入的一个同步工具类，适用于一些需要分阶段的任务的处理。它的功能与 **CyclicBarrier**和**CountDownLatch**有些类似，类似于一个多阶段的栅栏，并且功能更强大，我们来比较下这三者的功能：

| 同步器         | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| CountDownLatch | 倒数计数器，初始时设定计数器值，线程可以在计数器上等待，当计数器值归0后，所有等待的线程继续执行 |
| CyclicBarrier  | 循环栅栏，初始时设定参与线程数，当线程到达栅栏后，会等待其它线程的到达，当到达栅栏的总数满足指定数后，所有等待的线程继续执行 |
| Phaser         | 多阶段栅栏，可以在初始时设定参与线程数，也可以中途注册/注销参与者，当到达的参与者数量满足栅栏设定的数量后，会进行阶段升级（advance） |

**`Phaser`**中有一些比较重要的概念，理解了这些概念才能理解Phaser的功能。

#### 6.1.1、phase(阶段)

我们知道，在**`CyclicBarrier`**中，只有一个栅栏，线程在到达栅栏后会等待其它线程的到达。

`Phaser`也有栅栏，在Phaser中，栅栏的名称叫做**phase(阶段)**，在任意时间点，`Phaser`只处于某一个**phase(阶段)**，初始阶段为0，最大达到`Integerr.MAX_VALUE`，然后再次归零。当所有**`parties`**参与者都到达后，**`phase`**值会递增。

`Phaser`中的`phase`(阶段)这个概念其实和**`CyclicBarrier`**中的**Generation**很相似，只不过**Generation**没有计数。

#### 6.1.2、parties(参与者)

**`parties`(参与者)**其实就是**`CyclicBarrier`**中的参与线程的概念。

**CyclicBarrier**中的参与者在初始构造指定后就不能变更，而`Phaser`既可以在初始构造时指定参与者的数量，也可以中途通过`register`、`bulkRegister`、`arriveAndDeregister`等方法注册/注销参与者。

#### 6.1.3、arrive(到达) / advance(进阶)

`Phaser`注册完**`parties`（参与者）**之后，参与者的初始状态是**`unarrived`**的，当参与者**到达（`arrive`）**当前阶段（`phase`）后，状态就会变成**`arrived`**。当阶段的到达参与者数满足条件后（注册的数量等于到达的数量），阶段就会发生**进阶（`advance`）**——也就是`phase`值+1。

![clipboard.png](.\img\878416911-5b72af098eb7b_articlex.png)

#### 6.1.4、Termination（终止）

代表当前**`Phaser`**对象达到终止状态，有点类似于**`CyclicBarrier`**中的栅栏被破坏的概念。

#### 6.1.5、Tiering(分层)

`Phaser`支持**分层（`Tiering`）** —— 一种树形结构，通过构造函数可以指定当前待构造的`Phaser`对象的父结点。之所以引入**`Tiering`**，是因为当一个`Phaser`有大量**参与者（`parties`）**的时候，内部的同步操作会使性能急剧下降，而分层可以降低竞争，从而减小因同步导致的额外开销。

在一个分层`Phasers`的树结构中，注册和撤销子`Phaser`或父`Phaser`是自动被管理的。当一个`Phaser`的**参与者（`parties`）**数量变成0时，如果该`Phaser`有父结点，就会将它从父结点中移除。

### 6.2、Phaser示例

#### 6.2.1、示例一

通过Phaser控制多个线程的执行时机：有时候我们希望所有线程到达指定点后再同时开始执行，我们可以利用**`CyclicBarrier`**或**`CountDownLatch`**来实现，这里给出使用`Phaser`的版本。

```java
public class PhaserTest1 {
    public static void main(String[] args) {
        Phaser phaser = new Phaser();
        for (int i = 0; i < 10; i++) {
            phaser.register();                  // 注册各个参与者线程
       new Thread(new Task(phaser), "Thread-" + i).start();
        }
    }
}

class Task implements Runnable {
    private final Phaser phaser;

    Task(Phaser phaser) {
        this.phaser = phaser;
    }

    @Override
    public void run() {
        int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达
     // do something
        System.out.println(Thread.currentThread().getName() + ": 执行完任务，当前phase =" + i + "");
    }
}
```

**输出：**

```
Thread-8: 执行完任务，当前phase =1
Thread-4: 执行完任务，当前phase =1
Thread-3: 执行完任务，当前phase =1
Thread-0: 执行完任务，当前phase =1
Thread-5: 执行完任务，当前phase =1
Thread-6: 执行完任务，当前phase =1
Thread-7: 执行完任务，当前phase =1
Thread-9: 执行完任务，当前phase =1
Thread-1: 执行完任务，当前phase =1
Thread-2: 执行完任务，当前phase =1
```

以上示例中，创建了10个线程，并通过`register`方法注册Phaser的参与者数量为10，也可以在`Phaser`初始时指定`parties`数。当某个线程调用`arriveAndAwaitAdvance`方法后，**arrive**数量会加1，如果数量没有满足总数（参与者数量10），当前线程就是一直等待，当最后一个线程到达后，所有线程都会继续往下执行。

> **注意：**`arriveAndAwaitAdvance`方法是不响应中断的，也就是说即使当前线程被中断，**`arriveAndAwaitAdvance`**方法也不会返回或抛出异常，而是继续等待。如果希望能够响应中断，可以参考`awaitAdvanceInterruptibly`方法。

#### 6.2.2、示例二

用`Phaser`代替`CountDownLatch`。

`CountDownLatch`主要使用的有2个方法

- `await()`方法，可以使线程进入等待状态，在`Phaser`中，与之对应的方法是`awaitAdvance(int n)`。

- `countDown()`，使计数器减一，当计数器为0时所有等待的线程开始执行，在`Phaser`中，与之对应的方法是`arrive()`；

```java
public class Airplane {
    private Phaser phaser;
    private Random random;
    public Airplane(int peopleNum){
        phaser = new Phaser(peopleNum);
        random = new Random();
    }

    /**
     * 下机
     */
    public void getOffPlane(){
        try {
            String name = Thread.currentThread().getName();
            Thread.sleep(random.nextInt(500));
            System.out.println(name + " 在飞机在休息着....");
            Thread.sleep(random.nextInt(500));
            System.out.println(name + " 下飞机了");
            phaser.arrive();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void doWork(){

        String name = Thread.currentThread().getName();
        System.out.println(name + "准备做 清理 工作");
        // 等待指定的抵达阶段数
        phaser.awaitAdvance(phaser.getPhase());
        System.out.println("飞机的乘客都下机," + name + "可以开始做 清理 工作");

    }

}

public class TestMain {

    public static void main(String[] args) {
        String visitor = "明刚红丽黑白";
        String kongjie = "美惠花";

        Airplane airplane = new Airplane(visitor.length());
        Set<Thread> threads = new HashSet<>();
        for (int i = 0; i < visitor.length(); i ++){
            threads.add(new Thread(() -> {
                airplane.getOffPlane();
            }, "小" + visitor.charAt(i)));
        }
        for (int i = 0; i < kongjie.length(); i ++){
            threads.add(new Thread(() ->{
                airplane.doWork();
            }, "小" + kongjie.charAt(i) + "空姐"));
        }

        for (Thread thread : threads){
            thread.start();
        }
    }
}
```

运行结果

```
小花空姐准备做 清理 工作
小惠空姐准备做 清理 工作
小美空姐准备做 清理 工作
小黑 在飞机在休息着....
小明 在飞机在休息着....
小红 在飞机在休息着....
小丽 在飞机在休息着....
小刚 在飞机在休息着....
小明 下飞机了
小红 下飞机了
小黑 下飞机了
小白 在飞机在休息着....
小丽 下飞机了
小刚 下飞机了
小白 下飞机了
飞机的乘客都下机,小美空姐可以开始做 清理 工作
飞机的乘客都下机,小花空姐可以开始做 清理 工作
飞机的乘客都下机,小惠空姐可以开始做 清理 工作
```

#### 6.2.3、示例三

> 通过Phaser实现开关。在以前讲**CountDownLatch**时，我们给出过以**CountDownLatch**实现开关的示例，也就是说，我们希望一些外部条件得到满足后，然后打开开关，线程才能继续执行，我们看下如何用**Phaser**来实现此功能。

```java
public class PhaserTest2 {

    public static void main(String[] args) throws IOException {
        Phaser phaser = new Phaser(1);       // 注册主线程,当外部条件满足时,由主线程打开开关
        for (int i = 0; i < 10; i++) {
            phaser.register();                      // 注册各个参与者线程
            new Thread(new Task2(phaser), "Thread-" + i).start();
        }

        // 外部条件:等待用户输入命令
        System.out.println("Press ENTER to continue");
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        reader.readLine();

        // 打开开关
        phaser.arriveAndDeregister();
        System.out.println("主线程打开了开关");
    }
}

class Task2 implements Runnable {
    private final Phaser phaser;

    Task2(Phaser phaser) {
        this.phaser = phaser;
    }

    @Override
    public void run() {
        int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达

        // do something
        System.out.println(Thread.currentThread().getName() + ": 执行完任务，当前phase =" + i + "");
    }
}
```

**输出：**

```
主线程打开了开关
Thread-7: 执行完任务，当前phase =1
Thread-4: 执行完任务，当前phase =1
Thread-3: 执行完任务，当前phase =1
Thread-1: 执行完任务，当前phase =1
Thread-0: 执行完任务，当前phase =1
Thread-9: 执行完任务，当前phase =1
Thread-8: 执行完任务，当前phase =1
Thread-2: 执行完任务，当前phase =1
Thread-5: 执行完任务，当前phase =1
Thread-6: 执行完任务，当前phase =1
```

以上示例中，只有当用户按下回车之后，任务才真正开始执行。这里主线程Main相当于一个协调者，用来控制开关打开的时机，**`arriveAndDeregister`方法不会阻塞，该方法会将到达数加1，同时减少一个参与者数量，最终返回线程到达时的phase值。**

#### 6.2.4、示例四

> 通过**`Phaser`**控制任务的执行轮数

```java
public class PhaserTest3 {
    public static void main(String[] args) throws IOException {

        int repeats = 3;    // 指定任务最多执行的次数

        Phaser phaser = new Phaser() {
            /**
             * 覆写onAdvance()方法
             * @title onAdvance
             * @author Jjcc
             * @param phase 当前相位数在此方法进入前，该相位器是先进的
             * @param registeredParties 当前注册方的数量
             * @return boolean
             * @createTime 2019/12/16 11:47
             */
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                System.out.println("---------------PHASE[" + phase + "],Parties[" + registeredParties + "] ---------------");
                return phase + 1 >= repeats  || registeredParties == 0;
            }
        };

        for (int i = 0; i < 10; i++) {
            phaser.register();                      // 注册各个参与者线程
       new Thread(new Task3(phaser), "Thread-" + i).start();
        }
    }
}

class Task3 implements Runnable {
    private final Phaser phaser;

    Task3(Phaser phaser) {
        this.phaser = phaser;
    }

    @Override
    public void run() {
        while (!phaser.isTerminated()) {   //只要Phaser没有终止, 各个线程的任务就会一直执行
            int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达
            // do something
            System.out.println(Thread.currentThread().getName() + ": 执行完任务");
        }
    }
}
```

**输出：**

```
---------------PHASE[0],Parties[5] ---------------
Thread-4: 执行完任务
Thread-1: 执行完任务
Thread-2: 执行完任务
Thread-3: 执行完任务
Thread-0: 执行完任务
---------------PHASE[1],Parties[5] ---------------
Thread-0: 执行完任务
Thread-3: 执行完任务
Thread-1: 执行完任务
Thread-4: 执行完任务
Thread-2: 执行完任务
---------------PHASE[2],Parties[5] ---------------
Thread-2: 执行完任务
Thread-4: 执行完任务
Thread-1: 执行完任务
Thread-0: 执行完任务
Thread-3: 执行完任务
```

以上示例中，我们在创建Phaser对象时，覆写了`onAdvance`方法，这个方法类似于**CyclicBarrier**中的`barrierAction`任务。

也就是说，当最后一个参与者到达时，会触发`onAdvance`方法，入参**`phase`**表示到达时的`phase`值，**`registeredParties`**表示到达时的参与者数量，返回true表示需要终止Phaser。

我们通过`phase + 1 >= repeats` ，来控制**阶段（phase）**数的上限为2（从0开始计），最终控制了每个线程的执行任务次数为**`repeats`**次。

#### 6.2.5、示例五

前面两个例子都比较简单,现在我们还用`Phaser`一个比较高级一点用法.还是用旅游的例子
假如有这么一个场景,在旅游过程中,有可能很凑巧遇到几个朋友,然后他们听说你们在旅游,所以想要加入一起继续接下来的旅游.也有可能,在旅游过程中,突然其中有某几个人临时有事,想退出这次旅游了.在自由行的旅游,这是很常见的一些事情.如果现在我们使用`CyclicBarrier`这个类来实现,我们发现是实现不了,这是用`Phaser`就可实现这个功能.

- 首先,我们改写旅游类 `TourismRunnable`,这次改动相对比较多一点

```java
public class TourismRunnable implements Runnable{
    Phaser phaser;
    Random random;
    /**
     * 每个线程保存一个朋友计数器,比如小红第一次遇到一个朋友,则取名`小红的朋友0号`,
     * 然后旅游到其他景点的时候,如果小红又遇到一个朋友,这取名为`小红的朋友1号`
     */
    AtomicInteger frientCount = new AtomicInteger();
    public TourismRunnable(Phaser phaser) {
        this.phaser = phaser;
        this.random = new Random();
    }

    @Override
    public void run() {
        tourism();
    }

    /**
     * 旅游过程
     */
    private void tourism() {
        switch (phaser.getPhase()){
            case 0:if(!goToStartingPoint()) break;
            case 1:if(!goToHotel()) break;
            case 2:if(!goToTourismPoint1()) break;
            case 3:if(!goToTourismPoint2()) break;
            case 4:if(!goToTourismPoint3()) break;
            case 5:if(!goToEndPoint()) break;
        }
    }

    /**
     * 准备返程
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToEndPoint() {
        return goToPoint("飞机场,准备登机回家");
    }

    /**
     * 到达旅游点3
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToTourismPoint3() {
        return goToPoint("旅游点3");
    }

    /**
     * 到达旅游点2
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToTourismPoint2() {
        return goToPoint("旅游点2");
    }

    /**
     * 到达旅游点1
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToTourismPoint1() {
        return goToPoint("旅游点1");
    }

    /**
     * 入住酒店
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToHotel() {
        return goToPoint("酒店");
    }

    /**
     * 出发点集合
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToStartingPoint() {
        return goToPoint("出发点");
    }

    private int getRandomTime() throws InterruptedException {
        int time = random.nextInt(400) + 100;
        Thread.sleep(time);
        return time;
    }

    /**
     * @param point 集合点
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean goToPoint(String point){
        try {
            if(!randomEvent()){
                // #randomEvent()方法判断当前phaser是否终止；true为终止
                // #arriveAndDeregister()方法使当前线程退出，并且parties值减1
                phaser.arriveAndDeregister();
                return false;
            }
            String name = Thread.currentThread().getName();
            System.out.println(name + " 花了 " + getRandomTime() + " 时间才到了" + point);
            // #arriveAndAwaitAdvance()方法等待其它线程到达，当等待的线程数等于parties时，所有线程继续执行。
            phaser.arriveAndAwaitAdvance();
            return true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 随机事件
     * @return 返回true,说明还要继续旅游,否则就临时退出了
     */
    private boolean randomEvent() {
        int r = random.nextInt(100);
        String name = Thread.currentThread().getName();
        if (r < 10){
            int friendNum =  1;
            System.out.println(name + ":在这里竟然遇到了"+friendNum+"个朋友,他们说要一起去旅游...");
            // #bulkRegister()方法批量增加parties数量
            phaser.bulkRegister(friendNum);
            for (int i = 0; i < friendNum; i ++){
                new Thread(new TourismRunnable(phaser), name + "的朋友" + frientCount.getAndAdd(1) + "号").start();
            }
        }else if(r > 90){
            System.out.println(name + ":突然有事要离开一下,不和他们继续旅游了....");
            return false;
        }
        return true;
    }
}
```

代码解析

> - `tourism`这个方法的`case`写法看起有点怪异,如果是为了满足我们这个需求,这里的case的意思是-->`case 第几次集合: if(是否继续旅游) 若不继续则break,否则继续后面的旅游`
> - `phaser.getPhase()` 初始值为0,如果全部人到达集合点这个`Phase`+1,如果`phaser.getPhase()`达到Integer的最大值,这重新清空为0,在这里表示第几次集合了
> - `phaser.arriveAndDeregister();` 表示这个人旅游到这个景点之后,就离开这个旅游团了
> - `phaser.arriveAndAwaitAdvance();` 表示这个人在这个景点旅游完,在等待其他人
> - `phaser.bulkRegister(friendNum);` 表示这个人在这个景点遇到了`friendNum`个朋友,他们要加入一起旅游

```java
public class TestMain {

    public static void main(String[] args) {
        String name = "明刚红丽黑白";
        Phaser phaser = new SubPhaser(name.length());
        List<Thread> tourismThread = new ArrayList<>();
        for (char ch : name.toCharArray()){
            tourismThread.add(new Thread(new TourismRunnable(phaser), "小" + ch));
        }
        for (Thread thread : tourismThread){
            thread.start();
        }
    }
    public static class SubPhaser extends Phaser{
        public SubPhaser(int parties) {
            super(parties);
        }

        @Override
        protected boolean onAdvance(int phase, int registeredParties) {

            System.out.println(Thread.currentThread().getName() + ":全部"+getArrivedParties()+"个人都到齐了,现在是第"+(phase + 1)
                    +"次集合准备去下一个地方..................\n");
            return super.onAdvance(phase, registeredParties);
        }
    }
}
```

运行输出以下结果:

```
小白 花了 109 时间才到了出发点
小红 花了 135 时间才到了出发点
小丽 花了 218 时间才到了出发点
小黑 花了 297 时间才到了出发点
小明 花了 303 时间才到了出发点
小刚 花了 440 时间才到了出发点
小刚:全部6个人都到齐了,现在是第1次集合准备去下一个地方..................

小明:突然有事要离开一下,不和他们继续旅游了....
小刚:突然有事要离开一下,不和他们继续旅游了....
小红 花了 127 时间才到了酒店
小丽 花了 162 时间才到了酒店
小黑 花了 365 时间才到了酒店
小白 花了 474 时间才到了酒店
小白:全部4个人都到齐了,现在是第2次集合准备去下一个地方..................

小黑:突然有事要离开一下,不和他们继续旅游了....
小丽:突然有事要离开一下,不和他们继续旅游了....
小红 花了 348 时间才到了旅游点1
小白 花了 481 时间才到了旅游点1
小白:全部2个人都到齐了,现在是第3次集合准备去下一个地方..................

小白 花了 128 时间才到了旅游点2
小红 花了 486 时间才到了旅游点2
小红:全部2个人都到齐了,现在是第4次集合准备去下一个地方..................

小红 花了 159 时间才到了旅游点3
小白 花了 391 时间才到了旅游点3
小白:全部2个人都到齐了,现在是第5次集合准备去下一个地方..................

小白:在这里竟然遇到了1个朋友,他们说要一起去旅游...
小白 花了 169 时间才到了飞机场,准备登机回家
小红 花了 260 时间才到了飞机场,准备登机回家
小白的朋友0号 花了 478 时间才到了飞机场,准备登机回家
小白的朋友0号:全部3个人都到齐了,现在是第6次集合准备去下一个地方..................
```

#### 5.2.6、示例六

> Phaser支持分层功能，我们先来考虑下如何用利用`Phaser`的分层来实现高并发时的优化，在[示例三](https://segmentfault.com/a/1190000015979879#articleHeader9)中，我们其实创建了10个任务，然后10个线程共用一个Phaser对象，如下图：

![clipboard.png](.\img\3020558673-5b72b445d253d_articlex.png)

如果任务数继续增大，那么同步产生的开销会非常大，利用Phaser分层的功能，我们可以限定每个Phaser对象的最大使用线程（任务数），如下图：
![clipboard.png](.\img\1198662401-5b72b5de32cd5_articlex.png)

可以看到，上述Phasers其实构成了一颗多叉树，如果任务数继续增多，还可以将Phaser的叶子结点继续分裂，然后将分裂出的子结点供工作线程使用。

```java
public class PhaserTest4 {
    private static final int TASKS_PER_PHASER = 4;      // 每个Phaser对象对应的工作线程（任务）数

    public static void main(String[] args) throws IOException {

        int repeats = 3;    // 指定任务最多执行的次数
        Phaser phaser = new Phaser() {
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                System.out.println("---------------PHASE[" + phase + "],Parties[" + registeredParties + "] ---------------");
                return phase + 1 >= repeats || registeredParties == 0;
            }
        };

        Tasker[] taskers = new Tasker[10];
        build(taskers, 0, taskers.length, phaser);       // 根据任务数,为每个任务分配Phaser对象

        for (int i = 0; i < taskers.length; i++) {          // 执行任务
            Thread thread = new Thread(taskers[i]);
            thread.start();
        }
    }

    private static void build(Tasker[] taskers, int lo, int hi, Phaser phaser) {
        if (hi - lo > TASKS_PER_PHASER) {
            for (int i = lo; i < hi; i += TASKS_PER_PHASER) {
                int j = Math.min(i + TASKS_PER_PHASER, hi);
                build(taskers, i, j, new Phaser(phaser));
            }
        } else {
            for (int i = lo; i < hi; ++i)
                taskers[i] = new Tasker(i, phaser);
        }

    }
}

class Tasker implements Runnable {
    private final Phaser phaser;
    private int count;

    Tasker(Phaser phaser) {
        this.phaser = phaser;
        this.phaser.register();
    }
    Tasker(int i,Phaser phaser) {
        this.count = i;
        this.phaser = phaser;
        this.phaser.register();
    }

    @Override
    public void run() {
        while (!phaser.isTerminated()) {   //只要Phaser没有终止, 各个线程的任务就会一直执行
            int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达
            // do something
            System.out.println(Thread.currentThread().getName() + ": 执行完任务,count="+count);
        }
    }
}
```

**输出：**

```
---------------PHASE[0],Parties[3] ---------------
Thread-9: 执行完任务
Thread-6: 执行完任务
Thread-5: 执行完任务
Thread-4: 执行完任务
Thread-1: 执行完任务
Thread-0: 执行完任务
Thread-7: 执行完任务
Thread-8: 执行完任务
Thread-2: 执行完任务
Thread-3: 执行完任务
---------------PHASE[1],Parties[3] ---------------
Thread-3: 执行完任务
Thread-7: 执行完任务
Thread-0: 执行完任务
Thread-1: 执行完任务
Thread-5: 执行完任务
Thread-8: 执行完任务
Thread-2: 执行完任务
Thread-9: 执行完任务
Thread-6: 执行完任务
Thread-4: 执行完任务
---------------PHASE[2],Parties[3] ---------------
Thread-4: 执行完任务
Thread-2: 执行完任务
Thread-8: 执行完任务
Thread-0: 执行完任务
Thread-3: 执行完任务
Thread-9: 执行完任务
Thread-6: 执行完任务
Thread-7: 执行完任务
Thread-1: 执行完任务
Thread-5: 执行完任务
```

### 6.3、Phaser原理

https://segmentfault.com/a/1190000015979879#item-3

### 6.4、Phaser类接口声明

- `arriveAndAwaitAdvance()`：当前线程已经到达屏障(`parties-1`)，在此等待一段时间，等条件满足后继续向下一个屏障继续执行。
- `arriveAndDeregister()`：使当前线程退出，并且使`parties`值减1；该方法立即返回下一阶段的序号，并且其它线程需要等待的个数减一，并且把当前线程从之后需要等待的成员中移除。如果该`Phaser`是另外一个`Phaser`的子`Phaser`（层次化`Phaser`会在后文中讲到），并且该操作导致当前`Phaser`的成员数为0，则该操作也会将当前`Phaser`从其父`Phaser`中移除。
  `arrive()` 该方法不作任何等待，直接返回下一阶段的序号。
- `getPhase()`：获取的是已经到达第几个屏障。
- `onAdvance()`：每一阶段的最后一个参与者到达时，会触发`onAdvance`方法，`Phaser`都会执行其`onAdvance`方法。
- `getRegisteredParties()`：获得注册的parties数量。
- `register()`：每执行一次方法`register()`就动态添加一个`parties`值。
- `bulkRegister(int parties)`：批量增加`parties`数量，
- `getArrivedParties()和getUnarrivedParties()`：方法`getArrivedParties()`获得已经被使用的`parties`个数，方法`getUnarrivedParties()`获得未被使用的`parties`个数。
- `arrive()`：使`parties`值加1，并且不再屏障处等待，直接向下面的代码继续运行，并且`Phaser`类有计数重置功能。
- `awaitAdvance(int phase)`：如果传入参数`phase`值和当前`getPhase()`方法返回值一样，而在屏障处等待，否则继续向下面运行，有些类似于旁观者的作用，当观察的条件满足了就等待，如果条件不满足，则程序向下继续运行。
- `awaitAdvanceInterruptibly(int phase)`：效果与`awaitAdvance(int phase)`相当，唯一的不同在于若该线程在该方法等待时被中断，则该方法抛出`InterruptedException`。
- `awaitAdvanceInterruptibly(int phase,long timeout,TimeUnit unit)`：效果与`awaitAdvanceInterruptibly(int phase)`相当，区别在于如果超时则抛出`TimeoutException`。
- `forceTermination()`：强制让该`Phaser`进入终止状态。已经注册的`party`数不受影响。如果该`Phaser`有子`Phaser`，则其所有的子`Phaser`均进入终止状态。如果该`Phaser`已经处于终止状态，该方法调用不造成任何影响。
- `isTerminated()`：判断Phaser对象是否已经呈销毁状态；true--已销毁，flase--未销毁。



# 五、Executors框架

## 1、executors框架概述

### 1.1、executors框架简介

#### 1.1.1、executor简介

`java.util.concurrent.Executor` ，任务的执行者接口，线程池框架中几乎所有类都直接或者间接实现 Executor 接口，它是线程池框架的基础。

Executor 提供了一种将**“任务提交”与“任务执行”分离开来的机制(解耦任务本身和任务的执行)**，它仅提供了一个 `#execute(Runnable command)` 方法，用来执行已经提交的 Runnable 任务。代码如下：

```java
public interface Executor {
    /**
     * 执行给定的Runnable任务.
     * 根据Executor的实现不同, 具体执行方式也不相同.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be accepted for execution
     * @throws NullPointerException       if command is null
     */
    void execute(Runnable command);
}
```

我们可以像下面这样执行任务，而不必关心线程的创建：

```java
Executor executor = someExecutor;       // 创建具体的Executor对象
executor.execute(new RunnableTask1());
executor.execute(new RunnableTask2());
...
```

**Executor框架的两级调度模型**

在`HotSpot VM`的线程模型中，Java线程（`java.lang.Thread`）被 **一对一映射为本地操作系统线程**。Java线程启动时会创建一个本地操作系统线程；当该Java线程终止时，这个操作系统线程也会被回收。

操作系统会调度所有线程并将它们分配给可用的CPU。
 在上层，Java多线程程序通常把应用分解为若干个任务，然后使用用户级的调度器（`Executor框架`）将这些任务映射为固定数量的线程；在底层，操作系统内核将这些线程映射到硬件处理器上。这种两级调度模型的示意图下面有介绍。
 从下图中可以看出，应用程序通过`Executor框架`控制上层的调度；而下层的调度由操作系统内核控制，下层的调度不受应用程序的控制。

**Executor框架的结构与成员**

![img](.\img\1709375-110e230e9a24e9aa.webp)

#### 1.1.2、Executor框架的结构

- **任务**。包括被执行任务需要实现的接口：`Runnable接口` 或 `Callable接口`。

- **任务的执行**。包括任务执行机制的核心接口`Executor`，以及继承自`Executor`的`ExecutorService`接口。`Executor`框架有两个关键类实现了`ExecutorService`接口（`ThreadPoolExecutor` 和 `ScheduledThreadPoolExecutor`）。

- **异步计算的结果**。包括接口`Future`和实现`Future`接口的`FutureTask`类。

`Executor`框架包含的主要的类与接口如下图所示：

![img](.\img\1709375-194345ebb166c4b7.webp)

下面是这些类和接口的简介：

 下面是这些类和接口的简介：

- `Executor`是一个接口，它是`Executor`框架的基础，它将任务的提交与任务的执行分离开来。
- `ThreadPoolExecutor` 是线程池的核心实现类，用来执行被提交的任务。
- `ScheduledThreadPoolExecutor` 是一个实现类，可以在给定的延迟后运行命令，或者定期执行命令。`ScheduledThreadPoolExecutor`比`Timer`更灵活，功能更强大。
- `Future`接口和实现`Future`接口的`FutureTask`类，代表异步计算的结果。
- `Runnable`接口和`Callable`接口的实现类，都可以被`ThreadPoolExecutor` 或`ScheduledThreadPoolExecutor`执行。

#### 1.1.3、Executor框架的使用示意图

![img](.\img\1709375-1bde020b9a162154.webp)

主线程首先要创建实现`Runnable`或者`Callable`接口的任务对象。
工具类`Executors`可以通过以下两个方法把一个`Runnable`对象封装为一个`Callable`对象：

- `Executors.callable(Runnable task)`
- `Executors.callable(Runnable task, Object resule)`。

然后可以把`Runnable`对象直接交给`ExecutorService`执行`ExecutorService.execute(Runnable command)`；或者也可以把`Runnable`对象或`Callable`对象提交给`ExecutorService` 执行  `ExecutorService.submit(Runnable task)` 或 `ExecutorService.submit(Callabletask)`。

如果执行`ExecutorService.submit(…)`，`ExecutorService` 将返回一个实现 `Future` 接口的对象（`到目前为止的JDK中，返回的是FutureTask对象`）。由于`FutureTask`实现了`Runnable`，程序员也可以创建`FutureTask`，然后直接交给`ExecutorService`执行。

最后，主线程可以执行 `FutureTask.get()` 方法来等待任务执行完成。**主线程也可以执行
 `FutureTask.cancel(boolean mayInterruptIfRunning)`来取消此任务的执行。**

#### 1.1.4、增强的Executor——ExecutorService

`java.util.concurrent.ExcutorService` ，继承 `Executor` 接口，它是“执行者服务”接口，它是为”执行者接口 `Executor` “服务而存在的。准确的地说，`ExecutorService` 提供了”将任务提交给执行者的接口( submit 方法)”，”让执行者执行任务( invokeAll , invokeAny 方法)”的接口等等。代码如下：

![clipboard.png](.\img\88018759-5bb456296bc91_articlex.png)

可以看到，`ExecutorService`继承了`Executor`，它在`Executor`的基础上增强了对任务的控制，同时包括对自身生命周期的管理，主要有四类：

1. 关闭执行器，禁止任务的提交；
2. 监视执行器的状态；
3. 提供对异步任务的支持；
4. 提供对批处理任务的支持。

```java
public interface ExecutorService extends Executor {

    /**
     * 关闭执行器, 主要有以下特点:
     * 1. 已经提交给该执行器的任务将会继续执行, 但是不再接受新任务的提交;
     * 2. 如果执行器已经关闭了, 则再次调用没有副作用.
     */
    void shutdown();

    /**
     * 立即关闭执行器, 主要有以下特点:
     * 1. 尝试停止所有正在执行的任务, 无法保证能够停止成功, 但会尽力尝试(例如, 通过 Thread.interrupt中断任务, 但是不响应中断的任务可能无法终止);
     * 2. 暂停处理已经提交但未执行的任务;
     *
     * @return 返回已经提交但未执行的任务列表
     */
    List<Runnable> shutdownNow();

    /**
     * 如果该执行器已经关闭, 则返回true.
     */
    boolean isShutdown();

    /**
     * 判断执行器是否已经【终止】.
     * <p>
     * 仅当执行器已关闭且所有任务都已经执行完成, 才返回true.
     * 注意: 除非首先调用 shutdown 或 shutdownNow, 否则该方法永远返回false.
     */
    boolean isTerminated();

    /**
     * 阻塞调用线程, 等待执行器到达【终止】状态.
     *
     * @return {@code true} 如果执行器最终到达终止状态, 则返回true; 否则返回false
     * @throws InterruptedException if interrupted while waiting
     */
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;

    /**
     * 提交一个具有返回值的任务用于执行.
     * 注意: Future的get方法在成功完成时将会返回task的返回值.
     *
     * @param task 待提交的任务
     * @param <T>  任务的返回值类型
     * @return 返回该任务的Future对象
     * @throws RejectedExecutionException 如果任务无法安排执行
     * @throws NullPointerException       if the task is null
     */
    <T> Future<T> submit(Callable<T> task);

    /**
     * 提交一个 Runnable 任务用于执行.
     * 注意: Future的get方法在成功完成时将会返回给定的结果(入参时指定).
     *
     * @param task   待提交的任务
     * @param result 返回的结果
     * @param <T>    返回的结果类型
     * @return 返回该任务的Future对象
     * @throws RejectedExecutionException 如果任务无法安排执行
     * @throws NullPointerException       if the task is null
     */
    <T> Future<T> submit(Runnable task, T result);

    /**
     * 提交一个 Runnable 任务用于执行.
     * 注意: Future的get方法在成功完成时将会返回null.
     *
     * @param task 待提交的任务
     * @return 返回该任务的Future对象
     * @throws RejectedExecutionException 如果任务无法安排执行
     * @throws NullPointerException       if the task is null
     */
    Future<?> submit(Runnable task);

    /**
     * 执行给定集合中的所有任务, 当所有任务都执行完成后, 返回保持任务状态和结果的 Future 列表.
     * <p>
     * 注意: 该方法为同步方法. 返回列表中的所有元素的Future.isDone() 为 true.
     *
     * @param tasks 任务集合
     * @param <T>   任务的返回结果类型
     * @return 任务的Future对象列表，列表顺序与集合中的迭代器所生成的顺序相同，
     * @throws InterruptedException       如果等待时发生中断, 会将所有未完成的任务取消.
     * @throws NullPointerException       任一任务为 null
     * @throws RejectedExecutionException 如果任一任务无法安排执行
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;

    /**
     * 执行给定集合中的所有任务, 当所有任务都执行完成后或超时期满时（无论哪个首先发生）, 返回保持任务状态和结果的 Future 列表.
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;

    /**
     * 执行给定集合中的任务, 只有其中某个任务率先成功完成（未抛出异常）, 则返回其结果.
     * 一旦正常或异常返回后, 则取消尚未完成的任务.
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;

    /**
     * 执行给定集合中的任务, 如果在给定的超时期满前, 某个任务已成功完成（未抛出异常）, 则返回其结果.
     * 一旦正常或异常返回后, 则取消尚未完成的任务.
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

> `Future`对象提供了对任务异步执行的支持，也就是说调用线程无需等待任务执行完成，提交待执行的任务后，就会立即返回往下执行。然后，可以在需要时检查Future是否有结果了，如果任务已执行完毕，通过`Future.get()`方法可以获取到执行结果——`Future.get()`是阻塞方法。

#### 1.1.3、周期任务的调度——ScheduledExecutorService

在工业环境中，我们可能希望提交给执行器的某些任务能够定时执行或周期性地执行，这时我们可以自己实现`Executor`接口来创建符合我们需要的类，Doug Lea已经考虑到了这类需求，所以在ExecutorService的基础上，又提供了一个接口——`ScheduledExecutorService`，该接口也是在JDK1.5时，随着J.U.C引入的：

![clipboard.png](.\img\3848981137-5bbec29e234d8_articlex.png)

`ScheduledExecutorService`提供了一系列`schedule`方法，可以**在给定的延迟后执行提交的任务，或者每个指定的周期执行一次提交的任务**。

**示例：**

```java
public class ScheduleExecutorTest {
    public static void main(String[] args) {
        ScheduledExecutorService scheduler = someScheduler;     // 创建一个ScheduledExecutorService实例
        
        final ScheduledFuture<?> scheduledFuture = scheduler.scheduleAtFixedRate(new BeepTask(), 10, 10,
                TimeUnit.SECONDS);                              // 每隔10s蜂鸣一次
 
        scheduler.schedule(new Runnable() {
            @Override
            public void run() {
                scheduledFuture.cancel(true);
            }
        }, 1, TimeUnit.HOURS)       // 1小时后, 取消蜂鸣任务
    }
 
    private static class BeepTask implements Runnable {
        @Override
        public void run() {
            System.out.println("beep!");
        }
    }
}
```

上述示例先创建一个`ScheduledExecutorService`类型的执行器，然后利用`scheduleAtFixedRate`方法提交了一个“蜂鸣”任务，每隔10s该任务会执行一次。

**注意：**`scheduleAtFixedRate`方法返回一个`ScheduledFuture`对象，`ScheduledFuture`其实就是在Future的基础上增加了延迟的功能。通过`ScheduledFuture`，可以取消一个任务的执行，本例中我们利用`schedule`方法，设定在1小时后，执行任务的取消。

**`ScheduledExecutorService`完整的接口声明如下：**

```java
public interface ScheduledExecutorService extends ExecutorService {
 
    /**
     * 提交一个待执行的任务, 并在给定的延迟后执行该任务.
     *
     * @param command 待执行的任务
     * @param delay   延迟时间
     * @param unit    延迟时间的单位
     */
    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
 
    /**
     * 提交一个待执行的任务（具有返回值）, 并在给定的延迟后执行该任务.
     *
     * @param command 待执行的任务
     * @param delay   延迟时间
     * @param unit    延迟时间的单位
     * @param <V>     返回值类型
     */
    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
 
    /**
     * 提交一个待执行的任务.
     * 该任务在 initialDelay 后开始执行, 然后在 initialDelay+period 后执行, 接着在 initialDelay + 2 * period 后执行, 依此类推.
     *
     * @param command      待执行的任务
     * @param initialDelay 首次执行的延迟时间
     * @param period       连续执行之间的周期
     * @param unit         延迟时间的单位
     */
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
 
    /**
     * 提交一个待执行的任务.
     * 该任务在 initialDelay 后开始执行, 随后在每一次执行终止和下一次执行开始之间都存在给定的延迟.
     * 如果任务的任一执行遇到异常, 就会取消后续执行. 否则, 只能通过执行程序的取消或终止方法来终止该任务.
     *
     * @param command      待执行的任务
     * @param initialDelay 首次执行的延迟时间
     * @param delay        一次执行终止和下一次执行开始之间的延迟
     * @param unit         延迟时间的单位
     */
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
}
```

至此，Executors框架中的三个最核心的接口介绍完毕，这三个接口的关系如下图：

![clipboard.png](.\img\3724012404-5bbec224900b5_articlex.png)

### 1.2、生产executor的工厂

https://www.jianshu.com/p/8933aa93ee74

http://www.iocoder.cn/JUC/sike/ThreadPoolExecutor/

**`Executors`框架就是用来解耦任务本身与任务的执行，并提供了三个核心接口来满足使用者的需求：**

1. `Executor`：提交普通的可执行任务。
2. `ExecutorService`：提供对线程池生命周期的管理、异步任务的支持。
3. `ScheduledExecutorService`：提供对任务的周期性支持。

既然上面三种执行器只是接口，那么就一定存在具体的实现类，J.U.C提供了许多默认的接口实现，如果要用户自己去创建这些类的实例，就需要了解这些类的细节，有没有一种直接的方式，仅仅根据一些需要的特性（参数）就创建这些实例呢？因为对于用户来说，其实使用的只是这三个接口。

JDK1.5时，J.U.C中还提供了一个`Executors`类，专门用于创建上述接口的实现类对象。Executors其实就是一个简单工厂，它的所有方法都是static的，用户可以根据需要，选择需要创建的执行器实例，Executors一共提供了**五类**可供创建的Executor执行器实例。

#### 1.2.1、固定线程数的线程池

`Executors`提供了两种创建具有固定线程数的`Executor`方法，固定线程池在初始化时确定其中的线程总数，运行过程中会始终维持线程不变。**适用于为了满足资源管理的需求，而需要限制当前线程数量的应用场景，它适用于负载比较重的服务器。**

可以看到下面的两种创建方法其实都返回了一个`ThreadPoolExecutor`实例。`ThreadPoolExecutor`是一个`ExecutorService`接口的实现类，我们会在后面用专门章节讲解，现在只需要了解这是一种`Executor`，用来调度其中的线程的执行即可。

```java
/**
 * 创建一个具有固定线程数的Executor.
 */
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>());
}

/**
 * 创建一个具有固定线程数的Executor.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>(), threadFactory);

}
```

上面需要注意的是`ThreadFactory`这个接口：

```java
public interface ThreadFactory {
    Thread newThread(Runnable r);
}
```

既然返回的是一个线程池，那么就涉及线程的创建，一般我们需要通过` new Thread ()`这种方法创建一个新线程，但是我们可能希望设置一些线程属性，比如名称、守护程序状态、ThreadGroup 等等，线程池中的线程非常多，如果每个线程都这样手动配置势必非常繁琐，而**`ThreadFactory` 作为一个线程工厂可以让我们从这些繁琐的线程状态设置的工作中解放出来，还可以由外部指定ThreadFactory实例，以决定线程的具体创建方式**。

> 为什么需要用`ThreadFactory`来创建线程，而不是直接通过`new Thread()`的方式。这样做的好处是：一来`解耦对象的创建与使用`，二来`可以批量配置线程信息（优先级、线程名称、是否守护线程等），以自由设置池子中所有线程的状态。`
>

Executors提供了静态内部类，实现了`ThreadFactory`接口，最简单且常用的就是下面这个**DefaultThreadFactory** ：

```java
/**
 * 默认的线程工厂.
 */
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;
 
    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() : Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-";
    }
 
    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r, namePrefix + threadNumber.getAndIncrement(), 0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

可以看到，`DefaultThreadFactory` 初始化的时候定义了线程组、线程名称等信息，每创建一个线程，都给线程统一分配这些信息，避免了一个个手工通过`new`的方式创建线程，又可进行工厂的复用。

**`FixedThreadPool`的`execute()`方法的运行示意图如下图所示。**

![img](.\img\1709375-08b33a5234812c7a.webp)

- `图中1`：如果当前运行的线程数少于`corePoolSize`，则创建新线程来执行任务。
- `图中2`：在线程池完成预热之后（当前运行的线程数等于`corePoolSize`），将任务加入`LinkedBlockingQueue`。
- `图中3`：线程执行完`1`中的任务后，会在循环中反复从`LinkedBlockingQueue`获取任务来执行。

`FixedThreadPool`使用无界队列`LinkedBlockingQueue`作为线程池的工作队列（队列的容量为`Integer.MAX_VALUE`）。
 使用无界队列作为工作队列会对线程池带来如下影响：

- 当线程池中的线程数达到`corePoolSize`后，新任务将在无界队列中等待，因此线程池中的线程数不会超过`corePoolSize`。
- 由于上一点，使用无界队列时`maximumPoolSize`将是一个无效参数。
- 由于前面两点，使用无界队列时`keepAliveTime`将是一个无效参数。
- 由于使用无界队列，运行中的`FixedThreadPool`（未执行方法`shutdown()`或`shutdownNow()`）不会拒绝任务（不会调用`RejectedExecutionHandler.rejectedExecution`方法）。

#### 1.2.2、单个线程的线程池

除了固定线程数的线程池，`Executors`还提供了两种创建只有单个线程`Executor`的方法：

```java
/**
 * 创建一个使用单个 worker 线程的 Executor.
 */
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>()));
}
 
/**
 * 创建一个使用单个 worker 线程的 Executor.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>(), threadFactory));
}
```

可以看到，只有单个线程的线程池其实就是指定线程数为1的固定线程池，主要区别就是，返回的Executor实例用了一个`FinalizableDelegatedExecutorService`对象进行包装。**适用于需要保证顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的应用场景。**

我们来看下**FinalizableDelegatedExecutorService**，该类 只定义了一个finalize方法：

```java
static class FinalizableDelegatedExecutorService extends DelegatedExecutorService {
    FinalizableDelegatedExecutorService(ExecutorService executor) {
        super(executor);
    }
    protected void finalize() {
        super.shutdown();
    }
}
```

核心是其继承的**`DelegatedExecutorService`** ，这是一个包装类，实现了`ExecutorService`的所有方法，但是内部实现其实都委托给了传入的`ExecutorService` 实例：

```java
/**
 * ExecutorService实现类的包装类.
 */
static class DelegatedExecutorService extends AbstractExecutorService {
    private final ExecutorService e;
 
    DelegatedExecutorService(ExecutorService executor) {
        e = executor;
    }
 
    public void execute(Runnable command) {
        e.execute(command);
    }
 
    public void shutdown() {
        e.shutdown();
    }
 
    public List<Runnable> shutdownNow() {
        return e.shutdownNow();
    }
 
    public boolean isShutdown() {
        return e.isShutdown();
    }
 
    public boolean isTerminated() {
        return e.isTerminated();
    }
 
    public boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException {
        return e.awaitTermination(timeout, unit);
    }
 
    public Future<?> submit(Runnable task) {
        return e.submit(task);
    }
 
    public <T> Future<T> submit(Callable<T> task) {
        return e.submit(task);
    }
 
    public <T> Future<T> submit(Runnable task, T result) {
        return e.submit(task, result);
    }
 
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException {
        return e.invokeAll(tasks);
    }
 
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
            throws InterruptedException {
        return e.invokeAll(tasks, timeout, unit);
    }
 
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException {
        return e.invokeAny(tasks);
    }
 
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
            throws InterruptedException, ExecutionException, TimeoutException {
        return e.invokeAny(tasks, timeout, unit);
    }

}
```

> **为什么要多此一举，加上这样一个委托层？**因为**返回的`ThreadPoolExecutor`包含一些设置线程池大小的方法——比如`setCorePoolSize`，对于只有单个线程的线程池来说，我们是不希望用户通过强转的方式使用这些方法的，所以需要一个包装类，只暴露ExecutorService本身的方法**。

**`SingleThreadExecutor`的运行示意图如下图所示：**

![img](https:////upload-images.jianshu.io/upload_images/1709375-6f819c9c09de6cd6.png?imageMogr2/auto-orient/strip|imageView2/2/w/819/format/webp)

- `上图1:`如果当前运行的线程数少于`corePoolSize`（即线程池中无运行的线程），则创建一个新线程来执行任务。
- `上图2:`在线程池完成预热之后（当前线程池中有一个运行的线程），将任务加入`LinkedBlockingQueue`。
- `上图3:`线程执行完`上图1`中的任务后，会在一个无限循环中反复从`LinkedBlockingQueue`获取任务来执行。

#### 1.2.3、可缓存的线程池

有些情况下，我们虽然创建了具有一定线程数的线程池，但出**于资源利用率的考虑，可能希望在特定的时候对线程进行回收（比如线程超过指定时间没有被使用），是大小无界的线程池，适用于执行很多的短期异步任务的小程序，或者是负载较轻的服务器**。`Executors`就提供了这种类型的线程池：

```java
/**
 * 创建一个可缓存线程的Execotor.
 * 如果线程池中没有线程可用, 则创建一个新线程并添加到池中;
 * 如果有线程长时间未被使用(默认60s, 可通过threadFactory配置), 则从缓存中移除.
 */
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>());
}
 
/**
 * 创建一个可缓存线程的Execotor.
 * 如果线程池中没有线程可用, 则创建一个新线程并添加到池中;
 * 如果有线程长时间未被使用(默认60s, 可通过threadFactory配置), 则从缓存中移除.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(), threadFactory);
}
```

可以看到，返回的还是`ThreadPoolExecutor`对象，只是指定了超时时间，另外线程池中线程的数量在`[0, Integer.MAX_VALUE]`之间。

> 注意：`SynchronousQueue`队列是没有容量。与其他`BlockingQueue`不同，`SynchronousQueue`是一个不存储元素的`BlockingQueue`。**每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然**。

`CachedThreadPool`的`corePoolSize`被设置为0，即`corePool`为空；`maximumPoolSize`被设置为`Integer.MAX_VALUE`，即`maximumPool`是无界的。这里把`keepAliveTime`设置为`60L`，意味着`CachedThreadPool`中的空闲线程等待新任务的最长时间为`60秒`，空闲线程超过`60秒`后将会被终止。

`FixedThreadPool`和`SingleThreadExecutor`使用无界队列`LinkedBlockingQueue`作为线程池的工作队列。
`CachedThreadPool`使用没有容量的`SynchronousQueue`作为线程池的工作队列，但`CachedThreadPool`的`maximumPool`是无界的。
这意味着，如果主线程提交任务的速度高于`maximumPool`中线程处理任务的速度时，`CachedThreadPool`会不断创建新线程。
极端情况下，`CachedThreadPool`会因为创建过多线程而耗尽CPU和内存资源。

**`CachedThreadPool`的`execute()`方法的执行示意图如下图所示：**

![img](.\img\1709375-d6565c728133ef71.webp)

- `上图1：` 首先执行`SynchronousQueue.offer(Runnable task)`。如果当前`maximumPool`中有空闲线程正在执行`SynchronousQueue.poll(keepAliveTime，TimeUnit.NANOSECONDS)`，那么主线程执行`offer`操作与空闲线程执行的`poll`操作配对成功，主线程把任务交给空闲线程执行，`execute()`方法执行完成；否则执行下面的`步骤2`。
- `上图2：`当初始`maximumPool`为空，或者`maximumPool`中当前没有空闲线程时，将没有线程执行`SynchronousQueue.poll(keepAliveTime，TimeUnit.NANOSECONDS)`。这种情况下，`步骤1`将失败。此时`CachedThreadPool`会创建一个新线程执行任务，`execute()`方法执行完成。
- `上图3：`.在`步骤2`中新创建的线程将任务执行完后，会执行`SynchronousQueue.poll(keepAliveTime，TimeUnit.NANOSECONDS)`。这个`poll`操作会让空闲线程最多在`SynchronousQueue`中等待`60秒`钟。如果`60秒`钟内主线程提交了一个新任务（主线程执行`步骤1`），那么这个空闲线程将执行主线程提交的新任务；否则，这个空闲线程将终止。由于空闲`60秒`的空闲线程会被终止，因此长时间保持空闲的`CachedThreadPool`不会使用任何资源。

前面提到过，`SynchronousQueue`是一个没有容量的阻塞队列。每个插入操作必须等待另一个线程的对应移除操作，反之亦然。`CachedThreadPool`使用`SynchronousQueue`，把主线程提交的任务传递给空闲线程执行。
`CachedThreadPool`中任务传递的示意图如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/1709375-745c812615e44165.png?imageMogr2/auto-orient/strip|imageView2/2/w/801/format/webp)

#### 1.2.4、可延时/周期调度的线程池

如果有任务需要延迟/周期调用，就需要返回ScheduledExecutorService接口的实例，`ScheduledThreadPoolExecutor`就是实现了`ScheduledExecutorService`接口的一种`Executor`，和`ThreadPoolExecutor`一样

```java
/**
 * 创建一个具有固定线程数的 可调度Executor.
 * 它可安排任务在指定延迟后或周期性地执行.
 */
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
 
/**
 * 创建一个具有固定线程数的 可调度Executor.
 * 它可安排任务在指定延迟后或周期性地执行.
 * 在需要时使用提供的 ThreadFactory 创建新线程.
 */
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}

/**
 * 创建一个单一线程的ScheduledThreadExecutor
 * 它可安排任务在指定延迟后或周期性地执行.
 */
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new Executors.DelegatedScheduledExecutorService(new ScheduledThreadPoolExecutor(1));
}
public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory var0) {
    return new Executors.DelegatedScheduledExecutorService(new ScheduledThreadPoolExecutor(1, var0));
}
```

#### 1.2.5、Fork/Join线程池

`Fork/Join`线程池是比较特殊的一类线程池，在JDK1.7时才引入，其核心实现就是`ForkJoinPool`类。

```java
/**
 * 创建具有指定并行级别的ForkJoin线程池.
 */
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool(parallelism, ForkJoinPool.defaultForkJoinWorkerThreadFactory, null, true);
}
 
/**
 * 创建并行级别等于CPU核心数的ForkJoin线程池.
 */
public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool(Runtime.getRuntime().availableProcessors(), ForkJoinPool.defaultForkJoinWorkerThreadFactory,
            null, true);
}
```

### 1.3、总结

至此，`Executors`框架的整体结构基本就讲解完了，此时我们的脑海中应有大致如下的一幅类继承图：

![clipboard.png](.\img\1968768292-5bb459b665b53_articlex.png)

**下面来回顾一下，上面的各个接口/类的关系和作用：**

1.  **`Executor`**：

   执行器接口，也是最顶层的抽象核心接口，分离了任务和任务的执行。

2. **`ExecutorService`**：

   在`Executor`的基础上，提供了对线程池生命周期的管理，任务异步执行等功能。

3. **`ScheduledExecutorService`**：

   在`ExecutorService`的基础上提供了任务的延迟执行/周期执行等功能。

4. **`Executors`**：

   生产具体的执行器的工厂。

5. **`ThreadFactory`**：

   线程工厂，用于创建单个线程，减少手工创建线程的繁琐工作，同时能够复用工厂的特性。

6. **`AbstractExecutorService`**：

   `ExecutorService`的抽象实现，为各类执行器的实现提供基础。

7. **`ThreadPoolExecutor`**：

   线程池`Executor`，也是最常用的`Executor`，可以以线程池的方式管理线程。

8. **`ScheduledThreadPoolExecutor`**
   在`ThreadPoolExecutor`基础上，增加了对周期任务调度的支持。

9. **`ForkJoinPool`**
   Fork/Join线程池，在JDK1.7时引入，是实现`Fork/Join`框架的核心类。

**Executors**

静态工厂类，提供了 `Executor`、`ExecutorService` 、`ScheduledExecutorService`、`ThreadFactory` 、`Callable` 等类的静态工厂方法，通过这些工厂方法我们可以得到相对应的对象。

1. 创建并返回设置有常用配置字符串的 `ExecutorService` 的方法。
2. 创建并返回设置有常用配置字符串的 `ScheduledExecutorService` 的方法。
3. 创建并返回“包装的” `ExecutorService` 方法，它通过使特定于实现的方法不可访问来禁用重新配置。
4. 创建并返回 `ThreadFactory` 的方法，它可将新创建的线程设置为已知的状态。
5. 创建并返回非闭包形式的 `Callable` 的方法，这样可将其用于需要 `Callable` 的执行方法中。



## 2、ThreadPoolExecutor

### 2.1、ThreadPoolExecutor简介

通过`Executors`工厂，用户可以创建自己需要的执行器对象。`ThreadPoolExecutor`，它是J.U.C在JDK1.5时提供的一种实现了`ExecutorService`接口的执行器，或者说线程池。

![clipboard.png](.\img\1983609367-5bbcd4187218d_articlex.png)

`ThreadPoolExecutor`并没有自己直接实现`ExecutorService`接口，**因为它只是其中一种`Executor`的实现而已，所以Doug Lea把一些通用部分封装成一个抽象父类——`AbstractExecutorService`**，供J.U.C中的其它执行器继承。如果读者需要自己实现一个Executor，也可以继承该抽象类。

![clipboard.png](.\img\1208147182-5bbd6a8d56ad0_articlex.png)

#### 2.1.1、AbstractExecutorService

**`AbstractExecutorService`**提供了 `ExecutorService` 接口的默认实现——主要实现了 `submit`、`invokeAny` 、`invokeAll`这三类方法，就应该知道，`ExecutorService`的这三类方法几乎都是返回一个Future对象或Future类型的集合。而`Future`是一个接口，`AbstractExecutorService`既然实现了这些方法，必然要实现该`Future`接口，我们来看下`AbstractExecutorService`实现的**submit**方法：

```java
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
```

可以看到，上述方法将`Runnable`和返回值`value`进行了封装，通过`newTaskFor`方法，封装成了一个**`FutureTask`**对象，然后通过`execute`方法执行任务，最后返回异步任务对象。

> 这里其实是模板方法模式的运用，`execute`是抽象方法，需要由继承`AbstractExecutorService`的子类来实现。

上述需要注意的是`newTaskFor`方法，该方法创建了一个`Future`对象：

```java
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}
```

`FutureTask`其实就是`Future`接口的实现类：
![clipboard.png](.\img\3623712462-5bbd6ac6f05c6_articlex.png)

> 我们之前讲过，J.U.C中的`Future`接口是“Future模式”的多线程设计模式的实现，**可以让调用方以异步方式获取任务的执行结果**。而FutureTask便是这样一类支持异步返回结果的任务，既然是任务就需要实现Runnable接口，同时又要支持异步功能，所以又需要实现Future接口。J.U.C为了方便，新定义了一个接口——**RunnableFuture**，该接口同时继承Runnable和Future，代表支持异步处理的任务，而`FutureTask`便是它的默认实现。

#### 2.1.2、线程池简介

回到`ThreadPoolExecutor`，从该类的命名也可以看出，这是一种线程池执行器。线程池大家应该并不陌生，应用开发中经常需要用到数据库连接池，数据库连接池里维护着一些数据库连接，当应用需要连接数据库时，并不是自己创建连接，而是从连接池中获取可用连接；当关闭数据库连接时，只是将该连接还给连接池，以供复用。

而线程池也是类似的概念，**当有任务需要执行时，线程池会给该任务分配线程，如果当前没有可用线程，一般会将任务放进一个队列中，当有线程可用时，再从队列中取出任务并执行**，如下图：

![clipboard.png](.\img\1854158816-5bbd6adc3f7da_articlex.png)

线程池的引入，主要解决以下问题：

1. **减少系统因为频繁创建和销毁线程所带来的开销；**
2. **自动管理线程，对使用方透明，使其可以专注于任务的构建。**

### 2.2、ThreadPoolExecutor基本原理

#### 2.2.1、构建线程池

我们先来看下`ThreadPoolExecutor`的构造器，其实之前在讲`Executors`时已经接触过了，`Executors`工厂方法创建的三种线程池：`newFixedThreadPool`、`newSingleThreadExecutor`、`newCachedThreadPool`，内部都是通过`ThreadPoolExecutor`的下面这个构造器实例化了`ThreadPoolExecutor`对象：

```java
/**
 * 使用给定的参数创建ThreadPoolExecutor.
 *
 * @param corePoolSize    核心线程池中的最大线程数
 * @param maximumPoolSize 总线程池中的最大线程数
 * @param keepAliveTime   空闲线程的存活时间
 * @param unit            keepAliveTime的单位
 * @param workQueue       任务队列, 保存已经提交但尚未被执行的线程
 * @param threadFactory   线程工厂(用于指定如何创建一个线程)
 * @param handler         拒绝策略 (当任务太多导致工作队列满时的处理策略)
 */
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
                          BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);   // 使用纳秒保存存活时间
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

> 为了用户使用方便，`ThreadPoolExecutor`一共提供了4种构造器，但其它三种内部其实都调用了上面的构造器。
>

**共有七个参数，每个参数含义如下：**

- **`corePoolSize`**

  线程池中核心线程的数量。**当提交一个任务时，线程池会新建一个线程(核心线程)来执行任务，直到当前工作线程数等于`corePoolSize`，如果超过corePoolSize，则新建的是非核心线程**。调用了线程池的`prestartAllCoreThreads()`方法，线程池会提前创建并启动所有基本线程。**核心线程默认情况下会一直存活在线程池中，即使这个核心线程啥也不干(闲置状态)。如果指定`ThreadPoolExecutor`的`allowCoreThreadTimeOut`这个属性为`true`，那么核心线程如果不干活(闲置状态)的话，超过一定时间(时长下面参数决定)，就会被销毁掉**。

- **`maximumPoolSize`**

  线程池中允许的最大线程数。**线程池的阻塞队列满了之后，如果还有任务提交，如果当前的工作线程数小于`maximumPoolSize`，则会新建线程来执行任务。注意，如果使用的是无界队列，该参数也就没有什么效果了(无界工作队列永远不会阻塞，不会出现创建非核心线程的场景)**。

- **`keepAliveTime`**

  线程空闲的时间。线程的创建和销毁是需要代价的。线程执行完任务后不会立即销毁，而是继续存活一段时间：`keepAliveTime`。默认情况下，该参数只有在线程数大于`corePoolSize`时才会生效。

- **`unit`**

  `keepAliveTime`的单位。`TimeUnit`。

  - 天（`DAYS`）
  - 小时（`HOURS`）
  - 分钟（`MINUTES`）
  - 毫秒（`MILLISECONDS`）
  - 微秒（`MICROSECONDS`，千分之一毫秒）
  - 纳秒（`NANOSECONDS`，千分之一微秒）

- **`workQueue`**

  用来保存等待执行的任务的阻塞队列，等待的任务必须实现`Runnable`接口。我们可以选择如下几种：

  - `ArrayBlockingQueue`：可以限定队列的长度，**接收到任务的时候，如果工作线程数没有达到`corePoolSize`的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务，又如果工作线程数到了`maximumPoolSize`，并且队列也满了，则执行拒绝策略。**
  - `LinkedBlockingQueue`：基于链表结构的近似无界阻塞队列。**这个队列接收到任务的时候，如果当前工作线程数小于核心线程数，则新建线程(核心线程)处理任务；如果当前工作线程数等于核心线程数，则进入队列等待。由于这个队列没有线程最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了`maximumPoolSize`的设定失效，因为总线程数永远不会超过`corePoolSize`**
  - `SynchronousQueue`：不存储元素的阻塞队列，每个插入操作都必须等待一个移出操作，反之亦然。**这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，如果所有线程都在工作怎么办？那就新建一个线程来处理这个任务！所以为了保证不出现<线程数达到了`maximumPoolSize`而不能新建线程>的错误，使用这个类型队列的时候，`corePoolSize`指定成0，`maximumPoolSize`一般指定成`Integer.MAX_VALUE`，即无限大。**
  - `PriorityBlockingQueue`：具有优先界别的阻塞队列。
  - `DelayQueue`：队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，**只有达到了指定的延时时间，才会执行任务**

- **`threadFactory`**

  用于设置创建线程的工厂。该对象可以通过`Executors.defaultThreadFactory()`，如下：

```java
public static ThreadFactory defaultThreadFactory() {    
    return new DefaultThreadFactory();
}
```

​	返回的是DefaultThreadFactory对象，源码如下：

```java
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

​	`ThreadFactory`就是提供创建线程的功能的线程工厂。他是通过`newThread()`方法提供创建线程的功	能，`newThread()`方法创建的线程都是“非守护线程”而且“线程优先级都是Thread.NORM_PRIORITY=5”。

- **`handler`**

  `RejectedExecutionHandler`，线程池的拒绝策略。所谓**拒绝策略，是指将任务添加到线程池中时，线程池拒绝该任务所采取的相应策略。当向线程池中提交任务时，如果此时线程池中的线程已经饱和了，而且阻塞队列也已经满了，则线程池会选择一种拒绝策略来处理该任务。**

  线程池提供了四种拒绝策略：

  1. `AbortPolicy`：直接抛出异常，默认策略；
  2. `CallerRunsPolicy`：只用调用者所在的线程执行任务,重试添加当前的任务，它会自动重复调用`execute()`方法
  3. `DiscardOldestPolicy`：丢弃队列里最近的一个任务，并执行当前任务。
  4. `DiscardPolicy`：直接丢弃任务；

  **当然我们也可以实现自己的拒绝策略，例如记录日志等等，实现`RejectedExecutionHandler`接口即可。**

正是通过上述参数的组合变换，使得`Executors`工厂可以创建不同类型的线程池。这里先简要讲一下`corePoolSize`和`maximumPoolSize`这两个参数：

`ThreadPoolExecutor`在逻辑上将自身管理的线程池划分为两部分：**核心线程池（大小对应为corePoolSize）**、**非核心线程池（大小对应为`maximumPoolSize - corePoolSize`）**。
当我们向线程池提交一个任务时，将创建一个工作线程——我们称之为**Worker**，Worker在逻辑上从属于下图中的【核心线程池】或【非核心线程池】，具体属于哪一种，要根据`corePoolSize`、`maximumPoolSize`、`Worker`总数进行判断：

![clipboard.png](.\img\2202276841-5bbd6cb532011_articlex.png)

> 1. `ThreadPoolExecutor`中只有一种类型的线程，名叫**`Worker`**，它是`ThreadPoolExecutor`定义的内部类，同时封装着`Runnable`任务和执行该任务的`Thread`对象，我们称它为【工作线程】，它也是`ThreadPoolExecutor`唯一需要进行维护的线程；
> 2. 【核心线程池】【非核心线程池】都是逻辑上的概念，`ThreadPoolExecutor`在任务调度过程中会根据`corePoolSize`和`maximumPoolSize`的大小，判断应该如何调度任务.

#### 2.2.2、线程池状态和线程管理

线程有五种状态：新建，就绪，运行，阻塞，死亡，线程池同样有五种状态：`Running`, `SHUTDOWN`, `STOP`, `TIDYING`, `TERMINATED`。

```java
/**
 * 保存线程池状态和工作线程数:
 * 低29位: 工作线程数
 * 高3位 : 线程池状态
 */
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
 
private static final int COUNT_BITS = Integer.SIZE - 3;
 
// 最大线程数: 2^29-1
private static final int CAPACITY = (1 << COUNT_BITS) - 1;  // 00011111 11111111 11111111 11111111
 
// 线程池状态
private static final int RUNNING = -1 << COUNT_BITS;        // 11100000 00000000 00000000 00000000
private static final int SHUTDOWN = 0 << COUNT_BITS;        // 00000000 00000000 00000000 00000000
private static final int STOP = 1 << COUNT_BITS;            // 00100000 00000000 00000000 00000000
private static final int TIDYING = 2 << COUNT_BITS;         // 01000000 00000000 00000000 00000000
private static final int TERMINATED = 3 << COUNT_BITS;      // 01100000 00000000 00000000 00000000
```

变量**`ctl`**定义为`AtomicInteger` ，其功能非常强大，记录了“线程池中的任务数量”和“线程池的状态”两个信息。共32位，其中高3位表示”线程池状态”，低29位表示”线程池中的任务数量”。

```
RUNNING            -- 对应的高3位值是111。
SHUTDOWN       -- 对应的高3位值是000。
STOP                   -- 对应的高3位值是001。
TIDYING              -- 对应的高3位值是010。
TERMINATED     -- 对应的高3位值是011。
```

- **RUNNING** : 接受新任务, 且处理已经进入阻塞队列的任务
- **SHUTDOWN** : 不接受新任务, 但处理已经进入阻塞队列的任务
- **STOP** : 不接受新任务, 且不处理已经进入阻塞队列的任务, 同时中断正在运行的任务
- **TIDYING** : 当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为`TIDYING`状态。当线程池变为`TIDYING`状态时，会执行钩子函数`terminated()`。`terminated()`在`ThreadPoolExecutor`类中是空的，若用户想在线程池变为`TIDYING`时，进行相应的处理；可以通过重载`terminated()`函数来实现。
- **TERMINATED** : 线程池彻底终止的状态。

![线程池状态转换](.\img\20171007215630217)

另外，我们刚才也提到工作线程（`Worker`），`Worker`被定义为`ThreadPoolExecutor`的内部类，实现了AQS框架，`ThreadPoolExecutor`通过一个`HashSet`来保存工作线程：

```java
/**
 * 工作线程集合.
 */
private final HashSet<Worker> workers = new HashSet<Worker>();
```

工作线程的定义如下：

```java
/**
 * Worker表示线程池中的一个工作线程, 可以与任务相关联.
 * 由于实现了AQS框架, 其同步状态值的定义如下:
 * -1: 初始状态
 * 0:  无锁状态
 * 1:  加锁状态
 */
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
 
    /**
     * 与该Worker关联的线程.
     */
    final Thread thread;
    /**
     * 运行的任务task
     */
    Runnable firstTask;
    /**
     * Per-thread task counter
     */
    volatile long completedTasks;
 
 
    Worker(Runnable firstTask) {
        //设置AQS的同步状态private volatile int state，是一个计数器，大于0代表锁已经被获取
        setState(-1); // 初始的同步状态值
        this.firstTask = firstTask;
        // 利用ThreadFactory和 Worker这个Runnable创建的线程对象
        this.thread = getThreadFactory().newThread(this);
    }
 
    /**
     * 执行任务
     */
    public void run() {
        runWorker(this);
    }
 
    /**
     * 是否加锁
     */
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }
 
    /**
     * 尝试获取锁
     */
    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
 
    /**
     * 尝试释放锁
     */
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }
 
    public void lock() {
        acquire(1);
    }
 
    public boolean tryLock() {
        return tryAcquire(1);
    }
 
    public void unlock() {
        release(1);
    }
 
    public boolean isLocked() {
        return isHeldExclusively();
    }
 
    /**
     * 中断线程(仅任务非初始状态)
     */
    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

通过Worker的定义可以看到，每个`Worker`对象都有一个`Thread`线程对象与它相对应，当任务需要执行的时候，实际是调用内部Thread对象的`start`方法，而Thread对象是在Worker的构造器中通过`getThreadFactory().newThread(this)`方法创建的，创建的Thread将Worker自身作为任务，所以当调用Thread的`start`方法时，最终实际是调用了`Worker.run()`方法，该方法内部委托给`runWorker`方法执行任务。

#### 2.2.3、线程工厂

`ThreadFactory`用来创建单个线程，当线程池需要创建一个线程时，就要调用该类的`newThread(Runnable r)`方法创建线程（ThreadPoolExecutor中实际创建线程的时刻是在将任务包装成工作线程Worker时）。

ThreadPoolExecutor在构造时如果用户不指定ThreadFactory，则默认使用`Executors.defaultThreadFactory()`创建一个ThreadFactory，即Executors.DefaultThreadFactory：

```java
public static ThreadFactory defaultThreadFactory() {
    return new DefaultThreadFactory();
}

/**
 * 默认的线程工厂.
 */
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;
 
    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() : Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-";
    }
 
    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r, namePrefix + threadNumber.getAndIncrement(), 0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

> 这里的关键是要明白为什么需要用`ThreadFactory`来创建线程，而不是直接通过`new Thread()`的方式。这样做的好处是：**一来解耦对象的创建与使用，二来可以批量配置线程信息（优先级、线程名称、是否守护线程等），以自由设置池子中所有线程的状态。**

### 2.3、线程池的调度流程

`ExecutorService`的核心方法是**submit**方法——用于提交一个待执行的任务，它并没有覆写`ExecutorService`中的`submit`方法，而是直接沿用了父类`AbstractExecutorService`的模板，然后本身实现了`execute`方法：

```java
// AbstractExecutorService.java
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
```

`ThreadPoolExecutor`的`execute`方法定义如下：

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
 
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {        // CASE1: 工作线程数 < 核心线程池上限
        if (addWorker(command, true))             // 添加工作线程并执行
            return;
        c = ctl.get();
    }
 
    // 执行到此处, 说明工作线程创建失败 或 工作线程数≥核心线程池上限
    if (isRunning(c) && workQueue.offer(command)) {     // CASE2: 插入任务至队列
 
        // 再次检查线程池状态
        int recheck = ctl.get();
        if (!isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            //只是新建了一个没有任务的工作线程（Worker），该Worker就会从工作队列中取任务来执行（因为自			己没有绑定任务)
            addWorker(null, false);
    } else if (!addWorker(command, false))  
        // CASE3: 插入队列失败, 判断工作线程数 < 总线程池上限
        reject(command);    // 执行拒绝策略
}
```

上述`execute`的执行流程可以用下图描述：

![clipboard.png](.\img\4138540525-5bea42e74bcbf_articlex.png)

从图中可以看出，当提交一个新任务到线程池时，线程池的处理流程如下。

- **线程池判断核心线程池里的线程是否都在执行任务**。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
- **线程池判断工作队列是否已经满**。如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。
- **线程池判断线程池的线程是否都处于工作状态**。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

这里需要特别注意的是 **CASE2**中的`addWorker(null, false)`，当将任务成功添加到队列后，如果此时的工作线程数为0，就会执行这段代码。

> **一般来讲每个工作线程（`Worker`）都有一个`Runnable`任务和一个对应的执行线程`Thread`，当我们调用`addWorker`方法时，如果不传入相应的任务，那么就只是新建了一个没有任务的工作线程（`Worker`），该`Worker`就会从工作队列中取任务来执行（因为自己没有绑定任务）。如果传入了任务，新建的工作线程就会执行该任务。**

所以`execute`方法的**`CASE2`**中，将任务添加到队列后，需要判断工作线程数是否为0，如果是0那么就必须新建一个空任务的工作线程，将来在某一时刻它会去队列取任务执行，否则没有工作线程的话，该队列中的任务永远不会被执行。

> 再强调一遍，`maximumPoolSize`限定了整个线程池的大小，`corePoolSize`限定了核心线程池的大小，`corePoolSize≤maximumPoolSize`（当相等时表示为固定线程池）；`maximumPoolSize-corePoolSize`表示非核心线程池。

execute的整个执行流程关键是下面两点：

1. **如果工作线程数小于核心线程池上限（`CorePoolSize`），则直接新建一个工作线程并执行任务；**
2. **如果工作线程数大于等于CorePoolSize，则尝试将任务加入到队列等待以后执行。如果加入队列失败了（比如队列已满的情况），则在总线程池未满的情况下（`CorePoolSize ≤ 工作线程数 ＜ maximumPoolSize`）新建一个工作线程立即执行任务，否则执行拒绝策略。**

#### 2.3.1、工作线程的创建

了解了`ThreadPoolExecutor`的整个执行流程，我们来看下它是如何添加工作线程并执行任务的，`execute`方法内部调用了**`addWorker`**方法来添加工作线程并执行任务：

```java
/**
 * 添加工作线程并执行任务
 *
 * @param firstTask 如果指定了该参数, 表示将立即创建一个新工作线程执行该firstTask任务; 否则复用已有的工作线程，从工作队列中获取任务并执行
 * @param core      执行任务的工作线程归属于哪个线程池:  true-核心线程池  false-非核心线程池
 */
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (; ; ) {
        int c = ctl.get();
        int rs = runStateOf(c);             // 获取线程池状态
 
        /**
         * 这个if主要是判断哪些情况下, 线程池不再接受新任务执行, 而是直接返回.总结下, 有以下几种情况：
         * 1. 线程池状态为 STOP 或 TIDYING 或 TERMINATED: 线程池状态为上述任一一种时, 都不会再接受任务，所以直接返回
         * 2. 线程池状态≥ SHUTDOWN 且 firstTask != null: 因为当线程池状态≥ SHUTDOWN时, 不再接受新任务的提交，所以直接返回
         * 3. 线程池状态≥ SHUTDOWN 且 队列为空: 队列中已经没有任务了, 所以也就不需要执行任何任务了，可以直接返回
         */
        if (rs >= SHUTDOWN &&
                !(rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty()))
            return false;
 
        for (; ; ) {
            int wc = workerCountOf(c);      // 获取工作线程数
 
            /**
             * 这个if主要是判断工作线程数是否超限, 以下任一情况属于属于超限, 直接返回:
             * 1. 工作线程数超过最大工作线程数(2^29-1)
             * 2. 工作线程数超过核心线程池上限(入参core为true, 表示归属核心线程池)
             * 3. 工作线程数超过总线程池上限(入参core为false, 表示归属非核心线程池)
             */
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
 
            if (compareAndIncrementWorkerCount(c))  // 工作线程数加1
                break retry;                        // 跳出最外层循环
 			
            // CAS add worker 失败，再次读取ctl
            c = ctl.get();
            if (runStateOf(c) != rs)                // 线程池状态发生变化, 重新自旋判断
                continue retry;
        }
    }
 
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);                  // 将任务包装成工作线程
        final Thread t = w.thread;					// 当前线程
        if (t != null) {
            // 获取主锁：mainLock
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // 重新检查线程池状态
                int rs = runStateOf(ctl.get());
                // rs < SHUTDOWN ==> 线程处于RUNNING状态
                // 或者线程处于SHUTDOWN状态，且firstTask == null（可能是workQueue中仍有未执行完成的任务，创建没有初始任务的worker线程执行）
                if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive())               
                        throw new IllegalThreadStateException();
                    workers.add(w);          // 加入工作线程集合,workers是一个HashSet<Worker>
                    // 设置最大的池大小largestPoolSize，workerAdded设置为true
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                // 释放锁
                mainLock.unlock();
            }
            // 启动线程
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (!workerStarted)     // 创建/启动工作线程失败, 需要执行回滚操作
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

整个addWorker的逻辑并不复杂，分为两部分：
**第一部分**是一个自旋操作，主要是对线程池的状态进行一些判断，如果状态不适合接受新任务，或者工作线程数超出了限制，则直接返回false。

> 这里需要注意的就是`core`参数，为true时表示新建的工作线程在逻辑上归属于核心线程池，所以需要判断条件 `工作线程数 < corePoolSize` 是否满足；core为false时表示在新增的工作线程逻辑上属于非核心线程池，所以需要判断条件 `工作线程数 < maximumPoolSize`是否满足。

经过第一部分的过滤，**第二部分**才真正去创建工作线程并执行任务：
首先将`Runnable`任务包装成一个`Worker`对象，然后加入到一个工作线程集合中（名为`workers`的`HashSet`），最后调用工作线程中的`Thread`对象的**start**方法执行任务，其实最终是委托到`Worker`的下面方法执行：

```java
/**
 * 执行任务
 */
public void run() {
    runWorker(this);
}
```

#### 2.3.2、工作线程的执行

线程池状态和线程管理中有介绍`Worker`类，当线程`thread`启动（调用`start()`方法）时，其实就是执行`Worker`的`run()`方法，内部调用`runWorker()`。

**runWoker**用于执行任务，整体流程如下：

1. `while`循环不断的通过`getTask()`方法从队列中获取任务（如果工作线程自身携带着任务，则执行携带的任务）；
2. 控制执行线程的中断状态，保证如果线程池正在停止，则线程必须是中断状态，否则线程必须不是中断状态；
3. 调用`task.run()`执行任务；
4. 处理工作线程的退出工作。

```java
// Worker.java
private final class Worker extends AbstractQueuedSynchronizer
        implements Runnable {
    private static final long serialVersionUID = 6138294804551838833L;

    // task 的thread
    final Thread thread;

    // 运行的任务task
    Runnable firstTask;

    volatile long completedTasks;

    Worker(Runnable firstTask) {

        //设置AQS的同步状态private volatile int state，是一个计数器，大于0代表锁已经被获取
        setState(-1);
        this.firstTask = firstTask;

        // 利用ThreadFactory和 Worker这个Runnable创建的线程对象
        this.thread = getThreadFactory().newThread(this);
    }

    // 任务执行
    public void run() {
        runWorker(this);
    }

}

final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();     // 执行任务的线程
    Runnable task = w.firstTask;            // 任务, 如果是null则从队列取任务
    w.firstTask = null;
    w.unlock();                             // 允许执行线程被中断
    boolean completedAbruptly = true;       // 表示是否因为中断而导致退出
    try {
        while (task != null || (task = getTask()) != null) {    // 当task==null时会通过getTask从队列取任务
            w.lock();
 
            /**
             * 下面这个if判断的作用如下:
             * 1.保证当线程池状态为STOP/TIDYING/TERMINATED时，当前执行任务的线程wt是中断状态(因为线程池处于上述任一状态时，均不能再执行新任务)
             * 2.保证当线程池状态为RUNNING/SHUTDOWN时，当前执行任务的线程wt不是中断状态
             */
            if ((runStateAtLeast(ctl.get(), STOP) || (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                wt.interrupt();
 
            try {
                beforeExecute(wt, task);            // 钩子方法，由子类自定义实现
                Throwable thrown = null;
                try {
                    task.run();                     // 执行任务
                } catch (RuntimeException x) {
                    thrown = x;
                    throw x;
                } catch (Error x) {
                    thrown = x;
                    throw x;
                } catch (Throwable x) {
                    thrown = x;
                    throw new Error(x);
                } finally {
                    afterExecute(task, thrown);     // 钩子方法，由子类自定义实现
                }
            } finally {
                task = null;
                w.completedTasks++;     // 完成任务数+1
                w.unlock();
            }
        }
 
        // 执行到此处, 说明该工作线程自身既没有携带任务, 也没从任务队列中获取到任务
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);    // 处理工作线程的退出工作
    }

}
```

这里要特别注意第一个IF方法，该方法的核心作用，用一句话概括就是：

> 确保正在停止的线程池（`STOP/TIDYING/TERMINATED`）不再接受新任务，如果有新任务那么该任务的工作线程一定是中断状态；确保正常状态的线程池（RUNNING/SHUTDOWN），其所执行的任务都是不能被中断的。

另外，**`getTask`方法用于从任务队列中获取一个任务，如果获取不到任务，会跳出while循环，最终会通过`processWorkerExit`方法清理工作线程**。注意这里的`completedAbruptly`字段，它表示该工作线程是否是因为中断而退出，while循环的退出有以下几种可能：

1. 正常情况下，工作线程会存活着，不断从任务队列获取任务执行，如果获取不到任务了（`getTask`返回null），会置`completedAbruptly` 为`false`，然后执行清理工作——`processWorkerExit(worker,false)；`
2. 异常情况下，工作线程在执行过程中被中断或出现其它异常，会置`completedAbruptly` 为`true`，也会执行清理工作——`processWorkerExit(worker,true)；`

#### 2.3.3、工作线程的清理

通过上面的讨论，我们知道工作线程是在**`processWorkerExit`**中被清理的，来看下定义：

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly)          // 工作线程因异常情况而退出
        decrementWorkerCount();     // 工作线程数减1(如果工作线程执行时没有出现异常, 在getTask()方法中已经对线程数减1了)
 
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks; // completedTaskCount记录线程池完成的总任务数
        workers.remove(w);                      // 从工作线程集合中移除(该工作线程会自动被GC回收)
    } finally {
        mainLock.unlock();
    }
 
    tryTerminate();                             // 根据线程池状态, 判断是否需要终止线程池
 
    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {            // 如果线程池状态为RUNNING/SHUTDOWN
        if (!completedAbruptly) {               // 工作线程为正常退出
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            // 如果min ==0 或者workerQueue为空，min = 1
            if (min == 0 && !workQueue.isEmpty())
                min = 1;
            // 如果线程数量大于最少数量min，直接返回，不需要新增线程
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        addWorker(null, false);  // 添加一个没有firstTask的worker
    }
}
```

`processWorkerExit`的作用就是将该退出的工作线程清理掉，然后看下线程池是否需要终止。

`processWorkerExit`执行完之后，整个工作线程的生命周期也结束了，我们可以通过下图来回顾下它的整个生命周期：

![clipboard.png](.\img\2747215335-5bbd73760153c_articlex.png)

#### 2.3.4、任务的获取

最后，我们来看下任务的获取，也就是**runWorker**中使用的`getTask`方法：

```java
private Runnable getTask() {
    boolean timedOut = false;       // 表示上次从阻塞队列中取任务时是否超时
 
    for (; ; ) {
        int c = ctl.get();
        int rs = runStateOf(c);     // 获取线程池状态
 
        /**
         * 以下IF用于判断哪些情况下不允许再从队列获取任务:
         * 1. 线程池进入停止状态（STOP/TIDYING/TERMINATED）, 此时即使队列中还有任务未执行, 也不再执行
         * 2. 线程池非RUNNING状态, 且队列为空
         */
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount(); // 工作线程数减1
            return null;
        }
 
        int wc = workerCountOf(c);  // 获取工作线程数
 
        /**
         * timed变量用于判断是否需要进行超时控制:
         * 对于核心线程池中的工作线程, 除非设置了allowCoreThreadTimeOut==true, 否则不会超时回收;
         * 对于非核心线程池中的工作线程, 都需要超时控制
         */
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
 
        // 这里主要是当外部通过setMaximumPoolSize方法重新设置了最大线程数时,需要回收多出的工作线程
        if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }
 
        try {
            Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
            if (r != null)
                return r;
            timedOut = true;    // 超时仍未获取到任务
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

`timed == true`，调用`poll()`方法，如果在`keepAliveTime`时间内还没有获取task的话，则返回null，继续循环。`timed == false`，则调用`take()`方法，该方法为一个阻塞方法，没有任务时会一直阻塞挂起，直到有任务加入时对该线程唤醒，返回任务。

在`runWorker()`方法中，无论最终结果如何，都会执行`processWorkerExit()`方法对`worker`进行退出处理。

getTask方法的主要作用就是：**通过自旋，不断地尝试从阻塞队列中获取一个任务，如果获取失败则返回null。**

阻塞队列就是在我们构建`ThreadPoolExecutor`对象时，在构造器中指定的。由于队列是外部指定的，所以根据阻塞队列的特性不同，getTask方法的执行情况也不同。我们曾经在[J.U.C之collections框架](https://segmentfault.com/a/1190000016460411)系列中全面剖析过J.U.C中的所有阻塞队列：

| 队列特性 | 有界队列           | 近似无界队列                             | 无界队列            | 特殊队列                          |
| -------- | ------------------ | ---------------------------------------- | ------------------- | --------------------------------- |
| 有锁算法 | ArrayBlockingQueue | LinkedBlockingQueue、LinkedBlockingDeque | /                   | PriorityBlockingQueue、DelayQueue |
| 无锁算法 | /                  | /                                        | LinkedTransferQueue | SynchronousQueue                  |

我们可以根据业务需求、任务特点等选择上表中的某一种阻塞队列，根据Oracle官方文档的提示，任务在阻塞队列中排队一共有三种情况：

**1.直接提交**

即直接将任务提交给等待的工作线程，这时可以选择**`SynchronousQueue`**。因为`SynchronousQueue`是没有容量的，而且采用了无锁算法，所以性能较好，但是每个入队操作都要等待一个出队操作，反之亦然。

> **使用`SynchronousQueue`时，当核心线程池满了以后，如果不存在空闲的工作线程，则试图把任务加入队列将立即失败（execute方法中使用了队列的offer方法进行入队操作，而SynchronousQueue在调用offer时如果没有另一个线程等待出队操作，则会立即返回false），因此会构造一个新的工作线程（未超出最大线程池容量时）**。
> 由于，核心线程池是很容易满的，所以当使用`SynchronousQueue`时，一般需要将`maximumPoolSizes` 设置得比较大，否则入队很容易失败，最终导致执行拒绝策略，这也是为什么`Executors`工作默认提供的缓存线程池使用SynchronousQueue作为任务队列的原因。

**2.无界任务队列**

无界任务队列我们的选择主要有**`LinkedTransferQueue`**、**`LinkedBlockingQueue`**（近似无界，构造时不指定容量即可），**从性能角度来说`LinkedTransferQueue`采用了无锁算法，高并发环境下性能相对更好，但如果只是做任务队列使用相差并不大。**

> **使用无界队列需要特别注意系统资源的消耗情况，因为当核心线程池满了以后，会首先尝试将任务放入队列，由于是无界队列所以几乎一定会成功(`maximumPoolSize`将会无效，不会创建非核心线程池)，那么系统瓶颈其实就是硬件了。如果任务的创建速度远快于工作线程处理任务的速度，那么最终会导致系统资源耗尽**。`Executors`工厂中创建固定线程池的方法内部就是用了`LinkedBlockingQueue`。

**3.有界任务队列**

有界任务队列，比如**`ArrayBlockingQueue`** ，可以防止资源耗尽的情况。当核心线程池满了以后，如果队列也满了，则会创建归属于非核心线程池的工作线程，如果非核心线程池也满了 ，才会执行拒绝策略。

#### 2.3.5、拒绝策略

`ThreadPoolExecutor`在以下两种情况下会执行拒绝策略：

1. 当核心线程池满了以后，如果任务队列也满了，首先判断非核心线程池有没满，没有满就创建一个工作线程（归属非核心线程池）， 否则就会执行拒绝策略；
2. 提交任务时，`ThreadPoolExecutor`已经关闭了。

 所谓拒绝策略，就是在构造ThreadPoolExecutor时，传入的**`RejectedExecutionHandler`**对象：

```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

`ThreadPoolExecutor`一共提供了4种拒绝策略：

**1.AbortPolicy（默认）**

AbortPolicy策略其实就是抛出一个**`RejectedExecutionException`**异常：

```java
public static class AbortPolicy implements RejectedExecutionHandler {
    public AbortPolicy() {
    }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                " rejected from " +
                e.toString());
    }
}
```

**2.DiscardPolicy**

`DiscardPolicy`策略其实就是无为而治，什么都不做，等任务自己被回收：

```java
public static class DiscardPolicy implements RejectedExecutionHandler {
    public DiscardPolicy() {
    }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```

**3.DiscardOldestPolicy**

`DiscardOldestPolicy`策略是丢弃任务队列中的最近一个任务，并执行当前任务：

```java
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() {
    }
 
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {      // 线程池未关闭(RUNNING)
            e.getQueue().poll();    // 丢弃任务队列中的最近任务
            e.execute(r);           // 执行当前任务
        }
    }
}
```

**4.CallerRunsPolicy**

`CallerRunsPolicy`策略相当于以自身线程来执行任务，这样可以减缓新任务提交的速度。

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    public CallerRunsPolicy() {
    }
 
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {  // 线程池未关闭(RUNNING)
            r.run();            // 执行当前任务
        }
    }
}
```

### 2.4、线程池的关闭

`ExecutorService`接口提供两种方法来关闭线程池，这两种方法的区别主要在于是否会继续处理已经添加到任务队列中的任务。

#### 2.4.1、shutdown

`shutdown`方法将线程池切换到**`SHUTDOWN`**状态（如果已经停止，则不用切换），并调用`interruptIdleWorkers`方法中断所有空闲的工作线程，最后调用`tryTerminate`尝试结束线程池：

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(SHUTDOWN);  // 如果线程池为RUNNING状态, 则切换为SHUTDOWN状态
        interruptIdleWorkers();     // 中断所有空闲线程
        onShutdown();               // 钩子方法, 由子类实现
    } finally {
        mainLock.unlock();
    }
    tryTerminate();                 
}
```

> 这里要注意，如果执行`Runnable`任务的线程本身不响应中断，那么也就没有办法终止任务。

#### 2.4.2、shutdownNow

`shutdownNow`方法的主要不同之处就是，它会将线程池的状态至少置为**`STOP`**，同时中断所有工作线程（无论该线程是空闲还是运行中），同时返回任务队列中的所有任务。

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);  // 如果线程池为RUNNING或SHUTDOWN状态, 则切换为STOP状态
        interruptWorkers();     // 中断所有工作线程
        tasks = drainQueue();   // 抽空任务队列中的所有任务
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

> 与`shutdown`不同，`shutdownNow`会调用`interruptWorkers()`方法中断所有线程。
>
> 同时会调用`drainQueue()`方法返回等待执行到任务列表。

### 2.5、合理地配置线程池

要想合理地配置线程池，就必须首先分析任务特性，可以从以下几个角度来分析：

- **任务的性质**：CPU密集型任务、IO密集型任务和混合型任务。
- **任务的优先级**：高、中和低。
- **任务的执行时间**：长、中和短。
- **任务的依赖性**：是否依赖其他系统资源，如数据库连接。

任务一般可分为：CPU密集型、IO密集型、混合型，对于不同类型的任务需要分配不同大小的线程池。

- **CPU密集型任务** 
  **尽量使用较小的线程池，一般为CPU核心数+1。** 
  因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。
- **IO密集型任务** 
  **可以使用稍大的线程池，一般为`2*CPU`核心数。** 
  IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。
- **混合型任务** 
  **可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。** 
  只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。 
  因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。
  - 可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的`CPU`个数。

优先级不同的任务可以使用优先级队列`PriorityBlockingQueue`来处理。它可以让优先级高的任务先执行。

> 注意：如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。

**依赖数据库连接池的任务**，因为线程提交SQL后 **需要等待数据库返回结果**，等待的时间越长，则CPU空闲时间就越长，**那么线程数应该设置得越大，这样才能更好地利用CPU。**

**建议使用有界队列**。有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点儿，比如几千。

### 2.6、线程池的监控

如果在系统中 **大量使用线程池**，则有必要 **对线程池进行监控**，方便在出现问题时，可以根据线程池的使用状况快速定位问题。可以通过线程池提供的参数进行监控，在监控线程池的时候可以使用以下属性：

`taskCount`：线程池需要执行的任务数量。

`completedTaskCount`：线程池在运行过程中已完成的任务数量，小于或等于`taskCount`。

`largestPoolSize`：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。

`getPoolSize`：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。

`getActiveCount`：获取活动的线程数。

#### **通过扩展线程池进行监控**。

可以通过继承线程池来自定义线程池，重写线程池的`beforeExecute`、`afterExecute`和`terminated`方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。
 例如，监控任务的`平均执行时间`、`最大执行时间` 和 `最小执行时间` 等。这几个方法在线程池里是空方法。

```java
protected void beforeExecute(Thread t, Runnable r) { }
```

### 2.7、ThreadPoolExecutor示例

 **1) 自主定制非阻塞线程池**

```java
package com.zach.concurrency.threadpool;
 
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
 
/**
 * @Author:Zach
 * @Description: 定制属于自己的非阻塞线程池
 * @Date:Created in 15:26 2018/8/14
 * @Modified By:
 */
public class CustomThreadPoolExecutor {
    private ThreadPoolExecutor pool = null;
 
    /**
     * 线程池初始化方法
     *
     * corePoolSize 核心线程池大小----10
     * maximumPoolSize 最大线程池大小----30
     * keepAliveTime 线程池中超过corePoolSize数目的空闲线程最大存活时间----30+单位TimeUnit
     * TimeUnit keepAliveTime时间单位----TimeUnit.MINUTES
     * workQueue 阻塞队列----new ArrayBlockingQueue<Runnable>(10)====10容量的阻塞队列
     * threadFactory 新建线程工厂----new CustomThreadFactory()====定制的线程工厂
     * rejectedExecutionHandler 当提交任务数超过maxmumPoolSize+workQueue之和时,
     * 							即当提交第41个任务时(前面线程都没有执行完,此测试方法中用sleep(100)),
     * 						          任务会交给RejectedExecutionHandler来处理
     */
 
    public void init() {
 
        pool = new ThreadPoolExecutor(10,30,30,
                TimeUnit.MINUTES,new ArrayBlockingQueue<Runnable>(10),new CustomThreadFactory(), new CustomRejectedExecutionHandler());
    }
 
    public void destory() {
        if(pool !=null) {
            pool.shutdownNow();
        }
    }
 
    public ExecutorService getCustomThreadPoolExecutor() {
        return this.pool;
    }
 
 
    private class CustomRejectedExecutionHandler implements  RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            //记录异常
            System.out.println("error...................");
        }
    }
 
    private class CustomThreadFactory implements ThreadFactory {
 
        private AtomicInteger count = new AtomicInteger(0);
 
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
           String threadName =  CustomThreadPoolExecutor.class.getSimpleName()+count.addAndGet(1);
 
            System.out.println(threadName);
            t.setName(threadName);
            return t;
        }
    }
 
    public static void main(String[] args){
        CustomThreadPoolExecutor exec = new CustomThreadPoolExecutor();
 
        //1. 初始化
        exec.init();
 
        ExecutorService pool = exec.getCustomThreadPoolExecutor();
 
        for(int i=1;i<100;i++) {
            System.out.println("提交第"+i+"个任务");
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        System.out.println(">>>task is running========");
                       Thread.sleep(3000);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            });
        }
 
        //2. 销毁----此处不能销毁,因为任务没有提交执行完,如果销毁线程池,任务也就无法执行
        //exec.destory();
 
        try {
            Thread.sleep(10000);
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    /**
     * 方法中建立一个核心线程数为30个，缓冲队列有10个的线程池。每个线程任务，执行时会先睡眠3秒，保证提交10任务时，线程数目被占用完，再提交30任务时，阻塞队列被占用完，，这样提交第41个任务是，会交给CustomRejectedExecutionHandler 异常处理类来处理。
     提交任务的代码如下：
     
     /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    /**
     public void execute(Runnable command) {
     if (command == null)
     throw new NullPointerException();
     int c = ctl.get();
     if (workerCountOf(c) < corePoolSize) {
     if (addWorker(command, true))
     return;
     c = ctl.get();
     }
     if (isRunning(c) && workQueue.offer(command)) {
     int recheck = ctl.get();
     if (! isRunning(recheck) && remove(command))
     reject(command);
     else if (workerCountOf(recheck) == 0)
     addWorker(null, false);
     }
     else if (!addWorker(command, false))
     reject(command);
     }
        注意：41以后提交的任务就不能正常处理了，因为，execute中提交到任务队列是用的offer方法，如上面代码，
        这个方法是非阻塞的，所以就会交给CustomRejectedExecutionHandler 来处理，
         所以对于大数据量的任务来说，这种线程池，如果不设置队列长度会OOM，设置队列长度，会有任务得不到处理，接下来我们构建一个阻塞的自定义线程池
     */
}
```

2) **自主定制阻塞线程池**

```java
package com.zach.concurrency.threadpool;
 
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
 
/**
 * @Author:Zach
 * @Description: 定制属于自己的阻塞线程池
 * @Date:Created in 15:26 2018/8/14
 * @Modified By:
 */
public class CustomUnblockThreadPoolExecutor {
    private ThreadPoolExecutor pool = null;
 
    /**
     * 线程池初始化方法
     *
     * corePoolSize 核心线程池大小----1
     * maximumPoolSize 最大线程池大小----3
     * keepAliveTime 线程池中超过corePoolSize数目的空闲线程最大存活时间----30+单位TimeUnit
     * TimeUnit keepAliveTime时间单位----TimeUnit.MINUTES
     * workQueue 阻塞队列----new ArrayBlockingQueue<Runnable>(5)==== 5容量的阻塞队列
     * threadFactory 新建线程工厂----new CustomThreadFactory()====定制的线程工厂
     * rejectedExecutionHandler 当提交任务数超过maxmumPoolSize+workQueue之和时,
     * 							即当提交第9个任务时(前面线程都没有执行完,此测试方法中用sleep(100)),
     * 						          任务会交给RejectedExecutionHandler来处理
     */
 
    public void init() {
 
        pool = new ThreadPoolExecutor(1,3,30,
                TimeUnit.MINUTES,new ArrayBlockingQueue<Runnable>(5),new CustomThreadFactory(), new CustomRejectedExecutionHandler());
    }
 
    public void destory() {
        if(pool !=null) {
            pool.shutdownNow();
        }
    }
 
    public ExecutorService getCustomThreadPoolExecutor() {
        return this.pool;
    }
 
 
    private class CustomRejectedExecutionHandler implements  RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            //核心改造点,由blockingqueue的offer改成put阻塞方法
            try {
                executor.getQueue().put(r);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
 
    private class CustomThreadFactory implements ThreadFactory {
 
        private AtomicInteger count = new AtomicInteger(0);
 
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            String threadName =  CustomUnblockThreadPoolExecutor.class.getSimpleName()+count.addAndGet(1);
 
            System.out.println(threadName);
            t.setName(threadName);
            return t;
        }
    }
 
    public static void main(String[] args){
        CustomUnblockThreadPoolExecutor exec = new CustomUnblockThreadPoolExecutor();
 
        //1. 初始化
        exec.init();
 
        ExecutorService pool = exec.getCustomThreadPoolExecutor();
 
        for(int i=1;i<100;i++) {
            System.out.println("提交第"+i+"个任务");
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        System.out.println(">>>task is running========");
                        TimeUnit.SECONDS.sleep(10);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            });
        }
 
        //2. 销毁----此处不能销毁,因为任务没有提交执行完,如果销毁线程池,任务也就无法执行
        //exec.destory();
 
        try {
            Thread.sleep(10000);
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    /**
     * 解释：当提交任务被拒绝时，进入拒绝机制，我们实现拒绝方法，把任务重新用阻塞提交方法put提交，实现阻塞提交任务功能，防止队列过大，OOM，提交被拒绝方法在下面
     *
     * public void execute(Runnable command) {
     if (command == null)
     throw new NullPointerException();
     int c = ctl.get();
     if (workerCountOf(c) < corePoolSize) {
     if (addWorker(command, true))
     return;
     c = ctl.get();
     }
     if (isRunning(c) && workQueue.offer(command)) {
     int recheck = ctl.get();
     if (! isRunning(recheck) && remove(command))
     reject(command);
     else if (workerCountOf(recheck) == 0)
     addWorker(null, false);
     }
     else if (!addWorker(command, false))
     // 进入拒绝机制， 我们把runnable任务拿出来，重新用阻塞操作put，来实现提交阻塞功能
     reject(command);
     }
     */
}
```

### 2.8、总结

`ThreadPoolExecutor`的核心方法是`execute`，控制着工作线程的创建和任务的执行，如下图：

![clipboard.png](.\img\3038835245-5d8a4478d389c_articlex.png)

同时，`ThreadPoolExecutor`中有几个比较重要的组件：阻塞队列、核心线程池、拒绝策略，它们的关系如下图，图中的序号表示`execute`的执行顺序，可以配合上面的流程图来理解：

![clipboard.png](.\img\2541586687-5d8a44869de72_articlex.png)

关于`ThreadPoolExecutor`这个线程池，最重要的是根据系统实际情况，合理进行线程池参数的设置以及阻塞队列的选择。现实情况下，一般会自己通过`ThreadPoolExecutor`的构造器去构建线程池，而非直接使用`Executors`工厂创建，因为这样更利于对参数的控制和调优。

另外，根据任务的特点，要有选择的配置核心线程池的大小：

- 如果任务是 **CPU 密集型**（需要进行大量计算、处理），则应该配置尽量少的线程，比如 CPU 个数 + 1，这样可以避免出现每个线程都需要使用很长时间但是有太多线程争抢资源的情况；
- 如果任务是 **IO密集型**（主要时间都在 I/O，CPU 空闲时间比较多），则应该配置多一些线程，比如 CPU 数的两倍，这样可以更高地压榨 CPU。

***1、用`ThreadPoolExecutor`自定义线程池，看线程是的用途，如果任务量不大，可以用无界队列，如果任务量非常大，要用有界队列，防止OOM*** 
***2、如果任务量很大，还要求每个任务都处理成功，要对提交的任务进行阻塞提交，重写拒绝机制，改为阻塞提交。保证不抛弃一个任务*** 
***3、最大线程数一般设为2N+1最好，N是CPU核数*** 
***4、核心线程数，看应用，如果是任务，一天跑一次，设置为0，合适，因为跑完就停掉了，如果是常用线程池，看任务量，是保留一个核心还是几个核心线程数*** 
***5、如果要获取任务执行结果，用`CompletionService`，但是注意，获取任务的结果的要重新开一个线程获取，如果在主线程获取，就要等任务都提交后才获取，就会阻塞大量任务结果，队列过大OOM，所以最好异步开个线程获取结果***




























