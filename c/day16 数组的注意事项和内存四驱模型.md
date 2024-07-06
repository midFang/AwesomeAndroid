## native 数组的处理，和数据的同步和释放

在 java 层创建数据， 给 native 中进行数据
```java 
 var arr = intArrayOf(11, 22, -3, 2, 4, 6, -15)
        // 在 native 中，对数据进行排序
        nativeSort(arr)

        println("@@@@ MainActivity 数组： arr = ${arr.joinToString { it.toString() }}")
```

```c
/** native 中对数组的处理，对数组进行排序 **/
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_nativeSort(JNIEnv *env, jobject thiz, jintArray arr) {
    // 获取 java 中的数组
    jint *intArray = env->GetIntArrayElements(arr, nullptr);

    // 长度
    int len = env->GetArrayLength(arr);

    /**
     * qsort(void* __base, size_t __nmemb, size_t __size, int (*__comparator)(const void* __lhs, const void* __rhs))
     * 第一个参数， __base 数组的首地址
     * 第二个参数， __nmemb 数组的长度
     * 第三个参数，数组元素数据类型的大小
     * 第四个参数，数组的比较方法
     */
    qsort(intArray, len, sizeof(int), compare_ints);

    // 将数组同步给 java 层 （一定要同步，不然 java 层的数据都是老数据）
    /**
     * mode 参数：
     * 0 : 既要同步数据给 jarray ,又要释放 intArray
     * JNI_COMMIT:  会同步数据给 jarray ，但是不会释放 intArray
     * JNI_ABORT: 不同步数据给 jarray ，但是会释放 intArray
     */
    env->ReleaseIntArrayElements(arr, intArray, 0);
    /**
     * 注意还有其他各种释放 Release函数， 比如 ReleaseStringUTFChars，可以理解为对象的释放，且同步数据的函数 Api
     */
}
```



## 数组作为参数传递
```c
void printArray(int array[])
{
    int size = sizeof(array) / sizeof(int); // array 会退化变成一个指针, sizeof(array) 计算的是指针的大小
    printf("printArray method size = %d\n", size);

    for (int i = 0; i < size; i++)
    {
        printf("printArray method array %d = %d\n", i, array[i]);
    }
}

void printArray2(int array[], int length)
{
    printf("printArray2 method size = %d\n", length);

    for (int i = 0; i < length; i++)
    {
        printf("printArray2 method array %d = %d\n", i, array[i]);
    }
}

// 指针作为参数进行传递会出现的问题
void main()
{
    int array[] = {1, 5, 7, 9, 23};

    // 求数组的长度, 使用数组的总大小, 除以数组一个元素的大小
    int size = sizeof(array) / sizeof(int);
    printf("size = %d\n", size);

    // 打印
    for (int i = 0; i < size; i++)
    {
        printf("array %d = %d\n", i, array[i]);
    }

    // 上面是正常情况, 如果我通过 printArray 打印, 试试看
    // 结果是无法打印, 因为当数组通过参数传递的时候, 这个数组会退化成指针, 而上面的方法 sizeof(array) 计算的是指针的大小,而不是数组的大小
    printArray(array);

    // 所以要想解决这个问题, 只有将长度传递进去, 注: 数组的传递都需要这样做
    printArray2(array, size);
    /**
     * 输出:
     *   size = 5
     *   array 0 = 1
     *   array 1 = 5
     *   array 2 = 7
     *   array 3 = 9
     *   array 4 = 23
     *   printArray method size = 2
     *   printArray method array 0 = 1
     *   printArray method array 1 = 5
     *   printArray2 method size = 5
     *   printArray2 method array 0 = 1
     *   printArray2 method array 1 = 5
     *   printArray2 method array 2 = 7
     *   printArray2 method array 3 = 9
     *   printArray2 method array 4 = 23
     */
}
```

## 数据类型的剖析
- 数据类型的本质， 数据的别名 
- 数据类型的本质：一块连续大小的内存空间 (比如 int 占4个字节)
- 数据类型的别名：int32_t
- void指针数据类型：void* 代表任意的数据类型的指针

```c
struct Person
{
    char *name;
};

typedef struct Person student; // 这样是取一个别名， 叫 student
// 其实 C++ 自己也有一些取了别名的， 比如 int32, 它就是对 int 取别名
typedef int int32;

void main1()
{
    student s = {"fang"}; // 使用的是别名

    printf("Person = %s\n", s.name); 

    int32 a = 55; // 使用的是别名
    printf("int32 = %d\n", a);

    int arr[] = {1, 2, 3, 4, 5, 6}; // arr 数据类型的内存大小空间 ，arr 的大小是24

    printf("%d, %d, %d ,%d", arr, arr + 1, &arr, &arr + 1);
    /**
     * 输出的值是这样的：  6421952, 6421956, 6421952 ,6421976
     *
     * 分析一下，arr是6421952， arr + 1 是 6421956， 好像没什么问题，因为 int 是4个字节，偏移了4个字节
     * 但是为什么 &arr + 1 确实增加了 24 呢？
     *
     * 这是因为 arr 是第一个元素首地址， 代表的是 int 类型指针， 占4个字节，
     * 而数据类型的本质是 一块连续大小的内存空间，偏移 +1 ，其实偏移了一个内存空间, 所以  arr + 1 偏移了4个位置
     * 而 &arr 它代表的是整个 arr 数组的地址， 它的内存大小是 4*6=24，所以偏移 +1，偏移了24个位置
     *
     */

    // memcpy();  malloc 都是 void* 类型， 表示任意数据类型的指针 
}
```

## 内存的四驱模型-全局区
- 栈区：由编译器自动分配的，存放一些局部变量值和函数，这个里面内存是会自动进行回收的
- 堆区：一般都是由我们自己去开辟的，这个里面的内存需要手动进行释放  malloc  free  new  delete
- 全局/静态存储区：静态的一些常量，字符串 等等
- 程序代码区：存放的是函数体的二进制代码 （一般用不到，不用管）

```c
char *getStr1()
{
    char *str = "12345";
    return str;
}
char* getStr2(){
	char* str = "12345";
	return str;
}

// 内存四驱模型
void main2()
{
    char *str1 = NULL;
    char *str2 = NULL;

    str1 = getStr1();
    str2 = getStr2();

    printf("str1 = %d, str2 = %d \n", str1, str2);
    // str1 = 4210889, str2 = 4210889 输出的都是一样的
}
```

上述代码的内存四驱模型和执行情况
![](https://raw.githubusercontent.com/midFang/imgSource/main/20240416231106.png)


## 内存四驱模型-栈区
```c
char *getStr3()
{
    char buff[128];

    strcpy(buff, "12345");

    return buff;
}

void main3()
{
    char *str3 = getStr3();
    printf("str = %p, str3 = %s  %p", str, str3, str3); // str = 0000000000000000, str3 = (null)  0000000000000000
}

```
上述代码的内存四驱模型和执行情况

![](https://raw.githubusercontent.com/midFang/imgSource/main/20240416232845.png)

## 内存四驱模型-堆区
```c
char *getStr4()
{
    char *buff = (char*)malloc(128); // char* buff char* 是四字节的数据类型，存的是堆区开辟的 128 个字节的首地址
    strcpy(buff, "12345");

    return buff;
}

void main4()
{
    char *str4 = getStr4();
    printf(" str4 = %s  %p", str4, str4); // str4 = 12345  00000000009E15D0
}
```

上述代码的内存四驱模型和执行情况
![](https://raw.githubusercontent.com/midFang/imgSource/main/20240416234415.png)

## 栈的开口 

通过一个小栗子， 看看数据在栈内的地址值，就知道栈的开口
```c
void main()
{
    int a = 100;
    int b = 200;
    // 6421996 , 6421992
    printf("%d , %d", &a, &b); // 在 debug 的环境是： a的地址 > b 的地址
    // 注：数组不涉及开口的向上和向下，内存地址一直连续增长的， 因为数组他是一块连续的内存空间
}
```

![](https://raw.githubusercontent.com/midFang/imgSource/main/20240417225229.png)