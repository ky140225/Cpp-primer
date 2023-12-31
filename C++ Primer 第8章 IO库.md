# 第8章 IO库  

## 8.1 IO类

到目前为止， 我们已经使用过的IO类型和对象都是操纵char数据的。 默认情况下， 这些对象都是关联到用户的控制台窗口的。 当然， 我们不能限制实际应用程序仅从控制台窗口进行IO操作， 应用程序常常需要读写命名文件。 而且， 使用IO操作处理string中的字符会很方便。 此外， 应用程序还可能读写需要宽字符支持的语言。

为了支持这些不同种类的IO处理操作， 在istream和ostream之外， 标准库还定义了其他一些IO类型， 我们之前都已经使用过了。 **表8.1列出了这些类型， 分别定义在三个独立的头文件中： iostream定义了用于读写流的基本类型， fstream定义了读写命名文件的类型， sstream定义了读写内存string对象的类型。**

![表8.1 IO库类型和头文件](C:\Users\Grey\Desktop\C++ Primer\图片\表8.1 IO库类型和头文件.png)

**为了支持使用宽字符的语言， 标准库定义了一组类型和对象来操纵wchar_t类型的数据（参见2.1.1节， 第30页） 。 宽字符版本的类型和函数的名字以一个w开始。 例如， wcin、 wcout和wcerr是分别对应cin、cout和cerr的宽字符版对象。** 宽字符版本的类型和对象与其对应的普通char版本的类型定义在同一个头文件中。 例如， 头文件fstream定义了ifstream和wifstream类型。  



#### IO类型间的关系

概念上， 设备类型和字符大小都不会影响我们要执行的IO操作。 例如， 我们可以用>>读取数据， 而不用管是从一个控制台窗口， 一个磁盘文件， 还是一个string读取。 类似的， 我们也不用管读取的字符能存入一个char对象内， 还是需要一个wchar_t对象来存储。

标准库使我们能忽略这些不同类型的流之间的差异， 这是通过继承机制（inheritance） 实现的。 利用模板（参见3.3节， 第87页） ， 我们可以使用具有继承关系的类， 而不必了解继承机制如何工作的细节。 我们将在第15章和18.3节（第710页） 介绍C++是如何支持继承机制的。

**简单地说， 继承机制使我们可以声明一个特定的类继承自另一个类。 我们通常可以将一个派生类（继承类） 对象当作其基类（所继承的类） 对象来使用。**

类型ifstream和istringstream都继承自istream。 因此， 我们可以像使用istream对象一样来使用ifstream和istringstream对象。 也就是说， 我们是如何使用cin的， 就可以同样地使用这些类型的对象。 例如， 可以对一个ifstream或istringstream对象调用getline， 也可以使用>>从一个ifstream或istringstream对象中读取数据。 类似的， 类型ofstream和ostringstream都继承自ostream。 因此， 我们是如何使用cout的， 就可以同样地使用这些类型的对象。  



### 8.1.1 IO对象无拷贝或赋值

如我们在7.1.3节（第234页） 所见， **我们不能拷贝或对IO对象赋值**：  

```c++
ofstream outl, out2;
out1 = out2;					// 错误：不能对流对象赋值
ofstream print(ofstream);		// 错误：不能初始化ofstream参数
out2 = print(out2);				// 错误：不能拷贝流对象
```

**由于不能拷贝IO对象， 因此我们也不能将形参或返回类型设置为流类型（参见6.2.1节， 第188页） 。 进行IO操作的函数通常以引用方式传递和返回流。 读写一个IO对象会改变其状态， 因此传递和返回的引用不能是const的。**



### 8.1.2 条件状态

IO操作一个与生俱来的问题是可能发生错误。 一些错误是可恢复的， 而其他错误则发生在系统深处， 已经超出了应用程序可以修正的范围。 表8.2列出了IO类所定义的一些函数和标志， 可以帮助我们访问和操纵流的条件状态（condition state） 。  

![表8.2 IO库条件状态](C:\Users\Grey\Desktop\C++ Primer\图片\表8.2 IO库条件状态.png)

> cin.setstate(std::ios::goodbit);     0
>
> cin.setstate(std::ios::eofbit);        1
>
> cin.setstate(std::ios::failbit);        2
>
> cin.setstate(std::ios::badbit);       4
>
> 后面数字是用rdstate打印的结果

下面是一个IO错误的例子：

```c++
int ival;
cin >> ival;
```

如果我们在标准输入上键入Boo， 读操作就会失败。 代码中的输入运算符期待读取一个int， 但却得到了一个字符B。 这样， cin会进入错误状态。 类似的， 如果我们输入一个文件结束标识， cin也会进入错误状态。

**一个流一旦发生错误， 其上后续的IO操作都会失败。 只有当一个流处于无错状态时， 我们才可以从它读取数据， 向它写入数据。 由于流可能处于错误状态**， 因此代码通常应该在使用一个流之前检查它是否处于良好状态。 确定一个流对象的状态的最简单的方法是将它当作一个条件来使用：

```c++
while(cin >> word)
	// ok:读操作成功…
```

while循环检查>>表达式返回的流的状态。 如果输入操作成功， 流保持有效状态， 则条件为真。



#### 查询流的状态  

将流作为条件使用， 只能告诉我们流是否有效， 而无法告诉我们具体发生了什么。 有时我们也需要知道流为什么失败。 例如， 在键入文件结束标识后我们的应对措施， 可能与遇到一个IO设备错误的处理方式是不同的。

IO库定义了一个与机器无关的iostate类型， 它提供了表达流状态的完整功能。 这个类型应作为一个位集合来使用， 使用方式与我们在4.8节中（第137页） 使用quiz1的方式一样。 IO库定义了4个iostate类型的constexpr值（参见2.4.4节， 第58页） ， 表示特定的位模式。 这些值用来表示特定类型的IO条件， 可以与位运算符（参见4.8节， 第137页） 一起使用来一次性检测或设置多个标志位。

**badbit表示系统级错误， 如不可恢复的读写错误。 通常情况下， 一旦badbit被置位， 流就无法再使用了。 在发生可恢复错误后， failbit被置位， 如期望读取数值却读出一个字符等错误。 这种问题通常是可以修正的， 流还可以继续使用。 如果到达文件结束位置， eofbit和failbit都会被置位。 goodbit的值为0， 表示流未发生错误。 如果badbit、 failbit和eofbit任一个被置位， 则检测流状态的条件会失败。**

**标准库还定义了一组函数来查询这些标志位的状态。 操作good在所有错误位均未置位的情况下返回true， 而bad、 fail和eof则在对应错误位被置位时返回true。 此外， 在badbit被置位时， fail也会返回true。 这意味着， 使用good或fail是确定流的总体状态的正确方法。 实际上， 我们将流当作条件使用的代码就等价于 !fail()。 而eof和bad操作只能表示特定的错误。**



#### 管理条件状态

**流对象的rdstate成员返回一个iostate值， 对应流的当前状态。setstate操作将给定条件位置位， 表示发生了对应错误。** clear成员是一个重载的成员（参见6.4节， 第206页） ： 它有一个不接受参数的版本， 而另一个版本接受一个iostate类型的参数。

**clear不接受参数的版本清除（复位） 所有错误标志位。 执行clear() 后， 调用good会返回true。** 我们可以这样使用这些成员：

```c++
// 记住cin的当前状态
auto old_state = cin.rdstate();		// 记住cin的当前状态
cin.clear();						// 使cin有效
process_input(cin);					// 使用cin
cin.setstate(old_state);			// 将cin置为原有状态
```

**带参数的clear版本接受一个iostate值， 表示流的新状态。 为了复位单一的条件状态位， 我们首先用rdstate读出当前条件状态， 然后用位操作将所需位复位来生成新的状态。** 例如， 下面的代码将failbit和badbit复位， 但保持eofbit不变：  

```c++
// 复位failbit和badbit,保持其他标志位不变
cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
```



### 8.1.3 管理输出缓冲

每个输出流都管理一个缓冲区， 用来保存程序读写的数据。 例如，如果执行下面的代码

```c++
OS << "please enter a value:"
```

文本串可能立即打印出来， 但也有可能被操作系统保存在缓冲区中， 随后再打印。 有了缓冲机制， 操作系统就可以将程序的多个输出操作组合成单一的系统级写操作。 由于设备的写操作可能很耗时， 允许操作系统将多个输出操作组合为单一的设备写操作可以带来很大的性能提升。

导致缓冲刷新（即， 数据真正写到输出设备或文件） 的原因有很多：

 **· 程序正常结束， 作为main函数的return操作的一部分， 缓冲刷新被执行。**

 **· 缓冲区满时， 需要刷新缓冲， 而后新的数据才能继续写入缓冲区。**

 **· 我们可以使用操纵符如endl（参见1.2节， 第6页） 来显式刷新缓冲区。**

 **· 在每个输出操作之后， 我们可以用操纵符unitbuf设置流的内部状态， 来清空缓冲区。 默认情况下， 对cerr是设置unitbuf的， 因此写到cerr的内容都是立即刷新的。**

 **· 一个输出流可能被关联到另一个流。 在这种情况下， 当读写被关联的流时， 关联到的流的缓冲区会被刷新。 例如， 默认情况下， cin和cerr都关联到cout。 因此， 读cin或写cerr都会导致cout的缓冲区被刷新。**  



#### 刷新输出缓冲区

我们已经使用过**操纵符endl， 它完成换行并刷新缓冲区的工作。** IO库中还有两个类似的操纵符： **flush和ends。 flush刷新缓冲区， 但不输出任何额外的字符； ends向缓冲区插入一个空字符， 然后刷新缓冲区：**

```c++
cout << "hi！" << endl;			// 输出hi和一个换行，然后刷新缓冲区
cout << "hi！" << f1ush;			// 输出hi,然后刷新缓冲区，不附加任何额外字符
cout << "hi!" << ends;			// 输出hi和一个空字符，然后刷新缓冲区
```



#### unitbuf操纵符

**如果想在每次输出操作后都刷新缓冲区， 我们可以使用unitbuf操纵符。** 它告诉流在接下来的每次写操作之后都进行一次flush操作。 而nounitbuf操纵符则重置流， 使其恢复使用正常的系统管理的缓冲区刷新机制：

```c++
cout << unitbuf;				// 所有输出操作后都会立即刷新缓冲区
// 任何输出都立即刷新，无缓冲
cout << nounitbuf;				// 回到正常的缓冲方式
```

**<u>警告： 如果程序崩溃， 输出缓冲区不会被刷新</u>**

<u>如果程序异常终止， 输出缓冲区是不会被刷新的。 当一个程序崩溃后， 它所输出的数据很可能停留在输出缓冲区中等待打印。</u>

<u>当调试一个已经崩溃的程序时， 需要确认那些你认为已经输出的数据确实已经刷新了。 否则， 可能将大量时间浪费在追踪代码为什么没有执行上， 而实际上代码已经执行了， 只是程序崩溃后缓冲区没有被刷新， 输出数据被挂起没有打印而已。</u>  



#### 关联输入和输出流

当一个输入流被关联到一个输出流时， 任何试图从输入流读取数据的操作都会先刷新关联的输出流。 标准库将cout和cin关联在一起， 因此下面语句

```c++
cin >> ival;
```

导致cout的缓冲区被刷新。

<u>交互式系统通常应该关联输入流和输出流。 这意味着所有输出， 包括用户提示信息， 都会在读操作之前被打印出来。</u>

tie有两个重载的版本（参见6.4节， 第206页） ： **一个版本不带参数， 返回指向输出流的指针。 如果本对象当前关联到一个输出流， 则返回的就是指向这个流的指针， 如果对象未关联到流， 则返回空指针。 tie的第二个版本接受一个指向ostream的指针， 将自己关联到此ostream。即， x.tie（&o） 将流x关联到输出流o。**

我们既可以将一个istream对象关联到另一个ostream， 也可以将一个ostream关联到另一个ostream：

```c++
cin.tie(&cout);			// 仅仅是用来展示：标准库将cin和cout关联在一起
// old_tie指向当前关联到cin的流（如果有的话）
ostream *old_tie = cin.tie(nullptr);	// cin不再与其他流关联
// 将cin与cerr关联；这不是一个好主意，因为cin应该关联到cout
cin.tie(&cerr);							// 读取cin会刷新cerr而不是cout
cin.tie(old_tie);						// 重建cin和cout间的正常关联
```

在这段代码中， 为了将一个给定的流关联到一个新的输出流， 我们将新流的指针传递给了tie。 为了彻底解开流的关联， 我们传递了一个空指针。 **每个流同时最多关联到一个流， 但多个流可以同时关联到同一个ostream。**  	



## 8.2 文件输入输出

头文件fstream定义了三个类型来支持文件IO： ifstream从一个给定文件读取数据， ofstream向一个给定文件写入数据， 以及fstream可以读写给定文件。 在17.5.3节中（第676页） 我们将介绍如何对同一个文件流既读又写。

这些类型提供的操作与我们之前已经使用过的对象cin和cout的操作一样。 特别是， **我们可以用IO运算符（<<和>>） 来读写文件， 可以用getline（参见3.2.2节， 第79页） 从一个ifstream读取数据。**

除了继承自iostream类型的行为之外， fstream中定义的类型还增加了一些新的成员来管理与流关联的文件。 在表8.3中列出了这些操作，我们可以对fstream、 ifstream和ofstream对象调用这些操作， 但不能对其他IO类型调用这些操作。  

![表8.3 fstream特有的操作](C:\Users\Grey\Desktop\C++ Primer\图片\表8.3 fstream特有的操作.png)



### 8.2.1 使用文件流对象

当我们想要读写一个文件时， 可以定义一个文件流对象， 并将对象与文件关联起来。 每个文件流类都定义了一个名为open的成员函数， 它完成一些系统相关的操作， 来定位给定的文件， 并视情况打开为读或写模式。

创建文件流对象时， 我们可以提供文件名（可选的） 。 如果提供了一个文件名， 则open会自动被调用：

```c++
ifstream in(ifile);			// 构造一个ifstream并打开给定文件
ofstream out;				// 输出文件流未关联到任何文件
```

**这段代码定义了一个输入流in， 它被初始化为从文件读取数据， 文件名由string类型的参数ifile指定。 第二条语句定义了一个输出流out， 未与任何文件关联。 在新C++标准中， 文件名既可以是库类型string对象，也可以是C风格字符数组（参见3.5.4节， 第109页） 。 旧版本的标准库只允许C风格字符数组。**

> ifstream 参数文件（infile）需要先存在，
>
> ofsream可以没有并且每次打开文件内容会清空



#### 用fstream代替iostream&

我们在8.1节（第279页） 已经提到过， 在要求使用基类型对象的地方， 我们可以用继承类型的对象来替代。 这意味着， 接受一个iostream类型引用（或指针） 参数的函数， 可以用一个对应的fstream（或sstream） 类型来调用。 **也就是说， 如果有一个函数接受一个ostream&参数， 我们在调用这个函数时， 可以传递给它一个ofstream对象， 对istream&和ifstream也是类似的。**

例如， 我们可以用7.1.3节中的read和print函数来读写命名文件。 在本例中， 我们假定输入和输出文件的名字是通过传递给main函数的参数来指定的（参见6.2.5节， 第196页） ：

```c++
ifstream input(argv[1]);					// 打开销售记录文件
ofstream output(argv[2]);					// 打开输出文件
Sales_data total;							// 保存销售总额的变量
if (read(input, total)){					// 读取第一条销售记录
	Sales_data trans;						// 保存下一条销售记录的变量
	while(read(input,trans)){				// 读取剩余记录
		if (total.isbn () = trans.isbn ()	// 检查isbn
			total.combine(trans);			// 更新销售总额
		else
			print(output, total) << endl;	// 打印结果
	total = trans;							// 处理下一本书
	print(output,total) << endl;			// 打印最后一本书的销售额
else										// 文件中无输入数据
	cerr << "No data?!" << endl;
```

除了读写的是命名文件外， 这段程序与229页的加法程序几乎是完全相同的。 重要的部分是对read和print的调用。 虽然两个函数定义时指定的形参分别是istream&和ostream&， 但我们可以向它们传递fstream对象。

> 如果要用main形参需要先把argv[1]...保存到string变量中，再用 ifstream / ofstream 打开文件



#### 成员函数open和close  

如果我们定义了一个空文件流对象， 可以随后调用open来将它与文件关联起来：

```c++
ifstream in(ifile);				// 构筑一个ifstream并打开给定文件
ofstream out;					// 输出文件流未与任何文件相关联
out.open (ifile + "copy");		// 打开指定文件
```

> 可以直接用 ifstream / ofstream创建流并且打开文件 或者 
>
> 先创建流，再用open打开文件

**如果调用open失败， failbit会被置位（参见8.1.2节， 第280页） 。 因为调用open可能失败， 进行open是否成功的检测通常是一个好习惯：**

```c++
if(out)							// 检查open是否成功
								// open成功，我们可以使用文件了
```

这个条件判断与我们之前将cin用作条件相似。 如果open失败， 条件会为假， 我们就不会去使用out了。

一旦一个文件流已经打开， 它就保持与对应文件的关联。 实际上，对一个已经打开的文件流调用open会失败， 并会导致failbit被置位。 随后的试图使用文件流的操作都会失败。 **为了将文件流关联到另外一个文件， 必须首先关闭已经关联的文件。 一旦文件成功关闭， 我们可以打开新的文件：**

```c++
in.close()						// 关闭文件
in.open(ifile + "2");			// 打开另一个文件
```

如果open成功， 则open会设置流的状态， 使得good（） 为true。

> close关闭的是文件，流还在，所以可以继续打开别的文件



#### 自动构造和析构

考虑这样一个程序， 它的main函数接受一个要处理的文件列表（参见6.2.5节， 第196页） 。 这种程序可能会有如下的循环：

```c++
// 对每个传递给程序的文件执行循环操作
for(auto p = argv + 1; p != argv + argc; ++p){
	ifstream input(*p);				// 创建输出流并打开文件
	if (input){						// 如果文件打开成功，“处理”此文件
		process(input);
    } else
		cerr << "couldn't open: " + string(*p);
} // 每个循环步input都会离开作用域，因此会被销毁
```

每个循环步构造一个新的名为input的ifstream对象， 并打开它来读取给定的文件。 像之前一样， 我们检查open是否成功。 如果成功， 将文件传递给一个函数， 该函数负责读取并处理输入数据。 如果open失败，打印一条错误信息并继续处理下一个文件。

因为input是while循环的局部变量， 它在每个循环步中都要创建和销毁一次（参见5.4.1节， 第165页） 。 当一个fstream对象离开其作用域时， 与之关联的文件会自动关闭。 在下一步循环中， input会再次被创建。

<u>**当一个fstream对象被销毁时， close会自动被调用。**</u>  



### 8.2.2 文件模式

每个流都有一个关联的文件模式（file mode） ， 用来指出如何使用文件。 表8.4列出了文件模式和它们的含义。

![表8.4 文件模式](C:\Users\Grey\Desktop\C++ Primer\图片\表8.4 文件模式.png)

无论用哪种方式打开文件， 我们都可以指定文件模式， 调用open打开文件时可以， 用一个文件名初始化流来隐式打开文件时也可以。 指定文件模式有如下限制：

**· 只可以对ofstream或fstream对象设定out模式。**

**· 只可以对ifstream或fstream对象设定in模式。**

**· 只有当out也被设定时才可设定trunc模式。**

**· 只要trunc没被设定， 就可以设定app模式。 在app模式下， 即使没有显式指定out模式， 文件也总是以输出方式被打开。**

**· 默认情况下， 即使我们没有指定trunc， 以out模式打开的文件也会被截断。 为了保留以out模式打开的文件的内容， 我们必须同时指定app模式， 这样只会将数据追加写到文件末尾； 或者同时指定in模式， 即打开文件同时进行读写操作**（参见17.5.3节， 第676页， 将介绍对同一个文件既进行输入又进行输出的方法） 。

**· ate和binary模式可用于任何类型的文件流对象， 且可以与其他任何文件模式组合使用。**

**每个文件流类型都定义了一个默认的文件模式， 当我们未指定文件模式时， 就使用此默认模式。 与ifstream关联的文件默认以in模式打开；与ofstream关联的文件默认以out模式打开； 与fstream关联的文件默认以in和out模式打开。**  



#### 以out模式打开文件会丢弃已有数据

**默认情况下， 当我们打开一个ofstream时， 文件的内容会被丢弃。阻止一个ofstream清空给定文件内容的方法是同时指定app模式：**

```c++
// 在这几条语句中，fi1e1都被截断
ofstream out("file1");						// 隐含以输出模式打开文件并截断文件
ofstream out2("filel", ofstream::out);		// 隐含地截断文件
ofstream out3("filel", ofstream:out | ofstream:trunc)
// 为了保留文件内容，我们必须显式指定app模式
ofstream app("file2",ofstream::app);		// 隐含为输出模式
ofstream app2("file2",ofstream::out ofstream::app);
```

<u>保留被ofstream打开的文件中已有数据的唯一方法是显式指定app或in模式。</u>



#### 每次调用open时都会确定文件模式

对于一个给定流， 每当打开文件时， 都可以改变其文件模式。

```c++
ofstream out;							// 未指定文件打开模式
out.open("scratchpad");					// 模式隐含设置为输出和截断
out.c1ose();							// 关闭out,以便我们将其用于其他文件
out.open("precious", ofstream::app);	// 模式为输出和追加
out.c1ose（),
```

第一个open调用未显式指定输出模式， 文件隐式地以out模式打开。 通常情况下， out模式意味着同时使用trunc模式。 因此， 当前目录下名为scratchpad的文件的内容将被清空。 当打开名为precious的文件时， 我们指定了append模式。 文件中已有的数据都得以保留， 所有写操作都在文件末尾进行。

<u>在每次打开文件时， 都要设置文件模式， 可能是显式地设置， 也可能是隐式地设置。 当程序未指定模式时， 就使用默认值。</u>  



## 8.3 string流

### 8.3.1 使用istringstream

当我们的某些工作是对整行文本进行处理， 而其他一些工作是处理行内的单个单词时， 通常可以使用istringstream。

考虑这样一个例子， 假定有一个文件， 列出了一些人和他们的电话号码。 某些人只有一个号码， 而另一些人则有多个——家庭电话、 工作电话、 移动电话等。 我们的输入文件看起来可能是这样的：

`morgan 2015552368 8625550123`

`drew 9735550130`

`lee 6095550132 2015550175 8005550000`

文件中每条记录都以一个人名开始， 后面跟随一个或多个电话号码。 我们首先定义一个简单的类来描述输入数据：

```c++
//成员默认为公有；参见7.2节（第240页)
struct PersonInfo {
	string name;
	vector<string> phones;
};
```

类型PersonInfo的对象会有一个成员来表示人名， 还有一个vector来保存此人的所有电话号码。

我们的程序会读取数据文件， 并创建一个PersonInfo的vector。vector中每个元素对应文件中的一条记录。 我们在一个循环中处理输入数据， 每个循环步读取一条记录， 提取出一个人名和若干电话号码：

```c++
string line, word;						// 分别保存来自输入的一行和单词
vector<PersonInfo> people;				// 保存来自输入的所有记录
// 逐行从输入读取数据，直至cin遇到文件尾（或其他错误)
while (getline(cin,line)) {
	PersonInfo info;					// 创建一个保存此记录数据的对象
	istringstream record(line);			// 将记录绑定到刚读入的行
    // istringstream.clear();
	record >> info.name;				// 读取名字
	while (record >> word)				// 读取电话号码
		info.phones.push back(word);	// 保持它们
	people.push_back(info);				// 将此记录追加到people末尾
}
```

这里我们用getline从标准输入读取整条记录。 如果getline调用成功， 那么line中将保存着从输入文件而来的一条记录。 在while中， 我们定义了一个局部PersonInfo对象， 来保存当前记录中的数据。

接下来我们将一个istringstream与刚刚读取的文本行进行绑定， 这样就可以在此istringstream上使用输入运算符来读取当前记录中的每个元素。 我们首先读取人名， 随后用一个while循环读取此人的电话号码。

当读取完line中所有数据后， 内层while循环就结束了。 此循环的工作方式与前面章节中读取cin的循环很相似， 不同之处是， 此循环从一个string而不是标准输入读取数据。 当string中的数据全部读出后， 同样会触发“文件结束”信号， 在record上的下一个输入操作会失败。

我们将刚刚处理好的PersonInfo追加到vector中， 外层while循环的一个循环步就随之结束了。 外层while循环会持续执行， 直至遇到cin的文件结束标识。  

[stringstream流操作中.clear() 与 .str("")的使用](https://blog.csdn.net/qq_41282102/article/details/88381649)



### 8.3.2 使用ostringstream

当我们逐步构造输出， 希望最后一起打印时， ostringstream是很有用的。 例如， 对上一节的例子， 我们可能想逐个验证电话号码并改变其格式。 如果所有号码都是有效的， 我们希望输出一个新的文件， 包含改变格式后的号码。 对于那些无效的号码， 我们不会将它们输出到新文件中， 而是打印一条包含人名和无效号码的错误信息。

由于我们不希望输出有无效电话号码的人， 因此对每个人， 直到验证完所有电话号码后才可以进行输出操作。 但是， 我们可以先将输出内容“写入”到一个内存ostringstream中：

```c++
for(const auto &entry:people){			// 对people中每一项
ostringstream formatted,badNums;		// 每个循环步创建的对象
for(const auto &nums:entry.phones) {	// 对每个数
	if (!valid(nums)){
		badNums << " " << nums;			// 将数的字符串形式存入badNums
    } else
		// 将格式化的字符串“写入”formatted
		formatted << " " << format (nums);
}
if (badNums.str().empty())				// 没有错误的数
	os << entry.name << " "				// 打印名字
	   << formatted.str() << endl;		// 和格式化的数
else									// 否则，打印名字和错误的数
	cerr << "input error: " << entry.name
		 << "invalid number (s) " << badNums.str() << endl;
}
```

在此程序中， 我们假定已有两个函数， valid和format， 分别完成电话号码验证和改变格式的功能。 程序最有趣的部分是对字符串流formatted和badNums的使用。 我们使用标准的输出运算符（<<） 向这些对象写入数据， 但这些“写入”操作实际上转换为string操作， 分别向formatted和badNums中的string对象添字符。  =