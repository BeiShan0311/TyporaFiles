# Socket通信

## IP地址`Internet Protrol` - 区分通信的主机

查看IP地址：

ifconfig - linux

ipconfig - windows



## 端口 - Port - 区分通信的服务(软件)

端口号定义为一个unsigned short 16位整数，表示范围为0 - 65535。

每一个服务对应一个端口号：

TCP：6

UDP：17

HTTP：80

SSH：22

FTP：21

HTTPS：443

LOCALHOST：



## Socket网络编程

#### 字节序

主要有两种：

- 大端字节序 
  - 数据的低位字节存储在内存的高位地址。
  - 数据的高位字节存储在内存的地位地址。
  - 套接字通信一般是大端存储。
- 小端字节序
  - 数据的高位字节存储在内存的高位地址。
  - 数据的低位字节存储在内存的低位地址。
  - 常见PC机器一般是小端存储。

#### 字节序转化函数

- htons - 将短整型short数据转换为大端存储

- htonl - 将长整型long数据转换为大端存储
- ntohs - 将短整型short数据转换为小端存储
- ntohl - 将长整型long数据转换为小端存储

#### IP转换函数

- inet_pton - 将IP地址从文本输入转换为二进制。

```c
int inet_pton(int af, const char *src, void *dst);
```

`af` 指定地址族。AF_INET - IPv4地址族，AF_INET6 - IPv6地址族。

`src` 指向需要转换的字符串指针。

`dst` 传出参数，转换后的二进制IP地址。



- inet_ntop - 将二进制IP地址转换为点分十进制字符串。

```c
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```

`af` 指定地址族。AF_INET - IPv4地址族，AF_INET6 - IPv6地址族。

`src` 指向需要转换的二进制格式的IP地址。

`dst` 传出参数，转换后的点分十进制IP地址。

`size` 指定`dst`缓冲区的大小。

#### 套接字函数

##### socket函数 - 创建socket通信文件

```c
int socket(int domain, int type, int protocol);
```

- domain - 使用的地址族协议
  - AF_INET - IPv4协议
  - AF_INET6 - IPv6协议
- type - 传输的文件类型
  - stream - 流式文件，tcp使用
  - dgram - 报式文件， udp使用
- protocol - 一般使用默认的协议

##### bind函数 - 将建立的socket文件描述符与指定的IP和端口号进行绑定

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- **sockfd**：服务器套接字文件描述符。它标识了将要分配地址的套接字。AF_INET用于IPv4，AF_INET6用于IPv6。
- **addr**：是一个指向要分配的地址结构的指针，这个结构可以是`struct sockaddr`或其派生类型。`struct sockaddr_in`用于IPv4，`struct sockaddr_in6`用于IPv6。
- **addrlen**：是`addr`结构的大小（以字节为单位）。它表示在`addr`参数指向的地址结构中实际使用的字节数。

- `struct sockaddr`数据结构
  - **sin_family**：指定地址族，通常设置为 AF_INET，表示 IPv4 地址族。
  - **sin_port**：16 位的端口号，用于标识目标端口。需要注意的是，端口号以网络字节序（大端序）表示。
  - **sin_addr**：IPv4 地址信息，通常使用 `struct in_addr` 结构体进行表示。这个结构体的成员是 `s_addr`，它存储了 32 位的 IPv4 地址，同样以网络字节序存储。一般需要使用`inet_ntop`和函数`inet_pton`来转换需要的字节序。

##### listen函数 - 设置监听函数，接收传入的连接请求

```c
int listen(int sockfd, int backlog);
```

- **sockfd**：服务器套接字文件描述符。
- **backlog**：指定连接请求队列的最大长度。

##### accept函数 - 连接一个客户端请求

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

- **sockfd**：服务器套接字文件描述符。
- **addr**：是一个指向存储远程主机地址信息的结构体的指针，通常是 `struct sockaddr` 或其派生类型（例如 `struct sockaddr_in`）。这个参数是可选的，如果不关心远程主机的地址信息，可以传入 `NULL`。
- **addrlen**：是一个指向存储 `addr` 结构体长度的整数指针。同样，如果不关心远程主机地址信息，可以传入 `NULL`。

##### connect函数 - 从客户端建立与服务器的连接

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- **sockfd**：是客户端套接字的文件描述符。
- **addr**：是一个指向目标服务器地址结构体的指针，通常是 `struct sockaddr` 或其派生类型（`struct sockaddr_in`）。该参数指定了服务器的地址信息，包括 IP 地址和端口号等。
- **addrlen**：是地址结构体的长度，以字节为单位。



































