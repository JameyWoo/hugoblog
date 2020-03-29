---
title: "实现c语言的memcpy函数"
date: 2020-03-22T14:07:21+08:00
draft: false
---

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