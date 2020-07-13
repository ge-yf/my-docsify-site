# fork 函数详解

## fork入门

一个进程，包括代码、数据和分配给进程的资源。fork()函数通过系统调用创建一个与原来进程几乎完全相同的进程，也就是两个进程可以做完全相同的事，但如果初始参数或者传入的变量不同，两个进程也可以做不同的事。

一个进程调用fork（）函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的所有值都复制到新的新进程中，只有少数值与原来的进程的值不同。相当于克隆了一个自己。

### An example(fork test 1)

```C
#include <unistd.h>
#include <stdio.h>

int main()
{
    pid_t fpid;
    int count = 0;
    fpid = fork();

    if (fpid < 0) {
        printf("fork error!\n");
    }
    else if (fpid == 0) {
        // in child process
        printf("I am the child process, my process id is [%d]\n", getpid());
        printf("I am the child process, my father's process id is [%d]\n", getppid());
        count++;
    }
    else {
        // in father process
        printf("I am the father process, my process id is [%d]\n", getpid());
        printf("I am the father process, my child's process id is [%d]\n", fpid);
        count++;

        waitpid(fpid, NULL, 0);
    }

    printf("count == [%d]\n", count);

    return 0;
}

```

运行结果：

> I am the father process, my process id is [18365]  
> I am the father process, my child's process id is [18366]  
> I am the child process, my process id is [18366]  
> I am the child process, my father's process id is [18365]  
> count == [1]  
> count == [1]  

※由于父进程存在waitpid处理，因此第一次输出的"count == [1]"是子进程的，等到子进程退出后，父进程的"count == [1]"才输出。
※如果父进程没有waitpid处理，那么父进程可能会先于子进程退出，子进程取到的父进程的pid就会为1(因为子进程被init（pid=1）进程接管,所以用getppid取出来的是1)

在语句fpid=fork()之前，只有一个进程在执行这段代码，但在这条语句之后，就变成两个进程在执行了，这两个进程的几乎完全相同，将要执行的下一条语句都是if(fpid<0)……

为什么两个进程的fpid不同呢，这与fork函数的特性有关。fork调用的一个奇妙之处就是它仅仅被调用一次，却能够返回两次，它可能有三种不同的返回值：

1. 在父进程中，fork返回新创建子进程的进程ID；
2. 在子进程中，fork返回0；
3. 如果出现错误，fork返回一个负值；

fork出错可能有两种原因：

1. 当前的进程数已经达到了系统规定的上限，这时errno的值被设置为EAGAIN。
2. 系统内存不足，这时errno的值被设置为ENOMEM。

创建新进程成功后，系统中出现两个基本完全相同的进程，这两个进程执行没有固定的先后顺序，哪个进程先执行要看系统的进程调度策略。

每个进程都有一个独特（互不相同）的进程标识符（process ID），可以通过getpid（）函数获得，还有一个记录父进程pid的变量，可以通过getppid（）函数获得变量的值。

执行完fork后，父进程的变量为count=0，fpid！=0(fpid为子进程的pid)。子进程的变量为count=0，fpid=0，这两个进程的变量都是独立的，存在不同的地址中，不是共用的，这点要注意。可以说，我们就是通过fpid来识别和操作父子进程的。

还有人可能疑惑为什么不是从#include处开始复制代码的，这是因为fork是把进程当前的情况拷贝一份，执行fork时，进程已经执行完了int count=0;fork只拷贝下一个要执行的代码到新的进程。

## fork进阶

### An example(fork test 2)

```C
#include <unistd.h>
#include <stdio.h>

int main()
{
    int i = 0;
    printf("i\tchild/parent\tppid\tpid\tfpid\n");

    for (i = 0; i < 2; ++i) {
        pid_t fpid = fork();
        if (fpid == 0) {
            printf("%d\tchild\t\t%d\t%d\t%d\n", i, getppid(), getpid(), fpid);
        }
        else {
            printf("%d\tparent\t\t%d\t%d\t%d\n", i, getppid(), getpid(), fpid);
            waitpid(fpid, NULL, 0);
        }
    }

    return 0;
}
```

运行结果：

> i &emsp; child/parent&emsp;ppid&emsp;&emsp;pid&emsp;fpid  
> 0 &emsp; parent&emsp;&emsp;&emsp;14459&emsp;16518&emsp;16519  
> 1 &emsp; child&emsp;&emsp;&emsp;&emsp;16518&emsp;16519&emsp;0  
> 0 &emsp; parent&emsp;&emsp;&emsp;16518&emsp;16519&emsp;16520  
> 1 &emsp; child&emsp;&emsp;&emsp;&emsp;16519&emsp;16520&emsp;0  
> 1 &emsp; parent&emsp;&emsp;&emsp;14459&emsp;16518&emsp;16521  
> 1 &emsp; child&emsp;&emsp;&emsp;&emsp;16518&emsp;16521&emsp;0

第一步：在父进程中，指令执行到for循环中，i=0，接着执行fork，fork执行完后，系统中出现两个进程，分别是p16518和p16519（后面我都用pxxxx表示进程id为xxxx的进程）。可以看到父进程p16518的父进程是p14459，子进程p16519的父进程正好是p16518。我们用一个链表来表示这个关系：

> p14459->p16518->p16519

第一次fork后，p16518（父进程）的变量为i=0，fpid=16519（fork函数在父进程中返向子进程id）。

p16519（子进程）的变量为i=0，fpid=0（fork函数在子进程中返回0）。

所以打印出结果：
> 0 &emsp; parent&emsp;&emsp;&emsp;14459&emsp;16518&emsp;16519  
> 1 &emsp; child&emsp;&emsp;&emsp;&emsp;16518&emsp;16519&emsp;0  

第二步：假设子进程p16519先执行，当进入下一个循环时，i=1，接着执行fork，系统中又新增一个进程p16520，对于此时的父进程，p16518->p16519（当前进程）->p16520（被创建的子进程）。

之后子进程p16520开始执行，在进入下一个循环时，i=2， 不满足for循环条件，因此子进程p16520执行完毕。

此时打印出结果：
> 0 &emsp; parent&emsp;&emsp;&emsp;16518&emsp;16519&emsp;16520  
> 1 &emsp; child&emsp;&emsp;&emsp;&emsp;16519&emsp;16520&emsp;0  

第三步：父进程p16518继续执行，进入下个循环时，i=1,，接着执行fork，系统中又新增一个进程16521。

之后子进程16521开始执行，在进入下一个循环时，i=2， 不满足for循环条件，因此子进程16521执行完毕。

此时打印出结果：
> 1 &emsp; parent&emsp;&emsp;&emsp;14459&emsp;16518&emsp;16521  
> 1 &emsp; child&emsp;&emsp;&emsp;&emsp;16518&emsp;16521&emsp;0

总结一下，这个程序执行的流程如下：

![fork1](.\resource\fork\fork1.jpg)

这个程序最终产生了3个子进程，执行过6次printf（）函数。

### An example(fork test 3)

```C
#include <unistd.h>
#include <stdio.h>

int main()
{
    int i = 0;

    for (i = 0; i < 3; ++i) {
        pid_t fpid = fork();

        if (0 == fpid) {
            printf("son\n");
        }
        else {
            printf("father\n");
        }
    }

    return 0;
}
```

运行结果：
> father  
> son  
> father  
> father  
> father  
> father  
> son  
> father  
> son  
> son  
> son  
> father  
> son  
> son  

执行逻辑如下：

![fork2](.\resource\fork\fork2.jpg)

总结一下规律，对于这种N次循环的情况，执行printf函数的次数为2*（1+2+4+……+2N-1）次，创建的子进程数为1+2+4+……+2N-1个。
