---
layout: post
title: "【一文读不懂C++】论函数调用运算符，function类模版"
date: 2023-06-22
excerpt: "论函数调用运算符，function类模版"
tags: [cpp]
comments: true








---

* toc
{:toc}










# 一.函数调用运算符

一个函数

````
int func(int a){}

func(5)
````

会发现无论里面有没有参数，都要用()，其实圆括号()就是函数调用的明显标记，()有一个称呼叫做函数调用运算符

如果在类中重载了函数调用运算符()，那么就可以像使用函数一样使用该类的对象了。对象（实参）

如何使用函数调用运算符呢？

a)定义一个该类的对象

b)像函数一样使用该对象，也就是()中增加实参列表

```cpp
class Test()
{
public:
    //第一个int是函数的返回类型
    //把operator()看做是一个函数名
    int operator()(int value)
    {
        if(value < 0)
            cout << " value < 0" << endl;
        else
            return 0;
    } 
}

int i = 200;
Test obj;
int result = obj(i);
//或者，这两种是等价的
int result = obj.operator()(i);
```

需要注意的是，类中重载的话，operator()相当于是函数名，所以你单独调用的时候obj.operator()是成员函数名，你得再加个括号obj.operator()()这才是类似于func()，如果有参数的话，再加上就ok

那会不会和构造函数很像呢

比如

```cpp
class Test()
{
public:
    Test(int a) { cout << a << endl; }
    //第一个int是函数的返回类型
    int operator()(int value) const
    {
        if(value < 0)
            cout << " value < 0" << endl;
        else
            return 0;
    } 
}

Test obj(i);
```

那Test obj(i);

和

Test obj;
int result = obj(i);

两个一样吗，其实是不一样的

因为Test obj(i);这是对象定义并初始化，所以调用的是构造函数

但是如果前面没有类名，就像obj(2)，这就不是初始化了，所以也不会调用构造函数，这就是调用对象obj的()圆括号

结论：只要这个对象所属的类重载了()“函数调用运算符"，那么这个类对象就变成了可调用的了，而且可以调用多个版本的()， 只要在参数类型和数量上有差别就行

```cpp
class Test()
{
public:
    //第一个int是函数的返回类型
    //把operator()看做是一个函数名
    int operator()(int value)
    {
        if(value < 0)
            cout << " value < 0" << endl;
        else
            return 0;
    } 
    int operator()(int value, int value2)
    {
        return 1;
    } 
}
```

当这个类重载了()，那么该类的对象多了个新名字，叫做“函数对象”比如上面的obj ，因为可以调用这种对象，或者换一种说法：这些对象的行为像函数一样



# 二.不同调用对象的相同调用形式

```cpp
class Test()
{
public:
    Test(int a) { cout << a << endl; }
    //第一个int是函数的返回类型
    int operator()(int value) const
    {
        if(value < 0)
            cout << " value < 0" << endl;
        else
            return 0;
    } 
}

int name(int a) { return 0; }
```

这里的函数name和类Test的重载的()，这两个东西，调用参数和返回值相同，就叫做“调用形式相同”

一种调用形式 对应 一个函数类型：int(int);

函数类型；int(int)表示接受一个int参数，返回一个int值

引入概念叫做“可调用对象”，如下两个都是可调用对象（不只是对象，函数也是）

a)name函数

b)重载了函数调用运算符的Test类对象

把这些可调用对象的指针保存起来，目的是方便我们随时调用这些“可调用对象”，这些指针感觉像是我们C语言中的函数指针

```cpp
int(*p)(int x);//p就是定义的指针变量

p = max;//函数max的入口地址给p

int result = (*p)(5); //调用函数max

```

可以用map来存可调用对象,也就是函数指针

"add"  0x123

"red" 0x456

```cpp
map<string, int(*)(int)> myoper;
//记得就是把int(*p)(int x); p给干掉了，写p就报错了
myoper.insert({"nm", name}) 
Test obj;
myoper.insert({"ts", Test});
myoper.insert({"ts", obj});
//但是这两个都不行，都会报错的，那应该怎么搞呢，这就引出了第三个问题，function类型介绍
```

# 三.标准库function类型介绍

function类模版；要提供模版参数来表示该function类型能够表示的“对象调用形式”也就是前面说的一种调用形式 对应 一个函数类型：int(int);

```cpp
function<int(int)> //这就叫声明了一个function类型，用来代表一个可调用对象，它所代表的这个可调用对象是：接受一个int参数，返回的也是一个int参数
```

好了开始解决上面的问题

```cpp
function<int(int> f1 = name; //函数指针
function<int(int> f2 = obj;//类对象，因为类中有()重载
function<int(int> f2 = Test();//用类名生成一个对象，也可以，因为类中有()重载

//调用以下
f1(5);
f2(5);
f3(5);
```

所以不管是函数指针类型，还是类对象类型也罢，全给他搞成了function类型，都统一咯

这次重新改造map，因为map不能重载对象，只能重载函数

```cpp
map<string, int(*)(int)> myoper;
//把这个改成下面这个
map<string, function< int(int)>> myoper = {
    {"nm", name},
    {"ts", obj},
    {"ts2", Test()},
}
myoper["nm"](12); //就是调用了name函数
myoper["ts"](3); //调用obj对象的()操作符
cout << myoper["ts"](3) << endl;
myoper["ts2"](-3); //调用Test类对象的()操作符
```

但发现有个问题，就是万一name函数有重载呢，那就不符合function< int(int)>>，并且function不能识别到底调用哪个了

```cpp
int name(int a) { return 0; }
int name() { return 0; }
function<int(int)> f1 = name;//这行代码会报错的
```

如果函数有重载，就无法放到平function<>类型的对象中

我们可以通过定义函数指针来解决

```cpp
int(*fp)(int) = name;
int(*fp2)() = name;
int(*fp3)(int) = name;
```

调试时候，会发现fp和fp3的值是一样，说明确实可以这样搞，函数指针能够找到对应是哪个函数

定义函数指针，不会产生二义性，因为函数指针里有对应的参数类型和返回值类型

```cpp
function<int(int)> f1 = fp;//直接塞进去函数指针而不是函数名name
```



