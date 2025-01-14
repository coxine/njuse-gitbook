# 07-路由

### 基础知识

#### 内部组件

| 组件名称      | 作用                                          | 易失性             | 储存内容                                                                         |
| --------- | ------------------------------------------- | --------------- | ---------------------------------------------------------------------------- |
| RAM       | 内存                                          | 重启/关机后丢失        | <p>存储临时配置文件<br>路由表<br>ARP缓存<br>快速切换缓存<br>报文缓存：可能前面有正在处理的，需要等待<br>数据包保留队列</p> |
| NVRAM     |                                             | 重启/关机后不丢失       | 备份、启动配置文件                                                                    |
| Flash     | EEPROM，在不换闪存芯片的情况发更新系统                      | 重启/关机后不丢失       | （多个版本的）操作系统                                                                  |
| ROM       | 启动                                          | 无法重复写入，更新需更换芯片组 | <p>开机自检<br>启动程序<br>IOS的精简备份版本</p>                                            |
| Interface | <p>数据包通过其进入和离开路由器的网络连接口<br>附在主板上作为单独的模块</p> |                 |                                                                              |

#### 接口

* 物理接口：实际连接设备
* 逻辑接口：虚拟接口，用于管理和配置
  * VLAN接口：虚拟局域网
  * Loopback接口：用于路由器自身的测试和管理

### 路由器启动流程

#### 系统启动流程

1. 执行开机自检（POST）：通过ROM中的程序检测所有硬件
2. 验证CPU、内存、网络接口的基础使用正常
3. 初始化软件

#### 软件启动流程

1. ROM中的通用引导程序在CPU卡上执行
2. 找到操作系统所在的地址（在配置寄存器的引导字段中）
3. 加载操作系统镜像
4. 加载NVRAM中的配置文件进入主存，逐行执行
5. 若无配置文件，执行**设置模式**（通过交互式的询问进行初始化，只能进行有限范围的配置）

### 基本配置命令

#### 命名

```shell
Router(config)# hostname <name> # 命名
```

#### 密码

```shell
Router(config)# enable secret <password> # 设置特权模式密码
Router(config)# line console 0 # 进入控制台线路配置模式
Router(config-line)# password <password> # 设置控制台密码
Router(config-line)# login # 设置控制台登陆
Router(config)# line vty 0 4 # 进入虚拟终端线路配置模式
Router(config-line)# password <password> # 设置虚拟终端密码
Router(config-line)# login # 设置虚拟终端登陆
```

#### 登陆提示文字

```shell
Router(config)# banner motd <message> 
```

#### 接口配置

```shell
Router(config)# interface <interface> # 进入接口配置模式
Router(config-if)# ip address <ip> <mask> # 设置IP地址
Router(config-if)# no shutdown # 开启接口
```

#### 保存配置

```shell
Router# copy running-config startup-config # 保存配置
```

#### 查看配置

```shell
Router# show running-config # 查看运行配置
Router# show startup-config # 查看启动配置
Router# show ip route # 查看路由表
Router# show ip interface brief # 查看接口信息
Router# show ip interfaces # 查看接口详细信息
```

### 路由

* 将数据包从一个数据链路传输到另一个数据链路的设备
* 两个基本功能
  * **确定最佳路径**：选择数据包最合适的的下一跳
  * **交换路由**：将数据包从一个接口转发到另一个接口
* 默认路由：当路由表中没有匹配的路由时，将数据包转发到默认路由

| 对比项          | 静态路由            | 动态路由            |
| ------------ | --------------- | --------------- |
| 配置方式         | 手动配置            | 自动学习和更新路由信息     |
| 适用场景         | 小型网络            | 大型网络            |
| 资源占用（CPU/内存） | 少               | 多               |
| 带宽使用         | 不占用带宽           | 消耗带宽            |
| 故障恢复（收敛速度）   | 需要手动更改路由表，收敛速度慢 | 收敛速度快，能自动适应拓扑变化 |
| 安全性          | 风险小             | 风险大             |
| 灵活性          | 缺乏灵活性           | 高度灵活            |

### 静态路由

* Stub network：网络的一小部分，不知道整个网络的拓扑，仅通过一条路径和其他网络通信

#### 管理距离

* 在0-255之间
* 用于在出现目标相同的规则时确认优先级
* 数值越小，距离越短，优先级越高
* 0：直连网络
* 1：静态路由

### 动态路由

* 可通过不同路径传输数据，负载均衡
* 网络冗余，保证连通性
* 需要通过路由协议在路由器间交换路由信息，基于2个功能
  * 维护路由表
  * 将路由信息传递给其他路由器
* 收敛时间：当网络拓扑启动/发生变化时，路由器会重新计算路由表，直到稳定，覆盖整个拓扑的时间

### 配置路由

#### 配置静态路由

```shell
Router(config)# ip route <destination> <mask> <ip/interface> <distance> # 添加静态路由
# destination 目的网络
# mask 掩码
# ip/interface 下一跳地址或者出口接口
# distance 若不设置则默认为1
Router(config)# ip route 0.0.0.0 0.0.0.0 <ip> # 设置默认路由
# 将所有包都转发到指定的ip地址
```

#### 配置动态路由

```shell
Router(config)# router <protocol> <options> # 进入路由器配置模式
Router(config-router)# network <network> # 添加网络
# protocol  RIP、OSPF、IGRP、EIGRP
Router(config)# ip default-network <network> # 设置默认网络
```

### IGP & EGP

* IGP：Interior Gateway Protocol 内部网关协议
  * 用于在同一个自治系统内部传递路由信息
  * RIP、OSPF、IGRP、EIGRP
* EGP：Exterior Gateway Protocol 外部网关协议
  * 用于在不同自治系统之间传递路由信息
  * BGP、EGP

### Distance Vector 距离矢量协议

* 路由器只知道可达，**不知道具体拓扑**
* 基于距离矢量的路由算法（Bellman-Ford算法）周期性交换路由表
* RIP基于Distance Vector协议，最大跳数为15
* IGRP、EIGRP基于Distance Vector协议，最大跳数为255，90秒更新

#### 环路问题

* 当网络中有环路时，信息传递的时间差可能会导致路由表不一致（上例中Network 1 宕了，但是D经过一圈后传给A的消息代表还能继续访问，导致无限循环）
* 解决方案
  * 设置最大值：当跳数超过一定值时，不再传递，视为不可达
  * 路由毒化
    1. 当路由不可达时，将其距离设置为最大值
    2. Poision reverse：当其他路由器接收到路由毒化的消息时，将更新信息返回给发送者
    3. 最终所有路由器知道不可达
  * 水平分割
    * 当网络不可达时，邻近的路由器不再接受目标为不可达网络的信息
    * 若为其他路由（绕开故障链路）则继续接受
  * 计时器
    * 每条路由表信息都包含时效信息
    * 当不可达时启动计时器，超时后删除
    * 若在超时前收到更好的路由信息，则更新

#### 阻止发送路由更新信息

```shell
Router(config-router)# passive-interface <interface>  # 阻止发送路由更新信息
```

* 仅使用于Distance Vector协议

### Link State 链路状态协议

#### 步骤

1. 路由器从直接连接的网络开始交换LSA
2. 路由器收到LSA后，将其存储在链路状态数据库中
3. 路由器通过SPF算法计算最短路径树
4. 路由器将最短路径树转换为路由表

#### 缺点

* 内存和CPU开销大
* 初始广播消耗带宽大
* 当链路状态变化时，需要重新计算整个网络的最短路径

### 对比

| 对比项  | Distance Vector 距离矢量协议   | Link State 链路状态协议           |
| ---- | ------------------------ | --------------------------- |
| 视野   | 狭窄，无法获知具体拓扑              | 视野宽，知道整个拓扑结构                |
| 广播内容 | 路由表                      | 链路状态                        |
| 广播形式 | 被动，周期性广播                 | 主动，链路状态变化时广播                |
| 路由算法 | 通过跳数计算距离                 | 通过SPF（Dijkstra算法）计算最短路径     |
| 收敛速度 | 较慢，更新信息需要通过多个周期才能传播到整个网络 | 较快，一旦链路状态更新，所有路由器可以迅速重新计算路由 |
| 储存内容 | 路由表存储：存储所有路由器的路由表        | 链路状态数据库：存储所有路由器的链路状态信息      |

### Hybrid Routing 混合路由协议

* 结合二者优点（通过跳数计算距离+链路状态变化时广播）

### 常见的路由协议

| 协议      | 全称                                           | 类型              |
| ------- | -------------------------------------------- | --------------- |
| RIP     | Routing Information Protocol                 | Distance Vector |
| OSPF    | Open Shortest Path First                     | Link State      |
| (E)IGRP | (Enhanced) Interior Gateway Routing Protocol | Distance Vector |
| EIGRP   | Enhanced Interior Gateway Routing Protocol   | Hybrid          |
