---
title: 基于TCP/IP网络socket通信
date: 2019-08-09 12:51:01
index_img: /img/index_img/socket_logo.png
tags: 
    - Linux
    - Net
categories: 
    - OS
---

网络传输文件现有的工具为ssh或者tcp指令，ssh中的scp指令如下：

<!-- more -->

```
$ scp file name@IP:file_path
```

测试在百兆带宽的情况下，可以达到5~6MB/s。  
SSH的用户层工作原理，除去登陆密码以及公钥或者私钥的建立，底层的工作模式也需要考虑。 

- [网络设备驱动](#网络设备驱动)
  - [内核配置](#内核配置)
  - [设备树配置](#设备树配置)
- [传输协议](#传输协议)
  - [数据传输协议](#数据传输协议)
  - [文件传输协议](#文件传输协议)
  - [测试与改进](#测试与改进)
  - [配置数据包](#配置数据包)
- [应用程序](#应用程序)
  - [服务端](#服务端)
  - [客户端](#客户端)


## 网络设备驱动  

网络层次七层又可以分为应用层（应用层、表示层与会话层）、传输层、网络层、数据链路层以及物理层。  
![](/img/tcpip/网络层次图.png)  
网络设备驱动负责将数据包写入网络或者从网络中读取数据包，从而完成上层的请求，与其他接口开发的不同是  
* 网络设备不在/dev下创建设备文件；  
* 底层采用中断的工作方式，并将中断传递给上层应用程序（这与UART类似）。 
   
### 内核配置  
—> Networking support > Networking options

![](/img/tcpip/kernel1.png)

网络功能选择：在内核中配置EtherNet网口支持的功能。   
  
| 内核选项                          | 含义                 |
| --------------------------------- | -------------------- |
| Packet socket                     | 支持Socket通信       |
| Unix domain sockets               | Socket进程间通信     |
| TCP/IP networking                 | TCP/TP网络协议       |
| IP:multicasting                   | 组播，多目标发送     |
| IP:advanced router                | 高级路由，流量控制   |
| IP:kernel level autoconfiguration | 内核启动时自动获取IP |
| IP:DHCP suppo                     | DHCP 获取动态IP协议  |
| IP:BOOTP support                  | DHCP的前身           |
| IP:RARP support                   | 反向地址转换协议     |


— > Device Drivers > Network device support ─ Ethernet driver support


![](/img/tcpip/kernel2.png)
配置芯片驱动：选择对应厂商的芯片，若没有需要写适配的驱动源码！  
### 设备树配置  
直接引用dra7.dtsi设备树源文件中的以太网寄存器配置。  
``` dts
&davinci_mdio {
    phy0: ethernet-phy@1 {
        reg = <1>;
    };
    phy1: ethernet-phy@2 {
        reg = <2>;
    };
};
```
## 传输协议
### 数据传输协议
![](/img/tcpip/协议1.png)

* 服务端accept()函数为阻塞函数，主进程执行至accept后进程阻塞，直到客户端发出连接请求；  
* 接收请求后使用fork()函数创建子进程，这样就可以通过不同的子进程连接多个客户端；  
* Send()函数将指定长度数据发送至内存缓冲队列，发送后等待对方确认（客户端确认和服务端重新发送在底层完成）；  
* recv()函数也是阻塞函数，在队列中没有数据时，recv()进程进入阻塞，recv()成功读取则返回读取长度。  
   
### 文件传输协议
![](/img/tcpip/协议2.png)  
* 缓冲区长度有限，若客户端不执行recv()则服务端发送一定量的数据包后停止发送；  
* 客户端需要加入文件末尾判断的语句；  
* 若 T1>T2，则recv()函数并不能一次读取设定长度的数据，即有多少读多少，增加了客户端读取循环的次数，降低了传输效率；
    
### 测试与改进  
实际测试：文件长度50160000，约50MB；网络带宽100Mb，上限约12.5MB/s。设每次发送10000Byte，测得：服务端发送5016个数据包，客户端每次接收1000--10000长度不等的包，实际接收了13000个数据包，最终传输速率为2MB/s。  
改进：每次recv()先与对方确认包的长度，并将内容读满再返回，配置MSG_WAITALL。测试结果：服务端发送5016个数据包，客户端接收了5016个数据包，最终传输速率为10--12MB/s。  
### 配置数据包
![](/img/tcpip/协议3.png)
* 在应用层编写实现类似底层包的组帧结构的协议，可以完成多个文件的收发；  
* 由于组包与解包的加入，速度略有牺牲，测试速度约为4~5MB/s。  

## 应用程序
### 服务端
<details>
<summary>server.c</summary>  

```CPP
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <time.h>

#define PORT 8888
#define QUEUE_SIZE 10
#define BUFFER_SIZE 10000

//传进来的sockfd，就是互相建立好连接之后的socket文件描述符
//通过这个sockfd，可以完成 [服务端]<--->[客户端] 互相收发数据
void transfer_file(int sockfd)
{
    FILE *fp;
    int ch;
    time_t t_start, t_end;

    int send_num;
    long length = 0;
    //char name[LEN];    // storage for output filename
    int count = 0;

    fp = fopen("test.txt", "r");
    if (fp == NULL)
    {
        fprintf(stderr, "couldn't open the file \n");
        exit(EXIT_FAILURE);
    }

    fseek(fp, 0L, SEEK_END); //将文件指针移到末尾
    length = ftell(fp);    //获取文件长度
    printf("%ld\n", length);

    char str[10];
    sprintf(str, "%ld", length);   //将文件长度转换为字符串str
    send(sockfd, str, sizeof(str), 0);  //将文件长度发送给client

    char buffer1[BUFFER_SIZE];
    recv(sockfd, buffer1, sizeof(buffer1), 0);    //等待对方读取完长度后返回“ready”
    if (strcmp(buffer1, "ready") == 0)
    {
        fseek(fp, 0L, SEEK_SET);          //文件指针移到开头
        // char buffer[1];
        // copy data
        // while ((ch = getc(fp)) != EOF)
        // {
        //     buffer[0] = ch;
        //     send(sockfd, buffer, 1, 0);
        // }
        t_start = time(NULL);          //获取开始时间
        char buffer[BUFFER_SIZE];       //新建缓存
        send_num = 0;                  //计算发送包的个数
        while (fread(buffer, sizeof(buffer), 1, fp))    //fread 读文件到缓存，读到末尾会EOF（-1）
        {
            //printf("send%d %ld\n",send_num,sizeof(buffer));
            send(sockfd, buffer, sizeof(buffer), 0);      //发送一个文件包
            send_num++;
            //break;
        }
        t_end = time(NULL);  //获取结束时间
        printf("send %d times\n", send_num);     
        printf("speed: %.02f MB/s\n", length / 1024 / 1024 / difftime(t_end, t_start));
        if (fclose(fp) != 0)        //关闭文件
            fprintf(stderr, "Error in closing files\n");
    }
    else
    {
        printf("cannot get start!\n");
    }
}

int str_echo(int sockfd)     //回环函数
{
    char buffer[BUFFER_SIZE];     //新建内存缓冲区
    pid_t pid = getpid();
    while (1)
    {
        memset(buffer, 0, sizeof(buffer));    //将内存缓冲区清0，初始化
        int len = recv(sockfd, buffer, sizeof(buffer), 0);
        printf("pid:%d receive:\n", pid);
        fputs(buffer, stdout);
        if (strcmp(buffer, "exit\n") == 0)
        {
            printf("child process: %d exited.\n", pid);
            printf("the server shutdown.\n");
            break;
        }
        if (strcmp(buffer, "send\n") == 0)
        {
            printf("start transfer.\n");
            transfer_file(sockfd);
            printf("transfer end.\n");
            //sleep(10);
            //printf("the server shutdown.\n");
            continue;
        }
        send(sockfd, buffer, len, 0);
    }
    close(sockfd);
    return -1;
}

int main(int argc, char **argv)
{
    //定义IPV4的TCP连接的套接字描述符
    int server_sockfd = socket(AF_INET, SOCK_STREAM, 0);

    //定义sockaddr_in
    struct sockaddr_in server_sockaddr;
    server_sockaddr.sin_family = AF_INET;
    server_sockaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_sockaddr.sin_port = htons(PORT);

    //bind成功返回0，出错返回-1
    if (bind(server_sockfd, (struct sockaddr *)&server_sockaddr, sizeof(server_sockaddr)) == -1)
    {
        perror("bind");
        exit(1); //1为异常退出
    }
    printf("bind success.\n");

    //listen成功返回0，出错返回-1，允许同时帧听的连接数为QUEUE_SIZE
    if (listen(server_sockfd, QUEUE_SIZE) == -1)
    {
        perror("listen");
        exit(1);
    }
    printf("listen success.\n");

    for (;;)
    {
        struct sockaddr_in client_addr;
        socklen_t length = sizeof(client_addr);
        //进程阻塞在accept上，成功返回非负描述字，出错返回-1
        int conn = accept(server_sockfd, (struct sockaddr *)&client_addr, &length);
        if (conn < 0)
        {
            perror("connect");
            exit(1);
        }
        printf("new client accepted.\n");

        pid_t childid;
        if (childid = fork() == 0) //子进程
        {
            printf("child process: %d created.\n", getpid());
            close(server_sockfd);   //在子进程中关闭监听
            if (str_echo(conn) < 0) //处理监听的连接
            {
                exit(0);     //对方发送exit，返回-1，关闭子进程，主进程继续accep
            }
        } 
    }

    printf("closed.\n");
    close(server_sockfd);
    printf("end\n");
    exit(0);
    return 0;
}
```

</details>  
   
客户端编译后执行：  
```
$ ifconfig eth0 192.168.111.101
$ ./server
```
  
### 客户端
<details>
<summary>client.c</summary>  

```CPP
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <time.h>
#define PORT 8888
#define BUFFER_SIZE 10000

int main(int argc, char **argv)
{
    if (argc != 2)
    {
        printf("usage: client IP \n");
        exit(0);
    }

    //定义IPV4的TCP连接的套接字描述符
    int sock_cli = socket(AF_INET, SOCK_STREAM, 0);
    FILE *out;
    time_t t_start, t_end;
    long i, receive_length, file_length, length;
    int receive_num;
    //定义sockaddr_in
    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = inet_addr(argv[1]);
    servaddr.sin_port = htons(PORT); //服务器端口

    //连接服务器，成功返回0，错误返回-1
    if (connect(sock_cli, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0)
    {
        perror("connect");
        exit(1);
    }
    printf("connect server(IP:%s).\n", argv[1]);

    char sendbuf[BUFFER_SIZE];
    char recvbuf[BUFFER_SIZE];
    memset(sendbuf, 0, sizeof(sendbuf));
    memset(recvbuf, 0, sizeof(recvbuf));

    //客户端将控制台输入的信息发送给服务器端，服务器原样返回信息
    while (fgets(sendbuf, sizeof(sendbuf), stdin) != NULL)   //捕获命令行的字符串到sendbuf
    {

        memset(recvbuf, 0, sizeof(recvbuf));
        send(sock_cli, sendbuf, strlen(sendbuf), 0); ///发送
        if (strcmp(sendbuf, "exit\n") == 0)
        {
            printf("client exited.\n");
            break;
        }
        if (strcmp(sendbuf, "send\n") == 0)
        {
            if ((out = fopen("out.txt", "w")) == NULL)
            { // open file for writing
                fprintf(stderr, "Can't create output file.\n");
                exit(3);
            }

            recv(sock_cli, recvbuf, sizeof(recvbuf), 0);
            file_length = atoi(recvbuf);
            memset(recvbuf, 0, sizeof(recvbuf)); //获取文件长度

            memset(sendbuf, 0, sizeof(sendbuf));
            send(sock_cli, "ready", 5, 0); //发送就绪

            printf("receive file start.\n");

            t_start = time(NULL);
            length = 0;
            receive_num = 0;
            while (length < file_length)
            {
                receive_length = recv(sock_cli, recvbuf, sizeof(recvbuf), MSG_WAITALL); ///接收 //MSG_WAITALL是强行等待缓冲满再结束
                //sleep(1);
                //printf("%d length %d\n", receive_num,receive_length);

                //fputs(recvbuf, stdout);
                //printf("run dot1\n");
                //for (i = 0; i < receive_length; i++)
                //{
                //putc(recvbuf[i], out);
                //}
                fprintf(out, "%s", recvbuf);
                //memset(sendbuf, 0, sizeof(sendbuf));
                memset(recvbuf, 0, sizeof(recvbuf));
                //printf("run\n");
                length = length + receive_length;
                receive_num++;
                //break;
            }
            printf("receive %d times\n", receive_num);
            if (fclose(out) != 0)
                fprintf(stderr, "Error in closing files\n");
            t_end = time(NULL);
            printf("time : %.2f s\n", difftime(t_end, t_start));
            printf("speed: %.02f MB/s\n", file_length / 1024 / 1024 / difftime(t_end, t_start));
            printf("receive file end.\n");
            continue;
        }
        printf("client receive:\n");
        recv(sock_cli, recvbuf, sizeof(recvbuf), 0); ///接收
        fputs(recvbuf, stdout);
        memset(sendbuf, 0, sizeof(sendbuf));
    }

    close(sock_cli);
    return 0;
}
```

</details>
    
服务端编译后执行：  
```
$ ifconfig eth0 192.168.111.100
$ ./client 192.168.111.101
$ mesg   //对方回环
$ send   //对方发送文件
$ exit   //对方关闭服务子进程，客户端退出
```





