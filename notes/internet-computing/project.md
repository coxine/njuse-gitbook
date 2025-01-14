# 大作业要求

## 小程序

### 总体要求

1. 3-4 个页面
2. 可以考虑使用底部导航拦
3. 后端可以自己写（Java、Python、Node 等不限），或者使用微信小程序的云开发功能。

### 参考主题

1. 天气预报
2. 课程表
3. 豆瓣读书
4. 信息查询
5. 减脂助手
6. 若选择其他主题须与教师说明。

## Socket

### 基于 Java Socket API 搭建简单的 HTTP 客户端和服务器端程序

1. 不允许基于 netty 等框架，完全基于 Java Socket API 进行编写
2. 不分区使用的 IO 模型，BIO、NIO、AIO 都可以
3. 实现基础的 HTTP 请求、响应功能，具体要求如下：
   1. HTTP 客户端可以发送请求报文、呈现响应报文（命令行和 GUI 都可以）
   2. HTTP 客户端对`301`、`302`、`304`的状态码做相应的处理
   3. HTTP服务器端支持`GET`和`POST`请求
   4. HTTP服务器端支持`200`、`301`、`302`、`304`、`404`、`405`、`500`的状态码
   5. HTTP服务器端实现长连接
   6. MIME至少支持三种类型，包含一种非文本类型
4. 基于以上的要求，实现注册，登录功能 （数据无需持久化，存在内存中即可，只需要实现注册和登录的接口，可以使用postman等方法模拟请求发送，无需客户端）。

参考资料：

1. [https://docs.oracle.com/javase/tutorial/networking/sockets/index.html](https://docs.oracle.com/javase/tutorial/networking/sockets/index.html)
2. [http://www.runoob.com/java/java-networking.html](http://www.runoob.com/java/java-networking.html)
3. [https://tools.ietf.org/html/rfc2616](https://tools.ietf.org/html/rfc2616)

### 基于 UDP 或 Raw Socket 实现一种可靠的传输层协议（STP）

1. 使用原生 Java Socket API 编写
2. 功能要求如下：
   1. 使用类似 TCP 的 handshake 机制建立和断开连接
   2. 单工，连接分发送方和接收方，接收方除必要的控制报文外不应发送数据报文，发送的数据应是字节流而非字符流
   3. 需能配置 STP 报文的最大分段大小（MSS）
   4. 传输按固定窗口方式进行，只有当窗口中每个数据包都确认传达，才可发送下一个窗口的数据包，最大窗口大小 MWS 需能配置，可逐包确认（不必如 TCP 中只确认连续数据包的最后一个可用数据包）
   5. 需模拟数据包丢失、乱序、超时、出错的情况（如根据概率随机“丢失”数据包）
   6. 需模拟确认包丢失、乱序、超时的情况

### 难度相当的其他socket应用

需要和老师确认。

## Web应用开发

需要有前端和后端，至少5个功能页面。

## 其他网络开发相关主题

需要和教师确认。
