---
title: "侯捷C++内存管理机制"
date: 2021-10-01T13:52:40+08:00
lastmod: 2021-10-12T13:52:40+08:00
draft: false
keywords: []
description: ""
tags: ['cpp', 'jjh']
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

侯捷，yyds！

<!--more-->

# primitives

## big picture

![](/Picbed/2021_10/1001_00.png)

这门课的副标题是“从平地到万丈高楼”，那不妨先站在高处，对C++内存管理的big picture有一定的认识。

即 new/delete expression -> operator new()/delete() -> malloc()/free()，从上面的表可以知道，有修改权的只有作为中间层的 operator new()/delete()，这也好理解，malloc()/free()靠近硬件，new/delete expression 靠近语言，留出中间层，在稳定中寻求扩展。

![](/Picbed/2021_10/1001_01.png)

## basic use of c++ memory primitives

``` c++
// malloc()并不知晓单个元素大小，故而size是字节个数
void * p1 = malloc(512);		// 512 bytes
free(p1);

complex<int>* p2 = new complex<int>;		// 1 complex
delete p2;

void* p3 = ::operator new(512);		// 512 bytes
::operator delete(p3);

// allocator分配的时候，由模版参数已经可以推知单个元素所占字节数，故而size是元素个数
void *p4 = allocator<int>().allocate(10);		// 10 ints
allocator<int>().deallocate((int*)p4, 10);
```

## new/delete expression

![](/Picbed/2021_10/1001_02.png)

很好的解释了为啥C++面试八股文里有关new expression和malloc()总会那是否调用ctor说事儿，new expression做了两步，分配内存和构造对象，分配内存交由包装了malloc()的operator new()完成，对象构造自然是交给构造函数喽。

![](/Picbed/2021_10/1001_03.png)

delete expresson和free()同理，delete expression 调用析构函数销毁对象，调用free()释放内存。

## array new,array delete

大一初学C++，被 `delete[] p `丑到了，特别是高维数组：

``` c++
int **p = new int *[10];
for (int i = 0; i < 10; ++i) {
  p[i] = new int[5];
}
for (int i = 0; i < 10; ++i) {
  delete[] p[i];
}
```

完全不想去写下面释放内存的过程，两维尚且如此，三维四维更高维呢，用array/vector吧，:)。

所有老师都给你讲不这么写就会导致 memory leak，就像他们都会说类里不写构造函数会默认提供一个无参的构造函数一样。

实际上呢？试一试呗，尝试一维数组。~~鳖问我1014有啥特殊含义，写错了（不是~~

``` c++
// 喜闻乐见的教科书写法
#include <iostream>

using namespace std;

int main()
{
    constexpr int kSize = 1014*1024*1024;
    int *p = new int [kSize];
    
    delete[] p;
    
    return 0;
}
```

![](/Picbed/2021_10/1001_05.png)

![](/Picbed/2021_10/1001_06.png)

可以看到，标准写法可以成功回收内存空间，定然没有 memory leak 喽，那“野路子”呢？

![](/Picbed/2021_10/1001_07.png)

![](/Picbed/2021_10/1001_08.png)

好哦，也被成功回收了耶，;)。

但是当测试二维数组的时候，出现问题了， memory leak，:-(。这是为什么呢？先记录一下测试过程：

``` c++
// 教科书写法，全部回收掉了
#include <iostream>
using namespace std;

int main()
{
    constexpr int kSize = 1014*1024*1024;
    int **p = new int *[kSize];
    for (int i = 0; i < 4; ++i) {
        p[i] = new int[kSize];
    }
    
    for (int i = 0; i < 4; ++i) {
        deletep[] p[i];
    }
    
    return 0;
}

// 瞎写一通，基本上都没回收，memory leak 严重
#include <iostream>
using namespace std;

int main()
{
    constexpr int kSize = 1014*1024*1024;
    int **p = new int *[kSize];
    for (int i = 0; i < 4; ++i) {
        p[i] = new int[kSize];
    }
    
    for (int i = 0; i < 4; ++i) {
        deletep[] p[i];
    }
    
    return 0;
}

// 改进，大部分回收，还有一点点
int main()
{
    constexpr int kSize = 1014*1024*1024;
    int **p = new int *[kSize];
    for (int i = 0; i < 4; ++i) {
        p[i] = new int[kSize];
    }
    
    for (int i = 0; i < 4; ++i) {
        deletep p[i];
    }
    
    return 0;
}

// 只有一点没回收了，咋回事儿呢？delete p 一下？
// 就回收完了？
#include <iostream>

using namespace std;

int main()
{
    constexpr int kSize = 1014*1024*1024;
    int **p = new int *[kSize];
    for (int i = 0; i < 4; ++i) {
        p[i] = new int[kSize];
    }
    
    for (int i = 0; i < 4; ++i) {
        delete p[i];
    }
    delete p;
    
    return 0;
}
```

malloc()分配内存的时候会在分配的内存块上下加记录了内存块大小的cookie，我猜测，`delete[] p` 和 `delete p` 没差。~~这时候就体现c/c++的好了，不理解某现象就去看汇编嘛~~

``` code
//	constexpr int kSize = 1024;
0x100003f2f      c745f8000400.  mov dword [var_8h], 0x400    ; 1024

//	int *p1 = new int [kSize];
//	delete[] p1;
0x100003f36      bf00100000     mov edi, 0x1000
0x100003f3b      e862000000     call sym operator new[](unsigned long) ;[1]
0x100003f40      488945f0       mov qword [var_10h], rax
0x100003f44      488b45f0       mov rax, qword [var_10h]
0x100003f48      4883f800       cmp rax, 0
0x100003f4c      488945e0       mov qword [var_20h], rax
0x100003f50      0f840c000000   je 0x100003f62
0x100003f56      488b45e0       mov rax, qword [var_20h]
0x100003f5a      4889c7         mov rdi, rax
0x100003f5d      e834000000     call sym.imp.operator_delete___void_ ;[2]
; CODE XREF from entry0 @ 0x100003f50

//	int *p1 = new int [kSize];
//	delete p1;
0x100003f62      bf00100000     mov edi, 0x1000
0x100003f67      e836000000     call sym operator new[](unsigned long) ;[1]
0x100003f6c      488945e8       mov qword [var_18h], rax
0x100003f70      488b45e8       mov rax, qword [var_18h]
0x100003f74      4883f800       cmp rax, 0
0x100003f78      488945d8       mov qword [var_28h], rax
0x100003f7c      0f840c000000   je 0x100003f8e
0x100003f82      488b45d8       mov rax, qword [var_28h]
0x100003f86      4889c7         mov rdi, rax
0x100003f89      e80e000000     call sym operator delete(void*) ;[3]
; CODE XREF from entry0 @ 0x100003f7c
```

那还是有差别嘛，具体细节就要追踪 `static void operator delete(void* ptr)` 和 `static void operator delete[](void* ptr)` 了，会面会涉及到的。

回到正题，对数组的各种析构会不会 memory leak，我认为还是要看malloc了几次，malloc几次，需要相应的free几次。

![](/Picbed/2021_10/1001_04.png)

![](/Picbed/2021_10/1001_09.png)

![](/Picbed/2021_10/1001_10.png)

## placement new

placement new 可以将 object 构建于 allocated memory 中。

## 重载 ::operator new/ ::operator delete

global scope operator new/delete 虽然可以重载，但是并不建议，因为影响实在是太大了。

``` c++
// vc98\crt\src\newop21.cpp
void* opearator new(size_t size, const std::nothrow_t& _THROW0())
{
  	// try tor allocate size bytes
		void *p;
  	while ((p = malloc(size)) == 0)
    {
      	_TRY_BEGIN
          	if (__callnewh(size) == 0) break;
      	_CATCH(std::bad_alloc) return (0);
      	_CATCH_END
    }
  	return (p);
}

// vc98\crt\src\delop.cpp
void __cdecl operator delete(void *p) _THROW0()
{
  	// free an allocated object
  	free(p);
}
```



``` c++
// 仿照着vc98，重载 ::operator new/delete
// 不要包含在名字空间里

// ::operator new
void* operator new(size_t size)
{
  	return malloc(size);
}
void* operator new[](size_t size)
{
  	return malloc(size);
}

// ::operator delete
void operator delete(void *ptr)
{
  	free(ptr);
}
void operator delete[](void *ptr)
{
  	free(p);
}
```

## 重载 operator new/ delete

重载 ::operator new/delete 并是一个好的选择，如果有需要，重载 operator new/delete 会好一些，将影响限定在特定类中，因而也叫做 per-class allocator。

``` c++
class Foo {
public:
  	static void* operator new(size_t size);
  	static void* opearator new[](size_t size);
  
  	static void operator delete(void *ptr, size_t size);
  	static void operator delete[](void *ptr, size_t size);
  
private:
  	int id;
  	string name;
};

void* Foo::operator new(size_t size)
{
  	return (Foo*)malloc(size);
}
void* Foo::operator new[](size_t size)
{
  	return (Foo*)malloc(size);
}

void Foo::opeartor delete(void *ptr, size_t size)
{
  	free(ptr);
}
void Foo::operator delete[](void *ptr, size_t size)
{
  	// 这儿回到前面 array delete 的问题，operator delete 和 operator delete[] 在这个类的实现上并无差别
  	// size 变量没用着，我猜想是有用的吧（留坑以后翻源码看其他实现版本
  	free(ptr);
}
```

## 重载 placement new

operator new/delete 接口限定好了，那倘使构造对象的时候想传额外参数呢？placement new，:)。

placement new 的第一个参数类型必须是 `size_t` ，其他可任意，同理，placement delete 的第一个参数类型必须是 `void*`。

``` c++
// 比如我想这么写
Foo *p = new(6)Foo;

class Foo {
public:
  	// 模仿标准库提供的 placement new()
  	// 只传回 pointer，并不实际分配内存，因而可认为是定点的new
  	void* operator new(size_t size, void *start)
    {
      	return start;
    }
  	// 就奇怪了，为啥非要多分配一块内存呢，new expression 按 class 大小分配的内存空间还不够用的么？
  	void* operator new(size_t size, long extra)
    {
      	return malloc(size + extra);
    }
  
  	// 虽然有 placement delete 这种东西，但无法直接调用
  	// 只有当对应的 placement new 抛出异常，才会被自动调用
  	// 作用就是释放 placement new 分配所得的内存空间
  	// 额外参数需要和 placement new 一一对应
  	void operator delete(void *ptr, void *start)
    {
      	// do nothing
    }
  	void operator delete(void *ptr, long extra)
    {
      	free(ptr)
    }
private:
  	int id;
  	string name;
};
```

## version 1, per-class allocator

Fine，primitives 了解完了，开始造轮子～

``` c++
// per-class allocator version 1

#include <cstddef>
#include <iostream>

using namespace std;

class Foo {
public:
  	Foo(int v) : value(v) { }
  	~Foo() { }
  
  	// 重载 operator new/delete 接管内存分配
  	static void* operator new(size_t size);
  	static void operator delete(void* ptr, size_t size);
private:
  	int value;
  	Foo* next;	// 为了实现内存池，不得不牺牲些内存空间
  	static Foo* sFreeStore;
  	static const int sChunkSize;
};

// 维护一个内存池，数据结构选用链表
void* Foo::opeartor new(size_t size)
{
  	Foo *p;
  	// sFreeStore 指向空闲链表头，这个设计很巧妙把内存池的初始化和内存池空的操作统一起来了，值得仔细品一品
  	if (!sFreeStore) {
      	sFreeStore = p = reinterpret_cast<Foo*>(new char[sChunkSize * size]);
      	for ( ; p != &sFreeStore[sChunkSize-1]; ++p) {
          	p->next = p+1;
        }
      	p->next = nullptr;
    }
  	p = sFreeStore;
  	sFreeStore = sFreeStore->next;
  	return p;
}

void Foo::opeartor delete(void *ptr, size_t size)
{
  	// 得益于优雅的设计，回收空间显得异常简单
  	(static_cast<Foo*>(ptr))->next = sFreeStore;
  	sFreeStore = static_cast<Foo*>(ptr);
}
```

## version 2, per-class allocator

version 1 有一个很明显的缺点是为了维护内存池，每个 object 需要多占用一个 pointer 的空间，也就比普通 malloc 出来的 cookie 节省一半左右的空间，当 object 的 member data 很小的时候，额外开销太大了。有什么 tirck 能把指针的开销避免掉么？

他居然想到了用union！！！！！

我开始期待 final version 了，:)。

``` c++
// per-class allocator version 2, ref. effective c++ 2e, item10
class Airplane {
public:
    struct AirplaneRep {
        unsigned long miles;
        char type;
    };

    static void* operator new(size_t size);
    static void operator delete(void *ptr, size_t size);

    inline void set(unsigned long m, char t)
    {
        rep.miles = m;
        rep.type = t;
    }
    inline unsigned long get_miles()
    {
        return rep.miles;
    }
    inline char get_type()
    {
        return rep.type;
    }

private:
    // 直呼内行，听侯捷讲的时候，没明白，union 是互斥的啊，不会把数据给覆盖掉么
    // 是我太傻，rep 和 next 不应该共存的
    //     分配出去的结点 rep 才有价值，此时并不需要 next 来维护什么，覆盖掉也没影响
    //     空闲链表需要 next 指针来维护，但空闲链表里就没有数据啊，rep 自然用不到
    union {
        AirplaneRep rep;
        Airplane *next;
    };

    static constexpr int kBlockSize = 512;
    static Airplane *sHeadOfFreeList;
};

Airplane* Airplane::sHeadOfFreeList = nullptr;

void* Airplane::operator new(size_t size)
{
    // 这个 allocator 是专为 Airplane 这个类设计的，判断一下，防止被子类调用
    if (size != sizeof(Airplane))
        return ::operator new(size);

    // sHeadOfFreeList 指向空闲链表头
    Airplane *p = sHeadOfFreeList;
    if (p) {
        // 如果空闲链表没空，那么可以直接分配一个结点出去
        sHeadOfFreeList = p->next;
    } else {
        // 如果空闲链表空了，那么调用 ::operator new 分配 512 个元素大小的内存空间（为啥不用malloc呢
        Airplane *new_block = static_cast<Airplane*>((::operator new(kBlockSize * sizeof(Airplane))));

        // 和 version 1 一样，把大内存切块然后用指针串起来
        for (int i = 1; i < kBlockSize; ++i) {
            new_block[i].next = &new_block[i+1];
        }
        // 这步一定不要省略，是前面指针 p 判断的依据
        new_block[kBlockSize-1].next = nullptr;

        // 等价于：p = &p[0]
        p = new_block;
        // 第一个结点分配出去了，所以空闲链表是从第二个开始
        sHeadOfFreeList = &new_block[1];
    }

    return p;
}

void Airplane::operator delete(void *ptr, size_t size)
{
    if (sHeadOfFreeList == nullptr)
        return ;

    // 同理，防止子类调用
    if (size != sizeof(Airplane)) {
        ::operator delete(ptr);
        return ;
    }

    // 和 version 1 一样的
    Airplane *carcass = static_cast<Airplane*>(ptr);
    carcass->next = sHeadOfFreeList;
    sHeadOfFreeList = carcass;
}

void test()
{
    cout << sizeof(Airplane) << endl;

    constexpr size_t N = 100;
    Airplane *p[N];

    for (int i = 0; i < N; ++i) {
        p[i] = new Airplane;
    }

    p[1]->set(1000, 'a');
    p[3]->set(2000, 'b');
    p[5]->set(3000, 'c');

    cout << p[1]->get_miles() << " " << p[1]->get_type() << endl;
    cout << p[3]->get_miles() << " " << p[3]->get_type() << endl;
    cout << p[5]->get_miles() << " " << p[5]->get_type() << endl;

    for (int i = 0; i < 10; ++i)
        cout << p[i] << endl;

    for (int i = 0; i < N; ++i)
        delete p[i];
}

/*
 * result without per-class allocator
 * 16
 * 1000 a
 * 2000 b
 * 3000 c
 * 0x7fe1f7c05df0
 * 0x7fe1f7c05e00
 * 0x7fe1f7c05e10
 * 0x7fe1f7c05e20
 * 0x7fe1f7c05e30
 * 0x7fe1f7c05e40
 * 0x7fe1f7c05e50
 * 0x7fe1f7c05fe0
 * 0x7fe1f7c05ff0
 * 0x7fe1f7c06000
 *
 * result with per-class allocator
 * 16
 * 1000 a
 * 2000 b
 * 3000 c
 * 0x7fc063009800
 * 0x7fc063009810
 * 0x7fc063009820
 * 0x7fc063009830
 * 0x7fc063009840
 * 0x7fc063009850
 * 0x7fc063009860
 * 0x7fc063009870
 * 0x7fc063009880
 * 0x7fc063009890
 *
 * 带有 allocator 的内存地址按照预期那样输出了
 * 而没有 allocator 的内存地址有点儿乱，这么看，真就对 malloc 做优化了呗，也是塞进了一个内存池？
 */
```

## version 3, static allocator

version 3 没什么 tirck，应用 design pattern of Strategy，把重复的部分抽离出来。

``` c++
// version 3, static allocator
class allocator {
public:
    // 业界统一的接口
    void* allocate(size_t size);
    void deallocate(void *ptr, size_t size);
private:
    // version 3 又用回了指针啊
    struct obj {
        struct obj *next;
    };

    obj *free_store = nullptr;
    static constexpr int kChunk = 5;   // 这里只是为了观察现象容易，设置得比较小，实际上应该给予经验或统计 给定
};

void* allocator::allocate(size_t size)
{
    obj *p;

    if (!free_store) {
        p = free_store = static_cast<obj*>(malloc(kChunk * size));

        for (int i = 0; i < kChunk - 1; ++i) {
            // 注意这儿操作指针是先将其转为char
            p->next = (obj*)((char*)p + size);
            p = p->next;
        }
        p->next = nullptr;
    }

    p = free_store;
    free_store = free_store->next;

    return p;
}

void allocator::deallocate(void *ptr, size_t size)
{
    (static_cast<obj*>(ptr))->next = free_store;
    free_store = (static_cast<obj*>(ptr));
}


class Foo {
public:
    Foo(long v) : value(v) { }

    // 实际分配内存的任务交由 allocator 来进行
    static void* operator new(size_t size)
    {
        return alloc.allocate(size);
    }
    static void operator delete(void *ptr, size_t size)
    {
        return alloc.deallocate(ptr, size);
    }

    inline long get_value()
    {
        return value;
    }

private:
    long value;
    string name;
    static allocator alloc;
};

// static member data should initialized
allocator Foo::alloc;

void test()
{
    Foo *p[100];

    cout << "sizeof(allocator): " << sizeof(allocator) << endl;
    cout << "sizeof(Foo): " << sizeof(Foo) << endl;

    for (int i = 0; i < 20; ++i) {
        p[i] = new Foo(i);
        cout << p[i] << " " << p[i]->get_value() << endl;
    }

    for (int i = 0; i < 20; ++i) {
        delete p[i];
    }
}
```

## version 4, macro for static allocator

static allocator 的用法很制式，故而用 macro 适当偷懒。

``` c++
// DECLARE_POOL_ALLOC  -- used in class definiton
#define DECLARE_POOL_ALLOC()\
public:\
    void* operator new(size_t size) { return alloc.allocate(size); }\
    void operator delete(void *ptr, size_t size) { alloc.deallocate(ptr, size); }\
private:\
    static test07::allocator alloc;

// IMPLEMENT_POOL_ALLOC  -- used in class implementation file
#define IMPLEMENT_POOL_ALLOC(class_name)\
    test07::allocator class_name::alloc;

class Goo {
public:
    Goo(string s) : name(s) { };
    string get_name() {
        return name;
    }
private:
    DECLARE_POOL_ALLOC()

    string name;
};
IMPLEMENT_POOL_ALLOC(Goo)
```

# std::allocator

## 西北有高楼，上与浮云齐

![](/Picbed/2021_10/1010_00.png)

GCC 2.9 s td::alloc 的实现细节：

alloc维护18条链表，分别负责 $8 * n bytes$ 的内存空间分配，每一次会用 malloc 申请 $20 * 8 * n * 2   bytes$ 的空间，前一半留待使用，后一半是预分配，而超过 $128（即8*16）bytes$ 的空间申请，交由malloc 执行。

因而，分配空间的时候需要对齐，在此处也找到了实现依据，虽然好像也有出入的说。

![](/Picbed/2021_10/1010_01.png)

embended pointer 我能理解，但不理解为什么hj说可以把 `char client_data[1];` 去掉，并改成 `struct`，改完了还能访问到数据字段么？

## 运行一瞥

”这些连环图片是怎么来的，根据执行的数据来的，数据是怎么来的呢，你可以用调试器进去看，看它的变化。我当时，反正总之我也没用调试器，我直接改源代码。“  ~~学到了(不是~~

要注意的是，pool并不是说整个设计，而是预分配的那一部分，另外，碎片的处理也值得关注。

![](/Picbed/2021_10/1018_00.png)

![](/Picbed/2021_10/1018_01.png)

![](/Picbed/2021_10/1018_02.png)

![](/Picbed/2021_10/1018_03.png)

![](/Picbed/2021_10/1018_04.png)

![](/Picbed/2021_10/1018_05.png)

![](/Picbed/2021_10/1018_06.png)

![](/Picbed/2021_10/1018_07.png)

![](/Picbed/2021_10/1018_08.png)

![](/Picbed/2021_10/1018_09.png)

修改源代码，模拟内存耗尽的情况，来测试 `std::alloc` 的处理策略。

哇，best fit 分配策略啊，尽量保留大的空闲区块。

![](/Picbed/2021_10/1018_10.png)

![](/Picbed/2021_10/1018_11.png)

![](/Picbed/2021_10/1018_12.png)

![](/Picbed/2021_10/1018_13.png)

# malloc/free

# loki::allocator

# other issues



