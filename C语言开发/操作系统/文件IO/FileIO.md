# 文件IO常用函数

1. `open()`

   **函数作用：**

   打开文件并返回打开文件的文件描述符。

   **常用函数：**

   ```c
   int open(const char *pathname, int flags);
   int open(const char *pathname, int flags, mode_t mode);
   ```

   **函数参数：**

   `const char *pathname` 要打开的文件路径。

   `int flags` 打开文件的方式，以下参数都可以通过运算符`|`一起叠加使用。

   - `O_RDONLY`：只读打开文件。
   - `O_WRONLY`：只写打开文件。
   - `O_RDWR`：读写打开文件。
   - `O_CREAT`：如果文件不存在，则创建文件。
   - `O_EXCL`：与`O_CREAT`一起使用，如果文件存在，则返回错误。
   - `O_TRUNC`：如果文件存在且是只写或读写方式打开，则将其长度截断为0。
   - `O_APPEND`：在写入文件时追加到文件末尾。
   - `O_NONBLOCK`：以非阻塞模式打开文件。
   - `O_SYNC`：以同步写入模式打开文件。
   - `O_ASYNC`：启用异步IO操作。
   - `O_DIRECT`：直接IO，绕过内核缓冲区。

   `mode_t mode` 指定文件的访问权限，以下参数都可以通过运算符`|`一起叠加使用。

   - `S_IRWXU`：用户拥有读，写，执行权限（00700）

   - `S_IRUSR`：用户拥有读权限（00400）
   - `S_IWUSR`：用户拥有写权限（00200）
   - `S_IXUSR`：用户拥有执行权限（00100）
   - `S_IRWXG`：组拥有读，写，执行权限  （00070）
   - `S_IRGRP`：组拥有读权限（00040）
   - `S_IWGRP`：组拥有写权限（00020）
   - `S_IXGRP`：组拥有执行权限（00010）
   - `S_IRWXO`：其他用户拥有读，写，执行权限（00007）
   - `S_IROTH`：其他用户拥有读权限（00004）
   - `S_IWOTH`：其他用户拥有写权限（00002）
   - `S_IXOTH`：其他用户拥有执行权限（00001）

   **函数返回值：**

   - 打开成功返回一个`1-1024`的文件描述符。

   - 打开失败返回`-1`。

2. `close()`

   **函数作用：**

   关闭文件并释放文件描述符资源`fd`。

   **常用函数：**

   ```c
   int close(int fd);
   ```

   **函数参数：**

   `int fd` 需要关闭的文件的文件描述符。

   **函数返回值：**

   成功关闭返回`0`。

   当文件成功关闭时，文件描述符资源被回收，下次需要申请文件描述符资源时会自动分配最小的未被分配的文件描述符资源。

   关闭出错返回`-1` 。

3. `read()`

   **函数作用：**

   从文件`fd`读取`count`字节的文件数据到缓冲区，并返回读取到的字节数。

   **常用函数：**

   ```c
   ssize_t read(int fd, void *buf, size_t count);
   ```

   **函数参数：**

   `int fd` 需要读取的文件的文件描述符。

   `void *buf` 读取文件数据的缓冲区。

   `size_t count` 读取的字节数。

   **函数返回值：**

   成功返回读取的字节数。

   失败返回`-1`。

4. `write()`

   **函数作用：**

   向文件`fd`中写入`count`个字节的缓冲区数据。

   **常用函数：**

   ```c
   ssize_t write(int fd, const void *buf, size_t count);
   ```

   **函数参数：**

   `int fd` 需要写入的文件的文件描述符。

   `void *buf` 写入的文件数据的缓冲区。

   `size_t count` 写入的字节数。

   **函数返回值：**

   成功返回写入的字节数。

   失败返回`-1`。

5. `access()`

   **函数作用：**

   检查是否具有访问文件的特定权限。

   权限类型包括

   - `F_OK` 是否存在文件。
   - `R_OK` 是否具有文件的读取权限。
   - `W_OK` 是否具有文件的写入权限。
   - `X_OK` 是否具有文件的执行权限。

   **常用函数：**

   ```c
   int access(const char *pathname, int mode);
   ```

   **函数参数：**

   `const char *pathname` 文件路径。

   `int mode` 需要检查的文件权限类型。

   **函数返回值：**

   具备当前访问权限返回`0`。

   不具备当前权限返回`-1`。

6. `lseek()` 查询当前书签位置

   **函数作用：**

   查询当前的书签位置，即读取或写入文件的开始位置。

   **函数声明：**

   ```c
   off_t lseek(int fd, off_t offset, int whence);
   ```

   **函数参数：**

   `int fd` 要查询的文件描述符。

   `off_t offset` 偏移量。

   `int whence` 标签开始位置。

   - `SEEK_SET` 文件开始位置。
   - `SEEK_CUR` 当前标签的位置。
   - `SEEK_END` 文件的末尾位置。

   **函数返回值：**

   成功返回当前书签距离文件初始位置的字节数。

   失败返回`-1`。

7. `truncate` 以填充的方式打开文件

   **函数作用：**

   以填充的方式打开文件，多余的部分被删除，少于填充数量的部分以`0x0`填充。

   **函数声明：**

   ```c
   int truncate(const char *path, off_t length);
   ```

   **函数参数：**

   `const char *path`需要打开的文件路径。

   `off_t length`打开文件的填充字节数。

   **函数返回值：**

   打开成功返回`0`。

   打开失败返回`-1`。

8. `opendir()`打开目录

   **函数作用：**

   打开目录，使用绝对路径打开。

   **函数声明：**

   ```c
   DIR *opendir(const char *name);
   ```

   **函数参数：**

   `const char *name` 需要打开的路径。

   **函数返回值：**

   成功返回打开的目录指针。

   失败返回`NULL`。

9. `readdir()` 查看目录文件

   **函数作用：**

   读取目录中的文件。

   **函数声明：**

   ```c
   struct dirent *readdir(DIR *dirp);
   ```

   **函数参数：**

   `DIR *dirp` 需要读取的目录指针。

   **函数返回值：**

   读取到目录中的文件时，返回指向当前文件的指针。

   当读取到目录的末尾时，返回`NULL`。

   ```
   struct dirent
     {
   #ifndef __USE_FILE_OFFSET64
       __ino_t d_ino;
       __off_t d_off;
   #else
       __ino64_t d_ino;
       __off64_t d_off;
   #endif
       unsigned short int d_reclen;
       unsigned char d_type;
       char d_name[256];		/* We must not include limits.h! */
     };
   ```

   - `d_type` 文件类型。
     - `DT_UNKNOWN` (0)：表示目录项的类型未知。
     - `DT_FIFO` (1)：表示目录项是一个FIFO文件（也称为命名管道）。
     - `DT_CHR` (2)：表示目录项是一个字符设备文件。
     - `DT_DIR` (4)：表示目录项是一个目录。
     - `DT_BLK` (6)：表示目录项是一个块设备文件。
     - `DT_REG` (8)：表示目录项是一个普通文件。
     - `DT_LNK` (10)：表示目录项是一个符号链接。
     - `DT_SOCK` (12)：表示目录项是一个套接字。
   - `d_name`文件名称。

   





























































