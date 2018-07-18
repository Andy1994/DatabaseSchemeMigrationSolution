# SQLite Foreign Key
## 1. 外键简介
现在有两张表：
```
-- 父表
CREATE TABLE artist(
  artistid    INTEGER PRIMARY KEY, 
  artistname  TEXT
);
-- 子表
CREATE TABLE track(
  trackid     INTEGER, 
  trackname   TEXT, 
  trackartist INTEGER,
  FOREIGN KEY(trackartist) REFERENCES artist(artistid)
);
CREATE INDEX trackindex ON track(trackartist);

sqlite> SELECT * FROM artist;
artistid  artistname       
--------  -----------------
1         Dean Martin      
2         Frank Sinatra    

sqlite> SELECT * FROM track;
trackid  trackname          trackartist
-------  -----------------  -----------
11       That's Amore       1  
12       Christmas Blues    1  
13       My Way             2  
```
track 表中 trackartist 关联了 artist 表中的 artistid，接下来看看以下的 SQL 执行结果：
```
sqlite> INSERT INTO track VALUES(14, 'Mr. Bojangles', 3);
SQL error: foreign key constraint failed
-- 外键约束失败，是因为插入的 trackartist 为 3，对应 artist 没有 artistid 为 3 的行

sqlite> INSERT INTO track VALUES(14, 'Mr. Bojangles', NULL);
-- 成功，trackartist 为 NULL，这个操作是成功的

sqlite> UPDATE track SET trackartist = 3 WHERE trackname = 'Mr. Bojangles';
SQL error: foreign key constraint failed
-- 失败，由于 artist 仍然没有 artistid 为 3 的行，所以还是错误

sqlite> INSERT INTO artist VALUES(3, 'Sammy Davis Jr.');
sqlite> UPDATE track SET trackartist = 3 WHERE trackname = 'Mr. Bojangles';
-- 成功

sqlite> INSERT INTO track VALUES(15, 'Boogie Woogie', 3);
-- 成功

sqlite> DELETE FROM artist WHERE artistname = 'Frank Sinatra';
SQL error: foreign key constraint failed
-- 失败，是因为 track 表中仍然有用到 artist 表中这条数据

sqlite> DELETE FROM track WHERE trackname = 'My Way';
sqlite> DELETE FROM artist WHERE artistname = 'Frank Sinatra';
-- 成功
```

## 2. 开启外键支持
SQLite 为了向后兼容，默认是关闭外键的，可以用以下命令查看是否开启了外键：
`PRAGMA foreign_keys;`
开启外键：
`PRAGMA foreign_keys = ON;`
关闭外键：
`PRAGMA foreign_keys = OFF;`
以上语句在事务中无效。

## 3. 高级外键功能
3.1 复合外键
```
CREATE TABLE album(
  albumartist TEXT,
  albumname TEXT,
  albumcover BINARY,
  PRIMARY KEY(albumartist, albumname)
);

CREATE TABLE song(
  songid     INTEGER,
  songartist TEXT,
  songalbum TEXT,
  songname   TEXT,
  FOREIGN KEY(songartist, songalbum) REFERENCES album(albumartist, albumname)
);
```

3.2 ON DELETE 和 ON UPDATE 操作
1.NO ACTION：默认的，表示没有什么行为。
2.RESTRICT：当有一个 child 关联到 parent 时，禁止 delete 或 update parent。
3.SET NULL：当 parent 被 delete 或 update 时，child 的关联字段被置为NULL (如果字段有NOT NULL，就出错)。
4.SET DEFAULT：当 parent 被 delete 或 update 时，child 的关联字段被置为默认值，但是如果 parent 中还是没有这个默认值的行，还是会报错。
5.CASCADE：将实施在 parent 上的删除或更新操作，传播给与之关联的 child上。
对于 ON DELETE CASCADE，同被删除的父表中的行相关联的子表中的每1行，也会被删除。
对于 ON UPDATE CASCADE，存储在子表中的每1行，对应的字段的值会被自动修改成同新的父键匹配。