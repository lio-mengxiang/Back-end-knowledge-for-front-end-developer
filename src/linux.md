## 前言

题目均是真题，职位为Node.js开发、全栈（node.js + react）、大前端（需要会node.js的前端职位）中的关于linux的面试题。

我是为了拓宽自己的将来的就业面哈哈，其实本质来说完全具备全栈的能力了，所以总结一波node.js相关的面试题，整体难度偏初级和中级。既然是服务端，linux基础命令的问题肯定跑不脱。

本文记录在[node面试题系列里](https://github.com/lio-mengxiang/node-interview)
## 频率最高的题：请说一下你常用的linux命令

这个自由发挥吧，常用的不就是cd、ls、vim这些命令吗。。。 不过你不是非常熟悉的命令不要说，面试官会接着问，比如你说ps命令，他会问ps -ef和ps -aux的差别等等


## LINUX这边常用的查看日志的指令

1.  `tail`: 用于查看日志文件的末尾内容，通常用于实时监控日志文件的变化，常与`-f`选项一起使用，实时输出新增的日志信息。例如，`tail -f /var/log/syslog`可以实时查看系统日志文件的更新内容。
1.  `grep`: 用于在文本中搜索指定的模式，通常用于从日志文件中查找特定的信息。例如，`grep "error" /var/log/syslog`可以查找系统日志文件中包含"error"关键词的信息。

切记： 实时输出新增的日志信息是一个非常实用的命令，并且面试很常见，一定要记住。

## LINUX怎么杀掉一个进程

常见的就是 kill -15 pid

面试官会追问如何强杀一个进程，可以用 kill -9 pid, 我们顺便普及一下kill和信号是什么.

### 常见的信号

上面的 -9代表的是SIGKILL信号，-15代表的是SIGTERM信号，我们在使用nodejs的时候，通常会有在写代码时，可能执行退出命令，这时候就需要传一个信号量，就可以用我们这里的学到的9和15传递给它。

> 在 Node.js 中，`process.exit()` 方法用于终止当前进程，传递给该方法的数字参数表示进程的退出码。在这种情况下，`process.exit(15)` 和 `process.exit(2)` 的区别在于它们返回的退出码不同。

> 在 Unix/Linux 系统中，15 号信号（SIGTERM）表示终止进程的请求，而 2 号信号（SIGINT）表示中断进程的请求。因此，`process.exit(15)` 表示进程接收到终止请求，而 `process.exit(2)` 表示进程接收到中断请求。

简单来说SIGTERM：当前进程接收到这个信号时，大多会先释放自己的资源，再停止进程，属于正常关闭。但有时候，可能这个进程正在进行一些I/O操作所以不能立即关闭。

这时候SIGKILL：属于强制关闭进程，不管你现在是啥情况。

## linux下进程间通信方式？管道、共享内存什么场景下会用？两个进程不在同一个机器如何通信？

1.  管道（Pipe）：管道是一种半双工的通信方式，通常用于父子进程之间或者兄弟进程之间的通信。它的特点是只能在具有亲缘关系的进程之间使用，而且数据只能在一个方向上流动。
1.  命名管道（Named Pipe）：命名管道是一种特殊的文件，可以被多个进程共享访问，可以用于非亲缘关系进程之间的通信。
1.  共享内存（Shared Memory）：共享内存是一种快速的进程间通信方式，它将一段内存区域映射到多个进程的地址空间中，多个进程可以直接读写这段共享内存，以达到高效的通信目的。
1.  消息队列（Message Queue）：消息队列是一种基于消息的通信方式，它允许进程将消息发送到一个队列中，供其他进程读取。
1.  套接字（Socket）：套接字是一种通用的进程间通信方式，可以在本地或者远程不同的主机之间通信。

对于管道和共享内存，它们的使用场景如下：

1.  管道适用于数据流式的场景，比如父子进程之间的通信，或者进程输出传递给另一个进程进行处理等。
1.  共享内存适用于大量数据的读写场景，比如多个进程需要频繁地读写共享的数据区域，共享内存可以提高数据传输的效率。（在nodejs中主要体现在sharedArrayBufferr实现多个worker共享数据）

这里补充一下，node.js在linux下使用了Socket实现了IPC，并且内部它们与网络 socket 的行为比较类似，属于双向通信。不同的是它们在系统内核中就完成了进程间的通信，而不经过实际的网络层，非常高效。在 Node 中，IPC 通道被抽象为 Stream 对象。

不在一个机器可以使用socket通信，比如使用nodejs的net模块。

## 进程通信：信号、信号量是什么，进程间通信消息队列机制的本质是什么？

信号是IPC通信的一种机制，信号量是一种用于进程间同步和互斥的机制。

信号（Signal）是一种在进程间通信中用于处理异步事件的机制。当某个进程需要向另一个进程发送通知时，可以通过向该进程发送信号来实现。信号是一种软件中断，可以被操作系统或其他进程发送，并被接收进程处理。

Linux 中常见的信号包括 `SIGINT`、`SIGTERM`、`SIGKILL` 等，这些信号通常用于终止进程或处理其他进程间的通知事件。

信号量（Semaphore）是一种用于进程间同步和互斥的机制。它是一个计数器，用于控制多个进程对共享资源的访问。当一个进程进入临界区时，它会将信号量的值减一，表示已经使用了共享资源；当进程退出临界区时，它会将信号量的值加一，表示该共享资源已经释放。其他进程可以通过等待信号量变为可用状态，然后再进入临界区。

### 进程间通信消息队列机制的本质

消息队列是 Linux 进程间通信的一种方式，它允许多个进程通过向共享的队列中发送和接收消息来进行通信。

在 Linux 中，消息队列本质上是由内核维护的一段缓冲区，是存放在内存中的消息链表。

-   消息队列允许一个或多个进程向它写入或读取消息。
-   消息队列可以实现消息的「随机查询」，不一定非要以先进先出的次序读取消息，也可以按消息的类型读取。比有名管道的先进先出原则更有优势。
-   消息队列的生命周期随内核，如果没有释放消息队列或者没有关闭操作系统，消息队列就会一直存在。而匿名管道随进程的创建而建立，随进程的结束而销毁。

## 如何查看linux进程信息，其中vsz和rss有什么区别？

我一般是用 ps -aux，能看到的信息更全一些。

其中ps -aux 命令会显示vsz和rss两个指标的信息，简单来说rss就是程序实际上使用的内存，包括堆栈，动态链接库和本身正在内存里的代码大小。vsz会更大一些，因为它不管你实际用到多少，而是你整个代码引用了哪些动态链接库，使用了的全部代码，不管有没有在内存（有些存在磁盘上），都算进来了，并且vsz还包括了虚拟内存。


所以整体来看rss更能反映真实的代码内存占用情况。


**示例如下：**

  
假设进程A的二进制文件是500K，并且链接了一个2500K的动态库，堆和栈共使用了200K，其中100K在内存中（剩下的被换出或者不再被使用），一共加载了动态库中的1000K内容以及二进制文件中的400K内容至内存中，那么：

```
RSS: 400K + 1000K + 100K = 1500K

VSZ: 500K + 2500K + 200K = 3200K
```

## Linux文件描述符是什么

一个 Linux 进程启动后，会在内核空间中创建一个 PCB 控制块，PCB 内部有一个文件描述符表（File descriptor table），记录着当前进程所有可用的文件描述符，即当前进程所有打开的文件。


除了文件描述符表，系统还需要维护另外两张表：  


-   打开文件表（Open file table）
-   i-node 表（i-node table）

  
文件描述符表每个进程都有一个，打开文件表和 i-node 表整个系统只有一个，它们三者之间的关系如下图所示。  
  


![Linux文件描述符表示意图](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34f8b17928164e9295598f3872b49268~tplv-k3u1fbpfcp-zoom-1.image)

  
从本质上讲，这三种表都是结构体数组，0、1、2、73、1976 等都是数组下标。表头只是我自己添加的注释，数组本身是没有的。实线箭头表示指针的指向，虚线箭头是我自己添加的注释。  
  
**你看，文件描述符只不过是一个数组下标吗！**  
  
所以最终文件描述符找到了i-node表里的对应信息。

inode 表包含文件名、文件大小、文件创建时间、文件所有者、文件权限等。每个文件在文件系统中都有唯一的 inode 号，通过 inode 号可以找到该文件的所有元数据和数据块信息。当进程打开一个文件时，内核会从文件系统中查找该文件对应的 inode 号，并将其记录在文件的表项中，以便进行后续的文件操作。

## linux查找文件的指令

find命令，常见的使用 -name 根据文件名查询，或者 -size 根据文件大小查询。

### 将当前目录及其子目录下所有最近 20 天内更新过的文件列出:
```

find . -ctime  20
```

## Linux下查看网络端口？

一般都是用netstat，我简单来讲一下这个命令吧，这个也是非常容易考到的题。

### 含义：

netstat命令用于显示网络状态，在工作中经常要查看端口的占用情况，比如启动一个应用避免端口冲突。

常用参数：

- -t 显示tcp传输协议的连接情况
- -u 显示udp传输协议的连接状况
- -l 表示listening，显示正在监听状态的服务，也可以用-a表示所有状态
- -n： 表示numeric，用数字形式显示端口号
- -p：表示process 表示显示后台进程

常用组合： neststat -lntp



## Linux下查看负载情况？

平均负载，是指处于运行或不可打扰状态的进程的平均数。（主要是R和D状态的进程）

在 Linux 系统中，要查看负载情况一般使用 uptime 命令（w 命令和 top 命令也行）*

```
$ uptime
16:33:56 up 69 days,  5:10,  1 user,  load average: 0.14, 0.24, 0.29
```
以上信息的解析如下：

- 16:33:56 : 当前时间
- up 69 days, 5:10 : 系统运行了 69 天 5 小时 10 分
- 1 user : 当前有 1 个用户登录了系统 

- load average: 0.14, 0.24, 0.29 : 系统在过去 1 分钟内，5 分钟内，15 分钟内的平均负载

平均负载一般小于你的cpu总核心数就是正常的，说明有空闲的cpu。


## linux:统计一个文件里指定关键字的出现的次数

```
grep -wo 'hello' file.txt | wc -l
```

其中，“-w”选项可以确保只匹配完整的单词，“-o”选项可以让grep命令只输出匹配到的单词，“wc -l”命令可以统计行数，也就是匹配到的单词出现的次数

## 如何把客户机的公钥添加到服务器的列表中（SSH）

目的
避免重复输入用户密码，通过SSH自动登录，节约时间

在Linux系统中，可以使用ssh命令将客户机的公钥添加到服务器的authorized_keys文件中，实现无需密码登录。具体步骤如下：

1.  在客户机上生成公钥和私钥，如果已经生成过可以跳过这一步骤。使用ssh-keygen命令生成公钥和私钥，例如：

```
ssh-keygen
```

该命令会在用户的家目录下生成公钥文件（id_rsa.pub）和私钥文件（id_rsa）。

2.  将客户机的公钥添加到服务器的authorized_keys文件中。将公钥文件内容复制到剪贴板中，然后使用ssh命令将公钥添加到服务器的authorized_keys文件中。例如，假设客户机的公钥文件为“id_rsa.pub”，服务器的IP地址为“192.168.0.1”，用户名为“username”，可以输入以下命令：

```
ssh-copy-id -i ~/.ssh/id_rsa.pub username@192.168.0.1
```

在这个命令中，“-i”选项用来指定公钥文件路径，“username”是服务器的用户名，“192.168.0.1”是服务器的IP地址。

3.  输入服务器的登录密码。如果之前没有将客户机的公钥添加到服务器的authorized_keys文件中，会提示输入服务器的登录密码。输入密码后，客户机的公钥会被添加到服务器的authorized_keys文件中。
3.  完成添加。添加成功后，可以使用ssh命令无需密码登录服务器。例如，输入以下命令即可登录服务器：

```
ssh username@192.168.0.1
```

在这个命令中，“username”是服务器的用户名，“192.168.0.1”是服务器的IP地址。

## VIM（替换，删除等指令）

简单描述一些常见的方式：

替换命令：

1.  :s/old/new/g：将当前行中所有的“old”替换成“new”。其中，g表示全局替换，如果不加g，则只会替换当前行中第一个匹配到的“old”。
1.  :%s/old/new/g：将整个文档中所有的“old”替换成“new”。


删除命令：

1.  x：删除光标所在位置的字符。
1.  dd：删除当前行。


## Linux解压文件操作

 **打包文件**

将一个或多个文件或目录打包为单个tar文件：

```
tar -czvf archive.tar.gz /path/to/file1 /path/to/file2 /path/to/dir1
```

-   `-c`：表示创建一个新的tar文件。
-   `-z`：表示使用gzip进行压缩。
-   `-v`：表示输出详细信息，可以看到打包过程中的每个文件。
-   `-f`：表示指定输出的文件名，后面紧跟要打包的文件和目录。

**解压文件**

将tar文件解压到指定的目录：

```
tar -xzvf archive.tar.gz -C /path/to/destination
```

-   `-x`：表示解压缩一个已经存在的tar文件。
-   `-z`：表示使用gzip进行解压缩。
-   `-v`：表示输出详细信息，可以看到解压缩过程中的每个文件。
-   `-f`：表示指定要解压的tar文件名，后面紧跟要解压缩到的目录。
-   `-C`：表示指定解压缩到的目录。

## 100行5列的文件每列用空格隔开计算第三列数字的和
```
awk '{sum+=$3} END {print sum}' filename.txt
```

## Linux命令，查看8080端口的进程

```
lsof -i :8080
```
使用 `lsof` 命令可以查看哪些进程正在使用特定的文件或者网络端口


## linux系统下常用的监控命令?

我是常用top命令，所以简单介绍一下top吧。

top命令可以帮我们全面了解cpu，内存，进程等一系列当前服务的状态。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c145c0ad2d84a0b984ce477931f9037~tplv-k3u1fbpfcp-watermark.image?)

在top命令的输出中，可以看到以下信息：

-   第一行：系统当前时间、运行时间、登录用户数、系统负载。
-   第二行：总进程数、运行中进程数、睡眠中进程数、停止进程数、僵尸进程数。
-   第三行：CPU使用情况，包括用户态、系统态、空闲和等待IO的CPU占用百分比。
-   第四行：内存使用情况，包括物理内存总量、空闲内存、已用内存、缓存和缓存已用内存。
-   第五行：交换分区使用情况，包括总交换分区、空闲交换分区和已用交换分区。
-   进程信息：进程的PID、用户、CPU占用、内存占用、进程状态、启动时间、进程命令等。


## 当我们创建一个linux用户的用户密码存放在哪个文件目录？


在Linux中，用户密码不是明文存储的，而是经过加密处理后存储在系统中。具体来说，密码是以加密方式存储在系统的/etc/shadow文件中。

该文件只有root用户有读写权限，其他用户不能访问该文件。这样可以保护用户密码不被其他用户访问或泄露。

在/etc/shadow文件中，每个用户的密码信息占用一行，包括用户名、加密后的密码、最后一次修改密码的日期、密码过期时间等信息。用户密码的加密方式通常采用哈希（hash）算法，如MD5、SHA等。


## 查看权限命令

一般都是用`ls -l`: 显示当前目录下的所有文件和目录的详细信息，包括文件权限、所有者、所属组、文件大小、创建日期等信息

## Linux IO有哪些方式

总共有5种IO方式

**五种I/O模型：**

-   同步阻塞型、同步非阻塞型、IO多路复用型、信号驱动I/O型、异步I/O型

###  阻塞式I/O模型（Blocking I/O Model） 

同步阻塞IO模型是最简单的IO模型，用户线程在内核进行IO操作时被阻塞

### 非阻塞式I/O模型（Non-Blocking I/O Model）
在这种I/O模型中，当应用程序执行一个I/O操作时，它不会一直等待，而是立即返回，不管I/O操作是否完成。

用户线程需要不断地发起IO请求，直到数据到达后，才真正读取到数据，继续执行。即“轮询”机制

### I/O复用模型（I/O Multiplexing Model）
简单来说，上面非阻塞I/O会轮训问好没好，那如果请求有几百万个，是不是同时有几百万个线程人问，这样是对资源的巨大浪费。


为了解决这个问题，我们使用中介者模式，让一个线程作为代理，它去轮训问几百万个接口，应用程序可以在一个线程中等待多个I/O操作完成，从而减少线程数量和系统开销。 当一个I/O操作完成时，应用程序会被通知并处理完成的I/O操作。

### 信号驱动I/O模型（Signal Driven I/O Model）

但是上面的I/O复用模型仍然有一个线程要去轮训，有没有办法不轮询？
在这种I/O模型中，当应用程序执行一个I/O操作时，它会立即返回，并注册一个信号处理函数。当I/O操作完成时，内核会发送一个信号给应用程序，通知它I/O操作已完成。应用程序可以在信号处理函数中处理完成的I/O操作。

###  异步I/O模型（Asynchronous I/O Model）
在这种I/O模型中，当应用程序执行一个I/O操作时，它会立即返回，并注册一个回调函数。当I/O操作完成时，内核会调用回调函数，并传递完成的I/O操作的结果。应用程序可以在回调函数中处理完成的I/O操作。


## 用户程序写4KB的数据，写到磁盘上，经历的过程 


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48d865f229d94eb999c18d0bab12744f~tplv-k3u1fbpfcp-watermark.image?)

1.  用户程序将数据写入文件，通过系统调用将数据传递给操作系统内核。
2.  上图可以看到，写文件首先是借助虚拟文件系统去往缓冲区写数据的，虚拟文件系统屏蔽了各种具体的文件系统，提供了统一的调用方式。
3.  操作系统内核将数据存储到系统内核缓存（buffer cache）中，等待写入磁盘。缓存是操作系统用来暂存文件系统的数据和元数据的内存区域。当应用程序向磁盘写入数据时，这些数据首先被写入到内核缓存中，再由内核异步写入磁盘。
6.  当磁盘控制器将数据从磁盘缓存区写入磁盘后，会向操作系统内核发送一个中断信号（interrupt），表示数据已经成功写入磁盘。此时，操作系统内核会将文件系统的元数据更新，包括文件大小、修改时间等。
  

## 进程线程协程的区别


1.  进程是操作系统资源分配的基本单位，它拥有独立的内存空间、进程控制块等，进程之间的通信需要借助进程间通信（IPC）机制。而线程是进程的执行单位，它共享进程的内存空间和资源，线程之间的通信可以直接通过共享内存等方式实现。
1.  进程之间的切换需要涉及上下文切换，需要保存和恢复大量的进程状态信息，比较耗时。线程之间的切换则比较轻量级，只需要保存和恢复少量的线程状态信息。
1.  协程是一种轻量级的线程，它不依赖于操作系统的线程调度器，而是由程序自身控制。协程之间的切换是由程序显式调用的，因此比线程切换更加轻量级，同时由于不需要操作系统的参与，协程的创建和销毁的开销也比线程小。
1.  进程和线程之间的切换是由操作系统调度器控制的，因此具有较高的安全性和稳定性。而协程的调度和执行都由程序自身控制，因此对程序的编写和调试要求较高，同时需要注意协程之间的调度和同步问题，否则容易出现死锁等问题。

## Linux里面的软连接和硬链接

硬链接和原⽂件是共⽤⼀个inode号，说明他们是同⼀个⽂件，⽽软链接和原⽂件拥有不同的inode号，表明他们是两个不同的⽂件。

硬链接：由于Linux下的⽂件是通过索引节点（Inode）来识别⽂件，硬链接可以认为是⼀个指针，指向⽂件索引节点的指针，系统并不为它重新分配inode。每添加⼀个⼀个硬链接，⽂件的链接数就加1。


软链接：软链接克服了硬链接的不⾜，没有任何⽂件系统的限制，任何⽤户可以创建指向⽬录的符号链接。因⽽现在更为⼴泛使⽤，它具有更⼤的灵活性，甚⾄可以跨越不同机器、不同⽹络对⽂件进⾏链接。


当源文件被删除时，硬链接依然可以访问文件内容，因为实际上硬链接和源文件是同一个文件，只是文件名和目录项不同；而软链接则会失效，因为它仅仅是一个指向源文件的路径名符号。

## 例如Linux问如果有一台主机连不上外网了应该怎么排查问题，

1.  确认网络连接是否正常。可以使用 `ping` 命令来测试主机与外网的连通性，例如 `ping www.google.com`。如果无法连接到外网，可以尝试检查主机的网络配置，例如 IP 地址、网关、DNS 等是否正确配置。
1.  检查防火墙配置。如果主机上有防火墙，需要检查防火墙配置是否允许主机访问外网。可以使用 `iptables -L` 命令查看当前防火墙规则，如果需要允许主机访问外网，可以添加相应的规则。
1.  检查路由表配置。如果主机无法访问外网，可能是因为路由表配置不正确导致的。可以使用 `route -n` 命令查看当前路由表配置，如果需要添加或修改路由规则，可以使用 `route add` 命令。
1.  检查 DNS 配置。如果主机无法访问外网，可能是因为 DNS 配置不正确导致的。可以使用 `cat /etc/resolv.conf` 命令查看当前 DNS 配置，如果需要修改 DNS 配置，可以编辑 `/etc/resolv.conf` 文件或者使用 `nmcli` 命令进行配置。
1.  检查网络设备。如果主机无法访问外网，可能是因为网络设备（例如网卡、交换机等）故障或配置不正确导致的。可以检查网络设备的物理连接和配置是否正确。

## linux中断流程，谈谈你对中断上下文的理解

中断可以是硬件中断（例如 I/O 中断）或软件中断（例如系统调用）。

中断流程可以简单地描述为：

1.  硬件检测到一个中断事件，向 CPU 发送中断请求信号。
1.  CPU 收到中断请求信号，暂停当前执行的程序，并将当前 CPU 寄存器中的状态保存到内核堆栈中，以便在中断处理程序结束后能够恢复现场。
1.  CPU 跳转到中断服务程序的入口点开始执行中断处理程序，处理中断事件。
1.  中断处理程序执行完毕后，恢复中断前的现场，将 CPU 寄存器中的状态从内核堆栈中恢复，然后返回到中断前的程序继续执行。

在中断处理程序执行期间，中断处理程序可以在中断上下文中运行，也可以在进程上下文中运行。中断上下文是指中断服务程序执行期间，处理器无法访问当前进程的上下文环境，因此中断服务程序必须在内核上下文环境中执行。

## linux中的线程一般是怎么调度的？

inux 中的线程是由内核调度的，常见的调度模式例如基于时间片轮转算法实现的。Linux 内核为每个线程分配一个时间片，在时间片用完之前，线程可以一直执行。当时间片用完后，内核会将当前线程从 CPU 上移除，然后选择下一个可执行的线程继续执行，直到所有的线程都执行完毕。

Linux 内核还提供了一些控制线程调度的接口，例如 `nice` 命令可以通过设置线程的优先级来控制其调度

## vim如何显示行号，Shell脚本自动化如何做

要在 vim 中显示行号，可以使用 `:set number` 命令。这会在 vim 的左侧显示每一行的行号。如果要取消行号的显示，可以使用 `:set nonumber` 命令。

Shell 脚本自动化我知道有很多方式，我一般是使用定时任务去执行该脚本

1.  编写 Shell 脚本并手动执行：将需要自动化的操作封装到 Shell 脚本中，然后使用 cron 或其他定时任务工具在指定时间执行该脚本。

如果是分布式定时任务，会使用Kubernetes 中的 CronJob去执行。
