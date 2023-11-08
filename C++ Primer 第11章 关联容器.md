# 第11章　关联容器

关联容器和顺序容器有着根本的不同：**关联容器中的元素是按关键字来保存和访问的。与之相对，顺序容器中的元素是按它们在容器中的位置来顺序保存和访问的。**

虽然关联容器的很多行为与顺序容器相同，但其不同之处反映了关键字的作用。

**关联容器支持高效的关键字查找和访问。两个主要的关联容器（associative-container）类型是map和set。map中的元素是一些关键字-值（key-value）对：关键字起到索引的作用，值则表示与索引相关联的数据。set中每个元素只包含一个关键字；set支持高效的关键字查询操作——检查一个给定关键字是否在set中。**例如，在某些文本处理过程中，可以用一个set来保存想要忽略的单词。字典则是一个很好的使用map的例子：可以将单词作为关键字，将单词释义作为值。

标准库提供8个关联容器，如表11.1所示。这8个容器间的不同体现在三个维度上：每个容器（1）或者是一个set，或者是一个map；（2）或者要求不重复的关键字，或者允许重复关键字；（3）按顺序保存元素，或无序保存。允许重复关键字的容器的名字中都包含单词multi；关键字不按顺序存储的容器的名字都以单词unordered开头。因此一个unordered_multi_set是一个允许重复关键字，元素无序保存的集合，而一个set则是一个要求不重复关键字，有序存储的集合。无序容器使用哈希函数来组织元素，我们将在11.4节（第394页）中详细介绍有关哈希函数的更多内容。

**类型map和multimap定义在头文件map中；set和multiset定义在头文件set中；无序容器则定义在头文件unordered_map和unordered_set中。**

![表11.1 关联容器类型](C:\Users\Grey\Desktop\C++ Primer\图片\表11.1 关联容器类型.png)



## 11.1　使用关联容器

**map是关键字-值对的集合。例如，可以将一个人的名字作为关键字，将其电话号码作为值。我们称这样的数据结构为“将名字映射到电话号码”。map类型通常被称为关联数组（associative array）** 。关联数组与“正常”数组类似，不同之处在于其下标不必是整数。**我们通过一个关键字而不是位置来查找值。**给定一个名字到电话号码的map，我们可以使用一个人的名字作为下标来获取此人的电话号码。	

**与之相对，set就是关键字的简单集合。当只是想知道一个值是否存在时，set是最有用的。**例如，一个企业可以定义一个名为bad_checks的set来保存那些曾经开过空头支票的人的名字。在接受一张支票之前，可以查询bad_checks来检查顾客的名字是否在其中。



#### 使用map

一个经典的使用关联数组的例子是单词计数程序：

```c++
//统计每个单词在输入中出现的次数
map<string, size_t> word_count;			//string到size_t的空map
string word;
while (cin >> word)
	++word_count[word];					//提取word的计数器并将其加1
for(const auto &w : word_count)			//对map中的每个元素
    //打印结果
    cout << w.first << " occurs " << w.second
    	<< ((w.second > 1) ? " times" : " time") << endl;
```

此程序读取输入，报告每个单词出现多少次。

**类似顺序容器，关联容器也是模板**（参见3.3节，第86页）。为了定义一个map，我们必须指定关键字和值的类型。在此程序中，map保存的每个元素中，关键字是string类型，值是size_t类型（参见3.5.2节，第103页）。当对word_count进行下标操作时，我们使用一个string作为下标，获得与此string相关联的size_t类型的计数器。

while循环每次从标准输入读取一个单词。它使用每个单词对word_count进行下标操作。**如果word还未在map中，下标运算符会创建一个新元素，其关键字为word，值为0。**不管元素是否是新创建的，我们将其值加1。

一旦读取完所有输入，范围for语句（参见3.2.3节，第81页）就会遍历map，打印每个单词和对应的计数器。**当从map中提取一个元素时，会得到一个pair类型的对象**，我们将在11.2.3节（第379页）介绍它。简单来说，**pair是一个模板类型，保存两个名为first和second的（公有）数据成员。map所使用的pair用first成员保存关键字，用second成员保存对应的值。**因此，输出语句的效果是打印每个单词及其关联的计数器。

如果我们对本节第一段中的文本（指英文版中的文本）运行这个程序，输出将会是：

`Although occurs 1 time`

`Before occurs 1 time`

`an occurs 1 time`

`and occurs 1 time`

`...`



#### 使用set

上一个示例程序的一个合理扩展是：忽略常见单词，如"the"、"and"、"or"等。我们可以使用set保存想忽略的单词，只对不在集合中的单词统计出现次数：

```c++
//统计输入中每个单词出现的次数
map<string,size_t> word_count;			//string到size_t的空map
set<string> exclude {"The","But","And","Or","An","A",
					"the","but","and","or","an","a"};
string word;
while (cin >> word)
    //只统计不在exclude中的单词
    if (exclude.find(word) == exclude.end())
    	++word_count[word];				//获取并递增word的计数器
```

**与其他容器类似，set也是模板。为了定义一个set，必须指定其元素类型**，本例中是string。与顺序容器类似，**可以对一个关联容器的元素进行列表初始化**（参见9.2.4节，第300页）。集合exclude中保存了12个我们想忽略的单词。

此程序与前一个程序的重要不同是，在统计每个单词出现次数之前，我们检查单词是否在忽略集合中，这是在if语句中完成的：

```c++
 //只统计不在exclude中的单词
if (exclude.find(word) == exclude.end())
```

**find调用返回一个迭代器。如果给定关键字在set中，迭代器指向该关键字。否则，find返回尾后迭代器。**在此程序中，仅当word不在exclude中时我们才更新word的计数器。

如果用此程序处理与之前相同的输入，输出将会是：

`Although occurs 1 time`

`Before occurs 1 time`

`are occurs 1 time`

`as occurs 1 time`

`...`



## 11.2　关联容器概述

**关联容器（有序的和无序的）都支持9.2节（第294页）中介绍的普通容器操作（列于表9.2，第295页）。关联容器不支持顺序容器的位置相关的操作，例如push_front或push_back。原因是关联容器中元素是根据关键字存储的，这些操作对关联容器没有意义。而且，关联容器也不支持构造函数或插入操作这些接受一个元素值和一个数量值的操作。**

除了与顺序容器相同的操作之外，关联容器还支持一些顺序容器不支持的操作（参见表11.7，第388页）和类型别名（参见表11.3，第381页）。此外，无序容器还提供一些用来调整哈希性能的操作，我们将在11.4节（第394页）中介绍。

关联容器的迭代器都是双向的（参见10.5.1节，第365页）。

![表9.2 容器操作](C:\Users\Grey\Desktop\C++ Primer\图片\表9.2 容器操作.png)



### 11.2.1　定义关联容器

如前所示，当定义一个map时，必须既指明关键字类型又指明值类型；而定义一个set时，只需指明关键字类型，因为set中没有值。**每个关联容器都定义了一个默认构造函数，它创建一个指定类型的空容器。我们也可以将关联容器初始化为另一个同类型容器的拷贝，或是从一个值范围来初始化关联容器，只要这些值可以转化为容器所需类型就可以。在新标准下，我们也可以对关联容器进行值初始化：**

```c++
map<string,size_t> word_count;				//空容器
//列表初始化
set<string> exclude = {"the","but","and","or","an","a"
						"The","But","And","or""An","A"};
//三个元素；authors将姓映射为名
map<string, string> authors = {{"Joyce","James"},
                            {"Austen","Jane"},
                            {"Dickens","Charles"}}
```

与以往一样，初始化器必须能转换为容器中元素的类型。对于set，元素类型就是关键字类型。

当初始化一个map时，必须提供关键字类型和值类型。我们将每个关键字-值对包围在花括号中：

来指出它们一起构成了map中的一个元素。在每个花括号中，关键字是第一个元素，值是第二个。因此，authors将姓映射到名，初始化后它包含三个元素。



#### 初始化multimap或multiset

**一个map或set中的关键字必须是唯一的，即，对于一个给定的关键字，只能有一个元素的关键字等于它。容器multimap和multiset没有此限制，它们都允许多个元素具有相同的关键字。**例如，在我们用来统计单词数量的map中，每个单词只能有一个元素。另一方面，在一个词典中，一个特定单词则可具有多个与之关联的词义。

下面的例子展示了具有唯一关键字的容器与允许重复关键字的容器之间的区别。首先，我们将创建一个名为ivec的保存int的vector，它包含20个元素：0到9每个整数有两个拷贝。我们将使用此vector初始化一个set和一个multiset：

```c++
//定义一个有20个元素的vector,保存0到9每个整数的两个拷贝
vector<int> ivec;
for (vector<int>::size_type i = 0;i != 10; ++i){
    ivec.push_back(i);
    ivec.push_back(i);//每个数重复保存一次
}
    //iset包含来自ivec的不重复的元素；miset包含所有20个元素
    set<int> iset(ivec.cbegin(), ivec.cend());
    multiset<int> miset(ivec.cbegin(), ivec.cend());
    cout << ivec.size() << endl;			  	 //打印出20
    cout << iset.size() << endl;				 //打印出10
    cout << miset.size() << endl;				 //打印出20
```

即使我们用整个ivec容器来初始化iset，它也只含有10个元素：对应ivec中每个不同的元素。另一方面，miset有20个元素，与ivec中的元素数量一样多。



### 11.2.2　关键字类型的要求

关联容器对其关键字类型有一些限制。对于无序容器中关键字的要求，我们将在11.4节（第396页）中介绍。对于有序容器——map、multimap、set以及multiset，关键字类型必须定义元素比较的方法。默认情况下，标准库使用关键字类型的<运算符来比较两个关键字。在集合类型中，关键字类型就是元素类型；在映射类型中，关键字类型是元素的第一部分的类型。因此，11.2节（第377页）中word_count的关键字类型是string。类似的，exclude的关键字类型也是string。



#### 有序容器的关键字类型

可以向一个算法提供我们自己定义的比较操作（参见10.3节，第344页），与之类似，也可以提供自己定义的操作来代替关键字上的<运算符。**所提供的操作必须在关键字类型上定义一个严格弱序（strictweak ordering）。可以将严格弱序看作“小于等于”，虽然实际定义的操作可能是一个复杂的函数。无论我们怎样定义比较函数，它必须具备如下基本性质：**

· 两个关键字不能同时“小于等于”对方；如果k1“小于等于”k2，那么k2绝不能“小于等于”k1。

· 如果k1“小于等于”k2，且k2“小于等于”k3，那么k1必须“小于等于”k3。

· 如果存在两个关键字，任何一个都不“小于等于”另一个，那么我们称这两个关键字是“等价”的。如果k1“等价于”k2，且k2“等价于”k3，那么k1必须“等价于”k3。

如果两个关键字是等价的（即，任何一个都不“小于等于”另一个），那么容器将它们视作相等来处理。当用作map的关键字时，只能有一个元素与这两个关键字关联，我们可以用两者中任意一个来访问对应的值。



#### 使用关键字类型的比较函数

用来组织一个容器中元素的操作的类型也是该容器类型的一部分。为了指定使用自定义的操作，必须在定义关联容器类型时提供此操作的类型。如前所述，**用尖括号指出要定义哪种类型的容器，自定义的操作类型必须在尖括号中紧跟着元素类型给出。**

在尖括号中出现的每个类型，就仅仅是一个类型而已。当我们创建一个容器（对象）时，才会以构造函数参数的形式提供真正的比较操作（其类型必须与在尖括号中指定的类型相吻合）。

例如，我们不能直接定义一个Sales_data的multiset，因为Sales_data没有<运算符。但是，可以用10.3.1节练习（第345页）中的compareIsbn函数来定义一个multiset。此函数在Sales_data对象的ISBN成员上定义了一个严格弱序。函数compareIsbn应该像下面这样定义

```c++
bool compareIsbn(const Sales_data &lhs,const Sales_data &rhs)
{
	return lhs.isbn() < rhs.isbn()
}
```

为了使用自己定义的操作，在定义multiset时我们必须提供两个类型：关键字类型Sales_data，以及比较操作类型——应该是一种函数指针类型（参见6.7节，第221页），可以指向compareIsbn。当定义此容器类型的对象时，需要提供想要使用的操作的指针。在本例中，我们提供一个指向compareIsbn的指针：

```c++
//bookstore中多条记录可以有相同的ISBN
//bookstore中的元素以ISBN的顺序进行排列
multiset<Sales_data, decltype(compareIsbn)*> bookstore(compareIsbn);
```

此处，我们使用decltype来指出自定义操作的类型。记住，当用decltype来获得一个函数指针类型时，必须加上一个*来指出我们要使用一个给定函数类型的指针（参见6.7节，第223页）。用compareIsbn来初始化bookstore对象，这表示当我们向bookstore添加元素时，通过调用compareIsbn来为这些元素排序。即，bookstore中的元素将按它们的ISBN成员的值排序。可以用compareIsbn代替&compareIsbn作为构造函数的参数，因为当我们使用一个函数的名字时，在需要的情况下它会自动转化为一个指针（参见6.7节，第221页）。当然，使用&compareIsbn的效果也是一样的。



### 11.2.3　pair类型

在介绍关联容器操作之前，我们需要了解名为pair的标准库类型，它定义在头文件utility中。

**一个pair保存两个数据成员。类似容器，pair是一个用来生成特定类型的模板。当创建一个pair时，我们必须提供两个类型名，pair的数据成员将具有对应的类型。两个类型不要求一样：**

```c++
pair<string, string> anon;					//保存两个string
pair<string, size_t> word_count;			//保存一个string和一个size_t
pair<string, vector<int>> line;				//保存string和vector<int>
```

**pair的默认构造函数对数据成员进行值初始化**（参见3.3.1节，第88页）。因此，anon是一个包含两个空string的pair，line保存一个空string和一个空vector。word_count中的size_t成员值为0，而string成员被初始化为空。

我们也可以为每个成员提供初始化器：

```c++
pair<string,string> author{"James","Joyce"};
```

这条语句创建一个名为author的pair，两个成员被初始化为"James"和"Joyce"。

与其他标准库类型不同，pair的数据成员是public的（参见7.2节，第240页）。两个成员分别命名为first和second。我们用普通的成员访问符号（参见1.5.2节，第20页）来访问它们，例如，在第375页的单词计数程序的输出语句中我们就是这么做的：

```c++
//打印结果
cout << w.first << " occurs " << w.second
	<<((w.second < 1) ? " times " : " time ") << endl;
```

**此处，w是指向map中某个元素的引用。map的元素是pair**。在这条语句中，我们首先打印关键字——元素的first成员，接着打印计数器——second成员。标准库只定义了有限的几个pair操作，表11.2列出了这些操作。

![表11.2 pair上的操作](C:\Users\Grey\Desktop\C++ Primer\图片\表11.2 pair上的操作.png)



#### 创建pair对象的函数

想象有一个函数需要返回一个pair。在新标准下，我们可以对返回值进行列表初始化（参见6.3.2节，第203页）

```c++
pair<string,int> process (vector<string> &v)
{
    //处理v
    	
    if(!v.empty())
    	return{v.back(), v.back().size() };		//列表初始化
    else
    	return pair<string,int>();				//隐式构造返回值
}
```

若v不为空，我们返回一个由v中最后一个string及其大小组成的pair。否则，隐式构造一个空pair，并返回它。

在较早的C++版本中，不允许用花括号包围的初始化器来返回pair这种类型的对象，必须显式构造返回值：

```c++
if(!v.empty())
	return pair<string,int>(v.back(), v.back().size());		
```

我们还可以用make_pair来生成pair对象，pair的两个类型来自于make_pair的参数：

```c++
if (!v.empty())
	return make_pair(v.back(), v.back().size());
```



## 11.3　关联容器操作

除了表9.2（第295页）中列出的类型，关联容器还定义了表11.3中列出的类型。这些类型表示容器关键字和值的类型。

![表11.3 关联容器额外的类型别名](C:\Users\Grey\Desktop\C++ Primer\图片\表11.3 关联容器额外的类型别名.png)

对于set类型，key_type和value_type是一样的；set中保存的值就是关键字。在一个map中，元素是关键字-值对。即，每个元素是一个pair对象，包含一个关键字和一个关联的值。由于我们不能改变一个元素的关键字，因此这些pair的关键字部分是const的：

```c++
set<string>::value_type v1;				//v1是一个string
set<string>::key_type v2;				//v2是一个string
map<string, int>::value_type v3;		//v3是一个pair<const string, int>
map<string, int>:key_type v4;			//v4 string
map<string, int>:mapped_type v5;		//v5 int
```

与顺序容器一样（参见9.2.2节，第297页），我们使用作用域运算符来提取一个类型的成员——例如，map<string，int>::key_type。只有map类型（unordered_map、unordered_multimap、multimap和map）才定义了mapped_type。



### 11.3.1　关联容器迭代器

**当解引用一个关联容器迭代器时，我们会得到一个类型为容器的value_type的值的引用。**对map而言，value_type是一个pair类型，其first成员保存const的关键字，second成员保存值：

```c++
//获得指向word_count中一个元素的迭代器
auto map_it = word_count.begin();
//*map_it是指向一个pair<const string,size t>对象的引用
cout << map_it->first;				//打印此元素的关键字
cout << " " << map_it->second;		//打印此元素的值
map_it->first = "new key";			//错误：关键字是const的
++map_it->second;					//正确：我们可以通过迭代器改变元素
```

<u>必须记住，**一个map的value_type是一个pair，我们可以改变pair的值，但不能改变关键字成员的值。**</u>



#### set的迭代器是const的

**虽然set类型同时定义了iterator和const_iterator类型，但两种类型都只允许只读访问set中的元素。与不能改变一个map元素的关键字一样，一个set中的关键字也是const的。可以用一个set迭代器来读取元素的值，但不能修改：**

```c++
set<int> iset = {0,1,2,3,4,5,6,7,8,9};
set<int>::iterator set_it = iset.begin();
if (set_it != iset.end()){
	*set_it = 42;					//错误：set中的关键字是只读的
	cout << *set_it << endl;		//正确：可以读关键字
```

> map 迭代器不能改key的值，但可以改value的值
>
> set 迭代器不能改key的值，只能读

> map迭代器通过成员访问符获取元素
>
> set 迭代器通过解引用获取元素



#### 遍历关联容器

**map和set类型都支持表9.2（第295页）中的begin和end操作。**与往常一样，我们可以用这些函数获取迭代器，然后用迭代器来遍历容器。例如，我们可以编写一个循环来打印第375页中单词计数程序的结果，如下所示：

```c++
//获得一个指向首元素的迭代器
auto map_it = word_count.cbegin()
//比较当前迭代器和尾后迭代器
while (map_it != word_count.cend()){
    //解引用迭代器，打印关键字-值对
    cout << map_it->first << " occurs "
    	<< map_it->second << " times " << endl;
    ++map_it;				//递增迭代器，移动到下一个元素
}
```

while的循环条件和循环中的迭代器递增操作看起来很像我们之前编写的打印一个vector或一个string的程序。我们首先初始化迭代器map_it，让它指向word_count中的首元素。只要迭代器不等于end，就打印当前元素并递增迭代器。输出语句解引用map_it来获得pair的成员，否则与我们之前的程序一样。

<u>本程序的输出是按字典序排列的。**当使用一个迭代器遍历一个map、multimap、set或multiset时，迭代器按关键字升序遍历元素。**</u>



#### 关联容器和算法

**我们通常不对关联容器使用泛型算法（参见第10章）。关键字是const这一特性意味着不能将关联容器传递给修改或重排容器元素的算法**，因为这类算法需要向元素写入值，而set类型中的元素是const的，map中的元素是pair，其第一个成员是const的。

**关联容器可用于只读取元素的算法。但是，很多这类算法都要搜索序列。由于关联容器中的元素不能通过它们的关键字进行（快速）查找，因此对其使用泛型搜索算法几乎总是个坏主意。**例如，我们将在11.3.5节（第388页）中看到，关联容器定义了一个名为find的成员，它通过一个给定的关键字直接获取元素。我们可以用泛型find算法来查找一个元素，但此算法会进行顺序搜索。使用关联容器定义的专用的find成员会比调用泛型find快得多。

**在实际编程中，如果我们真要对一个关联容器使用算法，要么是将它当作一个源序列，要么当作一个目的位置。**例如，可以用泛型copy算法将元素从一个关联容器拷贝到另一个序列。类似的，可以调用inserter将一个插入器绑定（参见10.4.1节，第358页）到一个关联容器。通过使用inserter，我们可以将关联容器当作一个目的位置来调用另一个算法。



### 11.3.2　添加元素

![表11.4 关联容器insert操作](C:\Users\Grey\Desktop\C++ Primer\图片\表11.4 关联容器insert操作.png)

关联容器的insert成员（见表11.4，第384页）向容器中添加一个元素或一个元素范围。**由于map和set（以及对应的无序类型）包含不重复的关键字，因此插入一个已存在的元素对容器没有任何影响：**

```c++
vector<int> ivec = {2,4,6,8,2,4,6,8};			//ivec有8个元素
set<int> set2;									//空集合
set2.insert(ivec.cbegin(),ivec.cend());			//set2有4个元素
set2.insert({1,3,5,7,1,3,5,7});					//set2现在有8个元素
```

i**nsert有两个版本，分别接受一对迭代器，或是一个初始化器列表，**这两个版本的行为类似对应的构造函数（参见11.2.1节，第376页）——对于一个给定的关键字，只有第一个带此关键字的元素才被插入到容器中。



#### 向map添加元素

对一个map进行insert操作时，必须记住元素类型是pair。通常，对于想要插入的数据，并没有一个现成的pair对象。可以在insert的参数列表中创建一个pair：

```c++
//向word_count插入word的4种方法
word_count.insert({word, 1});
word_count.insert(make_pair(word, 1));
word_count.insert(pair<string, size_t>(word, 1));
word_count.insert(map<string, size_t>::value_type(word, 1));
```

如我们所见，在新标准下，创建一个pair最简单的方法是在参数列表中使用花括号初始化。也可以调用make_pair或显式构造pair。最后一个insert调用中的参数：

```c++
map<string, size_t>::value_type(s, 1)
```

构造一个恰当的pair类型，并构造该类型的一个新对象，插入到map中。



#### 检测insert的返回值

insert（或emplace）返回的值依赖于容器类型和参数。**对于不包含重复关键字的容器，添加单一元素的insert和emplace版本返回一个pair，告诉我们插入操作是否成功。pair的first成员是一个迭代器，指向具有给定关键字的元素；second成员是一个bool值，指出元素是插入成功还是已经存在于容器中。如果关键字已在容器中，则insert什么事情也不做，且返回值中的bool部分为false。如果关键字不存在，元素被插入容器中，且bool值为true。**

作为一个例子，我们用insert重写单词计数程序：

```c++
//统计每个单词在输入中出现次数的一种更烦琐的方法
map<string, size_t> word_count;			//从string到size_t的空map
string word;
while (cin >> word){
    //插入一个元素，关键字等于word,值为1；
    //若word已在word_count中，insert什么也不做
    auto ret = word_count.insert({ word,1 });
    if(!ret.second)						//word已在word_count中
    	++ret.first->second;			//递增计数器
}
```

对于每个word，我们尝试将其插入到容器中，对应的值为1。**若word已在map中，则什么都不做，特别是与word相关联的计数器的值不变。若word还未在map中，则此string对象被添加到map中，且其计数器的值被置为1。*(就第7行，不包括后面的代码)***

if语句检查返回值的bool部分，若为false，则表明插入操作未发生。在此情况下，word已存在于word_count中，因此必须递增此元素所关联的计数器。



#### 展开递增语句

在这个版本的单词计数程序中，递增计数器的语句很难理解。通过添加一些括号来反映出运算符的优先级（参见4.1.2节，第121页），会使表达式更容易理解一些：

```c++
++((ret.first)->second);			//等价的表达式
```

下面我们一步一步来解释此表达式：

ret 保存insert返回的值，是一个pair。

ret.first是pair的第一个成员，是一个map迭代器，指向具有给定关键字的元素。

ret.first-> 解引用此迭代器，提取map中的元素，元素也是一个pair。

ret.first->second map中元素的值部分。

++ret.first->second 递增此值。

再回到原来完整的递增语句，它提取匹配关键字word的元素的迭代器，并递增与我们试图插入的关键字相关联的计数器。

如果读者使用的是旧版本的编译器，或者是在阅读新标准推出之前编写的代码，ret的声明和初始化可能复杂些：

```c++
pair<map<string,size_t>::iterator, bool> ret = 
		word_count.insert(make_pair(word,1));
```

应该容易看出这条语句定义了一个pair，其第二个类型为bool类型。第一个类型理解起来有点儿困难，它是一个在map<string，size_t>类型上定义的iterator类型。



#### 向multiset或multimap添加元素

我们的单词计数程序依赖于这样一个事实：一个给定的关键字只能出现一次。这样，任意给定的单词只有一个关联的计数器。我们有时希望能添加具有相同关键字的多个元素。例如，可能想建立作者到他所著书籍题目的映射。在此情况下，每个作者可能有多个条目，因此我们应该使用multimap而不是map。由于一个multi容器中的关键字不必唯一，在这些类型上调用insert总会插入一个元素：

```c++
multimap<string, string> authors;
//插入第一个元素，关键字为Barth,John
authors.insert({"Barth,John","Sot-Weed Factor"});
//正确：添加第二个元素，关键字也是Barth,John
authors.insert({"Barth,John","Lost in the Funhouse"});
```

**对允许重复关键字的容器，接受单个元素的insert操作返回一个指向新元素的迭代器。这里无须返回一个bool值，因为insert总是向这类容器中加入一个新元素。**



### 11.3.3　删除元素

![表11.5 从关联容器删除元素](C:\Users\Grey\Desktop\C++ Primer\图片\表11.5 从关联容器删除元素.png)

关联容器定义了三个版本的erase，如表11.5所示。与顺序容器一样，我们可以通过传递给erase一个迭代器或一个迭代器对来删除一个元素或者一个元素范围。这两个版本的erase与对应的顺序容器的操作非常相似：指定的元素被删除，函数返回void。

关联容器提供一个额外的erase操作，它接受一个key_type参数。此版本删除所有匹配给定关键字的元素（如果存在的话），返回实际删除的元素的数量。我们可以用此版本在打印结果之前从word_count中删除一个特定的单词：

```c++
//删除一个关键字，返回删除的元素数量
if(word_count.erase(removal_word))
	cout << " ok: " << removal_word << " removed\n";
else cout << " oops: " << removal_word << " not found!\n";
```

对于保存不重复关键字的容器，erase的返回值总是0或1。若返回值为0，则表明想要删除的元素并不在容器中

对允许重复关键字的容器，删除元素的数量可能大于1：

```c++
auto cnt = authors.erase("Barth,John");
```

如果authors是我们在11.3.2节（第386页）中创建的multimap，则cnt的值为2。



### 11.3.4　map的下标操作

![表11.6 map和unordered._map的下标操作](C:\Users\Grey\Desktop\C++ Primer\图片\表11.6 map和unordered._map的下标操作.png)

**map和unordered_map容器提供了下标运算符和一个对应的at函数（参见9.3.2节，第311页 ），如表11.6所示。set类型不支持下标，因为set中没有与关键字相关联的“值”。**元素本身就是关键字，因此“获取与一个关键字相关联的值”的操作就没有意义了。**我们不能对一个multimap或一个unordered_multimap进行下标操作，因为这些容器中可能有多个值与一个关键字相关联。**

类似我们用过的其他下标运算符，map下标运算符接受一个索引（即，一个关键字），获取与此关键字相关联的值。但是，与其他下标运算符不同的是，**如果关键字并不在map中，会为它创建一个元素并插入到map中，关联值将进行值初始化**（参见3.3.1节，第88页）。

例如，如果我们编写如下代码

```c++
map <string, size_t> word_count;			//empty map
//插入一个关键字为Anna的元素，关联值进行值初始化；然后将1赋予它
word_count["Anna"] = 1;
```

将会执行如下操作：

· 在word_count中搜索关键字为Anna的元素，未找到。

· 将一个新的关键字-值对插入到word_count中。关键字是一个const string，保存Anna。值进行值初始化，在本例中意味着值为0。

· 提取出新插入的元素，并将值1赋予它。

由于下标运算符可能插入一个新元素，我们只可以对非const的map使用下标操作。



#### 使用下标操作的返回值

map的下标运算符与我们用过的其他下标运算符的另一个不同之处是其返回类型。通常情况下，**解引用一个迭代器所返回的类型与下标运算符返回的类型是一样的。但对map则不然：当对一个map进行下标操作时，会获得一个mapped_type对象；但当解引用一个map迭代器时，会得到一个value_type对象（参见11.3节，第381页）。**

**与其他下标运算符相同的是，map的下标运算符返回一个左值（参见4.1.1节，第121页）。由于返回的是一个左值，所以我们既可以读也可以写元素：**

```c++
cout << word_count["Anna"];			//用Anna作为下标提取元素；会打印出1
++word_count["Anna"]				//提取元素，将其增1
cout << word_count["Anna"];			//提取元素并打印它；会打印出2
```

**如果关键字还未在map中，下标运算符会添加一个新元素**，这一特性允许我们编写出异常简洁的程序，例如单词计数程序中的循环（参见11.1节，第375页）。另一方面，**有时只是想知道一个元素是否已在map中，但在不存在时并不想添加元素。在这种情况下，就不能使用下标运算符。**

> 当规则文件中存在多条规则转换相同单词时，下标+赋值的版本最终会用最后一条规则进行文本转换，而insert版本则会用第一条规则进行文本转换。



### 11.3.5　访问元素

![表11.7 在一个关联容器中查找元素的操作](C:\Users\Grey\Desktop\C++ Primer\图片\表11.7 在一个关联容器中查找元素的操作.png)

关联容器提供多种查找一个指定元素的方法，如表11.7所示。应该使用哪个操作依赖于我们要解决什么问题。**如果我们所关心的只不过是一个特定元素是否已在容器中，可能find是最佳选择。对于不允许重复关键字的容器，可能使用find还是count没什么区别。但对于允许重复关键字的容器，count还会做更多的工作：如果元素在容器中，它还会统计有多少个元素有相同的关键字。如果不需要计数，最好使用find：**

```c++
set<int> iset = { 0,1,2,3,4,5,6,7,8,9 };
iset.find(1);				//返回一个迭代器，指向key==1的元素
iset.find(11);				//返回一个迭代器，其值等于iset.end()
iset.count(1);				//返回1
iset.count(11);				//返回0
```



#### 对map使用find代替下标操作

对map和unordered_map类型，下标运算符提供了最简单的提取元素的方法。但是，如我们所见，使用下标操作有一个严重的副作用：如果关键字还未在map中，下标操作会插入一个具有给定关键字的元素。这种行为是否正确完全依赖于我们的预期是什么。例如，单词计数程序依赖于这样一个特性：使用一个不存在的关键字作为下标，会插入一个新元素，其关键字为给定关键字，其值为0。也就是说，下标操作的行为符合我们的预期。

但有时，我们只是想知道一个给定关键字是否在map中，而不想改变map。这样就不能使用下标运算符来检查一个元素是否存在，因为如果关键字不存在的话，下标运算符会插入一个新元素。在这种情况下，应该使用find：

```c++
if(word_count.find("foobar") == word_count.end())
	cout << "foobar is not in the map" << endl;
```



#### 在multimap或multiset中查找元素

在一个不允许重复关键字的关联容器中查找一个元素是一件很简单的事情——元素要么在容器中，要么不在。但对于允许重复关键字的容器来说，过程就更为复杂：**在容器中可能有很多元素具有给定的关键字。如果一个multimap或multiset中有多个元素具有给定关键字，则这些元素在容器中会相邻存储。**

例如，给定一个从作者到著作题目的映射，我们可能想打印一个特定作者的所有著作。可以用三种不同方法来解决这个问题。最直观的方法是使用find和count：

```c++
string search_item("Alain de Botton");				//要查找的作者
auto entries = authors.count(search_item);			//元素的数量
auto iter = authors.find(search_item);				//此作者的第一本书
//用一个循环查找此作者的所有著作
while(entries){
    cout << iter->second << endl;					//打印每个题目
    ++iter;											//前进到下一本书
    --entries;										//记录已经打印了多少本书
}
```

首先调用count确定此作者共有多少本著作，并调用find获得一个迭代器，指向第一个关键字为此作者的元素。for循环的迭代次数依赖于count的返回值。特别是，如果count返回0，则循环一次也不执行。



#### 一种不同的，面向迭代器的解决方法

我们还可以用lower_bound和upper_bound来解决此问题。这两个操作都接受一个关键字，返回一个迭代器。如果关键字在容器中，lower_bound返回的迭代器将指向第一个具有给定关键字的元素，而upper_bound返回的迭代器则指向最后一个匹配给定关键字的元素之后的位置。如果元素不在multimap中，则lower_bound和upper_bound会返回相等的迭代器——指向一个不影响排序的关键字插入位置。**因此，用相同的关键字调用lower_bound和upper_bound会得到一个迭代器范围（参见9.2.1节，第296页），表示所有具有该关键字的元素的范围。**

**当然，这两个操作返回的迭代器可能是容器的尾后迭代器。如果我们查找的元素具有容器中最大的关键字，则此关键字的upper_bound返回尾后迭代器。如果关键字不存在，且大于容器中任何关键字，则lower_bound返回的也是尾后迭代器。**

使用这两个操作，我们可以重写前面的程序：

```c++
//authors和search item的定义，与前面的程序一样
//beg和end表示对应此作者的元素的范围
for(auto beg = authors.lower_bound(search_item),
		end = authors.upper_bound(search_item); beg !=end; ++beg )	
	cout << beg->second << endl;		//打印每个题目
```

此程序与使用count和find的版本完成相同的工作，但更直接。对lower_bound的调用将beg定位到第一个与search_item匹配的元素（如果存在的话）。如果容器中没有这样的元素，beg将指向第一个关键字大于search_item的元素，有可能是尾后迭代器。upper_bound调用将end指向最后一个匹配指定关键字的元素之后的元素。这两个操作并不报告关键字是否存在，重要的是它们的返回值可作为一个迭代器范围（参见9.2.1节，第296页）。

如果没有元素与给定关键字匹配，则lower_bound和upper_bound会返回相等的迭代器——都指向给定关键字的插入点，能保持容器中元素顺序的插入位置。

假定有多个元素与给定关键字匹配，beg将指向其中第一个元素。我们可以通过递增beg来遍历这些元素。end中的迭代器会指出何时完成遍历——当beg等于end时，就表明已经遍历了所有匹配给定关键字的元素了。

由于这两个迭代器构成一个范围，我们可以用一个for循环来遍历这个范围。循环可能执行零次，如果存在给定作者的话，就会执行多次，打印出该作者的所有项。如果给定作者不存在，beg和end相等，循环就一次也不会执行。否则，我们知道递增beg最终会使它到达end，在此过程中我们就会打印出与此作者关联的每条记录。



#### equal_range函数

解决此问题的最后一种方法是三种方法中最直接的：不必再调用upper_bound和lower_bound，直接调用**equal_range即可。此函数接受一个关键字，返回一个迭代器pair。若关键字存在，则第一个迭代器指向第一个与关键字匹配的元素，第二个迭代器指向最后一个匹配元素之后的位置。若未找到匹配元素，则两个迭代器都指向关键字可以插入的位置。**

可以用equal_range来再次修改我们的程序：

```c++
//authors和search_item的定义，与前面的程序一样
//pos保存迭代器对，表示与关键字匹配的元素范围
for(auto pos = authors.equal_range(search_item);
		pos.first != pos.second; ++pos.first)
	cout << pos.first->second << endl;			//打印每个题目
```

此程序本质上与前一个使用upper_bound和lower_bound的程序是一样的。不同之处就是，没有用局部变量beg和end来保存元素范围，而是使用了equal_range返回的pair。此pair的first成员保存的迭代器与lower_bound返回的迭代器是一样的，second保存的迭代器与upper_bound的返回值是一样的。因此，在此程序中，pos.first等价于beg，pos.second等价于end。



### 11.3.6　一个单词转换的map

我们将以一个程序结束本节的内容，它将展示map的创建、搜索以及遍历。这个程序的功能是这样的：给定一个string，将它转换为另一个string。程序的输入是两个文件。第一个文件保存的是一些规则，用来转换第二个文件中的文本。每条规则由两部分组成：一个可能出现在输入文件中的单词和一个用来替换它的短语。表达的含义是，每当第一个单词出现在输入中时，我们就将它替换为对应的短语。第二个输入文件包含要转换的文本。

如果单词转换文件的内容如下所示：

`brb be right back`

`k okay?`

`y why`

`r are`

`u you`

`pic picture`

`thk thanks!`

`l8r later`

我们希望转换的文本为

`where r u`

`y dont u send me a pic`

`k thk l8r`

则程序应该生成这样的输出：

`where are you`

`why dont you send me a picture`

`okay? thanks! later`



#### 单词转换程序

我们的程序将使用三个函数。函数word_transform管理整个过程。它接受两个ifstream参数：第一个参数应绑定到单词转换文件，第二个参数应绑定到我们要转换的文本文件。函数buildMap会读取转换规则文件，并创建一个map，用于保存每个单词到其转换内容的映射。函数transform接受一个string，如果存在转换规则，返回转换后的内容。

我们首先定义word_transform函数。最重要的部分是调用buildMap和transform：

```c++
void word_transform(ifstream &map_file, ifstream &input)
{
    auto trans_map = buildMap(map_file);			//保存转换规则
    string text;									//保存输入中的每一行
    while (getline(input, text)){					//读取一行输入
        istringstream stream(text); 				//读取每个单词
        string word;
        bool firstword = true;						//控制是否打印空格
        while(stream >> word) {
            if (firstword)
                firstword = false;
            else
                cout << " ";						//在单词间打印一个空格
            //transform返回它的第一个参数或其转换之后的形式
            cout << transform(word, trans_map);		//打印输出
        }
        cout << endl;								//完成一行的转换
    }
}
```

函数首先调用buildMap来生成单词转换map，我们将它保存在trans_map中。函数的剩余部分处理输入文件。while循环用getline一行一行地读取输入文件。这样做的目的是使得输出中的换行位置能和输入文件中一样。为了从每行中获取单词，我们使用了一个嵌套的while循环，它用一个istringstream（参见8.3节，第287页）来处理当前行中的每个单词。

在输出过程中，内层while循环使用一个bool变量firstword来确定是否打印一个空格。它通过调用transform来获得要打印的单词。transform的返回值或者是word中原来的string，或者是trans_map中指出的对应的转换内容。



#### 建立转换映射

函数buildMap读入给定文件，建立起转换映射。

```c++
map<string,string> buildMap(ifstream &map_file)
{
    map<string, string> trans_map;				//保存转换规则
    string key; 								//要转换的单词
    string value;								//替换后的内容
    //读取第一个单词存入key中，行中剩余内容存入value
    while (map_file >> key && getline(map_file, value))
        if(value.size() > 1)					//检查是否有转换规则
        	trans_map[key] = value.substr(1);	//跳过前导空格
        else
        	throw runtime_error("no rule for ", key);
    return trans_map;
}
```

map_file中的每一行对应一条规则。每条规则由一个单词和一个短语组成，短语可能包含多个单词。我们用>>读取要转换的单词，存入key中，并调用getline读取这一行中的剩余内容存入value。由于getline不会跳过前导空格（参见3.2.2节，第78页），需要我们来跳过单词和它的转换内容之间的空格。在保存转换规则之前，检查是否获得了一个以上的字符。如果是，调用substr（参见9.5.1节，第321页）来跳过分隔单词及其转换短语之间的前导空格，并将得到的子字符串存入trans_map。

注意，我们使用下标运算符来添加关键字-值对。我们隐含地忽略了一个单词在转换文件中出现多次的情况。如果真的有单词出现多次，循环会将最后一个对应短语存入trans_map。当while循环结束后，trans_map中将保存着用来转换输入文本的规则。



#### 生成转换文本

函数transform进行实际的转换工作。其参数是需要转换的string的引用和转换规则map。如果给定string在map中，transform返回相应的短语。否则，transform直接返回原string：

```c++
const string transform(const string &s, const map<string,string> &m)
{
    //实际的转换工作；此部分是程序的核心
    auto map_it = m.find(s);				
    
    //如果单词在转换规则map中
    if (map_it != m.cend())
    	return map_it->second;			//使用替换短语
    else
    	return s;						//否则返回原string
}
```

函数首先调用find来确定给定string是否在map中。如果存在，则find返回一个指向对应元素的迭代器。否则，find返回尾后迭代器。如果元素存在，我们解引用迭代器，获得一个保存关键字和值的pair（参见11.3节，第381页），然后返回成员second，即用来替代s的内容。



## 11.4　无序容器

新标准定义了4个无序关联容器（unordered associative container）。这些容器不是使用比较运算符来组织元素，而是使用一个哈希函数（hash function）和关键字类型的 == 运算符。在关键字类型的元素没有明显的序关系的情况下，无序容器是非常有用的。在某些应用中，维护元素的序代价非常高昂，此时无序容器也很有用。

虽然理论上哈希技术能获得更好的平均性能，但在实际中想要达到很好的效果还需要进行一些性能测试和调优工作。因此，使用无序容器通常更为简单（通常也会有更好的性能）。

<u>如果关键字类型固有就是无序的，或者性能测试发现问题可以用哈希技术解决，就可以使用无序容器。</u>



#### 使用无序容器

**除了哈希管理操作之外，无序容器还提供了与有序容器相同的操作（find、insert等）。这意味着我们曾用于map和set的操作也能用于unordered_map和unordered_set。类似的，无序容器也有允许重复关键字的版本。**

因此，通常可以用一个无序容器替换对应的有序容器，反之亦然。但是，由于元素未按顺序存储，一个使用无序容器的程序的输出（通常）会与使用有序容器的版本不同。

例如，可以用unordered_map重写最初的单词计数程序（参见11.1节，第375页）：

```c++
//统计出现次数，但单词不会按字典序排列
unordered_map<string, size_t> word_count;
string word;
while (cin >> word)
	++word_count[word]					//提取并递增word的计数器
for(const auto &w : word_count)			//对map中的每个元素
    //打印结果
    cout << w.first << "occurs " << w.second
    		<< ((w.second > 1) ? " times":" time" ) << endl;
```

此程序与原程序的唯一区别是word_count的类型。如果在相同的输入数据上运行此版本，会得到这样的输出：

`containers.occurs 1 time`

`use occurs 1 time`

`can occurs 1 time`

`examples occurs 1 time`

`...`

对于每个单词，我们将得到相同的计数结果。但单词不太可能按字典序输出。



#### 管理桶

无序容器在存储上组织为一组桶，每个桶保存零个或多个元素。**无序容器使用一个哈希函数将元素映射到桶。**为了访问一个元素，容器首先计算元素的哈希值，它指出应该搜索哪个桶。容器将具有一个特定哈希值的所有元素都保存在相同的桶中。如果容器允许重复关键字，所有具有相同关键字的元素也都会在同一个桶中。因此，无序容器的性能依赖于哈希函数的质量和桶的数量和大小。

对于相同的参数，哈希函数必须总是产生相同的结果。理想情况下，哈希函数还能将每个特定的值映射到唯一的桶。但是，将不同关键字的元素映射到相同的桶也是允许的。当一个桶保存多个元素时，需要顺序搜索这些元素来查找我们想要的那个。计算一个元素的哈希值和在桶中搜索通常都是很快的操作。但是，如果一个桶中保存了很多元素，那么查找一个特定元素就需要大量比较操作。

无序容器提供了一组管理桶的函数，如表11.8所示。这些成员函数允许我们查询容器的状态以及在必要时强制容器进行重组。

![表11.8 无序容器管理操作](C:\Users\Grey\Desktop\C++ Primer\图片\表11.8 无序容器管理操作.png)



#### 无序容器对关键字类型的要求

默认情况下,  无序容器使用关键字类型的==运算符来比较元素，它们还使用一个hash<key_type> 类型的对象来生成每个元素的哈希值。**标准库为内置类型（包括指针）提供了hash模板。还为一些标准库类型，包括string和我们将要在第12章介绍的智能指针类型定义了hash。因此，我们可以直接定义关键字是内置类型（包括指针类型）、string还是智能指针类型的无序容器。**

**但是，我们不能直接定义关键字类型为自定义类类型的无序容器。与容器不同，不能直接使用哈希模板，而必须提供我们自己的hash模板版本。**我们将在16.5节（第626页）中介绍如何做到这一点。

我们不使用默认的hash，而是使用另一种方法，类似于为有序容器重载关键字类型的默认比较操作（参见11.2.2节，第378页）。**为了能将Sale_data用作关键字，我们需要提供函数来替代==运算符和哈希值计算函数。**我们从定义这些重载函数开始：

```c++
size_t hasher(const Sales_data &sd)
{
	return hash<string>()(sd.isbn());
}

bool egop(const Sales_data &lhs, const Sales_data &rhs)
{
	return lhs.isbn() == rhs.isbn();
}
```

我们的hasher函数使用一个标准库hash类型对象来计算ISBN成员的哈希值，该hash类型建立在string类型之上。类似的，eqop函数通过比较ISBN号来比较两个Sales_data。

我们使用这些函数来定义一个unordered_multiset

```c++
using SD_multiset = unordered_multiset<Sales_data,
					decltype(hasher) *, decltype(egop) *>;
//参数是桶大小、哈希函数指针和相等性判断运算符指针
SD_multiset_bookstore(42, hasher, eqop);
```

为了简化bookstore的定义，首先为unordered_multiset定义了一个类型别名（参见2.5.1节，第60页），此集合的哈希和相等性判断操作与hasher和eqop函数有着相同的类型。通过使用这种类型 ，在定义bookstore时可以将我们希望它使用的函数的指针传递给它。

**如果我们的类定义了==运算符，则可以只重载哈希函数：**

```c++
//使用FooHash生成哈希值；Foo必须有==运算符
unordered_set<Foo, decltype(FooHash)*> fooSet(10,FooHash);
```

