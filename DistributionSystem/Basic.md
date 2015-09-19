
## 单机组成的5要素

冯·诺依曼型计算机的5个组成部分

+ 输入
+ 输出
+ 控制器
+ 运算器
+ 存储器

如下图所示：

![组成计算机的5要素](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/ds-basic1.png?raw=true "组成计算机的5要素")

## 线程和进程的执行模式

#### 阿姆达尔定律

阿姆达尔定律（Amdahl's law）:

![阿姆达尔定律](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/Amdahl's-law.png?raw=true "Amdahl's law")

    P : 程序中可并行部分的程序在单核上执行时间的占比
    N : 处理器的个数（总核心数）
    S(N) : 程序在N个处理器（总核心数）相对在单个处理器（单核）中的速度提升比

这个公式表明 ： 
> 程序中可并行代码的比例决定你增加处理器（总核心数）所能带来的速度提升上限，是否能达到这个上限，还取决于很多其他的因素。
> 例如，当p = 0.5 时，我们可以计算出速度提升的上限就是2. 而如果p = 0.2，速度提升的上限就是 1.25 。
> 可见，再多核的时代，并发程序的开发或者说提升程序的并发性是多么重要 。

#### 几种常见的多线程模式

+ 互不通信的多线程模式 ： 多个线程在系统中并发执行，线程之间不需要处理共享的数据，也不需要进行动作协调，多个独立的线程各自完成各自的线程中的工作。（这种情况比较简单）
+ 基于共享容器协同的多线程模式 ： 在另一些场景中，我们需要在多个线程之间对共享的数据进行处理。对于这种在多线程环境下对同一份数据的访问，我们需要有所保护和控制以保证访问的正确性。
+ 通过事件协同的多线程模式 ： 除了对并发访问的控制，线程间会存在着协调的需求。例如A、B两个线程，B线程需要等到某个状态或者事件发生后才能继续自己的工作，而这个状态改变或事件产生和A线程相关。那么在这个场景下，就需要线程之间的协调

此外，线程之间还会传递数据。因为这些线程共享进程的内存空间，所以线程间的传递数据相对容易一些了

#### 多进程模式

多进程和多线程模式的不同与相同 ：

| 区别 | 原因 |
|:-----:|:----:|
|内存共享 | 线程是属于进程的，一个进程内的多个线程共享了进程的内存空间；而多个进程之间的内存空间是独立的 |
|交换数据的方式 | 线程是属于进程的，一个进程内的多个线程共享了进程的内存空间；而多个进程之间的内存空间是独立的 |
|通信、协调 | ... |
|事件通知 | ... |
|互斥锁的释放 | ... |

其它方面，比如 ：

+ 多进程相对于单进程多线程的方式来说，资源控制更加容易实现
+ 多进程的单个进程问题，不会造成整体的不可用

这两点是多进程区别于***单进程多线程***方式的两个特点 ，当让使用多进程会比多线程稍微复杂一些。多进程间可以共享数据，但是其代价比多线程要大，会涉及到序列化和反序列化的开销

而我们的分布式系统是多机组成的系统，可以近似看成单机多进程变为了***多机多进程***。 

+ 从单机到多机也有些变化。 原来在单机OS上支持的功能现在都需要去另外实现了
+ 也有些好处。例如，当单个机器出问题时，如果处理的好，就不会影响整体的集群

从单线程到多机的系统，每次的变化都使得我们在处理某些功能（例如贡献数据、通信）时会有不同。而在另外一个关乎故障的方面也会不一样，不同的方式可能造成的故障影响面也是不同的。比如 ：

+ 单线程和单进程多线程的程序遇到机器故障、OS问题或者自身的进程问题时，会导致整个功能不可用
+ 对于多进程的系统，如果遇到机器故障或者OS问题，那么功能也会整体不可用，而如果是多进程中的某个进程问题，那么有可能保持系统的部分功能正常--当然这取决与多进程系统自身的实现方式
+ 在多机系统中，如果遇到某些机器故障、OS故障或者某些机器的进程问题，我们都有机会来保证整体的功能大体可用--可能是部分功能失效，也可能是不再能承担正常情况下那么大的系统压力了

## 网络通信基础知识 

线程、进程的执行模式之后，就是和网络通信相关的知识。在分布式系统中，组件分布在网络上的多个节点中，通过消息的传递来通信并且进行动作的协调。因此通络通信在分布式系统中非常重要。

#### ISO的OSI网络模型 ：

![ISO的OSI模型](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/OSI.png?raw=true "ISO的OSI网络模型")

#### 网络IO的实现方式

实践中接触比较多的网络模型主要是以太网及TCP/IP协议栈，UDP在一些场景中也会用到。在我们使用Socket套接字进行网络通信开发时，我们会用的三种方式包括 ： BIO、NIO和AIO。

###### BIO方式

BIO 即 Blocking IO，采用阻塞的方式实现。 这种做法带来的主要问题是 ：使得一个线程只处理一个Socket，如果是Server端，那么在支持并发的连接时，就需要更多地线程来完成这个工作

BIO的工作方式如下所示 ：

![BIO的工作方式](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/BIO.png?raw=true "BIO的工作方式")

###### NIO方式

NIO 即 Nonblocking IO，是基于事件驱动思想，采用Reactor模式（如图所示）。这在Java的服务端实现中也是采用的比较多的一种方式。

> 相对于BIO，NIO的一个明显好处是不需要为每个Socket套接字分配一个线程，而可以在一个线程中处理多个Socket套接字相关的工作。

![Reactor模式](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/Reactor.png?raw=true "Reactor模式")

Reactor模式的工作方式 ：
> Reactor 会管理所有的Handler，并且把出现的事件交给相应的Handler去处理

具体如下图 ：Reactor-Applied-in-Communication.png

![Reactor模式](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/Reactor-Applied-in-Communication.png?raw=true "Reactor模式在通信中的应用")

如图 ， 在NIO的工作方式下 不是用单个线程去应对单个Socket套接字，而是统一通过Reactor对所有的客户端Socket套接字的事件做处理，然后派发到不同的线程中。这就解决了BIO中为支撑更多Socket套接字需要打开更多线程的问题

###### AIO方式

AIO 即 Asynchronous IO ，就是异步IO。 AIO 采用Proactor模式。如下图所示 ：

![Proactor模式](https://github.com/FrankBian/Attachments/blob/master/ReadingNote/DistributionSystem/Proactor.png?raw=true "Reactor模式")

AIO 与 NIO的差别是， AIO在进行读/写操作时，只需要调用相应的read/write方法。并且需要传入CompletionHandler（动作完成的处理器）；在动作完成后，会调用CompletionHandler，当然，在不同的系统上会用细微的差异，不同的语言在SDK上也会有差异，但总体就是这样的工作方式。
NIO的通知是发生在动作之前，是在可写、可读的时候，Selector发现这些事件之后调用Handler处理。

###### Summary

BIO、NIO和AIO这几种模型并不要求客户端和服务端采用同样的方式。客户端和服务端之间的交互主要在于数据格式或者说是通信协议。在客户端，如果同时连接数不多，采用BIO也是一个很好的选择。









