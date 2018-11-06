---
title: C语言中的内存布局
date: 2018-11-06 22:03
comments: true
tags:
- C语言
- 内存
categories:
- 计算机系统
---


# C语言中的数据存储

## 内存分区
### C语言运行时内存大致分为四个数据区：常量区、静态区、堆区、栈区

* 常量区存储了未被作为初始化使用的字符串常量和const修饰的全局变量，特点是只能读不能写，受到操作系统运行时保护，强行修改会导致segmentation fault,生命周期同程序运行过程
* 静态区存储了全部的全局变量，和所有被static修饰的变量（包括全局和局部），其特点是生命周期很长（为一次程序的运行过程）并且只被初始化一次（在编译之后就已完成
* 栈区存储了所有自动存储（不加任何存储类型关键字(static等)修饰或被auto修饰）的局部变量，其特点是生命周期很短，仅仅是该变量所在函数的一次调用过程。运行时有操作系统分配并在函数结束后回收
* 堆区是由操作系统负责维护的大片内存池，使用时需手动申请（调用malloc家族函数进行动态内存分配），但使用完毕后需手动释放，否则会造成严重的内存泄漏，直到该进程退出后才会被操作系统回收

### 哪些数据将会放入常量区
* 字符串常量，如 char* p  = "Hello World",编译器会将字符串“Hello World”放入常量区,ELF格式下就是.rodata部分，但是当一个字符串被用来为字符数组初始化时则不会被放入常量区而是按四个字节为一组转化为对应的32为整数来对数组进行初始化
* 被const修饰的全局变量，能够存放在常量区的常量要必须是全局变量且被const修饰，如果是const修饰的局部变量是存放在程序运行时栈上的，这是变量不可改变是语法层面上的，我们可以利用指针对其进行改变而不会引起运行时错误，但是如果改变常量区里的变量值会引发段错误被操作系统终止进程

### 静态区的特征
静态区是一个抽象笼统的概念，在实际的Linux/C的可执行程序中并没有静态区这个区域，具体来讲它主要由两个段组成：.data段和.bss段。其中.data段就是程序的数据段，在采用段式内存管理的架构中，数据段（data segment）通常是指用来存放程序中已初始化且不为0的全局变量或静态变量的一块内存区域。相反，BSS（Block Started by Symbol）通常是指用来存放程序中未初始化的或初始化为0的全局变量或静态变量的一块内存区域。.data段在程序编译期间其大小及数据被确定，而.bss段则没有直接分配空间而是由编译器在.data段之后为其预留空间，在程序装载进内存时被正式分配。尽管静态区由两个不同的段组成，但是在程序链接并装载进内存之后这两段不做区分。

静态区的变量拥有以下特征：

1)	生命周期长，直到该进程结束随进程空间一起被系统回收。

2)	只初始化一次，它的空间数据在编译期间被初始化，逻辑地址在链接期间固定。

那么哪些变量将被放在静态区呢？

* 全局变量，如果一个变量被定义为全局的，那么在同一个程序中，任何函数都可以去访问、存取该变量的数据。基于此，全局变量除拥有静态区变量的全部特征之外还具有作用域广的特点，其作用域在整个程序中（可以由多个源文件组成）全局可见。

* 静态变量，从字面上理解所谓静态变量就是被static关键字修饰的变量，只要被static修饰为静态变量那么都将被编译器分配在静态区，其也就拥有了静态区变量的全部特征。静态变量分两种：全局静态变量和局部静态变量。无论哪种只要被static修饰都将放在静态区，拥有静态区变量的全部特征。其区别仅在于作用域：如果是全局静态变量，那么该变量的作用域被限定只能在本源文件内使用（编译之后该变量的符号将不允许对外链接，但是仍然可以通过指针去间接访问）；如果是局部变量则没有变化（仅限函数内部使用）

* 被static修饰的全局变量具有本文件可见性，但是任然存储在静态区

### 堆区

动态内存分配的变量存储在这里

