# 10-指令调度

* 流水线
* VLIW
* 超标量
* 乱序执行
* 通过编译器的调度优化 ILP 的性能

## 限制

* 指令顺序需要满足数据依赖关系
* True-dependency：Read After Write (RAW)
* Anti-dependency：Write After Read (WAR)&#x20;
* Output-dependency：Write After Write (WAW)
* 最后两种都可使用寄存器重命名解决
* 寄存器分配/指令调度的先后顺序不一定，因程序而异

## 局部调度：依赖图

* 节点：指令
* 边：RAW 依赖关系
* 构建依赖图后，自底向上反推从结束点到每条指令开始执行时所耗费的时间
  * 无父节点的指令可以提前任意时间执行
* 从大到小执行指令，若出现相同的时间，则根据优先级执行
  * 最长的延迟路径：深度
  * 使用最多的资源
  * 如果存在阻塞，插入`NOP` 指令

## 全局调度

* Extended Basic Block (EBB)：满足以下条件的最大数量的块的集合
  * 只有单个入口
  * 所有除入口外的块只有一个前继
* EBB path：从入口到底的路径
* 在每个EBB内交换指令顺序
* 若存在分支，需要插入额外的指令
* 下移指令时的调整：
  * `src` 没 dominate `dst`：在调整指令后需在 `dst` 其他的前继中插入 `dst` 的副本，将合并点变为 `dst` 结束
  * `dst` 没 post-dom `src`：在 `src` 的其他后继补上相同指令
  * `src` dom `dst` 且 `dst` post-dom `src`：无需额外调整
* 上移指令时的调整：
  * `dst` 没 dominate `src`：在调整指令后需在 `src` 其他的前继中不上相同指令
  * `dst` 没 post-dom `src`：在其他后继前插入 `dst` 的副本，将分叉点变为 `dst` 开始
  * `src` post-dom `dst` 且 `dst` dom `src`：无需额外调整
* 当调整完一个 EBB 后，从图中移除，再调整下一个 EBB，直到所有 EBB 都调整完毕
* 在合并点的调整：一个前继插入了指令，另一个前继没有：生成块的不同副本，加入到图中
* Trace Scheduling：Profile 最常用的路径，JIT

## 软件流水线

* 循环展开
* 可能会带来 Cache 寄存器的压力
* 需要考虑循环的迭代次数，可能会导致代码膨胀
