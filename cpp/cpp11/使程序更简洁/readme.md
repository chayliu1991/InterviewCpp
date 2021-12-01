# 类型推导

## auto

auto 声明的变量必须马上初始化，以让编译器推断出它的实际类型，并在编译时将 auto 占位符替换为真正的类型。

auto 使用注意事项：

- 不显示指定为引用类型时，推导结果将丢弃引用
- 不显示指定为指针或者引用类型时，推导结果将丢弃 CV 属性

```
//@ 不显示指定，将丢弃引用
int a = 6;
int& r = a;
auto a1 = r;  //@ a1 -> int
auto& a2 = r; //@ a2 -> int&

//@ 不指定引用或者指针类型，将丢弃 cv 属性
const int i = 5;
const int& r = i;
auto a1 = r;  	//@ a1 -> int
auto& a2 = r;	//@ a2 -> const int&
auto b1 = &i; 	//@ b1 -> int* 
auto* b2 = &i;	//@ b2 -> const int*
```

auto 使用限制：

- 不能用于推导函数参数
- 不能用于推导类的非静态成员变量
- 不能用于推导数组
- 不能用于模板参数推导

```
void func(auto a = 1)  {} //@ 错误，auto 不能用于函数参数

class AutoTest
{
    auto i = 0; //@ 错误，auto不能用于非静态成员变量
    auto const static si = 0;  //@ si -> static const int，类成员变量中静态类型无法就地初始化，所以必须要用 const 修饰
};

int arr[10] = { 0 };
auto a1 = arr;  //@ a1 -> int*
auto a2[10] = arr; //@ 错误，auto 不能用于推导数组


template <typename T>
struct Test{};

Test<int> t;
Test<auto> a = t; //@ 错误，auto 不能用于模板参数推导
```

auto 使用场景：

- 遍历 STL 容器
- 类型未定变量声明

## decltype

decltype 用在编译时推导出表达式的类型，并且不会真正计算表达式的值。

```
decltype(exp)
```

推导规则：

- 如果 exp 是标识符，类访问表达式，decltype(exp) 与 exp 的类型一致，并且保留引用和 CV 属性，这一点与 auto 不同
- 如果 exp 是函数调用，decltype(exp) 和返回值得类型一致
- 其他情况，如果 exp 是一个左值， decltype(exp) 是 exp 的左值引用，否则与 exp 类型一致

```
//@ 保留 cv 特性
int i = 0;
const volatile int& cvr = i;
decltype(cvr) a = i;  //@ a -> const volatile int &

//@ 推导函数返回值
const int func_int_r(void);		//@ 返回纯右值
const int& func_int_l(void);	//@ 返回左值
const int&& func_int_x(void);	//@ 返回 x 值(右值引用本身是一个 xvalue)

int main()
{	
	int x = 0;
	decltype(func_int_r()) a;		//@ a -> int,去掉了 CV 特性
	decltype(func_int_l()) b = x;	//@ b -> const int&
	decltype(func_int_x()) c = 0;	//@ c -> const int&&
    return 0;
}

//@ 其它情况
const int x = 1;
int y = 2;
const int* p = &x;

decltype(x) a = 0;			//@ a->const int
decltype((x)) b = 0;		//@ b->const int&
decltype(x + y) c = 1;		//@ c->int
decltype(y += x) d = y;		//@ d->int&
decltype(y = 3) e = y;		//@ e->int&
decltype(*p) f = 0;			//@ f->const int&
```

这里需要注意的是：

- `(表达式)` 的类型为左值引用
- `m+n` 的类型是右值
- `m+=n` 的类型是左值引用

decltype 应用

- 模板中的类型推导

```
//@ 模板中的类型推导
template<typename T, typename U>
void func(T t, U u)
{
	decltype(t*u) tu = t * u;	//@ t*u 是 int 类型，类型提升了
	std::cout << tu << std::endl;
}

int main()
{
	short a = 1;
	char c = 3;
	func(a, c);

	return 0;
}

```

## auto 与 decltype

区别：

- decltype 能够保持变量的引用和 CV 特性
- 如果 decltype 使用的是一个不加括号的变量，那么得到的结果就是这个变量的类型。但是如果给这个变量加上一个或多层括号，那么编译器会把这个变量当作一个表达式看待，变量是一个可以作为左值的特殊表达式，所以这样的decltype就会返回引用类型

auto 与 decltype 联合使用：

返回值类型后置：返回值类型后置一般用于返回值依赖于参数类型而导致难以确定返回值的具体类型：

```
template<typename T, typename U>
auto func(T t, U u)->decltype(t + u)
{
	return t + u;
}
```

# 模板的改进

C++ 11 中要求编译器对模板的右尖括号做单独处理，使编译器能够正确判断出 ">>" 是一个右移操作符还是一个模板参数表的结束符。

## 模板别名

typedef 无法重重定义一个模板：

```
template <typename T>
typedef std::map<std::string, T> map_t;  //@ 错误
```

C++ 11 中使用 using 可以定义模板别名：

```
template <typename T>
using map_t = std::map<std::string, T> ;
```

## 函数模板的默认参数

C++ 11 之前允许类模板有默认参数，但是不允许函数模板有默认参数：

```
template <typename T,typename U = int,U N = 0>
struct Foo {};


template <typename T = int>  //@ C++ 11 之前不允许
void func()
{}
```

但是 C++ 11 中解除了上述对于函数模板默认参数的限制，在 C++ 11 中可以直接调用：

```
func();   //@ 如同一个普通函数
```

函数模板的默认参数不需要在数列表中从右向左书写，而是可以出现在任意位置：

```
template <typename X,typename T = int, typename U>
T func(X val1,U val2)
{
	return static_cast<int>(val1 + val2);
}
```

# 列表初始化

## 统一初始化方法

列表初始化统一了初始化方法，它可以用于任何类型对象的初始化：

```
int arr[]{ 1,2,3 };  //@ 初始化数组

//@ 初始化 STL
std::map<std::string, int> mm = { {"1",1},{ "2",2} ,{ "3",3 } ,{ "4",4 } };
std::set<int> ss{1,2,3,4,5,6};
std::vector<int> vec{1,2,3,4,5};

struct A
{
    int x;
    struct B
    {
        int y;
        int z;
    }b;
};
A a{ 1,{ 2,3 } }; //@ 初始化 POD 类型

int* array = new int[3]{ 1,2,3 }; //@ 初始化动态分配的内存

class Foo
{
public:
	Foo(int,double) {}
};

Foo func()
{
	return{ 1,12.34 }; //@ 初始化返回类型
}
```

## 初始化列表的赋值方式

- 对于聚合类型的初始化将以拷贝的形式，用初始化列表中的值来初始化
- 对于其它类型，需要使用构造函数来初始化

聚合类型：

- 普通数组， int[10]，char[6]
- 类型是一个类(struct，class，union)并且：
  - 无用户自定义的构造函数
  - 无私有或保护的非静态数据成员
  - 无基类
  - 无虚函数
  - 不能有直接初始化的非静态数据成员

## 类类型初始化列表初始化的限制

- 含有私有或者受保护的非静态数据成员的类不能使用初始化列表
- 含有静态成员的类，不能使用初始化列表初始化静态成员
- 含有虚函数的类，不能使用初始化列表初始化
- 有基类的类不能使用初始化列表初始化
- 类中包含直接初始化的非静态数据成员，不能使用初始化列表初始化

```
struct ST
{
	int x;
	double y;
protected:
	int z;
};
ST s{ 1,2.012,3 }; //@ 错误，类中有受保护的(私有的)非静态数据成员

struct ST
{
	int x;
	double y;
private:
	static int z;
};
ST s1{ 1,2.012}; //@ 正确
ST s2{ 1,2.012，3}; //@ 错误，静态成员不能通过初始化列表初始化，需要遵守静态成员的初始化方式

struct ST
{
	int x;
	double y;	
	virtual void func() {};
};
ST s{ 1,2.012}; //@ 错误，类中含有虚函数


struct Base {};
struct Derived : public Base
{
	int x;
	double y;	
};
Derived d{ 1,2.012}; //@ 错误，类有基类


struct ST
{
	int x;
	double y{ 0.0 }; //@ C++ 11 非静态数据成员允许声明时初始化，可以使用 {} 或者 =
};
ST s{1,2.3}; //@ 错误，类中包含了直接初始化的非静态数据成员
```

对于上述非聚合类型的类，要想使用初始化列表的唯一方法就是定义一个构造函数：

```
struct Base { virtual void func() {}; };
struct Derived : public Base
{
	Derived(int xx, double yy, int zz) : x(xx), y(yy), z(zz) {}
	int x;
	double y;

	virtual void func() {};

private:
	int z;
};
Derived d{ 1,2.012,3 };  //@ 正确
```

## 防止类型收窄

类型收窄是导致数据内容发生变化或者精度丢失的转换，初始化列表不支持这种转换：

- 浮点数转换成一个整型数，int i{2.1};
- 高精度浮点数转换成低精度浮点数，如从 long double 转换成 double 或者 float
- 整型数转换成浮点数，并且超过了浮点数的表示范围：float  x { (unsigned long long) -1};
- 整型数转换成长度较短的整型数，并且超过了较短整型数的表示范围，char x {65535}；

# 范围 for

```
std::vector<int> vec{ 1,2,3 };

for (auto v : vec)  //@ 对 v 的修改不会同步到 vec  
    //@ do something

for(auto &v : vec) //@ 对 v 的修改会同步到 vec 
	//@ do something

for(const auto& v : vec) //@ 对 v 不执行拷贝，并且不支持对 v 修改
	//@ do something
```

使用注意事项：

- `:` 后面的表达式只执行一次
- 防止在范围 for 中迭代器失效

```
std::vector<int> get_vec()
{
	std::cout << "get_vec()" << std::endl;
	return { 1,2,3 };
}
for (auto const v : get_vec()) //@ get_vec 函数只会执行一次，不会在每次遍历时反复执行
	std::cout << v << " ";

//@ 改变容器将会导致迭代器失效
std::vector<int> vec{ 1,2,3 };
for (auto const v : vec)
{
	std::cout << v << " ";
	vec.push_back(0);  //@ 迭代器失效
}
```

范围 for 的原理：

- 如容器是一个普通的数组对象，那么 begin 将是数组的首地址，end 将是数组首地址加上容器长度
- 如果容器是一个类对象，那么范围 for 将试图通过查找类的 begin() 和 end() 函数来定位 begin 和 end
- 否则范围 for 将使用全局的 begin() 和 end() 函数来定位 begin 和 end

自定义类型如果想使用范围 for ，需要分别实现 begin() 和 end() 函数。

# std::function 和 std::bind

## 可调用对象

- 函数指针
- 具有 `operator()` 成员函数的类对象(仿函数)
- 可以转换成函数指针的类对象
- 类的成员(函数)指针

```
void func(void)
{
	std::cout << "func" << std::endl;
}

struct Foo
{
	void operator()(void)
	{
		std::cout << "functor" << std::endl;
	}
};

struct Bar
{
	using fr_t = void(*)(void);

	static void func(void)  //@ 必须是静态函数，没有隐式的 this 参数
	{
		std::cout << "Bar::func" << std::endl;
	}

	operator fr_t(void)
	{
		return func;
	}
};

struct A
{
	int i_;
	void mem_func(void)
	{
		std::cout << "mem_func" << std::endl;
	}
};

int main()
{
	//@ 函数指针,也可以直接写成 void(*func_ptr)(void) = func;
	void(*func_ptr)(void) = &func; 
	func_ptr();

	//@ 仿函数
	Foo foo;
	foo(); 

	//@ 可转换为函数指针的类对象
	Bar bar;
	bar();  

	void (A::*mem_func_ptr)(void) = &A::mem_func;  //@ 类成员函数指针，这里必须使用 & 
	int A::*mem_obj_ptr = &A::i_;	 //@ 类成员指针
	A a;
	(a.*mem_func_ptr)();
	a.*mem_obj_ptr = 123;

	return 0;
}
```

## std::function

std::function 是可调用对象的包装器，是一个类模板，可以容纳除了类成员(函数)指针以外所有的可调用对象。

```
void func(void)
{
	std::cout << __FUNCTION__ << std::endl;
}

class Foo
{
public:
	static int foo_func(int a)
	{
		std::cout << __FUNCTION__ << "("<<a<<")"<<std::endl;
		return a;
	}
};

class Bar
{
public:
	int operator()(int a)
	{
		std::cout << __FUNCTION__ << "(" << a << ")" << std::endl;
		return a;
	}
};

int main()
{
	std::function<void(void)> fr1 = func;  //@ 绑定一个普通函数
	fr1();
	
	std::function<int(int)> fr2 = Foo::foo_func; //@ 绑定类的静态成员函数
	fr2(2);

	Bar bar;
	std::function<int(int)> fr3 = bar; //@ 绑定一个仿函数
	fr3(1);

	return 0;
}
```

## std::bind

不论是普通函数、函数对象、还是成员函数，成员变量都可以绑定。

std::bind 作用：

- 将可调用对象与其参数绑定成一个仿函数
- 改变函数调用时需要传参的个数和顺序
- 先将可调用的对象保存起来，在需要的时候再调用，是一种延迟计算的思想

改变参数的个数和调用顺序：

```
void func(int a, char c, float f)
{
	std::cout << a << " " << c << " " << f << std::endl;
}

int main()
{
	//@ 使用占位符
	auto f1 = std::bind(func, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3);
	f1(1, 'c', 3.14);

	//@ 改变传参顺序
	auto f2 = std::bind(func, std::placeholders::_2, std::placeholders::_3, std::placeholders::_1);
	f2(3.14,1,'c');

	//@ 使用固定参数改变参数个数
	auto f3 = std::bind(func, std::placeholders::_1, std::placeholders::_2, 3.14);
	f3(1,'c');
	f3(1, 'c', 9.999); //@ 第三个参数将会被忽略掉

	return 0;
}
```

绑定类的成员(函数)：

```
class Test
{
public:
	int mem_func(int a) 
	{
		std::cout << a << std::endl;
		return a + 1;
	}

	void mem_func2(int a, char c, float f)
	{
		std::cout << a << " " << c << " " << f << std::endl;
	}

	static int static_mem_func(int a)
	{
		std::cout << a << std::endl;
		return a + 100;
	}

public:
	int mem_variable = 1024;
};


int main()
{
	Test t;
	
	//@ 绑定成员函数
	auto f1 = std::bind(&Test::mem_func,t,std::placeholders::_1);
	f1(2);

	//@ 绑定多参数成员函数，改变传参顺序
	auto f2 = std::bind(&Test::mem_func2, t, std::placeholders::_3, std::placeholders::_2, std::placeholders::_1);
	f2(2.13, 'a',100);

	//@ 绑定静态成员函数
	auto f3 = std::bind(&Test::static_mem_func,std::placeholders::_1);
	f3(100);

	//@ 绑定成员变量
	auto f4 = std::bind(&Test::mem_variable,std::placeholders::_1);
	int var = f4(t);
	std::cout << var << std::endl;

	return 0;
}
```

绑定仿函数：

```
struct Test
{
	void operator()(int n)
	{
		std::cout << n << std::endl;
	}
};

int main()
{
	auto f1 = std::bind(Test(),std::placeholders::_1);
	f1(1000);

	Test t;
	auto f2 = std::bind(t, 10);
	f2();

	return 0;
}
```

std::function 不能容纳类的成员(函数)但是 std::bind 可以绑定类的成员(函数)，std::bind 之后的对象可以使用 std::function 表示，从而实现可调用对象的统一表示方法。

std::bind1st 和 std::bind2nd 是旧标准库中用于将二元算子转换成一元算子的方法：

```
	std::vector<int> vec{ 1,2,3,4,5,6,7,8,9,10 };

	//@ 旧标准
	int count1 = std::count_if(vec.begin(), vec.end(), std::bind1st(std::less<int>(), 5)); //@ 第一个参数固定为5，查找大于 5 的元素个数
	int count2 = std::count_if(vec.begin(), vec.end(), std::bind2nd(std::less<int>(), 5)); //@ 第二个参数固定为5，查找小于 5 的元素个数

	//@ 新标准
	int count3 = std::count_if(vec.begin(), vec.end(), std::bind(std::less<int>(), 5, std::placeholders::_1)); 
	int count4 = std::count_if(vec.begin(), vec.end(), std::bind(std::less<int>(), std::placeholders::_1, 5)); 
```

复合多个函数(闭包)：

```
std::vector<int> vec{ 1,2,3,4,5,6,7,8,9,10 };

using std::placeholders::_1;

auto f = std::bind(std::logical_and<bool>(),
    std::bind(std::greater<int>(),_1,5),
    std::bind(std::less_equal<int>(), _1, 10)
    );

int count = std::count_if(vec.begin(),vec.end(),f);
```

# labmbda 表达式

lambda 表达式定义了一个匿名函数，并且可以捕获一定范围内的变量，其语法形式：

```
[capture](params) opt->ret{ body; };
```

- capture：捕获列表
- params：参数列表
- opt：函数选项
- ret：返回值类型
- body：函数体

## 捕获列表

- [] : 不捕获任何变量
- [&] : 捕获外部作用域中的所有变量，并作为引用在函数体中使用(按引用捕获)
- [=] : 捕获外部作用域中的所有变量，并作为副本在函数体中使用(按值捕获)
- [=，&foo] : 按引用捕获 foo，并按值捕获所有其它变量
- [bar] : 按值捕获 bar 变量，不捕获其它变量
- [&bar] : 按引用捕获 bar 变量，不捕获其它变量
- [this] : 捕获当前类的 this 指针，让 lambda 表达式拥有当前成员函数同样的访问全选，使用 [&] 和 [=] 捕获，默认添加此项

按值捕获时，默认情况下 lambda 体中不允许修改变量的值，如果想要修改变量的值：

- 按引用捕获，修改变量后外部变量同步修改
- 使用 mutable 修饰，修改变量后外部变量不会同步修改。如果使用 mutable 修饰，即使没有参数，也需要写明参数列表

```
int a = 1;
auto f = [=]() {return ++a; };  //@ 错误，不允许修改
auto f = [=]() mutable {return ++a; };  //@ 可以修改 a，但是 lambda 表达式外部的 a 的值不变
auto f = [&]() mutable {return ++a; };  //@ 可以修改 a，但是 lambda 表达式外部的 a 的值同步改变
```

按值捕获时，需要注意延迟调用的副作用：

```
int a = 0;
auto f = [=] { return a; };
a += 1;
std::cout << f() << std::endl;  //@ 值是 0，如果想同步上下文，需要按引用捕获
```

## 参数列表

默认情况下，参数列表为空时，可以省略参数列表：

```
auto f = []	{ return 1;	}; //@  正确
```

## 返回值

返回值采用后置的语法实现。

允许省略返回值类型定义，编译器会根据语句自动推导：

- 采用自动推导时，如果有多处返回，类型需要保持一致
- 不能推导初始化列表

```
//@ 错误，lambda 无法推导返回类型
auto f = [](int x) 
{
    if (x & x)
   	 	return 0.0;
    else
    	return 1;
};

//@ 错误，无法根据初始化列表自动推导
auto f = []()
{
	return { 1,2 };
};
```

## lambda 表达式的类型

lambda 表达式在 C++ 11 中称为闭包类型，可以将其理解为一个带 operator() 的类，即仿函数。因而可以使用 std::function 来存储 lambda 表达式，也可以使用 std::bind  来操作 lambda 表达式。

lambda  表达式的 operator() 是 const 的，这也是为什么按值捕获无法修改变量的本质原因，使用 mutable 则取消了 operator() 的 const ，因而可以修改变量。

```
std::function<int(int)> f1 = [](int a) {return a; };
std::function<int(void)> f2 = std::bind([](int a) { return a; },123);
```

没有捕获任何变量的 lambda 表达式，可以转换成一个普通的函数指针：

```
using func_t = int(*)(int);
func_t f = [](int a) { return a; };
f(12);
```

## lambda 表达式的优点

lambda 表达式来源于函数式编程的概念，其具有以下优点：

- 声明式编程风格：就地匿名定义目标函数或者函数对象，不需要额外写一个命名函数或者函数对象，不需要额外写函数或者函数对象，避免了代码膨胀和功能分散
- 更灵活：在需要的时间地点实现函数闭包

声明式编程：

```
std::vector<int> vec{1,2,3,4,5,6,7,8,9};
int even_count{ 0 };
std::for_each(vec.begin(), vec.end(), [&even_count](int x) {if (!(x & 1)) ++even_count; }); //@ 不需要提前定义仿函数
```

在需要的时间地点实现函数闭包：

```
std::vector<int> vec{1,2,3,4,5,6,7,8,9};
std::count_if(vec.begin(), vec.end(), [](int val) {return val > 5 && val <= 10; });  //@ 大于 5 小于等于10
```

# std::tuple

std::tuple 元组是一个固定大小的不同类型值的集合。可以当作结构体用，又不需要创建结构体，但是对于多段结构体，为了可读性，建议不使用。

## 创建 std::tuple

- 使用 std::make_tuple
- 使用 std::tie 创建左值引用元组
- 使用 std::forward_as_tuple 创建右值引用元组

```
std::tuple<std::string, int> tp = std::make_tuple("hello",1);

int x = 1;
std::string str = "hello";
auto tp = std::tie(str, x); //@ tp-> std::tuple<std::string&, int&>

void print_pack(std::tuple<std::string&&, int&&> pack) 
{
	std::cout << std::get<0>(pack) << ", " << std::get<1>(pack) << '\n';
}
std::string str("John");
print_pack(std::forward_as_tuple(str + " Smith", 25));
```

## 获取 std::tuple 的值

- 使用 std::get  获取元组中的元素
- 使用 std::tie 解包，使用 std::ignore 忽略不想解包的元素

```
auto tp = std::make_tuple("hello",123);
auto elem1 = std::get<0>(tp); //@ 获取元组第一个元素
auto elem2 =  std::get<1>(tp); //@ 获取元组第二个元素

std::string str; 
int i;
std::tie(str, i) = tp;

std::string s; 
std::tie(s, std::ignore) = tp;
```

## 连接两个 std::tuple

```
auto tp1 = std::make_tuple("hello",123);
std::tuple<float, std::string, int> tp2{ 3.14,"world",8 };
auto tp = std::tuple_cat(tp1,tp2); //@  "hello", 123, 3.14000010, "world", 8
```

