### 第 3 章 Hive 数据类型和文件格式

本章我们将学习 Hive 数据类型和文件格式。Hive 数据类型主要分为：`基本数据类型` 和 `集合数据类型`。我们先来看基本数据类型，它和其他编程语言中的数据类型有什么区别吗？特别是Java。

#### 3.1 基本数据类型

##### 3.1.1 整数类型

Hive 主要有 `4` 种带符号的整数类型：`TINYINT`、`SMALINT`、`INT`、`BIGINT`，分别对应 Java 中的 `byte`、`short`、`int`、`long`。字节长度分别为1、2、4、8。如下表所示：

| Hive 整数类型 | Java 整数类型 |       字节长度        | 后缀 | 例子 |
| :-----------: | :-----------: | :-------------------: | :--: | :--: |
|    TINYINT    |     byte      | **1** byte 有符号整数 |  Y   | 100Y |
|    SMALINT    |     short     | **2** byte 有符号整数 |  S   | 100S |
|      INT      |      int      | **4** byte 有符号整数 |  —   |  —   |
|    BIGINT     |     long      | **8** byte 有符号整数 |  L   | 100L |

在使用整数字面量时，*默认情况下为* `INT`，如果要声明为其他类型，通过`后缀`来标识。

##### 3.1.2 小数类型

Hive 中的小数类型主要有：`FLOAT` 和 `DOUBLE` 两种，分别对应 Java 中的 `float` 和 `double`，字节数分别为 32 和 64 位。

| Hive 小数类型 | Java 小数类型 |                字节长度                | 后缀 |     例子     |
| :-----------: | :-----------: | :------------------------------------: | :--: | :----------: |
|     FLOAT     |     float     |         **4** byte 有符号整数          |  F   |      —       |
|    DOUBLE     |    double     |         **8** byte 有符号整数          |  D   |      —       |
|    DECIMAL    |  BigDecimal   | 表示任意精度的小数，通常在货币当中使用 |  —   | DECIMAL(5,2) |

##### 3.1.3 文本类型

在 Hive 中，主要有 `3` 种类型用于存储文本。

**STRING** 存储变长的文本，对长度没有限制。理论上将 STRING 可以存储的大小为 `2GB`，但是存储特别大的对象时效率可能受到影响，可以考虑使用 `Sqoop 提供的大对象` 支持。

**VARCHAR** 与 STRING 类似，但是长度上只允许在 1-65355 之间。

**CHAR** 则用固定长度来存储数据。

| Hive 文本类型 | Java 文本类型 |                             描述                             |    例子     |
| :-----------: | :-----------: | :----------------------------------------------------------: | :---------: |
|    STRING     |    String     |       String 类型可以用单引号（’）或双引号（”）定义。        | 'Hive 教程' |
|    VARCHAR    |       —       | varchar 类型由长度定义，范围为 1-65355 ，如果存入的字符串长度超过了定义的长度，超出部分会被截断。尾部的空格也会作为字符串的一部分，影响字符串的比较。 | 'Hive 教程' |
|     CHAR      |       —       | char 是固定长度的，最大长度 255，而且尾部的空格不影响字符串的比较。 | 'Hive 教程' |

##### 3.1.4 布尔类型

**BOOLEAN**：表示二元的 true 或 false。

##### 3.1.5 二进制数

**BINARY**：用于存储变长的二进制数据。

##### 3.1.6 时间类型

**TIMESTAMP** 主要用来存储纳秒级别的`时间戳`，同时 Hive 提供了一些内置函数用于在 TIMESTAMP 与 Unix 时间戳（秒）和字符串之间做转换。比如：

```shell
cast(date as date)
cast(timestamp as date)
cast(string as date)
cast(date as string)
```

时间戳类型的数据不包含任务的时区信息，但是 `to_utc_timestamp` 和 `from_utc_timestamp` 函数可以用于时区转换。

**DATE** 类型则表示日期，对应`年月日`三个部分。

#### 3.2 集合数据类型

有些时候，基本数据类型并不能满足我们的需求，在实际开发过程中，还可能会用到一些比较复杂的数据类型，同样，Hive 也给我们提供了 4 种集合数据类型：`ARRAY` ，`MAP` ， `STRUCT` ， `UNION`。

##### 3.2.1 ARRAY

**ARRAY** 是一组具有相同数据类型元素的集合，ARRAY 中的元素是有序的，每个元素都有一个`编号`，编号从`零`开始，因此可以通过编号获取 ARRAY 指定位置的元素。

```mysql
# 创建表结构：person_array
CREATE TABLE IF NOT EXISTS person_array (id int,name array<STRING>)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
COLLECTION ITEMS TERMINATED BY ':' 
STORED AS TEXTFILE;

# 创建原始数据：person_array.txt
1,'aa':'jp'
2,'bb':'cn'
3,'cc':'jp'
4,'dd':'cn'
5,'ee':'jp'

# 从 HDFS 上 /data 目录下导入数据
hive (default)> LOAD DATA INPATH '/data/person_array.txt' OVERWRITE INTO TABLE person_array;
Loading data to table default.person_array
Table default.person_array stats: [numFiles=1, numRows=0, totalSize=60, rawDataSize=0]
OK
Time taken: 0.434 seconds

# 查询所有数据
hive (default)> select * from person_array;
OK
person_array.id person_array.name
1       ["'aa'","'jp'"]
2       ["'bb'","'cn'"]
3       ["'cc'","'jp'"]
4       ["'dd'","'cn'"]
5       ["'ee'","'jp'"]
Time taken: 0.188 seconds, Fetched: 5 row(s)

# 指定查询 name：array 中的字段信息
hive (default)> select id,name[0],name[1] from person_array where name[1]="'jp'";
OK
id      _c1     _c2
1       'aa'    'jp'
3       'cc'    'jp'
5       'ee'    'jp'
```

##### 3.2.2 MAP

**Map** 是一种`键值对`形式的集合，通过 key(键)来快速检索 value(值)。在 Map 中，`key 是唯一的`，但 `value 可以重复`。键值对之间需要在创建表时指定分隔符。

```shell
# 创建表结构：person_map
CREATE TABLE IF NOT EXISTS person_map (id int,name map<STRING,STRING>)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
COLLECTION ITEMS TERMINATED BY ':' 
MAP KEYS TERMINATED BY ':'
STORED AS TEXTFILE;

```

##### 3.2.3 STRUCT

**STRUCT** 类似于 C、C# 语言，Hive 中定义的 struct 类型也可以使用`点`来访问，元素的数据类型可以不相同。从文件加载数据时，文件里的数据分隔符要和建表指定的一致。

```sql
# 创建表结构：person_struct
hive (default)> CREATE TABLE IF NOT EXISTS person_struct (id int,info struct<name:string,country:string>)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
COLLECTION ITEMS TERMINATED BY ':'  STORED AS TEXTFILE;

# 创建原始数据：person_struct.txt
1,'aa':'jp'
2,'bb':'cn'
3,'cc':'jp'
4,'dd':'cn'
5,'ee':'jp'

# 从 HDFS 上 /data 目录下导入数据
hive (default)> LOAD DATA INPATH '/data/person_struct.txt' INTO TABLE person_info;
Loading data to table default.person_info
Table default.person_info stats: [numFiles=1, numRows=0, totalSize=72, rawDataSize=0]
OK

# 查询所有数据
hive (default)> select * from person_struct;
OK
person_struct.id  person_struct.info
1       {"name":"'aa'","country":"'jp'"}
2       {"name":"'bb'","country":"'cn'"}
3       {"name":"'cc'","country":"'jp'"}
4       {"name":"'dd'","country":"'cn'"}
5       {"name":"'ee'","country":"'jp'"}
Time taken: 0.503 seconds, Fetched: 5 row(s)

# 指定查询 info:strcut 中的字段信息
hive (default)> select id,info.name,info.country from person_struct where info.name="'aa'";
OK
id      name    country
1       'aa'    'jp'
Time taken: 0.141 seconds, Fetched: 1 row(s)
```

##### 3.2.4 UNION

Hive 除了支持 `STRUCT`、`ARRAY`、`MAP` 这些原生集合类型，还支持集合的组合，但不支持集合里再组合多个集合。

简单示例 MAP 嵌套 ARRAY，手动设置集合格式的数据非常麻烦，建议采用 `INSERT INTO SELECT` 形式构造数据再插入UNION 表。

```

```

#### 3.3 数据类型转换

类型转换现象指的是：在运算过程中，如果有不同的数据类型进行运算，则 Hive 会先将小的数据类型转换为大的数据类型后，再进行运算，转换的过程我们是看不见的，由编译器自动完成。

显示转换指的是：需要我们手动指定转换。

##### 3.3.1 隐式转换

在 Hive 的类型层次中，可以根据需要进行隐式的类型转换，例如 `TINYINT` 与 `INT` 相加，则会将 TINYINT 转化成 INT 然后 INT 做加法。隐式转换的规则大致可以归纳如下：

- 任意数值类型都可以转换成更宽的数据类型（不会导致精度丢失）或者文本类型。
- 所有的文本类型都可以隐式地转换成另一种文本类型。也可以被转换成 DOUBLE 或者 DECIMAL，转换失败时抛出异常。
- BOOLEAN 不能做任何的类型转换。
- 时间戳和日期可以隐式地转换成文本类型。

##### 3.3.2 显示转换

显示转换主要使用 `CAST` 进行显式的类型转换，比如：

```shell
CAST('1' as INT)
```

如果转换失败，CAST 返回 NULL。

#### 3.3 文件格式