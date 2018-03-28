---
layout: post
title:  '数据链路层'
categories: 计算机网络
author: CHH
---

* content
{:toc}




## 三个问题

- 封装成帧：加首部、尾部（SOH、EOT）
- 透明传输：加 ESC ，用来对 SOH、EOT、ESC 进行转义
- 差错检测：循环冗余检验（CRC）

## PPP 协议

用户计算机和 ISP 通信

## CSMA/CD 协议

Carrier Sense Multiple Access/Collision Detection

特点：广播信道，同一时间只能允许一台计算机发送数据。

用途：以太网中使用该协议。（以太网是一种总线型局域网）

多点接入：总线型网络，一条总线连多台计算机。

载波监听：每个站都不停地监听信道。

碰撞检测：在发送前，如果信道被占用，则等待直至信道空闲。由于电磁波传播时延，有可能会发生碰撞。

## MAC 层

![MAC 层](https://upload-images.jianshu.io/upload_images/5690299-42b493fc91bcd2ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前同步码 ：为了计算 FCS 临时加入的，计算结束之后会丢弃。

目的地址：目的主机网卡 MAC 地址。6 个字节。

类型：标记上层协议，即 ipv4 还是 ipv6。

FCS ：帧检验序列，使用的是 CRC 检验方法。

## 知识串联

数据链路层分层，分成 LLC 层和 MAC 层（LLC 层不用管，可以把 MAC 层肤浅地当作数据链路层 ）。
这一层要实现用户主机与 ISP 通信，于是有了 PPP 协议，相应地也有 PPP 帧。
要实现以太网通信，于是有了 CSMA/CD 协议，相应地也有了 MAC 帧。
无论采用哪种协议，都要解决三个基本问题：封装成帧、透明传输、差错控制