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







