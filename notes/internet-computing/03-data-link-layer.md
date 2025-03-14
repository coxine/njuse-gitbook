# 03-数据链路层

## 数据链路层概述

* 数据链路层提供了：介质的访问 + 物理传输
* 服务类型
  * 无连接，无确认的服务：在局域网、可靠链路（上层确保数据准确）、实时任务中使用
  * 无连接，有确认的服务：在无线等不可靠链路中使用
  * 面向连接的服务：蓝牙
* 数据链路层协议定义了
  * 数据帧的封装格式
  * 节点如何访问介质
* 主要任务：流控制、网络拓扑、错误检测
* 两个部分
  * MAC (Media Access Control) 子层：介质访问控制
  * LLC (Logical Link Control) 子层：逻辑链路控制

### 数据帧

```plaintext
 +-------------------------+
 |    Start Frame Field    |
 +-------------------------+
 |    Address Field        |
 +-------------------------+
 |    Type/Length Field    |
 +-------------------------+
 |    Data Field           |
 +-------------------------+
 |    Frame Check Sequence |
 +-------------------------+
 |    Stop Frame Field     |
 +-------------------------+
```

### 对比

| 物理层                  | 链路层          |
| -------------------- | ------------ |
| 无法与上层通信              | 通过LLC与上层通信   |
| 无法确定哪台主机将会传输或接受二进制数据 | 通过MAC确定      |
| 无法命名/区分主机            | 通过寻址或命名过程来实现 |
| 仅仅能描述比特流             | 通过帧来组织/分组比特  |

## 局域网基础知识

### 局域网标准

* 定义
  * 物理介质和接口规范
  * 设备在链路层的交流方式

### MAC 子层 IEEE 802.3

* 定义如何在物理线上传递帧
* 处理物理地址
* 定义网络拓扑、线路规范

#### MAC 帧

```plaintext
 +0------7-------15--------------31--------------47--------------63
 |                       Preamble (8 bytes)                      |
 +---------------------------------------------------------------+
 |      Destination MAC Address (6 bytes)        |   Source MAC  |
 +---------------------------------------------------------------+
 |      Address (6 bytes)        | Length (2)    | Data(Variable)|
 +---------------------------------------------------------------+
 |       Data (Variable)         |      FCS (4 bytes)            |
 +---------------------------------------------------------------+
```

* Preamble：同步序列，用于接收端同步时钟，`10101011`
* MAC 地址：48 位，前 24 位为厂商标识，后 24 位为序列号
* Length：数据长度，最大 1500 字节
* FCS (Frame Check Sequence)：CRC 校验

#### 局域网中的 MAC

| 局域网类型 | 物理拓扑 | 逻辑拓扑 | 备注                          |
| ----- | ---- | ---- | --------------------------- |
| 以太网   | 星形   | 总线   | 物理布线是连接到集线器/交换机，但是实际传输是广播式的 |
| 令牌环   | 星形   | 环形   | 物理布线是星形，但是实际传输依次经过每个节点      |
| FDDI  | 双环   | 环形   | 物理布线是双环，但是实际传输只用一个环         |

#### 确定性 MAC 协议

* 使用：令牌环、FDDI
* 轮询
* 工作流程
  1. 一个特殊的令牌在环中循环
  2. 某节点需要发送数据时，将令牌替换为数据帧
  3. 数据帧到达目标后继续传递，直到返回到发送者
  4. 发送者移除数据帧，并生成新的令牌供下一个节点使用
* 优点：无冲突、公平

### LLC 子层 IEEE 802.2

* 管理链路上设备的连接
* 在逻辑上区分不同协议，随后封装到帧中
* 对于数据链路层的抽象和封装，将 LLC 和 MAC 层分离
* 实现：封装数据报，增加
  * DSAP (Destination Service Access Point): 目的地址
  * SSAP (Source Service Access Point): 源地址

#### 封装

```plaintext
 |MAC|LLC|Packet|MAC|
 ​
 +---------------+
 |     网络层     |
 +---------------+
 |     LLC       |
 +---------------+
 |     MAC       |
 +---------------+
 |     物理层     |
 +---------------+
```

### 局域网传输模式

* 单播 unicast：点对点
* 多播 multicast：一对多
* 广播 broadcast：一对所有
  * 通过 MAC 地址 `FF-FF-FF-FF-FF-FF` 实现
  * 影响性能，仅在 MAC 未知或目的地为所有设备时使用

## CSMA/CD & 以太网

{% hint style="info" %}
CSMA/CD 的前身

* 纯 ALOHA：任何时候都可以发送数据，传输中若冲突则等待随机时间后重发
* Slotted ALOHA：将时间分为时隙，只在时隙开始时发送数据，传输中若冲突则等待随机时隙后重发
* 1-persist CSMA：信道空闲则发送（概率为1），否则监听信道并等待等待，传输中若冲突则等待随机时间后重发
* non-persist CSMA：信道空闲则发送（概率为1），否则等待随机时间后重试，传输中若冲突则等待随机时间后重发
* p-persist CSMA：信道空闲则发送（在该时隙发送概率为p，在下一时隙发送概率为1-p），否则等待随机时间后重试，传输中若冲突则等待随机时间后重发
{% endhint %}

### CSMA/CD 流程

1. 信道空闲则发送数据
2. 发送数据时监听信道
3. 若检测到冲突则立即停止发送，广播`JAM`信号
4. 通过重发算法决定何时重发

### 以太网

* 特点：广播式，任何设备都可以接收到数据，通过 MAC 地址区分
* 以太网中 CSMA/CD 每次停止发送都将尝试次数加一，当达到最大尝试次数后停止发送，向上层报告错误

## CSMA/CA & 无线局域网

### WLAN 无线局域网

* 蜂窝网络：将区域划分为多个蜂窝
* 短距离通信
* 标准：IEEE 802.11

### Infrastructure Mode

* BSS (Basic Service Set)：基本服务集
  * BS (Base Station)：基站，作为 AP (Access Point) 提供服务
  * 终端：同一BSS里的所有终端可直接访问
* ESS (Extended Service Set)：扩展服务集，多个 BSS 通过 DS (Distribution System) 连接

### 访问过程

* 当设备在 WLAN 中激活时，开始**扫描**兼容的能关联的设备

#### 主动扫描

* 设备主动发送请求 SSID 的信号，等待 AP 的响应
* AP 响应后，设备发送关联请求，AP 发送关联响应
* AP 无需发送自己的信息

#### 被动扫描

* 设备等待 AP 的信号，AP 发送信号后设备发送关联请求

### WLAN 帧

* 不使用 IEEE 802.3 的 MAC 帧
* 最大载荷：1500 字节，虽然 IEEE 802.11 允许更大的载荷
* 分类
  * 控制帧：管理网络连接
  * 数据帧：传输数据
  * 管理帧：管理网络连接

### CSMA/CA

* 解决问题：设备无法准确得知无线信道是否空闲，导致冲突
* 原理：在发送数据前，设备向接受站点发送控制帧，接受站点回复应答帧，周围站点可监听该帧，在一段时间内避免数据发送
* 流程
  1. A 向 B 发送 RTS 帧，A 周围的站点不发送数据，以保证 CTS 帧传输到A
  2. B 收到 RTS 帧后，向 A 发送 CTS 帧，B 周围的站点不发送数据，以保证数据帧传输到A
  3. A 收到 CTS 帧后，向 B 发送数据帧
  4. 若冲突后，等待随机时间后重试

### 实际通量的影响因素

* 回复 ACK：消耗 50% 的可用带宽
* 信号衰减：信号强度下降，导致重传，降低通量

## 第二层设备

### NIC 网卡 Network Interface Controller

* 携带唯一固定代码：MAC地址
* 提供主机对网络媒介的访问权限，控制网络上主机的数据通信
* 将计算机总线传输的**并行**信号转换成**串行**格式通过网络发送
* 把数据封装为帧

### Bridge 网桥

* 更智能的集线器
* 维护 MAC 地址表，基于 MAC 地址转发/过滤数据包
* 局域网分段：将不同网段连接在一起，减少冲突域
* 适用于低流量网络，在大流量网络中会出现瓶颈
* 透明网桥：局域网上的设备不知道帧将经过哪些网桥
  * 原理：接口`x`收到了A的帧，则从`x`可以将帧传送到A
  * 流程：网桥收到帧后将源地址和接口对应关系存入转发表中，在后续转发中进行匹配
  * 需额外记录帧到达网桥的时间，以便反应最新状况
* 源路由网桥：发送帧时包含路由信息，使沿途的网桥知道帧的路径
  * 当发送信息时，源站以广播方式发送帧，帧中包含了路由信息
  * 到达目的站后，帧原路返回源站，源站根据返回信息筛选最佳路由
  * 缺点：占用大，导致广播风暴

### Switch 交换机

* 集线器 + (多端口的)网桥：连通性 + 流量管制
* 全带宽：不用将同一信息广播到所有设备
* 直接将数据通过对应的端口发送到目的地
* 提供单独的数据路径
* 优点：减少不必要的流量、提高网络性能
* 在物理和逻辑上都是星形拓扑

#### 广播域和冲突域

* 广播域：网络中，广播帧（如 ARP 请求）能够到达的范围
* 冲突域：可能发生数据帧碰撞的范围
  * 网桥分割冲突域：软件分割冲突域，性能低下
  * 交换机分割冲突域：硬件分割冲突域，性能高
  * 路由器：需要查看路由表判断，性能不高

{% hint style="info" %}
网络设备 & 广播域 & 冲突域

* 中继器、集线器：不能分隔广播域，也不能分隔冲突域
* 交换机、网桥：不能分隔广播域，但可以分隔冲突域：交换机会转发广播帧，但为每一个端口分配一个独立的冲突域
* 路由器可以分割广播域，也可以分割冲突域
{% endhint %}
