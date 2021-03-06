---
title: 智能指针笔记
date: 2015-04-11 12:26:26
tags: 
- CPP
categories:
- CPP
---


有些错误是编译器查不到的, 这种错误是最可怕的, 当项目大了之后, 
即使用 Valgrind 也很难定位,
因为裸指针在团队合作中使用很容易导致其他成员忘记释放或多次释放, 所以在团队合作中一般使用智能指针. 

而智能指针用的不好, 结果可能适得其反.

所以我们聊一下智能指针的几点注意事项.

总结自 C++ Primer.



# 一个简单的包含删除器的例子演示

``` c++
#include<iostream>
#include<functional>
#include<memory>

using std::cout;
using std::endl;
using std::bind;
using namespace std::placeholders;

int testBind( int* a, int b, int c )
{
	cout << *a + b + c << endl;
	return *a;
}

struct Foo
{
	Foo() = default;
	Foo( const Foo & a )
	{
		data = a.data;
		std::cout << "复制构造" << std::endl;
	}
	void print_sum( int n1, int n2 )
	{
		std::cout << n1 + n2 << '\n';
	}
	int data = 10;
};

int main()
{
	//绑定类成员函数用对象的指针
	Foo foo;
	auto f3 = std::bind( &Foo::print_sum, &foo, 95, _1 );
	f3( 5 );

	auto check_testBind = std::bind( testBind, std::placeholders::_1, 3, 9 );
	int * p = new int( 7 );
	cout << check_testBind( p ) << endl;

	shared_ptr<int> pi( new int(),
		check_testBind );
	*pi = 88;

	shared_ptr<int> pii( new int( 12 ),
		std::bind( testBind, std::placeholders::_1, 32, 19 ) );

	std::function< int( int* ) > test_function_bind =
		std::bind( testBind, std::placeholders::_1, 331, 9 );

	cout << "test_function_bind( p, 331, 9 ) = " << test_function_bind( p ) << endl;;

	shared_ptr<int> piii( new int( 112 ),
		test_function_bind );

	return 0;
}
```

打印结果 :

	100
	119
	7
	347
	test_function_bind( p, 331, 9 ) = 7
	452
	63
	100
	请按任意键继续. . .





# 智能指针陷阱

智能指针可以提供对动态分配的内存安全而又方便的管理，但这建立在正确使用的
前提下 。 为了正确使用智能指针，我们必须坚持一些基本规范 :

- 不使用相同的内置指针值初始化(或 reset) 多个智能指针 。
- 不 delete get ( ) 返 回的指针 。
- 不使用 get () 初始化或 reset 另 一 个智能指针 。
- 如果你使用 get () 返 回的指针，记住当最后一个对应的智能指针销 毁 后， 你 的
指 针就 变为无 效 了 。
- 如果你使用智能指针管理的资源不是 new 分配的内存 ， 记住传递给它一个删除
器( 参见 12. 1.4 节 ， 第 415 页和 12. 1.5 节 ， 第 419 页)。


# 尽量用make_shared而非new


shared_ptr 可以协调对象的析构 ， 但这仅限于其自身的拷贝 ( 也 是 shared_ptr)
之间。

**这也是为什么我们推荐使用 make_shared 而不是 new 的原因 。**

**这样 ， 我们就能在分配对象的同时就将 shared_ptr 与之绑定，**
**从而避免了无意中将同一块内存绑定到多个独立创建的 shared_ptr 上 。(这是最容易犯的错)**

**总结 : 所以我们要尽量一开始就用make_shared来分配动态内存, 而不是先new一个出来, 再找机会将它转为智能指针.**

考虑下面对 shared_ptr 进行操作的函数 :

```  c++
// 在函数被调用时 ptr 被创建并初始化
void process(shared_ptr<int> ptr)
{
  // 使用 ptr
} // ptr 离 开作用域，被销毁
```

process 的参数是传值方式传递的，因此实参会被拷贝到 ptr 巾 。 拷贝 一 个 shared_ptr 
会递增其引用讨数，因此， 在 process 运行过程中，引用七| 数值至少为 2 。 当 process
结束时， ptr 的引用计数会边喊，但不会变为 0 。 因此，当用音11变 11 ptr 被销毁时， ptr
指向的内存不会被释放 。

使用此函数的正确方法是传递给它一个 shared_ptr :

```  c++
shared_ptr<int> p(new int(42)) ; // 引用计数为 1
process(p); // 拷贝 p 会递增它的引用计数 ;在 process 中引用计数位为 2
int i = *p; // 正确:引用计数位为 1
```

虽然不能传递给 process 一 个内置指针，但可以传递 给它 一 个(临时的)
shared_ptr ， 这个 shared_ptr 是用 一个内 置指针显式构造的 。 但是，这样做很可能
会导致错误 :

```  c++
int *x(new int(1024)); // 危险 x 是一个普通指针，不是一个智能指针
process(x) ; // 错误 : 不能将 int* 转换为 一个 shared_ptr<int>
process(shared_ptr<int>(x)); // 合法的，但内存会被释放!
int j = *x ; //未定义的 x 是一个空悬指针!
```

在上面的调用中 ， 我们将一个临时 shared_ptr 传递给 process 。 当这个调用所在的表
达式结束时，这个临时对象就被销毁了 。 销毁这个临时变量会递减引用计数，此时引用计
数就变为 0 了 。 因此，当临时对象被销毁时 ， 它所指向的内存会被释放 。
但 x 继续指 向 (已经释放的)内存，从而变成一个空悬指针。如果试图使用 x 的值，
其行为是未定义的 。

当将一个 shared_ptr 绑定到一个普通指针时 ， 我们就将内存的管理责任交给了这
个 shared_ptr 一旦这样做 了 ， 我们就不应该再使用内置指针来访问 shared_ptr 所
指向的内存了 


# 不要使用 get 初始化另一个智能指针或为智能指针赋值

智能指针类型定义了 一个名为 get 的函数(参见表 1 2. J )，它返回一个内置指针，
指向智能指针管理的对象 。 此函数是为了这样一种情况而设计的 : 我们需要向不能使用智
能指针的代码传递一个内置指针。使用 get 返回的指告| 的代码不能 delete 此指针 。
虽然编译器不会给出错误信息 ， 但将另一个智能指针也绑定到 get 返回的指针上是
错误的 :

``` c++
shared_ptr<int> p(new int(42)) ; // 引 用计数为 1
int *q = p . get() ; // 正确 · 但使用 q 时妥注意，不要让它管理的指针被释放
{ // 新程序块
// 未定义:两个独立的 shared_ptr 指 向 相同的内存
    shared ptr<int> (q) ;
} // 程序块结束， q 被销毁 ， 它指向的内存被待放
int foo = *p ; // 未定义 p 指向的内存 已 经被释放了
```

在本例中， p 和 q 指 向相同的内存。由于它们是相互独立创建的，因此各自的引用计数都
是 1。 当 q 所在的程序块结束时 ， q 被销毁 ， 这会导致 q 指向的内存被释放 。 从而 p 变成
一个空悬指针，意味着当我们试图使用 p 时，将发生未定义的行为 。 而且 ， 当 p 被销毁时 ，
这块内存会被第二次 delete 。

get 用来将指针的访问权限传递给代码，你只有在确定代码不会 get. 
特别是，永远不要用 get 初始化另一个智能指针 del ete 指针或者为另一个智能指针赋值.

