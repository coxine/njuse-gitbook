# 11-网络安全 & ACL

{% hint style="info" %}
考试范围

本章ACL部分为重点，前面网络安全部分了解即可。
{% endhint %}

## ACL 概述

* 规则列表
* 告诉路由器`permit`或`deny`某些流量
* 判断依据：源地址、目的地址、协议、端口

## 路由器匹配规则

* 查看路由表->查看ACL->匹配规则->执行动作
* 若ACL已配置，路由器顺序验证规则，当匹配到第一个规则时停止
* 在匹配最后存在隐式的`deny any`，**拒绝所有未匹配的流量**

## ACL配置

```bash
 Router(config)# access-list <number> <permit/deny> <source> <wildcard> # 创建标准ACL
 Router(config)# access-list <number> <permit/deny> <protocol> <source> <wildcard> <destination> <wildcard> <port> [log]# 创建扩展ACL
 Router(config)# <protocol> access-group <number> in/out # 应用ACL
 Router(config)# ip access-list standard <name> # 创建命名ACL
 Router# show access-list # 查看ACL
 Router# show ip interface # 查看接口应用的ACL
 Router# show running-config # 查看当前配置
```

* `number`：
  * 1-99 标准ACL：仅根据源地址匹配
  * 100-199 扩展ACL：根据源地址、目的地址、协议、端口匹配
* `protocol`：`ip`、`tcp`、`udp`、`icmp`、`http`等
* `wildcard`：和OSPF路由表的`wildcard`相同，相当于掩码取反
* `port`：可使用`eq`、`gt`、`lt`、`range`等比较符

### ACL配置技巧

* 可在最后加上`permit any`的规则，避免隐式的`deny any`
* 可在ACL中使用`host`关键字，代表单个IP地址(`access-list 1 permit 192.5.5.10 0.0.0.0`等价于`access-list 1 permit host 192.5.5.10`)
* 标准ACL应放置在**离终点最近**的接口上，以避免误伤（牺牲一点带宽）
* 扩展ACL应放置在**离源主机最近**的接口上，以避免不必要的流量进入网络
