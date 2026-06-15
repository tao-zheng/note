# 第01章附录：MinGW编译器的安装和配置

## 1、MinGW介绍

MinGW（Minimalist GNU on Windows）实际上是GCC在Windows平台上的移植版本，因此可以将源代码编译为可在 Windows 中运行的可执行程序。MinGW是，主要用于在Windows上本地编译C和C++代码。


> 目前适用于 Windows 平台、受欢迎的 GCC 移植版主要有 2 种，分别为 MinGW 和 Cygwin。
> 其中：
>
> MinGW 侧重于服务 Windows 用户可以使用 GCC 编译环境，是真正的GCC移植，相比后者体积更小，使用更方便；
> Cygwin 只是提供一个类unix的环境内部是原生的GCC，借助它不仅可以在 Windows 平台上使用 GCC 编译器，理论上可以运行 Linux 平台上所有的程序；
>
> Cygwin 只是提供一个类unix的环境内部是原生的GCC。
>
> 如果读者仅需要在 Windows 平台上使用 GCC，可以使用 MinGW 或者 Cygwin；除此之外，如果还有更高的需求（例如运行 POSIX 应用程序），就只能选择安装 Cygwin。

## 2、下载与安装

### 2.1 下载

下载地址：<https://sourceforge.net/projects/mingw/files/>

点击“Download Latest Version”即可==

![1692674344122](images/1692674344122.png)

### 2.2 安装

下载完成后，会得到一个名为 mingw-get-setup.exe 的安装包，双击打开它，可以看到如下的对话框：

![1692674777924](images/1692674777924.png)

直接点击“Install”，进入下面的对话框：

![1692674793945](images/1692674793945.png)

读者可根据自己操作系统的实际情况，自定义 MinGW 的安装位置（`建议安装到非C盘的指定目录下`），然后点击“continue”，进入下面的对话框：

![1692674846199](images/1692674846199.png)

​	进入安装 MinGW 配置器的界面，耐心等待安装完成（显示 100%）即可。

![1692675129933](images/1692675129933.png)

安装完成之后，继续点击“continue”，进入下面的对话框，这是一个名为 “`MinGW Installer Manager`” 的软件，借助它，我们可以随时根据需要修改 GCC 编译器的配置。

![1692675199055](images/1692675199055.png)

常见的安装包介绍如下。

| 安装包名称          | 作用                                             |
| ------------------- | ------------------------------------------------ |
| mingw32-binutils    | 用于编译生成的 .o 文件的链接、汇编、生成静态库等 |
| mingw32-gcc         | 核心的 C 编译器                                  |
| mingw32-gcc-ada     | Ada 编译器                                       |
| mingw32-gcc-fortran | Fortran 编译器                                   |
| mingw32-gcc-g++     | C++ 编译器                                       |
| mingw32-gcc-objc    | Objective-C 编译器                               |
| mingw32-libgcc      | C 编译器编译出来的程序的运行库                   |

其中minw32-gcc-g++支持C++编译和minw32-gcc支持C编译。

![1692693959356](images/1692693959356.png)

为使 GCC 同时支持编译 `C` 语言和 `C++`，需勾选上图中标注的 2 项。选中其中一项，鼠标右键点击，选择“Mark for Installation”，如图所示。

![1692675410414](images/1692675410414.png)

标记完以后如图所示。

![1692698300912](images/1692698300912.png)

> GCC 还支持其它编程语言，读者可借助此配置器，随时根据需要安装自己需要的编译环境。

勾选完成后，在菜单栏中选择 `Installation -> Apply Changes`，

![1692675512853](images/1692675512853.png)

弹出如下对话框：

![1692675528090](images/1692675528090.png)

选择“Apply”。然后耐心等待，直至安装成功，即可关闭此界面。

![1692675556189](images/1692675556189.png)

## 3、配置：path环境变量

在安装完成的基础上，我们需要手动配置 PATH 环境变量。

1）依次 `右击计算机（我的电脑） -> 属性 -> 高级系统设置 -> 环境变量`，例如我将其安装到了C:\MinGW文件夹中，因此 PATH 环境变量的设置如下：

<img src="images/image-20230822181721360.png" alt="image-20230822181721360" style="zoom:67%;" />

![image-20230822181849566](images/image-20230822181849566.png)



2）打开命令行窗口（通过在搜索栏中执行 cmd 指令即可），输入`gcc -v`指令，如果输出 GCC 编译器的具体信息，则表示安装成功，例如：

![image-20230822182303099](images/image-20230822182303099.png)

通过上面的安装，我们就可以在当前 Windows 平台上编译、运行 C 或者 C++ 程序了。

因为 MinGW-w64 本来就是将 GCC 移植到 Windows 上的产物，所以操作方式和 GCC 一样，只是在 Linux 下命令是被键入到“终端”中，而 Windows 下则是被键入到“命令提示符”里。



# C开发利器：CLion的使用

讲师：尚硅谷-宋红康（江湖人称：康师傅）

官网：[http://www.atguigu.com](http://www.atguigu.com/)

***

## 1. 认识CLion

### 1.1 JetBrains  公司介绍

CLion，是 JetBrains (https://www.jetbrains.com/)公司的产品，该公司成立于2000年，总部位于捷克的布拉格，致力于为开发者打造最高效智能的开发工具。

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692692710108.png" alt="1692692710108" style="zoom:80%;" />

公司旗下的各种产品：

- CLion：用于开发 C/C++
- IDEA：用于开发 Java
- PyCharm：用于开发 python

* WebStorm：用于开发 JavaScript、HTML5、CSS3 等前端技术
* PhpStorm：用于开发 PHP
* RubyMine：用于开发 Ruby/Rails
* AppCode：用于开发 Objective - C/Swift
* DataGrip：用于开发数据库和 SQL
* Rider：用于开发.NET
* GoLand：用于开发 Go

### 1.2 CLion 介绍

Clion是一款专门开发C以及C++所设计的跨平台的集成开发环境(IDE)。它是以IntelliJ为基础设计的，包含了许多智能功能来提高开发人员的生产力。这种强大的IDE帮助开发人员在Linux、OS X和Windows上来开发C/C++，同时它还能使用智能编辑器来提高代码质量、自动代码重构并且深度整合Cmake编译系统，从而提高开发人员的工作效率。

### 1.3 Clion  的下载

- 下载网址： https://www.jetbrains.com/clion/download/#section=windows


- Clion 目前没有 `社区版(Community)`。

![1692685638894](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692685638894.png)

官网提供的详细使用文档：
https://www.jetbrains.com.cn/clion/features/

## 2. 安装

### 2.1 安装前的准备

* 64 位 Microsoft Windows 11、10、8
* 最低 2 GB 可用 RAM，推荐 8 GB 系统总 RAM
* 3.5 GB 硬盘空间，推荐 至少5G的SSD硬盘
* 最低屏幕分辨率 1024x768，推荐1920×1080

### 2.2 安装过程

1、下载完安装包，双击直接安装![1692685747275](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692685747275.png)

2、欢迎安装

![1692685818554](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692685818554.png)

4、选择安装目录

![1692685875043](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692685875043.png)

选择安装目录，目录中要避免中文和空格。

![1692685853085](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692685853085.png)

5、创建桌面快捷图标等

![1692686073729](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686073729.png)

确认是否与.c、.h、.cpp格式文件进行关联。这里建议不关联。

6、在【开始】菜单新建一个文件夹（这里需要确认文件夹的名称），来管理CLion的相关内容。

![1692686114854](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686114854.png)

![1692686134454](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686134454.png)

7、完成安装

![1692686267537](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686267537.png)

8、双击打开

![1692686314524](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686314524.png)

9、选择“Do not import settings”，点击“OK”按钮

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686365691.png" alt="1692686365691" style="zoom:80%;" />

10、如图所示，需要激活CLion

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686434862.png" alt="1692686434862" style="zoom: 80%;" />

### 2.3 注册

- 选择1：适用30天。在CLion版本中，需要先登录，才能开启试用。

  <img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686513431.png" alt="1692686513431" style="zoom:80%;" />

- 选择2：付费购买旗舰版

  <img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686556166.png" alt="1692686556166" style="zoom:80%;" />

  <img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686590056.png" alt="1692686590056" style="zoom: 67%;" />

- 选择3：（推荐）

  - 大家参照《`02-软件\CLion-2022.1.3安装包\CLion注册文档\CLion注册文档.docx`》操作即可。
  - 由于存在时效性，如果失效，大家可以自行搜索注册方式即可。

## 3. HelloWorld的实现

### 3.1 新建Project

选择"New Project"：

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692686679397.png" alt="1692686679397" style="zoom:80%;" />

指定创建C可执行文件、工程目录，图中的“untitled1”需要修改为自己的工程名称。如下所示：

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692687949240.png" alt="1692687949240" style="zoom:80%;" />

选择C可执行文件，修改工程名称为demo1

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692687854607.png" alt="1692687854607" style="zoom:80%;" />

点击“Create”进行下一步，如图所示

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692688033686.png" alt="1692688033686" style="zoom:80%;" />

此处选择编译器，默认MinGW即可，点击“OK”按钮，如图所示，默认创建了main.c文件。

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692688090496.png" alt="1692688090496" style="zoom:67%;" />

### 3.2 运行

点击执行按钮，如下所示

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692688278731.png" alt="1692688278731" style="zoom: 67%;" />



## 4. 详细设置

![image-20230823143234863](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20230823143234863.png)

CLion的设置都在 File - Settings 中进行。

### 4.1 设置整体主题

#### 1、选择主题

![image-20230823143412251](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20230823143412251.png)

#### 2、设置菜单和窗口字体和大小

![image-20230823143554720](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20230823143554720.png)

### 4.2 设置编辑器主题样式

#### 1、字体大小

![1655136907073](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1655136907073.png)

更详细的字体与颜色如下：

![image-20230823144141946](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20230823144141946.png)

> 温馨提示：如果选择某个font字体，中文乱码，可以在fallback font（备选字体）中选择一个支持中文的字体。

#### 2、注释的字体颜色

![image-20220616121435182](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20220616121435182.png)

- Block comment：修改多行注释的字体颜色
- Doc Comment –> Text：修改文档注释的字体颜色
- Line comment：修改单行注释的字体颜色

### 4.3 代码智能提示功能

![1655137649491](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1655137649491.png)

代码提示和补充功能有一个特性：`区分大小写`。 如果想不区分大小写的话，就把这个对勾去掉。`建议去掉勾选`。

### 4.4 设置项目文件编码（一定要改）

![image-20220615190832482](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20220615190832482.png)

说明： Transparent native-to-ascii conversion主要用于转换ascii，显式原生内容。一般都要勾选。

### 4.5 设置控制台的字符编码

![image-20230823144626692](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20230823144626692.png)

## 5. 插件的使用(重要)

**1、为何安装C/C++ Single File Execution插件？**

前面已经创建了一个demo1工程，项目文件夹内存在一个代码文件，名为`main.c`。如果再创建一个C源文件，内部如果也包含main()函数，则会报错！因为默认C工程下只能有一个main()函数。如何解决此问题呢？

2、安装并测试

1）在 File - Settings - Plugins 中搜索 `C/C++ Single File Execution` 插件并安装

![image-20230823145107293](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/image-20230823145107293.png)

2）在需要运行的代码中右键，点击 Add executable for single c/cpp file

![1692774502830](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692774502830.png)

3）此时可以在 Cmakelists.text 文件中看到多出的这一行代码，这就是插件帮我们完成的事情

![1692774556495](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692774556495.png)

4）右键项目文件夹，点击 Reload CMake Project 进行刷新

![1692774575597](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692774575597.png)

5）此时右上角标签处已经增加了我们的文件选项，选择需要的标签

![1692774598633](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692774598633.png)

6）点击小三角，或右键代码处点击 Run 选项，即可运行代码。

![1692774678384](C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692774678384.png)

7）在该工程下创建main2.c文件，文件中的代码如下所示，执行上面相同的步骤。

```c
#include <stdio.h>

int main() {
    printf("Hello, World2!\n");
    return 0;
}
```

可以发现一个工程中允许存在多个main方法了，而且可以独立允许。

## 6. 快捷键的使用

### 6.1 常用快捷键

#### 第1组：通用型

| 说明            | 快捷键           |
| --------------- | ---------------- |
| 复制代码-copy   | ctrl + c         |
| 粘贴-paste      | ctrl + v         |
| 剪切-cut        | ctrl + x         |
| 撤销-undo       | ctrl + z         |
| 反撤销-redo     | ctrl + shift + z |
| 保存-save all   | ctrl + s         |
| 全选-select all | ctrl + a         |

#### 第2组：提高编写速度（上）

| 说明                                               | 快捷键           |
| -------------------------------------------------- | ---------------- |
| 提示代码模板-insert live template                  | ctrl+j           |
| 使用xx块环绕-surround with ...                     | ctrl+alt+t       |
| 调出生成getter/setter/构造器等结构-generate ...    | alt+insert       |
| 自动生成返回值变量-introduce variable ...          | ctrl+alt+v       |
| 复制指定行的代码-duplicate line or selection       | ctrl+d           |
| 删除指定行的代码-delete line                       | ctrl+y           |
| 切换到下一行代码空位-start new line                | shift + enter    |
| 切换到上一行代码空位-start new line before current | ctrl +alt+ enter |
| 向上移动代码-move statement up                     | ctrl+shift+↑     |
| 向下移动代码-move statement down                   | ctrl+shift+↓     |
| 向上移动一行-move line up                          | alt+shift+↑      |
| 向下移动一行-move line down                        | alt+shift+↓      |

#### 第3组：提高编写速度（下）

| 说明                                        | 快捷键       |
| ------------------------------------------- | ------------ |
| 批量修改指定的变量名、方法名、类名等-rename | shift+f6     |
| 抽取代码重构方法-extract method ...         | ctrl+alt+m   |
| 选中的结构的大小写的切换-toggle case        | ctrl+shift+u |

#### 第4组：类结构、查找和查看源码

| 说明                                          | 快捷键        |
| --------------------------------------------- | ------------- |
| 退回到前一个编辑的页面-back                   | ctrl+alt+←    |
| 进入到下一个编辑的页面-forward                | ctrl+alt+→    |
| 打开的类文件之间切换-select previous/next tab | alt+←/→       |
| 定位某行-go to line/column                    | ctrl+g        |
| 回溯变量或方法的来源-go to implementation(s)  | ctrl+alt+b    |
| 折叠方法实现-collapse all                     | ctrl+shift+ - |
| 展开方法实现-expand all                       | ctrl+shift+ + |

#### 第5组：查找、替换与关闭

| 说明                                               | 快捷键       |
| -------------------------------------------------- | ------------ |
| 查找指定的结构                                     | ctlr+f       |
| 快速查找：选中的Word快速定位到下一个-find next     | ctrl+l       |
| 查找与替换-replace                                 | ctrl+r       |
| 直接定位到当前行的首位-move caret to line start    | home         |
| 直接定位到当前行的末位 -move caret to line end     | end          |
| 查询当前元素在当前文件中的引用，然后按 F3 可以选择 | ctrl+f7      |
| 全项目搜索文本-find in path ...                    | ctrl+shift+f |
| 关闭当前窗口-close                                 | ctrl+f4      |

#### 第6组：调整格式

| 说明                                         | 快捷键           |
| -------------------------------------------- | ---------------- |
| 格式化代码-reformat code                     | ctrl+alt+l       |
| 使用单行注释-comment with line comment       | ctrl + /         |
| 使用/取消多行注释-comment with block comment | ctrl + shift + / |
| 选中数行，整体往后移动-tab                   | tab              |
| 选中数行，整体往前移动-prev tab              | shift + tab      |

#### 第7组-Debug快捷键

| 说明                                                  | 快捷键        |
| ----------------------------------------------------- | ------------- |
| 单步调试（不进入函数内部）- step over                 | F8            |
| 单步调试（进入函数内部）- step into                   | F7            |
| 强制单步调试（进入函数内部） - force step into        | alt+shift+f7  |
| 选择要进入的函数 - smart step into                    | shift + F7    |
| 跳出函数 - step out                                   | shift + F8    |
| 运行到断点 - run to cursor                            | alt + F9      |
| 继续执行，进入下一个断点或执行完程序 - resume program | F9            |
| 停止 - stop                                           | Ctrl+F2       |
| 查看断点 - view breakpoints                           | Ctrl+Shift+F8 |
| 关闭 - close                                          | Ctrl+F4       |

### 6.2 查看快捷键

#### 1、已知快捷键操作名，未知快捷键

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692752261713.png" alt="1692752261713" style="zoom:80%;" />

#### 2、已知快捷键，不知道对应的操作名

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692752357783.png" alt="1692752357783" style="zoom: 70%;" />

### 6.3 自定义快捷键

<img src="C:/Users/17630/Downloads/尚硅谷宋红康c语言资料/课件/C语言课件/images/1692752523743.png" alt="1692752523743" style="zoom:80%;" />

































