## ElasticSearch是什么
- 一个分布式的实时文档存储，每个字段可以被索引与搜索
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持PB级别的结构化或者非结构化数据

## 倒排索引
- 倒排索引(Inverted Index)：通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。倒排索引主要由两个部分组成：“单词词典”和“倒排文件”。
- Term词典(Lexicon)：文档集合中出现过的所有单词构成的字符串集合，单词词典内每条索引项记载单词本身的一些信息以及指向“倒排列表”的指针。
- 倒排列表(PostingList)：记载了出现过某个单词的文档列表及单词在该文档中出现的位置信息。
- 倒排文件(Inverted File)：所有单词的倒排列表往往顺序地存储在磁盘的某个文件里，这个文件即被称之为倒排文件，倒排文件是存储倒排索引的物理文件。

### Lucene使用FST存储Term词典
![term词典存储结构](https://github.com/southCountry/omar-blog/raw/master/images/elasticsearch/term-dic.png)