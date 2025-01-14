# 06-应用层

## 第5层 会话层 Session Layer

* 职责
  * 建立、管理、终止会话
  * 会话同步
* Dialogue separation：将会话分离成独立的对话，便于隔离错误和并行处理
* Checkpoint：检查点，位于Dialog之间，分割Dialog
* Client/Server

### 会话层的应用

| 简称       | 全称                        |
| -------- | ------------------------- |
| RPC      | Remote Procedure Call     |
| SQL      | Structured Query Language |
| NFS      | Network File System       |
| X Window | X Window System           |

## 第6层 展示层 Presentation Layer

* 职责：数据格式转换
* 任务
  * 数据格式化：进行编码转换，如ASCII、EBCDIC、UTF-8等
  * 数据压缩
  * 数据加密

## 第7层 应用层 Application Layer

* 职责：为用户提供通信功能的接口，支持应用程序之间的交互
  * 任务：识别目标通信对象是否可用，并建立连接
  * 同步数据传输
  * 建立错误恢复流程
  * 数据完整性检查

### HTTP

* 面向事务：一次事务即为请求-响应
* 无状态：不存在上下文，事务之间无关联
* 无连接：HTTP不关心底层的连接管理，每次事务的请求都是独立的，请求完成后不保留连接状态
* 报文格式
  * 请求行：`Method URL Version\r`
  * 请求头：`Key: Value \r\n ... Key: Value \r`
  * 空行：`\r`
  * 请求体：`Data` 非必须
* 请求方法

| 方法      | 描述           |
| ------- | ------------ |
| GET     | 获取资源         |
| POST    | 提交数据         |
| PUT     | 更新资源         |
| DELETE  | 删除资源         |
| HEAD    | 获取资源的头部信息    |
| OPTIONS | 获取服务器支持的方法   |
| TRACE   | 追踪请求-响应的传输路径 |
| CONNECT | 建立连接隧道       |

### URL Uniform Resource Locator 统一资源定位符

* 语法：`scheme://host:port/path?query#fragment`

### HTML 略 详见名词解释

### FTP/TFTP

|    | FTP                        | TFTP                           |
| -- | -------------------------- | ------------------------------ |
| 全称 | File Transfer Protocol     | Trivial File Transfer Protocol |
| 连接 | 面向连接：在传输数据前双方需先在`21`建立可靠连接 | 无连接                            |
| 特点 | 可靠                         | 简单                             |
| 底层 | TCP                        | UDP                            |

1. 先在`21`端口建立控制连接
2. 等待客户请求
3. 处理客户请求，使用`20`端口传输数据
4. 传输结束后关闭`20`端口，保持`21`端口连接，继续等待客户请求

### Telnet

* 可用Telenet客户端登录装有Telnet Server的远程主机，执行命令
* 有连接

### SMTP/POP3

* SMTP：Simple Mail Transfer Protocol 发送邮件
* POP3：Post Office Protocol 3 接收邮件
* 都基于TCP/IP协议

### MIME

* Multipurpose Internet Mail Extensions 多用途互联网邮件扩展
* SMTP的拓展，用于在邮件中传输非ASCII文本和多媒体文件
* `Content-Type`：文件类型
* `Content-Transfer-Encoding`：编码方式

### SNMP Simple Network Management Protocol 简单网络管理协议

* 监控管理网络设备

### DNS Domain Name System 域名系统

* 管理域名，将域名解析为IP地址
* 分层设置，若本地DNS无法解析，向上级DNS请求解析

### 域名

* TLD Top Level Domain 顶级域名
  * nTLD 国家顶级域名：`.cn` `.us`
  * gTLD 通用顶级域名：`.com` `.org`
* 二级域名：`cos.tg`

## 常见端口

| 端口   | 服务      |
| ---- | ------- |
| `20` | FTP数据传输 |
| `21` | FTP控制   |
| `23` | Telnet  |
| `53` | DNS     |
| `80` | HTTP    |
