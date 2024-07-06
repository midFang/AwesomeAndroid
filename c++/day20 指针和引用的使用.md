## 两个数的替换

```c++
void swap1(int *num1, int *num2)
{
    int temp = *num1;
    *num1 = *num2;
    *num2 = temp;
}

void swap2(int &num1, int &num2)
{
    int temp = num1;
    num1 = num2;
    num2 = temp;
}

void main()
{
    // 引用类型 , 和直接赋值有什么区别有关系

    int a = 10;

    // int b = a; // b 是栈中的一块新的内存地址, 和 a是不一样的, 可以认为是a的值给b了

    int &b = a;
    // 引用赋值是这样的, (和 java 中的引用赋值差不多), 其实引用赋值就是地址赋值, 由于两个地址一样, 修改其中一个, a,b都会改变
    b = 100;

    cout << "a 的地址: " << &a << endl;
    cout << "b 的地址: " << &b << endl;
    cout << "a 的值: " << a << endl;
    cout << "b 的值: " << b << endl;

    int num1 = 42;
    int num2 = 89;

    cout << "num1 的值: " << num1 << endl;
    cout << "num2 的值: " << num2 << endl;

    // 指针方式的数据交换, 操作的是指针, 传递给方法里面了
    swap1(&num1, &num2);
    cout << "num1 的值: " << num1 << endl;
    cout << "num2 的值: " << num2 << endl;

    // 引用传递到方法里面, 其实也算是指针了
    swap2(num1, num2);
    cout << "num1 的值: " << num1 << endl;
    cout << "num2 的值: " << num2 << endl;
}
```

## 函数重载和默认参数
```c++
int add(int &num1){ 

}
int add(int &num1, int &num2, bool isAdd = 0) // 和 kotlin 差不多, 可以给一个默认值
{
    // bool 类型, 1, -1, 99  是true , 0 是false, 所以 0 是false, 非0都是true
    if (isAdd)
    {
        printf("is add type true \n");
        return num1 + num2;
    }
    else
    {
        printf("is add type false \n");
        return num1 - num2;
    }
}

void main2()
{
    // 函数重载和默认参数
    int a = 1;
    int b = 2;

    int result = add(a, b); // 默认调用了 3 个参数的方法, 即使是两个参数

    printf("result = %d\n", result);
}

```