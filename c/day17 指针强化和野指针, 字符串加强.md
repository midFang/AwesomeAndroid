## NULL 地址指针问题
> NULL 地址是系统标识的一个地址, 不可访问

```c
int strcpyCompat(char **str)
{
    *str = "new value";
    return 1;
}

void main()
{
    // NULL 地址指针的问题
    char *name = NULL;

    printf("name = %s\n", name);
    printf("name 的地址是 %p\n", name);

    // Exception has occurred. 运行报错, 因为 NULL 地址是系统标示的一个地址, 不可访问
    // EXC_BAD_ACCESS (code=1, address=0x0)
    // 因为strcpy传递的是值, 而不是指针, 而又因为 name的值是 NULL 的地址, 是不可访问的, 所以报错了
    // strcpy(name, "value");

    /**
     * name 指针变量的地址指向的是 NULL 地址, 而传入 strcpy 是看起来是name变量, 但内部其实是传入的 NULL 值, 所以不能修改
     * 而 strcpyCompat 可以, 是因为传递的是 name 变量的地址, 而不是 NULL 地址 (指针间接赋值的意思)
     */
    strcpyCompat(&name);

    printf("name = %s\n", name);
}

/**
*   输出: 
    name = (null)
    name 的地址是 0000000000000000
    name = new value
*/
```

## 初始化的常见的几种方式
```c
void main()
{
    // 字符串强化, 初始化的常见的几种方式

    // char buffer[100] = "midFang"; // len = 7, sizeOf = 100 , 后面的都是 0 ,字符会被 \0 替换掉
    char buffer[100] = {'m', 'i', 'd', 'F', 'a', 'n', 'g'}; // len = 7, sizeOf = 100 , 后面的都是 0 是一样的

    // char buff[100] = { 'd','a', 'r', 'r', 'e', 'n' };// 后面 6 - 99 都是默认值 0
    // char buff[5] = { 'd', 'r', 'r', 'e', 'n' };
    // char buff[] = {'d','r','r','e','n'}; // 长度是 12（'\0'） ，size 是 5 （默认统计里面的个数）
    // char buff[2] = { 'd', 'r', 'r', 'e', 'n' };// 编译不通过，长度超出
    // char buff[100] = { 0 };// 把数组初始化为 0
    // char buff[100] 数据都是默认值 -52

    // char buff[] = "123456";// len 是 6 （'\0'），size 是 7 ？
    // 相当于 char buff[] = {1，2，3，4，5，6，\0}
    char *buff = "123456";

    // 纠结一下 char buff[] = "123456" 和 char* buff = "123456"; malloc 的方式  啥区别 ？
    // 字符串可以在任何地方开辟内存，栈区，堆区，常量区

    // 大小 size 100
    int len = strlen(buff); // len 5  碰到 '\0' 就结束了
    int size = sizeof(buff);

    printf("len = %d, sizeOf = %d\n", len, size);
    printf("后面的: %d,   %d\n", buffer[20], buffer[50]);
}
```

## 指针间接赋值
```c
// 定义获取bitMap信息类, 在c中相当于是结构体
struct AndroidBitmapInfo
{
    int width;
    int height;
};

// 获取 bitmap 信息
int AndroidBitmapInfo_get(struct AndroidBitmapInfo *info)
{
    if (info == NULL)
    {
        printf("bitmapInfo 是 NULL 无法进行操作，错误码 %d", -1);
        return -1; // 失败
    }

    info->width = 200;
    info->height = 300;

    return 0; // 成功
}

void main()
{
    // 通过指针间接赋值很常见 (堆内存上开辟)
    struct AndroidBitmapInfo *androidBitmapInfo = (AndroidBitmapInfo*)malloc(sizeof(struct AndroidBitmapInfo));

    int result = AndroidBitmapInfo_get(androidBitmapInfo);

    if (result != -1)
    {
        printf("宽高是: %d, %d\n", androidBitmapInfo->width, androidBitmapInfo->height);
    }

    if (androidBitmapInfo != NULL)
    {
        free(androidBitmapInfo);
        androidBitmapInfo = NULL; // 避免出现野指针的情况 (野指针就是 free 了对象, 但是 androidBitmapInfo 指向的还是原来的那块地址, 但是这个地址上的值已经不存在了)
    }
}
```