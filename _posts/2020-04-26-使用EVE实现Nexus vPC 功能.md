---
layout:     post   				    
title:      使用EVE实现Nexus vPC 功能 
subtitle:   使用EVE实现Nexus vPC 功能
date:       2020-04-26
author:     i9u 						
header-img: img/post-bg-2015.jpg 	
catalog: true 						
tags:								
    - EVE-NG
---

## 使用EVE实现Nexus vPC 功能
>这是我的第一篇博客。

#### 前言
vPC是Cisco交换机虚拟化功能，能够在双上联非堆叠的情况下使用port channel的技术，当然我想看这编文章的人也相当清楚这个技术的原理和应用场景。

#### 实验环境
1. 模拟器 EVE-ng社区版

#### 镜像
1. Cisco NX-OSv 9k-7.0.3.17.4
2. i86bi_linux_l2-adventerprisek9-ms.SS

#### 拓扑图
![vpc topology](https://kdrdya.bn.files.1drv.com/y4mfKsPsBA3HhpZ7-0x9TFqqST4ME9AEPEXrI0TbcqAKmowXtMpdmpSK2KIOAtBy6HYughD_nN7Mwlv58lZk598g_tN9kQSFEfcstqGQAs4EcsYO_aqyKumR2bJjn39p_uELdhgrwMMJAeA6sdosUOqDX1z0y4C69pN67QHfKpdwPNgle5JJgQL4rX1QBYliawTFTxmsspR0hZjEaF9dvo-gw/vpc%20topo.png?psid=1 "vpc topology")

#### 简单描述
1. Mgmt 0 口用作直连peer keep alive检测用
2. E1/2-3则作为vPC peer link 用
3. E1/4-5是属于vPC member port ，用于连接下挂的交换机。
4. 配置port-channel mode的时候其实可以不使用LACP亦能够完成对接，这里的member port就使用静态的on模式，peer-link则使用LACP。
