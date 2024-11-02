## Socket网络通信过程

服务端Server：

1. 初始化Socket，得到文件描述符
2. 服务端调用bind函数，将socket绑定再指定的IP地址和端口；
3. 服务端调用listen函数，进行监听
4. 服务端调用accept函数，等待客户端连接
5. 客户端调用connect函数，向服务端的地址和端口发起连接请求
6. 服务端的accept返回用于传输的socket的文件描述符

### listen函数有什么参数，以及它的意义

```c++
#include<sys/socket.h>

int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen); //文件描述符，myaddr对应的ip、端口，addrlen是socket的长度
//成功时返回0，失败时返回-1

int listen(int sockfd, int backlog);// 成功返回0， 失败返回-1
//sockfd为文件描述符，backlog现在通常认为是accept队列
// listen函数将socket转化为可接受连接状态
```

accpet 系统调用并不参与 TCP 三次握手过程，它只是负责从 TCP 全连接队列取出一个已经建立连接的 socket，用户层通过 accpet 系统调用拿到了已经建立连接的 socket，就可以对该 socket 进行读写操作了。

![3](D:\Desktop\3.webp)





## socketIO多路复用

1. 什么是IO多路复用
2. IO多路复用解决什么问题
3. 目前有哪些IO多路复用的方案
4. 具体怎么用和不同多路复用方案的优缺点

### 1. 什么是IO多路复用

单线程或单进程同时检测若干个文件描述符是否可以执行IO操作的能力

阻塞IO

当我们发起一次IO操作后一直等待成功或失败之后才返回，在这期间程序不能做其它的事情。阻塞IO操作只能对单个文件描述符进行操作

非阻塞IO

通过对文件描述符设置O_NONBLOCK flag来指定该文件描述符的IO操作为非阻塞。非阻塞IO通常发生在一个for循环当中，因为每次进行IO操作时要么IO操作成功，要么当IO操作会阻塞时返回错误EWOULDBLOCK/EAGAIN，然后再根据需要进行下一次的for循环操作，这种类似轮询的方式会浪费很多不必要的CPU资源，是一种糟糕的设计。

IO多路复用

服务器端采用单线程通过select/epoll等系统调用获取fd列表，遍历有事件的fd进行accept/recv/send，使其能支持更多的并发连接请求

```c++
fds = [listen_fd]
// 伪代码描述
while(1) {
  // 通过内核获取有读写事件发生的fd，只要有一个则返回，无则阻塞
  // 整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，accept/recv是不会阻塞
  for (fd in select(fds)) {
    if (fd == listen_fd) {
        client_fd = accept(listen_fd)
        fds.append(client_fd)
    } elseif (len = recv(fd) && len != -1) { 
      // logic
    }
  }  
}

```

select采用的是线性扫描，采用轮询的方法，效率较低

select和poll只会通知用户进程有Socket就绪，但不确定具体是哪个Socket，需要用户进程逐个遍历Socket来确认。epoll则会在通知用户进程Socket就绪的同时，把已就绪的Socket写入用户空间。



## Reactor模型和Proactor模型

### 传统IO模型

