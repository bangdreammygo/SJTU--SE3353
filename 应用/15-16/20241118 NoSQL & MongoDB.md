## **20241118 NoSQL & MongoDB**

---

### **1. 关系型数据库（如 MySQL）的局限性**
- **数据量限制**：
  - 当数据量超过 1000 万条记录时，性能会急剧下降。
  - 原因是关系型数据库使用 B 树索引，随机 I/O 操作会导致性能瓶颈。
  - 即使通过分区（Partition）将数据分布到不同存储位置，也无法彻底解决问题。
  - 例如，MySQL 的分区功能可以将数据按范围或哈希分布到不同表中，但查询时仍需要扫描多个分区，性能提升有限。
- **结构化数据限制**：
  - 关系型数据库只能处理结构化数据（如固定字段的表）。
  - 无法有效处理非结构化数据（如文本、图片、音频、视频等）。
  - 例如，搜索引擎中的大段文本无法拆解成固定字段的表结构。
  - 如果强行将非结构化数据存储到关系型数据库中，会导致表结构复杂、查询效率低下。
- **扩展性问题**：
  - 关系型数据库难以线性扩展，无法通过增加服务器数量来线性提升性能。
  - 例如，一台服务器能处理 100 个并发用户，但两台服务器未必能处理 200 个。
  - 这是因为关系型数据库的 ACID 特性（原子性、一致性、隔离性、持久性）限制了其扩展性。
- **锁机制**：
  - 关系型数据库的锁机制在高并发场景下会成为性能瓶颈。
  - 例如，频繁的写操作会导致锁竞争，降低性能。
  - 即使使用行级锁或乐观锁，也无法完全避免锁冲突。

---

### **2. 非关系型数据库（NoSQL）的优势**
- **处理非结构化数据**：
  - NoSQL 数据库（如 MongoDB）可以处理非结构化或半结构化数据（如 JSON 格式的文档）。
  - 例如，搜索引擎中的大段文本可以直接存储为文档，无需拆解成固定字段。
  - 这种灵活性使得 NoSQL 数据库在处理复杂数据结构时更具优势。
- **高扩展性**：
  - NoSQL 数据库支持分布式存储，能够线性扩展。
  - 例如，通过分片（Sharding）将数据分布到多台服务器上。
  - 这种设计使得 NoSQL 数据库能够轻松应对数据量的增长。
- **弱化锁机制**：
  - 由于数据通常是只写一次、多次读取，锁机制被弱化，性能更高。
  - 例如，日志系统中的数据写入后很少修改，锁竞争减少。
  - 这种设计使得 NoSQL 数据库在高并发场景下表现更佳。
- **动态 Schema**：
  - NoSQL 数据库支持动态 Schema，允许同一集合（Collection）中的文档具有不同的结构。
  - 例如，一个集合中的文档可以有不同的字段。
  - 这种灵活性使得 NoSQL 数据库在处理数据结构频繁变化的场景时更具优势。

---

### **3. MongoDB 的特点**
- **文档型数据库**：
  - MongoDB 是一种文档型数据库，数据以 JSON 格式存储。
  - 例如，一个文档可以包含多个键值对（Key-Value）。
  - 这种存储方式使得 MongoDB 在处理复杂数据结构时更加灵活。
- **高性能**：
  - 支持复杂索引（如 2D 地理空间索引），查询性能高。
  - 例如，可以通过地理空间索引快速查找附近的点。
  - 这种高性能使得 MongoDB 在实时查询场景中表现优异。
- **高可用性**：
  - 通过副本集（Replica Set）实现高可用性。
  - 例如，主节点负责写操作，从节点负责读操作。
  - 这种设计使得 MongoDB 在主节点故障时能够快速切换到从节点，保证系统的高可用性。
- **自动分片（Auto-Sharding）**：
  - 支持自动分片，将数据分布到多台服务器上，实现负载均衡。
  - 例如，数据按 ID 范围分布到不同的分片服务器上。
  - 这种设计使得 MongoDB 能够轻松应对数据量的增长。
- **灵活查询**：
  - 支持丰富的查询语言，支持聚合操作（如 MapReduce、Pipeline）。
  - 例如，可以通过聚合管道对数据进行分组、排序、统计。
  - 这种灵活性使得 MongoDB 在处理复杂查询时更加高效。

---

### **4. MongoDB 的核心概念**
- **文档（Document）**：
  - MongoDB 中的基本数据单元，类似于关系型数据库中的一行记录，存储为 JSON 格式。
  - 例如，一个文档可以包含多个键值对（Key-Value）。
  - 文档的结构可以动态变化，无需预先定义 Schema。
- **集合（Collection）**：
  - 类似于关系型数据库中的表，但集合中的文档可以有不同的结构（Schema-Free）。
  - 例如，一个集合中的文档可以有不同的字段。
  - 这种设计使得 MongoDB 在处理数据结构频繁变化的场景时更加灵活。
- **数据库（Database）**：
  - 多个集合的集合，用于组织数据。
  - 例如，一个数据库可以包含多个集合。
  - 这种设计使得 MongoDB 能够更好地组织和管理数据。
- **主键（_id）**：
  - 每个文档都有一个唯一的主键（_id），默认自动生成。
  - 例如，主键用于唯一标识一个文档。
  - 这种设计使得 MongoDB 能够快速定位和查询文档。

---

### **5. MongoDB 的索引**
- **默认索引**：
  - 每个集合默认在 _id 字段上创建索引。
  - 例如，主键索引用于快速查找文档。
  - 这种设计使得 MongoDB 在查询主键时能够快速定位文档。
- **复杂索引**：
  - 支持 2D 地理空间索引，用于处理地理位置相关的查询。
  - 例如，可以通过地理空间索引查找附近的点。
  - 这种设计使得 MongoDB 在处理地理位置相关的查询时更加高效。
- **索引性能**：
  - 索引可以显著提升查询性能，但会增加写入开销。
  - 例如，频繁的写操作会导致索引更新，降低性能。
  - 这种设计使得 MongoDB 在写入频繁的场景中需要权衡索引的使用。

---

### **6. MongoDB 的分片与集群**
- **分片（Sharding）**：
  - 将数据分布到多台服务器上，支持水平扩展。
  - 例如，数据按 ID 范围分布到不同的分片服务器上。
  - 这种设计使得 MongoDB 能够轻松应对数据量的增长。
- **副本集（Replica Set）**：
  - 通过主从复制实现高可用性，主节点负责写操作，从节点负责读操作。
  - 例如，主节点故障时，从节点可以接管写操作。
  - 这种设计使得 MongoDB 在主节点故障时能够快速切换到从节点，保证系统的高可用性。
- **Config Server**：
  - 存储分片信息，用于定位数据所在的服务器。
  - 例如，查询请求会先到 Config Server 确定数据位置。
  - 这种设计使得 MongoDB 在分布式环境中能够快速定位数据。

---

### **7. MongoDB 的查询与操作**
- **插入数据**：
  - 使用 `insertOne` 或 `insertMany` 插入文档。
  - 例如，插入一个包含多个键值对的文档。
  - 这种操作使得 MongoDB 能够快速插入数据。
- **查询数据**：
  - 使用 `find` 方法查询文档，支持条件过滤（如 `$gt`、`$lt`、`$in` 等）。
  - 例如，查找评分大于 8 的电影。
  - 这种操作使得 MongoDB 能够快速查询数据。
- **更新数据**：
  - 使用 `updateOne` 或 `updateMany` 更新文档，支持 `$set` 操作符。
  - 例如，更新电影的评分。
  - 这种操作使得 MongoDB 能够快速更新数据。
- **删除数据**：
  - 使用 `deleteOne` 或 `deleteMany` 删除文档。
  - 例如，删除评分低于 5 的电影。
  - 这种操作使得 MongoDB 能够快速删除数据。
- **聚合操作**：
  - 使用聚合管道（Aggregation Pipeline）进行复杂的数据处理，如分组、排序、统计等。
  - 例如，统计电影的平均评分并按评分降序排列。
  - 这种操作使得 MongoDB 能够高效处理复杂查询。

---

### **8. MongoDB 的应用场景**
- **大数据存储**：
  - 适合存储海量非结构化数据。
  - 例如，日志系统、搜索引擎。
  - 这种场景下，MongoDB 的高扩展性和高性能表现优异。
- **高并发读写**：
  - 适合高并发场景，如实时分析系统。
  - 例如，电商网站的订单系统。
  - 这种场景下，MongoDB 的高可用性和弱化锁机制表现优异。
- **动态 Schema**：
  - 适合数据结构频繁变化的场景。
  - 例如，社交媒体的用户数据。
  - 这种场景下，MongoDB 的动态 Schema 表现优异。

---

### **9. MongoDB 与其他 NoSQL 数据库的比较**
- **MongoDB vs Redis**：
  - Redis 是内存数据库，适合缓存和实时数据处理。
  - MongoDB 是文档数据库，适合存储和查询复杂数据。
  - 例如，Redis 适合存储会话数据，MongoDB 适合存储用户数据。
- **MongoDB vs HBase**：
  - HBase 是列式数据库，适合大规模分布式存储。
  - MongoDB 更适合灵活的文档存储。
  - 例如，HBase 适合存储日志数据，MongoDB 适合存储商品数据。

---

### **10. MongoDB 的安装与使用**
- **安装**：
  - 可以通过包管理工具（如 apt、yum）或官方安装包安装。
  - 例如，使用 `apt-get install mongodb` 安装。
  - 这种安装方式使得 MongoDB 能够快速部署。
- **客户端工具**：
  - 可以使用 MongoDB Compass（图形化工具）或 mongo shell（命令行工具）进行操作。
  - 例如，使用 `mongo` 命令进入命令行工具。
  - 这种工具使得 MongoDB 的操作更加便捷。
- **基本命令**：
  - 切换数据库：`use <database_name>`
  - 插入文档：`db.collection.insertOne({...})`
  - 查询文档：`db.collection.find({...})`
  - 更新文档：`db.collection.updateOne({...}, {$set: {...}})`
  - 删除文档：`db.collection.deleteOne({...})`
  - 这些命令使得 MongoDB 的操作更加直观。

---

### **11. MongoDB 的聚合操作**
- **聚合管道**：
  - 通过多个阶段（Stage）对数据进行处理，如 `$match`（过滤）、`$group`（分组）、`$sort`（排序）等。
  - 例如，统计电影的平均评分并按评分降序排列：
    ```javascript
    db.movies.aggregate([
      { $unwind: "$genres" },
      { $group: { _id: "$genres", avgRating: { $avg: "$rating" } } },
      { $sort: { avgRating: -1 } }
    ]);
    ```
  - 这种操作使得 MongoDB 能够高效处理复杂查询。

---

### **12. MongoDB 的适用场景与局限性**
- **适用场景**：
  - 非结构化数据存储。
  - 高并发读写场景。
  - 数据结构频繁变化的场景。
- **局限性**：
  - 不支持复杂的事务处理（如多文档事务）。
  - 不适合强一致性要求的场景。

---

### **13. MongoDB Compass 的使用**
- **打开 MongoDB Compass**：
  - MongoDB Compass 是 MongoDB 官方提供的图形化管理工具，支持直观的数据操作。
  - 在 Compass 中，可以直接输入查询语句（如 `{ "title": "Inception" }`），按回车后，符合条件的电影数据会显示在界面中。
  - 这种方式与在 MongoDB Shell 中执行查询的效果一致，但 Compass 提供了更友好的可视化界面。
  - Compass 还支持数据导入、导出、索引管理、性能分析等功能。
- **数据集来源**：
  - 数据集来自 GitHub 上的一个开源项目，文件名为 `movies.json`，大小约为几百兆。
  - 该文件包含大量电影信息，如电影标题、类型、评分、演员等。
  - 通过 MongoDB Compass 的 `Import Data` 功能，可以将 `movies.json` 文件导入到 MongoDB 的某个集合中。
  - 导入时，Compass 会自动解析 JSON 文件的结构，并生成对应的集合和文档。
  - 导入完成后，可以在 Compass 中查看集合的结构和数据。

---

### **14. 聚合操作（Aggregation）**
- **聚合管道（Aggregation Pipeline）**：
  - 聚合管道是 MongoDB 中用于处理数据的强大工具，由多个阶段（Stage）组成，每个阶段对数据进行处理。
  - 例如，`$project` 阶段类似于 SQL 中的 `SELECT`，用于选择特定字段。
  - 在 `$project` 阶段，可以通过设置字段值为 `1` 或 `0` 来选择或排除字段。例如：
    ```json
    { "$project": { "title": 1, "rating": 1, "_id": 0 } }
    ```
    表示只选择 `title` 和 `rating` 字段，并排除 `_id` 字段。
- **展开数组（$unwind）**：
  - 使用 `$unwind` 阶段展开数组字段。例如，电影的类型（genres）是一个数组，展开后每条记录对应一个类型。
  - 示例：
    ```json
    { "$unwind": "$genres" }
    ```
    展开后，如果一部电影有多个类型（如动作、科幻），则会生成多条记录，每条记录对应一个类型。
- **分组与平均值（$group 和 $avg）**：
  - 使用 `$group` 阶段按类型分组，并计算每个类型的平均评分。
  - 示例：
    ```json
    { "$group": { "_id": "$genres", "avgRating": { "$avg": "$rating" } } }
    ```
    表示按 `genres` 字段分组，并计算每个类型的平均评分。
- **排序（$sort）**：
  - 使用 `$sort` 阶段按平均评分降序排列结果。
  - 示例：
    ```json
    { "$sort": { "avgRating": -1 } }
    ```
    表示按 `avgRating` 字段降序排列，评分最高的类型排在最前面。
- **保存聚合管道**：
  - 可以将聚合管道保存为模板，方便后续重复使用。
  - 例如，保存为 `sample` 模板后，可以直接运行该模板获取结果，无需重新编写聚合管道。

---

### **15. MongoDB 与 Spring Data 集成**
- **简单示例**：
  - 定义一个 `Person` 类，使用 `@Document` 注解映射到 MongoDB 的集合。
  - 示例代码：
    ```java
    @Document(collection = "people")
    public class Person {
        @Id
        private String id;
        private String firstName;
        private String lastName;
        // Getters and Setters
    }
    ```
  - 通过 `MongoRepository` 实现增删改查操作。
  - 示例代码：
    ```java
    public interface PersonRepository extends MongoRepository<Person, String> {
        List<Person> findByFirstName(String firstName);
        List<Person> findByLastName(String lastName);
    }
    ```
  - 插入曹操、刘备、孙权三个人的数据，并通过 `findByFirstName` 和 `findByLastName` 查询。
- **复杂示例**：
  - 数据存储在 MySQL 和 MongoDB 两个数据库中。
  - MySQL 存储人物基本信息（如姓名、年龄），MongoDB 存储人物图片（Base64 编码）。
  - 通过 Spring Data 将两个数据源的数据合并返回。
  - 示例代码：
    ```java
    public PersonDTO getPersonInfo(String id) {
        PersonInfo info = mysqlRepository.findById(id); // 从 MySQL 获取基本信息
        String image = mongoRepository.findImageById(id); // 从 MongoDB 获取图片
        return new PersonDTO(info, image); // 合并返回
    }
    ```
  - 例如，查询曹操的信息时，从 MySQL 获取姓名和年龄，从 MongoDB 获取图片。

---

### **16. MongoDB 的索引**
- **单列索引**：
  - 在单个字段上创建索引，支持升序或降序。
  - 示例：
    ```json
    db.collection.createIndex({ "age": 1 })
    ```
    表示在 `age` 字段上创建升序索引。
- **复合索引**：
  - 在多个字段上创建索引，支持多级排序。
  - 示例：
    ```json
    db.collection.createIndex({ "firstName": 1, "lastName": 1 })
    ```
    表示在 `firstName` 和 `lastName` 字段上创建复合索引。
- **地理空间索引**：
  - 支持二维地理空间索引，用于查找附近的点。
  - 示例：
    ```json
    db.collection.createIndex({ "location": "2dsphere" })
    ```
    表示在 `location` 字段上创建地理空间索引。
  - 地理空间索引要求字段为包含两个数值的数组（如经度和纬度）。

---

### **17. 地理空间查询**
- **查找附近的点**：
  - 使用 `$near` 操作符查找距离某个点最近的记录。
  - 示例：
    ```json
    db.collection.find({
        "location": {
            "$near": {
                "$geometry": { "type": "Point", "coordinates": [100, 90] },
                "$maxDistance": 10
            }
        }
    })
    ```
    表示查找距离 `[100, 90]` 最近的记录，最大距离为 10。
- **自定义查询**：
  - 通过 `MongoRepository` 自定义查询方法。
  - 示例代码：
    ```java
    public interface LocationRepository extends MongoRepository<Location, String> {
        List<Location> findByPropertiesNear(Point point, Distance distance);
    }
    ```
    表示定义 `findByPropertiesNear` 方法，查找距离某个点最近的记录。

---

### **18. MongoDB 的优势**
- **灵活的数据模型**：
  - 不同文档可以有不同的字段，无需预先定义 Schema。
  - 例如，存储手机信息时，不同手机可以有不同的参数（如质量、价格）。
- **无需表连接**：
  - 在关系型数据库中，需要通过表连接查询关联数据。
  - 在 MongoDB 中，所有数据可以存储在一个集合中，无需连接操作。
- **高效查询**：
  - 支持复杂查询和索引，查询性能高。
  - 例如，查找待机时间大于 100 且屏幕为 OLED 的手机。

---

### **19. 分片（Sharding）**
- **分片概述**：
  - 将数据分布到多台服务器上，支持水平扩展。
  - 例如，按 ID 范围将数据分布到不同的分片服务器上。
- **分片策略**：
  - 分片策略需要确保数据均匀分布，避免热点问题。
  - 例如，使用哈希分片策略将数据均匀分布到多个分片上。

---

### **20. 总结**
- **MongoDB 的适用场景**：
  - 非结构化数据存储。
  - 高并发读写场景。
  - 数据结构频繁变化的场景。
- **MongoDB 的局限性**：
  - 不支持复杂的事务处理。
  - 不适合强一致性要求的场景。

---
