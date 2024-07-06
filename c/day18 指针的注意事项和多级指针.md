## 指针常量和常量指针的区别
- 常量指针 
    - 常量指针是指向常量的指针，即指针所指向的数据是不可修改的，但是指针可以修改    
    ```c
    const int* ptr;
    ```

- 指针常量
    - 指针常量是一个指针，它本身是不可修改的，即指针所指向的地址是固定的，但是数据是可以修改的
    ```c
    int* const ptr;
    ```
- 指向常量的指针常量
    - 值和指针都不能被修改
    ```c
    const int *const ptr
    ```

## 示例
```c
void main()
{
    const int number = 32; // 常量
    int number2 = 34;

    // number = 22; 错误, 常量的值是不能被修改的

    const int *numberP = &number; // 是一个常量指针，值不能被修改，但可以修改指针

    // *numberP = 200; （值）不能被修改
    numberP = &number2; // 但是可以修改地址值， 所以这样间接的也算修改了数据
    printf("numberP = %d\n", *numberP); // 34

    // 指针常量的指针是不能被修改的, 值可以修改
    int *const numberP2 = &number2;

    // *numberP2 = 88; 可以修改
    // numberP2 = &number; // 错误, 值不能修改

    // 这样修饰的, 值不能修改, 指针也不能修改
    const int *const number3 = &number2;
}
```

## 数组和多级指针的使用
> 二维数组和二级指针其实是一样的意思

```c
void main()
{
    // 二维数组【元素个数】【元素长度（包含结束符号\0）】
    char str[10][20] = {"mid", "fang", "end"};

    // 一级指针的数组
    // 或者是这样, 3 元素代表元素个数, 只能存 3个元素个数 (一维存放的是一级指针的地址, 二维存放的是元素字符串)
    char *str1[3] = {"mid", "fang", "end"};

    for (int i = 0; i < 3; i++)
    {
        // 打印
        printf("str1 值打印 - %s\n", str1[i]);
    }

    // 指针方式的二维数组 这样不行会报错,因为把 右边一个整体的直接当成是一个二级指针了(实际右边是一个一维数组),指针数组了
    // char **str3 = {"mid", "fang", "end"};

    // 动态内存开辟的方式
    // 开启二维数组， 其实就是二级指针， 指针就是数组，可以这么理解
    // 二级指针的值是一级指针的地址
    char **str4 = (char **)malloc(sizeof(char *) * 4);

    for (int i = 0; i < 4; i++)
    {
        // 动态开辟内存,  // 开辟一维数组
        str4[i] = (char *)malloc(sizeof(char) * 100);

        sprintf(str4[i], "number_%d", i);

        // strcpy(*(str4)+i, "number_");
        // strcpy(str4[i], "number_"); // 这样代码为什么会报错
    }

    for (int i = 0; i < 4; i++)
    {
        // 打印
        printf("值打印 - %s\n", str4[i]);
    }

    // 释放: 先释放一维数组, 再释放二维数组
    for (int i = 0; i < 4; i++)
    {
        if (str4[i] != NULL)
        {
            free(str4[i]);
            str4[i] = NULL;
        }
    }

    // 释放二维数组
    if (str4 != NULL)
    {
        free(str4);
        str4 = NULL; // 避免野指针的出现
    }

    // 查看是否已经被释放了
    // if (str4 == NULL && str4[0] == NULL) //str4[0] 不能打印了, 因为上面 str一维数组已经比释放了
    // {
    //     printf("数组被释放了");
    // }

    输出：
        str1 值打印 - mid
        str1 值打印 - fang
        str1 值打印 - end
        值打印 - number_0
        值打印 - number_1
        值打印 - number_2
        值打印 - number_3
}
```

## 多级指针的取值
在C语言中，三级指针的取值需要通过多次解引用来实现。假设我们有一个三级指针 char ***ptr，那么：
- ptr 是一个三级指针，它存储的是一个二级指针的地址。
- *ptr 是一个二级指针，它存储的是一个一级指针的地址。
- **ptr 是一个一级指针，它存储的是一个具体数据（例如字符数组）的地址。
- \*\*\*ptr 则是具体的数据值，如果 ptr 指向的是字符数组的指针，那么  \*\*\*ptr 就是字符数组首元素的值。

## 多级指针的间接赋值

```c
void setStr(char ***str, int len)
{
    // 使用一个临时变量去操作, 更加安全
    char **temp = (char **)malloc(sizeof(char *) * len);

    // 如果是第一种方式, 需要保存首地址
    char **index = temp;

    for (int i = 0; i < len; i++)
    {
        *(temp) = (char *)malloc(sizeof(char) * 100);
        sprintf(*temp, "第一种 n = %d", i);
        // 指针后移, 如果是指针偏移的话, 因为这里的赋值偏移到了最后一个位置,如果最后一行赋值错误的话, 外部也是错误了
        temp++;

        // *(temp + i) = malloc(sizeof(char) * 100);
        // sprintf(*(temp + i), "第二种 n = %d", i);

        // 这样赋值也是一样的
        // temp[i] = malloc(sizeof(char) * 100);
        // sprintf(temp[i], "第三种方式 n = %d", i);
    }

    // 第一种方式的话, 需要将首地址返回,因为指针偏移了
    *str = index; // str 是一个3级指针，取值变为二级指针，index 是一个二级指针，也就是 temp 的首地址

    // *str = temp;
}

// 多级指针通过方法的赋值
void main()
{

    char **str = NULL;

    int number = 3;
    setStr(&str, number);

    // 打印
    for (int i = 0; i < number; i++)
    {
        printf("值打印 - %s\n", str[i]);
    }
}

输出：
    值打印 - 第一种 n = 0
    值打印 - 第一种 n = 1
    值打印 - 第一种 n = 2
```
