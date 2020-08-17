# 操作系统安全基础

## 一. 计算机系统组成

* **硬件**

  –提供基本计算资源 (CPU, memory, I/O devices).

* **操作系统** 

  –控制、协调不同的应用程序使用计算机硬件.

* **应用程序**

  –定义人们使用系统资源的方式.

* **用户** 

  –如 人, 机器, 其他计算机 

> 层次组成示意图

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200101223959445.png" alt="image-20200101223959445" style="zoom: 67%;" />

## 二. 操作系统的安全性方法

#### 1.操作系统的安全目标

* **目标 1: 允许多用户`安全的`共享单机**

  –进程、内存、文件设备等的分离与共享

  * 如何实现?

    –内存保护

    –处理器模式

    –鉴别

    –文件的访问控制

* **目标 2: 确保网络环境下的安全操作**

  * 如何实现?

    –鉴别

    –访问控制

    –安全通信 (利用加密手段)

    –记录日志 & 审计

    –入侵检测

    –恢复

#### 2.保护方法

##### 1)内存保护模式

* 保证一个用户的进程不能访问其他人的内存空间

  –栅栏

  * 分段
  * 分页

  –基/堆寄存器

  –重分配

  –…

* 操作系统进程、用户进程：具有不同的权限

##### 2)cpu运行模式（处理器模式，特权模式）

* 系统模式(特权模式,主模式,超模式,内核模式)

  –可以执行任意指令、访问任意内存地址、 硬件设备、中断操作、改变处理器特权状态、访问内存管理单元、修改寄存器. 

* 用户模式

  –受限的内存访问, 有些指令不能执行

  –不能：停止中断, 改变任意进程状态, 访问内存管理单元等

* 从用户模式转到系统模式的切换必须通过系统调用 (system calls)

- [x] **内核空间VS用户空间**

  * 操作系统的一部分运行在内核模式

    –故称为 **OS kernel**

  * 操作系统的其他部分运行在用户模式，包括服务程序 (daemon programs), 用户的应用等

    –作为进程运行

    –形成用户空间

  * root (or superuser, administrator)运行的进程可能在内核模式或用户模式
  

> 内核空间和用户空间的层次性视图

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200101225727925.png" alt="image-20200101225727925" style="zoom: 80%;" />

- [x] **内核的实现方法：**

  * **单核**

    一个大内核提供所有的服务，包括文件系统、网络服务、设备驱动等

    –所有内核代码运行在单地址空间，互相之间会产生影响

    –如, Linux 2.6 内核有 6 百万条代码

    –优点: 高效

    –缺点: 复杂, 某部分的bug会影响整个系统

    例如:

    –UNIX-variants:FreeBSD，SunOS，AIX，NetBSD

    提供可加载内核模块的内核仍然是单核,如MULTICS

  * **微内核**

    内核较小，仅提供执行系统服务必须的机制

    –内核提供：低地址空间的管理，线程管理，进程间通信 (IPC)

    –操作系统的服务可在用户模式下工作：包括设备驱动、协议栈、文件系统、用户的接口代码

    –优点: 可实现最小特权, 容忍设备驱动的失败/错误等

    –缺点: 性能差, 系统的关键服务出错后会使得系统停机

#### 3.系统调用

##### 1)含义

* **从用户模式进入内核模式的系统程序**

  ```
  * -所有的操作系统都提供多种服务的入口点，由调用程序向内核请求服务。系统调用是不能更改的
    -系统调用把应用程序的请求传给内核，调用相应的内核函数完成所需的处理，将处理结果返回给应用程序，如果没有系统调用和内核函数，用户编写大型应用程序非常困难
    -UNIX为每个系统调用在标准C库中设置一个同样名字的函数。用户进程用标准C调用序列来调用这些函数，启动函数调用对应的内核服务
    -从执行者角度，系统调用和库函数有重大区别
    -从用户角度，区别不重要
  ```

> C函数库，系统调用关系

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200101230817190.png" alt="image-20200101230817190" style="zoom:80%;" />

* 进程控制系统调用(fork,exec,wait)通常由用户的应用程序直接调用。UNIX系统也提供了库函数system和open

> 存储器分配函数malloc,sbrk系统调用关系

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200101231038868.png" alt="image-20200101231038868" style="zoom:80%;" />

* 内核中的系统调用分配另外一块空间给进程，而库函数malloc管理这一空间

##### 2)分类

进程控制类：

| 名称        | 作用                 |
| :---------- | -------------------- |
| fork        | 创建一个新进程       |
| clone       | 按指定条件创建子进程 |
| execve      | 运行可执行文件       |
| exit        | 中止进程             |
| getpid      | 获取进程标识号       |
| getppid     | 获取父进程标识号     |
| getpriority | 获取调度优先级       |
| setpriority | 设置调度优先级       |
| pause       | 挂起进程，等待信号   |
| prctl       | 对进程进行特定操作   |
| ptrace      | 进程跟踪             |
| wait        | 等待子进程终止       |

文件管理类：

| 名称     | 作用                   |
| -------- | ---------------------- |
| fcntl    | 文件控制               |
| open     | 打开文件               |
| creat    | 创建新文件             |
| close    | 关闭文件描述字         |
| read     | 读文件                 |
| write    | 写文件                 |
| umask    | 设置文件权限掩码       |
| chdir    | 改变当前工作目录       |
| chmod    | 改变文件方式           |
| fchmod   | 参见chmod              |
| chown    | 改变文件的属主或用户组 |
| truncate | 截断文件               |

用户管理类：

| 名称      | 作用                                  |
| --------- | ------------------------------------- |
| getuid    | 获取用户标识号                        |
| setuid    | 设置用户标志号                        |
| getgid    | 获取组标识号                          |
| setgid    | 设置组标志号                          |
| getegid   | 获取有效组标识号                      |
| setegid   | 设置有效组标识号                      |
| geteuid   | 获取有效用户标识号                    |
| seteuid   | 设置有效用户标识号                    |
| setreuid  | 分别设置真实和有效的用户标识号        |
| getresgid | 获取真实的,有效的和保存过的组标识号   |
| setresgid | 设置真实的,有效的和保存过的组标识号   |
| getresuid | 获取真实的,有效的和保存过的用户标识号 |
| setresuid | 设置真实的,有效的和保存过的用户标识号 |

信息维护类：

- get time or date, set time or date 
- get system data, set system data 
- get process, file, or device attributes 
- set process, file, or device attributes 

通信类：

- create, delete communication connection
- send, receive messages
- transfer status information 
- attach or detach remote devices 

## 三. 用户空间的安全机制

#### 1. 鉴别 Authentication

* 用户访问计算机系统或系统资源时，需要鉴别用户的身份，以确保其是合法用户

#### 2. 访问控制 Access Control

* 参考监控器 Reference Monitor 

  –检测所有与安全相关的操作

  * 创建进程, 访问文件, …

* 主体 Subjects

* 客体 Objects

* 访问 Access

#### 3. 记录日志和审计 Logging & Auditing

* 将系统信息记录到日志中

* 记录什么到日志?

  –事件和数据相关信息 、正常的和可疑的, 比如:

  * 尝试登录

  * 修改时间

  * 网络连接事件

 * 如何实现

    –应用日志, API hooking, 检测系统调用, 捕包, …

#### 4. 入侵检测  Intrusion Detection 

* 检测和记录网络相关事件、计算机系统遭到入侵和攻击事件

* 主要方法

  被动式 IDS vs. 响应式 IDS (also known as IPS)

  基于主机 IDS vs. 基于网络 IDS

#### 5. 恢复  Recovery