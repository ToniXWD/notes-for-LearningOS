# 2023.10.27
## 配置了rCore的环境, 在此小小地记录一下
整体上参考 官网[实验环境配置](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter0/5setup-devel-env.html)即可, 在此补充一下gdb的配置

按照指导书进行环境配置对于基础的代码运行是没有问题，但我发现自己按照指导书操作后无法进行`gdb`调试, 经过总结后在此处给出我的2种解决方案:
## 方案1: 安装完整的 `riscv-gnu-toolchain`
安装完整的`riscv-gnu-toolchain`流程如下, 次方法费时较长, 且占据空间较大, 更推荐第二种方法。
1. 安装依赖
    ```bash
    $ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
    ```
2. 克隆`riscv-gnu-toolchain`
    ```bash
    $ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
    ```
3. 编译安装
    ```bash
    $ cd riscv-gnu-toolchain
    $ ./configure --prefix=/usr/local
    $ sudo make
    ```
## 方案2: 编译安装 `riscv64-unknown-elf-gdb`
1. 安装依赖
    ```bash
    $ sudo apt-get install libncurses5-dev python2 python2-dev texinfo libreadline-dev
    ```
2. 下载`gdb`源码
此处我选择gdb-13.1, 该版本在`wsl2 Ubuntu22.04`上使用正常。
    ```bash
    # 推荐清华源下载
    wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gdb/gdb-13.1.tar.xz
    # 解压
    tar -xvf gdb-13.1.tar.x
    ```
3. 编译安装
只需要指定编译安装`riscv64-unknown-elf`并配置相关参数。
    ```bash
    $ cd gdb-13.1
    $ mkdir build && cd build
    $ ../configure --prefix=/your_path --target=riscv64-unknown-elf --enable-tui=yes
    $ make -j$(nproc)
    $ sudo make install
    ```

# 2023.10.28
## 阅读了ch0-ch2章节, 同时补充学习了`riscv`体系结构和汇编代码
`ch1`是构建裸机的执行环境, 虽然整体上能看懂代码逻辑, 但其中还是涉及到很多自己没有接触过的盲区, 比如连接脚本和`Rust`内联汇编等知识。`ch2`是将所有的用户代码链接到内核以实现批处理程序
这2个章节虽然困难，但展示了操作系统是如何从0开始搭建的，使我对地址和连接这些概念更熟悉了

# 2023.10.29
## 阅读ch3并完成了作业
`ch3`实现了分时多任务系统， 并实现了任务的掉地。
这一章最核心的收获在于进一步理解了调度的机制，我之前学习过`xv6`，但并未完全理解系统的调度机制，这一次终于有了进一步的收获。任务切换的本质就是一个暂时不会返回的函数， 这一函数保存了当前任务的多个寄存器， 并回复了另一个任务的寄存器。

以上的知识点在`xv6`时已经学习过了， 但我进一步发现， 其实内核一开始就存在自己的一个调度线程， 线程调度是先切换到这个线程， 再从这个线程执行相关的调度任务的

## ch3 作业
实现`sys_task_info`, 相对简单, 也就是在每次系统调用时找到任务的统计数组进行自增, 并判断任务是否第一次运行

# 2023.10.29-2023.10.30
## 阅读ch4并完成作业
`ch4` 就是实现`rCore`的虚拟内存和页表, 这里从逻辑上和`xv6`关系不大, 不过我更干兴趣的是`Rust`基于语言特性对于页表的设计:

- `Drop`特性
  代码为页帧实现了`Drop`特性, 因此只要其被托管的集合机构移除, 就可以让分配器再回收, 这是一个另外感到耳目一新的设计
- `MemorySet` 的设计
  `Memoryset`中包含多个逻辑段`MapArea`, 这样的设计使得地址空间的概念更让人容易理解

## ch4 作业
本章的作业中实现`mmap`难度较大, 因为需要对地址空间的数据结构 `MemorySet`, `MapArea`等有着较为清晰的逻辑认知

# 2023.11.1
## 阅读ch5并完成作业
`ch5`引入了进程的概念, 由于之前对进程等的基础知识了解较多, 因此本章的学习相对轻松愉快
不过解析`elf`格式的相关代码阅读起来还是挺困难, 相比是编译原理的知识不熟悉

# ch5作业
实现`stride`调度算法, 相对简单, 只需要在`tun_tasks`中`fetch`时进行排序, 并在`fetch`后更新`stride`即可

# 2023.11.2
## 阅读ch6并完成作业
`ch6`引入了文件系统

`rCore`的文件系统类型`xv6`, 可以划分为超级块, `inode`区域和数据块, `inode`区域和数据块又可以分为位图区域和数据区域, 存储的基本单位是`block`。

除了这个简单的文件系统本身之外，代码还展现出了内核和文件系统玻璃的特性， 内核对文件系统实现抽象， 文件系统通过抽象对内核提供服务，因此，文件系统应该是可以任意更换的， 这也符合真实世界操作系统的特性

## ch6作业
完成硬链接的创建于删除
硬链接创还能和删除本质的逻辑不难， 但较为繁琐，涉及到`inode`位置查询，判断是否分配新的`block`等操作，简单但繁琐
实际上, 繁琐的操作大部分是可以利用写好的接口实现的, 只是需要熟读代码

实际上我的实现是有`bug`的, 我在删除根节点目录项后, 没有判断根节点是否有剩余的`block`需要回收, 这对于`rCore`的测试来说没有问题, 毕竟目录项很少, 但实事求是的说是不完善的, 哈哈

# 2023.11.3
## 阅读ch7
`ch7`描述了管道这一古老`ipc`的实现
管道的底层实现是一个循环数组, 对齐实现各种`trait`后抽象成一个文件描述符
添加读端和写端的指针或引用计数, 控制管道的打开与关闭

除了管道本身外, 这种文件描述符的抽象手段已经不止一次让我感叹`Unix`设计的精巧

总所周知, `socket`也是一个文件描述符, 希望以后有机会仔细研究下`socket`的实现

# 2023.11.4
## 阅读ch8并完成了作业
`ch8` 介绍了各种线程和各种线程同步资源的视线
这一部分其实就是对进程进行了进一步的剥离, 有了之前的基础这一部分的代码读起来不算困难, 但是作业是我任务最难的一个

## ch8作业
`ch8`的作业是实现死锁检测, 算法本身描述的足够清晰, 但难点在于算法实现线上, 难点包括但不限于
- 各个数据结构应该放在`PCB`还是`TCB`上? 还是说各自放一部分?
- 同时访问`PCB`和多个`TCB`的资源如何在语言特性上避免借用冲突?
- 由于`m_available`或`s_available`等资源是动态扩张的, 什么时候控制其扩容以避免后续的数组越界?
- 本作业其实是要实现`sys_get_time`系统调用的
