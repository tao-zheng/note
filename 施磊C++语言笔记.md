# C++语言



## 第一章 三块核心内容

### 1. 进程虚拟地址空间内存划分与布局

```c++
地址范围               分段名称 (Segment)           存储内容与变量类型说明
(高地址)
0xFFFFFFFF +----------------------------------+  
           |         内核空间 (Kernel)         |  内核代码、全局变量、页表 (32位通常1GB)
           +----------------------------------+  --------------------------------------
           |      环境变量与命令行参数         |  envp[], argv[] 字符串
           +----------------------------------+  
           |            栈 (Stack)             |  局部变量、函数参数、返回地址
           |             |                     |  [向下增长 ↓]
           +-------------|--------------------+  
           |             V                     |  
           |          (空闲区)                 |  由 ASLR 随机化起始位置
           |             ^                     |  
           +-------------|--------------------+  
           |      动态库 / 内存映射区           |  libc.so、mmap 映射的文件或匿名内存
           +----------------------------------+  
           |             ^                     |  
           |             |                     |  
           +-------------|--------------------+  
           |            堆 (Heap)              |  malloc / new 动态申请的空间
           |                                   |  [向上增长 ↑]
           +----------------------------------+  
           |         BSS 段 (.bss)             |  1. 未初始化的全局/静态变量
           |   (Block Started by Symbol)       |  2. 初始化为 0 的全局/静态变量 (优化)
           +----------------------------------+  
           |        数据段 (.data)             |  已初始化且非零的全局/静态变量
           +----------------------------------+  
           |        代码段 (.text)             |  二进制指令、只读常量 (如 "hello" 字符串)
           +----------------------------------+  
           |      不可访问预留区 (Reserved)     |  捕获 NULL 指针，触发段错误
(低地址)    |  (0x00000000 - 0x08048000 左右)   |  保护系统安全
0x00000000 +----------------------------------+
```

```c++
#include <iostream>
 using namespace std;

int gdata1 = 10;   // -->.data     
int gdata2 = 0;    // -->.bss
int gdata3;        // -->.bss
static int gdata4 = 11; // -->.data  
static int gdata5 = 0;  // -->.bss
static int gdata6;      // -->.bss

int main{
    int a = 12;  // -->Stack
    int b = 0;   // -->Stack
    int c;       // -->Stack
    static int e = 13;  // -->.data  
    static int f = 0;   // -->.bss
    static int g;       // -->.bss
    
    return 0;
}
```

**==每一个进程的用户空间是私有的，内核空间是共享的==**



### 2. 函数的调用堆栈详细过程

```c++
#include <iostream>
using namespace std;

int sum(int a,int b){
    int temp = 0;
    temp = a + b;
    return temp;
}

int main(){
    int a = 10;
    int b = 20;
    int ret = sum(a,b);
    cout<<ret<<endl;
    return 0;
}
```

**==第一阶段：main 函数栈帧完全初始化==**

**程序刚开始运行，在 main 内部为局部变量分配空间。**

```c++
内存地址           栈内存内容 (4字节/格)        说明 / 寄存器状态
(高地址)
0x0060FE00       |   旧 EBP (系统级)         | <--- main 的 EBP (基址)
0x0060FDFC       |   变量 a = 10            | [EBP - 4]
0x0060FDF8       |   变量 b = 20            | [EBP - 8]
0x0060FDF4       |   变量 ret (初始随机值)    | [EBP - 12] 
0x0060FDF0       +--------------------------+ <--- main 的 ESP (栈顶)
(低地址)
```

**==第二阶段：main 函数执行压栈传参==**

**main 准备调用 sum(a, b)。根据调用约定，参数从右往左压入。**

```c++
内存地址 (十六进制)   栈内存内容 (4字节/格)        说明 / 寄存器状态
(高地址)
0x0060FE00       |   旧 EBP (系统级)         | <--- main 的 EBP
0x0060FDFC       |   变量 a = 10            | 
0x0060FDF8       |   变量 b = 20            | 
0x0060FDF4       |   变量 ret               | 
0x0060FDF0       |   参数 b = 20 (副本)      | [EBP - 16] 准备传给 sum
0x0060FDEC       |   参数 a = 10 (副本)      | [EBP - 20] 准备传给 sum
0x0060FDE8       +--------------------------+ <--- ESP (此时指向参数 a)
(低地址)
```

**==第三阶段：进入 sum 函数（两个栈帧并存的完整状态）==**

**执行了 CALL sum（压入返回地址），进入 sum 后执行 push ebp（保存 main 的基址）并分配 temp。**

```c++
内存地址 (十六进制)   栈内存内容 (4字节/格)        说明 / 寄存器状态
(高地址)
0x0060FE00       |   旧 EBP (系统级)         | --- [main 栈帧开始]
0x0060FDFC       |   变量 a = 10            |
0x0060FDF8       |   变量 b = 20            |
0x0060FDF4       |   变量 ret               |
0x0060FDF0       |   参数 b = 20 (副本)      | [EBP_sum + 12]
0x0060FDEC       |   参数 a = 10 (副本)      | [EBP_sum + 8]
0x0060FDE8       |   返回地址 (RET ADDR)     | [EBP_sum + 4] (cout指令地址)
0x0060FDE4       |   旧 EBP (0x0060FE00)    | <--- sum 的 EBP 指向这里 (0x0060FDE4)
0x0060FDE0       |   变量 temp = 30         | [EBP_sum - 4] --- [sum 栈帧结束]
0x0060FDDC       +--------------------------+ <--- sum 的 ESP (当前栈顶)
(低地址)
```

==**第四阶段：sum 运行完毕，准备销毁栈帧**==

**执行 return temp;（结果存入 EAX 寄存器），接着执行 mov esp, ebp 和 pop ebp。 此时：EBP重新指向main底部，ESP 弹出了旧 EBP 走到了返回地址处。**

```c++
内存地址 (十六进制)   栈内存内容 (4字节/格)        说明 / 寄存器状态
(高地址)
0x0060FE00       |   旧 EBP (系统级)         | <--- EBP 重新指向 main 底部
0x0060FDFC       |   变量 a = 10            |
0x0060FDF8       |   变量 b = 20            |
0x0060FDF4       |   变量 ret               |
0x0060FDF0       |   参数 b = 20 (副本)      | 
0x0060FDEC       |   参数 a = 10 (副本)      | 
0x0060FDE8       |   返回地址 (RET ADDR)     | <--- ESP 指向这里，准备执行 ret
0x0060FDE4       +--------------------------+ 
(低地址)
```

**==第五阶段：回到 main 后的现场清理==**

**执行 ret，跳转回 main。main 负责清理栈上的参数（add esp, 8），并将 EAX 里的结果赋给 ret。**

```c++
内存地址 (十六进制)   栈内存内容 (4字节/格)        说明 / 寄存器状态
(高地址)
0x0060FE00       |   旧 EBP (系统级)         | <--- EBP (main)
0x0060FDFC       |   变量 a = 10            | 
0x0060FDF8       |   变量 b = 20            | 
0x0060FDF4       |   变量 ret = 30          | <--- 赋值完成
0x0060FDF0       +--------------------------+ <--- ESP (恢复到调用前)
(低地址)
```

==**1. sum 执行完之后怎么知道回到哪个函数？**==

**通过栈中保存的“旧 EBP”实现。sum 栈帧的底部存储了 main 的基地址 。当 sum 执行到最后，指令 pop ebp 会把这个值从栈里弹出，重新装进 CPU 的 EBP 寄存器。**

**==2. 回到 main 之后，怎么知道从哪一行继续执行？==**

**通过栈中保存的“返回地址 (Return Address)”实现。 在 main 执行 CALL sum 的瞬间，硬件自动在栈地址处压入一个 地址。这个地址对应被调用函数下面那一行的指令地址。**

**==3.函数调用参数为什么从右向左压栈？==**

**C，C++要支持可变参数**，从右向左压栈是为了让**第一个参数的位置固定在栈顶附近**。这样即使函数不知道参数的总个数（如 `printf`），它也能先拿到第一个参数，再通过第一个参数提供的信息去处理后续内容。



### 3.程序的编译链接原理

**==1.预处理==**

**编译器处理以 # 开头的指令，将 #include 的头文件内容直接拷贝到 .cpp 文件中、展开宏定义（#define）、处理预编译指令（#if）、擦除所有注释。**

**==2.编译==**

**将 C++ 代码翻译成汇编代码**。在这个阶段，编译器会检查语法错误（如分号缺失、类型不匹配）。

**==3.汇编==**

**将汇编指令翻译成机器可以识别的二进制目标代码，生成 .o（Linux）目标文件。.o 文件已经按照我们之前讨论的 .text、.data、.bss 段分好了块，但它还不能直接运行，因为里面的函数跳转地址（比如 call sum）还是空的占位符。**

**==4.链接==**

**一个程序通常由多个源文件组成，且会调用外部库。链接器的作用就是把这些分散的二进制文件“粘合”在一起。**

**①符号解析：为每一个“引用”找到对应的“定义”。**

**②段合并 ：**编译器生成的每个目标文件都有类似的结构（代码段 .text、数据段 .data、未初始化数据段 .bss 等）。 **链接器会将所有输入的 .o 文件中属性相同的段合并。**例如，把所有文件的代码段放在一起，形成一个巨大的代码段。

**③重定位 ：**在编译阶段，编译器并不知道函数 main 或变量 a  最终会被加载到内存的哪个地址，所以它在机器指令中会先填入一个**占位符**（比如全 0 地址）。**链接器在合并完所有段后，会计算出每个符号在内存中的绝对地址，然后回过头去修改机器指令中的占位符，将其替换为真实地址。**



## 第二章  C++基础部分

### 1.new/delete，malloc/free

**==new/delete 和 malloc/free 的区别是什么？==**

| **特性**          | **malloc / free**         | **new / delete**               |
| ----------------- | ------------------------- | ------------------------------ |
| **本质**          | **标准库函数 (stdlib.h)** | **C++ 关键字/运算符**          |
| **构造/析构函数** | **不调用**（只管分内存）  | **自动调用**（分内存+初始化）  |
| **返回类型**      | **void*（需要强转）**     | **对应类型的指针（类型安全）** |
| **内存大小**      | 需手动计算（`sizeof`）    | 编译器自动计算                 |
| **分配失败**      | **返回 NULL**             | **抛出 std::bad_alloc 异常**   |
| **数组处理**      | 无法区分单对象或数组      | 有专门的 `new[]` 和 `delete[]` |

```c++
int main(){
    int *p = (int *)malloc(sizeof(int));
    *p = 20;
    free(p);
    
    int *p1 = new int(5);
    delete p1;
    
    //
    int *q = (int *)malloc(sizeof(int)*20);
    free(q);
    
    int *q1 = new int[20]; //内存不初始化
    int *q1 = new int[20](); // 内存初始化为0
    delete []q1;
    
    
    return 0;
}
```



**==new 有多少种？==**

**==①表达式 new==：这是我们最常用的方式**

```c++
int* p = new int(10); // 申请内存并初始化为 10
MyClass* obj = new MyClass(); // 申请内存并调用构造函数
```

**==②数组 new==：当我们申请一个对象数组时，使用的是 new[]**

```c++
int* arr = new int[10]; // 申请 10 个 int 的空间
delete[] arr; // 必须配对使用 delete[]
```

**==③函数 new==：这是一个普通的函数（类似 malloc），它只负责分配内存，不会调用构造函数。**

```c++
// 手动分配 40 字节原始内存，不执行构造函数
void* rawMemory = operator new(40); 

// 对应的释放函数
operator delete(rawMemory);
```

**==④定位 new==：在已经分配好的一块内存地址上，“原地”构造一个对象。**

```c++
char buffer[1024]; // 预先分配好的栈空间
// 在 buffer 的起始地址上构造一个对象
Player* p = new (buffer) Player(); 

// 注意：不能用 delete p，必须手动调用析构函数
p->~Player();
```

**==⑤nothrow new==：不会抛出异常，失败返回NULL**

```c++
int* p = new (std::nothrow) int[1000000000]; 
if (p == nullptr) {
    // 内存分配失败，不会抛出异常
}
```



### 2.const、指针和引用的结合

==**C代码与C++代码 const 区别：**==

| **特性**       | **C 语言 (const)**         | **C++ 语言 (const)**                         |
| -------------- | -------------------------- | -------------------------------------------- |
| **性质**       | **只读的变量**             | **真正的常量（默认）**                       |
| **链接属性**   | 默认外部链接 (`extern`)    | 默认内部链接 (`static`)                      |
| **数组定义**   | 不可用于定义数组长度       | 可以用于定义数组长度                         |
| **编译器处理** | 运行时分配内存读取         | 编译期符号表替换（常量折叠），不去内存中读取 |
| **初始化**     | 可以不初始化（虽然没意义） | 必须初始化                                   |

```c++
// C++ 示例
const int a = 10;
int* p = (int*)&a;
*p = 20; 
printf("%d", a); // 依然输出 10 (常量折叠)
printf("%d", *p); // 输出 20 (内存确实改了，但 a 被替换成了 10)
```



**==const 修饰的变量常出现的错误：==**

**1.一定要初始化**

```c++
const int max_score; // 错误！const 变量必须初始化
```

**2.不能修改只读变量**

```c++
const int limit = 100;
limit = 200; // 错误：assignment of read-only variable 'limit'
limit++;    // 错误
```

**3.非 const 指针不能指向 const 变量**

```c++
const int val = 10;
int* ptr = &val;       // 错误！不能用普通指针指向 const 变量
const int* cptr = &val; // 正确
```

**4.常量指针与指向常量的指针混淆**

```c++
int x = 5;
const int* p1 = &x; // 指向常量的指针：不能通过 p1 修改 x
int* const p2 = &x; // 指针本身是常量：不能让 p2 指向别的地址

*p1 = 10; // 错误！
p2 = &val; // 错误！
```

**5.在 const 成员函数中不能修改非 static 成员**

```c++
class Counter {
    int count;
    void print() const {
        count++; // 错误！const 函数不能修改成员变量
    }
};
// 如果确实需要修改，需给变量加上 mutable 关键字。
```



**==C++ 引用和指针的区别==**

**1.引用是必须初始化的，指针就可以不初始化。**

**2.引用只有一级引用，没有多级引用；指针可以有一级指针，也可以有多级指针。**

**3.定义一个引用变量和定义一个指针变量，其汇编指令是一样的；通过引用变量修改变量的值，和通过指针变量修改变量的值，其底层汇编指令也是一样的。**

```c++
int main{
	int a = 10;
    int *p = &a;
    int &b = a;
    
    *p = 20;
    cout<<a<<*P<<b<<endl;
    b = 30;
    cout<<a<<*P<<b<<endl;
    
    int array[5] = {};
    *p = array;
    int (&q)[5] = array;
    cout<<sizeof(array)<<sizeof(p)<<sizeof(q)<<endl; //20,4,20
    return 0;
}
```



**==左值引用和右值引用==**

**左值：有内存、有名字，值是可以修改的；**

**右值：没内存、没名字**

**==左值引用==**：**只能绑定到左值（除非是 const T&）。**

```c++
int x = 10;
int& ref = x;      // 正确：左值引用绑定到左值
// int& ref2 = 10; // 错误！右值不能绑定到非 const 左值引用
const int& ref3 = 10; // 正确：const 引用延长了右值的生命周期，只读
```

**==右值引用==：只能绑定到右值。**

**①指令上，可以自动产生临时量，然后直接引用临时量；**

**②右值引用变量本身是一个左值，只能使用左值引用来引用它；**

```c++
int&& rref = 10;   // 正确,rref 是一个变量，它的类型是“右值引用（int&&）”。但是 rref 这个变量本身是有名字的，也可以使用 &rref 获取它的地址,因此它是一个左值。

int x = 5;
// int&& rref2 = x; // 错误！右值引用不能直接绑定左值
```



### 3.默认参数函数、inline函数、函数重载

**==默认参数函数==**

**1.默认参数必须==从右向左依次定义==。一旦某个参数有了默认值，它右边的所有参数都必须有默认值。**

**2.形参默认值可以由函数定义给出，也可以由函数声明给出，不管是谁给，==形参默认值只能出现一次。==**

```c++
#include <iostream>
using namespace std;

int sum(int a = 100,int b = 200){
    return a+b;
}

int main(){
    int a = 10;
    int b = 20;
    int ret = sum();
    ret = sum(a);
    ret = sum(a,b);
    cout<<ret<<endl;
}
```



==**inline内联函数**==

**1.inline内联函数：在编译过程中，==建议编译器用函数代码替换函数调用==，没有函数调用的开销了**

**2.==内联发生在编译阶段==。编译器必须在编译每个 .cpp 文件时就能看到函数的完整定义。**

```c++
#include <iostream>
using namespace std;

inline int sum(int a = 100,int b = 200){
    return a+b;
}

int main(){
    int a = 10;
    int b = 20;
    int ret = sum();
    ret = sum(a);
    ret = sum(a,b);
    cout<<ret<<endl;
}
```



==**函数重载**==

**同一作用域下一组函数，其函数名相同，参数列表的个数或者类型不同，那么这一组函数就称为函数重载。**

```c++
#include <iostream>
using namespace std;

bool compare(int a,int b){
    cout<<"int_int"<<endl;
    return a>b;
}
bool compare(double a,double b){
    cout<<"double_double"<<endl;
    return a>b;
}
bool compare(const char*a,const char* b){
    cout<<"char*_char*"<<endl;
    return strcmp(a,b)>0;
}

int main(){
    compare(10,20);
    compare(10.0,20.0);
    compare("aaa","bbb");
	
    return 0;
}
```

**==1.为什么C++支持函数重载，C语言不支持函数重载？==**

**C代码在编译函数名的时候，直接由函数名来决定；C++在编译函数名的时候，是由函数名+参数列表决定的，从而规避了后续链接阶段的符号冲突。**

**==2.函数重载需要注意什么？==**

**值传递 vs 引用传递：** 无法重载。

```c++
void test(int a);
void test(int& a); // 错误！调用 test(x) 时编译器无法区分
```

**常量性重载：** 指针和引用可以根据 `const` 进行重载。

```c++
void work(int* p);       // 版本 A
void work(const int* p); // 版本 B  正确
// 传入变量的地址 int* 调用 A，传入常量的地址(const int b = 20) const int* 调用 B
```

**==3.C++和C语言代码之间如何互相调用？==**

**C++调用C语言代码，不能直接调用。原因在于名字修饰导致的符号不匹配。**

**C++调用C语言：**

```c++
// main.cpp
extern "C" {f
    int add(int a, int b); // 告诉编译器 add 是 C 风格的符号
    #include "my_c_library.h" //或者包含整个 C 头文件（最常用）
}

int main() {
    add(1, 2); // 编译器会直接寻找符号 "add"，而不是 "_Z3addii"
    return 0;
}
```

C语言调用C++：

```c++
// sum.cpp
extern "C"{
    int sum(int a,int b){
        return a + b;
    }
}
```

```c
// main.c
int sum(int a,int b);

int main() {
    int ret = sum(10,20); // 直接调用
    return 0;
}
```





## 第三章  C++面向对象

### 1.this指针

==**OOP语言的四大特征：抽象，封装，继承，多态**==

**访问限定符**：public , private , protected

```c++
#include <iostream>
using namespace std;

const int NAME_LEN = 20;

class CGoods{
public: //给外部提供公有的成员方法，来访问私有的属性
    void init(const char* name,double price,int amount);
    void show();
    void setName(const char* name) {strcpy(_name,name);} 
    void setPrice(double price){_price = price;}
    void setAmount(int amount){_amount = amount;}
    const char* getName(){return _name;}
    double getPrice(){return _price;}
    int getAmount(){return _amount}
private: // 成员属性一般都是私有的
    char _name[NAME_LEN];
    double _price;
    int _amount;
};
void CGoods::init(const char* name,double price,int amount){
    strcpy(_name,name);
    _price = price;
    _amount = amount;
}
void CGoods::show(){
    cout<<_name<<endl;
    cout<<_price<<endl;
    cout<<_amount<<endl;
}

int main(){
    CGoods good;
    goods.init("面包",10,200);
    good.show();
    
    good.setPrice(20.5);
    good.setAmount(100);
    good.show();
    
    CGood good2;
    good2.init("空调",10000.0,50);
    good2.show();
    return 0;
}
```

**==类的大小：==**

**自身对齐：** 成员变量的起始地址必须是其自身大小的整数倍（例如 int 占 4 字节，它必须存放在 4 的倍数地址上）。

**整体对齐：** 类的总大小必须是类中**最大基本类型成员**大小的整数倍。

| **元素**         | **是否占用空间** | **占用多少**           |
| ---------------- | ---------------- | ---------------------- |
| **普通成员变量** | 是               | 自身大小 + 对齐补齐    |
| **静态成员变量** | 否               | 0 （在全局区）         |
| **普通成员函数** | 否               | 0（在代码区）          |
| **虚函数**       | 是 (指针)        | 4 或 8 字节 (一个指针) |
| **空类**         | 是               | 1 字节                 |

**==this指针：==**

**①CGood 类可以定义无数的对象，每一个对象都有自己的成员变量，但是它们共享一套成员方法。那么init(name,price,amount) 是怎么知道把信息初始化给哪一个对象的？**

**因为类的方法一经编译，所有的==方法参数，都会增加一个 this 指针，接收调用该方法的对象的地址==。**



### 2.构造函数与析构函数

**OOP实现一个顺序栈**

```c++
#include <iostream>
using namespace std;

class SeqStack{
public:
    SeqStack(int size = 10){ // 构造函数，带参数，可以被重载
        cout<<"构造函数调用"<<endl;
        _pstack = new int[size];
        _top = -1;
        _size = size;
    }
    ~SeqStack(){  // 析构函数,不带参数，不能被重载
        cout<<"析构函数调用"<<endl;
        delete [] _pstack;
        _pstack = nullptr;
    }
    void push(int val){
        if(full()){
            resize();
        }
        _pstack[++_top] = val;
    }
    void pop(){
        if(empty()){
            return;
        }
        --_top;
    }
    int top(){
        return _pstack[_top];
    }
    bool empty() {return _top == -1;}
    bool full() {return _top == _size - 1;}
    
private:
    int *_pstack; // 动态开辟数组，存储元素
    int _top;     // 指向栈顶元素的位置
    int _size;    // 数组扩容的总大小
    void resize(){
        int *ptmp = new int[_size*2];
        for(int i = 0;i < _size;i++){
            ptmp[i] = _pstack[i]; // 调用拷贝赋值运算符
        } // memcpy(ptmp,_pstack,sizeof(int)*_size)可能会有问题如果拷贝对象的话.如果存的是 含有指针的对象，memcpy 会导致浅拷贝，引发两次 delete 崩溃。
        delete []_pstack;
        _pstack = ptmp;
        _size = _size*2;
    }
};

int main(){
    SeqStack s;
    
    for(int i = 0;i < 15;i++){
        s.push(rand()%100);
    }
    
    while(!s.empty()){
        cout<<s.top()<<" ";
        s.pop();
    }

    SeqStack s1(50);  //先构造s的后析构，后构造s1的先析构
    
    SeqStack *ps = new SeqStack(60);
    ps->push(70);
    ps->push(80);
    ps->pop();
    delete ps;
     
    return 0;
}
```



### 3.对象深拷贝与浅拷贝

```c++
#include <iostream>
using namespace std;

class SeqStack{
public:
    SeqStack(int size = 10){ // 构造函数，带参数，可以被重载
        cout<<"构造函数调用"<<endl;
        _pstack = new int[size];
        _top = -1;
        _size = size;
    }
    SeqStack(const SeqStack &src){ //自定义拷贝构造函数
        cout<<"拷贝构造函数调用"<<endl;
        _pstack = new int[src._size];
        for(int i = 0;i <= src._top;i++){
            _pstack[i] = src._pstack[i];
        }
        _top = src._top;
        _size = src._size;
    }
    void operator=(const SeqStack &src){ // 重载拷贝赋值运算符
        cout<<"拷贝赋值运算符调用"<<endl;
        if(this == &src){ // 防止自赋值
            return;
        }
        delete []_pstack; //需要先释放当前对象占用的外部资源
        _pstack = new int[src._size];
        for(int i = 0;i <= src._top;i++){
            _pstack[i] = src._pstack[i];
        }
        _top = src._top;
        _size = src._size;   
    }
    ~SeqStack(){  // 析构函数,不带参数，不能被重载
        cout<<"析构函数调用"<<endl;
        delete [] _pstack;
        _pstack = nullptr;
    }
    void push(int val){
        if(full()){
            resize();
        }
        _pstack[++_top] = val;
    }
    void pop(){
        if(empty()){
            return;
        }
        --_top;
    }
    int top(){
        return _pstack[_top];
    }
    bool empty() {return _top == -1;}
    bool full() {return _top == _size - 1;}
    
private:
    int *_pstack; // 动态开辟数组，存储元素
    int _top;     // 指向栈顶元素的位置
    int _size;    // 数组扩容的总大小
    void resize(){
        int *ptmp = new int[_size*2];
        for(int i = 0;i < _size;i++){
            ptmp[i] = _pstack[i];
        } 
        delete []_pstack;
        _pstack = ptmp;
        _size = _size*2;
    }
};

int main(){
    SeqStack s1(10);
    // 默认拷贝构造函数-->直接进行内存拷贝,是浅拷贝
    SeqStack s2 = s1; // 自定义拷贝构造函数，深拷贝
    SeqStack s3(s1);  // 拷贝构造函数，同上
    
    SeqStack s4;
    s4 = s3; // 拷贝赋值运算符
    
    
    
    return 0；
}
```



==**string 类**==

```c++
class String{
public:
    String(const char* str = nullptr){ // 普通构造函数
        if(str != nullptr){
            m_data = new char[strlen(str)+1];
        	strcpy(this->m_data,str);
        }else{
            m_data = new char[1];
            *m_data = '\0';
        }  
    }
    String(const String &other){
        m_data = new char[strlen(other.m_data)+1];
        strcpy(m_data,other.m_data);
    }
    String& operator=(const String &other){
        if(this == &other){
            return *this;
        }
        delete [] m_data;
        m_data = new char[strlen(other.m_data)+1];
        strcpy(m_data,other.m_data);
        return *this;
    }
    ~String(){
        delete[]m_data;
        m_data = nullptr;
    }
private:
    char* m_data;
};
int main(){
    String str1;
    String str2("hello");
    
    String str3;
    str3 = str1 = str2; // str1 = str2;str1.operator=(str2)=>void   str3 = void ?因此赋值运算符最好返回当前类型的引用,为了连续赋值;返回引用不会产生临时对象,开销很小
    
    return 0;
}
```



**==构造函数的初始化列表==**

```c++
class CDate{
public:
    CDate(int y,int m,int d){ //自定义的构造函数，编译器不会产生默认构造函数了
        _year = y;
        _month = m;
        _day = d;
    }
    void show(){
        cout<<_year<<_month<<_day<<endl;
    }
private:
    int _year;
    int _month;
    int _day;
};

class CGoods{
public:
    CGoods(char* n,int a,double p，int y,int m,int d):_data(y,m,d),_amount(a),_price(p){ //初始化列表
        strcpy(_name,n);
    }
    void show(){
        cout<<_name<<endl;
        cout<<_amount<<endl;
        cout<<_price<<endl;
    }
private: // 可以在定义时直接初始化，构造函数依然可以覆盖他们
    char _name[20];
    int _amount = 0;
    double _price = 0.0;
    CDate _date{2006,1,1} // 成员对象 1.分配内存 2.调用构造函数
}
```



==成员变量在初始化列表中的**初始化顺序**是按照它们在**类中声明的顺序**决定的，而不是按照你在初始化列表中写的顺序。==

```c++
class Test{
public:
    Test(int data = 10):mb(data),ma(mb){}
    void show(){
        cout<<ma<<endl;
        cout<<mb<<endl;
    }
private:
    int ma;
    int mb;
}

int main(){
    Test t;
    t.show();
    
    return 0;
}
```



==**指向类成员（成员变量和成员方法）的指针**==

```c++
class Test{
public:
    void func(){
        cout<<"func()"<<endl;  
    }
    static void static_func(){
        cout<<"static_func()"<<endl;
    }
    int ma;
    static int mb;
};

int main(){
    Test t1;
    
    // 指向普通变量
    int Test::*p = &Test::ma;
    t1.*p = 20;
    cout<<t1.*p<<endl;
    
    // 指向static变量
    int* p1 = &Test::mb;
    *p1 = 40;
    cout<<*p1<<endl;
    
    // 指向普通成员函数
    void (Test::*pfunc)() = &Test::func;
	(t1.*pfunc)();      // 对象调用
    
    // 指向静态成员函数
    void (*pstatic_func)() = &Test::static_func;
	pstatic_func();     // 直接调用，无需对象
    
    return 0;
}
```



### 4.普通、static、const三类成员方法

| **特性**        | **普通成员方法**   | **静态成员方法 (static)**   | **常成员方法 (const)**         |
| --------------- | ------------------ | --------------------------- | ------------------------------ |
| **`this` 指针** | 有（指向当前对象） | **无**                      | 有（指向常对象，**const T***） |
| **访问成员**    | 可访问所有成员     | **仅可访问静态成员**        | 可访问所有成员，但**不能修改** |
| **调用方式**    | 对象.方法()        | 类名::方法() 或 对象.方法() | 对象.方法() 或 常对象.方法()   |
| **内存属性**    | 属于对象层级       | **属于类层级**              | 属于对象层级                   |

```c++
class CDate{
public:
    CDate(int y,int m,int d){ //自定义的构造函数，编译器不会产生默认构造函数了
        _year = y;
        _month = m;
        _day = d;
    }
    void show(){
        cout<<_year<<_month<<_day<<endl;
    }
private:
    int _year;
    int _month;
    int _day;
};

class CGoods{
public:
    CGoods(char* n,int a,double p，int y,int m,int d):_data(y,m,d),_amount(a),_price(p){
        strcpy(_name,n);
        _count++;  //记录所有产生的新对象的数量
    }
    static void showCount(){ // static成员方法,编译后没有this指针,因此不知道哪个对象在调用它，因此只能访问静态成员变量
        cout<<_count<<endl;
    }
    void show() const{  // 常对象只能调用常方法，编译后为const CGoods* this，接收常对象地址
        cout<<_name<<endl;
    }
private: 
    char _name[20];
    int _amount = 0;
    double _price = 0.0;
    CDate _date{2006,1,1};
    static int _count;  // 声明，不属于对象，属于类本身
}
int CGoods::_count = 0; // static 成员一定要在类外定义并进行初始化

int main(){
    CGoods g1("111",100,35.0,2019,5,12);
    CGoods g2("222",100,35.0,2020,5,12);
    CGoods g3("333",100,35.0,2021,5,12);
    CGoods g4("444",100,35.0,2022,5,12);
    CGoods g5("555",100,35.0,2023,5,12);
    
    g5.showCount();
    CGoods::showCount();
    
    const CGoods g6("555",100,35.0,2023,5,12);;
    g6.show();
}
```



## 第四章  C++模板

### 1.函数模板和类模板

**==函数模板==**

==**注意：** 模板代码通常必须放在**头文件**中，因为编译器在每个调用点都需要看到模板的完整定义才能生成对应的函数代码。==

```c++
template <typename T>
bool compare(T a,T b){ // compare是一个函数模板
    cout<<"template_compare"<<endl;
    return a>b;
}
template<> // 模板的完全特例化，针对compare函数模板，提供const char*类型的特例化版本
bool compare<const char*>(const char* a,const char* b){
    cout<<"compare<const char*>"<<endl;
    return strcmp(a,b)>0;
}
bool compare(const char* a,const char* b){
    cout<<"普通函数"<<endl;
    return strcmp(a,b)>0;
}
/*
在函数调用点，编译器根据用户指定的类型，从原模板实例化一份函数代码出来，叫做模板函数，并进行编译
bool compare<int>(int a,int b){
	return a>b;
}
bool compare<double>(double a,double b){
	return a>b;
}
*/
int main(){
    // 模板名+参数列表=函数名
    compare<int>(10,20);
    compare<double>(10.5,20.5);
    compare(20,30); // 实参推演，参数类型一样可以，类型不同(int,double)报错
    
    // 函数模板实参的推演T const char*
    // 对于某些类型来说，依赖编译器默认实例化的模板代码，代码处理逻辑错误❌
    compare<const char*>("aaa","bbb");// 使用的特例化版本
	compare("aaa","bbb");  // 使用的是普通函数
    
    return 0;
}
```



**==模板、完全特例化、部分特例化==**

```c++
template<typename T>
class Vector{
public:
    Vector() { cout<<"call Vector template"<<endl;}
};

template<>
class Vector<char*>{//完全特例化
public:
    Vector() { cout<<"call Vector<char*>"<<endl;}
};

template<typename T>//部分特例化
class Vector<T*>{
public:
    Vector() { cout<<"call Vector<T*>"<<endl;}
};
//有完全特例化就匹配完全特例化，有部分特例化就匹配部分特例化，没有就从模板实例化
//函数模板：只支持全特例化。如果你需要类似“偏特例化”的效果，通常直接使用函数重载。
int main(){
    Vector<int> vec1;
    Vector<char*> vec2;
    Vector<int*> vec3;
    
    return 0;
}
```



**==模板的非类型参数==**

```c++
template<typename T,int SIZE>
void sort(T *arr){
    for(int i = -;i < SIZE-1;i++){
        for(int j = 0;j < SIZE-1-i;j++){
            if(arr[j] > arr[j+1]){
                T temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

int main(){
    int arr[] = {12,5,7,89,32,21,35}; 
    const int size = sizeof(arr)/sizeof(arr[0]);
    sort<int,size>(arr,size);
    for(int val:arr){
        cout<<val<<endl;
    }
    
    return 0;
}
```



**==类模板==**

```c++
template<typename T>
class SeqStack{ //模板名称+类型参数列表 = 类名称
public:
    SeqStack<T>(int size = 10):_pstack(new T[size]),_top(0),_size(size){}
    SeqStack<T>(const SeqStack<T> &stack):_top(stack._top),_size(stack._size){
        _pstack = new T[_size];
        for(int i = 0;i < _top;i++){
            _pstack[i] = stack._pstack[i];
        }
    }
    SeqStack<T>& operator=(const SeqStack<T> &stack){
        if(this == &stack){
            return *this;
        }
        delete [] _pstack;
        _top = stack._top;
        _size = stack._size;
        _pstack = new T[_size];
        for(int i = 0;i < _top;i++){
            _pstack[i] = stack._pstack[i];
        }
        return *this;
    }
    ~SeqStack<T>(){
        delete [] _pstack;
        _pstack = nullptr;
    }
    
    void push(const T &val);
    void pop(){
        if(empty()){
            return;
        }
        _top--;
    }
    T top() const{
        if(empty()){
            throw "stack is empty"; //抛异常也代表返回
        }
        return _pstack[_top-1];
    }
    bool full() const{
        return _top == _size;
    }
    bool empty() const{
        return _top == 0;
    }
private:
    T *_pstack;
    int _top;
    int _size;
    
    void expand(){
        T *ptmp = new T[_size*2];
        for(int i = 0;i < _top;i++){
            ptmp[i] = _pstack[i];
        }
        delete []_pstack;
        _pstack = ptmp;
        _size = _size*2;
    }
};
template<typename T>  // 每个类外定义的函数头部都要带上template <typename T>
void SeqStack<T>::push(const T &val){
        if(full()){
            expand();
        }
        _pstack[_top++] = val;
    }

int main(){
    SeqStack<int> s1;
	s1.push(10);
    s1.push(20);
        
    cout << "Top: " << s1.top() << endl;
        
    s1.pop();
    s1.pop();    
    
    return 0;
}
```



### 2.vector容器模板实现

```c++
template<typename T>
class vector{
public:
    vector<T>(int size = 10){
        _first = new T[size]; // 开辟空间，调用构造函数
        _last = _first;
        _end = _first + size;
    }
    vector(const vector<T> &rhs){
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
        _first = new T[size];
        for(int i = 0;i < len;i++){
            _first[i] = rhs._first[i];
        }
        _last = _first + len;
        _end = _first + size;
    }
    vector<T>& opertor=(const vector<T>& rhs){
        if(this == &rhs){
            return *this;
        }
        
        delete [] _first;
        
        int size = rhs._end - rhs._first; 
        int len = rhs._last - rhs._first;
        _first = new T[size];
        for(int i = 0;i < len;i++){
            _first[i] = rhs._first[i];
        }
        _last = _first + len;
        _end = _first + size;
        return *this;
    }
    ~vector<T>(){
        delete [] _first;
        _first = _last = _end = nullptr;
    }
    void push_back(const T& val){
        if(full()){
            expand();
        }
        *_last = val;
        _last++;
    }
    void pop_back(){
        if(empty()){
            return;
        }
        _last--;
    }
    T back() const{
        return *(_last-1);
    }
    bool full() const{
        return _last == _end;
    }
    bool empty() const{
        return _first == _last;
    }
    int size() const{
        return _last - _first;
    }
private:
    T* _first; // 指向数组起始位置
    T* _last; // 指向数组有效元素的后继位置
    T* _end; // 指向数组空间的后继位置
    
    void expand(){
        int size = _end - _first;
        T* ptmp = new T[size*2];
        for(int i = 0;i < size;i++){
            ptmp[i] = _first[i];
        }
        delete [] _first;
        _first = ptmp;
        _last = _first+size;
        _end = _first+size*2;
    }
};

int main(){
    vector<int> vec;
    for(int i = 0;i < 20;i++){
        vec.push_back(rand()%100);
    }
    
    while(!vec.empty()){
        cout<<vec.back()<<endl;
        vec.pop_back();
    }
    
    return 0;
}



```

==目前的 **vector 使用 new T[size]**。这会产生一个很严重的问题：**对象构造的浪费**。==

假设 **T** 是一个复杂的类（比如 **class Use**r），当你执行 **vector<User> vec(100);** 时：

①它会立刻调用 **100 次** **User** 的构造函数。

②当你 **push_back** 时，又会执行一次**赋值运算符**。

③当你 **pop_back** 时，对象其实还留在内存里没析构，直到整个 **vector** 销毁。

==**真正的 std::vector 流程：**==

①**==内存分配==**：只分配原始字节流，不调用构造函数。

②**==添加元素==**：在 _last 指向的原始内存上，使用 **定位 new (placement new)** 构造对象。

③**==删除元素==**：显式调用对象的析构函数，但**不释放内存**，这样下次 push_back 就不需要重新申请空间了。



### 3.容器空间配置器allocator

空间配置器做四件事情：内存开辟、内存释放、对象构造、对象析构

```c++
template<typename T>
class Allocator{
public:
    T* allocate(size_t size){ // 负责开辟内存
        return (T*)malloc(sizeof(T)*size);
    }
    void deallocate(void* p){ // 负责释放内存
        free(p);
    }
    void construct(T *p,const T &val){  // 负责对象构造
        new (p) T(val);// 定位new
    }
    void destroy(T *p){  // 负责对象析构
        p->~T(); //~T()代表了T类型的析构函数
    }
};

class Test{
public:
    Test(){
        cout<<"test构造函数"<<endl;
    }
    ~Test(){
        cout<<"Test析构函数"<<endl;
    }
}

                           // 类名=模板名+参数列表
template<typename T,typename Alloc = Allocator<T>>
class vector{
public:
    vector<T>(int size = 10){
        //_first = new T[size]; // 开辟空间，调用构造函数
        _first = _allocator.allocate(size);
        _last = _first;
        _end = _first + size;
    }
    vector(const vector<T> &rhs){
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
       // _first = new T[size];
        _first = _allocator.allocate(size);
        for(int i = 0;i < len;i++){
            //_first[i] = rhs._first[i];
            _allocator.construct(_first+i,rhs._first[i]);
        }
        _last = _first + len;
        _end = _first + size;
    }
    vector<T>& operator=(const vector<T>& rhs){
        if(this == &rhs){
            return *this;
        }
        
        //delete [] _first;
         for(T *p = _first;p != _last;p++){
            _allocator.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
        }
        _allocator.deallocate(_first); //释放堆上的数字内存
        
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
       // _first = new T[size];
        _first = _allocator.allocate(size);
        for(int i = 0;i < len;i++){
            //_first[i] = rhs._first[i];
            _allocator.construct(_first+i,rhs._first[i]);
        }
        _last = _first + len;
        _end = _first + size;
        return *this;
    }
    ~vector<T>(){
        //delete [] _first;
        for(T *p = _first;p != _last;p++){
            _allocator.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
        }
        _allocator.deallocate(_first); //释放堆上的数字内存
        _first = _last = _end = nullptr;
    }
    void push_back(const T& val){
        if(full()){
            expand();
        }
        //*_last = val;
        //_last++;
        _allocator.construct(_last,val); //_last指针指向的内存构造一个值为val的函数
        _last++;
    }
    void pop_back(){
        if(empty()){
            return;
        }
        _last--;//不仅要把_last指针--,还需要析构删除的元素
        _allocator.destory(_last);
    }
    T back() const{
        return *(_last-1);
    }
    bool full() const{
        return _last == _end;
    }
    bool empty() const{
        return _first == _last;
    }
    int size() const{
        return _last - _first;
    }
private:
    T* _first; // 指向数组起始位置
    T* _last; // 指向数组有效元素的后继位置
    T* _end; // 指向数组空间的后继位置
    Alloc _allocator; // 指定容器的空间配置器对象
    
    void expand(){
        int size = _end - _first;
        //T* ptmp = new T[size*2];
        T *ptmp = _allocator.allocate(size*2);
        for(int i = 0;i < size;i++){ 
            //ptmp[i] = _first[i];
            _allocator.construct(ptmp+i,_first[i]);
        }
        //delete [] _first;
        for(T *p = _first;p != _last;p++){
            _allocator.destroy(p);
        }
        _allocator.deallocate(_first);
        _first = ptmp;
        _last = _first+size;
        _end = _first+size*2;
    }
};

int main() {
    Test t1, t2, t3; // 打印 3 次 "test构造函数"
    vector<Test> vec; // 默认分配 10 个 Test 空间的内存，但不构造对象！
    
    vec.push_back(t1); // 打印 1 次拷贝构造（如果写了的话）
    vec.push_back(t2); 
    vec.push_back(t3); 
    
    vec.pop_back(); // 打印 1 次 "Test析构函数" (由 allocator.destroy 触发)
    
    return 0; // 程序结束，vec 析构，剩余 2 个对象被析构，最后 t3,t2,t1 析构
}


```



## 第五章  C++运算符重载

**C++运算符重载：使对象的运算表现的和编译器内置类型一样**

### 1.复数类

```c++
class CComplex{
public:
    CComplex(int r = 0,int i = 0):mreal(r),mimage(i) {}
    CComplex operator+(const CComplex &c){ // 运算符重载，指导编译器怎么做CComplex类对象的加法操作
        CComplex cc;
        cc.mreal = this->mreal + c.mreal;
        cc.mimage = this->mimage + c.mimage;
        return cc;
    }
    CComplex operator++(int){ //后置++，带有一个不起作用的 int 占位符
        CComplex comp = *this;
        mreal = mreal+1;
        mimage = mimage + 1;
        return comp;
    }
    CComplex& operator++(){ // 前置++
        mreal +=1;
        mimage += 1;
        return *this;
    }
    void show(){cout<<mreal<<mimage<<endl;}
private:
    int mreal;
    int mimage;
    friend ostream& operator<<(ostream &out,const CComplex & src);
};
ostream& operator<<(ostream &out,const CComplex & src){// <<左边没有对象，因此不能使用成员方法重载
    out<<src.mreal<<src.mimage<,endl;
    return out;
}

}
int main(){
    CComplex c1(10,10);
    CComplex c2(20,20);
    CComplex c3 = c1+c2;
    c3.show();
    
    CComplex c4 = ++c1; // c1变成(11,11), c4也是(11,11)
    c1.show();
    c4.show();
    
    CComplex c5 = c1++; // c1变成(12,12), c5拿到的是增加前的(11,11)
    c1.show();
    c5.show();
    
    cout<<c1<<endl;
    
}
```



### 2.string类

**实现一个字符串类**

```c++
class String{
public:
    String(const char* p = nullptr){
        if(p != nullptr){
            _pstr = new char[strlen(p)+1];
            strcpy(_pstr,p);
        }else{
            _pstr = new char[1];
            *_pstr = '\0';
        }
    }
    String(const String &str){
        _pstr = new char[strlen(str._pstr)+1];
        strcpy(_pstr,str._pstr);
    }
    String& operator=(const String &str){
        if(this == &str){
            return *this;
        }
        
        delete [] _pstr;
        
        _pstr = new char[strlen(str._pstr)+1];
        strcpy(_pstr,str._pstr);
        return *this;
    }
    ~String(){
        delete [] _pstr;
        _pstr = nullptr;
    }
    bool operator>(const String &str) const{
        return strcmp(_pstr,str._pstr)>0;
    }
    bool operator<(const String &str) const{
        return strcmp(_pstr,str._pstr)<0;
    }
    bool operator==(const String &str) const{
        return strcmp(_pstr,str._pstr)=0;
    }
    int length()const{
        return strlen(_pstr);
    }
    char& operator[](int index){ //可以支持str6[5] = 'i',支持修改
        return _pstr[index];
    }
    const char& operator[](int index) const{ //不允许修改
        return _pstr[index];
    }
    const char* c_str()const{
        return _pstr;
    }
private:
    char* _pstr;
    friend ostream& operator<<(ostream &out,const String &str);
    friend String operator+(const String &lhs,const String& rhs);
}
String operator+(const String &lhs,const String& rhs){
    String tmp;
    tmp._pstr = new char[strlen(lhs._pstr)+strlen(rhs._pstr)+1];
    strcpy(tmp._pstr,lhs._pstr);
    strcat(tmp._pstr,rhs._pstr);
    return tmp;
}
ostream& operator<<(ostream &out,const String &str){
    out<<str._pstr;
    return out;
}

```



### 3.容器迭代器iterator

```c++
class String{
public:
    String(const char* p = nullptr){
        if(p != nullptr){
            _pstr = new char[strlen(p)+1];
            strcpy(_pstr,p);
        }else{
            _pstr = new char[1];
            *_pstr = '\0';
        }
    }
    String(const String &str){
        _pstr = new char[strlen(str._pstr)+1];
        strcpy(_pstr,str._pstr);
    }
    String& operator=(const String &str){
        if(this == &str){
            return *this;
        }
        
        delete [] _pstr;
        
        _pstr = new char[strlen(str._pstr)+1];
        strcpy(_pstr,str._pstr);
        return *this;
    }
    ~String(){
        delete [] _pstr;
        _pstr = nullptr;
    }
    bool operator>(const String &str) const{
        return strcmp(_pstr,str._pstr)>0;
    }
    bool operator<(const String &str) const{
        return strcmp(_pstr,str._pstr)<0;
    }
    bool operator==(const String &str) const{
        return strcmp(_pstr,str._pstr)=0;
    }
    int length()const{
        return strlen(_pstr);
    }
    char& operator[](int index){ //可以支持str6[5] = 'i',支持修改
        return _pstr[index];
    }
    const char& operator[](int index) const{ //不允许修改
        return _pstr[index];
    }
    const char* c_str()const{
        return _pstr;
    }
    class iterator{//给String类提供的迭代器实现
    public:
        iterator(char *p = nullptr):_p(p){}
        bool operator!=(const iterator &it){
            return _p != it._p;
        }
        iterator& operator++(){
            ++_p;
            return *this;
        }
        char& operator*(){
            return *_p;
        }
    private:
        char* _p;
    };
    iterator begin(){// begin返回的是容器底层首元素的迭代器的表示
        return iterator(_pstr);
    }
    iterator end(){// end返回的是容器末尾后继位置的迭代器的表示
        return iterator(_pstr + length());
    }
private:
    char* _pstr;
    friend ostream& operator<<(ostream &out,const String &str);
    friend String operator+(const String &lhs,const String& rhs);
}
String operator+(const String &lhs,const String& rhs){
    String tmp;
    tmp._pstr = new char[strlen(lhs._pstr)+strlen(rhs._pstr)+1];
    strcpy(tmp._pstr,lhs._pstr);
    strcat(tmp._pstr,rhs._pstr);
    return tmp;
}
ostream& operator<<(ostream &out,const String &str){
    out<<str._pstr;
    return out;
}



int main(){
    String str1 = "hello_world";
    String::iterator it = str1.begin();
    for(;it != str1.end();++it){
        cout<<*it<<endl;
    }
    
    // C++11 foreach的方式来遍历容器的内部元素的值，底层还是通过迭代器
    for (char c : str1) {
        cout << c << " ";
    }
    
    return 0;
}
```



### 4.vector的迭代器

```c++
                           // 类名=模板名+参数列表
template<typename T,typename Alloc = Allocator<T>>
class vector{
public:
    vector<T>(int size = 10){
        //_first = new T[size]; // 开辟空间，调用构造函数
        _first = _allocator.allocate(size);
        _last = _first;
        _end = _first + size;
    }
    vector(const vector<T> &rhs){
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
       // _first = new T[size];
        _first = _allocator.allocate(size);
        for(int i = 0;i < len;i++){
            //_first[i] = rhs._first[i];
            _allocator.construct(_first+i,rhs._first[i]);
        }
        _last = _first + len;
        _end = _first + size;
    }
    vector<T>& operator=(const vector<T>& rhs){
        if(this == &rhs){
            return *this;
        }
        
        //delete [] _first;
         for(T *p = _first;p != _last;p++){
            _allocator.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
        }
        _allocator.deallocate(_first); //释放堆上的数字内存
        
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
       // _first = new T[size];
        _first = _allocator.allocate(size);
        for(int i = 0;i < len;i++){
            //_first[i] = rhs._first[i];
            _allocator.construct(_first+i,rhs._first[i]);
        }
        _last = _first + len;
        _end = _first + size;
        return *this;
    }
    ~vector<T>(){
        //delete [] _first;
        for(T *p = _first;p != _last;p++){
            _allocator.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
        }
        _allocator.deallocate(_first); //释放堆上的数字内存
        _first = _last = _end = nullptr;
    }
    void push_back(const T& val){
        if(full()){
            expand();
        }
        //*_last = val;
        //_last++;
        _allocator.construct(_last,val); //_last指针指向的内存构造一个值为val的函数
        _last++;
    }
    void pop_back(){
        if(empty()){
            return;
        }
        _last--;//不仅要把_last指针--,还需要析构删除的元素
        _allocator.destory(_last);
    }
    T back() const {return *(_last-1);}
    bool full() const {return _last == _end;}
    bool empty() const {return _first == _last;}
    int size() const {return _last - _first;}
    T& operator[](int index){return _first[index];}
    class iterator{ //迭代器一般实现为容器的嵌套类型
    public:
        iterator(T* ptr = nullptr):_ptr(ptr) {}
        bool operator!=(const iterator &it)const{
            return _ptr!=it._ptr;
        }
        iterator& operator++(){
            _ptr++;
            return *this;
        }
        T& operator*() {return *_ptr;}
    private:
        T* _ptr;
    };
    iterator begin() {return iterator(_first);}
    iterator end() {return iterator(_last);}
private:
    T* _first; // 指向数组起始位置
    T* _last; // 指向数组有效元素的后继位置
    T* _end; // 指向数组空间的后继位置
    Alloc _allocator; // 指定容器的空间配置器对象
    
    void expand(){
        int size = _end - _first;
        //T* ptmp = new T[size*2];
        T *ptmp = _allocator.allocate(size*2);
        for(int i = 0;i < size;i++){ 
            //ptmp[i] = _first[i];
            _allocator.construct(ptmp+i,_first[i]);
        }
        //delete [] _first;
        for(T *p = _first;p != _last;p++){
            _allocator.destroy(p);
        }
        _allocator.deallocate(_first);
        _first = ptmp;
        _last = _first+size;
        _end = _first+size*2;
    }
};

int main() {
	vector<int> vec;
    for(int i = 0;i < 100;i++){
        vec.push_back(rand()%100+1);
    }
    
	for(auto it = vec.begin();it != vec.end();++it){
        cout<<*it<<endl;
    }
    
    return 0; 
}

```



**==迭代器的失效问题==**

==**①插入/删除引起的局部失效**==

==**erase 操作**： **被删除元素位置之后的迭代器全部失效**。==因为 vector 会将后面的元素统一前移，虽然内存地址没变，但迭代器原本指向的“逻辑元素”已经变了，或者指向了越界位置。

==**insert 操作**： **插入点之后的迭代器全部失效。**==因为为了腾出空间，后面的元素会全部向后挪动。

```c++
// ❌ 错误示范
for (auto it = vec.begin(); it != vec.end(); ++it) {
    if (*it % 2 == 0) {
        vec.erase(it); // 执行后 it 失效，下一次 ++it 会崩溃
    }
}
// ✅ erase正确示范
auto it = vec.begin();
while (it != vec.end()) {
    if (*it % 2 == 0) {
        // 更新 it 为删除后的下一个有效位置
        it = vec.erase(it); 
    } else {
        ++it;
    }
}
// ✅ insert正确示范
auto it = vec.begin();
while (it != vec.end()) {
    if (*it % 2 == 0) {
        // 1. 接收返回值：it 现在指向新插入的 0
        it = vec.insert(it, 0); 
        
        // 2. 跳过新插入的 0 和当前的偶数，防止死循环
        it += 2; 
    } else {
        ++it;
    }
}
```

**==②扩容引起的全员失效==**

==当你调用 **push_back** 且容器刚好**满载**时，会触发扩容，此时所有指向旧内存的迭代器全部变成了**野指针**。==

```c++
vector<int>::iterator it = vec.begin();
vec.push_back(100); // 假设触发了扩容
// 此时 it 指向的旧地址已被 deallocate 释放
cout << *it << endl; // ❌ 运行时错误：非法内存访问
```



### 5.operator new 与operator delete

| **申请方式**   | **释放方式**   | **行为结果**    | **底层原因**                        |
| -------------- | -------------- | --------------- | ----------------------------------- |
| `new T`        | `delete`       | ✅ 正常          | 匹配                                |
| `new T[N]`     | `delete[]`     | ✅ 正常          | 匹配，正确读取 Cookie 并析构 $N$ 次 |
| **`new T[N]`** | **`delete`**   | ❌ **崩溃/泄漏** | 只析构 1 次；释放内存的地址偏移错误 |
| **`new T`**    | **`delete[]`** | ❌ **崩溃**      | 错误读取 Cookie 导致内存定位越界    |

当你使用 **new[]** 开辟一个对象数组时，编译器通常会在分配的内存头部多申请 4 或 8 个字节，用来存储**数组的大小，这个区域叫做 ArrayCookie。**

**==①new[] 申请，delete 释放 --> 内存泄漏 + 程序崩溃==**

**==析构次数错误==**：**delete** 只会调用**第一个元素**的析构函数。剩下的 $N-1$ 个对象不会被析构，如果这些对象内部申请了资源，就会造成严重的内存泄漏。 

**==堆管理崩溃（致命）==**：由于 **new[]** 返回给你的指针 **p** 实际上是指向 **Cookie** 之后的地址。而 **delete** 释放内存时，会直接把 **p** 传给底层的物理内存释放函数（如 free）。因为 **p** 并不是这块内存真正的起始地址，操作系统会认为这是一个无效指针，直接导致程序运行时 **Crash（崩溃）**。

**==②new 申请，delete[] 释放 --> 程序直接崩溃==**

当执行 **delete[]** 时，编译器会默认指针 p 的前面几个字节存着数组长度。它会尝试读取 **p - 4 或 p - 8** 位置的数据，并将其解析为一个巨大的整数作为析构次数。编译器会尝试根据这个随机的“巨大次数”去调用析构函数，并最终试图释放一个完全错误的起始内存地址。

```c++
class Test{
public:
    Test(){cout<<"Test()"<<endl;}
    ~Test(){ cout<<"~Test()"<<endl;}
};

int main(){
    // 1. 申请 3 个对象的数组
    // 底层：分配了 sizeof(Test)*3 + Cookie(4字节) 的空间
    // 返回给 p 的地址是跳过 Cookie 后的地址
    Test* p = new Test[3]; 
    cout << "--- 准备 delete ---" << endl;
    // 2. 错误：使用 delete 释放
    // 行为 A：编译器只认为 p 指向一个对象，所以只调用 1 次 ~Test()
    // 行为 B：编译器直接把 p 指向的地址传给 free()
    // 报错：free() 发现这个地址不是它当初分发的起始地址（少了偏移量），直接 Crash
    delete p;
    
    // 3. 申请单个对象
    // 底层：只分配了 sizeof(Test) 空间，没有 Cookie！
    Test* p = new Test(); 
    cout << "--- 准备 delete[] ---" << endl;
    // 4. 错误：使用 delete[] 释放
    // 行为 A：delete[] 会尝试去 p 的左边寻找 Cookie (p-4 或 p-8)
    // 行为 B：读取到那个位置的原始随机二进制数据，假设这个值是 123456
    // 行为 C：编译器会试图在 p 开始的内存上调用 123456 次析构函数（内存越界）
    // 报错：Segfault（段错误）
    delete[] p;
    
   	return 0;
}
```



## 第六章 C++继承和多态

### 1.继承的本质与原理

==**公有继承**==

```c++
class Father {
public:    void work() { cout << "挣钱"; }
protected: void sleep() { cout << "打呼噜"; }
private:   void secret() { cout << "藏私房钱"; }
};

// 公有继承：权限不降级
class Son : public Father {
    void daily() {
        work();  // ✅ 父亲的 public，儿子内部可以访问
        sleep(); // ✅ 父亲的 protected，儿子内部可以访问
        // secret(); // ❌ 父亲的 private，儿子看不见，报错
    }
};

int main() {
    Son s;
    s.work();  // ✅ 路人（外部）可以看见并调用公有成员
    // s.sleep(); // ❌ 路人（外部）看不见保护成员，报错
}
```

**==保护/私有继承==**

```c++
class Engine {
public:    void start() { cout << "引擎启动"; }
protected: void internalCheck() { cout << "自检"; }
};

// 私有继承：天花板变成了 private，所有继承下来的都变私有
class Car : private Engine {
public:
    void drive() {
        start();         // ✅ 内部可以访问（虽然在内部已经变成了 private）
        internalCheck(); // ✅ 内部可以访问
    }
};

int main() {
    Car myCar;
    myCar.drive(); // ✅ 外部只能调用 Car 自己的 public 方法
    // myCar.start(); // ❌ 报错！Engine 的 start 已经变成 Car 的私有成员了
}
```

**==①==基类私有，派生类不可见。**

**==②==继承方式变严格，权限只会缩紧不会放宽。**

**==③==外部（对象）永远只能摸到 public。**



### 2.派生类的构造过程

**==派生类对象构造和析构的过程==**

**==①==派生类调用基类的构造函数，初始化从基类继承过来的成员；**

**==②==调用派生类自己的构造函数，初始化派生类自己特有的成员；**

**==③==调用派生类的析构函数，释放派生类成员占用的外部资源；**

**==④==调用基类的析构函数，释放从基类继承来的成员可能占用的外部资源；**

```c++
class Base{
public:
    Base(int data):ma(data) { cout<<"Base()"<<endl;}
    ~Base() { cout<<"~Base()"<<endl;}
protected:
    int ma;
};

class Derive:public Base{
public:
    Derive(int data):Base(data),mb(data){
        cout<<"Derive()"<<endl;
    }
    ~Derive(){
        cout<<"~Derive()"<<endl;
    }
private:
    int mb;
}

int main(){
    Derive d(20); 
    
    return 0;
}
```



### 3.重载、隐藏、覆盖

| **特性**     | **重载 (Overload)**          | **隐藏 (Hiding / Scope shadowing)**              |
| ------------ | ---------------------------- | ------------------------------------------------ |
| **作用域**   | 必须在**同一个**类或作用域中 | 发生在**基类和派生类**之间                       |
| **函数名**   | 相同                         | 相同                                             |
| **参数列表** | **必须不同**                 | **无论是否相同**，都会发生隐藏                   |
| **触发条件** | 编译器根据参数选择最佳匹配   | **==只要名字撞车，派生类就“掩盖”基类同名成员==** |

如果 **Derive** 中也定义了一个 **int ma;**，那么 **Base::ma** 也会被隐藏。解决办法是显式作用域限定（**Base::**）或 **using** 声明。

```c++
class Base{
public:
    Base(int data = 10):ma(data) {}
    void show() {cout<<"Base::show()"<<endl;}
    void show(int) {cout<<"Base::show(int)"<<endl;}
protected:
    int ma;
}

class Derive:public Base{
public:
    Derive(int data = 20):Base(data),mb(data) {}
    void show() {cout<<"Derive::show()"<<endl;}
private:
    int mb;
};

int main(){
    Derive d;
    
    d.show();
    d.Base::show();
    
    d.show(10); //❌，隐藏了，不能调用
    d.Base::show(10);

    return 0;
}
```

**继承结构，也是从上（基类）到下（派生类）的结构。在继承结构中进行上下的转换，默认只支持==从下到上==的类型的转换。**

```c++
int main(){
    Base b(10);
    Derive d(20);
    // 教师可以转化为人，人不一定可以转化为教师
    b = d;// ✅编译器只会将 d 中属于 Base 的那部分成员变量（ma）拷贝给 b
    
    d = b;// ❌
    
    Base* pb = &d;  //✅
    pb->show();  //Base::show()
    pb->show(10);//Base::show(int)
    
    Derive* pd = &b;//❌ 内存越界
}
```



### 4.静态绑定和动态绑定

==**静态绑定**==

```c++
class Base{
public:
    Base(int data = 10):ma(data) {}
    void show() {cout<<"Base::show()"<<endl;} 
    void show(int) {cout<<"Base::show(int)"<<endl;} 
protected:
    int ma;
};
class Derive:public Base{
public:
    Derive(int data = 20):Base(data),mb(data) {}
    void show() cout<<"Derive::show()"<<endl;
private:
    int mb;
};

int main(){
    Derive d(50);
    Base* pb = &d;
    pb->show(); //静态（编译时期）的绑定（函数的调用）
    pb->show(10);//静态绑定
    
    cout<<sizeof(Base)<<endl;//4
    cout<<sizeof(Derive)<<endl;//8
}
```



==**动态绑定**==

**==①==如果类里面定义了虚函数，那么==编译阶段，编译器会为该类生成一个 vftable 虚函数表==，==虚函数表中主要存储 RTTI（运行时类型信息）指针和虚函数的地址==。当程序运行时，每一张虚函数表都会加载到内存的.rodata区。**

**==②==一个类里面定义了虚函数，那么这个==类定义的对象，其内存中多存储一个 vfptr 虚函数指针，指向对应的虚函数表 vftable==。一个类定义的n个对象，他们的vfptr指向的都是同一张虚函数表。**

**==③==一个类里面虚函数的个数，不影响对象内存大小（vfptr），影响的是虚函数表的大小。**

**==④覆盖/重写（override）==:如果派生类中的方法，和基类中的某个方法的返回值、函数名、参数列表都相同，而且基类的方法是 virtual 虚函数，那么派生类的这个方法自动处理为虚函数。**

**==⑤动态绑定==：当你写下 pb->show() 时，编译器不再直接生成 call Base::show 的汇编代码，而是生成类似下面的逻辑：通过 pb 找到对象的首地址；取出首地址处的 vfptr,根据 vfptr 找到对应的 vftable；从 vftable 的固定索引位置取出函数地址；执行 call 跳转到该地址。 **

```c++
    Base 对象的内存布局
    +----------------+
    |     vfptr      | -----> [ 指向 vftable ]
    +----------------+
    |      ma        |
    +----------------+
```

```c++
     Base vftable虚函数表 (只读数据段)
    +---------------------------+
    |   RTTI (type_info) Pointer|  --> 存储类信息: "Base"
    +---------------------------+
    |    Base::show() Address  |  --> 指向代码段中的函数体
    +---------------------------+
    |   Base::show(int) Address  |  --> 指向代码段中的函数体
    +---------------------------+
    |         NULL / End        |  --> 表结束标志 (某些编译器实现)
    +---------------------------+

//如果 Derive 继承了 Base 并重写（Override）了show()：         
	   Derive vftable虚函数表
    +---------------------------+
    |   RTTI (type_info) Pointer|  --> 存储类信息: "Derive"
    +---------------------------+
    |   Derive::show() Address |  --> [覆盖] 变成了子类函数地址
    +---------------------------+
    |   Base::show(int) Address  |  --> [继承] 依然是父类函数地址
    +---------------------------+ 

// 谁生成表？ 编译器在编译阶段生成。
// 谁初始化指针？ vfptr（虚函数表指针）是在构造函数执行时初始化的。

//存储位置：
// vftable：在只读数据段（.rodata）。
// vfptr：在每个对象的首地址处（如果是多继承，可能会有多个 vfptr）。  
        
```



```c++
class Base{
public:
    Base(int data = 10):ma(data) {}
    virtual void show() {cout<<"Base::show()"<<endl;} //虚函数
    virtual void show(int) {cout<<"Base::show(int)"<<endl;} //虚函数
protected:
    int ma;
};
class Derive:public Base{
public:
    Derive(int data = 20):Base(data),mb(data) {}
    void show() {cout<<"Derive::show()"<<endl;} 
private:
    int mb;
};

int main(){
    Derive d(50);
    Base* pb = &d;
    pb->show(); //Derive::show()
    pb->show(10); //Base::show(int)
    
    cout<<sizeof(Base)<<endl;//8
    cout<<sizeof(Derive)<<endl;//12
}
```



**==哪些函数不能实现为虚函数？==**

虚函数的核心逻辑是：**通过对象内存中的 vfptr（虚函数表指针）找到 vftable（虚函数表），从而实现运行时多态。**

| **函数类型** | **能否为虚函数** | **核心原因**                                           |
| ------------ | ---------------- | ------------------------------------------------------ |
| **构造函数** | ❌ 绝对不可       | 此时 **vfptr** 尚未初始化完成                          |
| **静态函数** | ❌ 不可           | 不属于对象，没有 **this** 指针和 **vfptr**             |
| **析构函数** | ✅ **建议必须**   | 确保通过基类指针释放时能正确调用子类析构，防止内存泄漏 |

**==黄金法则==**：如果一个类会被继承（即包含任何虚函数），那么它的析构函数**必须**声明为 **virtual**。

```c++
class Base {
public:
    Base() { cout << "Base() 构造" << endl; }
    ~Base() { cout << "~Base() 析构 (非虚)" << endl; } // ❌ 隐患：非虚析构
};

class Derive : public Base {
public:
    Derive() : ptr(new int(10)) { cout << "Derive() 构造" << endl; }
    ~Derive() { 
        delete ptr; 
        cout << "~Derive() 析构并释放堆内存" << endl; 
    }
private:
    int* ptr; // 派生类申请了额外的堆资源
};

int main() {
    Base* pb = new Derive(); // 基类指针指向派生类对象
    delete pb;  //❌编译器看到 pb 的类型是 Base*，而 ~Base() 不是虚函数，于是直接“静态绑定”到 ~Base(),~Derive() 根本没有被调用！造成了内存泄漏。
    return 0;
}
//Base() 构造
//Derive() 构造
//~Base() 析构 (非虚)
//--------------------------------------------------------------------------------
class Base {
public:
    Base() { cout << "Base() 构造" << endl; }
    // ✅ 修正：声明为虚析构函数
    virtual ~Base() { cout << "~Base() 析构" << endl; }
};

class Derive : public Base {
public:
    Derive() : ptr(new int(10)) { cout << "Derive() 构造" << endl; }
    ~Derive() override { // 重写析构函数
        delete ptr; 
        cout << "~Derive() 析构并释放堆内存" << endl; 
    }
private:
    int* ptr;
};

int main() {
    Base* pb = new Derive();
    delete pb; 
    return 0;
}
//Base() 构造
//Derive() 构造
//~Derive() 析构并释放堆内存
//~Base() 析构
```



### 5.多态

**==静态多态==**：函数重载、模板（函数模板和类模板）

**==动态多态==：在继承结构中，基类指针（引用）指向派生类对象，通过该指针调用虚函数，基类指针指向哪个派生类对象，就会调用哪个派生类对象的同名覆盖方法，称为多态。**

```c++
class Animal{
public:
    Animal(string name):_name(name) {}
    virtual void bark() {}
protected:
    string _name;
};
class Cat:public Animal{
public:
    Cat(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:miaomiao"<<endl;}
};
class Dog:public Animal{
public:
    Dog(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:wangwang"<<endl;}
};
class Pig:public Animal{
public:
    Pig(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:hengheng"<<endl;}
};

void bark(Animal* a){
    a->bark();
}
int main(){
    Cat c("mimi");
    Dog d("erha");
    Pig p("pigi");
    
    bark(&c);
    bark(&d);
    bark(&p);
    
    return 0;
}
```



### 6.抽象类的设计

一个类中包含**至少一个纯虚函数（在虚函数声明的末尾加上 = 0）**，这个类就被称为**抽象类**。抽象类不能实例化对象，强制派生类重写，否则派生类也会变成抽象类，但是可以定义指针和引用。

```c++
class Animal{
public:
    Animal(string name):_name(name) {}
    virtual void bark() = 0; //纯虚函数，拥有纯虚函数的类为抽象类，抽象类不能实例化对象
protected:
    string _name;
};
class Cat:public Animal{
public:
    Cat(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:miaomiao"<<endl;}
};
class Dog:public Animal{
public:
    Dog(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:wangwang"<<endl;}
};
class Pig:public Animal{
public:
    Pig(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:hengheng"<<endl;}
};

void bark(Animal* a){
    a->bark();
}
int main(){
    Cat c("mimi");
    Dog d("erha");
    Pig p("pigi");
    
    bark(&c);
    bark(&d);
    bark(&p);
    
    return 0;
}

```

==**一些笔试题**==

```c++
class Animal{
public:
    Animal(string name):_name(name){}
    virtual void bark() = 0;
protected:
    string _name;
};
class Cat:public Animal{
public:
    Cat(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:miaomiao"<<endl;}
};
class Dog:public Animal{
public:
    Dog(string name):Animal(name) {}
    void bark() {cout<<_name<<"bark:wangwang"<<endl;}
};

int main(){
    Animal* p1 = new Cat("加菲猫");//对象 p1和 p2的内存首地址处都存储着一个指针——vfptr。
    Animal* p2 = new Dog("二哈");
    
    int* p11 = (int*)p1;//把对象指针转成了整型指针，这样就可以通过下标 [0] 访问对象内存的前 4（或 8）个字节
    int* p22 = (int*)p2;
    int tmp = p11[0];
    p11[0] = p22[0];
    p22[0] = tmp;
    
    p1->bark();
    p2->bark();
    
    delete p1;
    delete p2;
    return 0;
}
//加菲猫bark:wangwang
//二哈bark:miaomiao
```

```c++
class Base{
public:
    virtual void show(){
        cout<<"Base::show"<<endl;
    }
};
class Derive:public Base{
private:
    void show(){
        cout<<"Derive::show"<<endl;
    }
};

int main(){
    Base* p = new Derive();
    //编译期的权限检查：编译器在处理 p->show() 时，首先看指针p的类型。p 是 Base*，而 Base::show是public的。因此，编译器认为这次调用是合法的，权限检查通过。
    p->show();//运行期的动态绑定：由于 show 是虚函数，运行时程序会通过 p 指向的对象（Derive 对象）的 vfptr 去查找虚函数表（vftable）。
    delete p;
    
    return 0;
}
// 输出Derive::show
```

```c++
class Base{
public:
    Base(){
        cout<<"Base()"<<endl;
        clear();
    }
    void clear() {memset(this,0,sizeof(*this));}
    virtual void show(){
        cout<<"Base::show()"<<endl;
    }
};
class Derive:public Base{
public:
    Derive(){
   		cout<<"Derive()"<<endl;
	}
    void show(){
        cout<<"Derive::show()"<<endl;
    }
    
};

int main(){
    Base *pb1 = new Base();
    pb1->show();//❌崩溃，当 new Base() 执行时，编译器首先会初始化 vfptr，使其指向 Base 的虚函数表（vftable），接着进入 Base 构造函数的函数体，执行 cout << "Base()"，memset 的毁灭性打击：你调用了 clear()，里面执行了 memset(this, 0, sizeof(*this))。重点：这行代码会把对象内存的前 4（或 8）个字节也清零。这意味着刚刚初始化好的 vfptr 变成了 0x00000000（NULL）。导致空指针访问错误。
    delete pb1;
    
    Base *pb2 = new Derive();
    pb2->show();//正确✅，输出Derive::show()
    delete pb2;
    
    return 0;
}
```



### 7.虚基类和虚继承

**==抽象类==：有纯虚函数的类-->`virtual void show() = 0;`**

**==虚基类==：virtual 可以修饰继承方式，为虚继承，被虚继承的类，称为虚基类；可以解决菱形继承的命名冲突和数据冗余问题。**

```c++
class A{
public:  
private:
    int ma;
};

class B:virtual public A{
public: 
private:
    int ma;
};

int main(){
    
    return 0;
}
```

```c++
当执行 class B : virtual public A 时，对象 B 的内部结构如下：
      Object B Memory Layout
    +--------------------------+  <-- 对象 B 的起始地址 (this)
    |    vbptr (虚基类表指针)    |  ----[ 指向 vbtable ]
    +--------------------------+
    |    B::ma (B 自己的成员)    |
    +--------------------------+
    |    A::ma (A 的虚基类成员)  |  <-- 虚基类成员被“挪”到了最后
    +--------------------------+
    
vbptr 指向的这张表存储了关键的 偏移量（Offset） 信息，以便在运行时找到共享的基类部分：    
         vbtable (只读数据段)
    +---------------------------+
    |  0 (相对于 vbptr 的偏移)   |  --> 指向 B(this) 的起始位置
    +---------------------------+
    |  8 (相对于 vbptr 的偏移)   |  --> 指向 A::ma 的起始位置
    +---------------------------+
```



### 8.多重继承以及问题

**多重继承：一个派生类继承多个基类**

**虚基类唯一化**：无论有多少个派生类虚继承自 `A`，在最终的派生类 `D` 中，`A` 的部分只会被存储一次。

```c++
class A{
public:
    A(int data):ma(data) {cout<<"A()"<<endl;}
    ~A() {cout<<"~A()"<<endl;}
protected:
    int ma;
};
class B:virtual public A{
public:
   B(int data):A(data),mb(data) {cout<<"B()"<<endl;}
    ~B() {cout<<"~B()"<<endl;}
protected:
    int mb;
};
class C:virtual public A{
public:
   C(int data):A(data),mc(data) {cout<<"C()"<<endl;}
    ~C() {cout<<"~C()"<<endl;}
protected:
    int mc;
};
class D:public B,public C{
public:
   D(int data):A(data),B(data),C(data),md(data) {cout<<"D()"<<endl;}
    ~D() {cout<<"~D()"<<endl;}
protected:
    int md;
};
int main(){
    D d(10);
    
    return 0;
}
```



### 9.C++四种类型强转

| **转换类型**           | **检查时机** | **主要用途**                           | **安全性**             |
| ---------------------- | ------------ | -------------------------------------- | ---------------------- |
| **`static_cast`**      | 编译时       | **基础类型转换、上行转换**             | 中（依赖开发者的逻辑） |
| **`dynamic_cast`**     | 运行时       | **安全的多态下行转换(基类转为派生类)** | 高（失败返回空指针）   |
| **`const_cast`**       | 编译时       | 去掉指针（引用） **const** 属性        | 低（易导致逻辑错误）   |
| **`reinterpret_cast`** | 编译时       | 底层位模式重新解释                     | 极低（非必要不使用）   |

```c++
double d = 3.14;
int i = static_cast<int>(d); // 基础类型转换
void* ptr = &i;
int* pInt = static_cast<int*>(ptr); // void* 转换

Base* pb = new Derived();
Derived* pd = dynamic_cast<Derived*>(pb); 
if (pd) {
    // 转换成功
}

const int constant = 20;
int* modifier = const_cast<int*>(&constant);
// *modifier = 30; // 危险操作！

int* p = new int(65);
char* ch = reinterpret_cast<char*>(p); // 将 int 指针视为 char 指针
```



## 第七章 C++ STL

### 1.顺序容器 vector、deque、list

**==vector==**：向量容器，底层数据结构为**动态开辟的数组**，每次以原来空间的 2 倍大小进行扩容。

| **分类**       | **函数 / 迭代器**           | **功能描述**     | **时间复杂度** | **核心备注**                     |
| -------------- | --------------------------- | ---------------- | -------------- | -------------------------------- |
| **增加操作**   | `push_back(val)`            | 尾部添加元素     | 均摊 $O(1)$    | 最常用，可能触发内存重分配       |
|                | `emplace_back()`            | **原位构造**元素 | 均摊 $O(1)$    | 直接在容器内存构造，减少拷贝     |
|                | `insert()`                  | 任意位置插入     | $O(n)$         | 插入点后的元素需全部向后移动     |
| **删除操作**   | `pop_back()`                | 删除尾部元素     | $O(1)$         | 不释放内存，仅销毁对象           |
|                | `erase()`                   | 删除指定元素     | $O(n)$         | 之后元素向前移，返回下个迭代器   |
|                | `clear()`                   | 清空所有元素     | $O(n)$         | 销毁对象，但保持 `capacity` 不变 |
| **访问操作**   | `operator[]`                | 下标访问         | $O(1)$         | **不检查越界**，追求极致性能     |
|                | `at()`                      | 下标访问         | $O(1)$         | **检查越界**，安全性高（抛异常） |
|                | `front()` / `back()`        | 首尾引用访问     | $O(1)$         | 返回第一个/最后一个元素的引用    |
| **容量管理**   | `size()`                    | 当前元素数量     | $O(1)$         | 实际存储的元素个数               |
|                | `capacity()`                | 总容量大小       | $O(1)$         | 扩容前能容纳的最大元素数         |
|                | `reserve()`                 | **预留空间**     | $O(n)$         | 提前分配内存，避免频繁扩容       |
|                | `shrink_to_fit()`           | 释放多余空间     | $O(n)$         | 使 `capacity` 收缩至等于 `size`  |
| ---            | ---                         | ---              | ---            | ---                              |
| **迭代器类型** | **`begin()` / `end()`**     | **正向迭代器**   | $O(1)$         | 指向首元素 / 末尾的下一个位置    |
| (随机访问)     | **`cbegin()` / `cend()`**   | **常量正向**     | $O(1)$         | **只读**，防止在遍历时修改数据   |
|                | **`rbegin()` / `rend()`**   | **反向迭代器**   | $O(1)$         | 从后往前遍历（`++` 操作向左移）  |
|                | **`crbegin()` / `crend()`** | **常量反向**     | $O(1)$         | 只读的反向遍历，增强代码安全性   |

**==注意==：对容器进行连续插入或者删除操作（insert/erase），一定要更新迭代器，否则第一次insert或者erase完成之后，迭代器就失效了。**

```c++
int main(){
    vector<int> vec;
    for(int i = 0;i < 20;i++){
        vec.push_back(rand()%100+1);
    }
    
    int size = vec.size();
    for(int i = 0;i < size;i++){
        cout<<vec[i]<<endl;
    }
    
    return 0;
}
```



**==deque==**：双端队列容器，底层容器为**分段连续的空间，由中控器和缓冲区**组成。**Map（中控器）**：`deque` 维护一个类似数组的结构，其中每个元素都是一个指针，指向一个真正的缓冲区。**缓冲区（Buffer）**：实际存储数据的内存块。

| **分类**         | **函数名称**                     | **功能描述**                                    | **时间复杂度** |
| ---------------- | -------------------------------- | ----------------------------------------------- | -------------- |
| **双端增删**     | `push_front` / `pop_front`       | 在**头部**插入 / 删除元素->**==首部预留空间==** | **$O(1)$**     |
|                  | `push_back` / `pop_back`         | 在**尾部**插入 / 删除元素                       | **$O(1)$**     |
|                  | `emplace_front` / `emplace_back` | 在头/尾部**原位构造**元素（更高效）             | $O(1)$         |
| **任意位置增删** | `insert(pos, val)`               | 在指定位置插入元素                              | $O(n)$         |
|                  | `erase(pos)`                     | 删除指定位置元素                                | $O(n)$         |
|                  | `clear()`                        | 清空所有元素                                    | $O(n)$         |
| **元素访问**     | **`operator[]` / `at()`**        | **随机访问**（支持下标访问）                    | **$O(1)$**     |
|                  | `front()` / `back()`             | 获取第一个 / 最后一个元素的引用                 | $O(1)$         |
| **容量操作**     | `size()`                         | 返回当前元素个数                                | $O(1)$         |
|                  | `empty()`                        | 判断是否为空                                    | $O(1)$         |
|                  | `resize(n)`                      | 调整元素个数                                    | $O(n)$         |
|                  | `shrink_to_fit()`                | 释放未使用的缓冲区以节约内存                    | $O(n)$         |
| ==**迭代器**==   | **`begin()` / `end()`**          | 返回指向首/尾后继的迭代器                       | $O(1)$         |
|                  | **`rbegin()` / `rend()`**        | 反向迭代器，用于从后往前遍历                    | $O(1)$         |
|                  | **`cbegin()` / `cend()`**        | 常量迭代器，不允许修改指向的内容                | $O(1)$         |



**==list==**：双向链表容器，内存空间非连续。迭代器类型为**双向迭代器**（支持 `++it`, `--it`；不支持 `it + n`, `it < it_end`）

| **分类**         | **函数名称**                   | **功能描述**                                      | **时间复杂度** |
| ---------------- | ------------------------------ | ------------------------------------------------- | -------------- |
| **基础增删**     | `push_front()` / `pop_front()` | 在**头部**插入 / 删除元素                         | $O(1)$         |
|                  | `push_back()` / `pop_back()`   | 在**尾部**插入 / 删除元素                         | $O(1)$         |
|                  | `insert(pos, val)`             | 在迭代器 `pos` **之前**插入元素                   | $O(1)$         |
|                  | `erase(pos)`                   | 删除迭代器 `pos` 指向的元素                       | $O(1)$         |
|                  | `clear()`                      | 清空所有元素                                      | $O(n)$         |
| **元素访问**     | `front()` / `back()`           | 获取第一个 / 最后一个元素的引用                   | $O(1)$         |
|                  | `begin()` / `end()`            | 获取首位 / 末尾后继的迭代器                       | $O(1)$         |
| **链表特有操作** | **`sort()`**                   | 链表内部排序（非全局 `std::sort`）                | $O(n \log n)$  |
|                  | **`reverse()`**                | 反转链表元素顺序                                  | $O(n)$         |
|                  | **`splice()`**                 | **接驳操作**：将另一个 list 的节点移动到当前 list | $O(1)$         |
| **大小查询**     | `size()`                       | 返回元素个数                                      | $O(1)$         |
|                  | `empty()`                      | 判断链表是否为空                                  | $O(1)$         |



**==vector,deque,list对比==**

| **特性**          | **std::vector**             | **std::deque**                   | **std::list**                |
| ----------------- | --------------------------- | -------------------------------- | ---------------------------- |
| **底层结构**      | **单块连续内存** (动态数组) | **分段连续内存** (中控器+缓冲区) | **双向链表** (不连续节点)    |
| **随机访问 `[]`** | **极快 ($O(1)$)**           | 较快 ($O(1)$)                    | **不支持**                   |
| **头部增删**      | 极慢 ($O(n)$)               | **极快 ($O(1)$->首部预留空间)**  | **极快 ($O(1)$)**            |
| **尾部增删**      | **极快 (均摊 $O(1)$)**      | **极快 ($O(1)$)**                | **极快 ($O(1)$)**            |
| **中间增删**      | 较慢 ($O(n)$)               | 较慢 ($O(n)$)                    | **极快 ($O(1)$)** (已知位置) |
| **迭代器类型**    | **随机访问迭代器**          | **随机访问迭代器**               | **双向迭代器**               |

**==在中间进行insert或者erase，deque比vector的效率更高==**：它会计算插入点距离**头部**近，还是距离**尾部**近。如果离**头部**更近，它会移动插入点**之前**的所有元素。如果离**尾部**更近，它会移动插入点**之后**的所有元素。



### 2.适配器 stack、queue、priority_queue

**==容器适配器==是将现有的序列容器（如 `vector`、`deque`、`list`）进行封装，通过限制它们的功能，提供一种特定的数据结构。**

| **适配器**                | **特点 (逻辑结构)**      | **默认底层容器** | **暴露的操作**                                            |
| ------------------------- | ------------------------ | ---------------- | --------------------------------------------------------- |
| **`std::stack`**          | **后进先出 (LIFO)**      | `std::deque`     | `push()`, `pop()`, `top()`,`empty()`,`size()`             |
| **`std::queue`**          | **先进先出 (FIFO)**      | `std::deque`     | `push()`, `pop()`, `front()`, `back()`,`empty()`,`size()` |
| **`std::priority_queue`** | **优先队列 (大/小顶堆)** | `std::vector`    | `push()`, `pop()`,`top()`,`empty()`,`size()`              |

```c++
#include <stack>
#include <vector>
#include <list>


// stack,queue为什么默认使用deque?
// 1️⃣vector 不支持 pop_front（效率极低），而 list 虽然支持所有操作，但内存不连续，性能较差。deque 原生支持头尾两端的高效增删，完美满足 stack 和 queue 的所有功能需求。
// 2️⃣deque 的分段结构比 list（每个节点都要分配内存并存指针）更省内存，比 vector（可能预留大量未使用的连续空间）更灵活。当存储大量数据时，deque内存利用率更高一些。
// priority_queue为什么默认使用vector?
// 1️⃣堆是一颗完全二叉树。在数据结构中，完全二叉树最高效的存储方式就是连续数组。
// 2️⃣在堆的调整过程中，需要频繁地比较和交换父子节点，vector的内存绝对连续， 以上操作效率更高。

// 1. 默认底层 (使用 deque)
std::stack<int> s1; 

// 2. 指定底层为 vector (因为 stack 需要 back(), push_back(), pop_back())
std::stack<int, std::vector<int>> s2;

// 3. 指定底层为 list
std::stack<int, std::list<int>> s3;

// 4. 默认是大顶堆 (less)
std::priority_queue<int> maxHeap; 

// 5. 小顶堆 (greater)
std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
```



### 3.有序关联容器 set/multiset、map/multimap

**multiset**允许存储多个相同的key，**multimap**允许一个key可以对应多个value

**迭代器**：支持**双向迭代器**。对有序容器进行中序遍历（从 `begin()` 到 `end()`），得到的是一个**有序序列**

这些函数对于**四种容器**基本通用：

| **函数**                           | **功能描述**                                         | **复杂度**  |
| ---------------------------------- | ---------------------------------------------------- | ----------- |
| **`insert(val)`**                  | 插入元素                                             | $O(\log n)$ |
| **`erase(key)`**，**`erase(pos)`** | 删除指定**键/位置**的元素                            | $O(\log n)$ |
| **`find(key)`**                    | 查找键是否存在，**返回迭代器**（若无则返回 `end()`） | $O(\log n)$ |
| **`count(key)`**                   | 返回键**出现的次数**（`set/map` 只能是 0 或 1）      | $O(\log n)$ |
| **`lower_bound(k)`**               | 返回第一个 **$\ge k$** 的元素迭代器                  | $O(\log n)$ |
| **`upper_bound(k)`**               | 返回第一个 **$> k$** 的元素迭代器                    | $O(\log n)$ |
| **`equal_range(k)`**               | 返回一对迭代器，表示键等于 $k$ 的区间                | $O(\log n)$ |
| **`size()`**                       | 返回容器中当前**元素的总个数**                       | $O(1)$      |



### 4.无序关联容器 unordered_set/unordered_multiset、unordered_map/unordered_multimap

**无序关联容器**是基于 **哈希表（Hash Table）** 实现的。与有序关联容器（红黑树实现）相比，它们不保证元素的顺序，但在**查找、插入和删除**的平均性能上达到了惊人的 $O(1)$。

| **特性**                 | **有序容器 (map/set)**    | **无序容器 (unordered_map/set)**    |
| ------------------------ | ------------------------- | ----------------------------------- |
| **底层实现**             | 红黑树 (平衡二叉树)       | 哈希表                              |
| **平均查找速度**         | $O(\log n)$               | **$O(1)$**                          |
| **最坏查找速度**         | $O(\log n)$               | $O(n)$ (哈希冲突严重时)             |
| **元素顺序**             | 有序（支持范围查找）      | 乱序                                |
| **内存开销**             | 较低（每个节点 3 个指针） | 较高（需维护桶数组和链表）          |
| ==**自定义Key 的要求**== | 需支持 `operator<`        | 需支持 `operator==` 和 **哈希函数** |



### 5.容器的迭代器

**每个容器都提供了一套标准的接口来获取迭代器**：

| **函数**                    | **对应迭代器类型** | **备注**                                 |
| --------------------------- | ------------------ | ---------------------------------------- |
| **`begin()` / `end()`**     | 正向迭代器         | `end()` 指向最后一个元素的**下一个位置** |
| **`cbegin()` / `cend()`**   | **常量**正向迭代器 | C++11 引入，**只读**，不能修改元素       |
| **`rbegin()` / `rend()`**   | **反向**迭代器     | `++it` 实际上是向左移动                  |
| **`crbegin()` / `crend()`** | 常量反向迭代器     | 只读且反向                               |

==**`rbegin()` **指向容器的**最后一个**元素，**`rend()`**指向容器**第一个元素的前一个**位置。==

| **迭代器等级**     | **对应操作**                                           | **典型容器**                                          |
| ------------------ | ------------------------------------------------------ | ----------------------------------------------------- |
| **前向迭代器**     | `it++`, `*it`, `==`, `!=`                              | **`forward_list, unordered_set`**,**`unordered_map`** |
| **双向迭代器**     | 以上所有 + **`it--`**                                  | **`list`**, **`map`**, **`set`**                      |
| **随机访问迭代器** | 以上所有 + **`it + n`**, **`it[n]`**, **`<`**, **`>`** | **`vector`**, **`deque`**, **`array`**                |



### 6.函数对象

**函数对象**，通常也被称为 **仿函数**。函数对象就是一个**重载了括号运算符 `operator()` 的类对象**。它的行为看起来像函数，但本质上是对象。

```c++
struct MyAdder {
    int operator()(int a, int b) const {
        return a + b;
    }
};

// 使用方法
MyAdder add; 
int sum = add(10, 20); // 像调用函数一样调用对象
```

```c++
// 函数指针实现
template <typename T>
bool mygreater(T a,T b){
    return a>b;
}
template <typename T>
bool myless(T a,T b){
    return a<b;
}

template<typename T,typename Compare> //函数指针
bool compare(T a,T b,Compare comp){
    //通过函数指针调用函数，是没有办法内联的，效率很低，会有函数调用开销
    return comp(a,b);
}

int main(){
    cout<<compare(10,20,mygreater<int>)<<endl;//以函数指针的形式传递参数
    cout<<compare(10,20,myless<int>)<<endl;
}

--------------------------------------------------------------------------------------
// C++函数对象的版本实现
template <typename T>
class mygreater{
public:
    bool operator()(T a,T b){
    return a>b;
	}
};
template <typename T>
class myless{
public:
    bool operator()(T a,T b){
    return a<b;
	}
};
template<typename T,typename Compare> //函数指针
bool compare(T a,T b,Compare comp){
    return comp(a,b);
}

int main(){
    cout<<compare(10,20,mygreater<int>())<<endl;//传入的是对象，不是函数地址,可以内联
    cout<<compare(10,20,myless<int>())<<endl;
}
```

==**函数对象的使用**==

```c++
sort(v.begin(), v.end(), greater<int>());

// 默认 set 是升序的,现在改为降序的
std::set<int, std::greater<int>> s; 
s.insert(10);
s.insert(50);
s.insert(20); // 遍历输出结果：50, 20, 10

// 默认大根堆，现在改为小根堆
priority_queue<int, vector<int>, greater<int>> minHeap;
```

| **场景**             | **所在位置** | **填入的内容**        | **为什么？**                                                 |
| -------------------- | ------------ | --------------------- | ------------------------------------------------------------ |
| **`std::sort(...)`** | **函数参数** | `std::greater<int>()` | 这是一个普通函数调用，它需要一个具体的**实例对象**来执行比较动作。 |
| **`std::set<...>`**  | **模板参数** | `std::greater<int>`   | 这是一个类型声明，告诉编译器这个容器的**内部结构类型**。     |



### 7.泛型算法

1️⃣**==find==：在指定区间内查找等于目标值的第一个元素。**

```c++
vector<int> v = {1, 2, 3, 4, 5};
auto it = find(v.begin(), v.end(), 3);
if (it != v.end()) cout << "找到了：" << *it;
```

2️⃣**==find_if==：查找第一个满足特定条件（谓词/Lambda）的元素。**

```c++
vector<int> v = {1, 2, 3, 4, 5};
// 查找第一个大于 3 的数
auto it = find_if(v.begin(), v.end(), [](int n){ return n > 3; });
```

3️⃣**==count==：统计目标值在区间内出现的次数。**

```c++
vector<int> v = {1, 2, 2, 3, 2};
int n = count(v.begin(), v.end(), 2); // n = 3
```

4️⃣**==replace==：将区间内所有等于 `old_value` 的元素替换为 `new_value`。**

```c++
vector<int> v = {1, 2, 2, 3, 2};
replace(v.begin(), v.end(), 2, 99); // 所有的 2 都变成 99
```

5️⃣**==fill==：将区间内的所有元素统一赋值为目标值。**

```c++
vector<int> v = {1, 2, 2, 3, 2};
fill(v.begin(), v.end(), 0); // 容器所有元素清零
```

6️⃣**==sort==：对区间进行快速排序（非稳定，平均 $O(n \log n)$）。**

```c++
vector<int> v = {1, 2, 2, 3, 2};
sort(v.begin(), v.end()); // 默认升序
```

7️⃣**==reverse==：反转区间内元素的顺序。**

```c++
vector<int> v = {1, 2, 3};
reverse(v.begin(), v.end()); // {1,2,3} -> {3,2,1}
```

8️⃣**==nth_element==：重新排列区间，使第 `n` 个位置存放的是全序列排序后该位置应有的元素。左边都比它小，右边都比它大。**

```c++
vector<int> v = {1, 2, 3, 4, 5};
// 找出中位数放在 v.begin() + v.size()/2 的位置
nth_element(v.begin(), v.begin() + v.size()/2, v.end());
```

9️⃣**==remove==：将不等于目标值的元素移到前面，返回指向“逻辑结尾”的迭代器。**

```c++
vector<int> v = {1, 2, 2, 3, 2};
// 真正删除所有 2
v.erase(remove(v.begin(), v.end(), 2), v.end());
```

🔟**==unique==：剔除相邻的重复元素（通常配合 `sort` 使用）。**

```c++
vector<int> v = {1, 2, 2, 3, 2};
sort(v.begin(), v.end()); // 必须先排序，使重复元素相邻

//unique 的返回值是一个迭代器，它指向“去重”后序列中最后一个不重复元素的下一个位置（即逻辑上的 end）
v.erase(unique(v.begin(), v.end()), v.end());//
```

1️⃣1️⃣**==accumulate==：计算区间的累加和（也可以自定义累积规则）。**

```c++
vector<int> v = {1, 2, 3};
int sum = accumulate(v.begin(), v.end(), 0); // 0 为初始值
```





## 第八章 对象的优化

==**对象的应用优化**==

```c++
class Test{
public:
    Test(int a = 10):ma(a) {cout<<"Test()"<<endl;}
    ~Test(){cout<<"~Test()"<<endl;}
    Test(const Test&t):ma(t.ma) {cout<<"Test(const TEst&)"<<endl;}
    Test& operator=(const Test&t){
        cout<<"operator="<<endl;
        ma = t.ma;
        return *this;
    }
private:
    int ma;   
};

inr main(){
    Test t1; //构造函数
    Test t2(t1);//拷贝构造函数
    Test t3 = t1;//拷贝构造函数
    
    //Test(20)显式生成临时对象，生存周期：所在的语句
    //C++编译器对于对象构造的优化：用临时对象生成新对象的时候，临时对象就不产生了，直接构造新对象就可以了；但临时对象给另外一个对象赋值的时候就必须生成临时对象了
    Test t4 = Test(20);//与Test t4(20)没有区别
    
    t4 = Test(30);//临时对象必须生成
    t4 = (Test)30;//同上，int->Test
    t4 = 30;//同上，隐式生成临时对象，int->Test(int)
    
    Test* p = &Test(40);//出了该行语句指针p指向的是一个已经析构的临时对象，变成悬挂指针，后续使用p->ma会导致未定义行为
    const Test &ref = Test(50)//出了该行语句，临时对象不析构，ref.ma仍然可以使用；临时对象是一个右值，不能使用左值引用，只能使用右值引用或者常左值引用
    
    
    return 0;
}
```

==**实例2**==

```c++
class Test{
public:
    Test(int a = 5,int b = 5):ma(a),mb(b){
        cout<<"Test(int,int)"<<endl;
    }
    ~Test(){
        cout<<"~Test()"<<endl;
    }
    Test(const Test &src):ma(src.ma),mb(src.mb){
        cout<<"Test(const Test&)"<<endl;
    }
    void operator=(const Test &src){
        ma = src.ma;
        mb = src.mb;
        cout<<"operator="<<endl;
    }
private:
    int ma;
    int mb;
};
Test t1(10,10); //1.Test(int,int),全局对象：在 main 函数执行前构造，程序结束时析构

int main(){
    Test t2(20,20); //3.Test(int,int)
    Test t3 = t2; //4.Test(const Test&)
    static Test t4 = Test(30,30); //5.Test(int,int),无临时对象生成
    t2 = Test(40,40); //6.Test(int,int)--operator=--~Test()
    t2 = (Test)(50,50); //7.等同于(Test)50,Test(int,int)--operator=--~Test()
    t2 = 60; //8.Test(int,int)--operator=--~Test()
    Test *p1 = new Test(70,70); //9.Test(int,int),堆上的临时对象不会立即释放
    Test *p2 = new Test[2]; //10.Test(int,int)--Test(int,int)
    Test *p3 = &Test(80,80); //11.Test(int,int)--~Test()
    const Test &p4 = Test(90,90); //12.Test(int,int)
    delete p1; //13.~Test()
    delete [] p2; //14.~Test()--~Test()
    
    return 0;
    //退出 main 后析构顺序
    //p4~Test()--t3~Test()--t2~Test()--t4~Test()--t5~Test()--t1~Test()
}
Test t5(100,100);//2.Test(int,int)
```

==**实例3**==

```c++
class Test{
public:
    Test(int data = 10):ma(data){
        cout<<"Test(int)"<<endl;
    }
    ~Test(){
        cout<<"~Test()"<<endl;
    }
    Test(const Test &t):ma(t.ma){
        cout<<"Test(const Test&)"<<endl;
    }
    void operator=(const Test& t){
        cout<<"operator="<<endl;
        ma = t.ma;
    }
    int getData()const {return ma;}
private:
    int ma;
};

Test GetObject(Test t){//3.Test(const Test&)
    int val = t.getData();
    Test tmp(val);//4.Test(int)
    return tmp;
}

int main(){
    Test t1;//1.Test(10)
    Test t2;//2.Test(10)
    t2 = GetObject(t1);//5.Test(const Test&)->tmp为外部临时对象（存在main函数栈帧中）拷贝构造
    				   //6.~Test()->tmp析构
    				   //7.~Test()形参t的析构
    				   //8.operator=->临时对象对t2的赋值运算
    				   //9.~Test()->临时对象析构
    
    return 0;//10.~Test()->t2析构
    		 //11.~Test()->t1析构
}
```

**==1.函数参数传递过程中，对象优先按引用传递==，不要按值传递（消灭了形参 t 的拷贝构造函数与析构函数，11->9）。**

**==2.返回临时对象 return Test(val);==临时对象不能出GetObject函数作用域，因此意图在main函数栈帧拷贝构造另一个临时对象（用一个临时对象拷贝构造一个新对象，其实不会生成临时对象，而是在新对象处直接调用构造函数，节省了tmp的构造函数与析构函数，9->7）。**

**==3.优先按初始化的方式接收，不要按赋值的方式接收。==Test t2 = GetObject(t1);意图用在main函数栈帧中生成的临时对象拷贝构造t2，那么不会生成任何临时对象，会直接构造t2，7->4。**

**==实例4==**

```c++
class CMyString{
public:
    CMyString(const char *str = nullptr){
        cout<<"CMyString(const char*)"<<endl;
        if(str != nullptr){
            mptr = new char[strlen(str)+1];
            strcpy(mptr,str);
        }else{
            mptr = new char[1];
            *mptr = '\0';
        }
    }
    ~CMyString(){
        cout<<"~CMyString()"<<endl;
        delete [] mptr;
        mptr = nullptr;
    }
    CMyString(const CMyString &str){
        cout<<"CMyString(const CMyString&)"<<endl;
        mptr = new char[strlen(str.mptr)+1];
        strcpy(mptr,str.mptr);
    }
    CMyString& operator=(const CMyString &str){
        cout<<"operator=(const CMyString&)"<<endl;
        if(this == &str){
            return *this;
        }
        delete [] mptr;
        mptr = new char[strlen(str.mptr)+1];
        strcpy(mptr,str.mptr);
        return *this;
    }
    const char* c_str()const {return mptr;}
private:
    char *mptr;
};

CMyString GetString(CMyString &str){
    cosnt char* pstr = str.c_str();
    CMyString tmpStr(pstr);
    return tmpStr;
}

int main(){
    CMyString str1("aaaaaa");
    CMyString str2;
    str2 = GetString(str1);
    cout<<str2.c_str()<<endl;
    
    return 0;
}
/* 输出为下：
CMyString(const char*)
CMyString(const char*)
CMyString(const char*)
CMyString(const CMyString&)-->tmpStr拷贝构造main函数栈帧上的临时对象
~CMyString()
operator=(const CMyString&)-->main函数栈帧上的临时对象给t2赋值
~CMyString()
aaaaa
~CMyString()
~CMyString()
*/
```

**优化方法：带右值引用参数的拷贝构造函数和赋值运算符**

```c++
class CMyString{
public:
    CMyString(const char *str = nullptr){
        cout<<"CMyString(const char*)"<<endl;
        if(str != nullptr){
            mptr = new char[strlen(str)+1];
            strcpy(mptr,str);
        }else{
            mptr = new char[1];
            *mptr = '\0';
        }
    }
    ~CMyString(){
        cout<<"~CMyString()"<<endl;
        delete [] mptr;
        mptr = nullptr;
    }
    CMyString(const CMyString &str){
        cout<<"CMyString(const CMyString&)"<<endl;
        mptr = new char[strlen(str.mptr)+1];
        strcpy(mptr,str.mptr);
    }
    CMyString(CMyString &&str){//带右值引用参数的拷贝构造函数，临时对象匹配到此
        cout<<"CMyString(const CMyString&&)"<<endl;
        mptr= str.mptr;
        str.mptr = nullptr;
    }
    CMyString& operator=(const CMyString &str){
        cout<<"operator=(const CMyString&)"<<endl;
        if(this == &str){
            return *this;
        }
        delete [] mptr;
        mptr = new char[strlen(str.mptr)+1];
        strcpy(mptr,str.mptr);
        return *this;
    }
    CMyString& operator=(CMyString &&str){//带右值引用参数的赋值运算符，临时对象的赋值拷贝到此
        cout<<"operator=(CMyString&&)"<<endl;
        if(this == &str){
            return *this;
        }
        delete [] mptr;
        mptr = str.mptr;
        str.mptr = nullptr;
        return *this;
    }
    const char* c_str()const {return mptr;}
private:
    char *mptr;
};

CMyString GetString(CMyString &str){
    const char* pstr = str.c_str();
    CMyString tmpStr(pstr);
    return tmpStr;//在 return 语句中，如果返回的是一个局部自动对象，编译器必须优先尝试将其作为右值处理。自动触发 std::move 的语义
}

int main(){
    CMyString str1("aaaaaa");
    CMyString str2;
    str2 = GetString(str1);
    cout<<str2.c_str()<<endl;
    
    return 0;
}
/* 输出为下：
CMyString(const char*)
CMyString(const char*)
CMyString(const char*)
CMyString(const CMyString&&)-->移动构造函数，tmpStr移动构造main函数栈帧上的临时对象
~CMyString()
operator=(const CMyString&&)-->移动赋值运算符，main函数栈帧上的临时对象给t2赋值
~CMyString()
aaaaa
~CMyString()
~CMyString()
*/
```

**==move移动语义和forward类型完美转发==**

```c++
class CMyString{
public:
    CMyString(const char *str = nullptr){
        cout<<"CMyString(const char*)"<<endl;
        if(str != nullptr){
            mptr = new char[strlen(str)+1];
            strcpy(mptr,str);
        }else{
            mptr = new char[1];
            *mptr = '\0';
        }
    }
    ~CMyString(){
        cout<<"~CMyString()"<<endl;
        delete [] mptr;
        mptr = nullptr;
    }
    CMyString(const CMyString &str){
        cout<<"CMyString(const CMyString&)"<<endl;
        mptr = new char[strlen(str.mptr)+1];
        strcpy(mptr,str.mptr);
    }
    CMyString(CMyString &&str){//带右值引用参数的拷贝构造函数，临时对象匹配到此
        cout<<"CMyString(const CMyString&&)"<<endl;
        mptr= str.mptr;
        str.mptr = nullptr;
    }
    CMyString& operator=(const CMyString &str){
        cout<<"operator=(const CMyString&)"<<endl;
        if(this == &str){
            return *this;
        }
        delete [] mptr;
        mptr = new char[strlen(str.mptr)+1];
        strcpy(mptr,str.mptr);
        return *this;
    }
    CMyString& operator=(CMyString &&str){//带右值引用参数的赋值运算符，临时对象的赋值拷贝到此
        cout<<"operator=(CMyString&&)"<<endl;
        if(this == &str){
            return *this;
        }
        delete [] mptr;
        mptr = str.mptr;
        str.mptr = nullptr;
        return *this;
    }
    const char* c_str()const {return mptr;}
private:
    char *mptr;
};

CMyString GetString(CMyString &str){
    const char* pstr = str.c_str();
    CMyString tmpStr(pstr);
    return tmpStr;//在 return 语句中，如果返回的是一个局部自动对象，编译器必须优先尝试将其作为右值处理。自动触发 std::move 的语义
}


template<typename T>
class Allocator{
public:
    T* allocate(size_t size){ // 负责开辟内存
        return (T*)malloc(sizeof(T)*size);
    }
    void deallocate(void* p){ // 负责释放内存
        free(p);
    }
------------------------------------------------------------------------------------
    void construct(T *p,const T &val){  // 负责对象构造
        new (p) T(val);// 定位new
    }
    void construct(T *p,const T &&val){  // 负责对象构造
        new (p) T(std::move(val);// 强制转换
    }
------------------------------------------------------------------------------------
    void destroy(T *p){  // 负责对象析构
        p->~T(); //~T()代表了T类型的析构函数
    }
};

                           // 类名=模板名+参数列表
template<typename T,typename Alloc = Allocator<T>>
class vector{
public:
    vector<T>(int size = 10){
        _first = _allocator.allocate(size);
        _last = _first;
        _end = _first + size;
    }
    vector(const vector<T> &rhs){
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
        _first = _allocator.allocate(size);
        for(int i = 0;i < len;i++){
            _allocator.construct(_first+i,rhs._first[i]);
        }
        _last = _first + len;
        _end = _first + size;
    }
    vector<T>& operator=(const vector<T>& rhs){
        if(this == &rhs){
            return *this;
        }
         for(T *p = _first;p != _last;p++){
            _allocator.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
        }
        _allocator.deallocate(_first); //释放堆上的数字内存
        
        int size = rhs._end - rhs._first; //rhs的空间大小
        int len = rhs._last - rhs._first;
        _first = _allocator.allocate(size);
        for(int i = 0;i < len;i++){
            _allocator.construct(_first+i,rhs._first[i]);
        }
        _last = _first + len;
        _end = _first + size;
        return *this;
    }
    ~vector<T>(){
        for(T *p = _first;p != _last;p++){
            _allocator.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
        }
        _allocator.deallocate(_first); //释放堆上的数字内存
        _first = _last = _end = nullptr;
    }
-----------------------------------------------------------------------------------
/*    void push_back(const T& val){//接收左值
        if(full()){
            expand();
        }
        _allocator.construct(_last,val); //_last指针指向的内存构造一个值为val的函数
        _last++;
    }
    void push_back(T &&val){//接收右值，右值引用变量本身是左值，val为左值
        if(full()){
            expand();
        }
        _allocator.construct(_last,std::move(val));//move将左值强制转换为右值
        _last++;
    }*/
    template<typename Ty>
    //传入左值,Ty推导为CMyString&,Ty&&变为CMyString& &&,引用折叠为CMyString&,接收左值
    //传入右值，Ty推导为CMyString&&，Ty&&变为CMyString&& &&，引用折叠为CMyString&&，接收右值
    //但是不管接收左值还是接收右值,val都是一个左值,需要std::forward<Ty>(val) 根据Ty的推导结果来进行完美转发,决定是把val作左值还是右值传递下去。
    void push_back(Ty &&val){
    
    	if(full()){
            expand();
        }
        _allocator.construct(_last,std::forward<Ty>(val);//完美转发
        _last++;
	}
----------------------------------------------------------------------------------
    void pop_back(){
        if(empty()){
            return;
        }
        _last--;//不仅要把_last指针--,还需要析构删除的元素
        _allocator.destory(_last);
    }
    T back() const{
        return *(_last-1);
    }
    bool full() const{
        return _last == _end;
    }
    bool empty() const{
        return _first == _last;
    }
    int size() const{
        return _last - _first;
    }
private:
    T* _first; // 指向数组起始位置
    T* _last; // 指向数组有效元素的后继位置
    T* _end; // 指向数组空间的后继位置
    Alloc _allocator; // 指定容器的空间配置器对象
    
    void expand(){
        int size = _end - _first;
        T *ptmp = _allocator.allocate(size*2);
        for(int i = 0;i < size;i++){ 
            _allocator.construct(ptmp+i,_first[i]);
        }
        for(T *p = _first;p != _last;p++){
            _allocator.destroy(p);
        }
        _allocator.deallocate(_first);
        _first = ptmp;
        _last = _first+size;
        _end = _first+size*2;
    }
};

int main() {
    CMyString str1 = "aaa";
    vector<CMyString> vec;
    cout<<"-----------------------------------"<<endl;
    vec.push_back(str1);//传左值
    vec.push_back(CMyString("bbb"));//传右值  move,forward
    cout<<"___________________________________"<<endl;
}
// move底层：右值引用类型强转
```



## 第九章 智能指针

**==智能指针==：保证能实现资源的自动释放,==利用栈上的对象出作用域自动析构的特点==，来做到资源的自动释放。**

**自己实现智能指针**

```c++
template <typename T>
class CSmartPtr{
public:
    CSmartPtr(T *ptr=nullptr):mptr(ptr) {}
    ~CSmartPtr() {delete mptr;}
    T& operator*() {return *mptr;}
    T* operator->() {return mptr;}
private:
    T* mptr;
};

int main(){
    CSmartPtr<int>ptr1(new int);
    *ptr1 = 20;
    
    class Test{
    public:
        void test() {cout<<"Test::test()"<<endl;}
    };
    CSmartPtr<Test> ptr2(new Test());
    ptr2->test();//解析为 (ptr2.operator->())->test()
    
    CSmartPtr<Test> ptr3(ptr2);//错误❌，浅拷贝导致内存重复释放
    
    return 0;
}
```



**==不带引用计数的智能指针 unique_ptr==**：**同一时间内，一个对象只能由一个 unique_ptr 所拥有。**

```c++
template <typename T>
class CSmartPtr{
public:
    CSmartPtr(T *ptr=nullptr):mptr(ptr) {}
    ~CSmartPtr() {delete mptr;}
    CSmartPtr(const CSmartPtr<T>&) = delete;
    CSmartPtr<T>& operator=(const CSmartPtr<T>&) = delete;
    CSmartPtr(CSmartPtr<T>&& rhs):mptr(rhs.mptr){
        rhs.mptr = nullptr;
    }
    CSmartPtr<T>& operator=(CSmartPtr<T>&& rhs){
        if(this != &rhs){
            delete mptr;
        	mptr = rhs.mptr;
        	rhs.mptr = nullptr;
        }
        return *this;
    }
    T& operator*() {return *mptr;}
    T* operator->() {return mptr;}
private:
    T* mptr;
};

class Test {
public:
    Test() { std::cout << "Test 构造" << std::endl; }
    ~Test() { std::cout << "Test 析构" << std::endl; }
};

int main() {
    // unique_ptr<Test> p1 = make_unique<Test>();
    CSmartPtr<Test> p1(new Test()); 

    // 尝试拷贝（会编译报错，证明 delete 起作用了）
    // CSmartPtr<Test> p2 = p1; 

    // 转移所有权（调用移动构造函数）
    CSmartPtr<Test> p3 = std::move(p1); 

    *p1 = 20;//错误❌，运行时崩溃

    return 0; // p3 析构，Test 对象自动释放
}
```



**==带引用计数的智能指针shared_ptr,weak_ptr==**：**同一时间内，一个对象可以被多个 share_ptr 所拥有。给每一个对象资源匹配一个引用计数（堆上），当智能指针指向资源的时候，引用计数+1，当超出作用域范围的时候，引用计数-1，引用计数为0的时候，资源被释放。一个 `shared_ptr` 在栈上实际上占两个指针的大小：`mptr`：指向堆区的原始对象；`ref_cnt_ptr`：指向堆区的控制块（包含资源引用计数和弱引用计数）。**

```c++
struct Monster {
    std::string name;
    Monster(std::string n) : name(n) { std::cout << name << " 出生了\n"; }
    ~Monster() { std::cout << name << " 被消灭了\n"; }
};

int main() {
    // 方式 A：推荐！一次性分配内存，安全高效
    auto p1 = std::make_shared<Monster>("哥斯拉");

    // 方式 B：传统的初始化（不推荐，存在异常安全风险且内存分配两次）
    // std::shared_ptr<Monster> p1(new Monster("哥斯拉"));

    std::cout << "当前关注者数量: " << p1.use_count() << std::endl; // 输出 1

    {
        auto p2 = p1; // 拷贝构造，股份共享
        std::cout << "p2 加入后，数量: " << p1.use_count() << std::endl; // 输出 2
    } // p2 离开作用域，计数减 1

    std::cout << "p2 走后，数量: " << p1.use_count() << std::endl; // 输出 1

    return 0; // p1 离开作用域，计数归零，Monster 析构
}
```

**==1️⃣shared_ptr 是线程安全的吗？==**

①**引用计数操作是线程安全的**：标准库内部使用原子操作（Atomic）来 $+1$ 或 $-1$。

②**智能指针对象本身不是线程安全的**：如果你在多个线程中同时修改**同一个** `shared_ptr` 对象的指向，依然需要加锁。

③**指向的对象不是线程安全的**：智能指针不管业务数据的并发安全。

2️⃣**==为什么推荐使用make_unique()和make_shared()替代传入裸指针？==**

**①使用 `make_unique`，对象的创建和智能指针的构造被封装在一个原子操作中，保证了即便抛出异常，内存也会被正确释放。**

**②只需调用一次内存分配。普通写法 (new)： 需要执行两次内存分配，一次给对象本身，另一次给智能指针内部的“控制块”。这两个块在内存中是分开的。使用 `make_shared`： 编译器会分配一整块连续的内存，同时容纳对象和控制块。**

**③代码简洁，不需要重复写两次类名。**

```cpp
// 冗余
unique_ptr<VeryLongClassName> p(new VeryLongClassName());
// 简洁
auto p = make_unique<VeryLongClassName>();
```



**==智能指针的交叉引用问题==**

**==shared_ptr 有一个致命缺陷：循环引用。如果 A 持有 B 的 shared_ptr，B 也持有 A 的 shared_ptr，它们的计数器永远是 1，内存永不释放。==**

```c++
       栈内存 (Stack)                 堆内存 (Heap)
      +--------------+         +--------------------------+
      |  ptrParent   | ------> |      对象 Parent          |
      | (shared_ptr) |         |  {                       |
      +--------------+         |    shared_ptr child_ptr; | --+
  ptrParent.use_count()=2      |  }                       |   |
                               +--------------------------+   |
                                            ^                 |
                                            |                 |
                                            | 互相指向         |
                                            |                 |
                               +--------------------------+   |
      +--------------+         |      对象 Child           |   |
      |   ptrChild   | ------> |  {                       |   |
      | (shared_ptr) |         |    shared_ptr parent_ptr;| <-+
      +--------------+         |  }                       |
    ptrChild.use_count()=2     +--------------------------+
//作用域结束：
//栈上的 ptrParent 析构，Parent 的计数减 1（变为 1）。因为计数不为 0，Parent 不会析构。
//栈上的 ptrChild 析构，Child 的计数减 1（变为 1）。因为计数不为 0，Child 不会析构。    
```



**==weak_ptr 的出现就是为了解决循环引用：它指向对象，但不增加引用计数，也不直接支持 operator* 或 operator->==**

==**如果你想用它，必须调用 lock() 方法将其提升为shared_ptr**。==如果对象还活着，它会返回一个 **shared_ptr**；如果对象已经死了，它返回空。

```c++
class Child;
class Parent {
public:
    // 如果这里用 shared_ptr，会形成循环引用
    std::weak_ptr<Child> child_ptr; 
    ~Parent() { std::cout << "Parent 析构" << std::endl; }
};

class Child {
public:
    std::shared_ptr<Parent> parent_ptr;
    ~Child() { std::cout << "Child 析构" << std::endl; }
};

int main() {
    auto p = std::make_shared<Parent>();
    auto c = std::make_shared<Child>();
    p->child_ptr = c; // weak_ptr 指向，p->usecount()=2
    c->parent_ptr = p; // shared_ptr 指向，c->usecount()=1
    
    return 0; // 正常析构！因为 p 释放时，其计数器能减到 0
}
```



**==多线程访问共享对象问题==**

```c++
#include <memory>
#include <thread>

class A{
public:
    A() { cout<<"A()"<<endl;}
    ~A() { cout<<~A()<<endl;}
    void testA() {  cout<<"非常好用的方法"<<endl;}
};
// 子线程
void handler01(weak_ptr<A> wk_ptr) {
    // 尝试提升为 shared_ptr
    if (std::shared_ptr<A> q = wk_ptr.lock()) {
        q->testA(); // 提升成功，说明对象还活着
    } else {
        std::cout << "对象已经析构了，不再执行操作" << std::endl;
    }
}

int main() {
    auto p = std::make_shared<A>();
    std::thread t1(handler01, std::weak_ptr<A>(p)); // 传入弱引用，计数仍为 1
    
    // 主线程主动销毁对象
    p.reset(); 
    
    t1.join();
    return 0;
}
```



**==智能指针的删除器==**

**智能指针默认使用 delete 来释放它所管理的资源。但在处理非内存资源（如文件句柄、数据库连接）时，自定义删除器就派上大用场了。**

```c++
template<typename T>
class MyDeleter{
public:
    void operator()(T* ptr) const{
        cout<<"operator()"<<endl;
        delete [] ptr;
    }
};

int main(){
    unique_ptr<int,MyDeleter<int>>ptr1(new int[100]);//delete []ptr
    
    unique_ptr<int,function<void (int*)>>ptr2(new int[100],[](int* p){delete []p;});
    unique_ptr<FILE,function<void(FILE*)>>ptr3(fopen("data.txt","w"),[](FILE* p){fclose(p);});
    return 0;
}
```



## 第十章 绑定器和函数对象

**==bind和function机制==**

**==bind==**：**将一个函数与其参数进行绑定，生成一个新的可调用对象（Function Object）**。

```c++
void printSum(int a, int b) {
    cout << a + b << endl;
}

int main() {
    // 将第一个参数 a 绑定为 10，_1 表示新函数的第一个参数将传给 b
    auto addTen = bind(printSum, 10, placeholders::_1);

    addTen(5);  // 输出 15 (等同于调用 printSum(10, 5))
    addTen(20); // 输出 30 (等同于调用 printSum(10, 20))
}
```



**==function==：存储和调用一个可调用对象,虽然 bind 的很多功能已经被更直观的 Lambda 取代了，但 std::function 作为“万能包装器”的地位依然不可撼动。**

==**function简单实现**==

```c++
template<typename Fty>
class myfunction {};
template <typename R,typename A1>
class mufunction<R(A1)>{ //部分特例化
public:
    using PFUNC = R(*)(A1);
    myfunction(PFUNC pfunc):_pfunc(pfunc) {}
    R operator()(A1 arg){
        return _pfunc(arg);
    }
private:
	PFUNC _pfunc;
};
template <typename R,typename A1,typename A2>
class mufunction<R(A1,A2)>{  //部分特例化
public:
    using PFUNC = R(*)(A1,A2);
    myfunction(PFUNC pfunc):_pfunc(pfunc) {}
    R operator()(A1 arg,A2 arg2){
        return _pfunc(arg1,arg2);
    }
private:
	PFUNC _pfunc;
};

void hello(string s){ cout<<s<<endl;}
int sum(int a,int b) {return a+b;}

int main(){
    myfunction<void(string)> func1(hello);
    func1("hello_world");//func1.operator()("hello_world")
    myfunction<int(int,int)> func2(sum);
    func2(2,3);
}
```

**==实例1==**

```c++
void normalFunction(int n) { std::cout << "普通函数: " << n << std::endl; }

struct Functor {
    void operator()(int n) { std::cout << "仿函数: " << n << std::endl; }
};

int main() {
    // 声明一个接受 int，返回 void 的容器
    // 1. 存储普通函数
    function<void(int)> func = normalFunction;
    func(1);

    // 2. 存储 Lambda 表达式
    function<void(int)> func2 = [](int n) { cout << "Lambda: " << n << std::endl; };
    func2(2);

    // 3. 存储仿函数
    function<void(int)> func3 = Functor();//临时对象赋值给func
    func3(3);

    // 4. 存储 bind 的结果
    auto boundFunc = std::bind(normalFunction, std::placeholders::_1);
    function<void(int)> func4 = boundFunc;
    func4(4);

    return 0;
}
```



**==lambda表达式的实现原理==**

**编译器会自动为你生成一个匿名的“仿函数”类，并重载()运算符，在调用的时候实例化对象并调用成员函数。**

```c++
//语法
//[捕获变量](形参列表)->返回值{操作代码};
int main(){
	int x = 10;
	auto lambda = [x](int a) { return x + a; };
	int res = lambda(5);
    return 0;
}
//自动翻译为⬇️
// 1. 编译器自动生成的匿名类名（由编译器随机命名，如 __Lambda_uuid）
class __Lambda_1 {
public:
    // 构造函数：负责把捕获的变量存进来
    __Lambda_1(int x) : _x(x) {}

    // 重载 operator()：Lambda 的主体逻辑
    //在默认情况下，Lambda 生成的 operator() 是 const 成员函数,如果你加了mutable关键字：[x]() mutable { x++; }，编译器就会去掉那个 const 修饰符
    int operator()(int a) const {
        return _x + a;
    }
private:
    int _x; // 捕获的变量成为了类的成员变量
};
// 2. 调用处实际发生的事情
int x = 10;
__Lambda_1 lambda(x); // 实例化这个类
int res = lambda.operator()(5); // 调用成员函数
//修改操作：
//按值捕获（必须加 mutable），并且仅影响 Lambda 内部的副本,
//按引用捕获（不加 mutable）,直接影响外部真实的 x
```



## 第十一章 C++ 11特性

✅**auto、nullptr、foreach、右值引用、std::move移动语义、std::forward完美转发**

**✅绑定器 bind 、函数对象 function、lambda表达式**

**✅智能指针unique_ptr、shared_ptr、weak_ptr**

**✅容器 array、list**

**✅C++ 语言级别多线程（代码可以跨平台）**： C++ 标准库并不是自己发明了线程，它只是一个**“翻译官”**。在 Linux 上，它把 `std::thread` 翻译成 `pthread_create`；在 Windows 上，它翻译成 `CreateThread`。

### std::thread线程库

```c++
#include <thread>

void threadHandle1(int time){
    std::this_thread::sleep_for(std::chrono::seconds(time));//this_thread:命名空间,chrono:命名空间
    cout<<"hello_world"<<endl;
}
int main(){
    thread t1(threadHandle1,2);//定义一个线程对象，传入一个线程函数，新线程就开始运行了
    //t1.join();//主线程等待子线程结束，主线程继续往下运行
    t1.detach();//把子线程设置为分离线程
    
    return;
}
```



### 线程互斥操作

```c++
int tcount = 100;

void sellTicket_Mutex(int index) {
    while (tcount > 0) {
        mtx.lock();
        if (tcount <= 0) {
            mtx.unlock(); // 关键：退出前必须手动解锁
            break; 
        }
        //tcount--这行代码在 CPU 层面分为三步➡️:1️⃣mov eax tcount内存的值读入寄存器;2️⃣sub eax 1;3️⃣mov tcount eax;期间任何一步可能该线程失去时间片
        cout << "窗口" << index << " 卖出: " << tcount-- << endl;
        mtx.unlock(); 
        this_thread::sleep_for(chrono::milliseconds(100));
    }
}
void sellTicket_LockGuard(int index) {
    while (tcount > 0) {
        {
            // 通过大括号限制 lock 的生命周期，构造时加锁，析构时自动解锁
            std::lock_guard<std::mutex> lock(mtx); 
            if (tcount <= 0) break;
            cout << "窗口" << index << " 卖出: " << tcount-- << endl;
        } // 锁在这里自动释放（lock 析构）
        // 此时其他线程可以抢锁，当前线程去休息
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}
void sellTicket_UniqueLock(int index) {
    while (tcount > 0) {
        std::unique_lock<std::mutex> lock(mtx);
        if (tcount <= 0) break; //析构函数会自动释放锁。
        cout << "窗口" << index << " 卖出: " << tcount-- << endl; 
        // unique_lock 的优势：可以手动提前解锁，延迟加锁
        lock.unlock(); 
        // 可以在解锁后做一些耗时操作
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

int main() {
    vector<jthread> tlist; //std::jthread 会在生命周期结束时自动 join
    
    for (int i = 1; i <= 3; i++) {
        // emplace_back 会自动根据参数调用 thread 的构造函数
        tlist.emplace_back(sellTicket_LockGuard, i); 
    }
 
    cout << "票已售罄，主线程安全退出。" << endl;
    return 0;
}

```



### 多线程同步通信

==**多线程编程的两个问题**==：
1️⃣**线程间互斥**
**竞态条件->临界区代码段->保证原子操作->互斥锁mutex,轻量级的无锁实现CAS**
2️⃣**线程间的同步通信**
**生产者-消费者线程模型**

**==生产者消费者模型==**是并发编程中最经典的设计模式。它通过一个**中间容器（缓冲区）将“产生数据的角色”和“处理数据的角色”解耦，解决它们之间速度不匹配**的问题。

**==生产者== **：**生产数据，放入缓冲区。如果缓冲区满了，就睡觉等待。**

**==消费者==**：**从缓冲区取数据处理。如果缓冲区空了，就睡觉等待。**

**==缓冲区==**：**通常是一个原始队列 `std::queue`。**

==**同步机制**==：

**①std::mutex：保护队列的线程安全。**

**②std::condition_variable：负责线程间的“通知”和“唤醒”。**

```c++
#include <iostream>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>

using namespace std;

queue<int> buffer;      // 缓冲区
const int MAX_SIZE = 10; // 缓冲区最大容量
mutex mtx;              // 全局锁
condition_variable cv;  // 条件变量

void producer(int id) {
    int data = 0; // 每个线程都会在自己的栈空间里创建一个属于自己的 data，初始值都是 0
    while (true) {
        unique_lock<mutex> lock(mtx);
        
        // wait作用：让出 CPU 调度权并进入阻塞状态，直到满足特定条件才醒来
        // 1. 等待缓冲区有空位 (谓词写法防止虚假唤醒)
        cv.wait(lock, [] { return buffer.size() < MAX_SIZE; });

        // 2. 生产数据
        data++;
        buffer.push(data);
        cout<<"生产者"<<id<<"生产了:"<<data<<" (当前容量:"<<buffer.size()<<")"<<endl;

        // 3. 通知消费者可以取票了
        lock.unlock(); // 提前手动解锁提升性能
        cv.notify_all(); 

        this_thread::sleep_for(chrono::milliseconds(200)); // 生产速度
    }
}

void consumer(int id) {
    while (true) {
        unique_lock<mutex> lock(mtx);

        //wait作用：让出 CPU 调度权并进入阻塞状态，直到满足特定条件才醒来
        //自动释放锁：它会立即释放你传进去的unique_lock。这样其他线程（比如生产者）才能拿到这把锁去修改共享数据。
        //休眠挂起： 线程进入阻塞状态，不再占用 CPU 资源。
        //重新拿锁:当被notify唤醒且满足条件后,它会先去抢锁.抢到锁后,wait才会返回,继续执行后面的代码.
        // 1. 等待缓冲区不为空
        cv.wait(lock, [] { return !buffer.empty(); });

        // 2. 消费数据
        int data = buffer.front();
        buffer.pop();
        cout << "消费者 " << id << " 消费了: " << data << endl;

        // 3. 通知生产者可以继续生产了
        lock.unlock();
        cv.notify_all();

        this_thread::sleep_for(chrono::milliseconds(500)); // 消费较慢
    }
}

int main() {
    vector<thread> threads;
    // 2个生产者
    for(int i=0; i<2; ++i) threads.emplace_back(producer, i);
    // 3个消费者
    for(int i=0; i<3; ++i) threads.emplace_back(consumer, i);

    for(auto& t : threads) t.join();
    return 0;
}
/*
wait 过程的“接力”演示
想象你在一个奶茶店（生产者消费者模型）：
消费者 (你)： 发现柜台没奶茶了 (buffer.empty())。
执行 wait： 你放开柜台的通行证 (lock.unlock())，去旁边长椅上坐着闭目养神 (Block)。
生产者 (店员)： 拿到通行证，做好了奶茶放柜台，喊一声“奶茶好了！” (notify)。
消费者 (你)： 睁开眼，重新去排队抢通行证 (lock.lock())。
确认： 抢到证后看一眼柜台，果然有奶茶，wait 彻底结束，你拿走奶茶。
*/
```

1️⃣**为什么必须用 `unique_lock` 而不是 `lock_guard`？**

因为 `std::condition_variable::wait()` 在进入等待状态时，会自动执行 `mtx.unlock()` 释放锁，以便其他线程能操作队列。被唤醒时，它又会自动执行 `mtx.lock()` 重新拿回锁。 **`lock_guard` 没有 `unlock` 的接口，无法配合条件变量使用。**

2️⃣**虚假唤醒** 

线程有时会在没有任何人 `notify` 的情况下莫名其妙醒来。所以必须使用 `wait(lock, pred)` 这种带 lambda 表达式的写法：

```c++
cv.wait(lock, [] { return !buffer.empty(); });
// 等价于：
while (buffer.empty()) {
    cv.wait(lock);
}
```

这样即使虚假唤醒，由于 `while` 条件不成立，它会立刻再次进入睡眠。

3️⃣ **notify_one vs notify_all**

**notify_one**：只唤醒一个正在等待的线程。效率高，但如果逻辑复杂可能导致死锁（比如唤醒了另一个生产者而非消费者）。

**notify_all**：唤醒所有等待线程。更安全，但在线程极多时会有“惊群效应”影响性能。



### **基于 CAS 操作的 atomic 原子类型**

**==std::atomic==** **的核心秘诀在于：==它不使用操作系统层面的“锁”（Mutex），而是直接利用 CPU 提供的 CAS(Compare And Swap) 指令==。这是一种乐观锁的思想。**

1️⃣==**什么是 CAS 操作？**==

**CAS 是一条原子指令，它包含三个操作数：**

**内存地址 V**：要修改的变量。

**旧的预期值 A**：你认为现在变量应该是多少。

**新的目标值 B**：你想把它改成多少。

**逻辑是：** “我认为变量现在是 A，如果是，请帮我改成 B；如果不是（说明别人抢先改了），那就别动，告诉我失败了。”

2️⃣==**为什么它比 Mutex 快？**==

**Mutex（悲观锁）：** 线程去拿锁，如果拿不到，操作系统会把线程挂起（切出 CPU）。这种**上下文切换**非常昂贵。

**Atomic（乐观锁/自旋）：** 线程不挂起。它会不停地尝试 CAS，失败了就重试。因为 `tcount--` 这种操作极快，重试的开销远小于线程挂起的开销。

```c++
#include <iostream>
#include <atomic>
#include <thread>
#include <vector>

// 1. 定义原子类型
std::atomic<int> tcount{100}; 

void sellTicket_Atomic(int index) {
    while (true) {
        // 2. 使用 fetch_sub 进行原子自减
        // 这行代码底层就是 CPU 的原子指令（如 x86 的 LOCK SUB）,它返回的是减 1 之前的值
        int current = tcount.fetch_sub(1);
        if (current <= 0) {
            // 防止减成负数，修正回来
            tcount = 0; 
            break;
        }
        // 注意：输出依然可能会乱序，但 tcount 的数值绝对准确
        std::cout << "窗口 " << index << " 卖出: " << current << "号票" << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}

int main() {
    std::vector<jthread> tlist;
    for (int i = 1; i <= 3; i++) {
        tlist.emplace_back(sellTicket_Atomic, i);
    }

    return 0;
}
```



## 第十二章 C++设计模式

### 1.单例模式

**一个类不管创建多少次对象，永远只能得到该类型一个对象的实例，比如日志模块**

```c++
class Singleton {
public:
    // 1. 获取实例的静态方法
    static Singleton& getInstance() {
        static Singleton instance; // C++11 保证了这里是线程安全的原子初始化
        return instance;
    }
    // 2. 禁止拷贝和赋值（核心点！）
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
private:
    // 3. 私有化构造函数
    Singleton() { /* 比如打开数据库连接 */ }
    ~Singleton() { /* 比如关闭数据库连接 */ }
};
/*
1️⃣单例模式为什么需要是线程安全的？
防止多个线程同时构造对象导致程序崩溃或逻辑错误
2️⃣为什么静态局部变量的初始化是线程安全的？
编译器在处理静态局部变量时，插入了一套守护逻辑
编译器会自动生成一个隐形标志位(原子类型的)，检查是否完成初始化；如果没有初始化，那么加锁完成初始化，并更改标志位；如果已经完成初始化则什么也不做
*/
```



### 2.简单工厂、工厂方法和抽象工厂

**==工厂模式：主要是封装了类的创建==**

**==简单工厂==：一个工厂类根据传入的参数，决定创建哪一种产品类的实例。**

```c++
class Phone { public: virtual void info() = 0; virtual ~Phone() {} };
class iPhone : public Phone { public: void info() override { std::cout << "生产了一台 iPhone\n"; } };
class Huawei : public Phone { public: void info() override { std::cout << "生产了一台华为手机\n"; } };

class SimpleFactory {
public:
    static std::unique_ptr<Phone> createPhone(const std::string& type) {
        if (type == "Apple") return std::make_unique<iPhone>();
        if (type == "Huawei") return std::make_unique<Huawei>();
        return nullptr;
    }
};
// 使用例子
int main() {
    auto p1 = SimpleFactory::createPhone("Apple");
    if (p1) p1->info();
    return 0;
}
// 违反了开闭原则。每次增加新产品，都要修改工厂类的 if-else 逻辑。
```

**==工厂方法==：不再由一个统一的工厂类生产所有产品，==而是每个产品都对应一个专属工厂==。**

```c++
// 抽象工厂接口
class Factory { public: virtual std::unique_ptr<Phone> create() = 0; virtual ~Factory() {} };

// 专门生产 iPhone 的工厂
class AppleFactory : public Factory {
public:
    std::unique_ptr<Phone> create() override { return std::make_unique<iPhone>(); }
};

// 专门生产华为的工厂
class HuaweiFactory : public Factory {
public:
    std::unique_ptr<Phone> create() override { return std::make_unique<Huawei>(); }
};

// 使用例子
int main() {
    std::unique_ptr<Factory> myFactory = std::make_unique<AppleFactory>();
    auto p2 = myFactory->create(); // 以后想换华为，只需改动上面一行 new 的工厂对象
    p2->info();
    return 0;
}
```

**==抽象工厂==：提供一个接口，==用于创建对象家族==，而不需要指定具体类。**

```c++
//特点： 它可以强制让“手机”和“手表”配套（比如不会出现苹果工厂产出华为手表的情况）。它解决的是产品族的问题。
// 产品 B：手表
class Watch { public: virtual void info() = 0; virtual ~Watch() {} };
class AppleWatch : public Watch { public: void info() override { cout << "生产了一块 Apple Watch\n"; } };
class HuaweiWatch : public Watch { public: void info() override { cout << "生产了一块华为手表\n"; } };

// 抽象工厂：能生产一整套全家桶
class AbstractFactory {
public:
    virtual unique_ptr<Phone> createPhone() = 0;
    virtual unique_ptr<Watch> createWatch() = 0;
};

// 具体的全家桶工厂
class AppleChainFactory : public AbstractFactory {
public:
    unique_ptr<Phone> createPhone() override{ return make_unique<iPhone>(); }
    unique_ptr<Watch> createWatch() override{return make_unique<AppleWatch>(); }
};

// 使用例子
int main() {
    unique_ptr<AbstractFactory> factory = make_unique<AppleChainFactory>();
    auto p3 = factory->createPhone();
    auto w3 = factory->createWatch();
    p3->info();
    w3->info();
    return 0;
}
```



### 3.代理模式

**==代理模式==：为其他对象(老板，委托类)提供一种代理(助理，代理类)以控制对这个对象(老板)的访问（智能指针实际上就是原指针的代理）。**

```c++
// 1. 抽象主题
class Internet {
public:
    virtual void connectTo(string serverhost) = 0;
    virtual ~Internet() {}
};

// 2. 真实主题：真正的网络连接
class RealInternet : public Internet {
public:
    void connectTo(string serverhost) override {
        cout << "正在连接到: " << serverhost << endl;
    }
};

// 3. 代理类：增加了安全过滤功能
class ProxyInternet : public Internet {
private:
    RealInternet* realInternet; // 🌟🌟🌟 持有真实对象的引用
    static vector<string> bannedSites;

public:
    ProxyInternet() : realInternet(new RealInternet()) {}
    ~ProxyInternet() { delete realInternet; }

    void connectTo(string serverhost) override {
        for (auto site : bannedSites) {// 在调用真实对象前进行“审查操作”
            if (serverhost == site) {
                cout << "访问拒绝：网站 [" << serverhost << "] 已被屏蔽！" << endl;
                return;
            }
        }
        realInternet->connectTo(serverhost); // 转发请求
    }
};
vector<string> ProxyInternet::bannedSites = {"abc.com","bad.com"};//静态成员的类外初始化

// 使用例子
int main() {
    Internet* internet = new ProxyInternet();
    internet->connectTo("google.com"); // 允许访问
    internet->connectTo("abc.com");    // 代理会拦截
    delete internet;
    return 0;
}
```



### 4.装饰器模式

**==动态地给一个对象添加一些额外的功能，而不改变其结构==。相比生成子类（继承）更为灵活，因为一种子类只能添加一种功能**

```c++
// 1. 抽象组件：咖啡基类
class Coffee {
public:
    virtual std::string getDescription() = 0;
    virtual double cost() = 0;
    virtual ~Coffee() {}
};

// 2. 具体组件：浓缩咖啡
class Espresso : public Coffee {
public:
    std::string getDescription() override { return "浓缩咖啡"; }
    double cost() override { return 12.0; }
};

// 3. 抽象装饰器：必须继承自 Coffee 且包含一个 Coffee 指针
class CondimentDecorator : public Coffee {
protected:
    unique_ptr<Coffee> _coffee; // 被包裹的对象
public:
    CondimentDecorator(unique_ptr<Coffee> c) : _coffee(move(c)) {}
};

// 4. 具体装饰器：牛奶
class Milk : public CondimentDecorator {
public:
    Milk(unique_ptr<Coffee> c) : CondimentDecorator(move(c)) {}

    string getDescription() override {
        return _coffee->getDescription() + " + 牛奶";
    }
    double cost() override {
        return _coffee->cost() + 3.0; // 原价 + 牛奶钱
    }
};

// 5. 具体装饰器：摩卡
class Mocha : public CondimentDecorator {
public:
    Mocha(unique_ptr<Coffee> c) : CondimentDecorator(move(c)) {}

    string getDescription() override {
        return _coffee->getDescription() + " + 摩卡";
    }
    double cost() override {
        return _coffee->cost() + 5.0;
    }
};

// 使用例子
int main() {
    // 1. 点一杯浓缩咖啡
    unique_ptr<Coffee> myCoffee = make_unique<Espresso>();

    // 2. 加牛奶 (包裹一层)
    myCoffee = make_unique<Milk>(move(myCoffee));//move表示智能指针所有权的转移

    // 3. 再加摩卡 (再包裹一层)
    myCoffee = make_unique<Mocha>(move(myCoffee));

    cout << "订单详情: " << myCoffee->getDescription() << endl;
    cout << "总价格: " << myCoffee->cost() << " 元" << endl;

    return 0;
}
```



### 5.适配器模式

**==把一个类的接口转换成客户希望的另外一个接口==，你的电源适配器（将 220V 电压转为 5V 供给手机）。**

```c++
// 1. 目标接口：用户想要的 USB 充电接口
class USBTarget {
public:
    virtual void chargeWithUSB() = 0;
    virtual ~USBTarget() {}
};

// 2. 被适配者：现有的三相插座 (接口不匹配)
class TriplePhaseSocket {
public:
    void powerSupply() {
        std::cout << "三相插座正在供电..." << std::endl;
    }
};

// 3. 适配器：把三相插座包装成 USB 接口
class PowerAdapter : public USBTarget {
private:
    TriplePhaseSocket* _socket; // 持有被适配者的指针 (组合)

public:
    PowerAdapter(TriplePhaseSocket* s) : _socket(s) {}

    void chargeWithUSB() override {
        std::cout << "适配器将电流转换为 USB 标准 -> ";
        _socket->powerSupply(); // 最终还是调用现有的功能
    }
};

// 使用例子
int main() {
    // 墙上有一个老式三相插座
    TriplePhaseSocket* oldSocket = new TriplePhaseSocket();

    // 买一个适配器接在老插座上
    USBTarget* myAdapter = new PowerAdapter(oldSocket);

    // 手机开始通过适配器充电
    myAdapter->chargeWithUSB();

    delete myAdapter;
    delete oldSocket;
    return 0;
}
```



### 6.观察者模式

**定义对象间的一对多依赖，当一个对象状态改变时，所有依赖者都会收到通知。**

```c++
// 1. 观察者接口
class Observer {
public:
    virtual void update(const std::string& message) = 0;
    virtual ~Observer() {}
};

// 2. 被观察者 (主题)
class Subject {
private:
    std::vector<Observer*> _observers; 
public:
    void attach(Observer* observer) {
        _observers.push_back(observer);
    }
    void detach(Observer* observer) {
        // Erase-Remove Idiom: 只有匹配的那个指针会被删除
        _observers.erase(remove(_observers.begin(), _observers.end(), observer), _observers.end());
    }

    void notify(const string& message) {
        for (auto obs : _observers) {
            obs->update(message);
        }
    }
};

// 3. 具体的观察者
class User : public Observer {
private:
    std::string _name;
public:
    User(std::string name) : _name(name) {}
    void update(const std::string& message) override {
        std::cout << "User [" << _name << "] received notification: " << message << std::endl;
    }
};

int main() {
    // 实例化被观察者
    Subject techChannel;

    // 创建观察者
    User* user1 = new User("Alice");
    User* user2 = new User("Bob");
    // 订阅
    techChannel.attach(user1);
    techChannel.attach(user2);
    // 发送广播
    techChannel.notify("New video on 'Observer Pattern' is out!");
    // 取消订阅
    techChannel.detach(user1);    
    // 再次广播
    techChannel.notify("Upcoming: Advanced C++ Templates.");
    // 清理内存
    delete user1;
    delete user2;

    return 0;
}
/*
User [Alice] received notification: New video on 'Observer Pattern' is out!
User [Bob] received notification: New video on 'Observer Pattern' is out!
User [Bob] received notification: Advanced C++ Templates.
*/
```



## 第十三章 一些面试问题

#### **==1.海量数据的查重/去重==**

1️⃣**哈希表：内存可能爆炸💥**

2️⃣**分治思想**

```properties
1.有一个文件，有大量的整数（50亿个），内存限制400M，找出文件中重复的元素，重复的次数

分治法的思想：大文件划分成小文件，使得每一个小文件可以加载到内存中，求出对应的重复的元素，把结果写到一个存储重复元素的文件中。
小文件的个数：40G/400M≈128
遍历大文件的元素，把每一个元素根据哈希映射函数（除留余数法），放到对应序号的小文件当中（data%127 = file_index），值相同的，通过一样的哈希函数肯定放在同一个小文件当中的。把小文件加载到内存中进行处理

2.a,b两个文件，里面都有10亿个整数，内存限制400M，求出两个文件中重复的元素有哪些？
10亿->4G大小->两个文件->8G大小->8G/400M->27个小文件
把a和b分别划分成个数相等的一系列（27个）小文件
a1.txt,a2.txt...a26.txt
b1.txt,b2.txt...b26.txt
从a文件中读取数据，通过 数据%27=file_index
从b文件中读取数据，通过 数据%27=file_index
a,b两个文件中，数据相同的元素，进行哈希映射以后，肯定在相同序号的小文件中，那么每次把序号相同的小文件加载到内存中进行处理

//如果你的数据中，某个数字出现了 10 亿次怎么办？
对该小文件更换哈希函数再次切分（二次哈希），直到它能被装进内存。
```

3️⃣**Bloom Filter（布隆过滤器）**

4️⃣**TrieTree字典树（字符串类型）**



#### **==2.海量数据的TOP k==**

1️⃣**求最大/最小前 k 个元素**

2️⃣**求最大/最小第 k 个元素**

```cpp
问题：10000个整数，找值前10大的元素

解法1：大根堆/小根堆（priority_queue）
空间复杂度：O(K)
时间复杂度：O(Nlog K)      (N=10000,K=10)
先用前10个整数创建一个小根堆（最小值在堆顶），然后遍历剩下的整数，如果整数比堆顶元素大，那么删除堆顶元素（出堆），然后再把整数入堆，遍历完所有整数，小根堆里放的就是值最大的前10个元素，而堆顶就是第10大的元素了。

解法2：快排分割函数
时间复杂度：O(N)
空间复杂度：O(1)->原地修改数组
致命缺点：必须将所有数据一次性加载到内存。如果数据量是 100 亿，内存放不下，这个方法就没法直接用。
快排的核心函数是 partition，它能把数组分成两半：左边都比基准值小，右边都比基准值大。
执行一次 partition，得到基准值的索引 pivot_index。
如果 pivot_index == K：恭喜，基准值就是第 K 大（或小），左边的就是前 K 个。
如果 pivot_index > K：说明第 K 大在左半部分，递归左边。
如果 pivot_index < K：说明在右半部分，递归右边。

// 分区函数：把比 pivot 小的放左边，大的放右边
int partition(vector<int>& nums, int left, int right) {
    int pivot = nums[left]; // 选择最左边的作为基准
    int l = left, r = right;    
    while (l < r) {
        // 从右向左找第一个小于 pivot 的数
        while (l < r && nums[r] >= pivot) r--;
        nums[l] = nums[r];       
        // 从左向右找第一个大于 pivot 的数
        while (l < r && nums[l] <= pivot) l++;
        nums[r] = nums[l];
    }    
    nums[l] = pivot; // 基准值归位
    return l;        // 返回基准值最终所在的索引
}

// 快速选择函数
int quickSelect(vector<int>& nums, int left, int right, int k) {
    if (left == right) return nums[left];
    // 执行一次分区，得到索引 p
    int p = partition(nums, left, right);
    if (p == k) {
        // 运气好，正好是第 k 个
        return nums[p];
    } else if (p > k) {
        // 第 k 个在左半部分，递归左边
        return quickSelect(nums, left, p - 1, k);
    } else {
        // 第 k 个在右半部分，递归右边
        return quickSelect(nums, p + 1, right, k);
    }
}

int main() {
    vector<int> data = {3, 2, 1, 5, 6, 4};
    int k = 2; // 我们想找第 2 小的元素（即索引为 1 的位置，结果应该是 2）
    
    int result = quickSelect(data, 0, data.size() - 1, k - 1);
    
    cout << "第 " << k << " 小的元素是: " << result << endl;
    return 0;
}
```

**==3.构造函数和析构函数能不能抛出异常？==**

**构造函数可以抛出异常，但析构函数绝对不应该抛出异常。**

```properties
1.如果对象在初始化时发生了不可恢复的错误（比如内存不足），构造函数可以抛出异常，告诉外界对象创建失败了。
副作用（重要）：如果构造函数抛出了异常，该对象的析构函数将不会被调用。
内存泄漏风险： 如果在构造函数里已经 new 了一块内存，然后抛出异常，这块内存就会泄露。我们可以使用 RAII（如unique_ptr）。即使构造函数中断，已经构造完成的成员变量（智能指针）会自动销毁，释放其持有的内存。

2.强烈建议析构函数不抛出异常。在 C++11 之后，析构函数默认被标记为 noexcept。
原因：析构函数通常负责释放多个资源。如果在释放第一个资源时抛出异常，后面的 delete 或关闭文件的逻辑就会被跳过，导致资源残留。

```

**==4.宏和内联函数的区别==**

```properties
宏：在预处理阶段处理。它只是简单的文本替换，编译器根本不知道宏的存在，调试极其困难。

内联函数：在编译阶段处理。它是真正的函数，由编译器决定是否将其展开，支持调试。
```

**==5.拷贝构造函数，为什么传引用而不传值？==**

```properties
传值本身就需要调用拷贝构造函数，如果不传引用而传值，会引发“无限递归调用”，直到栈溢出（Stack Overflow）导致程序崩溃。
```

**==6.如何实现一个不可以被继承的类？==**

```cpp
//这是最简单、最直接的方法。C++11 引入了 final 关键字，专门用于禁止类被继承。
class Base final {
    // 这个类不可以作为基类
};

// 错误！编译器会报错：cannot derive from 'final' base
class Derived : public Base { 
};
```

**==7.虚函数表放在哪里？==**

```properties
.text 段：存放编译后的机器指令（函数体代码）。

//理由：虚函数表在编译期就已经确定了，运行期间不应该被修改。为了安全，操作系统将其放在只读区域。
.rodata 段（Read-Only Data）：存放常量、字符串字面量，以及虚函数表。

.data / .bss 段：存放全局变量和静态变量。

堆（Heap）：存放 new 出来的对象。

栈（Stack）：存放局部对象。
```

**==8.C++11容器emplace()原理剖析==**

```c++
class Test{
public:
    Test(int a) {cout<<"Test(int)"<<endl;}
    Test(int a,int b) {cout<<"Test(int,int)"<<endl;}
    ~Test() {cout<<"~Test()"<<endl;}
	Test(const Test&) {cout<<"Test(const Test&)"<<endl;}
    Test(Test&&) {cout<<"Test(Test&&)"<<endl;}
};

int main(){
    Test t1(10);
    vector<Test> v;
    v.reserve(100);
    
    //直接插入对象，push_back()与emplace_back()没有区别
    v.push_back(t1); //Test(const Test&)
    v.emplace_back(t1);//Test(const Test&)
    
    //emplace_back()传入构造函数的参数，直接在容器底层构造对象
    v.push_back(20);//Test(int)->Test(Test&&)->~Test()
    v.emplace_back(20);//Test(int)
    v.emplace_back(30,40);//Test(int,int)
    
    return 0;
}
```

