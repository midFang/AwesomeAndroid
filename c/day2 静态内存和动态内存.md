## 静态开辟内存

> 方法在执行完成后， 内存会被自动释放

```c
void staticMalloc()  // 这个方法执行完成后, 内部所使用的变量占用的内存, 都会自动释放
{
    int arr[5];
    int i = 0;
    for (; i < 5; i++)
    {
        arr[i] = i;
    }
}

void main(){
    while(true){
        sleep(1000);
        staticMalloc(); // 开启无限循环, 程序并不会卡死
    }
}
```

## 动态开辟内存

> 需要手动申请内存和手动释放内存

```c
// 数组打印
void arrPrintf(int *array, int len)
{
    int i = 0;
    for (; i < len; i++)
    {
        printf("array%d=%d\n", i, *(array + i));
    }
}

void main()
{
    int num; // 4字节大小
    printf("请输入个数");
    scanf("%d", &num);

    int *d_array = malloc(sizeof(int) * num); // 动态申请内存；内存大小是： （sizeOf=4） * num

    int i = 0;
    int scan_m;
    for (; i < num; i++)
    {
        printf("请输入要存入的数 ");
        scanf("%d", &scan_m);
        *(d_array + i) = scan_m;
        // d_array[i]= scan_m; // 这个也是一样的， 本质上是指针的偏移操作
    }

    // 打印
    arrPrintf(d_array, num);
}
```

## 动态申请内存二次分配

> realloc 是给一个已经分配了地址的指针重新分配动态内存空间

```c
// 重新分配动态内存空间
void main()
{
    int num;
    printf("请输入要存入的个数");
    scanf("%d", &num);
    int *arr = malloc(sizeof(int) * num); // arr 可以看做是一个数组

    int i = 0;
    int scan_m;
    for (; i < num; i++)
    {
        printf("请输入要存入的值");
        scanf("%d", &scan_m);
        arr[i] = scan_m;
    }

    arrPrintf(arr, num);

    // 查看地址值，arr 的首地址
    printf("arr的首地址 %p\n", &arr);

    // 想再存入数据， 可以再次分配
    printf("请输入要新增加的数");
    int newNum;
    scanf("%d", &newNum);
    int *newArr = realloc(arr, sizeof(int) * (num + newNum));

    // 需要判null，内存不足的时候，可能会申请失败
    // if(newArr) 等价于 if(newArr != NULL)

    // 一般情况下， &arr == &newArr
    // 地址可能会不一样，内存不足的时候，不能够连续的时候，会重新找一个可以放置容量大小的区域
    printf("arr新的的首地址 %p\n", &arr);

    i = num; // 从num后新增加数
    for (; i < (num + newNum); i++) 
    {
        printf("请输入要新增加的数值");
        scanf("%d", &scan_m);
        newArr[i] = scan_m;
    }

    arrPrintf(newArr, num + newNum);

    if (newArr)
    {
        // 申请到了内存, 在申请到了内存的情况下（重新新增加），旧的 arr 会被释放
        free(newArr);
        // 注意事项： 要设置 NULL
        newArr = NULL;
    }
    else
    {
        // 没有申请到内存， 把老的释放
        free(arr);
        arr = NULL;
    }
}
```

## 总结

- 静态内存开辟： 内存的大小是2M，方法执行完后，内存会被自动释放
- 动态内存开辟： 需要手动申请内存和需要手动释放，内存大小约是内存最大值80%， 后续可动态再次申请内存

## 注意事项

1. 必须需要手动释放
2. 动态内存开辟，内存不够的时候，需要再次申请的时候，前面的内存不需要再重新赋值，也不需要释放，因为内部已经释放了，并将原始的分配到一块新的内存上了
3. 二次分配内存地址的时候，内存地址可能不是原来的，（其中一块区域被系统占用了），会重新找一块新的内存区域进行存储
