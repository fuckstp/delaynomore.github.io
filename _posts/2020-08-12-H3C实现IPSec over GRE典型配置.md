---
layout:     post
title:      H3C实现IPsec over GRE典型配置
subtitle:   使用的模拟器是H3C的官方模拟器HCL，可以从它的官网处下载。
date:       2020-08-12
author:     i9u
header-img: img/Ec5R55jWkAAssjO-min.png
catalog: true
tags:
    - H3C
    - IPSec
    - GRE
    - VPN
    - 配置案例
---

# H3C实现IPsec over GRE典型配置

Router A和Router B之间建立GRE隧道，Router A和Router B下的PC网段间流量走GRE，并使用IPSec对在GRE中对流量进行加密，Route C则模拟在ISP侧的设备。





## 0.实验环境

模拟器：H3C官方模拟器HCL，即H3C Cloud Lab

设备：H3C MSR36-20 Version 7.1.075, Alpha 7571

拓扑图：

![image-20200812151037409](https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/sdn/image-20200812151037409.png)

## 1.基础配置

A和B之间使用静态路由使出口路由可达，并且先起个OSPF进程，使用环回口作为Router-id，因为最后它们要在tunnel上实现OSPF的互联。



Route A：

```
#
ospf 1 router-id 100.1.1.1
 area 0.0.0.0
#
interface LoopBack0
 ip address 100.1.1.1 255.255.255.0
 ospf 1 area 0.0.0.0
#
interface GigabitEthernet0/0
 port link-mode route
 ip address 10.1.1.2 255.255.255.0
#
interface GigabitEthernet0/1
 port link-mode route
 ip address 192.168.1.1 255.255.255.0
 ospf 1 area 0.0.0.0

#
 ip route-static 0.0.0.0 0 10.1.1.1
#
```

Route C：

```

#
interface GigabitEthernet0/0
 port link-mode route
 ip address 10.1.1.1 255.255.255.0
#
interface GigabitEthernet0/1
 port link-mode route
 ip address 20.1.1.1 255.255.255.0

#
```

Route B：

```
#
ospf 1 router-id 200.1.1.1
 area 0.0.0.0
#
interface LoopBack0
 ip address 200.1.1.1 255.255.255.0
 ospf 1 area 0.0.0.0
#
interface GigabitEthernet0/0
 port link-mode route
 ip address 20.1.1.2 255.255.255.0
#
interface GigabitEthernet0/1
 port link-mode route
 ip address 192.168.2.1 255.255.255.0
 ospf 1 area 0.0.0.0

#
 ip route-static 0.0.0.0 0 20.1.1.1
#
```



## 2.GRE tunnel 配置

模拟器默认设备系统版本都是V7，需要在创建Tunnel的时候就指定为GRE隧道。Tunnel要将源设定为自己的出口IP，目的是对端的设备的出口IP，以A为视角的话，Tunnel source 就是它的出口IP，destination是B的出口IP，反过来也是这样。顺便将OSPF网络设定成P2P模式，方便快速建立邻居。

Route A：

```

interface Tunnel0 mode gre
 ip address 172.16.1.1 255.255.255.0
 ospf network-type p2p
 ospf 1 area 0.0.0.0
 source 10.1.1.2
 destination 20.1.1.2

```

Route B：

```

interface Tunnel0 mode gre
 ip address 172.16.1.2 255.255.255.0
 ospf network-type p2p
 ospf 1 area 0.0.0.0
 source 20.1.1.2
 destination 10.1.1.2

```



## 3.IPSec 配置

两台设备都需要创建ACL抓取需要加密的流量，不然不会触发IPSec加密行为。同时需要注意transform-set的加密和认真参数需要一支， V7设备预共享密钥的配置需要在ike keychain中配置，不同于V5设备。建议在实际生产环境中操作的时候最好直接使用复制粘贴来配置IPSec policy。



Route A：

```
acl advanced 3000
 rule 0 permit ip source 192.168.1.0 0.0.0.255 destination 192.168.2.0 0.0.0.255
 rule 5 permit ip source 192.168.2.0 0.0.0.255 destination 192.168.1.0 0.0.0.255

#
ipsec transform-set 123
 esp encryption-algorithm 3des-cbc
 esp authentication-algorithm md5
#
ipsec policy 123 1 isakmp
 transform-set 123
 security acl 3000
 remote-address 172.16.1.2
#
ike keychain 1
 pre-shared-key address 172.16.1.2 255.255.255.255 key simply 123

```

Route B：

```
acl advanced 3000
 rule 0 permit ip source 192.168.1.0 0.0.0.255 destination 192.168.2.0 0.0.0.255
 rule 5 permit ip source 192.168.2.0 0.0.0.255 destination 192.168.1.0 0.0.0.255

#
ipsec transform-set 123
 esp encryption-algorithm 3des-cbc
 esp authentication-algorithm md5
#
ipsec policy 123 1 isakmp
 transform-set 123
 security acl 3000
 remote-address 172.16.1.1
#
ike keychain 1
 pre-shared-key address 172.16.1.1 255.255.255.255 key simply 123

```



## 4.启用IPSec

最后，我们只要在tunnel口下把在第三步创建的IPSec policy 调用出来就可以了。

Route A/B：

```
interface Tunnel0 mode gre
 ipsec apply policy 123
```

## 5.验证结果

我们直接ping下，会发现IPSec第一个包都会超时，这是正常的，只要通了那就代表配置没错了。

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/sdn/image-20200812153509793.png" alt="image-20200812153509793" style="zoom:67%;" />

由于Router A和Router B内网的流量已在ACL中定义为需加密保护的流量，因此将触发IPsec进行加密。可以通过如下显示信息看到，IKE协商成功，生成了SA。

```
<H3C>display ike sa
    Connection-ID   Remote                Flag         DOI
------------------------------------------------------------------
    1               172.16.1.1            RD           IPsec
Flags:
RD--READY RL--REPLACED FD-FADING RK-REKEY

```

可以通过如下显示信息查看协商生成的IPsec SA。

```
<H3C>display ipsec sa
-------------------------------
Interface: Tunnel0
-------------------------------

  -----------------------------
  IPsec policy: 123
  Sequence number: 1
  Mode: ISAKMP
  -----------------------------
    Tunnel id: 0
    Encapsulation mode: tunnel
    Perfect Forward Secrecy:
    Inside VPN:
    Extended Sequence Numbers enable: N
    Traffic Flow Confidentiality enable: N
    Path MTU: 1420
    Tunnel:
        local  address: 172.16.1.2
        remote address: 172.16.1.1
    Flow:
        sour addr: 192.168.2.0/255.255.255.0  port: 0  protocol: ip
        dest addr: 192.168.1.0/255.255.255.0  port: 0  protocol: ip

    [Inbound ESP SAs]
      SPI: 1503174692 (0x5998a024)
      Connection ID: 30064771075
      Transform set: ESP-ENCRYPT-3DES-CBC ESP-AUTH-MD5
      SA duration (kilobytes/sec): 1843200/3600
      SA remaining duration (kilobytes/sec): 1843199/3316
      Max received sequence-number: 4
      Anti-replay check enable: Y
      Anti-replay window size: 64
      UDP encapsulation used for NAT traversal: N
      Status: Active

    [Outbound ESP SAs]
      SPI: 3411111458 (0xcb516e22)
      Connection ID: 30064771074
      Transform set: ESP-ENCRYPT-3DES-CBC ESP-AUTH-MD5
      SA duration (kilobytes/sec): 1843200/3600
      SA remaining duration (kilobytes/sec): 1843199/3316
      Max sent sequence-number: 4
      UDP encapsulation used for NAT traversal: N
      Status: Active

```

可以通过如下显示信息查看GRE隧道的建立情况。

```
[H3C]display interface tunnel0
Tunnel0
Current state: UP
Line protocol state: UP
Description: Tunnel0 Interface
Bandwidth: 64 kbps
Maximum transmission unit: 1476
Internet address: 172.16.1.2/24 (primary)
Tunnel source 20.1.1.2, destination 10.1.1.2
Tunnel keepalive disabled
Tunnel TTL 255
Tunnel protocol/transport GRE/IP
    GRE key disabled
    Checksumming of GRE packets disabled
Output queue - Urgent queuing: Size/Length/Discards 0/100/0
Output queue - Protocol queuing: Size/Length/Discards 0/500/0
Output queue - FIFO queuing: Size/Length/Discards 0/75/0
Last clearing of counters: Never
Last 300 seconds input rate: 18 bytes/sec, 144 bits/sec, 0 packets/sec
Last 300 seconds output rate: 18 bytes/sec, 144 bits/sec, 0 packets/sec
Input: 1711 packets, 122752 bytes, 0 drops
Output: 1699 packets, 120360 bytes, 0 drops

```

