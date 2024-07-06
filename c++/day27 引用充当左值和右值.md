# 引用做左值和右值的问


## 局部函数做右值
```c++
 int get()
 {
     int number = 10;
     return number;
 }
 
 void main(){
	 int number1 = get(); // 10
     cout << number1 << endl;
 }
 
```

局部函数 get() 调用完成后, 会被复制作为返回值,赋值给 number1 变量, 所以 number1 的值是 10

## 局部函数引用做右值

```c++
 int &get1()
 {
     int number11 = 10;
     return number11;
 }


 void main()
 {
      int &number3 = get1(); //? 会变成野指针 地址: 12478
      cout << number3 << endl;
 }
```

局部函数 get1() 调用完成后, 返回了指向 number11 的引用, 但是由于这个函数调用完成之后, number11 都被销毁了, 所以返回的指针指向了那块被销毁的区域了, 也就变成了野指针 (和上面不一样的是, 返回值被拷贝给 number1 变量了)

## 局部函数引用做左值

```c++
class Sudent
{
private:
    string name;

public:
    Sudent(string name)
    {
        this->name = name;
    }

public:
    string &getName()
    { 	// Java 想都别想
        return this->name;
    }
};

void main()
{
    Sudent stu = Sudent("fang");

    cout << stu.getName() << endl;

    stu.getName() = "fanidgfg";

    cout << "充当左值修改：" << stu.getName() << endl; // 打印 fanidgfg
}
```