# 稳定性与兼容性

## 宏

C++ 11 中定义了与预处理指令 `#pragma` 功能相同的操作符 `_Pragma`，格式为：

```
_Pragma (字符串字面量)
```

例如：

```
_Pragma("once");
```

变长参数的宏定义是指在宏定义中参数列表的最后一个参数为省略号，预定义宏 `__VA_ARGS__` 则可以在宏定义实现部分替换省略号所代表的字符串：

```
#define LOG(...)                                              \
    {                                                         \
        fprintf(stderr, "%s:Line %d:\t", __FILE__, __LINE__); \
        fprintf(stderr, __VA_ARGS__);                         \
        fprintf(stderr, "\n");                                \
    }
```

##　long long

C++ 11 整型的最大改变就是多了 long long 类型：long long 和 unsigned long long，可以使用 LL 后缀或者 ll 后缀 标识 long long 类型，ULL，ull，Ull，uLL 标识 unsigned long long 类型的字面量。

C++ 11 要求 long long 类型在不同的平台上有不同的长度，但是至少 64 位。

## 断言

断言就是将一个返回值总是需要为真的判别式放在语句中，用于排除在设计的逻辑上不应该产生的情况。`<cassert>` 中定义的 `assert` 宏用于运行时断言。使用 `NDEBUG` 可以禁用 `assert` 宏：

```
#ifdef NDEBUG
#define assert(expr)   (static_cast<void>(0))
#else
...
#endif
```

静态断言 `static_assert(表达式,警告信息)` 作用在编译时期，`static_assert` 的断言表达式的结果必须是在编译时期可以计算的表达式，即必须是常量表达式。

## noexcept 修饰符与 noexcept 操作符

`noexcept` 修饰符有两种形式：

- 简单地在函数声明后加上 `noexcept`

```
void func() noexcept;
```

- 可以接受一个常量表达式作为参数，常量表达式会被转换成一个 bool 类型的值，该值为 true 表示函数不会抛出异常，反之则可能抛出异常

```
void func() noexcept(常量表达式);
```

C++ 11 中如果 `noexcept` 修饰的函数抛出了异常，编译器可以直接选择调用 `std::terminate()` 函数终止程序。

`noexcept` 操作符通常可以用于模板：

```
template <typename T>
void func() noexcept(noexcept(T()))
{}
```

第二个 `noexcept` 是一个操作符，其参数 `T()` 有可能抛出异常的时候返回 false，否则返回 true。

C++ 11 标准中让类的析构函数默认是 `noexcept(true)` 的，如果显示的指定了析构函数 `noexcept` 或者类的基类或者成员中有 `noexcept(false)` 析构函数，析构函数就不会保持默认值。

## 快速初始化成员变量

C++ 11 中允许对非静态成员变量使用 `=` 或者 `{}` 执行就地初始化。

```
struct TEST
{
    std::string s1{"hello"};  //@ OK
    std::string s2 = "hello"; //@ OK
    std::string s3("hello");  //@ ERROR
};
```

对于非常量的静态成员变量，C++ 11 与 C++ 98 保持一致，程序需要在头文件外定义它，这会保证编译时，类静态成员的定义最后只存在于一个目标文件中。

## 扩展的 friend 语法

firend 关键字用于声明类的友元，友元可以无视类中成员的属性，无论成员是 public，protected，private的，友元类或友元函数都可以访问，这就破坏了封装性，因而存在一定的争议。

C++ 11 中声明一个类为另一个类的友元时， 不需要使用 class 关键字。这一改变使得可以为类模板声明友元。

```
class P;

template <typename T>
class People
{
	friend T;
};

People<P> pp;	//@ 类型 P 是 People 类型的友元
People<int> pi;	//@ 对于 int 类型模板参数，友元声明被忽略
```

