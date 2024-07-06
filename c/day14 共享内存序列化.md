## 序列化本质和作用是什么
- 它的本质在于将内存中的数据转换为一种可以在不同系统之间传输或存储的格式，以便在稍后的时间点或在不同的地点重新构建相同的数据结构或对象。序列化的主要目的是持久化数据，使其能够在不同的环境中进行传输、存储和重建，而不会丢失其原始结构和信息

## Serializable 和 Parcelable 有什么区别
- Serializable 是序列化和反序列化是使用 ObjectOutputStream和ObjectInputStream 来进行操作的，会涉及到 IO，可以用于文件或网络流中
- 而 Parcelable 不涉及 IO，它是直接操作一块共享内存（下面会举例），仅支持在内存中访问，不支持写入文件

## 常见的几种序列化方式有什么区别


## Android 中 Parcelable 的简单使用方式

```
public class MyParcelBean implements Parcelable {

    private int age;
    private String name;
    private boolean sex;

    // 读取的顺序需要和写入的顺序一致，读取的数据才会正确
    protected MyParcelBean(Parcel in) {
        age = in.readInt();
        name = in.readString();
        sex = in.readByte() != 0;
    }

    // 序列化，将数据写入到内存，需要注意的是，这里的顺序怎么存储的，读取的顺序也必须保持一致
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(age);
        dest.writeString(name);
        dest.writeByte((byte) (sex ? 1 : 0));
    }

    @Override
    public int describeContents() {
        return 0;
    }

    // 必须要实现的方法，用于反序列化
    public static final Creator<MyParcelBean> CREATOR = new Creator<MyParcelBean>() {
        @Override
        public MyParcelBean createFromParcel(Parcel in) {
            return new MyParcelBean(in);
        }

        @Override
        public MyParcelBean[] newArray(int size) {
            return new MyParcelBean[size];
        }
    };
}
```
上面代码需要注意的写入数据和读取数据的顺序需要保持一致，至于为什么，下面手写 C 层的代码，就能知道它的原理了

## 创建 Java 写入和读取数据方法

```kotlin
class MyParcel {

    // 用于存储 c对象首地址
    private var mNativePtr: Long = 0L // used by native code

    init {
        System.loadLibrary("ndksimple")

        mNativePtr = createObj() // 创建 Navtive 层级的操作共享内存的对象
    }

    // 创建对象
    private external fun createObj(): Long

    fun writeInt(value: Int) = nativeWriteInt(mNativePtr, value)

    fun readInt() = nativeReadInt(mNativePtr)

    fun setPos(pos: Int) = nativeSetPos(mNativePtr, pos)

    private external fun nativeSetPos(nativePtr: Long, pos: Int)

    private external fun nativeReadInt(nativePtr: Long): Int

    private external fun nativeWriteInt(nativePtr: Long, value: Int)
}
```

## 定义 C 层共享对象

```c
class Parcel {
    // 用于存储数据, 使用 char 数组代表一个字节 (共享内存首地址)
    char *m_data;
    // 用录当前的于记指针位置 （每操作数据，指针位置会变动）
    int m_pos;

public:
    Parcel() {
        m_data = static_cast<char *>(malloc(1024));
        m_pos = 0;
    }


    void writeInt(jint value) {
        // 将 jint 转换为 char 数组
        *reinterpret_cast<int *>(m_data + m_pos) = value;
        // 指针偏移
        m_pos += sizeof(int);
    }

    void setPos(jint pos) {  //恢复指针位置
        m_pos = pos;
    }

    jint readInt() {
        // 实际上是指针的偏移, 数据的读取
        int result = *reinterpret_cast<int *>(m_data + m_pos);
        m_pos += sizeof(int);
        return result;
    }
};

```

## Jni 操作数据

```c
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_day13_MyParcel_nativeWriteInt(JNIEnv *env, jobject thiz,
                                                         jlong m_native_ptr, jint value) {

    // 将 jlong 转换为 Parcel 对象
    Parcel *parcel = reinterpret_cast<Parcel *>(m_native_ptr);
    parcel->writeInt(value);
}

// 创建 c 对象
extern "C"
JNIEXPORT jlong JNICALL
Java_com_midFang_ndksimple_day13_MyParcel_createObj(JNIEnv *env, jobject thiz) {

    Parcel *parcel = new Parcel();
    // 将 parcel 对象的地址传递给 java 层
    return reinterpret_cast<jlong>(parcel);
}
extern "C"
JNIEXPORT jint JNICALL
Java_com_midFang_ndksimple_day13_MyParcel_nativeReadInt(JNIEnv *env, jobject thiz,
                                                        jlong native_ptr) {
    // 将 jlong 转换为 Parcel 对象
    Parcel *parcel = reinterpret_cast<Parcel *>(native_ptr);
    jint result = parcel->readInt();
    return result;
}

extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_day13_MyParcel_nativeSetPos(JNIEnv *env, jobject thiz, jlong native_ptr,
                                                       jint pos) {
    Parcel *parcel = reinterpret_cast<Parcel *>(native_ptr);
    parcel->setPos(pos);
}
```

## 使用
```kotlin
        // Java 对象和 C 对应对应关系
        // android 共享内存序列化
        val myParcel = MyParcel() // 创建对象

        // 写入数据
        myParcel.writeInt(3)
        myParcel.writeInt(8)
        myParcel.writeInt(97)

        // 恢复指针位置
        myParcel.setPos(0)

        // 读取数据
        println("打印共享内存的值")
        println(myParcel.readInt())
        println(myParcel.readInt())
        println(myParcel.readInt())
```

## 总结
以上大体的步骤是
- 在 Java 层级创建的对象与之对应的会在 C 层级创建一个 Parcel 对象，而这个 Parcel 内部维护了一个指针，也可以理解为它是一个数组，可以用来存储数据，而 writeInt readInt 等相关的方法，就是对数据的存储操作，需要注意的是，在 C 层级对数据的存储需要对指针进行偏移，之前说了为什么要保证如何存入数据的，读取的时候顺序也要保持一致，例子中虽然都是写入的是 Int 类型的数据，但是有可能会写入对象，或者 String 类型的数据，而它们的字节大小都不一样，如果你不按照顺序，那么可能读取的数据会不正确
- 我们的数据都是一块连续的内存地址上进行存储的，是通过 C 直接操作的内存，所以它的性能会更好，想对比 Serializable 不涉及 IO, Android 中的 Parcelable 的数据存储本质是共享内存