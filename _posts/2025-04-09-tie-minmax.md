---
title: std::tieとstd::minmaxを使う際の注意点
date: 2025-4-9 22:29:00 0900
categories: [C++]
tags: [STL]
math : true
---

```cpp
#include <bits/stdc++.h>
int main() {
    int a = 2, b = 1;
    std::tie(a, b) = std::minmax(a, b);
    std::cout << a << " " << b << std::endl;
}
```
このコードを実行すると
```
1 1
```
が出力される。```std::minmax```が参照を返すためだ。
```cpp
#include <bits/stdc++.h>
int main() {
    int a = 2, b = 1;
    std::tie(a, b) = std::minmax({a, b});
    std::cout << a << " " << b << std::endl;
}
```
などと修正すると
```
1 2
```
が出力される。  
ついついやりがちなので注意

