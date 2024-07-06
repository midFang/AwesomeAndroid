## 文件操作相关 Api

- 文件打开与关闭：
  
  - fopen()：打开一个文件，返回文件指针。
  - fclose()：关闭一个已打开的文件

- 读写文件：
  
  - rewind() 是 C/C++ 语言中的一个标准库函数，其功能是将文件指针移动到文件的开头
  - fgetc()：从文件中读取一个字符
  - fgets()：从文件中读取一行文本
  - fputc()：将一个字符写入文件
  - fputs()：将一个字符串写入文件
  - fprintf()：格式化输出到文件

- 二进制文件读写：
  
  - fread()：从文件中读取二进制数据
  - fwrite()：向文件中写入二进制数据

- 文件指针操作：
  
  - fseek()：设置文件指针的位置
  - ftell()：获取当前文件指针的位置

- 文件状态查询与设置：
  
  - feof()：检查是否已到达文件末尾。 file_end_of
  - ferror()：检查是否有文件操作错误。
  - clearerr()：清除文件错误标志。


## 打开文件,读文件
```c
void main()
{
    // 打开文件
    char *filePath = "simple.txt";
    // r 可读, w, 可写, rw, 可读可写
    // 当 mode 为w的时候,可写,不存在将会创建一个新的文件
    FILE *file = fopen(filePath, "rw");

    if (!file)
    {
        printf("文件不存在\n");
        return;
    }

    // 缓冲区
    char buffer[10];

    while (fgets(buffer, 10, file)) // 从文件中读取一行文本
    {
        printf("%s", buffer);
    }
    fclose(file); // 打开了文件, 必须 fclose, 关闭资源占用
}
```

## 写入文件

```c
void main()
{
    char *filePath = "simple.txt";
    FILE *file = fopen(filePath, "w"); // w模式: 以写入模式打开文件。如果文件不存在，则会创建一个新的空文件；如果文件存在，则会清空文件，并从头开始写入

    // 将文件指针偏移至文件末尾
    fseek(file, 0, SEEK_END); // 设置文件指针的位置。

    fputs("write new line", file); // 将一个字符串写入文件

    fclose(file);
}
```

## 文件模式

```c
“r”：以只读模式打开文件。文件必须存在，否则打开失败。
“w”：以写入模式打开文件。如果文件不存在，则会创建一个新的空文件；如果文件存在，则会清空文件，并从头开始写入。
“a”：以追加模式打开文件。如果文件不存在，则会创建一个新的空文件；如果文件存在，则会将内容追加到文件末尾，而不会清空原有的内容。
“r+”：以读写模式打开文件。文件必须存在，否则打开失败。可以对文件进行读取和写入操作。
“w+”：以读写模式打开文件。如果文件不存在，则会创建一个新的空文件；如果文件存在，则会清空文件，并从头开始读写。可以对文件进行读取和写入操作。
“a+”：以读写模式打开文件。如果文件不存在，则会创建一个新的空文件；如果文件存在，则会将内容追加到文件末尾，而不会清空原有的内容。可以对文件进行读取和写入操作。
“b”：以二进制模式打开文件。可与上述模式组合使用，如"rb"、"wb+"等。在某些平台上，此模式与文本模式没有明显区别。但是在其他平台上（如Windows），使用二进制模式打开文件可能会导致在读取和写入文件时执行不同的换行符转换。
```

## 文件追加
```c
void main()
{
    char *filePath = "simple.txt";
    FILE *file = fopen(filePath, "a");

    // 会从最后一行追加写入到文件中
    fputs("\nwrite new line", file);

    fclose(file);
}
```

## 文件复制，当做二进制文件来操作
```c
void main()
{
    FILE *file = fopen("simple.txt", "r+");
    FILE *copy_file = fopen("simple_copy.txt", "w+");

    if (!file && !copy_file)
    {
        printf("文件操作失败");
        return;
    }

    // 用 int 或者用 char 都可以,它代表的只是字节的长度
    int buffer[512]; // 读取字节数为 int 4字节, 所以4*512=2048
    int len = 0;
    while ((len = fread(buffer, sizeof(int), 512, file)) != 0)
    {
        fwrite(buffer, sizeof(int), 512, copy_file);
    }

    // 记得关闭文件,释放系统资源
    fclose(file);
    fclose(copy_file);
}
```

## 获取文件的大小
> 获取文件大小,其实就是指针偏移量
```c
void main()
{
    // file 返回的就是头指针的位置
    FILE *file = fopen("simple.txt", "r+");

    /**
     * SEEK_SET（开头）, SEEK_CUR（当前）, SEEK_END（移动到最后）
     * fseek(file, 0, SEEK_SET)，将文件指针移动到文件开头。
     * fseek(file, 0, SEEK_END)，将文件指针移动到文件末尾。
     * fseek(file, offset, SEEK_CUR)，将文件指针从当前位置向前或向后移动 offset 个字节。
     * fseek(file, offset, SEEK_SET)，将文件指针移动到离文件开头 offset 个字节的位置。
     * fseek(file, -offset, SEEK_END)，将文件指针移动到离文件末尾 offset 个字节的位置。
     */

    // 移动到最后的位置
    fseek(file, 0, SEEK_END);

    // 统计偏移量
    long file_size = ftell(file);

    printf("文件大小：%ld", file_size);

    fclose(file);
}
```

## 文件加密
> 文件加密,就是对一个文件进行破坏,让其无法再读取里面的内容, 解密就是相反的操作
```c
void main()
{
    FILE *file = fopen("image.JPG", "r+");
    // 创建一个新的文件, 不存在会创建, 存在则重新创建
    FILE *copy_file = fopen("image_en.JPG", "w+");

    // 文件加密,就是对一个文件进行破坏,让其无法再读取里面的内容
    // 将文件每一个字节都取出来, 进行算法的操作 (这里做异或操作)

    // 10^5 异或  加密过程
    //  1010
    // ^0101
    //  1111  加密

    // 解密 同样的去异或 5
    //  1111
    // ^0101
    //  1010    解密后 10
    int c;
    while ((c = fgetc(file)) != EOF) // 从文件中读取一个字符。
    {
        // 写入一个字符
        fputc(c ^ 5, copy_file);
    }

    fclose(file);
    fclose(copy_file);
}
```

## 文件解密

```c
void main()
{
    FILE *en_file = fopen("image_en.JPG", "r+");
    FILE *de_file = fopen("image_dc.JPG", "w+");

    int c;
    while ((c = fgetc(en_file)) != EOF)
    {
        fputc(c ^ 5, de_file);
    }

    fclose(en_file);
    fclose(de_file);

    /**
     * 问：加密就是对原有文件进行破坏，但是我每个字节都 & 一个数，这样文件是不是会变大了，我能不能直接修改其中的一个字节呢 ？ 
     * 答：理论上是支持的，但是实际运行测试的情况是无法支持，可能是这样不具备算法的完整性，容易被破解
     */
}
```

## 使用密码进行加密
```c
void main()
{
    FILE *file = fopen("image.JPG", "r+");
    FILE *pwd_file = fopen("image_pwd.JPG", "w+");

    char *pwd = "0928332";
    int pwd_len = strlen(pwd);
    int index = 0;
    printf("test8-index=%d\n", index);
    int c;
    while ((c = fgetc(file)) != EOF)
    {
        // 让每个字节都异或不一样的密码字符（力度加强） pwd[index % pwd_len]
        fputc(c ^ pwd[index % pwd_len], pwd_file);
        index++;
    }

    fclose(file);
    fclose(pwd_file);
}
```

## 使用密码进行解密
```c
void main()
{

    FILE *pwd_file = fopen("image_pwd.JPG", "r+");
    FILE *file_de = fopen("image_pwd_de.JPG", "w+");

    char *pwd = "0928332"; // 轮流进行^操作
    int len = strlen(pwd);
    int index;
    printf("test9-index=%d\n", index);
  
    int c;
    while ((c = fgetc(pwd_file)) != EOF)
    {
        fputc(c ^ pwd[index % len], file_de);
        index++;
    }

    fclose(file_de);
    fclose(pwd_file);
}
```

## 文件的切割
> 可以理解为解压缩
```c
int getFileSize(FILE *file)
{

    // 记录当前的指针
    int cur_point = ftell(file);
    fseek(file, 0, SEEK_END);

    // 获取当前文件的指针位置
    int size = ftell(file);

    // 需要特别注意, 这里传递的是文件的指针到这个方法中, 如果操作了指针的偏移, 需要恢复回去, 不然操作文件可能会出现未知的异常
    fseek(file, cur_point, SEEK_SET);

    return size;
}

void main()
{
    char *fileName = "image.JPG";
    FILE *imgFile = fopen(fileName, "rb");

    int size = getFileSize(imgFile);
    printf("图片文件大小是 %d\n", size);

    // 切割成 3 份
    int part = 3;
    int partSize = size / part + (size % part); // 可能会有余数, 整除不了，所以需要考虑余数部分
    printf("一份的图片大小 partSize = %d\n", partSize);

    // 定义切割几份的文件名称
    // 定义一个二级指针数组, 二级指针的一级指针就是字符串, 也就是定义的路径
    char **fileNames = (char **)malloc(sizeof(char *) * part);
    int i = 0;
    for (; i < part; i++)
    {
        fileNames[i] = (char *)malloc(sizeof(char) * 100); // '\0'
        // fileNames[i] 就是一级指针, 或者 *fileNames
        // sprintf 函数的作用是根据指定的格式(format)，输出到一个字符串中。
        sprintf(fileNames[i], "image_%d.JPG", i);
        printf("一级指针文件路径是: %s\n", fileNames[i]);
    }

    // 文件进行切割
    i = 0;
    for (; i < part; i++)
    {
        int start = i * partSize;  // 开始的位置
        int end = (i + 1) * partSize; // 结束的位置
        if (i == part - 1)
        {
            end = size; // 指向最后一个位置
        }

        FILE *cur_file = fopen(fileNames[i], "wb");

        for (int j = start; j < end; j++)
        {
            // fgetc(); 从文件中读取一个字符。
            // fputc(); 将一个字符写入文件。
            fputc(fgetc(imgFile), cur_file);
        }

        fclose(cur_file);
        free(fileNames[i]); // 释放一级申请的内存
    }

    fclose(imgFile);
    free(fileNames); // 释放申请的内存
}
```

## 切割的文件进行合并
> 一个一个字节读取
```c
void main()
{
    int part = 3;
    char **file_names = (char **)malloc(sizeof(char *) * part);

    // 创建合并的文件 0kb
    FILE *f_merge = fopen("image_merge.JPG", "wb");

    // 二级指针
    FILE **files = (FILE **)malloc(sizeof(FILE *) * part);

    int i = 0;
    for (; i < part; i++)
    {
        file_names[i] = (char *)malloc(sizeof(char) * 100);
        sprintf(file_names[i], "image_%d.JPG", i);

        files[i] = fopen(file_names[i], "rb");

        int c = 0;
        while ((c = fgetc(files[i])) != EOF)  // 一个字节一个字节读取文件
        {
            fputc(c, f_merge);// 写入文件
        }

        fclose(files[i]);
        free(file_names[i]);
    }

    fclose(f_merge);
    free(files);
    free(file_names);
}
```

## 切割的文件进行合并（使用缓冲区）
> 性能更好
```c
void main()
{
    FILE *destinationFile = fopen("image_dest.JPG", "w+");

    int part = 3;
    char **file_names = (char **)malloc(sizeof(char *) * part);

    int i = 0;
    for (; i < part; i++)
    {
        file_names[i] = (char *)malloc(sizeof(char) * 100);
        sprintf(file_names[i], "image_%d.JPG", i);

        FILE *sourece_file = fopen(file_names[i], "r+");

        // 从缓冲区里面读取
        char buffer[BUFFER_SIZE];
        int bytesRead;
        do
        {
            bytesRead = fread(buffer, sizeof(char), BUFFER_SIZE, sourece_file);
            fwrite(buffer, sizeof(char), bytesRead, destinationFile); // 把缓冲区的数据写入目标文件
        } while (bytesRead == BUFFER_SIZE); // 不相等的时候, 说明读取完毕了
        fclose(sourece_file);
        free(file_names[i]);
    }

    fclose(destinationFile);
    free(file_names);
}
```