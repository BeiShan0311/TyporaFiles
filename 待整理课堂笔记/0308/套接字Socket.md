# Socket编程

## 广域网和局域网

局域网：一定范围和区域的私有网络。

广域网：连接不同地区进行计算机通信的远程公共网络。



## IP - `Internet Protrol` - 区分主机

IPv4和IPv6

弹性公网IP，由ISP运营商提供。

查看IP地址：

ifconfig - linux

ipconfig - windows





## 端口 - Port - 区分服务(软件)

unsigned short 16位整数 0 - 65535。

每一个服务对应一个端口号：

TCP：6

UDP：17

HTTP：80

SSH：22

FTP：21

HTTPS：443

LOCALHOST：



## Socket编程

### 字节序

主要有两种：

- 大端字节序 
  - 数据的低位字节存储在内存的高位地址。
  - 数据的高位字节存储在内存的地位地址。
  - 套接字通信一般是大端存储。
- 小端字节序
  - 数据的高位字节存储在内存的高位地址。
  - 数据的低位字节存储在内存的低位地址。
  - 常见PC机器一般是小端存储。

### 字节序转化函数

- htons - 将短整型short数据转换为大端存储

- htonl - 将长整型long数据转换为大端存储
- ntohs - 将短整型short数据转换为小端存储
- ntohl - 将长整型long数据转换为小端存储



### IP转换

- inet_pton - 将IP地址从文本输入转换为二进制。

```c
int inet_pton(int af, const char *src, void *dst);
```



### 套接字函数

#### socket函数

创建socket通信

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

### bind函数

将文件描述符与本地IP和端口进行绑定

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```



数据结构sockaddr：





### listen函数

给监听函数设置监听





### accept函数

接收客户端的连接

























































