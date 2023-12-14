---
layout: post
title:  "C++ overview"
date:   2023-08-01 23:59:59 +0800
categories: c++
---


## 文件格式

## 头文件和命名空间(名字空间)

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <cmath>

using namespace std;
```

> 头文件可以定义类、值在编译时就已知道的 const 对象和 inline 函数

> 不应该含有变量或函数的定义。

命名空间引用语法：

> using 命名空间名::标识符名;

> using namespace 命名空间名;

通常，头文件有一个 **include 防范**或 `#pragma once` 指令，用于确保它们不会多次插入到单个 .cpp 文件中。

## C++类型系统

**标量类型**：保存已定义范围的单个值的类型。 标量包括算术类型 (整型或浮点值) 、枚举类型成员、指针类型、指向成员的指针类型和 `std::nullptr_t`。 基本类型通常是标量类型。

**复合类型**：不是标量类型的类型。 复合类型包括数组类型、函数类型、类 (或结构) 类型、联合类型、枚举、引用和指向非静态类成员的指针。

**变量**：数据数量的符号名称。 该名称可用于访问它所引用的整个代码范围内的数据。 在 C++ 中， *变量* 通常用于引用标量数据类型的实例，而其他类型的实例通常称为 *对象*。

**对象**：为简单起见和一致性，本文使用术语 *对象* 来引用类或结构的任何实例。 在一般意义上使用它时，它包括所有类型的变量，甚至标量变量。

## 指定变量和函数类型

> POD 类型（纯旧数据）：C++ 中的此类非正式数据类型类别是指作为标量（参见基础类型部分）的类型或 POD 类。 POD 类没有非 POD 的静态数据成员，也没有用户定义的构造函数、用户定义的析构函数或用户定义的赋值运算符。 此外，POD 类无虚函数、基类、私有的或受保护的非静态数据成员。 POD 类型通常用于外部数据交换，例如与用 C 语言编写的模块（仅具有 POD 类型）进行的数据交换。

C++ 既是 *强类型* 语言，也是 *静态类型语言* ;每个对象都有一个类型，并且该类型永远不会更改。 在代码中声明变量时，你必须显式指定其类型或使用 `auto` 关键字指示编译器通过初始值设定项推断类型。 在代码中声明函数时，必须指定其返回值的类型和每个参数的类型。 如果函数未返回任何值，请使用返回值类型 `void` 。 使用函数模板时例外，函数模板允许任意类型的参数。

首次声明变量后，不能在以后的某个时间更改其类型。 但是，可以将变量的值或函数的返回值复制到另一个不同类型的变量中。 此类操作称作“类型转换”，这些操作有时很必要，但也是造成数据丢失或不正确的潜在原因。

声明 POD 类型的变量时，强烈建议 *对其进行初始化* ，这意味着为其指定初始值。 在初始化某个变量之前，该变量会有一个“垃圾”值，该值包含之前正好位于该内存位置的位数。 这是需要记住的 C++ 的一个重要方面，特别是如果你来自另一种处理初始化的语言。 声明非 POD 类类型的变量时，构造函数将处理初始化。

## 语法

### 关键字

#### c++关键字

asm do if return try auto double inline short typedef bool dynamic_cast int signed typeid break else long sizeof typename case enum mutable static union catch explicit namespace static_cast unsigned char export new struct using class extern operator switch virtual const false private template void const_cast float protected this volatile continue for public throw wchar_t default friend register true while delete goto reinterpret_cast C++ 还保留了一些词用作各种操作符的替代名。这些替代名用于支持某些不 支持标准C++操作符号集的字符集。它们也不能用作标识符。表 2.3 列出了这些 替代名。

## 变量和基本类型

# 基本（内置）类型

| **类型**          | **大小** | **评论**                                                                                                 |
| --------------- | ------ | ------------------------------------------------------------------------------------------------------ |
| `int`           | 4 个字节  | 整数值的默认选择。                                                                                              |
| `double`        | 8 字节   | 浮点值的默认选择。                                                                                              |
| `bool`          | 1 个字节  | 表示可为 true 或 false 的值。                                                                                  |
| `char`          | 1 个字节  | 用于早期 C 样式字符串或 std:: 字符串对象中无需转换为 UNICODE 的 ASCII 字符。                                                    |
| `wchar_t`       | 2 个字节  | 表示可能以 UNICODE 格式进行编码的“宽”字符值（Windows 上为 UTF-16，其他操作系统上可能不同）。 `wchar_t` 是在 类型字符串中使用的字符类型 `std::wstring`。 |
| `unsigned char` | 1 个字节  | C++ 没有内置字节类型。 使用 `unsigned char` 来表示字节值。                                                               |
| `unsigned int`  | 4 个字节  | 位标志的默认选项。                                                                                              |
| `long long`     | 8 字节   | 表示更大的整数值范围。                                                                                            |

| **类型**      | **含义**                            | **最小存储空间**                 |
| ----------- | --------------------------------- | -------------------------- |
| bool        | boolean                           | NA                         |
| char        | character                         | 8bits(byte)                |
| wchar_t     | wide character                    | 16bits                     |
| short       | short integer                     | 16bits                     |
| int         | integer                           | 16bits                     |
| long        | long integer                      | 32bits                     |
| float       | single-precision floating-point   | 6 significant digits[有效数字] |
| double      | double-precision floating-point   | 10 significant digits      |
| long double | extended-precision floating-point | 10 significant digits      |

int,short,long默认带符号的，无符号必须指定类型为unsigned.

char三种类型plain char,unsigned char,signed char

> 符号位为1，值为负数；符号位为0，值为正数

> 负数可以赋值unsigned类型，但编译器会对该值取模计算

## 字面值常量

#### 整型字面值

> 十进制（decimal）

> 八进制（octal）0__

> 十六进制（hexadecimal）0x|X__

字面值整数常量的默认类型是int或long类型。增加后缀强制将字面值整数常量转换为 long、unsigned 或 unsigned long类型。

#### 浮点字面值

#### 布尔字面值（true & false）

#### 字符字面值

```cpp
'A' '' 's'
L'A' // 转换成宽字符字面值
```

#### 非打印字符的转义序列

c++中定义的转义字符

```cpp
换行符 \n 水平制表符\t
纵向制表符 \v 退格符 \b
回车符 \r 进纸符 \f
报警（响铃）符 \a 反斜线 \\
疑问号 \? 单引号 \'
双引号 \"
```

#### 字符串字面值&字符串字面值的连接

> 为了兼容C语言，，C++ 中所有的字符串字面值都由编译器自动在末尾添加一个空字符。

#### 多行字面值

```cpp
在行尾 \ 可以分隔变量
```

### 变量

左值右值、枚举

# c++变量

左值：变量的地址，或者是一个代表“对象在内存中的位置”的表达式

右值：变量的值

变量初始化

- 复制初始化
- 直接初始化

内置类型变量初始化

> 建议每个内置对象都要初始化

## 变量定义

```cpp
extern ...
声明

extern double pi = 3.1416; // definition,分配存储空间
```

声明用于表明变量的类型和名字，extern声明不是定义也不分配存储空间。定义也是声明，但是定义只允许一次。

## typedef

```cpp
typedef double hourly,weekly;
```

定义类型的同义词

## 枚举

```cpp
enum open_modes {input, output, append};
// 默认赋值0 1 2
```

### 常量

> 常量需要初始化

```cpp
const int a = 1;
```

### 操作符

bool值 false用0表示，true用1

```json
x < y < z // 不合法
// x < y == 1｜0
```

sizeof操作符

#### 强制类型转换

static_cast

dynamic_cast 支持运行时识别指针或者引用所指向的对象

const_cast

reinterpret_cast

## 语句控制

IF ElSE 悬垂else

### switch

⚠️只能在switch最后一个case标号或default后定义变量，或为某个特殊的case定义变量引入块语句。

## 数组

```cpp
const unsigned array_size = 3;
int arr[array_size]
int ai[] = {} //显示初始化数组不需要指定数组的维数值

char ca3[] = "C++"; // 4 null terminator added automatically
```

## 类

[c++面向对象](craftdocs://open?blockId=6D61F73B-210D-4CAB-AE12-A7FA845AB31D&spaceId=ac457752-3d55-6de6-afc5-acfea35b1e0e)

----

## 强制类型转换运算符

> static_cast<类型名>(表达式)

> const_cast<类型名>(表达式)

函数参数的默认值

C++规定，提供默认值时必须按照从右至左的顺序提供，即有默认值的形参必须在形参列表的最后。


# c++指针

```cpp
strig *ptr;

string* ptr;

string* ps1; ps2; // ps1 is a pointer, ps2 is a string.
```

指针只能初始化或赋值为同类型的变量地址或另一指针

void*指针，可以保存任何类型对象的地址

![Image.tiff](https://res.craft.do/user/full/ac457752-3d55-6de6-afc5-acfea35b1e0e/doc/606CE408-D486-4A10-A529-2A5533F10F43/F382B989-6158-4460-9479-EF29475A533A_2/PC69N6eSBPNbEkmM3yEDQRvxxaOU1cpUP9mS0Fhkolgz/Image.tiff)

指向指针的指针

```json
int ival = 1024;
int *pi = &ival; // pi points to an int
int **ppi = &pi; // ppi points to a pointer to int
```

![Image (2).tiff](https://res.craft.do/user/full/ac457752-3d55-6de6-afc5-acfea35b1e0e/doc/606CE408-D486-4A10-A529-2A5533F10F43/9141D47C-CFC9-45B9-BED4-7979B013C261_2/GyDH6dMnc1bRZKv2A2hs3KbCNyxa4hC0z1gYUxbNLGQz/Image%202.tiff)

```json
int ia[] = {0,2,4,6,8};
int *ip = ia; // ip points to ia[0]
ip = &ia[4]; // ip points to last element in ia
```

> 通常，在指针上加上（或减去）一个整型数值 n 等效于获得一个新指针，该新指针指向指针原来指向的元素之后（或之前）的第 n 个元素

const限定符修饰的变量不能改变，不可以左值使用

const唯一且在*号左侧，表示指针所指数据是常量，数据不能通过本指针修改，但指针可以指向其他的内存单元；

const唯一且在*号右侧，表示指针本身是常量，不可以修改指针指向其他内存地址，指针所指的数据可以通过本指针修改；

const在*号两侧，禁止修改指针及通过指针修改所指向的数据

| **指向const对象的指针**  |           |   |
| ----------------- | --------- | - |
| const指针           | 必须在定义时初始化 |   |
| 指向const对象的const指针 |           |   |

```json
const double *const cptr = &ccc;
// *右边变量名前，代表常量变量不能更改指针引用
// *左边代表常量类型
```

## 动态数组

```json
string *ptr = new string[10]; // ptr指向数组的第一个元素
// 将会调用类的默认构造函数对元素进行初始化
// 内置类型不会初始化
int *ptr = new int[10](); // 初始化int类型,值为0

const int *pci_ok = new const int[100](); // const数组每个元素都要初始化
```

> 使用new运算符动态申请的内存空间需要在使用完毕时释放。

> 动态创建的数组必须显示的删除

```json
delete [] p;
// delete不可以删除指针指向空间并不是new运算符分配的，动态分配数组需要写[],否则可能无法完全释放
```

删除指针

delete point >> 悬垂指针 >> 如果需要删除指针所指向对象 需要立即将指针置为0
