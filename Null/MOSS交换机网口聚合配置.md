# MOSS交换机网口聚合配置

## 1 、串口使用介绍

#### 步骤1

使用 Console 线缆将计算机与交换机的串口连接。盛科设备的串口位于交换机左侧，面板标示为“CON”。

![1574651925102](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1574651925102.png)

#### 步骤2

在计算机上打开终端仿真软件，新建连接，设置连接的端口及通信参数。计算机终端的
新建连接的通信参数配置要和交换机的串口的缺省配置保持一致。

> 580 系列交换机串口缺省配置默认为：
> − 传输速率（speed）：115200
> − 数据位（Data bits）：8
> − 校验方式（Parity）： 无
> − 停止位（Stop bits）：1
> − 流控方式（Flow control）： 无

![1574652179673](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1574652179673.png)



#### 步骤3

以上步骤完成后，在仿真终端软件的串口连接页面，输入回车键就可以进入交换机串口界面，输入 enable 可以进入交换机用户视图。

```
Switch> enable
Switch#
```

## 2 管理网口使用介绍

#### 步骤 1 

使用网线将计算机与交换机的管理网口相连接。盛科设备的管理网口位于交换机左侧，面板标示为“ETH”

![1574652400596](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1574652400596.png)

#### 步骤 2 

使用串口（串口的使用方法请见上一章节）配置交换机的管理网口静态 IP 地址以及网关

**管理网口的静态 IP 地址配置方式如下：**

```
Centec# configure terminal
Enter configuration commands， one per line. End with CNTL/Z.
Centec(config)# no management ip address dhcp
Centec(config)# management ip address 172.0.22.1/16
```

> NOTE：因为交换机出厂默认配置了 `management ip address dhcp`，需要在配置模式下先删除这条配置才可以配置管理网口的静态 ip，删除命令为 `no management ip address dhcp`。
>
> 机房交换机管理网口ip地址需要按照规划配置，并贴上标签。

**管理网口的网关配置方式如下：**

```
DUT1(config)# management route add gateway 172.0.0.254
```

> 172.0.0.254 是公司的网关地址。

**交换机的管理 IP 地址即被设置为 172.0.11.1，网关为 172.0.0.254**

> NOT：计算机如果和管理网口是直连的，计算机的 IP 地址应该和交换机管理网口的 IP 地址在同一个网段，这种情况下交换机管理网口不需要配置网关；计算机如果和管理网口不是直连的，中间还有一些其他网络设备，那么计算机的 IP 地址和交换机管理网口的 IP 地址可以不是一个网段的，但是交换机的管理网口必须配置有效的网关。

#### 步骤 3 

在交换机上使用 ping 功能检查管理网口与计算机的网络是否通畅。
比如计算机的 IP 地址为 172.0.10.204，则在交换机上使用 Ping：

```
Centec# ping mgmt-if 172.0.10.204
```

> 该操作需要exit出配置模式后执行。配置模式时命令行前面提示符为`Centec(config)#`，推出后为`Centec#`。
>

至此，交换机的管理网口配置完毕。

## 3、用户管理配置

### 3.1 概述

用户管理功能可用来增加系统的安全性，用户可以通过密码来登录。系统会限制登录用户的数量。

交换机上有三种模式登录：

- “no login”模式，任何人都可以直接登录交换机并且不需要密码；

- “login”模式，只有默认的用户登录；

- “login local”模式，假如用户想登录交换机，必须在系统中创建一个用户帐号。在本地创建用户帐号和密码可以帮助用户登录交换机。

每个交换机只有 32 个账户。在用户启用本地账户验证之前，必须提前创建一个账户。
用户可以为每个用户名设置不同的密码。每个用户名不能超过 32 个字符。
用户可以设置每个账户的等级，有效的等级 1-4。只有一个账户可以进入配置模式。

### 3.2 创建用户

**1、创建用户**

操作配置步骤和说明如下：

| 命令                                                         | 说明             |
| ------------------------------------------------------------ | ---------------- |
| `Switch# configure terminal`                                 | 进入配置模式     |
| `Switch(config)# line vty 0 7`                               | 进入用户模式     |
| `Switch(config-line)# login local`                           | 设置验证模式     |
| Switch(config-line)# exit                                    | 退出用户模式     |
| `Switch(config)# username username privilege 4 password passwd` | 创建用户名和密码 |
| `Switch(config)# exit`                                       | 退出配置模式     |

**2、命令验证**
经过以上配置，登录交换机时，系统会提示类似如下验证信息：

```
Username: username
Passwod:
```

## 4、使用ssh登录交换机

经过以上配置步骤，使用给交换机配置的**静态ip**，**用户名**和**密码**，可以通过ssh终端登录交换机，方便之后进行配置交换机网口聚合的操作。

## 5、配置交换机网口聚合

### 5.1 组网图

![绘图1](C:\Users\leixiao\Desktop\绘图1.png)

### 5.2 组网需求

moss1 和 moss2 处于同一网段，其双网卡使用 active-active 的方式双归接入到 MLAG设备，需求组网无环路，moss1 与 moss2 之间实现二层互通。其中 MLAG 1 和 MLAG 2 都使用静态链路聚合。

### 5.3 配置步骤

**1、配置 peer-link ，允许所有 vlan 通过**

配置 SWITCH A

```
#配置 SWITCH A
Switch_A# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch_A (config)# interface range eth-0-1 - 2
Switch_A (config-if-range)# no shutdown
Switch_A (config-if-range)# switchport mode trunk
Switch_A (config-if-range)# switchport trunk allowed vlan all
Switch_A (config-if-range)# static-channel-group 55
Switch_A (config-if-range)# exit
Switch_A (config)# interface agg 55
Switch_A (config-if)# spanning-tree port disable
Switch_A (config-if)#exit
Switch_A(config)# mlag configuration
Switch_A(config-mlag)# peer-link agg 55
Switch_A(config-mlag)# end
```

配置 SWITCH B

```
#配置 SWITCH B
Switch_B# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch_B(config)# interface range eth-0-1 – 2
Switch_B (config-if-range)# no shutdown
Switch_B(config-if-range)# switchport mode trunk
Switch_B(config-if-range)# switchport trunk allowed vlan all
Switch_B(config-if-range)# static-channel-group 55
Switch_B(config-if-range)# exit
Switch_B(config)# interface agg 55
Switch_B(config-if)# spanning-tree port disable
Switch_B(config-if)#exit
Switch_B(config)# mlag configuration
Switch_B(config-mlag)# peer-link agg 55
Switch_B(config-mlag)# end
```

**2、配置三层接口，地址用作 peer-address ，借用 peer-link 通道建立 MLAG 邻居**

配置 SWITCH A

```
#配置 SWITCH A
Switch_A# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch_A(config)# vlan database
Switch_A(config-vlan)# vlan 4094
Switch_A(config-vlan)# exit
Switch_A(config)# interface vlan 4094
Switch_A(config-if)# ip address 10.10.0.1/24
Switch_A(config-if)# exit
Switch_A(config)# mlag configuration
Switch_A(config-mlag)# peer-address 10.10.0.2
Switch_A(config-mlag)# end
```

配置 SWITCH B

```
#配置 SWITCH B
Switch_B# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch_B(config)# vlan database
Switch_B(config-vlan)# vlan 4094
Switch_B(config-vlan)# exit
Switch_B(config)# interface vlan 4094
Switch_B(config-if)# ip address 10.10.0.2/24
Switch_B(config-if)# exit
Switch_B(config)# mlag configuration
Switch_B(config-mlag)# peer-address 10.10.0.1
Switch_B(config-mlag)# end
```

**3、配置 MLAG 组，将接口加入 MLAG 组中**

配置 SWITCH A

```
#配置 SWITCH A
Switch_A# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch_A(config)# vlan database
Switch_A(config-vlan)# vlan 10
Switch_A(config-vlan)# exit
Switch_A(config)# interface eth-0-3
Switch_A(config-if)# no shutdown
Switch_A(config-if)# switchport access vlan 10
Switch_A(config-if)# static-channel-group 1
Switch_A(config-if)# exit
Switch_A(config)# interface eth-0-4
Switch_A(config-if)# no shutdown
Switch_A(config-if)# switchport access vlan 10
Switch_A(config-if)# static-channel-group 2
Switch_A(config-if)# exit
Switch_A(config)# interface agg 1
Switch_A(config-if)# mlag 1
Switch_A(config-if)# exit
Switch_A(config)# interface agg 2
Switch_A(config-if)# mlag 2
Switch_A(config-if)# end
```

配置 SWITCH B

```
#配置 SWITCH B
Switch_A# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch_A(config)# vlan database
Switch_A(config-vlan)# vlan 10
Switch_A(config-vlan)# exit
Switch_A(config)# interface eth-0-3
Switch_A(config-if)# no shutdown
Switch_A(config-if)# switchport access vlan 10
Switch_A(config-if)# static-channel-group 1
Switch_A(config-if)# exit
Switch_A(config)# interface eth-0-4
Switch_A(config-if)# no shutdown
Switch_A(config-if)# switchport access vlan 10
Switch_A(config-if)# static-channel-group 2
Switch_A(config-if)# exit
Switch_A(config)# interface agg 1
Switch_A(config-if)# mlag 1
Switch_A(config-if)# exit
Switch_A(config)# interface agg 2
Switch_A(config-if)# mlag 2
Switch_A(config-if)# end
```

**4、验证配置结果**

**检查 MLAG 邻居状态**

```
#检查 MLAG 邻居状态，配置完成后 MLAG 处于 Established 状态
Switch_A# show mlag peer
MLAG neighbor is 10.10.0.2, MLAG version 1
MLAG state = Established, up for 00:00:01
Last read 00:00:01, hold time is 240, keepalive interval is 60 seconds
Received 5 messages,Sent 6 messages
Open : received 1, sent 1
KAlive : received 1, sent 1
Fdb sync : received 0, sent 0
Failover : received 0, sent 0
Conf : received 0, sent 0
Syspri : received 1, sent 1
Peer fdb : received 1, sent 1
STP Total: received 2, sent 3
Global : received 2, sent 3
Packet : received 0, sent 0
Instance: received 0, sent 0
State : received 0, sent 0
Connections established 1; dropped 0
Local host: 10.10.0.1, Local port: 50040
Foreign host: 10.10.0.2, Foreign port: 61000
remote_sysid: 1a53.71e9.c000
```

**检查 MLAG 设备状态**

```
#检查 MLAG 设备状态，两台设备处于 Master/Slave 状态
Switch_A# show mlag
MLAG configuration:
-----------------
role : Master
local_sysid : 8e79.b120.2e00
remote_sysid : 1a53.71e9.c000
mlag_sysid : 8e79.b120.2e00
local_syspri : 32768
remote_syspri: 32768
mlag_syspri : 32768
peer-link : agg55
peer conf : Yes
reload-delay : 300

#处于Slave状态
Switch_B# show mlag
MLAG configuration:
-----------------
role : Slave
local_sysid : 1a53.71e9.c000
remote_sysid : 8e79.b120.2e00
mlag_sysid : 8e79.b120.2e00
local_syspri : 32768
remote_syspri: 32768
mlag_syspri : 32768
peer-link : agg55
peer conf : Yes
reload-delay : 300
```

**检查 MLAG 组状态**，所有接口应都处于 UP 状态

```
Switch_A# show mlag interface
mlagid local-if local-state remote-state
1 agg1 up up
2 agg2 up up
```

## 6、当前交换机配置情况

当前已经配置两台交换机之间的peer-link，一台交换机的管理口ip为172.0.22.1，另一台管理口ip为172.0.22.2。用户名密码为：

| 用户名   | 密码   |
| -------- | ------ |
| username | passwd |

目前网口聚合环境都使用这两台交换机。

使用时只需要按照 **3、配置 MLAG 组，将接口加入 MLAG 组中** 这一步配置。

按照步骤配置时，需要修改的地方包括：

| 修改的地方(X代表一个数字) | 规则                                                         |
| ------------------------- | ------------------------------------------------------------ |
| interface eth-0-X         | 实际使用的网口编号                                           |
| switchport access vlan X  | vlan编号，如果环境使用的网口是5、6、7、8，<br />则vlan编号是最小网口编号加一个零，这里即是50 |
| static-channel-group X    | 和实际使用的网口编号一致                                     |
| mlag X                    | 和实际使用的网口编号一致                                     |

