# type_traits

type_traits 提供了编译器计算，查询，判断，转换和选择的帮助类。在一定程序上可以消除冗长的 switch-case 或者 if-else 的语句。type_traits  的类型判断在编译期间就可以检查出是否是正确的类型，以便编写更安全的代码。

 ## std::integral_constant

在 C++ 11中定义编译期常量，只需要从 std::integral_constant 派生：

```
template <typename T>
struct GetLeftSize : std::integral_constant<int, 1>
{
};

int main()
{
	auto res = GetLeftSize<int>::value;  //@ res = 1
	return 0;
}
```

编译期 true 和 false 类型 true_type 和 false_type：

```
typedef std::integral_constant<bool, true> true_type; 
typedef std::integral_constant<bool, false> false_type;
```

## 类型判断的 type_traits

这些 type_traits 从 std::integral_constant 派生，用来检查模板类型是否为某种类型，通过这些 traits 可以获取编译期检查的 bool 值结果：

```
template <typename T>
struct is_void;

template <typename T>
struct is_array;

template <typename T>
struct is_reference;

template <typename T>
struct is_member_pointer;

...
```

使用：

```
std::cout << "int: " << std::is_const<int>::value << std::endl; //@ false
std::cout << "const int: " << std::is_const<const int>::value << std::endl; //@ true
std::cout << "const int&: " << std::is_pointer<const int*>::value << std::endl; //@ true
std::cout << "const int*: " << std::is_reference<const int&>::value << std::endl; //@ true
```

## 判断两个类型之间关系的 type_traits

```
template <typename T, typename U>
struct is_same;

template <typename Base, typename Derived>
struct is_base_of; 

template <typename From, typename To>
struct is_convertible; 
```

使用：

```
std::cout << std::is_same<int, int>::value << std::endl;  //@ true
std::cout << std::is_same<int, unsigned int>::value << std::endl;  //@ false
std::cout << std::is_same<int, int&>::value << std::endl;  //@ false

std::cout << std::is_base_of<Base, Derived>::value << std::endl; //@ true
std::cout << std::is_base_of<Derived, Base>::value << std::endl; //@ false

std::cout << std::is_convertible<Base*, Derived*>::value << std::endl; //@ false
std::cout << std::is_convertible<Derived*, Base*>::value << std::endl; //@ true
```

## 类型转换的 type_traits

```
template<typename T>
struct remove_const; 

template<typename T>
struct add_pointer;

 template<typename T>
 struct decay; 
 
 template<typename T>
 struct common_type;
 
 template<typename T>
 struct remove_extent; 
 
 ...
```

使用：

```
std::cout << std::is_same<const int, std::add_const<int>::type>::value << std::endl; //@ true
std::cout << std::is_same<int, std::remove_const<const int>::type>::value << std::endl; //@ true
std::cout << std::is_same<int&, std::add_lvalue_reference<int>::type > ::value << std::endl; //@ true
std::cout << std::is_same<int&&, std::add_rvalue_reference<int>::type > ::value << std::endl; //@ true
std::cout << std::is_same<int, std::remove_reference<int&>::type > ::value << std::endl; //@ true
std::cout << std::is_same<int, std::remove_reference<int&&>::type > ::value << std::endl; //@ true
std::cout << std::is_same<int*, std::add_pointer<int>::type > ::value << std::endl; //@ true

std::cout << std::is_same<int[2], std::remove_extent<int[][2]>::type > ::value << std::endl; //@ true
std::cout << std::is_same<int, std::remove_all_extents<int[][2][3]>::type > ::value << std::endl; //@ true

typedef std::common_type<unsigned char, short, int>::type NumericType; 
std::cout << std::is_same<int, NumericType>::value << std::endl; //@ true
```

移除引用和 cv 属性：

```
template<typename T>
typename std::remove_cv<typename std::remove_reference<T>::type>::type*
Create()
{
	typedef std::remove_cv<typename std::remove_reference<T>::type>::type U;
	return new U();
}
```

可以使用 decay 来简化：

```
template<typename T>
typename std::decay<T>::type* Create()
{
	typedef std::decay<T>::type U;
	return new U();
}
```

std::decay 除了用于普通类型外，std::decay 还可以用于数组和函数，具体的转换规则：

- 先移除 T 类型的引用，得到 U 类型，U 定义为 `remove_reference<T>::type`
- 如果 `is_array<U>::value`  为 true，修改类型 type 为 `remove_extent<U>::type*`
- 否则，如果 `is_function<U>::value` 为 true，修改类型 type 将为 `add_pointer<U>::type`
- 否则，修改类型 type 为 `remove_cv<U>::type`

利用这一点可以将函数变成函数指针类型，从而将函数指针变量保存起来，以便在后面延迟执行：

```
template<typename F>
struct SimpFunction
{
	using FnType = typename std::decay<F>::type;
	SimpFunction(F& f) : m_fn(f)
	{
	}

	void Run()
	{
		m_fn();
	}

	FnType m_fn;
};
```

## 根据条件选择的 type_traits

std::condition 在编译期根据一个判断式选择两个类型中的一个，和条件表达式的语义类似，类似于一个三元表达式：

```
template<bool B,typename T,typename F>
struct conditional;
```

如果 B 为 true，则 conditional::type 为T，否则为 F。

```
typedef std::conditional<true, int, float>::type A;  //@ int
typedef std::conditional<false, int, float>::type B; //@ float
typedef std::conditional < (sizeof(long long) > sizeof(long double)), long long, long double > ::type max_size_t;
std::cout << typeid(max_size_t).name() << std::endl;
```

## 获取可调用对象返回类型的 type_traits

通常做法：

```
template <typename F,typename Arg>
auto Func(F f, Arg arg)->decltype(f(arg))
{
	return f(arg);
}
```

但是有些时候不能通过 decltype 来获取类型，比如：

```
class A
{
	A() = delete;
public:
	int operator()(int i)
	{
		return i;
	}
};

int main()
{
	decltype(A()(0)) i = 5; //@ 错误，A 没有默认构造函数
	return 0;
}
```

对于这种没有默认构造函数的类型，如果希望能推导其成员函数的返回类型，则需要借助 std::declval ：

```
decltype(std::declval<A>()(std::declval<int>())) j = 6;
```

std::declval 能获取任何类型的临时值，而不管它是不是有默认构造函数。

虽然结合返回类型后置，decltype 和 declval 能够解决推断函数返回类型的问题，但不够简洁，C++ 11 提供了另外一个 traits ——  std::result_of，用来在编译期获取一个可调用对象的返回类型。 std::result_of 原型：

```
//@ 第一个模板参数为调用对象的类型
//@ 第二个模板参数为参数的类型
template <typename F,typename ... ArgTypes>
class std::result_of<F(ArgTypes...)>; 
```

使用：

```
int fn(int) { return int(); }
typedef int(&fn_ref)(int);
typedef int(*fn_ptr)(int);
struct fn_class { int operator()(int i) { return i; } };

int main()
{
	typedef std::result_of<decltype(fn)& (int)>::type A;
	typedef std::result_of<fn_ref(int)>::type B;
	typedef std::result_of<fn_ptr(int)>::type C;
	typedef std::result_of<fn_class(int)>::type D;

	std::cout << std::is_same<int, A>::value << std::endl;  //@ true
	std::cout << std::is_same<int, B>::value << std::endl;  //@ true
	std::cout << std::is_same<int, C>::value << std::endl;  //@ true
	std::cout << std::is_same<int, D>::value << std::endl;  //@ true

	return 0;
}
```

`std::result_of<F(ArgTypes...)>;`  要求 F 必须是一个可调用对象，不能是一个函数类型，因此下面的方式是错误的：

```
typedef std::result_of<decltype(fn)(int)>::type A;
```

## 根据条件禁用或启用某种或某些 type_traits

编译器在匹配重载函数时会匹配所有的重载函数，找到一个最精确匹配的函数，在匹配过程中可能会有一些失败的尝试，当匹配失败时再重试匹配其它的重载函数，这个规则就是 SFINAE(substitution-failure-is-not-an-error)，替换失败并非错误。

 std::enable_if 利用 SFINAE 实现根据条件选择重载函数，其原型为：

```
template<bool B,class T = void>
struct enable_if;
```

使用：

```
template <class T>
typename std::enable_if<std::is_arithmetic<T>::value, T>::type foo(T t)
{
	return t;
}

int main()
{
	auto r1 = foo(1);
	auto r2 = foo(1.2);
	auto r3 = foo("test");

	return 0;
}
```

上面对模板参数 T 做了限定，即只能是 arithmetic 类型，如果为非 arithmetic  类型，则编译不过。

std::enable_if 还可以用于模板定义、类模板的特化、入参限定。

限定入参：

```
//@ 限定入参为 int 类型
template <typename T>
T foo1(T t, typename std::enable_if<std::is_integral<T>::value, int>::type = 0)
{
	return t;
}
foo1(1, 2);  //@ 编译通过
foo1(1, 2.1);  //@ 编译不通过
```

限定模板参数：

```\
//@ 对模板参数 T 做了限定，T 只能是 int 类型
template <typename T,class = typename std::enable_if<std::is_integral<T>::value>::type>
T foo2(T t)
{
	return t;
}
foo2(1); //@ 编译通过
foo2(2.2);  //@ 编译不通过
```

类模板特化：

```
template <typename T, class Enable = void>
class A;

//@ 模板特化，对模板参数做了限定，模板参数类型只能是浮点型
template <typename T>
class A<T, typename std::enable_if<std::is_floating_point<T>::value>::type> {};

A<double> a1; //@ 编译通过
A<int> a2;  //@ 编译不通过
```

可以通过判断式和非判断式来将入参分为两大类，从而满足所有的入参类型：

```
//@ 对于 arithmetic 类型返回 0
template<typename T>
typename std::enable_if<std::is_arithmetic<T>::value, int>::type foo(T t)
{
	std::cout << t << std::endl;
	return 0;
}

//@ 对于非 arithmetic 类型返回 1
template<typename T>
typename std::enable_if<!std::is_arithmetic<T>::value, int>::type foo(T& t)
{
	std::cout << typeid(T).name() << std::endl;
	return 1;
}

//@ 如果第二个模板参数是默认模板参数 void 类型，函数没有返回值时，后面的模板参数可以省略
template <typename T>
typename std::enable_if<std::is_arithmetic<T>::value>::type foo(T t)
{
	std::cout << typeid(T).name() << std::endl;
}
```

std::enable_if  可以实现强大的重载机制，可以利用这个特性来消除圈复杂度较高的 switch-case/if-else 语句：

```
template <typename T>
typename std::enable_if<std::is_arithmetic<T>::value, std::string>::type ToString(T t)
{
	return std::to_string(t);
}

template <typename T>
typename std::enable_if<!std::is_arithmetic<T>::value, std::string>::type ToString(T& t)
{
	return t;
}
```

# 可变参数模板

声明可变参数模板时需要在 class 或 typename 关键字前面加上 ...：

- 声明一个参数包，这个参数包中可以包含 0 个到任意个模板参数
- 在模板定义的右边，可以将参数包展开成一个一个独立的参数



## 可变参数模板函数

```
template <typename ...T>
void func(T ...args)
{
	std::cout << sizeof...(args) << std::endl; //@ 不要写成 sizeof(...(args))
}

int main()
{
	func();  //@ 0
	func(1);  //@ 1
	func(2, "hello", 3.14);  //@ 3

	return 0;
}
```

参数包展开的两种方法：

- 递归模板函数来将参数包展开
- 通过逗号表达式和初始化列表方式展开

### 递归模板函数来将参数包展开

递归函数方式展开参数包：

````
//@ 递归终止函数
void print()
{
	std::cout << "----- ending -----" << std::endl;
}

//@ 参数包展开过程中递归调用自己，直到参数为空时调用非模板的递归终止函数
template <typename T,typename ...Args>
void print(T head, Args ... rest)
{
	std::cout << "parameter:" << head << std::endl;
	print(rest...);
}
int main()
{
	print(1,2,3,4,5,6); 
	print(2, "hello", 3.14);

	return 0;
}
````

还可以采用 type_traits 来实现：

```
template <std::size_t I = 0, typename Tuple>
typename  std::enable_if<I == std::tuple_size<Tuple>::value>::type  printTP(Tuple t)
{
	std::cout << "----- ending -----" << std::endl;
}

template <std::size_t I = 0, typename Tuple>
typename  std::enable_if<I < std::tuple_size<Tuple>::value>::type  printTP(Tuple t)
{
	//@ 通过递增索引值，获取当前值
	std::cout << std::get<I>(t) << std::endl;
	printTP<I + 1>(t);
}

template <typename ...Args>
void print(Args... args)
{
	//@ 转换成 std::tuple
	printTP(std::make_tuple(args...));
}
```

### 通过逗号表达式和初始化列表方式展开

逗号表达式和初始化列表方式展开参数包：

采用逗号表达式和初始化列表则需要定义终止函数：

```
void printArg(T t)
{
	std::cout << t << std::endl;
}

template <class ...Args>
void expand(Args... args)
{
	//@ printArg(args),0) 先执行 printArg(args) 再返回 0
	//@ 依次展开 printArg(args1),0)，printArg(args2),0)，printArg(args3),0)...
	//@ 最终创建元素都为0 的 arr[sizeof(args)]
	int arr[] = { (printArg(args),0)... };

	std::cout << "-------------------------------------" << std::endl;
	std::cout << sizeof(arr)/sizeof(int) << std::endl;
}
```

使用初始化列表代替数组：

```
template <class ...Args>
void expand(Args... args)
{
	std::initializer_list<int>{ (printArg(args),0)... };
}
```

使用 lambda 表达式进一步改进：

```
template <class ...Args>
void expand(Args... args)
{
	std::initializer_list<int>{([&] {std::cout << args << std::endl; }(),0)...};
}
```

## 可变参数模板类

 可变参数模板类的参数包展开需要通过：

- 模板特化
- 继承方式

### 模板特化

模板递归和特化方式展开参数包：

```
//@ 前向声明，声明 Size 是一个可变参模板类
template <typename ...Args>
struct Size;

//@ 定义一个部分展开的可变参模板类，如何递归展开参数包
//@ 这里也限定了 Size 类必须至少有一个参数，因为可变参模板类可以有 0 个方式，0 个参数没有意义时可以通过这样的声明来限制
template <typename First, typename...Rest>
struct Size<First, Rest...>
{
	enum { value = Size<First>::value + Size<Rest...>::value };
};

//@ 特化的终止类
template <typename Last>
struct Size<Last>
{
	enum { value = sizeof(Last) };
};

int main()
{
	auto sz = Size<int, double, short>::value; 
	return 0;
}
```

可以去掉前向声明：

```
template <typename First, typename...Rest>
struct Size
{
	enum { value = Size<First>::value + Size<Rest...>::value };
};

//@ 特化的终止类
template <typename Last>
struct Size<Last>
{
	enum { value = sizeof(Last) };
};
```

可以使用 std::integral_constant 来消除 value 的定义：

```
//@ 前向声明
template <typename ...Args>
struct Size;

//@ 基本定义
template <typename First,typename ...Rest>
struct Size<First, Rest...> : std::integral_constant<int, Size<First>::value + Size<Rest...>::value>
{
};

//@ 递归终止
template <typename Last>
struct Size<Last> : std::integral_constant<int, sizeof(Last)>
{
};

int main()
{
	auto sz = Size<int, double, short>::value;
	return 0;
}
```

### 继承方式

继承方式展开参数包：

```
//@ 整型序列的定义
template<int...>
struct IndexSeq {};

//@ 继承方式，开始展开参数包
//@ MakeIndexes 继承于自身的一个特化的模板类，这个特化的模板类同时也在展开参数包
//@ 参数包的展开过程是通过继承发起的，直到遇到特化的终止条件展开过程才结束
//@ MakeIndexes<3,IndexSeq<>> : MakeIndexes<2,IndexSeq<2>>{}
//@ MakeIndexes<2,IndexSeq<2>> : MakeIndexes<1,IndexSeq<1,2>>{}
//@ MakeIndexes<1,IndexSeq<1,2>> : MakeIndexes<0,IndexSeq<0,1,2>>{typedef }
template<int N, int... Indexes>
struct MakeIndexes : MakeIndexes<N - 1, N - 1, Indexes...> {};

//@ 模板特化，终止展开参数包的条件
template<int... Indexes>
struct MakeIndexes<0, Indexes...>
{
	typedef IndexSeq<Indexes...> type;
};

int main()
{
	using T = MakeIndexes<3>::type;
	std::cout << typeid(T).name() << std::endl; //@ struct IndexSeq<0,1,2>

	return 0;
}
```

# Optional  的实现

`Optional<T>`  内部可能存储了 T 类型，也可能没有存储 T 类型，只有当 Optional  被 T 初始化之后，Optional  才是有效的，否则无效。

```
#pragma once
#include <type_traits>
#include <utility>
#include <stdexcept>

template<typename T>
class Optional
{
    using DataType = typename std::aligned_storage<sizeof(T), std::alignment_of<T>::value>::type;

public:
    Optional() : has_init_(false) {}

    Optional(const T& v)
    {
        create(v);
    }

    Optional(T&& v) : has_init_(false)
    {
        create(std::move(v));
    }

    ~Optional()
    {
        destroy();
    }

    Optional(const Optional& other) : has_init_(false)
    {
        if (other.is_init())
            assign(other);
    }

    Optional(Optional&& other) : has_init_(false)
    {
        if (other.is_init())
        {
            assign(std::move(other));
            other.destroy();
        }
    }

    Optional& operator=(Optional &&other)
    {
        assign(std::move(other));
        return *this;
    }

    Optional& operator=(const Optional &other)
    {
        assign(other);
        return *this;
    }

    template<class... Args>
    void emplace(Args&&... args)
    {
        destroy();
        create(std::forward<Args>(args)...);
    }

    bool is_init() const { return has_init_; }

    explicit operator bool() const 
    {
        return is_init();
    }

    T& operator*()
    {
        if(is_init())
        {
            return *((T*) (&data_));
        }

        throw std::logic_error{"try to get data in a Optional which is not initialized"};
    }

    const T& operator*() const
    {
        if(is_init())
        {
            return *((T*) (&data_));
        }

        throw std::logic_error{"try to get data in a Optional which is not initialized"};
    }

    T* operator->()
    {
        return &operator*();
    }

    const T* operator->() const
    {
        return &operator*();
    }

    bool operator==(const Optional<T>& rhs) const
    {
        return (!bool(*this)) != (!rhs) ? false : (!bool(*this) ? true : (*(*this)) == (*rhs));
    }

    bool operator<(const Optional<T>& rhs) const
    {
        return !rhs ? false : (!bool(*this) ? true : (*(*this) < (*rhs)));
    }

    bool operator!=(const Optional<T>& rhs)
    {
        return !(*this == (rhs));
    }

private:
    template<class... Args>
    void create(Args&&... args)
    {
        new (&data_) T(std::forward<Args>(args)...);
        has_init_ = true;
    }

    void destroy()
    {
        if (has_init_)
        {
            has_init_ = false;
            ((T*) (&data_))->~T();
        }
    }

    void assign(const Optional& other)
    {
        if (other.is_init())
        {
            copy(other.data_);
            has_init_ = true;
        }
        else
        {
            destroy();
        }
    }

    void assign(Optional&& other)
    {
        if (other.is_init())
        {
            move(std::move(other.data_));
            has_init_ = true;
            other.destroy();
        }
        else
        {
            destroy();
        }
    }

    void move(DataType&& val)
    {
        destroy();
        new (&data_) T(std::move(*((T*)(&val))));
    }

    void copy(const DataType& val)
    {
        destroy();
        new (&data_) T(*((T*) (&val)));
    }

private:
    bool has_init_;
    DataType data_;
};
```

测试：

```
struct MyStruct
{
	MyStruct(int a, int b) :m_a(a), m_b(b) {}
	int m_a;
	int m_b;
};

int main()
{
	Optional<std::string> s1("hello");
	if (s1)
		std::cout << "s1 init:" << *s1 << std::endl;

	Optional<std::string> s2 = s1;
	if (s1)
		std::cout << "s1 init:" << *s1 << std::endl;
	if (s2)
		std::cout << "s2 init:" << *s2 << std::endl;

	Optional<MyStruct> t;
	if (t)
		std::cout << "t init:" << (*t).m_a << "," << (*t).m_b << std::endl;
	t.emplace(2, 5);
	if (t)
		std::cout << "t init:" << (*t).m_a << "," << (*t).m_b << std::endl;

	return 0;
}
```

# 惰性求值

惰性求值一般用于函数式编程语言中，使用延迟求值的时候，表达式不在它被绑定到变量之后就立即求值，而是在需要的时候再求值。

```
#pragma once
#include "Optional.hpp"

template <typename T>
struct Lazy
{
	Lazy() {}

	template<typename Func, typename ...Args>
	Lazy(Func& f, Args&&...args)
	{
		func_ = [&f, &args...]{ return f(args...); };
		//func_ = std::bind(f,std::forward<Args...>(args)...);  //@ ok
	}

	T& value()
	{
		if (!value_.is_init())
		{
			value_ = func_();
		}
		return *value_;
	}

	bool is_value_created()const
	{
		return value_.is_init();
	}

private:
	std::function<T()> func_; //@ 用于存放传入的函数,返回 T 类型，无参数，参数可以通过 lambda 表达式绑定
	Optional<T> value_;    //@ 用于存放求值结果，可以通过 Optional 特性判断是否已经初始化，节省变量
};

//@ 包装函数，用于自动推断返回类型
template<class Func, typename ...Args>
Lazy<typename std::result_of<Func(Args...)>::type> make_lazy(Func&& func, Args&& ...args)
{
	return Lazy<typename std::result_of<Func(Args...)>::type>(std::forward<Func>(func), std::forward<Args>(args)...);
}
```

测试：

```
struct BigObject
{
	BigObject()
	{
		std::cout << "load big object" << std::endl;
	}
};

struct SmallObj
{
	SmallObj()
	{
		lazy_big_obj_ = make_lazy([] {return std::make_shared<BigObject>(); });
	}

	void load()
	{
		lazy_big_obj_.value();
	}

	Lazy<std::shared_ptr<BigObject>> lazy_big_obj_;
};

int test(int x)
{
	return x * 2;
}

int main()
{
	//@ 带参数的普通函数
	int y = 4;
	auto lz1 = make_lazy(test, y);
	std::cout << lz1.value() << std::endl;

	//@ 不带参数的 lambda 
	Lazy<int> lz2 = make_lazy([] {return 12; });
	std::cout << lz2.value() << std::endl;

	//@ 带参数的 fuction
	std::function <int(int)> f = [](int x) {return x + 2; };
	auto lz3 = make_lazy(f, 3);
	std::cout << lz3.value() << std::endl;

	//@ 延迟加载大的对象
	SmallObj st;
	st.load();

	return 0;
}
```

# lambda 链式调用

```
#include <functional>
#include <iostream>
#include <type_traits>

template <typename T>
class Task;

template <typename R, typename ...Args>
class Task<R(Args...)>
{
public:
	Task(std::function<R(Args...)>&& f) : func_(std::move(f)) {}
	Task(std::function<R(Args...)>& f) : func_(f) {}

	R run(Args&&...args)
	{
		return func_(std::forward<Args>(args)...);
	}

	template <typename F>
	auto then(F&& f)->Task<typename std::result_of<F(R)>::type(Args...)>
	{
		using return_type = typename std::result_of<F(R)>::type;
		auto func = std::move(func_);
		return Task<return_type(Args...)>([func, &f](Args...args)
		{
			return f(func(std::forward<Args>(args)...));
		});
	}

private:
	std::function<R(Args...)> func_;
};
```

说明：

```
int main()
{
	Task<int(int)> task([](int i) { return i; });
	auto res = task.then([](int i) { return i + 1; }).then([](int i) {return i + 2; }).then([](int i) {return i + 3; }).run(10);
	std::cout << res << std::endl; //@ 16

	return 0;
}
```

# Any 的实现

Any 类是一个特殊的只能容纳一个元素的容器，它可以擦除类型，可以赋给它任何类型的值，在实际使用时再根据实际的类型进行转换。

```
#pragma once
#include <memory>
#include <typeindex>
#include <exception>
#include <iostream>

struct Any
{
	Any(void) : tp_index_(std::type_index(typeid(void))) {}
	Any(const Any& that) : bptr_(that.clone()), tp_index_(that.tp_index_) {}
	Any(Any && that) : bptr_(std::move(that.bptr_)), tp_index_(that.tp_index_) {}

	//@ 创建智能指针时，对于一般的类型，通过std::decay来移除引用和cv符，从而获取原始类型
	template<typename U, class = typename std::enable_if<!std::is_same<typename std::decay<U>::type, Any>::value, U>::type> 
	Any(U && value) : bptr_(new Derived < typename std::decay<U>::type>(std::forward<U>(value))),
					  tp_index_(std::type_index(typeid(typename std::decay<U>::type))) {}

	bool is_null() const { return !bool(bptr_); }

	template<class U> 
	bool is() const
	{
		return tp_index_ == std::type_index(typeid(U));
	}

	//@ 将Any转换为实际的类型
	template<class U>
	U& any_cast()
	{
		if (!is<U>())
		{
			std::cout << "can not cast " << typeid(U).name() << " to " << tp_index_.name() << std::endl;
			throw std::logic_error{ "bad cast" };
		}

		auto derived = dynamic_cast<Derived<U>*> (bptr_.get());
		return derived->value_;
	}

	Any& operator=(const Any& a)
	{
		if (bptr_ == a.bptr_)
			return *this;

		bptr_ = a.clone();
		tp_index_ = a.tp_index_;
		return *this;
	}

	Any& operator=(Any&& a)
	{
		if (bptr_ == a.bptr_)
			return *this;

		bptr_ = std::move(a.bptr_);
		tp_index_ = a.tp_index_;
		return *this;
	}

private:
	struct Base;
	typedef std::unique_ptr<Base> BasePtr;

	struct Base
	{
		virtual ~Base() {}
		virtual BasePtr clone() const = 0;
	};

	template<typename T>
	struct Derived : Base
	{
		template<typename U>
		Derived(U && value) : value_(std::forward<U>(value)) { }

		BasePtr clone() const
		{
			return BasePtr(new Derived<T>(value_));
		}

		T value_;
	};

	BasePtr clone() const
	{
		if (bptr_ != nullptr)
			return bptr_->clone();
		return nullptr;
	}

	BasePtr bptr_;
	std::type_index tp_index_;
};
```

测试：

```
int main()
{
	std::vector<Any> vec;
	vec.push_back(1);
	vec.push_back(std::string("hello"));
	auto v1 = vec[0].any_cast<int>();
	auto v2 = vec[1].any_cast<std::string>();

	Any any2 = 100;
	auto res = any2.any_cast<int>();

	Any any;
	auto r = any.is_null();  //@ true
	std::string s1 = "hello";
	any = s1;
	any.any_cast<int>(); //@ crash

	return 0;
}
```

# function_traits

function_traits 可以用来获取函数的实际类型、返回类型、参数个数、参数具体的类型。

```
template<typename T>
struct function_traits;

//@ 普通函数.
template<typename Ret, typename... Args>
struct function_traits<Ret(Args...)>
{
public:
	enum { arity = sizeof...(Args) };
	typedef Ret function_type(Args...);
	typedef Ret return_type;
	using stl_function_type = std::function<function_type>;
	typedef Ret(*pointer)(Args...);

	template<size_t I>
	struct args
	{
		static_assert(I < arity, "index is out of range, index must less than sizeof Args");
		using type = typename std::tuple_element<I, std::tuple<Args...>>::type; //@ 获取指定位置的元素类型
	};

	typedef std::tuple<std::remove_cv_t<std::remove_reference_t<Args>>...> tuple_type;
	typedef std::tuple<std::remove_const_t<std::remove_reference_t<Args>>...> bare_tuple_type;
};

//@函数指针.
template<typename Ret, typename... Args>
struct function_traits<Ret(*)(Args...)> : function_traits<Ret(Args...)> {};

//@ std::function.
template <typename Ret, typename... Args>
struct function_traits<std::function<Ret(Args...)>> : function_traits<Ret(Args...)> {};

//@ member function.
#define FUNCTION_TRAITS(...)\
template <typename ReturnType, typename ClassType, typename... Args>\
struct function_traits<ReturnType(ClassType::*)(Args...) __VA_ARGS__> : function_traits<ReturnType(Args...)>{};

FUNCTION_TRAITS()
FUNCTION_TRAITS(const)
FUNCTION_TRAITS(volatile)
FUNCTION_TRAITS(const volatile)

//@ 函数对象
template<typename Callable>
struct function_traits : function_traits<decltype(&Callable::operator())> {};

template <typename Function>
typename function_traits<Function>::stl_function_type to_function(const Function& lambda)
{
	return static_cast<typename function_traits<Function>::stl_function_type>(lambda);
}

template <typename Function>
typename function_traits<Function>::stl_function_type to_function(Function&& lambda)
{
	return static_cast<typename function_traits<Function>::stl_function_type>(std::forward<Function>(lambda));
}

template <typename Function>
typename function_traits<Function>::pointer to_function_pointer(const Function& lambda)
{
	return static_cast<typename function_traits<Function>::pointer>(lambda);
}
```

测试：

```
template<typename T>
void print_type()
{
	std::cout << typeid(T).name() << std::endl;
}

float(*func_ptr)(std::string, int);
static float static_function(const std::string& a, int b)
{
	return (float)a.size() / b;
}

struct Test
{
	int f(int a, int b)volatile { return a + b; }
	int operator()(int)const { return 0; }
};

int main()
{
	std::function<int(int)> f = [](int a) {return a; };
	print_type<function_traits<std::function<int(int)>>::function_type>();
	print_type<function_traits<std::function<int(int)>>::args<0>::type>();

	print_type<function_traits<decltype(f)>::function_type>();
	print_type<function_traits<decltype(func_ptr)>::function_type>();
	print_type<function_traits<decltype(static_function)>::function_type>();

	print_type<function_traits<Test>::function_type>();
	using T = decltype(&Test::f);
	print_type<T>();

	print_type<function_traits<decltype(&Test::f)>::function_type>();

	return 0;
}
```

# Variant 的实现

Variant 类似于 union，它能代表定义的多种类型，允许赋不同类型的值给它。它的具体类型是在初始化赋值时确定的。

```
#pragma once

#include <typeindex>
#include <iostream>

/** 获取最大的整数 */
template <size_t arg, size_t... rest>
struct IntegerMax;

template <size_t arg>
struct IntegerMax<arg> : std::integral_constant<size_t, arg>
{
};

template <size_t arg1, size_t arg2, size_t... rest>
struct IntegerMax<arg1, arg2, rest...> : std::integral_constant<size_t, arg1 >= arg2 ? IntegerMax<arg1, rest...>::value
	: IntegerMax<arg2, rest...>::value>
{
};

/** 获取最大的align */
template<typename... Args>
struct MaxAlign : std::integral_constant<int, IntegerMax<std::alignment_of<Args>::value...>::value>
{
};

/** 是否包含某个类型 */
template <typename T, typename... List>
struct Contains;

template <typename T, typename Head, typename... Rest>
struct Contains<T, Head, Rest...>
	: std::conditional<std::is_same<T, Head>::value, std::true_type, Contains<T, Rest... >> ::type
{
};

template <typename T>
struct Contains<T> : std::false_type
{
};

template <typename T, typename... List>
struct IndexOf;

template <typename T, typename Head, typename... Rest>
struct IndexOf<T, Head, Rest...>
{
	enum { value = IndexOf<T, Rest...>::value + 1 };
};

template <typename T, typename... Rest>
struct IndexOf<T, T, Rest...>
{
	enum { value = 0 };
};

template <typename T>
struct IndexOf<T>
{
	enum { value = -1 };
};

template<int index, typename... Types>
struct At;

template<int index, typename First, typename... Types>
struct At<index, First, Types...>
{
	using type = typename At<index - 1, Types...>::type;
};

template<typename T, typename... Types>
struct At<0, T, Types...>
{
	using type = T;
};

template<typename... Types>
class Variant
{
	enum
	{
		data_size = IntegerMax<sizeof(Types)...>::value,
		align_size = MaxAlign<Types...>::value
	};
	
	using DataType = typename std::aligned_storage<data_size, align_size>::type;
	
public:
	template<int index>
	using IndexType = typename At<index, Types...>::type;

	Variant(void) :type_index_(typeid(void))
	{
	}

	~Variant()
	{
		destroy(type_index_, &data_);
	}

	Variant(Variant<Types...>&& old) : type_index_(old.type_index_)
	{
		move(old.type_index_, &old.data_, &data_);
	}

	Variant(const Variant<Types...>& old) : type_index_(old.type_index_)
	{
		copy(old.type_index_, &old.data_, &data_);
	}

	Variant& operator=(const Variant& old)
	{
		copy(old.type_index_, &old.data_, &data_);
		type_index_ = old.type_index_;
		return *this;
	}

	Variant& operator=(Variant&& old)
	{
		move(old.type_index_, &old.data_, &data_);
		type_index_ = old.type_index_;
		return *this;
	}

	template <class T,
		class = typename std::enable_if<Contains<typename std::decay<T>::type, Types...>::value>::type>
		Variant(T&& value) : type_index_(typeid(void))
	{
		destroy(type_index_, &data_);
		typedef typename std::decay<T>::type U;
		new(&data_) U(std::forward<T>(value));
		type_index_ = std::type_index(typeid(U));
	}

	template<typename T>
	bool is() const
	{
		return (type_index_ == std::type_index(typeid(T)));
	}

	bool empty() const
	{
		return type_index_ == std::type_index(typeid(void));
	}

	std::type_index type() const
	{
		return type_index_;
	}

	template<typename T>
	typename std::decay<T>::type& get()
	{
		using U = typename std::decay<T>::type;
		if (!is<U>())
		{
			std::cout << typeid(U).name() << " is not defined. "
				<< "current type is " << type_index_.name() << std::endl;
			throw std::bad_cast{};
		}

		return *(U*)(&data_);
	}

	template <typename T>
	int index_of()
	{
		return IndexOf<T, Types...>::value;
	}

	bool operator==(const Variant& rhs) const
	{
		return type_index_ == rhs.type_index_;
	}

	bool operator<(const Variant& rhs) const
	{
		return type_index_ < rhs.type_index_;
	}

private:
	void destroy(const std::type_index& index, void *buf)
	{
		[](Types&&...) {}((destroy0<Types>(index, buf), 0)...);
	}

	template<typename T>
	void destroy0(const std::type_index& id, void *data)
	{
		if (id == std::type_index(typeid(T)))
			reinterpret_cast<T*>(data)->~T();
	}

	void move(const std::type_index& old_t, void *old_v, void *new_v)
	{
		[](Types&&...) {}((move0<Types>(old_t, old_v, new_v), 0)...);
	}

	template<typename T>
	void move0(const std::type_index& old_t, void *old_v, void *new_v)
	{
		if (old_t == std::type_index(typeid(T)))
			new (new_v)T(std::move(*reinterpret_cast<T*>(old_v)));
	}

	void copy(const std::type_index& old_t, const void *old_v, void *new_v)
	{
		[](Types&&...) {}((copy0<Types>(old_t, old_v, new_v), 0)...);
	}

	template<typename T>
	void copy0(const std::type_index& old_t, const void *old_v, void *new_v)
	{
		if (old_t == std::type_index(typeid(T)))
			new (new_v)T(*reinterpret_cast<const T*>(old_v));
	}

private:
	DataType data_;
	std::type_index type_index_;
};
```

测试：

```
int main()
{
	typedef Variant<int, char, double> Var;
	int x = 10;

	Var v = x;
	v = 1;
	std::cout << v.get<int>() << std::endl; //@ 1

	v = 1.123;
	std::cout << v.get<double>() << std::endl; //@ 1.123

	v = 'c';
	std::cout << v.get<char>() << std::endl; //@ c

	std::cout << v.get<double>() << std::endl; //@ double is not defined. current type is char

	return 0;
}
```

# ScopeGuard

ScopeGuard 的作用是确保资源面对非正常返回(函数在中途返回，中途抛出异常)导致后面释放资源的代码没有被执行时能够自动释放资源，没有发生异常则正常结束。

ScopeGuard 利用了局部变量析构函数来管理资源，利用了 RAII 机制。

```
#pragma once

template <typename F>
class ScopeGuard
{
public:
	explicit ScopeGuard(F && f) : func_(std::move(f)), dismiss_(false) {}
	explicit ScopeGuard(const F& f) : func_(f), dismiss_(false) {}

	~ScopeGuard()
	{
		if (!dismiss_ && func_ != nullptr)
			func_();
	}

	ScopeGuard(ScopeGuard && rhs) : func_(std::move(rhs.func_)), dismiss_(rhs.dismiss_)
	{
		rhs.Dismiss();
	}

	void dismiss()
	{
		dismiss_ = true;
	}

private:
	F func_;
	bool dismiss_;

	ScopeGuard();
	ScopeGuard(const ScopeGuard&);
	ScopeGuard& operator= (const ScopeGuard&);
};

template <typename F>
ScopeGuard<typename std::decay<F>::type> make_guard(F && f)
{
	return ScopeGuard<typename std::decay<F>::type>(std::forward<F>(f));
}
```

测试：

```
int main()
{
	std::function < void()> f = []()
	{ std::cout << "cleanup from abnormal exit" << std::endl; };

	{
		auto gd = make_guard(f);
		//...
		gd.dismiss();  //表明前面我是正常的清理了资源，属于正常退出的
	}


	try
	{
		auto gd = make_guard(f);
		throw 1;
	}
	catch (...)
	{
		std::cout << "exception" << std::endl;
	}


	{
		auto gd = make_guard(f);
		return -1;  //非正常退出表示资源还没清理呢，，等着ScopeGuard自动清理
				 //...
	}

	return 0;
}
```

# tuple_helper

std::tuple 具有很多编译期计算的特性，正是这些独特的特性使得它既能用于编译期计算，又可以用于运行期计算。

## 打印 tuple

模板类特化和递归调用结合展开 tuple：

```
template <class Tuple, size_t N>
struct TuplePrinter
{
	static void print(const Tuple& t)
	{
		TuplePrinter<Tuple, N - 1>::print(t);
		std::cout << "," << std::get<N - 1>(t);
	}
};

template <class Tuple>
struct TuplePrinter<Tuple, 1>
{
	static void print(const Tuple& t)
	{
		std::cout << std::get<0>(t);
	}
};

template<class...Args>
void print_tuple(const std::tuple<Args...>& t)
{
	std::cout << "(";
	TuplePrinter<decltype(t), sizeof...(Args)>::print(t);
	std::cout << ")\n";
}
```

测试：

```
int main()
{
	std::tuple<int, short, double, char, std::string> tp = std::make_tuple(1, 2, 3, 'a', std::string("hello"));
	print_tuple(tp);

	return 0;
}
```

通过索引序列来展开：

```
template <int...>
struct IndexTuple {};

template <int N, int...Indexs>
struct MakeIndexes : MakeIndexes<N - 1, N - 1, Indexs...> {};

template <int...Indexs>
struct MakeIndexes<0, Indexs...>
{
	typedef IndexTuple<Indexs...> type;
};

template <typename T>
void print(T t)
{
	std::cout << t << std::endl;
}

template <typename T, typename ...Args>
void print(T t, Args...args)
{
	std::cout << t << std::endl;
	print(args...);
}

template <typename Tuple, int...Indexs>
void print_tuple(IndexTuple<Indexs...>& in, Tuple& tp)
{
	print(std::get<Indexs>(tp)...);
}
```

测试：

```
int main()
{
	using Tuple = std::tuple<int, double, char>;
	Tuple tp = std::make_tuple(1, 2.9, 'a');
	print_tuple(MakeIndexes<std::tuple_size<Tuple>::value>::type(), tp);

	return 0;
}
```

## 根据元素值获取索引位置

```
namespace detail
{
	//@ 对于可转换的类型值则直接比较
	template <typename T, typename U>
	typename std::enable_if<std::is_convertible<T, U>::value || std::is_convertible<U, T>::value, bool>::type
	compare(T t, U u)
	{
		return t == u;
	}

	//@ 不能互相转换的则直接返回false
	bool compare(...)
	{
		return false;
	}


	//@ 根据值查找索引
	template<int I, typename T, typename... Args>
	struct find_index
	{
		static int call(const std::tuple<Args...>& t, T&& val)
		{
			return (compare(std::get<I - 1>(t), val) ? I - 1 : find_index<I - 1, T, Args...>::call(t, std::forward<T>(val)));
		}
	};

	template<typename T, typename... Args>
	struct find_index<0, T, Args...>
	{
		static int call(std::tuple<Args...> const& t, T&& val)
		{
			return compare(std::get<0>(t), val) ? 0 : -1;
		}
	};
}

template<typename T, typename... Args>
int find_index(std::tuple<Args...> const& t, T&& val)
{
	return detail::find_index<sizeof...(Args), T, Args...>::call(t, std::forward<T>(val));
}
```

测试：

```
int main()
{
	std::tuple<int, double, std::string> tp = std::make_tuple(1, 2, std::string("OK"));
	int index = find_index(tp, std::string("OK"));

	std::cout << index << std::endl;

	return 0;
}
```

## 在运行期根据索引获取元素

通常情况下不允许在运行期间根据索引获取值：

```
int i = 0;
std::get<i>(tuple); //@ 不合法
```

要通过运行时的变量来获取 tuple 中元素值，需要将运行期的变量映射为编译期常量。

```
template <size_t k, typename Tuple>
typename std::enable_if<(k == std::tuple_size<Tuple>::value)>::type
get_parameter_by_index(size_t index, Tuple& tp)
{
	throw std::invalid_argument("arg index out of range");
}

template <size_t k = 0, typename Tuple>
typename std::enable_if<(k < std::tuple_size<Tuple>::value)>::type
	get_parameter_by_index(size_t index, Tuple& tp)
{
	if (k == index)
	{
		std::cout << std::get<k>(tp) << std::endl;
	}
	else
	{
		get_parameter_by_index<k + 1>(index, tp);
	}
}
```

测试：

```
int main()
{
	using Tuple = std::tuple<int, double, std::string, int>;
	Tuple tp = std::make_tuple(1, 2, "test", 3);
	const size_t length = std::tuple_size<Tuple>::value;

	//@ 打印每个元素
	for (size_t i = 0; i < length; ++i)
	{
		get_arguement_by_index<0>(i, tp);
	}

	get_arguement_by_index(4, tp);  //@ 索引超出范围将抛出异常

	return 0;
}
```

这里通过递归方式自增编译期常量 K，将 K 与运行期变量 index 做比较，两者相等时，调用编译期常量 K 来获取 tuple 中的第 K 个元素。

还可以通过参数包逐步展开的方式：

```
template <typename Arg>
void GetArgByIndex(int index, std::tuple<Arg>& tp)
{
	std::cout << std::get<0>(tp) << std::endl;
}

template <typename Arg, typename... Args>
void GetArgByIndex(int index, std::tuple<Arg, Args...>& tp)
{
	if (index < 0 || index >= std::tuple_size<std::tuple<Arg, Args...>>::value)
	{
		throw std::invalid_argument("index is not valid");
	}

	if (index > 0)
	{
		GetArgByIndex(index - 1, (std::tuple<Args...>&) tp);
	}
	else
	{
		std::cout << std::get<0>(tp) << std::endl;
	}
}
```

## 遍历 tuple

```
template <int...>
struct IndexTuple {};

template <int N, int...Indexs>
struct make_indexes : make_indexes<N - 1, N - 1, Indexs...> {};

template <int...Indexs>
struct make_indexes<0, Indexs...>
{
	typedef IndexTuple<Indexs...> type;
};

namespace detail {

	template<typename Func, typename Last>
	void for_each_impl(Func&& f, Last&& last)
	{
		f(last);
	}

	template<typename Func, typename First, typename ... Rest>
	void for_each_impl(Func&& f, First&& first, Rest&&...rest)
	{
		f(first);
		for_each_impl(std::forward<Func>(f), rest...);
	}

	template<typename Func, int ... Indexes, typename ... Args>
	void for_each_helper(Func&& f, IndexTuple<Indexes...>, std::tuple<Args...>&& tup)
	{
		for_each_impl(std::forward<Func>(f), std::forward<Args>(std::get<Indexes>(tup))...);
	}

} // namespace detail

template<typename Func, typename Tuple>
void tp_for_each(Func&& f, Tuple& tup)
{
	using namespace detail;
	for_each_helper(std::forward<Func>(f), typename make_indexes<
		std::tuple_size<Tuple>::value>::type(), tup);
}

template<typename Func, typename Tuple>
void tp_for_each(Func&& f, Tuple&& tup)
{
	using namespace detail;
	for_each_helper(std::forward<Func>(f), typename make_indexes<
		std::tuple_size<Tuple>::value>::type(),
		std::forward<Tuple>(tup));
}

struct Functor
{
	template <typename T>
	void operator()(T& t) const
	{
		//t.doSomething();
		std::cout << t << std::endl;
	}
};
```

测试：

```
int main()
{
	tp_for_each(Functor(), std::make_tuple<int, double>(1, 2.5));

	return 0;
}
```

## 反转 tuple

```
template <int...>
struct IndexTuple {};

template <int N, int...Indexs>
struct MakeIndexes : MakeIndexes<N - 1, N - 1, Indexs...> {};

template <int...Indexs>
struct MakeIndexes<0, Indexs...>
{
	typedef IndexTuple<Indexs...> type;
};

template<int I, typename IndexTuple, typename... Types>
struct make_indexes_reverse_impl;

//declare
template<int I, int... Indexes, typename T, typename... Types>
struct make_indexes_reverse_impl<I, IndexTuple<Indexes...>, T, Types...>
{
	using type = typename make_indexes_reverse_impl<I - 1, IndexTuple<Indexes..., I - 1>, Types...>::type;
};

//terminate
template<int I, int... Indexes>
struct make_indexes_reverse_impl<I, IndexTuple<Indexes...>>
{
	using type = IndexTuple<Indexes...>;
};

//type trait
template<typename ... Types>
struct make_reverse_indexes : make_indexes_reverse_impl<sizeof...(Types), IndexTuple<>, Types...>
{};

//反转
template <class...Args, int...Indexes>
auto reverse_impl(std::tuple<Args...>&& tup, IndexTuple<Indexes...>&&) ->
decltype(std::make_tuple(std::get<Indexes>(std::forward<std::tuple<Args...>>(tup))...))
{
	return std::make_tuple(std::get<Indexes>(std::forward<std::tuple<Args...>>(tup))...);
}

template <class...Args>
auto Reverse(std::tuple<Args...>&& tup) ->
decltype(reverse_impl(std::forward<std::tuple<Args...>>(tup),
	typename make_reverse_indexes<Args...>::type()))
{
	return reverse_impl(std::forward<std::tuple<Args...>>(tup),
		typename make_reverse_indexes<Args...>::type());
}

template <class Tuple, std::size_t N>
struct TuplePrinter
{
	static void print(const Tuple& t)
	{
		TuplePrinter<Tuple, N - 1>::print(t);
		std::cout << ", " << std::get<N - 1>(t);
	}
};

template <class Tuple>
struct TuplePrinter<Tuple, 1>
{
	static void print(const Tuple& t)
	{
		std::cout << std::get<0>(t);
	}
};

template <class...Args>
void PrintTuple(const std::tuple<Args...>& t)
{
	std::cout << "(";
	TuplePrinter<decltype(t), sizeof...(Args)>::print(t);
	std::cout << ")\n";
}
```

测试：

```
int main()
{
	auto tp1 = std::make_tuple<int, short, double, char>(1, 2, 2.5, 'a');
	auto tp2 = Reverse(std::make_tuple<int, short, double, char>(1, 2, 2.5, 'a'));

	PrintTuple(tp1);
	PrintTuple(tp2);

	return 0;
}
```

## 应用于函数

将 tuple 应用于函数是将 tuple 展开作为函数的参数。

```
template <int...>
struct IndexTuple {};

template <int N, int...Indexs>
struct MakeIndexes : MakeIndexes<N - 1, N - 1, Indexs...> {};

template <int...Indexs>
struct MakeIndexes<0, Indexs...>
{
	typedef IndexTuple<Indexs...> type;
};

template <typename F, typename Tuple, int...Indexes>
auto apply(F&& f, IndexTuple<Indexes...>&& in, Tuple&& tp) ->
decltype(std::forward<F>(f)(std::get<Indexes>(tp)...))
{
	std::forward<F>(f)(std::get<Indexes>(tp)...);
}
```

测试：

```
void TestF(int a, double b)
{
	std::cout << a + b << std::endl;
}

int main(void)
{
	apply(TestF, MakeIndexes<2>::type(), std::make_tuple(1, 2));  //输出 : 3
	return 0;
}
```

## tuple_zip

合并 tuple 前一个 tuple 中的每个元素都为 key，后一个 tuple 中的每一个元素都是 value，组成一个 pair：

```
//将两个tuple合起来，前一个tuple中的每个元素为key，后一个tuple中的每个元素为value，组成一个pair集合
namespace details
{
	template <int...>
	struct IndexTuple {};

	template <int N, int...Indexs>
	struct MakeIndexes : MakeIndexes<N - 1, N - 1, Indexs...> {};

	template <int...Indexs>
	struct MakeIndexes<0, Indexs...>
	{
		typedef IndexTuple<Indexs...> type;
	};

	template<std::size_t N, typename T1, typename T2>
	using pair_type = std::pair<typename std::tuple_element<N, T1>::type, typename std::tuple_element<N, T2>::type>;

	template<std::size_t N, typename T1, typename T2>
	pair_type<N, T1, T2> pair(const T1& tup1, const T2& tup2)
	{
		//
		return std::make_pair(std::get<N>(tup1), std::get<N>(tup2));
	}

	template<int... Indexes, typename T1, typename T2>
	auto pairs_helper(IndexTuple<Indexes...>, const T1& tup1, const T2& tup2) -> decltype(std::make_tuple(pair<Indexes>(tup1, tup2)...))
	{
		return std::make_tuple(pair<Indexes>(tup1, tup2)...);
	}

} // namespace details

template<typename Tuple1, typename Tuple2>
auto Zip(Tuple1 tup1, Tuple2 tup2) -> decltype(details::pairs_helper(
	typename details::MakeIndexes<std::tuple_size<Tuple1>::value>::type(), tup1, tup2))
{
	static_assert(std::tuple_size<Tuple1>::value == std::tuple_size<Tuple2>::value,
		"tuples should be the same size.");
	return details::pairs_helper(typename
		details::MakeIndexes<std::tuple_size<Tuple1>::value>::type(), tup1, tup2);
}
```

测试：

```
int main()
{
	auto tp1 = std::make_tuple<int, short, double, char>(1, 2, 2.5, 'a');
	auto tp2 = std::make_tuple<double, short, double, char>(1.5, 2, 2.5, 'z');
	auto mypairs = Zip(tp1, tp2);

	return 0;
}
```

