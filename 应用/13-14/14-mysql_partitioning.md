# 14-mysql partitioning

## MySQL 8.0 分区支持

### 分区概述
- **支持的存储引擎**：`InnoDB` 和 `NDB`。
- **分区类型**：
  - **水平分区**：将表的不同行分配到不同的物理分区。
  - **垂直分区**：MySQL 8.0 不支持。

---

## 分区的优势

### 主要优点
1. **存储扩展**：
   - 单个表可以存储超过单个磁盘或文件系统分区的数据。
2. **数据管理**：
   - 通过删除分区轻松移除无用数据。
   - 通过添加新分区简化新数据的插入。
3. **查询优化**：
   - 查询时自动排除不相关的分区。
   - 支持显式分区选择，例如：
     ```sql
     SELECT * FROM t PARTITION (p0, p1) WHERE c < 5;
     ```
   - 支持 `DELETE`、`INSERT`、`REPLACE`、`UPDATE` 和 `LOAD DATA` 等操作的分区选择。

---

## 分区类型

### 1. **RANGE 分区**
- **定义**：根据列值的范围将行分配到分区。
- **示例**：
  ```sql
  CREATE TABLE members (
      firstname VARCHAR(25) NOT NULL,
      lastname VARCHAR(25) NOT NULL,
      username VARCHAR(16) NOT NULL,
      email VARCHAR(35),
      joined DATE NOT NULL
  )
  PARTITION BY RANGE(YEAR(joined)) (
      PARTITION p0 VALUES LESS THAN (1960),
      PARTITION p1 VALUES LESS THAN (1970),
      PARTITION p2 VALUES LESS THAN (1980),
      PARTITION p3 VALUES LESS THAN (1990),
      PARTITION p4 VALUES LESS THAN MAXVALUE
  );
  ```

### 2. **LIST 分区**
- **定义**：根据列值匹配一组离散值来选择分区。
- **示例**：
  ```sql
  CREATE TABLE members (
      firstname VARCHAR(25) NOT NULL,
      lastname VARCHAR(25) NOT NULL,
      username VARCHAR(16) NOT NULL,
      email VARCHAR(35),
      joined DATE NOT NULL
  )
  PARTITION BY LIST(YEAR(joined)) (
      PARTITION p0 VALUES IN (1960, 1970),
      PARTITION p1 VALUES IN (1980, 1990)
  );
  ```

### 3. **HASH 分区**
- **定义**：根据用户定义的表达式返回值选择分区。
- **示例**：
  ```sql
  CREATE TABLE members (
      firstname VARCHAR(25) NOT NULL,
      lastname VARCHAR(25) NOT NULL,
      username VARCHAR(16) NOT NULL,
      email VARCHAR(35),
      joined DATE NOT NULL
  )
  PARTITION BY HASH(YEAR(joined))
  PARTITIONS 6;
  ```

### 4. **KEY 分区**
- **定义**：类似于 `HASH` 分区，但使用 MySQL 提供的哈希函数。
- **示例**：
  ```sql
  CREATE TABLE members (
      firstname VARCHAR(25) NOT NULL,
      lastname VARCHAR(25) NOT NULL,
      username VARCHAR(16) NOT NULL,
      email VARCHAR(35),
      joined DATE NOT NULL
  )
  PARTITION BY KEY(joined)
  PARTITIONS 6;
  ```

---

## 分区的常见用途

### 按日期分区
- **场景**：按日期范围分区，便于管理和查询。
- **示例**：
  ```sql
  CREATE TABLE members (
      firstname VARCHAR(25) NOT NULL,
      lastname VARCHAR(25) NOT NULL,
      username VARCHAR(16) NOT NULL,
      email VARCHAR(35),
      joined DATE NOT NULL
  )
  PARTITION BY RANGE(YEAR(joined)) (
      PARTITION p0 VALUES LESS THAN (1960),
      PARTITION p1 VALUES LESS THAN (1970),
      PARTITION p2 VALUES LESS THAN (1980),
      PARTITION p3 VALUES LESS THAN (1990),
      PARTITION p4 VALUES LESS THAN MAXVALUE
  );
  ```

---

### 对比总结
| **分区类型** | **定义**                           | **适用场景**                 |
| ------------ | ---------------------------------- | ---------------------------- |
| **RANGE**    | 根据列值的范围分配分区             | 按日期、数值范围分区         |
| **LIST**     | 根据列值匹配一组离散值分配分区     | 按离散值（如地区、类别）分区 |
| **HASH**     | 根据用户定义的表达式返回值分配分区 | 均匀分布数据                 |
| **KEY**      | 根据 MySQL 提供的哈希函数分配分区  | 均匀分布数据，支持非整数列   |

## MySQL 分区表：RANGE 分区

### RANGE 分区概述
- **定义**：根据分区表达式的值范围将表的行分配到不同的分区。
- **特点**：
  - 分区范围应连续且不重叠。
  - 使用 `VALUES LESS THAN` 定义分区范围。

---

## 创建 RANGE 分区表

### 示例 1：按 `store_id` 分区
```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN (21)
);
```

### 示例 2：按 `store_id` 分区，包含 `MAXVALUE`
```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

### 示例 3：按 `job_code` 分区
```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (job_code) (
    PARTITION p0 VALUES LESS THAN (100),
    PARTITION p1 VALUES LESS THAN (1000),
    PARTITION p2 VALUES LESS THAN (10000)
);
```

### 示例 4：按 `YEAR(separated)` 分区
```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (YEAR(separated)) (
    PARTITION p0 VALUES LESS THAN (1991),
    PARTITION p1 VALUES LESS THAN (1996),
    PARTITION p2 VALUES LESS THAN (2001),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

### 示例 5：按 `UNIX_TIMESTAMP(report_updated)` 分区
```sql
CREATE TABLE quarterly_report_status (
    report_id INT NOT NULL,
    report_status VARCHAR(20) NOT NULL,
    report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)
PARTITION BY RANGE (UNIX_TIMESTAMP(report_updated)) (
    PARTITION p0 VALUES LESS THAN (UNIX_TIMESTAMP('2008-01-01 00:00:00')),
    PARTITION p1 VALUES LESS THAN (UNIX_TIMESTAMP('2008-04-01 00:00:00')),
    PARTITION p2 VALUES LESS THAN (UNIX_TIMESTAMP('2008-07-01 00:00:00')),
    PARTITION p3 VALUES LESS THAN (UNIX_TIMESTAMP('2008-10-01 00:00:00')),
    PARTITION p4 VALUES LESS THAN (UNIX_TIMESTAMP('2009-01-01 00:00:00')),
    PARTITION p5 VALUES LESS THAN (UNIX_TIMESTAMP('2009-04-01 00:00:00')),
    PARTITION p6 VALUES LESS THAN (UNIX_TIMESTAMP('2009-07-01 00:00:00')),
    PARTITION p7 VALUES LESS THAN (UNIX_TIMESTAMP('2009-10-01 00:00:00')),
    PARTITION p8 VALUES LESS THAN (UNIX_TIMESTAMP('2010-01-01 00:00:00')),
    PARTITION p9 VALUES LESS THAN (MAXVALUE)
);
```

### 示例 6：按 `YEAR(joined)` 分区
```sql
CREATE TABLE members (
    firstname VARCHAR(25) NOT NULL,
    lastname VARCHAR(25) NOT NULL,
    username VARCHAR(16) NOT NULL,
    email VARCHAR(35),
    joined DATE NOT NULL
)
PARTITION BY RANGE (YEAR(joined)) (
    PARTITION p0 VALUES LESS THAN (1960),
    PARTITION p1 VALUES LESS THAN (1970),
    PARTITION p2 VALUES LESS THAN (1980),
    PARTITION p3 VALUES LESS THAN (1990),
    PARTITION p4 VALUES LESS THAN MAXVALUE
);
```

### 示例 7：按 `RANGE COLUMNS(joined)` 分区
```sql
CREATE TABLE members (
    firstname VARCHAR(25) NOT NULL,
    lastname VARCHAR(25) NOT NULL,
    username VARCHAR(16) NOT NULL,
    email VARCHAR(35),
    joined DATE NOT NULL
)
PARTITION BY RANGE COLUMNS(joined) (
    PARTITION p0 VALUES LESS THAN ('1960-01-01'),
    PARTITION p1 VALUES LESS THAN ('1970-01-01'),
    PARTITION p2 VALUES LESS THAN ('1980-01-01'),
    PARTITION p3 VALUES LESS THAN ('1990-01-01'),
    PARTITION p4 VALUES LESS THAN MAXVALUE
);
```

---

### 对比总结
| **分区方式**      | **定义**                         | **适用场景**           |
| ----------------- | -------------------------------- | ---------------------- |
| **RANGE**         | 根据列值的范围分配分区           | 按数值、日期范围分区   |
| **RANGE COLUMNS** | 根据列值的范围分配分区，支持多列 | 按多列范围分区         |
| **MAXVALUE**      | 用于定义最后一个分区的上限       | 处理超出定义范围的数据 |

## MySQL 分区表：LIST 分区

### LIST 分区概述
- **定义**：根据列值是否属于一组离散值列表来选择分区。
- **特点**：
  - 每个分区对应一组特定的列值。
  - 不支持类似 `MAXVALUE` 的“兜底”分区，所有可能的值必须显式定义。

---

## 创建 LIST 分区表

### 示例 1：按 `store_id` 分区
```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY LIST(store_id) (
    PARTITION pNorth VALUES IN (3, 5, 6, 9, 17),
    PARTITION pEast VALUES IN (1, 2, 10, 11, 19, 20),
    PARTITION pWest VALUES IN (4, 12, 13, 14, 18),
    PARTITION pCentral VALUES IN (7, 8, 15, 16)
);
```

### 示例 2：按 `c1` 分区
```sql
CREATE TABLE h2 (
    c1 INT,
    c2 INT
)
PARTITION BY LIST(c1) (
    PARTITION p0 VALUES IN (1, 4, 7),
    PARTITION p1 VALUES IN (2, 5, 8)
);
```

---

## LIST 分区的注意事项

### 1. **未定义值的处理**
- 如果插入的值未在分区定义中列出，MySQL 会抛出错误。
- **示例**：
  ```sql
  INSERT INTO h2 VALUES (3, 5);
  ```
  **错误**：
  ```
  ERROR 1525 (HY000): Table has no partition for value 3
  ```

### 2. **使用 `IGNORE` 忽略错误**
- 使用 `INSERT IGNORE` 可以忽略未定义值的错误。
- **示例**：
  ```sql
  INSERT IGNORE INTO h2 VALUES (2, 5), (6, 10), (7, 5), (3, 1), (1, 9);
  ```
  **结果**：
  - 成功插入 `(2, 5)`、`(7, 5)` 和 `(1, 9)`。
  - 忽略 `(6, 10)` 和 `(3, 1)`。

---

### 对比总结
| **分区方式** | **定义**                                 | **适用场景**                 |
| ------------ | ---------------------------------------- | ---------------------------- |
| **LIST**     | 根据列值是否属于一组离散值列表来选择分区 | 按离散值（如地区、类别）分区 |
| **RANGE**    | 根据列值的范围分配分区                   | 按数值、日期范围分区         |
| **MAXVALUE** | 用于定义最后一个分区的上限               | 处理超出定义范围的数据       |

## MySQL 分区表：COLUMNS 分区

### COLUMNS 分区概述
- **定义**：`COLUMNS` 分区是 `RANGE` 和 `LIST` 分区的变体，支持使用多列作为分区键。
- **特点**：
  - 支持非整数列（如 `DATE`、`DATETIME`、`CHAR`、`VARCHAR` 等）。
  - 分区键的列值用于确定行的存储位置和分区修剪。

---

## RANGE COLUMNS 分区

### 特点
- **与 RANGE 分区的区别**：
  - 不接受表达式，仅接受列名。
  - 支持多列分区键。
  - 基于列值的元组比较，而非标量值比较。
  - 支持字符串、`DATE` 和 `DATETIME` 列作为分区键。

### 语法
```sql
CREATE TABLE table_name
PARTITION BY RANGE COLUMNS(column_list) (
    PARTITION partition_name VALUES LESS THAN (value_list)[,
    PARTITION partition_name VALUES LESS THAN (value_list)][,
    ...]
);
```

### 示例 1：按多列分区
```sql
CREATE TABLE rcx (
    a INT,
    b INT,
    c CHAR(3),
    d INT
)
PARTITION BY RANGE COLUMNS(a, d, c) (
    PARTITION p0 VALUES LESS THAN (5, 10, 'ggg'),
    PARTITION p1 VALUES LESS THAN (10, 20, 'mmm'),
    PARTITION p2 VALUES LESS THAN (15, 30, 'sss'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE, MAXVALUE, MAXVALUE)
);
```

### 示例 2：按单列分区
```sql
CREATE TABLE rx (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a) (
    PARTITION p0 VALUES LESS THAN (5),
    PARTITION p1 VALUES LESS THAN (MAXVALUE)
);
```

### 示例 3：按 `lname` 列分区
```sql
CREATE TABLE employees_by_lname (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE COLUMNS(lname) (
    PARTITION p0 VALUES LESS THAN ('g'),
    PARTITION p1 VALUES LESS THAN ('m'),
    PARTITION p2 VALUES LESS THAN ('t'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
```

### 示例 4：按 `hired` 列分区
```sql
ALTER TABLE employees
PARTITION BY RANGE COLUMNS(hired) (
    PARTITION p0 VALUES LESS THAN ('1970-01-01'),
    PARTITION p1 VALUES LESS THAN ('1980-01-01'),
    PARTITION p2 VALUES LESS THAN ('1990-01-01'),
    PARTITION p3 VALUES LESS THAN ('2000-01-01'),
    PARTITION p4 VALUES LESS THAN ('2010-01-01'),
    PARTITION p5 VALUES LESS THAN (MAXVALUE)
);
```

---

## LIST COLUMNS 分区

### 特点
- **定义**：`LIST COLUMNS` 分区是 `LIST` 分区的变体，支持使用多列作为分区键。
- **适用场景**：按离散值（如城市、日期）分区。

### 示例 1：按 `city` 列分区
```sql
CREATE TABLE customers_1 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS(city) (
    PARTITION pRegion_1 VALUES IN('Oskarshamn', 'Högsby', 'Mönsterås'),
    PARTITION pRegion_2 VALUES IN('Vimmerby', 'Hultsfred', 'Västervik'),
    PARTITION pRegion_3 VALUES IN('Nässjö', 'Eksjö', 'Vetlanda'),
    PARTITION pRegion_4 VALUES IN('Uppvidinge', 'Alvesta', 'Växjo')
);
```

### 示例 2：按 `renewal` 列分区
```sql
CREATE TABLE customers_1 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS(renewal) (
    PARTITION pWeek_1 VALUES IN('2010-02-01', '2010-02-02', '2010-02-03',
                               '2010-02-04', '2010-02-05', '2010-02-06', '2010-02-07'),
    PARTITION pWeek_2 VALUES IN('2010-02-08', '2010-02-09', '2010-02-10',
                               '2010-02-11', '2010-02-12', '2010-02-13', '2010-02-14'),
    PARTITION pWeek_3 VALUES IN('2010-02-15', '2010-02-16', '2010-02-17',
                               '2010-02-18', '2010-02-19', '2010-02-20', '2010-02-21'),
    PARTITION pWeek_4 VALUES IN('2010-02-22', '2010-02-23', '2010-02-24',
                               '2010-02-25', '2010-02-26', '2010-02-27', '2010-02-28')
);
```

---

### 对比总结
| **分区方式**      | **定义**                                   | **适用场景**                 |
| ----------------- | ------------------------------------------ | ---------------------------- |
| **RANGE COLUMNS** | 根据多列值的范围分配分区                   | 按多列范围分区               |
| **LIST COLUMNS**  | 根据多列值是否属于一组离散值列表来选择分区 | 按离散值（如城市、日期）分区 |
| **MAXVALUE**      | 用于定义最后一个分区的上限                 | 处理超出定义范围的数据       |

## MySQL 分区技术

### HASH 分区
HASH 分区主要用于确保数据在预定义的分区中均匀分布。通过使用 HASH 函数对某一列的值进行计算，MySQL 将数据分配到不同的分区中。

#### 示例
```sql
CREATE TABLE employees (
    id INT NOT NULL, 
    fname VARCHAR(30), 
    lname VARCHAR(30), 
    hired DATE NOT NULL DEFAULT '1970-01-01', 
    separated DATE NOT NULL DEFAULT '9999-12-31', 
    job_code INT, 
    store_id INT 
) 
PARTITION BY HASH(store_id) 
PARTITIONS 4;
```

#### 使用表达式进行 HASH 分区
```sql
CREATE TABLE employees (
    id INT NOT NULL, 
    fname VARCHAR(30), 
    lname VARCHAR(30), 
    hired DATE NOT NULL DEFAULT '1970-01-01', 
    separated DATE NOT NULL DEFAULT '9999-12-31', 
    job_code INT, 
    store_id INT 
) 
PARTITION BY HASH(YEAR(hired)) 
PARTITIONS 4;
```

### 线性 HASH 分区
线性 HASH 分区与普通 HASH 分区的区别在于，线性 HASH 使用线性二次幂算法，而普通 HASH 使用取模运算。

#### 示例
```sql
CREATE TABLE employees (
    id INT NOT NULL, 
    fname VARCHAR(30), 
    lname VARCHAR(30), 
    hired DATE NOT NULL DEFAULT '1970-01-01', 
    separated DATE NOT NULL DEFAULT '9999-12-31', 
    job_code INT, 
    store_id INT 
) 
PARTITION BY LINEAR HASH(YEAR(hired)) 
PARTITIONS 4;
```

#### 线性 HASH 分区算法
给定表达式 `expr`，线性 HASH 分区将记录存储在分区号 `N` 中，`N` 的计算步骤如下：
1. 找到大于分区数 `num` 的下一个 2 的幂，记为 `V`。
   ```sql
   V = POWER(2, CEILING(LOG(2, num)))
   ```
2. 计算 `N = F(column_list) & (V - 1)`。
3. 如果 `N >= num`，则：
   - 设置 `V = V / 2`
   - 设置 `N = N & (V - 1)`

#### 示例计算
```sql
CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATE) 
PARTITION BY LINEAR HASH( YEAR(col3) ) 
PARTITIONS 6;

-- 示例 1
V = POWER(2, CEILING( LOG(2,6) )) = 8 
N = YEAR('2003-04-14') & (8 - 1) 
    = 2003 & 7 
    = 3 
-- 3 >= 6 为 FALSE，记录存储在分区 #3

-- 示例 2
V = 8 
N = YEAR('1998-10-19') & (8 - 1) 
    = 1998 & 7 
    = 6 
-- 6 >= 6 为 TRUE，需要进一步计算
N = 6 & ((8 / 2) - 1) 
    = 6 & 3 
    = 2 
-- 2 >= 6 为 FALSE，记录存储在分区 #2
```

### KEY 分区
KEY 分区与 HASH 分区类似，区别在于 KEY 分区的哈希函数由 MySQL 服务器提供，而不是用户定义的表达式。

#### 示例
```sql
CREATE TABLE k1 (
    id INT NOT NULL PRIMARY KEY, 
    name VARCHAR(20) 
) 
PARTITION BY KEY() 
PARTITIONS 2;
```

#### 线性 KEY 分区
```sql
CREATE TABLE tk ( col1 INT NOT NULL, col2 CHAR(5), col3 DATE ) 
PARTITION BY LINEAR KEY (col1) 
PARTITIONS 3;
```

### 复合分区（子分区）
复合分区（子分区）是对分区表中的每个分区进行进一步划分。

#### 示例
```sql
CREATE TABLE ts (id INT, purchased DATE) 
    PARTITION BY RANGE( YEAR(purchased) ) 
    SUBPARTITION BY HASH( TO_DAYS(purchased) ) 
    SUBPARTITIONS 2 ( 
        PARTITION p0 VALUES LESS THAN (1990), 
        PARTITION p1 VALUES LESS THAN (2000), 
        PARTITION p2 VALUES LESS THAN MAXVALUE 
    );
```

#### 显式定义子分区
```sql
CREATE TABLE ts (id INT, purchased DATE) 
PARTITION BY RANGE( YEAR(purchased) ) 
SUBPARTITION BY HASH( TO_DAYS(purchased) ) ( 
    PARTITION p0 VALUES LESS THAN (1990) ( 
        SUBPARTITION s0, 
        SUBPARTITION s1 
    ), 
    PARTITION p1 VALUES LESS THAN (2000) ( 
        SUBPARTITION s2, 
        SUBPARTITION s3 
    ), 
    PARTITION p2 VALUES LESS THAN MAXVALUE ( 
        SUBPARTITION s4, 
        SUBPARTITION s5 
    ) 
);
```

### 总结
- **HASH 分区**：通过哈希函数均匀分布数据。
- **线性 HASH 分区**：使用线性二次幂算法进行分区。
- **KEY 分区**：由 MySQL 提供哈希函数。
- **复合分区**：对分区进一步划分，支持更细粒度的数据管理。

## MySQL 分区中的 NULL 值处理

MySQL 分区表在处理 `NULL` 值时，行为因分区类型的不同而有所差异。以下是针对不同分区类型的 `NULL` 值处理方式。

---

### RANGE 分区中的 NULL 值处理
在 RANGE 分区中，如果插入的行的分区列值为 `NULL`，则该行会被插入到最低的分区中。

#### 示例
```sql
CREATE TABLE t1 (
    c1 INT,
    c2 VARCHAR(20)
)
PARTITION BY RANGE(c1) (
    PARTITION p0 VALUES LESS THAN (0),
    PARTITION p1 VALUES LESS THAN (10),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);

CREATE TABLE t2 (
    c1 INT,
    c2 VARCHAR(20)
)
PARTITION BY RANGE(c1) (
    PARTITION p0 VALUES LESS THAN (-5),
    PARTITION p1 VALUES LESS THAN (0),
    PARTITION p2 VALUES LESS THAN (10),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

#### 插入 NULL 值
```sql
INSERT INTO t1 VALUES (NULL, 'mothra');
INSERT INTO t2 VALUES (NULL, 'mothra');
```

#### 查询分区信息
```sql
SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 't_';
```

#### 结果
- `t1` 和 `t2` 中的 `NULL` 值都被插入到最低的分区（`p0`）。
- 删除分区 `p0` 后，`NULL` 值也会被删除。

---

### LIST 分区中的 NULL 值处理
在 LIST 分区中，只有明确为 `NULL` 值定义了分区时，才能插入 `NULL` 值。

#### 示例
```sql
CREATE TABLE ts1 (
    c1 INT,
    c2 VARCHAR(20)
)
PARTITION BY LIST(c1) (
    PARTITION p0 VALUES IN (0, 3, 6),
    PARTITION p1 VALUES IN (1, 4, 7),
    PARTITION p2 VALUES IN (2, 5, 8)
);

-- 插入 NULL 值会失败
INSERT INTO ts1 VALUES (NULL, 'mothra'); -- 报错：Table has no partition for value NULL
```

#### 允许 NULL 值的 LIST 分区
```sql
CREATE TABLE ts2 (
    c1 INT,
    c2 VARCHAR(20)
)
PARTITION BY LIST(c1) (
    PARTITION p0 VALUES IN (0, 3, 6),
    PARTITION p1 VALUES IN (1, 4, 7),
    PARTITION p2 VALUES IN (2, 5, 8),
    PARTITION p3 VALUES IN (NULL)
);

CREATE TABLE ts3 (
    c1 INT,
    c2 VARCHAR(20)
)
PARTITION BY LIST(c1) (
    PARTITION p0 VALUES IN (0, 3, 6),
    PARTITION p1 VALUES IN (1, 4, 7, NULL),
    PARTITION p2 VALUES IN (2, 5, 8)
);
```

#### 插入 NULL 值
```sql
INSERT INTO ts2 VALUES (NULL, 'mothra'); -- 成功插入到 p3
INSERT INTO ts3 VALUES (NULL, 'mothra'); -- 成功插入到 p1
```

#### 查询分区信息
```sql
SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 't_';
```

---

### HASH 和 KEY 分区中的 NULL 值处理
在 HASH 和 KEY 分区中，任何产生 `NULL` 值的分区表达式都会被当作 `0` 处理。

#### 示例
```sql
CREATE TABLE th (
    c1 INT,
    c2 VARCHAR(20)
)
PARTITION BY HASH(c1)
PARTITIONS 2;
```

#### 插入 NULL 值
```sql
INSERT INTO th VALUES (NULL, 'mothra'), (0, 'gigan');
```

#### 查询分区信息
```sql
SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME = 'th';
```

#### 结果
- `NULL` 值被当作 `0` 处理，插入到分区 `p0`。
- 分区 `p0` 包含两行数据（`NULL` 和 `0`）。

---

### 总结
- **RANGE 分区**：`NULL` 值插入到最低分区。
- **LIST 分区**：必须显式定义 `NULL` 值分区，否则插入 `NULL` 值会失败。
- **HASH 和 KEY 分区**：`NULL` 值被当作 `0` 处理。

通过合理设计分区策略，可以有效管理包含 `NULL` 值的数据。

## MySQL 分区表操作详解

以下是对 MySQL 分区表操作的详细说明和示例，涵盖创建分区表、插入数据、查询分区、删除分区、添加分区以及重组分区等操作。

---

### 创建分区表
使用 `RANGE` 分区，按 `YEAR(purchased)` 分区。

```sql
CREATE TABLE tr (
    id INT,
    name VARCHAR(50),
    purchased DATE
)
PARTITION BY RANGE( YEAR(purchased) ) (
    PARTITION p0 VALUES LESS THAN (1990),
    PARTITION p1 VALUES LESS THAN (1995),
    PARTITION p2 VALUES LESS THAN (2000),
    PARTITION p3 VALUES LESS THAN (2005),
    PARTITION p4 VALUES LESS THAN (2010),
    PARTITION p5 VALUES LESS THAN (2015)
);
```

---

### 插入数据
向分区表中插入数据。

```sql
INSERT INTO tr VALUES
(1, 'desk organiser', '2003-10-15'),
(2, 'alarm clock', '1997-11-05'),
(3, 'chair', '2009-03-10'),
(4, 'bookcase', '1989-01-10'),
(5, 'exercise bike', '2014-05-09'),
(6, 'sofa', '1987-06-05'),
(7, 'espresso maker', '2011-11-22'),
(8, 'aquarium', '1992-08-04'),
(9, 'study desk', '2006-09-16'),
(10, 'lava lamp', '1998-12-25');
```

---

### 查询分区数据
#### 查询特定时间范围内的数据
```sql
SELECT * FROM tr
WHERE purchased BETWEEN '1995-01-01' AND '1999-12-31';
```

#### 结果
```plaintext
+------+--------------+------------+
| id   | name         | purchased  |
+------+--------------+------------+
| 2    | alarm clock  | 1997-11-05 |
| 10   | lava lamp    | 1998-12-25 |
+------+--------------+------------+
```

#### 查询特定分区的数据
```sql
SELECT * FROM tr PARTITION (p2);
```

#### 结果
```plaintext
+------+--------------+------------+
| id   | name         | purchased  |
+------+--------------+------------+
| 2    | alarm clock  | 1997-11-05 |
| 10   | lava lamp    | 1998-12-25 |
+------+--------------+------------+
```

---

### 删除分区
删除分区 `p2`。

```sql
ALTER TABLE tr DROP PARTITION p2;
```

#### 验证删除分区后的查询
```sql
SELECT * FROM tr
WHERE purchased BETWEEN '1995-01-01' AND '1999-12-31';
```

#### 结果
```plaintext
Empty set (0.00 sec)
```

---

### 查看表结构
查看分区表的结构。

```sql
SHOW CREATE TABLE tr\G
```

#### 结果
```plaintext
*************************** 1. row ***************************
       Table: tr
Create Table: CREATE TABLE `tr` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(50) DEFAULT NULL,
  `purchased` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
/*!50100 PARTITION BY RANGE ( YEAR(purchased))
(PARTITION p0 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (1995) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (2005) ENGINE = InnoDB,
 PARTITION p4 VALUES LESS THAN (2010) ENGINE = InnoDB,
 PARTITION p5 VALUES LESS THAN (2015) ENGINE = InnoDB) */
```

---

### 插入新数据并查询
插入新数据并查询特定时间范围内的数据。

```sql
INSERT INTO tr VALUES (11, 'pencil holder', '1995-07-12');
SELECT * FROM tr
WHERE purchased BETWEEN '1995-01-01' AND '2004-12-31';
```

#### 结果
```plaintext
+-----+----------------+------------+
| id  | name           | purchased  |
+-----+----------------+------------+
| 1   | desk organizer | 2003-10-15 |
| 11  | pencil holder  | 1995-07-12 |
+-----+----------------+------------+
```

---

### 删除分区并验证
删除分区 `p3` 并验证查询结果。

```sql
ALTER TABLE tr DROP PARTITION p3;
SELECT * FROM tr
WHERE purchased BETWEEN '1995-01-01' AND '2004-12-31';
```

#### 结果
```plaintext
Empty set (0.00 sec)
```

---

### 添加分区
向分区表中添加新分区。

```sql
CREATE TABLE members (
    id INT,
    fname VARCHAR(25),
    lname VARCHAR(25),
    dob DATE
)
PARTITION BY RANGE( YEAR(dob) ) (
    PARTITION p0 VALUES LESS THAN (1980),
    PARTITION p1 VALUES LESS THAN (1990),
    PARTITION p2 VALUES LESS THAN (2000)
);

ALTER TABLE members ADD PARTITION (PARTITION p3 VALUES LESS THAN (2010));
```

---

### 重组分区
重组分区以调整分区范围。

```sql
ALTER TABLE members REORGANIZE PARTITION p0 INTO (
    PARTITION n0 VALUES LESS THAN (1970),
    PARTITION n1 VALUES LESS THAN (1980)
);
```

---

### 错误处理
#### 添加分区时值必须严格递增
```sql
ALTER TABLE members ADD PARTITION (PARTITION n VALUES LESS THAN (1970));
-- 错误：VALUES LESS THAN value must be strictly increasing for each partition
```

#### LIST 分区中常量重复定义
```sql
ALTER TABLE tt ADD PARTITION (PARTITION np VALUES IN (4, 8, 12));
-- 错误：Multiple definition of same constant in list partitioning
```

---

### 总结
- **创建分区表**：使用 `RANGE` 或 `LIST` 分区。
- **插入数据**：数据会根据分区规则自动分配到对应的分区。
- **查询分区数据**：可以直接查询特定分区或按条件查询。
- **删除分区**：使用 `DROP PARTITION` 删除分区及其数据。
- **添加分区**：使用 `ADD PARTITION` 扩展分区范围。
- **重组分区**：使用 `REORGANIZE PARTITION` 调整分区范围或合并分区。

通过合理使用分区表，可以提高查询性能并简化数据管理。





## MySQL 分区表操作笔记

### 分区表的基本操作

#### 创建分区表
```sql
CREATE TABLE clients (
    id INT, 
    fname VARCHAR(30), 
    lname VARCHAR(30), 
    signed DATE 
) 
PARTITION BY HASH( MONTH(signed) ) 
PARTITIONS 12;
```

#### 合并分区
对于使用 `HASH` 或 `KEY` 分区的表，不能直接删除分区，但可以通过 `ALTER TABLE ... COALESCE PARTITION` 合并分区。

```sql
ALTER TABLE clients COALESCE PARTITION 4;
```
- 合并分区时，不能删除所有分区，否则会报错：
```sql
ERROR 1478 (HY000): Cannot remove all partitions, use DROP TABLE instead
```

#### 添加分区
```sql
ALTER TABLE clients ADD PARTITION PARTITIONS 6;
```

### 分区表与普通表的交换

在 MySQL 8.0 中，可以使用 `ALTER TABLE ... EXCHANGE PARTITION` 将分区表的分区与普通表进行交换。

#### 创建分区表
```sql
CREATE TABLE e (
    id INT NOT NULL, 
    fname VARCHAR(30), 
    lname VARCHAR(30) 
) 
PARTITION BY RANGE (id) ( 
    PARTITION p0 VALUES LESS THAN (50), 
    PARTITION p1 VALUES LESS THAN (100), 
    PARTITION p2 VALUES LESS THAN (150), 
    PARTITION p3 VALUES LESS THAN (MAXVALUE) 
);
```

#### 插入数据
```sql
INSERT INTO e VALUES 
    (1669, "Jim", "Smith"), 
    (337, "Mary", "Jones"), 
    (16, "Frank", "White"), 
    (2005, "Linda", "Black");
```

#### 创建普通表并移除分区
```sql
CREATE TABLE e2 LIKE e;
ALTER TABLE e2 REMOVE PARTITIONING;
```

#### 交换分区
```sql
ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
```

#### 查询分区信息
```sql
SELECT PARTITION_NAME, TABLE_ROWS 
FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'e';
```

#### 查询普通表数据
```sql
SELECT * FROM e2;
```

### 非匹配行的处理

如果普通表中存在不符合分区定义的行，交换分区时会报错。可以通过 `WITHOUT VALIDATION` 忽略验证。

```sql
INSERT INTO e2 VALUES (51, "Ellen", "McDonald");
ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
-- 报错：ERROR 1707 (HY000): Found row that does not match the partition

ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2 WITHOUT VALIDATION;
```

### 分区维护操作

#### 重建分区
```sql
ALTER TABLE t1 REBUILD PARTITION p0, p1;
```

#### 优化分区
```sql
ALTER TABLE t1 OPTIMIZE PARTITION p0, p1;
```

#### 分析分区
```sql
ALTER TABLE t1 ANALYZE PARTITION p3;
```

#### 修复分区
```sql
ALTER TABLE t1 REPAIR PARTITION p0,p1;
```

#### 检查分区
```sql
ALTER TABLE trb3 CHECK PARTITION p1;
```

### 分区修剪与选择

#### 分区修剪
分区修剪是自动进行的，查询时只检查相关的分区。

```sql
CREATE TABLE t1 (
    fname VARCHAR(50) NOT NULL, 
    lname VARCHAR(50) NOT NULL, 
    region_code TINYINT UNSIGNED NOT NULL, 
    dob DATE NOT NULL 
) 
PARTITION BY RANGE( region_code ) ( 
    PARTITION p0 VALUES LESS THAN (64), 
    PARTITION p1 VALUES LESS THAN (128), 
    PARTITION p2 VALUES LESS THAN (192), 
    PARTITION p3 VALUES LESS THAN MAXVALUE 
);

SELECT fname, lname, region_code, dob 
FROM t1 
WHERE region_code > 125 AND region_code < 130;
```

#### 分区选择
分区选择与分区修剪类似，但需要手动指定要检查的分区。支持以下 SQL 语句：
- `SELECT`
- `DELETE`
- `INSERT`
- `REPLACE`
- `UPDATE`
- `LOAD DATA`
- `LOAD XML`