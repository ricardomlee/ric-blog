---
description: ''

pubDatetime: 2019-08-29 13:20:31+08:00
tags:
- 笔记
title: VS Code 部署 WSL+GCC 编译环境
---

## 安装 WSL
- 管理员打开PowerShell命令行运行以下命令：
`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`
（或打开控制面板\程序\启用或关闭Windows功能——勾选适用于Linux的Windows子系统）
- 重启
- 打开应用商店，搜索WSL，选择Ubuntu（或其他版本）安装
- 开始菜单中打开Ubuntu，第一次进入设置用户名密码。

## 安装 VS Code
[Download](https://code.visualstudio.com/Download)

## WSL 设置
```shell
#创建工程文件夹
mkdir projects
cd projects
mkdir helloworld

sudo apt-get update
sudo apt-get install build-essential gdb
#检查g++，gdb
whereis g++
whereis gdb 
```

## VS Code 设置
### 创建工程目录，例如`C:\Users\ricar\projects\new`，用vscode打开此目录
PowerShell操作如下
```shell
mkdir projects
cd projects
mkdir new
cd new
code .
```
### 将WSL设置为默认Terminal
`Ctrl+Shift+P`打开命令行，找到 **Terminal: Select Default Shell**，选择WSL

### 设置编译路径
`Ctrl+Shift+P` **C/C++:Edit Configurations (UI)**打开设置页
找到**Compiler path**，输入`/usr/bin/g++`
**IntelliSense mode**设置为 `${default}`或`gcc-x64`.

### 创建 build task
命令行找到**Tasks: Configure Default Build Task**
选择**Create tasks.json file from template**，**Others**
task.json替换成以下内容
```json
 {
     "version": "2.0.0",
     "windows": {
         "options": {
             "shell": {
                 "executable": "bash.exe",
                 "args": ["-c"]
             }
         }
     },
     "tasks": [
         {
             "label": "build hello world on WSL",
             "type": "shell",
             "command": "g++",
             "args": [
                 "-g",
                 "-o",
                 "/home/<第一步中设置的linux用户名>/projects/new/new.out",
                 "new.cpp"
             ],
             "group": {
                 "kind": "build",
                 "isDefault": true
             }
         }
     ]
 }
```

### 设置debug
F5开始调试，Debug > Add Configuration...C++ (GDB/LLDB)
修改launch.json内容
```json
{
"version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "/home/<linux用户名>/projects/new/new.out",
            "args": [""],
            "stopAtEntry": false,
            "cwd": "/home/<linux用户名>/projects/new/",
            "environment": [],
            "externalConsole": true,
            "windows": {
                "MIMode": "gdb",
                "miDebuggerPath": "/usr/bin/gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            },
            "pipeTransport": {
                "pipeCwd": "",
                "pipeProgram": "c:\\windows\\sysnative\\bash.exe",
                "pipeArgs": ["-c"],
                "debuggerPath": "/usr/bin/gdb"
            },
            "sourceFileMap": {
                "/mnt/c": "${env:systemdrive}/",
                "/usr": "C:\\Users\\<path to WSL directory which you will place here later>"
            }
        }
    ]
}
```

创建new.cpp
```c++
 #include <iostream>
 #include <vector>
 #include <string>

 using namespace std;

 int main()
 {

     vector<string> msg {"Hello", "C++", "World", "from", "VS Code!", "and the C++ extension!"};

     for (const string& word : msg)
     {
         cout << word << " ";
     }
     cout << endl;
 }
```
选中vector，右键跳转到定义 Go to definition
复制头文件的路径，提取usr路径放到launch.json中sourceFileMap下，例如：
`C:\\Users\\ricar\\AppData\\Local\\Packages\\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\\LocalState\\rootfs\\usr\\`

### 编译
`Ctrl+Shift+B`

### 调试
`F5`

## 移植
创建workspace, 复制三个 .json 文件到 .vscode文件夹
需要修改的内容：
- tasks.json:
```json
"args": [
                "-g",
                "-o",
                ".out路径",
                ".cpp文件"
            ],
```
- launch.json:
```json
"program": ".out路径"
...
"cwd": "工程目录"
```