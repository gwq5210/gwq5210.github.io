title: STL容器之stack
date: 2021-10-26 00:15:34
tags: [STL, 容器库, 栈]
categories: [编程, STL]
---

# stack概述

stack是一种先进后出(First In Last Out, FILO)的结构，它只有一个入口和出口。

stack不允许有遍历行为，所以不提供迭代器

![stack](https://gwq5210.com/images/stack.png)

stack是一种很简单的数据接口，只允许在栈顶新增，删除，获取元素

通常使用deque来作为stack的底层实现，这种修改接口的方法被称为[适配器模式](/2021/10/26/设计模式之适配器模式/)

因此stack往往不归类为container（容器），而被归类为container adapter（容器适配器）

STL的实现中，可以通过模板参数修改stack的底层容器

# stack实现

stack的实现非常简单，完全调用底层容器的接口

```cpp
template <typename T, typename Container = Deque<T>>
class Stack {
 public:
  using container_type = Container;
  using value_type = typename Container::value_type;
  using size_type = typename Container::size_type;
  using reference = typename Container::reference;
  using const_reference = typename Container::const_reference;

  template <typename T_, typename Container_>
  friend bool operator==(const Stack<T_, Container_>& lhs, const Stack<T_, Container_>& rhs);
  template <typename T_, typename Container_>
  friend bool operator<(const Stack<T_, Container_>& lhs, const Stack<T_, Container_>& rhs);

  Stack() = default;
  explicit Stack(const Container& c) : c_(c) {}
  explicit Stack(Container&& c) : c_(std::move(c)) {}
  template <typename InputIt, typename Category = typename std::iterator_traits<InputIt>::iterator_category>
  Stack(InputIt first, InputIt last) : c_(first, last) {}

  // Element access
  reference top() { return c_.back(); }
  const_reference top() const { return c_.back(); }

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
  void pop() { c_.pop_back(); }
  void swap(Stack& other) { std::swap(c_, other.c_); }

 private:
  Container c_;
};  // class Stack
```

# 参考

- [容器库](https://zh.cppreference.com/w/cpp/container/stack)
- [STL源码剖析](https://item.jd.com/11821611.html)
