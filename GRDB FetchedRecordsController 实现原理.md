# GRDB FetchedRecordsController 实现原理
FetchedRecordsController 是依靠 TransactionObserver.swift 来监听数据库的变化，下面详细简绍 TransactionObserver.swift。

1. **DatabaseObservationBroker**
* This class provides support for transaction observers.
* Let’s have a detailed look at how a transaction observer is notified:
```
class MyObserver: TransactionObserver {
     func observes(eventsOfKind eventKind: DatabaseEventKind) -> Bool
     func databaseDidChange(with event: DatabaseEvent)
     func databaseWillCommit() throws
     func databaseDidCommit(_ db: Database)
     func databaseDidRollback(_ db: Database)
}
```
* First observer is added, and a transaction is started. At this point, there’s not much to say:
```
let observer = MyObserver()
dbQueue.add(transactionObserver: observer)
dbQueue.inDatabase { db in
    try db.execute("BEGIN TRANSACTION")
// Then a statement is executed:
	try db.execute("INSERT INTO documents ...")
```
* The observation process starts when the statement is **compiled**: sqlite3_set_authorizer tells that the statement performs insertion into the `documents` table. Generally speaking, statements may have many effects, by the mean of foreign key actions and SQL triggers. SQLite takes care of exposing all those effects to sqlite3_set_authorizer.
* When the statement is **about to be executed**, the broker queries the observer.observes(eventsOfKind:) method. If it returns true, the observer is **activated**.
* During the statement **execution**, SQLite tells that a row has been inserted through sqlite3_update_hook: the broker calls the observer.databaseDidChange(with:) method, if and only if the observer has been activated at the previous step.
* Now a savepoint is started:
`try db.execute("SAVEPOINT foo")`
* Statement compilation has sqlite3_set_authorizer tell that this statement begins a “foo” savepoint.
* After the statement **has been executed**, the broker knows that the SQLite [savepoint stack](https://www.sqlite.org/lang_savepoint.html) contains the “foo” savepoint.
* Then another statement is executed:
`try db.execute("INSERT INTO documents ...")`
* This time, when the statement is **executed** and SQLite tells that a row has been inserted, the broker buffers the change event instead of immediately notifying the activated observers. That is because the savepoint can be rollbacked, and GRDB guarantees observers that they are only notified of changes that have an opportunity to be committed.
* The savepoint is released:
`try db.execute("RELEASE SAVEPOINT foo")`
* Statement compilation has sqlite3_set_authorizer tell that this statement releases the “foo” savepoint.
* After the statement **has been executed**, the broker knows that the SQLite [savepoint stack](https://www.sqlite.org/lang_savepoint.html) is now empty, and notifies the buffered changes to activated observers.
* Finally the transaction is committed:
`try db.execute("COMMIT")`
* During the statement **execution**, SQlite tells the broker that the transaction is about to be committed through sqlite3_commit_hook. The broker invokes observer.databaseWillCommit(). If the observer throws an error, the broker asks SQLite to rollback the transaction. Otherwise, the broker lets the transaction complete.
* After the statement **has been executed**, the broker calls observer.databaseDidCommit().


2. **protocol TransactionObserver**
* A transaction observer is notified of all changes and transactions committed or rollbacked on a database.

**func observes(eventsOfKind eventKind: DatabaseEventKind) -> Bool**
* Filters database changes that should be notified the the databaseDidChange(with:) method.

**func databaseDidChange(with event: DatabaseEvent)**
* Notifies a database change (insert, update, or delete).
* The change is pending until the current transaction ends. See databaseWillCommit, databaseDidCommit and databaseDidRollback.
* This method is called in a protected dispatch queue, serialized will all database updates.
* The event is only valid for the duration of this method call. If you need to keep it longer, store a copy: `event.copy()`
* The observer has an opportunity to stop receiving further change events from the current transaction by calling the stopObservingDatabaseChangesUntilNextTransaction() method.
* - **warning**: this method must not change the database.

**func databaseWillCommit() throws**
* When a transaction is about to be committed, the transaction observer has an opportunity to rollback pending changes by throwing an error.
* This method is called on the database queue.
* - **warning**: this method must not change the database.
* - **throws**: An eventual error that rollbacks pending changes.

**func databaseDidCommit(_ db: Database)**
* Database changes have been committed.
* This method is called on the database queue. It can change the database.

**func databaseDidRollback(_ db: Database)**
* Database changes have been rollbacked.
* This method is called on the database queue. It can change the database.


3. **SQLite hooks**
**installCommitAndRollbackHooks**
```
func installCommitAndRollbackHooks() {
        let brokerPointer = Unmanaged.passUnretained(self).toOpaque()

        sqlite3_commit_hook(database.sqliteConnection, { brokerPointer in
            let broker = Unmanaged<DatabaseObservationBroker>.fromOpaque(brokerPointer!).takeUnretainedValue()
            do {
                try broker.databaseWillCommit()
                broker.transactionState = .commit
                // Next step: updateStatementDidExecute()
                return 0
            } catch {
                broker.transactionState = .cancelledCommit(error)
                // Next step: sqlite3_rollback_hook callback
                return 1
            }
        }, brokerPointer)        

        sqlite3_rollback_hook(database.sqliteConnection, { brokerPointer in
            let broker = Unmanaged<DatabaseObservationBroker>.fromOpaque(brokerPointer!).takeUnretainedValue()
            switch broker.transactionState {
            case .cancelledCommit:
                // Next step: updateStatementDidFail()
                break
            default:
                broker.transactionState = .rollback
                // Next step: updateStatementDidExecute()
            }
        }, brokerPointer)
    }
```

**installUpdateHook**
```
func installUpdateHook() {
        let brokerPointer = Unmanaged.passUnretained(self).toOpaque()        

        sqlite3_update_hook(database.sqliteConnection, { (brokerPointer, updateKind, databaseNameCString, tableNameCString, rowID) in
            let broker = Unmanaged<DatabaseObservationBroker>.fromOpaque(brokerPointer!).takeUnretainedValue()
            broker.databaseDidChange(with: DatabaseEvent(
                kind: DatabaseEvent.Kind(rawValue: updateKind)!,
                rowID: rowID,
                databaseNameCString: databaseNameCString,
                tableNameCString: tableNameCString))
        }, brokerPointer)
}
```
