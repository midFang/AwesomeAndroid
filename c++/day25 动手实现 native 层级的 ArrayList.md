# Native ArrayList  原理
> 实际上是对 array 数组(也就是指针)的操作，内存地址的操作,  本质上和 Java ArrayList 差不多, 只是用 c 来写而已, 了解它的整个过程 

## 声明
```c

template<class E>
class ArrayList {
private:
    E *array = nullptr;
    int index = 0;
    int len = 0;
public:
    /** 构造函数 **/
    ArrayList();

    /** 带参构造 **/
    ArrayList(int len);

    /** 析构函数 **/
    ~ArrayList();

    void add(E e);

    /**实现 拷贝构造函数
     *  拷贝构造函数的作用是什么 ？
     * 拷贝构造函数就是对 对象的拷贝一个副本出来了，是因为 c++ 中传递一个对象给函数中， 其实不是传递了一个对象给函数中，
     * 而是传递了一个副本到函数中， 这个函数执行完成了，就结束了，也就是会调用析构函数释放
     * 如果不写拷贝构造函数， 如果对象在函数中传递就会发生 NULL
     * */

    /**  拷贝构造函数:  **/
    ArrayList(const ArrayList &arrayList);

    void remove(int index);

    E get(int index);

    int size();

private:
    void ensureCapacityInternal(int capacity);

    void grow(int capacity);

    /**  拷贝构造函数 **/
};
```

## 实现

```c

/** 模板方法 **/
template<class E>
ArrayList<E>::ArrayList() {
    /** 默认构造函数 **/
}

template<class E>
ArrayList<E>::ArrayList(int len) {
    if (len <= 0) {
        return;
    }

    /** 申请内存 **/
    this->len = len;
    this->array = malloc(sizeof(E) * len);
}

template<class E>
ArrayList<E>::ArrayList(const ArrayList &other) {
    // 拷贝其他对象的长度
    this->len = other.len;

    // 分配新的数组
    this->array = (E *) malloc(sizeof(E) * this->len);

    // 拷贝其他对象的数组内容到新数组
    memcpy(this->array, other.array, sizeof(E) * this->len);

    // 设置当前对象的索引
    this->index = other.index;
}

template<class E>
ArrayList<E>::~ArrayList() {
    if (this->array) {
        free(this->array);
        // 防止野指针
        this->array = NULL;
    }
}

template<class E>
E ArrayList<E>::get(int index) {
    return this->array[index];
}

template<class E>
void ArrayList<E>::remove(int index) {
    /**
     * 1.删除就是指针的位置改变，
     * 2.获取数组发生拷贝
     */

    if (index < 0 || index >= this->index) {
        /** index  索引越界 **/
        return;
    }

    /** 移动数组元素, 后面的数组往前移动 **/
    for (int i = index + 1; i < this->index; ++i) {
        this->array[i - 1] = this->array[i];
    }

    /** 更新数组长度 **/
    this->index -= 1;
}

template<class E>
int ArrayList<E>::size() {
    return this->index;
}

/** 检查数组容量 **/
template<class E>
void ArrayList<E>::ensureCapacityInternal(int capacity) {
    if (this->array == NULL) {
        capacity = 10;
    }

    if (capacity - len > 0) {
        // 数组扩容
        grow(capacity);
    }
}

template<class E>
void ArrayList<E>::grow(int capacity) {
    /** 数组扩容 **/
    /** 扩容大小 **/
    int new_len = len + (len >> 1);

    if (capacity - new_len > 0) {
        new_len = capacity;
    }

    // 创建新的数组， 旧数组复制到新的数组中
    E *new_arr = (E *) malloc(sizeof(E) * new_len);

    if (this->array) {
        // 原来的数据拷贝到 new_arr
        memcpy(new_arr, this->array, sizeof(E) * index);// sizeof(E)*index 字节
        free(this->array);// 内存泄漏, 释放原来的
    }

    this->array = new_arr;
    this->len = new_len;
}

template<class E>
void ArrayList<E>::add(E e) {
    /** 检查数组容量 **/
    ensureCapacityInternal(index + 1);  // Increments modCount!!
    /** 数组赋值 **/
    this->array[index++] = e;
}

```

## 使用 
```c
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_nativeArrayListTest(JNIEnv *env, jobject thiz) {

    LOGI("@@@@ ", "native 层级的 ArrayList 测试1111");

    // new 创建的对象是指针
    ArrayList<int> *arrayList = new ArrayList<int>();

    arrayList->add(1);
    arrayList->add(2);
    arrayList->add(4);
    arrayList->add(6);

    // 迭代
    LOGI("@@@@ ", " ArrayList 个数 %d ", arrayList->size());

    /** 获取中间的某一个值 **/
    LOGI("@@@@", "获取下标 %d 值 %d ", 2, arrayList->get(2));

    for (int index = 0; index < arrayList->size(); ++index) {
        LOGI("@@@@", " index %d value = %d ", index, arrayList->get(index));
    }

    LOGI("@@@@ ", " 删除数组元素 ");

    arrayList->remove(1);

    for (int index = 0; index < arrayList->size(); ++index) {
        LOGI("@@@@", " index %d value = %d ", index, arrayList->get(index));
    }
}
```