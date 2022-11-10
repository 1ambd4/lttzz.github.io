---
title: "Cpp"
date: 2021-10-07T09:27:45+08:00
lastmod: 2021-10-07T09:27:45+08:00
draft: false
keywords: []
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

## 访问控制权限

1. 访问权由类本身授予，而不是但方面获取。

   类作为被保护的单位，基本规则是你不能授予自己访问某个类的权利，只有安放在类内部的声明（家电是由类的拥有者放置的）可以授予你访问权。按照默认规则，所有信息都是私有的。

2. 受控制的是访问权，而不是可见性。

   ``` c++
   int a = 37;
   
   class A {
   public:
   private:
     	int a;
   };
   
   class B : public A {
   public:
     	print()
       {
         	std::cout << a << std::endl;
       }
   private:
   };
   ```

   用B的实例调用`print()`方法会报错，因为类A里的私有成员变量a遮蔽了全局变量a，而类B对来自于类A的数据成员a不具有访问权，因而报错，也即并非不可见，而是无权访问。倘使此处设计为可见性，那么派生类胡乱写会导致程序逻辑的极度混乱。

3. c++不需要垃圾回收

   在1985年，还是c with class的时候，c++之父考虑过垃圾回收，但考虑到对于一个已经被用在实时处理和硬核心系统（如设备驱动程序）的语言而言，垃圾回收的引入并不合适。

# 基本语法

## 函数声明

``` c++
// 如何正确理解下面的语句？

(*(void(*)())0)();
```

解读之前，先谈一谈类型转换和函数指针。

``` c++
// func是一个指针，指向一个函数，这个函数的返回值是float
// 即func是指向返回值为float的函数指针
float (*func)();
// 那么，其类型就是去掉变量名和末尾分号
float (*)()
// c/c++是可以显式强转的
double d = 233.332;
// 以下三种写法都是接受的
int i = (int)d;
int i = int(d);
int i = (int)(d);

// 其他非内建类型强转同理，因而开始的函数指针对应的类型转换符如下
(float (*)())

// 函数指针
// 在c/c++中，如果pf是一个函数指针，那么下面两条调用语句等价（语法糖了
(*pf)();
pf();
// 所以千万不要省略括号，否则就是另一个意思了
*pf();	// *((*pf)())


// 说完了函数指针调用，那如何调用特定地址处的函数呢？比如说0x00000000
(*0)();	// 好想这么写，但不行，因为字面量0默认不会被当作地址来解释
// 所以需要显式强转
// 如何转成一个返回void的函数指针呢？先声明一个此类型变量吧
void (*func)();  // 调用：(*func)();
// 因而可得类型转换符
(void (*)())
// 对0强转
(void (*)())0;
// 现在是一个函数指针了，调用
(*(void (*)())0)();
```



# STL

## container

### vector

#### reserve

``` c++
// 看 c++ primer 的时候，随手就写了下面的代码
vector<int> nums;
nums.reserve(100);

for (intt i = 0; i < 100; ++i) {
  	nums[i] = static_cast<int>(rand() % 200);
}

vector<int>::const_iterator vit = find(nums.cbegin(), nums.cend(), 37);
cout << (vit != nums.cend() ? "no" : "yes") << endl;

// 不管 target 怎么换，即使换成了 nums[0]，还都只是输出 no
// 尝试输出，有值啊
for (int i = 0; i < 100; ++i) {
  	cout << nums[i] << " ";
}
cout << endl;
// 但是 range-base for 没输出？
for (const auto& num : nums) {
  	cout << num << " ";
}
cout << endl;

// 觉得不对劲儿，遂想到输出 size 和 capacity
cout << nums.size() << " " << nums.capacity() << endl;
// 果然，输出 0 100
// cppreference 看了下 vector::reserve 果然，仅仅是建议增大 capacity 而已。
// Increase the capacity of the vector to a value that's greater or equal to new_cap. 
// If new_cap is greater than the current capacity(), new  storage is allocated, otherwise the function does nothing.
// 而 range-based for 和 find 都用到了 size，自然得不到想要的结果了。
```



## iterator

越来越觉得 Iterator 可爱的很。

### common iterator

+ iterator
+ const_iterator

### insert iterator

+ back_inserter
+ front_inserter
+ Inserter

``` c++
vector<int> nums { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
list<int> case1, case2, case3;
// algorithm 里的 generic algorithm 如果操作两个容器，且有一个容器只给了 begin，那么其实需要程序员确保这个 container 不比另一个 container 小，否则结果是未定义的
// 这样一来，copy 之类的对空 container 操作并不能使用普通的 iterator
// inserter iterator 操作会转化成相应容器的插入操作，并在之后移动自身比指向
copy(nums.begin(), nums.end(), back_inserter(case1));
copy(nums.begin(), nums.end(), front_inserter(case2));
copy(nums.begin(), nums.end(), inserter(case3, case3.begin()));
```

### stream iterator

``` c++
// 从标准输入读取数据进 vector，排序后输出到标准输出

istream_iterator<int> in_iter(cin), eof;
ostream_iterator<int> out_iter(cout, " ");
vector<int> nums(in_iter, eof);
sort(nums.begin(), nums.end());
copy(nums.cbegin(), nums.cend(), out_iter);
cout << endl;
```

### reverse iterator

``` c++
// 出了 forward_list 外都支持 reverse iterator
sort(nums.begin(), nums.end());
sort(nums.rbegin(), nums.rend());
```



# Effective C++

## chapter 1 Accustoming youself to C++

declaration：告知编译器某个对象的名称和类型，但略去细节。

definition：提供一些声明式遗漏的细节给编译器。

initialization：给予对象初值。

只这么说，挺抽象的，喜欢 C++ 的一个理由就是，搞不清楚的细节，总可以去看汇编代码和源码，正应了侯捷所说：“源码之前，了无秘密”。

``` c++
extern const int a0;	// 毫无疑问是 declaration

int a1;	// 这是 declaration 还是 definition 呢？
// 唔，谁说谁有理，但我相信编译器
int a;
int a;	// 这么写的报错信息是 error: redeclaration of 'int a'
// 破案了，是 declaration，可似乎没注意到对象有报过 redefinition error哎
// function redefinition 和 class redefinition 倒是常见

// 这么看起来，像 int a 这种，该属 definition

int main()
{
    extern int a0;	// 无对应汇编代码
    int a1;	// 无对应汇编代码
    int a2 = 2;	// movl    $2, -4(%rbp)
}

main:
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $2, -4(%rbp)
        movl    $0, %eax
        popq    %rbp
        ret
          
// 当 initialization 或者 assignment 的时候才会分配内存
int main()
{
    int a, b;
    a = 3;
    int a2 = 2;
    b = a2;

    return 0;
}
main:
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $3, -4(%rbp)	// a
        movl    $2, -8(%rbp)	// a2
        movl    -8(%rbp), %eax
        movl    %eax, -12(%rbp)	// b
        movl    $0, %eax
        popq    %rbp
        ret
```



### item 01: View C++ as a federation of languages

C++ 高效编程守则视状况而变，取决于使用C++的哪一部分。

+ C

  block、statement、preprocsssor、built-in data type、array、pointer...

+ Object-Oriented C++

  class、encapsulation、inheritance、polymorphism、virtual function...

+ Template C++

  Generic programming

+ STL

  container、allocator、iterator、algorithm、function object、adaptor

### item 02: Prefer consts， enums and inlines to #defines

\#define在预处理阶段完成简单的替换，对编译器透明，不涉及作用域，无法定义类的私有常量。

``` c++
class CPP {
private:
  	static const int version;	// 类常量声明，位于头文件内
};
// in-class initialization 现在也开始支持了
const double CPP::version = 202100;	// 类常量定义，位于实现文件内

// the enum hack？
class CPP {
private:
  	enum { version = 202100; }
};

// #define macro么，我还真没用过
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
// 使用 inline template 替换 macro，一个
template <typename T>
inline void call_with_max(const T& a, const T& b)
{
  	f(a > b ? a : b);
}
```

### item 03: Use const whenever possible

### 修饰对象

const 修饰指针/引用的时候有 top-level const 和 low-level const 之分

``` c++
// low-level const 不能修改指向的对象
int const *p1 = new int(37);
// *p1 = 73;	// error: assignment of read-only location '* p1'
p1 = new int(73);

// top-level const 不能修改自身的指向
int* const p2 = new int(37);
*p2 = 73;
// p2 = new int(73);	// error: assignment of read-only variable 'p2'
```

### 修饰函数

``` c++
// 修饰函数返回值
// 考虑有如下有理数类，重载”*“运算符
class Rational { ... };
const Rational operator * (const Rational& lhs, const Rational& rhs) { ... }
// 倘使没考虑到返回常量，那么有可能出现下面的情况
Rational a, b, c;
(a * b) = c;	// operator * 后又调用了 operator =，完全搞不懂想做什么
if (a*b = c) { ... }	// 误写，本意是判断 a*b == c

// 修饰成员函数
// 如果把 function 视作 object 的话，那么自然也有同样也该有 top/low 之分
// 只不过这里称作 bitwise constness（或 physical constness） 和 logical constness 
// bitwise constness 坚持只要不改动任何类的成员即可，对应于 top-level const
// logical constness 则坚持类成员指向的对象也应当保持 const 属性，对应于 low-level const
class CPP{
public:
    char& operator[](std::size_t positon) const	// 遵循 bitwise constness，违背 logical constness
    {
        return text[positon];
    }
private:
  	char *text;
};
const CPP cctb("hello");
char *p = &cctb[0];
*p = 'j';	// cctb 声明为常量了，此处却可以对其进行修改，实在不应该

// const 成员函数和普通成员函数构成 override
// 遵循 DIP，应当让 no-const 版本调用 const 版本以避免代码重复
```

### item 04: Make sure that objects are initialized before they are used

鉴于规则复杂，记忆起来困难，最佳的处理方式是：永远在使用对象之前先对其进行初始化。

内建类型手工初始化，其余交由构造函数完成，但注意区分 assignment 和 initialization。

``` c++
// C++规定，对象的成员变量的初始化动作发生在进入构造函数体之前
// 语法层面，提供了 member initialization list 机制，供初始化成员变量之用
// initialization
class CPP1 {
public:
    CPP1(int value) : value(value) {}
private:
    int value;
};

// assignment
class CPP2 {
public:
    CPP2(int value) { value = value; }
private:
    int value;
};

int main()
{
    CPP1 c1(1);
    CPP2 c2(2);
}

// 从汇编可以看出来，member initialization list 并不是语法糖
CPP1::CPP1(int) [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     DWORD PTR [rbp-12], esi
        mov     rax, QWORD PTR [rbp-8]
        mov     edx, DWORD PTR [rbp-12]
        mov     DWORD PTR [rax], edx
        nop
        pop     rbp
        ret
CPP2::CPP2(int) [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     DWORD PTR [rbp-12], esi
        nop
        pop     rbp
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        lea     rax, [rbp-4]
        mov     esi, 1
        mov     rdi, rax
        call    CPP1::CPP1(int) [complete object constructor]
        lea     rax, [rbp-8]
        mov     esi, 2
        mov     rdi, rax
        call    CPP2::CPP2(int) [complete object constructor]
        mov     eax, 0
        leave
        ret
```

下面的例子没看明白，留坑。

## chapter 2 Constructors, Destructors, and Assignment Operators

### item 05: Know what functions C++ cliently writes and call

好了好了，又到了欢乐的时刻了，各种人都会告诉你，如果你不写 constructor、destructor、copy constructor、assignemtn operator...等等的时候，编译器会自动生成默认版本。

这话错吗，也没错吧，这就好像小学数学告诉你三角形内角之和为180°，但不会说此结论仅在欧氏几何中适用，那时候也接触不到非欧几何不是~~，别说小学了，大学毕业都没怎么接触非欧几何~~。此处同理，这是符合认知规律的。

扯远了，说回来，就目前我的知识储备对此问题的理解如下：提不提供取决于有没必要提供以及能不能提供。

``` c++
// 先说有没有必要提供
// 用到了才会提供，用不到提供干嘛呢
class CPP {
public:
private:
};

int main()
{
    CPP c1, c2;
    c1 = c2;
    CPP c3(c1);
    return 0;
}

// 猜猜看，对应的汇编代码长啥样子
main:
        push    rbp
        mov     rbp, rsp
        mov     eax, 0
        pop     rbp
        ret
// surprising，啥也没，和 main 函数体为空生成的汇编代码一样
// 编译优化掉了么？并不是，编译参数我特意加了 -O0
// O3 优化后的汇编代码如下
main:
        xor     eax, eax
        ret

// 上面是勿需提供的极端，empty class，试着给类添加些成员变量呢？
// 扔一个 integer 看看          
class CPP {
public:
private:
    int value;
};
// ctor、dtor、copy ctor、operator = 都没如约出现
// 原因么，没多么复杂，还记得 chapter1 里面强调的注意区分 declaration 和 initialization 么，这里显然只是 declaration

// 改用 initialization
class CPP {
private:
    int id;
};

int main()
{
    CPP *c1 = new CPP();
  	return 0;
}

main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     edi, 4
        call    operator new(unsigned long)
        mov     DWORD PTR [rax], 0
        mov     QWORD PTR [rbp-8], rax
        mov     eax, 0
        leave
        ret
// 看汇编仅调用了 operator new，operator new 会调用 malloc 分配以内，以及 ctor 进行初始化操作
// 但这儿究竟有没有调用 default ctor 呢？很不明显
class CPP {
private:
    int id = 0;
};

int main()
{
    CPP *c1 = new CPP();

    return 0;
}
// 对应的汇编，operator new 和 default ctor都调用了，也生成了 default ctor
CPP::CPP() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     DWORD PTR [rax], 0
        nop
        pop     rbp
        ret
main:
        push    rbp
        mov     rbp, rsp
        push    rbx
        sub     rsp, 24
        mov     edi, 4
        call    operator new(unsigned long)
        mov     rbx, rax
        mov     DWORD PTR [rbx], 0
        mov     rdi, rbx
        call    CPP::CPP() [complete object constructor]
        mov     QWORD PTR [rbp-24], rbx
        mov     eax, 0
        mov     rbx, QWORD PTR [rbp-8]
        leave
        ret

// ctor 对这儿说的差不多了，copy ctor 同理，但有点...
class CPP {
private:
    int id = 0;
};

int main()
{
    CPP *c1 = new CPP();
    CPP *c2(c1);		// 我以为怎么着这儿也要生成 copy ctor 吧

    return 0;
}

// 实际上并没有
CPP::CPP() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     DWORD PTR [rax], 0
        nop
        pop     rbp
        ret
main:
        push    rbp
        mov     rbp, rsp
        push    rbx
        sub     rsp, 24
        mov     edi, 4
        call    operator new(unsigned long)
        mov     rbx, rax
        mov     DWORD PTR [rbx], 0
        mov     rdi, rbx
        call    CPP::CPP() [complete object constructor]
        mov     QWORD PTR [rbp-24], rbx
        mov     rax, QWORD PTR [rbp-24]		// 这一行和下一行完成 c2 的构造，居然仅是 mov 指令，出乎意料
        mov     QWORD PTR [rbp-32], rax
        mov     eax, 0
        mov     rbx, QWORD PTR [rbp-8]
        leave
        ret

          
// 不玩了不玩了，太搞心态了，都一起出来吧！
class CPP {
private:
    std::string value;
};

int main()
{
    CPP c1, c2;
    c2 = c1;
    CPP c3(c1);;

    return 0;
}
// 对应汇编
CPP::CPP() [base object constructor]:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdi, rax
        call    std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string() [complete object constructor]
        nop
        leave
        ret
CPP::~CPP() [base object destructor]:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdi, rax
        call    std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string() [complete object destructor]
        nop
        leave
        ret
CPP::operator=(CPP const&):
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     QWORD PTR [rbp-16], rsi
        mov     rdx, QWORD PTR [rbp-16]
        mov     rax, QWORD PTR [rbp-8]
        mov     rsi, rdx
        mov     rdi, rax
        call    std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >::operator=(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)
        mov     rax, QWORD PTR [rbp-8]
        leave
        ret
CPP::CPP(CPP const&) [base object constructor]:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     QWORD PTR [rbp-8], rdi
        mov     QWORD PTR [rbp-16], rsi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdx, QWORD PTR [rbp-16]
        mov     rsi, rdx
        mov     rdi, rax
        call    std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&) [complete object constructor]
        nop
        leave
        ret
main:
        push    rbp
        mov     rbp, rsp
        push    rbx
        sub     rsp, 104
        lea     rax, [rbp-48]
        mov     rdi, rax
        call    CPP::CPP() [complete object constructor]
        lea     rax, [rbp-80]
        mov     rdi, rax
        call    CPP::CPP() [complete object constructor]
        lea     rdx, [rbp-48]
        lea     rax, [rbp-80]
        mov     rsi, rdx
        mov     rdi, rax
        call    CPP::operator=(CPP const&)
        lea     rdx, [rbp-48]
        lea     rax, [rbp-112]
        mov     rsi, rdx
        mov     rdi, rax
        call    CPP::CPP(CPP const&) [complete object constructor]
        mov     ebx, 0
        lea     rax, [rbp-112]
        mov     rdi, rax
        call    CPP::~CPP() [complete object destructor]
        lea     rax, [rbp-80]
        mov     rdi, rax
        call    CPP::~CPP() [complete object destructor]
        lea     rax, [rbp-48]
        mov     rdi, rax
        call    CPP::~CPP() [complete object destructor]
        mov     eax, ebx
        jmp     .L10
        mov     rbx, rax
        lea     rax, [rbp-80]
        mov     rdi, rax
        call    CPP::~CPP() [complete object destructor]
        lea     rax, [rbp-48]
        mov     rdi, rax
        call    CPP::~CPP() [complete object destructor]
        mov     rax, rbx
        mov     rdi, rax
        call    _Unwind_Resume
.L10:
        mov     rbx, QWORD PTR [rbp-8]
        leave
        ret
```



``` c++
// 再说能不能提供
// 如果提供了，会违背更一般性的原则，则报错

class CPP {
public:
    explicit CPP(std::string& value) : value(value) { }
private:
    std::string& value;
};

int main()
{
    std::string value = "hahaha";
    CPP c1(value), c2(c1);		// 这不是 ctor 和 copy ctor 么？但汇编代码里没有啊
  
    return 0;
}
// 这也不是 default ctor 或者 copy ctor 啊
CPP::CPP(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >&) [base object constructor]:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi
        mov     QWORD PTR [rbp-16], rsi
        mov     rax, QWORD PTR [rbp-8]
        mov     rdx, QWORD PTR [rbp-16]
        mov     QWORD PTR [rax], rdx
        nop
        pop     rbp
        ret
.LC0:
        .string "hahaha"
main:		// 为方便查看，将 main 生成的汇编删除掉
				...
.L6:
        mov     rbx, QWORD PTR [rbp-8]
        leave
        ret
    
          
// 不懂了不懂了，实际要讨论的是如下的情况
class CPP {
public:
    explicit CPP(std::string& value) : value(value) { }
private:
    std::string& value;
};

int main()
{
    std::string value1 = "hahaha";
    std::string value2 = "tatata";
    CPP c1(value1), c2(value2);
  
  	// 如果提供了默认的 copy ctor，并进行简单的变量赋值操作
  	// 那么意味着更改了引用型成员变量 value 的指向，因而无法通过编译
    c2 = c1;

    return 0;
}
```

我裂开了，每次都能发现些奇奇怪怪和以前理解不一样的现象。

### item 06: Explicitly disallow the use of compiler-generated functions you do not want

item05 提到了编译器提供的默认版本 ctor、dtor、copy ctor 和 operator =，但有些时候并不需要，甚至不该需要默认提供的版本。

举拷贝构造函数的例子来说，对于独有的东西，不希望有一份相同的拷贝，显然不该为其提供 copy ctor，可问题是，编译器发觉没手动提供 copy ctor，可能会生成默认版本的 copy ctor，如果不想编译器提供，那么需要手动提供，完美，无解了。

书里提供的方法是将 ctor、dtor、copy ctor、operator = 等不希望编译器提供的声明为 private，函数体为空。

C++ 11 之前，这是比较好的解决思路，但 C++ 11 可以如下，使用 delete 显式地拒绝编译器提供默认版本。

``` c++
class CPP {
public:
private:
  	CPP() = delete;
  	CPP(const CPP& rhs) = delete;
  	CPP& operator = (const CPP& rhs) = delete;
  	~CPP() = delete;
};

// 另外，= default 显示让编译器提供，= 0 表示纯虚函数
```

### item 07: Declare destructors virtual in polymorphic base classes

析构函数是否声明为 `virtual` 有两方面的考虑：

+ 该声明为 `virtual` 的没声明为 `virtual`，可能导致局部析构
+ 不该声明为 `virtual` 的声明为 `virtual` 了，可能导致移植性差

``` c++
// case 1: 基类析构函数没有声明为 virtual，当销毁上转型的子类对象时，其结果是未定义的
// 通常只调用基类的析构函数，从而导致局部析构，属于子类的部分无法正确析构
class Base {
public:
  	Base() = default;
  	~Base() = default;	// 只是漏掉了基类 dtor 的 virtual 声明，会导致子类析构出现问题
private:
};

class Derive {
public:
  	Derive(int v) : p(new int(v)) { }
  	~Derive() = default;
private:
  	int *p;	// 无法被正常析构
};

int main()
{
  	Base *drive = new Drive();
  	delete drive;
  	return 0;
}
// 注意 STL 里的绝大多数 container 析构函数都没有声明为 virual
// 这意味着 class CPP : public string {}; 会引起未定义行为

// case 2: 不该声明为 virtual 的声明为 virtual 了
class CPP {
public:
  	CPP() = default;
  	virtual ~CPP() { cout << "dtor" << endl; }
private:
};
// 须知 no free lunch，虚函数使得运行时多态成为可能，代价是需要额外的内存空间保存虚函数表
// 体现在对象内存里是多附加一个 vptr（virtual table pointer） 虚函数表指针
// 而其他多数语言并没有 vptr
```

原则是：只有当类内有其他的虚函数时，才将其析构函数声明为 `virtual`。

说明白点，就是只有当类可能成为基类的时候，才声明析构函数为 `virtual`。

# 单元测试

主流的单元测试框架是[googletest](https://github.com/google/googletest)，官方文档在[这儿](https://google.github.io/googletest/)。

从如下几个方面定义良好的测试：

+ 测试应当独立且可重复
+ 测试应当组织好以反映测试代码的结构性
+ 测试应当具有可移植性
+ 当测未通过时，应当提供尽可能多的错误信息
+ 测试框架应当简单易用，使得测试人员能够更好的关注待测试的内容本身

## 说明

`ASSERT_*` 不通过会终止当前函数，`EXPECT_*` 则不会，一般情况下，推荐使用 `EXPECT_*`。

## 例子

``` c++
// 注意区分 testsuit 和 test
TEST(TestSuiteName, TestName) {
  ... test body ...
}


int Factorial(int n);  // Returns the factorial of n

// Tests factorial of 0.
TEST(FactorialTest, HandlesZeroInput) {
  EXPECT_EQ(Factorial(0), 1);
}

// Tests factorial of positive numbers.
TEST(FactorialTest, HandlesPositiveInput) {
  EXPECT_EQ(Factorial(1), 1);
  EXPECT_EQ(Factorial(2), 2);
  EXPECT_EQ(Factorial(3), 6);
  EXPECT_EQ(Factorial(8), 40320);
}
```



# 八股文

