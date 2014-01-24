---
layout: post
title: 我是如何错误使用std::is_sorted的
description: 简单分析了错误使用std::is_sorted的原因
category: blog
tag: ['is_sorted', 'stl']
---


这篇算一个小笔记，很短。

有这样一个简单需求，判断一列数是否严格递增，即已排好序，并且不存在相等的情况。

一开始我是这样写的

```cpp
std::vector<int> val = {1, 2, 3};

std::cout << is_sorted(val.begin(), val.end(), std::less<int>()) << std::endl;
```

结果当然是 `1` ，可是这样写就正确吗？

把 `{1, 2, 3}` 换成 `{1, 1, 3}` 之后，结果仍然是 `1` ,这我就想不通了， `1` 和 `1` 明显不符合小于的关系，可是结果为什么会是这样。

然后又换了 `std::less_equal` 试一试，没想到结果却是正确的，百思不得其解后，翻开 `std::is_sorted` 的源码看看，下面就是gcc使用的头文件中的代码，稍微进行了修改，以便看得更清楚(重点在第16行)。

```cpp
template<typename ForwardIterator, typename Compare>
bool is_sorted(ForwardIterator first, ForwardIterator last, Compare comp)
{
    return std::is_sorted_until(first, last, comp) == last;
}

template<typename ForwardIterator, typename Compare>
ForwardIterator is_sorted_until(ForwardIterator first, ForwardIterator last, Compare comp)
{
    if (first == last) {
        return last;
    }

    ForwardIterator next = first;
    for (++next; next != last; first = next, ++next) {
        if (comp(*next, *first)) {
            return next;
        }
    }
    return next;
}
```

好吧，这下知道了，调用 `comp` 时并不是把前一个跟后一个比较，而是后一个跟前一个比较，才会导致这样的结果，以后使用的时候该注意了。

