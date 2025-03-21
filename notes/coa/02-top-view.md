# 02-计算机的顶层视图

## CPU（5、6、7、13、15）

* CPU的频率不能无限提高 -> 改进CPU芯片结构（14）
* 内存墙：主存和CPU之间传输数据的速度跟不上CPU的速度：采用高速缓存（9）
  * 添加一级或多级缓存以减少存储器访问频率并提高数据传输速率
  * 增大总线的数据宽度，来增加每次所能取出的位数
* CPU等待IO：采用中断机制，将中断周期加入指令周期（顺序中断处理、嵌套中断处理）（17）

## 储存器（3、8、13）

* 层次式储存结构：使用存储器层次结构 而不是依赖单个存储器组件（8、9、10、11、12）

## IO（17）

* 不同设备间差异大：设立缓冲区
* 新的接口
* 不同的IO操作技术

## 总线（16）

* 控制信号只能从CPU发出
* 控制线不能复用，数据线和地址线可以复用
* 所有的互联取消
  * 好处：提高利用率
  * 坏处：冲突，总线同一时刻只能在一对设备间传输信息
