## 1. IO功能的发展
随着计算机系统的发展，单个部件的复杂度和完善度也随之增加，这在IO功能上表现最为明显。IO功能的发展可概括为以下阶段：
1. 处理器直接控制外围设备，这在简单的微处理器控制设备中可以见到
2. 增加了控制器或IO模块。处理器使用非中断的程序控制IO。在这一阶段，处理器开始从外部设备接口的具体细节中分离出来
3. 与阶段2唯一区别是采用了中断方式，处理器无须费时间等待执行一次IO操作
4. IO模块通过DMA直接控制存储器。现在可以在没有处理器参与的情况下，从内存中移出或往内存中移入一块数据，仅在传送开始和结束时才需要用到处理器
5. IO模块有一个单独的处理器，有专门为IO设计的指令集。CPU指导IO处理器执行内存中的一个IO程序。IO处理器在没有CPU干涉的情况下取指令并执行这些指令。这就使得CPU可以指定一系列的IO活动，且仅在整个序列执行完成后CPU才被中断
6. IO模块有自己的局部存储器，事实上本身是台计算机。使用这种体系结构可以控制许多IO设备，并使需要CPU参与的程度达到最小。这种结构通常用于控制与交互终端的通信，IO处理器负责大多数控制终端的任务

## 2. 直接存储器访问DMA
<img src="D:\Project\IT-notes\操作系统\img\典型的DMA框图.png" style="width:200px;height:300px;" />

DMA单元能够模拟处理器，且实际上能够像处理器一样获得系统总线的控制权。这样做是为了能利用系统总线与存储器进行双向数据传送

DMA技术工作流程如下：处理器试图读或写一块数据时，它通过向DMA模块发送以下信息来给DMA模块发出一条命令
1. 请求读操作或写操作的信号，通过在处理器和DMA模块之间使用读写控制线发送
2. 相关的IO设备地址，通过数据线传送
3. 从存储器中读或向存储器中写的起始地址，在数据线上传送，并由DMA模块保存在其地址寄存器中
4. 读或写的字数，也通过数据线传送，并由DMA模块保存在其数据计数寄存器中

然后处理器继续执行其他工作，这个IO操作已经委托给DMA模块执行（DMA充当代理角色）。DMA模块直接从存储器中或向存储器中传送整块数据，并且数据不再需要通过处理器。传送结束后，DMA模块向处理器发送一个中断信号（只有在传送开始与结束时才会用到处理器）

<img src="D:\Project\IT-notes\操作系统\img\可选DMA配置.png" style="width:400px;height:400px;" />

## 3. IO缓冲
假设某个用户进程需要从磁盘中读入多个数据块，每次读一块，每块的长度为512字节。这些数据将被读入用户进程地址空间中的某个区域，如从虚拟地址1000到1511的区域。最简单的方法是对磁盘单元执行一个IO命令，并等待数据传送完毕。这个等待可以是忙等待，也可以是进程被中断挂起

但存在两个问题：
1. 程序被挂起，等待相对比较慢的IO完成
2. 在数据传送期间，从1000到1511的虚拟地址单元必须保留在内存中，如果使用分页机制，则需要将目标地址的页锁定
3. 该进程的一部分页可以换出，但不能把进程的所有页换出，可能会造成单进程死锁（进程等待IO事件发生，IO等待进程换入）

基于避免开销和低效的操作，使用了缓冲（在输入请求发出前就开始执行输入传送，并且在输出请求发出一段时间之后才开始执行输出传送）

**存在面向块的IO设备和面向流的IO设备：面向块的设备将信息保存在块中，块的大小通常是固定的，传送过程中一次传送一块；面向流的设备以字节流的方式输入输出数据**

<img src="D:\Project\IT-notes\操作系统\img\IO缓冲方案－输入.png" style="width:350px;height:500px;" />

### 单缓冲
输入传送的数据被放到系统缓冲区中。当传送完成时，进程把该块移到用户空间，并立即请求另一块（**预读或预先输入**）。用户进程可在下一数据块读取的同时，处理已读入的数据块。由于输入发生在系统内存而非用户进程内存中，因此操作系统可以将该进程换出

也适用于面向块的输出。当准备将数据发送到一台设备时，首先把这些数据从用户空间复制到系统缓冲区，它最终是从系统缓冲区中被写出的。发请求的进程现在可以自由执行，或者在必要时换出

面向流的单缓冲方案以每次传送一行的方式或每次传送一字节的方式使用，可以使用缓冲区保存单独一行数据。在输入期间用户进程被挂起，等待整行的到达。对于输出，用户进程可以把一行输出放在缓冲区中，然后继续执行

### 双缓冲
在一个进程向一个缓冲区传送数据的同时，操作系统正在清空另一个缓冲区

### 循环缓冲
如果进程需要执行大量IO操作，那么仅有双缓冲不够，通常使用多于两个缓冲区的方案弥补不足。而这组缓冲区本身被当作循环缓冲区

## 4. 磁盘调度

<img src="D:\Project\IT-notes\操作系统\img\磁盘调度算法比较.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\操作系统\img\磁盘调度算法.png" style="width:700px;height:300px;" />

### 先进先出FIFO
按顺序处理队列中的项目

### 优先级
按照一定规律设置磁盘IO请求队列中任务的优先级

### 后进先出

### 最短服务时间优先SSTF
选择使磁头臂从当前位置开始移动最少的磁盘IO请求，因此SSTF策略总是选择最小寻道时间的请求

### 电梯策略SCAN
要求磁头臂仅沿一个方向移动，并在途中满足所有未完成的请求，直到它到达这个方向上的最后一个磁道，或者在这个方向上没有其他请求为止，接着发转服务方向，沿相反方向扫描，同样按顺序完成所有请求
SCAN策略偏爱请求接近最靠里或最靠外的磁道的作业，且偏爱最近的作业

### C-SCAN
把扫描限定在一个方向上，当访问到沿某个方向的最后一个磁道时，磁头臂返回到磁盘相反方向末端的磁道，并再次开始扫描

### N步SCAN
把磁盘请求队列分成长度为N的几个子队列，每次用SCAN处理一个子队列。在处理某个队列时，新请求必须添加到其他某个队列中。如果在扫描的最后，剩下的请求数小于N，则它们全在下一次扫描时处理

### FSCAN
使用两个子队列，扫描开始时，所有请求都在一个队列中，另一个队列为空。在扫描过程中，所有新到的请求都放入另一个队列中。因此对新请求的服务会延迟到老请求处理完成之后

## 5. RAID
<img src="D:\Project\IT-notes\操作系统\img\RAID级别.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\操作系统\img\RAID级别图示.png" style="width:400px;height:850px;" />

## 6. 磁盘高速缓存
磁盘高速缓存是内存中为磁盘扇区设置的一个缓冲区，包含有磁盘中某些扇区的副本。出现对某一特定扇区的IO请求时，首先会进行检测，以确定该扇区是否在磁盘高速缓存中。若在，则该请求可通过这个高速缓存来满足；若不在，则把被请求的扇区从磁盘读到磁盘高速缓存中

当一个IO请求从磁盘高速缓存中得到满足，磁盘高速缓存中的数据必须传送到发送请求的进程：在内存中把这一块数据从磁盘高速缓存传送到分配给该用户进程的存储空间中；或者简单使用一个共享内存，传送指向磁盘高速缓存中相应项的指针

当一个新扇区被读入磁盘高速缓存时，需要一个页面置换策略，最常用的是**最近最少使用算法LRU**
其次是**最补偿使用页面置换算法LFU**：置换集合中访问次数最少的块，通过给每个块关联一个计数器实现
为了克服LFU中由于局部性原理出现短时间重复访问某一块导致计数器值高，但该块将来并没有很快被访问的情况，出现一种基于频率的置换算法

<img src="D:\Project\IT-notes\操作系统\img\基于频率的置换.png" style="width:400px;height:350px;" />