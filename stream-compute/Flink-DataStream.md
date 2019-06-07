### 时间
时间是实时计算中绝大多数操作的基础。因此，对于时间的定义变得十分重要。

flink提供三种语义的时间。

- event time 事件发生的时间，最经常使用
- processing time 引擎对事件执行操作的时间，需要使用系统时间作为判断基准时使用，比如需要统计系统每小时的吞吐量
- Ingestion time 事件流入flink系统的时间，使用较少

![time](https://github.com/southCountry/omar-blog/raw/master/images/flink/time.png)

指定时间类型操作如下：
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);
```

### watermark
watermark是flink用来统计事件流进度的一个工具，其自身也是事件流的一部分并且携带一个时间戳。比如watermark(20)，标识20之前的事件已经到达。

watermark对乱序的事件流来说十分重要，通过它，可以来决定一个时间窗口是否可以结束。

watermark一般和事件一同生成或者紧接着事件发生之后生成。

![watermark](https://github.com/southCountry/omar-blog/raw/master/images/flink/watermark.png)

对于并行流，每个subtask各自独立地生成watermark

当watermark流经某个operator的时候，它会更新该operator的event time，或者可以理解为，它为推进当前operator的时间。这样也就保证了任务流在向前推进。
![parallel-watermark](https://github.com/southCountry/omar-blog/raw/master/images/flink/parallel-watermark.png)

### 生成时间戳和watermark
有两种方式可以用于生成事件时间和watermark
1. 直接在数据流中定义时间
```java
@Override
public void run(SourceContext<MyType> ctx) throws Exception {
	while (/* condition */) {
		MyType next = getNext();
		ctx.collectWithTimestamp(next, next.getEventTimestamp());

		if (next.hasWatermarkTime()) {
			ctx.emitWatermark(new Watermark(next.getWatermarkTime()));
		}
	}
}
```
2.1 通过一个外部的时间生成器
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

DataStream<MyEvent> stream = env.readFile(
        myFormat, myFilePath, FileProcessingMode.PROCESS_CONTINUOUSLY, 100,
        FilePathFilter.createDefaultFilter(), typeInfo);

DataStream<MyEvent> withTimestampsAndWatermarks = stream
        .filter( event -> event.severity() == WARNING )
        .assignTimestampsAndWatermarks(new MyTimestampsAndWatermarks());
```
2.2 周期地生成时间和watermark
```java
/**
 * This generator generates watermarks assuming that elements arrive out of order,
 * but only to a certain degree. The latest elements for a certain timestamp t will arrive
 * at most n milliseconds after the earliest elements for timestamp t.
 */
public class BoundedOutOfOrdernessGenerator implements AssignerWithPeriodicWatermarks<MyEvent> {

    private final long maxOutOfOrderness = 3500; // 3.5 seconds

    private long currentMaxTimestamp;

    @Override
    public long extractTimestamp(MyEvent element, long previousElementTimestamp) {
        long timestamp = element.getCreationTime();
        currentMaxTimestamp = Math.max(timestamp, currentMaxTimestamp);
        return timestamp;
    }

    @Override
    public Watermark getCurrentWatermark() {
        // return the watermark as current highest timestamp minus the out-of-orderness bound
        return new Watermark(currentMaxTimestamp - maxOutOfOrderness);
    }
}

/**
 * This generator generates watermarks that are lagging behind processing time by a fixed amount.
 * It assumes that elements arrive in Flink after a bounded delay.
 */
public class TimeLagWatermarkGenerator implements AssignerWithPeriodicWatermarks<MyEvent> {

	private final long maxTimeLag = 5000; // 5 seconds

	@Override
	public long extractTimestamp(MyEvent element, long previousElementTimestamp) {
		return element.getCreationTime();
	}

	@Override
	public Watermark getCurrentWatermark() {
		// return the watermark as current time minus the maximum time lag
		return new Watermark(System.currentTimeMillis() - maxTimeLag);
	}
}
```

2.3 某一特定时间下生成watermark
```java
public class PunctuatedAssigner implements AssignerWithPunctuatedWatermarks<MyEvent> {

	@Override
	public long extractTimestamp(MyEvent element, long previousElementTimestamp) {
		return element.getCreationTime();
	}

	@Override
	public Watermark checkAndGetNextWatermark(MyEvent lastElement, long extractedTimestamp) {
		return lastElement.hasWatermarkMarker() ? new Watermark(extractedTimestamp) : null;
	}
}
```

#### 预定义的两种时间戳和watermark生成器
1. 如果能够保证每个事件流的时间是自然增加的，那么可以直接使用事件时间作为watermark
2. 如果事件流中存在部分乱序，需要定义一个可以容忍的延迟时间来确定watermark的生成，如下
```java
DataStream<MyEvent> withTimestampsAndWatermarks =
    stream.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<MyEvent>(Time.seconds(10)) {

        @Override
        public long extractTimestamp(MyEvent element) {
            return element.getCreationTime();
        }
});
```


