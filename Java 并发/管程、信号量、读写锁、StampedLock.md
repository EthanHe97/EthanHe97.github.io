### 管程

- synchronized，内含一个条件变量（wait()、notify()、notifyAll()）
- Lock & Condition，可用多个条件变量（ReentrantLock、Condition、await、signal、signalAll）



### 信号量（Semaphore）

- 一个计数器（对外是透明的）
- 一个等待队列（对外是透明的）
- 三个方法（init、down、up）

在 Java SDK 并发包里，down 和 up 对应的则是 acquire 和 release。



### 读写锁（ReadWriteLock）

- 允许多个线程同时读共享变量
- 只允许一个线程写共享变量
- 如果一个写线程正在执行写操作，此时禁止读线程读共享变量

读写锁实现缓存：

```java
class Cache<K, V> {
    final Map<K, V> m = new HashMap<>();
    final ReadWriteLock rwl = new ReentrantReadWriteLock();
    // 读锁
    final Lock r = rwl.readLock();
    // 写锁
    final Lock w = rwl.writeLock();
    // 读缓存
    V get(K key) {
        r.lock();
        try {
            return m.get(key);
        } finally {
            r.unlock();
        }
    }
    // 写缓存
    V put(K key, V v) {
        w.lock();
        try {
            return m.put(key, v);
        } finally {
            w.unlock();
        }
    }
}
```

懒加载

> 也称为按需加载，指的是只有当应用查询缓存，并且数据不在缓存里的时候，才触发加载源头相关数据进行缓存的操作。

读写锁的升级与降级

> 在 ReadWriteLock 中，锁的升级（先获取读锁，再升级为写锁）不支持，而锁的降级（先获取写锁，再降级为读锁）是支持的。

### StampedLock

- 写锁、悲观读锁、乐观读

- 写锁和悲观读锁与 ReadWriteLock 中的写锁和读锁类似，但不同的是：StampedLock 里的写锁和悲观读锁加锁成功后，都会返回一个 stamp，然后解锁的时候，需要传入这个 stamp。

  ```java
  final StampedLock s1 = new StampedLock();
  // 悲观读锁
  long stamp = s1.readLock();
  try {
      // 省略业务相关代码
  } finally {
      s1.unlockRead(stamp);
  }
  
  // 写锁
  long stamp = s1.writeLock();
  try {
      // 省略业务相关代码
  } finally {
      s1.unlockWrite(stamp);
  }
  ```

- ReadWriteLock 支持多个线程同时读，但是当多个线程同时读的时候，所有的写操作都会被阻塞；而 StampedLock 提供的乐观读，是允许一个线程获取写锁的，也就是说不是所有的写操作都会被阻塞。

- 乐观读 是无锁的

  ```java
  class Point {
      private int x, y;
      final StampedLock s1 = new StampedLock();
      // 计算到原点的距离
      int distanceFromOrigin() {
          // 乐观读
          long stamp = s1.tryOptimisticRead();
          // 读入局部变量，
          // 读的过程可能被修改
          int curX = x, curY = y;
          // 判断执行读操作期间，
          // 是否存在写操作，如果存在，
          // 则 s1.validate 返回 false
          if(!s1.validate(stamp)) {
              // 升级为悲观读锁
              stamp = s1.readLock();
              try {
                  curX = x;
                  curY = y;
              } finally {
                  // 释放悲观读锁
                  s1.unlockRead(stamp);
              }
          }
          return Math.sqrt(curX * curX + curY * curY);
      }
  }
  ```

  