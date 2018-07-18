# Database Scheme Migration Solution
* 相关文章
1. FetchedRecordsController [链接](https://medium.com/@gwendal.roue/grdb-stories-1da44bdb53ab)
2. Loose Coupling between Objects and Database [链接](https://medium.com/@gwendal.roue/grdb-stories-9bfcfb8b6a88)

* 相关三方库
- [ ] GRDB.swift [链接](https://github.com/groue/GRDB.swift)
- [ ] BLDatabase [链接](https://github.com/hrt941009/BLDatabase)
- [ ] DataMigration [链接](https://github.com/KirstenDunst/DataMigration)
- [x] fmdb-migration-manager [链接](https://github.com/mocra/fmdb-migration-manager)  (更新时间久远，比较不专业)
- [x] FMDBMigrationManager [链接](https://github.com/layerhq/FMDBMigrationManager)  (已出文档)
- [x] SQLiteMigrationManager.swift [链接](https://github.com/garriguv/SQLiteMigrationManager.swift)  (是 FMDBMigrationManager Swift 下的实现)
- [ ] LKDaoBase [链接](https://github.com/li6185377/LKDaoBase)
- [ ] LKDBHelper-SQLite-ORM [链接](https://github.com/li6185377/LKDBHelper-SQLite-ORM)
- [ ] SQLiteKit [链接](https://github.com/alaborie/SQLiteKit)
- [ ] wcdb [链接](https://github.com/Tencent/wcdb)

* 调研结果
1. [GRDB FetchedRecordsController 实现原理](https://github.com/Andy1994/DatabaseSchemeMigrationSolution/blob/master/GRDB%20FetchedRecordsController%20%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.md)
2. [FMDBMigrationManager 调研](https://github.com/Andy1994/DatabaseSchemeMigrationSolution/blob/master/FMDBMigrationManager%20%E8%B0%83%E7%A0%94.md)

* SQLite sql 调研
1. [SQLite Autoincrement](https://github.com/Andy1994/DatabaseSchemeMigrationSolution/blob/master/SQLite%20Autoincrement.md)
2. [SQLite Foreign Key](https://github.com/Andy1994/DatabaseSchemeMigrationSolution/blob/master/SQLite%20Foreign%20Key.md)

* 调研计划
有空余时间按照列的三方库列表，归纳里面的实现，出一个文档。
