# Unix访问控制

## 一. 文件的组织与权限表示

#### 1.文件的组织形式

* 文件存在于目录中

* 目录也一种特殊的文件

* 文件采用层次结构存放

* 文件的基本操作

  –open、read、write、lseek、close

  –lseek：打开文件后，相对当前文件的位移量

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200102214645516.png" alt="image-20200102214645516" style="zoom: 67%;" />

* 打开现存文件或创建新文件，内核向进程返回一个文件描述符
* 对内核而言，所有文件都由文件描述符引用

> UNIX inodes：每个文件都有一个对应的Inode

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200102215935461.png" alt="image-20200102215935461" style="zoom:67%;" />

#### 2.文件的普通权限表示

##### 1)基本权限位

* Read 权限：控制读文件内容

  – read system call

* Write权限：控制读文件内容

  – write system call

* Execute权限：控制将文件调入内存并执行

  – execve system call

##### 2)UNIX文件权限表示

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200102220559439.png" alt="image-20200102220559439" style="zoom: 67%;" />

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200102220712308.png" alt="image-20200102220712308" style="zoom: 67%;" />

* 权限位数字用八进制表示

  eg.chmod函数修改权限位：

  #chmod 777 [filename] (对应rwxrwxrwx,二进制为111111111)

  #chmod 444 [filename] (对应r--r--r--,二进制为100100100)

* fchmod函数：对已打开文件更改权限


##### 3)操作文件时权限的检查次序

* 权限检查时依次检查拥有者、组、其它:

  –当用户是文件的拥有者, 运行时检查owner的r/w/x位

  –再检查组权限, 查看组 r/w/x 权限位

  –最后, 检查其它的 r/w/x 权限位

* 可以做负授权么？

  –如， A组中仅B用户不能访问cc文件?（不能吧）

#### 3.文件的执行权限

##### 1)文件执行权

* 二进制文件 vs. 脚本文件（script file)

  –有执行权限但无读权限，能运行文件么?

  能运行二进制文件，不能运行脚本文件

  –有执行权限但无读权限，能运行脚本文件么?

  不能

  –有读权限但无执行权限，能运行脚本文件么? 

  不能

  **Conclude:**二进制文件需要执行权限，脚本文件需要读权限和执行权限

##### 2)权限位的特殊性

* 权限位不是直接授权用户操作某程序，而是授权给用户可以使用相应的**系统调用** 
* 命令如 cat 和 more 调用了read() 系统调用, 但仅当对要访问的文件和目录具有 r 权限时才允许用户调用read() 系统调用 
* 因此当用户不能使用 more、 vi等操作时，是因为对相应的文件没有 r 权限
* 同样, 用户必须有 w 权限才能调用 write()系统调用写文件或目录, 在满足权限前提下，通过系统调用修改文件或目录

##### 3)文件权限检查过程

* 改文件名、删除文件不需要对文件有 write() 系统调用 

  –不是文件的owner 也可删除文件

  –需对**目录**有**写权限**

* mv 命令将文件从一个目录转移到另一个目录, (如, mv foo /floppy) 系统需将文件拷贝到另一块磁盘空间，对被转移文件不需要读权限

#### 4.文件的特殊权限

##### 1)文件的suid位

* 每个文件有下列信息:

  12 个权限位

  –read/write/execute for user, group, and others, 

  –suid, sgid, sticky

* suid位：可以在文件的执行权限上设置一个特殊标志

  –当执行此文件时，将进程的euid设置为文件的ruid

  eg.chmod 4111 foo

  –chmod函数可以更改文件的存取权限
  

 **ps.set-user-ID的文件**

* 用户对某程序具有执行权限,用户运行程序后是**该进程拥有者**，进程代表该用户身份，而不再代表程序拥有者身份

>eg.用户Jane 运行命令 view memo.txt, view命令和文件memo.txt 权限:      -rwx--x--x 1 root bin 4515 Aug 14 13:08 view
>-rw------- 1 root bin 218 Aug 14 13:08 memo.txt 
>• Jane可以运行view, 但不能读文件memo.txt. 因为view程序调用read系统调用读文件时，访问文件被拒绝
>
>改变 view 程序权限，设置 SUID 位: 
>-rws--x--x 1 root bin 4515 Aug 14 13:08 view 
>•Jane运行设置了SUID位的程序, 再访问memo.txt 则成功. 因为 view 调用read读文件时, 系统不是以Jane身份读, 而是以root身份读.

##### 2)文件的sgid位

* 文件的组权限上可以设置特殊标志

  –当执行此文件时，将进程的egid设置为文件的rgid

  –root将httpd文件设置了sgid位

> eg.$ls -l httpd
> -rwxrwSr-x root root 392 Jan 18 08:46 httpd
> –test组的用户zhang在执行httpd文件时，其egid为root的组ID

##### 3)文件的sticky位

* sticky位

  –如果一个可执行程序这一位被设置，该程序执行结束时，程序正文的一个文本被保存在交换区。下次执行该程序时能较快装入内存eg.drwxrwxrw**t** 5 root root 1024 Feb 11 20:43 /tmp

- [x] 目录设置了sticky位，保护目录下的文件不能轻易被删除。
  - [x] 满足下面条件才能删除或更名目录下文件
  - [ ] 拥有此文件
  - [ ] 拥有此目录
  - [ ] 是超级用户

### 二. 目录的组织与权限表示

#### 1.UNIX目录

* 保存文件名和inode指针（索引节点）

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200104230036805.png" alt="image-20200104230036805" style="zoom: 50%;" />

* 目录的读权限位：仅能显示目录内的文件名

  –系统调用read、 opendir、 readdir, 该系统调用需要r权限

  –ls  dir

  –stat函数:  返回文件的有关结构信息

* 目录的执行位：可以遍历目录内文件属性信息

  –可以访问文件名对应的 inode 信息

  –cd dir   对转移后的目录需要具有执行权限

  –目录的读权限不能访问文件的inode 指针

* 目录的写权限位

  –写权限+执行权限可在目录下创建/删除文件（不需要对文件具有权限）

  –UNIX中目录的权限不具有继承性

  –**访问一个路径下的文件时，需要整个路径上的目录都有执行权限**

* 一个目录的拥有者，对该目录没有执行权限, 该用户不能访问目录下文件，即使对文件有权限也不能访问

  –有读权限而无执行权限				有执行权限而无读权限： 

  只能运行   ls someDir						可运行 ls -l someDir**/file** 

  而不能运行 ls **-l** someDir     			 不能运行 ls someDir 

### 三. 文件连接

#### 1.文件系统结构

![image-20200104235218423](C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200104235218423.png)

![image-20200104235304814](C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200104235304814.png)

#### 2.文件的硬链接

* 定义：

  两个目录指向同一i节点

  –每个i节点中有一个连接计数，是指向该点的目录项数

  –只有当连接计数减少为0时，才可删除该文件

  –删除一个目录项函数：unlink

  –删除一个文件的连接，并非释放该文件占用的磁盘块

  * link函数

    –多个目录项指向一个i节点

    –int link(const char *existingphth, const char *newpath)

    **–只有超级用户可以创建执行一个目录的硬连接**

#### 3.文件的符号连接

* 符号连接

  –对一个文件的间接指针，不同于硬连接，容易被清除

  –**任何用户可创建指向目录的符号连接**

#### 4.文件操作的系统调用

* –open(2), opendir(2), 

  – read(2), readdir(2), readlink(2), 

  –write(2),chdir(2),

  –stat(2), chmod(2), …

### 三. 用户，主体和客体

#### 1.用户标识

###### 1)用户ID

* 系统中，每个用户有一个唯一的UID

  –root的uid：0

  –用户不能更改其用户ID

  –内核根据用户ID检验其权限

  –/etc/passwd文件：保存用户名和用户ID映射关系

  –getuid()，getgid()

###### 2)组ID

* 组ID是系统管理员分配的。

  –用户可属于多个组

  –组文件/etc/group：组用户、组ID的映射关系

* 同组中的各成员之间可共享资源

#### 2.主体

* 用户 (accounts) 访问文件

* 具体的操作由主体(subjects)执行, 主体是**进程 (processes)**

* 当一个主体访问文件时, 需要知道其以哪个用户身份访问

  –ruid,euid,suid

* 进程保存相关联的用户信息

  | uid       |                                              |
  | --------- | -------------------------------------------- |
  | ruid,rgid | 用户是谁，在登录时取自口令文件中的登录项     |
  | euid,egid | 用于文件存取许可权检查，决定了对文件的访问权 |
  | suid,sgid | 由exec函数保存，保存了euid和egid的副本       |

* 执行一个文件操作时，进程的euid就是ruid

#### 3.进程标识

* 每个进程有一个非负整数的唯一进程ID

* 其他标识符

  | 函数                    | 作用                     |
  | ----------------------- | ------------------------ |
  | pid_t get**pid**(void)  | 返回调用进程的进程ID     |
  | pid_t get**ppid**(void) | 返回调用进程的父进程ID   |
  | pid_t get**uid**(void)  | 返回调用进程的实际用户ID |
  | pid_t get**euid**(void) | 返回调用进程的有效用户ID |
  | pid_t get**gid(**void)  | 返回调用进程的实际组ID   |
  | pid_t get**egid**(void) | 返回调用进程的有效组ID   |

* fork函数：内核创建新进程

* fork创建的新进程：子进程

  函数调用一次，返回两次

  –子进程返回值：0

  –父进程中返回值：子进程ID

  –子进程调用getppid获得父进程ID

#### 4.文件共享

* UNIX支持在不同进程间共享打开文件

* 内核使用三个数据结构，使多进程共享一个文件

  –（1）每个进程在进程表中都有一个记录项，每个记录项中包含一张打开文件描述符，包含：

  * 文件描述符标志
  * 指向一个文件表项的指针

  –（2）内核为所有打开文件维持一张文件表，包含：

  * 文件状态标志：读、写、增写、同步等
  * 当前文件位移量
  * 指向该文件v节点表项的指针

  –（3）每个打开文件(设备)都有一个v节点结构

  * 文件类型、文件指针信息

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200105114425545.png" alt="image-20200105114425545" style="zoom: 50%;" />

* 如果两个独立进程各自打开同一个文件

  –每个进程有自己的文件表项

  –同一文件只有一个v节点表项

  –可能有多个文件描述符指向同一文件表项

     如，fork后父、子进程对于每个打开的文件描述符共享同一表项

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200105114543796.png" alt="image-20200105114543796" style="zoom: 67%;" />

> 如何防止多个进程写同一个文件

* 原子操作：打开文件时设置O_APPEND标志

  在完成每个write后，在文件表项中的当前位移量增加所写字节数。如果文件位移量超过当前文件长度，则修改i节点表中文件长度

  如果用O_ APPEND标志打开一个文件，相应标志也写入文件表项的文件状态标志中。对文件做写操作，都添加到文件结尾处

  lseek函数值修改文件表项中的文件位移量，无任何I/O操作

  > eg.两个独立进程A和B，对同一文件添加操作
  >
  > –每个进程都打开了文件，每个进程都有自己的独立表项，但共享一个v节点表项
  >
  > –进程A写结束后，修改inode表中文件长度，调用lseek将文件位移量指向文件尾
  >
  > –B调用write前，当前文件位移量已指向文件尾，写结束后再调用lseek将文件位移量指向文件尾

### 五. 进程的uid切换

#### 1.UNIX中进程的uid

* 每个进程有三个 user IDs

  –real user ID (ruid) 进程拥有者

  –effective user ID (euid) 访问控制时需要的uid

  –saved user ID (suid) 保存的uid

* 每个进程有三个 group IDs

  –real group ID

  –effective group ID

  –saved group ID

* 进程由fork创建：子进程继承父进程的三个uid

* 当进程由*exec执行某文件时*

  –未设置setuid时，保持三个 user IDs 

  –当文件位设置了setuid时, 指定effective uid 和 saved uid 是所执行文件的拥有者

#### 2.exec函数

* 一般，fork进程后，子进程往往调用exec函数以执行另一程序
* 子进程执行新程序
* 新程序的进程ID并不改变
* 用新程序替换了当前进程的正文、数据、堆和栈
* 保持不变的特征：–进程ID、父进程ID、ruid、rgid、当前工作目录、文件锁、资源限制等

exec函数族

* execlp、execvp：指定文件名作参数

  –如果filename中包含/，则视为路径

  –否则按PATH环境变量，在有关目录中搜索可执行文件

<img src="C:\Users\13797\AppData\Roaming\Typora\typora-user-images\image-20200105123352515.png" alt="image-20200105123352515" style="zoom:50%;" />

#### 3.suid安全性

* 违背了最小特权原则

  –每个程序以及每个用户都应在最小特权下进行操作

解决方法：改变euid

* 一个进程执行一个 set-uid 程序后可放弃其权限

  –永久性放弃权限

  * 三个user IDs 都修改为低权限的user id

  –临时性放弃权限

  * 仅将effective uid 修改为低权限的user id ，同时将effective uid 保存在 saved uid；进程后续还可恢复为高权限用户继续运行 （ seteuid(suid) ）

* 选择合适的系统调用

  –Setresuid 具有清晰的语法含义

  –设置有效的 uid,seteuid(new_euid)

  –设置两个 user IDs, setreuid(new_uid,new_uid)

  –**应避免使用Setuid**