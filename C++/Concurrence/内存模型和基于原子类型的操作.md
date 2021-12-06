为了避免 race condition，线程就要规定执行顺序：

- 一种方式是使用 mutex，后一线程必须等待前一线程解锁
- 第二种方式是使用原子操作来避免竞争访问同一内存位置

原子操作是不可分割的操作，要么做了要么没做，不存在做一半的状态。

# C++ 中的原子操作和原子类型

## 标准原子类型

标准原子类型定义在 `<atomic>` 中。也可以用 mutex 模拟原子操作，实际上标准原子类型可能就是这样实现的，它们都有一个 is_lock_free 函数，返回 true  说明该原子类型操作是无锁的，用的是原子指令，返回 false 则是用锁。

```
struct A { int a[1000]; };
struct B { int x, y; };

std::cout << std::boolalpha
    << std::atomic<A>{}.is_lock_free()
    << "\n"
    << std::atomic<B>{}.is_lock_free()
    << "\n";
```

原子操作的主要用处是替代 mutex 实现同步。如果原子操作内部是用 mutex 实现的，就不会有期望的性能提升，还不如直接用 mutex 来同步。C++17 中每个原子类型都有一个 is_always_lock_free  成员变量，为 true 时表示该原子类型在此平台上 lock-free。

```
std::cout << std::atomic<int>{}.is_always_lock_free; //@ 1
```

C++17 之前可以用标准库为各个原子类型定义的 ATOMIC_xxx_LOCK_FREE 宏来判断该类型是否无锁：

- 值为 0 表示原子类型是有锁的
- 值为 2 表示无锁
- 值为 1 表示运行时才能确定

```
//@ VS2017中的定义
#define ATOMIC_BOOL_LOCK_FREE        2
#define ATOMIC_CHAR_LOCK_FREE        2
#define ATOMIC_CHAR16_T_LOCK_FREE    2
#define ATOMIC_CHAR32_T_LOCK_FREE    2
#define ATOMIC_WCHAR_T_LOCK_FREE     2
#define ATOMIC_SHORT_LOCK_FREE       2
#define ATOMIC_INT_LOCK_FREE         2
#define ATOMIC_LONG_LOCK_FREE        2
#define ATOMIC_LLONG_LOCK_FREE       2
#define ATOMIC_POINTER_LOCK_FREE     2
```

标准库中为 std::atomic 对内置类型的特化定义了类型别名，比如：

```
namespace std {
    using atomic_bool = std::atomic<bool>;
    using atomic_char = std::atomic<char>;
}
```

通常类型 `std::atomic<T>` 的别名就是 atomic_T，只有以下几种例外：signed 缩写为 s，unsigned 缩写为 u，long long 缩写为 llong：

```
namespace std {
    using atomic_schar = std::atomic<signed char>;
    using atomic_uchar = std::atomic<unsigned char>;
    using atomic_uint = std::atomic<unsigned>;
    using atomic_ushort = std::atomic<unsigned short>;
    using atomic_ulong = std::atomic<unsigned long>;
    using atomic_llong = std::atomic<long long>;
    using atomic_ullong = std::atomic<unsigned long long>;
}
```

原子类型不允许由另一个原子类型拷贝赋值，因为拷贝赋值调用了两个对象，破坏了操作的原子性。但可以用对应的内置类型赋值：

```
T operator=(T desired) noexcept;
T operator=(T desired) volatile noexcept;
atomic& operator=(const atomic&) = delete;
atomic& operator=(const atomic&) volatile = delete;
```

std::atomic 为支持赋值提供了成员函数：

```
std::atomic<T>::store //@ 替换当前值
std::atomic<T>::load //@ 返回当前值
std::atomic<T>::exchange //@ 替换值，并返回被替换前的值

//@ 与期望值比较，不等则将期望值设为原值并返回false
//@ 相等则将原子值设为目标值并返回true
//@ 在缺少CAS（compare-and-exchange）指令的机器上，weak版本在相等时可能替换失败并返回false
//@ 因此weak版本通常要求循环，而strong版本返回false就能确保不相等
std::atomic<T>::compare_exchange_weak
std::atomic<T>::compare_exchange_strong

std::atomic<T>::fetch_add //@ 原子加法，返回相加前的值
std::atomic<T>::fetch_sub
std::atomic<T>::fetch_and
std::atomic<T>::fetch_or
std::atomic<T>::fetch_xor
std::atomic<T>::operator++ //@ 前自增等价于fetch_add(1)+1
std::atomic<T>::operator++(int) //@ 后自增等价于fetch_add(1)
std::atomic<T>::operator-- //@ fetch_sub(1)-1
std::atomic<T>::operator--(int) //@ fetch_sub(1)
std::atomic<T>::operator+= //@ fetch_add(x)+x
std::atomic<T>::operator-= //@ fetch_sub(x)-x
std::atomic<T>::operator&= //@ fetch_and(x)&x
std::atomic<T>::operator|= //@ fetch_or(x)|x
std::atomic<T>::operator^= //@ fetch_xor(x)^x
```

这些成员函数有一个用来指定内存序的参数 std::memory_order：

```
typedef enum memory_order {
    memory_order_relaxed,
    memory_order_consume,
    memory_order_acquire,
    memory_order_release,
    memory_order_acq_rel,
    memory_order_seq_cst
} memory_order;
```

store 的顺序参数只能是：

- memory_order_relaxed
- memory_order_release
- memory_order_seq_cst (默认)

load的顺序参数只能是：

- memory_order_relaxed
- memory_order_consume
- memory_order_acquire
- memory_order_seq_cst (默认)

## std::atomic_flag

std::atomic_flag 是一个原子的布尔类型，也是唯一保证 lock-free 的原子类型。它只能在  set 和 clear 两个状态之间切换。

如果在初始化时没有明确使用 ATOMIC_FLAG_INIT初始化，那么新创建的 std::atomic_flag 对象的状态是未指定的（unspecified）（既没有被 set 也没有被 clear。）

ATOMIC_FLAG_INIT: 如果某个 std::atomic_flag 对象使用该宏初始化，那么可以保证该 std::atomic_flag 对象在创建时处于 clear 状态。

只支持两种操作：

- test-and-set
  - test_and_set 函数检查 std::atomic_flag 标志，如果 std::atomic_flag 之前没有被设置过，则设置 std::atomic_flag 的标志，并返回先前该 std::atomic_flag 对象是否被设置过，如果之前 std::atomic_flag 对象已被设置，则返回 true，否则返回 false
  - test-and-set 操作是原子的，可以指定 Memory Order
- clear
  - 清除 std::atomic_flag 对象的标志位，即设置 atomic_flag 的值为 false
  - test-and-set 操作是原子的，可以指定 Memory Order

用 std::atomic_flag 实现自旋锁：

```
class  SpinMutex
{
public:
	void lock()
	{
		while(flag.test_and_set(std::memory_order_acquire))
			;
	}

	void unlock()
	{
		flag.clear(std::memory_order_release);
	}

private:
	std::atomic_flag flag = ATOMIC_FLAG_INIT;
};
```

测试：

```
SpinMutex g_mutex;

void fn1()
{
	for (int i = 0; i < 5; i++)
	{
		g_mutex.lock();
		std::cout << "thread\t" << std::this_thread::get_id() << ":\t" << "The fun1 is called !" << std::endl;
		g_mutex.unlock();
	}
}

void fn2()
{
	for (int i = 0; i < 5; i++)
	{
		g_mutex.lock();
		std::cout << "thread\t" << std::this_thread::get_id() << ":\t" << "The fun2 is called !" << std::endl;
		g_mutex.unlock();
	}
}

int main()
{
	std::thread t1(fn1);
	std::thread t2(fn2);
	t1.join();
	t2.join();


	return 0;
}
```

## 其他原子类型

### std::atomic_bool

std::atomic_flag 功能过于局限，甚至无法像布尔类型一样使用，相比之下，`std::atomic<bool>` 更易用。但是 `std::atomic<bool>` 不保证 lock-free，可以用 is_lock_free 检验在当前平台上是否 lock-free。

```
std::atomic<bool> x(true);
x = false;
bool y = x.load(std::memory_order_acquire); //@ 读取x值返回给y
x.store(true); //@ x写为true
y = x.exchange(false, std::memory_order_acq_rel); //@ x用false替换，并返回旧值true给y

bool expected = false; //@ 期望值
/* 不等则将期望值设为x并返回false，相等则将x设为目标值true并返回true
weak版本在相等时也可能替换失败而返回false，因此一般用于循环 */
while (!x.compare_exchange_weak(expected, true) && !expected)
	;
```

### 指针原子类型

指针原子类型 `std::atomic<T*>` 也支持 is_lock_free、load、store、exchange、compare_exchange_weak 和 compare_exchange_strong，与 `std::atomic<bool>` 语义相同。此外指针原子类型还支持运算操作：fetch_add、fetch_sub、++、--、+=、-= 。

```
class A {};
A a[5];
std::atomic<A*> p(a); //@ p为&a[0]
A* x = p.fetch_add(2); //@ p为&a[2]，并返回原始值a[0]
assert(x == a);
assert(p.load() == &a[2]);
x = (p -= 1);  //@ p为&a[1]，并返回给x，相当于x = p.fetch_sub(1) - 1
assert(x == &a[1]);
assert(p.load() == &a[1]);
```

### 整型原子类型

整型原子类型（如 `std::atomic<int>`）在上述操作之外，还支持 fetch_or、fetch_and、fetch_xor、=、&=、^=。

```
std::atomic<int> i(5);
int j = i.fetch_and(3); //@ 101 & 011 = 001，i为1，j为5
```

### 自定义原子类型

如果原子类型是自定义类型，该自定义类型必须可平凡复制（trivially copyable），也就意味着该类型不能有虚函数或虚基类。这可以用 is_trivially_copyable 检验。

```
#include <iostream>
#include <type_traits>

struct A {
	int m;
};

struct B {
	B(B const&) {}
};

struct C {
	virtual void foo();
};

struct D {
	int m;

	D(D const&) = default; // -> trivially copyable
	D(int x) : m(x + 1) {}
};

int main()
{
	std::cout << std::boolalpha;
	std::cout << std::is_trivially_copyable<A>::value << '\n'; //@ true
	std::cout << std::is_trivially_copyable<B>::value << '\n'; //@ false
	std::cout << std::is_trivially_copyable<C>::value << '\n'; //@ false
	std::cout << std::is_trivially_copyable<D>::value << '\n'; //@ true
}
```

自定义类型的原子类型不允许运算操作，只允许 is_lock_free、load、store、exchange、compare_exchange_weak 和 compare_exchange_strong，以及赋值操作和向自定义类型转换的操作。

### C  风格 API

除了每个类型各自的成员函数，原子操作库还提供了通用的 C 风格 API，只不过函数名多了一个 `atomic_` 前缀，参数变为指针类型：

```
std::atomic<int> i(42);
int j = std::atomic_load(&i); //@ 等价于i.load()
```

compare_exchange_weak 和 compare_exchange_strong 的第一个参数是引用，因此 std::atomic_compare_exchange_weak 和 std::atomic_compare_exchange_strong 的参数用的是指针：

```
bool compare_exchange_weak(T& expected, T desired,
	std::memory_order success,
	std::memory_order failure);

template<class T>
bool atomic_compare_exchange_weak(std::atomic<T>* obj,
	typename std::atomic<T>::value_type* expected,
	typename std::atomic<T>::value_type desired);

template<class T>
bool atomic_compare_exchange_weak_explicit(std::atomic<T>* obj,
	typename std::atomic<T>::value_type* expected,
	typename std::atomic<T>::value_type desired,
	std::memory_order succ,
	std::memory_order fail);
```

除 std::atomic_is_lock_free 外，每个自由函数有一个 `_explicit` 后缀版本，`_explicit` 函数额外接受一个 std::memory_order 参数。

```
std::atomic<int> i(42);
//@ i.load(std::memory_order_acquire) 
std::atomic_load_explicit(&i, std::memory_order_acquire); 
```

自由函数不仅可用于原子类型，还为 std::shared_ptr 提供了特化版本：

```
std::shared_ptr<int> p(new int(42));
std::shared_ptr<int> x = std::atomic_load(&p);
std::shared_ptr<int> q;
std::atomic_store(&q, p);
```

这个特化将在 C++20 中弃用，C++20 直接允许 std::atomic 的模板参数为 std::shared_ptr：

```
std::atomic<std::shared_ptr<int>> x; //@ C++20
```

# 内存模型

动态内存模型可理解为存储一致性模型，主要是从行为方面来看多个线程对同一个对象同时(读写)操作时所做的约束，动态内存模型理解起来稍微复杂一些，涉及了内存，Cache，CPU 各个层次的交互，尤其是在共享存储系统中，为了保证程序执行的正确性，就需要对访存事件施加严格的限制。

## 顺序一致性

在多处理器和多线程环境下理解顺序一致性包括两个方面：

- 从多个线程平行角度来看，程序最终的执行结果相当于多个线程某种交织执行的结果
- 从单个线程内部执行顺序来看，该线程中的指令是按照程序事先已规定的顺序执行的(即不考虑运行时 CPU 乱序执行和Memory Reorder)

另外，现代的 CPU 大都支持多发射和乱序执行，在乱序执行时，指令被执行的逻辑可能和程序汇编指令的逻辑不一致，在单线程条件下，CPU 的乱序执行不会带来大问题，但是在多核多线程时代，当多线程共享某一变量时，不同线程对共享变量的读写就应该格外小心，不适当的乱序执行可能导致程序运行错误。因此，CPU 的乱序执行也需要作出适当的约束。

必须对编译器和 CPU 作出一定的约束才能合理正确地优化你的程序，这个约束就是内存模型。

## 内存模型

C++ 中的内存模型：

```
typedef enum memory_order {
    memory_order_relaxed, //@ 无同步或顺序限制，只保证当前操作原子性
    memory_order_consume, //@ 标记读操作，依赖于该值的读写不能重排到此操作前
    memory_order_acquire, //@ 标记读操作，之后的读写不能重排到此操作前
    memory_order_release, //@ 标记写操作，之前的读写不能重排到此操作后
    memory_order_acq_rel, //@ 仅标记读改写操作，读操作相当于acquire，写操作相当于release
    memory_order_seq_cst //@ sequential consistency：顺序一致性不允许重排，所有原子操作的默认选项
} memory_order;
```

std::memory_order 规定了普通访存操作和相邻的原子访存操作之间的次序是如何安排的，在原子类型的 API 中，我们可以通过额外的参数指定该原子操作的访存次序(内存序)，默认的内存序是 std::memory_order_seq_cst。

### memory_order_relexed

- 只保证当前操作的原子性
- 不强制要求并发内存的访问顺序

```
std::atomic<int> x = 0, y = 0;
static int r1, r2;

//@ 线程1
void thr1_func()
{
	r1 = y.load(std::memory_order_relaxed); //@ step_1
	x.store(r1, std::memory_order_relaxed); //@ step_2
}

//@ 线程2
void thr2_func()
{
	r2 = x.load(std::memory_order_relaxed); //@ step_3 
	y.store(42, std::memory_order_relaxed); //@ step_4
}

int main()
{
	std::thread threads[2];
	threads[0] = std::thread(thr1_func);
	threads[1] = std::thread(thr2_func);

	threads[0].join();
	threads[1].join();

	std::cout << "x:" << x.load() << " y:" << y.load() << " r1:" << r1 << " r2:" << r2 << std::endl;

	return 0;
}
```

Relaxed ordering  不允许循环依赖：

```
std::atomic<int> x = 0, y = 0;
static int r1, r2;

//@ 线程1
void thr1_func()
{
	r1 = y.load(std::memory_order_relaxed); //@ step_1
	if (r1 == 42)
		x.store(r1, std::memory_order_relaxed); //@ step_2
}

//@ 线程2
void thr2_func()
{
	r2 = x.load(std::memory_order_relaxed); //@ step_3
	if (r2 == 42)
		y.store(42, std::memory_order_relaxed); //@ step_4
}

int main()
{
	std::thread threads[2];
	threads[0] = std::thread(thr1_func);
	threads[1] = std::thread(thr2_func);

	threads[0].join();
	threads[1].join();

	// @结果不会为 x == 42, y == 42，因为要产生这个结果，step_1 依赖 step_4，step_4 依赖 step_3，step_3 依赖 step_2，step_2 依赖 step_1
	std::cout << "x:" << x.load() << " y:" << y.load() << " r1:" << r1 << " r2:" << r2 << std::endl;

	return 0;
}
```

典型使用场景是自增计数器，比如 std::shared_ptr 的引用计数器，它只要求原子性，不要求顺序和同步：

```
std::atomic<int> cnt = 0;
void f()
{
	for (int i = 0; i < 100; ++i)
	{
		cnt.fetch_add(1, std::memory_order_relaxed);
	}
}

int main()
{
	std::vector<std::thread> v;
	for (int i = 0; i < 100; ++i)
		v.emplace_back(f);
	std::for_each(v.begin(), v.end(), std::mem_fun_ref(&std::thread::join));
	std::cout << cnt << "\n"; //@ 10000
}
```

### memory_order_acquire

- 对读取(load)实施 acquire 语义。相当于插入一个内存读屏障，保证当前线程之后的读写操作不会被重排到该操作之前
- 保证其他线程中在同一原子变量上所有的写操作在当前线程可见

```
std::atomic<int*> x;
int i;

void producer()
{
	int* p = new int(42);
	i = 42;
	x.store(p, std::memory_order_release);
}

void consumer()
{
	int* q;
	while (!(q = x.load(std::memory_order_acquire)));
	assert(*q == 42); //@ 一定不出错
	assert(i == 42); //@ 一定不出错
}

int main()
{
	std::thread t1(producer);
	std::thread t2(consumer);
	t1.join();
	t2.join();

	return 0;
}
```

### memory_order_consume

类似 memory_order_acquire，但只对与这块内存有关的读写操作起作用。

暂时不鼓励使用 memory_order_consume。

### memory_order_acq_rel

- 相当于写操作是 memory_order_release，读操作是 memory_order_acquire
- 当前线程的读写不允许重排到这个写操作之前或之后，其他线程中 release 该原子变量的写操作在修改前可见，并且此修改对其他 acquire 该原子变量的线程可见

```
std::atomic<bool> x = false;
std::atomic<bool> y = false;
std::atomic<int> z = 0;

void write_x()
{
	x.store(true, std::memory_order_release); //@ step_1：happens-before 3（由于3的循环）
}

void write_y()
{
	y.store(true, std::memory_order_release); //@ step_2: happens-before 5（由于5的循环）
}

void read_x_then_y()
{
	while (!x.load(std::memory_order_acquire)); //@ step_3: happens-before 4
	if (y.load(std::memory_order_acquire)) ++z; //@ step_4
}

void read_y_then_x()
{
	while (!y.load(std::memory_order_acquire)); //@ step_5: happens-before 6
	if (x.load(std::memory_order_acquire)) ++z; //@ step_6
}

int main()
{
	std::thread t1(write_x);
	std::thread t2(write_y);
	std::thread t3(read_x_then_y);
	std::thread t4(read_y_then_x);
	t1.join();
	t2.join();
	t3.join();
	t4.join();
	assert(z.load() != 0); //@ z可能为0：134y为false 256x为false，但12之间没有关系
}
```

### memory_order_release

- 对写入 (store) 实施 release 语义。保证在此之前的读写操作不会被重排到该操作之后
- 该操作之前当前线程内的所有写操作，对于其他对这个原子变量进行 acquire 或 assume 的线程可见

```
std::atomic<int*> x;
int i;

void producer()
{
    int* p = new int(42);
    i = 42;
    x.store(p, std::memory_order_release);
}

void consumer()
{
    int* q;
    while (!(q = x.load(std::memory_order_consume)));
    assert(*q == 42); //@ 一定不出错：*q带有x的依赖
    assert(i == 42); //@ 可能出错也可能不出错：i不依赖于x
}

int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join();
    t2.join();
}
```

### memory_order_seq_cst

memory_order_seq_cst 是所有原子操作的默认选项，由于要求全局的线程同步，因此也是开销最大的。

- 读操作相当于 memory_order_acquire
- 写操作相当于 memory_order_release
- 读改写操作相当于 memory_order_acq_rel
- 附加一个单独的 total ordering，即所有线程对同一操作看到的顺序也是相同的











### Release-Consume ordering

对于标记为 memory_order_consume 原子变量 x 的读操作 R，当前线程中依赖于 x  的读写不允许重排到 R 之前，其他线程中对依赖于 x 的变量写操作对当前线程可见。

如果线程 A 对一个原子变量 x 的写操作为 memory_order_release，线程 B 对同一原子变量的读操作为 memory_order_consume，带来的副作用是，线程 A 中所有 dependency-ordered-before 该写操作的其他写操作（non-atomic 和 relaxed atomic），在线程 B 的其他依赖于该变量的读操作中可见。

顺序的规范正在修订中，并且暂时不鼓励使用 memory_order_consume。

典型使用场景是访问很少进行写操作的数据结构（比如路由表），以及以指针为中介的 publisher-subscriber 场景，即生产者发布一个指针给消费者访问信息，但生产者写入内存的其他内容不需要对消费者可见，这个场景的一个例子是 RCU（Read-Copy Update）。

```

```

### Release-Acquire ordering

对于标记为 memory_order_acquire 的读操作R，当前线程的其他读写操作不允许重排到R之前，其他线程中在同一原子变量上所有的写操作在当前线程可见。

如果线程 A 对一个原子变量的写操作W为 memory_order_release，线程 B 对同一原子变量的读操作为 memory_order_acquire，带来的副作用是，线程 A 中所有 happens-before  W 的写操作（non-atomic 和 relaxed atomic）都在线程 B 中可见。

典型使用场景是互斥锁，线程 A 的释放后被线程B获取，则 A 中释放锁之前发生在 critical section 的所有内容都在 B 中可见。

```

```

对于标记为 memory_order_release 的写操作 W，当前线程中的其他读写操作不允许重排到W之后，若其他线程 acquire 该原子变量，则当前线程所有 happens-before 的写操作在其他线程中可见，若其他线程 consume 该原子变量，则当前线程所有 dependency-ordered-before W 的其他写操作在其他线程中可见。

对于标记为 memory_order_acq_rel 的读改写（read-modify-write）操作，相当于写操作是 memory_order_release，读操作是 memory_order_acquire，当前线程的读写不允许重排到这个写操作之前或之后，其他线程中 release 该原子变量的写操作在修改前可见，并且此修改对其他 acquire 该原子变量的线程可见：

```

```

为了使两个写操作有序，将其放到一个线程里：

```
std::atomic<bool> x = false;
std::atomic<bool> y = false;
std::atomic<int> z = 0;

void write_x_then_y()
{
	x.store(true, std::memory_order_relaxed); //@ step_1：happens-before 2
	y.store(true, std::memory_order_release); //@ step_2：happens-before 3（由于3的循环）
}

void read_y_then_x()
{
	while (!y.load(std::memory_order_acquire)); //@ step_3：happens-before 4
	if (x.load(std::memory_order_relaxed)) ++z; //@ step_4
}

int main()
{
	std::thread t1(write_x_then_y);
	std::thread t2(read_y_then_x);
	t1.join();
	t2.join();
	assert(z.load() != 0); //@ z一定不为0：顺序一定为1234
}
```

利用 Release-Acquire ordering 可以传递同步：

```
std::atomic<bool> x = false;
std::atomic<bool> y = false;
std::atomic<int> v[2];

void f()
{
	//@ v[0]、v[1]的设置没有先后顺序，但都happens-before step_1
	v[0].store(1, std::memory_order_relaxed);
	v[1].store(2, std::memory_order_relaxed);
	x.store(true, std::memory_order_release); //@ step_1：happens-before 2（由于2的循环）
}

void g()
{
	while (!x.load(std::memory_order_acquire)); //@ step_2：happens-before 3
	y.store(true, std::memory_order_release); //@ step_3：happens-before 4（由于4的循环）
}

void h()
{
	while (!y.load(std::memory_order_acquire)); //@ step_4 happens-before v[0]、v[1]的读取
	assert(v[0].load(std::memory_order_relaxed) == 1);
	assert(v[1].load(std::memory_order_relaxed) == 2);
}
```

### Sequentially-consistent ordering

memory_order_seq_cst 是所有原子操作的默认选项，由于要求全局的线程同步，因此也是开销最大的。

- 读操作相当于 memory_order_acquire
- 写操作相当于 memory_order_release
- 读改写操作相当于 memory_order_acq_rel
- 附加一个单独的 total ordering，即所有线程对同一操作看到的顺序也是相同的

```
//@ 要么1 happens-before 2，要么2 happens-before 1
void write_x()
{
	x.store(true, std::memory_order_seq_cst); //@ step_1：happens-before 3（由于3的循环）
}

void write_y()
{
	y.store(true, std::memory_order_seq_cst); //@ step_2：happens-before 5（由于5的循环）
}

void read_x_then_y()
{
	while (!x.load(std::memory_order_seq_cst)); //@ step_3：happens-before 4
	if (y.load(std::memory_order_seq_cst)) //@ step_4：如果返回false则一定是1 happens-before 2
		++z;
}

void read_y_then_x()
{
	while (!y.load(std::memory_order_seq_cst)); //@ step_5：happens-before 6
	if (x.load(std::memory_order_seq_cst)) //@ step_6：如果返回false则一定是2 happens-before 1
		++z;
}

int main()
{

	std::thread t1(write_x);
	std::thread t2(write_y);
	std::thread t3(read_x_then_y);
	std::thread t4(read_y_then_x);
	t1.join();
	t2.join();
	t3.join();
	t4.join();
	std::cout << z.load() << std::endl; // @ z可能为1、2，一定不为0, 12之间必定存在happens - before关系
	
	return 0;
}
```



## std::atomic_thread_fence

```
std::atomic<bool> x, y;
std::atomic<int> z;

void f()
{
	x.store(true, std::memory_order_relaxed); //@ step_1：happens-before 2
	std::atomic_thread_fence(std::memory_order_release); //@ step_2：synchronizes-with 3
	y.store(true, std::memory_order_relaxed);
}

void g()
{
	while (!y.load(std::memory_order_relaxed));
	std::atomic_thread_fence(std::memory_order_acquire); //@ step_3：happens-before 4
	if (x.load(std::memory_order_relaxed)) ++z; //@ step_4
}

int main()
{
	x = false;
	y = false;
	z = 0;
	std::thread t1(f);
	std::thread t2(g);
	t1.join();
	t2.join();
	assert(z.load() != 0); //@ step_1 happens-before 4
}
```

将 x 替换为非原子 bool 类型，行为也一样：

```
bool x = false;
std::atomic<bool> y;
std::atomic<int> z;

void f()
{
	x = true; //@ step_1：happens-before 2
	std::atomic_thread_fence(std::memory_order_release); //@ step_2：synchronizes-with 3
	y.store(true, std::memory_order_relaxed);
}

void g()
{
	while (!y.load(std::memory_order_relaxed));
	std::atomic_thread_fence(std::memory_order_acquire); //@ step_3：happens-before 4
	if (x) ++z; //@ step_4
}

int main()
{
	x = false;
	y = false;
	z = 0;
	std::thread t1(f);
	std::thread t2(g);
	t1.join();
	t2.join();
	assert(z.load() != 0); //@ 1 happens-before 4
}
```











