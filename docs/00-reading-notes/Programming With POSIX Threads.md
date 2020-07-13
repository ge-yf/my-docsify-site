# Programming With POSIX Threads

    作者: David R. Butenhof
    出版社: Addison-Wesley Professional
    出版年: 1997-5-26

## Introduction

Process: an instance of a computer program that is being executed.

进程是一个实体。每一个进程都有它自己的地址空间，一般情况下，包括文本区域（text region）、数据区域（data region）和堆栈（stack region）。文本区域存储处理器执行的代码；数据区域存储变量和进程执行期间使用的动态分配的内存；堆栈区域存储着活动过程调用的指令和本地变量。

Thread: 一个标准的线程由线程ID，当前指令指针(PC），寄存器集合和堆栈组成。另外，线程是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不拥有系统资源，只拥有一点儿在运行中必不可少的资源，但它可与同属一个进程的其它线程共享进程所拥有的全部资源。

一个线程可以创建和撤消另一个线程，同一进程中的多个线程之间可以并发执行。由于线程之间的相互制约，致使线程在运行中呈现出间断性。线程也有就绪、阻塞和运行三种基本状态。

进程与线程的区别：

1. 进程是资源分配的最小单位，线程是程序执行的最小单位（资源调度的最小单位）
2. 进程有自己的独立地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段，这种操作非常昂贵。而线程是共享进程中的数据的，使用相同的地址空间，因此CPU切换一个线程的花费远比进程要小很多，同时创建一个线程的开销也比进程要小很多。
3. 线程之间的通信更方便，同一进程下的线程共享全局变量、静态变量等数据，而进程之间的通信需要以通信的方式（IPC)进行。不过如何处理好同步与互斥是编写多线程程序的难点。
4. 但是多进程程序更健壮，多线程程序只要有一个线程死掉，整个进程也死掉了，而一个进程死掉并不会对另外一个进程造成影响，因为进程有自己独立的地址空间。

### 异步（Asynchronous）

> Asynchronous means that things happen independently (concurrently) unless there's some enforced dependency.

异步表明事情相互独立地(并发地)发生，除非有强加的依赖性。

### 并发（Concurrency）

Concurrency describes the behavior of threads or processes on a uniprocessor system.

The definition of concurrent execution in POSIX requires that "functions that suspend the execution of the calling thread shall not cause the execution of other threads to be indefinitely suspended."

并发(concurrency)：指在**单处理器**的系统中，同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。

### 并行（Parallelism）

Parallelism describes concurrent sequences tha proceed simultaneously.

In outher words, software "parallelism" is the same as English "concurrency" and different from software "concurrency".

Parallelism has a vaguely redeeming analogy to the English definition: It refers to things proceeding in the same direction independently (without intersection).

Ture parallelism can occur only on a multiprocessor system, but concurrency can occur on both uniprocessor and multiprocessor systems.

### 线程安全和可重入（Thread safety and reentrancy）

"Thread-safe"是指代码能够被多个线程调用而不会产生灾难性的后果。它不要求代码能够高效地在多线程环境中运行，只是要求能够安全地运行。

大多数现有的函数可以利用Pthread提供的工具， mutexes， condition variables和线程私有数据来实现thread-safe。

不需要保存永久状态的函数，可以通过整个函数调用的串行化来实现Thread-safe。

for example, by locking a mutex on entry to the function, and unlocking the mutex before returning. Functions made thread-safe by serializing the entire function can called in multiple threads - but only one thread can truly preform the function at a time.

The term "reentrant" is sometimes used to mean "efficiently thread-safe". That is, the code was made thread-safe by some more sophistcated measures than converting the function or library into a single serial region.

如果有一个函数不幸被设计成为这样：那么不同任务调用这个函数时可能修改其他任务调用这个函数的数据，从而导致不可预料的后果。这样的函数是不安全的函数，也叫不可重入函数。

可重入是指一个可以被多个任务调用的过程，任务在调用时不必担心数据是否会出错。

一个可重入的函数简单来说就是可以被中断的函数，也就是说，可以在这个函数执行的任何时刻中断它，转入OS调度下去执行另外一段代码，而返回控制时不会出现什么错误；而不可重入的函数由于使用了一些系统资源，比如全局变量区，中断向量表等，所以它如果被中断的话，可能会出现问题，这类函数是不能运行在多任务环境下的。

编写可重入函数时，若使用全局变量，则应通过关中断、信号量（即P、V操作）等手段对其加以保护。

保证函数的可重入性的方法：

1. 在写函数时候尽量使用局部变量（例如寄存器、堆栈中的变量）；
2. 对于要使用的全局变量要加以保护（如采取关中断、信号量等互斥方法），这样构成的函数就一定是一个可重入的函数。

### 并发控制功能（Concurrency control functions）

Here are three essential facilities, or aspects of any concurrent system:

1. **Execution context** is the state of a concurrent entity. A concurrent system must provide a way to create and delete execution context, and maintain their state independently. It must be able to save the state of one context and dispatch to another at various times, for example, when one needs to wait for an external event. It  must be able to continue a context from the point where is last executed, with the same register contents, at a later time.
2. **Scheduling** determines which context (or set of contexts) should execute at any given point in time, and switches between contexts when necessary.
3. **Synchronization** provides mechanisms for concurrent execution contexts to coordinate their use of shared resources. We use this term in a way that is nearly the opposite of the standard dictionary meaning.You'll find a definition much like "cause to occur at the same time," whereas we usually mean something that might better be expressed as "prevent form occurring at the same time." In a thesaurus, you may find that "cooperate" is a synonym for "synchronize" --- and synchronization is the mechanism by which threads cooperate to accomplish a task. This book will use the term "synchronization." though, because that is what you'll see used, almost universally.

