```c
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <ctype.h>

#define SERVER_PORT 8080
#define BUFFER_SIZE 64

int main()
{
    // 绑定文件符与端口和本地IP连接
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1)
    {
        perror("socket error");
        exit(-1);
    }
    // 文件描述符最小是3
    printf("socketfd is %d\n", sockfd);

    // 设置端口复用
    int optval = 1;
    int ret = setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const void *)&optval, sizeof(optval));
    if (ret == -1)
    {
        perror("set socket port reuse error");
        exit(-1);
    }

    struct sockaddr_in localAddress;
    // 清除脏数据
    memset(&localAddress, 0, sizeof(localAddress));
    localAddress.sin_family = AF_INET; // IPv4
    localAddress.sin_port = htons(SERVER_PORT);
    localAddress.sin_addr.s_addr = INADDR_ANY; // 接收所有地址

    socklen_t localAddressLength = sizeof(localAddress);

    // 绑定端口IP
    ret = bind(sockfd, (const struct sockaddr *)&localAddress, localAddressLength);
    if (ret == -1)
    {
        perror("bind error");
        exit(-1);
    }

    // 监听套接字
    ret = listen(sockfd, 10);
    if (ret == -1)
    {
        perror("listen error");
        exit(-1);
    }

    // 等待并连接客户端请求，建立连接
    struct sockaddr_in clinetAddress;
    memset(&clinetAddress, 0, sizeof(clinetAddress));
    socklen_t clinetAddressLength = 0;

    // accept的返回值是和`客户端通信`的文件描述符
    // 阻塞函数：等待客户端连接
    int connfd = accept(sockfd, (struct sockaddr *)&clinetAddress, &clinetAddressLength);
    if (connfd == -1)
    {
        perror("accept error");
        exit(-1);
    }
    printf("connfd is %d\n", connfd);

    // 服务器接收客户端的信息并做出反馈
    // 服务器不允许宕机
    char readBuffer[BUFFER_SIZE] = {0};
    int readBytes = 0;

    char writeBuffer[BUFFER_SIZE] = { 0 };
    int writeBytes = 0;
    while (1)
    {
        readBytes = read(connfd, readBuffer, BUFFER_SIZE - 1);
        if (readBytes == -1)
        {
            // 读通信句柄失败
            perror("read error");
        }
        else if (readBytes == 0)
        {
            // 客户端下线
            // 读取的字节数为零
            printf("no clinet\n");
            break;
        }
        else
        {
            // 打印读取到的信息
            printf("readBytes:%d\tbuffer:%s\n", readBytes, readBuffer);
            for (int idx = 0; idx < readBytes; idx++)
            {
                writeBuffer[idx] = toupper(readBuffer[idx]);
            }
            writeBytes = write(connfd, writeBuffer, readBytes);
        }
    }
    // strcpy(buffer, "偶像");
    // write(connfd, buffer, sizeof(buffer) - 1);

    // 回收资源
    close(sockfd);
    close(connfd);

    return 0;
}
```

