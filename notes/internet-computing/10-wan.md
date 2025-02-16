# 10-广域网 & PPP

{% hint style="info" %}
考试范围

本章PPP部分为重点，广域网部分仅了解概念即可。
{% endhint %}

## 广域网

* 通过广域网服务提供商（WAN SP）连接LAN
* 集中在物理层和数据链路层

### 物理层

* DTE (Data Terminal Equipment)：数据终端设备 <==Modem/CSU/DSU==> DCE (Data Circuit-terminating Equipment)：数据电路终端设备
* 标准：RS-232等

### 数据链路层

* PPP (Point-to-Point Protocol) 点对点协议：在WAN中用于在两点间串行传输数据，包含用于识别网络层协议的协议字段
* HDLC (High-Level Data Link Control) 高级数据链路控制：ISO标准，抽象规范，不同供应商之间不兼容的HDLC，支持点对点/多点配置
  * 同一厂商设备可以用HDLC，不同厂商设备使用PPP（标准化）
* 帧中继 (Frame Relay)：面向连接的数据链路层协议，特点：简化的封装、不提供差错检测
* ISDN (Integrated Services Digital Network)：通过电话网络传输声音、数据（拨号上网）

## PPP

* 可检测链路质量
* 验证
* 有窗口/流量控制
* 有多种网络层协议字段
* 动态IP分配
* 压缩
* 错误检测

## PPP链路建立/维护/终止

### 建立链路

* LCP (Link Control Protocol)：链路控制协议
* 开启连接时，每一台PPP设备发送LCP配置请求
* LCP包含自定义字段
* 若配置字段不在LCP包中，使用默认值
* 当配置确认帧收到后，结束LCP阶段

### 链路质量确定

* 当配置时，定期发送LCP检测错误率
* 若有身份认证，在本步完成
* LCP可以延迟网络层协议信息的传输，直到完成此阶段（在此之前不能传输网络层数据）

### 网络层协议配置

* PPP设备发送NCP (Network Control Protocol) 配置请求
* 完成此步后可发送数据报

### 终止链路

* 可在任意时刻终止链路
* 当终止时通知网络层协议

## 帧格式

* PPP：`Flag | Address | Control | Protocol | Data | FCS | Flag`
  * Flag：始终为 `01111110`
  * Address：始终为 `11111111` 广播地址
  * Control
  * Protocol
  * Data: 0-1500字节
  * FCS：纠错码
* HDLC：`Flag | Address | Control | Proprietary | Data | FCS | Flag`
  * HDLC 有Proprietary字段，不同厂商设备之间不兼容，只有两端都是 Cisco 设备时才能使用

## PAP/CHAP

| 特性       | PAP        | CHAP                      |
| -------- | ---------- | ------------------------- |
| 认证方式     | 用户名和密码明文传输 | 挑战-应答机制，不允许发起方在没有挑战的情况下认证 |
| 握手过程     | 双向握手       | 三次握手                      |
| 密码传输安全性  | 明文传输，安全性低  | 传输加密的哈希值，安全性高             |
| 认证频率     | 仅认证一次      | 可定期重新认证                   |
| 认证失败后的行为 | 重试多次发送认证请求 | 失败后连接直接终止                 |
| 安全性      | 较低         | 较高                        |

### 握手过程

* PAP
  1. 发起方发送用户名和密码
  2. 接收方验证，回复 Accept/Reject
* CHAP
  1. 接收方发出含随机数的挑战请求
  2. 发起方使用用户名、密码、随机数计算哈希值，回复
  3. 接收方验证回复值

### 配置

```bash
 # PAP 发起方
 Router(config)# interface <interface>
 Router(config-if)# encapsulation ppp
 Router(config-if)# ppp pap sent-username <username> password <password>
 Router(config-if)# no shutdown
```

```bash
 # CHAP 发起方
 Router(config)# username <username> password <password> # 和 PAP 配置用户名密码的位置不同
 Router(config)# interface <interface>
 Router(config-if)# encapsulation ppp
 Router(config-if)# no shutdown
```

```bash
 # 接收方
 Router(config)# username <username> password <password>
 Router(config)# interface <interface>
 Router(config-if)# encapsulation ppp
 Router(config-if)# ppp authentication [pap | chap]
 Router(config-if)# no shutdown
```
