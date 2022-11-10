---
title: "C++对象模型"
date: 2022-01-16T17:12:38+08:00
lastmod: 2022-01-16T17:12:38+08:00
draft: false
keywords: [cpp]
description: ""
tags: []
categories: []
author: "lttzz"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

<!--more-->

# The Semantics of Constructors

## Default Constructor

误解：编译器会为任何没有显示提供构造函数的类提供默认构造函数，这个默认构造函数会初始化所有数据成员。

编译器提供的defalut constructor分为两种情况，一是看起来没什么用的implicit nontrivial default constructor和真的没什么用（并不会有代码对应）的implicit trivial default constructor。

implicit nontrivial default constructor的出现是为了满足编译器的需要，而不是满足程序员的需要，主要涉及两方面，一是需要调用其他类的构造函数以初始化类类型数据成员，二是需要初始化虚函数表以实现多态。

1. **继承自含有default constructor的base class；**
2. **内含有default constructor的member class object；**
3. **继承自virtual base class；**
4. **声明或继承有virtual functon；**

下面来说一下什么叫“满足编译器的需要”。 首先要明确一点，偷懒是天性，程序员也会，把重复的无技术劳动交给任劳任怨的计算机是很愉快的事情，这就是编译器暗地里多做的一些事儿，~~不说别的，标题的default constructor就属此范畴。~~而为了使得编译器的小动作在语法规则下闭合，编译器不得不做更多的小动作。

编译器的小动作包括class object初始化的时候，会自动调用构造函数以分配内存和初始化数据成员，如下代码所示：

``` c++
class Foo {
public:
    Foo() {}	// 显示提供一个空构造函数
};
int main()
{
    Foo foo;
    return 0;
}

/////
Foo::Foo() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        nop
        pop     rbp
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        // ------ Foo foo ------
        lea     rax, [rbp-1]	// 计算foo的地址（为啥不直接存到rdi里呢？
        mov     rdi, rax	// 规定参数传递用的是rdi，故而foo地址传进去 
        call    Foo::Foo() [complete object constructor] // 调用构造函数
        // ---------------------
        mov     eax, 0
        leave
        ret
```

这是最理想的情况，程序员手写了构造函数，并且也能够满足代码的需要。有一天，我不想写这个函数体为空的没有实际意义（程序员的角度看）的构造函数了，偷懒不写，成吗？成！

``` c++
class Foo {
public:
};

int main()
{
    Foo foo;
    return 0;
}

/////
main:
        push    rbp
        mov     rbp, rsp
        mov     eax, 0
        pop     rbp
        ret
```

对应的汇编代码怎么变了？foo呢，怎么没了？原因是编译器发现声明了一个未使用的变量，程序员又偷懒没写任何构造函数，编译器干脆也偷懒，啥汇编代码也不生成。我觉得编译器此处的偷懒不可谓不高明，编译器的工作说直白点就是翻译，将高级语言翻译成机器语言，翻译呢有三重境界：信、达、雅，编译器此处做到了雅（废话直接不译）。

好嘛，被编译器惯坏了，能不写的构造函数坚决不写（并不提倡）。

``` c++
// 其他人的代码
class Foo {
public:
    Foo() { }
};

// 我写的代码，懒，不想写构造函数
class Bar {
public:
    Foo foo;
};
int main()
{
    Bar bar;
    return 0;
}

/////
Foo::Foo() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        nop
        pop     rbp
        ret
Bar::Bar() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdi, rax
        call    Foo::Foo() [complete object constructor] // 编译器调用了foo的构造函数以初始化foo
        nop
        leave
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        lea     rax, [rbp-1]
        mov     rdi, rax
        call    Bar::Bar() [complete object constructor] // 编译器提供了构造函数
        mov     eax, 0
        leave
        ret
          
// 从汇编代码来看，给类Bar生成的构造函数等价于如下：
Bar() { Foo::Foo(); }	// 虽然这在语法上是错误的
```

继承还有虚函数的情况呢？

``` c++
class Foo {
public:
    Foo() { }
};

class VFoo {
public:
    virtual void func() { }
};

class Bar : public VFoo {
public:
    Foo foo;
};

int main()
{
    Bar bar;
    return 0;
}


/////
Foo::Foo() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        nop
        pop     rbp
        ret
VFoo::func():
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        nop
        pop     rbp
        ret
VFoo::VFoo() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     edx, OFFSET FLAT:vtable for VFoo+16
        mov     rax, QWORD PTR [rbp-8]
        mov     QWORD PTR [rax], rdx
        nop
        pop     rbp
        ret
// 编译器生成的析构函数首先调用虚基类的构造函数，接着是虚函数表初始化，最后是数据成员的构造函数
Bar::Bar() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdi, rax
        call    VFoo::VFoo() [base object constructor]
        mov     edx, OFFSET FLAT:vtable for Bar+16
        mov     rax, QWORD PTR [rbp-8]
        mov     QWORD PTR [rax], rdx
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 8
        mov     rdi, rax
        call    Foo::Foo() [complete object constructor]
        nop
        leave
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        lea     rax, [rbp-16]
        mov     rdi, rax
        call    Bar::Bar() [complete object constructor]
        mov     eax, 0
        leave
        ret
```

以上都是数据结构简单，缺省构造函数的情况，某天需要显式提供构造函数，前面的一切还成立么？

``` c++
class Foo {
public:
    Foo() { }
};

class VFoo {
public:
    virtual void func() { }
};

class Bar : public VFoo {
public:
    Bar() { val = 233; }	// 显式的构造函数
    Foo foo;
    int val;
};

int main()
{
    Bar bar;
    return 0;
}


/////
Foo::Foo() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        nop
        pop     rbp
        ret
VFoo::func():
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        nop
        pop     rbp
        ret
VFoo::VFoo() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     edx, OFFSET FLAT:vtable for VFoo+16
        mov     rax, QWORD PTR [rbp-8]
        mov     QWORD PTR [rax], rdx
        nop
        pop     rbp
        ret
// 顺序依次是
// 1、基类构造函数  2、虚函数表  3、数据成员构造函数  4、自己类的构造函数
// 其中前三部分是编译器生成的，这些由编译器也本该由编译器生成
// 因为有可能基类之类的是有其他人编写，那么基类的使用者应当不需要了解基类的细节即可使用
// 而第四部分则需要由类的编写者提供
Bar::Bar() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdi, rax
        call    VFoo::VFoo() [base object constructor]
        mov     edx, OFFSET FLAT:vtable for Bar+16
        mov     rax, QWORD PTR [rbp-8]
        mov     QWORD PTR [rax], rdx
        mov     rax, QWORD PTR [rbp-8]
        add     rax, 8
        mov     rdi, rax
        call    Foo::Foo() [complete object constructor]
        mov     rax, QWORD PTR [rbp-8]
        mov     DWORD PTR [rax+12], 233
        nop
        leave
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        lea     rax, [rbp-16]
        mov     rdi, rax
        call    Bar::Bar() [complete object constructor]
        mov     eax, 0
        leave
        ret
```

那么，来看看这个？

``` c++
class Foo {
public:
    Foo() { }
};

class VFoo {
public:
    virtual void func() { }
};

class Bar : public VFoo {
public:
    Bar() { val = 233; }
    Foo foo;
    int val;
};

int main()
{
    // warning: empty parentheses were disambiguated as a function declaration
    // 从编译器的警告发现加了括号被编译器误判为函数声明了，故而对应的汇编代码为空。
  	Bar bar();
    return 0;
}

/////
main:
        push    rbp
        mov     rbp, rsp
        mov     eax, 0
        pop     rbp
        ret
```

## Copy Constructor

和default constructor一样，在编译器需要的时候提供inplicit nontrivial copy constructor，同样是四种情况：

1. **继承自有copy constructor（无论是explicit还是inplicit）的base class；**
2. **内含有copy constructor的member class object；**
3. **派生自virtual base class；**
4. **声明了一个或多个virtual function；**

同样的，前两种情况是为了调用其他类的copy construtor，而后两种情况与virtual funtion table pointer（vptr）有关，其中不太直观的是最后一种情况，举例说明一下（举不出来，太难了

``` c++

```

## Program Transformation Semantics

1. explicit initialization；
2. argument initialization；
3. return value initialization；

### explicit initialization

``` c++
class Foo { ... };

int main()
{
  	Foo f0;
  	Foo f1(f0);
  	Foo f2 = f0;
  	Foo f3 = Foo(f0);
}

// 等效于
Foo f1, f2, f3;
f1.Foo::Foo(f0);
f2.Foo::Foo(f0);
f3.Foo::Foo(f0);
```

### argument initialization

``` c++
class Foo { ... };
void func(Foo foo) { ... }

func(foo);

// 函数调用等效于
Foo tmp;
tmp.Foo::Foo(foo);
func(tmp);
```

也即传参的时候，先调用copy constructor构建一个临时对象，然后将这个临时对象以bitwise的方式传给函数。

### return value initialization

``` c++
// bar的返回值如何从局部变量foo处取得呢？
Foo bar()
{
  	Foo foo;
  	// ...
  	return foo;
}

// stroustrup在cfront中采用如下思路
//		1.给函数添加一个额外的引用参数，同时返回值改为void
//		2.在return语句前添加copy constructor的调用操作
void bar(Foo &res)
{
  	Foo foo;
  	foo.Foo::Foo();	// 编译器生成的对foo的default constructor
  	// ...
  	res.Foo::Foo(foo);	// 编译器生成的对res的copy constructor
  	return;
}
```

#### NRV优化

上面的例子foo和res两个对象做了重复的工作，因而引入NRV优化。Named Return Value（NRV）优化是编译器层面做的优化，只保留res，对foo的操作都作用在res上。伪代码展示如下：

``` c++
Foo bar()
{
  	Foo foo;
  	// ...
  	return foo;
}

// 优化后
void bar(Foo &res)
{
  	res.Foo::Foo();
  	// ...
  	return res;
}
```

判断是否需要显式提供copy constructor的依据是是否大量的memberwise初始化操作，如果需要，则最好提供一个copy constructor，这也有利于编译器进行NRV优化。

## Member Initialization List

必须需要使用initialization list的情况有：

1. **当初始化一个reference member时；**
2. **当初始化一个const member时；**
3. **当需要调用一个base class的有参constructor时；**
4. **当需要调用一个member class的有参constructor时；**

从使用情景来看，不难得出，initialization list是对constructor功能的补充，提供了变量声明之前的一个时机。initialization list的初始化顺序与变量在类中的声明顺序一致，与list中的顺序无关，且initialization list生成的代码被插入到任何explicit user code之前。



# The Semantics of Data

``` c++
class Foo {
  
};

cout << sizeof(Foo) << endl;	// 结果为1

// 也即空空如也的类会被编译器安插进去一个char大小的数据
// 这是出于不让两个空类的对象有相同内存空间的考虑
Foo foo1, foo2;
if (&foo1 == &foo2)
  	cerr << "Oh, no!" << endl;
```

