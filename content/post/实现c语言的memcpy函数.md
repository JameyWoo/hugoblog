---
title: "实现c语言的memcpy函数"
date: 2020-03-22T14:07:21+08:00
draft: false

categories: ["操作系统"]
---

# 实现c语言的memcpy函数

C 库函数 void *memcpy(void *str1, const void *str2, size_t n) 从存储区 str2 复制 n 个字符到存储区 str1。

实现这个函数需要注意以下几个点:
1. 传递进入memcpy的指针参数是 `void *`, 是通用的指针类型
2. 首先需要判断指针是否为NULL
3. 对字节进行操作, 因为size指的是字节数. 因此可以将`void *`转化为`char *`进行操作
4. 需要考虑到如果源地址和目标地址有重合的部分


下面是我的实现, main 中的测试包括 int的拷贝, 字符数组的拷贝, 重叠情况的拷贝.

```cpp
#include <iostream>
using namespace std;

void* memcpy(void* dst, void* src, size_t size) {
    if (dst == NULL || src == NULL)
        return NULL;
    char* new_dst = (char*)dst;
    char* new_src = (char*)src;

    if (dst > src && new_dst - new_src <= size) {
        for (int i = size - 1; i >= 0; i--) {
            new_dst[i] = new_src[i];
        }
    } else {
        for (int i = 0; i < size; i++) {
            new_dst[i] = new_src[i];
        }
    }

    return dst;
}

int main() {
    // 整形的拷贝
    int a = 12, b = 2;
    int* c = (int*)memcpy(&b, &a, sizeof(b));
    cout << (*c) << endl;

    // 字符串的拷贝
    char f[] = "abcfajlafd";
    char d[] = "fuck";
    cout << "sizeof d: " << sizeof(d) << endl;
    char* e = (char*)memcpy(f, d, sizeof(d));
    cout << e << endl;

    // ! 考虑地址重叠的情况
    char g[] = "12345";
    char* h  = (char*)memcpy(g + 2, g, sizeof(g));
    cout << h << endl;
}
```