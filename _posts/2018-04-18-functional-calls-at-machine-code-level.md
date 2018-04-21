---
layout: post
title:  '机器层面的函数调用'
categories: 底层原理
author: CHH
---

* content
{:toc}

![机器代码](https://upload-images.jianshu.io/upload_images/5690299-2732e7556ab769b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




## 一些概念

CPU 发出指令把硬盘程序指令搬到内存，操作系统给程序指令分配内存。然后操作系统会告诉 CPU 「程序入口点」，也就是第一条指令地址。

CPU 从内存出取得的指令一般是这三类：

1. 把数据从内存运到 CPU 寄存器。
2. 对寄存器数据进行运算。
3. 把寄存器数据写入到内存。

- x86 体系机器：每次函数调用都会创建一个帧。

- 帧，内存中一段连续空间。

![帧](https://upload-images.jianshu.io/upload_images/5690299-9ac2704fbd57844b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 多个函数帧在内存里排起来， 就像一个先进后出的栈一样。

- 但这个栈与普通的栈不一样，这个栈的栈底在上面，从上往下生长（从高地址往低地址生长）。

- 4 个字节为单位，4 个字节内从低地址往高地址生长。

- ebp，CPU 特殊寄存器，指向栈中当前函数帧的起始地址。

- esp，CPU 特殊寄存器，指向栈中当前函数帧的尾地址。

![ebp 与 esp](https://upload-images.jianshu.io/upload_images/5690299-5e2a472c198422fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 函数调用详细过程

一、 把 ebp 的值压入栈中。（CPU -> 内存）

这时，地址 800~804 的内容是 1000，esp 指向地址 800 。

二、 把 esp 的值赋给 ebp 。（CPU）

这时，esp、ebp 同时指向地址 800，一个新的函数帧诞生！

函数帧的起始地址是 800，  里边的内容是1000 。

三、把 esp 的值减去 24 。（CPU）

esp 指向地址 776，这是为准备函数实参腾出空间。

减去 24 是为了数据对齐。24 + 4（入栈的 ebp） + 4（返回地址） = 32 。

x86 编程规定函数帧是 16 的整数倍。

四、 把 10 放到 ebp 减去 4 的地址，把 20 放到 ebp 减去8的地址。（CPU -> 内存）

其实就是：int x=10; int y=20 。

五、 把 x 地址放到 esp 指向的地址，把 y 地址放到 esp+4 指向的地址。

其实就是准备实参 &x, &y 。

![准备函数实参](https://upload-images.jianshu.io/upload_images/5690299-7d7d108bbc47d35c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

六、把地址 100 压入栈中。

这是调用函数之前，调用方下一条指令地址（函数返回地址）压入栈。

```c
int sum = add(&x, &y); 
printf("the sum is %d\n",sum);  // 假设这条指令的地址是 100
```

![准备返回地址](https://upload-images.jianshu.io/upload_images/5690299-8f1af6e78e5346a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

七、函数调用前的保护现场，以便返回时恢复现场。 （CPU -> 内存）

把寄存器 ebp 的值压到栈里去

把 esp 的值赋给 ebp

把寄存器 ebx 的值压入栈

![保护现场](https://upload-images.jianshu.io/upload_images/5690299-956f4ac818ebe8e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

八、 CPU 取出数据计算。（CPU）

```c
int add(int *xp , int *yp){
    int x = *xp;
    int y = *yp;
    return x+y
}
```

把 ebp 加 8 的数据取出来放到 edx 寄存器。取得 xp 。

把 ebp 加 12 的数据取出来放到 ecx 寄存器。取得 yp 。

把 edx 指向的内存地址的数据取出来，放到 ebx 寄存器。取得 *xp 放入 ebx 。

把 ecx 指向的内存地址的数据取出来，放到 eax 寄存器。取得 *yp 放入 eax 。

把 ebx 和 eax 的值加起来，放到 eax 寄存器中。取得 *xp+\*yp，放入 eax 。

九、恢复现场。（内存 -> CPU）

把 esp 指向的数据弹出的 ebx 寄存器。

把 esp 指向的数据弹出到 ebp 寄存器。

![恢复现场](https://upload-images.jianshu.io/upload_images/5690299-fb4b0aabe9472787.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

十、返回。

CPU 取出返回地址 100，去那里找到指令继续执行。

## 函数调用总结

1. 把参数和返回地址准备好。

2. 新建函数帧：
把寄存器 ebp 的值压入栈。
把 esp 的值赋给 ebp。

3. 函数调用完后， 重置 ebp 和 esp，让他们重新指向调用方的函数帧。

## 内存缓冲区溢出

用户输入的数据是从低地址向高地址存放的。

黑客输入精心设计的数据过多以至于覆盖了返回地址，让返回地址指向恶意代码入口地址。

输入函数有边界检查就可避免攻击。

参考文档：

[http://mp.weixin.qq.com/s/hX8tHnoq4gdkhcLHUGzEbg](http://mp.weixin.qq.com/s/hX8tHnoq4gdkhcLHUGzEbg)

[http://suo.im/5pESl7](http://suo.im/5pESl7)