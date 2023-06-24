---
layout: post
title: "【一文读不懂C++】论function和bind"
date: 2023-06-24
excerpt: "论function和bind"
tags: [cpp]
comments: true









---

* toc
{:toc}












# 1.可调用对象

## 1.1.函数指针

```cpp
void myfunc(int tv)
{
    cout << "myfunc()函数执行了, tv = " << tv << endl;
}
int main()
{
    void(*pmf)(int) = &myfunc;//定义一个函数指针pmf并给了初值
    pmf(15);//调用函数,这就是可调用对象
}
```



## 1.2.具有operator()成员函数的类对象（仿函数）

仿函数的定义：仿函数（functor），它的行为类似于函数的东西（something that performs a function)

C++仿函数是通过在类中重载（）运算符实现，又称函数对象（function object）：能行使函数功能的类

```cpp
class TC()
{
    public:
    	void operator(int a)
        {
            cout << "类执行了" << endl;
        }
};
int main()
{
    TC tc;
    tc(20); //调用的是()操作符，也是个可调用对象
    //等价于 tc.operator()(20);
}
```



## 1.3.可被转换为函数指针的类对象（也可以叫做仿函数/函数对象）

```cpp
class TC2
{
public:
    using tfpoint = void(*)(int); //定义一个函数指针类型
    static void mysfunc(int tv) //静态成员函数
    {
        cout << "TC2::mysfunc()静态成员函数执行了，tv = " << tv << endl;
    }
    operator tfpoint() { return mysfunc; }
};

int main()
{
    TC2 tc2;
    tc2(50);//先调用tfpoint，再调用mysfunc：这也是个可调用对象
    //等价于tc2.operator   TC2::tfpoint2()(50);
}
```

其中operator tfpoint() { return mysfunc; }是类型转换函数，它可以将一个类类型的对象转换为另一种类型，这里operator tfpoint() 是将一个T2类型的对象转为一个函数指针类型，所以tc2(50)调用了由类型转换函数返回的函数指针，并将参数50传给了他，这等价于直接调用了TC2::mysfunc(50)

换个角度看，其实本来是TC2::operator()()，现在只不过是TC2::tfpoint2()(50)，而TC2::tfpoint2()返回的是mysfunc，所以TC2::tfpoint2()(50)也就是mysfunc(50)

## 1.4.类成员函数指针

```cpp
class TC()
{
    public:
    	void operator(int a)
        {
            cout << "TC::operator()执行了，tv= " << endl;
        }
    void ptfunc(int tv)
    {
        cout << "TC::ptfunc()执行了，tv= " << tv << endl;
    }
};
int main()
{
    Tc tc;
    void (TC::*myfpointpt)(int) = &TC::ptfunc; //类成员函数指针变量myfpoint定义并给初值
    (tc.*myfpointpt)(68); //也是一个可调用对象
}
```

可以看函数指针，加强对类成员函数指针的理解，其实都一样

## 1.5.总结

a)都看做对象

b)我们可以对其使用（）调用运算符，如果a是可调用对象，那么我们就可以编写a(param...)代码

如何能把各种不同的可调用对象的形式统一一下，统一的目的是为了方便咱们调用

这就会引入了function

# 2.std::function（可调用对象包装器） C++11

std::function是个类模版，用来装各种可调用对象，但不能装类成员函数指针

std::function类模版的特点，就是能够通过给它指定模版函数，它就能够用统一的方式来处理函数

## 2.1绑定普通函数

```cpp
void myfunc(int tv)
{
    cout << "myfunc()函数执行了, tv = " << tv << endl;
}

int main()
{
    std::function<void(int)> f1 = myfunc;
    f1(100);
}
```

## 2.2 绑定类的静态成员函数

```cpp
class TC()
{
    public:
    	void operator(int a)
        {
            cout << "TC::operator()执行了，tv= " << endl;
        }
    void ptfunc(int tv)
    {
        cout << "TC::ptfunc()执行了，tv= " << tv << endl;
    }
    static int stcfunc(int tv)
    {
        cout << "TC::stcfunc()静态函数执行了，tv = " << tv << endl;
        return tv;
    }
};
int main()
{
    std::function<int(int)> fs2 = TC::stcfunc; //绑定一个类的静态成员函数
    cout << fs2(110) << endl;
}
```

## 2.3 绑定仿函数

```cpp
class TC()
{
    public:
    	void operator(int a)
        {
            cout << "TC::operator()执行了，tv= " << endl;
        }
    void ptfunc(int tv)
    {
        cout << "TC::ptfunc()执行了，tv= " << tv << endl;
    }
    static int stcfunc(int tv)
    {
        cout << "TC::stcfunc()静态函数执行了，tv = " << tv << endl;
        return tv;
    }
};
class TC2
{
public:
    using tfpoint = void(*)(int); //定义一个函数指针类型
    static void mysfunc(int tv) //静态成员函数
    {
        cout << "TC2::mysfunc()静态成员函数执行了，tv = " << tv << endl;
    }
    operator tfpoint() { return mysfunc; }
};
int main()
{
    TC tc3;
    std::function<int(int)> f3 = tc3;
    f3(120);//调用operator
    
    TC2 tc4;
    std::function<void(int)> f4 = tc4;
    f4(150);
}
```

## 2.4 小范例演示

范例1

```cpp
class CB
{
    std::function<void()> fcallback;
public:
    CB(const std::function<void()>&f) :fcallback(f)  //&f引用
    {
        int i;
        i = 1;
    }
    void runcallback(void)
    {
        fcallback();
    }
};

class CT
{
public:
    void operator()(void)
    {
        cout << "operator()执行" << endl;
    }
};

int main()
{
    CT ct; // 可调用对象
    CB cb(ct); // cb需要可调用对象做参数来构造，ct因为有operator()所以可转化为std:function<void()>&)对象
    cb.runcallback()// 执行的CT里的operator()
}
```

输出结果是  operator()执行

范例2

```cpp
void mycallback(int cs, const std::function<void(int)> &f)
{
    f(cs);
}
void runfunc(int x)
{
    cout << x << endl;
}
int main()
{
	mycallback(1, runfunc); //runfunc(1)
}
```

# 3.std::bind绑定器，也是个类模版，C++11引入的

std::bind能够将对象以及相关的参数绑定到一起，绑定完后可以直接调用，也可以用std::function进行保存，再需要的调用

格式：

std::bind（待绑定的函数对象/函数指针/成员函数指针，参数绑定值1，参数绑定值2...参数绑定值n）

总结：

a)将可调用对象和函数绑定在一起，构成一个仿函数，所以可以直接调用

b)如果函数有多个参数，可以绑定一部分参数，其他参数在调用的时候指定

## 3.1绑定普通函数

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

```cpp
void func(int x, int y, int z)
{
    cout << x << y << z << endl;
}
int main()
{
    auto bf2 = std::bind(func, placeholders::_1, placeholders::_2, 30);
    bf2(5, 15); //输出是5,15,30
}
```

这表示绑定函数func的第三个参数为30，func的第一个和第二个参数分别由调用bf2时的第一二个参数指定

_1是标准库里定义的，占位符的含义，类似这样的占位符有20个（看源代码），足够咱们用了

```cpp
void func(int x, int y, int z)
{
    cout << x << y << z << endl;
}
int main()
{
    auto bf2 = std::bind(func, placeholders::_2, placeholders::_1, 30);
    bf2(5, 15); // 输出是15,5,30
}
```

注意接下来有个坑

```cpp
void func(int &x, int &y)
{
    x++;
    y++;
}
int main()
{
	int a = 2;
    int b = 3;
    auto bf4 = std::bind(func, a, placeholders::_1);
    bf4(b);
    cout << a << b << endl; // a = 2, b = 4
}
```

这说明，bind对于预先绑定的函数参数是通过值传递的，所以这个a实际是值传递

bind对于不事先绑定的参数，通过std::placeholder传递的参数，是通过引用传递的，所以b实际是引用传递

## 3.2bind怎么绑定成员函数

```cpp
class CT
{
public:
    void func()(int x, int y)
    {
        cout << x << y << endl;
        m_a = x;
    }
    int m_a = 0;
};

int main()
{
	CT ct;
    auto bf5 = std::bind(&CT::func, ct, std::placeholders::_1, std::placeholders::_2);
    bf5(10,20);
}
```

注意对成员函数bind，第二个参数不再是函数里的参数了，而是对象

发现确实输出的是10,20，但是调试过程你会发现ct里面的m_a值是0，而不是x的值10，这是为什么呢

这是auto bf5 = std::bind(&CT::func, ct, std::placeholders::\_1, std::placeholders::_2);里面的第二个参数ct，会导致调用CT的拷贝构造函数来生成一个CT类型的临时对象，作为std::bind的返回值（bind返回仿函数类型对象），后续的func调用修改的是临时对象的m_a值，并不影响真实的ct对象的m_a值

所以换个写法用引用

```cpp
auto bf5 = std::bind(&CT::func, &ct, std::placeholders::_1, std::placeholders::_2);
```

这次里面的m_a就是10了

所以ct前面如果加了&，就不生成临时的CT对象了，后续的func调用修改的是ct对象的m_a值，这说明此时bind返回的这个对象其实是ct对象本身（仿函数类型对象）

## 3.3bind和function配合使用

```cpp
class CT
{
public:
    void func()(int x, int y)
    {
        cout << x << y << endl;
        m_a = x;
    }
    int m_a = 0;
};

int main()
{
	CT ct;
    std::function<void(int, int)> bfc6 = std::bind(&CT::func, ct, std::placeholders::_1, std::placeholders::_2);
    bf6(10,20); // x为10，y为20
}
```

## 3.4把成员变量地址当函数一样绑定

绑定的结果放在std::function<int &(void)>里保存：说白了就是用一个可调用对象的方式来表示这个变量

void表示里面没有参数，参数为空

```cpp
class CT
{
public:
    void func()(int x, int y)
    {
        cout << x << y << endl;
        m_a = x;
    }
    int m_a = 0;
};

int main()
{
	CT ct;
    std::function<void &()> bf7 = std::bind(&CT::m_a, &ct);//如果不用&ct而用ct，那就m_a里面还是0，不是60了
    bf7() = 60;
 }
```

如果不用&ct而用ct，那么这个bind会导致两次CT拷贝构造函数的执行

第一次拷贝构造函数的执行是因为系统利用ct来产生一个临时的CT对象

第二次拷贝构造函数的执行是因为std::bind本身要返回一个CT对象，要返回的这个CT对象（仿函数）拷贝自临时的CT对象

但是std::bind执行完毕后，临时CT对象会被释放，返回的这个CT对象（仿函数）就弄到bf7里了

有两次构造就有两次析构，而用引用的话，直接省了这四次函数

再举个例子

```cpp
class CT
{
public:
    CT()
    {
        cout << "CT构造函数" << endl;
    }
    CT(const CT&tm)
    {
        cout << "CT拷贝构造函数" << endl;
    }
    ~CT()
    {
        cout << "CT析构函数" << endl;
    }
public:
    void operator()()
    {
        cout << "operator()函数被调用" << this << endl;
    }
int main()
{
	auto rt = std::bind(CT());
}
```

CT()是构造临时对象，然后又调用了拷贝构造函数生成了一个可调用对象，作为std::bind的返回内容

bind返回仿函数类型对象，就是用拷贝构造函数构造起来的对象

这个代码输出结果是

```
CT构造函数

CT拷贝构造函数

CT析构函数

CT析构函数
```

代码里再添加

```cpp
rt()
```

则输出结果是

```cpp
CT构造函数

CT拷贝构造函数

CT析构函数

operator()函数被调用OOAFF
    
CT析构函数
```

再讲一个

```cpp
void mycallback(int cs, const std::function<void(int)> &f)
{
    f(cs);
}
void runfunc(int x)
{
    cout << x << endl;
}
int main()
{
	auto bf = std::bind(runfunc, std::placeholders::_1); //runfunc的第一个参数由调用时的第一个参数指定
    mycallback(1, bf); //调用runfunc
}
```

# 4.总结

a)bind思想：所谓的延迟调用，将可调用对象统一格式，保存起来，需要的时候再调用

b)我们有std::function绑定一个可调用对象，类型成员不能绑。std::bind成员函数，成员变量等等都能绑

