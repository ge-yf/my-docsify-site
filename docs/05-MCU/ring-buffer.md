# 循环缓冲区（RingBuffer）

## 简介

### 概念

关于循环缓冲区（Ring Buffer）的概念，其实来自于Linux内核（Maybe），是为解决某些特殊情况下的竞争问题提供了一种免锁的方法。这种特殊的情况就是当生产者和消费者都只有一个，而在其它情况下使用它也是必须要加锁的。对应在Linux内核中有对它的定义：

```c
struct kfifo {  
         unsigned char *buffer;  
         unsigned int size;  
         unsigned int in;  
         unsigned int out;  
         spinlock_t *lock;  
};
```

其中buffer指向存放数据的缓冲区，size是缓冲区的大小，in是写指针下标，out是读指针下标，lock是加到struct kfifo上的自旋锁（上面说不加锁不是这里的锁），防止多个进程并发访问此数据结构。当in==out时，说明缓冲区为空；当(in-out)==size时，说明缓冲区已满。


