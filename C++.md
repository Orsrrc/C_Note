# C语言知识点

-------

## Main函数

main函数实际上有三个参数，不过在市面上的大多数编译器都默认去掉了，实际使用的时候可以添加

```cpp
int main(int argc, char *argv[], char ** env);
```

- `argc`：传入参数的个数
- `argv`：传入的参数值
- `env`：环境变量

对于`env` 大多用在Linux环境下，其本质就是一个环境变量

![image-20250722104947286](./C++.assets/image-20250722104947286.png)

对于**argc 总是大于等于1的，其默认有一个参数就是.exe文件**，其余的我们要从命令行中传入的参数从第二个开始计算`.exe x y z`。

## 库文件

为了保护我们代码开发者的知识产权 我们给用户提供功能，但不能提供源代码

因此，我们在编写API的时候，就需要将接口定义在.h文件中，将函数实现写在.cpp文件中，然后将其编译为静态库或者动态库，提供给用户。即发送文件给用户的时候，不能够提供.cpp文件（如果是开源项目则可以提供）。

linux/mac静态库

```shell
g++ -c your_api_impl.cpp -I./include -fPIC
ar rcs lib/libyourlib.a your_api_impl.o
```

linux/mac动态库

```shell
g++ -shared -fPIC -o lib/libyourlib.so your_api_impl.cpp -I./include
```

windows：基于virtual studio

```shell
cl /c your_api_impl.cpp /I.\include
lib your_api_impl.obj /OUT:lib\yourlib.lib
# 或者动态库
cl /LD your_api_impl.cpp /I.\include /link /OUT:bin\yourlib.dll
```

用户在使用时引入对应的.h文件

编译时添加库即可

```c
# Linux
g++ demo.cpp -I./include -L./lib -lyourlib -o demo

# Windows
cl demo.cpp /I.\include /link yourlib.lib
```

符号隐藏

```c
// 编译时隐藏内部符号
g++ -fvisibility=hidden -shared -fPIC -o libyourlib.so your_api_impl.cpp
```

### 动态库与静态库的区别

#### 静态库（Static Library）

- **编译时链接**：代码在编译时被复制到最终的可执行文件中
- **文件扩展名**：`.a`（Linux）、`.lib`（Windows）
- **独立运行**：生成的可执行文件不依赖外部库文件

#### 动态库（Dynamic Library）

- **运行时链接**：代码在程序运行时才被加载到内存
- **文件扩展名**：`.so`（Linux）、`.dll`（Windows）、`.dylib`（Mac）
- **依赖关系**：可执行文件需要动态库文件才能运行

### 静态库使用步骤

```bash
# 1. 编译静态库
g++ -c mylib.cpp -o mylib.o
ar rcs libmylib.a mylib.o

# 2. 链接到主程序
g++ main.cpp -L. -lmylib -o myapp

# 3. 运行（无需额外文件）
./myapp
```

动态库使用流程

```bash
# 1. 编译动态库
g++ -shared -fPIC -o libmylib.so mylib.cpp

# 2. 链接到主程序
g++ main.cpp -L. -lmylib -o myapp

# 3. 运行前设置库路径
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
./myapp
```



## 字面量（Literal）

是编程中**直接表示固定值**的符号，相当于“**所见即所得**”的数据表达方式。它在代码中直接写出具体值，无需通过变量存储或额外计算生成。

字面量是常量，程序运行时无法修改其值。例如在 C 语言中，`char *str = "abc";` 的字符串字面量内容不可在代码层面进行更改（存储在只读内存）。

字面量通常用于字符串。

## override(重载)

`override` 是 C++11 引入的关键字，旧版本需通过其他方式（如 IDE 提示）检查重载正确性。在函数的参数后面添加

例如`demo(int i , int j) override;` 

## 二级指针

一个指针  保存一个指针的地址 而不是指针所指向的内容的地址

```C
int** point_name;
```

指向一块内存空间的首地址的指针为一级指针

保存该一级指针的地址的指针称为二级指针  依次类推

使用场景

保存一个一级指针a的地址  完全可以通过另外一个一级指针b来完成

但是

当我们要修改a指针所指向的内存空间时 用b无法完成

b 为a指针的地址

*b为该地址中的内容  即a指针所指内存空间的地址

&b 为b的地址



而若使用二级指针c来保存a的地址

c 里面存放指针a的地址

*c代表指针a中存放的值

```c
#include<stdio.h>

int main(void)
{
        int x = 50;
        int* a = &x;
        int* b = &a;
        int** c = &a;

        printf("a point\n");
        printf("a == %p\n", a);
        printf("*a == %d\n", *a);
        printf("*a == %p\n", *a);
        printf("&a == %p\n", &a);

        printf("\nb point\n");
        printf("*b == %p\n", *b);
        printf("*b == %d\n", *b);
        printf("b == %p\n",  b);
        printf("&b == %p\n", &b);

        printf("\nc point\n");
        printf("*c == %p\n", *c);
        printf("&c == %p\n", &c);
        printf("c == %p\n", c);

        printf("\nchange a ad\n");

        int y = 7;
        int z = 8;
        *c = &y;
        printf("a == %p\n", a);
        *b = &z;
        printf("a == %p\n", a);

        return 0;
}


a point
a == 0xffffe70e5324
*a == 50
*a == 0x32
&a == 0xffffe70e5330

b point
*b == 0xe70e5324
*b == -418491612
b == 0xffffe70e5330
&b == 0xffffe70e5338

c point
*c == 0xffffe70e5324
&c == 0xffffe70e5340
c == 0xffffe70e5330

change a ad
a == 0xffffe70e5328
a == 0xffffe70e532c

 发现修改a指针所指向的值通过一级指针或者二级指针都可以修改
  二级指针更高级的一点在于函数传递
  
  
  
  #include <stdio.h>

void updatePointer(int **ptr) {
    int y = 20;
    *ptr = &y;
}

int main() {
    int x = 10;
    int *ptr1 = &x;

    printf("Before update: Value pointed by ptr1: %d\n", *ptr1);

    // 传递 ptr1 的地址，以便在函数中修改 ptr1 的值
    updatePointer(&ptr1);//传入一个地址 用二级指针来接收

    printf("After update: Value pointed by ptr1: %d\n", *ptr1);

    return 0;
}
```

## 联合体和结构体

~~~C++
union UN
{
	char a;
  int b;
} //联合体  a b共用同一块内存空间

struct UN
{
	char a;
  int b;
}//结构体 a b 有各自独有的一块内存空间
~~~



## goto

```c
again:
		语句;
		goto again;
```

![image-20240125130901399](C++.assets/image-20240125130901399.png)



## 数组大小的计算

```c
char str[] = {0x1, 0x2, 0x3, 0x4};
sizeof(str);
sizeof(&str);
```

sizeof(str)能正确返回数组的大小，sizeof(&str)求出的是str这根指针所占内存的大小。

换句话说，sizeof()传入一段内存的首地址，然后能够从这个首地址遍历到这段内存的末尾，然后计算得出读过几个字节。

本质上str也是一个指针，sizeof(str)能求出内存所占地址大小，是因为这是编译器帮忙计算的，sizeof也是编译器的关键字

## 文件操作

`FILE* file = NULL;`

`fopen_s(&file, filePath, "rb") `

VS下加上了s后缀，从filePath中读出数据了以后，按照`“rb”`方式即二进制方式读出，其文件指针放在file里面。其中，如果打开失败，返回NULL，那么可以通过判断file是否为NULL，来判断是否打开成功。

`fopen_s(&file, fileName, "wb");`

fopen还可以用来创建文件，即传入的路径如果不存在该文件，则创建一份，“wb”，以写的方式。

`fseek(file, 0, SEEK_END) `

**打开文件的时候，光标指向的是第一个字节**。SEEK_END，即让它指向末尾

`ULONG fileLen = ftell(file);`

读取文件长度

当文件的光标指向最后一个字节以后，我们想要让它指向文件起始字节，有两种方法，一个是关闭文件后重新打开，另一个是调用`rewind()`函数，让光标回到开头。

`fclose(file) `文件的关闭与打开必须要成对出现，即打开了一个文件就必须要关闭，否则可能会导致内存泄露。





## 宽字符与窄字符



## 宏开关





## 枚举与枚举类

### 枚举

```c
enum 枚举名称 {
    枚举常量1,
    枚举常量2,
    // ...
};
```

自定义枚举数据

```c
enum Status {
    SUCCESS = 200,
    NOT_FOUND = 404,
    ERROR = 500
};

// 部分赋值（未赋值的常量会自动递增）
enum Color {
    RED = 1,
    GREEN,  // 自动为 2
    BLUE    // 自动为 3
};
```

typedefine 简化代码

```c
typedef enum {
    OFF,
    ON
} SwitchState;

int main() {
    SwitchState light = ON; // 直接使用别名
    return 0;
}
```

### 枚举类                                                     

基本语法

```c
enum class 枚举名称 {
    枚举常量1,
    枚举常量2,
    // ...
};
```

指定枚举类型

```c
#include <iostream>

// 指定底层类型为 char
enum class Status : char {
    OK = 'O',
    ERROR = 'E',
    PENDING = 'P'
};

// 指定底层类型为 short
enum class Flags : unsigned short {
    READ = 0x01,
    WRITE = 0x02,
    EXECUTE = 0x04
};

int main() {
    Status s = Status::OK;
    Flags f = Flags::READ;
    
    std::cout << "Status size: " << sizeof(s) << " bytes" << std::endl; // 1 byte
    std::cout << "Flags size: " << sizeof(f) << " bytes" << std::endl;  // 2 bytes
    
    return 0;
}
```

字符串转换函数

```c
#include <iostream>
#include <string>
#include <unordered_map>

enum class LogLevel {
    DEBUG,
    INFO,
    WARNING,
    ERROR
};

// 转换为字符串
std::string to_string(LogLevel level) {
    switch(level) {
        case LogLevel::DEBUG: return "DEBUG";
        case LogLevel::INFO: return "INFO";
        case LogLevel::WARNING: return "WARNING";
        case LogLevel::ERROR: return "ERROR";
        default: return "UNKNOWN";
    }
}

// 从字符串转换
LogLevel from_string(const std::string& str) {
    static const std::unordered_map<std::string, LogLevel> mapping = {
        {"DEBUG", LogLevel::DEBUG},
        {"INFO", LogLevel::INFO},
        {"WARNING", LogLevel::WARNING},
        {"ERROR", LogLevel::ERROR}
    };
    
    auto it = mapping.find(str);
    return (it != mapping.end()) ? it->second : LogLevel::INFO;
}

int main() {
    LogLevel level = LogLevel::WARNING;
    std::cout << to_string(level) << std::endl; // 输出: WARNING
    
    level = from_string("ERROR");
    std::cout << to_string(level) << std::endl; // 输出: ERROR
    
    return 0;
}
```

对比示例

```c
// 传统 enum - 有问题
enum Color { RED, GREEN, BLUE };
enum TrafficLight { RED, YELLOW, GREEN }; // 错误！RED, GREEN 重定义

// enum class - 安全
enum class Color { RED, GREEN, BLUE };
enum class TrafficLight { RED, YELLOW, GREEN }; // 正确，作用域不同

void example() {
    // 传统 enum
    Color c = RED;           // 直接使用
    int value = c;           // 隐式转换
    
    // enum class
    Color c2 = Color::RED;   // 需要作用域
    // int value2 = c2;      // 错误！需要显式转换
    int value2 = static_cast<int>(c2); // 正确
}
```













































# C++2.0新特性

- 头文件
  - 到了C++11以后的版本中，头文件不比再加上.h后缀，对于C标准库，需要再前面加上c前缀
    - 例如：stdio.h->cstdio

## 数量不定的模板参数

- ...变成关键字
  - 表示一组、一包 package
  - 逻辑上代表其余的意思
    - 例如，我们可以在main函数中表明其余，这样在命令行中就可以接收多个·参数，然后我们对每个参数值进行判断，从而达到一个命令行的实现
  - 在不断的分解中，它将4个参数不断分解为1+3->1+2->1+1->1+0这种情况（当然，分解的方法是用户告诉编译器怎么分解的，这里是将其第一个参数取出，包中剩余的参数放入到下一个包中去）
    - 即一包里面也存在0的情况
  - sizeof...(args)，代表这一包东西里面有多少个，注意args并不是关键字，这里是形参名，即包名

![image-20251113122510267](./C++.assets/image-20251113122510267.png)

> 注意，省略号的位置。再模板定义里面，省略号再typename关键字后妈，而在函数中，省略号再参数名之前。

- bitset<二进制长度>(进制数)
  - 实现别的进制转换为二进制的一个模板

- 注意：

```c++
template<typename T, typename... Types>
void print(const T & firstArg, const Types&... args)

template<typename... Types>
void print(const Types&... args)
```

从以前的想法上这两种模板不能共存，因为这两个都适用于

`print("123", "456","sss")`这样的调用

当在新版本下，这样的参数模板可以共存

<img src="./C++.assets/image-20251113133752675.png" alt="image-20251113133752675" style="zoom: 200%;" />

即当调用的形式满足更加细化的特化版本函数时，会调用特化版本，否则就会调用细化版本。

Tuple 即打包 为一个对象  这个包里面可以有任意个类型

![image-20251113135723212](./C++.assets/image-20251113135723212.png)



## 小改动

1.两个模板在包含的时候，可以输入连续的两个尖括号，在以前需要用空格隔开。

![image-20251114194010271](./C++.assets/image-20251114194010271.png)

2.nullptr

用nullptr强调传入一个空地址，而不是数字零

![image-20251114195835027](./C++.assets/image-20251114195835027.png)

3.auto

auto必须用于赋值的时候，避免编译器无法推导该变量的类型

使用auto用于type很长，或者是接收的表达式很复杂，例如使用了lambda表达式



## Initiallizer_list

### 初始化一致性——{}

c++11以前初始化的多样性写法

![image-20251115151517368](./C++.assets/image-20251115151517368.png)

c++11以后所有的初始化都可以直接用大括号括起来

![image-20251115151452876](./C++.assets/image-20251115151452876.png)

![image-20251115153305384](./C++.assets/image-20251115153305384.png)

对于特定类型的初值能够进行匹配

![image-20251115160342734](./C++.assets/image-20251115160342734.png)

当使用大括号进行转换的时候，对于宽精度变为窄精度，使用{}，逻辑上不可以实现（实际上要具体看编译器公司是否支持），因为initializer_list主要处理可变的数据个数，对于数据的格式转变没有硬性要求。

即initializer_list可以支持参数不定，但是不支持参数的类型不同，因为其逻辑上是一个数组。

![image-20251115160537173](./C++.assets/image-20251115160537173.png)

initializer_list，可以被用户调用，使用者调用时也必须时initializer_list的形式

![image-20251115161516748](./C++.assets/image-20251115161516748.png)



![image-20251115162828963](./C++.assets/image-20251115162828963.png)

注意，我们无法显式的调用一个initializer_list，即无法使用`initializer_list<int> myinitalizer_list(...)`。只能通过{}，的方式去使用该类，让编译器去调用。

![image-20251115163516770](./C++.assets/image-20251115163516770.png)

这里也可以看出，initializer_list，它只是引用了一个外部准备好的array，而并没有在内部包含一个array。在实际实现时，并不会复制整个array的值，而是用指针指向同一块内存区域（浅拷贝）。

由于initializer_list的出现，C++的容器与算法部分出现了很大的改动，大部分的容器与算法由于以前只能接受部分容器，现在可以通过递归，实现出多个数据之间的运算。

![image-20251115170300701](./C++.assets/image-20251115170300701.png)

## explicit(明确的)

即告诉编译器，该函数要被明确的被调用，而不是由编译器自动进行转换。注意，只有非明确调用的只需要输入一个参数的构造函数才可以由编译器进行隐式的转换。

explicit对于构造函数仅有一个参数（调用时只需要输入一个参数）的时候:

![image-20251116140853798](./C++.assets/image-20251116140853798.png)

explicit对于构造函数有多个参数的时候:

![image-20251116141953923](./C++.assets/image-20251116141953923.png)



## for循环的特殊写法

![image-20251116143215227](./C++.assets/image-20251116143215227.png)

注意，关联式容器不可以直接通过该方法修改容器内容（例如set、hash）

![image-20251116144647780](./C++.assets/image-20251116144647780.png)



![image-20251116145411601](./C++.assets/image-20251116145411601.png)



## =default,=delete

通常，对于一个类，如果用户不编写构造函数，那么编译器就会自动给你添加一个default ctor 。如果是一个普通的类，那么default ctor是一个空函数

如果该类继承于父类的话，那么就需要调用到父类的构造函数，此时调用的部分就在这个default ctor中。

如果编写了ctor，但是在创建类的时候还是想要调用编译器给的那个什么都没有的默认构造函数的话，那么就用=default。

![image-20251117122146171](./C++.assets/image-20251117122146171.png)

构造函数、拷贝构造函数、拷贝复制、析构函数、搬移构造可以称作为big-five，这几种函数都是编译器会给出默认的版本，当用户自定义的时候采用用户的自定义函数。

![image-20251117124238950](./C++.assets/image-20251117124238950.png)



![image-20251117124748936](./C++.assets/image-20251117124748936.png)



![image-20251117125035933](./C++.assets/image-20251117125035933.png)



> 哪些类需要用户自行定义big-three？哪些使用编译器提供的默认函数即可？
>
> 类中如果携带了指针成员，那么该类就需要写出big-three函数。反之，使用默认即可。即是否需要使用指针来牵扯到一块内存。例如complex与string，由于complex已经确认了只有实部与虚部，而string需要更加广泛的接受多个字符，因此complex用编译器提供的即可而string需要写出big-three函数

Complex：

由于需要用到赋值操作，因此可以自行定义构造函数

![image-20251117130059865](./C++.assets/image-20251117130059865.png)





![image-20251117131116536](./C++.assets/image-20251117131116536.png)



![image-20251117131459120](./C++.assets/image-20251117131459120.png)



## constexpr

强制编译时求值

- 适用于数组大小、模版参数等需要常量表达式的场景

与const的区别

- `const`变量可在运行时初始化（如`const int x = getValue();`）。

- `const`仅保证变量不可修改，不强制编译时计算。



# Linux基础

## Linux文件

home 普通用户的家目录

内建命令

bash 解析命令 自带的命令

外部命令

外部软件 给的命令

## Linux命令格式

命令 选项（可以不要） 参数（可以没有）

help + 内建命令

外键命令 + --help

## 帮助文档查看方法

man + 命令/指令

man 页数 命令/指令

第一页是命令

第二页是系统调用

第三页是标准库

例如printf 就是 man 3 printf

## 编写C语言程序

gcc编译器工具链

(1) 生成gcc -E 预处理C文件 将导入的头文件 加载到C文件中

-o 决定预处理后的文件名称

严格关注大小写

（2）生成汇编文件

-S 预处理文件.i -o 指定文件名

（3）生成目标

gcc -c 文件名.s -o 指定目标文件名.o

file + 文件名 可以查看文件类型

LSB 小端模式 

ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped 

（4）生成可执行文件

gcc 文件名.o -o hello

执行文件 ./文件名

**一部到位 gcc 文件名.c -o 文件名无后缀 等价于上面四步**

### 静态连接和动态连接

将头文件加入到文件中 属于从源文件到可执行文件的第一步

未来不想给客户看到源代码 只提供这个功能 那么就需要将源代码写成库

链接分为两种：**静态链接**、**动态链接**。

**1）静态链接**

静态链接：由链接器在链接时将库的内容加入到可执行程序中。

优点：

- 对运行环境的依赖性较小，具有较好的兼容性

缺点：

- **生成的程序比较大**，需要更多的系统资源，**在装入内存时会消耗更多的时间**
- **库函数有了更新，必须重新编译应用程序**

**2）动态链接**

动态链接：连接器在链接时仅仅建立与所需库函数的之间的链接关系，在程序运行时才将所需资源调入可执行程序。 

优点：

- 在需要的时候才会调入对应的资源函数
- 简化程序的升级；有着较小的程序体积
- 实现进程之间的资源共享（避免重复拷贝） 

缺点：

- 依赖动态库，不能独立运行
- **动态库依赖版本问题严重**

**3）静态、动态编译对比**

前面我们编写的应用程序大量用到了标准库函数，系统**默认采用动态链接**的方式进行编译程序，若想采用静态编译，加入-static参数。

gcc默认给的可执行文件为a.out

![image-20221012161336927](C++.assets/image-20221012161336927.png)

发现静态链接生成的大小要比动态链接 生成的可执行文件大了不止十倍

cp 要复制的文件 生成的文件名

### 一件替换文件中的所有代码

**：%s/被替换的代码/替换后的代码/g（表示全部替换）**

### 静态库和动态库的制作

" " 优先搜索本地的文件

<> 优先搜索标准库

库分为三个部分

前缀 中间 后缀

libxxx.a

（1）将C源文件生成对应的.o文件

（2）用打包工具ar将.o文件打包为.a文件 libxxx.a

> ar -rcs 库函数文件 要打包的.o文件

.o文件时在生成可执行文件前的一部生成的文件 可以通过 gcc -c  源文件.c -o 目标文件名.o生成

**在使用ar工具是时候需要添加参数：rcs**

- r更新
- c创建
- s建立索引

**2）静态库使用**

静态库制作完成之后，需要将.a文件和头文件一起发布给用户。

假设测试文件为main.c，静态库文件为libtest.a头文件为head.h

编译命令：

> deng@itcast:~/test/4static_test$ gcc  test.c -L./ -I./ -ltest -o test

参数说明：

- -L：表示要连接的库所在目录
- -I./:  I(大写i) 表示指定头文件的目录为当前目录
- -l(小写L)：指定链接时需要的库，去掉前缀和后缀

生成的.a文件 是二进制文件 打开看不到信息

### 动态库制作

静态库需要加载入程序中 过于大 

动态库通过链接 在程序需要的时候再加载

动态库制作教程

### 生成目标文件 

**gcc -fPIC -c add.c** 

-c生成目标代码

**-fPIC创建与地址无关的编译程序**

**gcc -shared** 所有的 .o 文件 -o 输出文件名 lib文件名.so 注意后缀为.so

指定库的文件的名字

### 运行方式

把头文件和库发布 以保护核心源代码

gcc test.c -L. -I. -l(小写的L)库名称

但是链接器无法找到库的位置

有两种解决方式

（1）将动态库文件放入到标准库文件中去 /lib（绝对路径）

sudo cp 库文件 /lib

(2)系统动态载入器

在linux系统中编译.c文件生成的文件为.ELF文件格式

对于elf格式的可执行程序，是由ld-linux.so*来完成的，它先后搜索elf文件的 DT_RPATH段 — 环境变量LD_LIBRARY_PATH — /etc/ld.so.cache文件列表 — **/lib/, /usr/lib**目录找到库文件后将其载入内存。

我们可以通过对库文件的一个路径的添加来让程序找到我们的动态库文件

- 临时设置LD_LIBRARY_PATH：

export LD_LIBRARY_PATH=(不能添加空格)$LD_LIBRARY_PATH:库路径

这样只能在程序进行测试的时候使用，因为这里是一个临时变量 从而导致在别的终端 别的服务器上不能运行这个程序

所以我们需要进行永久设置

可以通过对bash配置文件进行追加 绝对路径

vim ~/ .bashrc

source ~/ .bashrc

## 网络常用命令

ping:测试主机与网络 主机与主机之间的网络连通性

ifconfig：配置和显示Linux系统网卡的网络参数

常用于查看IP地址 后使用ssh协议进行连接

route:显示并设置linux中静态路由表 /ru:t/

netstat:net /steat/

> netstat:用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。

kernel(内核)



## centos7防火墙基本使用

systemctl 是centos7特有的 status 在很多命令中都表示查看状态 firewall为防火墙 最后添加的d代表守护进程

> \#防火墙是开启 systemctl start firewalld
>
> #关闭防火墙 systemctl stop firewalld 
>
> #查看防火墙状态 systemctl status firewalld 
>
> #重启防火墙 systemctl restart firewalld





# MakeFile

[Linux系统编程-第03天（makefile-文件IO）.pdf](pdf_note\Linux系统编程-第03天（makefile-文件IO）.pdf) 

## 语法

一条规则：

目标可以为一个动作名称  如clean

目标(生成的东西):依赖文件列表(原材料) 目标必须存在

<Tab> 命令列表  //tab四个空格

```makefile
all:test1
	echo "hello all"

目标:all
依赖test1

可以看做一个if语句 当依赖存在时 执行命令

all:test1
	echo "hello all"

test1:
	echo "hello test1"

依赖可以看做一个函数
```

make -f name.mk

## 命令格式

make命令格式：

make [ -f file ][ options ][ targets ]

make 默认在工作目录中寻找名为GNUmakefile、makefile、Makefile的文件作为输入文件

- -f file ]：

  - make默认在工作目录中寻找名为GNUmakefile、makefile、Makefile的文件作为makefile输入文件

  - -f 可以指定以上名字以外的文件作为makefile输入文件

- option

  - -f 指定除以上名字以外的文件名

  - -v Makefile 版本

  - -w 在处理makefile之前和之后显示工作路径
  - -C dir 读取makefile之前改变工作路径至dir目录
  - -n 只打印要执行的命令但不执行
  - -s 执行但不显示执行的命令

- targets：

  - 指定执行某一target

  - make target -f name.mk 

  - 若使用make命令时没有指定目标，则make工具默认会实现makefile文件内的第一个目标，然后退出


  - 指定了make工具要实现的目标，目标可以是一个或多个（多个目标间用空格隔开）。

    MakeFile中的伪目标

    当与MakeFile同文件夹中有相同的指令

注意Makefile写好后，执行make命令时，默认只执行第一个目标，也就是说默认只识别了第一个目标。

如果第一个目标不存在，就去找依赖，执行对应的命令，第一个目标存在之后，后面的不会去管。

依赖也不存在 则去执行对应的命令

## 变量

类似模版  宏定义

自定义变量

- 定义变量方法
  - 变量名=变量值
- 引用变量
  - $(变量名)
  - ${变量名}
- 变量命名规则
  - makefile的变量名可用数字开头
  - 在开头定义
  - 作用域为全局

系统变量

大写

可对其直接复制

CC = gcc

CPPFLAGS

CFFLAGS

LFLAGS

变量不能单独使用  只能在命令中使用

$@ 表示目标

$^ 表示所有的依赖

$<表示第一个依赖

作用域为当前规则

即 作用域那个规则 $@表示那个目标 以此类推

```makefile
OBJS = add.o sub.o mul.o div.o test.o
TARGET = test

$(TARGET):$(OBJS)
	gcc $^ -o $@

add.o:add.c
	gcc -c $< -o $@ #生成add.o 当目标文件存在后 该条规则结束
	
sub.o:sub.c
	gcc -c $< -o $@#生成sub.o

mul.o:mul.c
	gcc -c $< -o $@#生成mul.o
	
div.o:div.c
	gcc -c $< -o $@#生成div.o
	
test.o:test.c
	gcc -c $< -o $@#生成test.o

clean:
	rm -rf $(OBJS) $(TARGET)
	
```

模式匹配  

所有的.o都依赖对应的.c将所有的.c生成对应的.o

```makefile
OBJS = add.o sub.o mul.o div.o test.o
TARGET = test

%.o:%.c
	gcc -c $< -o $@
	
clean:
	rm -rf $(OBJS) $(TARGET)
```

**MakeFile文件的后缀为.mk文件**

目标：依赖文件列表

<Tab> 命令列表

依赖文件可以没有 命令列表可以没有 但目标一定要有

#### Makefile基本规则三要素：

1）目标：

- 通常是要产生的文件名称，目标可以是可执行文件或其它obj文件，也可是一个动作的名称

- 指令可以作为目标 例如clean指令

- ```makefile
  clean:
      rm -rf $(OBJS) $(TARGET)
  
  ```

删除当前目录下全部的OBJS变量中的文件 和TARGET中的文件

2）依赖文件：

- 用来输入从而产生目标的文件
- 一个目标通常有几个依赖文件（可以没有）

3）命令：

- make执行的动作，一个规则可以含几个命令（可以没有）
- 有多个命令时，每个命令占一行
- **每一句命令开头应该使用TAB按钮**

#### Make命令

make是一个命令工具，它解释Makefile 中的指令（应该说是规则）。 

# 操作系统

用户通过系统调用间接与内核交流的媒介

用户程序调用C语言函数库函数，库函数调用系统调用接口与内核

linux系统

- 深度

- 靶场metasploitable

- kali
- ubuntubudgie

## SHELL

**shell (壳) 程序  让用户与系统进行交互  终端 cmd** 

窗口运行shell程序  但窗口不是shell

Linux下的shell称为bash

操作通过shell解释给内核

脚本语言可以通过shell解释给内核，不需要编译处理

python通过python解释器 解释给内核处理

**只有内核才能直接操作硬件**

### 环境变量

每个进程都有自己的一个环境变量表，表中的每个条目都是 

键=值

形式的环境变量

ubuntu终端下  env   envirment

**查看bash的全局环境变量**

echo $name 环境变量值

**键 通常使用大写**

如 SHELL=/bin/bash 等号左右无空格

USER=orsrrc

添加环境变量

FOOD=liurouduan

FOOD=dafengshou

环境变量可以被改变  



- 环境变量类别
  - 全局环境变量
    - 当前shell和其子进程(子进程可继承)可见
  - 局部(自定义)环境变量
    - 只有当前shell可见

局部环境变量 export 局部环境变量变为全局环境变量

**删除unset 环境变量名**



### PATH

记录bash对命令的检索路径   每个路径以:分隔

命令本质为操作系统自带的二进制的可执行程序  执行命令即为到对应的命令去找程序

执行命令时会去检索所有的PATH



whereis 检索文件路径



添加路径到PATH中

./ 当前目录

bash根据PATH环境变量找到可执行程序 若直接输入程序命令  则bash会去PATH中寻找

若提供命令对应的路径 则bash通过用户指定的路径 找到程序

```bash	
PATH=$PATH:.  在当前环境变量下 再添加一个.(相对)  当前路径
环境变量名前$表示环境变量值 
```

由于每个进程都有一个自己的PATH变量

故而当当前窗口关闭时 bash进程被杀死 那么再次打开时 为创建一个新的bash 上次配置的PATH消失

永久更改

在home目录下有.bashrc的脚本文件  每次bash进程启动前都会执行该脚本文件的内容

```bash
source~./.bashrc 

或者修改 .bashrc  在最后加上PATH=$PATH:.

此时 永久生效
```

$PS1 是一个局部变量  用于修改终端命令的值

可将变量放入bashrc中对终端进行修改

## 环境变量表

索引表  每一个进程都有一个环境变量表

具体实现为数组容器中放入指针  指针指向环境变量的内存首地址

指向环境变量表的首地址的指针environ

```C++
#include<stdio.h>

int main(void){
	extern char** environ; //调用其他文件中的全局变量  外部变量声明 关键字
  for(char **pp = environ; *pp; pp++)
  {
    //二级指针  存放指针地址的指针
    printf(“%s\n”, *pp);
  }
  return 0;
}
```

```C++
int main(int argc, char* argv[], char* envp[]//环境变量表)
{
  extern char** environ;
  printf("environ = %p", environ);
  printf("environ = %p\n", envp);
  for (char** pp = envp; *pp; pp++)
  {
    printf("%s\n", *pp);
  }
  return 0;
}
         //通过main函数的第三个参数也可以获取到环境变量表的首地址
```



## 文件

**系统调用就是内核提供给用户的接口**

**用户态 各种操作都有限制**

**内核态 毫无限制的访问各种资源**

**用户态 通过软件中断 到内核态**

错误处理函数

系统调用函数 fopen 打开文件 （“文件路径”， “文件权限（只读r 可读可写wr）”）返回一个指针 指针中存放的地址指向一个结构体

**结构体中有文件描述符**

**文件读写指针位置**

**缓冲区** 内存地址 一个printf函数的缓冲区大概有8k左右

errno 是记录系统的最后一次错误代码,返回值为一个整形的值，可以通过strerror函数将返回值转换为一个字符串

```c
#include <stdio.h>  //fopen
#include <errno.h>  //errno
#include <string.h> //strerror(errno)

int main()
{
    FILE *fp = fopen("xxxx", "r");
    if (NULL == fp)
    {
        printf("%d\n", errno);  //打印错误码
        printf("%s\n", strerror(errno)); //把errno的数字转换成相应的文字
        perror("fopen err");    //打印错误原因的字符串
    }

    return 0;
}
```

perror打印出错代码 作用为strerror(errno) errno == error number

> /usr/include/asm-generic/errno-base.h
>
> /usr/include/asm-generic/errno.h

可以在这两个文件中寻找到错误函数对应的一个绑定

![image-20221015165846814](C++.assets/image-20221015165846814.png)

因为计算机内核部分是不能被访问的 不然就会出现段错误

内存访问非法就是段错误

![image-20221015180242498](C++.assets/image-20221015180242498.png)

**其中局部全局变量和 全局变量存放的位置相同都在data区**

**0-3G用户可操作区 程序大部分是存放在这些部分**

**内核区 一般的程序无法写入 内核区留下部分接口 给用户区 就是系统调用**

**堆空间中存放动态内存分配** 例如 指针指向数组动态分配 **此时就需要析构函数进行删除 防止内存泄漏**

共享库：存储映射区

栈：存放局部的变量 通过递归可以查看栈的大小 或者通过命令 

只读数据段 ：存储字符段

在磁盘中找到可执行文件后

### 内存的虚拟空间

在进程里平时所说的指针变量，保存的就是虚拟地址。

###  文件描述符

![image-20221017143033101](C++.assets/image-20221017143033101.png)

当0 ，1，2 被占用时 打开的文件描述符一定为3 即为空闲的最小的一个文件描述符

对文件描述符进行操作 即为对文件进行操作

#### open 函数

当对open函数的帮助文档进行查看时发现

```c
   int open(const char *pathname, int flags);
   int open(const char *pathname, int flags, mode_t mode);
```

发现在文档中有两个相同的函数定义

但是在C语言中 没有函数的重载 想要实现函数重载怎么办呢 就可以通过  "..."表示参数不确定 从而实现函数重定义

说明open在函数定义时用的就是...表明参数不确定

implicit declaation of function '函数名' 代表 ( 函数的隐式解密 )函数没有头文件

#### 阻塞和非阻塞

程序会不断的读取管道中的内容 但如果管道中没有内容 那么就会造成资源的浪费 所以需要设置阻塞

读常规文件是不会阻塞的，不管读多少字节，read一定会在有限的时间内返回。

从终端设备或网络读则不一定，如果从终端输入的数据没有换行符，调用read读终端设备就会阻塞，如果网络上没有接收到数据包，调用read从网络读就会阻塞，至于会阻塞多长时间也是不确定的，如果一直没有数据到达就一直阻塞在那里。

同样，写常规文件是不会阻塞的，而向终端设备或网络写则不一定。

【注意】阻塞与非阻塞是对于文件而言的，而不是指read、write等的属性。

**例如c/cpp中的输入函数 程序执行到当前命令的时候 会阻塞而等待输入 不往下面的程序执行**

#### stat函数

符号链接（在windows中类似于快捷方式）

Modify: 2022-10-06 05:10:51.922623022 -0400
Change: 2022-10-06 05:10:51.922623022 -0400

Modify 是修改文件属性时间

Change是修改文件内容时间

有的操作修改文件内容会默认修改文件属性

在末行命令模式下

:r !head -10 文件名 代表将文件的前十行 复制到当前文件中

！代表不是强制 只是执行命令四要素

1、编号

2、名称

3、事件

4、默认处理动作

**man 7 signal**查看帮助文档获取

## 信号

### 状态

#### 1、产生

终端上按“Ctrl+c”组合键通常产生中断信号 SIGINT

终端上按“Ctrl+\”键通常产生中断信号 SIGQUIT

终端上按“Ctrl+z”键通常产生中断信号 SIGSTOP 

#### 2、未决状态：没有被处理

#### 3、递达状态：信号被处理了

### 阻塞信号集（信号屏蔽字）

将某些信号加入信号集 对其进行屏蔽 再收到该信号 该信号将再解除屏蔽后处理

类似电话黑名单

### 未决信号集

信号产生后由于某些原因(主要是阻塞)不能抵达在屏蔽解除前，信号一直处于未决状态。

### 信号产生函数

kill函数 给指定进程发送指定信号 不一定是关闭进程或杀死进程

```
int kill(pid_t pid, int sig);
```

```
pid : 取值有 4 种情况 :
        pid > 0:  将信号传送给进程 ID 为pid的进程。
        pid = 0 :  将信号传送给当前进程所在进程组中的所有进程。
        pid = -1 : 将信号传送给系统内所有的进程。
        pid < -1 : 将信号传给指定进程组的所有进程。这个进程组号等于 pid 的绝对值。
    sig : 信号的编号，这里可以填数字编号，也可以填信号的宏定义，可以通过命令 kill - l("l" 为字母)进行相应查看。不推荐直接使用数字，应使用宏名，因为不同操作系统信号编号可能不同，但名称一致。
```

root用户可以发送信号给任意用户 但普通用户只能给自己产生的进程发送信号 不能给root用户或者其他普通用户发送信号 子进程可以对父进程发送信号

raise函数

给自己发送指定信号 类似于 kill(getpid(), sig)

abort函数

```
给自己发送异常终止信号 (6) SIGABRT，并产生core文件，等价于kill(getpid(), SIGABRT);
```

### 不可重入函数

全局函数

不同任务调用这个函数时可能修改其他任务调用这个函数的数据

例如标准输入输出函数 因为标准的输入输出函数 与系统调用的输入输入函数不同的是

调用标准输入输出函数是带有缓冲区的 这样就导致别的函数可以通过缓冲区进行修改或打印不该打印和被获取的值。



满足下列条件的函数多数是不可重入（不安全）的：

- 函数体内使用了静态的数据结构；
- 函数体内调用了malloc() 或者 free() 函数(谨慎使用堆)；
- 函数体内调用了标准 I/O 函数。

## 可重入函数

- 在写函数时候尽量使用局部变量（例如寄存器、栈中的变量）；
- 对于要使用的全局变量要加以保护（如采取关中断、信号量等互斥方法），这样构成的函数就一定是一个可重入的函数。







## 进程

### 进程和程序

程序是一个静态的可执行文件 放在硬盘中

当程序被CPU调用 但没有执行完成时 就是进程 关闭程序 就是杀死进程

进程是cpu分配的最小单位 

单核CPU 不能同时执行多个进程 可以轮流执行不同的进程

可以添加多个处理器 就可以同时执行多个指令

或者一个处理器集成多个核 就可以同时执行多个指令

一个处理器集成多个核的性能 比一台PC集成多个处理器的性能好 但成本也就更高。

一个48核处理器的成本比 4个八核处理器的成本高 对于商用PC 更多的是集成多个处理器。

### 并行与并发

**并行(parallel)：**指在同一时刻，有多条指令在多个处理器上同时执行。

![image-20221019113157826](C++.assets/image-20221019113157826.png)

**并发(concurrency)：**指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。 

![image-20221019112804417](C++.assets/image-20221019112804417.png)

每一个时刻 只有一条指令在执行

### MMU（Memory Management Unit）内存管理单元

负责将虚拟地址映射为物理地址。在物理地址上内存可能不是连续的 所以需要虚拟地址来映射 让用户觉得是连续的。

![image-20221019115809332](C++.assets/image-20221019115809332.png)

### 进程控制块（PCB）

Linux内核的进程控制块是task_struct结构体。

掌握以下部分

- 进程id。系统中每个进程有唯一的id，在C语言中用pid_t类型表示，其实就是一个非负整数。
- 进程的状态，有就绪、运行、挂起、停止等状态。
- 进程切换时需要保存和恢复的一些CPU寄存器。
- 描述虚拟地址空间的信息。
- 描述控制终端的信息。
- 当前工作目录（Current Working Directory）。
- umask掩码。
- 文件描述符表，包含很多指向file结构体的指针。
- 和信号相关的信息。
- 用户id和组id。
- 会话（Session）和进程组。
- 进程可以使用的资源上限（Resource Limit)。

### 进程状态

进程不是死循环 会有一个运行到消亡的状态

**三态模型**

运行态：程序被CPU调度运行

就绪态：程序编译加载工作均已经完成 只等CPU调度

阻塞态：程序请求某种权限、资源时 由于被占用 只能排队等待

五态模型

![image-20221019143144815](C++.assets/image-20221019143144815.png)

1.运行态:

程序正在被CPU调度运行

2.就绪态

当程序一切的权限 资源申请调度都完成了 但还未被CPU调度 程序就处于就绪态 

3、僵尸态

程序运行结束后 没有被删除和销毁 

4、等待态

不可中断等待 程序在运行时 由于不接收外来指令和命令 从而不能命令中断和杀死

可中断等待 进程在运行时 可以接收到外来信号 能够被中断和停止

5、停止态

程序运行时 接受到了停止信号 从等待态变为停止态 再次需要调用时 就会变为就绪态 然后可以重新调度运行

#### 查看本机进程

**ps -aux**

USER（用户） PID（进程号） %CPU（占用CPU大小） %MEM（内存） VSZ（虚拟地址）RSS（空闲地址） TTY   （终端 端口号） STAT（状态） START（开始时间）   TIME（运行时间） COMMAND（命令）

通过ps -a 选项查看需要杀死的程序的进程号(PID)

kill + 进程号 进行杀死

#### top 动态显示进程

**bc调用linux程序计算器** quit命令退出

## 进程的创建

#### 创建子进程

在unix和linux系统下可以通过fork（）函数创建子进程

子进程会复制父进程的指令

但创建子进程后 无法确认谁先执行 因为计算机系统的调度算法不同 对于多核计算机来说 讨论这个几乎没有意义

当然子进程与父进程的进程号 进程ID肯定是不相同的

但由于无法确定父进程和子进程谁先执行时

![image-20221019162242521](C++.assets/image-20221019162242521.png)

就会出现这种情况

由于bash是所有进程的父进程

所以若调度算法 使得父进程优先执行 那么 就会出现![image-20221019162417894](C++.assets/image-20221019162417894.png)这种情况 输出父进程的hello world后 执行了一次bash 然后再执行子进程的hello world

如果是子进程有限 那么就会出现并行的情况

![image-20221019162604006](C++.assets/image-20221019162604006.png)

#### 父进程和子进程

它从父进程处继承了整个进程的地址空间：包括进程上下文（进程执行活动全过程的静态描述）、进程堆栈、打开的文件描述符、信号控制设定、进程优先级、进程组号等。

子进程所独有的只有它的进程号，计时器等（只有小量信息）。因此，使用 fork() 函数的代价是很大的。

![image-20221019165711510](C++.assets/image-20221019165711510.png)

所以对与父进程的数据区 子进程不是直接复制 而是**读是共享 写时复制**

**读时共享**

如果子进程不对数据进行修改 而只是读取数据的话 并不会将数据区的数据复制到子进程中 而是父进程和子进程共享这一数据

**写时复制**

若子进程要对数据进行修改 在读取到修改数据命令时 就会复制这个数据到进程本身 然后进行修改

进程退出函数

**exit()** 进程退出函数 

**return 0** 函数退出

_exit()

_EXIT()

这两个函数是直接退出 没有任何的善后处理 包括刷新缓冲区等等操作

#### KILL 杀死进程指令

KILL +【信号】+ 进程号 

KILL -l 查看所有的信号

**kill 杀死一个进程 默认信号为15**  

  1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
  2) SIGABRT	 7) SIGBUS	 8) SIGFPE	 **9) SIGKILL**	10) SIGUSR1
 3) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	**15) SIGTERM**
 4) SIGSTKFLT	17) SIGCHLD	1**8) SIGCONT**	**19) SIGSTOP**	20) SIGTSTP
 5) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
 6) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
 7) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
 8) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
 9) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
 10) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
 11) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
 12) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
 13) SIGRTMAX-1	64) SIGRTMAX	

**18和19信号分别为 进程继续 和进程挂起**

#### 孤儿进程

与僵尸进程相对的进程是 孤儿进程 **即为父进程已结束 而子进程依旧在运行** 当孤儿进程越来越多 而没有程序进行善后 删除操作时 就会占用系统资源 所以 当出现孤儿进程时 **程序会自动将孤儿进程的父进程设置为 init进程** init进程就是系统内核进程 这样当孤儿进程生命周期结束时 这些孤儿进程就会被回收

sudo init 6/0 可以进行关机开机   

#### 僵尸进程

父进程未结束 子进程结束 但未被父进程清理 这样就会生成僵尸进程 僵尸进程与孤儿进程不同 没有自动回收的机制 所以僵尸进程是很危险的 可以通过ps命令查看系统中的僵尸进程 STAT（状态）一栏中的 Z(zombie)状态就是僵尸进程

#### 进程替换

在程序执行进程时 通过替换进程的数据 堆栈区等 不执行现有进程 而执行新进程

![image-20221021114922640](C++.assets/image-20221021114922640.png)

进程调用一种 exec 函数时，该进程完全由新程序替换，而新程序则从其 main 函数开始执行。因为调用 exec  并不创建新进程，所以前后的进程 ID （当然还有父进程号、进程组号、当前工作目录……）并未改变。exec  只是用另一个新程序替换了当前进程的正文、数据、堆和栈段（进程替换）。



## 终端

用户与主机交互的地方就叫做终端

## 进程组

一个或多个进程的集合就叫进程组

进程组的ID可能和进程ID相同

## 会话

多个进程组的集合就叫会话

前台会话只能有一个 并且与终端相关联

当终端断开时 前台进程组中的进程就会被删除

后台进程组可以有多个

并且 脱离终端运行 即关闭终端后 也依然会运行

### 创建会话注意事项

1、**调用进程不能是进程组组长**，该进程变成新会话首进程(session header) 例如父进程创建了一个子进程 父进程就是进程组组长

2、**该调用进程是组长进程，则出错返回**

3、**该进程成为一个新进程组的组长进程**

4、需有root权限(ubuntu不需要)

5、新会话丢弃原有的控制终端，该会话没有控制终端

6、 建立新会话时，先调用fork, 父进程终止，子进程调用setsid

**组长进程的ID和组长ID相同**

## 守护进程

它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。一般采用以d结尾的名字。

只有当系统关闭 主机关机时 停止运行

每一个从此终端开始运行的进程都会依附于这个终端，这个终端就称为这些进程的控制终端，当控制终端被关闭时，相应的进程都会自动关闭。

守护进程是个特殊的孤儿进程，这种进程脱离终端，避免进程被任何终端所产生的信息所打断，**其在执行过程中的信息也不在任何终端上显示。**表明关闭了所有的文件描述符 并且所有的标准输出 标准错误输出函数都将信息直接输入到日志文件中。

孤儿进程一般会被一号进程收养 也就是内核

如果在GUI界面主机中使用的话 会被图形界面收养

tail -f动态的查看文件追加内容

​         







## 线程

一个进程可以同时拥有多个线程  至少有一个主线程

每个线程有自己独立的变量 故每个线程有自己独立的一个栈

进程是内存分配的最小单元 线程调度的最小单位

可以看做一个单核的CPU 交替运行任务

![image-20240117150707229](C++.assets/image-20240117150707229.png)

时间片+FCFS

线程共享进程中的变量



### 线程号	

通过pthread_self返回pthread_t 类型的当前线程的进程号， 函数总是成功 类似于umask掩码函数

### 创建线程(POSIX线程库)

POSIX标准 提供统一标准函数

创建线程后 执行线程的是内核 需要告诉内核执行哪一个函数 传入参数

#include<pthread.h>

p == POSIX

pthread_create(**pthread_t* tid**, **pthread_attr_t const* attr**, **void\*(\*start_routine)(void*)** //函数指针 线程从哪一函数开始执行(线程过程函数), **void\* arg**//线程的运行函数的参数) //当线程需要多个参数时  可以用结构体(多个不同类型的变量) 或者数组传入地址(多个相同类型的变量)





thread:线程标识符地址
attr:线程属性结构体地址，通常设置为NULL
start_routine:线程函数的入口地址
arg:传给线程函数的参数

返回值:成功返回0 失败返回错误码

### 主线程和子线程

main()是主线程的线程过程函数 main函数结束 意味着主线程结束

主线程结束  进程结束  则子线程全部结束

```C++
#include <stdio.h>
#include <pthread.h>
#include <string.h>
//线程过程函数
void* pthread_fun(void* arg)
{
  printf("%lu线程:我是子线程, 接受到%s\n", pthread_self(), (char* )arg);
  return NULL;
}


int main(void)
{
  printf("%lu线程：我是主线程，我要创建子线程\n", pthread_self() );
	//pthread_self() 返回线程ID %lu 表示无符号长整型
  pthread_t tid;//接收线程的ID
 	int error = pthread_create(&tid, NULL/*线程的属性*/, pthread_fun, "铁锅炖大鹅");
  if(error != 0)
  {
    fprintf(stderr, "pthread_created:%s\n",strerror(error));
    //strerror(错误码)返回错误信息
    return -1;
  }
  printf("%lu线程:我是主线程，我创建了%lu子线程\n", pthread_self(), tid);
  sleep(1);
  return 0;
}
```



**perror 输出最近一次出错的信息**

![image-20240117171455511](C++.assets/image-20240117171455511.png)

![image-20240117171519011](C++.assets/image-20240117171519011.png)

Compile and link with -pthread

gcc .c -o target -l pthread



![image-20240117172056644](C++.assets/image-20240117172056644.png)

![image-20240117172223332](C++.assets/image-20240117172223332.png)

接收的参数的地址的内存在子线程运行期间 存储区有效



> 小练习
>
> 创建线程
>
> 该线程负责圆的面积计算 s = pai * r^2 <math>
>
> 半径由主线程创建时 传递给子线程
>
> 线程将面积回传给主线程

```C++
#include <stdio.h>
#inlcude <string.h>
#include <pthread.h>
##include <unstd.h>
#define pai 3.14
double s = 0; //定义全局变量
//内核调用线程过程函数area 而不是main函数 故而在返回时 main函数无法接收area函数的返回值
void* area(void* arg){
  double r = *(double*)arg;
  s = pai * r * r;
  return NULL;
}

int main(void){
  pthread_t tid;
  double r = 10;
  pthread_create(&tid, NULL, area, &r);
  return 0;
} //也可以通过修改arg的值将圆的面积返回main
//在子线程中定义静态变量 表示函数结束时 该变量的内存不回收 但是仍然无法在main函数中访问子线程的变量
```

> 计算两个数字的平均数

```C++
#include <stdio.h>
#inlcude <string.h>
#include <pthread.h>
##include <unstd.h>

void* aver(void* arg){
  double *d = (double*)arg;
  //*d == d[0]
  //*(d+1) == d[1]
  d[2] = (d[0] + d[1]) / 2;
  return NULL;
}

int main(void*)
{
  pthread_t tid;
  double data[3] = {3, 7};
  pthread_create(&tid, NULL, area, &data);
  printf("%d", data[2]);
  return 0;
}

//指针的类型决定了 该指针访问内存时读取几个字节 所有类型的指针的容量都是固定的  唯一的区别是访问内存时所读取的字节数
```

### 汇合线程

![image-20240121182247024](C++.assets/image-20240121182247024.png)

![image-20240121182524875](C++.assets/image-20240121182524875.png)

将线程返回的指针地址 放入到 retval 二级指针中  接受一级指针的地址 用二级指针

```c++
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#define PI 3.14

//线程过程函数  计算圆的面积

void* area(void* arg)
{
  double r = *(double*) arg;
  double static s = PI * r * r; //由于s创建的时间比PI 
  //与 r 的时间早 故而不可以用一个未定义的变量给一个已经定义的变量
  double static s;
  s = PI * r * r;
  
  //动态分配 内存回收时间由free来决定
  double* s = (double*)malloc(sizeof(double));
  *s = PI * r * r; //线程间不共享各自的资源 线程间未知对方的缓冲区的地址  因此不能操作对方的缓冲区
  return &s;
}


int main(void)
{
  pthread_t tid;
  double r = 10;
  pthread_create(&tid, NULL, area, &r);
  double* res;//输出线程过程函数的返回值
  pthread_join(tid, (void**) &res); //二级指针是一个形参告诉用户这里需要放入指针的地址
  
  pthread_t tid2;
  double r2 = 20;
  pthread_create(&tid2, NULL, area, &r2);
  double* res2;
  pthread_join(tid, (void**)&res2);
  
  //若有两条线程  而s为static 则两个线程共享一个缓冲区s
  
  return 0;
  
}
```



### 分离线程

主线程不用管理子线程的回收 由内核回收























## 死锁

死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。 ![04-死锁](C++.assets/04-死锁.png)

如图 当线程1和线程2

一个先访问资源1 一个先访问资源2

在并行的状态下

两个线程都会顺利执行 但当走到第二步时 由于加锁的原因资源1 资源2都被占用 导致两个线程都阻塞 无法进行下一步 这样就形成死锁

解决办法之一就是同时访问资源1同时访问资源2 这样当其中一个访问时 资源被占用 另一个线程在第一步就会被阻塞





















































































# 网络

## 网卡

网络适配器 蓝牙也是一种网卡 只是协议不一样 

### MAC地址 6个字节 

标识网卡的ID

每块网卡的ID 是不一样的 但虚拟机可能相同 因为虚拟机需要你的网卡去上网

理论上ID全球唯一

MAC地址是不会变的

### IP地址 

标识主机的ID 这个ID是虚拟的 这个ID会改变

IPV4 32位 4个字节 

IPV6 128位 16个字节

公网使用的IPV6

局域网使用的是IPV4

一个IP将其分为子网ID和主机ID

子网ID和主机ID需要和子网掩码一起来看

192.168.11.23

255.255.255.0

被连续的1覆盖的位就是子网ID

被连续的0覆盖的位就是主机ID

**这里192.168.11.0 就是 网段地址**

23就是主机ID 

**广播地址 192.168.11.255** 如果是这个IP 发送

除去这两个地址不能做主机IP

主机ID分配的范围为：192.168.11.1——>192.168.11.254 2^8 - 2 = 254 最后一位 8个1 

192.168.1.23/24(255.255.255.0) 代表24个1

也就是说 前三位为子网ID 最后一位为主机ID 23

那么它的网段地址为 192.168.1.0

它的广播地址为192.168.1.255

**192.168.1.2/16(255.255.0.0)**

它的网段地址为192.168.0.0

广播地址为 192.168.255.255

能分配的IP地址范围为

两个八位 2^16 - 2个

ens33网卡名称

inet ip地址 netmask 子网掩码 broadcast 广播地址 mtu代表一次最大接受的数据包的长度



ping 命令 用来检测两台主机的网络联通性

A -->ping -->B

B-->ping-->A

lo本地回环地址 127.0.0.1 -- 127.255.254 都属于回环IP

ping lo网卡 测试本地网络

windows设置IP 在网络的适配器中点击IPV4服务 进行设置

Linux下设置IP可以在ifconfig 命令后加 网卡名称 ens33

### 桥接和net

### 桥接 

让物理机虚拟出一块网卡 这块网卡有自己专属的IP地址 即可以通过路由查询 但这块网卡在局域网内只能访问物理机的网卡 而无法访问 局域网内的别的主机 。

可以通过物理机来访问外部网络

### 端口

端口是用于区分不同的应用程序的缓冲区 当别的主机发来的消息包时 由于每个应用程序的端口不同 使得主机能够将这个消息包发给对应的应用程序。

## OSI七层模型

第零层

在物理层的下面的一层 这一层指代的是物理设备 例如网卡之类的

物理层

双绞线接口类型，光纤的传输速率等等

数据链路层 mac负责收发数据

网络层 ip给两台主句提供路径选择 区分是否是递送给本机的

传输层：port区分数据递送到哪一个应用程序

会话层 建立会话

表示层 解码

应用层



开发中使用

![image-20221115091456387](C++.assets/image-20221115091456387.png)

链路层 设备到设备 MAC 到 MAC 

网络层 ICMP IP IGMP

传输层 TCP UDP

应用层 FTP Telnet TFP NFS ...... 很多

![image-20221114162646488](C++.assets/image-20221114162646488.png)

收数据 从下往上搜

发数据 从上往下搜



## 协议

规定了数据传输的方式和格式

应用层协议:

FTP: 文件传输协议

HTTP: 超文本传输协议

NFS: 网络文件系统

传输层协议:

TCP:  传输控制协议

UDP:用户数据报协议

网络层:

IP:英特网互联协议

ICMP: 英特网控制报文协议 ping

IGMP: 英特网组管理协议

链路层协议:

ARP: 地址解析协议 通过ip找mac地址

RARP: 反向地址解析协议 通过mac找ip

### arp协议

#### arp地址解析协议

ip地址转换为 mac地址

对于一个路由器而言

一个LAN口可以连接 局域网内的一台主机

局域网内的主机通过UDP协议进行通信

这样就需要别的 主机的 IP 和 MAC 地址

如果没有怎么办呢

例如 A想跟B通信 但没有B的MAC地址 那么就需要在局域网内发送一个ARP广播 

向路由器发送一个ARP请求 然后每个通过LAN口连接的主机都会 收到一个ARP请求

然后 只有B会应答

其他主机就直接将这个请求的包丢掉

当局域网内的主机想要访问外网主机的时候

这个时候肯定就不能获取外网主机的MAC地址 

这个时候 局域网内的主机会将数据包发送给路由器的网关 

网关负责向外发送 此时数据包的MAC地址就填入网关的MAC地址

网关的主机IP地址一般为1和254 最大或者最小

##### ARP请求报文组包

ARP请求报文 由14个字节的mac头 和 28字节的arp报文组成

其中mac头尾 以太网首部 这是每个包都必须要的 其内容为

目的主机的MAC地址

源MAC地址

目的主机的MAC地址 

当需要在局域网内发送广播时 单纯添加某一主机的MAC地址时 只有该主机会收到ARP请求包

故而 MAC地址的包应该为ff:ff:ff:ff:ff::ff 当MAC地址为该地址时 局域网内的主机会无条件接收

![image-20240115113803794](C++.assets/image-20240115113803794.png)

##### 传输层 TCP与UDP

一般来说 在局域网中使用最多的是UDP 当需要夸网关传输时 使用的是TCP 

因为UDP传输效率高 但不重传 不安全

TCP比较安全 

UDP:用户数据报协议

报头中有八个字节 类似包裹的纸壳

客户机与服务器的连接

- 建立虚电路
- 交换数据
- 终止连接

特点

- 可靠连接
  - 每次发送一个数据包都需要确认
  - 超时重传
- 有序性
- 全双工

三次握手|三路握手

- 发一个SYN(同步)数据包 告知对方自己将在连接中发送数据的初始序列号
- TCP包的头部有部分保留的字节  即某一位为1 可看SYN置一
- 服务器的TCP协议栈向客户机发送一个数据包

![image-20221115094931044](C++.assets/image-20221115094931044.png)

UDP报头有八个字节

A给B发送数据 源端口号 为A

B为目的端口号

数据为另算的东西 类似包裹的内容

UDP16位校验和

两个字节两个字节的叠加 会生成一串数字 发出数据包时会将这串数字一起发送

当接收方接收数据的时候 会将接收到的数据也两个字节两个字节的叠加 生成一个校验和 两个校验和一比较就可以看出数据包是否丢失



### TCP报头

![image-20221116104137700](C++.assets/image-20221116104137700.png)

#### 头部长度即为首部长度

它为四位 代表数据最大值为15 但TCP报头总共有20个字节 那么20/4 = 5 说明 头部长度为5 

连续发四个这样的包 然后接收方通过序列号 确认号等组合起来

窗口尺寸代表 接收方的缓冲区大小 当超过这个大小时 发送的包就回丢失

紧急指针一般为0 当为1时就回立即发送数据

### IP报头

![image-20221116104603038](C++.assets/image-20221116104603038.png)

版本为IPV4 或者是 IPV6

当一个数据包很大的时候 就需要将一个包分开多次传输 片偏移就是用于组包的

TTL 每经过一个路由器就减一 默认值为64或128

![image-20221116110256471](C++.assets/image-20221116110256471.png)

说明经过了13个路由器才到达百度的路由器

作用：

当对一个IP地址发送数据包的时候 如果接收方的网络断开

那么这个数据包就回在路由器与路由器之间乱走 如果不删除这个数据包 当后面的数据来的时候就回形成阻塞

所以给数据包设置TTL 每当数据包遇到路由器时 TTL就减一 当TTL的值为零时 路由器就不再转发 直接将数据包给丢掉

协议类型：

通过协议类型可以知道向上发送数据包的时候时发给TCP还是UDP 每一个协议类型都有对应的数字 例如UDP协议为17

UDP 默认8个字节 TCP IP 都默认为20个字节



## 网络同通信协议过程

组包的过程由上往下 即 应用层 → 传输层 → 网络层 → 链路层

![image-20221116154335613](C++.assets/image-20221116154335613.png)

从应用层开始 主机A的IP为 192.168.1.1/24 通过应用传输消息给主机B IP 192.168.1.2/24 

其中 ![image-20221116154657956](C++.assets/image-20221116154657956.png)为物理地址

从物理层 发出消息 并且通过应用的协议 到达传输层

传输层填入8个字节的UDP 然后填入源地址端口src和目的地址dst端口

然后传入网络层

加入IP报头

输入源IP地址和目的IP地址

到达链路层加入mac头

14个字节

![image-20221116161129687](C++.assets/image-20221116161129687.png)

然后往主机B的链路层传输  判断主机B的MAC地址 与MAC头中的MAC地址是否一致 如果一致那么就传输到链路层中然后通过协议往上解包

![image-20221116154115518](C++.assets/image-20221116154115518.png)

然后被解开包 就被丢弃 类似 收到包裹以后 将包裹的外包装一次撕开

每一层被解开的包如果找不到对应的ip那么就回直接将包给丢掉





## 网络设计模式

B/S browser/server 浏览器和服务器

C/S cilent/server 客户端与服务器

### C/S cilent/server 客户端与服务器

**优点**：所有的计算都在本地进行计算 性能比较好

如果多人同时在线游玩一个游戏 都让服务器进行计算的话 服务器的计算量会非常的大

所以 在本地计算完成后 发送给服务器 可以有效减轻服务器的负荷

**缺点**：

客户端容易篡改数据 外挂 并且开发周期较长

### B/S browser/server 浏览器和服务器

优点：客户端安全 开发周期短

缺点：性能低 不能用于大型游戏

在网络协议上 由于是浏览器 所以协议必须是http/https 而客户端可以有自己的协议





### 网络序与主机序

- 网络序
  - 大端序

- 主机序
  - 小端序



## API

### 地址族

```c++
/*
 * Address families.
 */
#define AF_UNSPEC       0               /* unspecified */
#define AF_UNIX         1               /* local to host (pipes) */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define AF_LOCAL        AF_UNIX         /* backward compatibility */
#endif  /* (!_POSIX_C_SOURCE || _DARWIN_C_SOURCE) */
#define AF_INET         2               /* internetwork: UDP, TCP, etc. */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define AF_IMPLINK      3               /* arpanet imp addresses */
#define AF_PUP          4               /* pup protocols: e.g. BSP */
#define AF_CHAOS        5               /* mit CHAOS protocols */
#define AF_NS           6               /* XEROX NS protocols */
#define AF_ISO          7               /* ISO protocols */
#define AF_OSI          AF_ISO
#define AF_ECMA         8               /* European computer manufacturers */
#define AF_DATAKIT      9               /* datakit protocols */
#define AF_CCITT        10              /* CCITT protocols, X.25 etc */
#define AF_SNA          11              /* IBM SNA */
#define AF_DECnet       12              /* DECnet */
#define AF_DLI          13              /* DEC Direct data link interface */
#define AF_LAT          14              /* LAT */
#define AF_HYLINK       15              /* NSC Hyperchannel */
#define AF_APPLETALK    16              /* Apple Talk */
#define AF_ROUTE        17              /* Internal Routing Protocol */
#define AF_LINK         18              /* Link layer interface */
#define pseudo_AF_XTP   19              /* eXpress Transfer Protocol (no AF) */
#define AF_COIP         20              /* connection-oriented IP, aka ST II */
#define AF_CNT          21              /* Computer Network Technology */
#define pseudo_AF_RTIP  22              /* Help Identify RTIP packets */
#define AF_IPX          23              /* Novell Internet Protocol */
#define AF_SIP          24              /* Simple Internet Protocol */
#define pseudo_AF_PIP   25              /* Help Identify PIP packets */
#define AF_NDRV         27              /* Network Driver 'raw' access */
#define AF_ISDN         28              /* Integrated Services Digital Network */
#define AF_E164         AF_ISDN         /* CCITT E.164 recommendation */
#define pseudo_AF_KEY   29              /* Internal key-management function */
#endif  /* (!_POSIX_C_SOURCE || _DARWIN_C_SOURCE) */
#define AF_INET6        30              /* IPv6 */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define AF_NATM         31              /* native ATM access */
#define AF_SYSTEM       32              /* Kernel event messages */
#define AF_NETBIOS      33              /* NetBIOS */
#define AF_PPP          34              /* PPP communication protocol */
#define pseudo_AF_HDRCMPLT 35           /* Used by BPF to not rewrite headers
	                                 *  in interface output routine */
#define AF_RESERVED_36  36              /* Reserved for internal usage */
#define AF_IEEE80211    37              /* IEEE 802.11 protocol */
#define AF_UTUN         38
#define AF_VSOCK        40              /* VM Sockets */
#define AF_MAX          41
#endif  /* (!_POSIX_C_SOURCE || _DARWIN_C_SOURCE) */
```



### Socket()

#include <sys/socket.h>

int socket(int domain, int type, int protocol);

功能：创建套接字

参数 domain:通信域 协议族

- PF_LOCAL/PF_UNIX 本地套接字  进程间通信
- PF_INET IPV4通信
- PF_INET6 IPV6通信
- PF_PACKET 底层包的网络通信

**所有的PF\*  都可以更换为AF * 因为在源代码中 所有的仅对该变量的声明的名字重新定义没有做别的改变**

type 套接字类型

- SOCK_STREAM TCP协议  流式套接字
- SOCK_DGREAM UDP协议  数据报套接字
- SOCK_RAW 创建在工作在传输层以下的套接字

**对于使用TCP 和 UDP的套接字   那么在参数三protocol 传入0即可**

当传入套接字在传输层以下  **那么需要用户指定每一层的协议**

#### 工作在物理层的套接字的应用

由于物理层需要接收所有数据包 则可以用于抓包

PF_PACKET 底层包的网络通信

SOCK_RAW 创建在工作在传输层以下的套接字



### Inet_addr

**#include** **<arpa/inet.h>**

   in_addr_t

   **inet_addr**(const char *cp);

   int

   **inet_aton**(const char *cp, struct in_addr *pin);

   in_addr_t

   **inet_lnaof**(struct in_addr in);

   struct in_addr

   **inet_makeaddr**(in_addr_t net, in_addr_t lna);

   in_addr_t

   **inet_netof**(struct in_addr in);

   in_addr_t

   **inet_network**(const char *cp);

   char *

   **inet_ntoa**(struct in_addr in);

   const char *

   **inet_ntop**(int af, const void * restrict src, char * restrict dst, socklen_t size);

   int

   **inet_pton**(int af, const char * restrict src, void * restrict dst);





### Bind()

**#include** **<sys/socket.h>**

 int bind(int socket, const struct sockaddr *address, socklen_t address_len);

**DESCRIPTION**

 **bind**() assigns a name to an unnamed socket. When a socket is created with

 socket(2) it exists in a name space (address family) but has no name assigned.

 **bind**() requests that address be assigned to the socket.

**NOTES**

 Binding a name in the UNIX domain creates a socket in the file system that must be

 **deleted by the caller when it is no longer needed (using unlink(2)).**

 The rules used in name binding vary between communication domains. Consult the manual entries in section 4 for detailed information.

**RETURN** **VALUES**

 **Upon successful completion, a value of 0 is returned.** Otherwise, **a value of -1 is**  **returned** and the global integer variable errno is set to indicate the error.

#### Sockaddr 与 sockaddr_in

两者都是在绑定地址时使用

Sockaddr 在socket.h中定义

sockaddr_in 在netinet/in.h 中

由于sockaddr 在绑定时将所有的数据都放在sa_data中 不方便使用

Sockaddr_in 作为一种改进版 将各个部分的数据分离

```c++
struct sockaddr_in {
	__uint8_t       sin_len;
	sa_family_t     sin_family;
	in_port_t       sin_port;
	struct  in_addr sin_addr;
	char            sin_zero[8];
};


struct sockaddr {
	__uint8_t       sa_len;         /* total length */
	sa_family_t     sa_family;      /* [XSI] address family */
#if __has_ptrcheck
	char            sa_data[__counted_by(sa_len - 2)];
#else
	char            sa_data[14];    /* [XSI] addr value (actually smaller or larger) */
#endif
};
```

>  struct sockaddr_in ser;
>   ser.sin_family = AF_INET;
>   ser.sin_port = htons(999);
>   //绑定地址
>   if(bind(sockfd, (struct sockaddr*)&ser, sizeof(ser)) == -1)
>   {
>     perror("bind");
>     return -1;
>   }







> //准备地址结构
> 	struct sockaddr_in ser;
> 	ser.sin_family = AF_INET;
> 	ser.sin_port = htons(8888); 
> 	ser.sin_addr.s_addr = inet_addr("服务器IP地址  点分十进制");
> //请求连接
> 	if( connect(sockfd, (struct sockaddr*)&ser, sizeof(ser)) == -1)
>   {
>     perror("connect");
>     return -1;
>   }

### 主机字节序与网络字节序的转换

**小端  主机字节序  大端  网络字节序**

网络协议栈以大端方式(方便人类阅读)处理数据  但数据在计算机中存储方式为小端方式

故而  在传递数据时  需要将数据有小端方式改为大端方式后传输

       #include <arpa/inet.h>
    
       uint32_t htonl(uint32_t hostlong);
    
       uint16_t htons(uint16_t hostshort);
    
       uint32_t ntohl(uint32_t netlong);
    
       uint16_t ntohs(uint16_t netshort);

>The htonl() function converts the unsigned integer hostlong from ***host byte order** to **network byte** order.
>
>The  htons() function converts the unsigned short integer hostshort from **host byte order** to **network byte**
> **order**.
>
>The ntohl() function converts the unsigned integer netlong from **network byte order** to **host byte order**.
>
>The ntohs() function converts the unsigned short integer netshort from **network byte order** to  **host  byte**
> **order**.
>
>integer 整型   host byte order 主机字节序 network byte order 网络字节序

**ip地址的转换**

int_addr_t inet_addr(char const* ip);

**点分十进制字符串地址 转换为网络字节序形式整数地址**

int inet_aton(char const* ip, struct in_addr* nip);

char* inet_ntoa(struct in_addr nip);



### recvfrom()

**#include** **<sys/socket.h>**

   ssize_t

   **recv**(int socket, void *buffer, size_t length, int flags);

   ssize_t

   **recvfrom**(int socket, 

​		      void *restrict buffer, 

​		      size_t length,

​		      int flags,

​                      struct sockaddr *restrict address,

​		      socklen_t *restrict address_len);

ssize_t  **recvmsg**(int socket,

​			      struct msghdr *message, 

​			      int flags);



> The **recvfrom**() and **recvmsg**() system calls are used to receive messages from a  socket, and may be used to receive data on a socket whether or not it is connection-oriented.
>
>    If address is not a null pointer and the socket is not connection-oriented, the
>
>    source address of the message is filled in. The address_len argument is a value-
>
>    result argument, initialized to the size of the buffer associated with address, and
>
>    modified on return to indicate the actual size of the address stored there.
>
> > All three routines return the length of the message on successful completion. If a
> >
> >    message is too long to fit in the supplied buffer, excess bytes may be discarded
> >
> >    depending on the type of socket the message is received from (see socket(2)).
> >
> > 
> >
> >    **If no messages are available at the socket, the receive call waits for a message to**
> >
> >    **arrive, unless the socket is nonblocking** (see fcntl(2)) in which case the value -1  is returned and the external variable errno set to EAGAIN. The receive calls
> >
> >    normally return any data available, up to the requested amount, rather than waiting
> >
> >    for receipt of the full amount requested; this behavior is affected by the socket-
> >
> >    level options SO_RCVLOWAT and SO_RCVTIMEO described in getsockopt(2).
> >
> >    The select(2) system call may be used to determine when more data arrive.
> >
> >    **If no messages are available to be received and the peer has performed an orderly**
> >
> >    **shutdown, the value 0 is returned.**
> >
> >    The flags argument to a **recv**() function is formed by or'ing one or more of the
> >
> >    values:
> >
> > ​      MSG_OOB    process out-of-band data
> >
> > ​      MSG_PEEK    peek at incoming message
> >
> > ​      MSG_WAITALL  wait for full request or error

  返回值：

These calls return the number of bytes received, or -1 if an error occurred.

   For TCP sockets, the return value 0 means the peer has closed its half side of the connection.

```c++
char buf[128] = {};
    struct sockaddr_in cli;
    socklen_t len = sizeof(cli);
    ssize_t size = recvfrom(sockfd, buf, sizeof(buf) - 1, 0, (struct sockaddr*)&cli, &len);
    if(size == -1)
    {
      perror("recvfrom");
      return -1;
    }
```



### Send()

**#include** **<sys/socket.h>**

   ssize_t

   **send**(int socket, const void *buffer, size_t length, int flags);

   ssize_t

   **sendmsg**(int socket, const struct msghdr *message, int flags);

   ssize_t

   **sendto**(int socket, const void *buffer, size_t length, int flags,

​     const struct sockaddr *dest_addr, socklen_t dest_len); 



**send**(), **sendto**(), and **sendmsg**() are used to transmit a message to another socket.

   **send() may be used only when the socket is in a connected state, while sendto() and**

   **sendmsg() may be used at any time.**

send用于已建立链接的发送数据 即面向连接的传输  



> The address of the target is given by dest_addr with dest_len specifying its size.
>
> The length of the message is given by length. If the message is too long to pass
>
>    atomically through the underlying protocol, the error EMSGSIZE is returned, and the
>
>    message is not transmitted.

   返回值

Upon successful completion, the number of bytes which were sent is returned.

   Otherwise, -1 is returned and the global variable errno is set to indicate the

   error.





### wrap

#### wrap.h

```c++
#ifndef __WRAP_H_
#define __WRAP_H_
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <strings.h>

void perr_exit(const char *s);
int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr);
int Bind(int fd, const struct sockaddr *sa, socklen_t salen);
int Connect(int fd, const struct sockaddr *sa, socklen_t salen);
int Listen(int fd, int backlog);
int Socket(int family, int type, int protocol);
ssize_t Read(int fd, void *ptr, size_t nbytes);
ssize_t Write(int fd, const void *ptr, size_t nbytes);
int Close(int fd);
ssize_t Readn(int fd, void *vptr, size_t n);
ssize_t Writen(int fd, const void *vptr, size_t n);
ssize_t my_read(int fd, char *ptr);
ssize_t Readline(int fd, void *vptr, size_t maxlen);
int tcp4bind(short port,const char *IP);
#endif
```

#### wrap.c

```c++
// #include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <strings.h>

void perr_exit(const char *s)
{
	perror(s);
	exit(-1);
}

int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr)
{
	int n;

again:
	if ((n = accept(fd, sa, salenptr)) < 0) {
		if ((errno == ECONNABORTED) || (errno == EINTR))//如果是被信号中断和软件层次中断,不能退出
			goto again;
		else
			perr_exit("accept error");
	}
	return n;
}

int Bind(int fd, const struct sockaddr *sa, socklen_t salen)
{
    int n;

	if ((n = bind(fd, sa, salen)) < 0)
		perr_exit("bind error");

    return n;
}

int Connect(int fd, const struct sockaddr *sa, socklen_t salen)
{
    int n;

	if ((n = connect(fd, sa, salen)) < 0)
		perr_exit("connect error");

    return n;
}

int Listen(int fd, int backlog)
{
    int n;

	if ((n = listen(fd, backlog)) < 0)
		perr_exit("listen error");

    return n;
}

int Socket(int family, int type, int protocol)
{
	int n;

	if ((n = socket(family, type, protocol)) < 0)
		perr_exit("socket error");

	return n;
}

ssize_t Read(int fd, void *ptr, size_t nbytes)
{
	ssize_t n;

again:
	if ( (n = read(fd, ptr, nbytes)) == -1) {
		if (errno == EINTR)//如果是被信号中断,不应该退出
			goto again;
		else
			return -1;
	}
	return n;
}

ssize_t Write(int fd, const void *ptr, size_t nbytes)
{
	ssize_t n;

again:
	if ( (n = write(fd, ptr, nbytes)) == -1) {
		if (errno == EINTR)
			goto again;
		else
			return -1;
	}
	return n;
}

int Close(int fd)
{
    int n;
	if ((n = close(fd)) == -1)
		perr_exit("close error");

    return n;
}

/*参三: 应该读取固定的字节数数据*/
ssize_t Readn(int fd, void *vptr, size_t n)
{
	size_t  nleft;              //usigned int 剩余未读取的字节数
	ssize_t nread;              //int 实际读到的字节数
	char   *ptr;

	ptr = vptr;
	nleft = n;

	while (nleft > 0) {
		if ((nread = read(fd, ptr, nleft)) < 0) {
			if (errno == EINTR)
				nread = 0;
			else
				return -1;
		} else if (nread == 0)
			break;

		nleft -= nread;
		ptr += nread;
	}
	return n - nleft;
}
/*:固定的字节数数据*/
ssize_t Writen(int fd, const void *vptr, size_t n)
{
	size_t nleft;
	ssize_t nwritten;
	const char *ptr;

	ptr = vptr;
	nleft = n;
	while (nleft > 0) {
		if ( (nwritten = write(fd, ptr, nleft)) <= 0) {
			if (nwritten < 0 && errno == EINTR)
				nwritten = 0;
			else
				return -1;
		}

		nleft -= nwritten;
		ptr += nwritten;
	}
	return n;
}

static ssize_t my_read(int fd, char *ptr)
{
	static int read_cnt;
	static char *read_ptr;
	static char read_buf[100];

	if (read_cnt <= 0) {
again:
		if ( (read_cnt = read(fd, read_buf, sizeof(read_buf))) < 0) {
			if (errno == EINTR)
				goto again;
			return -1;
		} else if (read_cnt == 0)
			return 0;
		read_ptr = read_buf;
	}
	read_cnt--;
	*ptr = *read_ptr++;

	return 1;
}

ssize_t Readline(int fd, void *vptr, size_t maxlen)
{
	ssize_t n, rc;
	char    c, *ptr;

	ptr = vptr;
	for (n = 1; n < maxlen; n++) {
		if ( (rc = my_read(fd, &c)) == 1) {
			*ptr++ = c;
			if (c  == '\n')
				break;
		} else if (rc == 0) {
			*ptr = 0;
			return n - 1;
		} else
			return -1;
	}
	*ptr  = 0;

	return n;
}

int tcp4bind(short port,const char *IP)
{
    struct sockaddr_in serv_addr;
    int lfd = Socket(AF_INET,SOCK_STREAM,0);
    bzero(&serv_addr,sizeof(serv_addr));
    if(IP == NULL){
        //如果这样使用 0.0.0.0,任意ip将可以连接
        serv_addr.sin_addr.s_addr = INADDR_ANY;
    }else{
        if(inet_pton(AF_INET,IP,&serv_addr.sin_addr.s_addr) <= 0){
            perror(IP);//转换失败
            exit(1);
        }
    }
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port   = htons(port);
    int opt = 1;
	setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    Bind(lfd,(struct sockaddr *)&serv_addr,sizeof(serv_addr));
    return lfd;
}
```



### Select()

> select()  and  pselect() allow a program to monitor multiple file descriptors, waiting until one or more
>        of the file descriptors become "ready" for some class of I/O operation (e.g., input possible).   A  file
>        descriptor  is  considered  ready  if  it  is possible to perform the corresponding I/O operation (e.g.,
>        read(2)) without blocking.

/* According to POSIX.1-2001 */
       #include <sys/select.h>

/* According to earlier standards */
   #include <sys/time.h>
   #include <sys/types.h>
   #include <unistd.h>

 int select(int nfds, fd_set *readfds, fd_set *writefds,
              fd_set *exceptfds, struct timeval *timeout);

- 参数

  - nfds 最大文件描述符 + 1 

  - fd_set 文件描述符集合

  - readfds 需要监听的读的文件描述符存放集合

  - writefds 需要监听的写的文件描述符存放集合

  - exceptfds 需要监听的异常的文件描述符存放集合

    > - Three independent sets of file descriptors are watched.  Those listed in readfds will be watched to  see
    >   if characters become available for reading (more precisely, to see if a read will not block; in particular, a file descriptor is also ready on end-of-file), those in writefds will be  watched  to  see  if  a
    >   write will not block, and those in exceptfds will be watched for exceptions.  On exit, the sets are modified in place to indicate which file descriptors actually changed  status.   Each  of  the  three  file
    >   descriptor  sets may be specified as NULL if no file descriptors are to be watched for the corresponding
    >   class of events.

  - timeout 一次监听的时间 NULL 持续监听

    > **The timeout argument specifies the minimum interval that  select()  should  block  waiting  for  a  file**
    > **descriptor to become ready.**  (This interval will be rounded up to the system clock granularity, and kernel scheduling delays mean that the blocking interval may overrun by a small amount.)  If both fields of the  timeval  structure  are zero, then select() returns immediately.  (This is useful for polling.)  If
    > timeout is NULL (no timeout), select() can block indefinitely.

    - The timeout
             The time structures involved are defined in <sys/time.h> and look like

                 struct timeval {
                     long    tv_sec;         /* seconds */
                     long    tv_usec;        /* microseconds */
                 };
              
             and
              
                 struct timespec {
                     long    tv_sec;         /* seconds */
                     long    tv_nsec;        /* nanoseconds */
                 };

- 返回值

  - On success, select() and pselect() **return the number of file descriptors contained in the three returned**
    **descriptor  sets  (that is, the total number of bits that are set in readfds, writefds, exceptfds) which**
    **may be zero if the timeout expires before anything interesting happens.** 
  - **On error, -1 is  returned**,  and
    errno  is  set  appropriately;  the  sets and timeout become undefined, so do not rely on their contents
    after an error.

一次监听过程中 返回容器内存在变化的描述符的数量 若整个过程中没有描述符发生改变则返回0



>监听多个文件描述符的属性变化(读，写，异常)  
>
>void FD_CLR(int fd, fd_set *set);
>int  FD_ISSET(int fd, fd_set *set);
>void FD_SET(int fd, fd_set *set);
>void FD_ZERO(fd_set *set);

#### 监听原理

![Image [2]](C++.assets/Image [2].png)

> 应用层调用系统调用 让内核监听文件描述符 应用层每次传入一个位图表 表示监听哪几个文件描述符 每个描述符在监听的时间内变化则内核修改其在位图上的对应位置 监听结束后返回给应用 若监听lfd时 该描述符变化 则表示产生了新的描述符 则在位图表中将对应位置 置为1表示已分配 然后传递给内核 内核在下次监听时会根据位图表监听对应的文件描述符



```c++
#include <stdio.h>
#include <sys/select.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/time.h>
#include "wrap.h"
#include <sys/time.h>
#define PORT 8888

int main(int argc, char* argv[])
{
  //创建套接字 绑定
  int lfd = tcp4bind(PORT, NULL);
  //监听
  Listem(lfd, 128);
  int maxfd = lfd; //最大的文件描述符
  fd_set oldset, rset;
  FD_ZERO(&oldset);
  FD_ZERO(&rset);
  //将lfd添加到oldset集合中
  FD_SET(lfd, &oldset);

  //while
  while(1)
  {
    //select 监听
    rset = oldset; // 将oldset赋值给需要监听的集合rset
    int n = select(maxfd + 1, &rset, NULL, NULL, NULL);
    if(n < 0)
    {
      perror("");
      break;
    }
    else if(n == 0)
    {
      continue; //无变化
    }
    else
    {
      //lfd 变化
      if(FD_ISSET(ldf, &rset))
      {
        struct sockaddr_in cliaddr;
        socklen_t len = sizeof(cliaddr);
        char ip[16] = "";
        //提取新的连接
        int cfd = Accept(lfd, (struct sockaddr*)&cliaddr, &len);
        printf("new client ip = %s port = %d\n", 
               inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr,ip,16),
               ntohs(cliaddr.sin_port) );
        //将cfd添加至oldset集合中， 用以下一次监听
        FD_SET(cfd, oldset);
        //更新maxfd
        maxfd = cfd > maxfd ? cfd : maxfd;
        //如果只有lfd变化 continue
        if(--n == 0)
        {
          continue;
        }

        //cfd connetfd
        for(int i = lfd + 1; i <= maxfd; i++)
        {
          //如果i文件描述符在rset集合中
          if(FD_ISSET(i, &rset))
          {
            char buf[1500] = ""; //以太网的最大传输单元为1500B
            int ret = Read(i, buf, sizeof(buf));
            if(ret < 0) //出错将cfd关闭 从oldset中删除cfd
            {
              perror("");
              close(i);
              FD_CLR(i, &oldset);
              continue;
            }
            else if(ret == 0)
            {
              close(i);
              FD_CLR(i, &oldset);
              printf("客户端关闭\n");
					 }
            else
            {
             printf("%s\n", buf);
             write(i, buf, ret);
					}
          }//if
          
          
        }//for
        
      }
    }
  }
}
```







# TCP

## 状态转换图

![Image](C++.assets/Image.png)

主动方 客户端

被动方 服务器 

主动方主动发起TCP连接请求  被动方运行开始保持监听状态



## TCP编程模型

### 头文件

> <string.h>
>
> <ctype.h> 
>
> <sys/socket.h>
>
> <sys/types.h>
>
> <arpa/inet.h>

**toupper函数**

```
int toupper ( int c ); 
```

> toupper Convert lowercase letter to uppercase
>
> Convert lowercase letter to uppercase  小写转为大写
>
> Converts *c* to its uppercase equivalent if *c* is a lowercase letter and has an uppercase equivalent. If no such conversion is possible, the value returned is *c* unchanged.



**客户端和服务器端一对一通信**

```C++
//服务器端
//创建套接字
int sockfd = socket(AF_INET, SOCK_STREAM, 0); //返回套接字描述符  
//默认创建主动socket 无法接收
if (sockfd == -1) //规定sockfd为-1 表示创建失败
{
  perror("socket");
  return -1;
}
//准备地址结构 包括 地址族 ip地址  端口号
struct sockaddr_int ser;
ser.sin_family = AF_INET; //基于IPV4的协议
ser.sin_port = htons(8888); //字节序转换
ser.sin_addr.s_addr = inet_addr("192.168.222.128");//将字符串转换为整数
//若端口号被占用  则绑定地址时会报错
ser.sin_addr.s_addr = INADDR_ANY //接收任意IP地址数据
//绑定地址  将ip地址与socket绑定
bind(int sockfd, struct sockaddr const* addr, socklen_t addrlen);
//sockfd 套接字
//sockaddr 强转为泛化的结构体
//addrlen 地址的长度sizeof(ser)
//失败时 bind返回-1

//启动侦听
#include<sys/socket.h>
int listen(int sockfd, int backlog); /*指定接受窗口/接收缓冲区*/  ）;
if(listen(sockfd, 1024) == -1)//侦听套接字(该套接字负责接收所有的数据)
{
  perror("listen");
  return -1;
}
//等待连接 //进程阻塞 被动接收 接收连接请求 每一个accept为一个进程  多个进程发送连接请求时需要创建多个accept
int accept(int sockfd, struct sockaddr* addr, socklen_t* addrlen);
/*
	    sockfd 侦听套接字描述符
        addr 输出连接请求发送方的地址信息
        addrlen 输出连接请求发起方的地址信息字节数
        失败返回-1
        返回可用于后续通信套接字的fd
*/

struct sockaddr_in cli;
socklen_t len = sizeof(cli);
int conn = accept(sockfd， (struct sockaddr*)&cli, &len); //通信套接字
if(conn == -1)
{
  perror("accept");
  return -1;
}
printf("服务器接收到%s>%hu的客户端\n", inet_nota(cli.sin_addr), ntohs(cli.sin_port) );

//数据处理
bool isFlag = true;
while(isFlag)
{
  char buf[128] = {};
  ssize_t size =  read(conn,buf, sizeof(buf)-1/*减去串结束符\0*/)
    if(size == -1)
    {
      perror("read");
      return -1;
    }

  else if (size == 0)
  {
    printf("服务器:客户端退出\n")
      isFlag = false;
    break;
  }
  for(int i = 0; i < strlen(buf); i++)
  {
    toupper(buf[i]);
  }
  if (write(conn, buf, strlen(buf)) == -1)
  {
    perror("write");
    return -1;
  }
}    

//关闭套接字
close(conn);
close(sockfd);



//客户机端
//创建套接字
	int sockfd = socket(AF_INET, SOCK_STREAM, 0);
	if(socket == -1)//创建失败
  {
   	perror("socket");
    return -1;
  }
	
//准备地址结构
	struct sockaddr_in ser;
	ser.sin_family = AF_INET;
	ser.sin_port = htons(8888); 
	ser.sin_addr.s_addr = inet_addr("服务器IP地址  点分十进制");
//请求连接
	if( connect(sockfd, (struct sockaddr*)&ser, sizeof(ser)) == -1)
  {
    perror("connect");
    return -1;
  }
//数据处理
	for(;;)
  {
    //通过键盘获取小写消息
    char buf[128] = {};
    fgets(buf, sizeof(buf), stdin);
    //发送给服务器  服务器返回大写消息
    //输入数据"！"退出循环
    
    if(strcmp(buf, "!\n"))
    {
      break;
		}
    send()
    #include <sys/socket.h>
    sisize_t send(int sockfd, void const* buf, size_t count, int flags);
		//客户端接收服务器回传的数据
    //flag  MSG_DONTWAIT  以非阻塞方式接受数据
    //MSG_OOB 接收带外数据
    //MSG_DONTROUTE  不查路由表 直接在本地网络中寻找目的主机
    //返回值 成功返回实际发送的字节数 失败返回-1
  	send(aockfd, buf, strlen(buf), 0 == -1)
		if ( send(sockfd, buf, strlen(buf), 0) == -1)
    {
      perror("send");
      return -1;
    }
    ssize_t size = recv(sockfg, buf, sizeof(buf)-1, 0);
    if(size == -1)
    {
      perror("rece");
      return -1;
    }
   	
    // 关闭套接字
    printf("客户端:关闭套接字");
    close(sockfd)
  }
	
```



**多个客户端和一个服务器端通信**

> 服务器进程负责接收socket  
>
> 对数据的处理交给子进程进行  子进程负责发送处理完成的数据
>
> 子进程和父进程为继承关系 故而父进程接收和创建的socket 子进程均有一个副本
>
> 子进程仅使用通信套接字  父进程仅使用监听套接字
>
> 当客户端结束链接后
>
> 在信号中对子进程内存进行回收

```C++
  //服务器端
	//信号处理函数 负责回收
	void sigfun(int signum)
  {
		for(;;)
    {
      pid_t pid = waitpid(-1, NULL, WNOHANG); //非阻塞方式
      if(pid == -1)
      {
        if(errno == ECHILD)
        {
          printf("服务器:没有子进程");
          break;
        }
        else
        {
          perror("waitpid");
          return -1;
				}
      }
      else if (pid == 0)
      {
        printf("子进程在运行中");
        break;
      }
      else if (pid > 0)
      {
        printf("服务器: 回收了%d进程的僵尸", pid)
      }
    }
  }
	

  //对17号信号进行捕获处理
	if(signal(SIGCHLD, sigfun) == SIG_ERR)
  {
    perror("signal");
    return -1;
  }
  //创建套接字
  int sockfd = socket(AF_INET, SOCK_STREAM, 0); //返回套接字描述符  
  //默认创建主动socket 无法接受
  if (sockfd == -1) //规定sockfd为-1 表示创建失败
  {
    perror("socket");
    return -1;
  }
  //准备地址结构 包括 地址族 ip地址  端口号
  struct sockaddr_int ser;
  ser.sin_family = AF_INET; //基于IPV4的协议组
  ser.sin_port = htons(8888); //字节序转换
  ser.sin_addr.s_addr = inet_addr("192.168.222.128");//将字符串转换为整数
  //若端口号被占用  则绑定地址时会报错
  ser.sin_addr.s_addr = INADDR_ANY //接收任意IP地址数据
    //绑定地址  将ip地址与socket绑定
    bind(int sockfd, struct sockaddr const* addr, socklen_t addrlen);
  //sockfd 套接字
  //sockaddr 强转为泛化的结构体
  //addrlen 地址的长度sizeof(ser)
  //失败时 bind返回-1


  //启动侦听
  #include<sys/socket.h>
  int listen(int sockfd, int backlog); /*指定接受窗口/接收缓冲区*/  ）;
  if(listen(sockfd, 1024) == -1)//侦听套接字(该套接字负责接收所有的数据)
  {
    perror("listen");
    return -1;
  }
  //等待连接 //进程阻塞 被动接收 接收连接请求 每一个accept为一个进程  多个进程发送连接请求时需要创建多个accept
  int accept(int sockfd, struct sockaddr* addr, socklen_t* addrlen);
  /*
           sockfd 侦听套接字描述符
            addr 输出连接请求发送方的地址信息
            addrlen 输出连接请求发起方的地址信息字节数
            失败返回-1
            返回可用于后续通信套接字的fd
    */

  struct sockaddr_in cli;
  socklen_t len = sizeof(cli);
  int conn = accept(sockfd， (struct sockaddr*)&cli, &len); //通信套接字
//当客户端发送一个socket过来以后，服务器往该socket里面写数据，然后返回给客户端
  if(conn == -1)
  {
    perror("accept");
    return -1;
  }
  printf("服务器接收到%s>%hu的客户端\n", inet_nota(cli.sin_addr), ntohs(cli.sin_port) );

  pid_t pid = fork(); //创建子进程
  if (pid == -1)
  {
    perror("fork");
    return -1;
  }
  if (pid == 0) //子进程
  {
    close(sockfd);
    for(;;)
    {
      //数据处理
      bool isFlag = true;
      while(isFlag)
      {
        char buf[128] = {};
        ssize_t size =  read(conn,buf, sizeof(buf)-1/*减去串结束符\0*/)
          if(size == -1)
          {
            perror("read");
            return -1;
          }

        else if (size == 0)
        {
          printf("服务器:客户端退出\n")
            isFlag = false;
          break;
        }

        for(int i = 0; i < strlen(buf); i++)
        {
          toupper(buf[i]);
        }
        if (write(conn, buf, strlen(buf)) == -1)
        {
          perror("write");
          return -1;
        }
      }    
      //子进程关闭通信套接字
      close(conn);
      return 0;
    }
    //父进程关闭套接字 等待下一个连接请求
    close(conn);
   }
   close(sockfd);
   return 0;
  }
```

# UDP

## UDP编程模型

> UDP不面向连接 一个套接字与多个客户端通信 不需要先建立三次握手连接
>
> 不保证数据传输的可靠性和有序性
>
> 不提供流量控制
>
> 应用场景 对安全性要求不高  对传输速率要求高

#include<sys/socket.h>

ssize_t recvfrom(int sockfd, void* buf, size_t count, int flags, struct sockaddr* src_addr, socklen_t* addrlen)

- src_addr 输出源主机的地址信息  从哪里发来的请求 按照原有的地址发回去
- addrlen 输入和输出源主机的地址信息的字节数
- 成功返回实际接收的字节数  失败返回-1

```C++
//服务器端
#include<stdio.h>
#include<string.h>
#include<ctype.h>
#include<unistd.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<arpa/inet.h>

int main(void)
{
  //创建套接字
  printf("服务器创建套接字\n");
  int socketfd = socket(AF_INET, SOCK_DGRA, 0);
  if(socketfd == -1)
  {
    perror("socket");
    return -1;
  }
  
  //准备地址结构
  printf("服务器:组织地址结构\n");
  struct sockaddr_in ser;
  ser.sin_family = AF_INET;
  ser.sin_port = htons(999);
  //绑定地址
  if(bind(sockfd, (struct sockaddr*)&ser, sizeof(ser)) == -1)
  {
    perror("bind");
    return -1;
  }
  //数据处理
  for(;;)
  {
    char buf[128] = {};
    struct sockaddr_in cli;
    socklen_t len = sizeof(cli);
    ssize_t size = recvfrom(sockfd, buf, sizeof(buf) - 1, 0, (struct sockaddr*)&cli, &len);
    if(size == -1)
    {
      perror("recvfrom");
      return -1;
    }
    //'\0'的ASCII码为0 此时为假
    for(int i = 0; buf[i]; i++)
    {
      buf[i] = toupper(buf[i]);
    }
    //发送数据
    if(sendto(sockfd, buf, strlen(buf), 0, (struct sockaddr*)&cli, sizeof(cli)) == -1)
    {
      perror("sendto");
      return -1;
    }

  }
  //关闭套接字
  
  printf("服务器:关闭套接字\n");
  close(sockfd);
}

//客户端
//创建套接字
int main(void)
{
  printf("客户端:创建套接字\n");
  int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
  if(sockfd == -1)
  {
    perror("socket");
    return -1;
  }  
//准备地址结构
  struct sockaddr_in ser;
  ser.sin_family = AF_INET;
  ser.sin_port = htons(9999);
  ser.sin_addr.s_addr = inet_addr("192.168.222.128");
  
	//数据处理
  printf("客户端:业务处理\n");
  for(;;)
  {
    char buf[128] = {};
    fgets(buf, sizeof(buf), stdin);
    if(strcmp(buf, "!\n") == 0)
    {
      break;
    }
    if(sendto(sockfd, buf, strlen(buf), 0, (struct sockaddr*)&ser, sizeof(ser)) == -1)
    {
      perror("sendto");
      return -1;
    }
    //接收数据
    if(recv(sockfd, buf, sizeof(buf)-1, 0) == -1)
    {
      perror("recv");
      return -1;
    }
    printf("-->%s", buf);
  }
	
	
//关闭套接字
  printf("客户端:关闭套接字");
  close(sockfd);
  return 0;
}

```



# DNS域名解析

域名到IP地址的转换过程  由DNS服务器完成

> #include<netdb.h>
>
> struct hostent* gethostbyname(char const* host_name);
>
> 通过域名 获取主机信息
>
> - host_name 主机域名
> - 返回值 成功返回主机信息的结构体指针  失败返回NULL
> - char *h_name //主机官方名
> - char **h_aliases 主机别名表  官方名只有一个  主机别名表即 别名有多个
> - int h_length; //地址长度
> - char **h_addr_list //IP地址表

```C++
#include <stdio.h>
#include <netdb.h>
#include <arpa/inet.h>

int main(int argc, char* argv[])
{
  struct hostent* host = gethostbyname(argv[1]);
  if(host == NULL)
  {
    perror("gethostbyname");
    return -1;
  }
  printf("主机官方名:\n");
  printf("%S\n", host->h_name);
  printf("主机别名表\n");
  for(char** pp = host->h_aliases; *pp; pp++)
  {
    printf("%s\n", *pp);
  }
  printf("IP地址表:\n");
  for(struct in_addr** pp1 = (struct in_addr**)host->h_addr_list; *pp1; pp1++)
  {
		printf("%s\n", inet_ntoa(*pp1));
  }
  return 0;
}
//运行 通过执行时传入域名地址  ./dns www.baidu.com
```



# HTTP协议

应用层协议

客户端和服务器端请求和应答的标准

客户端 服务器通过协议标准发送 应答数据

## 请求头

![image-20240115220438475](C++.assets/image-20240115220438475.png)

**URL**:uniform resource locator 统一资源定位系统

![image-20240115220827208](C++.assets/image-20240115220827208.png)

若客户端发送请求头时 未指出获取的文件路径 那么浏览器在发送请求时URI部分为/

服务器接收到/后 返回该服务器的首页文件

> accept 主类型加子类型   
>
> 每一种拓展名的文件都由一个主类型加上子类型构成 邮箱拓展类型
>
> 如 .html 由 txt/html
>
>    .png 由image/pneg
>
>    .gif 由image/gif
>
> connection
>
> - close 只有一次
> - keep-alive 长久连接

结束\r\n  请求的末尾\r\n \r\n

## 响应头

![image-20240115221853796](C++.assets/image-20240115221853796.png)

![image-20240115222243131](C++.assets/image-20240115222243131.png)

状态码 200 OK  404 NOT FOUND  400 协议格式错误  

字段名:空格+内容

![image-20240115222408679](C++.assets/image-20240115222408679.png)

```C++
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>

int main(int argc, char* argv[]) //命令行参数 输入百度域名地址 百度服务器IP地址  URL
{
  char* name = argv[1];
  char* ip = argv[2];
  char* path = argv < 4 ? "" : argv[3];
  
  //创建套接字 tcp socket
  int sockfd = socket(AF_INET, SOCK_STREAM, 0);
  if (sockfd == -1)
  {
    perror("socket");
    return -1;
  }
  //组织服务器地址结构 struct sockaddr_in ser ; AF_INET 80 ser.ip = ip
  //代表一个通信的主体 (端口号 IP地址)
  struct sockaddr_in ser;
  ser.sin_family = AF_INET;
  ser.sin_port = htons(80); //http协议所使用80端口
  ser.sin_addr = inet_addr(ip);
  
  //向服务器发起连接  connect
  if(connect(sockfd, (struct sockaddr*)&ser, sizeof(ser)) == -1)
  {
    perror("connect");
    return -1;
	}
  
  //组织http请求  buf = "GET path HTTP/1.1\r\n HOST: name\r\n Accept: */*\r\n"
  char buf[1024] = {};
  sprintf(buf, "GET /%s HTTP/1.1\r\n 
          "HOST: %s\r\n" 
          "Accept: */*\r\n"
         	"User-agent: Mozilla/5.0\r\n"
          "Connection: Close\r\n"
         	"Referer: %s\r\n(对本行结束的表示符)\r\n(对请求头结束的表示符)",pathm name,name); //输出一个串到指定的存储区中
  //发送给服务器   send
  if(send(sockfd, buf, strlen(buf), 0) == -1 )
  {
    perror("send");
    return -1;
  }
  //接收服务器的响应 大小不确定 动态分配 循环接
  for(;;)
  {
    char respond[1024] = {};
    ssize_t size = recv(sockfd, respond, sizeof(respond)-1, 0);
  	if(size == -1)
    {
      perror("recv");
      return -1;
    }
    if(size == 0) //为收到数据时 停止接收
    {
      break;
    }
  	printf("%s", respond);
  }
  printf("\n");
  close(sockfd);
  return 0;
}
```

输出重定向 ./http www.baidu.com 220.181.38.150 >http/html

\r\n  == ^M

# 密码学

## OPENSSL加密的方式

https 中的s 用的就是openssl

C语言编写的一个开源的密码库

是一个安全套接字层密码库

没有官方帮助文档

ssl是安全套接字层协议

需要在网络通信时使用

TLS 传输层套接字协议

### protobuf 用于数据序列化 （谷歌）

跨平台传输的时候 一般都会将数据按照某种规定 组织排列起来  

保证 在一个平台到另一个平台的时候能够完完全全的解析出来

在不同系统中 不同数据类型的字节位数可能不同 例如在32位系统中 int占4个字节 到64位系统可能占8个字

**字节对齐**

`struct A`

`{`

`int a; //4`

`char b; //1`

`int c; //4`

`};`

在传输的时候，不同硬件平台不一定支持任何内存地址的存取。一般是以双字节 四字节为单位进行存取

所以在上面的结构体中 char 数据类型虽然只占一个字节 但是由于字节对齐 会给char 补足为4个字节

这样 将总字节为12的结构体进行序列化

所以如果多加一个char类型的变量

总值也是12字节

`struct A`
`{`
    `int a;	// 4`
    `char b;	// 1`
    `char cc;`
    `int c;	// 4`
`};`

- 序列化 -> 编码

  - 将原始数据按照某种格式进行封装 -> 特殊的字符串
  - 将特殊字符串发送给对方

- 反序列化 -> 解码

  - 接收到序列化的特殊字符串 -> 解析-> 原始数据
  - 安装业务需求处理原始数据

  ## 套接字通信

  - **tcp**

  - **线程池**

    服务器端使用

    在服务器端 线程使用完以后 就回直接销毁 回收 

    线程池就是将使用完的线程 回收后重新使用加载

  - **连接池**

    客户端使用

    多线程的使用

  ## 共享内存-> share memory

  **进程间通信的一种方式**

  - **效率最高**
    - **因为进程间通信的方式大多都需要使用到**fd 而共享内存不需要访问磁盘文件
      - **管道**
        - 匿名管道->不需要读磁盘 但是进攻有血缘关系的进程，进行进程间通信-
        - 有名管道->需要磁盘文件

**tortoiseGit 进行项目版本管理工具 管理源代码**

## 加密三要素

- 明文/密文
  - 明文->原始数据
  - 密文->加密之后的数据

- 秘钥

  - 定长的字符串
  - 自己生成 只适用于对称加密
  - 非对称加密->有对应的生成函数API 可以直接调用API
  - 在加密和解密的时候都会参加对应的行为

- 算法

  - 加密算法
    - 加密算法 + 明文 + 秘钥 = 密文

  - 解密算法
    - 解密算法 + 密文 + 秘钥 = 明文


  - 举例

    ```
    明文：123
    秘钥：111
    加密算法：明文 + 秘钥
    加密：123 + 111 = 密文 == 234
    密文：234
    解密算法：密文 - 秘钥 
    解密：234 -111 = 明文 ==123
    ```

### 常用的加密方式

  - 对称加密

    - 秘钥比较短
    - 秘钥只有一个
      - 加密和解密用的秘钥是相同的
    - 加密强度相对较低(相对于非对称加密) 因为秘钥比较短
    - 秘钥分发困难 秘钥必须要保密 所以在分发的时候不能泄露
      - 秘钥不能直接在网络环境中进行发送
        - 例如 A -> B发送消息 被C拦截 此时A 生成一个秘钥后 将秘钥和消息一起发送给B C还是能够拦截 并且可以将消息丢弃后 通过秘钥重新加密一个消息发送给B 形成重放攻击

  - 非对称加密

    - 秘钥比较长 所以相对的计算量比较高 

    - 效率比较低 但加密强度高

    - 加解密使用的秘钥对有两个，所有的非对称加密算法都有生成密钥对的函数

      - 这两个秘钥对保存到不同的文件中，一个文件是公钥(比较小)，一个文件是私钥（比较大）
      - `通过私钥可以把公钥给推出来`
      - 公钥 ->可以公开的
      - 私钥 ->不能公开 需要妥善保存

    - 加解密使用的秘钥不同

      - 如果使用公钥加密，`必须`使用私钥解密
      - 如果使用私钥加密，`必须`公钥解密

    - 秘钥可以直接分发 -> 分发的公钥

      - 举例

        ```发送方生成秘钥对
        A与B通信， 如果在A端生成密钥对 然后将公钥和消息包发送给B C拦截到消息包和公钥后 由于消息包是私钥加密的 所以呢根据私钥加密公钥解密的原则 那么C就可以根据公钥进行解密 获取到A的消息内容 但无法通过私钥加密后 进行重放攻击
        ```

        ```接收方生成密钥对
        A与B通信， 在B端生成秘钥对， 然后将公钥发送给A A根据B发送的公钥进行加密 然后发送一个A的公钥给B B根据自己的私钥解密 这样C获取到公钥和公钥加密的数据包 那么就无法知道数据包的内容 但可以发送一个加密的数据包给接收方
        这样做得坏处是 两端都会各自生成秘钥和公钥 大大降低效率 所以做法为将公钥分发出去 私钥各自保留
        ```

        

定长的字符串

是对于算法来说的

## 对称加密

### 算法

- DES/3DES
  - DES 到现阶段已被破解 不安全 但在未被破解前 使用的最广泛的对称加密
  - 秘钥长度 8bytes
  - 对数据进行分段， 每8个字节为一组 每个组通过公钥进行加密后 变为密文的一组 将多个密文段组合在一起就是一个完整的密文
    - 得到的密文和明文长度相同
    - 当有一组不够8个字节的时候 那一组会被填充为8个字节 解密的时候 把填充的数据删除
- 3DES -> 3重des
  - 安全的, 效率比较低
  - 对数据分段加密, 每组8字节
  - 得到的密文和明文长度是相同的  == 8字节
  - 秘钥长度24字节, 在算法内部会被平均分成3份, == 每份8字节
    - 看成是3个秘钥
    - 每个8字节
  - 加密处理逻辑:
    - 加密: 	-> 秘钥1  * 加密算法
    - 解密     -> 秘钥2   * 解密算法 
    - 加密     -> 秘钥3   * 加密算法
  - 三组秘钥都不同, 加密的等级是最高的
  - 这么设置可以兼容DES 因为当三个秘钥为相同的时候 前两次加密无效 直到第三次加密才是真正的加密

### 怎么填充的呢？

- 每组八个字节 假设需要发送的秘钥长度为17个字节 那么最后一组就需要填充7个数据包 形成8个字节的数据段 填充包中的数据为几呢？
  - 一般来说为填充的字节数 例如需要填充七个 那么就填充七个7 那么当读取的时候 看到最后一个数据段的最后一个数据为7就截取后七个数据包 然后保留对应剩下的为有效数据包
  - 当秘钥长度刚好够8整除的时候
    - 例如 秘钥长度为16 刚好够两组 那么此时还是会读取最后一个数据包的数据 怎么办呢？
      - 填充一组数据包 每个数据包的数据为8 这样读取到8的时候 截取八个字节 最后剩下的就是正确的数据段

### AES

- 目前最安全, 效率最高的公开的对称加密算法
- 秘钥长度: 16字节, 24字节, 32字节
  - 秘钥越长加密的数据越安全, 效率越低
  - 16 + 8 = 24 16 + 16 = 32
- `分组加密, 每组长度 16 字节`
- 每组的密文和明文的长度相同  == 16byte



## 非对称加密

### 算法

- RSA（**数字签名和密钥交换**）
  - 公钥私钥 是两个非常大的数字 生成的是字符串 由于没有合适的数据类型进行存储
- ECC **效率最高** （椭圆曲线加密算法）
  - 椭圆曲线和一条直线的交点
- Diffie-Hellman(DH 密钥交换)
- El Gamal (数字签名)
- DSA （数字签名）

### 密钥交换过程

```
通信的双方为:客户端C，服务器端S

为什么要交换？
非对称加密 密钥分发方便 但是生成秘钥的效率低 所以使用对称加密
使用对称加密 
秘钥分发困难
使用非对称加密进行秘钥分发
分发是对称加密的秘钥 本质就是一个字符串

秘钥交换过程
在服务器端生成一个非对称加密的秘钥对:公钥 私钥
服务器将公钥发送给客户端 客户端有了公钥
在客户端生成一个随机字符串
	对称加密所需要的秘钥
在客户端使用公钥对生成的对称加密的秘钥进行加密 生成密文
	公钥加密 私钥解密 私钥只有服务器端才有私钥
将密文发送给服务器 
服务器使用私钥解密生成对称加密的秘钥
双方使用同一个秘钥进行通信
```



### HASH算法（单项散列函数）

```
特点：
不管原始数据有多长
通过HASH算法进行运算
得到的结果的长度是固定的
	是一个二进制的字符串
只要是原始数据不同
得到的结果就不一样
	原始数据之间只有一个字符之差 得到的结果也是完全不同的
	散列值相似，原始数据之间差距也很多
有很强的抗碰撞性
	通过不断的造数据
	原始数据不同，但是通过同样的哈希算法进行计算能得到相同的结果
	结论
		数据不同得到的结果就不同
	应用场景：
		数据校验
         看哈希值是否相同 因为哈希值几乎唯一
		例如下载的软件等 软件官方会提供MD5 用于确定下载的文件与官方提供的文件是否相同
		如果对文件进行哈希运算 如果与提供的散列值不同 说明文件遭受修改
		登陆验证
			对用户的密码进行保护
			设置密码时 系统保护的不是密码本身，而是保护密码生成的散列值，当第二次登陆的时候，输入密码，对其进行运算的时候，与原先结果一致，说明密码正确，这样就有效防止，系统管理员能够看得到普通用户的密码
			对于网站论坛 手机支付应用等，数据库中存储的也是其经过算法后保存的加密序列。每次登陆时通过计算后比对，验证密码是否正确
			网盘秒传
			对于网盘秒传，不论多少带宽都能秒传上去，是系统对文件的散列值进行运算，与数据库中的文件散列值进行比对，如果数据库中有这个散列值，说明服务器有这个文件，那么就不需要再传一次到网盘中，这样就实现了秒传功能，但当数据库中没有这个功能时，则功能失效。
			结果不可逆
			得到的结果不能推导出原始数据
			因为结果是很短的一个字符串
			并不知道原始数据有多大
			如果能还原 时代将会翻天覆地，硬盘等只需要几K的字符 就能还原1G 或者1T的大小的值
```

哈希运算的结果

- 散列值
- 指纹
- 摘要
  - 对数据进行哈希运算得到的结果 称之为摘要



openssl 提供函数API

MD4/MD5

- 散列值长度：16字节
  - 模拟碰撞 得到的散列值相同
  - 抗碰撞性已被破解
  - 不安全
  - 通过不同的数据 通过模拟碰撞 使得有相同的散列值

SHA-1

- 散列值长度：20字节
- 抗性碰撞已被破解

SHA-2 一系列的算法

- sha 224
  - 散列值长度 224bit / 8 = 28 byte
- sha 256
  - 256bit / 8 = 32 byte
- sha 384
  - 384 bit / 8 = 48 byte
- sha 512
  - 512 bit / 8 = 64 bit
- 安全的算法 但效率低 散列值很长 

SHA3-224/SHA3-256/SHA3-384/SHA3-512

最后的编码为bit位 底层的hash算法不一样



HASH算法不能称为加密算法 因为加密后 不能还原

如果不需要还原 则可以使用HASH



### 消息验证码->HMAC

作用

- 并没有加密的功能

- 在通信的时候，只是**校验通信的时候是否被篡改**（被篡改）

使用

- 消息验证码 也是一个散列值

- （原始数据+秘钥）*哈希函数 = 消息认证码
  - 最关键的数据：秘钥 
  - 国产 SM加密系列

- 校验过程
  - 数据发送方A 数据接收方B
  - 在A或B端生成一个秘钥：X 将秘钥进行分发->A和B端都有秘钥X
  - 在A端进行散列值运算：（原始数据 + ）*哈希函数 = 得到散列值
  - 在A端：将原始数据和散列值同时发送给B

  - 在B端 AB端使用的哈希算法是相同的
    - 接收数据
    - 校验：（接收的原始数据 + x ） * 哈希函数（相同的算法）= 散列值 New
    - 比较散列值 散列值 New 和 接收的散列值 是不是相同
      - 相同: 没有篡改
      - 不同: 被篡改

- 缺点

  - 秘钥分发困难 由于AB端的秘钥相同通过网络分发
  - 不能够校验出消息的所有者 当秘钥泄漏时 第三方知道后 可以通过 秘钥 + 数据 *哈希算法 发送给接收方

### 数字签名

> 作用：
>
> - 校验数据有没有被篡改(完整性)
> - 鉴别数据的所有者
> - 不对数据进行加密
>
> 数字签名的过程：私钥加密 公钥解密 
>
> - 生成一个非对称加密的密钥对，分发公钥
> - 使用哈希函数对原始数据进行哈希运算->散列值 由于直接对原始数据进行非对称加密 则算法效率很低
> - 使用私钥对数据进行加密->密文 只有公钥能解 用的是对方的公钥 
> - 对散列值进行加密 ->密文 
> - 将原始数据和密文一起发送给接收者
>
> 校验签名的过程：
>
> - 接收签名一方分发的公钥
>
> - 接收签名者发送的数据：接收的原始数据+签名
>
> - 对数据进行判断： 
>
>   - 对接收的原始数据进行哈希运算->取得散列值New 哈希算法和签名时所用算法相同
>
>   - 使用得到的公钥和签名(密文)解密->得到了散列值 old
>
>   - 比对 两个散列值是否相同 new 与 old 
>
>     - 相同：数据所有者确实是A 并且数据没有被篡改
>     - 不相同:数据所有者不是A 或者数据被篡改了
>
> 



在古典密码体制中，倒序容码属于置换密码

完整的密码体制包括 明文空间 密文空间 秘钥空间

在DES算法中 输入密文K（64）位，去除校验位 得到64位秘钥

因为在DES算法中 密文长度与明文长度相同 且每八位一组 64位刚好除尽 所以会添加八位校验位也就是72位 72位减去八位得64位

