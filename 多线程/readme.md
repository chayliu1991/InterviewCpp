

# 生产者消费者

## 单生产者单消费者

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

static const int kItemRepositorySize = 10;
static const int kItemsToProduce = 1000;

struct ItemRepository
{
	int item_buffer[kItemRepositorySize];
	size_t consume_position = 0;
	size_t produce_position = 0;
	size_t item_count = 0;
	std::mutex mtx;
	std::condition_variable not_full;
	std::condition_variable not_empty;
};

ItemRepository g_item_repo;


void produce_task()
{
	for (int i = 0; i < kItemsToProduce; i++)
	{
		{
			std::unique_lock<std::mutex> lock(g_item_repo.mtx);
			g_item_repo.not_full.wait(lock,
				[] {return (g_item_repo.produce_position + 1) % kItemRepositorySize != g_item_repo.consume_position; }
			);
			g_item_repo.item_buffer[g_item_repo.produce_position] = i;
			g_item_repo.produce_position++;
			if (g_item_repo.produce_position == kItemRepositorySize)
				g_item_repo.produce_position = 0;

			std::cout << "produce the: " << i << " item" << std::endl;
			g_item_repo.not_empty.notify_one();
		}

	}
}

void consume_task()
{
	static int cnt = 0;
	while (true)
	{
		{
			std::unique_lock<std::mutex> lock(g_item_repo.mtx);
			g_item_repo.not_empty.wait(lock, []() {return g_item_repo.produce_position != g_item_repo.consume_position; }
			);
			int data = g_item_repo.item_buffer[g_item_repo.consume_position];
			g_item_repo.consume_position++;
			if (g_item_repo.consume_position == kItemRepositorySize)
				g_item_repo.consume_position = 0;

			++cnt;
			std::cout << "consume the: " << data << " item" << std::endl;
			g_item_repo.not_full.notify_one();

			if (cnt == kItemsToProduce)
				break;
		}
	}
}

int main()
{
	std::thread consume_thr(consume_task);
	std::thread produce_thr(produce_task);

	consume_thr.join();
	produce_thr.join();

	return 0;
}
```

## 单生产者多消费者

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

static const int kItemRepositorySize = 5;
static const int kItemsToProduce = 1000;

struct ItemRepository
{
	int item_buffer[kItemRepositorySize];
	size_t consume_position = 0;
	size_t produce_position = 0;
	size_t item_count = 0;
	std::mutex mtx;
	std::mutex item_count_mtx;
	std::condition_variable not_full;
	std::condition_variable not_empty;
};

ItemRepository g_item_repo;


void produce_task()
{
	for (int i = 0; i < kItemsToProduce; i++)
	{
		{
			std::unique_lock<std::mutex> lock(g_item_repo.mtx);
			g_item_repo.not_full.wait(lock,
				[] {return (g_item_repo.produce_position + 1) % kItemRepositorySize != g_item_repo.consume_position; }
			);
			g_item_repo.item_buffer[g_item_repo.produce_position] = i;
			g_item_repo.produce_position++;
			if (g_item_repo.produce_position == kItemRepositorySize)
				g_item_repo.produce_position = 0;

			std::cout << "produce the: " << i << " item" << std::endl;
			g_item_repo.not_empty.notify_one();
		}

	}
}

void consume_task()
{
	while (true)
	{
		{
			std::lock_guard<std::mutex> count_lock(g_item_repo.item_count_mtx);
			if (g_item_repo.item_count < kItemsToProduce)
			{
				std::unique_lock<std::mutex> lock(g_item_repo.mtx);
				g_item_repo.not_empty.wait(lock, []() {return g_item_repo.produce_position != g_item_repo.consume_position; }
				);

				int data = g_item_repo.item_buffer[g_item_repo.consume_position];
				g_item_repo.consume_position++;
				if (g_item_repo.consume_position == kItemRepositorySize)
					g_item_repo.consume_position = 0;

				++g_item_repo.item_count;
				std::cout << std::this_thread::get_id()<<" consume the: " << data << " item" << std::endl;
				g_item_repo.not_full.notify_one();
			}
			else
				break;
		}
	}
}

int main()
{
	std::thread produce_thr(produce_task);
	std::thread consume_thr1(consume_task);
	std::thread consume_thr2(consume_task);
	std::thread consume_thr3(consume_task);
	std::thread consume_thr4(consume_task);
	std::thread consume_thr5(consume_task);

	produce_thr.join();
	consume_thr1.join();
	consume_thr2.join();
	consume_thr3.join();
	consume_thr4.join();
	consume_thr5.join();

	return 0;
}
```

## 多生产者单消费者

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

static const int kItemRepositorySize = 10;
static const int kItemsToProduce = 1000;

struct ItemRepository
{
	int item_buffer[kItemRepositorySize];
	size_t consume_position = 0;
	size_t produce_position = 0;
	size_t item_count = 0;
	std::mutex mtx;
	std::mutex item_count_mtx;
	std::condition_variable not_full;
	std::condition_variable not_empty;
};

ItemRepository g_item_repo;


void produce_task()
{
	while (true)
	{

		{
			std::lock_guard<std::mutex> count_lock(g_item_repo.item_count_mtx);
			if (g_item_repo.item_count < kItemsToProduce)
			{
				std::unique_lock<std::mutex> lock(g_item_repo.mtx);
				g_item_repo.not_full.wait(lock,
					[] {return (g_item_repo.produce_position + 1) % kItemRepositorySize != g_item_repo.consume_position; }
				);

				g_item_repo.item_count += 1;
				int data = g_item_repo.item_count;
				g_item_repo.item_buffer[g_item_repo.produce_position] = data;
				g_item_repo.produce_position++;
				if (g_item_repo.produce_position == kItemRepositorySize)
					g_item_repo.produce_position = 0;

				std::cout << std::this_thread::get_id() << " produce the: " << data << " item" << std::endl;
				g_item_repo.not_empty.notify_one();
			}
			else
				break;

		}
	}
}

void consume_task()
{
	int consume_count = 0;
	while (true)
	{
		{
			if (consume_count < kItemsToProduce)
			{
				std::unique_lock<std::mutex> lock(g_item_repo.mtx);
				g_item_repo.not_empty.wait(lock, []() {return g_item_repo.produce_position != g_item_repo.consume_position; }
				);

				int data = g_item_repo.item_buffer[g_item_repo.consume_position];
				g_item_repo.consume_position++;
				if (g_item_repo.consume_position == kItemRepositorySize)
					g_item_repo.consume_position = 0;

				++consume_count;
				std::cout << "consume the: " << data << " item" << std::endl;
				g_item_repo.not_full.notify_one();
			}
			else
				break;
		}
	}
}

int main()
{
	std::thread produce_thr1(produce_task);
	std::thread produce_thr2(produce_task);
	std::thread produce_thr3(produce_task);
	std::thread produce_thr4(produce_task);
	std::thread produce_thr5(produce_task);
	std::thread consume_thr(consume_task);

	produce_thr1.join();
	produce_thr2.join();
	produce_thr3.join();
	produce_thr4.join();
	produce_thr5.join();
	consume_thr.join();

	return 0;
}
```

## 多生产者多消费者

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

static const int kItemRepositorySize = 10;
static const int kItemsToProduce = 1000;

struct ItemRepository
{
	int item_buffer[kItemRepositorySize];
	size_t consume_position = 0;
	size_t produce_position = 0;
	size_t produce_item_count = 0;
	size_t consume_item_count = 0;
	std::mutex mtx;
	std::mutex produce_item_count_mtx;
	std::mutex consume_item_count_mtx;
	std::condition_variable not_full;
	std::condition_variable not_empty;
};

ItemRepository g_item_repo;


void produce_task()
{
	while (true)
	{
		{
			std::lock_guard<std::mutex> count_lock(g_item_repo.produce_item_count_mtx);
			if (g_item_repo.produce_item_count < kItemsToProduce)
			{
				std::unique_lock<std::mutex> lock(g_item_repo.mtx);
				g_item_repo.not_full.wait(lock,
					[] {return (g_item_repo.produce_position + 1) % kItemRepositorySize != g_item_repo.consume_position; }
				);

				g_item_repo.produce_item_count += 1;
				int data = g_item_repo.produce_item_count;
				g_item_repo.item_buffer[g_item_repo.produce_position] = data;
				g_item_repo.produce_position++;
				if (g_item_repo.produce_position == kItemRepositorySize)
					g_item_repo.produce_position = 0;

				std::cout << std::this_thread::get_id() << " produce the: " << data << " item" << std::endl;
				g_item_repo.not_empty.notify_one();
			}
			else
				break;
		}

	}
}

void consume_task()
{
	while (true)
	{
		{
			std::lock_guard<std::mutex> count_lock(g_item_repo.consume_item_count_mtx);
			if (g_item_repo.consume_item_count < kItemsToProduce)
			{
				std::unique_lock<std::mutex> lock(g_item_repo.mtx);
				g_item_repo.not_empty.wait(lock, []() {return g_item_repo.produce_position != g_item_repo.consume_position; }
				);

				int data = g_item_repo.item_buffer[g_item_repo.consume_position];
				g_item_repo.consume_position++;
				if (g_item_repo.consume_position == kItemRepositorySize)
					g_item_repo.consume_position = 0;

				++g_item_repo.consume_item_count;
				std::cout << std::this_thread::get_id() << " consume the: " << data << " item" << std::endl;
				g_item_repo.not_full.notify_one();
			}
			else
				break;
		}
	}
}

int main()
{
	std::thread produce_thr1(produce_task);
	std::thread produce_thr2(produce_task);
	std::thread consume_thr1(consume_task);
	std::thread consume_th2(consume_task);
	
	produce_thr1.join();
	produce_thr2.join();
	consume_thr1.join();
	consume_th2.join();

	return 0;
}
```

# 半同步半异步线程池

## 同步队列

```
#include <iostream>
#include <memory>
#include <mutex>
#include <condition_variable>
#include <list>


template <typename T>
class SyncQueue
{
public:
	SyncQueue(size_t n) noexcept : capacity_(n) {}

	void push(T&& t)
	{
		add(t);
	}

	void push(const T& t)
	{
		add(t);
	}
	
	void pop(std::list<T> & list)
	{
		std::unique_lock<std::mutex> lock(mtx_);
		cv_not_empty_.wait(lock, [this] {return need_stop_ || not_empty(); });
		if (need_stop_)
			return;
		list = std::move(queue_);
		cv_not_full_.notify_one();
	}

	void pop(T & t)
	{
		std::unique_lock<std::mutex> lock(mtx_);
		cv_not_empty_.wait(lock, [this] {return need_stop_ || not_empty(); });
		if (need_stop_)
			return;
		t = std::move(queue_.front());
		queue_.pop_front();
		cv_not_full_.notify_one();
	}

	void stop()
	{
		{
			std::lock_guard<std::mutex> lock(mtx_);
			need_stop_ = true;
		}

		cv_not_empty_.notify_all();
		cv_not_full_.notify_all();
	}

	bool empty() const noexcept
	{
		std::lock_guard<std::mutex> lock(mtx_);
		return queue_.empty();
	}

	bool full() const noexcept
	{
		std::lock_guard<std::mutex> lock(mtx_);
		return queue_.size() >= capacity_;
	}

	size_t size() const noexcept
	{
		std::lock_guard<std::mutex> lock(mtx_);
		return queue_.size();
	}

private:
	bool not_full()
	{
		bool full = queue_.size() >= capacity_;
		//if (full)
		//	std::cout << "queue is full" << std::endl;
		return !full;
	}

	bool not_empty()
	{
		bool empty = queue_.empty();
		//if (empty)
		//	std::cout << "queue is empty" << std::endl;
		return !empty;
	}

	template <typename F>
	void add(F&& f)
	{
		std::unique_lock<std::mutex> lock(mtx_);
		cv_not_full_.wait(lock, [this] {return need_stop_ || not_full(); });
		if (need_stop_)
			return;

		queue_.push_back(std::forward<F>(f));
		cv_not_empty_.notify_one();
	}

private:
	size_t capacity_ = 0;
	bool need_stop_ = false;
	std::list<T> queue_;
	std::condition_variable cv_not_full_;
	std::condition_variable cv_not_empty_;
	std::mutex mtx_;
};
```

## 线程池

````
#include <list>
#include <thread>
#include <functional>
#include <memory>
#include <atomic>
#include <algorithm>

const int kMaxTaskNum = 20;

class ThreadPool
{
public:
	using Task = std::function<void()>;

	ThreadPool(int nums = std::thread::hardware_concurrency()) noexcept: queue_(kMaxTaskNum)
	{
		start(nums);
	}

	~ThreadPool()
	{
		stop();
	}

	void stop()
	{
		std::call_once(flag_, [this] {stop_threads(); });
	}

	void add_task(Task&& task)
	{
		queue_.push(std::forward<Task>(task));
	}

	void add_task(const Task& task)
	{
		queue_.push(task);
	}

private:
	void start(int nums)
	{
		running_.store(true);
		for (int i = 0; i < nums; i++)
		{
			threads_.emplace_back(std::make_shared<std::thread>(&ThreadPool::run_in_thread,this));
		}
	}

	void run_in_thread()
	{
		while (running_)
		{
			std::list<Task> tasks;
			queue_.pop(tasks);
			{
				for (const auto& task : tasks)
				{
					if (!running_)
						return;
					task();
				}
			}
		}
	}

	void stop_threads()
	{
		queue_.stop();
		running_ = false;

		std::for_each(threads_.begin(), threads_.end(),std::mem_fn(&std::thread::join));
		threads_.clear();
	}

private:
	SyncQueue<Task> queue_;
	std::atomic_bool running_;
	std::once_flag flag_;
	std::list<std::shared_ptr<std::thread>> threads_;
};



int main()
{
	ThreadPool pool;

	std::thread thr1([&pool] {
		for (int i = 0; i < 10; i++)
		{
			auto thr_id = std::this_thread::get_id();
			pool.add_task([&thr_id] {
				std::cout << "thread 1 id:" << thr_id << std::endl;
			});
		}
	});

	std::thread thr2([&pool] {
		for (int i = 0; i < 10; i++)
		{
			auto thr_id = std::this_thread::get_id();
			pool.add_task([&thr_id] {
				std::cout << "thread 2 id:" << thr_id << std::endl;
			});
		}
	});

	std::this_thread::sleep_for(std::chrono::milliseconds(200));
	getchar();

	pool.stop();
	thr1.join();
	thr2.join();

	return 0;
}
````

