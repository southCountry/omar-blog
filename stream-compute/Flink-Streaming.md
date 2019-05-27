## Flink Streaming

#### 数据源
- File-based
    - readTextFile
- Socket-based
    - socketTextStream 
- Collection-based
    - fromCollection
- Custom
    - addSourceaddSource(new FlinkKafkaConsumer08)
    
#### 数据输出
- writeAsText
- writeAsCsv
- print
- writeUsingOutputFormat
- writeToSocket 
- addSink

#### 迭代流
    IterativeStream<Integer> iteration = input.iterate();
    DataStream<Integer> iterationBody = iteration.map(/* this is executed many times */);
    iteration.closeWith(iterationBody.filter(/* one part of the stream */));
    DataStream<Integer> output = iterationBody.filter(/* some other part of the stream */);
    
#### 控制延迟
flink默认会缓存部分数据再发送，可以通过控制缓存大小来控制延迟。
    
    LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment();
    env.setBufferTimeout(timeoutMillis);

#### Watermarks
目的：用于测量事件时间的进度，以及判断窗口是否应该结束。

生成：Watermarks are generated at, or directly after, source functions.

#### 生成时间/watermark
1. Directly in the data stream source
2. Via a timestamp assigner / watermark generator: in Flink, timestamp assigners also define the watermarks to be emitted

watermark可以在source处周期生成，也可以在事件发生后立刻生成。

#### watermark生成工具
1. AscendingTimestampExtractor：当事件时间能够保证递增时，基于事件时间自动生成watermark
2. BoundedOutOfOrdernessTimestampExtractor：基于事件时间，允许一定延迟的生成方式
