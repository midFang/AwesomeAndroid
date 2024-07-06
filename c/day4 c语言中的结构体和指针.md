## 结构体的定义
> 使用 struct 关键字 

```c
struct Person
{
    int age;
    char name[20];
    char *address; 
};
```

## 结构体起别名

```c
struct Person1
{
    int age;
    char name[20];
    char *address;
} person1;
```

## 结构体初始化
```c
struct Person1
{
    int age;
    char name[20];
    char *address;
} person1 = {13, "zhangsan", "shandong"};
```

## 结构体嵌套
```c
struct Person2
{
    int age;
    Person person;
    struct Person3
    {
        int age;
    } p3; // 需要指定别名, 不然无法赋值

    // 要么就替换成如下这种方式也可以, 像是声明了一个变量一样, 才可以在内部赋值 (有点像内部类)
    // struct
    // {
    //     int age;
    // } Person3;
};
```

## 结构体常规定义和使用

```c
void main()
{
    // 结构体的常规使用
    const char *address = "guangdong";

    Person p = {24, "zhangsan", "guangdong"};

    printf("Person == age=%d, name=%s, address=%s\n", p.age, p.name, p.address);

    // Person1 直接定义名称结构体输出
    printf("Person == age=%d, name=%s, address=%s\n", person1.age, person1.name, person1.address);
}
```

## 结构体的嵌套使用

```c
void main()
{
    // 结构体的嵌套
    Person p = {24, "zhangsan", "guangdong"};
    struct Person2 p1 = {12, p, {33}};

    printf("person p = %d, 内部Person %d, %s, %s, 内部Person3 = %d\n", p1.age, p1.person.age, p1.person.name, p1.person.address, p1.p3.age);
    // 可以通过如下的方式修改值
    p1.age = 22;

    // 通过结构体指针操作
    Person2 *p_p = &p1;
    p_p->age = 55;
    p_p->p3.age = 93;
    printf("person p = %d, 内部Person %d, %s, %s, 内部Person3 = %d\n", p1.age, p1.person.age, p1.person.name, p1.person.address, p1.p3.age);
}

```

## 结构体指针和内存分配
```c
int main()
{

    Person *p = (Person *)malloc(sizeof(Person)); // cpp 文件中,需要强转
    printf("p 的大小是 = %ld\n", sizeof(p));      // 8

    p->age = 30;
    // p->name= "ddddd";
    strcpy(p->name, "zhangsan");

    printf("p age是 = %d, name = %s\n", p->age, p->name);
    printf("p 的大小是 = %ld\n", sizeof(p)); // 8

    if (p) // 动态申请后, 需要内存释放
    {
        free(p);
        p = NULL;
    }
}
```

## 结构体的数组
```c
void main()
{
    // 静态内存开辟
    Person pArray[10] = {{}, {}, {43, "lisi", "shenzhen"}, {}, {}, {}};

    printf("pArray - %d, %s, %s\n", pArray[2].age, pArray[2].name, pArray[2].address);

    // 动态内存开辟
    Person *pArray_p = (Person *)malloc(sizeof(Person) * 10);

    pArray_p->age = 45;
    printf("pArray_p = %d\n", pArray_p->age);
    pArray_p += 5; // 指针偏移 (数组和指针差不多, 指针可以认为是一个数组)
    pArray_p->age = 65;

    printf("pArray_p = %d\n", pArray_p->age);

    // 遍历
    int i = 0;
    pArray_p -= 5; // 指针复位, 因为指针已经偏移了
    for (; i < 10; i++)
    {
        printf("pArray_p[%d] = %d\n", i, pArray_p->age);
        pArray_p++;
    }

    if (pArray_p)
    {
        free(pArray_p);
        pArray_p = NULL;
    }
    
    /** 输出打印
     *  pArray - 43, lisi, shenzhen
        pArray_p = 45
        pArray_p = 65
        pArray_p[0] = 45
        pArray_p[1] = -1163005939
        pArray_p[2] = -1163005939
        pArray_p[3] = -1163005939
        pArray_p[4] = -1163005939
        pArray_p[5] = 65
        pArray_p[6] = -1163005939
        pArray_p[7] = -1163005939
        pArray_p[8] = -1163005939
        pArray_p[9] = -1163005939
     */
}
```

## 结构体和结构体指针取别名

```c
typedef Person p_type;
// 结构体指针取别名
typedef Person* person_p; // person_p 是 Person* 指针的别名

void main()
{
    p_type p = {13, "zhangsan", "shandong"};
    printf("Person == age=%d, name=%s, address=%s\n", p.age, p.name, p.address);

    // 结构体指针
    person_p p_p = &p;
    p_p->age = 66;
    printf("Person == age=%d, name=%s, address=%s\n", p_p->age, p_p->name, p_p->address);
}
```


## 联合体和枚举
```c
// 联合体
union PersonU
{
    int age;
    char name[10];
};

int test7(){
    // 联合体只能存在一个，要么是 age ，要么是 name (意思是同一时刻, 只能给一个属性赋值)
    PersonU pu = {12};
    // PersonU pu = {12,"fang"}; // 报错 too many initializers

    printf("pu = %d\n", pu.age); // 不用累加，找最大值 10, 字节对齐都能够被 12 整除, 12 就是sizeOf 的值
    printf("pu sizeof = %ld\n", sizeof(PersonU));
}

enum Type{
    // 枚举类型是一个 int 值, 是累加计数的, 如果不指定 text 默认是 0
    Text = 9, Image
};


int main()
{
    test7();

    printf("enum type = %d\n", Text); 

    printf("enum type = %d\n", Image); // 10 自动累加

    // 输出
    /**
     *  pu = 12
        pu sizeof = 12
        enum type = 9
        enum type = 10
     */
}
```
