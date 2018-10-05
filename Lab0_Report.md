# Lab0 实验报告

常朝坤

16307130138

[TOC]



## Lab1调试体验

一般情况下，本机的gdb调试可以直接通过gdb prog 进行调试

![1538720285914](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538720285914.png)

gdb调试的为可执行的目标文件，所以在调试前，需要使用gcc对代码或工程进行编译，产生可执行文件。编译可以选择不同的效果，这是由编译时的参数所决定的。

### 编译参数

gcc 在编译时有三个常用的参数，-g -o -c

- -g 表示在编译时取消一切优化，并且为创建程序的符号表。两步操作都是为了为gdb调试作准备。如果不使用-g参数，那么gdb调试时便没有符号表；而且会导致程序自动优化，调整部分代码的执行顺序，为调试带来麻烦。
  ![1538721822107](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538721822107.png)

- -o 选项可以指定生成的文件的文件名，`gcc -o outputfile progname`，不适用该参数时默认为a.out

- -c 选项实现了只编译而不连接的功能，后续可以使用其他命令进行手动链接。

lab1 的编译可以直接在lab1文件夹下，在命令行中使用make进行编译。

### ucore的源码级调试

结合gdb和qemu对ucore进行源码级调试，需要二者相互配合。

首先gcc在编译时要注意使用-g 参数，为gdb调试创建符号表。qemu在启动时，不能够让cpu直接开始运行，必须要等待gdb的接入，如此便需要在运行qemu时加入-S -s 这两个参数。

qemu运行后，在另一个命令行启动gdb，并使用target remote命令远程连接qemu进行调试，链接成功后在gdb中输入c可以让qemu继续执行：（127.0.0.1是本机ip，可省略不写，1234是qemu的默认端口号）
![1538725792566](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538725792566.png)

此时gdb仅仅是链接了qemu而已，并没有qemu相关的调试信息，所以无法进行调试，所以需要在键入c之前使用file命令进行加载：
![1538727153579](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538727153579.png)

*注：target命令和file命令的顺序可以颠倒，另外在启动gdb时也可以直接加载./bin/kernel：*
*`gdb bin/kernel`，这样便可以不必再gdb中使用file命令了*

准备工作就绪后，便可以像平常调试程序那样对gdb进行操作了，比如设置断点、单步调试、查看变量、堆栈信息等等。按照lab1的要求，下面以调试bootmain函数为例展示gdb的使用。
![1538731717605](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538731717605.png)

- 为了方便查看变量和源码等，可以使用gdb的tui功能，通过Ctrl+X+A可以快速打开tui。
  ![1538731862591](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538731862591.png)
  通过layout 命令可以选择查看各类信息，如layout src查看源码，layout asm查看汇编代码，layout regs查看寄存器等等。
  ![1538734373394](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538734373394.png)

- break(/b)命令可以设置断点，后面可以指定一个整数表示在第几行设置断点，或者直接指定一个函数名表示在函数位置设置断点；使用clear也可以删除断点；continue(/c/run)表示继续执行程序；step/next表示单步调试，不同的在于step不会直接执行函数而是进入被调用的函数内部继续调试。

- breaktrace可以查看调用栈帧，用于获得函数调用顺序；display可以查看某一个由程序变量组成的表达式的值（每次程序停止后会显示）；help可用于查看调试命令的帮助信息。

- 在gdb中，可以使用info命令查看各项信息。

  | 命令       | 功能               |                             图示                             |
  | ---------- | ------------------ | :----------------------------------------------------------: |
  | info files | 加载的文件信息     | ![1538733568093](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733568093.png) |
  | info prog  | 程序运行状态       | ![1538733636904](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733636904.png) |
  | info func  | 被调用所有函数名称 | ![1538733771791](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733771791.png) |
  | info break | 断点列表及到达次数 | ![1538733757280](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733757280.png) |
  | info var   | 全局变量           | ![1538733812237](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733812237.png)![1538733822481](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733822481.png)![1538733840105](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733840105.png) |
  | info local | 局部变量           | ![1538733789750](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538733789750.png) |

  *其他1：在gdb中可以设置当前使用的架构，当调式的程序不是i386保护模式时，可以使用set arch修改，如*
  *`set arch i8086`*

  *其他2：gdb动态链接库的加载时，gdb无法得知动态链接库被分配的地址时多少，所以对动态链接的代码进行调试的时候需要手动分配一个地址，要求gdb将调试信息加载到该地址上：如*
  *`add-symbol-file android_test/system/bin/linker 0x6fee6180`*

## 其他常用工具

### diff & patch

##### diff 用于比较两个文件的差异

如果指定要比较目录，则diff会比较目录中相同文件名的文件，但不会比较其中子目录。diff的命令格式为：

`diff [-abBcdefHilnNpPqrstTuvwy][-<行数>][-C <行数>][-D <巨集名称>][-I <字符或字符串>][-S <文件>][-W <宽度>][-x <文件或目录>][-X <文件>][--help][--left-column][--suppress-common-line][文件或目录1][文件或目录2]`

![1538741692544](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538741692544.png)

- "|"表示前后2个文件内容有不同
- "<"表示后面文件比前面文件少了1行内容
- ">"表示后面文件比前面文件多了1行内容

##### patch命令的作用时修补文件

可用于修改、更新原始文件。命令格式为：

`patch [-bceEflnNRstTuvZ][-B <备份字首字符串>][-d <工作目录>][-D <标示符号>][-F <监别列数>][-g <控制数值>][-i <修补文件>][-o <输出文件>][-p <剥离层级>][-r <拒绝文件>][-V <备份方式>][-Y <备份字首字符串>][-z <备份字尾字符串>][--backup-if -mismatch][--binary][--help][--nobackup-if-mismatch][--verbose][原始文件 <修补文件>] 或 path [-p <剥离层级>] < [修补文件]`

修补文件一般为diff产生的比较结果

![1538742371220](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538742371220.png)

### understand

understand是一个静态代码分析工具，它可以实现对代码结构的图形化展示等实用功能。下图为understand官方文档给软件做出的定位：
![1538736757091](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538736757091.png)

- 创建新项目

  | 操作                   |                                                              |
  | ---------------------- | ------------------------------------------------------------ |
  | new project            | ![1538738375369](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538738375369.png) |
  | 选择新代码所用的语言集 | ![1538738409492](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538738409492.png) |
  | 导入工程文件           | ![1538738498082](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538738498082.png) |
  | 代码分析               | ![1538738596652](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538738596652.png) |

- 文件内搜索：Ctrl + F
  ![1538738752447](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538738752447.png)

- 全局搜索：F5 或者Search工具栏，可以搜索类/方法/变量等
  ![1538739022747](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538739022747.png)![1538739122593](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538739122593.png)

- 项目视图：understand提供了用于代码分析的项目视图，可以生成各种流程图、类和方法的调用图等等。查看方法是在文件或者方法/类上右键，选择graphical views，该目录下会有各类项目试图可供选择。

  | 类/方法                                                      | 文件                                                         |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | ![1538739390288](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538739390288.png) | ![1538739417536](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538739417536.png) |

- 结构关系视图分类：
  1.Graph Architecture：展示一个框架节点的结构关系；
  2.Declaration:展示一个实体的结构关系，例如：展示参数，则返回类型和被调用函数，对于类，则展示私有成员变量（谁继承这个类，谁基于这个类）
  3.Parent Declaration:展示这个实体在哪里被声明了的结构关系；
  4.Declaration File:展示所选的文件中所有被定义的实体（例如函数，类型，变量，常量等）；
  5.Declaration Type:展示组成类型；
  6.Class Declaration:展示定义类和父类的成员变量；
  7.Data Members:展示类或者方法的组成，或者包含的类型；
  8.Control Flow:展示一个实体的控制流程图或者类似实体类型；

- 层级关系视图分类：
  1.Butterfly：如果两个实体间存在关系，就显示这两个实体间的调用和被调用关系；如下图为Activity中的一个方法的关系图：
  2.Calls：展示从你选择的这个方法开始的整个调用链条；
  3.Called By：展示了这个实体被哪些代码调用，这个结构图是从底部向上看或者从右到左看；
  4.Calls Relationship/Calledby Relationship:展示了两个实体之间的调用和被调用关系，操作方法：首先右键你要选择的第一个实体，然后点击另一个你要选择的实体，如果选择错误，可以再次点击其他正确即可，然后点击ok；
  5.Contains:展示一个实体中的层级图，也可以是一个文件，一条连接线读作”x includes y“；
  6.Extended By:展示这个类被哪些类所继承，
  7.Extends:展示这个类继承自那个类：

### meld

meld实际上就是一个可视化的diff和合并工具，作用就是用于实现diff 和 patch的功能。

- 比较文件，打开软件后直接选择比较 文件/文件夹 并导入相应的文件即可进行比较。
  ![1538740352123](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538740352123.png)
- 右上角的Same New Modified是多选项，选择则代表显示相应类型的结果（相同的/新添的/被修改过的）
- 结果中的颜色和字体具有特定的含义：
  ![1538740526867](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538740526867.png)
- 合并功能：在比较结果图中的上边缘中间，由CopyLeft和CopyRight，即为合并的按钮，它可以将文件从一边合并到另一边。
- 在进行文件/文件夹比较的时候，我们有时会容许一些类型的不同，这时可以使用Meld的过滤器功能，过滤器在Modified按钮右侧。
  ![1538740873964](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538740873964.png)
  图中为meld自带的几个过滤条件，用户自己也可以创建自己的过滤条件，在edit/Preferences/File Filters中
  ![1538741184666](C:\Users\70700\AppData\Roaming\Typora\typora-user-images\1538741184666.png)

## make和makefile

### make

make命令在第一次执行时会扫描一遍makefile，寻找编译目标及其依赖，如果依赖本身也是编译目标，则继续递归寻找，如果不是目标则直接建立关系。关系建立完毕后，便编译他们。当某个目标被修改后，make再次被调用时，它将只编译与该文件相关的目标文件，其他保持原样，由此进一步节省编译的时间。

常用指令的参数及其意义如下：

| 参数                | 意义                                                         |
| ------------------- | ------------------------------------------------------------ |
| -B                  | 按照前面的阐述，make指令不会编译上次编译后未经修改的文件，-B参数的作用就是覆盖这种默认行为，让所有的目标重新建立。 |
| --debug[=<options>] | 在编译时输出make的调试信息，没有option参数时输出最简单的调试信息（=b）,使用make -d 会输出所有的调试信息（=a） |
| -C                  | 在其他目录下进行make，该参数会在make前切换到makefile所在的目录，make结束后再切换回来 |
| -f/--file           | 默认情况下，make会依据makefile/Makefile/GNUmakefile来执行，但是如果修改的文件名，便需要使用-f参数指定把该文件当作是makefile |

其他的一些常见于make相关的操作有：

| 命令                   | 功能                                           |
| ---------------------- | ---------------------------------------------- |
| make install/uninstall | 安装/卸载已编译好的工程                        |
| make all               | 编译所有目标                                   |
| make clean             | 删除由make产生的文件（一般为目标文件即.o文件） |
| make distclean         | 删除由./configure产生的文件                    |
| make check             | 测试刚编译的软件                               |
| make installcheck      | 检查安装的库和程序                             |
| make dist              | 重新打包成packname-version.tar.gz              |
| make -j                | 使用所有的核心编译目标                         |
| make -j8               | 使用8个核心编译目标                            |

### makefile

#### makefile的规则

makefile文件由一系列的rules组成，每一条rule的格式类似，如下：
`<target>: <prerequisites>
[Tab]<commands>`

- target是一个目标文件，可以是Object File，也可以是执行文件，甚至是一个标签（伪目标）
- perequisites为前置条件，是生成target文件所必须的文件或是目标
- command是make所需要执行的命令（任意的Shell命令，比如可以是gcc -c main.c 、 gcc -o main.o func.o等等）

*第二行必须由一个Tab键起首，后接命令；目标是必须的，不可省略；前置条件和命令是可选的，但两者必须至少存在一个。如果命令太长或者目标文件太多，可以使用\换行。添加行注释使用#（makefile只有行注释）。*

#### makefile中的变量

- 变量定义：`varname = a.o b.o #等等，类似于宏定义`
- 变量引用：`$(varname)`，引用可以发生在任何地方

#### makefile的引用

makefile中有include关键字，可以把其他的makefile包含进来，被包含的文件会原模原样的放在当前文件的包含位置。语法如下：
`include<filename> #filename 可以是当前操作系统Shell的文件模式（可以包含路径和通配符）`
*注：include前可以有通配符，但是不可以有Tab*

#### make的自动推导（隐式规则）

只要make看到一个[.o]文件，它就会自动的把[.c]文件加在依赖关系中，如果make找到一个whatever.o，那么whatever.c，就会是whatever.o的依赖文件。并且 gcc -c whatever.c 也会被推导出来，这种规则使得makefile的书写得以简化（省略了command）。

#### makefile的伪目标

伪目标并不会被生成，该文件也一般不存在于make的目录及子目录下，所以make无法生成它的依赖关系和决定它是否要执行。使用伪目标需要使用 `.PHONY: phony_target` 指明。例如：

` .PHONY: clean
   clean:
​           rm *.o temp`

## 参考文献

1. https://objectkuan.gitbooks.io/ucore-docs  （实验指导）
2. https://www.cnblogs.com/zhangyang/p/7602485.html   （Understand使用指南）
3. https://blog.csdn.net/u011776903/article/details/73563957/  （Understand使用指南）
4. https://jingyan.baidu.com/article/25648fc1623c899191fd0030.html  （Meld使用指南）
5. http://www.wangchao.net.cn/bbsdetail_1632751.html  （patch命令使用的注意事项）
6. http://man.linuxde.net/make （make指令使用）
7. https://blog.csdn.net/qq_35451572/article/details/81092902 （make和makefile）
8. https://blog.csdn.net/ruglcc/article/details/7814546/  (Makefile 经典教程)