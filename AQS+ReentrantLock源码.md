# AQS源码

## 参考文章

https://www.cnblogs.com/funyoung/p/13619854.html

https://www.cnblogs.com/funyoung/p/13623109.html

## 概述

AQS中维护了一个被volatile修饰的int类型的同步状态state，以及CLH等待队列。

state同步状态用于维护同步资源被使用的情况，AQS本身并不关心state的值及其含义，完全由AQS的子类去定义以及维护。

CLH等待队列是由一个双向链表来实现的，存在head和tail指针分别指向链表中的头节点以及尾节点，同时链表中的节点由AQS中的Node静态内部类来表示。

## 获取 释放资源

AQS支持两种模式，一种是独占模式，一种是共享模式。

独占模式表示，同步资源在同一时刻只能被一个线程所持有，对应AQS的acquire()以及release()方法。

共享模式表示，同步资源在同一时刻可以被多个线程所持有，对应AQS的acquireShared()以及releaseShared()方法。

```java
acquire()方法：独占模式下获取同步资源。

release()方法：独占模式下释放同步资源。

acquireShared()方法：共享模式下获取同步资源。

releaseShared()方法：共享模式下释放同步资源。
```

AQS使用了模板方法设计模式，在acquire()、release()、acquireShared()、releaseShared()方法中都会调用其对应的try方法，比如acquire()方法中会调用tryAcquire()方法，release()方法中会调用tryRelease()方法，AQS子类只需要重写AQS提供的tryAcquire()、tryRelease()或tryAcquireShared()、tryReleaseShared()方法即可，只需要告诉AQS尝试获取同步资源或尝试释放同步资源是否成功，同时需要保证方法的实现是线程安全的。

tryAcquire()、tryRelease()、tryAcquireShared()、tryReleaseShared()方法都没有使用abstract进行修饰，同时方法中都会直接抛出UnsupportedOperationException异常，好处是不需要强制子类同时实现独占模式和共享模式中的方法，因为大多数AQS的子类都仅支持一种模式，用户只需要根据实际情况进行选择即可。

```java
tryAcquire(int arg)方法：独占模式下尝试获取同步资源，同时AQS规定，如果获取同步资源成功则返回true，否则返回false。

tryRelease(int arg)方法：独占模式下尝试释放同步资源，同时AQS规定，如果释放同步资源成功则返回true，否则返回false。

tryAcquireShared(int arg)方法：共享模式下尝试获取同步资源，同时AQS规定，如果获取同步资源成功则返回剩余的可用资源个数，否则返回负数。

tryReleaseShared(int arg)方法：共享模式下尝试释放同步资源，同时AQS规定，如果释放同步资源成功则返回true，否则返回false。
```

## Node静态内部类

![image-20201205162759160](AQS+ReentrantLock%E6%BA%90%E7%A0%81.assets/image-20201205162759160.png)

### 核心属性

```java
// 节点封装的线程
volatile Thread thread;
// 指向前驱节点的指针
volatile Node prev;
// 指向后继节点的指针
volatile Node next;
// 节点的等待状态（默认为0）（默认为0）（默认为0） 
volatile int waitStatus;
// 下一个正在等待的节点
Node nextWaiter;
// 共享模式下的标识节点
static final Node SHARED = new Node();
// 独占模式下的标识节点
static final Node EXCLUSIVE = null;
```

### 状态量

```java
// CANCELLED状态，表示线程已超时等等，处于CANCELLED状态的节点会从等待队列中剔除，不会参与到同步资源的竞争当中
static final int CANCELLED =  1;
// SIGNAL状态，如果节点的等待状态为SIGNAL，那么当它释放同步资源时，将会唤醒离它最近的同时等待状态不为CANCELLED的后继节点（同时也能说明节点存在后继节点）
static final int SIGNAL    = -1;
// 表示线程在指定的条件下进行等待
static final int CONDITION = -2;
// PROPAGATE状态，表示实际存在可用资源，需要再往下传播（唤醒）
static final int PROPAGATE = -3;
```

## 关键过程

### 获取资源

```java
public final void acquire(int arg) {
  if (!tryAcquire(arg) &&
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

```java
//具体的同步类有自己的实现
protected boolean tryAcquire(int arg) {
  throw new UnsupportedOperationException();
}
```

```java
private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode);
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail;
  if (pred != null) {
    node.prev = pred;
    //CAS加入队尾,防止中途插入新的节点丢失
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  //死循环加入队尾
  enq(node);
  return node;
}
```

```java
final boolean acquireQueued(final Node node, int arg) {
    // 失败标识
    boolean failed = true; 
    try {
         // 中断标识
        boolean interrupted = false;
        // 自旋
        for (;;) { 
            // 获取节点的前驱节点
            final Node p = node.predecessor(); 
            // 如果节点的前驱节点是头节点那么尝试获取同步资源
            // 强制要求队列中的节点获取同步资源的顺序必须是从队头到队尾，否则将会造成节点丢失，丢失
          	// 了的节点中的线程将会永远处于阻塞状态，同时只有当线程获取了同步资源后，它
          	// 才能成为头节点（队列初始化后的头节点除外），因此头节点肯定是已经获取过同步资
            // 源的（队列初始化后的头节点除外），因此为了遵循队列中的节点获取同步资源的
          	// 顺序必须是从队头到队尾，所以永远只有头节点的后继节点拥有尝
          	// 试获取同步资源的权利，因此当在尝试获取同步资源之前，需要先判断一下
          	//当前节点的前驱节点是否是头节点，如果不是就不用获取了
            if (p == head && tryAcquire(arg)) { 
                // 当获取同步资源成功，则将当前节点设置为头节点
                setHead(node); 
                // 将之前头节点的后继指针设置为null，帮助GC
                p.next = null; 
                failed = false; 
                // 返回中断标识
                return interrupted; 
            }
            // 如果节点的前驱节点不是头节点，或者尝试获取同步资源失败，那么将会调用shouldParkAfterFailedAcquire()方法，判断线程能否进行阻塞，当线程能够被阻塞时，将会调用parkAndCheckInterrupt()方法阻塞线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 如果在执行该方法的过程中，抛出了异常（线程超时等等），则failed标识为true，那么将会执行cancelAcquire()方法，将当前节点的等待状态设置为CANCELLED，同时从等待队列中剔除。
        if (failed)
            cancelAcquire(node);
    }
}
```

- 尝试获取资源，成功则直接继续工作

- 不成功则加入双向队列中，最终都会返回入队后的Node

- 调用acquiredQueued自旋获取资源，同时该方法的方法出口只有一个，也就是当节点的前驱节点是头节点，同时尝试获取同步资源成功，那么就会将当前节点设置为头节点。

	否则就会调用shouldParkAfterFailedAcquire()方法，判断线程能否进行阻塞，当线程能够被阻塞时，将会调用parkAndCheckInterrupt()方法阻塞线程，等待被唤醒。

	同时在执行acquireQueued()方法的过程中，如果抛出了异常，则failed标识为true，那么将会执行cancelAcquire()方法，将当前节点的等待状态设置为CANCELLED，同时从等待队列中剔除。

### 释放资源

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
      Node h = head;
      // 如果队列不等于空，同时头节点的等待状态不为0，也就是头节点存在后继节点，那么调用unparkSuccessor()方法，唤醒离头节点最近的同时等待状态不为CANCELLED的后继节点。
      if (h != null && h.waitStatus != 0)
        unparkSuccessor(h);
      return true;
    }
    return false;
}
```

```java
private void unparkSuccessor(Node node) {
    // 获取节点的等待状态
    int ws = node.waitStatus; 
    // 如果节点的等待状态不为CANCELLED，则通过CAS将节点的等待状态设置为0（恢复成队列初始化后的状态）
    if (ws < 0) 
        compareAndSetWaitStatus(node, ws, 0);
    // 获取节点的后继节点
    Node s = node.next;
    // 如果节点的后继指针为NULL（不能说明节点就没有后继节点）或者后继节点为CANCELLED状态，那么就从后往前寻找离当前节点最近的同时等待状态不为CANCELLED的后继节点
    if (s == null || s.waitStatus > 0) { 
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 唤醒该后继节点中的线程
        LockSupport.unpark(s.thread);
}
```

# ReentrantLock源码

## 概述

独享锁，基于AQS实现，底层自定义一个AQS的子类

ReentrantLock中state为0表示锁没有被线程所持有，使用state不为0表示锁经被线程所持有

公平和非公平两种模式

## 结构

![image-20201205173540861](AQS+ReentrantLock%E6%BA%90%E7%A0%81.assets/image-20201205173540861.png)

可以看到ReentrantLock中定义了一个抽象同步器（Sync）、非公平同步器（NonfairSync）、公平同步器（FairSync），同时非公平同步器和公平同步器都继承抽象同步器。

同时ReentrantLock中存在一个全局的抽象同步器属性，然后通过构建方法来进行初始化，通过fair参数来指定到底是使用公平同步器还是非公平同步器，默认情况下是使用非公平同步器。

同时ReentrantLock中的lock()方法将会调用抽象同步器声明的lock()方法，unlock()方法将会直接调用AQS的release()方法，而tryLock()方法将会调用抽象同步器提供的nonfairTryAcquire()方法。

### 基本同步器



### 公平同步器



### 非公平同步器

