## 结构体大小是什么
- 根据每个的偏移量计算得来的, 而结构体的大小和数据变量排放的位置有关系
- 结构变量的大小等于它包含所有变量的总大小

## 结构体大小的计算

```c
typedef struct
{
    int month;
    int week;
    int day;
} birthday; // 大小为 12

typedef struct
{
    double height; // 大小为 8
    double width;  // 大小为 8
    int a;         // 大小为 4
    int age;       // 大小为 4
    char cc1;      // 大小为 1
    char name[10]; // 大小为 10

    // 计算得来的是 35, 但是因为要补齐，能够被8整除（最宽基本类型成员）, 所以为 40, 大小一致了
} studuent;

void main(){
     studuent *stu = NULL;

    // 计算指针的偏移量 
    // printf("指针的偏移量: %d\n", &(stu->name) - (int)stu); // 新的编译器会报错，所以使用下面的方法
    printf("指针的偏移量: %ld\n", (uintptr_t) & (stu->name) - (uintptr_t)stu);

    // 结构体大小
    printf("大小: %d\n", sizeof(studuent));
}
```


这里解释一下 &(stu->width) - (int)stu) 是什么意思 ？ 
其中 stu 代表这个结构体的首地址，而 (stu->width) - (int)stu) 就是 width 的指针位置减去首地址的位置，这样就能得到 height 的大小了，其实它的大小就是指针偏移的位置,这里得出的结构是 8，所以指针偏移了 8 个位置，所以结构体内部的属性 height 占用 8 个字节大小

## 第一种情况
```c
 typedef struct
 {
     int age; // 4 , 字节对齐, 要看后一个变量, 这里本来是4, 但是因为后一个是8 , 字节对齐后, 这里也是 8
     double height; // 8
     char name[10]; // 10
     // 所以 8+8+10 = 28, 不能够被 8 整除, 补齐所以等于大小等于 32
 } studuent;
```
## 第二种情况
```c
 typedef struct
 {
     double height; // 8
     int age;       // 4 , 前一个后一个都不相等（height和name）, 不用对齐, 所以 8+4+10 = 22, 不能够被 4整除, 补齐2, 大小为 24
     char name[10]; // 10
   
     // 指针的偏移量: 12, 大小: 24
 } studuent;
```

上述两个情况， 仅仅是改变了成员位置，可以发现大小不一样了，所以可得出
- 大小怎么来的？ 是根据每个的偏移量计算得来的, 而结构体的大小和数据变量排放的位置有关系
- 把最小数据尽量往后靠 (基本数据类型), 结构体定义了一般不要轻易的改动位置, 因为有的地方的代码可能是根据偏移量才操作数据的

## 第三种情况
> 结构体嵌套
```c
 typedef struct
 {
     int month; // 4
     int week; // 4
     int day; // 4
 } birthday; // size  12

 typedef struct
 {
     double height; // 8
     int age;       // 4
     char name[10]; // 10
     birthday b; //  12
     // 8+4+10+12 = 34 , 补齐需要能够被 8 整除, 所以等于 40
 } studuent;
```

## 第四种情况
> 改变成员位置会影响大小
```c
 typedef struct
 {
     // 将数据量小的放在前面
     char cc1; // 1
     char cc44; // 1
     char cc2;  // 1
     double height; // 8
     double width;  // 8

     大小是 19, 不够被8整除, 补齐, 所以大小是 24
 } studuent;

 // 如果将其中一个 char 放在中间
 typedef struct
 {
     // 将数据量小的放在前面
     char cc1; // 1
     char cc44; // 1
     double height; // 8
     char cc2;  // 由于需要对其, 这里变成 8
     double width;  // 8

     大小是 26, 不够被8整除, 补齐, 所以大小是 32
     // 相较于上面的例子, 仅仅是位置改变了一下, 大小大了很多
 } studuent;
```

## 第五种情况
> 将占用小的放在前面
```c
 typedef struct
 {
     char cc1;      // 看后面的一位是 8, 需要对齐, 也是 8,
     double height; // 8
     double width;  // 8
     int a;         // 4
     int age;       // 4
     char name[10]; // 10

     // 24+8 = 32 +10 = 42,不够被8整除, 补齐为 48
 } studuent;

 // 尝试,如果将大的(基本数据类型)放在前面, 下面尝试将放在int类型字节前面, 大小为 40,
 如果放在 age 后面, 再看看
 typedef struct
 {
     double height; // 8
     double width;  // 8
     char cc1;      // 看后面的一位是 4, 需要对齐, 也是 4,
     int a;         // 4
     int age;       // 4
     char name[10]; // 10

     // 38 , 不够8整除, 补齐为 40
 } studuent;

 // 再挪动一个位置
 typedef struct
 {
     double height; // 8
     double width;  // 8
     int a;         // 4
     int age;       // 4
     char cc1;      // 1
     char name[10]; // 10

     // 计算得来的是 35, 虽然是小了一点, 但是因为要补齐, 所以为 40, 大小一致了
 } studuent;

```

## 结构体的赋值
```c
struct Person
{
    char *name;
    int age;
};

void main()
{
    struct Person p = {"midFang", 22};

    // 尝试修改name
    p.name = "msd"; // 常量区的地址赋值给 *name
    // 或者是 malloc 的方式也是可以的 p.name = malloc

    printf("p.name -> %s\n", p.name);
    printf("p.age -> %d\n", p.age);

    struct Person p2 = p; // 结构体赋值操作

    printf("p.name -> %s\n", p2.name);
    printf("p.age -> %d\n", p2.age);
}
```

## 浅拷贝
```c
void copyTo(struct Person *source, struct Person *to)
{
    *source = *to; // 这里是指针的赋值操作, 指针的赋值操作是浅拷贝, 在内存模型里面, 只是将 source 的引用指向了 to 的内存区域而已, 指向的是同一块区域
    // printf("p1 的地址%p, p2 的地址 %p \n", source->name, to->name);
}

// 深拷贝和浅拷贝
void main()
{
    struct Person p1 = {"zhangsan", 33};

    struct Person p2;

    copyTo(&p2, &p1);

    printf("p2 = %s  %d\n", p2.name, p2.age);

    // 查看内存地址
    printf("p1 的地址%p, p2 的地址 %p \n", p1.name, p2.name);

    // 进行释放
    if (p1.name != NULL)
    {
        free(p1.name);
        p1.name = NULL;
    }

    // pointer being freed was not allocated 会报错, 因为是浅拷贝, 指向的是同一块内存区域, 其实 p1.name 已经被释放掉了
    if (p2.name != NULL)
    {
        free(p2.name);
        p2.name = NULL;
    }
}
```

## 深拷贝
```c
void deepCopy(struct Person *source, struct Person *to)
{
    // *source = *to; // 浅拷贝, 这个代码不用也可以, 只是将值赋值而已, 而下面已经是重新分配内存了

    // 深度拷贝, 重新申请内存
    source->name = (char*)malloc(sizeof(100));
    // source->name = to->name; // 这样不行 ? 因为这是改变的指针的引用, 指针的赋值是浅拷贝

    strcpy(source->name, to->name);
}

// 深度拷贝
void main()
{
    struct Person p1 = {"zhangsan", 33};

    struct Person p2;

    printf("p1 的地址%s, p2 的地址 %s \n", p1.name, p2.name);
    deepCopy(&p2, &p1);

    printf("p1 的地址%p, p2 的地址 %p \n", &p1.name, &p2.name);
    printf("p1 的值%s, p2 的值 %s \n", p1.name, p2.name);

    if (p2.name != NULL)
    {
        printf("对 p2.name 开始进行 free \n");
        free(p2.name);
        printf("对 p2.name 结束 free \n");
        p2.name = NULL;
    }

    if (p1.name != NULL)
    {
        printf("对 p1.name 开始进行 free \n");
        // free(p1.name); 对这里 free 报错, 因为 p1.name 并未通过 malloc 分配内存，它只是一个静态字符串, 函数 free 只能被用于释放通过 malloc、calloc 或 realloc 函数获得的内存
        printf("对 p1.name 结束 free \n");
        p1.name = NULL;
    }
}
```