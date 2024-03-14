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
#include <sys/time.h>
#include <sys/select.h>

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

    int maxfd = sockfd;
    fd_set readfds;
    // 清空文件描述符集合
    FD_ZERO(&readfds);
    // 将sockfd放入readfds读集合中
    FD_SET(sockfd, &readfds);

    // 等待并连接客户端请求，建立连接
    struct sockaddr_in clinetAddress;
    memset(&clinetAddress, 0, sizeof(clinetAddress));
    socklen_t clinetAddressLength = 0;

    // 备份传入参数readfds
    // 防止传出参数改变readfds集合中的元素数量，导致后续内核监测的集合内容有误。

    while (1)
    {
        // tmpReadfds表示就绪的IO返回的通信句柄
        fd_set tmpReadfds = readfds;

        // 在监听读集合是否有读时间，监听写集合是否有写事件
        int res = select(maxfd + 1, &tmpReadfds, NULL, NULL, NULL);
        if (res == -1)
        {
            perror("select error");
            exit(-1);
        }

        // 到这里，tmpReadfds有当前可读的事件集合。
        // 判断当前传出的集合中是否有sockfd
        if (FD_ISSET(sockfd, &tmpReadfds))
        {
            // 有客户端需要和服务器通信
            // 处理连接
            int connfd = accept(sockfd, (struct sockaddr *)&clinetAddress, &clinetAddressLength);
            if (connfd == -1)
            {
                perror("accept error");
                exit(-1);
            }
            printf("accept fd:%d\n", connfd);

            // 将connfd加入到readfds里面，监听connnfd的读事件，否则select就只能监听sockfd的内容
            FD_SET(connfd, &readfds);
            // FD_SET(connfd, &tmpReadfds);

            // 更新最大文件描述符
            maxfd = maxfd > connfd ? maxfd : connfd;
            printf("maxfd:%d\n", maxfd);
        }

        // 程序到这里，说明有人与服务器通信
        // 通信句柄从4开始
        for (int idx = 4; idx < maxfd + 1; idx++)
        {
            printf("read %d\n", idx);

            // 判断当前的idx通信句柄是否返回的传出参数在读集合中
            if (FD_ISSET(idx, &tmpReadfds))
            {
                printf("in read\n");
                // 程序到这里,idx就是通信句柄connfd

                int connfd = idx;

                // 服务器接收客户端的信息并做出反馈
                // 服务器不允许宕机
                char readBuffer[BUFFER_SIZE] = {0};
                int readBytes = 0;

                char writeBuffer[BUFFER_SIZE] = {0};
                int writeBytes = 0;

                readBytes = read(connfd, readBuffer, BUFFER_SIZE - 1);
                if (readBytes == -1)
                {
                    // 读通信句柄失败
                    perror("read error");
                    // 读取失败，撤掉读集合中的相应句柄
                    FD_CLR(connfd, &readfds);
                }
                else if (readBytes == 0)
                {
                    // 客户端下线
                    // 读取的字节数为零
                    printf("no clinet\n");
                    // 读取完毕，撤掉读集合中的相应句柄
                    FD_CLR(connfd, &readfds);
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
                // strcpy(buffer, "偶像");
                // write(connfd, buffer, sizeof(buffer) - 1);
            }
        }

        // readfds：内核检测该集合内的文件描述符是否可读，内核检测该集合中的IO是否可读
        // readfds是一个传出参数，传出的是委托内核监听的事件
        // 传入：文件描述符3,4,5,6,7,8
        // 传出：3,5,7有读事件过来
        //      如果读事件中有3，说明有客户端跟服务器连接，应该调用accept函数，并将接收到的文件描述符加入到读集合中
        //      如果读事件中有5，说明5号客户端需要跟服务器通信。

        // 程序到这里，表示有客户端连接，触发了读取事件
        printf("res:%d\n", res);
    }

#if 0
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

    /*

    */
    char ip[BUFFER_SIZE] = {0};
    inet_ntop(AF_INET, &clinetAddress.sin_addr.s_addr, ip, sizeof(ip));
    int clientPort = ntohs(clinetAddress.sin_port);
    printf("clientIP:%s\tclientPort:%d\n", ip, clientPort);

    // 服务器接收客户端的信息并做出反馈
    // 服务器不允许宕机
    char readBuffer[BUFFER_SIZE] = {0};
    int readBytes = 0;

    char writeBuffer[BUFFER_SIZE] = {0};
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
#endif

    return 0;
}
```

