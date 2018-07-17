# SQLite Autoincrement
## 1. 总结
1. AUTOINCREMENT 关键字会占用额外的 CPU，内存，磁盘空间和磁盘 I/O 开销，如果不是很有必要，就避免使用它。
2. 在 SQLite 中，类型为 INTEGER PRIMARY KEY 的列是 ROWID 的别名 （WITHOUT ROWID 表除外），它始终是64位有符号整数。
3. 在 INSERT 上，如果未明确给出 ROWID 或 INTEGER PRIMARY KEY 列的值，它将使用未使用的数，这个数通常比已有最大的数大1。无论是否使用 AUTOINCREMENT 关键字，都是如此。
4. 如果 AUTOINCREMENT 关键字出现在 INTEGER PRIMARY KEY 之后，则会更改自动 ROWID 分配算法，以防止在数据库的生命周期内重用 ROWID。换句话说，AUTOINCREMENT 的目的是防止从先前删除的行重用 ROWID。

## 2. ROWID
1. 在 SQLite 中，table 的行通常有一个64位有符号整数的 ROWID，这个 ROWID 在同一张表中是唯一的。（WITHOUT ROWID 表除外）
2. 你可以使用特殊列比如 ROWID，_ROWID_，或者 OID 访问 SQLite 中的 ROWID。
3. 如果一个表包含了类型为 INTEGER PRIMARY KEY 的列，那么这一列是为 ROWID 的别名。
4. 将新行插入 SQLite 表时，可以将 ROWID 指定为 INSERT 语句的一部分，也可以由数据库引擎自动分配。要手动指定 ROWID，只需将其包含在要插入的值列表中。例如：
```
CREATE TABLE test1（a INT,b TEXT）; 
INSERT INTO test1（rowid,a,b）VALUES（123,5,'hello'）;
```
如果在插入 SQL 中未指定 ROWID，或者 ROWID 为 NULL，则会自动创建相应的ROWID。通常的算法是为新创建的行提供一个 ROWID，该 ROWID 比插入前表中的最大 ROWID 大1。如果表最初为空，则使用 ROWID 为1。如果最大的 ROWID 等于最大可能的整数（9223372036854775807），则数据库引擎开始随机选择正候选 ROWID，直到找到之前未使用的 ROWID。如果在合理的尝试次数后找不到未使用的 ROWID，则插入操作将失败，并显示 SQLITE_FULL 错误。如果未显式插入负 ROWID 值，则自动生成的 ROWID 值将始终大于零。

## 3. AUTOINCREMENT
1. 如果列的类型为 INTEGER PRIMARY KEY AUTOINCREMENT，则使用稍微不同的 ROWID 生成算法。新行的 ROWID 至少比同一个表中以前存在的最大 ROWID 大一个。如果表之前从未包含任何数据，则使用 ROWID 为1。如果先前已插入最大可能的 ROWID，则不允许新的 INSERT，并且任何插入新行的尝试都将失败，并出现 SQLITE_FULL 错误。仅考虑已提交的先前事务中的 ROWID 值。回滚的 ROWID 值将被忽略，可以重复使用。
2. SQLite 使用名为 sqlite_sequence 的内部表跟踪最大的 ROWID 。每当创建包含 AUTOINCREMENT 列的普通表时，都会自动创建并初始化 sqlite_sequence 表。可以使用普通的UPDATE，INSERT和DELETE语句修改sqlite_sequence 表的内容。但是对该表进行修改可能会扰乱 AUTOINCREMENT 密钥生成算法。在进行此类更改之前，请确保你知道自己在做什么。sqlite_sequence 表不跟踪与 UPDATE 语句关联的 ROWID 更改，仅跟踪 INSERT 语句。
3. 请注意，“单调增加”并不意味着 ROWID 总是增加1。1是通常的增量。但是，如果插入由于（例如）唯一性约束而失败，则失败的插入尝试的 ROWID 可能不会在后续插入中重用，从而导致 ROWID 序列中的间隙。AUTOINCREMENT 保证自动选择的 ROWID 将增加，但不会保证它们是连续的。
4. 由于 AUTOINCREMENT 关键字更改了 ROWID 选择算法的行为，因此不允许在 WITHOUT ROWID 表或 INTEGER PRIMARY KEY 以外的任何表列上使用 AUTOINCREMENT 。任何在 WITHOUT ROWID 表或 INTEGER PRIMARY KEY 列以外的列上使用 AUTOINCREMENT 的尝试都会导致错误。