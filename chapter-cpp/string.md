# C++ 字符串操作

## 创建

使用 `std::to_string` 将 `int` 转化为 `string`。当然，也可以用从 C 中继承来的 `itoa`。

```C++
string s = to_string(i);
```

## 修改

+ 插入：[`string::insert`](http://www.cplusplus.com/reference/string/string/insert/)
+ 删除: [`string::erase`](http://www.cplusplus.com/reference/string/string/erase/)
+ 追加：[`string::append`](http://www.cplusplus.com/reference/string/string/append/) 或 [`string::operator+=`](http://www.cplusplus.com/reference/string/string/operator+=/)，两者基本等价，但 `append` 功能更丰富
+ 交换：`std::swap(s[i], s[j])`
+ 排序：`std::sort(s.begin(), s.end())`

以上操作全为 in-place。

## 遍历

```C++
for (char c : str) {
    // ...
}
```