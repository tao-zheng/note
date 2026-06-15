# C Primer Plus



## 第 3 章  数据和C

#### 1️⃣ ==**整数类型**==

**int类型**

```cpp
int a = 1;
int b = 020; // 8进制
int c = 0x10; // 16进制

printf("dec = %d, oct = %o, hex = %x\n",a,a,a);
```

**unsigned与short类型,long类型,long long类型**

```cpp
// short类型不会比int类型长，long类型不会比int类型短
short erns;
long johns;
long long ribs;
unsigned int players;
unsigned long headcount;

printf("%d",erns);
print("%ld",johns);
print("%lld",ribs);
print("%u",players);
```

**整数溢出**

```cpp
int i = 2147483647; // INT_MAX;
unsigned int j = 4294967295; // UNSIGNED_MAX

printf("%d %d %d\n",i,i+1,i+2); // 2147483647 -2147483648 -2147483647
printf("%u %u %u\n",j,j+1,j+2); // 4294967295 0 1
```

**long常量和long long常量**

```cpp
12L   // long常量
020L  // long常量
0x10L // long常量
    
3LL  // long long常量
6ull // unsigned long long常量
9LLU // unsigned long long常量
```



#### 2️⃣ ==**字符类型**==

```cpp
char grade = 'A';
char grade = 65; // 可以，但不推荐

char ch;
scanf("%c",&ch);
printf("the code for %c is %d.\n",ch,ch)
```



#### 3️⃣ **==浮点类型==**

```cpp
// 声明
float a = 2.451856f;
double b = 3.1415926;


// 浮点常量
-1.26e12
1.87e-3
2.3f // float类型
5.35 // double类型
4.32L // long double类型
    
// 浮点值的上溢
float toobig = 3.4e37 * 100.0f;
printf("%f",toobig); // inf
printf("%5.2f",a); // 字段宽度为5个字符，小数点后2位
```



#### 4️⃣ **==类型大小==**

```cpp
printf("int 占用字节大小：%u\n",sizeof(int)); // 4
printf("long 占用字节大小：%u\n",sizeof(long)); // 4
printf("long long 占用字节大小：%u\n",sizeof(long long)); // 8

printf("char 占用字节大小：%u\n",sizeof(char)); // 1

printf("float 占用字节大小：%u\n",sizeof(float)); // 4
printf("double 占用字节大小：%u\n",sizeof(double)); // 8
printf("long double 占用字节大小：%u\n",sizeof(long double)); // 16
```



#### 5️⃣ ==scanf()、printf() 与 getchar()==

```cpp
//scanf()与getchar()的用法
//1. scanf("%d")
//工作规则：自动跳过前面的所有空白符。摸到数字开始抓，遇到非数字（比如字母、回车）立刻停手。
//残留物：那个让它停下来的“非数字字符”（通常是回车）会被留在传送带上。
//实战：传送带是 [回车]1024a -> 抓走 1024 -> 剩下 a。
//2. scanf("%s")
//工作规则：自动跳过前面的所有空白符。摸到可见字符开始抓，遇到下一个空白符立刻停手。
//残留物：那个让它停下来的空白符（比如单词后面的空格或回车）被留在传送带上。
//实战：传送带是 [空格]Tom[回车] -> 抓走 Tom -> 剩下 [回车]。
//3. scanf("%c")
//工作规则：绝对不跳过任何东西。不管传送带最前面是字母还是回车，直接一把抓走这唯一的一个字符。
//残留物：剩下的东西原地不动。
//实战：如果上一步 scanf("%d") 留下了 [回车]，它会直接把 [回车] 抓走，导致你根本没机会输入新字符。
//4. scanf(" %c") —— 智能清障的特技！✨
//工作规则：依靠前面的空格指令，自动扔掉传送带前方所有的空白符垃圾，直到摸到第一个非空白符，抓走它！
//残留物：后面的内容原地不动。
//实战：用来完美衔接在 %d 或 %s 之后。不管上一步留下了多少空格和回车，它都能完美滤过，准确抓到你输入的下一个字母。
//5. getchar() 
//工作规则：逻辑和无脑的 %c 一模一样，一次只从最前面拿一个字符（包括回车）。
//神级用法：用来手动清空传送带。写一行 while(getchar() != '\n');工人就会疯狂把传送带上的东西扔进垃圾桶，直到把回车也扔掉，还你一条干净的传送带。
--------------------------------------------------------------------------------------------------------
// printf()函数什么时候把输出传送给屏幕？
/*
1.首先，printf()函数将输出传递给一个被称作缓冲区(buffer)的存储区域，缓冲区的内容被不断的传递给屏幕；
2.满足以下 4 种条件，将缓冲区内容传给屏幕：
	①缓冲区满；
	②遇到换行符\n；
	③遇到输入scanf()；
	④程序结束了；
*/

```



#### 6️⃣ ==编程练习==

```cpp
//1. 通过试验的方法观察系统如何处理整数上溢、浮点数上溢和浮点数下溢的情况。
#include <limits.h>
#include <float.h>
int i = INT_MAX; // 2147483647
float f = FLT_MAX; // 3.402823e+38
float f2 = FLT_MIN; // 1.175494e-38

printf("%d %d %d",i,i+1,i+2); // 整数上溢：环绕 2147483647 -2147481648 -2147481647
printf("%e",f * 100.0f); // 浮点数上溢：无穷 inf
printf("%e",f2 / 100.0f); // 浮点数下溢：精度损失 -1.175493e-40
-----------------------------------------------------------------------------------
//2. 编写一个程序，要求输入一个ASCI 码值（如 66），然后输出相应的字符。
int ascii;
scanf("%d",&ascii);
printf("%c",ascii);
    
//4．编写一个程序，读入一个浮点数，并分别以小数形式和指数形式打印。
float f1;
scanf("%f",&f1);
printf("%f %e",f1,f1);

//5．一年约有3.156e7s。编写一个程序，要求输入您的年龄，然后显示该年龄合多少秒。
int age;
scanf("%d",&age);
printf("%le",age * 3.156e7);

//6. 1个水分子的质量约为3.0e-23g，1夸脱水大约有950g。编写一个程序，要求输入水的夸脱数，然后显示这么多水中包含多少个水分子。
float mass = 3.0e-23;
float mass_quart = 950;
float quarts;
scanf("%f",&quarts);
printf("%e",quarts * mass_quart / mass);

//7. 1英寸等于2.54cm。编写一个程序，要求输入您的身高，然后显示该身高值等于多少厘米。如果您愿意，也可以要求以厘米为单位输入身高，然后以英寸为单位进行显示。
float inches;
scanf("%f",&inches);
printf("%f cm",inches * 2.54);
```



## 第 4 章 字符串和格式化输入/输出

#### 1️⃣ ==使用字符串==

```cpp
// 使用字符串必须创建一个数组，把字符串中的字符逐个地放进数组中，还要记着在结尾添加一个\0。幸运的是，计算机可以自己处理这些细节问题。
#include <stdio.h>
int main(){
    char name[40];
	scanf("%s",name); // 遇到空格、制表、换行符停止读入，例如tao zheng,只读入tao，一般使用gets()函数
	printf("hello,%s",name);
    return 0;
}
```



#### 2️⃣ ==strlen()函数==

```cpp
#include <stdio.h>
#include <string.h>
#define PRAISE "what a super marvelous name!"
int main(){
    char name[40];
	scanf("%s",name); // Rick
	printf("name的字母数量：%d,name占用的字节数量：%d",strlen(name),sizeof(name));// 4,40
	printf("PRAISE字母数量：%d,PRAISE占用字节数量：%d",strlen(PRAISE),sizeof(PRAISE)); // 28,29
    return 0;
}
```



#### 3️⃣ ==编程练习==

```cpp
//1．编写一个程序，要求输入名字和姓氏，然后以“名字，姓氏”的格式打印。
char fname[40];
char lname[40];
scanf("%s",fname);
scanf("%s",lname);
printf("%s,%s",lname,fname);

//2. 编写一个程序，要求输入名字，并执行以下操作：
//a.把名字引在双引号中打印出来。
char name[40];
scanf("%s",name);
printf("\"%s\"",name);
//b.在宽度为20个字符的字段内打印名字，并且整个字段引在引号内。
printf("\"%20s\"",name);
//c.在宽度为20个字符的字段的左端打印名字，并且整个字段引在引号内。
printf("\"%-20s\"",name);

//3. 编写一个程序，读取一个浮点数，并且首先以小数点记数法，然后以指数记数法打印之。
float num;
scanf("%f",&num);
printf("%0.1f %0.1e",num,num);
printf("%+0.3f %0.3E",num,num);

//4. 编写一个程序，要求输入身高和名字，然后以如下形式显示：Dabney you are 6.208 feet tall
float height;
char name[40];
scanf("%f",&height);
scanf("%s",name);
printf("%s,you are %.3f feet tall",name,height)

//5. 编写一个程序，首先要求用户输入名字，然后要求用户输入姓氏。在一行打印输入的姓名，在下一行打印每个名字中字母的个数。把字母个数与相应名字的结尾对齐；然后打印相同的信息，但是字母个数与相应单词的开始对齐。
#include <string.h>
char fname[40];
char lname[40];
scanf("%s",fname);
scanf("%s",lname);
printf("%10s %10s\n",lname,fname);
printf("%10d %10d\n",strlen(lname),strlen(fname));
printf("%-10s %-10s\n",lname,fname);
printf("%-10d %-10d\n",strlen(lname),strlen(fname));
    
//6. 编写一个程序，设置一个值为1.0/3.0的double类型变量和一个值为1.0/3.0的float类型变量。每个变量的值显示三次：一次在小数点右侧显示4个数字，一次在小数点右侧显示12个数字，另一次在小数点右侧显示16个数字。同时要让程序包括 float.h文件，并显示FLT_DIG和DBL_DIG的值。1.0/3.0的显示值与这些值一致吗？
#include <float.h>
double a = 1.0/3.0;
float b = 1.0/3.0;
printf("%.4f %.12f %.16f",a,a,a);// 0.3333 0.333333333333 0.3333333333333333
printf("%.4f %.12f %.16f",b,b,b);// 0.3333 0.333333343267 0.3333333432674408
printf("%d",FLT_DIG); // 6 最少有效数字位数
printf("%d",DBL_DIG); // 15 最少有效数字位数
    
//7. 编写一个程序，要求用户输入行驶的英里数和消耗汽油的加仑数。接着应该计算和显示消耗每加仑汽油行驶的英里数，显示方式是在小数点右侧显示一个数字。然后，根据1加仑约等于3.785升，1英里约等于1.609公里的规则，它应该把每加仑英里数转换成每100公里的升数，并显示结果，显示方式是在小数点右侧显示一个数字。用符号常量表示两个转换系数（使用const 或#define）。
#define G_L 3.785
#define M_K 1.609
float miles;
float gallon;
scanf("%f",&miles);
scanf("%f",&gallons);
printf("%.1f\n",miles / gallons);
printf("%.1f\n",(gallons*G_L) / (miles*M_K) * 100);
```



## 第 5 章  运算符、表达式和语句

#### 1️⃣ ==运算符==

**sizeof 运算符与 size_t 类型**

```cpp
// sizeof 运算符以字节为单位返回操作数的大小
// sizeof 返回 size_t 类型的值，是一个无符号整数类型
```

**++ 运算符与 -- 运算符**

```cpp
// 如果一个变量出现在同一函数的多个参数中或多次出现在一个表达式中，不要使用 ++ 和 -- 运算符
n = 3;
y = n++ + n++; // n = 5,但是 y 的值不确定

```



#### 2️⃣ ==类型转换==

| **类型**             | **表达式计算时**         | **传参给 printf 等变参函数** | **传参给普通函数 (有原型)** |
| -------------------- | ------------------------ | ---------------------------- | --------------------------- |
| **`char` / `short`** | **自动提升**为 `int`     | **自动提升**为 `int`         | **自动提升**为 `int`        |
| **`float`**          | 只有遇到 `double` 才提升 | **自动提升**为 `double`      | **保持** `float`            |

**强制类型转换**

```cpp
int mice;
mice = (int)1.6 + (int)1.7;
```



#### 3️⃣ ==编程练习==

```cpp
//1．编写一个程序。将用分钟表示的时间转换成以小时和分钟表示的时间。使用#define或者const来创建一个代表60的符号常量。使用while循环来允许用户重复键入值，并且当键入一个小于等于0时终止循环。
#define MIN_PER_HOUR 60
int min,hour,left;
scanf("%d",&min);
while(min > 0){
    hour = min / MIN_PER_HOUR;
    left = min % MIN_PER_HOUR;
    printf("%d %d\n",hour,left);
    scanf("%d",min);
}
-------------------------------------------------------------------------------
//2. 编写一个程序，此程序要求输入一个整数，然后打印出从输入的值到比输入的值大10的所有整数值（也就是说，如果输入为5，那么输出就从5到15）。要求在各个输出值之间用空格、制表符或换行符分开。
int num;
int i = 0;
scanf("%d",&num);
while(i <= 10){
    printf("%d",num++);
    i++;
}
-------------------------------------------------------------------------------
//3. 编写一个程序，该程序要求用户输入天数，然后将该值转换为周数和天数。例如，此程序将把18天转换成2周4天。用下面的格式显示结果：18 days are 2 weeks, 4 days.使用一个while循环让用户重复输入天数;当用户输入一个非正数（如0或-20）时，程序将终止循环。
#define DAY_PER_WEEK 7
int day,week,left;
scanf("%d",&day);
while(day > 0){
    week = day / DAY_PER_WEEK;
    left = day % DAY_PER_WEEK;
    printf("%d %d\n",week,left);
    scanf("%d",day);
}
-------------------------------------------------------------------------------
//4. 编写一个程序让用户按厘米输入一个高度值，然后，程序按照厘米和英尺英寸显示这个高度值。允许厘米和英寸的值出现小数部分。程序允许用户继续输入，直到用户输入一个非正的数值。程序运行的示例如下所示：Enter a height in centimeters: 182  182.0 cm =5 feet，11.7 inches
#define CM_PER_INCH 2.54
#define INCH_PER_FEET 12
float cm;
int feet;
float inches;
scanf("%f",&cm);
while(cm > 0){
    inches = cm / CM_PER_INCH;
    feet = inches / INCH_PER_FEET;
    inches = inches - feet * INCH_PER_FEET;
    printf("%d %f",feet,inches);
    scanf("%f",&cm);
}
-------------------------------------------------------------------------------
//5. 编写一个程序计算如果您第一天得到$1，第二天得到$2，第三天得到$3，以此类推，您在20天里会挣多少钱。修改该程序，目的是您能交互地告诉程序计算将进行到哪里。也就是说，用一个读入的变量来代替20。
int sum = 0;
int n = 0;
int count = 0;
scanf("%d",&n);
while(count < n){
    count++;
    sum = sum + count;
}
printf("%d",sum);
-------------------------------------------------------------------------------
//6. 现在修改编程练习5中的程序，使它能够计算整数平方的和，如果您第一天得到$1，第二天得到$4，第三天得到$9，以此类推您将得到多少钱。
int sum = 0;
int n = 0;
int count = 0;
scanf("%d",&n);
while(count < n){
    count++;
    sum = sum + count*count;
}
printf("%d",sum);
-------------------------------------------------------------------------------
//7．编写一个程序，该程序要求输入一个float型数并打印该数的立方值。使用您自己设计的函数来计算该值的立方并且将它的立方打印出来。main()程序把输入的值传递给该函数。
void showCube(float num); // 函数声明
int main(){
    float num;
	scanf("%f",&num);
	showCube(num);
    return 0;
}
void showCube(float num){
    printf("%f",num * num * num);
}
-------------------------------------------------------------------------------
//8. 编写一个程序，该程序要求用户输入一个华氏温度。程序以double类型读入温度值，并将它作为一个参数传递给用户提供的函数Temperatures()。该函数将计算相应的摄氏温度和绝对温度，并以小数点右边有两位数字的精度显示这三种温度。它应该用每个值所代表的温度刻度来标识这3个值。下面是将华氏温度转换成摄氏温度的方程：Celsius = 1.8 * Fahrenheit + 32.0;通常用在科学上的绝对温度的刻度是0代表绝对零，是可能温度的下界。下面是将摄氏温度转换为绝对温度的方程：Kelvin =Celsius + 273.16;Temperatures()函数使用const来创建代表该转换里的3个常量的符号。main()函数将使用一个循环来允许用户重复地输入温度，当用户输入q或其他非数字值时，循环结束。
void Temperatures(float tem);
int main(){
    float tem;
    // scanf()函数返回成功读入的项目个数，如果他没有读取任何项目(当他期望一个数字而你输入了一个非数字字符串时),scanf()会返回值0;当他检测到文件结尾，他会返回EOF(-1)
    while(scanf("%f",&tem) == 1){
        Temperatures(tem);
    }
    return 0;
}
void Temperatures(float tem){
    const float F_C_1 = 1.8;
    const float F_C_2 = 32.0;
    const float C_K = 273.16;
    float c_t,k_t;
    c_t = F_C_1 * tem + F_C_2;
    k_t = c_t + C_K;
    printf("%f %f %f",tem,c_t,k_t);
    
}
```



## 第 6 章 C控制语句：循环

#### 1️⃣ ==while 循环==

```cpp
int main(){
    long num;
    long sum = 0L;
    int status;
    status = scanf("%ld",&num);
    while(status == 1){
        sum = sum + num;
        status = scanf("%ld",&num);
    }
    printf("%ld",sum);
    return 0;
}
```



#### 2️⃣ ==for 循环==

```cpp
// 第一个表达式进行初始化，它在for循环开始的时候执行一次。
// 第二个表达式是判断条件，在每次执行循环之前都要对它进行求值，表达式为空时表示真，当表达式为假时，循环就结束了。
// 第三个表达式进行更新，它在每次循环结束时进行计算。
const int NUMBER =22:
int count:
for (count = 1; count <= NUMBER; count++){
    printf ("Be my Valentine!\n"):
}

// 第2个表达式为空时表示真，死循环
for(; ; )
    
// 逗号运算符将两个表达式链接为一个表达式，并保证左边的表达式先计算，整个表达式的值为右边表达式的值
for(step = 2,fargo = 0;fargo < 1000;step *= 2)
    fargo += step;
```



#### 3️⃣ ==数组==

```cpp
#define SIZE 10
int main(){
    int scores[SIZE];
    int sum = 0;
    float average;
    for(int i = 0;i < SIZE;i++){
        scanf("%d",&scores[i]);
    }
    for(int i = 0;i < SIZEli++){
        printf("%4d",scores[i]);
    }
    for(int i = 0;i < SIZE;i++){
        sum += scores[i];
    }
    average = (float)sum / SIZE;
    printf("总分：%d,平均分:%.2f",sum,average);
    return 0;
}
```



#### 4️⃣ ==编程练习==

```cpp
// 1．编写一个程序，创建一个具有 26个元素的数组，并在其中存储 26个小写字母。并让该程序显示该数组的内容。
char vec[26];
for(char ch = 'a';ch <= 'z';ch++){
    vec[ch-'a'] = ch;
}
for(int i = 0;i < 26;i++){
    printf("%2c",vec[i]);
}

//2．使用嵌套循环产生下列图案：
//s
//ss
//sss
//ssss
//sssss
for(int i = 0;i < 5;i++){
    for(int j = 0;j <= i;j++){
        printf("s");
    }
    printf("\n");
}

//3. 使用嵌套循环产生下列图案：
//F
//FE
//FED
//FEDC
//FEDCB
//FEDCBA
for (int i = 0; i < 6; i++){
	char ch = 'F';
    for (int j = 0; j <= i; j++){
        printf("%c", ch--);
    }
    printf("\n");    
}

//4. 让程序要求用户输入一个大写字母，使用嵌套循环产生像下面这样的金字塔图案：
//    A
//   ABA
//  ABCBA
// ABCDCDA
//ABCDEDCBA
//这种图案要扩展到用户输入的字符。例如，前面的图案是在输入E时需要产生的。提示：使用一个外部循环来处理行，在每一行中使用三个内部循环，一个处理空格，一个以升序打印字母，一个以降序打印字母。
char ch;
scanf("%c", &ch);
int loopnum = ch - 'A' + 1;
for (int i = 0; i < loopnum; i++){
    for (int j = i; j < loopnum - 1; j++){
        printf(" ");
    }
    for (int k = 0; k <= i; k++){
        printf("%c", 'A' + k);
    }
    for (int k = i - 1; k >= 0; k--){
        printf("%c", 'A' + k);
    }
    printf("\n");
}

//5．编写一个程序打印一个表，表的每一行都给出一个整数、它的平方以及它的立方。要求用户输入表的上限与下限。使用一个 for 循环。
int low, up;
scanf("%d%d", &low, &up);
for (int i = low; i <= up; i++){
    printf("%10d%10d%10d\n", i, i * i, i * i * i);
}

//6．编写一个程序把一个单词读入一个字符数组，然后反向打印出这个词。提示：使用strlen()计算数组中最后一个字符的索引。
#include <string.h>
char word[100];
scanf("%s",word);
int len = strlen(word);
for(int i = len - 1;i >= 0;i--){
    printf("%c\n",word[i]);
}

//7．编写一个程序，要求输入两个浮点数，然后打印出用二者的差值除以二者的乘积所得的结果。在用户键入非数字的输入之前程序循环处理每对输入值。
float a,b;
while(scanf("%f%f",&a,&b) == 2){
    printf("%f\n",(a - b) / (a * b));
}

//9. 编写一个程序，要求用户输入下限整数和一个上限整数，然后，依次计算从下限到上限的每一个整数的平方的加和，最后显示结果。程序将不断提示用户输入下限整数和上限整数并显示出答案，直到用户输入的上限整数等于或小于下限整数为止。程序运行的结果示例应该如下所示：
//Enter lower and upper integer limits: 5 9
//The sums of the squares from 25 to 81 is 255
//Enter next set of limits: 3 25
//The sums of the squares from 9 to 625 is 5520
//Enter next set of limits:5 5
//Done
int low,up;
scanf("%d%d",&low,&up);
while(low < up){
    int sum = 0;
    for(int i = low;i <= up;i++){
        sum += i*i;
    }
    printf("The sums of the squares from %d to %d is %d\n",low*low,up*up,sum);
    scanf("%d%d",&low,&up);
}
printf("Done");

//10. 编写一个程序把8个整数读入一个数组中，然后以相反的顺序打印它们。
#define SIZE 8
int vec[SIZE];
for (int i = 0; i < SIZE; i++){
    scanf("%d", &vec[i]);
}
for (int i = SIZE - 1; i >= 0; i--){
    printf("%d\n", vec[i]);
}

//11. 考虑这两个无限序列：
// 1.0 +1.0/2.0 + 1.0/3.0 + 1.0/4.0 + ...
// 1.0 -1.0/2.0 + 1.0/3.0 - 1.0/4.0 + ...
// 编写一个程序来计算这两个序列不断变化的总和，直到达到某个次数。让用户交互地输入这个次数。看看在20次、100 次和500次之后的总和。是否每个序列都看上去要收敛于某个值？提示：奇数个-1相乘的值为-1，而偶数个-1相乘的值为1。
double sum1 = 0.0;
double sum2 = 0.0;
int count;
int sign = 1;
scanf("%d",&count);
for(int i = 0;i < count;i++){
    sum1 += 1.0 / (i + 1);
    sum2 += (1.0 * sign) / (i + 1);
    sign = -sign;
}
printf("%10.5f10.5f\n",sum1,sum2);

//12. 编写一个程序，创建一个8个元素的 int 数组，并且把元素分别设置为2的前8次幂，然后打印出它们的值。使用for循环来设置值.
const int SIZE = 8;
int vec[SIZE];
vec[0] = 1;
for(int i = 1;i < SIZE;i++){
    vec[i] = vec[i-1] * 2;
}
for(int i = 0;i < SIZE;i++){
    printf("%d\n",vec[i]);
}

//13. 编写一个程序，创建两个8元素的 double 数组，使用一个循环来让用户键入第一个数组的8个元素的值。程序把第二个数组的元素设置为第一个数组元素的累积和。例如，第二个数组的第4个元素应该等于第一个数组的前4个元素的和，第二个数组的第5个元素应该等于第一个数组的前5个元素的和。
const int len = 8;
double vec1[len];
double vec2[len];
for(int i = 0;i < len;i++){
    scanf("%lf",&vec1[i]);
}
vec2[0] = vec1[0];
for(int i = 1;i < len;i++){
    vec2[i] = vec2[i-1] + vec1[i];
}
for(int i = 0;i < len;i++){
    printf("%8.2f",vec1[i]);  
}
printf("\n");
for(int i = 0;i < len;i++){
    printf("%8.2f",vec2[i]);  
}

//14. 编写一个程序读入一行输入，然后反向打印该行。您可以把输入存储在一个char数组中；假定该行不超过 255个字符。回忆一下，您可以使用具有%c说明符的scanf（）从输入中一次读入一个字符，而且当您按下回车键时会产生换行字符（\n）。
const int LEN = 10;
char str[255];
for(int i = 0;i < LEN;i++){
    scanf("%c",&str[i]);
}
for(int i = LEN - 1;i >= 0;i--){
    printf("%c",str[i]);
}

//15. Daphne以 10%的单利息投资了 100 美元（也就是说，每年投资赢得的利息等于原始投资10%）。Deirdre则以每年5%的复合利息投资了100美元（也就是说，利息是当前结余的5%，其中包括以前的利息）。编写一个程序，计算需要多少年 Deirdre 的投资额才会超过 Daphne，并且显示出到那时两个人的投资额。
const double init = 100.0;
double sum1 = init;
double sum2 = init;
int year = 0;
while(sum1 >= sum2){
    sum1 += init * 0.1;
    sum2 = 1.05 * sum2;
    year++;
}
printf("第%d年,sum1 = %.2lf,sum2 = %.2lf\n",year,sum1,sum2);

//16. Chuckie Lucky赢了100 万美元，他把它存入一个每年赢得 8%的帐户。在每年的最后一天，Chuckie取出 10万美元。编写一个程序，计算需要多少年Chuckie 就会清空他的帐户。
double init = 1000000;
int year = 0;
while(init > 0){
    init = init * 1.08;
    init = init - 100000;
    year++;
}
printf("第%d年，破产",year);

```



## 第 7 章 C控制语句：分支和跳转

#### 1️⃣ ==if - else 语句==

```cpp
// 统计寒冷的天数
const int FREEZING = 0;
float temperature;
int cold_days = 0;
int all_days = 0;
while(scanf("%f",&temperature) == 1){
    all_days++;
    if(temperature < FREEZING){
        cold_days++;
    }
}
if(all_days != 0){
    printf("%d days total,%.1f%% were below freezing.\n",all_days,100.0*(float)cold_days / all_days);
}else{
    printf("No date entered\n");
}
```



#### 2️⃣ ==getchar() 与 putchar()==

```cpp
// 改变输入，只保留其中的空格
#define SPACE ' '
char ch;
ch = getchar(); // 无参数，等同于scanf("%c",&ch);
while(ch != '\n'){ // 一行未结束时，可替换为 while((ch = getchar()) != '\n')
    if(ch == SPACE){
        putchar(ch); // 等同于printf("%c",ch)
    }else{
        putchar(ch + 1);
    }
    ch = getchar();
}
putchar(ch); // 打印换行符

-------------------------------------------------------------------------------
// ctype.h 头文件
isalnum(ch) // 字母或数字,返回 bool 类型
isalpha(ch) // 纯字母
isdigit(ch) // 纯数字
islower()
isupper()
ch1 = tolowwer(ch) // 不改变ch
ch2 = toupper(ch) // 不改变ch
```



#### 3️⃣ ==逻辑运算符 && || ！==

```cpp
// 短路原则：如果左边的结果已经能决定全局，右边的代码直接被无视（不执行）
if (index < MAX && array[index] == 10) { 
    // 如果 index >= MAX，左边为假，&& 直接停下
    // 不会执行 array[index]，从而避免了非法内存访问（段错误）
}
if (1 || some_heavy_function()) {
    // 1 是真，|| 发现只要有一个真就行了。
    // some_heavy_function() 根本不会被运行，节省了资源。
}

```



#### 4️⃣ ==条件运算符 ？ ：==

```cpp
// 一种表示 if-else 的简写方式
x = (y < 0) ? -y : y; // 如果y<0，那么x=-y，否则x=y
```



#### 5️⃣ ==continue 与 break==

```cpp
// continue:用于 while 循环与 for 循环，运行到该语句时，将导致剩余的迭代部分被忽略，开始下一次迭代
const double MIN = 0.0;
const double MAX = 100.0;
double min = MIN;
double max = MAX;
int n = 0;
double score;
double total = 0.0;
while(scanf("%lf",&score) == 1){
    if(score < MIN || score > MAX){
        printf("无效分数\n")
        continue;
    }
    total += score;
    n++;
    min = (score < min) ? score : min;
    max = (score > max) ? score : max;
}
if(n > 0){
    printf("aversge:%lf,min:%lf,max:%lf\n",total / n,min,max);
}else{
    printf("无有效成绩");
}

// continue 被执行时，下一个被求值的表达式是循环判断条件count<10
count = 0;
while(count < 10){
    ch = getchar();
    if(ch == '\n'){
        continue;
    }
    putchar(ch);
    count++;
}
// continue被执行时，先求更新表达式的值count++，再求循环判断表达式的值count<10
for(count = 0;count < 10;count++){
    ch = getchar();
    if(ch == '\n'){
        continue;
    }
    putchar(ch);
}
----------------------------------------------------------------------------------
// break:循环中的break语句导致程序终止包含它的循环，进入程序的下一阶段
float length,width;
while(scanf("%f",&length) == 1){
    printf("length is %f\n",length);
    if(scanf("%f",&width) != 1){
        break;
    }
    printf("width is %f",width);
    printf("area is %f",width * length);
}
```



#### 6️⃣ ==switch 语句==

```cpp
// 使用switch语句
// switch 只能判断整型（包括 int, char, enum），不能用来判断字符串 "" 或者浮点数 float
char ch;
while((ch = getchar()) != '#'){
    if(ch == '\n'){
        continue;
    }
    ch = tolower(ch);
    switch(ch){
        case 'a':
            printf("ant\n");
            break;
        case 'b':
            printf("bee\n");
            break;
        default:
            printf("that's a stumper\n");
    }
}
```



#### 7️⃣ ==编程练习==

```cpp
//1. 编写一个程序。该程序读取输入直到遇到#字符，然后报告读取的空格数目、读取的换行符数目以及读取的所有其他字符数目。
char ch;
int n_space = 0, n_line = 0, n_char = 0;
while((ch = getchar()) != '#'){
    if(ch == ' '){
        n_space++;
    }else if(ch == '\n'){
        n_line++;
    }else{
        n_char++;
    }
}
printf("n_space:%d,n_line:%d,n_char:%d",n_space,n_line,n_char);
/*输入如下：
hello world
good morning
i love you weiwei
#
🌟🌟🌟🌟🌟输入的全部内容在缓冲区里其实长这样：h e l l o _ w o r l d \n g o o d _ m o r n i n g \n i _ l o v e _ y o u _ w e i w e i \n #
C 语言的行缓冲输入
①输入时数据先存在临时区（此时可退格修改），只有按下回车 (\n)，整行数据才涌入缓冲区，唤醒 getchar() 开始工作。
②虽然是按行发送，但 while 循环是一个字符跑一圈。回车符 (\n) 也是字符，它会实实在在地进入循环被处理。
③一旦 getchar() 读到终止符（如 #），while 循环即刻停止。如果 # 后面还有字符，它们会留在缓冲区，导致下一个输入函数“误读”。
*/
-------------------------------------------------------------------------------
//2. 编写一个程序。该程序读取输入直到遇到#字符。使程序打印每个输入的字符以及它的十进制ASCII码。每行打印8个字符/编码对。建议：利用字符计数和模运算符（%）在每8个循环周期时打印一个换行符。
char ch;
int count = 0;
while ((ch = getchar()) != '#') {
    if (ch == '\n') continue; // 跳过换行符，否则换行符也会占一个 count 计数
    printf("%c:%d  ", ch, ch); // 打印字符及其对应的 ASCII 码
    count++;
    if (count % 8 == 0){
        printf("\n");
    }
}
--------------------------------------------------------------------------------- 
//3. 编写一个程序。该程序读取整数，直到输入0。输入终止后，程序应该报告输入的偶数（不包括0）总个数、偶数的平均值，输入的奇数总个数以及奇数的平均值。
int num;
int sum_odd = 0;
int sum_even = 0;
int cnt_odd = 0;
int cnt_even = 0;
float average_odd = 0.0f;
float average_even = 0.0f;
scanf("%d",&num);
while(num != 0){
    if(num % 2 == 0){
        sum_even += num;
        cnt_even++;
    }else{
        sum_odd += num;
        cnt_odd++;
    }
    scanf("%d",&num);
}
if(cnt_even){
    printf("cnt_even:%d,average_even:%f\n",cnt_even,(float)sum_even / cnt_even);
}
if(cnt_odd){
    printf("cnt_odd:%d,average_odd:%f\n",cnt_odd,(float)sum_odd / cnt_odd);
}
--------------------------------------------------------------------------------
//4. 利用if-else语句编写程序读取输入，直到#。用一个感叹号代替每个句号，将原有的每个感叹号用两个感叹号代替，最后报告进行了多少次替代。
char ch;
int cnt = 0;
while((ch = getchar()) != '#'){
    if(ch == '.'){
        putchar('!');
        cnt++;
    }else if(ch == '!'){
        putchar('!');
        putchar('!');
        cnt++;
    }else{
        putchar(ch);
    }
}
printf("%d",cnt);
--------------------------------------------------------------------------------
//6. 编写一个程序读取输入，直到#，并报告序列ei出现的次数。此程序必须要记住前一个字符和当前的字符。用诸如“Receive your eieio award.”的输入测试它。
char ch;
char pre = '#';
int cnt = 0;
while((ch = getchar()) != '#'){
    if(ch == 'i' && pre == 'e'){
        cnt++;
    }
    pre = ch;
}
printf("%d\n",cnt);
--------------------------------------------------------------------------------
//7. 编写程序，要求输入一周中的工作小时数，然后打印工资总额、税金以及净工资。作如下假设：
//a.基本工资等级=10.00美元/小时
//b.加班（超过40小时）=1.5倍的时间
//c.税率 前300美元为15%,下一个150美元为20%,余下的为25%
//用#define定义常量，不必关心本例是否符合当前的税法。
#define BASE 10.0
#define OVERTIME_THRESHOLD 40.0
#define TAX_LIMIT_1 300.0
#define TAX_LIMIT_2 450.0
#define RATE_1 0.15
#define RATE_2 0.20
#define RATE_3 0.25
float hours;
float total;
float tax;
float income;
scanf("%f",&hours);
if(hours <= OVERTIME_THRESHOLD){
    total = hours * BASE;
}else{
    total = OVERTIME_THRESHOLD * BASE + (hours - OVERTIME_THRESHOLD) * 1.5 * BASE;
}
if(total <= TAX_LIMIT_1){
    tax = total * RATE_1;
}else if(total <= TAX_LIMIT_2){
    tax = TAX_LIMIT_1 * RATE_1 + (total - TAX_LIMIT_1) * RATE_2;
}else{
    tax = TAX_LIMIT_1 * RATE_1 + 150 * RATE_2 + (total - TAX_LIMIT_2) * RATE_3;
}
income = total - tax;
printf("total:%.2f,tax:%.2f,income:%.2f",total,tax,income);
-------------------------------------------------------------------------------   
//9. 编写一个程序，接受一个整数输入，然后显示所有小于或等于该数的素数。
#include <stdbool>
int num;
scanf("%d",&num);
for(int i = 2;i <= num;i++){//i=2: 内层循环 j*j <= 2 不成立，跳过。issu 为真，输出 2。
    bool issu = true;
    for(int j = 2;j * j <= i;j++){
        if(i % j == 0){
            issu = false;
            break;
        }
    }
    if(issu){
        printf("%d\n",i);
    }
}
--------------------------------------------------------------------------------
//10. United States Federal Tax Schedule分为4类，每类有两个等级。下面是其摘要：
//种类     			税金
//单身     			前17,850美元按15%，超出部分按28%
//户主     			前23,900美元按15%，超出部分按28%
//己婚，共有 		  前29,750美元按15%，超出部分按28%
//己婚，离异		 	  前14,875美元按15%，超出部分按28%
//例如，有20000美元应征税收入的单身雇佣劳动者应缴税金0.15×17850+0.28×（20000-17850）。编写一个程序，让用户指定税金种类和应征税收入，然后计算税金。使用循环以便用户可以多次输入。
char level;
double total, tax, limit;
printf("请输入类别 (A-D) 和金额 (输入 # 退出):\n");
// " %c" 前面的空格极其重要：它会跳过缓冲区里的所有空白符（如回车）
while (scanf(" %c", &level) == 1 && level != '#') {
    if (scanf("%lf", &total) != 1) break;
    // 核心简洁化：switch 只负责找“起征点”
    switch (level) {
        case 'A': limit = 17850.0; break;
        case 'B': limit = 23900.0; break;
        case 'C': limit = 29750.0; break;
        case 'D': limit = 14875.0; break;
        default: 
            printf("类别错误，请重新输入。\n");
            continue;
    }
    // 统一计算逻辑，不再重复写 if-else
    if (total <= limit) {
        tax = total * 0.15;
    } else {
        tax = limit * 0.15 + (total - limit) * 0.28;
    }
    printf("类别: %c | 税金: %.2f\n\n", level, tax);
}
```



## 第 8 章 字符输入/输出和输入确认

#### 1️⃣ ==缓冲区==

```cpp
/* 
缓冲分为两类:完全缓冲I/O和行缓冲I/O;
完全缓冲:缓冲区满时被清空,通常出现在文件输入中,大小通常为512字节或4096字节;
行缓冲:换行字符\n将清空缓冲区,键盘输入是行缓冲,按下回车将清空缓冲区;
*/
```



#### 2️⃣ ==终止键盘输入==

```cpp
/*
C 对待输入和输出设备与其对待普通文件相同,键盘和显示设备作为每个C程序自动打开的文件来对待。键盘输入由一个被称为stdin 的流表示，而到屏幕上的输出由一个被称为stdout的流表示。getchar,putchar,printf和scanf都是标准I/O包的成员，这些函数同这两个流打交道。
结论就是可以使用与处理文件相同的技术来处理键盘输入。例如，读取文件的程序需要一种方法来检测文件的结尾，以了解停止读取的位置。因此，C输入函数装备有一个内置的文件尾检测器。因为键盘输入是像文件一样被看待的，所以也应该能使用该文件尾检测器来终止键盘输入。
-----------------------------------------------------------------------------------
操作系统需要某种方式来判断文件的起始和结束的位置(例如存储文件大小信息,当读取的字符等于文件大小说明读完了),对于这些方法,C的处理方式是让getchar()和scanf()函数在到达文件结尾时返回一个特殊值，赋予该值为EOF(#define EOF -1)。
*/

// 键盘模拟文件结尾条件的方法：ctrl+D
#include <stdio.h>
int main(){
    int ch; // 无符号char类型取值0-255,不能取到EOF
    while((ch = getchar()) != EOF){ 
        putchar(ch);
    }
    return 0;
}
```



#### 3️⃣ ==重定向和文件==

```cpp
// 之前:读取键盘输入，输出到屏幕上
// 输入重定向:使用文档作为输入，输出到屏幕上
echo_eof < txt.demo1 // 命令行输入,将txt_demo1文件中的内容输出到屏幕上,echo_eof不关心输入来自文件还是键盘
// 输出重定向:使用键盘作为输入，但输出到文档中
echo_eof > txt.demo1
```



#### 4️⃣ ==使用缓冲输入==

```cpp
// 猜数游戏
int guess = 1;
while(getchar() != 'y'){
    printf("well,is it %d?\n",++guess);
}
printf("i know i can do it\n");
//运行示例: 
//n
//well,is it 2?
//well,is it 3?
//n
//well,is it 4?
//well,is it 5? // 原因：getchar()读取了n和换行\n，做了两次处理
// 猜数游戏-优化
int guess = 1;
while(getchar() != 'y'){
    printf("well,is it %d?\n",++guess);
    while(getchar() != '\n'){ // 清空缓冲区
        continue;
    }
}
printf("i know i can do it\n");
--------------------------------------------------------------------------------
// 混合输入数字与字符,并打印
int ch;
int rows,cols;
printf("输入一个字符和两个整数");
while((ch = getchar()) != '\n'){
    scanf("%d %d",&rows,&cols);
    printf("%c,%d,%d",ch,rows,cols);
}
printf("bye");
//运行示例：
// c 4 5
// c,4,5
// bye 
// 原因：getchar()读取到了上一次输入末尾的换行符\n，导致程序退出
// 混合输入数字与字符,并打印-优化
int ch;
int rows,cols;
printf("输入一个字符和两个整数");
while((ch = getchar()) != '\n'){
    scanf("%d %d",&rows,&cols);
    printf("%c,%d,%d",ch,rows,cols);
    while(getchar() != '\n'){ // 清空缓冲区
        continue;
    }
}
printf("bye");
```



#### 5️⃣ ==编程练习==

```cpp
// 下面的一些程序要求输入以EOF终止。如果您的操作系统难以使用或不能使用重定向，则使用一些其他的判断来终止输入，例如读取&字符。
//1. 设计一个程序，统计从输入到文件结尾为止的字符数。
int cnt = 0;
int ch;
while((ch = getchar()) != EOF){
    cnt++;
}
printf("%d",cnt);
--------------------------------------------------------------------------------
//2. 编写一个程序，把输入作为字符流读取，直到遇到EOF。令该程序打印每个输入字符及其ASCII编码的十进制值。注意在ASCII序列中空格字符前面的字符是非打印字符，要特殊处理这些字符。如果非打印字符是换行符或制表符，则分别打印\n或\t。否则使用控制字符符号。例如，ASCII的1是Crl+A，可以显示为^A。注意 A的 ASCII值是Ctrl+A 的值加 64。对其他非打印字符也保持相似的关系。除去每次遇到一个换行符时就开始一个新行之外，每行打印10对值。
int ch;
int cnt = 0;
while((ch = getchar()) != EOF){
    if(ch < ' '){
        if(ch == '\n'){
            printf("\\n:%d",ch);
        }else if(ch == '\t'){
            printf("\\t:%d",ch);
        }else{
            printf("^%c:%d",ch + 64,ch);
        }
    }else{
        printf("%c:%d",ch,ch);
    }
    if(++cnt % 10 == 0){
        printf("\n");
    }
}
-------------------------------------------------------------------------------
//3. 编写一个程序，把输入作为字符流读取，直至遇到EOF。令其报告输入中的大写字母个数和小写字母个数。假设小写字母的数值是连续的，大写字母也是如此。或者你可以使用ctype.h库中的合适的函数来区分大小写。
#include <ctype.h>
int lower = 0;
int upper = 0;
int ch;
while((ch = getchar()) != EOF){
    if(islower(ch)){
        lower++;
    }else if(isupper(ch)){
        upper++;
    }
}
printf("lower:%d,upper:%d",lower,upper);
```



## 第 9 章 函数

#### 1️⃣ ==函数类型==

```cpp
// 为了正确使用函数，程序在首次调用函数之前需要知道函数的类型；
// 方法1是在首次调用之前进行完整的函数定义，方法2(常用)是预先对函数进行声明；
#include <stdio.h>
int imin(int a,int b); // 函数声明
int main(){
    ...
    return 0;
}
```



#### 2️⃣ ==递归==

```cpp
// 1.调用自身 
// 2.结束条件
void up_down(int n);
int main(){
    up_down(1);
    return 0;
}
void up_down(int n){
    printf("level %d:n location %p\n",n,&n);
    if(n < 4){
        up_down(n+1);
    }
    printf("level %d:n location %p\n",n,&n);
}
// 输出示例：
//level 1:n location 0x7fffffffdc7c
//level 2:n location 0x7fffffffdc5c
//level 3:n location 0x7fffffffdc3c
//level 4:n location 0x7fffffffdc1c
//level 4:n location 0x7fffffffdc1c
//level 3:n location 0x7fffffffdc3c
//level 2:n location 0x7fffffffdc5c
//level 1:n location 0x7fffffffdc7c
//可以假想进行了一系列函数调用,即使用fun1()调用fun2(),fun2()调用 fun3(),fun3()调用 fun4();fun4()执行完后,fun3()会继续执行,而 fun3()执行完后,开始继续执行fun2(),最后fun2()返回到 fun1()中并执行后续代码。递归过程也是如此，只不过fun1()、fun2()、fun3()和fun4()是相同的函数;
//递归函数中，位于递归调用前的语句和各级被调函数有相同的执行顺序；递归调用后的语句和各级被调函数执行顺序相反；
```



#### 3️⃣ ==多个源代码文件程序的编译==

```cpp
// 头文件的使用
// 1.函数声明
// 2.常量定义

/*usehotel.c,旅馆房间收费程序,与hotel.c一起编译*/
#include <stdio.h>
#include "hotel.h"
int main(){
    int nights;
    double hotel_rate;
    int code;
    while((code = menu()) != QUIT){
        switch(code){
            case 1:
                hotel_rate = HOTEL1;
                break;
            case 2:
                hotel_rate = HOTEL2;
                break;
            case 3:
                hotel_rate = HOTEL3;
                break;
            case 4:
                hotel_rate = HOTEL4;
                break;
            default:
                break;        
        }
        nights = getnights();
        showprice(hotel_rate,nights);
    }
    printf("bye");
    return 0;
}
/*hotel.c*/
#include <stdio.h>
#include "hotel.h"
int menu(){
    int code;
    int status;
    printf("%s\n",STARTS);
    printf("enter the number of the desired hotel:\n");
    printf("1.A			2.B\n");
    printf("3.C			4.D\n");
    printf("5.Quit\n");
    printf("%s\n",STARTS);
    while((status = scanf("%d",&code)) != 1 || (code < 1 || code > 5)){
        if(status != 1){
            scanf("%*s"); // 跳过前置空白，读取一个字符串(直到空白)，然后丢掉
        }
        printf("enter an integer from 1-5.\n");
    }
    return code;
}
int getnights(){
    int nights;
    printf("how many nights are needed?\n");
    while(scanf("%d",&nights) != 1){
        scanf("%*s");
        printf("please enter an integer such as 2.\n");
    }
    return nights;
}
void showprice(double rate,int nights){
    int n;
    double total = 0.0;
    double facter = 1.0;
    for(n = 1;n <= nights;n++,facter *= DISCOUNT){
        total += rate * facter;
    }
    printf("total cost:%.2lf.\n",total);
}
/*hotel.h*/
#define HOTEL1 80.0
#define HOTEL2 125.0
#define HOTEL3 155.0
#define HOTEL4 200.0
#define STARTS "************************************************"
#define QUIT 5
#define DISCOUNT 0.95
int menu();
int getnights();
void showprice(double rate,int nights);
```



#### 4️⃣ ==指针简介==

```cpp
// 取地址运算符&:后跟一个变量名时，&给出该变量的地址
// 解引用运算符*:后跟一个指针名或地址时，*给出存储在被指向地址中的数值
---------------------------------------------------------------------------------
// 指针是一个数值为地址的变量
// 如果你将某个指针变量命名为ptr,可以使用如下语句:
ptr = &pooh; // ptr与&pooh的区别在于前者是一个变量，后者是一个常量
```



#### 5️⃣ ==编程练习==

```cpp
//1. 设计函数min(x,y)，返回两个 double 数值中较小的数值，同时用一个简单的驱动程序测试该函数。
double min(double x,double y){
    return (x < y) ? x : y;
}
-------------------------------------------------------------------------------
//2. 设计函数chline(ch,i,j),实现指定字符在i列到j列的输出，并用一个简单的驱动程序测试该函数。
void chline(char ch,int i,int j){
    for(int cnt = 1;cnt < i;cnt++){
        printf(" ");
    }
    for(int cnt = i;cnt <= j;cnt++){
        printf("%c",ch);
    }
}
---------------------------------------------------------------------------------
//3. 编写一个函数。函数的3个参数是一个字符和两个整数。字符参数是需要输出的字符。第一个整数说明了在每行中该字符输出的个数，而第二个整数指的是需要输出的行数。编写一个调用该函数的程序。
void show(char ch,int n,int rows){
    for(int i = 0;i < rows;i++){
        for(int j = 0;j < n;j++){
            printf("%c",ch);
        }
        printf("\n");
    }
} 
--------------------------------------------------------------------------------
//4. 两数值的谐均值可以这样计算：首先对两数值的倒数取平均值，最后再取倒数。编写一个带有两个double参数的函数，计算这两个参数的谐均值。
double harmonic_mean(double x, double y) {
    if (x == 0 || y == 0 || (x + y) == 0) { // 安全检查：如果 x+y 为 0 或者 x*y 为 0，谐均值没有意义
        printf("错误：输入值无法计算谐均值（存在0或互为相反数）。\n");
        return 0.0; 
    }    
    return (2.0 * x * y) / (x + y); // 使用简化公式：2xy / (x + y)
}
--------------------------------------------------------------------------------
//5. 编写并测试函数larger_of（)，其功能是将两个 double类型变量的数值替换成它们中的较大值。例如，larger_of(x，y)会把x 和y中的较大数值重新赋给变量x 和 y。
void larger_of(double* x,double* y){
    double temp = (*x > *y)?*x:*y;
    *x = temp;
    *y = temp;
}
--------------------------------------------------------------------------------
//6. 编写一个程序，使其从标准输入读取字符，直到遇到文件结尾。对于每个字符，程序需要检查并报告该字符是否是一个字母。如果是的话，程序还应报告该字母在字母表中的数值位置。例如，c和C的字母位置都是3。可以先实现这样一个函数：接受一个字符参数，如果该字符为字母则返回该字母的数值位置，否则返回-1。
#include <ctype.h>
int getPos(char ch){
    if(isalpha(ch)){
        ch = tolower(ch);
        return ch - 'a' + 1;
    }else{
        return -1;
    }
}
--------------------------------------------------------------------------------- 
//7. 在第6章“C控制语句：循环”的程序清单 6.20中，函数power（）的功能是返回一个 double 类型数的某个正整数次幂。现在改进该函数，使其能正确地计算负幂。同时，用该函数实现0的任何次幂为0，并且任何数值的0次幂为1。使用循环的方法编写该函数并在一个程序中测试它。
double power(double x,int n){
    if(n == 0) return 1.0;
    if(x == 0) return 0.0;
    double res = 1.0;
    int abs_n = (n > 0)?n:-n;
    for(int i = 0;i < abs_n;i++){
            res *= x;
    }
    if(n > 0){
        return res;
    }else{
        return 1.0/res;
    }
}
--------------------------------------------------------------------------------
//8. 使用递归函数重做练习7。
double r_power(double x, int n) {
    // 1. 基准情形 (Base Cases)
    if (n == 0) return 1.0;  // 任何数的 0 次幂
    if (x == 0) return 0.0;  // 0 的幂
    // 2. 处理负幂：将其转化为 1 / x^|n|
    if (n < 0) {
        return 1.0 / r_power(x, -n);
    }
    // 3. 递归步骤：x^n = x * x^(n-1)
    return x * r_power(x, n - 1);
}
-------------------------------------------------------------------------------- 
//10. 编写并测试一个函数Fibonacci（)，在该函数中使用循环代替递归完成斐波纳契数列的计算。
int getFibonacci(int n){
    if(n <= 2) return 1;
    int pre2 = 1;
    int pre1 = 1;
    int res = 0;
    for(int i = 3;i <= n;i++){
        res = pre2 + pre1;
        pre2 = pre1;
        pre1 = res;
    }
    return res;
}     
```



## 第 10 章 数组和指针

#### 1️⃣ ==数组==

```cpp
// 初始化
//1.前后都不省
int nums[5] = {10, 20, 30, 40, 50};
//2.前省后不省
int nums[] = {1, 3, 5, 7, 9}; // 编译器自动推断长度为 5
//3.前不省后省
int nums[5] = {1, 2}; // 实际内容：{1, 2, 0, 0, 0}
-------------------------------------------------------------------------------
// 对数组使用const，变为只读数组
const int days[MONTHS] = {31,28,31,30,31,30,31,31,30,31,30,31}; 
-------------------------------------------------------------------------------
// 指定数组大小
#define SIZE 10
int arr[SIZEl; // 符号整数常量,const定义的常量不行
double lots[144]; // 文字整数常量
int m = 8; float arr[m]; // C99标准的可变长度数组(VLA)
```



#### 2️⃣ ==二维数组==

```cpp
// 初始化
int board[2][3] = {{1, 2, 3},{4, 5, 6}};
int board[2][3] = {1, 2, 3, 4, 5, 6};
int board[][3] = {1, 2, 3, 4, 5, 6};
```



#### 3️⃣ ==数组和指针==

```cpp
// 数组名是该数组首元素的地址
flizny == &flizny[0] // 式子为真
------------------------------------------------------------------------------
// 指针+1的本质是“跨过一个对象”：地址增加的字节数等于指针指向类型的大小
#define SIZE 4
int main(){
    short dates[SIZE];
    short* pti;
    short index;
    double bills[SIZE];
    double* ptf;
    pti = dates;
    ptf = bills;
    for(index = 0;index < SIZE;index++){
        printf("pointer+%d = %10p %10p\n",index,pti+index,ptf+index);
	}  
    return 0;
}
// 运行示例:
pointer+0 = 0x7fffffffdc58 0x7fffffffdc60
pointer+1 = 0x7fffffffdc5a 0x7fffffffdc68
pointer+2 = 0x7fffffffdc5c 0x7fffffffdc70
pointer+3 = 0x7fffffffdc5e 0x7fffffffdc78
-------------------------------------------------------------------------------
dates + 2 == &date[2]; /* 相同的地址*/
*(dates + 2) == dates[2]; /* 相同的值 */   
*(dates + 2) /*dates 的第3个元素的值*/
*dates + 2 /*第1个元素的值和2相加*/
```



#### 4️⃣ ==函数、数组和指针==

```cpp
// 由于数组名就是数组首元素的地址，所以如果实际参数是一个数组名，那么形式参数必须是与之相匹配的指针。在（而且仅在）这种场合中，C对于int ar[]和int *ar作出同样解释，即ar是指向int的指针。因此下面的2种原型都是等价的：
int sum (int *ar, int n);
int sum (int ar[], int n);
-------------------------------------------------------------------------------
// 数组元素求和
#define SIZE 10
int sum(int ar[],int n);
int main(){
    int marbles[SIZE] = {20,10,5,39,4,16,19,26,31,20};
	int answer;
	answer = sum(marbles,SIZE);
	printf("sum:%d\n",answer);
    printf("size of marbles:%zu\n",sizeof(marbles));
    return 0;
}
int sum(int ar[],int n){
    int total = 0;
    for(int i = 0;i < n;i++){
        total += ar[i]; // 指针指向一个数组，指针名相当于数组名
    }
    return total;
}
-------------------------------------------------------------------------------
// 数组元素求和
int sum(int* start,int* end);
int main(){
    int marbles[SIZE] = {20,10,5,39,4,16,19,26,31,20};
	int answer;
	answer = sum(marbles,marbles + SIZE); // marbles+SIZE指向数组结尾之后的下一个元素
	printf("sum:%d\n",answer);
    printf("size of marbles:%zu\n",sizeof(marbles));
    return 0;
}
int sum(int* start,int* end){
    int total = 0;
    while(start < end){
        total += *start; // 可以合并为 total += *(start++);
        start++;
    }
    return total;
}    
```



#### 5️⃣ ==指针的操作==

```cpp
// 指针的操作：赋值=、取值*、取指针地址&、指针加法+、指针减法-、指针递增++、指针递减--、指针减指针
#include <stdio.h>
int main(){
    int urn[5] = {100,200,300,400,500};
    int *ptr1,*ptr2,*ptr3;
    ptr1 = urn;
    ptr2 = &urn[2];
    ptr3 = ptr1 + 4;
    printf("ptr1 = %p,*ptr1 = %d,&ptr1 = %p\n",ptr1,*ptr1,&ptr1);
    printf("ptr1+4 = %p,*(ptr1+3) = %d\n",ptr1+4,*(ptr1+3));      // 1.指针加法
    printf("ptr3 = %p,ptr3 - 2 = %p\n",ptr3,ptr3 - 2);            // 2.指针减法
    ptr1++;
    printf("ptr1 = %p,*ptr1 = %d,&ptr1 = %p\n",ptr1,*ptr1,&ptr1); // 3.指针递增
    ptr2--;
    printf("ptr2 = %p,*ptr2 = %d,&ptr2 = %p\n",ptr2,*ptr2,&ptr2); // 4.指针递减
    --ptr1; // 恢复初值
    ++ptr2; // 恢复初值
    printf("ptr2=%p,ptr1=%p,ptr2-ptr1=%d\n",ptr2,ptr1,ptr2-ptr1); // 5.指针减指针,得到指针之间的元素个数,ptr2-ptr1=2
    return 0;
}
-------------------------------------------------------------------------------
// 不能对未初始化的指针取值
int *pt; //未初始化的指针
*pt =5;  //一个可怕的错误    
```



#### 6️⃣ ==保护数组内容==

```cpp
// 函数使用指针可以高效地读写原数组，但有时我们不希望函数直接修改原始数据
// 如果设计意图是函数不改变数组的内容，那么可以在函数原型和定义的形参声明中使用const
-------------------------------------------------------------------------------
// 两个函数，一个展示数组，一个改变数组
#define SIZE 5
void show_array(const double ar[],int n);
void mult_array(double ar[],int n,double mult);
int main(){
    double dip[SIZE] = {20.0,17.66,8.2,15.3,22.22};
    show_array(dip,SIZE);
    mult_array(dip,SIZE,2.5);
    show_array(dip,SIZE);
    return 0;
}
void show_array(const double ar[],int n){// 常量指针
    for(int i = 0;i < n;i++){
        printf("%.3f",ar[i]);
    }
    putchar('\n');
}
void mult_array(double ar[],int n,double mult){
    for(int i = 0;i < n;i++){
        ar[i] *= mult;
    }
}
-------------------------------------------------------------------------------
// 有关const的其他内容
double rates[5] = {88.99,100.12,59.45,183.11,340.5};
const double PI = 3.14159; // 1.创建符号常量
const int days[3] = {31,28,31}; // 2.创建数组常量
const double* pd = rates; // 3.创建常量指针(指向常量的指针)，不能使用指针pd来修改它所指向的值
*pd = 29.98    //❌
pd[2] = 222.22 //❌
rates[2] = 222.22 //✅
double* const pc = rates; // 4.创建指针常量，指针不能改变指向
pc = &rates[2] //❌
*pc = 92.99    //✅
--------------------------------------------------------------------------------
// 将常量或非常量数据的地址赋给指向常量的指针是合法的
double rates[5] = {88.99,100.12,59.45,183.11,340.5};
const double 1ocked[4] = {0.08,0.075,0.0725,0.07};
const double * pc = rates;//合法
pc = locked;// 合法
pc = &rates[3];//合法
// 然而，只有非常量数据的地址才可以赋给普通的指针
double *pnc = rates;// 合法
pnc = locked;    // 非法
pnc = &rates[3];// 合法
// 这样的规则是合理的。否则，您就可以使用指针来修改被认为是常量的数据。
```



#### 7️⃣ ==指针和多维数组==

```cpp
int zippo[4][2];
// zippo的值等于&zippo[0],zippo[0]的值等于&zippo[0][0];
// zippo所指向的对象的大小是两个int，而zippo[0]所指向的对象的大小是一个int，因此zippo+1与zippo[0]+1结果不同
/*
			  zippo                  zippo+1                 zippo+2                 zippo+3
                |                       |                       |                       |
    |       zippo [0]       |       zippo [1]       |       zippo [2]       |       zippo [3]       |
    +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
    |   zippo   |   zippo   |   zippo   |   zippo   |   zippo   |   zippo   |   zippo   |   zippo   |
地址 |   [0][0]  |   [0][1]  |   [1][0]  |   [1][1]  |   [2][0]  |   [2][1]  |   [3][0]  |   [3][1]  |
    +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
    |    0BF2   |    0BF4   |    0BF6        0BF8        0BFA        0BFC        0BFE        0C00
    |           |           |
    |           |           V
    |           V         *zippo+2
    V         *zippo+1
  *zippo
*/

```



#### 8️⃣ ==指向多维数组的指针==

```cpp
// 如何声明一个指向二维数组的指针变量pz？在编写处理像zippo这样的数组的函数时会用到这类指针
int (*pz)[2]; // 数组指针，pz指向一个包含2个int值的数组
int *pz[2]; // 指针数组
--------------------------------------------------------------------------------
int zippo[4](2] ={{2,4},{6,8},{1,3},{5,7}};
int (*pz) [2];
pz = zippo;
printf("pz = %p,pz + 1 = %p\n",pz,pz + 1); // 指针指向数组，指针名替换数组名
printf（"pz[0] = %p, pz[0] + 1 = %p\n",pz[0],pz[0]+1);

```



#### 9️⃣ ==指针兼容性==

```cpp
int *p1;
const int* p2;
const int** pp2;
p1 = p2; // ❌把const指针赋值给非const指针
p2 = p1; // ✅非const指针赋值给const指针
pp2 = &p1; // ❌非const指针赋值给const指针
// 把const指针赋值给非const指针是错误的，因为可能会使用新指针来改变const数据，但是把非const指针赋值给const指针是允许的，但有一个前提，只进行一层间接运算，在进行两层间接运算时，可能会产生如下的问题：
const int **pp2;
int *p1;
const  int n = 13;
pp2 = &p1; //不允许，但假设允许
*pp2 = &n; //合法，两边都是const
*p1 = 10;  //合法，但改变了const n的值
```



#### 🔟 ==函数与多维数组==

```cpp
// array2d.c--处理2维数组的函数
// 只能处理固定列数为4的数组
#include <stdio.h>
#define ROWS 3
#define COLS 4
void sum_rows (int ar[][COLS], int rows);
void sum_cols (int [][COLS],int); //可以省略名称
int sum2d (int(*ar)[COLS],int rows);//另一种语法形式
int main (void)
{
	int junk[ROWS][COLS]={{2,4,6,8},{3,5,7,9},{12,10,8,6}};
	sum_rows(junk, ROWS);
	sum_cols(junk, ROWS);
	printf("%d",sum2d(junk, ROWS));
	return 0;
}
void sum_rows(int ar[][COLS],int rows){
    int r;
	int c;
	int tot;
	for (r = 0;r < rows; r++){
        tot = 0;
        for(c = 0;c < COLS;c++){
            tot += ar[r][c];
        }
        printf("row%d:%d\n",r,tot);
    }
}
void sum_cols(int ar[][COLS],int rows){
    int r;
    int c;
    int tot;
    for(c = 0;c < COLS;c++){
        tot = 0;
        for(r = 0;r < rows;r++){
            tot += ar[r][c];
        }
        printf("col%d:%d\n",c,tot);
    }
}
int sum2d(int ar[][COLS],int rows){
	int r;
    int c;
    int tot = 0;
    for(r = 0;r < rows;r++){
        for(c = 0;c < COLS;c++){
            tot += ar[r][c];
        }
    }
    return tot;
}
```



#### 1️⃣1️⃣ ==变长数组VLA==

```cpp
// C99标准引入变长数组，他允许使用变量定义数组各维,变的意思是其维度大小可以用变量来指定
int q = 4;
int r = 5;
double sales[r][q];// 一个变长数组vla
-----------------------------------------------------------------------------
// 计算任意维度二维int数组和的函数
int sun2d(int rows,int cols,int ar[rows][cols]); // ar是VLA，row与cols声明在ar前面
```



#### 1️⃣2️⃣ ==编程练习==

```cpp
//2. 编写一个程序，初始化一个double数组，然后把数组内容复制到另外两个数组（3个数组都需要在主程序中声明)。制作第一份拷贝的函数使用数组符号。制作第二份拷贝的函数使用指针符号，并使用指针的增量操作。把目标数组名和要复制的元素数目做为参数传递给函数。也就是说，如果给定了下列声明，函数调用应该如下面所示：
//double source[5] = {1.1,2.2,3.3,4.4,5.5};
//double targetl[5];
//double target2[5];
//copy_arr(source,targetl,5);
//copy_ptr(source,targetl,5);
void copy_arr(double arr1[],double arr2[],int n){
    for(int i = 0;i < n;i++){
        arr2[i] = arr1[i];
    }
}
void copy_ptr(double* ptr1,double* ptr2,int n){
    for(int i = 0;i < n;i++){
        *ptr2 = *ptr1;
        ptr1++;
        ptr2++;
    }
}
-------------------------------------------------------------------------------
//3. 编写一个函数，返回一个int 数组中存储的最大数值，并在一个简单的程序中测试这个函数。
#include <limits.h>
int getMax(int arr[],int n){
    int val = INT_MIN;
    for(int i = 0;i < n;i++){
        if(arr[i] > val){
            val = arr[i];
        }
    }
    return val;
}
-------------------------------------------------------------------------------
//4. 编写一个函数，返回一个double数组中存储的最大数值的索引，并在一个简单程序中测试这个函数。
int getMaxIndex(double arr[],int n){
    if(n <= 0) return -1;
    int idx = 0;
    for(int i = 1;i < n;i++){
        if(arr[i] > arr[idx]){
            idx = i;
        }
    }
    return idx;
}
-------------------------------------------------------------------------------
//5. 编写一个函数,返回一个double 数组中最大的和最小的数之间的差值，并在一个简单的程序中测试这个函数。
double getDiff(double arr[],int n){
    if(n <= 1) return 0.0;
    double maxVal = arr[0];
    double minVal = arr[0];
    for(int i = 1;i < n;i++){
        if(maxVal < arr[i]){
            maxVal = arr[i];
        }
        if(minVal > arr[i]){
            minVal = arr[i];
        }
    }
    return maxVal - minVal;
}     
-------------------------------------------------------------------------------
//6. 编写一个程序，初始化一个二维double 数组，并利用练习2中的任一函数来把这个数组复制到另一个二维数组（因为二维数组是数组的数组，所以可以使用处理一维数组的函数来复制数组的每个子数组）。
double arr1[2][3] = {1,2,3,4,5,6};
double arr2[2][3];
copy_arr(arr1[0],arr2[0],3);
copy_arr(arr1[1],arr2[1],3);
-------------------------------------------------------------------------------
//7. 利用练习2中的复制函数，把一个包含7个元素的数组内第3到第5元素复制到一个包含3个元素的数组中。函数本身不需要修改，只需要选择合适的实际参数（实际参数不需要是数组名和数组大小，而只须是数组元素的地址和需要复制的元素数目）。
int arr1[7] = {1,2,3,4,5,6,7};
int arr2[3];
copy_ptr(&arr1[2],arr2,3);
// 或者下面也可以
copy_ptr(arr1 + 2,arr2,3);//数组名是一个地址常量，我们不能写arr1=arr1+2,但可以进行求值运算，并没有修改arr1本身,而是产生了一个新的临时地址值
-------------------------------------------------------------------------------
//8. 编写一个程序，初始化一个 3x5的二维double 数组，并利用一个基于变长数组的函数把该数组复制到另一个二维数组。还要编写一个基于变长数组的函数来显示两个数组的内容。这两个函数应该能够处理任意的 NxM 数组。
// VLA不再强制要求你在函数参数中硬编码列数（如 int arr[][5]），而是允许你通过参数动态地传递 N 和 M。
void copyarr(int N,int M,double arr1[N][M],double arr2[N][M]){
    for(int i = 0;i < N;i++){
        for(int j = 0;j < M;j++){
            arr2[i][j] = arr1[i][j];
        }
    }
}   
-------------------------------------------------------------------------------   
//9. 编写一个函数，把两个数组内的相应元素相加，结果存储到第3个数组内。也就是说，如果数组1具有值2、4、5、8，数组2具有值1、0、4、6，则函数对数组3赋值为3、4、9、14。函数的参数包括3个数组名和数组大小。并在一个简单的程序中测试这个函数。
void arrsum(int arr1[],int arr2[],int arr3[],int n){
    for(int i = 0;i < n;i++){
        arr3[i] = arr1[i] + arr2[i];
    }
}    
-------------------------------------------------------------------------------
//12. 编写一个程序，提示用户输入3个数集，每个数集包括5个double值。程序应当实现下列所有功能：
//a.把输入信息存储到一个3x5 的数组中
//b.计算出每个数集（包含5个数值）的平均值
//c.计算所有数值的平均数
//d.找出这15个数中的最大值．
//每个任务需要用一个单独的函数来实现（使用传统C处理数组的方法)。对于任务b，需要编写计算并返回一维数组平均值的函数，循环3次调用该函数来实现任务b。对于其他任务，函数应当把整个数组做为参数，并且完成任务c和d的函数应该向它的调用函数返回答案。
void inputnum(double arr[][5],int rows){
    for(int i = 0;i < rows;i++){
        for(int j = 0;j < 5;j++){
            scanf("%lf",&arr[i][j]);
        }
    }
}
double getavg(double arr[],int n){
    double total = 0.0;
    for(int i = 0;i < n;i++){
        total += arr[i];
    }
    return total / n;
}
double get2davg(double arr[][5],int rows){
    double total = 0.0;
    for(int i = 0;i < rows;i++){
        for(int j = 0;j < 5;j++){
            total += arr[i][j];
        }
    }
    return total / (rows * 5);
}
double getmax(double arr[][5],int rows){
    double val = arr[0][0];
    for(int i = 0;i < rows;i++){
        for(int j = 0;j < 5;j++){
            if(val < arr[i][j]){
                val = arr[i][j];
            }
        }
    }
    return val;
}
```



## 第 11 章 字符串和字符串函数

#### 1️⃣ ==定义字符串==

```cpp
// 字符串常量，char数组，char指针，字符串数组
--------------------------------------------------------------------------------
//1.字符串常量:位于一对引号""中的任何字符，双引号里的字符加上编译器自动提供的结束标志\0，作为一个字符串被存储在内存里
printf("run,spot,run!\n");
//字符串常量属于静态存储类。静态存储是指如果在一个函数中使用字符串常量，即使是多次调用了这个函数，该字符串在程序运行过程中只存储一份，整个引号中的内容作为指向该字符串存储位置的指针。
// 把字符串看作指针
printf("%s,%p,%c","we","are",*"space farers");//we,0x555555556004,s
--------------------------------------------------------------------------------
//2.char数组:定义一个字符数组时，必须要让编译器知道它需要多大空间，或者让编译器决定数组的大小
const char m1[40] = "Limit yourself to one line's worth."
const char m2[] = "If you can't think of anything,fake it."
// 和任何数组名一样，字符数组名也是数组首元素的地址
--------------------------------------------------------------------------------
//3.char指针
char m3[] = "Enough about me - what's your name?";//数组初始化:从静态存储区把一个字符串复制给数组，m3看作是数组首元素的地址&m3[0]的同义词，m3是一个地址常量，可以m3+1标识下一个元素，但不能++m3
const char* m3 = "Enough about me - what's your name?";//指针初始化:复制字符串的地址，指针形式也在静态存储区为字符串预留38个元素的空间，此外程序执行时还要为指针变量m3预留一个存储位置来存储字符串的地址，这个指针变量初始时指向字符串的第一个字符，但是它的值是可以变的，++m3指向第二个字符n
// 字符数组的元素是变量(除非声明数组时带const),但是数组名不是变量
// 编译器会使用相同的地址来替代相同的字符串实例，如果编译器允许把p1[0]改为f，那将会影响到所有对这个字符串的使用，因此建议的做法是初始化指向字符串的指针时使用const修饰
char* p1 = "klingon";
p1[0] = 'f';// ok? ❌
printf("klingon");//
-------------------------------------------------------------------------------
//4.字符串数组
const char *mytal[LIM] ={"Adding numbers swiftly",
                         "Multiplying accurately",
                         "Stashing data",
                         "Following instructions to the letter",
                         "Understanding the C language"};
//mytal是一维数组，数组里的每一个元素都是一个char类型的地址值，第一个指针是mytal[0],它指向第一个字符串的第一个字符。mytal数组实际上并不存放字符串，它只是放字符串的地址
-------------------------------------------------------------------------------
// 指针和字符串
char* mesg = "Don't be a fool!";
char* copy;
copy = mesg; // 两个指针指向同一个地址
printf("mesg=%s,&mesg=%p,val=%p\n",mesg,&mesg,mesg);
printf("copy=%s,&copy=%p,val=%p\n",copy,&copy,copy);
//mesg=Don't be a fool!,&mesg=0x7fffffffdc78,val=0x555555556004
//copy=Don't be a fool!,&copy=0x7fffffffdc80,val=0x555555556004
```



#### 2️⃣ ==字符串输入==

```cpp
// 1.scanf()
char* name; // ❌指针未初始化，name可能指向任何地方
char name[81]; // ✅
scanf("%s",name); 
-------------------------------------------------------------------------------
// 2.gets():无法限制读取字符的数量，易导致缓冲区溢出，严禁使用
-------------------------------------------------------------------------------
// 3.fgets():第2个参数为最大读入字符数，为n表示最大读入n-1个字符或读完一个换行符为止，换行符会被放到字符串中，第3个参数表示从哪一个文件读取，从键盘上输入数据时，可以使用stdin
char name[81];
char8 ptr;
printf("what's your name?\n");
ptr = fgets(name,sizeof(name),stdin);// 换行符\n也读到字符串中了
printf("%s,%s",ptr,name);  
```



#### 3️⃣ ==字符串输出==

```cpp
// 1.printf()
char str[] = "Hello";
printf("%s", str);  
-------------------------------------------------------------------------------
// 2.fputs()
fputs(str,stdout);
-------------------------------------------------------------------------------
// 3.自定义字符串输入/输出函数
void put1(const char* string){ // 不会改变这个字符串
    while(*string != '\0'){
        putchar(*string++);
    }
}
```



#### 4️⃣ ==字符串函数==

```cpp
// strlen(),strcat(),strncat(),strcmp(),strncmp(),strcpy(),strncpy()
-------------------------------------------------------------------------------
//1.strlen():返回字符串的长度，不包括结尾的\0
#include <string.h>
void fit(char* string,unsigned int size){ //缩短字符串长度的函数
    if(strlen(string) > size){
        *(string + size) = '\0';
    }
}
------------------------------------------------------------------------------- //2.strcat():接受两个字符串，将第二个字符串的一份拷贝添加到第一个字符串的结尾，从而使第一个字符串成为新的字符串，函数返回第一个参数的值
char flower[50] = "Roses are "; // 确保数组足够大！
char color[] = "red.";
strcat(flower, color);
fputs(flower,stdout); // 输出：Roses are red.    
// strncat(target, source, n):strcat()会有溢出问题，为了解决溢出问题，strncat()多了一个参数：允许追加的最大字符数。strncat表示只追加source中的前n个字符，或者直到遇到source的\0
char target[20] = "Hello ";
char source[] = "World Wide Web";
int available = 20 - strlen(target) - 1;// 计算剩余空间：20 - 已用长度 - 1(给\0留位)
strncat(target, source, available); // 结果：Hello World Wide (不会溢出) 
------------------------------------------------------------------------------- //3.strcmp(): 比较两个字符串的内容是否相同，返回值0表示相同，正数或者负数代表不相同
char s1[] = "apple";
char s2[] = "apple";
char s3[] = "banana";
if (strcmp(s1, s2) == 0) {
    puts("s1 和 s2 相等");
}
if (strcmp(s1, s3) < 0) {
    puts("apple 在字典中排在 banana 前面");
}    
// strncmp() 只比较前 n 个字符。   
if (strncmp(command, "list ", 5) == 0) { // 检查字符串是否以 "list " 开头
    // 执行列表逻辑
}    
------------------------------------------------------------------------------- //4.strcpy():字符串拷贝，char* strcpy(char* s1,const char* s2),将s2指向的字符串复制到s1指向的位置，返回值是s1
char target[20] = "Old Message";
char source[] = "New!";
strcpy(target, source);
printf("target:%s\n",target);// 输出New!,原本的 "Message" 虽然还残留在内存里，但因为 'w' 后面紧跟了 '\0'，printf 就不再往后读了。    
//在现代工程开发中除非你百分之百确定空间足够，否则应该优先使用 strncpy(),它多了一个限制长度的参数 n,表示最多拷贝 n 个字符    
//strncpy(target, source, n),如果 source的长度超过了 n，它拷贝完后不会自动在末尾加 \0。
strncpy(target, source, sizeof(target) - 1);
target[sizeof(target) - 1] = '\0'; // 强制手动封口，万无一失   
-------------------------------------------------------------------------------
//5.strchr():返回字符串中第一个出现的指定字符的地址，没找到返回NULL，char *ptr = strchr(const char *s, int c)，
char *p = strchr(name, '\n'); // 找换行符
if (p) *p = '\0';             // 如果找到了，原地处决（替换为结束符）  
-------------------------------------------------------------------------------
//6.strstr():在一个大字符串里找一个小字符串,返回子串第一次出现的起始地址,char *ptr = strstr(const char *haystack, const char *needle)
if (strstr(input, "admin") != NULL) {
    puts("检测到管理员指令");
}
------------------------------------------------------------------------------ 
+-----------------+----------------------+-----------------------------------+
|  危险函数 (弃用)  |       潜在风险        |        安全替代方案 (推荐)           |
+-----------------+----------------------+-----------------------------------+
|      gets()     |  ⚠️ 必死无疑 (已删)    |              fgets()              |
+-----------------+----------------------+-----------------------------------+
|     strcat()    |  ⚠️ 缓冲区溢出        |             strncat()             |
+-----------------+----------------------+-----------------------------------+
|     strcpy()    |  ⚠️ 缓冲区溢出        |      strncpy() / memcpy()         |
+-----------------+----------------------+-----------------------------------+ 
```



#### 5️⃣ ==命令行参数==

```cpp
/* repeat.c */
#include <stdio.h>
int main(int argc,char* argv[]){
    int count;
    printf("有%d个命令行参数\n",argc);
    printf("有用的命令行参数有%d个\n",argc-1);
    for(int count = 0;count < argc;count++){
        printf("第%d个参数:%s\n",argv[count]);
    }
}
/* 
gcc repeat.c 
./a.out 100 200 300 abc
有5个命令行参数
有用的命令行参数有4个
第0个参数:./a.out
第1个参数:100
第2个参数:200
第3个参数:300
第4个参数:abc
*/
```



#### 6️⃣ ==把字符串转换为数字==

```cpp
// 命令行参数以字符串形式被读取，因此想要使用数字值，就必须先把字符串转换为数字
------------------------------------------------------------------------------
//1.strtol():把一个字符串转换为long型值
//2.strtoul():把一个字符串转换为unsigned long型值
//3.strtod()：把一个字符串转换为double型值
//long strtol(const char* nptr,char** endptr,int base);
//nptr:要转换的字符串,endptr:转换停止的位置（比如遇到第一个非数字字符）会存在这里,base:进制
------------------------------------------------------------------------------
#include <stdlib.h>
int main() {
    char *input = "123.45abc";
    char *end;    
    long val = strtol(input, &end, 10); // 转换为 10 进制长整型
    printf("成功转换的部分: %ld\n", val); // 输出 123
    printf("无法转换的残余: %s\n", end);  // 输出 .45abc    
    if (*end != '\0') {
        printf("提示：输入包含非整数内容。\n");
    }
    return 0;
}
```



#### 7️⃣ ==编程练习==

```cpp
//3. 设计并测试一个函数，其功能是读取输入行里的第一个单词到数组，并丢掉该行中其他的字符。一个单词的定义是一串字符，其中不含空格、制表符和换行符。
void getword(char word[]){
    scanf("%s",word);
    int ch;
    while((ch = getchar()) != '\n' && ch != EOF);
}
--------------------------------------------------------------------------------
//4. 设计并测试一个函数，其功能是搜索由函数的第一个参数指定的字符串，在其中查找由函数的第二个参数指定的字符的第一次出现的位置。如果找到，返回指向这个字符的指针;如果没有找到，返回空,这种方式和strchr()函数的功能一样。
char* getpos(char* str,char ch){
    while(*str){
        if(*str == ch){
            return str;
        }
        str++;
    }
    return NULL;
}
--------------------------------------------------------------------------------
//5. 编写一个函数is_within()。它接受两个参数，一个是字符，另一个是字符串指针。其功能是如果字符在字符串中，就返回一个非0值(真)；如果字符不在字符串中，就返回0值(假)。
#include <stdbool.h>
bool is_within(const char* str,char ch){
    while(*str){
        if(*str == ch){
            return true;
        }
        str++;
    }
    return false;
}
------------------------------------------------------------------------------- 
//6. strncpy(s1,s2,n)函数从 s2 复制n个字符给 sl，并在必要时截断 s2 或为其填充额外的空字符。如果s2的长度等于或大于n，目标字符串就没有标志结束的空字符。函数返回s1。自己编写这个函数。
char* my_strncpy(char* s1,char* s2,int n)  {
    char* tmp = s1;
    while(*s2 && n > 0){
        *s1++ = *s2++;
        n--;
    }
    while(n > 0){
        *s1++ = '\0';
        n--;
    }
    return tmp;
}  
-------------------------------------------------------------------------------
//7. 编写一个函数string_in()，它接受两个字符串指针参数。如果第二个字符串被包含在第一个字符串中，函数就返回被包含的字符串开始的地址。例如，string_in("hats"，"at")返回hats中a的地址，否则，函数返回空指针。
char* string_in(const char* str1, const char* str2) {
    if (*str2 == '\0') return (char*)str1;    // 如果子串为空，按照惯例返回主串首地址
    while (*str1 != '\0') {
        const char* p1 = str1;
        const char* p2 = str2;
        while (*p1 != '\0' && *p2 != '\0' && *p1 == *p2) { // 尝试匹配
            p1++;
            p2++;
        }
        if (*p2 == '\0') {  // 如果 p2 走到了尽头，说明完全匹配成功
            return (char*)str1;
        }
        str1++; // 匹配失败，主串指针后移一位，继续下一轮“尝试”
    }
    return NULL;
}
-------------------------------------------------------------------------------
//8. 编写一个函数，其功能是使输入字符串反序。
char* str_reverse(char* str){
    if(str == NULL || *str == '\0') return str;// 字符串为空(未分配内存)或者只有一个字符
    char* p1 = str;
    char* p2 = str;
    while(*p2 != '\0') p2++;
    p2--;
    while(p1 < p2){
        char tmp = *p2;
        *p2 = *p1;
        *p1 = tmp;
        p1++;
        p2--;
    }
    return str;
}
-------------------------------------------------------------------------------
//9. 编写一个函数。其参数为一个字符串，函数删除字符串中的空格。
char* del_space(char* str){
    if(str == NULL || *str == '\0') return str;//为空或者只有一个字符
    char* fast = str;//快指针：负责扫描原字符串,寻找“非空格”的宝贝,永远向前走
    char* slow = str;//慢指针：负责把快指针找到的“宝贝”按顺序存放到字符串的前面。
    while(*fast != '\0'){
        if(*fast != ' '){
            *slow = *fast;
            slow++;
        }
        fast++;
    }
    *slow = '\0';
    return str;
}    
```



## 第 12 章 存储类、链接和内存管理

#### 1️⃣ ==存储时期==

```c
// 存储时期:描述的是变量在内存中“活多久”
-------------------------------------------------------------------------------
// 静态存储时期(static)：程序运行期间一直存在。所有的全局变量和由static修饰的变量都属于此类。
// 自动存储时期(auto)：进入定义变量的代码块时创建，退出时销毁。普通局部变量默认就是这种。
```



#### 2️⃣ ==链接==

```c
// 链接:决定了你在一个文件中定义的变量或函数，在多大范围的程序内是“可见”的
--------------------------------------------------------------------------------
//1.外部链接:具有外部链接的标识符可以在整个程序（所有源文件）中使用
//在源文件（.c）的函数外部定义变量：int global_var = 10;所有的函数默认都是外部链接。
// file1.c
int count = 5; // 外部链接
// file2.c
extern int count; // 声明：我要用别处定义的那个 count
--------------------------------------------------------------------------------
//2.内部链接:只在当前的源文件（编译单元）内可见，对其他文件是“隐身”的
static int local_secret = 100; // 内部链接，file2 即使写了 extern 也找不到它
static void private_tool() { ... }    
--------------------------------------------------------------------------------
//3.空链接:只在当前的块（如一个 {} 内部）可见
void func() {
    int x = 10; // 空链接，只在 func 内部有效
}  
--------------------------------------------------------------------------------
+-----------------------------------------------------------------------------+
|                          C 语言链接属性 (Linkage) 概览                         |
+--------------+-------------------+----------------------+-------------------+
|   链接类型    |     关键关键字      |       可见范围        |      典型代表       |
+--------------+-------------------+----------------------+-------------------+
|              |                   |  [ 项目级可见 ]        |                   |
|   外部链接    |  默认 / extern     |  整个程序的所有文件     |  全局变量, 普通函数  |
|  (External)  |                   |  A.c <---> B.c       |                   |
+--------------+-------------------+----------------------+-------------------+
|              |                   |  [ 文件级可见 ]        |                   |
|   内部链接    |  static(外部定义)   |  仅限当前定义的源文件   | 静态全局变量,       |
|  (Internal)  |                   |  A.c只有 A.c 瞧得见    | 静态函数           |
+--------------+-------------------+----------------------+-------------------+
|              |                   |  [ 块级可见 ]         |                   |
|   空链接      |  无 (局部定义)      |  仅限当前代码块 {}     | 局部变量, 函数参数  |
|   (None)     |                   |  出了括号就不认识了     |                   |
+--------------+-------------------+----------------------+-------------------+

    [ 作用域范围示意图 ]
    Max ----------------------------------------------------------> Min
    
    [ 整个工程 ] >>>> [ 单个源文件 ] >>>> [ 函数内部 ] >>>> [ 代码块 ]
    (外部链接)        (内部链接)          (空链接)          (空链接)
```



#### 3️⃣ ==存储类==

```c
// 存储类:通过关键字来指定变量的时期、范围（作用域）和链接
+--------------+----------------+---------------+--------------+--------------------+
|   存储类      |    存储时期     |     作用域     |    链接       |      声明方式       |
+--------------+----------------+---------------+--------------+--------------------+
|   自动 (auto) |      自动      |      块        |     无       | 函数内: int a;      |
+--------------+----------------+---------------+--------------+--------------------+
|   寄存器      |      自动      |      块        |     无       | register int a;    |
|  (register)  | (存放在CPU中)   |               |              |                    |
+--------------+----------------+---------------+--------------+--------------------+
|  静态局部     |      静态       |      块       |     无        | 函数内: static int;|
+--------------+----------------+---------------+--------------+--------------------+
|  静态全局     |      静态       |      文件      |     内部     | 函数外: static int; |
+--------------+----------------+---------------+--------------+--------------------+
|  外部 (extern)|      静态      |      文件      |     外部      | 函数外: int a;     |
+--------------+----------------+---------------+--------------+--------------------+
--------------------------------------------------------------------------------
+----------------+----------------+----------------+--------------------------+
|    变量类型     |    存储时期      |   自动初始化?   |        缺省初始值          |
+----------------+----------------+----------------+--------------------------+
|    寄存器变量   |      自动       |      否         |  未定义 (寄存器残留值)      |
+----------------+----------------+----------------+--------------------------+ |  普通局部变量    |      自动       |      否        |  未定义 (内存残留垃圾值)    |
+----------------+----------------+----------------+--------------------------+ |  静态局部变量    |      静态       |      是        |  0 (整型) / NULL (指针)   |
+----------------+----------------+----------------+--------------------------+ |    全局变量     |      静态       |      是        |  0 (整型) / NULL (指针)   |
+----------------+----------------+----------------+--------------------------+ 
```



#### 4️⃣ ==随机数函数和静态变量==

```c
/* 包含函数rand1()和srand1()的文件*/
static unsigned int next = 1; //随机种子
int rand1(){
    next = next * 1103515245 + 12345; // 产生伪随机数的公式
    return (next/65536)%32768;
}
void srand1(unsigned int seed){
    next = seed;
}
------------------------------------------------------------------------------
/* 测试函数rand1()和srand1()*/
#include <stdio.h>
extern void srand1(unsigned int x);
extern int rand1();
int main(){
    int count;
    unsigned seed;
    printf("请输入种子\n");
    while(scanf("%d",&seed) == 1){
        srand1(seed); // 种植随机种子
        for(count = 0;count < 5;count++){
            printf("%d\n",rand1());
        }
    }
}
--------------------------------------------------------------------------------
// 自动重置种子:time(0); 返回1970年到现在的秒数
// ANSIC 函数 srand() 与 rand()
```



#### 5️⃣ ==分配内存：malloc() 和 free()==

```c
// malloc():从内存堆中申请一块连续的、指定大小的空间,如果内存不够,它会返回 NULL;void* malloc(size_t size);
// free():释放malloc()分配的内存,参数是malloc返回的地址;void free(void* ptr)
-------------------------------------------------------------------------------
#include <stdio.h>
#include <stdlib.h> // 必须包含 malloc 和 free 的头文件
int main() {
    int n;
    int *ptr;
    printf("您想存储多少个整数？ ");
    scanf("%d", &n);
    ptr = (int*)malloc(n * sizeof(int));// 1. 申请内存,malloc返回void*,所以要强制类型转换为 int*
    if (ptr == NULL) {    // 2. 检查是否申请成功（防御性编程）
        printf("内存分配失败，程序退出。\n");
        exit(EXIT_FAILURE);// 结束整个程序,EXIT_FAILURE指示程序异常终止, EXIT_SUCCESS(0) 指示程序正常终止
    }
    printf("请输入 %d 个数字：\n", n);    // 3. 使用内存：像操作普通数组一样操作指针
    for (int i = 0; i < n; i++) {
        scanf("%d", &ptr[i]); // 或者 scanf("%d", ptr + i);
    }
    printf("您输入的数字是：");
    for (int i = 0; i < n; i++) {
        printf("%d ", ptr[i]);
    }
    free(ptr);    // 4. 释放内存：这是最重要的步骤！    
    ptr = NULL;    // 5. 将指针置空：防止之后误操作这个“野指针”
    printf("内存已成功释放。\n");
    return 0;
}
--------------------------------------------------------------------------------
// calloc():如果你希望申请到的内存干干净净,全部自动初始化为 0
// 申请 n 个元素，每个元素大小为 sizeof(int)，且自动清零
ptr = (int*)calloc(n, sizeof(int));
```



#### 6️⃣ ==ANSI C 的类型限定词==

```c
/* const 类型限定词 */
// 1.定义常量 
const int nochange = 12; 

// 2.常量指针
const int a = 10; 
const int b = 20;
const int *p = &a; 
// *p = 15; // 错误：不能通过 p 修改 a 的值
p = &b;     // 正确：p 可以改为指向 b

// 3.指针常量
int a = 10;
int b = 20;
int * const p = &a;
*p = 15;    // 正确：可以修改 a 的值
// p = &b;  // 错误：p 不能再指向别的地方

// 4.函数参数
int get_length(const char *s) {
    // s[0] = 'A'; // 如果你试图在这里改动字符串，编译器会立刻拦住你
    return strlen(s);
}
--------------------------------------------------------------------------------
/* volatile 类型限定词:编译器读取变量时，必须去内存地址读，不要对它进行优化 */
-------------------------------------------------------------------------------- /* restrict 类型限定词:只能修饰指针，告诉指针它是访问它所指向对象的唯一且初始的方式，好处：消除指针别名，让编译器放心优化 */
void add_vec(int * restrict a, int * restrict b, int * restrict val) {
    *a += *val; //你既然保证了这三个指针互不重叠，那我就放心了！计算完第一行后，
    *b += *val; //我直接把 *val 的值留在寄存器里给第二行用，不需要再去内存加载一遍。
}

// memcpy（要求内存不能重叠）：由于加了 restrict，编译器可以使用最快的拷贝算法。
void *memcpy(void * restrict dest, const void * restrict src, size_t n);

// memmove（允许内存重叠）：没有 restrict，为了安全，它的实现通常比 memcpy 慢一点点。
void *memmove(void *dest, const void *src, size_t n);
```



## 第 13 章 文件输入/输出

#### 1️⃣ ==常用标准 I/O 函数==

```c
// 打开文件:FILE *fopen(const char *filename,const char *mode),成功返回指针,失败返回NULL
// 关闭文件:int fclose(FILE *stream),成功返回0,失败返回EOF
// 常见模式说明: "r" -> 只读方式打开，文件必须存在。
// 			   "w" -> 只写方式打开，若文件存在则清空，不存在则创建。
// 		       "a" -> 追加方式打开，在文件末尾写入。
int main() {
    FILE *fp = fopen("hello.txt", "w"); // 以写入模式打开
    if (fp == NULL) {
        perror("打开文件失败");
        return 1;
    }
    printf("文件已成功开启。\n");    
    fclose(fp); // 务必关闭，否则可能造成数据丢失或内存泄漏
    return 0;
}
--------------------------------------------------------------------------------
// 从文件读字符: int fgetc(FILE *stream)   
// 向文件写字符: int fputc(int char, FILE *stream)    
int main() {
    // 写入一个字符
    FILE *fp = fopen("char.txt", "w");
    fputc('G', fp);
    fclose(fp);
    // 读取这个字符
    fp = fopen("char.txt", "r");
    int ch = fgetc(fp); 
    if (ch != EOF) {
        printf("读到的字符是: %c\n", ch);
    }
    fclose(fp);
    return 0;
}
// 从文件读一行: char *fgets(char *str,int n,FILE *stream); (读取最多n-1个字符,遇到换行符停止)
// 向文件写一行: int fputs(const char *str, FILE *stream);
int main() {
    char buffer[50];
    FILE *fp = fopen("lines.txt", "w+"); // w+ 表示读写模式
    fputs("Hello Gemini!\n", fp);
    fputs("C programming is fun.\n", fp);
    rewind(fp); // 回到文件开头准备读取
    while (fgets(buffer, sizeof(buffer), fp) != NULL) {
        printf("读取行: %s", buffer);
    }
    fclose(fp);
    return 0;
}    
// 格式化读: int fscanf(FILE *stream, const char *format, ...);
// 格式化写: int fprintf(FILE *stream, const char *format, ...);
int main() {
    char name[20];
    int age;
    // 格式化写入
    FILE *fp = fopen("data.txt", "w");
    fprintf(fp, "%s %d", "Alice", 25);
    fclose(fp);
    // 格式化读取
    fp = fopen("data.txt", "r");
    fscanf(fp, "%s %d", name, &age);
    printf("从文件读到: 姓名=%s, 年龄=%d\n", name, age);
    fclose(fp);
    return 0;
}    
// 按块读: size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
// 按块写: size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
// size 是单个元素大小，nmemb 是元素个数
int main() {
    int numbers[] = {10, 20, 30, 40, 50};
    int read_nums[5];
    // 写入二进制数组
    FILE *fp = fopen("numbers.bin", "wb");
    fwrite(numbers, sizeof(int), 5, fp);
    fclose(fp);
    // 读取二进制数组
    fp = fopen("numbers.bin", "rb");
    fread(read_nums, sizeof(int), 5, fp);
    printf("读取的第三个数字是: %d\n", read_nums[2]);
    fclose(fp);
    return 0;
}    
-------------------------------------------------------------------------------- // 移动文件指针: int fseek(FILE *stream, long offset, int whence);whence 可选：SEEK_SET(开头), SEEK_CUR(当前), SEEK_END(末尾)   
// 获取当前位置: long ftell(FILE *stream);返回距离文件开头的字节数   
// 重置指针: void rewind(FILE *stream);将指针移回文件开头
int main() {
    FILE *fp = fopen("pos.txt", "w+");
    fputs("ABCDE", fp);
    // 1. ftell: 告诉我现在在哪
    long pos = ftell(fp);
    printf("当前指针位置: %ld\n", pos); // 应该是 5
    // 2. fseek: 移动到距离开头 2 字节的地方
    fseek(fp, 2, SEEK_SET); 
    printf("移动后读到的字符: %c\n", fgetc(fp)); // 应该是 'C'
    // 3. rewind: 直接滚回开头
    rewind(fp);
    printf("重置后读到的字符: %c\n", fgetc(fp)); // 应该是 'A'
    fclose(fp);
    return 0;
}      
// 检测结束: int feof(FILE *stream);如果已到达文件末尾，返回非零值
int main() {
    FILE *fp = fopen("test.txt", "r");
    if (fp == NULL) return 1;
    fgetc(fp); // 尝试读取
    if (feof(fp)) {
        printf("已经到达文件末尾。\n");
    } else if (ferror(fp)) {
        printf("读取过程中发生错误。\n");
    } else {
        printf("文件一切正常，还没读完。\n");
    }
    fclose(fp);
    return 0;
}

```



#### 2️⃣ ==倒序读取文件==

```c
// 频繁更改文件指针的位置，效率较低，更推荐使用fread()结合字符数组来倒序遍历数组
#include <stdio.h>
int main() {
    FILE *fp = fopen("txt1.txt", "r");
    if (fp == NULL) return 1;
    // 1. 定位到末尾获取总长度
    fseek(fp, 0, SEEK_END); 
    long last = ftell(fp);
    // 2. 从 1 循环到 last，确保覆盖所有字符
    for (int cnt = 1; cnt <= last; cnt++) { 
        fseek(fp, -cnt, SEEK_END); // 每次都从末尾往回跳 cnt 个字节
        int ch = fgetc(fp);
        putchar(ch);
    }
    fclose(fp);
    return 0;
}
```



#### 3️⃣ ==编程练习==

```c
//2. 编写一个文件复制程序。程序需要从命令行获得源文件名和目的文件名。尽可能使用标准IO和二进制模式。
#include <stdio.h>
#include <stdlib.h> // 为了使用 exit()
int main(int argc, char* argv[]) {
    // 1. 检查参数
    if (argc != 3) {
        fprintf(stderr, "用法: %s <源文件> <目的文件>\n", argv[0]);
        exit(1);
    }
    // 2. 以二进制模式打开文件
    FILE* fp1 = fopen(argv[1], "rb");
    FILE* fp2 = fopen(argv[2], "wb");
    if (fp1 == NULL || fp2 == NULL) {
        perror("文件打开失败"); // 打印具体的错误原因
        exit(1);
    }
    char buf[1024]; // 建议缓冲区可以大一些，比如 1024 或 4096
    size_t bytesRead; // 用于记录每次实际读到的字节数
    // 3. 循环读写
    // fread 的返回值是成功读取的元素个数
    while ((bytesRead = fread(buf, 1, sizeof(buf), fp1)) > 0) {
        fwrite(buf, 1, bytesRead, fp2); // 写入的长度必须等于实际读到的长度 bytesRead
    }
    // 4. 关闭资源
    fclose(fp1);
    fclose(fp2);
    printf("文件复制成功！\n");
    return 0;
}
---------------------------------------------------------------------------------
//3.编写一个文件复制程序，提示用户输入源文件名和输出文件名。在向输出文件写入时，程序应当使用ctype.h中定义的toupper()函数将所有的文本转换成大写。使用标准 I/O 和文本模式。
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>  // 必须包含，用于 toupper
int main() {
    char src_name[256], dest_name[256];
    char buf[1024];
    // 1. 提示用户输入（响应题目要求）
    printf("请输入源文件名: ");
    scanf("%255s", src_name);
    printf("请输入目标文件名: ");
    scanf("%255s", dest_name);
    // 2. 以文本模式打开
    FILE* fp1 = fopen(src_name, "r");
    FILE* fp2 = fopen(dest_name, "w");
    if (fp1 == NULL || fp2 == NULL) {
        perror("文件操作失败");
        exit(1);
    }
    // 3. 处理逻辑
    while (fgets(buf, sizeof(buf), fp1) != NULL) {
        for (int i = 0; buf[i] != '\0'; i++) {
            // 将字符转换为大写并写回数组,在调用 toupper 时，习惯上会将字符强转为 unsigned 				char。这是为了防止处理非 ASCII 字符（如某些扩展字符）时出现负数导致的未定义行为。
            buf[i] = (char)toupper((unsigned char)buf[i]);
        }
        fputs(buf, fp2);
    }
    // 4. 关闭文件
    fclose(fp1);
    fclose(fp2);
    printf("转换并复制完成！\n");
    return 0;
}
---------------------------------------------------------------------------------
//7. 编写一个打开两个文件的程序。可以使用命令行参数或者请求用户输入来获得文件名。
//a.让程序打印第一个文件的第一行、第二个文件的第一行、第一个文件的第二行、第二个文件的第二行，依此类推，直到打印完行数较多的文件的最后一行。
//b.修改程序，把行号相同的行打印到同一行上。
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char* argv[]) {
    if (argc != 3) {
        fprintf(stderr, "用法: %s <文件1> <文件2>\n", argv[0]);
        exit(1);
    }
    FILE *fp1 = fopen(argv[1], "r");
    FILE *fp2 = fopen(argv[2], "r");
    if (fp1 == NULL || fp2 == NULL) {
        perror("打开文件失败");
        exit(1);
    }
    char line1[1024], line2[1024];
    char *res1, *res2;
    // 只要其中一个文件还能读出内容，就继续循环
    while (1) {
        res1 = fgets(line1, sizeof(line1), fp1);
        res2 = fgets(line2, sizeof(line2), fp2);
        // 如果两个都读完了,退出循环,if(line1) 永远为真（非 NULL），它并不能判断这一次 fgets 是否真的读到了数据。
        if (res1 == NULL && res2 == NULL) break;
        if (res1 != NULL) printf("文件1: %s", line1);
        if (res2 != NULL) printf("文件2: %s", line2);
    }
    fclose(fp1);
    fclose(fp2);
    return 0;
}  
---------------------------------------------------------------------------------
//10. 编写一个程序，打开一个文本文件，文件名通过交互方式获得。建立一个循环，请求用户输入一个文件位置。然后程序打印文件中从该位置开始到下一换行符之间的部分。用户通过输入非数字字符来终止输入循环。
#include <stdio.h>
#include <stdlib.h>
int main() {
    char name[81];
    long pos; // 文件位置建议使用 long，因为文件可能很大
    char line[1024];
    // 1. 交互式获取文件名
    printf("请输入要打开的文件名: ");
    if (scanf("%80s", name) != 1) return 1;
    FILE* fp = fopen(name, "r");
    if (fp == NULL) {
        perror("无法打开文件");
        return 1;
    }
    // 2. 建立循环，请求位置
    printf("请输入文件位置（输入非数字字符退出）: ");
    while (scanf("%ld", &pos) == 1) { // 当 scanf 成功读取一个整数时返回 1，否则跳出循环 
        // 3. 定位到指定偏移量
        if (fseek(fp, pos, SEEK_SET) != 0) {// fseek定位成功返回0
            printf("偏移量超出范围或无效。\n");
        } else if(fgets(line, sizeof(line), fp) != NULL){
            // 4. 读取到下一个换行符
            printf("从位置 %ld 开始的内容: %s", pos, line);
        } else {
            printf("已到达文件末尾，无法读取内容。\n");
        }
        printf("\n请输入下一个位置（或输入非数字字符退出）: ");
    }
    // 5. 善后处理
    fclose(fp);
    printf("程序已结束。\n");
    return 0;
}
---------------------------------------------------------------------------------- 
//11. 编写一个程序，接受两个命令行参数。第一个参数为一个字符串；第二个为文件名。程序打印文件中包含该字符串的所有行。因为这一任务是面向行而不是面向字符的，所以要使用fgets()而不是getc()。使用标准C库函数strstr在每一行中搜索这一字符串。
#include <stdio.h>
#include <stdlib.h>
#include <string.h> // 必须包含，为了使用 strstr
int main(int argc, char* argv[]) {
    // 1. 参数检查
    if (argc != 3) {
        fprintf(stderr, "用法: %s <搜索字符串> <文件名>\n", argv[0]);
        exit(1);
    }
    char line[1024];
    // 2. 打开文件
    FILE* fp = fopen(argv[2], "r");     
    if (fp == NULL) {
        perror("打开文件失败");
        exit(1);
    }
    // 3. 核心逻辑：面向行的搜索
    while ((fgets(line, sizeof(line), fp)) != NULL) {
        // strstr 如果找到目标字符串，返回该位置的指针（非0即真）
        if (strstr(line, argv[1])) {
            printf("%s", line); // fgets 保留了换行符，所以直接打印即可
        }
    }
    fclose(fp);
    return 0;
}   
```



## 第 14 章 结构和其他数据形式

#### 1️⃣ ==建立结构声明==

```c
// 声明
struct book{
    char title[256];
    char author[256];
    float value;
};
// 定义结构变量
struct book dickens; // 变量dickens
struct book library = {"the pirate and the devious damsel","renee vivotte",1.95};
scanf("%f",&library.value);
```



#### 2️⃣ ==结构体数组==

```c
// 结构体数组本质上是内存中连续分布的相同结构体块
#include <stdio.h>
struct Student {
    char name[20];
    int age;
    float score;
};
int main() {
    // 声明并初始化一个包含 3 个元素的结构体数组
    struct Student classA[3] = {
        {"Alice", 18, 92.5},
        {"Bob", 19, 88.0},
        {"Charlie", 18, 95.0}
    };    
    // 访问方式：数组名[下标].成员名
    printf("第二个学生的名字: %s\n", classA[1].name);
    return 0;
}
```



#### 3️⃣ ==嵌套结构体==

```c
//有时我们不是直接嵌套结构体,而是嵌套结构体指针.这在处理“链表”或“树”这种复杂数据结构时经常用到
//直接嵌套：struct Date birthday; —— 空间直接分配在父结构体内
//指针嵌套：struct Date *ptr_birthday; —— 父结构体只存一个地址，具体数据存放在内存的其他地方（通常是堆区）
---------------------------------------------------------------------------------
#include <stdio.h>
// 1. 先定义“小”结构体
struct Date {
    int year;
    int month;
    int day;
};
// 2. 再定义“大”结构体，并将 Date 嵌入其中
struct Employee {
    int id;
    char name[20];
    struct Date birthday; // 嵌套在此
};
int main() {
    // 3. 初始化：使用嵌套的大括号
    struct Employee emp = {
        1001, 
        "张三", 
        {1995, 5, 20} // 对应 birthday 成员
    };
    // 4. 访问方式：连用点号（.）
    printf("员工姓名：%s\n", emp.name);
    printf("出生年份：%d\n", emp.birthday.year);
    return 0;
}
```



#### 4️⃣ ==指向结构体的指针==

```c
#include <stdio.h>
#include <string.h>
struct Book {
    char title[50];
    float price;
};
int main() {
    struct Book myBook = {"C Programming", 45.5};
    struct Book *ptr = &myBook; // 定义指针并指向结构体变量
    // 方式 A：解引用法（比较繁琐，需注意优先级）
    printf("书名: %s\n", (*ptr).title);
    // 方式 B：箭头操作符（推荐！直观、简洁）
    printf("书名: %s, 价格: %.2f\n", ptr->title, ptr->price);
    return 0;
}
```



#### 5️⃣ ==向函数传递结构信息==

```c
// 1.传递结构体成员
struct Student { 
    char name[20]; 
    float score; 
};
void checkPass(float s) {
    if (s >= 60) printf("及格\n");
}
int main(){
    struct Student stu1 = {"weiwei", 95};
    checkPass(stu1.score);
    return 0;
}
----------------------------------------------------------------------------------
// 2.传递结构体变量（值传递）
void printStudent(struct Student s) {
    printf("姓名: %s, 分数: %.1f\n", s.name, s.score);
}
----------------------------------------------------------------------------------
// 3.传递结构体指针（地址传递）
struct Vector {
    int x;
    int y;
};
// 传指针：为了修改原始值
void movePoint(struct Vector *v, int dx, int dy) {
    v->x += dx;
    v->y += dy;
}
// 传指针 + const：为了效率且禁止修改
void showPoint(const struct Vector *v) {
    // v->x = 10; // 如果这行取消注释，编译器会报错，保护了数据
    printf("当前坐标: (%d, %d)\n", v->x, v->y);
}
int main() {
    struct Vector pos = {10, 20};    
    movePoint(&pos, 5, 5); // 传地址
    showPoint(&pos);       // 传地址  
    return 0;
}    
```



#### 6️⃣ ==结构体拷贝==

```c
// 同类型结构体：可以直接赋值
// 包含数组的结构体：数组会自动被完整拷贝
// 包含指针的结构体：慎用直接赋值，可能需要手动进行“深拷贝”
#include <stdio.h>
#include <string.h>
struct Book {
    char title[50];
    float price;
};
int main() {
    struct Book b1 = {"C Primer Plus", 60.0f};
    struct Book b2;
    // 直接赋值
    b2 = b1; 
    printf("b2 的书名: %s, 价格: %.2f\n", b2.title, b2.price);    
    // 修改 b2 不会影响 b1
    b2.price = 50.0f;
    printf("修改后 b1 价格: %.2f, b2 价格: %.2f\n", b1.price, b2.price);
    return 0;
}  
----------------------------------------------------------------------------------
// 结构、指针和 malloc()
// 啥时候用数组 -> 数据长度固定(如身份证号、邮编),或者你需要直接把结构体保存到二进制文件时
// 啥时候用指针+malloc -> 数据长度差异很大(如文章标题、用户评论),或者你想在程序运行期间节省内存
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
struct Record {
    int id;
    char *description; // 使用指针以适应不同长度
};
int main() {
    struct Record *item;
    char temp[1024] = "This is a very long description...";
    // 1. 为结构体本身申请内存
    item = (struct Record *)malloc(sizeof(struct Record));
    item->id = 7;
    // 2. 为结构体内的指针成员申请内存
    // 长度为字符串长度 + 1 (为了 '\0')
    item->description = (char *)malloc(strlen(temp) + 1);
    strcpy(item->description, temp);
    printf("ID: %d, Desc: %s\n", item->id, item->description);
    // 3. 释放内存的顺序极其重要！像拆迁一样，先拆里面，再拆地基
    free(item->description); // 先拆内部指针
    free(item);              // 后拆整个结构体
    return 0;
}    
```



#### 7️⃣ ==联合体 union==

```c
// union:所有成员都从同一个内存地址开始存储,联合体的大小等于它最大的成员的大小
#include <stdio.h>
union Data {
    int i;
    float f;
    char str[20];
};
int main() {
    union Data data;    
    // 整个联合体占用 20 字节（str[20] 是最大成员）
    printf("占用内存大小: %zu\n", sizeof(data)); 
    data.i = 10;
    data.f = 220.5; // 这一步会覆写掉之前的 i    
    // 此时 i 的值已经损坏，因为内存被 f 占用了
    printf("data.i: %d (已被覆写)\n", data.i);     
    return 0;
}
```



#### 8️⃣ ==枚举类型 enum==

```c
// enum:允许你为一组整数常量定义易读的符号名称,用有意义的单词代替枯燥的数字,从而让代码从“写给机器看”变成“写给人看”
----------------------------------------------------------------------------------
#include <stdio.h>
// 1. 定义枚举类型
enum Direction {
    UP,     // 默认值为 0
    DOWN,   // 默认值为 1
    LEFT,   // 默认值为 2
    RIGHT   // 默认值为 3
};
int main() {
    // 2. 声明枚举变量
    enum Direction playerDir = UP;
    if (playerDir == UP) {
        printf("玩家正在向上移动\n");
    }
    // playerDir = 100; // ❌语法上竟然允许！
    printf("UP 的数值是: %d\n", UP); // 枚举本质上是整数,输出 0    
    return 0;
}
```



#### 9️⃣ ==typedef==

```c
// typedef:给已有的数据类型起别名
----------------------------------------------------------------------------------
// 1.简化结构体定义
typedef struct Student Student;
Student stu1; // 简洁明了
// 2.提高代码可移植性
typedef double Money;
// 3.简化复杂的指针声明
typedef void (*Handler)(int);// 定义一个名为 Handler 的类型，它是一个指向“接收 int 返回 void”的函数的指针
Handler func; // 现在定义函数指针就像定义 int 一样简单
```



#### 🔟 ==函数指针==

```c
// 函数指针:变量在内存里有地址,函数在内存里也有地址,函数指针就是一个保存了函数起始地址的变量。
// 定义函数指针的公式 -> 返回类型 (*指针名)(参数列表);
----------------------------------------------------------------------------------
#include <stdio.h>
void sayHello(int times) {
    for(int i = 0; i < times; i++) printf("Hello! ");
}
int main() {
    // 1. 定义一个函数指针 fptr
    void (*fptr)(int);
    // 2. 将函数的地址赋给指针,函数名本身就是地址,写 fptr= &sayHello 也可以,但通常直接写名字
    fptr = sayHello;
    // 3. 通过指针调用函数,就像普通调用一样,或者写 (*fptr)(3);
    fptr(3); 
    return 0;
}
----------------------------------------------------------------------------------
// 回调函数,定义一个排序函数,最后一个参数是“比较逻辑”的指针
void sort(int *arr, int n, int (*compare)(int, int)) {
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (compare(arr[i], arr[j]) > 0) {
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
}
// 具体的比较逻辑 A
int ascending(int a, int b) { return a - b; }
// 具体的比较逻辑 B
int descending(int a, int b) { return b - a; }
// 使用时：
// sort(nums, 5, ascending);  // 升序
// sort(nums, 5, descending); // 降序
```



#### 1️⃣1️⃣ ==编程练习==

```c
//11. 编写一个transform()函数，它接受4个参数：包含double类型数据的源数组名，double类型的目标数组名，表示数组元素个数的int 变量以及一个函数名（或者，等价的指向函数的指针)。transform()函数把指定的函数作用于源数组的每个元素，并将返回值放到目标数组中。例如:transform (source, target, 100, sin);这个函数调用把sin(source[0])赋给target[0],等等，共有100个元素。在一个程序中测试该函数，调用4次transform()，分别使用math.h函数库中的两个函数以及自已设计的两个适合的函数作为参数。
void transform(double source[],double target[],int num,double (*funptr) (double x)){
    for(int i = 0;i < num;i++){
        target[i] = funptr(source[i]);
    }
}
```



## 第 15 章 位操作

#### 1️⃣ ==位逻辑运算符==

```c
// 假设 unsigned char a = 6; (二进制00000110),unsigned char b = 3; (二进制 00000011)
// 1. a & b (与)：
  00000110 (6)
& 00000011 (3)
-----------
  00000010 (结果为 2)
// 2. a | b (或)：
  00000110 (6)
| 00000011 (3)
-----------
  00000111 (结果为 7)   
// 3. a ^ b (异或)：
  00000110 (6)
^ 00000011 (3)
-----------
  00000101 (结果为 5)     
----------------------------------------------------------------------------------
// 场景 A：掩码操作（用 & 检查某一位）
// 如果你想知道一个整数的第 3 位是不是 1：
if (data & 0x08) { // 0x08 二进制是 00001000
    printf("第 3 位是开着的！");
}

// 场景 B：打开特定位（用 | 设置位）
// 如果你想强制把第 3 位设为 1，而不影响其他位：
data = data | 0x08;

// 场景 C：翻转状态（用 ^ 切换开关）
// 如果第 3 位是 1 就变 0，是 0 就变 1：
data = data ^ 0x08;

// 场景 D：异或交换律（黑科技）
// 归零律：a ^ a = 0（自己异或自己，结果为 0）
// 恒等律：a ^ 0 = a（任何数异或 0，结果还是它自己）
// 交换律与结合律：a ^ b ^ c = a ^ (b ^ c)（顺序不重要）
// 不需要第三个变量就能交换两个整数：
a = a ^ b; // a = a ^ b;
b = a ^ b; // b = (a ^ b) ^ b = a ^ (b ^ b) = a
a = a ^ b; // a = (a ^ b) ^ a = (a ^ a) ^ b = b

// 场景 E：汉明重量计算
int cnt_bit_fast(int x) {
    int cnt = 0;
    while (x != 0) {
        // 这个位运算操作会消去 x 最右边的一个 1
        x &= (x - 1); 
        cnt++;
    }
    return cnt;
}
```



#### 2️⃣ ==移位运算符==

```c
// 1.左移:全部位向左移，低位补0，高位溢出丢弃,每左移1位,相当于乘以2(在不溢出的情况下)
// 例子1：基本用法
unsigned char a = 6;       // 二进制 00000110
unsigned char b = a << 2;  // 二进制 00011000，即 24
printf("%d\n", b);         // 输出 24
// 例子2：溢出警告
unsigned char c = 200;     // 二进制 11001000
unsigned char d = c << 1;  // 左移1位，实际：10010000,高位110溢出,结果 144，并非200×2=400（超出255范围）
--------------------------------------------------------------------------
// 2.右移
// 无符号整数右移,规则:左边统一补0 ; 效果:每右移1位，相当于除以2(向下取整)
unsigned char a = 12;      // 二进制 00001100
unsigned char b = a >> 2;  // 二进制 00000011，即 3
printf("%d\n", b);         // 输出 3
// 有符号整数右移,规则:左边补符号位(正数补0,负数补1),以保持正负不变
signed char c = -8;     // -8 的补码(8位)：11111000        
signed char d = c >> 2; // 右移后：11111110，即 -2 的补码    
printf("%d\n", d);         // 输出 -2
```



#### 3️⃣ ==位字段==

```c
// 除了使用位运算符来操控二进制位,还可以使用位字段
// 位字段是C语言结构体的一种特殊成员,它允许你直接把一个整数内部的不同位段当成独立变量来读写
----------------------------------------------------------------------------
// 没有 &、没有 |、没有移位数错乱的烦恼——语义非常清晰。
// 一个完整例子:用 1 个字节描述灯的 8 个状态
struct LampStatus {
    unsigned char bedroom   : 1;  // 卧室灯，1位（0或1）
    unsigned char kitchen   : 1;  // 厨房灯
    unsigned char bathroom  : 1;  // 卫生间灯
    unsigned char living    : 1;  // 客厅灯
    unsigned char hallway   : 1;  // 走廊灯
    unsigned char reserved  : 3;  // 保留位，占3位，不用
};
int main() {
    struct LampStatus lamps;    
    // 赋值：像布尔值一样使用
    lamps.bedroom  = 1;   // 开卧室灯
    lamps.kitchen  = 0;   // 关厨房灯
    lamps.bathroom = 1;
    lamps.living   = 0;
    lamps.hallway  = 1;
    lamps.reserved = 0;   // 保留位通常置0
    // 读取和判断
    if (lamps.bedroom) {
        printf("卧室灯亮着\n");
    }    
    printf("结构体占用 %d 字节\n", sizeof(lamps)); // 输出 1
    return 0;
}
```



#### 4️⃣ ==编程练习==

```c
//1．编写一个将二进制字符串转化为数字值的函数。也就是说，如果您有以下语句:char* pbin = "01001001";那么您可以将pbin 作为一个参数传送给该函数，使该函数返回一个int 值 73。
int stoi(const char* str){
    int val = 0;
    while(*str != '\0'){
        val = val << 1;
        if(*str == '1'){
            val += 1;
        }
        str++;
    }
    return val;
}
---------------------------------------------------------------------------- 
//3. 编写一个函数，该函数接受一个 int 参数，并返回这个参数中打开的位的数量。
int cnt_bit_fast(int x) {
    int cnt = 0;
    while (x != 0) {
        // 这个位运算操作会消去 x 最右边的一个 1
        x &= (x - 1); 
        cnt++;
    }
    return cnt;
}
----------------------------------------------------------------------------
//4. 编写一个函数，该函数接受两个int参数：一个值和一个位的位置。如果指定的位上的值是1，则该函数返回1，否则返回0。
int chkone(int n,int pos){
    if(pos >= sizeof(int)*8 || pos < 0) return 0;
    int x = 1;
    return (n & (x << pos)) ? 1 : 0;
}
----------------------------------------------------------------------------
//5. 编写一个函数，该函数将一个unsigned int中的所有位向左旋转指定数量的位。例如，rotate(x,4)将x中的所有位向左移动4个位置，而且从左端丢失的位会重新出现在右端。也就是说，把从高位移出的位放入低位。 
// result = (x << n) | (x >> (W - n))
unsigned int rotate(unsigned int x,int n){
    unsigned int total_bits = sizeof(unsigned int) * 8;
    // 1. 预处理：防止旋转次数大于总位数,例如旋转 33 位等同于旋转 1 位
    n = n % total_bits;
	// 2. 特殊情况：如果旋转 0 位，直接返回
    if (n == 0) return x;
    
    return (x << n) | (x >> (total_bits - n));    
}
```



## 第 16 章 C 预处理器和 C 库

#### 1️⃣ ==宏 define==

```c
// 1. 无参数宏：定义常量
#define PI 3.14159
#define MAX_BUFFER_SIZE 1024
----------------------------------------------------------------------------
// 2. 带参数宏：模拟函数,无函数调用开销
#define SQUARE(x) ((x) * (x))
int res = SQUARE(5); // 预处理后变成: int res = ((5) * (5));
----------------------------------------------------------------------------
```



#### 2️⃣ ==文件包含 #include==

```c
// 当编译器看到 #include时,预处理器会把目标文件的所有内容原封不动粘贴到当前指令所在的位置
----------------------------------------------------------------------------
// 1. #include <stdio.h>,编译器会直接去系统标准库目录(如Linux下的 /usr/include)查找 // 2. #include "my_header.h"(用户自定义头文件),编译器先在当前源文件所在的目录查找  
// 3. 头文件里应该写什么?
//      宏定义：#define PI 3.14
//      函数声明：int add(int a, int b); 
//	    结构体定义：struct Player { ... };
//	    外部变量声明：extern int global_score;
//注意:永远不要在头文件里写函数的具体实现(除非是 static inline),否则如果有多个文件包含了这个头文件,链接时会报“重复定义”错误
// 4. 防止重复包含:文件 A include文件 B,文件 C 同时也include A 和 B.这时文件 B 的内容会在文件 C 中出现两次,导致编译错误,为了解决这个问题,所有的头文件都必须加上“保护锁”：
#pragma once // 防止重复包含
// 头文件内容
void hello();
```



#### 3️⃣ ==其他指令==

```c
// 1. #undef指令:如果#define是创建,那么#undef就是它的撤销键,在C语言中,#undef指令用于删除之前定义的宏
#define LIMIT 100
...
#undef LIMIT
// 从这里开始，LIMIT 不再有任何意义
---------------------------------------------------------------------------
// 2. 条件编译:根据特定的条件,决定代码中的某一部分是否参与编译
// 核心指令:#if,#elif,#else,#endif   
#define LEVEL 2
#if LEVEL == 1
    printf("初级版本\n");
#elif LEVEL == 2 
    printf("中级版本\n");
#else
    printf("高级版本\n");
#endif
// 宏存在检查: #ifdef,#ifndef --> 用来检查某个宏是否被定义过
#define DEBUG
#ifdef DEBUG
    printf("Debug: 指针地址为 %p\n", ptr);
#endif
----------------------------------------------------------------------------
// 3. 预定义宏
// __DATE__	字符串	编译时的日期（"Mmm dd yyyy"）  
// __TIME__	字符串	编译时的时间（"hh:mm:ss"）
// __FILE__	字符串	当前源文件的全路径/文件名
// __LINE__	整型	 当前代码行号
// __STDC__	整型	 如果遵循 ANSI C 标准，其值为 1
// __func__	字符串 当前所在函数的函数名
#include <stdio.h>
void test_function(void) {
    printf("--- 进入函数: %s ---\n", __func__);// __func__ 获取当前函数名
    #if __STDC__ == 1     // 检查编译器是否遵循 ANSI C 标准
        printf("编译器状态: 严格遵循 ANSI C 标准 (__STDC__ = %d)\n", __STDC__);
    #else
        printf("编译器状态: 非标准 C 环境\n");
    #endif
    // 模拟一个警告或错误日志
    printf("日志埋点 - 文件: %s\n", __FILE__);
    printf("日志埋点 - 行号: %d\n", __LINE__);   
    printf("--- 退出函数: %s ---\n", __func__);
}
int main() {
    printf("==========================================\n");
    printf("系统启动信息:\n");    
    // 使用 __DATE__ 和 __TIME__ 记录编译版本时间
    printf("程序版本编译日期: %s\n", __DATE__);
    printf("程序版本编译时间: %s\n", __TIME__);
    printf("==========================================\n\n");
    test_function();
    return 0;
}
```



#### 4️⃣ ==内联函数==

```c
// 内联函数是对编译器的一种建议:当你把一个函数声明为 inline 时,编译器可能把它的代码嵌入到调用它的地方,不会再浪费时间去进行压栈、跳转和返回了
static inline int square(int x) {
    return x * x;
}
int main() {
    int res = square(5); 
    // 编译器可能会直接把这里替换成：int res = 5 * 5;
    return 0;
}
----------------------------------------------------------------------------
// 编译器在调用处需要知道函数的完整代码才能进行“嵌入”。因此，内联函数通常定义在头文件(.h) 中，并加上 static 修饰符：
// math_utils.h
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}
// 如果不加 static 且多个 .c 文件包含了它，可能会引发链接时的重复定义错误。
```



#### 5️⃣ ==string.h 库中的 memcpy() 与 memmove()==

```c
// memcpy() 和 memmove() 都是用来进行内存拷贝的函数,但在处理内存重叠时有区别
// 1. 函数原型
void *memcpy(void *dest, const void *src, size_t n);
void *memmove(void *dest, const void *src, size_t n);
// 2. memcpy():假设 src 区域和 dest 区域完全没有交集,通常从头到尾按顺序拷贝;如果有重叠,会导致数据污染,其行为在 C 标准中是未定义的。
// 3. memmove():先检查 src 和 dest 是否重叠,如果发现重叠且 dest 在 src 之后,它会选择从后往前（反向）拷贝;如果没有重叠或 dest 在 src 之前,则正常正向拷贝。
---------------------------------------------------------------------------
// 目标：把 "123" 拷贝到从 "3" 开始的位置
int s = {1,2,3,4,5,6,7,8};
memcpy(s + 2, s, 3); 
// 1. 把 '1' 拷给 '3' -> s: 1 2 1 4 5 6 7 8
// 2. 把 '2' 拷给 '4' -> s: 1 2 1 2 5 6 7 8
// 3. 此时原本的 '3' 已经变成 '1' 了！
// 结果可能是：1 2 1 2 1 6 7 8 (数据错误)
memmove(s + 2, s, 3);
// 它会从后往前拷贝，先拷 '3'，再拷 '2'，最后 '1'。
// 结果：1 2 1 2 3 6 7 8 (数据正确)
----------------------------------------------------------------------------
// 寿司 memmove()
void* my_memmove(void* tar, const void* src, size_t n) {
    // 1. 如果地址相同或者长度为0，直接返回
    if (tar == src || n == 0) return tar;
    // 2. 转换为字节指针，以便进行按字节拷贝
    unsigned char* d = (unsigned char*)tar;
    const unsigned char* s = (const unsigned char*)src;
    if (d > s && d < s + n) {  // 情况：目标在源之后且有重叠 -> 必须反向拷贝
        for (size_t i = n; i > 0; i--) {
            d[i - 1] = s[i - 1];
        }
    } else {  // 情况：目标在源之前，或者完全不重叠 -> 正向拷贝
        for (size_t i = 0; i < n; i++) {
            d[i] = s[i];
        }
    }
    return tar;
}
```
