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

#### state
1. keyed state: always relative to keys and can only be used in functions and operators on a KeyedStream
2. operator state: each operator state is bound to one parallel operator instance

#### state forms
1. Managed State is represented in data structures controlled by the Flink runtime, such as internal hash tables, or RocksDB. Examples are “ValueState”, “ListState”, etc. Flink’s runtime encodes the states and writes them into the checkpoints.

2. Raw State is state that operators keep in their own data structures. When checkpointed, they only write a sequence of bytes into the checkpoint. Flink knows nothing about the state’s data structures and sees only the raw bytes.

### managed keyed state
#### state ttl
By default, expired values are only removed when they are read out explicitly, e.g. by calling ValueState.value().

### managed operator state
To use managed operator state, a stateful function can implement either the more general CheckpointedFunction interface, or the ListCheckpointed<T extends Serializable> interface.

#### Stateful Source Functions
Stateful sources require a bit more care as opposed to other operators. In order to make the updates to the state and output collection atomic (required for exactly-once semantics on failure/recovery), the user is required to get a lock from the source’s context.

### Broadcast State Pattern
Broadcast state was introduced to support use cases where some data coming from one stream is required to be broadcasted to all downstream tasks, where it is stored locally and is used to process all incoming elements on the other stream.
- There is no cross-task communication
- Order of events in Broadcast State may differ across tasks
- All tasks checkpoint their broadcast state
- No RocksDB state backend

### Checkpointing
- Enabling and Configuring Checkpointing
    - By default, checkpointing is disabled
- Selecting a State Backend
   - By default, state is kept in memory in the TaskManagers and checkpoints are stored in memory in the JobManager.
   
### Queryable State
this feature exposes Flink’s managed keyed (partitioned) state (see Working with State) to the outside world and allows the user to query a job’s state from outside Flink.

#### Architecture
1. the QueryableStateClient, which (potentially) runs outside the Flink cluster and submits the user queries,
2. the QueryableStateClientProxy, which runs on each TaskManager (i.e. inside the Flink cluster) and is responsible for receiving the client’s queries, fetching the requested state from the responsible Task Manager on his behalf, and returning it to the client, and
3. the QueryableStateServer which runs on each TaskManager and is responsible for serving the locally stored state.

### State Schema Evolution
Currently, schema evolution is supported only for POJO and Avro types. Therefore, if you care about schema evolution for state, it is currently recommended to always use either Pojo or Avro for state data types.

### Operators 