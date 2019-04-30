# 浅谈C++编译

## 问题起源

出于学习的需要，前段时间开始接触OpenCV这个开源计算机视觉库。项目使用C++进行开发，所以需要安装它的C++版本。

俗话说的好，万事开头难。我以往的经历告诉我，在新接触一个计算机编程工具的时候，往往需要花费相当的时间去解决配置上的问题已达到可以正常使用的目的。直接使用官方提供的OpenCV解压缩包是无法直接运行项目代码的，因此需要自己编译源代码以便生成项目所依赖的库文件（CUDA版本）。

在编译的过程中经历了无数的坑不说，有一点引起了我的注意：无论在Visual Studio上还是在VIsual Studio Code上进行配置，有一步非常重要，就是需要在配置文件中指明所使用的OpenCV 头文件，库文件，以及dll文件的路径或者名称。当初配置这些的时候并没有觉得有啥特别的感觉，只是非常疑惑，为什么非要做这些步骤呢？（这一切放到Python中也就是一个pip命令的事情把）。

其实，根本原因在于我没把C/C++的编译过程弄清楚。

学过C的人多多少少都了解编译的大致流程吧，即：预处理-->编译-->汇编-->链接-->可执行程序.exe

我相信包括我在内的好多人基本上就只停留在知道这个流程这一层面上吧。一个人写C/C++程序的话，虽然说涉及到多文件编译，但是像VS这样的IDE可以为我们省去好多麻烦事，基本上一个F5按下去你就可以看到正在运行的“黑框框”了。我们把注意力主要放在了编码上，而不是这些看起来“理所应当”的事情上。

然而，稍微深入地了解一下这件看起来“理所应当”的事情是非常有好处的。至少，你终于可以明白在配置OpenCV这件事情上为什么“很繁琐”的原因了。

那么，我们开始愉快地聊一聊这几个过程吧 !

## 1. 预处理

### 1.1头文件包含

下面这句话就像吃饭一样熟悉！

```c++
#include<iostream>
```

在《C++ Primer Plus》（第六版）中，是这样描述的：

> 1. C++和C一样，也使用一个预处理器，该程序在进行主编译之前对源文件进行处理。
> 2. 程序清单2.1使用#include编译指令，该编译指令导致预处理器将iostream文件的内容添加到程序中。这是一种典型的预处理操作：在源代码被编译之前，替换或添加文本。
> 3. 这里提出了一个问题：为什么要将iosteam文件的内容添加到程序中呢？答案设计程序与外部世界之间的通信。C++的输入和输出方案涉iostream文件中的多个定义。为了使用cout来显示消息，程序需要这些定义。#include编译指令将导致iostream文件的内容随源代码文件的内容一起被发送给编辑器。
> 4. 实际上，iostream文件的内容将取代程序中的代码行#include"iostream"。原始文件没有被修改，而是将源代码文件和iostream组合成一个复合文件，编译的下一阶段将使用该文件。

### 1.2宏替换

预处理器在处理宏定义时，会对宏进行展开（即宏替换）。
宏替换首先将源文件中在宏定义随后所有出现的宏名均用其所代表的代码序列替换之，如果是带参数宏则接着将代码序列中的宏形参名替换为宏实参名。
宏替换只作代码字符序列的替换工作，不作任何语法的检查，也不作任何的中间计算。

宏定义指令，如：

```c++
#define Pi 3.1415
```

预处理阶段会将程序中所有的Pi用3.1415代替。

### 1.3条件编译

一般情况下，在进行编译时对源程序中的每一行都要编译，但是有时希望程序中某一部分内容只在满足一定条件时才进行编译，如果不满足这个条件，就不编译这部分内容，这就是条件编译。
条件编译指令，如

```c++
#ifdef  #ifndef  #else  #elif  #endif
```

条件编译主要是进行编译时进行有选择的挑选，注释掉一些指定的代码，以达到多个版本控制、防止对文件重复包含的功能。
此外，还有 #pragma 指令，它的作用是设定编译器的状态或指示编译器完成一些特定的动作。

### 1.4总结

**预处理只做两件事:**
**第一：以当前cpp文件为基础，整合该文件所需要的全部文本资源（包括引入/替换等操作）**
**第二：以当前cpp文件为基础，控制编译器在编译此文件过程中的逻辑（如果需要）**

### 1.5 现在，我们引入一个实例来具体了解一下上述过程

首先介绍一下运行环境，Windows下查看二进制文件实在是蛋疼，我的电脑正好是双系统的，那我们就切换到Linux下操作吧。
我的系统版本如下：

![系统版本.png](0)

接下来，我们定位到工作目录下查看当前都有哪些文件，如图所示：

![目录.png](0)

可以看到，当前目录下：
text.md是使用Markdown编辑的本文。
两个文件夹assets 和photo，它们用来处理text.md中的图片，我们不去理会以上这些。
这里一共有三个C++文件：main.cpp,String.cpp以及String.h。
稍后，我们将将会手动编译它们。

**本文使用的程序来自《C++ Primer Plus》(第六版)  第12章 例程12.5以及12.6，（我最近刚看过的内容）并且这里稍微做了一点修改。**

我并不想讨论这些程序中涉及到的细节，因为就目前来说，C++对我而言算是“新鲜事物”。随着学习的深入，以后我会讨论到它们的。

main.cpp:

```c++
#pragma warning(disable : 4996)
#include <iostream>
#include "String.h"

using namespace std;


int aa=1;
int bb;
static int cc;
extern int dd;

const int ArSize = 10;
const int Maxlen = 81;


int main()
{
    cout<<dd<<endl;
    String name;
	  cout << "Hi, what's your name? " << endl;
	  cin >> name;
	  cout << name << " Please enter up to " << ArSize << " short sayings <enpty line to quit>: " << endl;
	String sayings[ArSize];
	char temp[Maxlen];

	for (int i = 0; i < ArSize; i++)
	{
		cout << i + 1 << ": ";
		cin.get(temp, Maxlen);
		while (cin && cin.get() != '\n')
		{
			continue;
		}
		if (!cin || temp[0] == '\n')
		{
			break;
		}
		else
		{
			sayings[i] = temp;
		}
	}

	int total = ArSize - 1;

	if (total > 0)
	{
		cout << "Here is your sayings: " << endl;
		for (int i = 0; i < total; i++)
		{
			cout << sayings[i][0] << ":" << sayings[i] << endl;
		}
		int shortest = 0;
		int first = 0;
		for (int i = 1; i < total; i++)
		{
			if (sayings[i].length() < sayings[shortest].length())
			{
				shortest = i;
			}
			if (sayings[i] < sayings[first])
			{
				first = i;
			}
		}
		cout << "Shortest saying : \n"
				 << sayings[shortest] << endl;
		cout << "First alphabeticallyo: \n"
				 << sayings[first] << endl;
		cout << "Thwis is program used " << String::HowMany() << " String objects. Bye." << endl;
	}
	else
	{
		cout << "Not input ! Bye." << endl;
	}

	cin.get();
	cin.get();
	return 0;
}
```

String.h

```C++
#pragma once
#include<iostream>
using std::ostream;
using std::istream;


class String
{
private:
	char* str;
	int len;
	static int num_strings;
	static const int CINLIM = 80;
public:
	String();
	String(const char*);
	String(const String&);
	~String();

	int length()const { return len; }
	
	String& operator=(const String&);
	String& operator=(const char*);
	char& operator[](int i);
	const char& operator[](int i)const;

	friend bool operator<(const String& st1, const String& st2);
	friend bool operator>(const String& st1, const String& st2);
	friend bool operator==(const String& st1, const String & st2);
	friend ostream& operator<<(std::ostream& os, const String& st);
	friend istream& operator>>(std::istream& is,  String& st);

	static int HowMany();
};


```

String.cpp

```C++
#define _CRT_SECURE_NO_WARNINGS
#include "String.h"
#include <ostream>
#include <cstring>
using std::cin;
using std::cout;

int dd=10;

int String::num_strings = 0;

int String::HowMany()
{
	return num_strings;
}

String::String()
{
	len = 4;
	str = new char[1];
	str[0] = '\0';
	num_strings++;
}

String::String(const char *s)
{
	len = std::strlen(s);
	str = new char[len + 1];
	std::strcpy(str, s);
	num_strings++;
}

String::String(const String &st)
{
	num_strings++;
	len = st.len;
	str = new char[len + 1];
	std::strcpy(str, st.str);
}

String::~String()
{
	--num_strings;
	delete[] str;
}

String &String::operator=(const String &st)
{
	if (this == &st)
	{
		return *this;
	}
	else
	{
		delete[] str;
		len = st.len;
		str = new char[len + 1];
		std::strcpy(str, st.str);
		return *this;
	}
}

String &String::operator=(const char *st)
{
	delete[] str;
	len = std::strlen(st);
	str = new char[len + 1];
	std::strcpy(str, st);
	return *this;
}

char &String::operator[](int i)
{
	return str[i];
}

const char &String::operator[](int i) const
{
	return str[i];
}

bool operator<(const String &st1, const String &st2)
{
	return (std::strcmp(st1.str, st2.str) < 0);
}

bool operator>(const String &st1, const String &st2)
{
	return (st2 < st1);
}

bool operator==(const String &st1, const String &st2)
{
	return (std::strcmp(st1.str, st2.str) == 0);
}

ostream &operator<<(std::ostream &os, const String &st)
{
	os << st.str;
	return os;
}

istream &operator>>(std::istream &is, String &st)
{
	char temp[String::CINLIM];
	is.get(temp, String::CINLIM);
	if (is)
	{
		st = temp;
	}
	while (is && is.get() != '\n')
	{
		continue;
	}
	return is;
}
```

对于计算机而言，人类使用高级语言编写好的cpp/h文件和一堆乱码并没有本质上的区别，我们需要一个翻译装置将这些人类语言变换到10的状态（计算机最喜欢这些反人类的玩意儿）。这种翻译装置就叫做**编译器**。

下面，我么开始预编译。在工作目录下输入如下命令：

```c++
g++ -E main.cpp -o main.ii
```

好快，一瞬间就结束运行了

![预处理.png](1)

这里解释一下上述命令的含义：
选项 -E 使 g++ 将源代码用编译预处理器处理后不再执行其他动作。-o表示文件输出，结果保存在main .ii 文件中.
可以看到，当前目录下多出了一个叫做main,ii的预处理文件

![预处理目录.png](2)

对于原始的main.cpp，这里只有83行代码

![mian行数.png](3)

但是，对于main.ii，情况就有些“大变样”了，我们使用

```c++
wc -l //参数I返回文件的行数
```

查看main.ii文件，结果如下：

![行数.png](0)

恩，由不到100行扩展到了28257行！
发生了什么？我们打开文件看一下：
这是文件头部：
![头部](/assets/头部.png)
如同前文所说的一样，预编译器将iostream整个文件，包括其本身引用外部文件全部添加到了main.ii里面。在第28139行，我们看到了之前写好的String.h文件
![中部](/assets/中部.png)

再接着往下看，在第28177行，我们看到了Main.cpp：
![尾部](/assets/尾部.png)

这就是预编译期间编译器所做的事情，你还记得1.4小结么？

  **预处理只做两件事:**
  **第一：以当前cpp文件为基础，整合该文件所需要的全部文本资源（包括引入/替换等操作）**

  ```C++
  #include <iostream>
  #include "String.h"
  ```

上述指令要求编译器在预处理阶段将相关文件整合进一个资源文件中（本文并未体现出宏替换这一操作，不过相信你可以感受到这个过程）
**第二：以当前cpp文件为基础，控制编译器在编译此文件过程中的逻辑（如果需要）**

```C++
#pragma warning(disable : 4996)
```

该指令要求编译器忽略编号为4996的编译警告/错误

然而，这些仅仅只是预处理，真正的“翻译”过程即将在下面展开。

## 2. 编译

这里仅仅简要介绍一下编译的大致流程，有一个感性认识即可。

### 2.1 词法分析

词法分析器的功能输入源程序，按照构词规则分解成一系列单词符号。单词是语言中具有独立意义的最小单位，包括关键字、标识符、运算符、界符和常量等
(1) 关键字 是由程序语言定义的具有固定意义的标识符。例如，Pascal 中的begin，end，if，while都是保留字。这些字通常不用作一般标识符。
(2) 标识符 用来表示各种名字，如变量名，数组名，过程名等等。
(3) 常数  常数的类型一般有整型、实型、布尔型、文字型等。
(4) 运算符 如+、-、*、/等等。
(5) 界符  如逗号、分号、括号、等等。

### 2.2 语法分析

语法分析是根据某种给定的形式文法对由单词序列（如英语单词序列）构成的输入文本进行分析并确定其语法结构的一种过程。语法分析器通常是作为编译器或解释器的组件出现的，它的作用是进行语法检查、并构建由输入的单词组成的数据结构（一般是语法分析树、抽象语法树等层次化的数据结构）。

### 2.3 语义分析

语义分析，使用语法树和符号表中的信息来检测源程序是否和语言定义的语义一致。

### 2.4 小结

在编译环节，编译器检查代码的语法有效性，并且生成一个明确的以机器语言（汇编语言）表表示的代码中间文件，这一过程还会涉及到代码优化相关的内容，即根据机器平台做一些代码优化。
**简单来说：在编译阶段，人类可读的高级语言会被编译器在一系列的语法有效性检查后转换为汇编语言。**
**然而：在计算机看来，这一步仍然是反计算机的,因为人类是可以直接阅读汇编语言的。**
**（想想看，在忽略代码含义的前提下，你可以不假思索地直接读出那些奇奇怪怪的字母，我说的对吧！）**

### 2.5 现在，我们引入实例来感性地了解一下编辑器在这一阶段的一系列骚操作之后的结果

在当前工作目录下，输入如下命令：

```C++
g++ -S main.cpp -o main.s
```

解释一下上述命令：选项 -S 指示编译器将程序编译成汇编代码，输出汇编语言代码后结束。上述命令将由 C++ 源码文件生成汇编语言文件 main.s，生成的汇编语言依赖于编译器的目标平台。就像这样：
![编译](/assets/编译.png)
我们打开main.s看一下：
![main汇编](/assets/main汇编.png)
恩，我承认，我只学过汇编的皮毛。我表示这种反学术水平的语言我是看不懂的。
幸运的是，本文的重点不在于此，我们只需要稍微了解这一过程即可。
看到这里，也许你会这么想：”你瞧，我们已经从高级语言向汇编语言进化啦，是不是我们的程序马上就可以被计算机执行了呢？“
Emmmmmm，也许事情没那么简单，但至少我们向前迈进了一大步！

## 3. 汇编

### 3.1 汇编是干嘛的？

**在汇编阶段，编辑器把作为中间结果的汇编代码翻译成了机器代码，即目标代码。**

上命令：

```C++
g++ -c main.s -o main.o
```

选项 -c 用来告诉编译器将汇编代码（.s文件，或者直接对源代码）转换为目标文件（.o文件）。使用-o参数输出转换好的目标文件。就像这样：
![汇编s文件](/assets/汇编s文件.png)
我们打开main.o看一看？？？不好意思，普通的文本查看工具已经不支持此类文件的查看了。是时候简单了解一下目标文件了

### 3.2 啥是目标文件？

目标文件是从源代码文件产生程序文件这一过程的中间产物。它是编译器编译源代码后生成的文件。（Windows下面为.obj文件；Linux下面为.o文件）从结构上将，它是已经编译后的可执行文件格式，**只是还没有经过链接的过程**，其中可能有些符号或有些地址还没有被调整，目标文件本身就是按照可执行文件格式存储的。
**目标文件包含着机器代码 ，可直接被计算机中央处理器执行** 。此外，还包含代码在运行时使用的数据，如重定位信息，如用于链接或调试的程序符号（变量和函数的名字），此外还包括其他调试信息。
从以上讯息中我们了解到了什么？
第一：目标文件的内容已经由“高级的”汇编语言转变为可以被计算机识别的“低级的”机器代码了。[机器代码](https://zh.wikipedia.org/wiki/%E6%9C%BA%E5%99%A8%E8%AF%AD%E8%A8%80)
第二：目标文件在结构上已经与可执行程序无异，但是它还不能被直接运行。（我想，目标文件这个名字中“目标”二字的含义估计就是来源于此吧，因为它真的已经无限接近最终结果了）
第三：目标文件之所以还不能够直接运行起来，**主要由于它还没经过链接的过程**。
**总结：在汇编这一过程中，编辑器将汇编语言汇编为可以被计算机直接读取的二进制的目标文件。**

看来离程序的正式运行不远了！

### 3.3 目标文件长啥样啊？我能瞅瞅不？

借助反汇编工具，这里观察一下我们感兴趣的部分。

nm命令可以查看目标文件中的符号信息

```C++
nm main.o
```

![nm命令](/assets/nm命令_hexjrwb5d.png)
这里有很多看起来非常杂乱的符号。在解释图片中的部分内容之前，我们先说一点别的东西。

nm命令的输出默认有三列，它们分别表示：
左列：变量所在段的相对地址。
中列：变量所在的段的类型
右列：变量的名称

**什么是变量所在段的相对地址？**
汇编语言的知识告诉我们，相对地址其实就是偏移地址。显然，这里指的是段的偏移地址。
**什么是变量所在段的段类型？**
我们这里看到了D,C,b,T,U 这些类型的段，这里（包括后文）不会对所有段都做介绍，我们仅仅介绍那些我们感兴趣的部分。

 **d/D: 该类型表明当前变量是一个已初始化的变量，d 指明这是一个局部变量，D 则表示全局变量。**
 在main.cpp中，我们如此声明aa：

```C++
int aa=1;
0000000000000000 D aa
```

变量aa被保存在了D段，因为它已经被声明，并且作用域为全局。

**b/B: 对于非初始化的变量，我们用 b 来表示该变量是静态(static)或是局部的(local)，否则，用 B 或 C 来表示。**
在main.cpp中，我们是这样定义的：

```c++
int bb;
0000000000000000 B bb
```

bb只被声明但未被初始化，因此bb被保存在了B数据段

**t/T: 该类型指明了代码定义的位置。t 和 T 用于区分该函数是定义在文件内部(t)还是定义在文件外部(T)**

```C++
int main();
//对于编辑器而言，函数名只是一个符号，就算是main函数也不例外
0000000000000000 T main
```

段类型表明，main符号的作用于位于文件外部（被系统调用）

**U: 该类型表示未定义的引用，它们均未被分配段所在的相对地址。**
在main.cpp中，我们有如下代码：

```c++
//关键字extern表明int类型的变量dd已在外部文件中声明
extern int dd;
//String为自定义的一个类，定义及其实现全部放在外部文件中
//我们在很多地方引用到了String类，像这样：
String name;
String sayings[ArSize];
//以及一些iosteam文件中的函数，像：
cout << "Hi, what's your name? " << endl;
cin >> name;
```

以下这些看起来有些奇怪的符号名被放在了U段

```c++
                 U dd
                 U _ZlsRSoRK6String
                 U _ZltRK6StringS1_
                 U _ZN6String7HowManyEv
                 U _ZN6StringaSEPKc
                 U _ZN6StringC1Ev
                 U _ZN6StringD1Ev
                 U _ZN6StringixEi
                 U _ZrsRSiR6String
                 U _ZSt3cin
                 U _ZSt4cout
```

这么做就是为了告诉处理器：**虽然在本文件中没有这个变量或者函数定义，但是在其他文件中一定有!!!这也是为什么它们据未被分配偏移地址的原因。**

等等，这里有一个疑惑！
aa,bb,cc dd我还能理解，它们的名字我在反汇编的结果中一眼就能看的出来。
那么String类呢？我只看到了String这个关键字，你凭什么说它们就是String类的变量名呢？况且，这么多？？？
还有cin函数和cout函数怎么变名字了？？？

在《C++ Primer Plus》(第六版)的319页，作者说了这么一段话：

>链接程序要求每个不同的函数都有不同的符号名。在C语言中，一个名称只对应一个函数，因此这很容易实现。为满足内部需要，C语言编辑器可能将spiff这样的函数名翻译为_spiff。这种方法被称为C语言链接性。但在C++中，同一个名称可能对应多个函数，必须将这些函数翻译为不同的符号名称。。因此C++编译器执行名称校正或名称修饰，为重载函数生成不同符号的名称。例如，可能将spiff(int)转换为_spoff_i，而将spiff(double)转换为_spiff_d_d。这种方法被称为C++语言链接。

我们往前翻到第280页，作者写道：
>C++如何跟踪每一个重载函数呢？它给这些函数制指定了秘密身份。使用C++开发工具中的编辑器编写和编译程序时，C++编译器将执行一些神奇的操作——名称修饰（name decoration）或名称校正（name mangling），它根据函数原型中指定的形参类型对每个函数名进行加密。请看下述未经修饰的函数原型：
>long MyfunctionFoo(int , float)
>这种格式对于人类来说很适合，我们知道函数接受两个参数（一个int类型，一个float类型），并返回一个long值。而编译器将名称转换为一个不太好看的内部表示，来描述该接口，如下所示：
>？MyfunctionFoo@@YAXH
>对原始名称进行的表面看起来毫无意义的修饰（或校正，因人而异）将对参数数目和类型进行编码。添加的一组符号随函数特征标而异，而修饰时使用的约定随编辑器而异。

他想向我们表明一个啥意思呢？非常简单：
**链接程序要求每个不同的函数都有不同的符号名，为了满足链接性要求，C++编译器将执行一些神奇的操作——名称修饰（name decoration）或名称校正（name mangling）。它根据函数原型中指定的形参类型对每个函数名进行加密。**

这就是为什么函数名看起来非常奇怪的本质原因！

花一点小篇幅，我想好好夸一下作者Stephen Prata 老爷子。他的书不仅诙谐幽默（非显性，可能需要用心体会才能get到笑点），最最重要的，他把他的读者当成“傻子”一样对待。是的，他假设你是一个啥也不懂的门外汉，然后用非常长的篇幅向你解释语法为什么如此设计，解决了一个什么样的问题，以及潜在的风险。并且，他经常有意无意地向你解释某些看起来无关紧要的内容，就像上面的两个引用一样（我照着原文手打出来的），等你遇到它并且理解它的时候，就会产生一种茅塞顿开的感觉。这本书充满了这样的小提示。

为了能有一个更加直观的理解，我们看一下String类的声明文件：

```C++
class String
{
private:
	char* str;
	int len;
	static int num_strings;
	static const int CINLIM = 80;
public:
	String();
	String(const char*);
	String(const String&);
	~String();
	int length()const { return len; }

	String& operator=(const String&);
	String& operator=(const char*);
	char& operator[](int i);
	const char& operator[](int i)const;
	friend bool operator<(const String& st1, const String& st2);
	friend bool operator>(const String& st1, const String& st2);
	friend bool operator==(const String& st1, const String & st2);
	friend ostream& operator<<(std::ostream& os, const String& st);
	friend istream& operator>>(std::istream& is,  String& st);
	static int HowMany();
};

```
现在，我们String.cpp编译为目标文件，并查看其中包含的符号

```C++
//注意，这里是单独编译的
g++ -c String.cpp -o String.o && nm String.o
```

![Stringnm反编译](/assets/Stringnm反编译.png)

```C++
//我们以“友好”的方式再查看一下，注意实际名称以上面的那张图片为准，这里仅仅为了方便比较
nm -C String.o
```

![nm_String_可读](/assets/nm_String_可读.png)
可以看出，当前文件中定义的(类)函数全部被编译器加密为特定的函数名称，并保存在了代码段，编译器保证这些名称在**链接过程**中不会发生冲突。
此外，iostream文件中定义的函数被标记在了U段，这很正常，因为当前文件中并未提供它们的定义。


## 4. 链接

你也许注意到了，在第三小节中，我们开始越来越频繁地提起**链接**这个字眼，仅仅生成目标文件是不能够被操作系统运行的，需要经过链接的过程。那么，问题来了

### 4.1 什么是链接？

首先，我们试着把main.o文件链接一下

```C++
g++ main.o  -o main
```

输出如下：

![链接main](/assets/链接main_i567ew34o.png)

不出意外，编辑器报错了(是不是类似的场景觉得在哪里见过？)。为什么会报错呢？
原因在于，当前文件（main.cpp）使用了不属于本文件内定义的符号，而这些符号是在外部文件中被声明和定义的，然而由于编辑器找不到这些可以链接的目标文件，（没有把String添加进来一起链接）导致链接器无法得到一个完整可执行程序。换句话说，这个可执行程序是不完整的。

你可以这么理解：**整合各个目标文件中的符号至可执行程序的过程，就叫做链接。**
网上有位大神是这么描述的：**链接程序的主要工作就是将有关的目标文件彼此相连接，也即将在一个文件中引用的符号同该符号在另外一个文件中的定义连接起来，使得这些目标文件成为一个能够按操作系统装入执行的统一整体。**
说起链接，它有有两种方式：
**静态链接**：在链接阶段，会将汇编生成的目标文件.o与引用到的库一起链接打包到可执行文件中，程序运行的时候不再需要静态库文件。
**动态链接**：把调用的函数所在文件模块（DLL）和调用函数在文件中的位置等信息链接进目标程序，程序运行的时候再从DLL中寻找相应函数代码，因此需要相应DLL文件的支持。

### 4.2 链接的时候，编译器在做什么？

现在，我们把main.cpp和String.cpp联合编译一下：

```C++
g++ main.cpp String.cpp -o main
```

这次没有报错，并且得到了可执行程序main
![可执行main](/assets/可执行main_6ovx48g4g.png)
我们使用反汇编工具查看一下这个可执行文件的符号信息

```C++
nm -C  main
```

注意，这个main程序是可以被执行的
![nm_main](/assets/nm_main_sd019w774.png)
我们可以看到，String文件中的符号名称被正确地添加到了可执行程序main中。

**总结一下，链接的时候，编译器在做什么？**
**第一：对各个目标文件中没有定义的符号，在其他目标文件中寻找到相关的定义，并将其重命名后整合到统一的可执行文件下**
String类中定义的函数被赋予各种奇怪的名称后加入到了main程序中
**第二：把不同目标文件中生成的同类型的段进行合并。**
main.cpp中的外部变量dd被整合到了可执行程序main的D段
**第三：对不同目标文件中的变量进行地址重定位。**
在完成段整合后为dd变量赋予了相应的段偏移地址，不再是空地址

### 4.3 动态链接库（Windows下为 *.dll， Linux下为 *.so）

然而，即便是在一个可执行的程序中，我们依旧看到了位于U段中的未被分配偏移地址的符号，为什么会出现这种情况呢？

**这些“未定义”的“符号们”是为了支持动态链接库的功能而存在的。**
**更准确的说：一方面，为外部C++标准库文件预留位置。另一方面，为与操作系统交互预留位置。**

所谓动态链接库是指程序在运行的时候才去定位这个库，并且把这个库链接到进程的虚拟地址空间。对于某一个动态链接库来说，所有使用这个库的可执行文件都共享同一块物理地址空间，这个物理地址空间在当前动态链接库第一次被链接时加载到内存中。

我们使用ldd命令看一下main所依赖的外部动态链接库

```C++
//ldd 命令模拟加载可执行程序需要的动态链接库，但并不执行程序，后面的地址部分表示模拟装载过程中动态链接库的地址。
ldd main 
```

![ldd命令](/assets/ldd命令.png)
再次运行上述命令：
![ldd_1](/assets/ldd_1.png)

```c++
libstdc++ .so       提供c++程序的标准库
libgcc_s.so         gcc库文件
libm.so             glibc中对real-time部分的支持库
libc.so             C的库文件
linux-vdso.so       Linux内核库文件
ld-linux-x86-64.so  Linux内核库文件
```

可以看到，每次动态链接库的地址都是不一样的，因为这个地址是动态定位的，这正是动态链接库的名称由来。也就是说，main函数使用到的包含在上述库文件中的符号只有在程序运行时才会被动态的装载进程序中去。
如果程序在装在这些符号的过程中遇到了加载错误，会发生什么？

![找不到dll](/assets/找不到dll.png)

现在，你应该明白为什么会发生这种情况了吧（对于其他语言也是类似的）

### 4.4 静态链接库（Windows下为 *.lib， Linux下为 *.a）

**这里仅做简要介绍**
静态库可以简单看成是一组目标文件（.o/.obj文件）的集合，即很多目标文件经过压缩打包后形成的一个文件。之所以成为“静态库”，是因为在链接阶段，编译器会将汇编生成的目标文件.o与引用到的库一起链接打包到可执行文件中。因此对应的链接方式称为静态链接。
其特点为：
第一：静态库对函数库的链接是放在编译时期完成的，运行时不会再进行链接。
第二：程序在运行时与函数库再无瓜葛，移植方便。
第三：浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件。
第四：静态库对程序的更新、部署和发布页会带来麻烦。如果静态库liba.lib更新了，所以使用它的应用程序都需要重新编译、发布给用户（对于玩家来说，可能是一个很小的改动，却导致整个程序重新下载，全量更新）
![静态链接库](/assets/静态链接库.png)

正因如此，现在静态库已经很少被使用了。
不过，实际情况看起来似乎有点矛盾，我们回到问题的起点看一下。

## 5 回到起点——为什么要如此配置OpenCV？

在一份编译好的OpenCV中，以官方编译文件为例，常见的结构如下：

![OpenCV主目录](/assets/OpenCV主目录.png)

我们需要将include文件夹，也就是头文件包含目录添加到当前项目中，这个就不解释了。
接着，教程告诉我们，需要添加环境变量，也就是说把包含dll文件的目录添加到环境变量中去。（官方选择把dll打包编译了）

![环境变量](/assets/环境变量.png)

为什么这么做呢？前文已经说过了，dll动态链接库最大的优势就是可以节约内存资源，作为大型项目的OpenCV自然也会以这种方式链接可执行的应用程序。
什么是环境变量呢？通俗地说，环境变量可以被理解为系统级别的全局变量，这个变量一般来说指的是某个路径下的程序（dll也是一种程序）。
也许你曾使用过Win+R调出run.exe然后输入cmd打开命令行这种操作。这是因为cmd.exe已经被系统默认地添加到了环境变量中了。
你写好的OpenCV程序在运行时会自动寻找其所依赖的dll文件，在哪里寻找这些文件呢？就在你配置好的环境变量中寻找啊。如果你不指定路径，程序便无法通知操作系统调用想要的dll，这个时候就会提示找不到dll文件这种烦人的错误。当然，如果你路径设置正确，但不包含目标dll，那么效果是一样的。

紧接着，我们需要在配置文件中添加lib文件，就像这样：

![添加lib](/assets/添加lib.png)

**既然已经有dll文件了，为啥还要添加lib文件呢？**
俗话说得好，人总不能在一棵树上吊死不是？
动态库和静态库各有优劣，我们为什么不能将它们结合起来呢？
前文我们提到：**整合各个目标文件中的符号至可执行程序的过程，就叫做链接。**
这里谈一点我的理解：（正确性有待大神指正）
对于一个动态链接库，程序在链接的时候需要处理该dll下的全部符号，换句话说，程序至少需要遍历一遍dll数据才能进行后续操作。对于大型dll，这个解析过程可能会很长，这也许会导致程序加载缓慢的问题。为了提高效率，我们可以把dll的文件符号制作成一个索引数据然后放到一个静态链接库下。那么在链接的时候程序可以通过该索引直接查询到dll内对应的数据块，这样，就提高了程序的加载速度。

实际情况还真是这么做的：在使用动态库的情况下，有两个文件，一个是引入库（.LIB）文件，一个是DLL文件，引入库文件包含被DLL导出的函数的名称和位置，DLL则包含实际的函数和数据，应用程序使用LIB文件链接到所需要使用的DLL文件，因此在应用程序的可执行文件中，存放的不是被调用的函数代码，而是DLL中所要调用的函数的地址，这样当一个或多个应用程序运行时再把程序代码和被调用的DLL函数代码链接起来，从而节省了内存资源。

**这种情况下，DLL和.LIB文件必须随应用程序一起发行，缺.LIB文件导致编译错误，缺DLL文件导致运行错误。**
这就是为什么已经有dll库文件了仍然需要手动配置lib文件的原因。

本文结束，下期见。

