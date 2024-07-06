## 结构体大小计算（字节对齐）

### 字节对齐的计算方式
1. 按照最大的字节去计算
2. 算得时候只会按照基本数据类型去算
3. 首先会把所有字节数加起来，是否能够整除最大属性的字节数，如果不够为往上累加，一直加到能整除位置

```c
// 结构体大小计算（字节对齐）
struct Worker
{                  // 定义一个结构体，相当于 java 的 class
    char name[21]; // 21
    int age;       // 4
    double salary; // 8
                   // 如果是:  char name[10] 24 ，char name[18] 32, char name[21]
                   // 那么 size 等于 32, 32 怎么来的？ 18 + 4 +8 = 30 ，32
    //
};

int main()
{
    int size = sizeof(Worker);
    printf("size == %d\n", size); //  size == 40
}
```

