```c
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <netinet/in.h>

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

    struct sockaddr_in localAddress;
    // 清除脏数据
    memset(&localAddress, 0, sizeof(localAddress));
    localAddress.sin_family = AF_INET; // IPv4
    localAddress.sin_port = htons(SERVER_PORT);

    const char *ip = "172.26.227.166";
    // 设置IP
    inet_pton(AF_INET, ip, &(localAddress.sin_addr.s_addr));

    socklen_t localAddressLength = sizeof(localAddress);

    // // 绑定端口IP
    // int ret = bind(sockfd, (const struct sockaddr *)&localAddress, localAddressLength);
    // if (ret == -1)
    // {
    //     perror("bind error");
    //     exit(-1);
    // }
    int connfd = connect(sockfd, (const struct sockaddr *)&localAddress, localAddressLength);
    if (connfd == -1)
    {
        perror("connet error");
        exit(-1);
    }

    printf("hello world\n");

    char readBuf[BUFFER_SIZE] = {0};
    int readBytes = 0;

    char writeBuf[BUFFER_SIZE] = {0};
    int writeSize = 0;
    while (1)
    {
        printf("write in a string:");
        scanf("%s", writeBuf);
        writeSize = write(sockfd, writeBuf, strlen(writeBuf) + 1);
        usleep(20);
        readBytes = read(sockfd, readBuf, BUFFER_SIZE);
        if (readBytes == -1)
        {
            perror("read error");
        }
        else if (readBytes == 0)
        {
            printf("no message\n");
        }
        else
        {
            printf("server return string:%s\n", readBuf);
        }
        sleep(2);
    }

#if 0
    char buffer[32] = {0};
    int readBytes = read(sockfd, buffer, sizeof(buffer));
    if (readBytes < 0)
    {
    }
    else if (readBytes == 0)
    {
    }
    else
    {
        printf("buffer:%s\n", buffer);
    }
#endif

    close(connfd);
    close(sockfd);
    return 0;
}
```

