# 罗剑锋的C++实战笔记
**面向对象**是C++里另一个基本的编程范式。核心：抽象、封装，倡导的是把任务分解成一些高内聚低耦合的对象，这些对象互相通信协作来完成任务。它强调对象之间的关系和接口，而不是完成任务的具体步骤。

在C++里，面向对象范式包括class、public、private、virtual、this等类相关的关键字，还有构造函数、析构函数、友元函数等概念。

**泛式编程**是自STL（标准模版库）纳入到C++标准以后才逐渐流行起来的新范式，核心思想是“一切皆为类型”，或者说是“参数化类型”“类型擦除”，使用模版而不是继承的方式来复用代码，所以运行效率更高，代码也更简洁。

在C++里，泛型的基础就是template关键字，然后是庞大而复杂的标准库，里面有各种泛型容器和算法，比如：vector、map、sort等。

“泛式编程”类似“模版元编程”

C++程序等生命周期包括：编码、预处理、编译、运行四个阶段；
C++支持面向过程、面向对象、泛型、模版元、函数式共五种主要的编程范式；


#### 课下作业
1. 你是怎么理解 C++ 程序的生命周期和编程范式的？
2. 试着从程序的生命周期和编程范式的角度，把 C++ 和其他语言（例如 Java、Python）做个比较，说说 C++ 的优点和缺点分别是什么。

### 命名规则
变量名：用于循环的 i/j/k、用于计数的 count、表示指针的 p/ptr、表示缓冲区的 buf/buffer、表示变化量的 delta、表示总和的 sum...

使用前缀 i/n/sz 等来表示变量的类型
```
eg:
iNum/szName
```

1. 变量、函数名和名字空间用 snake_case,全局变量加“g_"前缀；
2. 自定义类名用 CamelCase，成员函数用 snake_case，成员变量加”m_“前缀；
3. 宏和常量应当全大写，单词之间用下划线连接；
4. 尽量不要用下划线作为变量的前缀或者后缀(比如 _local、name_)，很难识别。

```c++
#define MAX_PATH_LEN 256
int g_sys_flag;

name linux_sys {
    void get_rlimit_core();
}

class FilePath final
{
    public:
        void set_path(const string& str);
    private:
    string m_path;
    ing m_level;
}

```
模版函数get_value:
```c++
template<typename T>
int get_value(const T& v);
```

"#"也是一个预处理令，叫“空指令”，可以当作特别的预处理空行。
而“#”与后面的指令之间也可以有空格，从而实现缩进，方便排版。

#号都在行首，而且 if 里面的 define 有缩进，看起来还是比较清楚的。
以后你在写预处理代码的时候，可以参考这个格式。
```c++

#                           // 预处理空行
#if __linux__               // 预处理检查宏是否存在
#   define HAS_LINUX    1   // 宏定义，有缩进
#endif                      // 预处理条件语句结束
#

``` 

```c++
g++ lest03.cpp -E -o a.xxx      #输出预处理后的源码

#include “a.out"                // 完全合法的预处理包含指令，你可以试试
```

在写头文件的时候，为了防止代码被重复包含，通常要加上”Include Guard",也就使用 “#ifndef/#define/#endif”来保护整个头文件。
```c++
#ifndef _XXX_H_INCLUDED_
#define _XXX_H_INCLUDED_
// 头文件内容
#endif // _XXX_H_INCLUED_
```

编写一些代码片段，存进"*.inc"文件里，然后有选择地加载，用的号的话，可以实现“源码级别的抽象”
有一个用于数值计算的大数组，里面有成百上千数，放在文件里占了很多地方，特别“碍眼”

```c++
static uint32_t  calc_table[] = {  // 非常大的一个数组，有几十行
    0x00000000, 0x77073096, 0xee0e612c, 0x990951ba,
    0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3,
    0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988,
    0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91,
    ...                          
};
```
这个时候，你就可以把它单独摘出来，另存为一个“*.inc"文件，然后再用”#include“替换原来的大批数字。这样就节省了大量的空间，让代码更加整洁。
```c++
static uint32_t calc_table[] = {
# include "calc_values.inc"     // 非常大的一个数组，细节被隐藏
}；
```

使用宏的时间一定要谨慎，时刻记着以简化代码、清晰易懂为目标，不要“滥用”，避免导致源码混乱不堪，降低可读性。

因为宏的展开、替换发生在预处理阶段，不涉及函数调用、参数传递、指针寻址，没有任何运行期的效率损失，所以对于一些调用频繁的小代码片段来说，用宏来封装的效果比inline关键字要好，因为它真的是源码级别的无条件内联。

宏是没有作用域概念，永远是全局生效。所以，对于一些用来简化代码、起临时作用的宏，最好是用完后尽快用“#undef”取消定义，避免冲突的风险。
```c++
#define CUBE(a) (a) * (a) * (a)     // 定义一个简单的求立方的宏

cout << CUBE(10) << endl;           // 使用宏简化代码
cout << CUBE(15) << endl;           // 使用宏简化代码

#undef CUBE                         // 使用完毕后立即取消定义
```

宏定义前检查
```c++
#ifdef AUTH_PWD                     // 检查是否已经有宏定义
#   undef AUTH_PWD                  // 取消宏定义
#endif                              // 宏定义检查结束
#define AUTH_PWD "xxx"              // 重新宏定义
```

用宏来代替直接定义名字空间(namespace)：
```c++
#define BEGIN_NAMESPACE(x) namespace x {
#define END_NAMESPACE(x)
}

BEGIN_NAMESPACE(my_own)
    // functions and classes
END_NAMESPACE(my_own)
```

### 条件编译(#if/#else/#endif)
利用“#define”定义出的各种宏，我们还可以在预处理阶段实现分支处理，通过判断宏的数值来产生不同的源码，改变源码文件的形态，这就是“条件编译”。

条件编译有两个要点，一个是条件指令“#if”，另一个是后面的“判断依据”，也就是定义好的各种宏，而**这个“判断依据”是条件编译里最关键的部分**

编译环境都会有一些预定义宏，比如CPU支持的特殊指令集、操作系统/编译器/程序库的版本、语言特性等，

```c++
#ifdef __cplusplus                      // 定义了这个宏就是在用C++编译
    extern "C" {                        // 函数按照C的方式去处理
#endif
    void a_c_function(int a);
#ifdef __cplusplus                      // 检查是否是C++编译
    }                                   // extern "C" 结束
#endif

#if __cplusplus >= 201402                // 检查C++标准的版本号
    cout << "c++14 or later" << endl;    // 201402就是C++14
#elif __cplusplus >= 201103              // 检查C++标准的版本号
    cout << "c++11 or before" << endl;   // 201103是C++11
#else   // __cplusplus < 201103          // 199711是C++98
#   error "c++ is too old"               // 太低则预处理报错
#endif  // __cplusplus >= 201402         // 预处理语句结束

```

条件编译还有一个特殊的用法，那就是，使用“#if 1”“#if 0”来显示启用或者禁用大段代码，要比“/*...*/的注释方式安全得多
```c++
#if 0           // 0即禁用下面的代码，1则是启用
...
#endif          // 预处理结束

#if 1           // 1启用代码，用来强调下面代码的必要性
...
#endif          // 预处理结束
```

1. 预处理不属于C++语言，过多的预处理语句扰乱正常的代码，除非必要，应当少用慎用；
2. “#include"可以包含任意文件，所以可以写一写小的代码片段，再引进程序里；
3. 头文件应该加上"Indlude Guard",防止重复包含；
4. ”define“用于宏定义，非常灵活，但滥用文本替换可能会降低代码的可读性；
5. ”条件编译”其实就是预处理编程里的分支语句，可以改变源码的形态，针对系统生成最合适的代码。


- “自动”就是让计算机去做，而不是人去做，相对的是“手动”
- “类型”指的是操作目标，出来的是编译阶段的类型，而不是数值
- “推导”就是演算、运算、把隐含的值给算出来。

**在变量声明时应该尽量多用aotu**

为了保证效率，最好使用“const auto&”或者“auto&”
```c++
```
在 C++14 里，auto 还新增了一个应用场合，就是能够推导函数返回值，这样在写复杂函数的时候，比如返回一个 pair、容器或者迭代器，就会很省事。


### 课外小帖士
1. 在C语言里，auto关键字最早的含义是表示局部变量，与static同级，但因为用得极少，所以到了c++11，为了避免再新增关键字，就给“变废为宝”了。
2. C++标准又特别规定，类的静态成员变量允许使用auto自动推导类型，但我建议，为零与非静态成员保持一致，还是统一不使用auto比较好。
3. C++14新增了字面量后缀“s”，表示标准字符串，所以就可以用“auto str = ”xxx“s；”的形式直接推导出std::string类型。
4. C++17为auto增加了一种叫“结构化绑定”的功能，相当于简化的tie()用法。



## const与volatile
定义程序用到的数字、字符串常量，代替宏定义。
```c++
const int MAX_LEN       = 1024;
const std::string NAME  = "metroid";
```
const定义的常量在预处理阶段并不存在，而是直到运行阶段才会出现。
使用指针获取地址，
```c++
// 需要加上volatile修饰，运行时才能看到效果
const volatile int MAX_LEN  = 1024;

auto ptr = (int*)(&MAX_LEN);
*ptr = 2048;
cout << MAX_LEN << endl;
```

**const就成了常量引用和常量指针**
```c++
int x = 100;

const int& rx = x;
const int* px = &x;
```

const& 被称为**万能引用**

在设计函数的时候，我建议你尽可能地使用它作为入口参数，一来保证效率，而来保证安全。

const放在声明的最左边，表示指向常量的指针：
```c++
string name = "uncharted";
const string* ps1 = &name;  // 指向常量
*ps1 = "spiderman";         // 错误，不允许修改
```

const在“*”的右边，表示指针不能被修改，而指向的变量可以被修改：
```c++
string* const ps2 = &name;  // 指向变量，但指针本身不能被修改
*ps2 = "spiderman";         // 正确，允许修改
```

const用法都是面向过程的，在面向对象里，const也很有用。
定义const成员变量简单，但你用过const成员函数吗？像这样：
```C++
class DemoClass final
{
    private:
        const long MAX_SIZE = 256;      // const成员变量
        int        m_value;             // 成员变量
    public:
        int get_value() const           // const成员变量
        {
            return m_value;
        }
};
```

**成员函数是一个“只读操作”**

volatile可以用来修饰任何变量，而mutable却只能修饰类里面的成员变量，表示变量即使是在const对象里，也是可以修改的。

**对象内部用到了一个mutex来保证线程安全，或者有一个缓冲区来暂存数据，再或者一个原子变量做引用计数**

对于这些特殊作用的成员变量，你可以给它`加上mutable修饰，解除const的限制`，让任何成员函数都可以操作它。
```c++
class DemoClass final
{
    private:
        mutable mutex_type m_mutex;     // mutable成员变量
    public:
        void save_data() const
        {
            // do something with m_mutex
        }
}
```

## 小结
1. const
- 它是一个类型修饰符，可以给任何对象附加上“只读”属性，保证安全；
- 它可以修饰引用和指针，“const&”可以引用任何类型，是函数入口参数的最佳类型；
- 它还可以修饰成员函数，表示函数是“只读”的，const对象只能调用const成员函数。

2. volatile

- 它表示变量可能会被“不被察觉”地修改，禁止编译器优化，影响性能，应当少用。

3. mutable

- 它用来修饰成员变量，允许const成员函数修改，mutable变量的变化影响对象的常量性，但要小心不要误用损坏对象。

对于只读的函数，就要加上const修饰。

### 课外小帖士
1. 和预处理阶段的规则类似，常量的名字通常也用全大写的形式，但也有另外一种风格，就是在名字前加上“k”前缀。
2. 因为函数参数是“传值”语义，所以对于简单的值类型，如int、double、不同const&的形式，也不会影响效率。
3. const_cast是C++的四个转型操作符之一，专门用来去除“常量性”，可以用在某些极特殊的场景，比如调用纯C接口，但应当少用，最好不用。
4. 成员函数有一个隐含的this参数，所以从语义上来说，const成员函数实际上是传入了一个const this指针，但因为C++语法限制，无法声明const this，所以就把const放到了函数后面。
5. 依据应用场景，有点成员函数可能既是const又是非const，所以就会有两种重载形式，比如vector的front()、at()等，如果是const对象编译起就会调用const版本。
6. C++11里mutable又多了一种用法，可以修饰lambda表达式。
7. C++11引入了新关键字constexpr，能够表示真正的编译阶段常量，甚至能够编写在编译阶段运行的数值函数。

**指针式内存地址，引用是变量别名，指针可以是空，而引用不能为空。**


### 什么事智能指针？
指针式源自C语言的概念，本质上是一个内存地址索引，代表了一小片内存区域(也可能会很大)，能够直接读写内存。

因为它完全映射了计算机硬件，所以操作效率高，是C/C++高效的根源。
访问无效数据、指针越界，或者内存分配后没有及时释放，就会导致运行错误、内存泄漏、资源丢失等一系列严重的问题。

`C++里也有垃圾回收的，不过不是Java、Go那种严格意义上的垃圾回收，而是广义上的垃圾回收，这就是构造/析构函数和RAII惯用法`

应用代理模式，把裸指针包装起来，在构造函数里初始化，在析构函数里释放。
这样当对象失效销毁时，C++就会**自动**调用析构函数，完成内存释放、资源回收等清理工作。

智能指针：能够自动适应各种复杂的情况，防止无用指针导致的隐患。

常用的智能指针有两种：unique_ptr和shared_ptr
```c++
unique_ptr<int> ptr1(new int(10));      // int智能指针
assert(*ptr1 == 10);                    // 可以使用*取内容
assert(ptr1 != nullptr);                // 可以判断是否为空指针

unique_ptr<string> ptr2(new string("hello"));   // string智能指针
assert(*ptr2 == "hello");                       // 可以使用*取内容
assert(ptr2->size() == 5);                      // 可以使用->调用成员函数
```
unique_ptr**实际上并不是指针，而是一个对象。所以，不要企图对它调用delete，它会自动管理初始化时对指针，在离开作用域时析构释放内存**

##### 调用工厂函数make_unique()
强制创建智能指针的时候必须初始化。
```c++
auto ptr3 = make_unique<int>(42);           // 工厂函数创建智能指针
assert(ptr3 && *ptr3 == 42);

auto ptr4 = make_unique<string>("god of war");  // 工厂函数创建智能指针
assert(!ptr4->empty());
```

```c++
template<class T, class... Args>            // 可变参数模版
std::unique_ptr<T>                          // 返回智能指针
my_make_unique(Args&&... args)              // 可变参数模版的入口参数
{
    return std::unique_ptr<T>(
        new T(std::forward<Args>(args)...)); // 完美转发
}
```

shared_ptr支持安全共享的密码在于**内部使用了“引用计数”**

引用计数最开始的时候是1，表示只有一个持有者。如果发生拷贝赋值————也就是共享的时候，引用计数就增加，而发生析构销毁的时候，引用计数就减少。只有当引用计数减少到0。没有任何人使用这个指针的时候，它才会真正调用delete释放内存。

```c++
class Node final
{
    public:
        using this_type     = Node;
        using shared_type   = std::shared_ptr<this_type>;

    public:
        shared_type     next;          // 使用智能指针来指向下一个节点
};

auto n1 = make_shared<Node>();          // 工厂函数创建智能指针
auto n2 = make_shared<Node>();          // 工厂函数创建智能指针

assert(n1.use_count() == 1);            // 引用计数为1
assert(n2.use_count() == 1);

n1->next = n2;                          // 两个节点互指，形成了循环引用
n2->next = n1;

assert(n1.use_count() == 2);            // 引用计数为2
assert(n2.use_count() == 2);            // 无法减到0，无法销毁，导致内存泄漏
```

用好unique_ptr、shared_ptr

**异常只是C++为了处理错误而提出的一种解决方案，当然也不回是唯一的一种。**
```c++
int n = read_data(fd, ...);     // 读取数据

if (n == 0) {
    ...                         // 返回值不太对，适当处理
}

if (errno == EAGAIN) {
    ...                         // 适当处理错误
}
```

##### 异常就是针对错误的缺陷而设计的，它有三个特点：
1. 异常的处理流程是完全独立的。
2. 异常是绝对不能被忽略的，必须被处理。
3. 异常可以用在错误码无法使用的场合，

用try把可能发生异常的代码“包”起来，然后编写catch块捕获异常并处理。

```c++
try
{
    int n = read_data(fd, ...);     // 读取数据，可能抛出异常
    // do some right thing
}
catch(...)      
{
    // 集中处理各种错误情况
}
```


###### 闭包
闭包：一个“活的代码块”活的函数。

"[=]"表示按值捕获所有外部变量，表示式内部是值的拷贝，并且不能修改；
“[&]"是按值引用捕获所有外部变量，内部以引用的方式使用，可以修改；

using string = std::basic_string<char>;

using wstring = std::basic_string<wchar_t>;
using u16string = std::basic_string<char16_t>;
using u32string = std::basic_string<char32_t>;

string是一个功能比较齐全的字符串类，可以提取子串、比较大小、检查长度、搜索字符
```c++
string str = "abc";

assert(str.length() == 3);
assert(str < "xyz");
assert(str.substr(0, 1) == "a");
assert(str[1] == 'b');
assert(str.find("1") == string::npos);
assert(str + "d" == "abcd");

```

把每个字符都看作是一个不可变量的实体，才能在c++里真正的用好字符串。
原字符串表示形式：
```c++
auto str = R"nier:automata)";
```

```c++
auto str1 = R"(char""'')";      // 原样输出：char“”‘’
auto str2 = R"(\r\n\t\")";      // 原样输出：\r\n\t\"
auto str3 = R"(\\\$)";          // 原样输出：\\\$
auto str4 = "\\\\\\$";          // 转义后输出：\\\$

auto str5 = R"==(R"(xxx)")==";  // 原样输出：R"(xxx)"
```

regex_match()检查字符串
```c++
assert(regex_match(str, what, reg));

for(const auto& x : what) {
    cout << x << ',';
}
```

“并发”是指在一个时间段里有多个操作在同时进行，与“多线程”并不是一回事。

多线程的好处：
- 任务并行、避免 I/O 阻塞、充分利用 CPU、提高用户界面响应速度。

同步、死锁、数据竞争、系统调度开销等。