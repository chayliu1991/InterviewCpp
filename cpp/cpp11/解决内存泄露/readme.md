# std::shared_ptr 

std::shared_ptr 使用引用计数，每个 std::shared_ptr 的拷贝都指向相同的内存，在最后一个 std::shared_ptr 析构时，内存才会被释放。

## 基本用法

### 初始化

- 构造函数

- `std::make_shared<T>`，优先使用，更加高效

- reset，原来的智能指针如果有值，引用计数会减1

```
std::shared_ptr<int> p1(new int(1));
std::shared_ptr<int> p2;
p2.reset(new int(2));
std::shared_ptr<int> p3 = std::make_shared<int>(3);
```

不能将原始指针直接赋值给智能指针：

```
std::shared_ptr<int> p4 = new int(4); //@  错误
```

### 获取原始指针

```
std::shared_ptr<int> sp = std::make_shared<int>(0);
int* pOri = sp.get();
```

### 指定删除器

智能指针初始化时可以指定删除器，当引用计数为0时会自动调用删除器：

```
void delete_func(int* p)
{
	delete p;
}

struct DeleteFunctor
{
	void operator()(int * p)
	{
		std::cout << "delete..." << std::endl;
		delete p;
	}
};

int main()
{
	//@ 使用函数指针
	std::shared_ptr<int> sp1(new int(1), delete_func);

	//@ 使用 lambda 表达式
	std::shared_ptr<int> sp2(new int(1), [](int *p) {delete p; });

	//@ 使用仿函数
	std::shared_ptr<int> sp3(new int(1), DeleteFunctor());

	std::shared_ptr<int> sp3 = std::make_shared<int, DeleteIntPtr>(0); //@ 错误，没有这种形式

	return 0;
}
```

管理动态数组：

```
//@ 自定义删除器
std::shared_ptr<int> sp(new int[10]{1,2,3,4,5,6,7,8,9,0}, [](int*p) {delete[] p; });
std::cout << *sp.get() << std::endl;  //@ OK
std::cout << *(sp.get() + 3) << std::endl; //@ OK

std::cout << sp[0] << std::endl;  //@ 无法使用下标引用
```

std::default_delete 内部通过调用 delete 实现功能，所以也可以写成：

```
std::shared_ptr<int> sp(new int[10]{ 1,2,3,4,5,6,7,8,9,0 },std::default_delete<int[]>());
```

封装 make_shared_array 函数：

```
template <typename T>
std::shared_ptr<T> make_shared_array(size_t  size)
{
	return std::shared_ptr<T>(new T[size], std::default_delete<T[]>());
}

std::shared_ptr<int> p = make_shared_array<int>(10);
std::shared_ptr<char> p = make_shared_array<char>(100);
```

## 注意事项

### 禁止使用一个原始指针初始化多个 std::shared_ptr

```
int * ptr = new int;
std::shared_ptr<int> sp1(ptr);
std::shared_ptr<int> sp2(ptr); //@ 会导致 double free
```

### 禁止在函数实参中创建 std::shared_ptr

```
func(std::shared_ptr<int>(new int),g());
```

因为 C++ 函数的参数计算顺序在不同的编译器有不同的调用约定，一般时从右向左，也可能从左到右。可能的步骤：

- 先调用 new int
- 再调用 g() 
- 再创建  std::shared_ptr

如果 g() 函数调用时发生异常，则  std::shared_ptr 还没有创建，new int 的内存就会泄漏。正确的写法：

```
std::shared_ptr<int> sp(new int)
f(sp,g());
```

### 通过 shared_from_this() 返回 this 指针

不要将 this 指针作为 std::shared_ptr 返回，因为 this 本质上是一个裸指针，因此可能导致重复析构，其本质就是使用同一个裸指针初始化多个 std::shared_ptr：

```
struct A
{
	std::shared_ptr<A> self()
	{
		return std::shared_ptr<A>(this); //@ 不要这样做
	}
};

int main(void)
{
	//@ 使用同一个指针 this 构造了两个智能指针，而它们之间没有任何联系
	//@ 离开作用域之后 sp1 sp2 都将析构导致重复析构的问题
	std::shared_ptr<A> sp1(new A);
	std::shared_ptr<A> sp2 = sp1->self();

	return 0;
}
```

正确返回 this 的 std::shared_ptr  的方法是：让目标类通过派生 `std::enable_shared_from_this<T> ` 类，然后使用基类的成员函数 shared_from_this 来返回 this 的  std::shared_ptr：

```
struct A : std::enable_shared_from_this<A>
{
	std::shared_ptr<A> self()
	{
		return shared_from_this();
	}
};

int main(void)
{
	std::shared_ptr<A> sp1(new A);
	std::shared_ptr<A> sp2 = sp1->self();
	return 0;
}
```

### 避免循环引用

智能指针的循环引用将导致内存泄漏：

```
struct A;
struct B;

struct A {
	std::shared_ptr<B> bPtr;
	~A() { std::cout << "A is deleted!" << std::endl; }
};

struct B {
	std::shared_ptr<A> aPtr;
	~B() { std::cout << "B is deleted!" << std::endl; }
};


int main(void)
{
	std::shared_ptr<A> aP(new A);
	std::shared_ptr<B> bP(new B);
	aP->bPtr = bP;
	bP->aPtr = aP;

	return 0;
}
```

循环引用导致 aP，bP 的引用计数是2，在离开作用域之后， aP，bP 的引用计数都减少为1，并不会减少为0，导致两个指针都不会被析构，产生了内存泄漏。

解决办法是将 A 和 B 中任何一个成员变量改成 std::weak_ptr。

# std::unique_ptr 

std::unique_ptr 是一个独占的智能指针，它不允许其它的智能指针共享其内部的指针，不允许拷贝和赋值，只能移动。

```
std::unique_ptr<int> myPtr(new int(10));
std::unique_ptr<int> otherPtr = myPtr; //@ 错误，不能复制
std::unique_ptr<int> otherPtr2 = std::move(myPtr); //@ OK
```

## make_unique

实现 make_unique：

```
//@ 支持普通指针
template <typename T, typename...Args>
inline typename std::enable_if<!std::is_array<T>::value, std::unique_ptr<T>>::type
make_unique(Args&&...args)
{
	return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

//@ 支持动态数组
template <typename T>
inline typename std::enable_if<std::is_array<T>::value && std::extent<T>::value == 0, std::unique_ptr<T>>::type
make_unique(size_t size)
{
	typedef typename std::remove_extent<T>::type U;
	return std::unique_ptr<T>(new U[size]());
}

//@ 过滤掉定长数组
template <typename T, typename...Args>
typename std::enable_if<std::extent<T>::value != 0, void>::type make_unique(Args&&...) = delete;


std::unique_ptr<int> p = make_unique<int>(10); //@ OK
std::unique_ptr<int[]> pArray1 = make_unique<int[]>(10); //@ OK
std::unique_ptr<int[]> pArray2 = make_unique<int[10]>; //@ 错误，不能创建定长数组的 std::unique_ptr
```

## std::unique_ptr 与  std::shared_ptr

- std::unique_ptr  具有独占性
- std::unique_ptr  可以直接指向一个数组

```
std::unique_ptr<int[]> up(new int[3]{1,2,3}); //OK;
std::cout << up[0] << std::endl;  //@ OK
std::cout << up[1] << std::endl;  //@ OK
std::cout << up[2] << std::endl;  //@ OK

std::shared_ptr<int[]> sp(new int[10]); //@ 错误，不合法
```

- std::unique_ptr  指定删除器时需要确定删除器的类型

```
std::shared_ptr<int> ptr(new int(1), [](int *p) { delete p; }); //@ OK
std::unique_ptr<int> ptr2(new int(1), [](int *p) { delete p; }); //@ 错误

//@ lambda 没有捕捉变量时是正确的，因为没有捕获变量的 lambda 可以转换成函数指针，如果捕捉了变量则不可以
std::unique_ptr<int, void(*)(int*)> ptr3(new int(1), [](int *p) { delete p; });

//@ 如果希望 std::unique_ptr 的删除器支持 lambda 则应该写成：
std::unique_ptr<int, std::function<void(int*)>> ptr4(new int(1), [&](int *p) { delete p; });

//@ 使用仿函数作为删除器
struct MyDeleter
{
    void operator()(int*p)
    {
        std::cout << "delete" << std::endl;
        delete p;
    }
};
std::unique_ptr<int, MyDeleter> ptr5(new int(1));
```

# std::weak_ptr 

- 弱引用指针 std::weak_ptr 用来监视 std::shared_ptr 不会使引用计数增加，也不管理  std::shared_ptr 内部的指针，主要是监视 std::shared_ptr 的生命周期

- std::weak_ptr 没有重载 * 和 ->，因为它不共享指针，不能操作资源
- std::weak_ptr 可以用来解决 std::shared_ptr 的循环引用问题

## 基本用法

### use_count 

获取当前观测 std::shared_ptr 的引用计数：

```
std::shared_ptr<int> sp(new int(10));
std::weak_ptr<int> wp(sp);
std::cout << wp.use_count() << std::endl;  //@ 1
std::shared_ptr<int> sp2 = sp;
std::cout << wp.use_count() << std::endl;  //@ 2
```

### expired

判断所观测的 std::shared_ptr 是否释放：

```
std::shared_ptr<int> sp(new int(10));
std::weak_ptr<int> wp(sp);
if (wp.expired())
	std::cout << "std::weak_ptr invalid in:" << __LINE__ << std::endl;

sp.reset();
if (wp.expired())
	std::cout << "std::weak_ptr invalid in:" << __LINE__ << std::endl; //@ expired
```

### lock 

获取监视的 std::shared_ptr：

- 返回 std::shared_ptr
- std::shared_ptr 的引用计数加1

```
std::weak_ptr<int> g_wptr;

void func()
{
	if (g_wptr.expired())
	{
		std::cout << "g_wptr is expired" << std::endl;
	}
	else
	{
		std::cout << "use count before lock:" << g_wptr.use_count() << std::endl;
		auto spt = g_wptr.lock();  //@ 返回一个 std::shared_ptr,引用计数加1
		std::cout << "dereference:" << *spt << std::endl;
		std::cout << "use count after lock:" << g_wptr.use_count() << std::endl;
	}
}

int main()
{
	{
		auto sp = std::make_shared<int>(42);
		g_wptr = sp;
		func();
	}

	func();

	return 0;
}
```

##  std::enable_from_this 原理

本质上：

- std::enable_shared_from_this 内部有一个  std::weak_ptr，这个 std::weak_ptr 用来观测 this 指针的 std::shared_ptr
- 调用 shared_from_this  实际上内部调用了 std::weak_ptr 的 lock 方法返回一个 std::shared_ptr

## 解决循环引用

std::shared_ptr 的循环引用导致的内存泄露问题可以通过 std::weak_ptr 解决：

```
struct A;
struct B;

struct A {
	std::weak_ptr<B> bPtr;
	~A() { std::cout << "A is deleted!" << std::endl; }
};

struct B {
	std::shared_ptr<A> aPtr;
	//@ std::weak_ptr<A> aPtr;
	~B() { std::cout << "B is deleted!" << std::endl; }
};

int main(void)
{
	std::shared_ptr<A> aP(new A);
	std::shared_ptr<B> bP(new B);
	aP->bPtr = bP;
	bP->aPtr = aP;

	return 0;
}
```

