# 09-VLAN

## 交换

* 交换机的基本功能
  * 维护MAC地址表
  * 转发数据帧
* 对称交换：每个端口带宽相同
  * 缺点：跨网段通信时，会出现拥塞
* 非对称交换：为服务器端口分配更多带宽
  * 交换机中需要**内存缓冲**：在阻塞时存储帧
    * 基于端口的缓冲：不会影响其他端口
    * 共享的缓冲：另外记录端口号

### 交换方法

* 存储转发：接收到整个帧后，计算校验和，再转发（延迟大，可以发现所有错误）
* 直通：仅检查目的 MAC ，不计算校验和（延迟小，无法发现错误）
* Fragment-Free：仅检查前 64 字节（可以发现大部分错误）

### 多层交换机

* 默认交换机是第二层设备，可以简单获知第三、四层信息

## 生成树协议

* 网络回路：数据在网络中循环传输
  * 原因：冗余/配置错误
  * 后果：广播风暴（以太网帧没有 TTL）
* 生成树协议：计算生成树，在有冗余道路的拓扑中防止回路

### Bridge ID

* 8 字节
  * 2 字节：优先级，0-65535，默认 32768
  * 6 字节：MAC 地址
* 根桥（根交换机）：**Bridge ID 最小**的交换机被钦定为根桥，作为生成树的根节点，其他交换机通过 BPDU 交换计算生成树

### BPDU (Bridge Protocol Data Unit) 生成树帧

* 发送涉及裁判规则的相关信息，构建生成树
* 交换机之间，不涉及用户数据

### STP 状态

| 状态         | 是否转发数据帧 | 其他动作      |
| ---------- | ------- | --------- |
| Disabled   | 否       | 未听到 BPDU  |
| Blocking   | 否       | 听到 BPDU   |
| Listening  | 否       | 监听数据帧     |
| Learning   | 否       | 学习 MAC 地址 |
| Forwarding | 是       | 学习 MAC 地址 |

### STP 端口裁判规则

1. 选择到根桥的路径开销最短的作为路径 （路径开销与带宽负相关，若有多段路程开销相加）
2. 若开销相同，发送者 Bridge ID最小的作为路径
3. 若发送者 Bridge ID 相同，发送者端口 ID 最小的作为路径

### STP 初始化

1. 网络启动后，所有网桥将 BPDU 涌入网络
2. 选举根桥：先默认根桥为自己，若收到的 BPDU Bridge ID 更低，则更新
3. 选举每个桥的根端口：根据裁判规则选择根端口（开销通过收到的而非自己的 BPDU 计算）（根桥无根端口）
4. 选举各个网段（每两台设备之间）的指定端口：根据裁判规则比较，每个链路只有一个负责收发的指定端口，其余端口被阻塞

| 带宽       | 开销  |
| -------- | --- |
| 10 Mbps  | 100 |
| 100 Mbps | 19  |
| 1 Gbps   | 4   |
| 10 Gbps  | 2   |

## VLAN (Virtual Local Area Network) 虚拟局域网

* 通过软件逻辑分割设备，**分割广播域**
* 通过VLAN ID区分不同的广播域，只有当 VLAN ID 相同时，才会转发广播帧
* 划分方式：端口、MAC 地址、协议、应用
* 同一 VLAN 内的设备可以直接通信
* 不同 VLAN 需要经过路由器

### VLAN 中的帧

* 交换器根据帧的数据决定过滤/转发
* 主要方法
  * 帧过滤：交换机共享记录 VLAN ID 和 MAC 地址的表，根据表内信息决定是否过滤
  * 帧标记：在帧头添加 VLAN ID，交换机根据 VLAN ID 过滤

### 使用 VLAN

* 静态 VLAN：手动配置
* 动态 VLAN：根据 MAC 地址自动配置
* 以端口为基础的 VLAN：所有 VLAN 节点都连接到同一路由接口，统一管理

### Access / Trunk 链路

* Native VLAN：未打标 VLAN 的帧默认所属的 VLAN
* Access：只属于**一个 VLAN**
  * 被属于的 VLAN 是 Native VLAN
  * 所有链接到该端口的设备都不知道 VLAN 的存在（发的帧不带 VLAN ID，由 Access 端口添加）
* Trunk：可以传输多个 VLAN 的数据，帧头中有 VLAN ID
  * 同样存在 Native VLAN 作为备用

### VLAN 配置

```shell
# 常规配置
Switch# vlan database
Switch(vlan)# vlan <vlan-number> # 创建 VLAN
Switch(vlan)# exit
Switch(config)# interface <interface-id>
Switch(config-if)# switchport mode access # 设置为 Access
Switch(config-if)# switchport access vlan <vlan-number> # 设置 VLAN
Switch(config-if)# switchport mode trunk # 设置为 Trunk
Switch(config-if)# switchport trunk native vlan <vlan-number> # 设置 Native VLAN
Switch# show vlan # 查看 VLAN 信息
Switch# no vlan <vlan-number> # 删除 VLAN
```

```shell
# 子接口
Router(config)# interface <interface-id>.<subinterface-id>
Router(config-subif)# encapsulation dot1q <vlan-number> # 设置 VLAN
Router(config-subif)# ip address <ip-address> <subnet-mask> # 设置 IP 地址
```
