title: STL容器之queue
date: 2021-10-26 00:15:39
tags: [STL, 容器库, 队列]
categories: [编程, STL]
---

# queue概述

queue是一种先进先出(First In First Out, FIFO)，它有两个入口，但只允许在一个入口(队尾)添加元素，在另一个入口(队头)获取和删除元素

queue不允许有遍历行为，所以不提供迭代器

![queue](https://gwq5210.com/images/queue.png)

通常使用deque来作为queue的底层实现，这种修改接口的方法被称为[适配器模式](/2021/10/26/设计模式之适配器模式/)

因此queue往往不归类为container（容器），而被归类为container adapter（容器适配器）

STL的实现中，可以通过模板参数修改queue的底层容器

# queue实现

queue的实现非常简单，完全调用底层容器的接口

```cpp
template <typename T, typename Container = Deque<T>>
class Queue {
 public:
  using container_type = Container;
  using value_type = typename Container::value_type;
  using size_type = typename Container::size_type;
  using reference = typename Container::reference;
  using const_reference = typename Container::const_reference;

  template <typename T_, typename Container_>
  friend bool operator==(const Queue<T_, Container_>& lhs, const Queue<T_, Container_>& rhs);
  template <typename T_, typename Container_>
  friend bool operator<(const Queue<T_, Container_>& lhs, const Queue<T_, Container_>& rhs);

  Queue() = default;
  explicit Queue(const Container& c) : c_(c) {}
  explicit Queue(Container&& c) : c_(std::move(c)) {}
  template <typename InputIt, typename Category = typename std::iterator_traits<InputIt>::iterator_category>
  Queue(InputIt first, InputIt last) : c_(first, last) {}
  ~Queue() = default;

  // Element access
  reference front() { return c_.front(); }
  const_reference front() const { return c_.front(); }
  reference back() { return c_.back(); }
  const_reference back() const { return c_.back(); }

  // Capacity
  bool empty() const { return c_.empty(); }
  size_type size() const { return c_.size(); }

  // Modifiers
  void push(const T& v) { c_.push_back(v); }
  void push(T&& v) { c_.push_back(std::move(v)); }
  template <typename... Args>
  void emplace(Args&&... args) {
    c_.emplace_back(std::forward<Args>(args)...);
  }
  template <typename InputIt, typename Category = typename std::iterator_traits<InputIt>::iterator_category>
  void push(InputIt first, InputIt last) {
    for (; first != last; ++first) {
      emplace(*first);
    }
  }
  void pop() { c_.pop_front(); }
  void swap(Queue& other) { std::swap(c_, other.c_); }

 private:
  Container c_;
};  // class Queue
```

# 参考

- [容器库](https://zh.cppreference.com/w/cpp/container/queue)
- [STL源码剖析](https://item.jd.com/11821611.html)
