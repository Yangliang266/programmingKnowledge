**<span style="color:red">索引是DBMS(数据库管理系统)的数据结构</span>**



>  mysql 索引 使用的是b+tree 数据结构



#### B+tree 存在意义

1. B-Tree是为磁盘等外存储设备设计的一种平衡查找树。因此在讲B-Tree之前先了解下磁盘的相关知识。

2. 系统从磁盘读取数据到内存时是以磁盘块（block）为基本单位的，位于同一个磁盘块中的数据会被一次性读取出来，而不是需要什么取什么。

3. InnoDB存储引擎中有页（Page）的概念，页是其磁盘管理的最小单位。InnoDB存储引擎中默认每个页的大小为16KB，可通过参数innodb_page_size将页的大小设置为4K、8K、16K，在[MySQL](http://lib.csdn.net/base/mysql)中可通过如下命令查看页的大小：

   ```text
   mysql> show variables like 'innodb_page_size';
   ```



#### 索引的本质

1. 底层数据结构 b+tree：多路平衡查找树



#### 索引的特点

<img src="https://raw.githubusercontent.com/YangLiang-SoftWise/images/master/img/索引1.png" alt="img" style="zoom:80%;" />

1 根节点与支节点不存储数据，存放关键字与引用

2 多路平衡，即主节点的主键数与连接数一致

3 子节点存放数据，并且采用链表的形式进行关联前后数据，指针指向，有序

4 子节点遵循左闭右开原则



**分裂与合并**



**1. 文件表（File-Table）结构**

假设你已经装好了MySQL最新的5.7版本（译注：文章发布于17年4月），并且你创建了一个`windmills`库（schema）和`wmills`表。在文件目录（通常是`/var/lib/mysql/`）你会看到以下内容：

```text
data/
  windmills/
      wmills.ibd
      wmills.frm
```

这是因为从MySQL 5.6版本开始`innodb_file_per_table`参数默认设置为1。该配置下你的每一个表都会单独作为一个文件存储（如果有分区也可能有多个文件）。

目录下要注意的是这个叫`wmills.ibd`的文件。这个文件由多个段（segments）组成，每个段和一个索引相关。

文件的结构是不会随着数据行的删除而变化的，但段则会跟着构成它的更小一级单位——区的变化而变化。区仅存在于段内，并且每个区都是固定的1MB大小（页体积默认的情况下）。页则是区的下一级构成单位，默认体积为16KB。

按这样算，一个区可以容纳最多64个页，一个页可以容纳2-N个行。行的数量取决于它的大小，由你的表结构定义。InnoDB要求页至少要有两个行，因此可以算出行的大小最多为8000 bytes。

听起来就像俄罗斯娃娃（Matryoshka dolls）一样是么，没错！下面这张图能帮助你理解：

<img src="https://pic1.zhimg.com/v2-796e3785a939b3bfd27a942374d22f72_b.jpg" alt="img" style="zoom:80%;" />

**2. 根，分支与叶子**

每个页（逻辑上讲即叶子节点）是包含了2-N行数据，根据主键排列。树有着特殊的页区管理不同的分支，即内部节点（INodes）。

<img src="https://pic1.zhimg.com/v2-27e0fedd78db7bb25bec80b4429b02a3_b.jpg" alt="img" style="zoom:50%;" />

上图仅为示例，后文才是真实的结构描述。

具体来看一下：

```mysql
ROOT NODE #3: 4 records, 68 bytes
 NODE POINTER RECORD ≥ (id=2) → #197
 INTERNAL NODE #197: 464 records, 7888 bytes
 NODE POINTER RECORD ≥ (id=2) → #5
 LEAF NODE #5: 57 records, 7524 bytes
 RECORD: (id=2) → (uuid="884e471c-0e82-11e7-8bf6-08002734ed50", millid=139, kwatts_s=1956, date="2017-05-01", location="For beauty's pattern to succeeding men.Yet do thy", active=1, time="2017-03-21 22:05:45", strrecordtype="Wit")
```

下面是表结构：

   ```mysql
CREATE TABLE `wmills` (
     `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `uuid` char(36) COLLATE utf8_bin NOT NULL,
     `millid` smallint(6) NOT NULL,
  `kwatts_s` int(11) NOT NULL,
     `date` date NOT NULL,
  `location` varchar(50) COLLATE utf8_bin DEFAULT NULL,
     `active` tinyint(2) NOT NULL DEFAULT '1',
  `time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
     `strrecordtype` char(3) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (`id`),
     KEY `IDX_millid` (`millid`)
) ENGINE=InnoDB;
   ```

   所有的B树都有着一个入口，也就是根节点，在上图中#3就是根节点。根节点（页）包含了如索引ID、INodes数量等信息。INode页包含了关于页本身的信息、值的范围等。最后还有叶子节点，也就是我们数据实际所在的位置。在示例中，我们可以看到叶子节点#5有57行记录，共7524 bytes。在这行信息后是具体的记录，可以看到数据行内容。

   这里想引出的概念是当你使用InnoDB管理表和行，InnoDB会将他们会以分支、页和记录的形式组织起来。InnoDB不是按行的来操作的，它可操作的最小粒度是页，页加载进内存后才会通过扫描页来获取行/记录。

现在页的结构清楚了吗？好，我们继续。

**3. 页的内部原理**

页可以空或者填充满（100%），行记录会按照主键顺序来排列。例如在使用`AUTO_INCREMENT`时，你会有顺序的ID 1、2、3、4等。

![img](https://picb.zhimg.com/80/v2-3f061ac8f9efb6e1652f094a5983ddaa_720w.jpg)

页还有另一个重要的属性：`MERGE_THRESHOLD`。该参数的默认值是50%页的大小，它在InnoDB的合并操作中扮演了很重要的角色。

   ![img](https://pic4.zhimg.com/80/v2-6daba443c0b19c9956612fc3816dbd01_720w.jpg)

当你插入数据时，如果数据（大小）能够放的进页中的话，那他们是按顺序将页填满的。

若当前页满，则下一行记录会被插入下一页（NEXT）中。

![img](https://pic3.zhimg.com/80/v2-f3497d7e821abdfd51797a80bf0e68c2_720w.jpg)

根据B树的特性，它可以自顶向下遍历，但也可以在各叶子节点水平遍历。因为每个叶子节点都有着一个指向包含下一条（顺序）记录的页的指针。

例如，页#5有指向页#6的指针，页#6有指向前一页（#5）的指针和后一页（#7）的指针。

这种机制下可以做到快速的顺序扫描（如范围扫描）。之前提到过，这就是当你基于自增主键进行插入的情况。但如果你不仅插入还进行删除呢？

**4. 页合并**

   当你删了一行记录时，实际上记录并没有被物理删除，记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。

   ![img](https://pic2.zhimg.com/80/v2-6a9fd05c70648eabfb153f9c38b081d4_720w.jpg)

   当页中删除的记录达到`MERGE_THRESHOLD`（默认页体积的50%），InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。

   ![img](https://pic3.zhimg.com/80/v2-f3497d7e821abdfd51797a80bf0e68c2_720w.jpg)

   在示例中，页#6使用了不到一半的空间，页#5又有足够的删除数量，现在同样处于50%使用以下。从InnoDB的角度来看，它们能够进行合并。

   ![img](https://pic1.zhimg.com/80/v2-ed174fc87f4037da0ed81cd1268653b5_720w.jpg)

   合并操作使得页#5保留它之前的数据，并且容纳来自页#6的数据。页#6变成一个空页，可以接纳新数据。

   ![img](https://pic1.zhimg.com/80/v2-345d5647e15879a7342895d07fe1906e_720w.jpg)

   如果我们在UPDATE操作中让页中数据体积达到类似的阈值点，InnoDB也会进行一样的操作。

   规则就是：页合并发生在删除或更新操作中，关联到当前页的相邻页。如果页合并成功，在`INFOMATION_SCHEMA.INNODB_METRICS`中的`index_page_merge_successful`将会增加。

   **5. 页分裂**

   前面提到，页可能填充至100%，在页填满了之后，下一页会继续接管新的记录。但如果有下面这种情况呢？

   ![img](https://pic3.zhimg.com/80/v2-750b0a4f535435653c13cdcb0c853d06_720w.jpg)

   页#10没有足够空间去容纳新（或更新）的记录。根据“下一页”的逻辑，记录应该由页#11负责。然而：

   ![img](https://pic2.zhimg.com/80/v2-575acc29a4e0312db70a515ab71d6d90_720w.jpg)

   页#11也同样满了，数据也不可能不按顺序地插入。怎么办？

   还记得之前说的链表吗（译注：指B+树的每一层都是双向链表）？页#10有指向页#9和页#11的指针。

   InnoDB的做法是（简化版）：

   1. 创建新页
   2. 判断当前页（页#10）可以从哪里进行分裂（记录行层面）
   3. 移动记录行
   4. 重新定义页之间的关系

   ![img](https://picb.zhimg.com/80/v2-eb81b65c29711b609e2076af48c17146_720w.jpg)

   新的页#12被创建：

   ![img](https://pic2.zhimg.com/80/v2-0d05a58b7f1f1856894985e015db49c0_720w.jpg)

   页#11保持原样，只有页之间的关系发生了改变：

   - 页#10相邻的前一页为页#9，后一页为页#12
   - 页#12相邻的前一页为页#10，后一页为页#11
   - 页#11相邻的前一页为页#10，后一页为页#13

   （译注：页#13可能本来就有，这里意思为页#10与页#11之间插入了页#12）

   这样B树水平方向的一致性仍然满足，因为满足原定的顺序排列逻辑。然而从物理存储上讲页是乱序的，而且大概率会落到不同的区。

   规律总结：页分裂会发生在插入或更新，并且造成页的错位（dislocation，落入不同的区）

   InnoDB用`INFORMATION_SCHEMA.INNODB_METRICS`表来跟踪页的分裂数。可以查看其中的`index_page_splits`和`index_page_reorg_attempts/successful`统计。

   一旦创建分裂的页，唯一（译注：实则仍有其他方法，见下文）将原先顺序恢复的办法就是新分裂出来的页因为低于合并阈值（merge threshold）被删掉。这时候InnoDB用页合并将数据合并回来。

   另一种方式就是用`OPTIMIZE`重新整理表。这可能是个很重量级和耗时的过程，但可能是唯一将大量分布在不同区的页理顺的方法。

   另一方面，要记住在合并和分裂的过程，InnoDB会在索引树上加写锁（x-latch）。在操作频繁的系统中这可能会是个隐患。它可能会导致索引的锁争用（index latch contention）。如果表中没有合并和分裂（也就是写操作）的操作，称为“乐观”更新，只需要使用读锁（S）。带有合并也分裂操作则称为“悲观”更新，使用写锁（X）。



#### 索引的优点



1. 扫库扫表能力强

   磁盘块（页）之间是通过双向链表联系，数据之间使用单向链表链接

2. 磁盘读写强

   io能读取，page中存储的磁盘块更多

3. 排序能力，查询能力更强

   b+tree非叶子节点是不存储数据的，仅仅存储键值，如果不存储数据，那么就会存储更多的键值，相应的树的阶数（节点的子节点树）就会更大，树就会更矮更胖，如此一来我们查找数据进行磁盘的 IO 次数又会再次减少，数据查询的效率也会更快。采用链表的形式进行关联前后数据，指针指向，有序

4. 查询更稳定





#### 主键索引与辅助索引

**innoDB**

1.1 辅助索引通过找到主键值与数据引用进而找到对应数据

<img src="https://raw.githubusercontent.com/YangLiang-SoftWise/images/master/img/索引8.png" alt="img" style="zoom:80%;" />

<img src="https://raw.githubusercontent.com/YangLiang-SoftWise/images/master/img/索引7.png" alt="img" style="zoom:80%;" />







#### 索引使用原则

1 列的离散度：列不同的值越多，离散越高，反之越低

2 联合索引最左匹配原则  like abc% ok

3 覆盖原则 辅助索引 - 主键

​    主键值 - 数据

 查找列在包含的索引内

select * 用不到索引 不包含索引









