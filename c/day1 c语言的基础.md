
## 基本数据类型和字节相关

```c
void changeNum(int *d)
{
    *d = 666;
}

int main()
{

    // 基本数据类型, 和对应的数据类型打印
    int a = 1;
    double d = 2.0;
    float f = 3.0f;
    char c = c;
    _Bool bb = 1;
    short s = 100;
    long l = 150;
    long num = 1234567890L;
    unsigned long number = 1234567890;
    //  printf("The value is: %ld\n", number); 错误打印
    printf("The value is: %lu\n", number);

    printf("%ld", num);

    printf("a int == %d\n", a);     // 1
    printf("b double == %lf\n", d); // 2.000000
    printf("f float == %f\n", f);   // 3.000000
    printf("c char %d\n", bb);      // 1
    printf("s short %d\n", s);      // 100
    printf("l long %ld\n", l);      // 150
    printf("num long %ld\n", num);  // 150

    /**
        %d：输出十进制整数 decimal
        %f：输出浮点数 float
        %c：输出一个字符 character
        %s：输出一个字符串 string
        %p：输出指针变量的值 pointer
        %o：输出八进制整数 octal
        %x或%X：输出十六进制整数，%x输出小写字母，%X输出大写字母 hexadecimal
        如%u（输出无符号十进制整数unsigned）、
        %e或%E（输出科学计数法表示的浮点数exponential）等。
    */

    //////////////////////////////////////////////////////////////////////////////////////////

    // 2. 字节数
    printf("int sizeOf %lu\n", sizeof(int));        // 4
    printf("long sizeOf %lu\n", sizeof(long));      // 8
    printf("double sizeOf %lu\n", sizeof(double));  // 8
    printf("int* sizeof %lu\n", sizeof(int *));     // 8
    printf("char* sizeof %lu\n", sizeof(char *));   // 8
    printf("char** sizeof %lu\n", sizeof(char **)); // 8

    /**
    char：1字节
    short：2字节
    int：4字节
    long：4字节（32位系统）或8字节（64位系统）
    long long：8字节
    float：4字节
    double：8字节
    int* 4字节（32位系统）或8字节（64位系统）
    char* 4字节（32位系统）或8字节（64位系统）
    **/

    // 3. 指针
    // 指针变量和获取地址的值
    // 通过指针修改值
    // 怎么获取变量的指针地址
    // 通过指针怎么获取值， 有哪几种方式
    // 通过指针怎么修改值
    // 在方法中怎么做到对一个值修改
    // 题目：写个方法对两个值进行交换

    int p1 = 100;
    // 指针变量和获取地址的值
    printf("p1 的地址是 %p\n", &p1); // 获取p1变量的地址
    printf("p1地址取值是%d\n", *(&p1)); // 通过地址取值

    // 指针为什么要有类型， 他不是地址吗 ？ 按理说不应该只有 int 类型， 全都指向一个地址不就可以吗
    // 因为有字节大小，不同的类型是不同的大小，还有指针的偏移，也要知道大小

    // 通过指针修改值, *p_p1 就是一个指针变量，他们两个相等
    int *p_p1 = &p1;
    printf("p_p1地址取值是%d\n", *p_p1);
    printf("*p_p1 的地址是 %p\n, p1的地址是 %p\n, *p_p1==&p1=%d\n ", p_p1, &p1, p_p1 == &p1);

    // 指针对值的修改
    *(&p1) = 150;
    printf("p1取值是%d\n", p1);
    *(p_p1) = 200;
    printf("p_p1取值是%d\n", p1);

    // 方法指针值的修改
    // changeNum(&p1);
    changeNum(p_p1);
    printf("p1取值是%d\n", p1);

    // 题目：写个方法对两个值进行交换 (无额外内存开销)
    // int a = 100;
    // int b = 300;

    // a = b - a; // 200
    // b = b - a; // 100
    // a = a + b; // 300 // a = 300, b = 100

    unsigned long unsignedNumber = 1234567890;
    long signedNumber = -987654321;

    printf("Unsigned number: %lu\n", unsignedNumber);
    printf("Signed number: %lu\n", signedNumber); // 18446744072721897295打印值不正确
    printf("Signed number: %ld\n", signedNumber);
}
```

## 一级二级指针, 函数指针

```c
void addMethod(int a, int b)
{
    int c = a + b;
    printf("a+b=%d\n", c);
}

// 函数指针：返回值（*方法名）（入参）
void cauMethod(void (*callback)(int, int), int a, int b) // 可以认为传入一个方法
{
    callback(a, b);
}

int main()
{
    int a = 12;
    int *p = &a;   // 对 a取地址值, p 是个一级指针
    int **pp = &p; // pp 是一个二级指针 -- 二级指针的值是 p地址， 二级指针的地址值是它本身p的地址值
    printf("a的值是%d\np的一级指针地址取值%d\npp二级指针取值%d\n", a, *p, **pp);
    printf("a的地址是%p\n", &a);
    printf("p的地址是%p\n", p);
    printf("pp的值是%p\n", *pp); // pp 的值，就是一级指针，他的值也就是p的地址

    // 打印值或者修改值
    printf("p取值%d\n", *p);
    printf("pp取值%p\n", *pp);
    printf("pp取值再取值%d\n", **pp);
    *p = 13;
    **pp = 15;
    printf("p取值%d\n", *p);
    printf("pp取值再取值%d\n", **pp);

    // 数组和数组指针
    int arr[] = {1, 2, 3, 4};
    // 要用这种方式遍历，保证再 linux 中系统不会报错
    int i = 0;
    for (; i < 4; i++)
    {
        printf("arr遍历取值%d\n", arr[i]);
    }

    // 看一种现象,
    printf("arr 地址%p\n", arr);
    printf("&arr 地址%p\n", &arr);
    printf("arr 地址%p\n", &arr[0]);
    /**
        arr 地址0x7ffeea2c53f0
        &arr 地址0x7ffeea2c53f0
        arr 地址0x7ffeea2c53f0  3个地址都相同，因为数组arr代表的就是首地址，&arr 和 &arr[0]都是首地址的值
    **/

    // 指针循环遍历和赋值
    int *arr_p = arr;
    // arr_p ++; // 地址偏移
    i = 0;
    for (; i < 4; i++)
    {
        // 指针循环遍历赋值
        // *(arr_p + i) = i+2;
        // printf("arr遍历取值%d\n", arr[i]);
        printf("arr遍历取值%d\n", *arr_p);
        arr_p++;
    }

    // 函数指针使用, 相当于传入了一个方法
    cauMethod(addMethod, 5, 4);
}
```