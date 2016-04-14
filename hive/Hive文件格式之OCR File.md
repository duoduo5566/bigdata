# Hive文件格式之OCR File
标签（空格分隔）： Hive
---
    版本 0.11及以上
## Optimized Row Columnar (ORC) 
文件格式提供了一种高效的方式来存储Hive数据。它被设计于克服Hive其它文件格式的限制。底层文件使用ORC格式的话，可以提高Hive在查询，插入以及处理数据等操作时的性能。

与RC File比较，ORC File有以下优点：

 - 每个Task对应输出单个文件，从而降低了NameNode的负载
 - 支持的Hive数据类型包括datetime, decimal, 以及复合类型(struct, list, map, and union)
 - 轻量级的索引文件存储
   - 跳过行组时，不需要通过谓词过滤
   - 查找给定的行
 - 基于数据类型的块级别压缩
   - Integer 类型列的游程编码(一种与资料性质无关的无损数据压缩技术)
   - String 类型列的词典编码
 - 用多个互相独立的RecordReader并行读相同的文件
 - 无需扫描markers就可以分割文件
 - 限定需要读取或写入的内存量
 - 元数据使用 Protocol Buffer进行存储，因此允许增加或删除列操作

##文件结构

一个ORC文件包含很多行数据组，每个行数据组被称为条带(stripe)。与之跟随的是在File Footer里的辅助信息。在文件的结尾部分，有一块叫postscript的区域，它存储了压缩参数及压缩页脚的大小。

一个条带默认的大小是250M，更大尺寸的条带大小开启，将更高效的从HDFS上读取数据。

在File Footer区域中，包含了文件中的所有条带，每个条带的行数据总数，以及每一列的数据类型。它还包含列级别的一些集合结果，比如：count,min,max,sum。

下图阐述了ORC文件的文件结构：
![ORC文件结构图][1]

##条带结构

如上图所示，在ORC File中的每一个条带包含三个部分：Index data，Row data，Stripe footer。

Stripe footer包括一个流位置的目录，Row data 用于表扫描。

Index data包含每一列的最大值和最小值，以及每一列对应的行位置。行索引
目录提供了能在一个解压缩的块内查找正确的压缩块和字节的偏移量。需要注意的是，ORC Index仅被用于条带和行组的选择，并不是用来回应查询。

相对频繁的行索引使得在stripe中快速读取的过程中可以跳过很多行，尽管这个stripe的大小很大。在默认情况下，最大可以跳过10000行。拥有通过过滤谓词而跳过大量的行的能力，你可以在表的 secondary keys 进行排序，从而可以大幅减少执行时间。比如你的表的主分区是交易日期，那么你可以对次分区（state、zip code以及last name）进行排序

##HiveQL 语法
我们可以在表(分区)级别，对其文件格式进行设置。你可以通过以下HiveQL语句来指定ORC文件格式：

    CREATE TABLE ... STORED AS ORC
    ALTER TABLE ... [PARTITION partition_spec] SET FILEFORMAT ORC
    SET hive.default.fileformat=Orc

所有关于ORCFile的参数都是在HiveQL语句的TBLPROPERTIES字段里面出现，他们是：

| Key | Default | Notes |
| --- |   ---   |  ---  |
|orc.bloom.filter.columns|""|comma separated list of column names for which bloom filter should be created|
|orc.bloom.filter.fpp|0.05|false positive probability for bloom filter (must >0.0 and <1.0)|
|orc.compress|ZLIB|high level compression (one of NONE, ZLIB, SNAPPY)|
|orc.compress.size|262,144|number of bytes in each compression chunk|
|orc.create.index|true|whether to create row indexes|
|orc.row.index.stride|10,000|number of rows between index entries (must be >= 1000)|
|orc.stripe.size|268435456|number of bytes in each stripe|

举个例子，下面创建一个基于ORC存储格式的表，指定不启用压缩：

    create table Addresses (
    name string,
    street string,
    city string,
    state string,
    zip int
    ) stored as orc tblproperties ("orc.compress"="NONE");

  [1]: https://cwiki.apache.org/confluence/download/attachments/31818911/OrcFileLayout.png