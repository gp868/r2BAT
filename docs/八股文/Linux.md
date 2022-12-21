# Linux

# CPU占用过高如何检测？

1. 使用top命令，实时显示进程CPU百分比和内存使用情况，找出CPU占用较高的进程pid。

2. 使用ps命令，查询进程中哪个线程的cpu占用率高，记住TID：`ps -mp pid -o THREAD,tid,time`。

    其中，-m显示所有的线程，-p表示pid进程使用cpu的时间，-o表示该参数后是用户自定义格式，如：THREAD,tid,time表示线程、线程ID号、线程占用的时间。

3. 将TID转换为16进制格式（英文小写格式） `printf “%x\n” tid`

4. 通过`jstack`命令获取占用资源异常的线程栈：

    ```php
    jstack pid > jstack.pid.log #先保存文件，再从文件中查看
    或者
    jstack 514 |grep 202 -A 30  #直接命令行查看
    ```

5. 从上面日志文件或者命令行查看日志，从日志中能看到自己编写的代码的类和方法，一般情况是对应代码处产生了死循环。

# kill和kill -9的区别

kill和kill -9，两个命令在linux中都有杀死进程的效果。执行kill命令，系统会发送一个`SIGTERM`信号给对应的程序。程序接收到该信号后，会先释放自己的资源，然后再停止。但是也有程序可能接收信号后，做一些其他的事情（如果程序正在等待IO，可能就不会立马做出响应），也就是说，SIGTERM多半是会被阻塞的。而`kill -9`命令，系统给程序发送的信号是`SIGKILL`，即exit。exit信号不会被系统阻塞，所以kill -9能顺利杀掉进程。

# 写时拷贝

写时拷贝（copy on write, COW）。

父进程 fork 出的子进程与父进程共享内存空间，一开始父进程的数据不会复制给子进程，这样创建子进程的速度就很快了 (不用复制，直接指向父进程的物理空间)。只有当父子进程中有写入操作，再为子进程分配相应的物理空间。

fork之后，内核把父进程中所有的内存页的权限设置为只读，然后子进程的地址空间指向父进程，与父进程共享数据。当父子进程都只读内存时，正常执行。当某个进程写内存时，CPU检测到内存页是只读的，就会触发页异常中断，内核就会把触发异常的页复制一份出来，这样父子进程就各自持有独立的异常页（其余的页还是共享父进程的）。

写时拷贝可以减少分配和复制大量资源时带来的时间消耗；检查不必要的资源分配，比如fork进程时，并不是所有的页面都需要复制，父进程的代码段和只读数据段都不被允许修改，所以无需复制。

# gdb调试多进程、多线程

- GDB调试多线程

  1. info threads 显示当前可调式的所有线程 

  2. thread ID 切换当前调试的线程为指定ID的线程

  3. thread apply all command 所以的线程都执行command命令

  4. thread apply ID1,ID2.... command  指定线程执行command命令

  5. set scheduler-locking off|on|step： 
     在使用step或continue命令调试当前被调试线程的时候，其他线程也是同时执行的，如果我们只想要被调试的线程执行，而其他线程停止等待，那就要锁定要调试的线程，只让它运行。 
     - off：不锁定任何线程，所有线程都执行。 
     - on：只有当前被调试的线程会执行。 
     - step：阻止其他线程在当前线程单步调试的时候抢占当前线程。只有当next、continue、util以及finish的时候，其他线程才会获得重新运行的
  6. show scheduler-locking： 查看当前锁定线程的模式
  7. i threads 实现线程间切换

- GDB调试多进程

  1. 设置方法
	
     ```
     set follow-fork-mode [parent][child] 
	   set detach-on-fork [on|off] 
     ```
  2. 查看上述两个属性的值
  
     ```
        show follow-fork-mode //查看系统默认的模式
        show detach-on-fork
        /* 
        	parent                   on               只调试主进程（GDB默认）
        	child                    on               只调试子进程
        	parent                   off              同时调试两个进程，gdb跟主进程，子进程block在fork位置
        	child                    off              同时调试两个进程，gdb跟子进程，主进程block在fork位置
        */
     ```

  3. 查询正在调试的进程

       ```
       info inferiors  //查询正在调试的进程
       inferior 进程编号 // 切换调试的进程
       ```
  
  4. add-inferior [-copies n] [-exec executable] //添加新的调试进程
  
  5. detach inferior [进程编号] //释放掉 
  
  6. kill inferior [进程编号] 
  
  7. remove-inferior [进程编号] //删除该进程
  
  8. set schedule-multiple 
  
  9. set print interior-events on/off

# 如何调试死锁

- 借助Core Dump。在程序莫名其妙down掉了，此时操作系统会把当前的内存状况存储在一个core文件中，通过查看core文件就可以直观的看到程序是因为什么而垮掉了。有时候程序down了，但是core文件却没有生成，core文件的生成跟当前系统的环境设置有关系，可以用下面的语句设置一下，然后再运行程序便会生成core文件。

  ```bash
  ulimit -c unlimited
  ```

  core文件生成的位置一般于运行程序的路径相同，文件名一般为core.进程号。

  那么如何在多线程调试中使用Core Dump：

  1. 使用 kill 命令产生 core dump文件：`kill -11 pid`产生core文件。
  2. 使用gdb工具打开core文件：`gdb dead_lock_demo core`
  3. 打印堆栈信息：`thread apply all bt`

# gdb调试core文件

在一个程序崩溃时，它一般会在指定目录下生成一个core文件。core文件仅仅是一个内存映象(同时加上调试信息)，主要是用来调试的。通过core文件调试步骤：

1. ulimit -c unlimted（打开core，默认没有打开）
2. 运行./a.out（编译的时候加调试选项-g） 死锁阻塞,Ctrl+\ 产生core dump
3. gdb ./a.out core.xxx
4. thread apply all bt查看死锁位置

# 其他

- 怎么使一个命令在后台运行?
  
  使用 & 在命令结尾来让程序自动运行。(命令后可以不追加空格)
  
- 用什么命令对一个文件的内容进行统计？(行号、单词数、字节数)
  
  wc ：- c 统计字节数 - l 统计行数 - w 统计字数。(word count)
  
- scp：本地和远程互传文件

- ps：查看当前进程

  ![img](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/818047-20200905084700484-530981088.png)

- top：可实时显示进程CPU百分比和内存使用情况

  [Linux top命令详解 - 牛奔 - 博客园 (cnblogs.com)](https://www.cnblogs.com/niuben/p/12017242.html)

  ![img](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1303876-20191210152709726-52408463.png)

- job：查看后台任务

- lsof：查看所有被进程打开的文件(list opened files)

- 如何查看进程打开的文件

  - ps -aux 获得当前所有的进程，获得pid
  - lsof -p $PID
  - lsof -c programe-name
  - 看文件对应的进程： lsof file-name

- 介绍nm与ldd命令

  - nm (name) ：查看文件中的符号表，如常用的函数名、变量等，以及这些符号存储的区域，`nm [option(s)] [file(s)]`
  - ldd (list dynamic dependencies)：列出程序运行所需要的动态链接库，`ldd 可执行程序路径`

- shell命令查内存，端口 ，io访问量，读写速率

  - top监控系统状态

- 常见命令netstat iptable tcpdump top

  - tcpdump：root权限下使用的抓包工具，只能抓取流经本机的数据包
  - netstate：用于显示网络状态，netstate -a 显示所有连线中的socket
  - top：监控Linux的系统状况
  - iptables：对Linux系统中通信的数据包进行一定的检测，达到防火墙的目的

- gdb查看堆栈中所有遍历

  - `thread apply all bt` 查看所用线程堆栈信息

