# 08-路由协议

### RIP

* Distance Vector protocol
* Metric：跳数，最大为15跳

### RIP v1

#### 特点

* 30s广播一次
* 负载均衡：**跳数相同**时进行，默认4条，最多6条

#### 限制

* 只区分A类、B类、C类地址，不发送掩码，因而无法支持VLSM，CIDR
* 在`255.255.255.255`广播而非多播，占用带宽
* 无认证

### RIP v2

#### 特点

* 使用180s的计时器和水平分割避免环路
* 支持VLSM、CIDR
* 支持多播
* 支持认证

#### 运行方式

* 接口在`224.0.0.9`多播路由更新
* 当指定接口收到路由更新信息时，会被接受并更新路由表
* 该接口直接连接的子网会被广播

### RIP配置

```shell
# v1 配置
Router(config)# router rip # 配置RIP
Router(config-router)# network <network> # 添加网络
```

{% hint style="warning" %}
版本选择问题

直接使用`router rip`默认为v1
{% endhint %}

```shell
# v2 配置
Router(config)# router rip # 配置RIP
Router(config-router)# version 2 # 使用RIP v2
Router(config-router)# network <network> # 添加网络
```

```shell
# 查看RIP信息
Router# show ip route # 查看路由表
Router# show ip protocols # 查看路由协议
```

```output
Gateway of last resort is not set
  10.0.0.0  is variably subnetted, 2 subnets

R   10.2.2.0 (120/1) via 10.1.1.2 (120/2), 00:00:00, Serial0/0/0
# 120/1: 120为距离，1为度量值
```

```shell
# Debug
Router# debug ip rip # 查看RIP调试信息
Router# undebug all # 关闭所有调试
```

### OSPF

* Open Shortest Path First
* Link State protocol
* 相较于RIP系列更加先进

#### 基本术语

* Link：两个设备之间的物理链路
* Link-State：链路状态信息，包含接口信息和 Neighbor 的关系
* LSA：Link-State Advertisement，链路状态广播，包含链路状态信息
  * 发送 Router ID 时的 LSA 为Type 1

#### OSPF数据库

* Neighborship Database：存储已经建立双向连接的邻居信息（区域内所有机器不同）
* Topology Database：存储所有链路状态信息（区域内所有机器都相同）
* Routing Table：存储最短路径（区域内所有机器不同）

#### 度量值/开销

* $$Cost= \frac{10^8}{Bandwidth}$$
* 原因：距离短的路径若带宽小则不一定更快
* 同链路上的所有接口都需要设置相同的Cost

#### Router ID 选择顺序

1. 手动配置的Router ID
2. 最高的环回地址（Loopback地址）
3. 最高的活动接口的IP地址

#### OSPF网络种类

| 类型        | 描述          |
| --------- | ----------- |
| 广播多路复用网络  | 以太网         |
| 非广播多路复用网络 | Frame Relay |
| 点到点网络     | HDLC、PPP    |

#### DR/BDR

* 目的：减少链路状态更新的数量
* 适用于广播/非广播**多路复用**网络
* DR：Designated Router
  * 被钦定作为区域中的中心点，收集和分发路由信息
  * 路由器只需向DR发送链路状态更新，DR再向其他路由器发送
* BDR：Backup Designated Router，DR失效时接管
* 每个路由器都和DR/BDR建立邻接关系
* DR通过`224.0.0.5`多播地址发送链路状态信息
* 所有路由器通过`224.0.0.6`发送LSA供DR/BDR接受

#### 区域&分层

* Area：逻辑划分的区域，每个区域内的网络/路由器有相同的Area ID 和 Link-State
* Area ID 可以是32位的地址`0.0.0.0`，也可以是数字`0`
* Neighbor：直接连接的相邻设备，只有在同一个Area内的路由器才是Neighbor
* Area 0：骨干区
* 分为两层，其他所有区域只和骨干区相连，避免环路
* ABR：Area Border Router，区域边界路由器

{% hint style="info" %}
OSPF区域设计

为避免开销问题：

* &#x20;ABR最多不连接超过3个以上的路由器
* 一个Area中不能有超过50台路由器
{% endhint %}

#### OSPF Hello 协议

* 路由器启动OSPF时，以固定间隔向`224.0.0.5`发送Hello包
* 广播多路复用/点对点网络：10s一次
* 非广播多路复用网络：30s一次

#### OSPF包编号

| 编号 | 名称    | 作用                                               |
| -- | ----- | ------------------------------------------------ |
| 1  | Hello | **发现邻居**（以前考过）                                   |
| 2  | DBD   | 数据库描述                                            |
| 3  | LSR   | 链路状态请求：当路由器发现LSA不在链路状态数据库（LSDB）中/已过期，发送LSR包请求LSA |
| 4  | LSU   | 链路状态更新                                           |
| 5  | LSAck | 链路状态确认                                           |

### OSPF 流程

#### 建立邻接关系

* 定期发送Hello包，收到回复则建立邻接关系，加入邻居表
* 若是多路复用网络，选举DR/BDR
* 若是点对点网络，跳过下一步
* 若Hello包已有DR/BDR，跳过下一步

#### 选举DR/BDR

* 计算：先比较优先级，若相等比较Router ID，最大者为DR，次大者为BDR
* 优先级：0-255，默认为1
  * 0：不参与选举
* 若接口宕了，路由器会重新计算邻接关系，重新广播LSA，DR/BDR会重新选举
* 在DR/BDR选举完成后，及时新加入的设备优先级/Router ID更高，也不会立即成为DR/BDR，只有DR/BDR宕机后才会重新选举

#### 发现路线

* 确定主从关系
* 对应下面ExStart、Exchange、Loading状态

#### 选择合适路线

* 通过SPF算法计算最短路径，使用带宽作为度量值
* 若有相同开销的路径，会写入至多4条以负载均衡

#### 维护路由信息

* 定期发送Hello包，检测宕机和新加入的设备
* 发送周期
  * 点对点&广播多路复用：10s
  * 非广播多路复用：30s
* Dead Interval：若4次Hello包未收到回复，认为宕机，从邻居表中删除

### OSPF状态

{% hint style="warning" %}
糟糕的PPT

PPT上说有**seven**种状态，但实际上只写了6种，网上搜出来是8种😅

就按照下面来吧，~~这课烂完了~~
{% endhint %}

* Down：初始状态
* Init：A发送Hello包到B，B尚未回应，B将A写入邻居表
* 2-Way：B回复A，A将B写入邻居表
* ExStart：A和B交换Router ID，确认发送/接收方
* Exchange：优先级高的发送DBD包，低的处理后回复
* Loading：若信息不完整，发送LSR包请求LSA，接收LSU包
* Full：若信息完整，发送LSAck包，邻接关系建立，开始交换路由信息（**只有所有设备都完成Loading**后才能进入Full）

### OSPF配置

```shell
# 配置环回地址 Loopback Address
Router(config)# interface loopback <number> # 配置环回接口
Router(config-if)# ip address <ip> <mask> # 配置IP地址
# mask：使用255.255.255.255
# 无需 `no shutdown`
```

```shell
# 配置OSPF
Router(config)# router ospf <process-id> # 配置OSPF
# ID：1-65535，区分同一路由器上的不同OSPF进程 建议整个AS内统一 不统一也不影响
Router(config-router)# network <network> <wildcard-mask> area <area-id> # 添加网络
# wildcard-mask：子网掩码取反 计算时直接与IP地址取或即可获得范围的最后一个地址
```

```shell
# 更改OSPF优先级
Router(config)# interface <interface> # 进入接口配置
Router(config-if)# ip ospf priority <priority> # 设置优先级
```

```shell
# 查看OSPF信息
Router# show ip ospf <interface> # 查看OSPF接口信息
```

```shell
# 设置Cost
Router(config)# interface <interface> # 进入接口配置
Router(config-if)# ip ospf cost <cost> # 设置Cost
```

```shell
# 设置Timer
Router(config)# interface <interface> # 进入接口配置
Router(config-if)# ip ospf hello-interval <interval> # 设置Hello包发送间隔
Router(config-if)# ip ospf dead-interval <interval> # 设置Dead Interval
```

### 对比

| 标准   | RIP        | OSPF        |
| ---- | ---------- | ----------- |
| 度量   | 跳数         | 带宽          |
| 层次   | 单层         | 多层，可将网络继续划分 |
| 适用范围 | 小型网络 最多15跳 | 大型网络        |
| 收敛时间 | 慢          | 快           |
| 负载均衡 |            | 支持          |
| VLSM | 仅v2        | 支持          |
