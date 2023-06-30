---
layout: post
title: "C++11中引入的bind绑定器和function函数对象"
date: 2023-06-19
excerpt: "C++11中引入的bind绑定器和function函数对象"
tags: [cpp]
comments: true







---

* toc
{:toc}










C++语法我是真的弱，所以这一段再把基础知识学一学吧，因为本来要做项目的，但是总是被语法给难倒，果真这还是基础和内功

# 1.bind1st和bind2nd什么时候会用到

```cpp
#include <iostream>
#include <vector>
#include <vector>
#include <functional>
#include <algorithm>
#include <ctime>

using namespace std;
template <typename Container>
void showContainer(Container& con)
{
	typename Container::iterator it = con.begin();
	for (; it < con.end(); ++it)
	{
		cout << *it << " ";
	}
	cout << endl;
}

using namespace std;
int main()
{
	vector<int> vec;
	srand(time(nullptr));
	for (int i = 0; i < 20; i++)
	{
		vec.push_back(rand() % 100 + 1);
	}
	showContainer(vec);
	sort(vec.begin(), vec.end()); //默认从小到大排序
	showContainer(vec);

	//greater 二元函数对象
	//一般排序默认都是less，这个可以看源代码
	sort(vec.begin(), vec.end(), greater<int>()); //大到小
	showContainer(vec);

	/*
	把70按顺序插入到vec容器中，找第一个小于70的数字
	operator()(const T &val)
	greater a > b;
	less a < b;
	绑定器 + 二元函数对象 => 一元函数对象
	bind1st: + greater bool operator()(70, const _Ty& _Right)
	bind2nd: + less bool operator()(const _Ty& _Left, 70)
	*/
	auto it1 = find_if(vec.begin(), vec.end(), bind1st(greater<int>(), 70));
	//或者这样也行
	auto it1 = find_if(vec.begin(), vec.end(), bind2nd(less<int>(), 70));
	/*
	它使用了bind2nd函数适配器来将二元函数对象less<int>转换为一个一元谓词。
	less<int>是一个二元函数对象，它接受两个int类型的参数并返回它们的比较结果（即第一个参数是否小于第二个参数）。
	而bind2nd(less<int>(), 70)则将less<int>的第二个参数绑定为常量70
	从而得到一个一元谓词，它接受一个参数并返回该参数是否小于70。
	这是C++98中的，现在已经舍弃了
	*/
	if (it1 != vec.end())
	{
		vec.insert(it1, 70);
	}
	showContainer(vec);
}
```

# 2.bind1st和bind2nd底层原理

```cpp
#include <iostream>
#include <vector>
#include <vector>
#include <functional>
#include <algorithm>
#include <ctime>

using namespace std;
template <typename Container>
void showContainer(Container& con)
{
	typename Container::iterator it = con.begin();
	for (; it < con.end(); ++it)
	{
		cout << *it << " ";
	}
	cout << endl;
}

template <typename Iterator, typename Compare>
Iterator my_find_if(Iterator first, Iterator last, Compare comp)
{
	for (; first != last; first++)
	{
		if (comp(*first))	//com.opertor()(*first)相当于调用了小括号运算符重载
		{
			return first;
		}
	}
	return last;
}

template<typename Compare, typename T>
class _mybind1st // 绑定器是函数对象的一个应用
{
public:
	_mybind1st(Compare comp, T val)
		:_comp(comp), _val(val)
	{}
	bool operator()(const T& second)
	{
		return _comp(_val, second); // greater
	}
private:
	Compare _comp;
	T _val;
};

//mybind1st(greater<int>(), 70)
template<typename Compare, typename T>
_mybind1st<Compare, T> mybind1st(Compare comp, const T& val)
{
	//直接使用函数模板，好处是，可以进行类型的推演
	return _mybind1st<Compare, T>(comp, val);
}




using namespace std;
int main()
{
	vector<int> vec;
	srand(time(nullptr));
	for (int i = 0; i < 20; i++)
	{
		vec.push_back(rand() % 100 + 1);
	}
	showContainer(vec);
	sort(vec.begin(), vec.end()); //默认从小到大排序
	showContainer(vec);

	//greater 二元函数对象
	//一般排序默认都是less，这个可以看源代码
	sort(vec.begin(), vec.end(), greater<int>()); //大到小
	showContainer(vec);

	/*
	把70按顺序插入到vec容器中，找第一个小于70的数字
	operator()(const T &val)
	greater a > b;
	less a < b;
	绑定器 + 二元函数对象 => 一元函数对象
	bind1st: + greater bool operator()(70, const _Ty& _Right)
	bind2nd: + less bool operator()(const _Ty& _Left, 70)
	*/
	auto it1 = my_find_if(vec.begin(), vec.end(), mybind1st(greater<int>(), 70));
	//或者这样也行
	//auto it1 = my_find_if(vec.begin(), vec.end(), bind2nd(less<int>(), 70));
	/*
	它使用了bind2nd函数适配器来将二元函数对象less<int>转换为一个一元谓词。
	less<int>是一个二元函数对象，它接受两个int类型的参数并返回它们的比较结果（即第一个参数是否小于第二个参数）。
	而bind2nd(less<int>(), 70)则将less<int>的第二个参数绑定为常量70
	从而得到一个一元谓词，它接受一个参数并返回该参数是否小于70。
	这是C++98中的，现在已经舍弃了
	*/
	if (it1 != vec.end())
	{
		vec.insert(it1, 70);
	}
	showContainer(vec);
}
```

# 3.function函数对象类型的应用示例

## 3.1.function的应用



```cpp
#include <iostream>
#include <vector>
#include <vector>
#include <functional>
#include <algorithm>
#include <ctime>

using namespace std;

void hello1()
{
	cout << "hello" << endl;
}

void hello2(string s)
{
	cout << s << endl;
}

int sum(int a, int b)
{
	cout << a + b << endl;
	return a + b;
}

class Test
{
public:	//类的成员函数调用必须依赖一个对象void (Test::::*pfunc)(string)
	void hello(string str) { cout << str << endl; }
};
int main()
{
	// 从function的类模板定义处，看到希望用一个函数类型实例化function
	function<void()> func1 = hello1;
	func1();
	function<void(string)> func2 = hello2;
	func2("hello2");
	function<int(int, int)> func3 = sum;
	func3(1, 2);

	// operator()
	function<int(int, int)> func4 = [](int a, int b)->int {return a + b; };
	cout << func4(100, 200) << endl;

	// 因为成员函数的参数里面都有用一个指向自己的指针，也就是Test*
	function<void(Test*, string)> func5 = &Test::hello;
	// 对于成员方法的调用，是要指向一个对象的，也就是&Test()
	Test t;
	func5(&t, "hello test!");
	return 0;
}
// 总结
//1.function<函数类型>，用函数类型实例化function
//2.function<函数类型(参数)>  func1  使用时候也是得func1(参数)
```

## 3.2.function 的好处

那说了这么多，function的好处到底是哪里呢，为什么我要这样子干，而不是直接调用函数呢

好处就是它能把看到的各种类型保存下来

比如

func1 = hello1;

func4 = [](int a, int b)->int {return a + b; };

func5 = &Test::hello;

它能把函数，lambda表达式，成员函数的类型保存下来

举个例子，比如图书馆里系统图

```cpp
int main()
{
	int choice = 0;
	for (;;)
	{
		cout << "----------------" << endl;
		cout << "1.查看所有书籍信息" << endl;
		cout << "2.借书" << endl;
		cout << "3.还书" << endl;
		cout << "4.查询书籍" << endl;
		cout << "5.注销" << endl;
		cout << "----------------" << endl;
		cout << "请选择：" << endl;
		cin >> choice;
	}
	switch (choice)
	{
	case 1:
		break;
	case 2:
		break;
	case 3:
		break;
	case 4:
		break;
	case 5:
		break;
	default:
		break;
	}

}
```

但是这个代码不好，因为无法闭合，无法做到开闭原则，每次新增加一个功能 都要把switch里面的都改一下

我们可以把每件事情的功能都封装到一个函数里

```cpp
void doShowAllBook() { cout << "查看所有书籍信息" << endl; }
void doBorrow() { cout << "借书" << endl; }
void doBack() { cout << "还书" << endl; }
void doQueryBooks() { cout << "查询书籍" << endl; }
void doLoginOut() { cout << "注销" << endl; }
int main()
{
	int choice = 0;
	map <int, function<void()>> actionMap;
	actionMap.insert({ 1, doShowAllBook });
	actionMap.insert({ 2, doBorrow });
	actionMap.insert({ 3, doBack });
	actionMap.insert({ 4, doQueryBooks });
	actionMap.insert({ 5, doLoginOut });

	for (;;)
	{
		cout << "----------------" << endl;
		cout << "1.查看所有书籍信息" << endl;
		cout << "2.借书" << endl;
		cout << "3.还书" << endl;
		cout << "4.查询书籍" << endl;
		cout << "5.注销" << endl;
		cout << "----------------" << endl;
		cout << "请选择：" << endl;
		cin >> choice;
		auto it = actionMap.find(choice);
		if (it == actionMap.end())
		{
			cout << "failed" << endl;
		}
		else
		{
			it->second();
		}
	}
}
```

有人说那函数指针也可以做到呀，但是我们这里只是用了函数，那要是有lambda，有函数绑定怎么办呢，对吧，所以还是用function靠谱，可以返回任何的类型。

像lambda表达式只能作用在语句中，而有了function就可以随心所欲的用了，要不然就得重新写表达式或者重新绑定了

# 4.模板的完全特例化和部分特例化

```cpp
template<typename T>
bool compare(T a , T b)
{
	//cout << "template compare" << endl;
	return a > b;
}

int main()
{
	compare(10, 20);
	compare("aa", "bb");
	return 0;
}
```

像这段代码，10，20可以比较，但是aa和bb不能比较，到时候传入的只是他们的地址

所以编译器自动推演的类型，不符合我们实际处理的逻辑，所以我们需要自己特例化下

所以需要

```cpp
template<>
bool compare<const char*>(const char* a, const char* b)
{
	return strcmp(a, b) > 0;
}
```

这样才能比较，这叫做完全特例化，因为你看所有的都是compare<const char*>是完全已知的

template<> 是一个模板特化的语法。它允许你为特定的模板参数类型提供特定的实现

strcmp 是一个 C 标准库函数，用于比较两个 C 风格字符串（以空字符结尾的字符数组）。它按字典顺序比较两个字符串，并返回一个整数，表示两个字符串的相对顺序。如果第一个字符串小于第二个字符串，则返回值小于零；如果两个字符串相等，则返回零；如果第一个字符串大于第二个字符串，则返回值大于零

再看例子

```cpp
template<typename T>
class Vector
{
public:
	Vector() { cout << " call Vecotr template init" << endl; }
};

// 完全特例化，因为template<>
template<>
class Vector<char*>
{
public:
	Vector() { cout << " call Vecotr<char*> template init" << endl; }
};

// 我们知道要传入一个指针类型，但不知道具体是什么指针，所以用部分特例化
template<typename Ty>// 这是正常
class Vector<Ty*> // 在这里具体下，所以叫做部分特例化
{
public:
	Vector() { cout << " call Vecotr<Ty*> template init" << endl; }
};

// 部分特例化
// 指针函数指针，（有一个返回值，有两个形参变量）提供的部分特例化
template<typename R, typename A1, typename A2>
class Vector<R(*)(A1, A2)>
{
public:
	Vector<R(*)(A1, A2)>() { cout << " call Vecotr<R(*)(A1, A2)> template init" << endl; }
};

//函数类型部分特例化，（有一个返回值，有两个形参变量）
template<typename R, typename A1, typename A2>
class Vector<R(A1, A2)>
{
public:
	Vector<R(A1, A2)>() { cout << " call Vecotr<R(A1, A2)> template init" << endl; }
};

int sum(int a, int b) { return a + b; }

int main()
{
	Vector<int> vec1;
	Vector<char*> vec2;
	Vector<int*> vec3;
	Vector<int(*)(int, int)> vec4;//函数指针类型
	Vector<int(int, int)> vec5;//函数类型

	// 注意区分函数类型和函数指针类型
	// 函数指针类型
	typedef int(*Test)(int, int);
	Test pfunc = sum;
	cout << pfunc(10, 20) << endl;

	// 函数类型
	typedef int Test1(int, int); 
	Test1 *pfunc1 = sum; 
	cout << pfunc1(10, 20) << endl;
	// 表示 pfunc1 是一个指向 Test1 类型函数的指针，它被初始化为指向 sum 函数。
	// 由于函数名会自动转换为指向该函数的指针，因此可以直接将 sum 赋值给 pfunc1
	// 但是，由于 pfunc1 是一个指针，所以需要在其类型前加上 * 来表示它是一个指针。

	return 0; 
}
```

# 5.模板的实参推演

```cpp
template<typename T>
void func1(T) { cout << typeid(T).name() << endl; }

int sum(int a, int b) { return a + b; }

int main()
{
	func1(10);
	func1("aaa");
    func1(sum);
	return 0; 
}
```

输出结果是

```cpp
int
char const * __ptr64
int (__cdecl*)(int,int)
```

但是目前的问题是T是把所有的类型全一股脑搞出来了，那如果我们只想让他一个个出来怎么办呢

也有办法，看代码

```cpp
template<typename T>
void func1(T a) { cout << typeid(T).name() << endl; } 

template<typename R, typename A, typename B>
void func2(R(*a)(A, B))
{
	cout << typeid(R).name() << endl;
	cout << typeid(A).name() << endl;
	cout << typeid(B).name() << endl;
}

int sum(int a, int b) { return a + b; }

int main()
{
	func1(10);
	func1("aaa");
	func1(sum);
	func2(sum);
	return 0; 
}
```

上面的a写不写无所谓的，反正只是表明是T类型而已的一个形参

输入结果为

```cpp
int
char const * __ptr64
int (__cdecl*)(int,int)
int
int
int
```

这样的话sum的各个小块类型我们也可以得到，通过特例化

甚至我可以用下面的

```cpp
class Test
{
public:
	int sum(int a, int b) { return a + b; }
};
func1( &Test::sum);
```

输出为

```cpp
int (__cdecl Test::*)(int,int) __ptr64
```

也是个大类型，那我们怎么把它拆分下呢，写个func3

```cpp
template<typename R, typename T,typename A, typename B>
void func3(R(T::*a)(A, B))
{
	cout << typeid(R).name() << endl;
	cout << typeid(T).name() << endl;
	cout << typeid(A).name() << endl;
	cout << typeid(B).name() << endl;
}
func3(&Test::sum);
```

输出为

```cpp
int
class Test
int
int
```

所以模板实参可以把复杂类型一个个取出来，这就是它的魅力所在

#  6.function的实现原理

```cpp
void hello(string str) { cout << str << endl; }
int sum(int a, int b) { return a + b ; }
//类模板的声明
template<typename Fty>
class myfunction{};

template<typename R, typename A1>
class myfunction<R(A1)>
{
public:
	using PFUNC = R(*)(A1);
	myfunction(PFUNC pfunc) : _pfunc(pfunc){}
	R operator()(A1 arg)
	{
		return _pfunc(arg);
	}
private:
	PFUNC _pfunc;
};

template<typename R, typename A1, typename A2>
class myfunction<R(A1, A2)>
{
public:
	using PFUNC = R(*)(A1, A2);
	myfunction(PFUNC pfunc) : _pfunc(pfunc) {}
	R operator()(A1 arg1, A2 arg2)
	{
		return _pfunc(arg1, arg2);
	}
private:
	PFUNC _pfunc;
};


int main()
{
	myfunction<void(string)> func1(hello);
	func1("hello world!"); //func1.operator()("hello world!")
	myfunction<int(int, int)> func2(sum);
	cout << func2(10, 20) << endl;
	return 0;
}
```

有人会问，那总不能每次是一个新的函数，就重新写一个模板类吧

当然不会，咱们看R也就是void int，这个是确定的，一个R就可以表示，只不过是函数的参数有的是一个有的是两个，目前解决这个问题就行

```cpp
template<typename R, typename... A> //这表示A不是一个类型，是一组类型，一堆一堆的
class myfunction<R(A...)> //这表示形参的个数是可变的
    {
public:
	using PFUNC = R(*)(A...);
	myfunction(PFUNC pfunc) : _pfunc(pfunc){}
	R operator()(A... arg)
	{
		return _pfunc(arg...); //这表示一组参数
	}
private:
	PFUNC _pfunc;
};
```

把这个替换掉刚才的两个没有一点问题

这就是function底层实现的原理

...这个是C++11提供的可变参类型参数

其中上面的类模板声明不能删除，如果删除了 `template<typename Fty> class myfunction{};` 这一行代码，那么在定义特化版本的 `myfunction` 类模板时，编译器将无法找到 `myfunction` 类模板的原始声明。这将导致编译错误。因此，即使没有直接使用 `template<typename Fty> class myfunction{};` 这个类模板，它仍然是必需的，因为它为特化版本的 `myfunction` 类模板提供了原始声明。

# 7.Bind

std::bind绑定器，也是个类模板，C++11引入的

std::bind能够将对象以及相关的参数绑定到一起，绑定完后可以直接调用，也可以用std::function进行保存，再需要的调用

格式：

std::bind（待绑定的函数对象/函数指针/成员函数指针，参数绑定值1，参数绑定值2...参数绑定值n）

总结：

a)将可调用对象和函数绑定在一起，构成一个仿函数，所以可以直接调用

b)如果函数有多个参数，可以绑定一部分参数，其他参数在调用的时候指定





直接调用

```cpp
void hello(string str) { cout << str << endl; }

int main()
{
	std::bind(hello, "china")();
}
```



间接调用

```cpp
void hello(string str) { cout << str << endl; }

int main()
{
	auto test = std::bind(hello, "china");
	return 0;
}
```

占位符placeholders::_1表示这个位置（当前placeholders::\_1）将在函数调用时，被传入的第一个参数

