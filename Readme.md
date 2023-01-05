# 实现一个简单的操作系统

## 1. 搭建实验环境

### 工具安装

```
// Mac下安装
$ brew install x86_64-elf-gcc
$ brew install x86_64-elf-gdb
$ brew install cmake
brew install qemu


// Liunx下安装
$ sudo apt-get install gcc-i686-linux-gnu
$ sudo apt-get install gdb
$ sudo apt-get install cmake
$ sudo apt-get install qemu-system-x86
```

### 安装vscode插件

```
C/C++ Extension Pack  // C/C++扩展包
C/C++ // 必备
x86 and x86_64 Assembly // x86汇编语言支持
LinkerScript //提供链接脚本的语法高亮
Hex Editor // 十六进制编辑器
Makefile Tools // make文件工具
```

### 设置vscode任何文件都可以打断点

![](./images/breakPoint.png)

## 2. 代码分析

```
原始文件

--- scripts
--- source 
--- Makefile 



```