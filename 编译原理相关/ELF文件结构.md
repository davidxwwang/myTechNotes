## 编译流程

#### 编译：

目的

生成产物：ELF文件

ELF文件类型：.o可重定向文件  .a文件 .so共享文件 可执行文件等

#### 链接

目的：生成可执行文件

生成产物：可执行文件

## ELF文件结构

（1）链接视图

（2）装载视图

也就是linux平台上，.o以及那些可执行文件的文件结构

##### （1）文件头header

​		描述ELF文件的基本信息，比如版本号，目标机器型号，程序入口地址。

```c
readelf -h 文件名：比如---->
readelf -h   /usr/local/python3/lib/python3.8/config-3.8-x86_64-linux-gnu/python.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file) // 文件类型：可执行文件/可重定向文件(.a文件)/共享目标文件(.so文件)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0                  // 入口地址，操作系统加载完程序后从此地址开始执行程序指令，可重定向文件没有这个值，一般为0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          5528 (bytes into file)  // 段表在文件中的偏移位置
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)         // 每个段表描述符的大小，见下面“段表”
  Number of section headers:         25                 // 段表成员数目，见下面“段表”
  Section header string table index: 24

```



##### （2）段表（Section Header Table，其实是个数组）

描述各个段（.text .data .bss段）的信息，比如每个段的名字，段的长度，段在文件中的偏移，读写权限以及其他权限。

编译器，链接器，装载器都是靠它来定位和访问各个段的属性。

```c
 1，ojdump -h   // 查看主要段的信息
 2，readelf -S  // 以下面为例，有25个段
     
    objdump -h /usr/bin/pwd

/usr/bin/pwd:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .interp       0000001c  0000000000400238  0000000000400238  00000238  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.ABI-tag 00000020  0000000000400254  0000000000400254  00000254  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .note.gnu.build-id 00000024  0000000000400274  0000000000400274  00000274  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .gnu.hash     0000001c  0000000000400298  0000000000400298  00000298  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  ...
   10 .init         0000001a  0000000000401350  0000000000401350  00001350  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 11 .plt          000003f0  0000000000401370  0000000000401370  00001370  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 12 .text         0000333a  0000000000401760  0000000000401760  00001760  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
其中  VMA, or virtual memory address. This is the address the section will have when the output file is run.      LMA, or load memory address. This is the address at which the section will be loaded. In most cases the two addresses will be the same.

 
 readelf -S   /usr/local/python3/lib/python3.8/config-3.8-x86_64-linux-gnu/python.o
There are 25 section headers, starting at offset 0x1598:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       0000000000000000  0000000000000000  AX       0     0     1
  [ 2] .data             PROGBITS         0000000000000000  00000040
       0000000000000000  0000000000000000  WA       0     0     1
  [ 3] .bss              NOBITS           0000000000000000  00000040
       0000000000000000  0000000000000000  WA       0     0     1
  [ 4] .text.startup     PROGBITS         0000000000000000  00000040
       0000000000000005  0000000000000000  AX       0     0     16
  [ 5] .rela.text.startu RELA             0000000000000000  00000c30
       0000000000000018  0000000000000018   I      22     4     8
  [ 6] .debug_info       PROGBITS         0000000000000000  00000045
       0000000000000391  0000000000000000           0     0     1
  [ 7] .rela.debug_info  RELA             0000000000000000  00000c48
       0000000000000720  0000000000000018   I      22     6     8
  [ 8] .debug_abbrev     PROGBITS         0000000000000000  000003d6
       0000000000000117  0000000000000000           0     0     1
  [ 9] .debug_loc        PROGBITS         0000000000000000  000004ed
       0000000000000072  0000000000000000           0     0     1
  [10] .rela.debug_loc   RELA             0000000000000000  00001368
       00000000000000c0  0000000000000018   I      22     9     8
  [11] .debug_aranges    PROGBITS         0000000000000000  0000055f
       0000000000000030  0000000000000000           0     0     1
  [12] .rela.debug_arang RELA             0000000000000000  00001428
       0000000000000030  0000000000000018   I      22    11     8
  [13] .debug_ranges     PROGBITS         0000000000000000  0000058f
       0000000000000020  0000000000000000           0     0     1
  [14] .rela.debug_range RELA             0000000000000000  00001458
       0000000000000030  0000000000000018   I      22    13     8
  [15] .debug_line       PROGBITS         0000000000000000  000005af
       00000000000000e9  0000000000000000           0     0     1
  [16] .rela.debug_line  RELA             0000000000000000  00001488
       0000000000000018  0000000000000018   I      22    15     8
  [17] .debug_str        PROGBITS         0000000000000000  00000698
       000000000000036a  0000000000000001  MS       0     0     1
  [18] .comment          PROGBITS         0000000000000000  00000a02
       000000000000002e  0000000000000001  MS       0     0     1
  [19] .note.GNU-stack   PROGBITS         0000000000000000  00000a30
       0000000000000000  0000000000000000           0     0     1
  [20] .eh_frame         PROGBITS         0000000000000000  00000a30
       0000000000000030  0000000000000000   A       0     0     8
  [21] .rela.eh_frame    RELA             0000000000000000  000014a0
       0000000000000018  0000000000000018   I      22    20     8
  [22] .symtab           SYMTAB           0000000000000000  00000a60
       00000000000001b0  0000000000000018          23    16     8
  [23] .strtab           STRTAB           0000000000000000  00000c10
       000000000000001c  0000000000000000           0     0     1
  [24] .shstrtab         STRTAB           0000000000000000  000014b8
       00000000000000d9  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)

 以text段为例，分析下：     
     
 Name   Type   		Address  		  Offset    Size  			  EntSize  			Flags  Link  Info  Align
 text   PROGBITS    0000000000000000  00000040  0000000000000000  0000000000000000  AX       0     0     1
name是段名；type为段的类型（比如程序段，符号表，重定向表等）；Address是该段被加载后在地址空间的虚拟地址，否则为0（可执行文件不能为0） offset 段偏移 flags表示可读/可写/可执行 align段地址对齐（2的指数幂）
```

###### 几个重要的段表

1. 重定向表

2. 字符串表

   用于定位字符串的，表名一般叫"strtab",

   头文件中有个Section header string table index: 29，表示符号表位于段表数据索引为29的元素。

   

   

   3.符号表

   函数和变量都叫符号（symbol）

##### （3）各个段具体数据

```
可以使用 objdump -s -d 打印各段的内容
```



1）.text段：存放编译后的执行语句

2）.data段：存放初始化后的全局变量和局部静态变量

3）.bss段：存放未初始化的全局变量和局部静态变量

4）其他段：

比如"重定向表"：

​      因为一个模块一般情况下需要引用其他模块的方法，比如printf方法，所以此模块单独编译阶段，是无法找到printf，需要有个地方记录printf信息，在链接的时候将printf的实际地址替换掉，这个地方就叫“重定向表”。

​       “字符串表”：专门记录一些字符串

局部变量放在每个函数的栈里面即可。

分段的好处有许多：其中之一就是可以利用copy on write。

  







重定向表：

比如程序引用了printf方法，printf方法需要重定向。

## 链接：

链接的本质是把不同的目标文件粘在一起，将方法的定义与方法引用联系起来（让方法的引用可以找到方法的定义），

符号：（方法与变量），通过符号完成链接工作的，

#### 符号表

记录了每个目标文件用到的符号，例如：段名，全局符号，行号信息等），段名叫.symtab的段保存了符号表信息。

```c
 nm  /usr/local/python3/lib/python3.8/config-3.8-x86_64-linux-gnu/python.o  // 查看符号表
     
 readelf -s  /usr/local/python3/lib/python3.8/config-3.8-x86_64-linux-gnu/python.o

Symbol table '.symtab' contains 18 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS python.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    2
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    3
     ...
    15: 0000000000000000     0 SECTION LOCAL  DEFAULT   18
    16: 0000000000000000     5 FUNC    GLOBAL DEFAULT    4 main
    17: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND Py_BytesMain
Value表示符号值，Size符号大小，Type符号类型，Bind绑定信息，Ndx符号所在的段的下标（通过 readelf -S 可以查看对应的段信息），Name为符号名；
如果符号类型是方法或者变量，Value就是方法或者变量的地址；在可执行文件在，其表示虚拟地址。
类型为NOTYPE，表示是未知类型；类型是FILE，表示是文件；类型是SECTION，表示是段（.text段的下标为1，通过 readelf -S 可以查看）；类型是FUNC，表示是个方法；类型是OBJECT，表示是个对象；
main的Ndx = 4，通过readelf -S 可以得到对应下标是4的段叫  .text.startup 。
        
```

#### 强符号与弱符号 

同名的符号很有可能因为是弱符号的原因，链接成功，但是运行的时候会出错，这点要特别注意。所以全局符号最好少定义。

弱引用也可用在库的功能裁剪，这点也要留意。

#### 链接器

（查找所有输入文件的符号表组成的全局符号表，找到对应的符号以后后再重定向）





## 文件类型

### 共享对象

### 共享库(shared Library)

### 可执行文件

程序头表





## 动态链接库

共享对象的地址在编译阶段是不固定的，只有在装载阶段动态确定。



## 共享库的兼容和版本问题