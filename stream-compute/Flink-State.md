## flink的状态
对于有状态的计算而言，状态的存储是十分必要的。对比storm需要开发者自己开发状态的存储和读取逻辑，flink提供了一系列接口，用于状态的保存和读取。

flink有两种类型的状态：
1. Keyed State：顾名思义，跟key相关的状态，只能用于KeyedStream
2. Operator State：节点的状态

### key group
多个key state的集合，key group的数量和流的并行度相同。

### state的存储结构
1. Raw：数据结构用户自己维护
2. Managed：flink运行时提供了多种状态存储结构，如“ValueState”, “ListState”

### 状态的TTL
当一个基于key的存储状态到期了，flink会尽最大努力清除这个状态。

默认，过期的状态，仅在状态被显示读取的时候删除。

### Broadcast State
Broadcast State是一种特殊的状态，它会被广播到下游的每个子任务。一个使用场景是，广播一个会更新的规则流到子节点，子节点存储这些规则并依据规则执行任务。

## 检查点Checkpointing
flink的失败容错特性是使用检查点机制来进行数据快照和恢复的。

### Checkpointing快照和恢复流程