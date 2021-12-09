# 排序算法

## 快速排序

- 通过一趟排序将要排序的数据分割成独立的两部分：分割点左边都是比它小的数，右边都是比它大的数
- 然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列

```
int partition(std::vector<int>& nums, int left, int right)
{
	int key = nums[left];
	while (left < right)
	{
		while (left < right && nums[right] >= key)
			right--;
		if (left < right)
			nums[left] = nums[right];
		while (left < right && nums[left] <= key)
			left++;
		if (left < right)
			nums[right] = nums[left];
	}
	nums[left] = key;
	return left;
}

void quick_sort(std::vector<int>& nums, int left, int right)
{
	if (left >= right)
		return;

	int pivot = 0;
	pivot = partition(nums, left, right);
	quick_sort(nums, left, pivot - 1);
	quick_sort(nums, pivot + 1, right);
}
```

## 冒泡排序

- 冒泡排序是一种交换排序
- 重复地走访待排序的序列，依次比较两个相邻的元素，如果顺序是错误的就进行交换
- 当遍历一轮时候，不发生交换，可以提前结束

```
void bubble_sort(std::vector<int>& nums)
{
	bool exchange = false;
	for (size_t i = nums.size() - 1; i > 0; --i)
	{
		for (size_t j = 0; j < i; ++j)
		{
			if (nums[j] >= nums[j + 1])
			{
				std::swap(nums[j], nums[j + 1]);
				exchange = true;
			}
		}

		if (!exchange)
			break;
	}
}
```

## 选择排序

重复地走访待排序的序列，选择出最大的或者最小的元素放到合适位置上。

```
void select_sort(std::vector<int>& nums)
{
	for (size_t i = nums.size() - 1; i > 0; --i)
	{
		int pos = i;
		for (size_t j = 0; j < i; ++j)
		{
			if (nums[j] > nums[pos])
				pos = j;
		}
		if(pos != i)
			std::swap(nums[i],nums[pos]);
	}
}
```

