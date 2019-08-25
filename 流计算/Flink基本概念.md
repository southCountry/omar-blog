## 流计算引擎
1. Storm
2. Spark Streaming
3. Flink

### 对比
|引擎|架构|容错|处理模型与延迟|吞吐量|一致性语义|API|易用性|运维|
|---|---|---|---|---|---|---|---|---|
|Storm|主从模式，依赖ZK，处理过程中对主的依赖不大。流语义。||||||||
|Spark Streaming|架构依赖spark，主从模式，每个Batch处理都依赖主（driver），可以理解为时间维度上的sparkDAG。微批语义。||||||||
|Flink|架构介于 spark 和 storm 之间，主从结构与 spark streaming 相似，DataFlow Grpah 与 Storm 相似，数 据 流 可 以 被 表 示 为 一 个 有 向图。 每个顶点是一个用户定义的运算，每向边表示数据的流动。||||||||

## 什么是flink
Apache Flink is a framework and distributed processing engine for stateful computations over unbounded and bounded data streams.

### 基础处理语义
- stream：flink整体是以流为视角执行数据接入、转换和输出。对比storm的拓扑结构，flink在语义上理解起来更为简单。
- state：状态是flink的第一公民
- time：支持event time/processing time/ingestion time

### 多层级API
![flink application api](https://github.com/southCountry/omar-blog/raw/master/images/flink/flink-api.png)

### 架构
- 有界和无界数据流
- 部署灵活：支持standalone,yarn，k8s
- 极高伸缩性
- 极致流处理性能：本地状态存取，极致性能优化
![flink应用支持本地状态存取。flink通过本地状态持久化存储以及周期性触发异步检查点任务来保证exactly-once语义。](https://github.com/southCountry/omar-blog/raw/master/images/flink/architecture.png)

### Flink编码步骤
1. 获得执行环境
> StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
2. 定义数据输入源
> DataStream<String> text = env.readTextFile("file:///path/to/file");
3. 执行转化，处理数据
```java
 DataStream<Integer> parsed = input.map(new MapFunction<String, Integer>() {
          @Override
          public Integer map(String value) {
              return Integer.parseInt(value);
          }
      });
```
4. 定义输出目的
> writeAsText(String path)
5. 触发执行
> execute()

