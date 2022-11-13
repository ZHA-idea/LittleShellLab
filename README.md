# Shell

代码：[Github](https://github.com/ZHA-idea/CSAPP_ShellLab)

> 这一个lab要求用C语言实现一个完整的shell，感觉主要知识是shell对job的管理和进程间的信号通信。

## 1. 实验介绍

在本次实验中，你将编写一个简单的支持任务控制的Uinx shell程序，本次实验的目的是加深大家对进程控制和信号的理解。

实验代码的整体框架已经搭建好了，我们的任务是完成其中的某些函数。



## 2. 背景知识

控制流(control flow)：CPU执行的指令序列。

异常控制流(ECF, Exceptional Control Flow)：**系统为了应对系统状态的变化**，使控制流发生突变，这些突变称为异常控制流。

异常控制流存在于计算机系统的各个层次：

- 底层：

- - 异常(Exceptions)，更改控制流以应对某个系统事件，由硬件和操作系统共同实现。

- ![](http://pic.zha-node.com/uploads/big/b4cbb663d9d5a0e83f386a542c63f6f8.png)

- * 异常的分类

- ![](http://pic.zha-node.com/uploads/big/d0db8d22eb5acd2c2dd6e65631a8d917.png)

- 顶层：

- - 进程切换：由操作系统和硬件时钟共同实现
  - 信号：由操作系统实现
  - 非本地跳转：由C库实现

### 2.1 C库函数

* 获取进程pid

```C
#include <sys/types.h>
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);//获取父进程pid
```

* 创建和终止进程

```c
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
void exit(int status);
```

* 回收子进程（P516

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *statusp, int options)
```

### 2.2 Linux信号

它通知进程系统中发生了一个某种类型的事件，在终端，可以通过`kill -l`查看所有信号

- 类似于异常和中断
- 由操作系统内核发送给进程

##### 2.3.1. 信号的发送 （P529

内核通过**更新目标进程的某些状态**来发送一个信号给目标进程。

内核发送信号可能由于以下几种原因：

- 内核检测到了某个系统事件的发生
- 另外某个进程调用kill系统调用，要求内核给目标进程发送一个信号

##### 2.3.2. 信号的接收 （P531

信号接受的时机：

当内核把进程p**从内核模式切换到用户模式时**，它会检查进程p的未被阻塞的待处理信号的集合(pending & ~blocked)，如果集合为空，内核将控制传递到p的逻辑流中的下一条指令。然而，如果集合非空，那么内核将选择集合中的某个信号k并且强制p接收信号k。

![img](http://pic.zha-node.com/uploads/big/7ab9c92e8b5f031da61cc82f66325116.png)



每个信号都有一个预定义的默认行为，进程可以使用signal函数可以修改和信号关联的默认行为。

```c
#include <signal.h>
typedef void (*sighandler_t)(int);

sighandler_t signal(int signum, sighandler_t handler)
```

##### 2.3.3. 信号的阻塞 （P533

隐式阻塞：内核默认阻塞当前正在处理的信号

显式阻塞：进程可以使用**sigprocmask函数**明确地阻塞和解除阻塞选定的信号

某个信号s被阻塞后，进程可以接收别的进程发送来的s信号，但是暂时不对s信号进行相应。

### 2.3 Shell行为

Shell 本身是一个进程，用于代理用户与操作系统内核。

Shell 建立并维护一个 Jobs 列表，Jobs 列表中维护“作业”（Job），每个作业与一个进程对应，并用 jid 唯一标识。与进程不同的是，Jobs 由 shell 维护，它定义了作业能够存在的几种状态：Running、Stopped、Terminated。

当shell执行外部命令时，shell调用fork函数并在子进程中执行新进程，因此，所有在shell中调用的命令都是shell的子进程

## 3. 代码

代码：[Github](https://github.com/ZHA-idea/CSAPP_ShellLab)
