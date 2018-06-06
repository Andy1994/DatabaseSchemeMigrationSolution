# FMDBMigrationManager 调研
## 0. 链接
https://github.com/layerhq/FMDBMigrationManager
## 1. 简介
FMDBMigrationManager 已一种简单而灵活的解决方案补全了 FMDB 数据迁移的空白，可以将版本化的模式管理引入新的或现有的Cocoa应用程序。
## 2. 功能
* 支持在数据库中创建和管理专用的迁移表（ schema_migrations 表）。
* 使用 SQLite 事务安全地迁移数据。
* 基本的迁移功能为以版本编码和名称作为名字的 .sql 文件。
* 支持通过实现 FMDBMigrating 协议的类作为迁移版本，而不是通过 .sql 文件。
* 支持迁移进度展示，并能通过 NSProgress 取消迁移（progress.cancelled）。
## 3. 原版功能介绍
* Supports the creation and management of a dedicated migrations table within the host database.
* Applies migrations safely using SQLite transactions.
* Basic migrations are implemented as flat SQL files with a naming convention that encodes the version and name.
* Supports code migration implementations for performing object graph migrations not expressible as SQL.
* Discovers code based migrations via Objective-C runtime introspection of protocol conformance.
* Includes a lightweight, yet rich API for introspecting schema state.
* Exposes the status of migrations in progress and supports cancellation of migration via NSProgress.
## 4. 迁移流程图
![](https://github.com/Andy1994/DatabaseSchemeMigrationSolution/blob/master/img/FMDBMigrationManager%E8%B0%83%E7%A0%94-1.png)
## 5. schema_migrations 表结构
```
CREATE TABLE schema_migrations(
	version INTEGER UNIQUE NOT NULL
);
```

## 6. 其他接口
* NSArray *migrations：
```
 **@abstract** Returns all migrations discovered by the receiver. Each object returned conforms to the `FMDBMigrating` protocol. The array is returned in ascending order by version.

 **@discussion** The manager discovers migrations by analyzing all files that end in a .sql extension in the `migrationsBundle` and accumulating all classes that conform to the `FMDBMigrating` protocol. These migrations can then be sorted and applied to the target database.

 **@note** The list of migrations is memoized for efficiency.
```
* uint64_t originVersion：`SELECT MIN(version) FROM schema_migrations`
* uint64_t currentVersion：`SELECT MAX(version) FROM schema_migrations`
* NSArray *appliedVersions：`SELECT version FROM schema_migrations`
* NSArray  *pendingVersions：`migrations removeArray: appliedVersions`
## 7. Creating an Objective-C Migration
```
@interface MyAwesomeMigration : NSObject <FMDBMigrating>
@end

@implementation MyAwesomeMigration

- (NSString *)name {
    return @"My Object Migration";
}

- (uint64_t)version {
    return 201499000000000;
}

- (BOOL)migrateDatabase:(FMDatabase *)database error:(out NSError *__autoreleasing *)error {
    // Do something awesome
    return YES;
}

@end
```
## 8. 优点
* 只有一个 .h 和 .m 文件
* 既可以通过 .sql 进行迁移，还可以通过自己写类迁移
