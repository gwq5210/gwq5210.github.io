title: STL容器之priority_queue
date: 2021-10-26 00:15:50
tags: [STL, 容器库, 优先队列]
categories: [编程, STL]
---

# 优先队列

与stack和queue相同，priority_queue是容器适配器，可提供常数时间获取最大元素，对数复杂度的插入和删除元素

可以通过模板参数Compare更改顺序，默认使用std::less<T>进行比较，返回最大元素

priority_queue的底层用堆实现

# 二叉堆

堆并不属于容器组件，是一个数据结构，堆实际上是一个完全二叉树（整个二叉树除了最底层的叶节点外，是填满的），并且每个节点的键值都大于等于/小于等于其父节点的值

![heap](https://gwq5210.github.io/images/heap.png)

由于二叉树的性质，我们可以使用数组来表示一个二叉堆(假设下标从0开始)

- 下标为i的节点的子节点为`2*i+1`和`2*i+2`
- 下标为i的节点的父节点为`(i-1)/2`

## 插入操作

在已有的堆中插入元素，一般在堆最下层最右边的叶子之后插入一个元素（或者新增一层）

插入新元素之后，可能不满足堆的性质，这时候需要进行向上调整（sift_up）的操作

- 如果这个节点的值大于父节点的值，则与父节点进行交换，重复次过程直到满足堆的条件或者到达根

向上调整的时间复杂度是O(logn)

```cpp
void sift_up(int x) {
    for (int i = x; i > 0 && arr[(i - 1) / 2] < arr[i]; i = (i - 1) / 2) {
        std::swap(arr[i], arr[(i - 1) / 2]);
    }
}
```

## 删除操作

删除操作是删除堆的最大元素，一般将最大元素即根节点与最后一个节点交换，再将新的根节点进行向下调整（sift_down）的操作

- 在该节点的儿子中选择较大节点与该节点进行交换，重复此操作直到底层或已满足堆的条件

向下调整的时间复杂度是O(logn)

```cpp
void sift_down(int x) {
    for (int i = x; i < n / 2; i = 2 * x + 1) {
        int j = 2 * x + 1;
        if (j + 1 < n && arr[j + 1] > arr[j]) {
            ++j;
        }
        if (arr[i] >= arr[j]) {
            break;
        }
        std::swap(arr[i], arr[j]);
    }
}
```

## 构建堆

### 向上调整建堆

即相当于每次在堆的结尾添加一个元素，比较好理解

复杂度为log1+log2+...+logn=O(nlogn)

```cpp
for (int i = 0; i < n; ++i) {
    sift_up(i);
}
```

### 向下调整建堆

从叶子开始建堆，逐个向下调整，相当于每次"合并"两个已经调整好的堆

复杂度为O(n)，每个节点最多向下调整一次，只需要从最后一个非叶子结点进行调整

```cpp
for (int i = n / 2 - 1; i >= 0; --i) {
    sift_down(i);
}
```

# 堆操作算法

STL提供了以下几个算法，可以方便的实现优先队列

## push_heap

将位置`last-1`的元素插入到`[first, last - 1)`定义的堆中

```cpp
template <typename RandomIt, typename Compare>
void push_heap(RandomIt first, RandomIt last, Compare comp) {
  if (first != last) {
    gtl::sift_up(first, last, gtl::distance(first, last) - 1, comp);
  }
}

template <typename RandomIt>
void push_heap(RandomIt first, RandomIt last) {
  return gtl::push_heap(first, last, std::less<typename std::iterator_traits<RandomIt>::value_type>());
}
```

## pop_heap

交换在位置`first`的值和位置`last-1`的值，并将范围`[first, last-1)`调整为堆

这达到将范围`[first, last)`所定义的堆移除首个元素的效果

```cpp
template <typename RandomIt, typename Compare>
void pop_heap(RandomIt first, RandomIt last, Compare comp) {
  if (first != last) {
    --last;
    gtl::iter_swap(first, last);
    gtl::sift_down(first, last, 0, comp);
  }
}

template <typename RandomIt>
void pop_heap(RandomIt first, RandomIt last) {
  return gtl::pop_heap(first, last, std::less<typename std::iterator_traits<RandomIt>::value_type>());
}
```

## sort_heap

将最大堆`[first, last)`转换为为以升序排序的序列。

```cpp
template <typename RandomIt, typename Compare>
void sort_heap(RandomIt first, RandomIt last, Compare comp) {
  for (; first != last; --last) {
    gtl::pop_heap(first, last, comp);
  }
}

template <typename RandomIt>
void sort_heap(RandomIt first, RandomIt last) {
  return gtl::sort_heap(first, last, std::less<typename std::iterator_traits<RandomIt>::value_type>());
}
```

## make_heap

在范围`[first, last)`中构造最大堆

```cpp
template <typename RandomIt, typename Compare>
void make_heap(RandomIt first, RandomIt last, Compare comp) {
  if (first != last) {
    for (auto i = gtl::distance(first, last) / 2 - 1; i >= 0; --i) {
      gtl::sift_down(first, last, i, comp);
    }
  }
}

template <typename RandomIt>
void make_heap(RandomIt first, RandomIt last) {
  return gtl::make_heap(first, last, std::less<typename std::iterator_traits<RandomIt>::value_type>());
}
```

## is_heap_until

检验范围`[first, last)`并寻找始于`first`且为最大堆的最大范围

```cpp
template <typename RandomIt, typename Compare>
RandomIt is_heap_until(RandomIt first, RandomIt last, Compare comp) {
  auto n = gtl::distance(first, last);
  typename std::iterator_traits<RandomIt>::difference_type i = 0;
  if (n <= 1) {
    return last;
  }
  auto it = first;
  auto left_it = first + 1;
  for (; i < n / 2; ++i, ++it, ++left_it) {
    if (comp(*it, *left_it)) {
      return left_it;
    }
    ++left_it;
    if (left_it < last && comp(*it, *left_it)) {
      return left_it;
    }
  }

  return last;
}

template <typename RandomIt>
RandomIt is_heap_until(RandomIt first, RandomIt last) {
  return gtl::is_heap_until(first, last, std::less<typename std::iterator_traits<RandomIt>::value_type>());
}
```

## is_heap

检查范围 [first, last) 中的元素是否为最大堆。

```cpp
template <typename RandomIt, typename Compare>
bool is_heap(RandomIt first, RandomIt last, Compare comp) {
  return gtl::is_heap_until(first, last, comp) == last;
}

template <typename RandomIt>
bool is_heap(RandomIt first, RandomIt last) {
  return gtl::is_heap(first, last, std::less<typename std::iterator_traits<RandomIt>::value_type>());
}
```

# 优先队列的实现

有了堆的操作算法，很容易通过这些算法实现优先队列

具体代码如下

```cpp
template <typename T, typename Compare = std::less<T>, typename Container = Vector<T>>
class PriorityQueue {
 public:
  using container_type = Container;
  using value_type = typename Container::value_type;
  using size_type = typename Container::size_type;
  using reference = typename Container::reference;
  using const_reference = typename Container::const_reference;

  template <typename T_, typename Compare_, typename Container_>
  friend bool operator==(const PriorityQueue<T_, Compare_, Container_>& lhs,
                         const PriorityQueue<T_, Compare_, Container_>& rhs);
  template <typename T_, typename Compare_, typename Container_>
  friend bool operator<(const PriorityQueue<T_, Compare_, Container_>& lhs,
                        const PriorityQueue<T_, Compare_, Container_>& rhs);

  PriorityQueue() = default;
  explicit PriorityQueue(const Container& c) : c_(c) { gtl::make_heap(c_.begin(), c_.end(), compare_); }
  explicit PriorityQueue(Container&& c) : c_(std::move(c)) { gtl::make_heap(c_.begin(), c_.end(), compare_); }
  template <typename InputIt, typename Category = typename std::iterator_traits<InputIt>::iterator_category>
  PriorityQueue(InputIt first, InputIt last) : c_(first, last) {
    gtl::make_heap(c_.begin(), c_.end(), compare_);
  }

  // Element access
  const_reference top() const { return c_.front(); }

  // Capacity
  bool empty() const { return c_.empty(); }
  size_type size() const { return c_.size(); }

  // Modifiers
  void push(const T& v) {
    c_.push_back(v);
    gtl::push_heap(c_.begin(), c_.end(), compare_);
  }
  void push(T&& v) {
    c_.push_back(std::move(v));
    gtl::push_heap(c_.begin(), c_.end(), compare_);
  }
  template <typename... Args>
  void emplace(Args&&... args) {
    c_.emplace_back(std::forward<Args>(args)...);
    gtl::push_heap(c_.begin(), c_.end(), compare_);
  }
  template <typename InputIt, typename Category = typename std::iterator_traits<InputIt>::iterator_category>
  void push(InputIt first, InputIt last) {
    for (; first != last; ++first) {
      emplace(*first);
    }
  }
  void pop() {
    gtl::pop_heap(c_.begin(), c_.end());
    c_.pop_back();
  }
  void swap(PriorityQueue& other) { std::swap(c_, other.c_); }

 private:
  Container c_;
  Compare compare_;
};  // class PriorityQueue
```

# 参考

- [容器库](https://zh.cppreference.com/w/cpp/container/priority_queue)
- [算法库](https://zh.cppreference.com/w/cpp/algorithm)
- [STL源码剖析](https://item.jd.com/11821611.html)
- [二叉堆](https://oi-wiki.org/ds/binary-heap/)
