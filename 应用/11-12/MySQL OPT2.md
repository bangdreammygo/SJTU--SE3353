# MySQL OPT2

### 目录：

- **OPT InnoDB Table**
- **OPT  MEMORY Table**
- **Buffering  and  Caching**

### **OPT InnoDB Table**

- **OPTIMIZE TABLE：**数据库到达一定级别后数据基本稳定之后可以进行一次（OPTIMIZE TABLE）：把一块一块的数据copy放到一起，然后重新建立索引（减少碎片的数目，速度会变快，但是这次操作会浪费时间，隔一段时间做）。
- 尽量使用**varchar**：可以压缩，但是缺点是需要一个额外的位去维持varchar里有几个；允许为空用varchar
- **char**：如果学号很长很长，大家每个人的位数都一样，使用char合理
- **事务：**执行每一条SQL语句，都默认在一个事务里，每条语句一个事务。（AUTOCOMMIT决定的）但是频繁的落盘会效率比较低，所以可以手动划分事务边界。---->但是需要避免一个大事务里修改大量的行（roolback时候系统性能极其差）如果实在避免不了，尽量少commit。同时避免长时间运行的事务，因为会上锁导致其他事务无法进行，如果要降低事务隔离级别使其可以，会出现更多问题。
- **缓冲池（buffer pool）：** 尽可能稍微大一点，我们写完之后，会先缓存，不会立马落盘，性能提高。
- **READ-Only事务：** 开始事务之前声明 START TRANSACTION ONLY，不会回滚，select无锁
- **批量数据导入InnoDB表：** 可以先关掉autocommit，不会创建海量事务，无数次提交。只需要在都结束之后提交即可。（如果插入唯一UN的数据：这么做的前提是必须确认没有重复的；如果插入有外键的，也可以这么做）插入多行可以写single SQL 用INSERT (1,2)，(3,4)....; 批量数据时自增的锁的模式是2(interleaved)不是1(consecutive).
- **查询优化：** 每个表有主键，怎么优化见OPT1.（1.聚簇索引  2.主键不要太长，不要太多）
- **磁盘IO：**
  - 把缓存池增大，事务执行NO-Steal，NO-Force（内存上来）
  - 调整落盘时的策略 O_DSYNC（文件属性）
  - 配置文件同步的阈值（填一个具体数值使其周期性的往磁盘上刷）
  - 找非旋转的存储（不用HDD，对随机的IO比较差），用SSD处理随机的IO
  - 提高IO性能避免经常做的日志。**checkpoint** ：从t1到t2的增量，到一定时间做全量一次（涉及大量的IO）。
- **优化DDL**：删除表中的所有数据，使用TRUNCATE(直接把后续的指针置为空)不要使用DELETE *(一条一条删)，但是有外键的时候要注意顺序问题，主键一旦被创建尽量不要修改了。

### **OPT  MEMORY Table**

内存里的数据放一些常用的但是不能放关键的，容易丢，默认是哈希查找，但是也可以根据需求修改成B树。

### Buffering and Caching

- #### 缓存池尺寸

  pool 由一个一个的chunk组成，每个chunk size默认是128M，所以pool size必须是chunksize 的整数倍，所以你规定9G（只是一个建议），他会开成10G。

- #### 多个缓存池的实例

  减少不同进程对cached page 的竞争访问，实例多了，每个池的大小会变小，要做一个平衡。最多可以开到64个，最小得是1G。

- #### 抵抗扫描

  我们不是使用严格的LRU策略决定缓存池里放什么东西。放到LRU一半的位置（3/8的位置），很快被替换，保证数据hot性。甚至可以分segment，分开放热度不同的数据，保证热数据。

- #### 预读

  HDD磁盘转到这里的时候，我们会乘磁盘转的时候多读一些数据到缓存池里。（线性读出来前后/随机读出）

- #### Flushing

  缓存里的东西什么时候落盘。记录所有的脏页（更改过数据的没落下去）。**有几个线程负责脏页落盘**（默认是4个）？但是不会超过线程池的实例个数。**落下去的时间**？脏页的水位线，大概是10%。

- #### 保存和恢复缓存池状态

  会在服务器关闭时，把最常用的页存在硬盘上，启动时从硬盘上直接恢复。dump的百分比可以设置。

