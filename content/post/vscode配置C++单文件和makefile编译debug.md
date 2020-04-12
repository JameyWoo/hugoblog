---
title: "Vscode配置C++单文件和makefile编译debug"
date: 2020-04-02T08:38:01+08:00
draft: false
categories: ["开发工具"]
---





vscode配置C++非常麻烦, 我之前配置了好几次都失败了, 更别说配置debug. 

今天又尝试了以下, 惊奇的配置成功了. 就几个简单的步骤, 以前怎么就配置不成功呢...

值得一提的是, 以前总是会出现没有找到标准库path然后在include那里画波浪线的情况, 这次居然都好了, 估计是版本更新让它更好用了吧. 

分为两种情况, 一种是调试单个文件的. 另一种是调试用make生成的多文件调试. 下面分别介绍.



## 单文件调试

首先创建一个文件夹, 再创建一个test.cpp文件

写入以下内容

```cpp
#include <iostream>
using namespace std;

int main(int args, char **argv) {
    int a;
    a = 3;
    int b;
    cin >> b;
    cout << "b = " << b << endl;

    cout << "args: " << args << endl;
    for (int i = 0; i < args; i++) {
        cout << argv[i] << endl;
    } 
    return 0;
}
```

然后点到调试, 点击`create a launch.json file` 

![image-20200401211306628](/assets/image-20200401211306628.png)

选择 g++, 他就会自动生成一个launch.json. 然后就可以直接设置断点调试啦! 

奇怪我以前也这样试过但没有成功过...

这样还无法输入数据(cin), 我的解决方法是将launch.json中的这个字段设置为true

```
"externalConsole": true,
```

这样它会弹出一个窗口, 我们就可以输入数据了. 

类似的, 有时候我们执行的命令行程序需要有命令行参数, 怎么办呢?

还是修改launch.json中的字段

```
"args": ["arg1", "arg2", "arg3"],
```

下面是我生成的launch.json文件和tasks.json文件.

launch.json文件

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "test",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": ["arg1", "arg2", "arg3"],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "test"
        }
    ]
}
```

值得注意的是, 这里的`name`字段指的是这个配置的名字, 而最下面的`preLaunchTask`字段才是和tasks里的`label`配置一一对应的. 这个非常重要. 

同样的, 在一个launch.json里, 可以添加多个配置, 在debug界面会以下拉菜单选择使用. 这样就可以在一个文件夹里为每个单个的cpp文件添加调试配置啦.

tasks.json文件

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "test",
            "command": "C:\\mingw64\\bin\\g++.exe",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "C:\\mingw64\\bin"
            }
        }
    ],
    "version": "2.0.0"
}
```

这个command命令非常重要, 它是生成exe文件的关键, 也是可以修改的. 我们后面介绍的makefile编译调试就是主要要修改这个command.

值得注意的是, 下面这个`cwd`字段坑了我一段时间.

```
"options": {
	"cwd": "C:\\mingw64\\bin"
}
```

我执行makefile的debug的时候, 有这个字段, 于是make它总说找不到文件, 我添加绝对路径之后就好了. 我删掉它就好了(当然这里不要删).

我猜cwd是`change work directory`的缩写, 执行make的时候, 到这个工作目录下, 当然找不到文件了.



## make调试

支持make调试的不同就是`tasks.json`的不同, 它控制了exe文件的生成.

因此我们需要将command修改

新的tasks.json如下

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "echo",
            "command": "make",
            // "options": {
            //     "cwd": "C:\\mingw64\\bin"
            // }
        }
    ],
    "version": "2.0.0"
}
```

注意注释掉的那几行, 取消注释就错了. 这里我给出是为了提醒. 因为如果修改一个空模板, 不会有它, 而修改g++模板, 会有它. 这个需要注意.

当然, 想要调试, make中必须开启`-g`选项.



## 例子

下面提供一个make调试的例子. 

下面是目录树.

```
├── Makefile
├── add.cpp
├── sub.cpp
├── test.h
└── .vscode
   ├── launch.json
   └── tasks.json
```

Makefile

```makefile
add.exe: 
	g++ add.cpp sub.cpp -g -o add.exe
```

add.cpp

```cpp
#include <iostream>
using namespace std;

#include "test.h"

int add(int a, int b) {
    return a + b;
}

int main() {
    int a = 1;
    printf(" 2 + 3 = %d\n", add(2, 3));
    scanf("%d", &a);
    printf(" 2 - 3 = %d\n", sub(2, 3));
    return 1;
}
```

sub.cpp

```cpp
#include "test.h"

int sub(int a, int b) {
    return a - b;
}
```

test.h

```cpp
#ifndef _TEST_H
#define _TEST_H

int add(int a, int b);
int sub(int a, int b);
#endif
```

launch.json

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "debug-c++",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "echo"
        }
    ]
}
```

tasks.json

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "echo",
            "type": "shell",
            "command": "make"
        }
    ]
}
```

