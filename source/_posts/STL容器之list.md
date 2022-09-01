title: STL容器之list
date: 2021-10-26 00:15:25
tags: [STL, 容器库, 链表]
categories: [编程, STL]
---

# list概述

不同于vector，list的元素在逻辑上相邻，但是在物理内存上是不相邻的，正因为如此，它们之间通过前后指针进行关联

同时list也不会浪费空间(实际上会多占用前后指针的空间)，有多少个元素就占用多少空间

list的元素插入是常数级别，但无法想vector一样实现随机元素访问

在list内添加，删除或移动元素不会导致已有迭代器失效或引用非法，对应元素被删除时该迭代器失效或引用非法

# list的实现

## list节点

list本身和list的节点是不同的结构，常见的list节点如下，包含前后指针

```cpp
// 要实现一个通用的链表需要使用模板类
template <class T>
struct ListNode {
  ListNode* prev;
  ListNode* next;
  T data;
};
```

## list结构

在STL的list实现中，list不仅是一个双向链表，还是一个环形双向链表

实现list的一个技巧是添加一个虚拟的空白节点，让list的end迭代器指向该空白节点，既符合STL迭代器前闭后开的要求，也方便了编程的实现

![list](https://gwq5210.github.io/images/list.png)

代码如下

```cpp
template <typename T>
class List {
 public:
  List() {
    dummy_head_ = new ListHead(); // 分配一个空间，不设置值
    dummy_head_->prev = dummy_head_; // 头尾都指向自己
    dummy_head_->next = dummy_head_;
  }

  iterator begin() { return dummy_head_->next; } // begin是next节点
  iterator end() { return dummy_head_; } // end是虚拟节点自身
 private:
  ListHead* dummy_head_;
};
```

## list的迭代器

由于list的特殊实现方式，因此我们需要实现一个list的迭代器，允许用户遍历list中保存的元素

迭代器的表现类似指针，list的迭代器是双向迭代器，支持++和--操作，同时要实现*和->运算符

典型的实现如下

```cpp
template <class T>
struct ListIteratorBase {
  using iterator_category = std::bidirectional_iterator_tag;
  using Self = ListIteratorBase;
  ListNode<T>* node;
  ListIteratorBase() : node(nullptr) {}
  ListIteratorBase(ListNode<T>* n) : node(n) {}

  // ++it;
  Self& operator++() {
    node = node->next;
    return *this;
  }
  // it++;
  Self operator++(int) {
    ListNode<T>* ret = node;
    node = node->next;
    return Self(ret);
  }
  // --it;
  Self& operator--() {
    node = node->prev;
    return *this;
  }
  // it++;
  Self operator--(int) {
    ListNode<T>* ret = node;
    node = node->prev;
    return Self(ret);
  }

  reference operator*() const { return node->data; }
  // *(*this) *this即为迭代器本身，则第一个*为调用迭代器的operator*，返回数据的引用
  pointer operator->() const { return std::pointer_traits<pointer>::pointer_to(**this); }

  bool operator==(const Self& other) const { return node == other.node; }
  bool operator!=(const Self& other) const { return node != other.node; }
};
```

# 参考

- [容器库](https://zh.cppreference.com/w/cpp/container/list)
- [STL源码剖析](https://item.jd.com/11821611.html)
