## 什么是flink
Apache Flink is a framework and distributed processing engine for stateful computations over unbounded and bounded data streams.

### 基础处理语义
- stream
- state
- time

### 架构
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

### 惰性执行
flink程序仅在execute方法执行的时候才会生成执行计划

### flink key
key在flink中是一个很重要的概念。很多转化操作都基于key执行，比如 keyBy, groupBy。以下是key的几种定义方式：
1. Define keys for Tuples
> KeyedStream<Tuple3<Integer,String,Long>,Tuple> keyed = input.keyBy(0)

2. Define keys using Field Expressions
```java
public class WC {
    public String word;
    public int count;
  }
  DataStream<WC> words = // [...]
  DataStream<WC> wordCounts = words.keyBy("word").window(/*window specification*/);
```

3. Define keys using Key Selector Functions
```java
// some ordinary POJO
  public class WC {public String word; public int count;}
  DataStream<WC> words = // [...]
  KeyedStream<WC> keyed = words
    .keyBy(new KeySelector<WC, String>() {
       public String getKey(WC wc) { return wc.word; }
     });
```

### 定义转化逻辑
通过扩展MapFunction或者RichMapFunction并实现map方法，用户可以自定义转化逻辑。
```java
class MyMapFunction implements MapFunction<String, Integer> {
    public Integer map(String value) { return Integer.parseInt(value); }
  };
  data.map(new MyMapFunction());
```
  
### 数据类型
flink支持7大类数据类型

1. Java Tuples and Scala Case Classes
> new Tuple2<String, Integer>("hello", 1)
2. Java POJOs
```java
public class WordWithCount {
  
      public String word;
      public int count;
  
      public WordWithCount() {}
  
      public WordWithCount(String word, int count) {
          this.word = word;
          this.count = count;
      }
  }
  DataStream<WordWithCount> wordCounts = env.fromElements(
      new WordWithCount("hello", 1),
      new WordWithCount("world", 2));
```

3. Primitive Types
flink支持大多数Java，Scala的原生类型如Integer, String, and Double。前提是这些类型能够被序列化。

4. Regular Classes
对于那些非Java，Scala原生类型的对象，比如POJO，flink默认使用kryo框架进行序列化。

5. Values
values类型允许用户自定义数据的序列化逻辑，需要拓展org.apache.flinktypes.Value接口，并实现它的read和write方法。

flink提供了一些values类型，比如ByteValue, ShortValue, IntValue, LongValue, FloatValue, DoubleValue, StringValue, CharValue, BooleanValue

6. Hadoop Writables
使用该类型需要扩展org.apache.hadoop.Writable接口并实现write() 和 readFields()方法
 
7. Special Types

### 类型擦除和类型推断
Java虚拟机会擦除泛型中定义的类型，在运行时，JVM认为DataStream<String>和DataStream<Long>是一致的。

flink运行时，需要获取数据的类型信息， Flink Java API会将这些类型信息保存在数据集和操作符之中。

用户可以使用DataStream.getType()方法获取类型信息TypeInformation