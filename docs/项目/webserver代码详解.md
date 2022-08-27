# 🔥webserver代码详解

# 一、线程池

## 1、基础知识

[最新版Web服务器项目详解 - 01 线程同步机制封装类](https://mp.weixin.qq.com/s?__biz=MzAxNzU2MzcwMw==&mid=2649274278&idx=3&sn=5840ff698e3f963c7855d702e842ec47&chksm=83ffbefeb48837e86fed9754986bca6db364a6fe2e2923549a378e8e5dec6e3cf732cdb198e2&scene=0&xtrack=1#rd)

### RAII

RAII全称是“Resource Acquisition is Initialization”，直译过来是“资源获取即初始化”。

在构造函数中申请分配资源，在析构函数中释放资源。因为C++的语言机制保证了，当一个对象创建的时候，自动调用构造函数，当对象超出作用域的时候会自动调用析构函数。所以，在RAII的指导下，我们应该使用**类**来管理资源，将资源和对象的生命周期绑定。

RAII的核心思想是**将资源或者状态与对象的生命周期绑定**，通过C++的语言机制，实现资源和状态的安全管理，**智能指针**是RAII最好的例子。

### 信号量

信号量是一种特殊的变量，它只能取自然数值并且只支持两种操作：等待(P)和信号(V)。假设有信号量SV，对其的P、V操作如下：

 - P，如果SV的值大于0，则将其减一；若SV的值为0，则挂起执行
 - V，如果有其他进行因为等待SV而挂起，则唤醒；若没有，则将SV值加一

信号量的取值可以是任何自然数，最常用的，最简单的信号量是二进制信号量，只有0和1两个值.

 - sem_init函数用于初始化一个未命名的信号量
 - sem_destory函数用于销毁信号量
 - sem_wait函数将以原子操作方式将信号量减一，信号量为0时，sem_wait阻塞
 - sem_post函数以原子操作方式将信号量加一，信号量大于0时，唤醒调用sem_post的线程

以上，成功返回0，失败返回errno

### 互斥量

互斥锁，也成互斥量，可以保护关键代码段，以确保独占式访问。当进入关键代码段，获得互斥锁将其加锁；离开关键代码段，唤醒等待该互斥锁的线程。

 - pthread_mutex_init函数用于初始化互斥锁
 - pthread_mutex_destory函数用于销毁互斥锁
 - pthread_mutex_lock函数以原子操作方式给互斥锁加锁
 - pthread_mutex_unlock函数以原子操作方式给互斥锁解锁

以上，成功返回0，失败返回errno

### 条件变量

条件变量提供了一种线程间的通知机制，当某个共享数据达到某个值时，唤醒等待这个共享数据的线程。

 - pthread_cond_init函数用于初始化条件变量
- pthread_cond_destory函数销毁条件变量
 - pthread_cond_broadcast函数以广播的方式唤醒**所有**等待目标条件变量的线程
 - pthread_cond_wait函数用于等待目标条件变量。该函数调用时需要传入 **mutex参数(加锁的互斥锁)**，函数执行时，先把调用线程放入条件变量的请求队列，然后将互斥锁mutex解锁，当函数成功返回为0时，互斥锁会再次被锁上， **也就是说函数内部会有一次解锁和加锁操作**。

[最新版Web服务器项目详解 - 02 半同步半反应堆线程池（上）](https://mp.weixin.qq.com/s?__biz=MzAxNzU2MzcwMw==&mid=2649274278&idx=4&sn=caa323faf0c51d882453c0e0c6a62282&chksm=83ffbefeb48837e841a6dbff292217475d9075e91cbe14042ad6e55b87437dcd01e6d9219e7d&scene=0&xtrack=1#rd)

### 五种I/O模型

- **阻塞IO**: 调用者调用了某个函数，等待这个函数返回，期间什么也不做，不停的去检查这个函数有没有返回，必须等这个函数返回才能进行下一步动作；
- **非阻塞IO**: 非阻塞等待，每隔一段时间就去检测IO事件是否就绪，没有就绪就可以做其他事。非阻塞I/O执行系统调用总是立即返回，不管事件是否已经发生。若事件没有发生，则返回-1，此时可以根据errno区分这两种情况，对于accept，recv和send，事件未发生时，errno通常被设置成eagain；
- **信号驱动IO**: linux用套接口进行信号驱动IO，安装一个信号处理函数，进程继续运行并不阻塞，当IO事件就绪，进程收到SIGIO信号，然后处理IO事件；
- **IO复用**: linux用select/poll函数实现IO复用模型，这两个函数也会使进程阻塞，但是和阻塞IO所不同的是这两个函数可以同时阻塞多个IO操作，而且可以同时对多个读操作、写操作的IO函数进行检测。知道有数据可读或可写时，才真正调用IO操作函数；
- **异步IO**: linux中，可以调用aio_read函数告诉内核描述字缓冲区指针和缓冲区的大小、文件偏移及通知的方式，然后立即返回，当内核将数据拷贝到缓冲区后，再通知应用程序。

**注意**：**阻塞I/O，非阻塞I/O，信号驱动I/O和I/O复用都是同步I/O**。同步I/O指内核向应用程序通知的是就绪事件，比如只通知有客户端连接，要求用户代码自行执行I/O操作；异步I/O是指内核向应用程序通知的是完成事件，比如读取客户端的数据后才通知应用程序，由内核完成I/O操作。

### 并发编程模式

并发编程方法的实现有多线程和多进程两种，但这里涉及的并发模式指I/O处理单元与逻辑单元的协同完成任务的方法，分为：

- 半同步/半异步模式
- 领导者/追随者模式

### 半同步/半反应堆

半同步/半反应堆并发模式是半同步/半异步的变体，将半异步具体化为某种事件处理模式。

并发模式中的同步和异步：

 - 同步指的是程序完全按照代码序列的顺序执行
 - 异步指的是程序的执行需要由系统事件驱动

**半同步/半异步模式**工作流程：

 - 同步线程用于处理客户逻辑
 - 异步线程用于处理I/O事件
 - 异步线程监听到客户请求后，就将其封装成请求对象并插入请求队列中
 - 请求队列将通知某个工作在**同步模式的工作线程**来读取并处理该请求对象

**半同步/半反应堆**工作流程（以Proactor模式为例）：

 - 主线程充当**异步线程**，负责监听所有socket上的事件；
 - 若有新请求到来，主线程接收请求得到新的连接socket，然后往epoll内核事件表中注册该socket上的读写事件；
 - 如果连接socket上有读写事件发生，**主线程从socket上接收数据**，并将数据封装成请求对象插入到请求队列中；
 - 所有工作线程睡眠在请求队列上，当有任务到来时，通过竞争（如互斥锁）获得任务的接管权。

### 线程池

[大话操作系统（18）socket ⽹络模型_海岸星的清风的博客-CSDN博客](https://blog.csdn.net/weixin_42461320/article/details/123607783?ops_request_misc=%7B%22request%5Fid%22%3A%22164912298716780261951860%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fblog.%22%7D&request_id=164912298716780261951860&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-123607783.nonecase&utm_term=线程池&spm=1018.2226.3001.4450)

使⽤线程池的⽅式来避免线程的频繁创建和销毁，所谓的线程池，就是提前创建若⼲个线程，这样当由新连接建⽴时，将这个已连接的 Socket 放⼊到⼀个队列⾥，然后线程池⾥的线程负责从队列中取出已连接 Socket 进程处理。

需要注意的是，这个队列是全局的，每个线程都会操作，为了避免多线程竞争，线程在操作这个队列前要加锁。

- 空间换时间，浪费服务器的硬件资源，换取运行效率；
- 池是一组资源的集合，这组资源在服务器启动之初就被完全创建好并初始化，这称为静态资源；
- 当服务器进入正式运行阶段，开始处理客户请求的时候，如果它需要相关的资源，可以直接从池中获取，无需动态分配；
- 当服务器处理完一个客户连接后，可以把相关的资源放回池中，无需执行系统调用释放资源。

<img src="https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207021718823.png" alt="image-20220702171826767" style="zoom:80%;" />

[最新版Web服务器项目详解 - 03 半同步半反应堆线程池（下） (qq.com)](https://mp.weixin.qq.com/s/PB8vMwi8sB4Jw3WzAKpWOQ)

## 2、 线程池代码

线程池的设计模式为半同步/半反应堆，其中反应堆具体为Proactor事件处理模式。

具体的，主线程为异步线程，负责监听文件描述符，接收socket新连接。若当前监听的socket发生了读写事件，则将任务插入到请求队列。工作线程从请求队列中取出任务，完成读写数据的处理。

### 线程池类定义

线程池采用模板编程，这是为了增强其拓展性：各种任务种类都可支持。

具体定义可以看代码。需要注意，线程处理函数和运行函数设置为私有属性。

```php
template <typename T>
class threadpool
{
public:
    //thread_number是线程池中线程的数量
    //max_requests是请求队列中最多允许的、等待处理的请求的数量
    //connPool是数据库连接池指针
    threadpool(int actor_model, connection_pool *connPool, 
               int thread_number = 8, int max_request = 10000);
    ~threadpool();
    bool append(T *request, int state); //reactor模式下的请求入队
    bool append_p(T *request);          //proactor模式下的请求入队

private:
    /*工作线程运行的函数，它不断从工作队列中取出任务并执行*/
    static void *worker(void *arg);//为什么要用静态成员函数呢-----class specific
    void run();

private:
    int m_thread_number;        //线程池中的线程数
    int m_max_requests;         //请求队列中允许的最大请求数
    pthread_t *m_threads;       //描述线程池的数组，其大小为m_thread_number
    std::list<T *> m_workqueue; //请求队列
    locker m_queuelocker;       //保护请求队列的互斥锁
    sem m_queuestat;            //是否有任务需要处理
    connection_pool *m_connPool;  //数据库
    int m_actor_model;          //模型切换（这个切换是指Reactor/Proactor）
};
```

### 线程池创建与回收

构造函数中创建线程池，pthread_create函数中将类的对象作为参数传递给静态函数(worker)，在静态函数中引用这个对象，并调用其动态方法(run)。

具体的，类对象传递时用this指针，传递给静态函数后，将其转换为线程池类，并调用私有成员函数run。

**pthread_create函数**：

```php
#include <pthread.h>
//返回新生成的线程的id
int pthread_create(pthread_t *thread_tid,//新生成的线程的id         
				const pthread_attr_t *attr, //指向线程属性的指针,通常设置为NULL
				void * (*start_routine) (void *), //处理线程函数的地址
				void *arg);  //start_routine()中的参数

```

函数原型中的第三个参数，**为函数指针，指向处理线程函数的地址**，该函数**要求为静态函数**。如果处理线程函数为类成员函数时，需要将其设置为**静态成员函数，因为类的非静态成员函数有this指针，就跟void\*不匹配**。而静态成员函数就没有这个问题，它没有this指针。

```php
template <typename T>
//线程池构造函数
threadpool<T>::threadpool( int actor_model, connection_pool *connPool, int thread_number, int max_requests)
    : m_actor_model(actor_model),m_thread_number(thread_number), m_max_requests(max_requests), 
	m_threads(NULL),m_connPool(connPool)
{
    if (thread_number <= 0 || max_requests <= 0)
        throw std::exception();
    m_threads = new pthread_t[m_thread_number];     //pthread_t是长整型
    if (!m_threads)
        throw std::exception();
    for (int i = 0; i < thread_number; ++i)
    {
        //函数原型中的第三个参数，为函数指针，指向处理线程函数的地址。
        //若线程函数为类成员函数，则this指针会作为默认的参数被传进函数中，从而和线程函数参数(void*)不能匹配，不能通过编译。
        //静态成员函数就没有这个问题，因为里面没有this指针。
        //循环创建线程，并将工作线程按要求进行运行
        if (pthread_create(m_threads + i, NULL, worker, this) != 0)
        {
            delete[] m_threads;
            throw std::exception();
        }
        //将线程进行分离后，不用单独对工作线程进行回收
        //主要是将线程属性更改为unjoinable，使得主线程分离，便于资源的释放
        if (pthread_detach(m_threads[i]))//为啥要调用pthread_detach?
        {
            delete[] m_threads;
            throw std::exception();
        }
    }
}
template <typename T>
threadpool<T>::~threadpool()
{
    delete[] m_threads;
}
```

创建一个线程之后需要调用pthread_detech()，原因在于： linux线程有两种状态**joinable状态和unjoinable状态。**

如果线程是joinable状态，当线程函数自己退出**都不会释放线程所占用堆栈和线程描述符（总计8K多）**。只有当调用了pthread_join，**主线程阻塞等待子线程结束**，然后回收子线程资源。

而unjoinable属性可以在pthread_create时指定，或在线程创建后调用pthread_detach，将**主线程与子线程分离**，子线程结束后，资源自动回收。

### 向请求队列中添加任务

通过**list**容器创建请求队列，向队列中添加时，通过**互斥锁**保证线程安全，添加完成后通过**信号量**提醒有任务要处理，最后注意线程同步。

```php
template <typename T>
//reactor模式下的请求入队
bool threadpool<T>::append(T *request, int state)
{
    m_queuelocker.lock();
    if (m_workqueue.size() >= m_max_requests)
    {
        m_queuelocker.unlock();
        return false;
    }
    //读写事件
    request->m_state = state; //IO 事件类别:读为0, 写为1
    m_workqueue.push_back(request);
    m_queuelocker.unlock();
    m_queuestat.post();
    return true;
}

template <typename T>
//proactor模式下的请求入队
bool threadpool<T>::append_p(T *request)
{
    m_queuelocker.lock();
    if (m_workqueue.size() >= m_max_requests)
    {
        m_queuelocker.unlock();
        return false;
    }
    m_workqueue.push_back(request);
    m_queuelocker.unlock();
    m_queuestat.post();
    return true;
}
```

### 线程处理函数

内部访问私有成员函数run，完成线程处理要求。

```php
//工作线程:pthread_create时就调用了它
template <typename T>
void *threadpool<T>::worker(void *arg)
{
    //调用时 *arg是this！
    //所以该操作其实是获取threadpool对象地址
    threadpool *pool = (threadpool *)arg;
    //线程池中每一个线程创建时都会调用run()，睡眠在队列中
    pool->run();
    return pool;
}
```

### run执行任务

主要实现，工作线程从请求队列中取出某个任务进行处理，注意线程同步。

run()函数可以看做**是一个回环事件**，一直等待m_queuestat()信号变量post（在append()中），即新任务进入请求队列，这时从请求队列中取出一个任务进行处理。

```php
//线程池中的所有线程都睡眠，等待请求队列中新增任务
template <typename T>
void threadpool<T>::run()
{
    while (true)
    {
        //信号量等待
        m_queuestat.wait();
        //被唤醒后先加互斥锁
        m_queuelocker.lock();
        if (m_workqueue.empty())
        {
            m_queuelocker.unlock();
            continue;
        }
        //从请求队列中取出第一个任务
		//将任务从请求队列删除
        T *request = m_workqueue.front();
        m_workqueue.pop_front();
        m_queuelocker.unlock();
        if (!request)
            continue;
        //Reactor
        if (1 == m_actor_model)
        {
            //IO事件类型：0为读
            if (0 == request->m_state)
            {
                if (request->read_once())
                {
                    request->improv = 1;
                    connectionRAII mysqlcon(&request->mysql, m_connPool);
                    request->process();
                }
                else
                {
                    request->improv = 1;
                    request->timer_flag = 1;
                }
            }
            else
            {
                if (request->write())
                {
                    request->improv = 1;
                }
                else
                {
                    request->improv = 1;
                    request->timer_flag = 1;
                }
            }
        }
        //default:Proactor，线程池不需要进行数据读取，而是直接开始业务处理
        //之前的操作已经将数据读取到http的read和write的buffer中了
        else
        {
            connectionRAII mysqlcon(&request->mysql, m_connPool);
            request->process();
        }
    }
}
```

# 二、HTTP处理流程

HTTP的处理流程分为以下三个步骤：

- **连接处理：**浏览器端发出http连接请求，主线程创建http对象接收请求并将所有数据读入对应buffer，将该对象插入任务队列，等待工作线程从任务队列中取出一个任务进行处理。
- **处理报文请求**：工作线程取出任务后，调用进程处理函数process_read()，通过主、从状态机对请求报文进行解析。
- **返回响应报文：**解析完之后，调用do_request()函数生成响应报文，通过process_write写入buffer，返回给浏览器端。

## 1、连接处理

[最新版Web服务器项目详解 - 04 http连接处理（上）](https://mp.weixin.qq.com/s/BfnNl-3jc_x5WPrWEJGdzQ)

在连接阶段，最重要的是**tcp连接过程和读取http的请求报文**（其实读取请求报文就是读取客户端发送的数据而已），tcp连接过程涉及epoll内核事件创建等。

<img src="https://test1.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202204072021336.png" alt="image-20220407202158284" style="zoom:80%;" />

服务器是如何实现读取http的报文的呢？首先，服务器需要对每一个**已建立连接http建立一个http的类对象**，这部分代码如下（服务器一直在运行`eventloop`即回环事件，因为整个服务器其实是事件驱动的）：

```php
//事件回环（即服务器主线程）
void WebServer::eventLoop()
{
    bool timeout = false;
    bool stop_server = false;

    while (!stop_server)
    {
        //等待所监控文件描述符上有事件的产生
        int number = epoll_wait(m_epollfd, events, MAX_EVENT_NUMBER, -1);
        //EINTR错误的产生：当阻塞于某个慢系统调用的一个进程捕获某个信号
        //且相应信号处理函数返回时，该系统调用可能返回一个EINTR错误
        if (number < 0 && errno != EINTR)
        {
            LOG_ERROR("%s", "epoll failure");
            break;
        }
        //对所有就绪事件进行处理
        for (int i = 0; i < number; i++)
        {
            int sockfd = events[i].data.fd;
            //处理新到的客户连接
            if (sockfd == m_listenfd)
            {
                bool flag = dealclinetdata();
                if (false == flag)
                    continue;
            }
            //处理异常事件
            else if (events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR))
            {
                //服务器端关闭连接，移除对应的定时器
                util_timer *timer = users_timer[sockfd].timer;
                deal_timer(timer, sockfd);
            }
            //处理定时器信号
            else if ((sockfd == m_pipefd[0]) && (events[i].events & EPOLLIN))
            {
                //接收到SIGALRM信号，timeout设置为True
                bool flag = dealwithsignal(timeout, stop_server);
                if (false == flag)
                    LOG_ERROR("%s", "dealclientdata failure");
            }
            //处理客户连接上接收到的数据
            else if (events[i].events & EPOLLIN)
            {
                dealwithread(sockfd);
            }
            //处理客户连接上send的数据
            else if (events[i].events & EPOLLOUT)
            {
                dealwithwrite(sockfd);
            }
        }

        //处理定时器为非必须事件，收到信号并不是立马处理
        //完成读写事件后，再进行处理
        if (timeout)
        {
            utils.timer_handler();
            LOG_INFO("%s", "timer tick");
            timeout = false;
        }
    }
```



```php
//处理客户连接上接收到的数据
void WebServer::dealwithread(int sockfd)
{
    //创建定时器临时变量，将该连接对应的定时器取出来
    util_timer *timer = users_timer[sockfd].timer;

    //reactor
    if (1 == m_actormodel)
    {
        if (timer)
        {
            //将定时器往后延迟3个单位
            adjust_timer(timer);
        }

        //若监测到读事件，将该事件放入请求队列
        m_pool->append(users + sockfd, 0);
        while (true)
        {
            //是否正在处理中
            if (1 == users[sockfd].improv)
            {
                //事件类型关闭连接
                if (1 == users[sockfd].timer_flag)
                {
                    deal_timer(timer, sockfd);
                    users[sockfd].timer_flag = 0;
                }
                users[sockfd].improv = 0;
                break;
            }
        }
    }
    //proactor
    else
    {   
        //先读取数据，再放进请求队列
        if (users[sockfd].read_once())
        {
            LOG_INFO("deal with the client(%s)", inet_ntoa(users[sockfd].get_address()->sin_addr));
            //将该事件放入请求队列
            m_pool->append_p(users + sockfd);
            if (timer)
            {
                adjust_timer(timer);
            }
        }
        else
        {
            deal_timer(timer, sockfd);
        }
    }
}

//写操作
void WebServer::dealwithwrite(int sockfd)
{
    util_timer *timer = users_timer[sockfd].timer;
    //reactor
    if (1 == m_actormodel)
    {
        if (timer)
        {
            adjust_timer(timer);
        }

        m_pool->append(users + sockfd, 1);

        while (true)
        {
            if (1 == users[sockfd].improv)
            {
                if (1 == users[sockfd].timer_flag)
                {
                    deal_timer(timer, sockfd);
                    users[sockfd].timer_flag = 0;
                }
                users[sockfd].improv = 0;
                break;
            }
        }
    }
    else
    {
        //proactor
        if (users[sockfd].write())
        {
            LOG_INFO("send data to the client(%s)", inet_ntoa(users[sockfd].get_address()->sin_addr));

            if (timer)
            {
                adjust_timer(timer);
            }
        }
        else
        {
            deal_timer(timer, sockfd);
        }
    }
}
```

## 2、处理报文请求

[最新版Web服务器项目详解 - 05 http连接处理（中）](https://mp.weixin.qq.com/s/wAQHU-QZiRt1VACMZZjNlw)

**从状态机负责读取报文的一行，主状态机负责对该行数据进行解析**，主状态机内部调用从状态机，从状态机驱动主状态机。

![image-20220403105959011](https://test1.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202204031059253.png)

**主状态机**

三种状态，标识解析位置。

- CHECK_STATE_REQUESTLINE，解析请求行
- CHECK_STATE_HEADER，解析请求头
- CHECK_STATE_CONTENT，解析消息体，仅用于解析POST请求

**从状态机**

三种状态，标识解析一行的读取状态。

- LINE_OK，完整读取一行
- LINE_BAD，报文语法有误
- LINE_OPEN，读取的行不完整

### http报文解析

各子线程通过 `process` 函数对任务进行处理，调用 `process_read` 函数和 `process_write` 函数分别完成报文解析与报文响应两个任务。

```php
//处理http报文请求与报文响应
//根据read/write的buffer进行报文的解析和响应
void http_conn::process()
{
    //NO_REQUEST，表示请求不完整，需要继续接收请求数据
    HTTP_CODE read_ret = process_read();
    if (read_ret == NO_REQUEST)
    {
        //注册并监听读事件
        modfd(m_epollfd, m_sockfd, EPOLLIN, m_TRIGMode);
        return;
    }
    //调用process_write完成报文响应
    bool write_ret = process_write(read_ret);
    if (!write_ret)
    {
        close_conn();
    }
    //注册并监听写事件
    modfd(m_epollfd, m_sockfd, EPOLLOUT, m_TRIGMode);
}
```

**HTTP_CODE含义**

表示HTTP请求的处理结果，在头文件中初始化了八种情形，在报文解析时只涉及到四种。

- NO_REQUEST

  - 请求不完整，需要继续读取请求报文数据

- GET_REQUEST

  - 获得了完整的HTTP请求

- BAD_REQUEST

  - HTTP请求报文有语法错误

- INTERNAL_ERROR

  - 服务器内部错误，该结果在主状态机逻辑switch的default下，一般不会触发

### 报文解析流程

**process_read** 通过while循环，将主从状态机进行封装，对报文的每一行进行循环处理。这里的**主状态机**，指的是process_read()函数，**从状态机**是指parse_line()函数。

代码使用 `switch...case` 来体现**主状态机的选择**，而**主状态机的状态**是由`CHECK_STATE_REQUESTLINE/HEADER/CONTENT`这三个标志来表示的：**正在解析请求行、解析请求头、解析消息体（body）**。

- 判断条件

  - 主状态机转移到CHECK_STATE_CONTENT，该条件涉及解析消息体
  - 从状态机转移到LINE_OK，该条件涉及解析请求行和请求头部
  - 两者为或关系，当条件为真则继续循环，否则退出

- 循环体

  - 从状态机读取数据
  - 调用get_line函数，通过m_start_line将从状态机读取数据间接赋给text
  - 主状态机解析text

```php
 //m_start_line是行在buffer中的起始位置，将该位置后面的数据赋给text
 //此时从状态机已提前将一行的末尾字符\r\n变为\0\0，所以text可以直接取出完整的行进行解析
 
 //m_start_line是已经解析的字符
 //get_line用于将指针向后偏移，指向未处理的字符
 char* get_line(){
     return m_read_buf + m_start_line;
 }

//有限状态机处理请求报文
http_conn::HTTP_CODE http_conn::process_read()
{
	//初始化从状态机状态、HTTP请求解析结果
    LINE_STATUS line_status = LINE_OK;
    HTTP_CODE ret = NO_REQUEST;
    char *text = 0;
	
	//parse_line为从状态机的具体实现
    while ((m_check_state == CHECK_STATE_CONTENT && line_status == LINE_OK) ||
    		((line_status = parse_line()) == LINE_OK))
    {
        text = get_line();
        //m_start_line是每一个数据行在m_read_buf中的起始位置
        //m_checked_idx表示从状态机在m_read_buf中读取的位置
        m_start_line = m_checked_idx;
        LOG_INFO("%s", text);
        //主状态机的三种状态转移逻辑
        switch (m_check_state)
        {
        case CHECK_STATE_REQUESTLINE:
        {
        	//解析请求行
            ret = parse_request_line(text);
            if (ret == BAD_REQUEST)
                return BAD_REQUEST;
            break;
        }
        case CHECK_STATE_HEADER:
        {
        	//解析请求头
            ret = parse_headers(text);
            if (ret == BAD_REQUEST)
                return BAD_REQUEST;
            //完整解析GET请求后，跳转到报文响应函数
            else if (ret == GET_REQUEST)
            {
                return do_request();
            }
            break;
        }
        case CHECK_STATE_CONTENT:
        {
        	//解析消息体
            ret = parse_content(text);
            //完整解析POST请求后，跳转到报文响应函数
            if (ret == GET_REQUEST)
                return do_request();
            //解析完消息体即完成报文解析，避免再次进入循环，更新line_status
            line_status = LINE_OPEN;
            break;
        }
        default:
            return INTERNAL_ERROR;
        }
    }
    return NO_REQUEST;
}
```

主状态机初始状态是CHECK_STATE_REQUESTLINE，而后调用parse_request_line()解析请求行，获得HTTP的请求方法、目标URL以及HTTP版本号，状态变为CHECK_STATE_HEADER。

此时进入循环体之后，调用parse_headers()解析请求头部信息。先要判断是空行还是请求头，空行进一步区分POST还是GET。若是请求头，则更新长短连接状态、host等等。

注：GET和POST请求报文的区别之一是有无消息体部分。

当使用POST请求时，需要进行CHECK_STATE_CONTENT的解析，取出POST消息体中的**信息（用户名、密码）**。

### 从状态机逻辑

在HTTP报文中，每一行的数据由`\r\n`作为结束字符，空行则是仅仅是字符`\r\n`。因此，可以通过查找`\r\n`将报文拆解成单独的行进行解析，项目中便是利用了这一点。

从状态机负责读取buffer中的数据，将每行数据末尾的`\r\n`置为`\0\0`，并更新从状态机在buffer中读取的位置`m_checked_idx`，以此来驱动主状态机解析。

- 从状态机从m_read_buf中逐字节读取，判断当前字节是否为\r
  - 接下来的字符是\n，将\r\n修改成\0\0，将m_checked_idx指向下一行的开头，则返回LINE_OK
  - 接下来达到了buffer末尾，表示buffer还需要继续接收，返回LINE_OPEN
  - 否则，表示语法错误，返回LINE_BAD

- 当前字节不是\r，判断是否是\n（**一般是上次读取到\r就到了buffer末尾，没有接收完整，再次接收时会出现这种情况**）
  - 如果前一个字符是\r，则将\r\n修改成\0\0，将m_checked_idx指向下一行的开头，则返回LINE_OK
  - 当前字节既不是\r，也不是\n，表示接收不完整，需要继续接收，返回LINE_OPEN


```php
//从状态机，用于分析出一行内容
//返回值为行的读取状态，有LINE_OK, LINE_BAD, LINE_OPEN

//m_read_idx指向缓冲区m_read_buf的数据末尾的下一个字节
//m_checked_idx指向从状态机当前正在分析的字节
http_conn::LINE_STATUS http_conn::parse_line()
{
    char temp;
    for (; m_checked_idx < m_read_idx; ++m_checked_idx)
    {
    	//temp为将要分析的字节
        temp = m_read_buf[m_checked_idx];
        //如果当前是\r字符，则有可能会读取到完整行
        if (temp == '\r')
        {
        	//下一个字符达到了buffer结尾，则接收不完整，需要继续接收
            if ((m_checked_idx + 1) == m_read_idx)
                return LINE_OPEN;
            //下一个字符是\n，将\r\n改为\0\0
            else if (m_read_buf[m_checked_idx + 1] == '\n')
            {
                m_read_buf[m_checked_idx++] = '\0';
                m_read_buf[m_checked_idx++] = '\0';
                return LINE_OK;
            }
            //如果都不符合，则返回语法错误
            return LINE_BAD;
        }
        //如果当前字符是\n，也有可能读取到完整行
		//一般是上次读取到\r就到buffer末尾了，没有接收完整，再次接收时会出现这种情况
        else if (temp == '\n')
        {
        	//前一个字符是\r，则接收完整
            if (m_checked_idx > 1 && m_read_buf[m_checked_idx - 1] == '\r')
            {
                m_read_buf[m_checked_idx - 1] = '\0';
                m_read_buf[m_checked_idx++] = '\0';
                return LINE_OK;
            }
            return LINE_BAD;
        }
    }
    //并没有找到\r\n，需要继续接收
    return LINE_OPEN;
}
```

### 主状态机逻辑

主状态机初始状态是`CHECK_STATE_REQUESTLINE`，通过调用从状态机来驱动主状态机，在主状态机进行解析前，从状态机已经将每一行的末尾`\r\n`符号改为`\0\0`，以便于主状态机直接取出对应字符串进行处理。

解析完请求行后，主状态机继续分析请求头。在报文中，请求头和空行的处理使用的同一个函数，这里通过判断当前的text首位是不是\0字符。若是，则表示当前处理的是空行；若不是，则表示当前处理的是请求头。

- **CHECK_STATE_REQUESTLINE**
  - 主状态机的初始状态，调用parse_request_line函数解析请求行
  - 解析函数从m_read_buf中解析HTTP请求行，获得请求方法、目标URL及HTTP版本号
  - 解析完成后主状态机的状态变为CHECK_STATE_HEADER


```php
//解析http请求行，获得请求方法，目标url及http版本号
http_conn::HTTP_CODE http_conn::parse_request_line(char *text)
{
    //在HTTP报文中，请求行用来说明请求类型
    //要访问的资源以及所使用的HTTP版本，其中各个部分之间通过\t或空格分隔。
    //请求行中最先含有空格和\t任一字符的位置并返回

    //strpbrk(string pointer break)：在源字符串（s1）中找出最先含有
    //搜索字符串（s2）中任一字符的位置并返回，若找不到则返回空指针
    m_url = strpbrk(text, " \t");
    //如果没有空格或\t，则报文格式有误
    if (!m_url)
    {
        return BAD_REQUEST;
    }
    //将该位置改为\0，用于将前面数据取出
    //(已读取的数据不再会匹配到\t）
    *m_url++ = '\0';

    //取出数据，并通过与GET和POST比较，以确定请求方式
    char *method = text;
    if (strcasecmp(method, "GET") == 0)
        m_method = GET;
    else if (strcasecmp(method, "POST") == 0)
    {
        m_method = POST;
        cgi = 1;
    }
    else
        return BAD_REQUEST;
    
    //m_url此时跳过了第一个空格或\t字符，但不知道之后是否还有
    //将m_url向后偏移，通过查找，继续跳过空格和\t字符，指向请求资源的第一个字符
    //strspn(const char *str, const char * accept)：计算字符串 str 中连续有几个字符都属于字符串 accept
    //其返回值是字符串str开头连续包含字符串accept内的字符数目 (str s)
    m_url += strspn(m_url, " \t");

    //相同逻辑，判断HTTP版本号
    m_version = strpbrk(m_url, " \t");
    if (!m_version)
        return BAD_REQUEST;
    *m_version++ = '\0';
    m_version += strspn(m_version, " \t");
    //仅支持HTTP/1.1
    if (strcasecmp(m_version, "HTTP/1.1") != 0)
        return BAD_REQUEST;
    
    //对请求资源前7个字符进行判断
    //这里主要是有些报文的请求资源中会带有http://，这里需要对这种情况进行单独处理
    //strchr：返回第一个匹配的地方
    if (strncasecmp(m_url, "http://", 7) == 0)
    {
        m_url += 7;
        m_url = strchr(m_url, '/');
    }

    //同样增加https情况
    if (strncasecmp(m_url, "https://", 8) == 0)
    {
        m_url += 8;
        m_url = strchr(m_url, '/');
    }

    //一般的不会带有上述两种符号，直接是单独的/或/后面带访问资源
    if (!m_url || m_url[0] != '/')
        return BAD_REQUEST;
    //当url为/时，显示欢迎界面
    if (strlen(m_url) == 1)
        strcat(m_url, "judge.html");
    
    //请求行处理完毕，将主状态机转移处理请求头
    m_check_state = CHECK_STATE_HEADER;
    return NO_REQUEST;
}
```

- **CHECK_STATE_HEADER**
- 调用parse_headers函数解析请求头部信息
  
- 判断是空行还是请求头，这里通过判断当前的text首位是不是\0字符。若是，则表示当前处理的是空行；若不是，则表示当前处理的是请求头。
  
  - 若是空行，进而判断content-length是否为0：如果不是0，表明是POST请求，则状态转移到CHECK_STATE_CONTENT；否则说明是GET请求，则报文解析结束。
  
  - 若解析的是请求头部字段，则主要分析connection字段，content-length字段，其他字段可以直接跳过，各位也可以根据需求继续分析。
  
- connection字段判断是keep-alive还是close，决定是长连接还是短连接
  
- content-length字段，这里用于读取post请求的消息体长度


```php
//解析http请求的一个头部信息
http_conn::HTTP_CODE http_conn::parse_headers(char *text)
{
    //判断是空行还是请求头
    //解析空行
    if (text[0] == '\0')
    {
        //判断是GET还是POST请求
        //!0 is POST
        if (m_content_length != 0)
        {
            //POST需要跳转到消息体处理状态
            m_check_state = CHECK_STATE_CONTENT;
            return NO_REQUEST;
        }
        //==0 is GET
        return GET_REQUEST;
    }
    //解析请求头部connection字段
    else if (strncasecmp(text, "Connection:", 11) == 0)
    {
        text += 11;

        //跳过空格和\t字符
        text += strspn(text, " \t");
        if (strcasecmp(text, "keep-alive") == 0)
        {
            //如果是长连接，则将linger标志设置为true
            m_linger = true;
        }
    }
    //解析请求头部Content-length字段
    else if (strncasecmp(text, "Content-length:", 15) == 0)
    {
        text += 15;
        text += strspn(text, " \t");
        m_content_length = atol(text);
    }
    //解析请求头部Host字段
    else if (strncasecmp(text, "Host:", 5) == 0)
    {
        text += 5;
        text += strspn(text, " \t");
        m_host = text;
    }
    else
    {
        LOG_INFO("oop!unknow header: %s", text);
    }
    return NO_REQUEST;
}
```

如果仅仅是GET请求，如项目中的欢迎界面，那么主状态机只设置之前的两个状态足矣。

GET和POST请求报文的区别之一是有无消息体部分，GET请求没有消息体，当解析完空行之后，便完成了报文的解析。但为了实现后续的**登录和注册功能**，为了避免将用户名和密码直接暴露在URL中，我们在项目中改用了**POST**请求，将用户名和密码添加在报文中作为消息体进行了封装。

为此，我们需要在解析报文的部分添加解析消息体的模块。

```php
while((m_check_state==CHECK_STATE_CONTENT && line_status==LINE_OK)||			((line_status=parse_line())==LINE_OK))
```

**这里的判断条件为什么要写成这样呢？**


在**GET**请求报文中，每一行都是`\r\n`作为结束，所以对报文进行拆解时，仅用从状态机的状态`line_status=parse_line())==LINE_OK`语句即可。

但在**POST**请求报文中，消息体的末尾没有任何字符，所以不能使用从状态机的状态，这里转而使用主状态机的状态作为循环入口条件。

**后面的&& line_status==LINE_OK又是为什么？**


解析完消息体后，报文的完整解析就完成了，但此时主状态机的状态还是CHECK_STATE_CONTENT，也就是说，符合循环入口条件，还会再次进入循环，这并不是我们所希望的。

为此，增加了该语句，并在完成消息体解析后，将line_status变量更改为LINE_OPEN，此时可以跳出循环，完成报文解析任务。

- **CHECK_STATE_CONTENT**
  - 仅用于解析POST请求，调用parse_content函数解析消息体
  - 用于保存post请求消息体，为后面的登录和注册做准备


```php
//判断http请求是否被完整读入
http_conn::HTTP_CODE http_conn::parse_content(char *text)
{
    //判断buffer中是否读取了消息体
    if (m_read_idx >= (m_content_length + m_checked_idx))
    {
        text[m_content_length] = '\0';
        //POST请求中最后为输入的用户名和密码
        m_string = text;
        return GET_REQUEST;
    }
    return NO_REQUEST;
}
```

## 3、返回响应报文

[最新版Web服务器项目详解 - 06 http连接处理（下）](https://mp.weixin.qq.com/s/451xNaSFHxcxfKlPBV3OCg)

浏览器端发出HTTP请求报文，服务器端接收该报文并调用`process_read`对其进行解析，根据解析结果`HTTP_CODE`，进入相应的逻辑和模块。其中，服务器子线程完成报文的解析与响应；主线程监测读写事件，调用`read_once`和`http_conn::write`完成数据的读取与发送。

![image-20220403170241000](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202204031702260.png)

在process_read()中完成请求报文的解析之后，状态机会调用do_request()函数，该函数是处理功能逻辑的，将网站根目录和url文件拼接，然后通过stat判断该文件属性。url可以抽象成 ip:port/xxx，xxx通过html文件的action属性（即请求报文）进行设置。m_url为请求报文中解析出的请求资源，以/开头，也就是x，项目中解析后的m_url有8种情况，见do_request()函数。

其中，**stat函数用于获取文件的类型、大小等信息**；mmap用于将文件等映射到内存，提高访问速度，详见[mmap原理](https://link.zhihu.com/?target=https%3A//blog.csdn.net/bbzhaohui/article/details/81665370)；iovec定义向量元素，通常这个结构用作一个多元素的数组，详见[社长微信](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/451xNaSFHxcxfKlPBV3OCg)；writev为聚集写，详见[链接](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/fanweisheng/p/11138704.html)；

执行do_request()函数之后，子线程调用process_write()进行响应报文（add_status_line、add_headers等函数）的生成。在生成响应报文的过程中主要调用add_reponse()函数更新m_write_idx和m_write_buf。

值得注意的是，响应报文分为两种，一种是**请求文件的存在**，通过io向量机制iovec，声明两个iovec，第一个指向m_write_buf，第二个指向mmap的地址m_file_address；另一种是**请求出错**，这时候只申请一个iovec，指向m_write_buf 。

其实往响应报文里写的就是服务器中html的文件数据，浏览器端对其进行解析、渲染并显示在浏览器页面上。另外，用户登录注册的验证逻辑代码在do_request()中，通过对Mysql数据库进行查询或插入，验证、添加用户。

```php
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags,int fd, off_t offset);
    - 功能：将一个文件或者设备的数据映射到内存中
    - 参数：
        - void *addr: NULL, 由内核指定
        - length : 要映射的数据的长度，这个值不能为0。建议使用文件的长度。
                获取文件的长度：stat lseek
        - prot : 对申请的内存映射区的操作权限
            -PROT_EXEC ：可执行的权限
            -PROT_READ ：读权限
            -PROT_WRITE ：写权限
            -PROT_NONE ：没有权限
            要操作映射内存，必须要有读的权限。
            PROT_READ、PROT_READ|PROT_WRITE
        - flags :
            - MAP_SHARED : 映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项
            - MAP_PRIVATE ：不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（copy on write）
        - fd: 需要映射的那个文件的文件描述符
            - 通过open得到，open的是一个磁盘文件
            - 注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。
                prot: PROT_READ                open:只读/读写 
                prot: PROT_READ | PROT_WRITE   open:读写
        - offset：偏移量，一般不用。必须指定的是4k的整数倍，0表示不偏移。
    - 返回值：
        成功返回创建的内存的首地址
        失败返回MAP_FAILED，(void *) -1

int munmap(void *addr, size_t length);
    - 功能：释放内存映射
    - 参数：
        - addr : 要释放的内存的首地址
        - length : 要释放的内存的大小，要和mmap函数中的length参数的值一样。
```

**iovec**

定义一个向量元素，通常这个结构用作一个多元素的数组。

```php
struct iovec {
    void      *iov_base;      /* starting address of buffer */
    size_t    iov_len;        /* size of buffer */
};
```

- iov_base指向数据的地址
- iov_len表示数据的长度

**writev**

writev函数用于在一次函数调用中写多个非连续缓冲区，又称为聚集写。

```php
#include <sys/uio.h>
ssize_t writev(int filedes, const struct iovec *iov, int iovcnt);
```

- filedes表示文件描述符
- iov为前述io向量机制结构体iovec
- iovcnt为结构体的个数

若成功则返回已写的字节数，若出错则返回-1。

`writev`以顺序`iov[0]`，`iov[1]`至`iov[iovcnt-1]`从缓冲区中聚集输出数据。`writev`返回输出的字节总数，通常它应等于所有缓冲区长度之和。

**特别注意：** 循环调用writev时，需要重新处理iovec中的指针和长度，该函数不会对这两个成员做任何处理。writev的返回值为已写的字节数，但这个返回值“实用性”并不高，因为参数传入的是iovec数组，计量单位是iovcnt，而不是字节数，我们仍然需要通过遍历iovec来计算新的基址，另外写入数据的“结束点”可能位于一个iovec的中间某个位置，因此需要调整临界iovec的io_base和io_len。

**HTTP_CODE含义**

表示HTTP请求的处理结果，在头文件中初始化了八种情形，在报文解析与响应中只用到了七种。

- NO_REQUEST

  - 请求不完整，需要继续读取请求报文数据

  - 跳转主线程继续监测读事件

- GET_REQUEST

  - 获得了完整的HTTP请求

  - 调用do_request完成请求资源映射

- NO_RESOURCE

  - 请求资源不存在

  - 跳转process_write完成响应报文

- BAD_REQUEST

  - HTTP请求报文有语法错误或请求资源为目录

  - 跳转process_write完成响应报文

- FORBIDDEN_REQUEST

  - 请求资源禁止访问，没有读取权限

  - 跳转process_write完成响应报文

- FILE_REQUEST

  - 请求资源可以正常访问

  - 跳转process_write完成响应报文

- INTERNAL_ERROR

  - 服务器内部错误，该结果在主状态机逻辑switch的default下，一般不会触发

### **do_request**

`process_read`函数的返回值是对请求的文件分析后的结果，一部分是语法错误导致的`BAD_REQUEST`，一部分是`do_request`的返回结果。该函数将网站根目录和`url`文件拼接，然后通过`stat`判断该文件属性。另外，为了提高访问速度，通过`mmap`进行映射，将普通文件映射到内存逻辑地址。

为了更好的理解请求资源的访问流程，这里对各种各页面跳转机制进行简要介绍。其中，浏览器网址栏中的字符，即`url`，可以将其抽象成`ip:port/xxx`，`xxx`通过`html`文件的`action`属性进行设置。

m_url为请求报文中解析出的请求资源，以/开头，也就是`/xxx`，项目中解析后的m_url有8种情况。

- /

  - GET请求，跳转到judge.html，即欢迎访问页面

- /0

  - POST请求，跳转到register.html，即注册页面

- /1

  - POST请求，跳转到log.html，即登录页面

- /2CGISQL.cgi

  - POST请求，进行登录校验

  - 验证成功跳转到welcome.html，即资源请求成功页面
  - 验证失败跳转到logError.html，即登录失败页面

- /3CGISQL.cgi

  - POST请求，进行注册校验

  - 注册成功跳转到log.html，即登录页面
  - 注册失败跳转到registerError.html，即注册失败页面

- /5

  - POST请求，跳转到picture.html，即图片请求页面

- /6

  - POST请求，跳转到video.html，即视频请求页面

- /7

  - POST请求，跳转到fans.html，即关注页面

```php
//网站根目录，文件夹内存放请求的资源和跳转的html文件
const char* doc_root = "/home/qgy/github/ini_tinywebserver/root";

//功能逻辑单元
http_conn::HTTP_CODE http_conn::do_request()
{
    //将初始化的m_real_file赋值为网站根目录
    strcpy(m_real_file, doc_root);
    int len = strlen(doc_root);
    
    //找到m_url中/的位置
    //strrchr() 函数查找字符串在另一个字符串中最后一次出现的位置，并返回从该位置到字符串结尾的所有字符
    const char *p = strrchr(m_url, '/');

    //实现登录和注册校验
    if (cgi == 1 && (*(p + 1) == '2' || *(p + 1) == '3'))
    {

        //根据标志判断是登录检测还是注册检测
        char flag = m_url[1];
        
        char *m_url_real = (char *)malloc(sizeof(char) * 200);
        strcpy(m_url_real, "/");
        strcat(m_url_real, m_url + 2); //将两个char类型连接
        strncpy(m_real_file + len, m_url_real, FILENAME_LEN - len - 1);
        free(m_url_real);

        //将用户名和密码提取出来
        //eg:user=123&passwd=123
        char name[100], password[100];
        int i;
        for (i = 5; m_string[i] != '&'; ++i)
            name[i - 5] = m_string[i];
        name[i - 5] = '\0';

        int j = 0;
        for (i = i + 10; m_string[i] != '\0'; ++i, ++j)
            password[j] = m_string[i];
        password[j] = '\0';

        if (*(p + 1) == '3')
        {
            //如果是注册，先检测数据库中是否有重名的
            //没有重名的，进行增加数据
            char *sql_insert = (char *)malloc(sizeof(char) * 200);
            strcpy(sql_insert, "INSERT INTO user(username, passwd) VALUES(");
            strcat(sql_insert, "'");
            strcat(sql_insert, name);
            strcat(sql_insert, "', '");
            strcat(sql_insert, password);
            strcat(sql_insert, "')");

            if (users.find(name) == users.end())
            {
                m_lock.lock();
                int res = mysql_query(mysql, sql_insert);
                users.insert(pair<string, string>(name, password));
                m_lock.unlock();

                if (!res)
                    strcpy(m_url, "/log.html");
                else
                    strcpy(m_url, "/registerError.html");
            }
            else
                strcpy(m_url, "/registerError.html");
        }
        //如果是登录，直接判断
        //若浏览器端输入的用户名和密码在表中可以查找到，返回1，否则返回0
        else if (*(p + 1) == '2')
        {
            if (users.find(name) != users.end() && users[name] == password)
                strcpy(m_url, "/welcome.html");
            else
                strcpy(m_url, "/logError.html");
        }
    }
    //如果请求资源为/0，表示跳转注册界面
    if (*(p + 1) == '0')
    {
        char *m_url_real = (char *)malloc(sizeof(char) * 200);
        strcpy(m_url_real, "/register.html");
        //将网站目录和/register.html进行拼接，更新到m_real_file中
        strncpy(m_real_file + len, m_url_real, strlen(m_url_real));

        free(m_url_real);
    }
    //如果请求资源为/1，表示跳转登录界面
    else if (*(p + 1) == '1')
    {
        char *m_url_real = (char *)malloc(sizeof(char) * 200);
        strcpy(m_url_real, "/log.html");
        strncpy(m_real_file + len, m_url_real, strlen(m_url_real));

        free(m_url_real);
    }
    //如果请求资源为/5，表示跳转pic
    else if (*(p + 1) == '5')
    {
        char *m_url_real = (char *)malloc(sizeof(char) * 200);
        strcpy(m_url_real, "/picture.html");
        strncpy(m_real_file + len, m_url_real, strlen(m_url_real));

        free(m_url_real);
    }
    //如果请求资源为/6，表示跳转video
    else if (*(p + 1) == '6')
    {
        char *m_url_real = (char *)malloc(sizeof(char) * 200);
        strcpy(m_url_real, "/video.html");
        strncpy(m_real_file + len, m_url_real, strlen(m_url_real));

        free(m_url_real);
    }
    //如果请求资源为/7，表示跳转weixin
    else if (*(p + 1) == '7')
    {
        char *m_url_real = (char *)malloc(sizeof(char) * 200);
        strcpy(m_url_real, "/fans.html");
        strncpy(m_real_file + len, m_url_real, strlen(m_url_real));
        free(m_url_real);
    }
    else
        //如果以上均不符合，即不是登录和注册，直接将url与网站目录拼接
        //这里的情况是welcome界面，请求服务器上的一个图片
        strncpy(m_real_file + len, m_url, FILENAME_LEN - len - 1);
    //通过stat获取请求资源文件信息，成功则将信息更新到m_file_stat结构体
    //失败返回NO_RESOURCE状态，表示资源不存在
    if (stat(m_real_file, &m_file_stat) < 0)
        return NO_RESOURCE;

    //判断文件的权限，是否可读，不可读则返回FORBIDDEN_REQUEST状态
    if (!(m_file_stat.st_mode & S_IROTH))
        return FORBIDDEN_REQUEST;

    //判断文件类型，如果是目录，则返回BAD_REQUEST，表示请求报文有误
    if (S_ISDIR(m_file_stat.st_mode))
        return BAD_REQUEST;

    //以只读方式获取文件描述符，通过mmap将该文件映射到内存中
    int fd = open(m_real_file, O_RDONLY);
    m_file_address = (char *)mmap(0, m_file_stat.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    //避免文件描述符的浪费和占用
    close(fd);

    //表示请求文件存在，且可以访问
    return FILE_REQUEST;
}
```

### **process_write**

**响应报文**

HTTP响应由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

```php
HTTP/1.1 200 OK
Date: Fri, 22 May 2009 06:07:21 GMT
Content-Type: text/html; charset=UTF-8
空行
<html>
      <head></head>
      <body>
            <!--body goes here-->
      </body>
</html>
```

根据`do_request`的返回状态，服务器子线程调用`process_write`向`m_write_buf`中写入响应报文。

- add_status_line函数，添加状态行：http/1.1 状态码 状态消息

- add_headers函数添加消息报头，内部调用add_content_length和add_linger函数

  - content-length记录响应报文长度，用于浏览器端判断服务器是否发送完数据

  - connection记录连接状态，用于告诉浏览器端保持长连接

- add_blank_line添加空行

上述涉及的5个函数，均是内部调用`add_response`函数更新`m_write_idx`指针和缓冲区`m_write_buf`中的内容。

```php
//添加响应报文的公共函数
bool http_conn::add_response(const char *format, ...)
{
    //如果写入内容超出m_write_buf大小则报错
    if (m_write_idx >= WRITE_BUFFER_SIZE)
        return false;
    //定义可变参数列表
    va_list arg_list;
    //将变量arg_list初始化为传入参数
    va_start(arg_list, format);
    //将数据format从可变参数列表写入写缓冲区，返回写入数据的长度
    int len = vsnprintf(m_write_buf + m_write_idx, WRITE_BUFFER_SIZE - 1 - m_write_idx, format, arg_list);
    //如果写入的数据长度超过缓冲区剩余空间，则报错
    if (len >= (WRITE_BUFFER_SIZE - 1 - m_write_idx))
    {
        va_end(arg_list);
        return false;
    }
    //更新m_write_idx位置
    m_write_idx += len;
    //清空可变参列表
    va_end(arg_list);

    LOG_INFO("request:%s", m_write_buf);

    return true;
}

//添加状态行
bool http_conn::add_status_line(int status, const char *title)
{
    return add_response("%s %d %s\r\n", "HTTP/1.1", status, title);
}

//添加消息报头，具体的添加文本长度、连接状态和空行
bool http_conn::add_headers(int content_len)
{
    return add_content_length(content_len) && add_linger() &&
           add_blank_line();
}

//添加Content-Length，表示响应报文的长度
bool http_conn::add_content_length(int content_len)
{
    return add_response("Content-Length:%d\r\n", content_len);
}

//添加文本类型，这里是html
bool http_conn::add_content_type()
{
    return add_response("Content-Type:%s\r\n", "text/html");
}

//添加连接状态，通知浏览器端是保持连接还是关闭
bool http_conn::add_linger()
{
    return add_response("Connection:%s\r\n", (m_linger == true) ? "keep-alive" : "close");
}

//添加空行
bool http_conn::add_blank_line()
{
    return add_response("%s", "\r\n");
}

//添加文本content
bool http_conn::add_content(const char *content)
{
    return add_response("%s", content);
}
```


响应报文分为两种，一种是**请求文件存在**，通过`io`向量机制`iovec`，声明两个`iovec`，第一个指向`m_write_buf`，第二个指向`mmap`的地址`m_file_address`；一种是**请求出错**，这时候只申请一个`iovec`，指向`m_write_buf`。

- iovec是一个结构体，里面有两个元素，指针成员iov_base指向一个缓冲区，这个缓冲区是存放的是writev将要发送的数据。
- 成员iov_len表示实际写入的长度

```php
//生成响应报文
bool http_conn::process_write(HTTP_CODE ret)
{
    switch (ret)
    {
        //内部错误，500
        case INTERNAL_ERROR:
        {
            //状态行
            add_status_line(500, error_500_title);
            //消息报头
            add_headers(strlen(error_500_form));
            if (!add_content(error_500_form))
                return false;
            break;
        }
        //报文语法有误，404
        case BAD_REQUEST:
        {
            add_status_line(404, error_404_title);
            add_headers(strlen(error_404_form));
            if (!add_content(error_404_form))
                return false;
            break;
        }
        //资源没有访问权限，403
        case FORBIDDEN_REQUEST:
        {
            add_status_line(403, error_403_title);
            add_headers(strlen(error_403_form));
            if (!add_content(error_403_form))
                return false;
            break;
        }
        //文件存在，200
        case FILE_REQUEST:
        {
            add_status_line(200, ok_200_title);
            //如果请求的资源存在
            if (m_file_stat.st_size != 0)
            {
                add_headers(m_file_stat.st_size);
                //第一个iovec指针指向响应报文缓冲区，长度指向m_write_idx
                m_iv[0].iov_base = m_write_buf;
                m_iv[0].iov_len = m_write_idx;
                //第二个iovec指针指向mmap返回的文件指针，长度指向文件大小
                m_iv[1].iov_base = m_file_address;
                m_iv[1].iov_len = m_file_stat.st_size;
                m_iv_count = 2;
                //发送的全部数据为响应报文头部信息和文件大小
                bytes_to_send = m_write_idx + m_file_stat.st_size;
                return true;
            }
            else
            {
                //如果请求的资源大小为0，则返回空白html文件
                const char *ok_string = "<html><body></body></html>";
                add_headers(strlen(ok_string));
                if (!add_content(ok_string))
                    return false;
            }
        }
        default:
            return false;
    }
    //除FILE_REQUEST状态外，其余状态只申请一个iovec，指向响应报文缓冲区
    m_iv[0].iov_base = m_write_buf;
    m_iv[0].iov_len = m_write_idx;
    m_iv_count = 1;
    bytes_to_send = m_write_idx;
    return true;
}
```

### **http_conn::write**

服务器子线程调用`process_write`完成响应报文，随后注册`epollout`事件。服务器主线程检测写事件，并调用`http_conn::write`函数将响应报文发送给浏览器端。

该函数具体逻辑如下：

在生成响应报文时初始化byte_to_send，包括头部信息和文件数据大小。通过writev函数循环发送响应报文数据，根据返回值更新byte_have_send和iovec结构体的指针和长度，并判断响应报文整体是否发送成功。

- 若writev单次发送成功，更新byte_to_send和byte_have_send的大小，若响应报文整体发送成功则取消mmap映射，并判断是否是长连接
  - 长连接重置http类实例，注册读事件，不关闭连接
  - 短连接直接关闭连接

- 若writev单次发送不成功，判断是否是写缓冲区满了
  - 若不是因为缓冲区满了而失败，取消mmap映射，关闭连接
  - 若eagain则满了，更新iovec结构体的指针和长度，并注册写事件，等待下一次写事件触发（当写缓冲区从不可写变为可写，触发epollout），因此在此期间无法立即接收到同一用户的下一请求，但可以保证连接的完整性。


```php
//往响应报文写入数据
bool http_conn::write()
{
    int temp = 0;
    //若要发送的数据长度为0
    //表示响应报文为空，一般不会出现这种情况
    if (bytes_to_send == 0)
    {
        modfd(m_epollfd, m_sockfd, EPOLLIN, m_TRIGMode);
        init();
        return true;
    }

    while (1)
    {
        //将响应报文的状态行、消息头、空行和响应正文发送给浏览器端
        temp = writev(m_sockfd, m_iv, m_iv_count);
        //error
        if (temp < 0)
        {
            //判断缓冲区是否满了
            if (errno == EAGAIN)
            {
                modfd(m_epollfd, m_sockfd, EPOLLOUT, m_TRIGMode);
                return true;
            }
            unmap();
            return false;
        }
        //更新已发送字节
        bytes_have_send += temp;
        //更新未发送字节
        bytes_to_send -= temp;
        //第一个iovec头部信息的数据已发送完，发送第二个iovec数据
        if (bytes_have_send >= m_iv[0].iov_len)
        {
            //不再继续发送头部信息
            m_iv[0].iov_len = 0;
            m_iv[1].iov_base = m_file_address + (bytes_have_send - m_write_idx);
            m_iv[1].iov_len = bytes_to_send;
        }
        //继续发送第一个iovec头部信息的数据
        else
        {
            m_iv[0].iov_base = m_write_buf + bytes_have_send;
            m_iv[0].iov_len = m_iv[0].iov_len - bytes_have_send;
        }

        //判断条件，数据已全部发送完
        if (bytes_to_send <= 0)
        {
            //如果发送失败，但不是缓冲区问题，取消映射
            unmap();
            //重新注册写事件
            modfd(m_epollfd, m_sockfd, EPOLLIN, m_TRIGMode);

            //浏览器的请求为长连接
            if (m_linger)
            {
                //重新初始化HTTP对象*
                init();
                return true;
            }
            else
            {
                return false;
            }
        }
    }
}
```

# 三、定时器

[最新版Web服务器项目详解 - 07 定时器处理非活动连接（上） (qq.com)](https://mp.weixin.qq.com/s/mmXLqh_NywhBXJvI45hchA)

由于非活跃连接占用了连接资源，严重影响服务器的性能，通过实现一个服务器定时器，处理这种非活跃连接，释放连接资源。利用`alarm`函数周期性地触发`SIGALRM`信号，该信号的信号处理函数利用管道通知主循环执行定时器链表上的定时任务。

**非活跃**是指客户端（这里是浏览器）与服务器端建立连接后，长时间不交换数据，一直占用服务器端的文件描述符，导致连接资源的浪费。

**定时事件**是指固定一段时间之后触发某段代码，由该段代码处理一个事件，如从内核事件表删除事件，并关闭文件描述符，释放连接资源。

**定时器**是指利用结构体或其他形式，将多种定时事件进行封装起来。这里只涉及一种定时事件，即**定期检测非活跃连接**，这里将该定时事件与连接资源封装为一个结构体定时器。

**定时器容器**是指使用某种容器类数据结构，将上述多个定时器组合起来，便于对定时事件统一管理。项目中使用**升序链表**将所有定时器串联组织起来。

本项目中，服务器主循环为每一个连接创建一个定时器，并对每个连接进行定时。另外，利用**升序时间链表容器**将所有定时器串联起来，若主循环接收到定时通知，则在链表中依次执行定时任务。

`Linux`下提供了三种定时的方法:

- socket选项SO_RECVTIMEO和SO_SNDTIMEO
- SIGALRM信号
- I/O复用系统调用的超时参数

三种方法没有一劳永逸的应用场景，也没有绝对的优劣。由于项目中使用的是`SIGALRM`信号，这里仅对其进行介绍，另外两种方法可以查阅游双的Linux高性能服务器编程 第11章 定时器。

利用`alarm`函数周期性地触发`SIGALRM`信号，信号处理函数利用**管道**通知主循环，主循环接收到该信号后对升序链表上所有定时器进行处理，若该段时间内没有交换数据，则将该连接关闭，释放所占用的资源。

<img src="https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207021719437.png" alt="image-20220702171952348" style="zoom:80%;" />

定时器处理非活动连接模块，主要分为两部分：其一为定时方法与信号通知流程，其二为定时器及其容器设计与定时任务的处理。

相关信号处理API见：[一文搞懂「信号」和「信号集」_海岸星的清风的博客-CSDN博客](https://blog.csdn.net/weixin_42461320/article/details/123443777)

![image-20220706111137372](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207061111967.png)

![image-20220706111227242](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207061112913.png)

![image-20220706111257860](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207061112816.png)

![image-20220706111314346](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207061113310.png)

**sigaction**

```php
#include <signal.h>
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
    - 功能：检查或者改变信号的处理，即信号捕捉
    - 参数：
        - signum : 需要捕捉的信号的编号或者宏值（信号的名称）
        - act ：捕捉到信号之后的处理动作
        - oldact : 上一次对信号捕捉相关的设置，一般不使用，传递NULL
    - 返回值：
        成功 0
        失败 -1

 struct sigaction {
    // 函数指针，指向的函数就是信号捕捉到之后的处理函数
    void     (*sa_handler)(int);
    // 不常用，同样是信号处理函数，有三个参数，可以获得关于信号更详细的信息
    void     (*sa_sigaction)(int, siginfo_t *, void *);
    // 临时阻塞信号集，在信号捕捉函数执行过程中，临时阻塞某些信号。
    sigset_t   sa_mask;
    // 使用哪一个信号处理对捕捉到的信号进行处理
    int sa_flags;
    这个值可以是0，表示使用 sa_handler，也可以是 SA_SIGINFO 表示使用 sa_sigaction
    SA_RESTART，使被信号打断的系统调用自动重新发起
	SA_NOCLDSTOP，使父进程在它的子进程暂停或继续运行时不会收到 SIGCHLD 信号
	SA_NOCLDWAIT，使父进程在它的子进程退出时不会收到 SIGCHLD 信号，这时子进程如果退出也不会成为僵尸进程
	SA_NODEFER，使对信号的屏蔽无效，即在信号处理函数执行期间仍能发出这个信号
	SA_RESETHAND，信号处理之后重新设置为默认的处理方式
	SA_SIGINFO，使用 sa_sigaction 成员而不是 sa_handler 作为信号处理函数
    // 被废弃掉了
    void     (*sa_restorer)(void);
};
```

**alarm**

```php
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
    - 功能：设置定时器（闹钟）。函数调用，开始倒计时，当倒计时为0的时候，函数会给当前的进程发送一个信号：SIGALARM
    - 参数：
        seconds: 倒计时的时长，单位：秒。如果参数为0，定时器无效（不进行倒计时，不发信号）。
                 取消一个定时器，通过 alarm(0)。
    - 返回值：
        - 之前没有定时器，返回0
        - 之前有定时器，返回之前的定时器剩余的时间

- SIGALARM ：默认终止当前的进程，每一个进程都有且只有唯一的一个定时器。
    alarm(10);  -> 返回0
    过了1秒
    alarm(5);   -> 返回9

alarm(100) -> 该函数是不阻塞的
```

**sigfillset**

```php
int sigfillset(sigset_t *set);
    - 功能：将信号集中的所有的标志位置为1
    - 参数：set：传出参数，需要操作的信号集
    - 返回值：成功返回0， 失败返回-1
```

## 1、信号处理

自定义信号处理函数，创建sigaction结构体变量，设置信号函数。

```php
//信号处理函数
void Utils::sig_handler(int sig)
{
    //为保证函数的可重入性，保留原来的errno
    //可重入性表示中断后再次进入该函数，环境变量与之前相同，不会丢失数据
    int save_errno = errno;
    int msg = sig;
    //将信号值从管道写端写入，传输字符类型，而非整型
    send(u_pipefd[1], (char *)&msg, 1, 0);
    //将原来的errno赋值为当前的errno
    errno = save_errno;
}
```

信号处理函数中仅仅通过管道发送信号值，不处理信号对应的逻辑，缩短异步执行时间，减少对主程序的影响。

```php
//设置信号函数
void Utils::addsig(int sig, void(handler)(int), bool restart)
{
    //创建sigaction结构体变量
    struct sigaction sa;
    memset(&sa, '\0', sizeof(sa));
    //信号处理函数中仅仅发送信号值，不做对应逻辑处理
    sa.sa_handler = handler;
    if (restart)
        sa.sa_flags |= SA_RESTART;
    //将所有信号添加到信号集中
    sigfillset(&sa.sa_mask);
    //执行sigaction函数
    assert(sigaction(sig, &sa, NULL) != -1);
}
```

项目中设置信号函数，仅关注`SIGTERM`和`SIGALRM`两个信号。

## 2、信号通知

- 创建管道，其中管道写端写入信号值，管道读端通过I/O复用系统监测读事件；

- 设置信号处理函数SIGALRM（时间到了触发）和SIGTERM（kill会触发，Ctrl+C）；

- - 通过struct sigaction结构体和sigaction函数注册信号捕捉函数
  - 在结构体的handler参数设置信号处理函数，从管道写端写入信号的名字

- 利用I/O复用系统监听管道读端文件描述符的可读事件；

- 信息值传递给主循环，主循环再根据接收到的信号值执行目标信号对应的逻辑代码；

```php
//定时处理任务，重新定时以不断触发SIGALRM信号
void Utils::timer_handler()
{
    m_timer_lst.tick();
    //最小的时间单位为5s
    alarm(m_TIMESLOT);
}

//创建定时器容器链表
static sort_timer_lst timer_lst;
//创建连接资源数组
client_data *users_timer = new client_data[MAX_FD];
//超时默认为False
bool timeout = false;
//alarm定时触发SIGALRM信号
alarm(TIMESLOT);

void WebServer::eventLoop()
{
    bool timeout = false;
    bool stop_server = false;

    while (!stop_server)
    {
        //等待所监控文件描述符上有事件的产生
        int number = epoll_wait(m_epollfd, events, MAX_EVENT_NUMBER, -1);
        //EINTR错误的产生：当阻塞于某个系统调用的一个进程捕获某个信号
        //且相应信号处理函数返回时，该系统调用可能返回一个EINTR错误
        if (number < 0 && errno != EINTR)
        {
            LOG_ERROR("%s", "epoll failure");
            break;
        }
        //对所有就绪事件进行处理
        for (int i = 0; i < number; i++)
        {
            int sockfd = events[i].data.fd;
            //处理新到的客户连接
            if (sockfd == m_listenfd)
            {
                bool flag = dealclinetdata();
                if (false == flag)
                    continue;
            }
            //处理异常事件
            else if (events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR))
            {
                //服务器端关闭连接，移除对应的定时器
                util_timer *timer = users_timer[sockfd].timer;
                deal_timer(timer, sockfd);
            }
            //处理定时器信号
            else if ((sockfd == m_pipefd[0]) && (events[i].events & EPOLLIN))
            {
                //接收到SIGALRM信号，timeout设置为True
                bool flag = dealwithsignal(timeout, stop_server);
                if (false == flag)
                    LOG_ERROR("%s", "dealclientdata failure");
            }
            //处理客户连接上接收到的数据
            else if (events[i].events & EPOLLIN)
            {
                dealwithread(sockfd);
            }
            //处理客户连接上send的数据
            else if (events[i].events & EPOLLOUT)
            {
                dealwithwrite(sockfd);
            }
        }

        //处理定时器为非必须事件，收到信号并不是立马处理
        //完成读写事件后，再进行处理
        if (timeout)
        {
            utils.timer_handler();
            LOG_INFO("%s", "timer tick");
            timeout = false;
        }
    }
```

```php
//处理定时器信号,set the timeout ture
bool WebServer::dealwithsignal(bool &timeout, bool &stop_server)
{
    int ret = 0;
    int sig;
    char signals[1024];
    //从管道读端读出信号值，成功返回字节数，失败返回-1
    //正常情况下，这里的ret返回值总是1，只有14和15两个ASCII码对应的字符
    ret = recv(m_pipefd[0], signals, sizeof(signals), 0);
    if (ret == -1)
    {
        // handle the error
        return false;
    }
    else if (ret == 0)
    {
        return false;
    }
    else
    {
        //处理信号值对应的逻辑
        for (int i = 0; i < ret; ++i)
        {
            //这里面明明是字符
            switch (signals[i])
            {
            //这里是整型
            case SIGALRM:
            {
                timeout = true;
                break;
            }
            //关闭服务器
            case SIGTERM:
            {
                stop_server = true;
                break;
            }
            }
        }
    }
    return true;
}
```

**为什么管道写端要非阻塞？**

send是将信息发送给套接字缓冲区，如果缓冲区满了则会阻塞，这时候会进一步增加信号处理函数的执行时间，为此将其修改为非阻塞。

**没有对非阻塞返回值处理，如果阻塞是不是意味着这一次定时事件失效了？**

是的，但定时事件是非必须立即处理的事件，可以允许这样的情况发生。

**管道传递的是什么类型？switch-case的变量冲突？**

信号本身是整型数值，管道中传递的是ASCII码表中整型数值对应的字符。

switch的变量一般为字符或整型，当switch的变量为字符时，case中可以是字符，也可以是字符对应的ASCII码。

## 3、定时器设计

**定时器设计**，将连接资源和定时事件等封装起来，具体包括连接资源、超时时间和回调函数，这里的回调函数指向定时事件。

**定时器容器设计**，将多个定时器串联组织起来统一处理，具体包括升序链表设计。

**定时任务处理函数**，该函数封装在容器类中，函数遍历升序链表容器，根据超时时间，处理对应的定时器。

项目中将连接资源、定时事件和超时时间封装为定时器类：

- 连接资源包括客户端套接字地址、文件描述符和定时器；
- 定时事件为回调函数，将其封装起来由用户自定义，这里是删除非活动socket上的注册事件，并关闭；
- 定时器超时时间 = 浏览器和服务器连接时刻 + 固定时间(TIMESLOT)，可以看出，定时器使用绝对时间作为超时值，这里alarm设置为5秒，连接超时为15秒。

```php
//连接资源结构体成员需要用到定时器类
//需要前向声明
class util_timer;

//连接资源
struct client_data
{
    //客户端socket地址
    sockaddr_in address;
    //socket文件描述符
    int sockfd;
    //定时器
    util_timer *timer;
};

//定时器类，用一个双向链表实现
class util_timer
{
public:
    util_timer() : prev(NULL), next(NULL) {}

public:
    //超时时间
    time_t expire;
    //回调函数：从内核事件表删除事件，关闭文件描述符，释放连接资源
    void (* cb_func)(client_data *);
    //连接资源
    client_data *user_data;
    //前向定时器
    util_timer *prev;
    //后继定时器
    util_timer *next;
};
```

定时事件：从内核事件表删除事件，关闭文件描述符，释放连接资源。

```php
class Utils;
//定时器回调函数(callback)：从内核事件表删除事件，关闭文件描述符，释放连接资源
void cb_func(client_data *user_data)
{
    //删除非活动连接在socket上的注册事件
    epoll_ctl(Utils::u_epollfd, EPOLL_CTL_DEL, user_data->sockfd, 0);
    assert(user_data);
    //关闭文件描述符
    close(user_data->sockfd);
    //减少连接数
    http_conn::m_user_count--;
}
```

## 4、定时器容器设计

项目中的定时器容器为带头尾结点的升序双向链表，具体的为每个连接创建一个定时器，将其添加到链表中，并按照超时时间升序排列。执行定时任务时，将到期的定时器从链表中删除。

从实现上看，主要涉及双向链表的插入，删除操作，其中添加定时器的事件复杂度是O(n)，删除定时器的事件复杂度是O(1)。

升序双向链表主要逻辑如下：

- 创建头尾节点，其中头尾节点没有意义，仅仅统一方便调整

- add_timer函数，将目标定时器添加到链表中，添加时按照升序添加

  - 若当前链表中只有头尾节点，直接插入
  - 否则，将定时器按升序插入

- adjust_timer函数，当定时任务发生变化，调整对应定时器在链表中的位置

  - 客户端在设定时间内有数据收发，则当前时刻对该定时器重新设定时间，这里只是往后延长超时时间
  - 被调整的目标定时器在尾部，或定时器新的超时值仍然小于下一个定时器的超时，不用调整
  - 否则先将定时器从链表取出，重新插入链表

- del_timer函数将超时的定时器从链表中删除

  - 常规双向链表删除结点


```php
//定时器容器类
class sort_timer_lst
{
public:
    sort_timer_lst();
    //常规销毁链表
    ~sort_timer_lst();

    //添加定时器，内部调用私有成员add_timer
    void add_timer(util_timer *timer);
    //调整定时器，任务发生变化时，调整定时器在链表中的位置
    void adjust_timer(util_timer *timer);
    //删除定时器
    void del_timer(util_timer *timer);
    //定时任务处理函数
    void tick();

private:
    //私有成员，被公有成员add_timer和adjust_time调用
    //主要用于调整链表内部结点
    void add_timer(util_timer *timer, util_timer *lst_head);
    //头尾结点
    util_timer *head;
    util_timer *tail;
};
```

```php
//定时器容器类的构造函数
sort_timer_lst::sort_timer_lst()
{
    head = NULL;
    tail = NULL;
}
//析构函数，销毁链表
sort_timer_lst::~sort_timer_lst()
{
    util_timer *tmp = head;
    while (tmp)
    {
        head = tmp->next;
        delete tmp;
        tmp = head;
    }
}

//添加定时器，内部调用私有成员add_timer
void sort_timer_lst::add_timer(util_timer *timer)
{
    if (!timer)
    {
        return;
    }
    if (!head)
    {
        head = tail = timer;
        return;
    }
    //定时器中是按照expire从小到大排序
    //如果新的定时器超时时间小于当前头部结点，直接将当前定时器结点作为头部结点
    if (timer->expire < head->expire)
    {
        timer->next = head;
        head->prev = timer;
        head = timer;
        return;
    }
    //否则调用私有成员，调整内部结点
    add_timer(timer, head);
}

//调整定时器，任务发生变化时，调整定时器在链表中的位置
void sort_timer_lst::adjust_timer(util_timer *timer)
{
    if (!timer)
    {
        return;
    }

    util_timer *tmp = timer->next;
    //被调整的定时器在链表尾部
    //or 定时器超时值仍然小于下一个定时器超时值，不调整
    if (!tmp || (timer->expire < tmp->expire))
    {
        return;
    }

    //被调整定时器是链表头结点，将定时器取出，重新插入
    if (timer == head)
    {
        head = head->next;
        head->prev = NULL;
        timer->next = NULL;
        add_timer(timer, head);
    }
    //被调整定时器在内部，将定时器取出，重新插入
    else
    {
        timer->prev->next = timer->next;
        timer->next->prev = timer->prev;
        add_timer(timer, timer->next);
    }
}

//删除定时器，即双向链表节点的删除
void sort_timer_lst::del_timer(util_timer *timer)
{
    if (!timer)
    {
        return;
    }
    //链表中只有一个定时器，需要删除该定时器
    if ((timer == head) && (timer == tail))
    {
        delete timer;
        head = NULL;
        tail = NULL;
        return;
    }

    //被删除的定时器为头结点
    if (timer == head)
    {
        head = head->next;
        head->prev = NULL;
        delete timer;
        return;
    }

    //被删除的定时器为尾结点
    if (timer == tail)
    {
        tail = tail->prev;
        tail->next = NULL;
        delete timer;
        return;
    }

    //被删除的定时器在链表内部，常规链表结点删除
    timer->prev->next = timer->next;
    timer->next->prev = timer->prev;
    delete timer;
}


//私有成员，被公有成员add_timer和adjust_time调用
//主要用于调整链表内部结点
void sort_timer_lst::add_timer(util_timer *timer, util_timer *lst_head)
{
    util_timer *prev = lst_head;
    util_timer *tmp = prev->next;
    
    //从双向链表中找到该定时器应该放置的位置，即遍历一遍双向链表找到对应的位置
    
    //时间复杂度太高O(n)，这里可以考虑使用C++11的优先队列实现定时器
    
    //遍历当前结点之后的链表，按照超时时间找到目标定时器对应的位置，常规双向链表插入操作
    while (tmp)
    {
        if (timer->expire < tmp->expire)
        {
            prev->next = timer;
            timer->next = tmp;
            tmp->prev = timer;
            timer->prev = prev;
            break;
        }
        prev = tmp;
        tmp = tmp->next;
    }

    //遍历完发现，目标定时器需要放到尾结点处
    if (!tmp)
    {
        prev->next = timer;
        timer->prev = prev;
        timer->next = NULL;
        tail = timer;
    }
}
```

## 5、定时任务处理函数

使用统一事件源，SIGALRM信号每次被触发，主循环中调用一次定时任务处理函数，处理链表容器中到期的定时器。

具体的逻辑如下：

- 遍历定时器升序链表容器，从头结点开始依次处理每个定时器，直到遇到尚未到期的定时器；
- 若当前时间小于定时器超时时间，跳出循环，即未找到到期的定时器；
- 若当前时间大于定时器超时时间，即找到了到期的定时器，执行回调函数，然后将它从链表中删除，然后继续遍历；

```php
//定时任务处理函数
void sort_timer_lst::tick()
{
    if (!head)
    {
        return;
    }
    
    //获取当前时间
    time_t cur = time(NULL);
    util_timer *tmp = head;

    //遍历定时器链表
    while (tmp)
    {
        //当前时间小于定时器的超时时间，后面的定时器也没有到期
        if (cur < tmp->expire)
        {
            break;
        }

        //当前定时器到期，则调用回调函数，执行定时事件
        tmp->cb_func(tmp->user_data);

        //将处理后的定时器从链表容器中删除，并重置头结点
        head = tmp->next;
        if (head)
        {
            head->prev = NULL;
        }
        delete tmp;
        tmp = head;
    }
}
```

## 6、如何使用定时器

服务器首先创建定时器容器链表，然后用统一事件源将异常事件、读写事件和信号事件统一处理，根据不同事件的对应逻辑使用定时器。

<img src="https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207021721326.png" alt="image-20220702172100255" style="zoom:80%;" />

具体的逻辑如下：

- 浏览器与服务器连接时，创建该连接对应的定时器，并将该定时器添加到链表上；
- 处理异常事件时，执行定时事件，服务器关闭连接，从链表上移除对应定时器；
- 处理定时信号时，将定时标志设置为true；
- 处理读事件时，若某连接上发生读事件，将对应定时器向后移动，否则，执行定时事件；
- 处理写事件时，若服务器通过某连接给浏览器发送数据，将对应定时器向后移动，否则，执行定时事件；

```php
//定时处理任务，重新定时以不断触发SIGALRM信号
void Utils::timer_handler()
{
    m_timer_lst.tick();
    //最小的时间单位为5s
    alarm(m_TIMESLOT);
}

//创建定时器容器链表
static sort_timer_lst timer_lst;
//创建连接资源数组
client_data *users_timer = new client_data[MAX_FD];
//超时默认为False
bool timeout = false;
//alarm定时触发SIGALRM信号
alarm(TIMESLOT);

void WebServer::eventLoop()
{
    bool timeout = false;
    bool stop_server = false;

    while (!stop_server)
    {
        //等待所监控文件描述符上有事件的产生
        int number = epoll_wait(m_epollfd, events, MAX_EVENT_NUMBER, -1);
        //EINTR错误的产生：当阻塞于某个系统调用的一个进程捕获某个信号
        //且相应信号处理函数返回时，该系统调用可能返回一个EINTR错误
        if (number < 0 && errno != EINTR)
        {
            LOG_ERROR("%s", "epoll failure");
            break;
        }
        //对所有就绪事件进行处理
        for (int i = 0; i < number; i++)
        {
            int sockfd = events[i].data.fd;
            //处理新到的客户连接
            if (sockfd == m_listenfd)
            {
                bool flag = dealclinetdata();
                if (false == flag)
                    continue;
            }
            //处理异常事件
            else if (events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR))
            {
                //服务器端关闭连接，移除对应的定时器
                util_timer *timer = users_timer[sockfd].timer;
                deal_timer(timer, sockfd);
            }
            //处理定时器信号
            else if ((sockfd == m_pipefd[0]) && (events[i].events & EPOLLIN))
            {
                //接收到SIGALRM信号，timeout设置为True
                bool flag = dealwithsignal(timeout, stop_server);
                if (false == flag)
                    LOG_ERROR("%s", "dealclientdata failure");
            }
            //处理客户连接上接收到的数据
            else if (events[i].events & EPOLLIN)
            {
                dealwithread(sockfd);
            }
            //处理客户连接上send的数据
            else if (events[i].events & EPOLLOUT)
            {
                dealwithwrite(sockfd);
            }
        }

        //处理定时器为非必须事件，收到信号并不是立马处理
        //完成读写事件后，再进行处理
        if (timeout)
        {
            utils.timer_handler();
            LOG_INFO("%s", "timer tick");
            timeout = false;
        }
    }
```

```php
//http 处理用户数据
bool WebServer::dealclinetdata()
{
    struct sockaddr_in client_address;
    socklen_t client_addrlength = sizeof(client_address);
    //LT
    if (0 == m_LISTENTrigmode)
    {
        int connfd = accept(m_listenfd, (struct sockaddr *)&client_address, &client_addrlength);
        if (connfd < 0)
        {
            LOG_ERROR("%s:errno is:%d", "accept error", errno);
            return false;
        }
        if (http_conn::m_user_count >= MAX_FD)
        {
            utils.show_error(connfd, "Internal server busy");
            LOG_ERROR("%s", "Internal server busy");
            return false;
        }
        timer(connfd, client_address);
    }

    // ET
    else
    {
        //边缘触发需要一直accept直到为空
        while (1)
        {
            int connfd = accept(m_listenfd, (struct sockaddr *)&client_address, &client_addrlength);
            if (connfd < 0)
            {
                LOG_ERROR("%s:errno is:%d", "accept error", errno);
                break;
            }
            if (http_conn::m_user_count >= MAX_FD)
            {
                utils.show_error(connfd, "Internal server busy");
                LOG_ERROR("%s", "Internal server busy");
                break;
            }
            timer(connfd, client_address);
        }
        return false;
    }
    return true;
}

//创建一个定时器节点，将连接信息挂载
void WebServer::timer(int connfd, struct sockaddr_in client_address)
{
    users[connfd].init(connfd, client_address, m_root, m_CONNTrigmode,
                       m_close_log, m_user, m_passWord, m_databaseName);
    //初始化client_data数据
    //创建定时器，设置回调函数和超时时间，绑定用户数据，将定时器添加到链表中
    //初始化该连接对应的连接资源
    users_timer[connfd].address = client_address;
    users_timer[connfd].sockfd = connfd;
    //创建定时器临时变量
    util_timer *timer = new util_timer;
    //设置定时器对应的连接资源
    timer->user_data = &users_timer[connfd];
    //设置回调函数
    timer->cb_func = cb_func;
    //获取当前时间
    time_t cur = time(NULL);
    //TIMESLOT:最小时间间隔单位为5s，设置绝对超时时间
    timer->expire = cur + 3 * TIMESLOT;
    //创建该连接对应的定时器，初始化为前述临时变量
    users_timer[connfd].timer = timer;
    //将该定时器添加到链表中
    utils.m_timer_lst.add_timer(timer);
}
```

```php
//删除定时器节点，关闭连接
void WebServer::deal_timer(util_timer *timer, int sockfd)
{
    //服务器端关闭连接，移除对应的定时器
    timer->cb_func(&users_timer[sockfd]);
    if (timer)
    {
        utils.m_timer_lst.del_timer(timer);
    }

    LOG_INFO("close fd %d", users_timer[sockfd].sockfd);
}
```

```php
//处理定时器信号，将定时标志设置为true
bool WebServer::dealwithsignal(bool &timeout, bool &stop_server)
{
    int ret = 0;
    int sig;
    char signals[1024];
    //从管道读端读出信号值，成功返回字节数，失败返回-1
    //正常情况下，这里的ret返回值总是1，只有14和15两个ASCII码对应的字符
    ret = recv(m_pipefd[0], signals, sizeof(signals), 0);
    if (ret == -1)
    {
        // handle the error
        return false;
    }
    else if (ret == 0)
    {
        return false;
    }
    else
    {
        //处理信号值对应的逻辑
        for (int i = 0; i < ret; ++i)
        {
            
            //这里面明明是字符
            switch (signals[i])
            {
            //这里是整型
            case SIGALRM:
            {
                timeout = true;
                break;
            }
            //关闭服务器
            case SIGTERM:
            {
                stop_server = true;
                break;
            }
            }
        }
    }
    return true;
}
```

```php
//处理客户连接上接收到的数据
void WebServer::dealwithread(int sockfd)
{
    //创建定时器临时变量，将该连接对应的定时器取出来
    util_timer *timer = users_timer[sockfd].timer;

    //reactor
    if (1 == m_actormodel)
    {
        if (timer)
        {
 			//若有数据传输，则将定时器往后延迟3个单位
 			//对其在链表上的位置进行调整
            adjust_timer(timer);
        }

        //若监测到读事件，将该事件放入请求队列
        m_pool->append(users + sockfd, 0);
        while (true)
        {
            //是否正在处理中
            if (1 == users[sockfd].improv)
            {
                //事件类型关闭连接
                if (1 == users[sockfd].timer_flag)
                {
                    //服务器端关闭连接，移除对应的定时器
                    deal_timer(timer, sockfd);
                    users[sockfd].timer_flag = 0;
                }
                users[sockfd].improv = 0;
                break;
            }
        }
    }
    //proactor
    else
    {   
        //先读取数据，再放进请求队列
        if (users[sockfd].read_once())
        {
            LOG_INFO("deal with the client(%s)", inet_ntoa(users[sockfd].get_address()->sin_addr));
            //将该事件放入请求队列
            m_pool->append_p(users + sockfd);
            if (timer)
            {
				//若有数据传输，则将定时器往后延迟3个单位
 				//对其在链表上的位置进行调整
                adjust_timer(timer);
            }
        }
        else
        {
            //服务器端关闭连接，移除对应的定时器
            deal_timer(timer, sockfd);
        }
    }
}

//若数据活跃，则将定时器节点往后延迟3个时间单位
//并对新的定时器在链表上的位置进行调整
void WebServer::adjust_timer(util_timer *timer)
{
    time_t cur = time(NULL);
    timer->expire = cur + 3 * TIMESLOT;
    utils.m_timer_lst.adjust_timer(timer);

    LOG_INFO("%s", "adjust timer once");
}

//删除定时器节点，关闭连接
void WebServer::deal_timer(util_timer *timer, int sockfd)
{
    timer->cb_func(&users_timer[sockfd]);
    if (timer)
    {
        utils.m_timer_lst.del_timer(timer);
    }

    LOG_INFO("close fd %d", users_timer[sockfd].sockfd);
}
```

```php
//写操作
void WebServer::dealwithwrite(int sockfd)
{
    util_timer *timer = users_timer[sockfd].timer;
    //reactor
    if (1 == m_actormodel)
    {
        if (timer)
        {
            adjust_timer(timer);
        }

        m_pool->append(users + sockfd, 1);

        while (true)
        {
            if (1 == users[sockfd].improv)
            {
                if (1 == users[sockfd].timer_flag)
                {
                    deal_timer(timer, sockfd);
                    users[sockfd].timer_flag = 0;
                }
                users[sockfd].improv = 0;
                break;
            }
        }
    }
    else
    {
        //proactor
        if (users[sockfd].write())
        {
            LOG_INFO("send data to the client(%s)", inet_ntoa(users[sockfd].get_address()->sin_addr));

            if (timer)
            {
                adjust_timer(timer);
            }
        }
        else
        {
            deal_timer(timer, sockfd);
        }
    }
}
```

**有小伙伴问，连接资源中的address是不是有点鸡肋？**

确实如此，项目中虽然对该变量赋值，但并没有用到。类似的，可以对比HTTP类中address属性，只在日志输出中用到。但不能说这个变量没有用，因为我们可以找到客户端连接的ip地址，用它来做一些业务，比如通过ip来判断是否异地登录等等。

# 四、日志系统

[最新版Web服务器项目详解 - 09 日志系统（上） (qq.com)](https://mp.weixin.qq.com/s/IWAlPzVDkR2ZRI5iirEfCg)

为了记录服务器的运行状态，错误信息，访问数据的文件等，需要建立一个日志系统。

## 1、基础知识

**日志**，由服务器自动创建，并记录运行状态，错误信息，访问数据的文件。

**同步日志**，日志写入函数与工作线程串行执行，由于涉及到I/O操作，当单条日志比较大的时候，同步模式会阻塞整个处理流程，服务器所能处理的并发能力将有所下降，尤其是在峰值的时候，写日志可能成为系统的瓶颈。

**生产者——消费者模型**，并发编程中的经典模型。以多线程为例，为了实现线程间数据同步，生产者线程与消费者线程共享一个缓冲区，其中生产者线程往缓冲区中push消息，消费者线程从缓冲区中pop消息。

**阻塞队列**，将生产者——消费者模型进行封装，使用**循环数组**实现队列，作为两者共享的缓冲区。

**异步日志**，将所写的日志内容先存入阻塞队列，写线程从阻塞队列中取出内容，写入日志。

**单例模式**，最简单也是被问到最多的设计模式之一，保证一个类只创建一个实例，同时提供全局访问的方法。

本项目中，使用**单例模式**创建日志系统，对服务器运行状态、错误信息和访问数据进行记录，该系统可以实现按天分类，超行分类功能，可以根据实际情况分别使用**同步和异步**写入两种方式。

**同步/异步**日志系统主要涉及了两个模块，一个是日志模块，一个是阻塞队列模块。其中加入阻塞队列模块主要是解决异步写入日志做准备。

其中**异步**写入方式，将生产者——消费者模型封装为阻塞队列，创建一个**写线程**，工作线程将要写的内容push进队列，写线程从队列中取出内容，写入日志文件。

日志系统大致可以分成两部分，其一是单例模式与阻塞队列的定义，其二是日志类的定义与使用。

## 2、单例模式

单例模式作为最常用的设计模式之一，保证一个类仅有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享。

**实现思路**：私有化它的构造函数，以防止外界创建单例类的对象；使用类的私有静态指针变量指向类的唯一实例，并用一个公有的静态方法获取该实例。

单例模式有两种实现方法，分别是**懒汉**和**饿汉**模式。顾名思义，懒汉模式，即非常懒，不用的时候不去初始化，所以在第一次被使用时才进行初始化；饿汉模式，即迫不及待，在程序运行时立即初始化。

### 经典的线程安全懒汉模式

单例模式的实现思路如前述所示，其中，经典的**线程安全懒汉模式**，使用**双检测锁模式**。

```php
class single{
private:
    //私有静态指针变量指向唯一实例
    static single *p;
    //静态锁，是由于静态函数只能访问静态成员
    static pthread_mutex_t lock;
    //私有化构造函数
    single(){
        pthread_mutex_init(&lock, NULL);
    }
    ~single(){}
public:
    //公有静态方法获取实例
    static single* getinstance();
};

pthread_mutex_t single::lock;
single* single::p = NULL;
single* single::getinstance(){
    if (NULL == p){
        pthread_mutex_lock(&lock);
        if (NULL == p){
            p = new single;
        }
        pthread_mutex_unlock(&lock);
    }
    return p;
}
```

**为什么要用双检测，只检测一次不行吗？**

如果只检测一次，在每次调用获取实例的方法时，都需要加锁，这将严重影响程序性能。双层检测可以有效避免这种情况，仅在第一次创建单例的时候加锁，其他时候都不再符合NULL == p的情况，直接返回已创建好的实例。

### 局部静态变量之线程安全懒汉模式

前面的双检测锁模式，写起来不太优雅，《Effective C++》（Item 04）中的提出另一种更优雅的单例模式实现，使用函数内的局部静态对象，这种方法**不用加锁和解锁操作**。

```php
class single{
private:
    single(){}
    ~single(){}
public:
    static single* getinstance();
};
single* single::getinstance(){
    static single obj;
    return &obj;
}
```

**那这种方法不加锁不会造成线程安全问题吗？**

其实，C++0X以后，要求编译器保证内部静态变量的线程安全性，故C++0x之后该实现是线程安全的，C++0x之前仍需加锁，其中C++0x是C++11标准成为正式标准之前的草案临时名字。

所以，如果使用C++11之前的标准，还是需要加锁，这里同样给出加锁的版本。

```php
class single{
private:
    static pthread_mutex_t lock;
    single(){
        pthread_mutex_init(&lock, NULL);
    }
    ~single(){}
public:
    static single* getinstance();
};

pthread_mutex_t single::lock;
single* single::getinstance(){
    pthread_mutex_lock(&lock);
    static single obj;
    pthread_mutex_unlock(&lock);
    return &obj;
}
```

### 饿汉模式

饿汉模式**不需要用锁**，就可以实现线程安全。原因在于，在程序运行时就定义了对象，并对其初始化。之后，不管哪个线程调用成员函数getinstance()，都只不过是返回一个对象的指针而已。所以是线程安全的，不需要在获取实例的成员函数中加锁。

```php
class single{
private:
    static single* p;
    single(){}
    ~single(){}
public:
    static single* getinstance();
};

single* single::p = new single();
single* single::getinstance(){
    return p;
}

//测试方法
int main(){
    single *p1 = single::getinstance();
    single *p2 = single::getinstance();
    if (p1 == p2)
        cout << "same" << endl;
    system("pause");
    return 0;
}
```

饿汉模式虽好，但其存在隐藏的问题：非静态对象（函数外的static对象）在不同编译单元中的初始化顺序是未定义的，如果在初始化完成之前调用 getInstance() 方法会返回一个未定义的实例。

## 3、条件变量与生产者——消费者模型

### 条件变量API与陷阱

条件变量提供了一种线程间的通知机制，当某个共享数据达到某个值时，唤醒等待这个共享数据的线程。

- pthread_cond_init函数，用于初始化条件变量
- pthread_cond_destory函数，销毁条件变量
- pthread_cond_broadcast函数，以广播的方式唤醒**所有**等待目标条件变量的线程
- pthread_cond_wait函数，用于等待目标条件变量。该函数调用时需要传入 **mutex参数(加锁的互斥锁)** ，函数执行时，先把调用线程放入条件变量的请求队列，然后将互斥锁mutex解锁，当函数成功返回为0时，表示重新抢到了互斥锁，互斥锁会再次被锁上， **也就是说函数内部会有一次解锁和加锁操作**。

使用pthread_cond_wait方式如下：

```php
pthread _mutex_lock(&mutex)
while(线程执行的条件是否成立){
    pthread_cond_wait(&cond, &mutex);
}
pthread_mutex_unlock(&mutex);
```

pthread_cond_wait执行后的内部操作分为以下几步：

- 将线程放在条件变量的请求队列后，内部解锁；
- 线程等待被pthread_cond_broadcast信号唤醒或者pthread_cond_signal信号唤醒，唤醒后去竞争锁；
- 若竞争到互斥锁，内部再次加锁；

**使用前要加锁，为什么要加锁？**

多线程访问，为了避免资源竞争，所以要加锁，使得每个线程互斥的访问公有资源。

**pthread_cond_wait内部为什么要解锁？**

如果while或者if判断的时候，满足执行条件，线程便会调用pthread_cond_wait阻塞自己，此时它还在持有锁，如果他不解锁，那么其他线程将会无法访问公有资源。

具体到pthread_cond_wait的内部实现，当pthread_cond_wait被调用线程阻塞的时候，pthread_cond_wait会自动释放互斥锁。

**为什么要把调用线程放入条件变量的请求队列后再解锁？**

线程是并发执行的，如果在把调用线程A放在等待队列之前，就释放了互斥锁，这就意味着其他线程比如线程B可以获得互斥锁去访问公有资源，这时候线程A所等待的条件改变了，但是它没有被放在请求队列上，导致A忽略了等待条件被满足的信号。

倘若在线程A调用pthread_cond_wait开始，到把A放在等待队列的过程中，都持有互斥锁，其他线程无法得到互斥锁，就不能改变公有资源。

**为什么最后还要加锁？**

将线程放在条件变量的请求队列后，将其解锁，此时等待被唤醒，若成功竞争到互斥锁，再次加锁。

**为什么判断线程执行的条件用`while`而不是`if`？**

一般来说，在多线程资源竞争的时候，在一个使用资源的线程里面（消费者）判断资源是否可用，不可用，便调用pthread_cond_wait，在另一个线程里面（生产者）如果判断资源可用的话，则调用pthread_cond_signal发送一个资源可用信号。

在wait成功之后，资源就一定可以被使用么？答案是否定的，如果同时有两个或者两个以上的线程正在等待此资源，wait返回后，资源可能已经被使用了。

再具体点，有可能多个线程都在等待这个资源可用的信号，信号发出后只有一个资源可用，但是有A，B两个线程都在等待，B比较速度快，获得互斥锁，然后加锁，消耗资源，然后解锁，之后A获得互斥锁，但A回去发现资源已经被使用了，它便有两个选择，一个是去访问不存在的资源，另一个就是继续等待，那么继续等待下去的条件就是使用`while`，要不然使用`if`的话pthread_cond_wait返回后，就会顺序执行下去。

所以，在这种情况下，应该使用while而不是if:

```php
while(resource == FALSE)
    pthread_cond_wait(&cond, &mutex);
```

如果只有一个消费者，那么使用if是可以的。

### 生产者——消费者模型

这里摘抄《Unix 环境高级编程》中第11章线程关于pthread_cond_wait的介绍中有一个生产者——消费者的例子P311，其中，process_msg相当于消费者，enqueue_msg相当于生产者，struct msg* workq作为缓冲队列。

生产者和消费者是互斥关系，两者对缓冲区访问互斥，同时生产者和消费者又是一个相互协作与同步的关系，只有生产者生产之后，消费者才能消费。

```php
#include <pthread.h>
struct msg {
  struct msg *m_next;
  /* value...*/
};

struct msg* workq;
pthread_cond_t qready = PTHREAD_COND_INITIALIZER;
pthread_mutex_t qlock = PTHREAD_MUTEX_INITIALIZER;

//消费者
void process_msg() {
  struct msg* mp;
  for (;;) {
    pthread_mutex_lock(&qlock);
    //这里需要用while，而不是if
    while (workq == NULL) {
      pthread_cond_wait(&qread, &qlock);
    }
    mq = workq;
    workq = mp->m_next;
    pthread_mutex_unlock(&qlock);
    /* now process the message mp */
  }
}

//生产者
void enqueue_msg(struct msg* mp) {
    pthread_mutex_lock(&qlock);
    mp->m_next = workq;
    workq = mp;
    pthread_mutex_unlock(&qlock);
    /** 此时另外一个线程在signal之前，执行了process_msg，刚好把mp元素拿走*/
    pthread_cond_signal(&qready);
    /** 此时执行signal, 在pthread_cond_wait等待的线程被唤醒，
        但是mp元素已经被另外一个线程拿走，所以，workq还是NULL，因此需要继续等待*/
}
```

## 4、阻塞队列代码分析

阻塞队列类中封装了生产者——消费者模型，其中push成员是生产者，pop成员是消费者。

阻塞队列中，使用了**循环数组**实现了队列，作为两者共享缓冲区，队列也可以使用STL中的queue。

### 自定义队列

当队列为空时，从队列中获取元素的线程将会被挂起；当队列是满时，往队列里添加元素的线程将会挂起。

```php
/*************************************************************
*循环数组实现的阻塞队列，m_back = (m_back + 1) % m_max_size;  
*线程安全，每个操作前都要先加互斥锁，操作完后，再解锁
**************************************************************/
template <class T>
class block_queue
{
public:
    //初始化私有成员
    block_queue(int max_size = 1000)
    {
        if (max_size <= 0)
        {
            exit(-1);
        }
		//构造函数创建循环数组
        m_max_size = max_size;
        m_array = new T[max_size];
        m_size = 0;
        m_front = -1;
        m_back = -1;
    }

    void clear()
    {
        m_mutex.lock();
        m_size = 0;
        m_front = -1;
        m_back = -1;
        m_mutex.unlock();
    }

    ~block_queue()
    {
        m_mutex.lock();
        if (m_array != NULL)
            delete [] m_array;
        m_mutex.unlock();
    }
    //判断队列是否满了
    bool full() 
    {
        m_mutex.lock();
        if (m_size >= m_max_size)
        {
            m_mutex.unlock();
            return true;
        }
        m_mutex.unlock();
        return false;
    }
    //判断队列是否为空
    bool empty() 
    {
        m_mutex.lock();
        if (0 == m_size)
        {
            m_mutex.unlock();
            return true;
        }
        m_mutex.unlock();
        return false;
    }
    //返回队首元素
    bool front(T &value) 
    {
        m_mutex.lock();
        if (0 == m_size)
        {
            m_mutex.unlock();
            return false;
        }
        value = m_array[m_front];
        m_mutex.unlock();
        return true;
    }
    //返回队尾元素
    bool back(T &value) 
    {
        m_mutex.lock();
        if (0 == m_size)
        {
            m_mutex.unlock();
            return false;
        }
        value = m_array[m_back];
        m_mutex.unlock();
        return true;
    }

    int size() 
    {
        int tmp = 0;
        m_mutex.lock();
        tmp = m_size;
        m_mutex.unlock();
        return tmp;
    }

    int max_size()
    {
        int tmp = 0;
        m_mutex.lock();
        tmp = m_max_size;
        m_mutex.unlock();
        return tmp;
    }
    //往队列添加元素，需要将所有使用队列的线程先唤醒
    //当有元素push进队列，相当于生产者生产了一个元素
    //若当前没有线程等待条件变量，则唤醒无意义
    bool push(const T &item)
    {
        m_mutex.lock();
        //队列已满
        if (m_size >= m_max_size)
        {
            m_cond.broadcast();
            m_mutex.unlock();
            return false;
        }
        //将新增数据放在循环数组的对应位置
        m_back = (m_back + 1) % m_max_size;
        m_array[m_back] = item;
        m_size++;
        m_cond.broadcast();
        m_mutex.unlock();
        return true;
    }
    //pop时，如果当前队列没有元素，将会等待条件变量
    bool pop(T &item)
    {
        m_mutex.lock();
        //多个消费者的时候，这里要是用while而不是if
        while (m_size <= 0)
        {
            //当重新抢到互斥锁，pthread_cond_wait返回为0
            if (!m_cond.wait(m_mutex.get()))
            {
                m_mutex.unlock();
                return false;
            }
        }
        //取出队列首的元素，这里需要理解一下，使用循环数组模拟的队列 
        m_front = (m_front + 1) % m_max_size;
        item = m_array[m_front];
        m_size--;
        m_mutex.unlock();
        return true;
    }

    //增加了超时处理，在项目中没有使用到
    //在pthread_cond_wait基础上增加了等待的时间，在指定时间内能抢到互斥锁即可
 	//其他逻辑不变
    bool pop(T &item, int ms_timeout)
    {
        struct timespec t = {0, 0};
        struct timeval now = {0, 0};
        gettimeofday(&now, NULL);
        m_mutex.lock();
        if (m_size <= 0)
        {
            t.tv_sec = now.tv_sec + ms_timeout / 1000;
            t.tv_nsec = (ms_timeout % 1000) * 1000;
            if (!m_cond.timewait(m_mutex.get(), t))
            {
                m_mutex.unlock();
                return false;
            }
        }
        if (m_size <= 0)
        {
            m_mutex.unlock();
            return false;
        }
        m_front = (m_front + 1) % m_max_size;
        item = m_array[m_front];
        m_size--;
        m_mutex.unlock();
        return true;
    }

private:
    locker m_mutex;
    cond m_cond;
    T *m_array;
    int m_size;
    int m_max_size;
    int m_front;
    int m_back;
};

```

## 5、日志功能实现

### 日志相关API

[最新版Web服务器项目详解 - 10 日志系统（下） (qq.com)](https://mp.weixin.qq.com/s/f-ujwFyCe1LZa3EB561ehA)

**fputs**

```php
#include <stdio.h>
int fputs(const char *str, FILE *stream);
```

- str，一个数组，包含了要写入的以空字符终止的字符序列。
- stream，指向FILE对象的指针，该FILE对象标识了要被写入字符串的流。

**可变参数宏__VA_ARGS__**

`__VA_ARGS__`是一个可变参数的宏，定义时宏定义中参数列表的最后一个参数为省略号，在实际使用时会发现有时会加##，有时又不加。

```php
//最简单的定义
#define my_print1(...)  printf(__VA_ARGS__)

//搭配va_list的format使用
#define my_print2(format, ...) printf(format, __VA_ARGS__)  
#define my_print3(format, ...) printf(format, ##__VA_ARGS__)
```

`__VA_ARGS__`宏前面加上`##`的作用在于，当可变参数的个数为0时，这里printf参数列表中的`##`会把前面多余的`,`去掉，否则会编译出错，建议使用后面这种，使得程序更加健壮。

**fflush**

```php
#include <stdio.h>
int fflush(FILE *stream);
```

`fflush()`会强迫将缓冲区内的数据写回参数stream 指定的文件中，如果参数stream 为NULL，`fflush()`会将所有打开的文件数据更新。

在使用多个输出函数连续进行多次输出到控制台时，有可能下一个数据在上一个数据还没输出完毕，还在输出缓冲区中时，下一个printf就把另一个数据加入输出缓冲区，结果冲掉了原来的数据，出现输出错误。

在`prinf()`后加上`fflush(stdout);` 强制马上输出到控制台，可以避免出现上述错误。

### 日志类定义

- 日志文件
  - **局部变量的懒汉模式**获取实例
  - 生成日志文件，并判断同步和异步写入方式

- 同步
  - 判断是否分文件
  - 直接格式化输出内容，将信息写入日志文件

- 异步
  - 判断是否分文件
  - 格式化输出内容，将内容写入阻塞队列，创建一个写线程，从阻塞队列取出内容写入日志文件

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202204072013940.jpg)

通过**局部变量的懒汉单例模式**创建日志实例，对其进行初始化生成日志文件后，格式化输出内容，并根据不同的写入方式，完成对应逻辑，写入日志文件。

日志类包括但不限于如下方法：

- 公有的实例获取方法
- 初始化日志文件方法
- 异步日志写入方法，内部调用私有异步方法
- 内容格式化方法
- 刷新缓冲区

```php
class Log{
public:
    //C++11以后，使用局部变量懒汉不用加锁
    static Log *get_instance()
    {
        static Log instance;
        return &instance;
    }

    //异步写日志公有方法，调用私有方法async_write_log
    static void *flush_log_thread(void *args)
    {
        Log::get_instance()->async_write_log();
    }
    
    //可选择的参数有日志文件、日志缓冲区大小、最大行数以及最长日志条队列
    bool init(const char *file_name, int close_log, int log_buf_size = 8192, 
              int split_lines = 5000000, int max_queue_size = 0);
	//将输出内容按照标准格式整理
    void write_log(int level, const char *format, ...);
	//强制刷新缓冲区
    void flush(void);

private:
    Log();
    virtual ~Log();
    //异步写日志方法
    void *async_write_log()
    {
        string single_log;
        //从阻塞队列中取出一个日志string，写入文件
        while (m_log_queue->pop(single_log))
        {
            m_mutex.lock();
            fputs(single_log.c_str(), m_fp);
            m_mutex.unlock();
        }
    }

private:
    char dir_name[128]; //路径名
    char log_name[128]; //log文件名
    int m_split_lines;  //日志最大行数
    int m_log_buf_size; //日志缓冲区大小
    long long m_count;  //日志行数记录
    int m_today;        //因为按天分类,记录当前时间是那一天
    FILE *m_fp;         //打开log的文件指针
    char *m_buf;
    block_queue<string> *m_log_queue; //阻塞队列
    bool m_is_async;                  //是否同步标志位
    locker m_mutex;
    int m_close_log; //关闭日志
};

#define LOG_DEBUG(format, ...) if(0 == m_close_log) {Log::get_instance()->write_log(0, format, ##__VA_ARGS__); Log::get_instance()->flush();}
#define LOG_INFO(format, ...) if(0 == m_close_log) {Log::get_instance()->write_log(1, format, ##__VA_ARGS__); Log::get_instance()->flush();}
#define LOG_WARN(format, ...) if(0 == m_close_log) {Log::get_instance()->write_log(2, format, ##__VA_ARGS__); Log::get_instance()->flush();}
#define LOG_ERROR(format, ...) if(0 == m_close_log) {Log::get_instance()->write_log(3, format, ##__VA_ARGS__); Log::get_instance()->flush();}

#endif
```

日志类中的方法都不会被其他程序直接调用，末尾的四个可变参数宏提供了其他程序的调用方法。前述方法对日志等级进行分类，包括`DEBUG`，`INFO`，`WARN`和`ERROR`四种级别的日志。

`init`函数实现日志创建、写入方式的判断，`write_log`函数完成写入日志文件中的具体内容，主要实现日志分级、分文件、格式化输出内容。

### 生成日志文件 && 判断写入方式

通过单例模式获取唯一的日志类，调用`init`方法，初始化生成日志文件，服务器启动按当前时刻创建日志，前缀为时间，后缀为自定义log文件名，并记录创建日志的时间day和行数count。

写入方式通过初始化时**是否设置队列大小**（表示在队列中可以放几条数据）来判断，若队列大小为0，则为同步，否则为异步。

```php
//异步需要设置阻塞队列的长度，同步不需要设置
bool Log::init(const char *file_name, int close_log, int log_buf_size, int split_lines, int max_queue_size)
{
    //如果设置了max_queue_size，则设置为异步
    if (max_queue_size >= 1)
    {
        //设置写入方式flag
        m_is_async = true;
        //创建并设置阻塞队列长度
        m_log_queue = new block_queue<string>(max_queue_size);
        pthread_t tid;
        //flush_log_thread为回调函数，这里表示创建线程异步写日志
        pthread_create(&tid, NULL, flush_log_thread, NULL);
    }
    //输出内容的长度
    m_close_log = close_log;
    m_log_buf_size = log_buf_size;
    m_buf = new char[m_log_buf_size];
    memset(m_buf, '\0', m_log_buf_size);
    //日志的最大行数
    m_split_lines = split_lines;

    time_t t = time(NULL);
    struct tm *sys_tm = localtime(&t);
    struct tm my_tm = *sys_tm;

    //从后往前找到第一个/的位置
    const char *p = strrchr(file_name, '/');
    char log_full_name[256] = {0};

	//相当于自定义日志名
    //若输入的文件名没有/，则直接将时间+文件名作为日志名
    if (p == NULL)
    {
        snprintf(log_full_name, 255, "%d_%02d_%02d_%s", 
                 my_tm.tm_year + 1900, my_tm.tm_mon + 1, my_tm.tm_mday, file_name);
    }
    else
    {
        //将/的位置向后移动一个位置，然后复制到logname中
        //p - file_name + 1是文件所在路径文件夹的长度
        //dirname相当于./
        strcpy(log_name, p + 1);
        strncpy(dir_name, file_name, p - file_name + 1);
        //后面的参数跟format有关
        snprintf(log_full_name, 255, "%s%d_%02d_%02d_%s", 
                 dir_name, my_tm.tm_year + 1900, my_tm.tm_mon + 1, my_tm.tm_mday, log_name);
    }

    m_today = my_tm.tm_mday;
    
    m_fp = fopen(log_full_name, "a");
    if (m_fp == NULL)
    {
        return false;
    }

    return true;
}
```

### 日志分级与分文件

日志分级的实现大同小异，一般的会提供五种级别：

- Debug，调试代码时的输出，在系统实际运行时，一般不使用；
- Warn，这种警告与调试时终端的warning类似，同样是调试代码时使用；
- Info，报告系统当前的状态，当前执行的流程或接收的信息等；
- Error和Fatal，输出系统的错误信息；

项目中给出了除Fatal外的四种分级，实际使用了`Debug`，`Info`和`Error`三种。

超行、按天分文件逻辑：

- 日志写入前会判断当前day是否为创建日志的时间，行数是否超过最大行限制；

- 若为创建日志时间，写入日志，否则按当前时间创建新log，更新创建时间和行数；

- 若行数超过最大行限制，在当前日志的末尾加count/max_lines为后缀创建新log；

将系统信息格式化后输出，具体为：格式化时间 + 格式化内容。

```php
void Log::write_log(int level, const char *format, ...)
{
    struct timeval now = {0, 0};
    gettimeofday(&now, NULL);
    time_t t = now.tv_sec;
    struct tm *sys_tm = localtime(&t);
    struct tm my_tm = *sys_tm;
    char s[16] = {0};
    
    //日志分级
    switch (level)
    {
    case 0:
        strcpy(s, "[debug]:");
        break;
    case 1:
        strcpy(s, "[info]:");
        break;
    case 2:
        strcpy(s, "[warn]:");
        break;
    case 3:
        strcpy(s, "[erro]:");
        break;
    default:
        strcpy(s, "[info]:");
        break;
    }
    //写入一个log，m_count++, m_split_lines为最大行数
    m_mutex.lock();
    //更新现有行数
    m_count++;
	
    //日志不是今天或写入的日志行数是最大行的倍数
    if (m_today != my_tm.tm_mday || m_count % m_split_lines == 0) //everyday log
    {
        char new_log[256] = {0};
        fflush(m_fp);
        fclose(m_fp);
        char tail[16] = {0};
        //格式化日志名中的时间部分
        snprintf(tail, 16, "%d_%02d_%02d_", my_tm.tm_year + 1900, my_tm.tm_mon + 1, my_tm.tm_mday);
       	//如果时间不是今天，则创建今天的日志，更新m_today和m_count
        if (m_today != my_tm.tm_mday)
        {
            snprintf(new_log, 255, "%s%s%s", dir_name, tail, log_name);
            m_today = my_tm.tm_mday;
            m_count = 0;
        }
        else
        {
            //超过了最大行，在之前的日志名基础上加后缀, m_count/m_split_lines
            snprintf(new_log, 255, "%s%s%s.%lld", dir_name, tail, log_name, m_count / m_split_lines);
        }
        m_fp = fopen(new_log, "a");
    }
 
    m_mutex.unlock();

    va_list valst;
    //将传入的format参数赋值给valst，便于格式化输出
    va_start(valst, format);

    string log_str;
    m_mutex.lock();

    //写入内容格式：时间 + 内容
    //时间格式化，snprintf成功返回写字符的总数，其中不包括结尾的null字符
    int n = snprintf(m_buf, 48, "%d-%02d-%02d %02d:%02d:%02d.%06ld %s ",
                     my_tm.tm_year + 1900, my_tm.tm_mon + 1, my_tm.tm_mday,
                     my_tm.tm_hour, my_tm.tm_min, my_tm.tm_sec, now.tv_usec, s);
     //内容格式化，用于向字符串中打印数据、数据格式用户自定义，返回写入到字符数组str中的字符个数(不包含终止符)
    int m = vsnprintf(m_buf + n, m_log_buf_size - 1, format, valst);
    m_buf[n + m] = '\n';
    m_buf[n + m + 1] = '\0';
    log_str = m_buf;

    m_mutex.unlock();

    //若m_is_async为true表示异步，默认为同步
    //若异步，则将日志信息加入阻塞队列，同步则加锁向文件中写
    if (m_is_async && !m_log_queue->full())
    {
        m_log_queue->push(log_str);
    }
    else
    {
        m_mutex.lock();
        fputs(log_str.c_str(), m_fp);
        m_mutex.unlock();
    }

    va_end(valst);
}
```

# 五、数据库连接池

池可以看做资源的容器，所以多种实现方法，比如数组、链表、队列等。这里使用**单例模式**和**链表**创建数据库连接池，实现对数据库连接资源的复用。

项目中的数据库模块分为两部分，其一是数据库连接池的定义，其二是利用连接池完成登录和注册的校验功能。工作线程从数据库连接池取得一个连接，访问数据库中的数据，访问完毕后将连接交还连接池。

本篇将介绍数据库连接池的定义，具体的涉及到单例模式创建、连接池代码实现、RAII机制释放数据库连接。

**单例模式创建**，结合代码描述连接池的单例实现。

**连接池代码实现**，结合代码对连接池的外部访问接口进行详解。

**RAII机制释放数据库连接**，描述连接释放的封装逻辑。

## 单例模式创建

使用局部静态变量懒汉模式创建连接池。

```php
class connection_pool
{
public:
    //局部静态变量单例模式
    static connection_pool *GetInstance();

private:
    connection_pool();
    ~connection_pool();
}

connection_pool *connection_pool::GetInstance()
{
    static connection_pool connPool;
    return &connPool;
}

```

## 连接池代码实现

- `mysql_init`
  - 初始化数据库
- `mysql_real_connect`
  - 连接数据库服务器
  - [PHP mysqli_real_connect() 函数 | 菜鸟教程 (runoob.com)](https://www.runoob.com/php/func-mysqli-real-connect.html)
- `mysql_close`
  - 关闭数据库连接

连接池的功能主要有：初始化，获取连接、释放连接，销毁连接池。

### **初始化**

值得注意的是，销毁连接池没有直接被外部调用，而是通过RAII机制来完成自动释放；使用**信号量**实现多线程争夺连接的同步机制，这里将信号量初始化为数据库的连接总数。

```php
connection_pool::connection_pool()
{
    this->CurConn = 0;
    this->FreeConn = 0;
}

//RAII机制销毁连接池
connection_pool::~connection_pool()
{
    DestroyPool();
}

//构造初始化
void connection_pool::init(string url, string User, string PassWord, 
                           string DBName, int Port, unsigned int MaxConn)
{
    //初始化数据库信息
    this->url = url;
    this->Port = Port;
    this->User = User;
    this->PassWord = PassWord;
    this->DatabaseName = DBName;

    //创建MaxConn条数据库连接
    for (int i = 0; i < MaxConn; i++)
    {
        MYSQL *con = NULL;
        con = mysql_init(con);

        if (con == NULL)
        {
            cout << "Error:" << mysql_error(con);
            exit(1);
        }
        con = mysql_real_connect(con, url.c_str(), User.c_str(), PassWord.c_str(), 
                                 DBName.c_str(), Port, NULL, 0);

        if (con == NULL)
        {
            cout << "Error: " << mysql_error(con);
            exit(1);
        }

        //更新连接池和空闲连接数量
        connList.push_back(con);
        ++FreeConn;
    }

    //将信号量初始化为最大连接次数
    reserve = sem(FreeConn);
    this->MaxConn = FreeConn;
}
```

### **获取、释放连接**

当线程数量大于数据库连接数量时，使用信号量进行同步，每次取出连接，信号量原子减1，释放连接原子加1，若连接池内没有连接了，则阻塞等待。另外，由于多线程操作连接池，会造成竞争，这里使用互斥锁完成同步，具体的同步机制均使用lock.h中封装好的类。

```php
//当有请求时，从数据库连接池中返回一个可用连接，更新使用和空闲连接数
MYSQL *connection_pool::GetConnection()
    MYSQL *con = NULL;
    if (0 == connList.size())
        return NULL;
    //取出连接，信号量原子减1，为0则等待
    reserve.wait();
    lock.lock();
    con = connList.front();
    connList.pop_front();
    //这里的两个变量，并没有用到，非常鸡肋...
    --FreeConn;
    ++CurConn;
    lock.unlock();
    return con;
}

//释放当前使用的连接
bool connection_pool::ReleaseConnection(MYSQL *con)
{
    if (NULL == con)
        return false;
    lock.lock();
    connList.push_back(con);
    ++FreeConn;
    --CurConn;
    lock.unlock();
    //释放连接原子加1
    reserve.post();
    return true;
}
```

### **销毁连接池**

通过迭代器遍历连接池链表，关闭对应数据库连接，清空链表并重置空闲连接和现有连接数量。

```php
//销毁数据库连接池
void connection_pool::DestroyPool()
{
    lock.lock();
    if (connList.size() > 0)
    {
        //通过迭代器遍历，关闭数据库连接
        list<MYSQL *>::iterator it;
        for (it = connList.begin(); it != connList.end(); ++it)
        {
            MYSQL *con = *it;
            mysql_close(con);
        }
        CurConn = 0;
        FreeConn = 0;
        //清空list
        connList.clear();
        lock.unlock();
    }
    lock.unlock();
}
```

## RAII机制释放数据库连接

将数据库连接的获取与释放通过RAII机制封装，避免手动释放。

### **定义**

这里需要注意的是，在获取连接时，通过有参构造对传入的参数进行修改。其中数据库连接本身是指针类型，所以参数需要通过双指针才能对其进行修改。

```php
class connectionRAII{
public:
    //双指针对MYSQL *con修改
    connectionRAII(MYSQL **con, connection_pool *connPool);
    ~connectionRAII();

private:
    MYSQL *conRAII;
    connection_pool *poolRAII;
};
```

### **实现**

不直接调用获取和释放连接的接口，将其封装起来，通过RAII机制进行获取和释放。

```php
connectionRAII::connectionRAII(MYSQL **SQL, connection_pool *connPool){
    *SQL = connPool->GetConnection();
    conRAII = *SQL;
    poolRAII = connPool;
}

connectionRAII::~connectionRAII(){
    poolRAII->ReleaseConnection(conRAII);
}
```

# 六、注册登录

本项目中，使用数据库连接池实现服务器访问数据库的功能，使用POST请求完成注册和登录的校验工作。本篇将介绍同步实现注册登录功能，具体的涉及到流程图，载入数据库表，提取用户名和密码，注册登录流程与页面跳转的的代码实现。

**流程图**，描述服务器从报文中提取出用户名密码，并完成注册和登录校验后，实现页面跳转的逻辑。

**载入数据库表**，结合代码将数据库中的数据载入到服务器中。

**提取用户名和密码**，结合代码对报文进行解析，提取用户名和密码。

**注册登录流程**，结合代码对描述服务器进行注册和登录校验的流程。

**页面跳转**，结合代码对页面跳转机制进行详解。

![image-20220709153537304](https://test1.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202207091535148.png)

- `mysql_query`
  - 执行一条 MySQL 查询
- `mysql_store_result`
  - 本次查询的所有结果都缓存到客户端，对于成功检索了数据的每个查询（SELECT、SHOW、DESCRIBE、EXPLAIN、CHECK TABLE等），必须调用mysql_store_result()或mysql_use_result() 
- `mysql_num_fields`
  - 返回结果集中字段（列）的数量
- `mysql_fetch_fields`
  - 返回结果集中代表字段（列）的对象的数组
- `mysql_fetch_row`
  - 从结果集中取得一行作为数字数组

### 载入数据库表

将数据库中的用户名和密码载入到服务器的map中来，map中的key为用户名，value为密码。

```php
//用户名和密码
map<string, string> users;

void http_conn::initmysql_result(connection_pool *connPool)
{
    //先从连接池中取一个连接
    MYSQL *mysql = NULL;
    connectionRAII mysqlcon(&mysql, connPool);

    //在user表中检索username，passwd数据，浏览器端输入
    if (mysql_query(mysql, "SELECT username,passwd FROM user"))
    {
        LOG_ERROR("SELECT error:%s\n", mysql_error(mysql));
    }

    //从表中检索完整的结果集
    MYSQL_RES *result = mysql_store_result(mysql);

    //返回结果集中的列数
    int num_fields = mysql_num_fields(result);

    //返回所有字段结构的数组
    MYSQL_FIELD *fields = mysql_fetch_fields(result);

    //从结果集中获取下一行，将对应的用户名和密码，存入map中
    while (MYSQL_ROW row = mysql_fetch_row(result))
    {
        string temp1(row[0]);
        string temp2(row[1]);
        users[temp1] = temp2;
    }
}
```

### **提取用户名和密码**

服务器端解析浏览器的请求报文，当解析为POST请求时，cgi标志位设置为1，并将请求报文的消息体赋值给m_string，进而提取出用户名和密码。

```php
//判断http请求是否被完整读入
http_conn::HTTP_CODE http_conn::parse_content(char *text)
{
    if (m_read_idx >= (m_content_length + m_checked_idx))
    {
        text[m_content_length] = '\0';
        //POST请求中最后为输入的用户名和密码
        m_string = text;
        return GET_REQUEST;
    }
    return NO_REQUEST;
}

//根据标志判断是登录检测还是注册检测
char flag = m_url[1];

char *m_url_real = (char *)malloc(sizeof(char) * 200);
strcpy(m_url_real, "/");
strcat(m_url_real, m_url + 2);
strncpy(m_real_file + len, m_url_real, FILENAME_LEN - len - 1);
free(m_url_real);

//将用户名和密码提取出来
//user=123&password=123
char name[100], password[100];
int i;

//以&为分隔符，前面的为用户名
for (i = 5; m_string[i] != '&'; ++i)
    name[i - 5] = m_string[i];
name[i - 5] = '\0';

//以&为分隔符，后面的是密码
int j = 0;
for (i = i + 10; m_string[i] != '\0'; ++i, ++j)
    password[j] = m_string[i];
password[j] = '\0';
```

### 同步线程登录注册

通过m_url定位/所在位置，根据/后的第一个字符判断是登录还是注册校验。

- 2
  - 登录校验


- 3
  - 注册校验


根据校验结果，跳转对应页面。另外，对数据库进行操作时，需要通过锁来同步。

```php
const char *p = strrchr(m_url, '/');
if (0 == m_SQLVerify)
{
    if (*(p + 1) == '3')
    {
        //如果是注册，先检测数据库中是否有重名的
        //没有重名的，进行增加数据
        char *sql_insert = (char *)malloc(sizeof(char) * 200);
        strcpy(sql_insert, "INSERT INTO user(username, passwd) VALUES(");
        strcat(sql_insert, "'");
        strcat(sql_insert, name);
        strcat(sql_insert, "', '");
        strcat(sql_insert, password);
        strcat(sql_insert, "')");

        //判断map中能否找到重复的用户名
        if (users.find(name) == users.end())
        {
            //向数据库中插入数据时，需要通过锁来同步数据
            m_lock.lock();
            int res = mysql_query(mysql, sql_insert);
            users.insert(pair<string, string>(name, password));
            m_lock.unlock();
            //校验成功，跳转登录页面
            if (!res)
                strcpy(m_url, "/log.html");
            //校验失败，跳转注册失败页面
            else
                strcpy(m_url, "/registerError.html");
        }
        else
            strcpy(m_url, "/registerError.html");
    }
    //如果是登录，直接判断
    //若浏览器端输入的用户名和密码在表中可以查找到，返回1，否则返回0
    else if (*(p + 1) == '2')
    {
        if (users.find(name) != users.end() && users[name] == password)
            strcpy(m_url, "/welcome.html");
        else
            strcpy(m_url, "/logError.html");
    }
}
```

### 页面跳转

通过m_url定位/所在位置，根据/后的第一个字符，使用分支语句实现页面跳转。具体的，

- 0
  - 跳转注册页面，GET

- 1
  - 跳转登录页面，GET

- 5
  - 显示图片页面，POST

- 6
  - 显示视频页面，POST

- 7
  - 显示关注页面，POST


```php
//找到url中/所在位置，进而判断/后第一个字符
const char *p = strrchr(m_url, '/');

//注册页面
if (*(p + 1) == '0')
{
    char *m_url_real = (char *)malloc(sizeof(char) * 200);
    strcpy(m_url_real, "/register.html");
    strncpy(m_real_file + len, m_url_real, strlen(m_url_real));
    free(m_url_real);
}

//登录页面
else if (*(p + 1) == '1')
{
    char *m_url_real = (char *)malloc(sizeof(char) * 200);
    strcpy(m_url_real, "/log.html"); 
    strncpy(m_real_file + len, m_url_real, strlen(m_url_real));
    free(m_url_real);
}

//图片页面
else if (*(p + 1) == '5')
{
    char *m_url_real = (char *)malloc(sizeof(char) * 200);
    strcpy(m_url_real, "/picture.html");
    strncpy(m_real_file + len, m_url_real, strlen(m_url_real));
    free(m_url_real);
}

//视频页面
else if (*(p + 1) == '6')
{
    char *m_url_real = (char *)malloc(sizeof(char) * 200);
    strcpy(m_url_real, "/video.html");
    strncpy(m_real_file + len, m_url_real, strlen(m_url_real));
    free(m_url_real);
}

//关注页面
else if (*(p + 1) == '7')
{
    char *m_url_real = (char *)malloc(sizeof(char) * 200);
    strcpy(m_url_real, "/fans.html");
    strncpy(m_real_file + len, m_url_real, strlen(m_url_real));
    free(m_url_real);
}

//否则发送url实际请求的文件
else
    strncpy(m_real_file + len, m_url, FILENAME_LEN - len - 1);
```

# 七、踩坑和面试题

## **大文件传输**

先看下之前的大文件传输，也就是游双书上的代码，发送数据只调用了writev函数，并对其返回值是否异常做了处理。

```php
bool http_conn::write()
{
    int temp = 0;
    int bytes_have_send = 0;
    int bytes_to_send = m_write_idx;
    if (bytes_to_send == 0)
    {
        modfd(m_epollfd, m_sockfd, EPOLLIN);
        init();
        return true;
    }
    while (1)
    {
        temp = writev(m_sockfd, m_iv, m_iv_count);
        if (temp <= -1)
        {
            if (errno == EAGAIN)
            {
                modfd(m_epollfd, m_sockfd, EPOLLOUT);
                return true;
            }
            unmap();
            return false;
        }
        bytes_to_send -= temp;
        bytes_have_send += temp;
        if (bytes_to_send <= bytes_have_send)
        {
            unmap();
            if (m_linger)
            {
                init();
                modfd(m_epollfd, m_sockfd, EPOLLIN);
                return true;
            }
            else
            {
                modfd(m_epollfd, m_sockfd, EPOLLIN);
                return false;
            }
        }
    }
}
```

在实际测试中发现，当请求小文件，也就是调用一次writev函数就可以将数据全部发送出去的时候，不会报错，此时不会再次进入while循环。

一旦请求服务器文件较大文件时，需要多次调用writev函数，便会出现问题，不是文件显示不全，就是无法显示。

对数据传输过程分析后，定位到writev的m_iv结构体成员有问题，每次传输后不会自动偏移文件指针和传输长度，还会按照原有指针和原有长度发送数据。

根据前面的基础API分析，我们知道writev以顺序iov[0]，iov[1]至iov[iovcnt-1]从缓冲区中聚集输出数据。项目中，申请了2个iov，其中iov[0]为存储报文状态行的缓冲区，iov[1]指向资源文件指针。

对上述代码做了修改如下：

- 由于报文消息报头较小，第一次传输后，需要更新m_iv[1].iov_base和iov_len，m_iv[0].iov_len置成0，只传输文件，不用传输响应消息头
- 每次传输后都要更新下次传输的文件起始位置和长度

更新后，大文件传输得到了解决。

```php
bool http_conn::write()
{
    int temp = 0;

    int newadd = 0;

    if (bytes_to_send == 0)
    {
        modfd(m_epollfd, m_sockfd, EPOLLIN, m_TRIGMode);
        init();
        return true;
    }

    while (1)
    {
        temp = writev(m_sockfd, m_iv, m_iv_count);

        if (temp >= 0)
        {
            bytes_have_send += temp;
            newadd = bytes_have_send - m_write_idx;
        }
        else
        {
            if (errno == EAGAIN)
            {
                if (bytes_have_send >= m_iv[0].iov_len)
                {
                    m_iv[0].iov_len = 0;
                    m_iv[1].iov_base = m_file_address + newadd;
                    m_iv[1].iov_len = bytes_to_send;
                }
                else
                {
                    m_iv[0].iov_base = m_write_buf + bytes_have_send;
                    m_iv[0].iov_len = m_iv[0].iov_len - bytes_have_send;
                }
                modfd(m_epollfd, m_sockfd, EPOLLOUT, m_TRIGMode);
                return true;
            }
            unmap();
            return false;
        }
        bytes_to_send -= temp;
        if (bytes_to_send <= 0)

        {
            unmap();
            modfd(m_epollfd, m_sockfd, EPOLLIN, m_TRIGMode);

            if (m_linger)
            {
                init();
                return true;
            }
            else
            {
                return false;
            }
        }
    }
}
```

## 面试题

包括项目介绍，线程池相关，并发模型相关，HTTP报文解析相关，定时器相关，日志相关，压测相关，综合能力等。

### **项目介绍**

- 为什么要做这样一个项目？
- 介绍下你的项目

### **线程池相关**

- 手写线程池
- 线程的同步机制有哪些？
- 线程池中的工作线程是一直等待吗？
- 你的线程池工作线程处理完一个任务后的状态是什么？
- 如果同时1000个客户端进行访问请求，线程数不多，怎么能及时响应处理每一个呢？
- 如果一个客户请求需要占用线程很久的时间，会不会影响接下来的客户请求呢，有什么好的策略呢?

### **并发模型相关**

- 简单说一下服务器使用的并发模型？
- reactor、proactor、主从reactor模型的区别？
- 你用了epoll，说一下为什么用epoll，还有其他复用方式吗？区别是什么？

### **HTTP报文解析相关**

- 用了状态机啊，为什么要用状态机？
- 状态机的转移图画一下
- https协议为什么安全？
- https的ssl连接过程
- GET和POST的区别

### **数据库登录注册相关**

- 登录说一下？
- 你这个保存状态了吗？如果要保存，你会怎么做？（cookie和session）
- 登录中的用户名和密码你是load到本地，然后使用map匹配的，如果有10亿数据，即使load到本地后hash，也是很耗时的，你要怎么优化？
- 用的mysql啊，redis了解吗？用过吗？

### **定时器相关**

- 为什么要用定时器？
- 说一下定时器的工作原理
- 双向链表啊，删除和添加的时间复杂度说一下？还可以优化吗？
- 最小堆优化？说一下时间复杂度和工作原理

### **日志相关**

- 说下你的日志系统的运行机制？
- 为什么要异步？和同步的区别是什么？
- 现在你要监控一台服务器的状态，输出监控日志，请问如何将该日志分发到不同的机器上？（消息队列）

### **压测相关**

- 服务器并发量测试过吗？怎么测试的？
- webbench是什么？介绍一下原理
- 测试的时候有没有遇到问题？

### **综合能力**

- 你的项目解决了哪些其他同类项目没有解决的问题？
- 说一下前端发送请求后，服务器处理的过程，中间涉及哪些协议？



















