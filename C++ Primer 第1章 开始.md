# 第1章 开始

## 1.1 编写一个简单的C++程序

每个C++程序都包含一个或多个函数（function） ， 其中一个必须命名为main。 操作系统通过调用main来运行C++程序。  

```C++
int main()
{
	return 0;
}
```

一个函数的定义包含四部分： 返回类型（return type） 、 函数名（function name） 、一个括号包围的形参列表（parameter list， 允许为空） 以及函数体（function body） 。 虽然main函数在某种程度上比较特殊，但其定义与其他函数是一样的。  

**main函数的返回类型必须为int， 即整数类型。** int类型是一种内置类型，即语言自身定义的类型。 函数定义的最后一部分是函数体，它是一个以左花括号（curly brace）开始，以右花括号结束的语句块（block of statements）：

```c++
{
	return 0;
}
```

这个语句块中唯一的一条语句是return， 它结束函数的执行。 在本例中， return还会向调用者返回一个值。 当return语句包括一个值时， 此返回值的类型必须与函数的返回类型相容。 在本例中， main的返回类型是int， 而返回值0的确是一个int类型的值。  

在大多数系统中， main的返回值被用来指示状态。 返回值0表明成功， 非0的返回值的含义由系统定义， 通常用来指出错误类型。



### 1.1.1 编译，运行程序

#### 程序源文件命名约定

无论你使用命令行界面或者IDE， 大多数编译器都要求程序源码存储在一个或多个文件中。 程序文件通常被称为源文件（source file）。 在大多数系统中， 源文件的名字以一个后缀为结尾， 后缀是由一个句点后接一个或多个字符组成的。 后缀告诉系统这个文件是一个C++程序。 不同编译器使用不同的后缀命名约定， 最常见的括.cc、 .cxx、 .cpp、 .cp 及.C。  



## 1.2 初识输入输出  

C++语言并未定义任何输入输出（IO） 语句， 取而代之， 包含了一个全面的标准库（standard library） 来提供IO机制（以及很多其他设施） 。

本书中的很多示例都使用了iostream库。 iostream库包含两个基础类型istream和ostream， 分别表示输入流和输出流。 **一个流是一个字符序列， 是从IO设备读出或写入IO设备的。术语“流”（stream）想要表达的是， 随着时间的推移， 字符是顺序生成或消耗的。**



#### 标准输入输出对象 

标准库定义了4个IO对象。 为了处理输入， 我们使用一个名为cin（发音为see-in）的istream类型的对象。这个对象也被称为标准输入（standard input）。对于输出，我们使用一个名为cout（发音为see-out） 的ostream类型的对象。 此对象也被称为标准输出（standard output）。标准库还定义了其他两个ostream对象，名为cerr和clog（发音分别为see-err和see-log） 。 我们通常用cerr来输出警告和错误消息， 因此它也被称为标准错误（standard error） 。而clog用来输出程序运行时的一般性信息。

系统通常将程序所运行的窗口与这些对象关联起来。 因此， 当我们读取cin， 数据将从程序正在运行的窗口读入， 当我们向cout、 cerr和clog写入数据时， 将会写到同一个窗口。



#### 一个使用IO库的程序   

在书店程序中， 我们需要将多条记录合并成单一的汇总记录。 作为一个相关的， 但更简单的问题， 我们先来看一下如何将两个数相加。 通过使用IO库， 我们可以扩展main程序， 使之能提示用户输入两个数， 然后输出它们的和： 

```c++
#include <iostream>

int main()
{
  // prompt user to enter two numbers
  std::cout << "Enter two numbers:" << std::endl; 
  int v1 = 0, v2 = 0;
  std::cin >> v1 >> v2;  
  std::cout << "The sum of " << v1 << " and " << v2
      		<< " is " << v1 + v2 << std::endl;
  return 0;
}
```

这个程序开始时在用户屏幕打印

​		**Enter two numbers:**
​		然后等待用户输入。 如果用户键入

​		**3 7**
​		然后键入一个回车， 则程序产生如下输出：

​		***The sum of 3 and 7 is 10***  

程序的第一行  

```c++
#include <iostream>
```

告诉编译器我们想要使用iostream库。 尖括号中的名字（本例中是iostream） 指出了一个头文件（header） 。每个使用标准库设施的程序都必须包含相关的头文件。#include指令和头文件的名字必须写在同一行中。 通常情况下， #include指令必须出现在所有函数之外。 我们一般将一个程序的所有#include指令都放在源文件的开始位置。  



#### 向流写入数据  

main的函数体的第一条语句执行了一个表达式（expression） 。在C++中， 一个表达式产生一个计算结果， 它由一个或多个运算对象和（通常是） 一个运算符组成。 这条语句中的表达式使用了输出运算符（<<） 在标准输出上打印消息：  

```c++
std::cout << "Enter two numbers:" << std::endl; 
```

<<运算符接受两个运算对象： 左侧的运算对象必须是一个ostream对象， 右侧的运算对象是要打印的值。 **此运算符将给定的值写到给定的ostream对象中。 输出运算符的计算结果就是其左侧运算对象。 即计算结果就是我们写入给定值的那个ostream对象。**

我们的输出语句使用了两次<<运算符。 因为此运算符返回其左侧的运算对象， 因此第一个运算符的结果成为了第二个运算符的左侧运算对象。 这样， 我们就可以将输出请求连接起来。 因此， 我们的表达式等价于 

```c++
(std::cout << "Enter two numbers:") << std::endl; 
```

链中每个运算符的左侧运算对象都是相同的， 在本例中是std::cout。 我们也可以用两条语句生成相同的输出：  

```c++
std::cout << "Enter two numbers: "; 
std::cout << std::endl; 
```

第一个输出运算符给用户打印一条消息。这个消息是一个字符串字面值常量( string literal )，是用一对双引号包围的字符序列。 在双引号之间的文本被打印到标准输出。

第二个运算符打印endl， 这是一个被称为操纵符（manipulator） 的特殊值。 写入endl的效果是结束当前行， 并将与设备关联的缓冲区（buffer） 中的内容刷到设备中。 缓冲刷新操作可以保证到目前为止程序所产生的所有输出都真正写入输出流中， 而不是仅停留在内存中等待写入流。  

<u>WARNING: 程序员常常在调试时添加打印语句。 这类语句应该保证“一直”刷新流。 否则， 如果程序崩溃， 输出可能还留在缓冲区中， 从而导致关于程序崩溃位置的错误推断。</u>  



#### 使用标准库中的名字  

这个程序使用了std::cout和std::endl，而不是直接的cout和endl。前缀std::指出名字cout和endl是定义在名为std的命名空间（namespace）中的。 命名空间可以帮助我们避免不经意的名字定义冲突， 以及使用库中相同名字导致的冲突。 标准库定义的所有名字都在命名空间std中。

通过命名空间使用标准库有一个副作用： **当使用标准库中的一个名字时，必须显式说明我们想使用来自命名空间std中的名字。 例如，需要写出std::cout，通过使用作用域运算符（::）来指出我们想使用定义在命名空间std中的名字cout。**3.1节（第74页） 将给出一个更简单的访问标准库中名字的方法  



#### 从流读取数据  

在提示用户输入数据之后， 接下来我们希望读入用户的输入。 首先定义两个名为v1和v2的变量（variable） 来保存输入：  

```c++
  int v1 = 0, v2 = 0;
```

我们将这两个变量定义为int类型， int是一种内置类型， 用来表示整数。 还将它们初始化（initialize）为0。 初始化一个变量， 就是在变量创建的同时为它赋予一个值。下一条语句是  

```c++
  std::cin >> v1 >> v2;  
```

它读入输入数据。 **输入运算符（>>） 接受一个istream作为其左侧运算对象， 接受一个对象作为其右侧运算对象。它从给定的istream读入数据， 并存入给定对象中。 与输出运算符类似，输入运算符返回其左侧运算对象作为其计算结果。** 因此， 此表达式等价于  

```c++
(std::cin >> v1) >> v2;  
```

由于此运算符返回其左侧运算对象， 因此我们可以将一系列输入请求合并到单一语句中本例中的输入操作从std::cin读入两个值，并将第一个值存入v1，将第二个值存入v2。 换句话说， 它与下面两条语句的执行结果是一样的  

```c++
  std::cin >> v1;  
  std::cin >> v2;
```



#### 完成程序

剩下的就是打印计算结果了：  

```c++
  std::cout << "The sum of " << v1 << " and " << v2
      		<< " is " << v1 + v2 << std::endl;
```

这条语句虽然比提示用户输入的打印语句更长， 但原理上是一样的， 它将每个运算对象打印在标准输出上。本例一个有意思的地方在于，运算对象并不都是相同类型的值。 某些运算对象是字符串字面值常量， 例如"The sum of "。 其他运算对象则是int值， 如v1、 v2以及算术表达式v1+v2的计算结果。 标准库定义了不同版本的输入输出运算符， 来处理这些不同类型的运算对象。  



## 1.3 注释简介  

#### C++中注释的种类  

C++中有两种注释： 单行注释和界定符对注释。 单行注释以双斜线（//） 开始， 以换行符结束。 当前行双斜线右侧的所有内容都会被编译器忽略， 这种注释可以包含任何文本， 包括额外的双斜线。

另一种注释使用继承自C语言的两个界定符（/＊ 和＊/） 。 这种注释以/＊ 开始， 以＊/ 结束， 可以包含除＊/ 外的任意内容， 包括换行符。编译器将落在 /＊ 和＊/ 之间的所有内容都当作注释。  

注释界定符可以放置于任何允许放置制表符、 空格符或换行符的地方。 注释界定符可以跨越程序中的多行， 但这并不是必须的。 当注释界定符跨越多行时， 最好能显式指出其内部的程序行都属于多行注释的一部分。 我们所采用的风格是， 注释内的每行都以一个星号开头， 从而指出整个范围都是多行注释的一部分。

程序中通常同时包含两种形式的注释。 注释界定符对通常用于多行解释， 而双斜线注释常用于半行和单行附注。  

```c++
#include <iostream>
/*
 *	简单主函数：
 *	读取两个数，求它们的和
 */
int main()
{
	//提示用户输入两个数
    std::cout << "Enter two numbers:" << std:endl;
    int v1 = 0, v2 = 0;				//保存我们读入的输入数据的变量
    std::cin >> v1 >> v2;			//读取输入数据
    std:cout << "The sum of "<< v1 <<" and " << v2
   			 << " is " << v1 + v2 << std::end1;
    return 0;
}
```

#### 注释界定符不能嵌套

**界定符对形式的注释是以/＊ 开始， 以＊ /结束的。 因此， 一个注释不能嵌套在另一个注释之内。** 编译器对这类问题所给出的错误信息可能是难以理解、 令人迷惑的。 例如， 在你的系统中编译下面的程序， 就会产生错误：  

```c++
/*
*	注释对/* */不能嵌套。
*	“不能嵌套”几个字会被认为是源码，
*	像剩余程序一样处理
*/
int main()
{
	return 0;
}
```



## 1.4 控制流  

### 1.4.1 while 语句  

while语句反复执行一段代码， 直至给定条件为假为止。 我们可以用while语句编写一段程序， 求1到10这10个数之和：  

```c++
#include <iostream>

int main()
{
    int sum = 0, val = 1;
    // keep executing the while as long as val is less than or equal to 10
    while (val <= 10) {
        sum += val;  		// assigns sum + val to sum
        ++val;       		// add 1 to val
    }
    std::cout << "Sum of 1 to 10 inclusive is " 
              << sum << std::endl;

	return 0;
}
```

我们编译并执行这个程序， 它会打印出

`​Sum of 1 to 10 inclusive is 55`

与之前的例子一样，我们首先包含头文件iostream，然后定义main。在main中，我们定义两个int变量： sum用来保存和； val用来表示从1到10的每个数。 我们将sum的初值设置为0， val从1开始。这个程序的新内容是while语句。 while语句的形式为  

```c++
	while (condition)		
        statement
```

while语句的执行过程是交替地检测condition条件和执行关联的语句statement，直至condition为假时停止。所谓条件（condition） 就是一个产生真或假的结果的表达式。只要condition为真，statement就会被执行。 当执行完statement，会再次检测condition。 如果condition仍为真， statement再次被执行。 while语句持续地交替检测condition和执行statement， 直至condition为假为止。  



### 1.4.2 for 语句  

在我们的while循环例子中， 使用了变量val来控制循环执行次数。我们在循环条件中检测val的值， 在while循环体中将val递增。

这种在循环条件中检测变量、 在循环体中递增变量的模式使用非常频繁， 以至于C++语言专门定义了第二种循环语句——for语句， 来简化符合这种模式的语句。 可以用for语句来重写从1加到10的程序：  

```c++
#include <iostream>

int main()
{
    int sum = 0;

    // sum values from 1 through 10 inclusive
    for (int val = 1; val <= 10; ++val) 
        sum += val;  // equivalent to sum = sum + val
    std::cout << "Sum of 1 to 10 inclusive is " 
              << sum << std::endl;
    return 0;
}
```

简要重述一下for循环的总体执行流程：

​	1.创建变量val， 将其初始化为1。

​	2.检测val是否小于等于10。 若检测成功， 执行for循环体。 若失败，退出循环， 继续执行for循环体之后的第一条语句。

​	3.将val的值增加1。

​	4.重复第2步中的条件检测， 只要条件为真就继续执行剩余步骤。  



### 1.4.3 读取数量不定的输入数据  

在前一节中， 我们编写程序实现了1到10这10个整数求和。 扩展此程序一个很自然的方向是实现对用户输入的一组数求和。 在这种情况下， 我们预先不知道要对多少个数求和， 这就需要不断读取数据直至没有新的输入为止：  

```c++
#include <iostream> 

int main() 
{
	int sum = 0, value = 0;
	// read until end-of-file, calculating a running total of all values read
	while (std::cin >> value) 
		sum += value; // equivalent to sum = sum + value

	std::cout << "Sum is: " << sum << std::endl;
	return 0;
}

```

如果我们输入

`​3 4 5 6`

则程序会输出

`Sum is: 18`

main的首行定义了两个名为sum和value的int变量， 均初始化为0。我们使用value保存用户输入的每个数， 数据读取操作是在while的循环条件中完成的:

```c++
	while (std::cin >> value) 
```

while循环条件的求值就是执行表达式  

```c++
	std::cin >> value
```

此表达式从标准输入读取下一个数， 保存在value中。 输入运算符返回其左侧运算对象， 在本例中是std::cin。 因此， 此循环条件实际上检测的是std::cin。

当我们使用一个istream对象作为条件时， 其效果是检测流的状态。 如果流是有效的， 即流未遇到错误 ， 那么检测成功。 当遇到文件结束符（end-of-file） ，或遇到一个无效输入时（例如读入的值不是一个整数 ），istream对象的状态会变为无效。处于无效状态的istream对象会使条件变为假。

因此， 我们的while循环会一直执行直至遇到文件结束符（或输入错误）。 while循环体使用复合赋值运算符将当前值加到sum上。 一旦条件失败， while循环将会结束。 我们将执行下一条语句， 打印sum的值和一个endl。  



<u>从键盘输入文件结束符</u>

<u>当从键盘向程序输入数据时， 对于如何指出文件结束， 不同操作系统有不同的约定。 在Windows系统中， 输入文件结束符的方法是敲Ctrl+Z（按住Ctrl键的同时按Z键） ， 然后按Enter或Return键。 在UNIX系统中， 包括Mac OS X系统中， 文件结束符输入是用Ctrl+D。</u>



#### <u>再探编译</u>  

<u>编译器的一部分工作是寻找程序文本中的错误。 编译器没有能力检查一个程序是否按照其作者的意图工作， 但可以检查形式（form）上的错误。 下面列出了一些最常见的编译器可以检查出的错误。</u>

<u>语法错误（syntax error）：程序员犯了C++语言文法上的错误。下面程序展示了一些常见的语法错误； 每条注释描述了下一行中语句存在的错误：</u>

```C++
//错误：main的参数列表漏掉了
int main(
	//错误：endl后使用了冒号而非分号
	std::cout << "Read each file." << std::endl:
	//错误：字符串字面常量的两侧漏掉了引号
	std:cout << Update master. << std::endl;
	//错误：漏掉了第二个输出运算符
	std::cout << "Write new master." std::endl
	//错误：return语句漏掉了分号
	return 0
}
```

<u>类型错误（type error）:  C++中每个数据项都有其类型。 例如，10的类型是int（或者更通俗地说， “10是一个int型数据”） 。 单词"hello"， 包括两侧的双引号标记， 则是一个字符串字面值常量。 一个类型错误的例子是， 向一个期望参数为int的函数传递了一个字符串字面值常量。</u>

<u>声明错误（declaration error）：C++程序中的每个名字都要先声明后使用。 名字声明失败通常会导致一条错误信息。 两种常见的声明错误是： 对来自标准库的名字忘记使用std::、 标识符名字拼写错误：</u>  

```C++
#include <iostream>
int main(
{ 
	int v1 = 0, v2 = 0:
	std::cin >> v >> v2;				//错误：使用了"v"而非"v1"
	cout << v1 + v2 << std::endl;		//错误：cout未定义；应该是std::cout
	return 0;
}
```

<u>错误信息通常包含一个行号和一条简短描述， 描述了编译器认为的我们所犯的错误。 按照报告的顺序来逐个修正错误， 是一种好习惯。 因为一个单个错误常常会具有传递效应， 导致编译器在其后报告比实际数量多得多的错误信息。 另一个好习惯是在每修正一个错误后就立即重新编译代码， 或者最多是修正了一小部分明显的错误后就重新编译。 这就是所谓的“编辑-编译-调试”（edit-compile-debug） 周期。</u>  



### 1.4.4 if 语句  

与大多数语言一样， C++也提供了if语句来支持条件执行。 我们可以用if语句写一个程序， 来统计在输入中每个值连续出现了多少次：  

```c++
#include <iostream>

int main()
{
	int currVal = 0, val = 0;

	if (std::cin >> currVal) {        
		int cnt = 1;  
		while (std::cin >> val) { 
			if (val == currVal) 
				++cnt;            
			else {
				std::cout << currVal << " occurs " 
				          << cnt << " times" << std::endl;
				currVal = val;    
				cnt = 1;        
			}
		} 
		std::cout << currVal << " occurs " 
		          << cnt << " times" << std::endl;
	}
    
	return 0;
}
```

如果我们输入如下内容：

`42 42 42 42 42 55 55 62 100 100 100`

则输出应该是：

`42 occurs 5 times`

`55 occurs 2 times`

`62 occurs 1 times`

`100 occurs 3 times`

第一条if语句  

```c++
if (std::cin >> currVal)
{
	// ...
}
```

保证输入不为空。 与while语句类似， if也对一个条件进行求值。 第一条if语句的条件是读取一个数值存入currVal中。 如果读取成功， 则条件为真， 我们继续执行条件之后的语句块。 该语句块以左花括号开始，以return语句之前的右花括号结束。

如果需要统计出现次数的值， 我们就定义cnt， 用来统计每个数值连续出现的次数。 与上一小节的程序类似， 我们用一个while循环反复从标准输入读取整数。

while的循环体是一个语句块， 它包含了第二条if语句：  

```c++
if (val == currVal) 
	++cnt;        
else
{
   	std::cout << currVal << " occurs " << cnt << " times" << std::endl;
	currVal = val; 
	cnt = 1;       
}
```

这条if语句中的条件使用了相等运算符（==） 来检测val是否等于currVal。 如果是， 我们执行紧跟在条件之后的语句。 这条语句将cnt增加1， 表明我们再次看到了currVal。

如果条件为假， 即val不等于currVal， 则执行else之后的语句。 这条语句是一个由一条输出语句和两条赋值语句组成的语句块。 输出语句打印我们刚刚统计完的值的出现次数。 赋值语句将cnt重置为1， 将currVal重置为刚刚读入的值val。  



## 1.5 类简介  

### 1.5.1 Sales_item类  

Sales_item类的作用是表示一本书的总销售额、 售出册数和平均售价。 我们现在不关心这些数据如何存储、 如何计算。 为了使用一个类， 我们不必关心它是如何实现的， 只需知道类对象可以执行什么操作。

**每个类实际上都定义了一个新的类型，其类型名就是类名。因此，我们Sales_item类定义了一个名为Sales_item的类型。**与内置类型一样，我们可以定义类类型的变量。当我们写下如下语句

```c++
Sales_item item;
```

是想表达item是一个Sales_item类型的对象。 我们通常将“一个Sales_item类型的对象”简单说成“一个Sales_item对象”， 或更简单的“一个Sales_item”。除了可以定义Sales_item类型的变量之外， 我们还可以：

​	· 调用一个名为isbn的函数从一个Sales_item对象中提取ISBN书号。

​	· 用输入运算符（>>） 和输出运算符（<<） 读、 写Sales_item类型的对象。

​	· 用赋值运算符（=） 将一个Sales_item对象的值赋予另一个Sales_item对象。

​	· 用加法运算符（+） 将两个Sales_item对象相加。 两个对象必须表示同一本书（相同的ISBN） 。 加法结果是一个新的Sales_item对象， 其ISBN与两个运算对象相同， 而其总销售额和售出册数则是两个运算对象的对应值之和。

​	· 使用复合赋值运算符（+=） 将一个Sales_item对象加到另一个对象上。  



<u>关键概念： 类定义了行为</u>

<u>当你读这些程序时， 一件要牢记的重要事情是， 类Sales_item的作者定义了类对象可以执行的所有动作。 即， Sales_item类定义了创建一个Sales_item对象时会发生什么事情， 以及对Sales_item对象进行赋值、 加法或输入输出运算时会发生什么事情。</u>

<u>一般而言， 类的作者决定了类类型对象上可以使用的所有操作。 当前， 我们所知道的可以在Sales_item对象上执行的全部操作就是本节所列出的那些操作。</u>  



```c++
// Sales_item.h

#ifndef SALESITEM_H
// we're here only if SALESITEM_H has not yet been defined 
#define SALESITEM_H

//#include "Version_test.h" 

// Definition of Sales_item class and related functions goes here
#include <iostream>
#include <string>

class Sales_item {
    // these declarations are explained section 7.2.1, p. 270 
    // and in chapter 14, pages 557, 558, 561
    friend std::istream& operator>>(std::istream&, Sales_item&);
    friend std::ostream& operator<<(std::ostream&, const Sales_item&);
    friend bool operator<(const Sales_item&, const Sales_item&);
    friend bool
        operator==(const Sales_item&, const Sales_item&);
public:
    // constructors are explained in section 7.1.4, pages 262 - 265
    // default constructor needed to initialize members of built-in type
#if defined(IN_CLASS_INITS) && defined(DEFAULT_FCNS)
    Sales_item() = default;
#else
    Sales_item() : units_sold(0), revenue(0.0) { }
#endif
    Sales_item(const std::string& book) :
        bookNo(book), units_sold(0), revenue(0.0) { }
    Sales_item(std::istream& is) { is >> *this; }
public:
    // operations on Sales_item objects
    // member binary operator: left-hand operand bound to implicit this pointer
    Sales_item& operator+=(const Sales_item&);

    // operations on Sales_item objects
    std::string isbn() const { return bookNo; }
    double avg_price() const;
    // private members as before
private:
    std::string bookNo;      // implicitly initialized to the empty string
#ifdef IN_CLASS_INITS
    unsigned units_sold = 0; // explicitly initialized
    double revenue = 0.0;
#else
    unsigned units_sold;
    double revenue;
#endif
};

// used in chapter 10
inline
bool compareIsbn(const Sales_item& lhs, const Sales_item& rhs)
{
    return lhs.isbn() == rhs.isbn();
}

// nonmember binary operator: must declare a parameter for each operand
Sales_item operator+(const Sales_item&, const Sales_item&);

inline bool
operator==(const Sales_item& lhs, const Sales_item& rhs)
{
    // must be made a friend of Sales_item
    return lhs.units_sold == rhs.units_sold &&
        lhs.revenue == rhs.revenue &&
        lhs.isbn() == rhs.isbn();
}

inline bool
operator!=(const Sales_item& lhs, const Sales_item& rhs)
{
    return !(lhs == rhs); // != defined in terms of operator==
}

// assumes that both objects refer to the same ISBN
Sales_item& Sales_item::operator+=(const Sales_item& rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;
}

// assumes that both objects refer to the same ISBN
Sales_item
operator+(const Sales_item& lhs, const Sales_item& rhs)
{
    Sales_item ret(lhs);  // copy (|lhs|) into a local object that we'll return
    ret += rhs;           // add in the contents of (|rhs|) 
    return ret;           // return (|ret|) by value
}

std::istream&
operator>>(std::istream& in, Sales_item& s)
{
    double price;
    in >> s.bookNo >> s.units_sold >> price;
    // check that the inputs succeeded
    if (in)
        s.revenue = s.units_sold * price;
    else
        s = Sales_item();  // input failed: reset object to default state
    return in;
}

std::ostream&
operator<<(std::ostream& out, const Sales_item& s)
{
    out << s.isbn() << " " << s.units_sold << " "
        << s.revenue << " " << s.avg_price();
    return out;
}

double Sales_item::avg_price() const
{
    if (units_sold)
        return revenue / units_sold;
    else
        return 0;
}
#endif
```



#### 读写Sales_item  

既然已经知道可以对Sales_item对象执行哪些操作， 我们现在就可以编写使用类的程序了。 例如， 下面的程序从标准输入读入数据， 存入一个Sales_item对象中， 然后将Sales_item的内容写回到标准输出：  

```c++
#include <iostream>
#include "Sales_item.h"
int main()
{
	Sales_item book;
	//读入ISBN号、售出的册数以及销售价格
	std::cin >> book;
	//写入ISBN、售出的册数、总销售额和平均价格
	std::cout << book << std::endl;
	return 0;
}
```

如果输入：

`0-201-70353-X 4 24.99`

则输出为：

`0-201-70353-X 4 99.96 24.99`

输入表示我们以每本24.99美元的价格售出了4册书， 而输出告诉我们总售出册数为4， 总销售额为99.96美元， 而每册书的平均销售价格为24.99美元。

此程序以两个#include指令开始， 其中一个使用了新的形式。 包含来自标准库的头文件时， 也应该用尖括号（<>） 包围头文件名。 对于不属于标准库的头文件， 则用双引号（" "） 包围。

在main中我们定义了一个名为book的对象， 用来保存从标准输入读取出的数据。下一条语句读取数据存入对象中，第三条语句将对象打印到标准输出上并打印一个endl。  



#### Sales_item对象的加法  

下面是一个更有意思的例子， 将两个Sales_item对象相加：  

```c++
#include <iostream>
#include "Sales_item.h"

int main()
{
	Sales_item iteml, item2;
	std::cin >> iteml >> item2;
	//读取一对交易记录
	std::cout << iteml + item2 << std::endl;//打印它们的和
	return 0;
}
```

如果输入如下内容：

`0-201-78345-X 3 20.00`

`0-201-78345-X 2 25.00`

则输出为：

`0-201-78345-X 5 110 22`

此程序开始包含了Sales_item和iostream两个头文件。然后定义了两个Sales_item对象来保存销售记录。 我们从标准输入读取数据， 存入两个对象之中。 输出表达式完成加法运算并打印结果。

值得注意的是， 此程序看起来与第5页的程序非常相似： 读取两个输入数据并输出它们的和。 造成如此相似的原因是， 我们只不过将运算对象从两个整数变为两个Sales_item而已， 但读取与打印和的运算方式没有发生任何变化。 两个程序的另一个不同之处是， “和”的概念是完全不一样的。 对于int， 我们计算传统意义上的和——两个数值的算术加法结果。对于Sales_item对象，我们用了一个全新的“和”的概念——两个Sales_item对象的成员对应相加的结果。



### 1.5.2 初识成员函数  

将两个Sales_item对象相加的程序首先应该检查两个对象是否具有相同的ISBN。 方法如下：  

```c++
#include <iostream>
#include "Sales_item.h"

int main()
{
	Sales_item item1, item2;
	std::cin >> item1 >> item2;
	//首先检查iteml和item2是否表示相同的书
	if (item1.isbn() == item2.isbn ()) {
		std::cout << item1 + item2 << std::endl;
		return 0;		//表示成功
	} else {
		std::cerr << "Data must refer to same ISBN" << std::endl;
		return -1;		//表示失败
	}
}
```

此程序与上一版本的差别是if语句及其else分支。 即使不了解这个if语句的检测条件， 我们也很容易理解这个程序在干什么。 如果条件成立， 程序打印计算结果， 并返回0， 表明成功。 如果条件失败， 我们执行跟在else之后的语句块， 打印一条错误信息， 并返回一个错误标识。  



#### 什么是成员函数？  

这个if语句的检测条件  

```c++
item1.isbn() == item2.isbn()
```

调用名为isbn的成员函数（member function） 。成员函数是定义为类的一部分的函数，有时也被称为方法（method） 。

我们通常以一个类对象的名义来调用成员函数。 例如， 上面相等表达式左侧运算对象的第一部分  

```c++
item1.isbn()
```

使用点运算符（.）来表达我们需要 “名为item1的对象的isbn成员” 。点运算符只能用于类类型的对象。其左侧运算对象必须是一个类类型的对象， 右侧运算对象必须是该类型的一个成员名， 运算结果为右侧运算对象指定的成员。

当用点运算符访问一个成员函数时，通常我们是想（效果也确实是）调用该函数。我们使用调用运算符（ () ）来调用一个函数。调用运算符是一对圆括号，里面放置实参（argument） 列表（可能为空）。成员函数isbn并不接受参数。 因此

```c++
item1.isbn()
```

调用名为item1的对象的成员函数isbn， 此函数返回item1中保存的ISBN书号。

在这个if条件中，相等运算符的右侧运算对象也是这样执行的——它返回保存在item2中的ISBN书号。 如果ISBN相同， 条件为真， 否则为假。  



## 1.6 书店程序  

现在我们已经准备好完成书店程序了。 我们需要从一个文件中读取销售记录，生成每本书的销售报告， 显示售出册数、 总销售额和平均售价。 我们假定每个ISBN书号的所有销售记录在文件中是聚在一起保存的。

我们的程序会将每个ISBN的所有数据合并起来， 存入名为total的变量中。 我们使用另一个名为trans的变量保存读取的每条销售记录。 如果trans和total指向相同的ISBN， 我们会更新total的值。 否则， 我们会打印total的值， 并将其重置为刚刚读取的数据（trans）：  

```c++
#include <iostream>
#include "Sales_item.h"

int main()
{
    // variable to hold data for the next transaction
    Sales_item total; 

    // read the first transaction and ensure that there are data to process
    if (std::cin >> total) {
        Sales_item trans; 	// variable to hold the running sum
        // read and process the remaining transactions
        while (std::cin >> trans) {
            // if we're still processing the same book
            if (total.isbn() == trans.isbn())
                total += trans; // update the running total 
            else {
                // print results for the previous book 
                std::cout << total << std::endl;
                total = trans; // total now refers to the next book
            }
        }
        // print the last transaction
        std::cout << total << std::endl; 
    }
    else {
        // no input! warn the user
        std::cerr << "No data?!" << std::endl;
        return -1;  // indicate failure
    }
    
    return 0;
}

```

与往常一样， 首先包含要使用的头文件： 来自标准库的iostream和自己定义的Sales_item.h。在main中，我们定义了一个名为total的变量， 用来保存一个给定的ISBN的数据之和。 我们首先读取第一条销售记录， 存入total中， 并检测这次读取操作是否成功。 如果读取失败， 则意味着没有任何销售记录， 于是直接跳到最外层的else分支， 打印一条警告信息， 告诉用户没有输入。

假定已经成功读取了一条销售记录， 我们继续执行最外层if之后的语句块。 这个语句块首先定义一个名为trans的对象， 它保存读取的销售记录。 接下来的while语句将读取剩下的所有销售记录。 与我们之前的程序一样，while条件是一个从标准输入读取值的操作。 在本例中， 我们读取一个Sales_item对象， 存入trans中。 只要读取成功， 就执行while循环体。

while的循环体是一个单个的if语句， 它检查ISBN是否相等。 如果相等， 使用复合赋值运算符将trans加到total中。 如果ISBN不等， 我们打印保存在total中的值， 并将其重置为trans的值。 在执行完if语句后， 返回到while的循环条件， 读取下一条销售记录， 如此反复， 直至所有销售记录都处理完。

当while语句终止时， total保存着文件中最后一个ISBN的数据。 我们在语句块的最后一条语句中打印这最后一个ISBN的total值， 至此最外层if语句就结束了。
