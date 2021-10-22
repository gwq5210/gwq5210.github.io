title: STL容器概述
date: 2021-10-22 11:59:54
tags: [STL, 容器库]
categories: [编程, STL]
---

# STL容器概述

容器被设计来存储数据，可以使程序员简单的访问常见的数据结构

每种不同的容器底层使用不同的数据结构，并拥有不同的特征，可在特定场景提供高效的数据存储和访问的方式

每种容器提供[迭代器](/2021/10/22/设计模式之迭代器模式/)来访问存储在其中的元素

容器一般分为以下几种类别

# 顺序容器

顺序容器实现能按顺序访问的数据结构

- array：静态连续数组
- vector：动态连续数组
- deque：双端队列
- forward_list：单链表
- list：双链表

# 关联容器

关联容器实现能快速查找（ O(log n) 复杂度）的数据结构。

- set：有序集合，键唯一
- map：键值对集合，键唯一
- multiset：键集合，键不唯一
- multimap：键值对集合，键不唯一

# 无序关联容器

无序关联容器提供能快速查找（均摊 O(1) ，最坏情况 O(n) 的复杂度）的无序（哈希）数据结构。

- unordered_set：无序集合，键唯一
- unordered_map：无序键值对集合，键唯一
- unordered_multiset：无序集合，键不唯一
- unordered_multimap：无序键值对集合，键不唯一

# 容器适配器

容器适配器提供顺序容器的不同接口

- stack：栈
- queue：队列
- priority_queue：优先队列

# 参考

- [容器库](https://zh.cppreference.com/w/cpp/container)
- [STL源码剖析](https://item.jd.com/11821611.html)
