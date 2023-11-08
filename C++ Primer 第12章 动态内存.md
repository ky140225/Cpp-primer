# 第12章 动态内存  

**全局对象在程序启动时分配， 在程序结束时销毁。 对于局部自动对象， 当我们进入其定义所在的程序块时被创建， 在离开块时销毁。 局部static对象在第一次使用前分配， 在程序结束时销毁。**

除了自动和static对象外， C++还支持动态分配对象。 动态分配的对象的生存期与它们在哪里创建是无关的， 只有当显式地被释放时， 这些对象才会销毁。

动态对象的正确释放被证明是编程中极其容易出错的地方。 为了更安全地使用动态对象， 标准库定义了两个智能指针类型来管理动态分配的对象。 当一个对象应该被释放时， 指向它的智能指针可以确保自动地释放它。

我们的程序到目前为止只使用过静态内存或栈内存。 静态内存用来保存局部static对象（参见6.6.1节， 第185页） 、 类static数据成员（参见7.6节， 第268页） 以及定义在任何函数之外的变量 。 栈内存用来保存定义在函数内的非static对象。 分配在静态或栈内存中的对象由编译器自动创建和销毁。 对于栈对象，仅在其定义的程序块运行时才存在； static对象在使用之前分配， 在程序结束时销毁。

除了静态内存和栈内存， 每个程序还拥有一个内存池。 这部分内存被称作自由空间 （free store） 或堆（heap） 。 程序用堆来存储动态分配（dynamically allocate） 的对象——即， 那些在程序运行时分配的对象。 动态对象的生存期由程序来控制，也就是说，当动态对象不再使用时，我们的代码必须显式地销毁它们。



## 12.1 动态内存与智能指针

**在C++中， 动态内存的管理是通过一对运算符来完成的： new， 在动态内存中为对象分配空间并返回一个指向该对象的指针， 我们可以选择对对象进行初始化； delete， 接受一个动态对象的指针， 销毁该对象， 并释放与之关联的内存。**

动态内存的使用很容易出问题， 因为确保在正确的时间释放内存是极其困难的。 有时我们会忘记释放内存， 在这种情况下就会产生内存泄漏； 有时在尚有指针引用内存的情况下我们就释放了它 ， 在这种情况下就会产生引用非法内存的指针。

为了更容易（同时也更安全） 地使用动态内存， 新的标准库提供了两种**智能指针（smart pointer） 类型来管理动态对象。 智能指针的行为类似常规指针，重要的区别是它负责自动释放所指向的对象。** 新标准库提供的这两种智能指针的**区别在于管理底层指针的方式： shared_ptr允许多个指针指向同一个对象； unique_ptr则“独占”所指向的对象。 标准库还定义了一个名为weak_ptr的伴随类， 它是一种弱引用， 指向shared_ptr所管理的对象。 这三种类型都定义在memory头文件中。**



### 12.1.1 shared_ptr类

**类似vector， 智能指针也是模板（参见3.3节， 第86页） 。 因此， 当我们创建一个智能指针时 ， 必须提供额外的信息——指针可以指向的类型。** 与vector一样， 我们在尖括号内给出类型， 之后是所定义的这种智能指针的名字：

```c++
shared_ptr<string> p1;
//shared_ptr,可以指向string
shared_ptr<list<int>> p2;
//shared_ptr,可以指向int的list
```

**默认初始化的智能指针中保存着一个空指针**（参见2.3.2节， 第48页） 。在12.1.3节中（见第412页） ，我们将介绍初始化智能指针的其他方法。

**智能指针的使用方式与普通指针类似。 解引用一个智能指针返回它指向的对象。 如果在一个条件判断中使用智能指针， 效果就是检测它是否为空：**

```c++
//如果p1不为空，检查它是否指向一个空string
if(p1 && p1->empty())
	*p1 = "hi";			//如果p1指向一个空string，解引用p1，将一个新值赋予string
```

表12.1列出了shared_ptr和unique_ptr都支持的操作。 只适用于shared_p tr的操作列于表12.2中。

![表12.1 shared_ptr和unique_ptr都支持的操作](C:\Users\Grey\Desktop\C++ Primer\图片\表12.1 shared_ptr和unique_ptr都支持的操作.png)

![表12.2 shared ptr独有的操作](C:\Users\Grey\Desktop\C++ Primer\图片\表12.2 shared ptr独有的操作.png)



#### make_shared函数

最安全的分配和使用动态内存的方法是调用一个名为make_shared的标准库函数。此函数在动态内存中分配一个对象并初始化它， 返回指向此对象的shared_ptr。 与智能指针一样， make_shared也定义在头文件memory中。

当要用make_shared时， 必须指定想要创建的对象的类型。 定义方式与模板类相同， 在函数名之后跟一个尖括号， 在其中给出类型：

```c++
//指向一个值为42的int的shared_ptr
shared_ptr<int> p3 = make_shared<int>(42);

//p4指向一个值为"9999999999"的string
shared_ptr<string> p4 = make_shared<string>(10,'9');

//p5指向一个值初始化的（参见3.3.1节，第88页）int,即，值为0
shared_ptr<int> p5 = make_shared<int>();
```

类似顺序容器的emplace成员（参见9.3.1节， 第308页） ， make_shared用其参数来构造给定类型的对象。 例如， 调用make_shared<string> 时传递的参数必须与string的某个构造函数相匹配，调用make_shared<int> 时传递的参数必须能用来初始化一个int， 依此类推。 **如果我们不传递任何参数， 对象就会进行值初始化**（参见3.3.1节，第88页） 。

当然， 我们通常用auto（参见2.5.2节， 第61页） 定义一个对象来保存make_shared的结果， 这种方式较为简单：

```c++
//p6指向一个动态分配的空vector<string>
auto p6 = make_shared<vector<string>>();
```



#### shared_ptr的拷贝和赋值

当进行拷贝或赋值操作时， 每个shared_ptr都会记录有多少个其他shared_ptr指向相同的对象：

```c++
auto p = make_shared<int>(42);		  //p指向的对象只有p一个引用者
auto q(p);							//p和q指向相同对象，此对象有两个引用者
```

我们可以认为每个shared_ptr都有一个关联的计数器， 通常称其为引用计数（reference count ） 。 无论何时我们拷贝一个shared_ptr， 计数器都会递增。 例如， **当用一个shared_ptr初始化另一个shared_ptr， 或将它作为参数传递给一个函数（参见6.2.1节， 第188页） 以及作为函数的返回值（参见6.3.2节， 第201页） 时， 它所关联的计数器就会递增。** **当我们给shared_ptr赋予一个新值或是shared_ptr被销毁（例如一个局部的shared_ptr离开其作用域（参见6.1.1节，第184页）） 时， 计数器就会递减。**

一旦一个shared_ptr的计数器变为0， 它就会自动释放自己所管理的对象：

```c++
auto r = make_shared<int>(42)；		//r指向的int只有一个引用者
r = q;								//给上赋值，令它指向另一个地址
                                    //递增q指向的对象的引用计数
                                    //递减r原来指向的对象的引用计数
                                    //r原来指向的对象已没有引用者，会自动释放
```

此例中我们分配了一个int，将其指针保存在r中。接下来，我们将一个新值赋予r。 在此情况下， r是唯一指向此int的shared_ptr， 在把q赋给r的过程中， 此int被自动释放。



#### shared_ptr自动销毁所管理的对象……

当指向一个对象的最后一个shared_ptr被销毁时，shared_ptr类会自动销毁此对象。它是通过另一个特殊的成员函数——析构函数（destructor）完成销毁工作的。类似于构造函数，每个类都有一个析构函数。就像构造函数控制初始化一样，析构函数控制此类型的对象销毁时做什么操作。

析构函数一般用来释放对象所分配的资源。 例如， string的构造函数（以及其他string成员）会分配内存来保存构成string的字符。string的析构函数就负责释放这些内存。类似的，vector的若干操作都会分配内存来保存其元素。vector的析构函数就负责销毁这些元素，并释放它们所占用的内存。

**shared_ptr的析构函数会递减它所指向的对象的引用计数。如果引用计数变为0，shared_ptr的析构函数就会销毁对象，并释放它占用的内存。**



#### ……shared_ptr还会自动释放相关联的内存

当动态对象不再被使用时，shared_ptr类会自动释放动态对象，这一特性使得动态内存的使用变得非常容易。 例如，我们可能有一个函数，它返回一个shared_ptr，指向一个Foo类型的动态分配的对象，对象是通过一个类型为T的参数进行初始化的：

```c++
//factory返回一个shared_ptr,指向一个动态分配的对象
shared_ptr<Foo> factory(T arg)
{
    //恰当地处理arg
    //shared_ptr负责释放内存
    return make_shared<Foo>(arg);
}
```

由于factory返回一个shared_ptr，所以我们可以确保它分配的对象会在恰当的时刻被释放。例如，下面的函数将factory返回的shared_ptr保存在局部变量中：

```c++
void use_factory(T arg)
{
    shared_ptr<Foo> p = factory(arg);
    //使用p
}	//p离开了作用域，它指向的内存会被自动释放掉

```

由于p是use_factory的局部变量，在use_factory结束时它将被销毁（参见6.1.1节，第184页）。当p被销毁时，将递减其引用计数并检查它是否为0。在此例中，p是唯一引用factory返回的内存的对象。由于p将要销毁，p指向的这个对象也会被销毁，所占用的内存会被释放。

但如果有其他shared_ptr也指向这块内存，它就不会被释放掉：

```c++
shared_ptr<Foo>  use_factory(T arg)
{
    shared_ptr<Foo> p = factory(arg);
    //使用p
    return p;			//当我们返回p时，引用计数进行了递增操作
}	//p离开了作用域，但它指向的内存不会被释放掉
```

在此版本中， use_factory中的return语句向此函数的调用者返回一个p的拷贝。 拷贝一个shared_ptr会增加所管理对象的引用计数值。现在当p被销毁时，它所指向的内存还有其他使用者。对于一块内存，shared_ptr类保证只要有任何shared_ptr对象引用它，它就不会被释放掉。

由于在最后一个shared_ptr销毁前内存都不会释放，保证shared_ptr在无用之后不再保留就非常重要了。 如果你忘记了销毁程序不再需要的shared_ptr， 程序仍会正确执行， 但会浪费内存。**share_ptr在无用之后仍然保留的一种可能情况是，你将shared_ptr存放在一个容器中，随后重排了容器，从而不再需要某些元素。在这种情况下，你应该确保用erase删除那些不再需要的shared_ptr元素。**



#### 使用了动态生存期的资源的类

程序使用动态内存出于以下三种原因之一： 

1. 程序不知道自己需要使用多少对象

2. 程序不知道所需对象的准确类型

3. 程序需要在多个对象间共享数据

容器类是出于第一种原因而使用动态内存的典型例子， 我们将在第15章看到出于第二种原因而使用动态内存的例子。 在本节中， 我们将定义一个类， 它使用动态内存是为了让多个对象能共享相同的底层数据。

到目前为止， 我们使用过的类中， 分配的资源都与对应对象生存期一致。 例如， 每个vector“拥有”其自己的元素。 当我们拷贝一个vector时， 原vector和副本vector中的元素是相互分离的：

```c++
vector<string> v1;		//空vector

{	//新作用域
    vector<string> v2 = {"a","an","the"};
    v1 = v2;			//从v2拷贝元素到v1中
}	//v2被销毁，其中的元素也被销毁
	//v1有三个元素，是原来v2中元素的拷贝
```

> **v2离开作用域被销毁了，但v1已经拷贝成功了。v1，v2内存地址是不一样的**

由一个vector分配的元素只有当这个vector存在时才存在。 当一个vector被销毁时， 这个vector中的元素也都被销毁。

但某些类分配的资源具有与原对象相独立的生存期。 例如， 假定我们希望定义一个名为Blob的类， 保存一组元素。 与容器不同， 我们希望Blob对象的不同拷贝之间共享相同的元素。 即， 当我们拷贝一个Blob时， 原Blob对象及其拷贝应该引用相同的底层元素*（同一地址的元素）*。

一般而言， 如果两个对象共享底层的数据， 当某个对象被销毁时，我们不能单方面地销毁底层数据：

```c++
Blob<string> b1;		//Blob

{	//新作用域
	Blob<string> b2 = {"a","an","the"};
	b1 = b2;			//b1和b2共享相同的元素
}	//b2被销毁了，但b2中的元素不能销毁
	//b1指向最初由b2创建的元素
```

> b1和b2使用堆内存，共享相同的元素，当b2被销毁后，b1仍使用这块数据

在此例中， b1和b2共享相同的元素。 当b2离开作用域时， 这些元素必须保留， 因为b1仍然在使用它们。

<u>使用动态内存的一个常见原因是允许多个对象共享相同的状态。</u>



#### 定义StrBlob类

最终， 我们会将Blob类实现为一个模板， 但我们直到16.1.2节（第583页） 才会学习模板的相关知识。 因此， 现在我们先定义一个管理string的类， 此版本命名为StrBlob。

实现一个新的集合类型的最简单方法是使用某个标准库容器来管理元素。 采用这种方法， 我们可以借助标准库类型来管理元素所使用的内存空间。 在本例中， 我们将使用vector来保存元素。

但是， 我们不能在一个Blob对象内直接保存vector， 因为一个对象的成员在对象销毁时也会被销毁。 例如， 假定b1和b2是两个Blob对象，共享相同的vector。 如果此vector保存在其中一个Blob中——例如b2中，那么当b2离开作用域时， 此vector也将被销毁， 也就是说其中的元素都将不复存在。 为了保证vector中的元素继续存在， 我们将vector保存在动态内存中。

为了实现我们所希望的数据共享， 我们为每个StrBlob设置一个shared_ptr来管理动态分配的vector。 此shared_ptr的成员将记录有多少个StrBlob共享相同的vector， 并在vector的最后一个使用者被销毁时释放vector。

我们还需要确定这个类应该提供什么操作。 当前， 我们将实现一个vector操作的小的子集。 我们会修改访问元素的操作（如front和back） ： 在我们的类中， 如果用户试图访问不存在的元素， 这些操作会抛出一个异常。

我们的类有一个默认构造函数和一个构造函数， 接受单一的initializer_list<string> 类型参数（参见6.2.6节， 第198页） 。 此构造函数可以接受一个初始化器的花括号列表。

```c++
class StrBlob {
public:
    typedef std::vector<std::string>::size_type size_type;
    
    StrBlob();
    StrBlob(std::initializer_list<std::string> il);
    
    size_type size() const { return data->size(); }
    
    bool empty() const { return data->empty(); }
    
    //添加和删除元素
    void push_back(const std::string &t){data->push_back(t);}
    void pop_back();
    
    //元素访问
    std::string& front();
    std::string& back();
    
private:
    std::shared_ptr<std::vector<std::string>> data;
    
    //如果data[i]不合法，抛出一个异常
    void check(size_type i, const std::string &msg) const;
}；
```

在此类中， 我们实现了size、 empty和push_back成员。 这些成员通过指向底层vector的data成员来完成它们的工作。 例如， 对一个StrBlob对象调用size() 会调用data->size() ， 依此类推。



#### StrBlob构造函数

两个构造函数都使用初始化列表（参见7.1.4节， 第237页） 来初始化其data成员， 令它指向一个动态分配的vector。 默认构造函数分配一个空vector：

```c++
StrBlob::StrBlob(): data(make_shared<vector<string>>()){}
StrBlob::StrBlob(initializer_list<string> il):
					data(make_shared<vector<string>>(il)){}
```

接受一个initializer_list的构造函数将其参数传递给对应的vector构造函数（参见2.2.1节， 第39页） 。 此构造函数通过拷贝列表中的值来初始化vector的元素。



#### 元素访问成员函数

pop_back、 front和back操作访问vector中的元素。 这些操作在试图访问元素之前必须检查元素是否存在。 由于这些成员函数需要做相同的检查操作， 我们为StrBlob定义了一个名为check的private工具函数， 它检查一个给定索引是否在合法范围内。 除了索引， check还接受一个string参数， 它会将此参数传递给异常处理程序， 这个string描述了错误内容：

```c++
void StrBlob::check(size_type i, const string &msg) const
{
    if ( i >= data->size())
 	   throw out_of_range(msg);
3
```

pop_back和元素访问成员函数首先调用check。 如果check成功， 这些成员函数继续利用底层vector的操作来完成自己的工作：

```c++
string& strBlob::front()
{
    //如果vector为空，check会抛出一个异常
    check(0, "front on empty StrBlob");
    return data->front()
}
    
string& StrBlob::back()
{
    check(0, "back on empty StrBlob");
    return data->back()
}
    
void StrBlob::pop_back()
{
    check(0, "pop back on empty StrBlob");
    data->pop_back();
}
```

front和back应该对const进行重载（参见7.3.2节， 第247页） 



#### StrBlob的拷贝、 赋值和销毁

类似Sales_data类， StrBlob使用默认版本的拷贝、 赋值和销毁成员函数来对此类型的对象进行这些操作（参见7.1.5节， 第239页） 。 默认情况下， 这些操作拷贝、 赋值和销毁类的数据成员。 我们的StrBlob类只有一个数据成员， 它是shared_ptr类型。 因此， 当我们拷贝、 赋值或销毁一个StrBlob对象时， 它的shared_ptr成员会被拷贝、 赋值或销毁。

如前所见， 拷贝一个shared_ptr会递增其引用计数； 将一个shared_ptr赋予另一个shared_ptr会递增赋值号右侧shared_ptr的引用计数，而递减左侧shared_ptr的引用计数。 如果一个shared_ptr的引用计数变为0， 它所指向的对象会被自动销毁。 因此， 对于由StrBlob构造函数分配的vector， 当最后一个指向它的StrBlob对象被销毁时， 它会随之被自动销毁。



### 12.1.2 直接管理内存

C++语言定义了两个运算符来分配和释放动态内存。 运算符new分配内存， delete释放new分配的内存。

相对于智能指针，使用这两个运算符管理内存非常容易出错。 而且， 自己直接管理内存的类与使用智能指针的类不同， 它们不能依赖类对象拷贝、 赋值和销毁操作的任何默认定义（参见7.1.4节， 第237页） 。 因此， 使用智能指针的程序更容易编写和调试。



#### 使用new动态分配和初始化对象

在自由空间分配的内存是无名的，因此new无法为其分配的对象命名， 而是返回一个指向该对象的指针：

```c++
int *pi = new int;					//pi指向一个动态分配的、未初始化的无名对象
```

此new表达式在自由空间构造一个int型对象， 并返回指向该对象的指针。

默认情况下， **动态分配的对象是默认初始化的（参见2.2.1节， 第40页） ， 这意味着内置类型或组合类型的对象的值将是未定义的， 而类类型对象将用默认构造函数进行初始化：**

```c++
string *ps = new string;				 //初始化为空string
int *pi = new int;						//pi指向一个未初始化的int
```

我们**可以使用直接初始化方式（参见3.2.1节， 第76页） 来初始化一个动态分配的对象。** 我们可以使用传统的构造方式（使用圆括号） ， 在新标准下， 也可以使用列表初始化（使用花括号） ：

```c++
int *pi = new int(1024);				//pi指向的对象的值为1024

string *ps = new string(10,'9');	     //*ps为"9999999999"

//vector有10个元素，值依次从0到9
vector<int> *pv = new vector<int>{0,1,2,3,4,5,6,7,8,9};
```

**也可以对动态分配的对象进行值初始化**（参见3.3.1节， 第88页） ，只需在类型名之后跟一对空括号即可：

```c++
string *ps1 = new string;				 //默认初始化为空string
string *ps = new string();				 //值初始化为空string
int *pil = new int;						//默认初始化；*pi1的值未定义
int *pi2 = new int();					//值初始化为0；*p12为0
```

对于定义了自己的构造函数（参见7.1.4节， 第235页） 的类类型（例如string） 来说， 要求值初始化是没有意义的；不管采用什么形式，对象都会通过默认构造函数来初始化。 但对于内置类型， 两种形式的差别就很大了； 值初始化的内置类型对象有着良好定义的值， 而默认初始化的对象的值则是未定义的。 类似的， 对于类中那些依赖于编译器合成的默认构造函数的内置类型成员， 如果它们未在类内被初始化， 那么它们的值也是未定义的（参见7.1.4节， 第236页） 。

如果我们提供了一个括号包围的初始化器， 就可以使用auto（参见2.5.2节， 第61页） 从此初始化器来推断我们想要分配的对象的类型。 但是，**由于编译器要用初始化器的类型来推断要分配的类型 ， 只有当括号中仅有单一初始化器时才可以使用auto：**

```c++
auto pl = new auto(obj);			//p1指向一个与obj类型相同的对象
									//该对象用obj进行初始化
auto p2 = new auto(a, b, c);		//错误：括号中只能有单个初始化器
```

p1的类型是一个指针， 指向从obj自动推断出的类型。 若obj是一个int， 那么p1就是int *； 若obj是一个string， 那么p1是一个string * ； 依此类推。 新分配的对象用obj的值进行初始化。



#### 动态分配的const对象

用new分配const对象是合法的：

```c++
//分配并初始化一个const int
const int *pci = new const int(1024);

//分配并默认初始化一个const的空string
const string *pcs = new const string;
```

类似其他任何const对象， 一个动态分配的const对象必须进行初始化。 对于一个定义了默认构造函数（参见7.1.4节， 第236页） 的类类型， 其const动态对象可以隐式初始化， 而其他类型的对象就必须显式初始化。 **由于分配的对象是const的， new返回的指针是一个指向const的指针**（参见2.4.2节， 第56页） 。



#### 内存耗尽

虽然现代计算机通常都配备大容量内存， 但是自由空间被耗尽的情况还是有可能发生。 一旦一个程序用光了它所有可用的内存， new表达式就会失败。 默认情况下， **如果new不能分配所要求的内存空间， 它会抛出一个类型为bad_alloc（参见5.6节， 第173页） 的异常。** 我们可以改变使用new的方式来阻止它抛出异常：

```c++
//如果分配失败，new返回一个空指针
int *p1 = new int;				//如果分配失败，new抛出std::bad_alloc
int *p2 = new (nothrow) int;	 //如果分配失败，new返回一个空指针
```

我们称这种形式的new为定位new（placement new） ， 其原因我们将在19.1.2节（第729页） 中解释。 **定位new表达式允许我们向new传递额外的参数。** 在此例中， 我们传递给它一个由标准库定义的名为nothrow的对象。 如果将nothrow传递给new，我们的意图是告诉它不能抛出异常。 如果这种形式的new不能分配所需内存，它会返回一个空指针。bad_alloc和nothrow都定义在头文件new中。



#### 释放动态内存

为了防止内存耗尽， 在动态内存使用完毕后， 必须将其归还给系统。 我们通过delete表达式（delete expression） 来将动态内存归还给系统。 delete表达式接受一个指针， 指向我们想要释放的对象：

```c++
delete p;						//p必须指向一个动态分配的对象或是一个空指针
```

与new类型类似， delete表达式也执行两个动作： 销毁给定的指针指向的对象； 释放对应的内存。

> 建议delete之后，再把p指向为空（p = nullptr）



#### 指针值和delete

**我们传递给delete的指针必须指向动态分配的内存， 或者是一个空指针**（参见2.3.2节， 第48页） 。 **释放一块并非new分配的内存， 或者将相同的指针值释放多次， 其行为是未定义的：**

```c++
int i, *pil = &i, *pi2 = nullptr;
double *pd = new double(33), *pd2 = pd;
delete i;			//错误：i不是一个指针
delete pi1;			//未定义：pi1指向一个局部变量
delete pd;			//正确
delete pd2;			//未定义：pd2指向的内存已经被释放了
delete pi2;			//正确：释放一个空指针总是没有错误的
```

对于delete i 的请求， 编译器会生成一个错误信息， 因为它知道 i 不是一个指针。 执行delete pi1和pd2所产生的错误则更具潜在危害： **通常情况下，编译器不能分辨一个指针指向的是静态还是动态分配的对象。 类似的， 编译器也不能分辨一个指针所指向的内存是否已经被释放了。 对于这些delete表达式， 大多数编译器会编译通过， 尽管它们是错误的。**

虽然一个const对象的值不能被改变， 但它本身是可以被销毁的。 如同任何其他动态对象一样， 想要释放一个const动态对象， 只要delete指向它的指针即可：

```c++
const int *pci = new const int(1024);
delete pci;				//正确：释放一个const对象
```



#### 动态对象的生存期直到被释放时为止

如12.1.1节（第402页） 所述， 由shared_ptr管理的内存在最后一个shared_ptr销毁时会被自动释放。 但对于通过内置指针类型来管理的内存， 就不是这样了。 对于一个由内置指针管理的动态对象， 直到被显式释放之前它都是存在的。

**返回指向动态内存的指针（而不是智能指针） 的函数给其调用者增加了一个额外负担——调用者必须记得释放内存：**

```c++
//factory返回一个指针，指向一个动态分配的对象
Foo* factory(T arg)
{
    //视情况处理arg
    return new Foo(arg);		//调用者负责释放此内存
}
```

类似我们之前定义的factory函数（参见12.1.1节， 第403页） ， 这个版本的factory分配一个对象， 但并不delete它。 factory的调用者负责在不需要此对象时释放它。 不幸的是， 调用者经常忘记释放对象：

```c++
void use_factory(T arg)
{
    Foo *p = factory(arg);
    //使用p但不delete它
}	//p离开了它的作用域，但它所指向的内存没有被释放！
```

此处， use_factory函数调用factory， 后者分配一个类型为Foo的新对象。 当use_factory返回时， 局部变量p被销毁。 此变量是一个内置指针，而不是一个智能指针。

与类类型不同， 内置类型的对象被销毁时什么也不会发生。 特别是， 当一个指针离开其作用域时， 它所指向的对象什么也不会发生。 如果这个指针指向的是动态内存， 那么内存将不会被自动释放。

在本例中， p是指向factory分配的内存的唯一指针。 一旦use_factory返回， 程序就没有办法释放这块内存了。 根据整个程序的逻辑， 修正这个错误的正确方法是在use_factory中记得释放内存：

```c++
void use_factory(T arg)
{
    Foo *p = factory(arg);
    //使用p
    delete p;				//现在记得释放内存，我们已经不需要它了
}
```

还有一种可能， 我们的系统中的其他代码要使用use_factory

所分配的对象， 我们就应该修改此函数， 让它返回一个指针， 指向它分配的内存：

```c++
Foo* use_factory(T arg)
{
    Foo *p = factory(arg);
    //使用p
    return p;				//调用者必须释放内存
}
```



<u>小心： 动态内存的管理非常容易出错</u>

<u>使用new和delete</u>

<u>管理动态内存存在三个常见问题：</u>

<u>**1.忘记delete内存。 忘记释放动态内存会导致人们常说的“内存泄漏”问题， 因为这种内存永远不可能被归还给自由空间了。** 查找内存泄露错误是非常困难的， 因为通常应用程序运行很长时间后， 真正耗尽内存时， 才能检测到这种错误。</u>

**<u>2.使用已经释放掉的对象。 通过在释放内存后将指针置为空， 有时可以检测出这种错误。</u>**

<u>**3.同一块内存释放两次。 当有两个指针指向相同的动态分配对象时， 可能发生这种错误。** 如果对其中一个指针进行了delete操作， 对象的内存就被归还给自由空间了。如果我们随后又delete第二个指针， 自由空间就可能被破坏。 相对于查找和修正这些错误来说， 制造出这些错误要简单得多。</u>

<u>坚持只使用智能指针， 就可以避免所有这些问题。 对于一块内存， 只有在没有任何智能指针指向它的情况下， 智能指针才会自动释放它</u>



#### delete之后重置指针值……

当我们delete一个指针后， 指针值就变为无效了。 虽然指针已经无效， 但**在很多机器上指针仍然保存着（已经释放了的） 动态内存的地址。 在delete之后， 指针就变成了人们所说的空悬指针（dangling pointer） ， 即， 指向一块曾经保存数据对象但现在已经无效的内存的指针。**

未初始化指针（ 参见2.3.2节， 第49页） 的所有缺点空悬指针也都有。 **有一种方法可以避免空悬指针的问题：在指针即将要离开其作用域之前释放掉它所关联的内存。** 这样，在指针关联的内存被释放掉之后，就没有机会继续使用指针了。 **如果我们需要保留指针， 可以在delete之后将nullptr赋予指针， 这样就清楚地指出指针不指向任何对象。**



#### ……这只是提供了有限的保护

动态内存的一个基本问题是可能有多个指针指向相同的内存。 在delete内存之后重置指针的方法只对这个指针有效， 对其他任何仍指向（ 已释放的） 内存的指针是没有作用的。 例如：

```c++
int *p(new int(42));			 //p指向动态内存
auto q = p;						//p和q指向相同的内存
delete p;						//p和q均变为无效
p = nullptr;					//指出p不再绑定到任何对象
```

本例中p和q指向相同的动态分配的对象。 **我们delete此内存， 然后将p置为nullptr， 指出它不再指向任何对象。 但是， 重置p对q没有任何作用， 在我们释放p所指向的（ 同时也是q所指向的！ ） 内存时， q也变为无效了。** 在实际系统中， 查找指向相同内存的所有指针是异常困难的。



### 12.1.3 shared_ptr和new结合使用

![表12.3 定义和改变shared_ptr的其他方法](C:\Users\Grey\Desktop\C++ Primer\图片\表12.3 定义和改变shared_ptr的其他方法.png)

如前所述， **如果我们不初始化一个智能指针， 它就会被初始化为一个空指针。** 如表12.3所示， 我们还可以用new返回的指针来初始化智能指针：

```c++
shared_ptr<double> pl;					//shared_ptr可以指向一个double
shared_ptr<int> p2(new int(42));		 //p2指向一个值为42的int
```

**接受指针参数的智能指针构造函数是explicit的（参见7.5.4节， 第265页） 。 因此， 我们不能将一个内置指针隐式转换为一个智能指针，必须使用直接初始化形式（参见3.2.1节， 第76页） 来初始化一个智能指针：**

```c++
shared_ptr<int> p1 = new int(1024);		//错误：必须使用直接初始化形式
shared_ptr<int> p2(new int(1024));		//正确：使用了直接初始化形式
```

p1的初始化隐式地要求编译器用一个new返回的int * 来创建一个shared_ptr。 由于我们不能进行内置指针到智能指针间的隐式转换，因此这条初始化语句是错误的。 出于相同的原因， **一个返回shared_ptr的函数不能在其返回语句中隐式转换一个普通指针：**

```c++
shared_ptr<int> clone(int p) {
	return new int(p);					//错误：隐式转换为shared_ptr<int>
}
```

我们必须将shared_ptr显式绑定到一个想要返回的指针上：

```c++
shared_ptr<int> clone(int p) {
    //正确：显式地用int*创建shared ptr<int>
    return shared_ptr<int>(new int (p));
}
```

默认情况下， 一个用来初始化智能指针的普通指针必须指向动态内存， 因为智能指针默认使用delete释放它所关联的对象。 **我们可以将智能指针绑定到一个指向其他类型的资源的指针上，但是为了这样做，必须提供自己的操作来替代delete。** 我们将在12.1.4节（第415页） 介绍如何定义自己的释放操作。

> **智能指针与new结合使用**
>
> ​	**定义变量时不能用赋值初始化，只能使用直接初始化**
>
> ​	**函数返回时返回值需要显式声明是智能指针，函数调用时也一样**



#### 不要混合使用普通指针和智能指针……

shared_ptr可以协调对象的析构， 但这仅限于其自身的拷贝（也是shared_ptr） 之间。 这也是为什么我们推荐使用make_shared而不是new的原因。 这样， 我们就能在分配对象的同时就将shared_ptr与之绑定， 从而避免了无意中将同一块内存绑定到多个独立创建的shared_ptr上。

考虑下面对shared_ptr进行操作的函数：

```c++
//在函数被调用时ptr被创建并初始化
void process(shared_ptr<int> ptr) {
	//使用ptr
}	//ptr离开作用域，被销毁
```

process的参数是传值方式传递的， 因此实参会被拷贝到ptr中。 拷贝一个shared_ptr会递增其引用计数 ， 因此， 在process运行过程中， 引用计数值至少为2。 当process结束时， ptr的引用计数会递减， 但不会变为0。 因此， 当局部变量ptr被销毁时， ptr指向的内存不会被释放。

使用此函数的正确方法是传递给它一个shared_ptr：

```c++
shared_ptr<int> p(new int(42));			//引用计数为1
process(p);					//拷贝p会递增它的引用计数；在process中引用计数值为2
int i = *p;					//正确：引用计数值为1
```

**虽然不能传递给process一个内置指针，但可以传递给它一个（临时的） shared_ptr**，这个shared_ptr**是用一个内置指针显式构造的。 但是，这样做很可能会导致错误：**

```c++
int *x(new int (1024));				//危险：×是一个普通指针，不是一个智能指针
process(x);						   //错误：不能将int*转换为一个shared_ptr<int>
process(shared_ptr<int>(x));		//合法的，但内存会被释放！
int j = *x;						   //未定义的：x是一个空悬指针！
```

> 将一个普通指针传递给函数形参shared_ptr。在函数内生成了一个智能指针，当这个函数调用结束时，引用计数减少为0，内存被释放，这个普通指针就不能使用了。 

在上面的调用中， **我们将一个临时shared_ptr传递给process。 当这个调用所在的表达式结束时， 这个临时对象就被销毁了。 销毁这个临时变量会递减引用计数， 此时引用计数就变为0了。 因此， 当临时对象被销毁时， 它所指向的内存会被释放。**

但x继续指向（已经释放的）内存， 从而变成一个空悬指针。 如果试图使用x的值， 其行为是未定义的

**当将一个shared_ptr绑定到一个普通指针时， 我们就将内存的管理责任交给了这个shared_ptr。 一旦这样做了， 我们就不应该再使用内置指针来访问shared_ptr所指向的内存了。**



#### ……也不要使用get初始化另一个智能指针或为智能指针赋值

**智能指针类型定义了一个名为get的函数（参见表12.1） ， 它返回一个内置指针， 指向智能指针管理的对象。此函数是为了这样一种情况而设计的： 我们需要向不能使用智能指针的代码传递一个内置指针。使用get返回的指针的代码不能delete此指针。**

> 用delete释放 get() 获取的地址。智能指针的引用计数仍为1，但其指向的int对象已经被释放了。智能指针成为类似空悬指针的shared_ptr。

虽然编译器不会给出错误信息， 但**将另一个智能指针也绑定到get返回的指针上是错误的：**

```c++
shared_ptr<int> p(new int(42));		  //引用计数为1
int *q = p.get();				//正确：但使用q时要注意，不要让它管理的指针被释放

{	//新程序块
	//未定义：两个独立的shared_ptr指向相同的内存
	shared ptr<int>(q);
}	//程序块结束，q被销毁，它指向的内存被释放

int foo = *p;				//未定义：p指向的内存已经被释放了
```

> GCC编译时并且正常运行，但Clang编译会报错
>
> **编译器会认为p和q是使用两个地址（虽然它们相等）创建的两个不相干的shared_ptr，而非共享同一个动态对象。这样，两者的引用计数均为1。当代码块执行完毕后，q的引用计数减为0，所管理的内存地址被释放，而此内存就是p所管理的。p成为一个管理空悬指针的shared_ptr。**

在本例中， p和q指向相同的内存。 由于它们是相互独立创建的， 因此各自的引用计数都是1。 当q所在的程序块结束时，q被销毁， 这会导致q指向的内存被释放。 从而p变成一个空悬指针，意味着当我们试图使用p时， 将发生未定义的行为。 而且， 当p被销毁时， 这块内存会被第二次delete。

<u>get用来将指针的访问权限传递给代码， **你只有在确定代码不会delete指针的情况下， 才能使用get。 特别是， 永远不要用get初始化另一个智能指针或者为另一个智能指针赋值。**</u>



#### 其他shared_ptr操作

shared_ptr还定义了其他一些操作， 参见表12.2和表12.3所示。 我们可以用reset来将一个新的指针赋予一个shared_ptr：

```c++
p = new int(1024);					//错误：不能将一个指针赋予shared_ptr
p.reset(new int(1024));				//正确：p指向一个新对象
```

与赋值类似， reset会更新引用计数， 如果需要的话， 会释放p指向的对象。 reset成员经常与unique一起使用， 来控制多个shared_ptr共享的对象。 在改变底层对象之前， 我们检查自己是否是当前对象仅有的用户。 如果不是， 在改变之前要制作一份新的拷贝：

```c++
if (!p.unique())
	p.reset(new string(*p));		//我们不是唯一用户；分配新的拷贝
*p += newVal;					   //现在我们知道自己是唯一的用户，可以改变对象的值
```

> reset()中的参数可以是内置指针，不能是智能指针



### 12.1.4 智能指针和异常

5.6.2节（第175页） 中介绍了使用异常处理的程序能在异常发生后令程序流程继续， 我们注意到， 这种程序需要确保在异常发生后资源能被正确地释放。 一个简单的确保资源被释放的方法是使用智能指针。

如果使用智能指针， 即使程序块过早结束， 智能指针类也能确保在内存不再需要时将其释放：

```c++
void f()
{
    shared_ptr<int> sp(new int(42));		//分配一个新对象
    //这段代码抛出一个异常，且在f中未被捕获
}	//在函数结束时shared_ptr自动释放内存
```

**函数的退出有两种可能， 正常处理结束或者发生了异常， 无论哪种情况， 局部对象都会被销毁。** 在上面的程序中， sp是一个shared_ptr， 因此sp销毁时会检查引用计数。 在此例中， sp是指向这块内存的唯一指针， 因此内存会被释放掉。

与之相对的， 当发生异常时， 我们直接管理的内存是不会自动释放的。 **如果使用内置指针管理内存， 且在new之后在对应的delete之前发生了异常， 则内存不会被释放：**

```c++
void f()
{
    int *ip = new int(42);			//动态分配一个新对象
    //这段代码抛出一个异常，且在f中未被捕获
    delete ip;						//在退出之前释放内存
}
```

如果在new和delete之间发生异常， 且异常未在 f 中被捕获， 则内存就永远不会被释放了。 在函数 f 之外没有指针指向这块内存， 因此就无法释放它了。



#### 智能指针和哑类

包括所有标准库类在内的很多C++类都定义了析构函数（参见12.1.1节， 第402页） ， 负责清理对象使用的资源。 但是，不是所有的类都是这样良好定义的。 特别是那些为C和C++两种语言设计的类，通常都要求用户显式地释放所使用的任何资源。

那些分配了资源， 而又没有定义析构函数来释放这些资源的类， 可能会遇到与使用动态内存相同的错误——程序员非常容易忘记释放资源。 类似的， 如果在资源分配和释放之间发生了异常， 程序也会发生资源泄漏。

与管理动态内存类似， 我们通常可以使用类似的技术来管理不具有良好定义的析构函数的类。 例如， 假定我们正在使用一个C和C++都使用的网络库， 使用这个库的代码可能是这样的：

```c++
struct destination;							//表示我们正在连接什么
struct connection;							//使用连接所需的信息

connection connect(destination *);			//打开连接
void disconnect(connection);				//关闭给定的连接

void f(destination &d /*其他参数*/)
{
    //获得一个连接；记住使用完后要关闭它
    connection c = connect(&d);
    //使用连接
    //如果我们在f退出前忘记调用disconnect,就无法关闭c了
}
```

如果connection有一个析构函数，就可以在 f 结束时由析构函数自动关闭连接。但是connection没有析构函数。 这个问题与我们上一个程序中使用shared_ptr避免内存泄漏几乎是等价的。 使用shared_ptr来保证connection被正确关闭， 已被证明是一种有效的方法。



#### 使用我们自己的释放操作

默认情况下， shared_ptr假定它们指向的是动态内存。 因此， 当一个shared_ptr被销毁时， 它默认地对它管理的指针进行delete操作。 为了用shared_ptr来管理一个connection， 我们必须首先定义一个函数来代替delete。 这个删除器（deleter） 函数必须能够完成对shared_ptr中保存的指针进行释放的操作。 在本例中， 我们的删除器必须接受单个类型为connection * 的参数：

```c++
void end_connection(connection *p){ disconnect(*p); }
```

当我们创建一个shared_ptr时， 可以传递一个（可选的） 指向删除器函数的参数（参见6.7节， 第221页） ：

```c++
void f(destination &d /*其他参数*/)
{
    connection c = connect(&d);
    shared_ptr<connection> p(&c, end_connection);
    //使用连接
    //当f退出时（即使是由于异常而退出），connection会被正确关闭
}
```

当p被销毁时， 它不会对自己保存的指针执行delete， 而是调用end_connection。 接下来， end_connection会调用disconnect， 从而确保连接被关闭。 如果f正常退出， 那么p的销毁会作为结束处理的一部分。 如果发生了异常， p同样会被销毁， 从而连接被关闭。



<u>注意： 智能指针陷阱</u>

<u>智能指针可以提供对动态分配的内存安全而又方便的管理， 但这建立在正确使用的前提下。 为了正确使用智能指针， 我们必须坚持一些基本规范：</u>

**<u>· 不使用相同的内置指针值初始化（或reset） 多个智能指针。</u>**

**<u>· 不delete get() 返回的指针。</u>**

**<u>· 不使用get()  初始化或reset另一个智能指针。</u>**

**<u>· 如果你使用get() 返回的指针， 记住当最后一个对应的智能指针销毁后， 你的指针就变为无效了。</u>**

**<u>· 如果你使用智能指针管理的资源不是new分配的内存， 记住传递给它一个删除器（参见12.1.4节， 第415页和12.1.5节， 第419页） 。</u>**



### 12.1.5 unique_ptr

![表12.4 unique ptr操作（另参见表12.1，第401页）](C:\Users\Grey\Desktop\C++ Primer\图片\表12.4 unique ptr操作（另参见表12.1，第401页）.png)

> release() 返回内置指针，reset() 参数也为内置指针

一个unique_ptr“拥有”它所指向的对象。 与shared_ptr不同， **某个时刻只能有一个unique_ptr指向一个给定对象。 当unique_ptr被销毁时， 它所指向的对象也被销毁。** 表12.4列出了unique_ptr特有的操作。 与shared_ptr相同的操作列在表12.1（第401页） 中。

与shared_ptr不同， 没有类似make_shared的标准库函数返回一个unique_ptr 。 当我们定义一个unique_ptr时， 需要将其绑定到一个new返回的指针上。 类似shared_ptr， **初始化unique_ptr必须采用直接初始化形式：**

> **C++14 已经有了 make_unique<T>()**

```c++
unique_ptr<double> p1;				//可以指向一个double的unique_ptr
unique_ptr<int> p2(new int(42));	 //p2指向一个值为42的int
```

由于一个unique_ptr拥有它指向的对象， 因此**unique_ptr不支持普通的拷贝或赋值操作**：

```c++
unique_ptr<string> p1(new string("Stegosaurus"));
unique_ptr<string> p2(p1);		  //错误：unique_ptr不支持拷贝
unique ptr<string> p3;
p3 = p2;						//错误：unique_ptr不支持赋值
```

> **只能使用直接初始化，拷贝初始化（ = ）是不对的，除了make_unique**

虽然我们不能拷贝或赋值unique_ptr， 但**可以通过调用release或reset将指针的所有权从一个（非const） unique_ptr转移给另一个unique：**

```c++
//将所有权从p1(指向string Stegosaurus)转移给p2
unique_ptr<string> p2(p1.release());		//release将p1置为空

unique_ptr<string> p3(new string("Trex"));
//将所有权从p3转移给p2
p2.reset(p3.release());						//reset释放了p2原来指向的内存
```

**release成员返回unique_ptr当前保存的指针并将其置为空。 因此， p2被初始化为p1原来保存的指针 ， 而p1被置为空。**

**reset成员接受一个可选的指针参数， 令unique_ptr重新指向给定的指针。** 如果unique_ptr不为空， 它原来指向的对象被释放。 因此， 对p2调用reset释放了用"Stegosaurus"初始化的string所使用的内存， 将p3对指针的所有权转移给p2， 并将p3置为空。

调用release会切断unique_ptr和它原来管理的对象间的联系。 **release返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。** 在本例中， 管理内存的责任简单地从一个智能指针转移给另一个。但是， **如果我们不用另一个智能指针来保存release返回的指针， 我们的程序就要负责资源的释放：**

```c++
p2.release();				//错误：p2不会释放内存，而且我们丢失了指针
auto p = p2.release();		 //正确，但我们必须记得delete(p)
```



#### 传递unique_ptr参数和返回unique_ptr

不能拷贝unique_ptr的规则有一个例外： **我们可以拷贝或赋值一个将要被销毁的unique_ptr。 最常见的例子是从函数返回一个unique_ptr：**

```c++
unique_ptr<int> clone(int p){
    //正确：从int*创建一个unique_ptr<int>
    return unique_ptr<int>(new int(p));
}
```

**还可以返回一个局部对象的拷贝：**

```c++
unique_ptr<int> clone(int p){
    unique_ptr<int> ret(new int(p));
    //...
    return ret;
}
```

对于两段代码， 编译器都知道要返回的对象将要被销毁。 在此情况下，编译器执行一种特殊的“拷贝”， 我们将在13.6.2节（第473页） 中介绍它。



<u>向后兼容： auto_ptr</u>

<u>标准库的较早版本包含了一个名为auto_ptr的类， 它具有unique_ptr的部分特性，但不是全部。 特别是， 我们不能在容器中保存auto_ptr， 也不能从函数中返回auto_ptr。</u>

<u>虽然auto_ptr仍是标准库的一部分， 但编写程序时应该使用unique_ptr。</u>



#### 向unique_ptr传递删除器

类似shared_ptr， unique_ptr默认情况下用delete释放它指向的对象。与shared_ptr一样， **我们可以重载一个unique_ptr中默认的删除器**（参见12.1.4节， 第415页） 。 但是， unique_ptr管理删除器的方式与shared_ptr不同， 其原因我们将在16.1.6节（第599页） 中介绍。

重载一个unique_ptr中的删除器会影响到unique_ptr类型以及如何构造（或reset） 该类型的对象。 与重载关联容器的比较操作（参见11.2.2节， 第378页） 类似， **我们必须在尖括号中unique_ptr指向类型之后提供删除器类型。 在创建或reset一个这种unique_ptr类型的对象时， 必须提供一个指定类型的可调用对象**（删除器） ：

```c++
//p指向一个类型为objT的对象，并使用一个类型为de1T的对象释放objT对象
//它会调用一个名为fcn的delT类型对象
unique_ptr<objT, delT> p(new objT, fcn);
```

作为一个更具体的例子， 我们将重写连接程序， 用unique_ptr来代替shared_ptr， 如下所示：

```c++
void f(destination &d /*其他需要的参数*/)
{
    connection c = connect(&d);		//打开连接
    //当p被销毁时，连接将会关闭
    unique_ptr<connection, decltype(end_connection)*> p(&c, end_connection);
    //使用连接
    //当f退出时（即使是由于异常而退出），connection会被正确关闭
}
```

在本例中我们使用了decltype（参见2.5.3节， 第62页） 来指明函数指针类型。 由于decltype（end_connection） 返回一个函数类型， 所以我们必须添加一个 * 来指出我们正在使用该类型的一个指针（参见6.7节， 第223页） 。

> ```c++
> shared_ptr的形式是 
>     shared_ptr<connection> p(&c, end_connection);
> 
> unique_ptr的形式是 
>     unique_ptr<connection, decltype(end_connection)*> p(&c, end_connection);
> 
> unique尖括号里多了decltype(end_connection)*
> ```



> 可以两个unique_ptr指向相同的内存地址，如下两个例子
>
> ```c++
> unique_ptr< int > p1(new int(2));
> unique_ptr< int > p2(p1.get());
> ```
>
> 	auto p = new int(2);
> 	unique_ptr< int > p1(p);
>
> 当其中一个unique_ptr被销毁（或调用reset释放对象）时，该内存被释放，另一个unique_ptr变为空悬指针。



### 12.1.6 weak_ptr

![表12.5 weak_ptr](C:\Users\Grey\Desktop\C++ Primer\图片\表12.5 weak_ptr.png)

> weak_ptr 可以用来解决循环依赖问题

**weak_ptr（见表12.5） 是一种不控制所指向对象生存期的智能指针， 它指向由一个shared_ptr管理的对象。将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的引用计数。 一旦最后一个指向对象的shared_ptr被销毁， 对象就会被释放。 即使有weak_ptr指向对象，对象也还是会被释放**，因此，weak_ptr的名字抓住了这种智能指针“弱”共享对象的特点。

当我们创建一个weak_ptr时， 要用一个shared_ptr来初始化它：

```c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);		//wp弱共享p，但p的引用计数未改变
```

本例中wp和p指向相同的对象。 由于是弱共享， 创建wp不会改变p的引用计数； wp指向的对象可能被释放掉。

**由于对象可能不存在，我们不能使用weak_ptr直接访问对象，而必须调用lock。** 此函数检查weak_ptr指向的对象是否仍存在。 如果存在，lock返回一个指向共享对象的shared_ptr。 与任何其他shared_ptr类似 ， 只要此shared_ptr存在， 它所指向的底层对象也就会一直存在。 例如：

```c++
if (shared_ptr<int> np = wp.lock()) {		//如果np不为空则条件成立
	//在if中，np与p共享对象
}
```

在这段代码中， 只有当lock调用返回true时我们才会进入if语句体。在if中， 使用np访问共享对象是安全的。



#### 核查指针类

作为weak_ptr用途的一个展示， 我们将为StrBlob类定义一个伴随指针类。 我们的指针类将命名为StrBlobPtr，会保存一个weak_ptr，指向StrBlob的data成员，这是初始化时提供给它的。通过使用weak_ptr ， 不会影响一个给定的StrBlob所指向的vector的生存期。 但是， 可以阻止用户访问一个不再存在的vector的企图。

StrBlobPtr会有两个数据成员：wptr 要么为空， 要么指向一个StrBlob中的vector；curr 保存当前对象所表示的元素的下标。 类似它的伴随类StrBlob， 我们的指针类也有一个check成员来检查解引用StrBlobPtr是否安全：

```c++
//对于访问一个不存在元素的尝试，StrBlobPtr抛出一个异常
class StrBlobPtr {
public:
    StrBlobPtr() :curr(0){}
    StrBlobPtr(StrBlob &a, size_t sz = 0): wptr(a.data), curr(sz){}
    std::string& deref() const;
	StrBlobPtr& incr();					//前缀递增
    
private:
    //若检查成功，check返回一个指向ector的shared_ptr
    std::shared_ptr<std::vector<std::string>>
    					check(std::size_t, const std::string&) const;
    //保存一个weak_ptr, 意味着底层vector可能会被销毁
    std::weak_ptr<std::vector<std::string>> wptr;
    std::size_t curr;					//在数组中的当前位置
};
```

默认构造函数生成一个空的StrBlobPtr。 其构造函数初始化列表（参见7.1.4节， 第237页） 将curr显式初始化为0，并将wptr隐式初始化为一个空weak_ptr。 第二个构造函数接受一个StrBlob引用和一个可选的索引值。 此构造函数初始化wptr，令其指向给定StrBlob对象的shared_ptr中的vector， 并将curr初始化为sz的值。 我们使用了默认参数（参见6.5.1节， 第211页），表示默认情况下将curr初始化为第一个元素的下标。 我们将会看到， StrBlob的end成员将会用到参数sz。

值得注意的是， **我们不能将StrBlobPtr绑定到一个const StrBlob对象。 这个限制是由于构造函数接受一个非const StrBlob对象的引用而导致的。**

StrBlobPtr的check成员与StrBlob中的同名成员不同， 它还要检查指针指向的vector是否还存在：

```c++
std::shared_ptr<std::vector<std::string>>
			StrBlobptr::check(std::size_t i, const std::string &msg) const
{
    auto ret = wptr.lock();				//vector还存在吗？
    if (!ret)
    	throw std::runtime_error("unbound StrBlobptr");
    if (i >= ret->size())
   		throw std::out_of_range(msg);
    return ret;							//否则，返回指向vector的shared_ptr
}
```

由于一个weak_ptr不参与其对应的shared_ptr的引用计数， StrBlobPtr指向的vector可能已经被释放了 。 如果vector已销毁， lock将返回一个空指针。 在本例中， 任何vector的引用都会失败， 于是抛出一个异常。 否则， check会检查给定索引， 如果索引值合法， check返回从lock获得的shared_ptr。



#### 指针操作

我们将在第14章学习如何定义自己的运算符。 现在， 我们将定义名为deref和incr的函数， 分别用来解引用和递增StrBlobPtr。

deref成员调用check， 检查使用vector是否安全以及curr是否在合法范围内：

```c++
std::string& StrBlobPtr::deref() const
{
    auto p = check(curr, "dereference past end");
    return (*p)[curr];					//(*p)是对象所指向的vector
}
```

如果check成功， p就是一个shared_ptr， 指向StrBlobPtr所指向的vector。 表达式 (*p)[curr] 解引用shared_ptr来获得vector， 然后使用下标运算符提取并返回curr位置上的元素。

```c++
//incr成员也调用check:
//前缀递增：返回递增后的对象的引用
StrBlobPtr& StrBlobPtr::incr()
{
    //如果cu上已经指向容器的尾后位置，就不能递增它
    check(curr, "increment past end of StrBlobptr");
    ++curr；						//推进当前位置
    return *this;
}
```

当然， 为了访问data成员， 我们的指针类必须声明为StrBlob的friend（参见7.3.4节， 第250页） 。 我们还要为StrBlob类定义begin和end操作， 返回一个指向它自身的StrBlobPtr：

```c++
//对于StrB1ob中的友元声明来说，此前置声明是必要的
class StrBlobPtr;
class strBlob {
    friend class StrBlobPtr;
    //其他成员与12.1.1节（第405页)中声明相同
    //返回指向首元素和尾后元素的StrBlobPtr
    StrBlobPtr begin(){ return StrBlobPtr(*this); }
    StrBlobPtr end(){auto ret StrBlobPtr(*this,data->size()); return ret;}
};
```



## 12.2 动态数组

new和delete运算符一次分配/释放一个对象， 但某些应用需要一次为很多对象分配内存的功能。 例如 ， vector和string都是在连续内存中保存它们的元素， 因此， 当容器需要重新分配内存时（参见9.4节， 第317页） ， 必须一次性为很多元素分配内存。

为了支持这种需求， C++语言和标准库提供了两种一次分配一个对象数组的方法。 **C++语言定义了另一种new表达式语法， 可以分配并初始化一个对象数组。 标准库中包含一个名为allocator的类， 允许我们将分配和初始化分离。 使用allocator通常会提供更好的性能和更灵活的内存管理能力**， 原因我们将在12.2.2节（第427页） 中解释。

很多（可能是大多数） 应用都没有直接访问动态数组的需求。 当一个应用需要可变数量的对象时， 我们在StrBlob中所采用的方法几乎总是更简单、 更快速并且更安全的——即， 使用vector（或其他标准库容器） 。 如我们将在13.6节（第470页） 中看到的， 使用标准库容器的优势在新标准下更为显著。 在支持新标准的标准库中， 容器操作比之前的版本要快速得多。

**<u>大多数应用应该使用标准库容器而不是动态分配的数组。 使用容器更为简单、 更不容易出现内存管理错误并且可能有更好的性能。</u>**

如前所述， 使用容器的类可以使用默认版本的拷贝、 赋值和析构操作（参见7.1.5节， 第239页） 。 分配动态数组的类则必须定义自己版本的操作， 在拷贝、 复制以及销毁对象时管理所关联的内存。



### 12.2.1 new和数组

为了让new分配一个对象数组， 我们要在类型名之后跟一对方括号， 在其中指明要分配的对象的数目 。 在下例中， new分配要求数量的对象并（假定分配成功后） 返回指向第一个对象的指针：

```c++
//调用get_size确定分配多少个int
int *pia = new int[get_size()];			//pia指向第一个int
```

**方括号中的大小必须是整型， 但不必是常量。**

也可以用一个表示数组类型的类型别名（参见2.5.1节， 第60页） 来分配一个数组， 这样， new表达式中就不需要方括号了：

```c++
typedef int arrT[42];			//arrT表示42个int的数组类型
int *p = new arrT;				//分配一个42个int的数组；p指向第一个int
```

在本例中， new分配一个int数组， 并返回指向第一个int的指针。 即使这段代码中没有方括号， 编译器执行这个表达式时还是会用new[]。即， 编译器执行如下形式：

```c++
int *p = new int[42]
```



#### 分配一个数组会得到一个元素类型的指针

虽然我们通常称new T[]分配的内存为“动态数组”， 但这种叫法某种程度上有些误导。 **当用new分配一个数组时， 我们并未得到一个数组类型的对象， 而是得到一个数组元素类型的指针。 即使我们使用类型别名定义了一个数组类型， new也不会分配一个数组类型的对象。** 在上例中， 我们正在分配一个数组的事实甚至都是不可见的——连[num]都没有。 **new返回的是一个元素类型的指针。**

由于分配的内存并不是一个数组类型， **因此不能对动态数组调用begin或end**（参见3.5.3节， 第106页） 。 这些函数使用数组维度（回忆一下， 维度是数组类型的一部分） 来返回指向首元素和尾后元素的指针。出于相同的原因， **也不能用范围for语句来处理（所谓的） 动态数组中的元素。**

**<u>要记住我们所说的动态数组并不是数组类型， 这是很重要的。</u>**



#### 初始化动态分配对象的数组

默认情况下， **new分配的对象， 不管是单个分配的还是数组中的，都是默认初始化的。** 可以对数组中的元素进行值初始化（参见3.3.1节，第88页） ， 方法是在大小之后跟一对空括号。

```c++
int *pia = new int[10];					//10个未初始化的int
int *pia2 = new int[10]()				//10个值初始化为0的1nt
string *psa = new string[10];			//10个空string
string *psa2 = new string[10]()			//10 string
```

在新标准中， 我们还可以提供一个元素初始化器的花括号列表：

```c++
//10个1nt分别用列表中对应的初始化器初始化
int *pia3 = new int[10]{0,1,2,3,4,5,6,7,8,9}:

//10个string,前4个用给定的初始化器初始化，剩余的进行值初始化
string *psa3 = new string[10]{"a","an","the",string(3,x')};
```

与内置数组对象的列表初始化（参见3.5.1节， 第102页） 一样， 初始化器会用来初始化动态数组中开始部分的元素。 如果初始化器数目小于元素数目， 剩余元素将进行值初始化。 如果初始化器数目大于元素数目， 则new表达式失败， 不会分配任何内存。 在本例中， new会抛出一个类型为bad_array_new_length的异常。 类似bad_alloc， 此类型定义在头文件new中。

虽然我们用空括号对数组中元素进行值初始化， 但不能在括号中给出初始化器， 这意味着不能用auto分配数组（参见12.1.2节， 第407页） 。



#### 动态分配一个空数组是合法的

可以用任意表达式来确定要分配的对象的数目：

```c++
size_t n = get_size();				//get_size返回需要的元素的数目
int *p = new int[n];				//分配数组保存元素
for (int *q = p; q != p + n; ++q)			
    /*处理数组*/；
```

这产生了一个有意思的问题： 如果get_size返回0， 会发生什么？ 答案是代码仍能正常工作。 **虽然我们不能创建一个大小为0的静态数组对象， 但当n等于0时， 调用new[n]是合法的：**

```c++
char arr[0];						//错误：不能定义长度为0的数组
char *cp = new char[0];				//正确：但cp不能解引用
```

当我们用new分配一个大小为0的数组时， new返回一个合法的非空指针。 此指针保证与new返回的其他任何指针都不相同。 **对于零长度的数组来说， 此指针就像尾后指针一样**（参见3.5.3节， 第106页） ， 我们可以像使用尾后迭代器一样使用这个指针。 可以用此指针进行比较操作， 就像上面循环代码中那样。 可以向此指针加上（或从此指针减去） 0， 也可以从此指针减去自身从而得到0。 但此指针不能解引用——毕竟它不指向任何元素。

在我们假想的循环中， 若get_size返回0， 则n也是0， new会分配0个对象。 for循环中的条件会失败（p等于q+n， 因为n为0） 。 因此， 循环体不会被执行。



#### 释放动态数组

**为了释放动态数组， 我们使用一种特殊形式的delete——在指针前加上一个空方括号对：**

```c++
delete p;				//p必须指向一个动态分配的对象或为空
delete []pa;			//pa必须指向一个动态分配的数组或为空
```

第二条语句销毁pa指向的数组中的元素， 并释放对应的内存。 **数组中的元素按逆序销毁**， 即， 最后一个元素首先被销毁， 然后是倒数第二个， 依此类推。

**当我们释放一个指向数组的指针时， 空方括号对是必需的**： 它指示编译器此指针指向一个对象数组的第一个元素。 如果我们在delete一个指向数组的指针时忽略了方括号（或者在delete一个指向单一对象的指针时使用了方括号） ， 其行为是未定义的。

回忆一下， 当我们使用一个类型别名来定义一个数组类型时， 在new表达式中不使用[]。 即使是这样 ， 在释放一个数组指针时也必须使用方括号：

```c++
typedef int arrT[42];			//arrT是42个int的数组的类型别名
int *p = new arrT;				//分配一个42个int的数组；p指向第一个元素
delete []p;						//方括号是必需的，因为我们当初分配的是一个数组
```

不管外表如何， p指向一个对象数组的首元素， 而不是一个类型为arrT的单一对象。 因此， 在释放p时我们必须使用[]。



#### 智能指针和动态数组

![表12.6 指向数组的unique_ptr](C:\Users\Grey\Desktop\C++ Primer\图片\表12.6 指向数组的unique_ptr.png)

标准库提供了一个可以管理new分配的数组的unique_ptr版本。 为了用一个unique_ptr管理动态数组， 我们必须在对象类型后面跟一对空方括号：

```c++
//up指向一个包含10个未初始化int的数组
unique_ptr<int[]> up(new int[10]);
up.release();				//自动用delete[]销毁其指针
```

类型说明符中的方括号（ <int[ ]> ）指出up指向一个int数组而不是一个int。 由于up指向一个数组， 当up销毁它管理的指针时， 会自动使用delete[ ]。

指向数组的unique_ptr提供的操作与我们在12.1.5节（第417页） 中使用的那些操作有一些不同， 我们在表12.6中描述了这些操作。 **当一个unique_ptr指向一个数组时， 我们不能使用点和箭头成员运算符。** 毕竟unique_ptr指向的是一个数组而不是单个对象， 因此这些运算符是无意义的。 另一方面， 当一个**unique_ptr指向一个数组时， 我们可以使用下标运算符来访问数组中的元素**：

```c++
for (size_t i 0; i != 10; ++i)
	up[i] = i;				//为每个元素赋予一个新值
```

**与unique_ptr不同， shared_ptr不直接支持管理动态数组。 如果希望使用shared_ptr管理一个动态数组 ， 必须提供自己定义的删除器：**

```c++
//为了使用shared ptr,必须提供一个删除器
shared_ptr<int> sp(new int[10], [](int *p) {delete []p; });
sp.reset();		//使用我们提供的lambda释放数组，它使用delete[]
```

本例中我们传递给shared_ptr一个lambda（参见10.3.2节， 第346页）作为删除器， 它使用delete[ ]释放数组。

如果未提供删除器， 这段代码将是未定义的。 默认情况下， shared_ptr使用delete销毁它指向的对象 。 如果此对象是一个动态数组，对其使用delete所产生的问题与释放一个动态数组指针时忘记[ ]产生的问题一样（参见12.2.1节， 第425页） 。

**shared_ptr不直接支持动态数组管理这一特性会影响我们如何访问数组中的元素：**

```c++
//shared ptr未定义下标运算符，并且不支持指针的算术运算
for (size_t i = 0;i != 10; ++i)
	*(sp.get() + i) = i;		//使用get获取一个内置指针
```

shared_ptr未定义下标运算符， 而且智能指针类型不支持指针算术运算。 因此， 为了访问数组中的元素， 必须用get获取一个内置指针，然后用它来访问数组元素。



### 12.2.2 allocator类

new有一些灵活性上的局限， 其中一方面表现在它将内存分配和对象构造组合在了一起。 类似的， delete将对象析构和内存释放组合在了一起。 我们分配单个对象时， 通常希望将内存分配和对象初始化组合在一起。 因为在这种情况下， 我们几乎肯定知道对象应有什么值。

当分配一大块内存时， 我们通常计划在这块内存上按需构造对象。在此情况下， 我们希望将内存分配和对象构造分离。 这意味着我们可以分配大块内存， 但只在真正需要时才真正执行对象创建操作（同时付出一定开销） 。

一般情况下， 将内存分配和对象构造组合在一起可能会导致不必要的浪费。 例如：

```c++
string* const p = new string[n];		//构造n个空string
string s;
string *q = p;							//q指向第一个string
while (cin >> s && q != p + n)
	*q++ = s;							//赋予*q一个新值
const size_t size = q - pi				//记住我们读取了多少个string
//使用数组
delete []p;							//p指向一个数组；记得用delete[]来释放
```

new表达式分配并初始化了n个string。 但是， 我们可能不需要n个string， 少量string可能就足够了。 这样， 我们就可能创建了一些永远也用不到的对象。 而且， 对于那些确实要使用的对象， 我们也在初始化之后立即赋予了它们新值。 每个使用到的元素都被赋值了两次： 第一次是在默认初始化时， 随后是在赋值时。

更重要的是， 那些**没有默认构造函数的类就不能动态分配数组了。**



#### allocator类

![表12.7 标准库allocator类及其算法](C:\Users\Grey\Desktop\C++ Primer\图片\表12.7 标准库allocator类及其算法.png)

**标准库allocator类定义在头文件memory中， 它帮助我们将内存分配和对象构造分离开来。** 它提供一种类型感知的内存分配方法， 它分配的内存是原始的、 未构造的。 表12.7概述了allocator支持的操作。 在本节中， 我们将介绍这些allocator操作。 在13.5节（第464页），我们将看到如何使用这个类的典型例子。

类似vector， allocator是一个模板（参见3.3节， 第86页） 。 为了定义一个allocator对象， 我们必须指明这个allocator可以分配的对象类型。当一个allocator对象分配内存时， 它会根据给定的对象类型来确定恰当的内存大小和对齐位置：

```c++
allocator<string> alloc;				//可以分配string的a1 locator对象
auto const p = alloc.allocate(n);		//分配n个未初始化的string
```

这个allocate调用为n个string分配了内存。



#### allocator分配未构造的内存

**allocator分配的内存是未构造的（unconstructed）。 我们按需在此内存中构造对象。 在新标准库中 ， construct成员函数接受一个指针和零个或多个额外参数， 在给定位置构造一个元素。 额外参数用来初始化构造的对象。** 类似make_shared的参数（参见12.1.1节， 第401页） ， 这些额外参数必须是与构造的对象的类型相匹配的合法的初始化器：

```c++
auto q = p;								//q指向最后构造的元素之后的位置
alloc.construct(q++);					//*q为空字符串
alloc.construct(q++, 10, 'c');			//*q为cccccccccc
alloc.construct(q++, "hi");				//*q为hi!
```

在早期版本的标准库中， construct只接受两个参数： 指向创建对象位置的指针和一个元素类型的值。 因此， 我们只能将一个元素拷贝到未构造空间中， 而不能用元素类型的任何其他构造函数来构造一个元素。

**还未构造对象的情况下就使用原始内存是错误的：**

```c++
cout << *p << endl;				//正确：使用string的输出运算符
cout << *q << endl;				//灾难：q指向未构造的内存！
```

<u>为了使用allocate返回的内存， 我们必须用construct构造对象。 使用未构造的内存， 其行为是未定义的。</u>

**当我们用完对象后， 必须对每个构造的元素调用destroy来销毁它们。 函数destroy接受一个指针， 对指向的对象执行析构函数**（参见12.1.1节， 第402页） ：

```c++
while( q != p)
	alloc.destroy(--q);			//释放我们真正构造的string
```

在循环开始处， q指向最后构造的元素之后的位置。 我们在调用destroy之前对q进行了递减操作。 因此， 第一次调用destroy时， q指向最后一个构造的元素。 最后一步循环中我们destroy了第一个构造的元素，随后q将与p相等， 循环结束。

**一旦元素被销毁后， 就可以重新使用这部分内存来保存其他string， 也可以将其归还给系统。 释放内存通过调用deallocate来完成：**

```c++
alloc.deallocate(p, n);
```

**我们传递给deallocate的指针不能为空，它必须指向由allocate分配的内存。 而且， 传递给deallocate的大小参数必须与调用allocated分配内存时提供的大小参数具有一样的值。**



#### 拷贝和填充未初始化内存的算法

![表12.8 allocator算法](C:\Users\Grey\Desktop\C++ Primer\图片\表12.8 allocator算法.png)

**标准库还为allocator类定义了两个伴随算法， 可以在未初始化内存中创建对象。 表12.8描述了这些函数， 它们都定义在头文件memory中。**

作为一个例子， 假定有一个int的vector， 希望将其内容拷贝到动态内存中。 我们将分配一块比vector中元素所占用空间大一倍的动态内存， 然后将原vector中的元素拷贝到前一半空间， 对后一半空间用一个给定值进行填充：

```c++
//分配比Vi中元素所占用空间大一倍的动态内存
auto p = alloc.allocate(vi.size() * 2);

//通过拷贝ⅴi中的元素来构造从p开始的元素
auto q = uninitialized_copy(vi.begin(), vi.end(), p);

//将剩余元素初始化为42
uninitialized_fill_n(q, vi.size(), 42);
```

类似拷贝算法（参见10.2.2节， 第341页） ， **uninitialized_copy接受三个迭代器参数。 前两个表示输入序列， 第三个表示这些元素将要拷贝到的目的空间。** 传递给uninitialized_copy的目的位置迭代器必须指向未构造的内存。 与copy不同， **uninitialized_copy在给定目的位置构造元素。**

类似copy， uninitialized_copy返回（递增后的）目的位置迭代器。因此， 一次uninitialized_copy调用会返回一个指针， 指向最后一个构造的元素之后的位置。 在本例中， 我们将此指针保存在q中， 然后将q传递给uninitialized_fill_n。 此函数类似fill_n（参见10.2.2节， 第340页） ， 接受一个指向目的位置的指针 、 一个计数和一个值。 它会在目的位置指针指向的内存中创建给定数目个对象， 用给定值对它们进行初始化。



## 12.3 使用标准库： 文本查询程序

我们将实现一个简单的文本查询程序，作为标准库相关内容学习的总结。我们的程序允许用户在一个给定文件中查询单词。查询结果是单词在文件中出现的次数及其所在行的列表。如果一个单词在一行中出现多次， 此行只列出一次。 行会按照升序输出——即， 第7行会在第9行之前显示， 依此类推。

例如，我们可能读入一个包含本章内容（指英文版中的文本）的文件， 在其中寻找单词element。 输出结果的前几行应该是这样的：

`element occurs 112 times`

`(line 36) A set element contains only a key;` 

`(line 158) operator creates a new element` 

`(line 160) Regardless of whether the element`

`(line 168) When we fetch an element from a map,we` 

`(line 214) If the element is not found,find returns`

接下来还有大约100行， 都是单词element出现的位置。



### 12.3.1 文本查询程序设计

开始一个程序的设计的一种好方法是列出程序的操作。 了解需要哪些操作会帮助我们分析出需要什么样的数据结构。 从需求入手， 我们的文本查询程序需要完成如下任务：

· 当程序读取输入文件时， 它必须记住单词出现的每一行。 因此，程序需要逐行读取输入文件， 并将每一行分解为独立的单词

· 当程序生成输出时，

— 它必须能提取每个单词所关联的行号

— 行号必须按升序出现且无重复

— 它必须能打印给定行号中的文本。

利用多种标准库设施， 我们可以很漂亮地实现这些要求：

· 我们将使用一个vector<string>来保存整个输入文件的一份拷贝。输入文件中的每行保存为vector中的一个元素。 当需要打印一行时， 可以用行号作为下标来提取行文本。

· 我们使用一个istringstream（参见8.3节， 第287页） 来将每行分解为单词。

· 我们使用一个set来保存每个单词在输入文本中出现的行号。 这保证了每行只出现一次且行号按升序保存。

· 我们使用一个map来将每个单词与它出现的行号set关联起来。 这样我们就可以方便地提取任意单词的set。

我们的解决方案还使用了shared_ptr， 原因稍后进行解释。



#### 数据结构

虽然我们可以用vector、 set和map来直接编写文本查询程序， 但如果定义一个更为抽象的解决方案， 会更为有效。 我们将从定义一个保存输入文件的类开始， 这会令文件查询更为容易。 我们将这个类命名为TextQuery， 它包含一个vector和一个map。 vector用来保存输入文件的文本， map用来关联每个单词和它出现的行号的set。 这个类将会有一个用来读取给定输入文件的构造函数和一个执行查询的操作。

查询操作要完成的任务非常简单： 查找map成员， 检查给定单词是否出现。 设计这个函数的难点是确定应该返回什么内容。 一旦找到了一个单词， 我们需要知道它出现了多少次、 它出现的行号以及每行的文本。

返回所有这些内容的最简单的方法是定义另一个类， 可以命名为QueryResult， 来保存查询结果。 这个类会有一个print函数， 完成结果打印工作。



#### 在类之间共享数据

我们的QueryResult类要表达查询的结果。 这些结果包括与给定单词关联的行号的set和这些行对应的文本。 这些数据都保存在TextQuery类型的对象中。

由于QueryResult所需要的数据都保存在一个TextQuery对象中， 我们就必须确定如何访问它们。 我们可以拷贝行号的set， 但这样做可能很耗时。 而且， 我们当然不希望拷贝vector， 因为这可能会引起整个文件的拷贝， 而目标只不过是为了打印文件的一小部分而已（通常会是这样） 。

通过返回指向TextQuery对象内部的迭代器（或指针） ， 我们可以避免拷贝操作。 但是， 这种方法开启了一个陷阱： 如果TextQuery对象在对应的QueryResult对象之前被销毁， 会发生什么？ 在此情况下， QueryResult就将引用一个不再存在的对象中的数据。对于QueryResult对象和对应的TextQuery对象的生存期应该同步这一观察结果， 其实已经暗示了问题的解决方案。 考虑到这两个类概念上“共享”了数据， 可以使用shared_ptr（参见12.1.1节， 第400页） 来反映数据结构中的这种共享关系。



#### 使用TextQuery类

当我们设计一个类时， 在真正实现成员之前先编写程序使用这个类， 是一种非常有用的方法。 通过这种方法， 可以看到类是否具有我们所需要的操作。 例如， 下面的程序使用了TextQuery和QueryResult类。这个函数接受一个指向要处理的文件的ifstream， 并与用户交互， 打印给定单词的查询结果

```c++
void runQueries(ifstream &infile)
{
    //infile是一个ifstream,指向我们要处理的文件
    TextQuery tq(infile);			//保存文件并建立查询map
    //与用户交互：提示用户输入要查询的单词，完成查询并打印结果
    while (true){
        cout << "enter word to look for,or g to quit: ";
        string s;
        //若遇到文件尾或用户输入了'α'时循环终止
        if (!(cin >> s) || s == "g")	break;
        //指向查询并打印结果
        print(cout, tq.query(s)) << endl
	}
}
```

我们首先用给定的ifstream初始化一个名为tq的TextQuery对象。TextQuery的构造函数读取输入文件， 保存在vector中， 并建立单词到所在行号的map。

while（无限） 循环提示用户输入一个要查询的单词， 并打印出查询结果， 如此往复。 循环条件检测字面常量true（参见2.1.3节， 第37页） ， 因此永远成功。 循环的退出是通过if语句中的break（参见5.5.1节， 第170页） 实现的。 此if语句检查输入是否成功。 如果成功， 它再检查用户是否输入了q。 输入失败或用户输入了q都会使循环终止。 一旦用户输入了要查询的单词， 我们要求tq查找这个单词， 然后调用print打印搜索结果。



### 12.3.2 文本查询程序类的定义

我们以TextQuery类的定义开始。 用户创建此类的对象时会提供一个istream， 用来读取输入文件。 这个类还提供一个query操作， 接受一个string， 返回一个QueryResult表示string出现的那些行。

设计类的数据成员时， 需要考虑与QueryResult对象共享数据的需求。 QueryResult类需要共享保存输入文件的vector和保存单词关联的行号的set。 因此， 这个类应该有两个数据成员： 一个指向动态分配的vector（保存输入文件） 的shared_ptr和一个string到shared_ptr<set>的map。 map将文件中每个单词关联到一个动态分配的set上， 而此set保存了该单词所出现的行号。

为了使代码更易读， 我们还会定义一个类型成员（参见7.3.1节， 第243页） 来引用行号， 即string的vector中的下标：

```c++
class QueryResult;//为了定义函数query的返回类型，这个定义是必需的
class TextQuery {
public:
    using line_no = std::vector<std:string>::size_type;
    TextQuery(std::ifstream&);
    QueryResult query(const std::string&) const;
private:
    std::shared_ptr<std::vector<std::string>> file;		//输入文件
    //每个单词到它所在的行号的集合的映射
    std::map<std::string,
    std::shared_ptr<std::set<line_no>>> wm;
}
```

这个类定义最困难的部分是解开类名。 与往常一样， 对于可能置于头文件中的代码， 在使用标准库名字时要加上std::（参见3.1节， 第74页） 。 在本例中， 我们反复使用了std:: ， 使得代码开始可能有些难读。 例如，

```c++
std::map<std::string, std::shared_ptr<std::set<line_no>>> wm;
```

如果写成下面的形式可能就更好理解一些

```c++
map< string, shared_ptr< set<line_no> > >wm;
```



#### TextQuery构造函数

TextQuery的构造函数接受一个ifstream， 逐行读取输入文件：

```c++
//读取输入文件并建立单词到行号的映射
TextQuery::TextQuery(ifstream &is) :file(new vector<string>)
{
    string text;
    while (getline(is,text)){				//对文件中每一行
        file->push_back(text);				//保存此行文本
        int n = file->size() - 1; 			//当前行号
        istringstream line(text);			//将行文本分解为单词
        string word;
        while (line >> word){				//对行中每个单词
            //如果单词不在wm中，以之为下标在wm中添加一项
            auto &lines = wm[word];		//lines是一个shared_ptr
            if(!lines)					//在我们第一次遇到这个单词时，此指针为空
            	lines.reset(new set<line_no>);		//分配一个新的set
            lines->insert(n);			//将此行号插入set中
   		}
    }
}
```

构造函数的初始化器分配一个新的vector来保存输入文件中的文本。 我们用getline逐行读取输入文件， 并存入vector中。 由于file是一个shared_ptr， 我们用->运算符解引用file来提取file指向的vector对象的push_back成员。

接下来我们用一个istringstream（参见8.3节， 第287页） 来处理刚刚读入的一行中的每个单词。 内层while循环用istringstream的输入运算符来从当前行读取每个单词， 存入word中。 在while循环内， 我们用map下标运算符提取与word相关联的shared_ptr<set>， 并将lines绑定到此指针。 注意， lines是一个引用， 因此改变lines也会改变wm中的元素。

若word不在map中， 下标运算符会将word添加到wm中（参见11.3.4节， 第387页） ， 与word关联的值进行值初始化。 这意味着， 如果下标运算符将word添加到wm中， lines将是一个空指针。 如果lines为空， 我们分配一个新的set， 并调用reset更新lines引用的shared_ptr， 使其指向这个新分配的set。

不管是否创建了一个新的set， 我们都调用insert将当前行号添加到set中。 由于lines是一个引用， 对insert的调用会将新元素添加到wm中的set中。 如果一个给定单词在同一行中出现多次， 对insert的调用什么都不会做。



#### QueryResult类

QueryResult类有三个数据成员： 一个string， 保存查询单词； 一个shared_ptr， 指向保存输入文件的vector； 一个shared_ptr， 指向保存单词出现行号的set。 它唯一的一个成员函数是一个构造函数， 初始化这三个数据成员：

```c++
class QueryResult {
    friend std::ostream& print(std:ostream&, const QueryResult&);
public:
    QueryResult (std::string s,
    			std::shared_ptr<std:set<line no>>p,
    			std::shared_ptr<std::vector<std:string>>f)
    :sought(s), lines(p), file(f) {}
private:
    std::string sought;//查询单词 
    std::shared_ptr<std::set<line_no>> lines;
    //出现的行号
    std::shared_ptr<std::vector<std::string>> file;
    //输入文件
}
```

构造函数的唯一工作是将参数保存在对应的数据成员中， 这是在其初始化器列表中完成的（参见7.1.4节， 第237页） 。



#### query函数

query函数接受一个string参数， 即查询单词， query用它来在map中定位对应的行号set。 如果找到了这个string， query函数构造一个QueryResult， 保存给定string、 TextQuery的file成员以及从wm中提取的set。

唯一的问题是： 如果给定string未找到， 我们应该返回什么？ 在这种情况下， 没有可返回的set。 为了解决此问题， 我们定义了一个局部static对象， 它是一个指向空的行号set的shared_ptr。 当未找到给定单词时， 我们返回此对象的一个拷贝：

```c++
QueryResult TextQuery:query(const string &sought) const
{
    //如果未找到sought,我们将返回一个指向此set的指针
    static shared_ptr<set<line_no>> nodata(new set<line_no>);
    //使用find而不是下标运算符来查找单词，避免将单词添加到wm中！
    auto loc = wm.find(sought);
    if (loc == wm.end())
    	return QueryResult(sought, nodata, file);		//未找到
    else
    	return QueryResult(sought, loc->second, file);
}
```



#### 打印结果

print函数在给定的流上打印出给定的QueryResult对象：

```c++
ostream &print(ostream os, const QueryResult &qr)
{
    //如果找到了单词，打印出现次数和所有出现的位置
    os << qr.sought << " occurs " << gr.lines->size() << " "
    	<< make_plural(qr.lines->size(), "time", "s") <<endl;
    //打印单词出现的每一行
    for(auto num : *gr.lines)			//对set中每个单词
    //避免行号从0开始给用户带来的困惑
        os << "\t(1ine" << num+1 << ")" << *(qr.file->begin() + num) << endl;
    return os;
}
```

我们调用qr.lines指向的set的size成员来报告单词出现了多少次。 由于set是一个shared_ptr， 必须解引用lines。 调用make_plural（参见6.3.2节， 第201页） 来根据大小是否等于1打印time或times。

在for循环中， 我们遍历lines所指向的set。 for循环体打印行号， 并按人们习惯的方式调整计数值。 set中的数值就是vector中元素的下标，从0开始编号。 但大多数用户认为第一行的行号应该是1， 因此我们对每个行号都加上1， 转换为人们更习惯的形式。

我们用行号从file指向的vector中提取一行文本。 回忆一下， 当给一个迭代器加上一个数时， 会得到vector中相应偏移之后位置的元素（参见3.4.2节， 第99页） 。 因此， file->begin() + num即为file指向的vector中第num个位置的元素。

注意此函数能正确处理未找到单词的情况。 在此情况下， set为空。第一条输出语句会注意到单词出现了0次。 由于 *res.lines为空， for循环一次也不会执行。