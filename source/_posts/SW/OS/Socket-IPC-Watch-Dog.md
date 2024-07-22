---
title: socket进程间通信及喂狗框架
date: 2019-09-13 23:24:34
index_img: /img/post_pics/index_img/服务器软件框图.bmp
tags: 
    - Linux
    - Net
categories: 
    - OS
---

简单的服务器完成以下功能:

1. 与客户端创建连接建立握手  
2. 转发客户端的消息，发向指定客户端  
3. 完成客户端喂狗信号的处理  
<!-- more -->  

- [库函数](#库函数)
  - [socket](#socket)
    - [tcpsocket 实现](#tcpsocket-实现)
    - [socket()](#socket-1)
    - [bind()](#bind)
    - [listen()](#listen)
    - [accept()](#accept)
    - [connect()](#connect)
    - [send()和recv()](#send和recv)
  - [select()](#select)
    - [select()相关API](#select相关api)
    - [使用范例](#使用范例)
    - [深入理解select](#深入理解select)
    - [select的优缺点](#select的优缺点)
  - [pthread\_create()](#pthread_create)
    - [线程](#线程)
    - [线程与进程](#线程与进程)
    - [线程创建与使用](#线程创建与使用)
    - [线程传参](#线程传参)
    - [线程局部变量](#线程局部变量)
    - [线程编译](#线程编译)
- [方案选择](#方案选择)
- [自定义函数及功能](#自定义函数及功能)
  - [运行框图](#运行框图)
- [守护脚本](#守护脚本)
- [实现结果](#实现结果)
- [代码](#代码)
  
## 库函数  

### socket  

参考：[socket编程以及select、epoll、poll示例详解](https://blog.csdn.net/jyy305/article/details/73012706)  
socket这个词可以表示很多概念，在TCP/IP协议中“IP地址+TCP或UDP端口号”唯一标识网络通讯中的一个进程，“IP+端口号”就称为socket。在TCP协议中，建立连接的两个进程各自有一个socket来标识，那么两个socket组成的socket pair就唯一标识一个连接。  
> **网络字节序**：内存中多字节数据相对于内存地址有大端小端之分，磁盘文件中的多字节数据相对于文件中的偏移地址也有大端小端之分。网络数据流同样有大端小端之分，所以发送主机通常将发送缓冲区中的数据按内存地址从低到高的顺序发出，接收主机把从网络上接收到的字节按内存从低到高的顺序保存，因此网络数据流的地址应该规定：`先发出的数据是低地址，后发出的数据是高地址`。TCP/IP协议规定网络数据流应该采用大端字节序，即低地址高字节。所以发送主机和接收主机是小段字节序的在发送和接收之前需要做字节序的转换。  
>
#### tcpsocket 实现

实现模型：

1. **服务器端**： socket -> bind -> listen -> accept(阻塞,三次握手)-> send。  
2. **客户端**： socket -> connect(阻塞,三次握手)-> recv。  
  
#### socket()

```C
int socket(int family, int type, int protocol)
```
  
- **family**：指定协议的类型本次选择AF_INET（IPv4协议）；
- **type**：网络数据类型，TCP是面向字节流的—SOCK_STREAM；
- **protocol**：前两个参数一般确定了协议类型通常传0；
- **返回值**：成功返回套接字符；失败返回-1设置相关错误码。
  
#### bind()

```C
int bind(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen)
```
  
- **sockfd**：socket函数成功时候返回的套接字描述符；  
- **servaddr**：服务器的IP和端口；  
- **addrlen**： 长度（sizeof(servaddr)）；  
- **返回值**：成功返回0；失败返回-1，并设置相关错误码。  
  
#### listen()

```C
int listen(int sockfd, int backlog)
```
  
- **sockfd**：socket函数成功时候返回的套接字描述符；  
- **backlog**：内核中套接字排队的最大个数；  
- **返回值**：成功返回0；失败返回-1，并设置相关错误码。  
  
#### accept()

```C
int accept(int sockfd, const struct sockaddr *servaddr, socklen_t *addrlen)
```
  
- **sockfd**：socket函数成功时候返回的套接字描述符；
- **servaddr**：输出型参数，客户端的ip和端口；
- **addrlen**：长度（sizeof(servaddr)）；
- **返回值**：成功:从监听套接字返回已连接套接字；失败返回-1，并设置相关错误码。
  
#### connect()

```C
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen)
```
  
- **sockfd**：函数返回的套接字描述符；
- **servaddr**：服务器的IP和端口；
- **addrlen**：长度（sizeof(servaddr)）；
- **返回值**：成功返回0；失败返回-1，并设置相关错误码
  
#### send()和recv()

* [**Socket中send()函数和recv()函数详解**](https://blog.csdn.net/ly0303521/article/details/52290217)  
- [**常用socket函数详解**](https://blog.csdn.net/G_BrightBoy/article/details/12854117)  

### select()

```C
int select(int maxfdp,fd_set *readfds,fd_set *writefds,fd_set *errorfds,struct timeval *timeout);
```
  
- **maxfdp**: 需要监视的最大文件描述符加1；  
- **readfds**、**writefds**、**errorfds**：分别对应于需要检测的可读文件描述符的集合，可写文件描述符的集合及异常文件描述符的集合；  
- **timeout**：等待时间，这个时间内，需要监视的描述符没有事件，timeout == NULL表示等待无限时长；  
- **返回值**：发⽣生则函数返回，返回值为0。设为NULL 表示阻塞式等待，一直等到有事件就绪，函数才会返回，0表示非阻塞式等待，没有事件就立即返回，大于0表示等待的时间。大于0表示就绪时间的个数，等于0表示timeout等待时间到了，小于0表示调用失败。  
  
#### select()相关API

```C
//延时结构体
struct timeval
{      
    long tv_sec;   /*秒 */
    long tv_usec;  /*微秒 */   
};

#include <sys/select.h>   
int FD_ZERO(int fd, fd_set *fdset);   //一个 fd_set类型变量的所有位都设为 0
int FD_CLR(int fd, fd_set *fdset);  //清除某个位时可以使用
int FD_SET(int fd, fd_set *fd_set);   //设置变量的某个位置位
int FD_ISSET(int fd, fd_set *fdset); //测试某个位是否被置位
```

#### 使用范例  

当声明了一个文件描述符集后，必须用FD_ZERO将所有位置零。之后将我们所感兴趣的描述符所对应的位置位，操作如下：

```C
fd_set rset;   
int fd;   
FD_ZERO(&rset);   
FD_SET(fd, &rset);   
FD_SET(stdin, &rset);
//调用select
select(fd+1, &rset, NULL, NULL,NULL);
//select返回后用FD_ISSET测试给定位是否置位
if(FD_ISSET(fd, &rset)   
{ 
    ... 
    //do something  
}
```
  
#### 深入理解select

从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大的区别，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。**但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一个线程内同时处理多个IO请求的目的。**而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。  
理解select模型的关键在于理解fd_set,为说明方便，取fd_set长度为1字节，fd_set中的每一bit可以对应一个文件描述符fd。则1字节长的fd_set最大可以对应8个fd。  

1. 执行fd_set set; FD_ZERO(&set); 则set用位表示是0000,0000。
2. 若fd＝5,执行FD_SET(fd,&set);后set变为0001,0000(第5位置为1)
3. 若再加入fd＝2，fd=1,则set变为0001,0011
4. 执行select(6,&set,0,0,0)阻塞等待
5. 若fd=1,fd=2上都发生可读事件，则select返回，此时set变为0000,0011。**注意：没有事件发生的fd=5被清空。**  
  
基于上面的讨论，可以轻松得出select模型的特点：  

1. 可监控的文件描述符个数取决与sizeof(fd_set)的值。我这边服务器上sizeof(fd_set)＝512，每bit表示一个文件描述符，则我服务器上支持的最大文件描述符是512*8=4096。据说可调，另有说虽然可调，但调整上限受于编译内核时的变量值。
2. 将fd加入select监控集的同时，还要再使用一个数据结构array保存放到select监控集中的fd，一是用于再select返回后，array作为源数据和fd_set进行FD_ISSET判断。二是select返回后会把以前加入的但并无事件发生的fd清空，则每次开始select前都要重新从array取得fd逐一加入（FD_ZERO最先），扫描array的同时取得fd最大值maxfd，用于select的第一个参数。
3. 可见select模型必须在select前循环加fd，取maxfd，select返回后利用FD_ISSET判断是否有事件发生。  
  
> 注意：select在设置定时时，检测系统调用中的`SIGALARM`信号，会与其他的定时函数，例如alarm()、settimer()等函数。
  
#### select的优缺点

- **优点：**

1. select的可移植性好，在某些unix下不支持poll.  
2. select对超时值提供了很好的精度，精确到微秒，而poll式毫秒。  

* **缺点：**

1. 单个进程可监视的fd数量被限制，默认是1024。  
2. 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。  
3. 对fd进行扫描时是线性扫描，fd剧增后，IO效率降低，每次调用都对fd进行线性扫描遍历，随着fd的增加会造成遍历速度慢的问题。  
4. select函数超时参数在返回时也是未定义的，考虑到可移植性，每次超时之后进入下一个select之前都要重新设置超时参数。  
  
### pthread_create()

#### 线程

1. 是操作系统能够进行调度的最小单位；  
2. 线程被包含在进程之中，是进程中的实际运作单位；  
3. 一个线程指的是进程中一个单一顺序的控制流；  
4. 一个进程可以并发多个线程，每个线程执行不同的任务。  
  
> - **优点**  
>
> 1. 创建一个新的线程的代价要比创建一个新进程的代价小得多；
> 2. 与进程之间的切换相比，线程之间的切换需要操作系统做的工作要少很多；
> 3. 线程占有的资源要比进程少；
> 4. 线程之间共享数据更容易（线程是在一个房间中，所以相互通信比较容易，而进程之间通话需要跑到另一个房间）。
>
> - **缺点**  
>
> 1. 编码/调试难度提高，因为线程之间谁先执行不确定，在一个是共享资源的问题。  
> 2. 缺乏访问控制，一个线程崩溃，会导致整个进程都异常终止，一个线程中调用某些函数，会影响整个进程。  
>
#### 线程与进程  

1. 进程：进程（或者任务）是资源分配的基本单位  
2. 线程：线程（或者轻量级进程）是调度/执行的基本单位  
3. 线程运行在进程中  
4. 一个进程至少都有一个线程
  
> 注：参考阮一峰的博客[**进程与线程的一个简单解释**](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)
  
| 进程    | 线程           | 描述                                 |
| ------- | -------------- | ------------------------------------ |
| fork    | pthread_create | 新建控制流                           |
| waitpid | pthread_join   | 等待控制流，从当前控制流得到退出状态 |
| exit    | pthread_exit   | 从当前控制流中退出                   |
| getpid  | pthread_self   | 获取当前控制流的ID                   |
| abort   | pthread_cancel | 使指定的控制流异常退出               |
  
#### 线程创建与使用

```C
#include <pthread.h>
void * thread_test(void *arg)
{
    //test
    pthread(NULL); //退出，也可以return
    //return (void *)1;
}
int main()
{
    pthread_t thread1;
    pthread_create(&thread1,NULL,thread_test,NULL);
    //pthread_cancel(thread1); //关闭线程
    
    int wait_thread_end;
    //void *thread1_return;
    //wait_thread_end = pthread_join(thread1, &thread1_return);
    //等待线程的返回，并获取返回值

}
```
  
#### 线程传参

```C
//存储客户端sock_cli的结构体
typedef struct
{
    int sock_cli[10];
    int num;
} SERVER_SOC;

void *wdt_thread(void *arg)
{
    SERVER_SOC *wdt_sockfd;         //新建一个SERVER_SOC结构体指针
    wdt_sockfd = (SERVER_SOC *)arg; //接收传递来的参数结构体
    //对wdt_sockfd->元素操作

    //return (void *)1; //退出线程
    pthread_exit(0); //退出线程
}
int main()
{
    //...
    pthread_create(&thread2, NULL, wdt_thread, (void *)sockfd);
    //...
}
```
  
#### 线程局部变量  

当一个线程多次创建时，线程内定义的变量需要设置为局部变量，一种利用键值的demo如下：  
<details>
<summary>局部变量</summary>  

```C
pthread_key_t key_msg;    //声明一个线程局部变量的共享键值
pthread_key_t key_buffer; //声明一个线程局部变量的共享键值
static void recv_func(int sock_cli)
{
    MSG_STRUCT *recv_msg = pthread_getspecific(key_msg); //从键值中获取局部变量
    char *buffer = pthread_getspecific(key_buffer);      //从键值中获取局部变量
    //...
}

void *msg_thread(void *arg)
{
    SERVER_SOC *msg_sockfd;                                          //新建一个SERVER_SOC结构体指针
    msg_sockfd = (SERVER_SOC *)arg;                                  //接收传递来的参数结构体
    MSG_STRUCT *recv_msg = (MSG_STRUCT *)malloc(sizeof(MSG_STRUCT)); //分配消息结构体指针
    char *buffer = (char *)malloc(sizeof(MSG_STRUCT));               //新建一个接收的缓存区域
    pthread_setspecific(key_msg, recv_msg);                          //将recv_msg与key_msg键值绑定
    pthread_setspecific(key_buffer, buffer);                         //将buffer与key_buffer键值绑定
    printf("线程ID为%lu\n", pthread_self());
    recv_func(msg_sockfd->sock_cli[msg_sockfd->num]); //接收服务端发来是信号

    //return (void *)1; //退出线程
    pthread_exit(0); //退出线程
}

int main()
{
    pthread_key_create(&key_msg, NULL);
    pthread_key_create(&key_buffer, NULL);
    while(1)
    {
        pthread_create(&thread1, NULL, msg_thread, (void *)sockfd); //传参

    }
    pthread_key_delete(key_msg);    //销毁键值
    pthread_key_delete(key_buffer); //销毁键值
}
```

</details>  
  
#### 线程编译

``` bash
gcc -o pthread -lpthread pthread.c  #-lpthread调用libpthread库 
```

```C
# cmake中添加链接库
LINK_LIBRARIES(pthread)     #添加链接库，gcc里面需要添加 -lpthread
```

## 方案选择

实现服务器架构，综上而言，客户端较少的情况下，使用select查找法更为容易，对内存的开销较小，但是当客户端增加后服务器处理客户端消息的实时性将大打折扣。  
  
使用多进程的方式，服务端的子进程之间还需要建立通信，相对而言使用多线程更为容易，且多线程一样具有多进程的实时性。多线程开发中要注意变量的读写，如果变量比较重要且均在线程内，应当设置为局部变量。当然，变量作为全局变量，可以允许各线程加以修改，但一定要解决同步问题。对于单核的处理器而言，每个时间片只运行一个进程的一个线程的一部分，因此微观上，各线程读或者写的顺序是无法控制的，可能会出现读的线程发生在写之前，读到了旧值，这个问题需要非常重视。一个可以反映出问题的demo如下：
<details>
<summary>test.c</summary>  

```C
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
// pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
int g_val = 0;
void* add(void *argv)
{
    for(int i = 0 ; i < 5000; ++i)
    {
        // g_val++;
        // pthread_mutex_lock(&lock);  //上锁
        int tmp = g_val;  //可能读取旧值，而另一个线程已经在加
        g_val = tmp+1;
        usleep(10);
        // pthread_mutex_unlock(&lock); //解锁
    }
}

int main(int argc, char const *argv[])
{
    pthread_t id1,id2;

    pthread_create(&id1,NULL,add,NULL);
    pthread_create(&id2,NULL,add,NULL);

    pthread_join(id1,NULL);
    pthread_join(id2,NULL);

    printf("%d\n",g_val);
    return 0;
}

```

</details>  
代码实际运行，各线程累加5000次，应该值为10000，但实际上如果不加锁和解锁，数值最终无法到达10000。

参考：[linux多线程-互斥&条件变量与同步](https://www.cnblogs.com/lang5230/p/5686868.html)。

## 自定义函数及功能  

<details>
<summary>socket.c</summary>  

```C
#include "am5728.h"
#include "sock.h"

/**
 * @功能 : 获取指定进程的pid
 * @输入 : 进程名称
 * @返回 : 进程pid
 */
int get_other_process_pid(char *process_name)
{
    char popen_cmd[50] = {'\0'};
    sprintf(popen_cmd, "ps -a | grep \'%s\' | awk \'{print $1}\'",
            process_name); //遍历查询PID
    FILE *fp = popen(popen_cmd, "r");
    char pid_buffer[10] = {'\0'};
    fgets(pid_buffer, 10, fp);
    pclose(fp);
    return atoi(pid_buffer);
}

/**
 * @功能 : 向表中添加一个client的接收线程fd，超出范围则返回-1，当有新客户端注册接收线程sockfd时调用
 * @输入 : 客户端sockfd表结构体地址，客户端进程名称，注册的sockfd
 * @返回 : 正确添加返回1；超出范围返回-1
 */
int add_table(CLIENT_TABLE *table, char *client_name, int sock)
{
    int i = 0;
    for (i = 0; i < CLI_NUM; i++)
    {
        if (table->client_recv_sock[i] < 0)
        {
            strcpy(table->client_name[i], client_name);
            table->client_recv_sock[i] = sock;
            // printf("[server ] Add %dth data in client_sockfd:name = %s\tsock = %d\n", i, table->client_name[i], table->client_recv_sock[i]);
            //break;
            return 1;
        }
    }
    return -1;
}

/**
 * @功能 : 从表中搜寻指定client_name的fd并返回，当服务器需要转发消息时调用
 * @输入 : 客户端sockfd表结构体地址，客户端进程名称
 * @返回 : 正确搜寻到返回sockfd，无法查询返回-1
 */
int search_fd(CLIENT_TABLE *table, char *client_name)
{
    int i = 0;
    int sock;
    while (i < CLI_NUM)
    {
        if (strcmp(client_name, table->client_name[i]) == 0)
        {
            sock = table->client_recv_sock[i];
            return sock;
        }
        i++;
    }
    return -1;
}

/**
 * @功能 : 从表中删除一组client的接收线程sockfd，在client突然断开时调用
 * @输入 : 客户端sockfd表结构体地址，注册的sockfd
 * @返回 : 正确搜寻并删除返回1，无法查找返回-1
 */
int delete_fd(CLIENT_TABLE *table, int sock)
{
    int i = 0;
    for (i = 0; i < CLI_NUM; i++)
    {
        if (table->client_recv_sock[i] == sock)
        {
            strcpy(table->client_name[i], "");
            table->client_recv_sock[i] = -1;
            // printf("[server ] Delete %dth data in client_sockfd\n", i);
            return 1;
        }
    }
    return -1;
}
/**
 * @功能 : 客户端注册看门狗时调用，向wdt表中添加对应的进程名称，并将wdt_count置0（默认值-1）
 * @输入 : 看门狗注册表结构体地址，客户端进程名
 * @返回 : 正常添加返回1，已注册满返回-1
 */
int add_wdt(WDT_STRUCT *wdt, char *client_name)
{
    int i = 0;
    for (i = 0; i < CLI_NUM; i++)
    {
        if (wdt->wdt_count[i] < 0)
        {
            strcpy(wdt->wdt_client_name[i], client_name);
            wdt->wdt_count[i] = 0;
            // printf("[server ] Add %dth data in wdt_table:name = %s\twdt_count = %d\n", i, wdt->wdt_client_name[i], wdt->wdt_count[i]);
            //break;
            return -1;
        }
    }
    return 1;
}
/**
 * @功能 : 将看门狗注册表中相应的客户端计数置0，在客户端喂狗时调用
 * @输入 : 看门狗注册表结构体地址，客户端进程名
 * @返回 : 成功置0返回1，无法查询返回-1
 */
int set_wdt_count_zero(WDT_STRUCT *wdt, char *client_name)
{
    int i = 0;
    while (i < CLI_NUM)
    {
        if (strcmp(client_name, wdt->wdt_client_name[i]) == 0)
        {
            wdt->wdt_count[i] = 0;
            // printf("[server ] set %s process's wdt_count to be zero\n", wdt->wdt_client_name[i]);
            return 1;
        }
        i++;
    }
    return -1;
}
/**
 * @功能 : 将看门狗注册表中的所有计数加1，如果超出阈值kill相应进程
 * @输入 : 看门狗注册表结构体地址
 * @返回 : 1
 */
int wdt_count(WDT_STRUCT *wdt)
{
    int i = 0;
    while (i < CLI_NUM)
    {
        if (wdt->wdt_count[i] >= 0) //挑选出注册的wdt
        {
            wdt->wdt_count[i]++;
            // printf("[server ] 进程 %s 的计数加一，wdt_count = %d\n", wdt->wdt_client_name[i], wdt->wdt_count[i]);
            if (wdt->wdt_count[i] >= 3)
            {
                printf("[server ] %s process's(PID = %d) wdt_count is larger than 3\n", wdt->wdt_client_name[i], get_other_process_pid(wdt->wdt_client_name[i]));
                kill(get_other_process_pid(wdt->wdt_client_name[i]), SIGKILL);
                wdt->wdt_count[i] = -1; //重置wdt_count，防止下次注册重复
                strcpy(wdt->wdt_client_name[i], "");
            }
        }
        i++;
    }
    return 1;
}


/**
 * @功能 : 服务器新建socket程序
 * @输入 : ip地址，port端口
 * @返回 : 正常建立返回sock，无法建立返回-1
 */
int server_socket(const char *_ip, int _port)
{
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0)
    {
        perror("[server ] socket");
        return -1;
    }
    struct sockaddr_in local;
    local.sin_family = AF_INET;
    local.sin_port = htons(_port);
    local.sin_addr.s_addr = inet_addr(_ip);
    if (bind(sock, (struct sockaddr *)&local, sizeof(local)) < 0)
    {
        perror("[server ] bind");
        return -1;
    }
    if (listen(sock, 10) < 0)
    {
        perror("[server ] listen");
        return -1;
    }
    return sock;
}


/**
 * @功能 : 处理服务器接收到的消息，分别有客户端接收线程fd注册，看门狗注册，看门狗置0以及消息转发
 * @输入 : 消息结构体地址，客户端sockfd表结构体地址，注册的sockfd，看门狗注册表结构体地址
 * @返回 : 正常完成返回1，错误完成返回-1
 */
int msg_option(PACKET *msg, CLIENT_TABLE *table, int sock, WDT_STRUCT *wdt)
{
    switch (msg->cmd)
    {
    case CMD_RECV: //客户端建立接收线程，调用add_table向表中注册信息
    {
        add_table(table, msg->source_name, sock);
        return 1;
    }
    case CMD_WDT: //客户端发送看门狗消息
    {
        switch (msg->wdt)
        {
        case WDT_REGISTER:
        {
            //printf("wdt : 正在向表中注册client_name\n");
            add_wdt(wdt, msg->source_name);
            break;
        }
        case WDT_SIGNAL:
        {
            //根据source_name查询wdt_count并置零
            //printf("wdt : 正在计数清零\n");
            set_wdt_count_zero(wdt, msg->source_name);
            break;
        }
        default:
        {
            return -1;
        }
        }
        return 1;
    }
    case CMD_TRANSFER: //客户端发送转发消息，查表获取目标fd并转发
    {
        printf("[server ] Transfering a message from %s to %s\n", msg->source_name, msg->target_name);
        char *buf = (char *)malloc(MSG_LEGHTN);
        memcpy(buf, msg, MSG_LEGHTN);
        int target_sock = search_fd(table, msg->target_name);
        // printf("[server ] Searching sockfd...\n");
        if (target_sock < 0)
        {
            printf("[server ] Cannot find sockfd, no route to send!\n");
            return -1;
        }
        write(target_sock, buf, MSG_LEGHTN);
        printf("[server ] Send successfully!\n");
        return 1;
    }
    default:
    {
        return -1;
    }
    }
}
/**
 * @功能 : 客户端新建socket并connect服务器函数
 * @输入 : NULL
 * @返回 : 成功创建返回sockfd，无法连接返回-1
 */
int client_socket()
{
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0)
    {
        perror("socket");
        return -1;
    }
    struct sockaddr_in server;
    server.sin_family = AF_INET;
    server.sin_port = htons(8000);
    server.sin_addr.s_addr = inet_addr("127.1.1.1");
    if (connect(sock, (struct sockaddr *)&server, sizeof(server)) < 0)
    {
        perror("connect");
        return -1;
    }
    return sock;
}

/**
 * @功能 : 转发消息，msg->cmd = 2
 * @输入 : 服务器sockfd，目标进程名，源进程名，消息内容
 * @返回 : 成功发送返回1，发送失败返回-1
 */
int transfer_msg(int sock, char *target_name, char *source_name, char *content)
{
    PACKET *msg = (PACKET *)malloc(MSG_LEGHTN);
    sprintf(msg->head, "%s", "head");
    msg->cmd = 2; //transfer cmd
    msg->wdt = 0;
    sprintf(msg->target_name, "%s", target_name);
    sprintf(msg->source_name, "%s", source_name);
    sprintf(msg->msg, "%s", content);
    char *buf = (char *)malloc(MSG_LEGHTN);
    memcpy(buf, msg, MSG_LEGHTN);
    int result = write(sock, buf, MSG_LEGHTN);
    if (result <= 0)
    {
        printf ("[%s] Cannot send!\n", source_name);
        return -1;
    }
    return 1;
}

/**
 * @功能 : 注册看门狗信息，msg->cmd = 1，msg->wdt = 0
 * @输入 : 服务器sockfd，源进程名
 * @返回 : 成功发送返回1，发送失败返回-1
 */
int register_wdt(int sock, char *source_name)
{
    PACKET *msg = (PACKET *)malloc(MSG_LEGHTN);
    sprintf(msg->head, "%s", "head");
    msg->cmd = 1; //transfer cmd
    msg->wdt = 0; //wdt register
    sprintf(msg->target_name, "%s", "null");
    sprintf(msg->source_name, "%s", source_name);
    sprintf(msg->msg, "%s", "null");
    char *buf = (char *)malloc(MSG_LEGHTN);
    memcpy(buf, msg, MSG_LEGHTN);
    int result= write(sock, buf, MSG_LEGHTN);
    if (result <= 0)
    {
        printf ("[%s] Cannot send!\n", source_name);
        return -1;
    }
    return 1;
}

/**
 * @功能 : 喂狗，msg->cmd = 1，msg->wdt = 1
 * @输入 : 服务器sockfd，源进程名
 * @返回 : 成功发送返回1，发送失败返回-1
 */
int send_wdt(int sock, char *source_name)
{
    PACKET *msg = (PACKET *)malloc(MSG_LEGHTN);
    sprintf(msg->head, "%s", "head");
    msg->cmd = 1; //transfer cmd
    msg->wdt = 1; //wdt signal
    sprintf(msg->target_name, "%s", "null");
    sprintf(msg->source_name, "%s", source_name);
    sprintf(msg->msg, "%s", "null");
    char *buf = (char *)malloc(MSG_LEGHTN);
    memcpy(buf, msg, MSG_LEGHTN);
    int result = write(sock, buf, MSG_LEGHTN);
    if (result <= 0)
    {
        printf ("[%s] Cannot send!\n", source_name);
        return -1;
    }
    return 1;
}
```

</details>  

### 运行框图

服务端  
![](/img/post_pics/tcpip/服务器软件框图.bmp)
客户端  
![](/img/post_pics/tcpip/客户端软件框图.bmp)

## 守护脚本

守护脚本的功能是，当检测到某进程不存在是，立即重启该进程，对应着服务器检测到某个进程不喂狗时立即杀死该进程。两者想结合完成对客户端及服务器端的保护。守护进程中用shell语言编写了protect函数，输入参数为`$1`，即进程名。启动守护脚本后，将分先后运行服务器和客户端可执行文件并运行在后台，再在主循环中轮询守护对应的进程。
<details>
<summary>main.sh</summary>  

``` bash
./zhx_server &
./client1 &
./client2 &

function protect {
    # PRO_NAME = $1   
    #用ps获取$PRO_NAME进程数量
    NUM=`ps aux | grep -w $1 | grep -v grep |wc -l`
    #echo $NUM
    #少于1，重启进程
    if [ "${NUM}" -lt "1" ];then
        echo "[main.sh] $1 was killed"
        ./$1 &
    #大于1，杀掉所有进程，重启
    elif [ "${NUM}" -gt "1" ];then
        echo "[main.sh] more than 1 $1,killall $1"
        killall -9 $PRO_NAME
        ./$1 &
    fi
    #kill僵尸进程
    NUM_STAT=`ps aux | grep -w $1 | grep T | grep -v grep | wc -l`
    if [ "${NUM_STAT}" -gt "0" ];then
        killall -9 $1
        ./$1 &
    fi
    # sleep 1s
}

# NAME="client1"
while true ; do
    protect "zhx_server"
    protect "client1"
    protect "client2"
done
exit 0
```

</details>  

## 实现结果

![](/img/post_pics/tcpip/socket服务器运行结果.JPG)

```
[client1] Creating recv thread...                                   #客户端建立接收线程
[client2] Creating recv thread...
[server ] Transfering a message from client1 to client2             #服务端转发客户端的消息
[server ] Send successfully!                                        #转发成功
[client2] thread recv msg:"this message is from client1 to client2" #client2成功接收
[server ] client1 process's(PID = 5061) wdt_count is larger than 3  #client1计数超时
[main.sh] client1 was killed                                        #服务器关闭客户端
[client1] Creating recv thread...                                   #守护脚本再次重启客户端
......
```

## 代码

<details>
<summary>server.c</summary>  

```C
#include "am5728.h"
#include "sock.h"
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
WDT_STRUCT wdt;          //新建一个客户端进程看门狗维护表的结构体，相关函数传递 &wdt
int client_sockfd[4096]; //所有客户端连接的fd存储数组
/**
 * @功能 : 看门狗信号处理线程，独立于主线程
 * @输入 : NULL
 * @返回 : NULL
 */
void *wdt_opt(void *arg)
{
    while (1)
    {
        sleep(3); //设置定时
        pthread_mutex_lock(&lock);
        printf("[server ] 看门狗线程正在上锁\n");
        wdt_count(&wdt); //计数
        pthread_mutex_unlock(&lock);
    }
}

int main(int argc, char *argv[])
{
    //******************************************************************************************************
    //                                           建立socket和fd_set变量
    //******************************************************************************************************
    int listensock = server_socket("127.1.1.1", 8000);
    if (listensock < 0)
    {
        perror("[server ] Create socket error!\n");
        exit(1);
    }
    int maxfd = 0; //初始化maxfd
    fd_set rfds;   //可读
    struct timeval tv;

    //******************************************************************************************************
    //                                           初始化table结构体
    //******************************************************************************************************
    CLIENT_TABLE *table = (CLIENT_TABLE *)malloc(sizeof(CLIENT_TABLE)); //新建一个客户端进程与接收fd的表的结构体指针
    memset(table, '\0', sizeof(CLIENT_TABLE));                          //初始化表
    int m = 0;                                                          //循环计数值
    for (m = 0; m < CLI_NUM; m++)                                       //将table->client_recv_sock[10]所有内容初始化为-1
        table->client_recv_sock[m] = -1;

    //******************************************************************************************************
    //                                           初始化client_sockfd
    //******************************************************************************************************
    client_sockfd[0] = listensock;                                             //client_sockfd[]的第一个值为server自己的sockfd
    int i = 1;                                                                 //client_sockfd置数循环计数
    int client_sockfd_size = sizeof(client_sockfd) / sizeof(client_sockfd[0]); //client_sockfd_size=4096
    for (; i < client_sockfd_size; i++)                                        //将保存fd的client_sockfd除去第一位，全部预先设为-1
        client_sockfd[i] = -1;
    //******************************************************************************************************
    //                                           初始化wdt结构体
    //******************************************************************************************************
    memset(&wdt, '\0', sizeof(WDT_STRUCT)); //初始化表
    int n = 0;                              //循环计数
    for (n = 0; n < CLI_NUM; n++)
    {
        wdt.wdt_count[n] = -1;
        strcpy(wdt.wdt_client_name[n], "");
    }

    //******************************************************************************************************
    //                                           开启看门狗线程
    //******************************************************************************************************
    pthread_t thread1;                             //新建线程变量
    pthread_create(&thread1, NULL, wdt_opt, NULL); //创建线程

    //******************************************************************************************************
    //                                           主循环
    //******************************************************************************************************
    while (1)
    {
        FD_ZERO(&rfds);                          //将所有位清0
        for (i = 0; i < client_sockfd_size; ++i) //将client_sockfd中成员与fd_set绑定
        {
            if (client_sockfd[i] > 0)
            {
                FD_SET(client_sockfd[i], &rfds);
                if (client_sockfd[i] > maxfd)
                    maxfd = client_sockfd[i]; //取出最大的fd，作为select的第一个参数
            }
        }

        //******************************************************************************************************
        //                                           等待select返回文件描述符变化
        //******************************************************************************************************
        switch (select(maxfd + 1, &rfds, NULL, NULL, NULL)) //第一个参数是被监听的文件描述符的总数
        {
        case 0:
        {
            // printf("timeout\t"); //超时=0
            break;
        }
        case -1:
        {
            //perror("select error"); //错误小于0
            break;
        }
        default: //大于0返回就绪的文件描述符的个数
        {
            int j = 0;
            for (; j < client_sockfd_size; ++j) //遍历查询client_sockfd[]里面所有内容，用FD_ISSET捕获变化位
            {
                //******************************************************************************************************
                //                                           处理客户端创建连接
                //******************************************************************************************************
                if (j == 0 && FD_ISSET(client_sockfd[j], &rfds)) //判断client_sockfd[0]是否有变化
                {
                    struct sockaddr_in client;
                    socklen_t len = sizeof(client);
                    int new_sock = accept(listensock, (struct sockaddr *)&client, &len); //获得accept返回值
                    if (new_sock <= 0)                                                   //accept failed
                    {
                        perror("[server ] accept");
                        continue;
                    }
                    // printf("[server ] get a new client, sock = %d\n", new_sock);
                    fflush(stdout);
                    int k = 1;
                    for (; k < client_sockfd_size; ++k) //找最靠前的-1的项赋值
                    {
                        if (client_sockfd[k] < 0)
                        {
                            client_sockfd[k] = new_sock;
                            if (new_sock > maxfd) //获取新的最大描述符
                                maxfd = new_sock;
                            break;
                        }
                    }
                    if (k == client_sockfd_size) //fd达到上限
                        close(new_sock);
                }
                //******************************************************************************************************
                //                                           处理客户端发送消息
                //******************************************************************************************************
                else if (j != 0 && FD_ISSET(client_sockfd[j], &rfds)) //如果是client_sockfd[1]以后的fd有读的变化
                {
                    char *buf = (char *)malloc(MSG_LEGHTN);              //新建一个接收消息的缓冲buf
                    ssize_t s = read(client_sockfd[j], buf, MSG_LEGHTN); //将队列中的内容读至buf中
                    PACKET *msg = (PACKET *)malloc(MSG_LEGHTN);          //新建一个msg结构体地址，将消息传给消息处理函数
                    memcpy(msg, buf, MSG_LEGHTN);                        //将buf内容还原位结构体，保存到msg指向的结构体
                    if (s > 0)                                           //读到正确的消息
                    {
                        pthread_mutex_lock(&lock);
                        printf("[server ] 主线程正在上锁\n");
                        msg_option(msg, table, client_sockfd[j], &wdt);
                        pthread_mutex_unlock(&lock);
                    }
                    else if (0 == s) //客户端离线
                    {
                        close(client_sockfd[j]);                         //关闭客户端
                        int result = delete_fd(table, client_sockfd[j]); //先删除再把client_sockfd[j]置为-1
                        client_sockfd[j] = -1;                           //还原client_sockfd[j]
                    }
                    else
                    {
                        perror("[server ] read");
                        close(client_sockfd[j]);
                        client_sockfd[j] = -1;
                    }
                }
            }
            break;
        }
        }
    }
    return 0;
}
```

</details>
  
<details>
<summary>client1.c</summary>  

```C
#include "am5728.h"
#include "sock.h"

void *client_recv_thread(void *arg)
{
    int sock_recv; //存放socket_client函数运行的返回值，也是recv函数的第一个参数

    PACKET *send_msg = (PACKET *)malloc(MSG_LEGHTN);
    PACKET *recv_msg = (PACKET *)malloc(MSG_LEGHTN);

    sprintf(send_msg->head, "%s", "head");
    send_msg->cmd = 0;
    send_msg->wdt = 0;
    sprintf(send_msg->target_name, "%s", "null");
    sprintf(send_msg->source_name, "%s", "client1");
    sprintf(send_msg->msg, "%s", "null");
    char *buffer = (char *)malloc(MSG_LEGHTN);
    if ((sock_recv = client_socket()) < 0)
    {
        perror("[client1] connect failed!\n");
        return (void *)-1;
    }

    // printf("客户端的sock_cli值为：%d\n", sock_recv);
    memcpy(buffer, send_msg, MSG_LEGHTN);
    write(sock_recv, buffer, MSG_LEGHTN);

    while (1)
    {
        memset(recv_msg, '\0', MSG_LEGHTN);
        memset(buffer, '\0', MSG_LEGHTN);

        //recv(sock_recv, buffer, MSG_LEGHTN,0);  //让线程进入阻塞，此处不用read()
        ssize_t s = read(sock_recv, buffer, MSG_LEGHTN);
        if (s > 0)
        {
            memcpy(recv_msg, buffer, MSG_LEGHTN);
            //sleep(1);
            //对接收的数据操作
            printf("[client1] thread recv:msg=\"%s\"\n", recv_msg->msg);
        }
    }
    close(sock_recv);
    pthread_exit(0);
}

int main(int argc, char *argv[])
{
    pthread_t recv_thread;
    pthread_create(&recv_thread, NULL, client_recv_thread, NULL);
    printf("[client1] Creating recv thread...\n");
    sleep(1);
    int sock = client_socket();
    if (sock < 0)
    {
        perror("[client1] Connect failed");
        exit(1);
    }

    register_wdt(sock, "client1");

    while (get_other_process_pid("zhx_server") > 0)
    {
        // memcpy(buf, msg, MSG_LEGHTN);
        // write(sock, buf, MSG_LEGHTN);
        //sleep(4);
        transfer_msg(sock, "client2", "client1", "this message is from client1 to client2");
        sleep(10);
        send_wdt(sock,"client1");
    }

    close(sock);
    return 0;
}
```

</details>  
  
<details>
<summary>sock.h</summary>  

```C
#include "am5728.h"

#define CMD_RECV 0     //recv标志 0
#define CMD_WDT 1      //wdt标志 1
#define CMD_TRANSFER 2 //transfer标志 2
#define WDT_REGISTER 0 //注册wdt表
#define WDT_SIGNAL 1   //wdt发来的测试信号
//消息结构体
typedef struct
{
    char head[10];
    int cmd;
    int wdt;
    char target_name[10];
    char source_name[10];
    char msg[512];
} PACKET;
#define MSG_LEGHTN sizeof(PACKET) //PACKET结构体的长度，也是接收和发送指定的长度

#define CLI_NUM 10                //允许的客户端接收线程上限
//客户端进程与接收fd的表，结构体
typedef struct
{
    char client_name[CLI_NUM][10]; //存储客户端进程名的二维数组
    int client_recv_sock[CLI_NUM]; //存储客户端接收线程fd的数组
} CLIENT_TABLE;

//存放客户端看门狗计数的结构体
typedef struct
{
    int wdt_count[CLI_NUM];
    char wdt_client_name[CLI_NUM][10];
} WDT_STRUCT;

int get_other_process_pid(char *process_name);
int add_table(CLIENT_TABLE *table, char *client_name, int sock);
int search_fd(CLIENT_TABLE *table, char *client_name);
int delete_fd(CLIENT_TABLE *table, int sock);
int add_wdt(WDT_STRUCT *wdt, char *client_name);
int set_wdt_count_zero(WDT_STRUCT *wdt, char *client_name);
int wdt_count(WDT_STRUCT *wdt);
int server_socket(const char *_ip, int _port);
int msg_option(PACKET *msg, CLIENT_TABLE *table, int sock, WDT_STRUCT *wdt);

int client_socket();
int transfer_msg(int sock, char *target_name, char *source_name, char *content);
int register_wdt(int sock, char *source_name);
int send_wdt(int sock, char *source_name);
```

</details>  
  
<details>
<summary>am5728.h</summary>  

```C
#ifndef _AM5728_H
#define _AM5728_H

#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <string.h>
#include <sys/time.h>
#include <sys/select.h>
#include <signal.h>
#include <pthread.h>
#include <memory.h>

#endif
```

</details>  
