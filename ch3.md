# 第3章 数据存储与检索

[TOC]

# 摘要

针对事务型工作负载和针对分析型负载的存储引擎优化存在很大的差异。

日志结构的存储引擎 vs 面向页的存储引擎（如B-tree）

**内存排序**有很多树状数据结构，例如红黑树或AVL树。使用这些数据结构，可以按任意顺序插入并以排序后的顺序读取。

# 数据结构

## 日志

许多数据库内部使用日志，日志是一个仅支持追加式更新的数据文件。

日志结构提供最佳的写入性能。

日志结构索引将数据库分解为可变大小的段，通常大小为几M字节或更大，并且始终按顺序写入。

## 索引

索引是基于原始数据派生而来的额外数据结构。很多数据库允许单独添加和删除索引，而不影响数据库的内容，它只会影响查询性能。维护额外的结构势必会引入开销，特别是在新数据写入时。由于每次写数据时，都需要更新索引，因此任何类型的索引通常都会降低写的速度。

### 哈希索引

### SSTable和LSM-Tree

SSTable（排序字符串表）：k-v对的顺序按键排序。

LSM-Tree：基本思想（在后台合并一系列的SSTable）足够简单有效。即使数据集远远大于可用内存，它仍然能够正常工作。由于数据按键排序存储，因此可以有效地执行区间查询（从最小到最大扫描所有的键），并且由于磁盘是顺序写入的，所以SSL-Tree可以支持非常高的写入吞吐量。

LSM：基于合并和压缩排序文件原理的存储引擎通常被称为LSM存储引擎。

当查找数据库中某个不存在的键时，LSM-Tree算法可能很慢：在确定键不存在之前，必须先检查内存表，然后将段一直回溯访问到最旧的段文件（可能必须从磁盘多次读取）。

**布隆过滤器**是内存高效的数据结构。如果数据库中不存在某个键，它能很快告诉你结果，从而节省了很多对于不存在的键的不必要的磁盘读取。

### B-trees

最广泛使用的索引结构。

B-tree和SSTable一样，按键排序k-v对，这样可以实现高效的k-v查找和区间查找。

B-tree将数据库分解成固定大小的块或页，传统上大小为4KB（MYSQL InnoDB页大小默认16KB），页是内部读/写的最小单元。

## 其他索引结构

#### 在索引中存储值

##### 聚集索引

​	将索引行直接存储在索引中。

​	在MySQL InnoDB存储引擎中，表的主键始终是聚集索引，二级索引引用主键。

##### 非聚集索引

​	仅存储索引中的数据库的引用。

##### 覆盖索引

​	在索引中保存一些表的列值。它可以支持只通过索引即可回答某些简单查询（在这种情况下，称索引覆盖了查询）

#### 多列索引

##### 级联索引

通过将一列追加到另一列，将几个字段简单的组成一个键(索引的定义指定字段连接的顺序)

##### 多维索引

多维索引是更普遍的一次查询多列的方法，对地理空间数据尤为重要。

#### 全文搜索和模糊索引

#### 在内存中保存所有内容

Memcached 主要用于缓存，如果机器重启造成的数据丢失是可以接受的。

其他内存数据库旨在实现持久性。而Redis和Couchbase通过异步写入磁盘提供较弱的持久性。RAMCloud是一个开源的、具有持久性的内存k-v存储（对内存和磁盘上的数据使用日志结构）。

**内存数据库的性能优势并不是因为他们不需要从磁盘读取。如果有足够的内存，即使是基于磁盘的存储引擎，也可能永远不需要从磁盘读取，因为操作系统将最近使用的磁盘块缓存在内存中。相反，内存数据库可以更快，是因为他们避免使用写磁盘的格式对内存数据结构编码的开销。**



# 事务处理与分析处理

在线事务处理（online transaction processing，OLTP）

在线分析处理（online analytic processing，OLAP）

## 数据仓库

将数据导入数据仓库的过程称为提取-转换-加载（Extract-Transform-Load，ETL）

### OLTP数据库与数据仓库直接的差异

数据仓库最常见的数据模型是关系型，因为SQL通常适合分析查询。例如下钻、切片和切丁等。

### 星型与雪花型分析模式

分析型业务的数据模型，常用的有星型模式（维度建模）、雪花模式。

### 列式存储

事实表通常超过100列，但是典型的数据仓库查询往往一次只访问其中的4或者5列。在处理这样的查询的时候，面向行的存储引擎需要将所有行（每个由超过100个熟悉组成）从磁盘加载到内存中、解析 他们，并过滤出不符合所需条件的行。这可能需要很长时间。

列存储的想法很简单：不要将一行中的所有值存储在一起，而是将每列中的所有值存储在一起。如果每个列存储在一个单独文件中，查询只需要读取和解析在该查询中使用的那些列，这可以节省大量的工作。

面向列的存储布局依赖一组列文件，每个文件以相同顺序保存着数据行。因此，如果需要重新组装整行，可以从每个单独的列文件中获取第23个条目，并将它们放在一起构成表的第23行。

### 列压缩

### 内存带宽和矢量化处理

### 列存储中的排序

### 列存储的写操作



