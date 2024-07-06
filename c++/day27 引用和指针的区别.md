## 引用本质就是 指针

## 替换
```c++
void change(int &number1)
{                 // 相当于 change(int* number1)
    number1 = 12; // 相当于 *number1 = 12;
}

void main()
{

    int number1 = 10;
    int number2 = 20;

    change(number1);

    cout << number1 << endl;

    getchar();
}
```
