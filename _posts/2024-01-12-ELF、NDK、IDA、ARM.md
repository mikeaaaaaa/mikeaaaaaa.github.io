### ELF文件

ELF文件：可执行、可链接的二进制文件,是linux中的二进制文件。可以理解成windws中的PE文件。在Android中可理解为dll，方便函数的移植，常用于Android逆向难度

[参考链接](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1282554)

#### ELF文件组成(**从上往下**)：

+ **ELF Header**：文件头，描述文件的基本信息
+ **program Header Table**:程序头表，描述进程映像的布局
+ **Section Table**：节区表，描述文件的各个节区
  + .text:代码段，存放程序的指令
  + .data：数据段，存放已初始化的全局变量和静态变量
  + .rodata：只读数据段
  + bss：未初始化数据段，存放未初始化的全局变量和静态变量
  + symtab：符号表，存放符号信息
  + .strtab：字符串表，存放字符串数据
  + .dynsym：动态符号表，存放动态链接需要的符号信息
  + dynamic：动态链接信息，存放动态连接器需要的信**息**
+ **Section Header**：节区头

![img](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1.png)

 **在安卓源码中，有个elf.h文件，这个文件定义了我们解析时需要用到的所有数据结构，并且给出了参考注释，是很好的参考资料。** 

位置：![1700708452601](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700708452601.png)

#### ELF头部

ELF头部的数据结构在elf.h文件已经给出

![1700708701919](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700708701919.png)

每个字段详细解释：![1700708846856](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700708846856.png)

对于e_type字段：![1700708965965](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700708965965.png)



#### 程序头

elf.h文件给出的结构：![1700709075878](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700709075878.png)

每个字段解释：![1700709093681](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700709093681.png)

​	对于p_type字段：![1700709232606](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700709232606.png)



#### 节区头

![1700709291549](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700709291549.png)

字段解释：![1700709358829](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700709358829.png)

节区类型：![1700717612467](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700717612467.png)

#### 字符串解析

 在elf头部中有个e_shstrndx字段，该字段指明了.shstrtab节区头部是文件中第几个节区头部，我们可以根据这找到.shstrtab节区的偏移地址，然后读取出来，就可以为每个节区名字赋值了，然后就可以顺着锁定剩下的两个字符串节区。

    在elf文件中，字符串表示方式如下：字符串的头部和尾部用标示字节00标志，同时上一个字符串尾部标识符00作为下一个字符串头部标识符。例如我有两个紧邻的字符串分别是a和b，那么他们在elf文件中16进制为00 97 00 98 00。

#### .dynamic解析

![1700718165235](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/1700718165235.png)

#### .text解析

​	.text节区存储着可执行指令，我们可以通过节区头部的名字锁定.text的偏移地址和大小，找到该节区后，我们会发现这个节区存储的就是arm机器码，直接照着指令集翻译即可，没有其他的结构。

### NDK

#### NDK开发

[参考链接1](https://bbs.kanxue.com/thread-267367.htm)

[参考链接2](https://blog.csdn.net/weixin_34583170/article/details/94864797)

#### 介绍

 NDK是开发套件，是**一整套编译和调试的工具集** ，JNI（java native interface）才是调用的框架，所以可以说是JNI的开发。不过NDK是Android提供的开发套件，而JNI可不是。JNI是调用Native语言的一种特性，通过JNI可以使得Java于C/C++机型交互，可以在java代码中调用C/C++代码，或者在C/C++调用JAVA代码.JNI**不是android平台特有**，但凡**有JVM的地方都支持**.

**NDK开发的so不再具有跨平台性** ，基于不同架构的平台，需要编译不同平台支持的二进制接口ABI(Application Binary Interface) 。

为什么要有JNI：

- Java虽然跨平台，开发快捷，但是性能不足，没有C/C++性能强悍
- Java代码易被反编译，安全性没有C/C++好
- 如果Java程序可以调用目前已存在的海量C/C++库完成复用，将大大节省开发时间并提高Java应用程序性能



#### JNI的两种注册方法：

为啥需要注册，注册就是让jni知道你这个函数的存在呗 

##### JNI静态注册：

- 优点: 理解和使用方式简单, 属于傻瓜式操作, 使用相关工具按流程操作就行, 出错率低

- 缺点: 当需要更改类名,包名或者方法时, 需要按照之前方法重新生成头文件, 灵活性不高

  

  **c++文件**

```c++
#include <jni.h> // JNI头文件，提供了JNI函数和数据类型的定义
#include <string> // C++标准库的string类

// 声明一个jni函数，该函数将会被Java代码调用
// JNIEXPORT表示这个函数是可导出的，并且可以被其他代码使用
// jstring表示这个函数返回的是一个Java字符串对象
// JNICALL是JNI函数的调用约定
// Java_com_example_ndkdemo_MainActivity_stringFromJNI是JNI函数的命名规则，与Java中对应的方法名对应
// Java打头，1包名,2类名,3方法名字;"_"号隔开
extern "C" JNIEXPORT jstring JNICALL
Java_com_example_ndkdemo_MainActivity_stringFromJNI(
        JNIEnv* env, // JNIEnv是指向JNI环境的指针，可以用来访问JNI提供的功能
        jobject /* this */) { // jobject是指向Java对象的指针，在本例中并没有使用

    std::string hello = "Hello from C++"; // 创建一个C++字符串对象
    return env->NewStringUTF(hello.c_str()); // 将C++字符串对象转换为Java字符串对象并返回
}

```

**使用cmake**：

```python
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# 设置构建本地库所需的CMake的最小版本要求
cmake_minimum_required(VERSION 3.22.1)

# 声明和命名项目
project("ndkdemo")

# 创建并命名一个库，设置其类型为STATIC或SHARED，并指定源代码的相对路径
# 可以定义多个库，CMake会为您构建它们
# Gradle会自动将共享库打包到APK中
add_library(
        # 设置库的名称
        ndkdemo

        # 设置库类型为共享库
        SHARED

        # 提供源文件的相对路径
        native-lib.cpp)

# 搜索指定的预构建库并将路径存储为变量。
# 由于CMake默认在搜索路径中包含系统库，因此您只需指定要添加的公共NDK库的名称。
# CMake会在完成构建之前验证该库是否存在。
find_library(
        # 设置路径变量的名称
        log-lib

        # 指定要让CMake定位的NDK库的名称
        log)

# 指定CMake应链接到目标库的库。
# 您可以链接多个库，例如在此构建脚本中定义的库、预构建的第三方库或系统库。
target_link_libraries(
        # 指定目标库
        ndkdemo

        # 将目标库链接到NDK中包含的log库
        ${log-lib})

```

**java文件**

```java
public class MainActivity extends AppCompatActivity {

    // Used to load the 'ndkdemo' library on application startup.
    static {
        System.loadLibrary("ndkdemo"); // 加载名为"ndkdemo"的库
    }

    private ActivityMainBinding binding; // 声明一个ActivityMainBinding变量

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityMainBinding.inflate(getLayoutInflater()); // 使用ViewBinding将布局文件解析为一个ActivityMainBinding对象
        setContentView(binding.getRoot()); // 将Activity的布局设置为根布局

        // Example of a call to a native method
        TextView tv = binding.sampleText; // 获取布局文件中的TextView控件
        tv.setText(stringFromJNI()); // 调用本地方法stringFromJNI()并将其返回的字符串设置为TextView的文本内容
    }

    /**
     * A native method that is implemented by the 'ndkdemo' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI(); // 声明一个native方法stringFromJNI()
}
```



##### JNI动态注册：

动态注册相较于静态注册，是比较麻烦的，但是它有个好处具体调用的函数名称有我们自己定义，而不用按照静态注册那样把包名等等七七八八的东西全部要写上。不过有个麻烦的东西要写java方法的签名，各个签名的基本变量如下表 

```c++
#include <jni.h>
#include <string>

extern "C" {

JNIEXPORT jstring JNICALL Java_com_example_ndkdemo_MainActivity_nativeGetStringFromJNI(JNIEnv* env, jobject obj) {
    std::string hello = "Hello wuaipojie";
    return env->NewStringUTF(hello.c_str());
}

// 定义本地方法注册函数
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }

    // 定义要注册的本地方法(关键部分，传入方法名，参数)
    JNINativeMethod methods[] = {
        {"nativeGetStringFromJNI", "()Ljava/lang/String;", reinterpret_cast<void*>(Java_com_example_ndkdemo_MainActivity_nativeGetStringFromJNI)}
    };

    // 获取类引用
    jclass clazz = env->FindClass("com/example/ndkdemo/MainActivity");
    if (clazz == nullptr) {
        return -1;
    }

    // 注册本地方法
    if (env->RegisterNatives(clazz, methods, sizeof(methods) / sizeof(methods[0])) < 0) {
        return -1;
    }

    return JNI_VERSION_1_6;
}

} // extern "C"

```

**数据类型**

下面是一些常见的C++数据类型和它们在Java中的对应关系，以及它们在JNI动态注册中的数据类型签名（signature）：

| C++ 数据类型  | Java 数据类型 | JNI 数据类型签名      |
| ------------- | ------------- | --------------------- |
| jint          | int           | "I"                   |
| jboolean      | boolean       | "Z"                   |
| jbyte         | byte          | "B"                   |
| jchar         | char          | "C"                   |
| jshort        | short         | "S"                   |
| jlong         | long          | "J"                   |
| jfloat        | float         | "F"                   |
| jdouble       | double        | "D"                   |
| jobject       | Object        | "Ljava/lang/Object;"  |
| jstring       | String        | "Ljava/lang/String;"  |
| jarray        | Array         | "[elementType"        |
| jobjectArray  | Object[]      | "[Ljava/lang/Object;" |
| jbooleanArray | boolean[]     | "[Z"                  |
| jbyteArray    | byte[]        | "[B"                  |
| jcharArray    | char[]        | "[C"                  |
| jshortArray   | short[]       | "[S"                  |
| jintArray     | int[]         | "[I"                  |
| jlongArray    | long[]        | "[J"                  |
| jfloatArray   | float[]       | "[F"                  |
| jdoubleArray  | double[]      | "[D"                  |

在JNI动态注册中，需要使用正确的数据类型签名来声明本地方法。例如，如果你要注册一个返回`int`类型的本地方法，其数据类型签名应为`I`。



### IDA

#### 简介：

IDA Pro（Interactive Disassembler Professional）是一款功能强大的交互式反汇编和调试工具，广泛应用于软件逆向工程、漏洞分析和二进制代码分析。它支持多种处理器架构和可执行文件格式，包括但不限于x86、ARM、MIPS、PowerPC等。通过使用IDA Pro，可以对程序进行静态分析、动态调试和代码修改等操作。

[使用教程](https://bbs.kanxue.com/thread-266021.htm)

[超级详细教程](https://blog.csdn.net/freeking101/article/details/106531466)

#### 快捷键：

| 快捷键      | 功能                         |
| ----------- | ---------------------------- |
| Esc         | 回到上一个位置               |
| Enter       | 跳转到当前光标处的地址       |
| -           | 折叠代码                     |
| +           | 展开代码                     |
| *           | 创建一个结构                 |
| Alt + A     | 手动定义一个数组             |
| Alt + F     | 寻找直接引用的函数           |
| Alt + G     | 跳转到特定的地址             |
| Alt + T     | 显示调用树                   |
| Alt + X     | 重命名                       |
| Ctrl + G    | 快速跳转到指定地址           |
| Ctrl + J    | 显示引用列表                 |
| Ctrl + K    | 显示 XREF 到选中的函数/数据  |
| Ctrl + N    | 创建一个函数                 |
| Ctrl + Q    | 快速重命名                   |
| Ctrl + X    | 显示从选中的函数/数据的 XREF |
| Ctrl + E    | 显示结构类型                 |
| Ctrl + R    | 手动定义一个数据结构         |
| Ctrl + W    | 打开函数列表                 |
| Ctrl + D    | 以十进制显示当前值           |
| Ctrl + B    | 以二进制显示当前值           |
| Ctrl + H    | 以十六进制显示当前值         |
| Space       | 在图形/文本视图中切换        |
| shift + f12 | 打开字符串窗口               |
| F5          | 转伪C代码                    |

#### Patch方式

1.keypatch插件快速修补
快捷键：Ctrl+Alt+k
录完教程发现Patching这个插件更好用些，下节课再做演示
Patching - <https://github.com/gaasedelen/patching>
2.ARM TO Hex
[ARM to HEX](https://armconverter.com/)

### ARM基础知识

- **x86 架构：** 由英特尔（Intel）和 AMD 等公司生产的处理器采用的指令集架构，包括 32 位和 64 位版本。
- **ARM 架构：** 在许多移动设备和嵌入式系统中广泛使用的处理器架构，其设计注重低功耗和高效能。

##### 常见寻址方式

| 寻址方式                       | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 立即数寻址                     | 直接使用立即数值作为操作数，例如：`MOV R0, #5`               |
| 寄存器直接寻址                 | 使用寄存器中的值作为操作数，例如：`MOV R0, R1`               |
| 寄存器间接寻址                 | 使用寄存器中的值作为内存地址，访问该地址中的数据，例如：`LDR R0, [R1]` |
| 寄存器相对寻址                 | 使用寄存器中的值加上一个立即偏移量作为内存地址，例如：`LDR R0, [R1, #4]` |
| 寄存器变址寻址                 | 使用两个寄存器中的值相加作为内存地址，例如：`LDR R0, [R1, R2]` |
| 带有变址寄存器的寄存器相对寻址 | 使用寄存器中的值加上另一个寄存器的值乘以一个比例因子作为内存地址，例如：`LDR R0, [R1, R2, LSL #2]` |
| 堆栈寻址                       | 使用堆栈指针寄存器（如SP）进行操作，例如：`PUSH {R0, R1}` 或 `POP {R0, R1}` |

##### 压栈和出栈指令

| 指令类型 | 指令示例             | 描述                                         |
| -------- | -------------------- | -------------------------------------------- |
| 压栈     | `PUSH {R0, R1}`      | 将寄存器R0和R1的内容压入堆栈中               |
| 压栈     | `PUSH {R0-R5}`       | 将寄存器R0到R5的内容压入堆栈中               |
| 压栈     | `STMDB SP!, {R0-R5}` | 将寄存器R0到R5的内容压入堆栈中（与PUSH等效） |
| 出栈     | `POP {R0, R1}`       | 从堆栈中弹出数据，恢复到寄存器R0和R1中       |
| 出栈     | `POP {R0-R5}`        | 从堆栈中弹出数据，恢复到寄存器R0到R5中       |

##### 跳转指令

| 指令类型   | 指令示例   | 描述                                                         |
| ---------- | ---------- | ------------------------------------------------------------ |
| 无条件跳转 | `B label`  | 无条件跳转到标签`label`指向的位置                            |
| 子程序调用 | `BL label` | 调用子程序，将当前指令的下一条指令地址存入链接寄存器（LR），然后跳转到标签`label`指向的位置 |
| 子程序返回 | `BX LR`    | 返回子程序调用前的位置，跳转到链接寄存器（LR）中存储的地址   |
| 寄存器跳转 | `BX Rn`    | 跳转到寄存器Rn中存储的地址                                   |

##### 算术运算指令

汇编中也可以进行算术运算， 比如加减乘除，常用的运算指令用法如表 所示：

| 指令               | 计算公式                | 备注                         |
| ------------------ | ----------------------- | ---------------------------- |
| ADD Rd, Rn, Rm     | Rd = Rn + Rm            | 加法运算，指令为 ADD         |
| ADD Rd, Rn, #immed | Rd = Rn + #immed        | 加法运算，指令为 ADD         |
| ADC Rd, Rn, Rm     | Rd = Rn + Rm + 进位     | 带进位的加法运算，指令为 ADC |
| ADC Rd, Rn, #immed | Rd = Rn + #immed + 进位 | 带进位的加法运算，指令为 ADC |
| SUB Rd, Rn, Rm     | Rd = Rn - Rm            | 减法                         |
| SUB Rd, #immed     | Rd = Rd - #immed        | 减法                         |
| SUB Rd, Rn, #immed | Rd = Rn - #immed        | 减法                         |
| SBC Rd, Rn, #immed | Rd = Rn - #immed - 借位 | 带借位的减法                 |
| SBC Rd, Rn ,Rm     | Rd = Rn - Rm - 借位     | 带借位的减法                 |
| MUL Rd, Rn, Rm     | Rd = Rn * Rm            | 乘法 (32 位)                 |
| UDIV Rd, Rn, Rm    | Rd = Rn / Rm            | 无符号除法                   |
| SDIV Rd, Rn, Rm    | Rd = Rn / Rm            | 有符号除法                   |

##### 逻辑运算

汇编语言的时候也可以使用逻辑运算指令，常用的运算指令用法如表 所示：
![img](/assets/image/2024-01-12-ELF、NDK、IDA、ARM/35d0832fb2523805c2a2165ec5458caa.png)

##### 寄存器：

在ARM64架构下，CPU提供了33个寄存器, 其中前31个（0~30）是通用寄存器 (`general-purpose integer registers`)，最后2个(31,32)是专用寄存器（`sp` 寄存器和 `pc` 寄存器)。 

前面0~30个通用寄存器的访问方式有2种：

- 当将其作为 `32bit` 寄存器的时候，使用 `W0` ~ `W30` 来引用它们。（数据保存在寄存器的低32位）
- 当将其作为 `64bit` 寄存器的时候，使用 `X0` ~ `X30` 来引用它们

第31个专用寄存器的访问方式有4种:

- 当将其作为 `32bit` 栈帧指针寄存器（`stack pointer`) 的时候，使用 `WSP` 来引用它。
- 当将其作为 `62bit` 栈帧指针寄存器（`stack pointer`) 的时候，使用 `SP` 来引用它。
- 当将其作为 `32bit` 零寄存器( `zero register` )的时候，使用 `WZR` 来引用它。
- 当将其作为 `62bit` 零寄存器( `zero register` )的时候，使用 `ZR` 来引用它。

了解重要的寄存器的作用：

| 寄存器         | 说明                         |
| -------------- | ---------------------------- |
| X0 寄存器      | 用来保存返回值（或传参）     |
| X1 ~ X7 寄存器 | 用来保存函数的传参           |
| X8寄存器       | 也可以用来保存返回值         |
| X9 ~ X28寄存器 | 一般寄存器，无特殊用途       |
| x29(FP)寄存器  | 用来保存栈底地址             |
| X30 (LR)寄存器 | 用来保存返回地址             |
| X31(SP) 寄存器 | 用来保存栈顶地址             |
| X31(ZR)寄存器  | 零寄存器，恒为0              |
| X32(PC)寄存器  | 用来保存当前执行的指令的地址 |

##### arm内存布局

ARM64的内存布局，首先一个ARM64的进行会拥有一个非常大的虚拟内存映射空间，其中又分为两大块 ：

- 内核地址（`0xffff_ffff_ffff_ffff` ~ `0xffff_0000_0000_0000`范围的256TB的寻址空间),
- 用户地址 (`0x0000_ffff_ffff_ffff` ~ `0x0000000000000_0000`范围的256TB的寻址空间) 。

这里我们只关心用户地址，其中有分为两大块：

- 栈内存( `Stack`)，从高位向低位生长。
- 堆内存 ( `Heap` ), 从低位向高位生长。

其中我们知道栈内存首先是按照线程为单元的，每个线程都有自己的栈内存块，著名的 `StackOverflow` 所指的就是线程的栈溢出。然后每个线程的栈内存又可以根据函数的调用层级关系分为不同的栈帧( `Stack Frame` )。

与每个 **栈帧有关的寄存器：**

- `pc` 寄存器，记录当前CPU正在哪个指令。
- `sp` 寄存器，记录当前栈顶。
- `fp` 寄存器，记录当前栈的栈底。
- `lr` 寄存器，记录当前栈的返回地址，即这个函数调用完成后应该返回到哪里。

##### 写一个helloWorld程序

写一个 `.s` 程序

首先开头：

```assembly
.text
    .file   "main.c"
    .globl  main                            // -- Begin function main
    .p2align    2
    .type   main,@function
main:                                   // @main
// %bb.0:
    sub sp, sp, #32                     // =32   申请32bytes的栈空间
    stp x29, x30, [sp, #16]             // 16-byte Folded Spill  将 FP(x29), LR(x30) 保存在栈上。即在准备调用 printf 函数之前，将现在咱们 main 函数的 lr 和 fp 寄存器（以及其他所有需要保存的寄存器）的数据都先备份到栈内存上面。以 ST 开头的指令都是将寄存器的值 Store 到内存地址上。
    add x29, sp, #16                    // =16   缩小栈大小16bytes 
    mov w8, wzr                         // 将 zero寄存器的值0 移动到 w8 寄存器
    stur    wzr, [x29, #-4]             // 
    adrp    x0, .L.str
    add x0, x0, :lo12:.L.str
    str w8, [sp, #8]                    // 4-byte Folded Spill
    bl  printf
    ldr w8, [sp, #8]                    // 4-byte Folded Reload
    mov w0, w8
    ldp x29, x30, [sp, #16]             // 16-byte Folded Reload
    add sp, sp, #32                     // =32
    ret
.Lfunc_end0:
    .size   main, .Lfunc_end0-main
                                        // -- End function
    .type   .L.str,@object                  // @.str
    .section    .rodata.str1.1,"aMS",@progbits,1
.L.str:
    .asciz  "Hello World!\n"
    .size   .L.str, 14

    .ident  "Android (7155654, based on r399163b1) clang version 11.0.5 (https://android.googlesource.com/toolchain/llvm-project 87f1315dfbea7c137aa2e6d362dbb457e388158d)"
    .section    ".note.GNU-stack","",@progbits
```

其中以 `.`开头的汇编指令（Assembler Directive）又被称之为伪指令，因为其不属于`ARM` 指令，其中  `.global main` 指令表示：文件里面有一个 叫做 `main`的函数将要被导出。

在 `ARM`汇编中，凡是以 `:`结尾的都会被视为 `label`，在这里我们定义一个叫做 `main` 的标签，并且使用 `.type` 伪指令定义这个标签的类型是一个函数(`function`)，到此我们就定义了我们的 `main` 函数。 

ARM汇编的格式 ：`<opcode> {<cond>} {S} <Rd>,<Rn>,<shifter_operand>`

- `opcode`: 为指令，在我们第一句的指令是 `sub` ,表示减法。
- `Rd`: 为指令操作**目的寄存器**，在我们第一句中是 `sp` 寄存器。
- `Rn`: 为指令第一**源操作数**，在我们第一句中是 `sp` 寄存器
- `shifter_operand`: 数据处理指令，这里我们第一句是**立即数寻址**，即 `#32`

因此 `sub sp, sp, #32` === `sp = sp - 32`

[具体的分析过程](https://zhuanlan.zhihu.com/p/388683540)：



在写下了这14句汇编以后，我们就可以使用 `clang` 编译器将其编译成可执行的二进制文件： 

>  aarch64-linux-android29-clang -o main_arm main.S