title: STL容器之空间配置器
date: 2021-10-26 00:12:46
tags: [STL, 空间配置器]
categories: [STL, 空间配置器]
---

# 空间配置器

从STL使用的角度，我们一般不需要了解空间配置器（allocator）；但是从STL实现的角度，空间配置器就很重要了

allocator为什么叫做空间配置器，不叫内存配置器？因为allocator的实现不仅可以从内存分配空间，也可以从磁盘分配空间

在STL容器的实现中，每个容器都有一个模板参数，可以指定使用的空间配置器

allocator包含以下类型

- value_type：T
- pointer (C++17 中弃用)(C++20 中移除)：T*
- const_pointer (C++17 中弃用)(C++20 中移除)：const T*
- reference (C++17 中弃用)(C++20 中移除)：T&
- const_reference (C++17 中弃用)(C++20 中移除)：const T&
- size_type：std::size_t
- difference_type：std::ptrdiff_t

allocator主要包含以下成员函数

- allocate：分配未初始化的存储
- deallocate：解分配存储
- construct(C++17 中弃用)(C++20 中移除)：在分配的存储构造对象
- destroy (C++17 中弃用)(C++20 中移除)：析构在已分配存储中的对象

在C++的头文件`<memory>`中定义了一些与未初始化内存相关的函数，在STL容器的实现中，起到了很大的作用，如construct_at(C++20)，uninitialized_copy，uninitialized_fill等

# 空间配置器在STL容器中的应用

STL容器的实现思想中，实际上将空间的分配/释放和对象的构造/析构进行了拆分：空间配置器负责空间的分配/释放；`<memory>`中的未初始化内存相关的函数负责对象构造/析构，移动等

能够实现上述拆分，主要由于C++的以下特性

- operator new只进行内存分配，不进行对象构造
- placement new是operator new的一个特殊版本，在已分配的内存上构造对象
- new关键字的作用是：调用operator new，并调用对象的构造函数

以下两种代码实际上是等价的

```cpp
#include <cstdio>

#include <string>
#include <vector>

class Test {
 public:
  Test(const std::string& name): name_(name) {
    printf("construct %s\n", name_.c_str());
  }
  ~Test() {
    printf("destroy %s\n", name_.c_str());
  }

 private:
  std::string name_;
};

int main(int argc, char* argv[]) {
  {
    // 代码1
    auto* t = new Test("new_test");
    delete t;
  }
  {
    // 代码2
    void* p = ::operator new(sizeof(Test));
    auto* t = reinterpret_cast<Test*>(p);
    t->Test::Test("allocator_test");
    // auto* t = new (p) Test("allocator_test"); // placement new
    t->~Test();
    ::operator delete(p);
  }
  return 0;
}
/*
output:
construct new_test
destroy new_test
construct allocator_test
destroy allocator_test
*/
```

# 未初始化内存算法的实现

有了上述基础，可以实现简单的未初始化内存的算法，更多实现参见[gtl_memory.h](https://github.com/gwq5210/gtl/blob/main/gtl/algorithm/gtl_memory.h)

construct_at用到了std::forward的完美转发（C++11），两次强制转换是为了除去cv限定符

```cpp

// 在给定地址 p 创建以参数 args... 初始化的 T 对象
template <typename T, typename... Args>
T* construct_at(T* p, Args&&... args) {
  ::new (const_cast<void*>(static_cast<const volatile void*>(p))) T(std::forward<Args>(args)...);
  return p;
}

/**
 * @brief 销毁指针p指向的对象
 * 若 T 不是数组类型，则调用 p 所指向对象的析构函数，如同用 p->~T()
 * 若 T 是数组类型，则按顺序递归地销毁 *p 的元素，如同通过调用 std::destroy(std::begin(*p), std::end(*p))
 * C++17 version:
 * template<class T> void destroy_at(T* p) { p->~T(); }
 *
 * @param p 指向要被销毁的对象的指针
 */
template <typename T>
void destroy_at(T* p) {
  if constexpr (std::is_array_v<T>) {  // C++ 20
    for (auto& v : *p) {
      gtl::destroy_at(gtl::addressof(v));
    }
  } else {
    p->~T();
  }
}

template <typename InputIt, typename ForwardIt>
ForwardIt uninitialized_copy_range(InputIt first, InputIt last, ForwardIt d_first, std::input_iterator_tag,
                                   std::forward_iterator_tag) {
  for (; first != last; ++first, ++d_first) {
    gtl::construct_at(gtl::addressof(*d_first), *first);
  }
  return d_first;
}

template <typename InputIt, typename SizeType, typename ForwardIt>
ForwardIt uninitialized_copy_range_n(InputIt first, SizeType count, ForwardIt d_first, std::input_iterator_tag,
                                     std::forward_iterator_tag) {
  for (; count > 0; ++first, ++d_first, --count) {
    gtl::construct_at(gtl::addressof(*d_first), *first);
  }
  return d_first;
}

/**
 * @brief 复制来自范围 [first, last) 的元素到始于 d_first 的未初始化内存
 * 若初始化中抛异常，则以未指定顺序销毁已构造的对象
 *
 * @return ForwardIt 指向最后复制的元素后一元素的迭代器
 *
 * TODO: 处理异常的情况
 */
template <typename InputIt, typename ForwardIt>
ForwardIt uninitialized_copy(InputIt first, InputIt last, ForwardIt d_first) {
  using input_iterator_category = typename std::iterator_traits<InputIt>::iterator_category;
  using forward_iterator_category = typename std::iterator_traits<ForwardIt>::iterator_category;
  return gtl::uninitialized_copy_range(first, last, d_first, input_iterator_category(), forward_iterator_category());
}
```

# 参考

- [STL源码剖析](https://item.jd.com/11821611.html)
- [std::allocator](https://zh.cppreference.com/w/cpp/memory/allocator)
