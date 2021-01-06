---
layout:     post
title:      Virtual lan
subtitle:   还没想好写什么
date:       2020-12-29
author:     i9u
header-img: img/EiAbWPnU4AAhLSA.jpg
catalog: true
tags:
    - layer 2
    - 不学不行系列
---







### 前言

Virtual lan我们一般都叫做“VLAN”，所以下文我会统称为VLAN。一台交换机在默认情况下，所有端口都属于一个VLAN，我们通过VLAN技术可以将特定端口分配到特定的VLAN中，可以理解成交换机被分虚拟化成多个 了。有书也会说VLAN是一种很基本的网络虚拟化功能，因为它的确使交换机在逻辑上被分割成了多个，相同的VLAN的端口就属于一台被分割出来的交换机，它们不同VLAN的端口之间是无法通信的。

VLAN的基本实现原理是通过在数据包中打上VLAN Tag，使得无论是单播、组播、广播都只会在配置了相同VLAN的端口中转发。这种技术有两种标准，一种是思科私有的“ISL”，另一种是IEEE定义的802.1Q。在802.1Q是主流的今天，ISL已经是昨日黄花，你不必去学习它。

### VLAN的作用

VLAN除了 将交换机逻辑分割成多台，它还是实现了：

1.更安全，将有敏感数据的部门划分到不同的VLAN中，实现简单的隔离。

2.更方便，管理人员通过分割不同的VLAN来给不同的业务、部门，方便管理员管理。

3.更效率，分割的VLAN会阻隔其他VLAN的广播泛洪，减少不必要的流量从而提高性能。

4.抑制广播风暴，分割的VLAN如同第3点所讲，还能抑制广播风暴的影响，由于阻隔了广播泛洪，可以防止网络收到大量的广播报文影响。

这是802.1Q格式，它被插入到源mac地址后：

|  TPID   |  PRI   |  CFI   | VLAN ID |
| :-----: | :----: | :----: | :-----: |
| 2 Bytes | 3 Bits | 1 Bits | 12 Bits |

VLAN标签各字段含义：

TPID：Tag Protocol Identifier（标签协议标识符），表示数据帧类型。

PRI：Priority，表示数据帧的802.1p优先级。

CFI：Canonical Format Indicator（标准格式指示位），用于兼容以太网和令牌环网。

VLAN ID：VLAN ID，表示该数据帧所属VLAN的编号。

这四个字段，我们初学者只要知道VLAN ID就行了，12个Bit限制了它的数量范围：0~4095。而VLAN ID中又被区分出来用于不同的地方，它们的范围和用途看这里：

1.VLAN 0 和 VLAN 4095 仅限系统使用，用户是无法查看的。

2.VLAN 1 正常，是Cisco的默认端口VLAN，用户可以使用但是无法删除。

3.VLAN 2 ~ 1001 正常，用于Ethernet的VLAN，用户可以创建和删除。

4.VLAN 1002 ~ 1005 正常，用于FDDI和令牌环的Cisco默认vlan，用户不能删除。

5.VLAN 1006 ~ 1024 保留，仅仅限于系统使用，用户不能查看和使用

6.VLAN 1025 ~ 4094 扩展，用于Ethernet的VLAN，用户可以创建和删除。

### 配置 VLAN

VLAN的配置很简单，这是配置的步骤：

1.创建VLAN本身

2.将特定端口划入到特定的VLAN

尽管VLAN的划分方法还有很多种类，但是这里我们只根据端口来配置划分，我们来根据这图划分对应的VLAN。

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20201230172508037.png" alt="image-20201230172508037" style="zoom:67%;" />

按照步骤，首先在交换机创建VLAN

```
Switch(config)#vlan 10
Switch(config-vlan)#name blue
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#name red
Switch(config-vlan)#exit
```

创建完成后，可以查看到现有的VLAN

```
Switch#show vlan 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
10   blue                             active    
20   red                              active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

根据图所示，将接口e0/0、e0/2划入到VLAN 10，接口e0/1、e0/3划入到VLAN 20

```
VLAN 10

Switch(config)#inter ether0/0
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shu
Switch(config-if)#exit
Switch(config)#inter ether0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shu
Switch(config-if)#exit

VLAN 20

Switch(config)#inter ether0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20
Switch(config-if)#no shu
Switch(config-if)#exit
Switch(config)#inter ether0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20
Switch(config-if)#no shu
Switch(config-if)#exit
```

完成后，使用如下图的命令可以确认配置结果

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20201230173123385.png" alt="image-20201230173123385" style="zoom:67%;" />



这时候，如果我们给上图两个VLAN内的4台配置上分别一个网段的IP：

VPC1:10.1.1.1/24

VPC2:10.1.1.2/24

VPC3:10.1.1.3/24

VPC4:10.1.1.4/24

然后使用VPC 1 发起Ping到VPC 3，网络是正常的，ARP双方都有询问，并且ICMP正常的交互：

![](C:\Users\SYUU\AppData\Roaming\Typora\typora-user-images\image-20201230181024954.png)

但是当VPC 1 去访问VPC 2 的时候：

![](C:\Users\SYUU\AppData\Roaming\Typora\typora-user-images\image-20201230181100272.png)

尽管VPC 1发出了多次ARP Request，但是没人回复它，那是因为VLAN将广播域隔开了，从逻辑上来看，拓扑图似乎变成了这样：

![](https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20201230181141544.png)

其实就像开头说的那样，VLAN是一种网络虚拟化技术，实现了将交换机基于端口的来分割成多台。上图的VLAN10\20，基于端口的将这四个口分别分割成了两台虚拟交换机（或者像书面那样，虚拟局域网），这样使得这4台电脑就算是同一个子网内，ARP的请求包也无法跨过它们的VLAN，这也就实现了上面说的安全功能。

当然，实际在生产环境中，我们会基于VLAN分配一个网段，比如VLAN 10 会给10.1.1.0/24、VLAN 20 会给20.1.1.0/24，上述的例子只是为了方便演示VLAN隔离广播域的效果。



### Access、Trunk和Hybrid

我们可以将交换机端口配置成这三种类型：Access、Trunk和Hybrid，它们的作用分别是：

Access：改变端口的Native VLAN，修改后，所有从该端口进入的数据包的Navtie VLAN都会变成Access的那个VLAN，一般连接PC用。

```
Cisco
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10

Huawei
[HUAWEI-GigabitEthernet1/0/1] port link-type access
[HUAWEI-GigabitEthernet1/0/1] port default vlan 10
```

Trunk：对数据包打上802.1Q的VLAN Tag，然后根据放行的VLAN来允许这些报文通过，一般连接虚拟化服务器和交换机用。

```
Cisco
Switch(config-if)#switch mode trunk
Switch(config-if)#switch trunk encapsulation dot1q 
Switch(config-if)#sw trunk allowed vlan 10,20

Huawei
[HUAWEI-GigabitEthernet1/0/2] port link-type trunk
[HUAWEI-GigabitEthernet1/0/2] port trunk allow-pass vlan 100
```

Hybrid：华为、华三等国产设备独有，可以允许多个VLAN通过，但是打不打VLAN Tag则有用户来决定。

```
[HUAWEI] interface gigabitethernet 1/0/3
[HUAWEI-GigabitEthernet1/0/3] port link-type hybrid
[HUAWEI-GigabitEthernet1/0/3] port hybrid tagged vlan 10
```



### Native VLAN和VLAN 1



其实当VLAN 1 通过Trunk的时候，如果你抓包会发现报文里面的VLAN字段是空的，那是因为默认的情况下VLAN 1就是Native VLAN，而Native VLAN在通过Trunk的时候就是不打VLAN Tag的。

VLAN 1除了是默认的Native VLAN外，还是Cisco的所有交换机端口的默认VLAN，而且还是Cisco在传递Control Plane Traffic 时候所使用的VLAN，比如生成树的BPDU传递、VTP、CDP等等。所以为了安全，一般不要将VLAN 1分配给PC使用，因为PC使用Wireshark等抓包软件很容就能抓到Control Plane的重要流量。

Navtie VLAN可以通过命令查看、修改：

```
Switch#show inter trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      10

interface Ethernet0/0
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 10
 switchport mode trunk

```

### 跨交换的VLAN 访问

我们来完整的做一个实例，通过配置VLAN、ACCESS、TRUNK来实现这4台PC的跨交换机VLAN内互访

![image-20210104155812617](https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20210104155812617.png)

1.两台交换机创建相应的本地VLAN

```
Switch(config)#vlan 10
Switch(config-vlan)#name blue
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#name red
Switch(config-vlan)#exit
```

2.将相应的接口使用Access 来划入到各自的VLAN里

```
左边的交换机

Switch(config)#inter ether0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shu
Switch(config-if)#exit
Switch(config)#inter ether0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20
Switch(config-if)#no shu
Switch(config-if)#exit

右边的交换机

Switch(config)#inter ether0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shu
Switch(config-if)#exit
Switch(config)#inter ether0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20
Switch(config-if)#no shu
Switch(config-if)#exit
```

3.两台交换机之间的互连接口配置Trunk，并放行VLAN 10 20

```
Switch(config-if)#switch mode trunk
Switch(config-if)#switch trunk encapsulation dot1q 
Switch(config-if)#sw trunk allowed vlan 10,20
```

4.验证Trunk的配置状态是Trunking，模式是on。

```
Switch#show inter trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      10
```

5.然后，尝试VLAN内互Ping和跨VLAN的互Ping

```
VPC1#ping 10.1.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
VPC1#ping 20.1.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 20.1.1.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

6.在交换机的Trunk口中 抓包，可以清楚看到VLAN ID

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20210104160805656.png" alt="image-20210104160805656" style="zoom:80%;" />

7.若将VLAN 10设定成Native VLAN，是抓不到有VLAN ID的

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20210104160850405.png" alt="image-20210104160850405" style="zoom: 80%;" />

### 跨VLAN的访问

由于VLAN将广播域分割了，这时候如果我们想要实现跨VLAN访问的时候，就需要有网关来帮我们做转发，试图回想下ARP的内容，我们在跨子网访问的时候同样也需要有网关，这里的道理是一样的。

实现跨VLAN访问的方法：

1.单臂路由

简单的来说就是通过交换机和一台带有Layer 3功能的设备互联，使用子接口加Dot1q来区分不同VLAN的数据包，当数据包到达Layer 3设备（通常是路由器）使它自己进行A	RP查表转发。

它的优点是成本低，实现难度低，缺点是因为要在一条线缆上拆分出多子接口，流量会被平摊掉。

我们来尝试下单臂路由的实验，下图是拓扑图：

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20210106165619600.png" alt="image-20210106165619600" style="zoom: 67%;" />

蓝色是VLAN 10、网关在R1上，为10.1.1.254，VPC3 IP 为10.1.1.1，VPC4 IP 为10.1.1.2 

红色是VLAN 20、网关在R1上，为20.1.1.254，VPC1 IP 为20.1.1.1，VPC2 IP 为20.1.1.2 

下面是R1和SW的配置，注意SW上要创建相应的VLAN 10 \ 20，这里就不贴出来了：

```
R1

interface Ethernet0/0
 no ip address
 duplex auto
!
interface Ethernet0/0.1
 encapsulation dot1Q 10
 ip address 10.1.1.254 255.255.255.0
!
interface Ethernet0/0.2
 encapsulation dot1Q 20
 ip address 20.1.1.254 255.255.255.0
 
 
SW
interface Ethernet0/0
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
 interface Ethernet1/0
 switchport access vlan 10
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 10
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 20
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 20
 switchport mode access
```

当完成后，下面是VPC3 访问同VLAN和跨VLAN时的流量走势，蓝色是VLAN内互访，红色是跨VLAN互访，由于所有跨VLAN访问都要流经SW和R1的路径，而这条路径只有一条，所以叫做”单臂”路由。

<img src="https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20210106171122519.png" alt="image-20210106171122519" style="zoom:67%;" />

2.三层交换机起SVI

笼统的来说就是给VLAN 上个网关，而这个网关是配置在交换机上，思科叫SVI，华为叫VLANif。这个功能是Layer 2 交换机做不到的，但是现在市面上的交换机一般来说都是支持Layer 3，所以很常见。

我们来尝试下SVI的实验，下图是拓扑图：

![image-20210106173147451](https://cdn.jsdelivr.net/gh/fuckstp/fuckstp.github.io@master/img/vlan/image-20210106173147451.png)

基本上的逻辑是一样，只不过我们这次将网关配在了交换机上，所以连R1都不需要了：

```
interface Ethernet1/0
 switchport access vlan 10
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 10
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 20
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 20
 switchport mode access
!
interface Vlan10
 ip address 10.1.1.254 255.255.255.0
!
interface Vlan20
 ip address 20.1.1.254 255.255.255.0
```

配置了SVI后，我们可以通过命令查看接口可以发现有两个VLAN接口被新增到里面了，这就是SVI：

```
Switch#show ip inter brief
Interface              IP-Address      OK? Method Status                Protocol
Ethernet1/0            unassigned      YES unset  up                    up      
Ethernet1/1            unassigned      YES unset  up                    up      
Ethernet1/2            unassigned      YES unset  up                    up      
Ethernet1/3            unassigned      YES unset  up                    up      
Vlan10                 10.1.1.254      YES manual up                    up      
Vlan20                 20.1.1.254      YES manual up                    up   
```

注意它的Status和Protocol字段都是“UP”的情况下才算是正常工作，这种我们叫做“双UP”，要使SVI实现“双UP”的前提：

1.本地有这个VLAN；

2.有放行这个VLAN的物理接口，并且这个物理接口也是“双UP”。

那么，这个情况下就能实现跨VLAN的互访了，当然别忘记了交换机也需要支持Layer 3功能，如果是Cisco或者DELL的交换机，还额外需要加一条命令来开始Layer 3 功能：

```
Switch(config)#ip routing
```

