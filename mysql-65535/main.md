# MySQL row size limits 问题记录

## 问题信息

工作时有一个大 json 存储的列给的长度太小了，想要将这个列从 varchar(2048)扩充到 varchar(10240)，但是执行的时候报了如下的错误信息，稍微换个行，方便看。

```
SQL 错误 [1118] [42000]: Row size too large.
The maximum row size for the used table type, not counting BLOBs, is 65535.
This includes storage overhead, check the manual.
You have to change some columns to TEXT or BLOBs
```

当时看了报错信息没有太注意看，扫了一眼就看到了 maximum row size 和 65535，本能地认为是整行的字段大小加起来超过 65535 所以报错了。所以有了错误的理解。

正确的意思是：除 Text 及 Blob 类型外，其余列的长度总和超过 65535bytes 了，如果有需要，将一些大长度的 Varchar 改为 Text 或者 Blob。

## 问题原因

之后我回过头去看表结构，varchar 的总和甚至不到 20000，所以我大概查了一些资料和官方的文档。

在 utf-8 的编码格式下（因为一些历史原因，MySQL 的 utf8 并不是真的 utf8），varchar 所占 bytes 数为 varchar 长度*3+1（如果长度不超过 255，超过了就+2），所以就有了我上述遇到的问题。（utf8mb4 是 varchar 长度*4，+1 还是 2 的规则和前述相同）

解决方案就是将大 varchar 转为 text 存储，当然最好的还是将无关紧要的数据剔除掉，或者拆表。

那为什么转为 text 就可以规避这个长度为 65535 的限制呢？这里应该要介绍一下 MySQL 的数据结构所占的 bytes 问题。

数字类型的类型及占位需求

| Data Type                  | Storage Required                                                            |
| -------------------------- | --------------------------------------------------------------------------- |
| TINYINT                    | 1 byte                                                                      |
| SMALLINT                   | 2 bytes                                                                     |
| MEDIUMINT                  | 3 bytes                                                                     |
| INT, INTEGER               | 4 bytes                                                                     |
| BIGINT                     | 8 bytes                                                                     |
| FLOAT(p)                   | 4 bytes if 0 &lt;= p &lt;= 24, 8 bytes if 25 &lt;= p &lt;= 53               |
| FLOAT                      | 4 bytes                                                                     |
| DOUBLE [PRECISION], REAL   | 8 bytes                                                                     |
| DECIMAL(M,D), NUMERIC(M,D) | 比较复杂，每被 8 整除，需要 4bytes，余下的除以二，四舍五入,详情见下面的表格 |
| BIT(M)                     | approximately (M+7)/8 bytes                                                 |

DECIMAL和NUMERIC所需存储计算规则

| Leftover Digits | Number of Bytes |
| --------------- | --------------- |
| 0               | 0               |
| 1               | 1               |
| 2               | 1               |
| 3               | 2               |
| 4               | 2               |
| 5               | 3               |
| 6               | 3               |
| 7               | 4               |
| 8               | 4               |
