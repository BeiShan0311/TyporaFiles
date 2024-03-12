什么是套接字？

套接字有插头的意思，可以引申为网络连接。是计算机进行连接因特网的工具。

如何进行简单的socket通信？

进行socket网络通信，需要一下几个过程：

在服务端：

1. 使用socket函数创建一个套接字，在linux操作系统中，这个函数会返回一个int类型的文件描述符，表示一个socket套接字文件。
   - `domain` 

```c
int socket(int domain, int type, int protocol);
```



2. 使用bind函数绑定通信的IP地址和端口号。

```c
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
```



3. 调用listen函数将当前的socket文件描述符代表的socket文件转为监听模式。

```c
int listen(int sockfd, int backlog);
```



4. 当listen函数接收到连接请求时，需要调用accept函数接收连接请求，并接收对方传输的数据信息，即一个socket文件。

在客户端：

1. 