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

# 线程池

