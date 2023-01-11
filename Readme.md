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

## 2. 项目运行

```
原始文件

--- scripts // 代码启动调试脚本文件
	--- qemu-debug-liunx.sh liunx下的qemu启动脚本文件
	--- qemu-debug-osx.sh Mac下的qemu启动脚本文件
	--- qemu-debug-win.bat window下的qemu启动脚本文件
--- source  // 我们的代码
	--- start.S 启动第一扇区的代码
	--- os.c 剩余其他扇区的代码
	--- os.h 头文件
--- Makefile  // 编译文件
--- image  // 镜像文件
	--- disk.img 
```

### makeFile文件分析

```c
all: source/os.c source/os.h source/start.S
	$(TOOL_PREFIX)gcc $(CFLAGS) source/start.S // 编译start.S汇编文件，生成start.o文件
	$(TOOL_PREFIX)gcc $(CFLAGS) source/os.c	 // 编译os.cC语言文件,生成os.o文件
	$(TOOL_PREFIX)ld -m elf_i386 -Ttext=0x7c00 start.o os.o -o os.elf // 将生成的.o文件进行链接,生成os.elf文件,并指定生成的代码段的内存地址是0x7c00，运行的时候会放到0x7c00的内存地址上去
	${TOOL_PREFIX}objcopy -O binary os.elf os.bin // 复制成为os.bin文件
	${TOOL_PREFIX}objdump -x -d -S  os.elf > os_dis.txt	 // 头部文件内容输出
	${TOOL_PREFIX}readelf -a  os.elf > os_elf.txt	 // 头部文件内容输出
	dd if=os.bin of=../image/disk.img conv=notrunc // 将os.bin文件写入到disk.img中去，可理解为start.S中的代码会放入到镜像文件的第一个扇区中

```
 ***目的: 将代码段放置到0x7c00处，qemu会从这个内存的这个地方读取***

### 调试启动步骤

1. 首先 终端 -->  运行任务 : 启动我们的qemu任务
2. 运行 ---> 启动调试 : 启动我们的qemu调试工具

> Mac下稍微麻烦一点,要先启动qemu后再开启调试


## 3. 添加引导标志

前置: 1、镜像文件中存放启动代码,第一个扇区 2、代码启动时候从0x7c00处启动

在start.S中，我们使用了.org 0x1fe的伪指令，通知gcc工具链，将0X55、0XAA这两个值放在相对于生成的二进制文件os.bin开头偏移510个字节的地方。然后os.bin会由dd命令写到磁盘映像文件的最开始处。通过这种方式就实现了在磁盘映像文件的第1个扇区最后两个字节添加引导标志的目的。

## 4. 设置寄存器的初始化值

```c
此时代码应该是从0x7c00处开始执行的,$_start代表着内存地址0x7c00
	mov $0, %ax				// 设置代码段
	mov %ax, %ds			// 设置数据段
	mov %ax, %es			// 设置数据段
	mov %ax, %ss			// 设置栈段
	mov $_start, %esp		// 设置栈的起始地址

// 这里会将代码段寄存器、数据段寄存器、栈段寄存器设置为0,即便不设置，初始化的值也是0，栈顶是0x7c00往上，栈是从高地址向低地址增长
```
此时，计算机应该还是实模式，实模式下访问的是真实的内存地址,大概的访问范围是0 - 1M , 2的20次方 === 1M ,寻址范围是0x00000 - 0xFFFFF,为啥不是2的16次方呢？主要是最早的8086的寄存器都是16位的，后面内存扩充成1M，16位的寄存器无法访问到全部，就采用了段寄存器 << 4  + 偏移(16)位的组合来形成20位的地址,实现对1MB内存的访问

## 5. 加载自己的剩余部分

```c
read_self_all:
	mov $_start_32, %bx // 读取到的内存地址
	mov $0x2, %cx // ch:磁道号, cl:起始扇区
	mov $0x0240, %ax // ah:0x42读磁盘命令,al=0x40 64个扇区,多读一些,32kb
	mov %0x80, %dx // dh:磁头号 ,dl驱动器号0x80 磁盘1
	int $0x0013 // 调用中断,读取磁盘信息
	jc read_self_all // 读取失败则重复

第一个扇区           第2到64个扇区
   ↓                     ↓
---------- | -----------------------|
  0x7c00   |       0x7E00           |
---------- | -----------------------|		
```
	 
    







