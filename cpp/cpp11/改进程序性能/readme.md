# 右值和右值引用

## 左值和右值

- 左值是表达式结束后依然存在的持久对象，右值是表达式结束就不会继续存在的临时对象
- 如果能对表达式取地址就是左值，否则就是右值
- 具名变量或对象都是左值，而右值是不具名的

C++ 11 中右值由两个概念构成：

- 一个是将亡值
  - 将要被移动的对象
  - T&& 类型函数返回值
  - std::move 返回值
  - 转换为 T&& 的类型的转换函数的返回值

- 一个是纯右值

  - 非引用返回的临时变量
  - 运算表达式产生的临时变量
  - 原始字面量
  - lambda 表达式

C++ 11 中所有的值必属于：左值，将亡值，纯右值 三者之一，将亡值和纯右值都属于右值。

## 右值引用

右值引用是对一个右值进行引用的类型，因为右值不具名，只能通过引用的方式找到它。

需要注意：

- 无论左值引用还是右值引用都必须立即进行初始化
- 常量左值引用是一个 "万能" 引用，可以接受左值，右值，常量左值，常量右值。普通的左值引用不能接受右值

```
void f1(std::string& str)
{}

void f2(const std::string& str)
{}

f1("hello,world");  //@ 错误，普通左值引用不能绑定右值
f2("hello,world");  //@ 正确，const 左值引用是万能引用类型
```

## && 的类型

- 左值和右值是独立于它们的类型的，右值引用类型可能是左值，也可能是右值
- auto&& 和函数参数类型自动推导的 T&& 是一个未定的引用类型，称为 universal references，可能是左值引用也可能是右值引用类型，取决于初始化的值类型

```
template<typename T>
void f1(T&& t);  //@ T的类型需要被推导，t 是一个 universal reference

template<typename T>
class Test
{
	Test(Test&& rhs);  //@ 已经定义了一个特定的类型，没有类型推断，rhs 是一个右值引用
};

void f2(Test&& t); //@ 已经定义了一个确定的类型，没有类型推断，&& 是一个右值引用

template<typename>
void f3(std::vector<T>&& v); //@ 调用这个函数之前，std::vector<T> 类型已经被推断，函数调用时没有类型推断，因此 v 是右值引用
```

## 引用折叠

- 所有的右值引用折叠到右值引用仍然是一个右值引用
- 所有的其它类型之间的叠加都将变成左值引用

```
int w1, w2;
auto&& v1 = w1; //@ v1 是一个 universal reference，因为被左值初始化，所以最终是一个左值
decltype(w1) && v2 = w2; //@ 错误， v2 是一个右值引用类型，但是被一个左值初始化，这是不合法的
decltype(w1) && v2 = std::move(w2); //@ 正确，std::move 可以将左值转换成右值
```

# 移动语义

移动提高性能：

默认的拷贝构造函数是执行浅拷贝，对于含有指针的类，这是极为不安全的，因此通常对于含有指针的类自定义拷贝构造函数执行深拷贝，但是深拷贝的性能是比较低的，因此一些情况下可以使用移动提高性能：

```
class A
{
public:
	A() :m_ptr(new int(0))
	{
		std::cout << "constructor" << std::endl;
	}

	//@ 深拷贝
	A(const A& a) :m_ptr(new int(*a.m_ptr))
	{
		std::cout << "copy constructor" << std::endl;
	}

	//@ 移动构造
	A(A&& a) :m_ptr(a.m_ptr)
	{
		a.m_ptr = nullptr;
		std::cout << "move constructor" << std::endl;
	}

	~A()
	{
		std::cout << "destructor" << std::endl;
		delete m_ptr;
	}

private:
	int* m_ptr;
};
```

## std::move

std::move 只是转移了资源的控制权，本质上是将左值强制转换为右值，以使用移动语义，避免含有资源的对象发生了无谓的拷贝。

- move 对于拥有内存，文件句柄等资源对象的成员有效
- 如果是一些基本类型，比如 int 和 char[10] 数组等，如果使用 move 还是会发生拷贝，因为它们没有对应的移动构造函数

# std::forward

完美转发实现了参数在传递过程中保持其值属性的功能，即若是左值，则传递之后仍然是左值，若是右值，则传递之后仍然是右值。

完美转发利用 std::forward 实现：

- std::forward 不仅可以保持左值或者右值不变，同时还可以保持 CV 和引用等属性不变
- std::forward 只有在它的参数绑定到一个右值上的时候，它才转换参数到右值

```
template <typename T>
void printT(T& t)
{
	std::cout << "lvalue" << std::endl;
}

template <typename T>
void printT(T&& t)
{
	std::cout << "rvalue" << std::endl;
}

template<typename T>
void test_forward(T&& t)
{
	printT(std::forward<T>(t));
}

int main()
{
	int x = 1;

	test_forward(1); //@ rvalue
	test_forward(x); //@ lvalue
	test_forward(std::forward<int>(x)); //@ rvalue
	test_forward(std::forward<int&>(x)); //@ lvalue
	test_forward(std::forward<int&&>(x)); //@ rvalue
	test_forward(std::move(x)); //@ rvalue

	return 0;
}
```

右值引用、完美转发再结合可变模板参数可以写一个万能的函数包装器：

```
template<class Function, class...Args>
inline auto function_wrapper(Function&& func, Args&&...args)->decltype(func(std::forward<Args>(args)...))
{
	return func(std::forward<Args>(args)...);
}
```

引用：

```
template<class Function, class...Args>
inline auto function_wrapper(Function&& func, Args&&...args)->decltype(func(std::forward<Args>(args)...))
{
	return func(std::forward<Args>(args)...);
}

void test0()
{
	std::cout << "void" << std::endl;
}

int test1()
{
	return 1;
}

int test2(int x)
{
	return x;
}

std::string test3(const std::string s1, const std::string s2)
{
	return s1 + s2;
}


int main()
{
	function_wrapper(test0);  //@ 没有返回值
	auto res1 = function_wrapper(test1);  //@ 返回1
	auto res2 = function_wrapper(test2, 2);		//@ 返回2
	auto res3 = function_wrapper(test3, "aa", "bb");   //@ 返回aabb


	return 0;
}
```

# emplace_back 

emplace_back 能就地通过函数构造对象，不需要拷贝或者移动内存，相比于 push_back  能更好地避免内存的拷贝与移动，使容器插入元素的性能得到进一步提升。在大多数情况下应该优先使用 emplace_back 来代替 push_back。标准库中类似的方法有：emplace，emplace_hint，emplace_front，emplace_after，emplace_back。

emplace_back  通过构造函数的参数就可以构造对象，因此，要求对象有相应的构造函数，如果没有会编译报错。

emplace_back 和 push_back  对比测试：

```
struct Complicated
{
	int year_;
	double country_;
	std::string name_;

	Complicated(int a, double b, std::string s) :year_(a), country_(b), name_(s)
	{
		std::cout << "is constructed" << std::endl;
	}

	Complicated(const Complicated& other) :year_(other.year_), country_(other.country_), name_(other.name_)
	{
		std::cout << "is copied" << std::endl;
	}

	Complicated(Complicated&& other) :year_(other.year_), country_(other.country_), name_(other.name_)
	{
		std::cout << "is moved" << std::endl;
	}
};

int main()
{
	std::map<int, Complicated> m;
	int i = 4;
	double d = 5.0;
	std::string s = "C++";
	std::cout << "---insert---" << std::endl;
	m.insert(std::make_pair(1, Complicated(i, d, s)));

	std::cout << "---emplace---" << std::endl;
	m.emplace(2, Complicated(i, d, s));

	std::vector<Complicated> v;
	std::cout << "---emplace_back---" << std::endl;
	v.emplace_back(i, d, s);
	std::cout << "---push_back---" << std::endl;
	v.push_back(Complicated(i, d, s));

	return 0;
}
```

# unordered_container

有序容器内部是红黑树，插入元素时会自动排序。无序容器内部是散列表，通过哈希实现，不需要排序，因而效率更高。

因为无序容器内部是散列表，因此无序容器的 key 需要提供 hash_value 函数。对于自定义 key，需要提供 hash 函数和相等比较函数。

```
struct Key
{
	std::string first;
	std::string second;
};

struct KeyHash
{
	std::size_t operator()(const Key& k) const
	{
		return std::hash<std::string>()(k.first) ^ (std::hash<std::string>()(k.second) << 1);
	}
};

struct KeyEqual
{
	bool operator()(const Key& lhs, const Key& rhs) const
	{
		return lhs.first == rhs.first && lhs.second == lhs.second;
	}
};

int main()
{
	//@ 自定义类型需要提供哈希函数和比较函数
	std::unordered_map<Key, std::string, KeyHash, KeyEqual> m = {
		{{"John","Doe"},"example"},
		{{"Mary","Sue"},"another"}
	};
	return 0;
}
```

