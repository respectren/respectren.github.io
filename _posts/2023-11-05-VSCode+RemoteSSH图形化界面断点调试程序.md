---
layout:     post
title:      VS Code + Remote SSH图形化界面断点调试程序
subtitle:   
date:       2023-11-05
author:     Respect Ren
header-style: text
catalog: true
tags:
    - VS Code
    - Remote SSh
    - debug
---

# 问题背景
日常开发过程中的配置是：Windows打开VS Code，编译构建环境在Linux上。VS Code推出的Remote SSH极大的解决了Windows和Linux同步代码等问题，代码可以全部维护在Linux上，Windows上的VS Code只作为一个前端。

同时，借助gdb调试程序时，使用命令打断点不够方便和直观，使用图形化界面效率倍增。

# 操作步骤
### 1. Windows上安装VS Code
### 2. 在Windows机器上安装Remote SSH插件，并SSH连接到远程Linux机器。
### 3. 在Linux机器上安装插件C/C++
打开VS Code的插件市场，搜索`C/C++`，找到Microsoft官方的C/C++插件，将其安装到远程Linux机器上，或直接通过下面链接（https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools）来安装。
### 4. 新建一个文件夹，如helloworld。在helloworld文件夹内新建一个main.c
参考代码：
```
// main.c
#include <stdio.h>

int main()
{
    printf("hello, world\n");

    int a = 1;
    int b = 2;
    int c = a + b;
    printf("c=%d\n", c);
    
    return 0;
}
```

### 5. 配置task
打开main.c，点击菜单栏的Terminal -> Configure Tasks，会自动在工作区目录的.vscode下创建一个tasks.json。
下面我将标出可能需要修改之处。

```
// tasks.json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "C/C++: gcc build active file",		// 每个任务的label，可以改成自定义的任务名称
			"command": "/usr/bin/gcc",
			"args": [
				"-fdiagnostics-color=always",
				"-g",
				"${file}",								    // 需要编译的源文件，${file}表示当前打开的文件的路径，运行此任务时必须打开想要编译的源文件，即active
				"-o",
				"${fileDirname}/${fileBasenameNoExtension}"	// 编译输出文件，默认配置表示在生成一个与源文件同级的可执行文件，不带扩展名
			],
			"options": {
				"cwd": "${fileDirname}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": "build",
			"detail": "compiler: /usr/bin/gcc"
		}	
	]
}
```

如果你的工程构建的输入输出很明确，并且运行任务时希望与当前打开的文件无关，可参考下面借助workspaceFolder变量的任务
```
// tasks.json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "Build Hello World",					// 每个任务的label，可以改成自定义的任务名称: Build Hello World
			"command": "/usr/bin/gcc",
			"args": [
				"-fdiagnostics-color=always",
				"-g",
				"${workspaceFolder}/main.c",				// 编译源文件为当前工作区根目录下的main.c
				"-o",
				"${workspaceFolder}/hello.out"				// 编译输出的可执行文件放在当前工作区根目录下，命名为hello.out
			],
			"problemMatcher": [
				"$gcc"
			],
			"group": "build",
			"detail": "compiler: /usr/bin/gcc"
		},		
	]
}
```

配置好tasks.json后，点击菜单栏Terminal -> Run Task，找到tasks.json中定义的任务并点击。
可以看到下方Terminal中有类似`Build finished successfully.`的字样，即表示运行编译任务成功。

### 6. 配置调试用的launch.json
vscode的launch.json用于配置调试相关的参数。
打开main.c，使其active，点击vscode左侧的`Run and Debug`，点击`Show all automatic debug configurations`，点击`Add Configuration`，选择`C/C++: (gdb) Launch`，即会给出一个默认模板。
下面我将标出需要修改之处。
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/hello.out",      // enter program name, for example ${workspaceFolder}/a.out
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "preLaunchTask": "Build Hello World",           // 在debug前需要执行编译任务，后面写完代码直接F5即可以重新编译+调试了
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

### 7. 愉快地在vscode界面打断点调试啦
配置好tasks.json和launch.json之后，就可以直接像其他商用IDE一样编译+调试了。F5 GO~