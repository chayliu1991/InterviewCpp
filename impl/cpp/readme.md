# 生命周期和编程范式

## C++ 程序的生命周期

![](./img/cpp_code.png)

###  编译阶段编程

```
template <int N>
struct Fib
{
	static const int value = Fib<N - 1>::value + Fib<N - 2>::value;
};

template <>
struct Fib<1>
{
	static const int value = 1;
};

template <>
struct Fib<0>
{
	static const int value = 1;
};

int main()
{
	std::cout << Fib<2>::value << std::endl; 
	std::cout << Fib<40>::value << std::endl;
	return 0;
}
```

### 静态断言

```
template <int N>
struct Fib
{
	static_assert(N >= 0,"N >= 0");
	static const int value = Fib<N - 1>::value + Fib<N - 2>::value;
};
```

static_assert 运行在编译阶段，只能看到编译时的常数和类型，看不到运行时的变量、指针、内存数据等。  

```
template <typename I,typename P>
struct Test{
	static_assert(std::is_integral<I>::value,"int");
	static_assert(std::is_pointer<P>::value, "ptr");
};
```

## C++ 语言的编程范式：

![](./img/cpp_program_method.png)

### 面向对象编程

面向对象编程的核心思想：封装，继承和多态。

注意：尽量少用继承和虚函数，减少继承的层次，一般不应该超过三层。  

"final" 标识符可以用于禁止继承：

```
struct Interface {
};

struct Implement final : public Interface{
};
```

现代 C++ 中一共有 6 个基本函数：

- 默认构造函数，拷贝构造函数，移动构造函数
- 赋值函数，移动赋值函数
- 析构函数

"default" 可以强制编译器提供某个函数的默认实现，"delete" 可以强制编译器删除某个函数的实现：

```
struct Test final
{
	Test() = default;
	Test(const Test& t) = delete;
};
```

"explicit"  关键字可以防止类隐式类型转换：

```
struct Test
{
	explicit Test(int i)  //@ 显式单参构造
	{
	}
	
	explicit operator bool() //@ 显式转换为 bool 类型
	{}
};
```

#### 委托构造函数

一个构造函数直接调用另一个构造函数，把构造工作“委托”出去，既简单又高效。  

```
struct Test final
{
public:
	Test(int a) : i(a)
	{
	}

	Test() : Test(0)
	{
	}

	Test(const std::string& s) : i(std::stoi(s))
	{
	}
private:
	int i;
};
```

#### 类内初始化

在 C++11 里，你可以在类里声明变量的同时给它赋值，实现初始化，这样不但简单清晰，也消除了隐患。  

```
struct Test final
{
public:
	Test(int a) : i(a) //@ 可以单独初始化某个数据成员
	{
	}

	Test() = default;  //@ 需要默认构造函数
	~Test() = default; //@ 需要默认析构函数

private:
	int i = 10;
	std::string s{"hello"};
	std::vector<int> v{ 1,2,3 };
};
```

#### 类型别名

C++11 扩展了关键字 using 的用法，增加了 typedef 的能力，可以定义类型别名。  

```
struct Test final
{
public:
	using VecI = std::vector<int>;
};
```

# 类型推导

## auto

auto 的“自动推导”能力只能用在“初始化”的场合。  具体来说，就是赋值初始化或者花括号初始化（初始化列表、Initializer list），变量右边必须要有一个表达式。这样你才能在左边放上 auto，编译器才能找到表达式，帮你自动计算类型。如果不是初始化的形式，只是“纯”变量声明，那就无法使用 auto。因为这个时候没有表达式可以让 auto 去推导：

```
auto x = 0UL; //@ x -> unsigned long
auto y = &x;  //@ y -> unsigned long
auto z{ &x }; //@ z -> unsigned long*
auto err; //@ 错误，没有用于推导的表达式
```

类成员变量初始化时候，不能使用 auto 进行类型推导：

```
struct Test final
{
	auto a = 10;  //@ 错误，类内初始化不支持 auto 类型推导
};
```

auto 推导规则：

- auto 总是推导出“值类型”，绝不会是“引用”，auto 可以附加上 const、volatile、*、& 这样的类型修饰符，得到新的类型  
- auto 将忽略顶层 const 而保留底层 const
- auto 可以附加上 const、volatile、*、& 这样的类型修饰符，得到新的类型  

```
int a = 1;
int &r = a;
const int* const  p = &a;
auto b = p; //@ b -> const int*
auto c = r; //@ c -> int
const auto & d = c; //@ d -> const int &
```

## decltype

decltype 的形式很像函数，后面的圆括号里就是可用于计算类型的表达式。

decltype  推导规则：

- decltype 不仅能够推导出值类型，还能够推导出引用类型，也就是表达式的“原始类型”
- auto 可以附加上 const、volatile、*、& 这样的类型修饰符，得到新的类型  

```
int x = 0;
int& r = x;

decltype(x) x1; //@ x1 -> int
decltype(&x) x2; //@ x2 -> int*
decltype(x)& x3 = x; //@ x3 -> int&
decltype(r)& x4 = x; //@ x4 -> int&
decltype(x)* x5; //@ x5 -> int*

const decltype(x) x6 = 0; //@ x6 -> const int
```

完全可以把 decltype 看成是一个真正的类型名，用在变量声明、函数参数 / 返回值、模板参数等任何类型能出现的地方，只不过这个类型是在编译阶段通过表达式“计算”得到的。  

```
int x;
using int_ptr = decltype(&x);
```

它也有个缺点，就是写起来略麻烦，特别在用于初始化的时候，表达式要重复两次（左边的类型计算，右边的初始化），把简化代码的优势完全给抵消了。所以，C++14 就又增加了一个“decltype(auto)”的形式，既可以精确推导类型，又能像 auto 一样方便使用。  

```
int x = 0;

decltype(auto) x1 = (x); //@ x1->int&,因为 (expr) 是引用类型
decltype(auto) x2 = &x; //@ x2->int*
decltype(auto) x3 = x1; //@ x1->int&

auto a = (x);  //@ a -> int，auto 推导只是值推导
decltype((x)) a = x;  //@ a -> int&，decltype 推导是完整类型推导
```

## auto 与 decltype 的应用

auto：

- 在变量声明时应该尽量多用 auto，尤其是在 range-based-for 中
- 在 C++14 里，auto 还新增了一个应用场合，就是能够推导函数返回值  

```
auto get_vec()
{
	return std::vector<int>{1,2,3,4};
}
```

decltype：

它是 auto 的高级形式，更侧重于编译阶段的类型计算，所以常用在泛型编程里，获取各种类型，配合 typedef 或者 using 会更加方便。当你感觉“这里我需要一个特殊类型”的时候，选它就对了。  

```
//@ UNIX 信号函数原型
void(*signal(int signo, void(*func)(int)))(int);
using sig_func_ptr_t = decltype(&signal);
```

在定义类的时候，因为 auto 被禁用了，所以这也是 decltype 可以“显身手”的地方。它可以搭配别名任意定义类型，再应用到成员变量、成员函数上，变通地实现 auto 的功能。

```
struct Test final
{
    using set_type = std::set<int>;

    set_type set_;
    using iter_type = decltype(set_.begin());
    iter_type pos_;
};
```

# const/volatile/mutable

## const  和 volatile

最简单的用法就是，定义程序用到的数字、字符串常量，代替宏定义。它和宏定义还是有本质区别的：const 定义的常量在预处理阶段并不存在，而是直到运行阶段才会出现。所以，准确地说，它实际上是运行时的“变量”，只不过不允许修改，是“只读”的，叫“只读变量”更合适。既然它是“变量”，那么，使用指针获取地址，再“强制”写入也是可以的：      

```
//@ 需要加入 volatile 修饰，运行时才能看到效果
const volatile int kMaxLen = 1024;
auto ptr = (int*)(&kMaxLen);

*ptr = 2048;
std::cout << kMaxLen << std::endl; //@ 输出 2048
```

编译器会对“真正的常数”进行优化，例如在用到常数的时候直接替换，const 常量虽然不是“真正的常数”，但在大多数情况下，它都可以被认为是常数，在运行期间不会改变。所以，对于没有 volatile 修饰的 const 常量来说，虽然你用指针改了常量的值，但这个值在运行阶段根本没有用到，因为它在编译阶段就被优化掉了。    

volatile  含义是“不稳定的”“易变的”，在 C++ 里，表示变量的值可能会以“难以察觉”的方式被修改（比如操作系统信号、外界其他的代码），所以要禁止编译器做任何形式的优化，每次使用的时候都必须“老老实实”地去取值。 kMaxLen 虽然是个“只读变量”，但加上了 volatile 修饰，就表示它不稳定，可能会悄悄地改变。编译器在生成二进制机器码的时候，不会再去做那些可能有副作用的优化，而是用最“保守”的方式去使用 kMaxLen。也就是说，编译器不会再把 kMaxLen 替换为 1024，而是去内存里取值。

volatile 会禁止编译器做优化，所以除非必要，应当少用 volatile，最好不要用。

## 基本的 const 用法  

常量引用：

```
int x = 0;

const int& cr = x;
```

const& 被称为万能引用，它可以引用任何类型，即不管是值、指针、左引用还是右引用，它都能“照单全收”。而且，它还会给变量附加上 const 特性，这样“变量”就成了“常量”，只能读、禁止写。  在设计函数的时候，尽可能地使用它作为入口参数，一来保证效率，二来保证安全。    

const 与指针：

常量指针：表示指针指向的内容是常量，不能通过指针修改其值

```
int x = 0;
int y = 1;

const int* cp = &x;
*cp = y;	//@ 不允许使用指针修改变量的值
cp = &y;	//@ OK
```

指针常量：指针本身是常量，不能指向别的对象，但是它指向的变量的值可以修改

```
int x = 0;
int y = 1;

int* const cp = &x;
*cp = y; //@ OK
cp = &y; //@ 不允许，指针本身是常量类型
```

指向常量的指针常量：

```
int x = 0;
int y = 1;

const int* const cp = &x;
*cp = y; //@ 不允许
cp = &y  //@ 不允许
```

顶层 const 与底层 const

- 顶层 const：指 const 定义的变量本身是一个常量
- 底层 const：指 const 定义的变量所指向的对象是一个常量

```
const int i = 0; 　　 　　//@ 顶层 const，变量i就是常量
const int  * a =  &i;    //@ 底层 const, a 所指向的对象 *a 是常量
int * const b = &i;　　　 //@ 顶层 const, 变量 b 本身就是一个常量
```

- 执行对象拷贝时有限制，常量的底层 const 不能赋值给非常量的底层 const
- const_cast 只能改变运算对象的底层const

```
int num = 3;
const int * p = &num;
int * p2 = p; //@ 不允许

int *p3 = const_cast<int*>(p);  //@ OK
```

## 与类相关的 const 用法  

const 成员函数，const 成员变量：

```
struct Test final
{
public:
	int get_value() const
	{
		return value;
	}
	
private:
	const long kMaxSize = 256L;
	int value;
};
```

const 成员函数的作用：函数的执行过程是 const 的，不会修改对象的状态（即成员变量），也就是说，成员函数是一个“只读操作”。  

因为“常量引用”“常量指针”关联的对象是只读、不可修改的，那么也就意味着，对它的任何操作也应该是只读、不可修改的，否则就无法保证它的安全性。所以，编译器会检查const 对象相关的代码，如果成员函数不是 const，就不允许调用。  

![](./img/const.png)

## mutable  

mutable 却只能修饰类里面的成员变量，表示变量即使是在 const 对象里，也是可以修改的。换句话说，就是标记为 mutable 的成员不会改变对象的状态，也就是不影响对象的常量性，所以允许 const 成员函数改写 mutable 成员变量。  

对于有特殊作用的成员变量，你可以给它加上 mutable 修饰，解除 const 的限制，让任何成员函数都可以操作它。  

```
struct Test final
{
public:
	void save_data() const
	{
		//@ do something wirth mutex
	}
private:
	mutable std::mutex mutex;
};
```

不过要当心，mutable 也不要乱用，太多的 mutable 就丧失了 const 的好处  

# 智能指针

C++ 里也是有垃圾回收的，不过不是 Java、Go 那种严格意义上的垃圾回收，而是广义上的垃圾回收，这就是构造 / 析构函数和 RAII 惯用法（Resource Acquisition Is Initialization）。  

在现代 C++ 中，绝对不要再使用“裸指针（naked pointer）”了，而是应该使用“智能指针（smart pointer）”。  

## unique_ptr 

```
std::unique_ptr<std::string> ptr(new std::string("hello"));
assert(ptr != nullptr); //@ 可以判断是否为空指针
assert(*ptr == "hello"); //@ 可以判断是否为空指针
ptr->size(); //@ 可以使用->调用成员函数
```

unique_ptr 虽然名字叫指针，用起来也很像，但它实际上并不是指针，而是一个对象。所以，不要企图对它调用 delete，它会自动管理初始化时的指针，在离开
作用域时析构释放内存。  

unique_ptr 没有定义算术运算：

```
std::unique_ptr<int> ptr(new int (42));
ptr ++; //@ 不允许
ptr += 2; //@ 不允许
```

未初始化的 unique_ptr 表示空指针，这样就相当于直接操作了空指针，运行时就会产生致命的错误（比如 core dump）：

```
std::unique_ptr<int> ptr;
*ptr = 42;
```

C++ 14 中工厂函数 make_unique()，强制创建智能指针的时候必须初始化。  在 C++ 11 中实现：

```
template <typename T,typename... Args>
std::unique_ptr<T> make_unique(Args&& ... args)
{
	return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

unique_ptr  表示指针的所有权是“唯一”的，不允许共享，任何时候只能有一个“人”持有它  

unique_ptr 应用了 C++ 的“转移”（move）语义，同时禁止了拷贝赋值，所以，在向另一个 unique_ptr 赋值的时候，要特别留意，必须用 std::move() 函
数显式地声明所有权转移。  赋值操作之后，指针的所有权就被转走了，原来的 unique_ptr 变成了空指针，新的unique_ptr 接替了管理权，保证所有权的唯一性：  

```
std::unique_ptr<int> ptr = make_unique<int>(42);
assert(ptr); //@ OK
std::unique_ptr<int> ptr2 = std::move(ptr);
assert(ptr); //@  错误
```

## shared_ptr

```
std::shared_ptr<std::string> ptr(new std::string("hello"));	
assert(*ptr == "hello"); //@ 可以使用*取内容
ptr->size(); //@ 可以使用->调用成员函数
assert(ptr != nullptr); //@ 可以判断是否为空指针

auto ptr2 = std::make_shared<std::string>("zelda"); //@ 工厂函数创建智能指针
```

shared_ptr 的所有权是可以被安全共享的，也就是说支持拷贝赋值，允许被多个“人”同时持有，就像原始指针一样。  

```
auto ptr = std::make_shared<std::string>("hello"); 
assert(ptr.unique()); //@ 此时智能指针唯一
auto ptr2 = ptr;  //@ 直接拷贝，不需要 move
assert(ptr.use_count() == 2); //@ 引用计数为 2
```

shared_ptr 支持安全共享的秘密在于内部使用了“引用计数”。引用计数最开始的时候是 1，表示只有一个持有者。如果发生拷贝赋值——也就是共享的时候，引用计数就增加，而发生析构销毁的时候，引用计数就减少。只有当引用计数减少到0，也就是说，没有任何人使用这个指针的时候，它才会真正调用 delete 释放内存。  

因为 shared_ptr 具有完整的“值语义”（即可以拷贝赋值），所以，它可以在任何场合替代原始指针，而不用再担心资源回收的问题。

shared_ptr  缺陷：

- 引用计数的存储和管理都是成本
- 导致循环引用 

## weak_ptr  

weak_ptr 专门为打破循环引用而设计，只观察指针，不会增加引用计数（弱引用），但在需要的时候，可以调用成员函数 lock()，获取shared_ptr（强引用）。

# exception

## 异常的使用

异常就是针对错误码的缺陷而设计的，它有三个特点：

- 异常的处理流程是完全独立的，throw 抛出异常后就可以不用管了，错误处理代码都集中在专门的 catch 块里。这样就彻底分离了业务逻辑与错误逻辑，看起来更清楚
- 异常是绝对不能被忽略的，必须被处理。如果你有意或者无意不写 catch 捕获异常，那么它会一直向上传播出去，直至找到一个能够处理的 catch 块。如果实在没有，那就会导致程序立即停止运行，明白地提示你发生了错误，而不会“坚持带病工作”
- 异常可以用在错误码无法使用的场合，这也算是 C++ 的“私人原因”。因为它比 C 语言多了构造 / 析构函数、操作符重载等新特性，有的函数根本就没有返回值，或者返回值无法表示错误，而全局的 errno 实在是“太不优雅”了，与 C++ 的理念不符，所以也必须使用异常来报告错误  

异常的用法和使用方式：

用 try 把可能发生异常的代码“包”起来，然后编写 catch 块捕获异常并处理。  

```
try
{
	//@ do something
}
catch (...)
{
	//@ handle exception
}
```

C++ 里对异常的定义非常宽松，任何类型都可以用 throw 抛出，也就是说，你可以直接把错误码（int）、或者错误消息（char*、string）抛出，catch 也能接
住，然后处理。  

但是最好使用 C++ 已经定义的异常类型体：

![](./img/exception.png)

自定义异常类型：

```
class Exception : public std::runtime_error
{
public:
	Exception(const char* msg) : std::runtime_error(msg)
	{
	}

	Exception() = default;
	~Exception() = default;
};
```

在抛出异常的时候，  最好不要直接用 throw 关键字，而是要封装成一个函数，这和不要直接用 new、delete 关键字是类似的道理——通过引入一个“中间层”来获得更多的可读性、安全性和灵活性。  

```
[[noretrurn]] void raise(const char* msg)
{
	throw Exception(msg);
}
```

使用 catch 捕获异常的时候也要注意，C++ 允许编写多个 catch 块，捕获不同的异常，再分别处理。但是，异常只能按照 catch 块在代码里的顺序依次匹配，而不会去找最佳匹配。 所以，最好只用一个 catch 块，绕过这个“坑”。  

```
try {
	raise("error occured");
}
catch (const std::exception& e)  //@ 使用基类，使用引用
{
	std::cout << e.what() << std::endl;
}
```

function-try：

所谓 function-try，就是把整个函数体视为一个大 try 块，而 catch 块放在后面，与函数体同级并列，这样做的好处很明显，不仅能够捕获函数执行过程中所有可能产生的异常，而且少了一级缩进层次，处理逻辑更清晰，建议多用。   

```
void func(int i)
try {
	raise("error occured");
}
catch (const std::exception& e)  //@ 使用基类，使用引用
{
	std::cout << e.what() << std::endl;
}

int main()
{
	func(102);
	return 0;
}
```

应当使用异常的判断准则：  

- 不允许被忽略的错误
- 极少数情况下才会发生的错误
- 严重影响正常流程，很难恢复到正常状态的错误
- 无法本地处理，必须“穿透”调用栈，传递到上层才能被处理的错误

## 保证不抛出异常  

noexcept 专门用来修饰函数，告诉编译器：这个函数不会抛出异常。编译器看到noexcept，就得到了一个“保证”，就可以对函数做优化，不去加那些栈展开的额外代码，消除异常处理的成本。  

不过你要注意，noexcept 只是做出了一个“不可靠的承诺”，不是“强保证”，编译器无法彻底检查它的行为，标记为 noexcept 的函数也有可能抛出异常：  

```
void func_maybe_noexcept() noexcept
{
	throw "fatal";
}
```

# lambda

## lambda 的形式  

C++ 也鼓励尽量“匿名”使用 lambda 表达式。也就是说，它不必显式赋值给一个有名字的变量，直接声明就能用：

```
std::vector<int> vec{1,2,3,4,8};
std::cout << std::count_if(vec.begin(), vec.end(), [](int x) {return x > 3; })<< std::endl;
```

## lambda 的变量捕获  

- [=] 表示按值捕获所有外部变量，表达式内部是值的拷贝，并且不能修改
- [&] 是按引用捕获所有外部变量，内部以引用的方式使用，可以修改
- 可以在 [] 里明确写出外部变量名，指定按值或者按引用捕获  

## 泛型的 lambda  

在 C++14 里，lambda 表达式又多了一项新本领，可以实现“泛型化”，相当于简化了的模板函数：

```
auto pow2 = [](const auto & x)
{
	return x * x;
};
```

# 字符串

string 是模板类特化形式的别名：

```
using string = std::basic_string<char>;
```

## 字面量后缀  

C++11 为方便使用字符串，新增了一个字面量的后缀“s”，明确地表示它是 string 字符串类型，而不是 C 字符串，这就可以利用 auto 来自动类型推导：

```
auto str = "std string";  //@ str -> const char*

using namespace std::literals::string_literals; //@ 需要打开命名空间
auto str2 = "std string"s;  //@ str2 -> std::string
```

C++11 还为字面量增加了一个“原始字符串”的新表示形式，比原来的引号多了一个大写字母 R 和一对圆括号，就像下面这样：  

```
auto str1 = R"(\r\n\t\)"; //@ 原样输出 \r\n\t
auto str2 = R"(\\\\\\\$)"; //@ 原样输出 \\\\\\\$
```

如果原始字符串中有引号或者圆括号，需要在圆括号的两边加上最多 16 个字符的特别“界定符”（delimiter），这样就能够保证不与字符串内容发生冲突：  

```
auto str = R"==("('xx')")=="; //@ 原样输出 "('xx')"
```

## 字符串转换函数  

- stoi()、stol()、stoll() 等把字符串转换成整数
- stof()、stod() 等把字符串转换成浮点数
- to_string() 把整数、浮点数转换成字符串  

## 字符串视图类  

string 的成本问题。它确实有点“重”，大字符串的拷贝、修改代价很高，所以我们通常都尽量用 const string&，但有的时候还是无法避免。

在 C++17 里，就有这么一个完美满足所有需求的东西，叫 string_view。顾名思义，它是一个字符串的视图，成本很低，内部只保存一个指针和长度，无论是拷贝，还是修改，都非常廉价。  

C++11 里实现一个简化版本：

```
class string_view final
{
public:
	using size_type = size_t;

	string_view() = default;
	~string_view() = default;

	string_view(const std::string& str) noexcept: ptr_(str.data()), len_(str.length())
	{
	}

	const char* data() const
	{
		return ptr_;
	}

	size_type size() const 
	{
		return len_;
	}

private:
	const char* ptr_ = nullptr;
	size_type len_ = 0;
};
```

## 正则表达式  

C++ 正则表达式主要有两个类：

- regex：表示一个正则表达式，是 basic_regex 的特化形式
- smatch：表示正则表达式的匹配结果，是 match_results 的特化形式  

C++ 正则匹配有三个算法，注意它们都是“只读”的，不会变动原字符串：

- regex_match()：完全匹配一个字符串
- regex_search()：在字符串里查找一个正则匹配
- regex_replace()：正则查找再做替换  

在写正则的时候，记得最好要用“原始字符串” 。

# 容器

## 容器的通用特性  

容器里存储的是元素的拷贝、副本，而不是引用。从这个基本特性可以得出一个推论，容器操作元素的很大一块成本就是值的拷贝。  

一个解决办法是，尽量为元素实现转移构造和转移赋值函数，在加入容器的时候使用 std::move() 来“转移”，减少元素复制的成本：  

```
Point pt; //@ 拷贝代价很高的类
vec.push_back(pt); //@ 拷贝对象，代价比较高
vec.push_back(std::move(pt)); //@ 移动构造
```

C++11 为容器新增加的 emplace 操作函数，它可以“就地”构造元素，免去了构造后再拷贝、转移的成本，不但高效，而且用起来也很方便：  

```
struct Test final
{
	Test(double d, const std::string& s) : data(d), str(s)
	{
	}
	double data;
	std::string str;
};

std::vector<Test> vec;
vec.emplace_back(10.8, "hello");
vec.emplace_back(0.8, "hi");
```

## 容器的具体特性  

C++ 里的容器很多，常见的一种分类是依据元素的访问方式，分成顺序容器、有序容器和无序容器三大类别。

### 顺序容器

顺序容器就是数据结构里的线性表，一共有 5 种：array、vector、deque、list、forward_list，按照存储结构，这 5 种容器又可以再细分成两组：

- 连续存储的数组：array、vector 和 deque  
- 指针结构的链表：list 和 forward_list  

array 和 vector 直接对应 C 的内置数组，内存布局与 C 完全兼容，所以是开销最低、速度最快的容器。它们两个的区别在于容量能否动态增长。array 是静态数组，大小在初始化的时候就固定了，不能再容纳更多的元素。而 vector 是动态数组，虽然初始化的时候设定了大小，但可以在后面随需增长，容纳任意数量的元素。    
