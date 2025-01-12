# 名词解释

本文档根据往年题目，以及作者本人觉得较为重要的概念，将所有名词解释浓缩为 2 页。在 2024 年命中了 87.5% 的题目。（以往考过的5个全命中，其余新出现的3个命中了2个）

如需查看计网的所有名词解释，可参考 Spricoder 学长的[文档](https://spricoder.github.io/2020/07/05/2020-Internet-computing/2020-Internet-computing-exam2-%E5%90%8D%E8%AF%8D%E5%88%86%E6%9E%90/)。

为便于大家复习，本页面同时提供 PDF 打印版，可在[下载区](https://cos.tg/jiwang)的`Notes/22-名词解释精简版.xlsx`目录下找到。

### 往年出现过的名词解释

| ADSL    | Asymmetric Digital Subscriber Line                     | 非对称数字用户线路    | <p>用数字技术对模拟电话用户线进行改造，增加宽带功能<br>ADSL下行带宽远远大于上行带宽，因此得名“非对称”</p>     | 10                   |
| ------- | ------------------------------------------------------ | ------------ | ----------------------------------------------------------------- | -------------------- |
| ARP     | Address Resolution Protocol                            | 地址解析协议       | 把IP地址解析为MAC地址                                                     | 18 19                |
| CHAP    | Challenge Handshake Authentication Protocol            | 挑战握手认证协议     | 用于广域网PPP中的验证协议，通过三次握手和哈希函数验证链路对端身份                                | 17 18 20             |
| CIDR    | Classless Inter-Domain Routing                         | 无类域间路由选择     | 通过以/和数字表示的子网掩码划分子网                                                | 17 19 20             |
| CSMA/CA | Carrier Sense Multiple Access with Collision Avoid     | 载波侦听多路访问避免碰撞 | 在无线网络中的避免数据冲突的协议                                                  | 20                   |
| CSMA/CD | Carrier Sense Multiple Access with Collision Detection | 载波侦听多路访问碰撞检测 | 在有线网络中的避免数据冲突的协议                                                  | 11 13 15 16 17 18 24 |
| DHCP    | Dynamic Host Configuraion Protocol                     | 动态主机配置协议     | 用于动态分配IP地址的协议，局域网协议                                               | 24                   |
| DNS     | Domain Name System                                     | 域名系统         | 分布式数据库，用于映射域名到IP地址                                                | 11 13 15 16          |
| FTP     | File Transfer Protocol                                 | 文件传输协议       | 可靠的，面向连接的文件传输服务，基于20和21端口                                         | 16 17                |
| HTML    | HyperText Markup Language                              | 超文本标记语言      | 用于创建网页的标准标记语言，定义了网页内容的含义和结构                                       | 19 20                |
| HTTP    | Hypertext Transfer Protocol                            | 超文本传输协议      | 应用层协议，用于客户端和服务器之间的请求，无状态协议                                        | 15 18                |
| ICMP    | Internet Control Message Protocol                      | 因特网控制报文协议    | 网络层协议，用于网络设备之间传递查询与错误信息                                           | 16 17 18 19 20       |
| ISP     | Internet Service Providers                             | 互联网服务提供商     | 提供互联网接入及相关服务的机构，分层次结构                                             | 19 20                |
| NAT     | Network Address Translation                            | 网络地址转换       | 将网络内部的私有IP地址转换为公有IP地址以节省IP地址的方法                                   | 16                   |
| OSPF    | Open Shortest Path First                               | 开发式最短路径优先    | 基于链路状态的动态路由协议，适用于大型网络                                             | 24                   |
| PDU     | Protocol Data Unit                                     | 协议数据单元       | 对等层次之间传递的数据单位。 物理层 bit，数据链路层frame，网络层 packet，传输层 segment，更高层 data | 24                   |
| PPP     | Point-to-Point Protocol                                | 点对点协议        | 数据链路层协议，用于在广域网之中的两点传输数据，全双工，标准化协议，兼容各类设备                          | 15 16 17 18 19 20    |
| RARP    | Reverse Address Resolution Protocol                    | 反向地址解析协议     | MAC地址到IP的映射，在动态IP中使用，设备向RARP服务器请求IP地址                             | 11 13 15 17 20       |
| SMTP    | Simple Mail Transfer Protocol                          | 简单邮件传输协议     | 用于发送电子邮件，属于TCP/IP协议族                                              | 18 19 2              |
| STP     | Spanning Tree Protocol                                 | 生成树协议        | 用于在网络中建立拓扑                                                        | 11 13                |
| TDM     | Time Division Multiplexing                             | 时分复用         | 采用同一物理连接的不同时段来传输不同的信号，达到多路传输的目的                                   | 11 13 15 16 17 18    |
| UDP     | User Datagram Protocol                                 | 用户数据包协议      | 传输层协议，特点为简单不可靠，无连接，无控制                                            | 15 24                |
| URL     | Uniform Resource Locator                               | 统一资源定位符      | 用于定位网络资源，格式为协议://域名:端口/路径?参数                                      | 19 24                |

### 其他较为重要的名词解释

<table><thead><tr><th width="78">英文简称</th><th width="294">英文全称</th><th width="164">中文全称</th><th width="366">定义/功能</th></tr></thead><tbody><tr><td>ACL</td><td>Access Control Lists</td><td>访问控制列表</td><td>控制规则组，用于允许/组织路由器/交换机端口的数据包进出</td></tr><tr><td>BDR</td><td>Back-up designated router</td><td>备用指定路由器</td><td>DR的备份</td></tr><tr><td>CDM(A)</td><td>Code Division Multiplexing/Muliple Access</td><td>码分复用/多址</td><td>通过不同的编码区分原始信号，复用信道</td></tr><tr><td>DR</td><td>Designated Router</td><td>指定路由器</td><td>在OSPF协议中被选举为代表所有路由的路由</td></tr><tr><td>FDM</td><td>Frequency Division Multiplexing</td><td>频分复用</td><td>通过不同频带同时传输不同信号</td></tr><tr><td>MAC</td><td>Media Access Control</td><td>介质访问控制</td><td>MAC地址是硬件设备的物理地址，定义在物理线路上如何传输帧</td></tr><tr><td>RIP</td><td>Routing Information Protocol</td><td>路由信息协议</td><td>基于距离向量的动态路由协议，以跳数作为衡量，只适用于小型网络</td></tr><tr><td>STP</td><td>Shielded Twisted Pair</td><td>屏蔽双绞线</td><td>广泛用于数据传输的铜质双绞线</td></tr><tr><td>TCP</td><td>Transmission Control Protocol</td><td>传输控制协议</td><td>面向连接的、可靠的、基于字节流的传输层通信协议</td></tr><tr><td>UTP</td><td>Unshielded Twisted Pair</td><td>无屏蔽双绞线</td><td>无屏蔽材料的双绞线，价格低廉，安装方便</td></tr><tr><td>VLAN</td><td>Virtual Local Area Network</td><td>虚拟局域网</td><td>通过逻辑分组技术将物理局域网中的设备划分为多个虚拟网络的技术，可分割广播域，提高网络灵活性和安全性</td></tr><tr><td>VLSM</td><td>Variable Length Subnet Mask</td><td>可变长子网掩码</td><td>子网掩码长度可变，可用于进一步划分子网</td></tr><tr><td>WDM(A)</td><td>Wavelength Division Multiplexing</td><td>波分复用</td><td>光的频分复用，通过不同波长的光同时传输不同信号</td></tr></tbody></table>
