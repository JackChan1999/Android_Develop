## **SQLite 简介**
SQLite 是一款内置到移动设备上的轻量型的数据库，是遵守ACID（原子性、一致性、隔离性、持久性）的关联式数据库管理系统，多用于嵌入式系统中

SQLite 数据库是无类型的，可以向一个integer 的列中添加一个字符串，但它又支持常见的类型比如:NULL，VARCHAR， TEXT，INTEGER，BLOB，CLOB 等

Android 系统内置了SQLite，并提供了一系列API 方便对其进行操作。

## 1. 使用SQLiteOpenHelper 创建数据库

SQLiteOpenHelper 是Android 提供的一个抽象工具类，负责管理数据库的创建、升级工作。如果我们想创建数据库，就需要自定义一个类继承SQLiteOpenHelper，然后覆写其中的抽象方法。
| 方法                    | 说明                                       |
| --------------------- | ---------------------------------------- |
| getWritableDatabase() | 打开可读写的数据库，没有权限或磁盘已满时会抛异常                 |
| getReadableDatabase() | 在磁盘空间不足时打开只读数据库，否则打开可读写数据库；有异常时返回一个只读数据库 |

创建的数据库位于：/data/data/包名/databases/目录中

### **1.1 自定义SQLiteOpenHelper**

```java
public class MySQLiteOpenHelper extends SQLiteOpenHelper {

    private static final String DB_NAME = "user.db";
    private static final int VERSION = 1;

    public MySQLiteOpenHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
    }

    public MySQLiteOpenHelper(Context context) {
        super(context, DB_NAME, null, VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        //数据库创建
        db.execSQL("create table person (_id integer primary key autoincrement, " +
                "name char(10), phone char(20), money integer(20))");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        //数据库升级
    }

    @Override
    public void onOpen(SQLiteDatabase db) {
        super.onOpen(db);
    }
}
```
**注意**：上面的代码我们只是定义了一个MySQLiteOpenHelper 类继承了SQLiteOpenHelper 类。在onCreate()方法中通过执行sql 语句实现表的创建。
##**2. 数据库的增删改查**
###2.1 SQLiteDatabase
| 方法                         | 说明            |
| -------------------------- | ------------- |
| execSQL()                  | 执行SQL语句实现增删改查 |
| Cursor rawQuery()          | 执行sql查询语句     |
| insert()                   | 插入            |
| delete()                   | 删除            |
| update()                   | 更新            |
| query()                    | 查询            |
| beginTransaction()         | 开始事务          |
| setTransactionSuccessful() | 设置事务成功        |
| endTransaction()           | 结束事务          |

###2.2 执行SQL语句实现增删改查

```sql lite
insert into person (name, phone, money) values ('张三', '159874611', 2000);//插入
delete from person where name = '李四' and _id = 4;//删除
update person set money = 6000 where name = '李四';//更新
select name, phone from person where name = '张三';//查询
alter table t_user add c_money float;//升级
create table t_user(uid integer primary key not null,c_name varchar(20),c_age integer,c_phone varchar(20))";//创建
```

```java
db.execSQL("insert into person (name, phone, money) values (?, ?, ?);", 
        new Object[]{"张三", 15987461, 75000});
Cursor cursor = db.rawQuery("select _id, name, money from person where name = ?;", 
        new String[]{"张三"});
```
###2.3 使用api实现增删改查
#### 插入

```java
//以键值对的形式保存要存入数据库的数据
ContentValues cv = new ContentValues();
cv.put("name", "刘能");
cv.put("phone", 1651646);
cv.put("money", 3500);
//返回值是改行的主键，如果出错返回-1
long i = db.insert("person", null, cv);
```

nullColumnHack：当ContentValues为空的时候，会将nullColumnHack 作为“NULL”的属性，几乎用不到，一般为null

#### 修改

```java
ContentValues cv = new ContentValues();
cv.put("money", 25000);
int i = db.update("person", cv, "name = ?", new String[]{"赵四"});
```
#### 删除

```java
//返回值是删除的行数
int i = db.delete("person", "_id = ? and name = ?", new String[]{"1", "张三"});
```

#### 查询

```java
db.query(table, columns, selection, selectionArgs, groupBy, having, orderBy, limit);
```
| 参数名           | 作用                                 |
| ------------- | ---------------------------------- |
| table         | 表名                                 |
| columns       | 查询的字段                              |
| selection     | 查询条件                               |
| selectionArgs | 填充查询条件的占位符，条件中？对应的值                |
| groupBy       | 分组查询参数                             |
| having        | 分组查询条件                             |
| orderBy       | 排序字段和规则，“字段名 desc/asc”             |
| limit         | 分页查询，“m，n”，m表示从第几条开始查，n表示一个查询多少条数据 |

#### limit分页查询 

```sql
select * from blacknumber limit pagesize offset startindex
```
pageSize每页有多少条数据，startIndex 从哪条数据开始
##**3. Cursor**
| 方法               | 说明                           |
| ---------------- | ---------------------------- |
| moveToNext()     | 游标移动到下一行位置                   |
| moveToFirst()    | 游标移动到第一行位置                   |
| moveToPrevious() | 游标移动到前一行位置                   |
| moveToLast()     | 游标移动到最后一行位置                  |
| getXxx()         | 获取列的值，如：getInt(),getString() |
| getColumnIndex() | 获取列索引                        |
| getColumnNames() | 获取所有字段名字                     |
| getCount()       | 获取游标的记录行数                    |

## 4. CursorFactory

游标工厂，创建游标

##**5. SQL语句**
- insert into person (name, phone, money) values ('张三', '159874611', 2000);
- delete from person where name = '李四' and _id = 4;
- update person set money = 6000 where name = '李四';
- select name, phone from person where name = '张三';
##**6. ContentValues**

```java
ContentValues values = new ContentValues();
values.put("name", "lisi");
values.clear();
```

##**7. 事务Transaction**	
在SQLiteDatabase 提供了对事务的支持，处于事务中的操作都是“临时性”的，只有事务提交了才会将数据保存到数据库。

事务的使用不仅可以保证数据的一致性，也可以提高批处理时的执行效率。

SQLiteDatabase 提供的beginTransaction()打开事务，endTransaction()结束事务。注意：结束事务并不代表事务提交，如果想让数据写入的数据库需要在结束事务前执行setTransactionSuccessful()方法。这是事务提交的唯一方式。

使用事务会大大提高批量处理的效率

```java
try {
    //开启事务
    db.beginTransaction();
    ...........
    //设置事务执行成功
    db.setTransactionSuccessful();
} finally{
    //关闭事务
    //如果此时已经设置事务执行成功，则sql语句生效，否则不生效
    db.endTransaction();
}
```
##**8. 两种SQLiteDatabase 的不同**
SQLiteOpenHelper 有两个方法均可返回SQLiteDatabase 对象：

- getWritableDatabase()

该方法返回的对象和另外一个方法返回的对象没有任何差异，返回的对象对数据库都可以进行读、写操作，当磁盘已满或者权限不足的情况下该方法会抛出异常。

- getReadableDatabase()

跟另外一个方法相比，在磁盘已满的情况下，该方法不会抛出异常，而是返回一个只读的数据库操作对象。根据这两种方法返回对象的差异，如果需要对数据库进行查询操作则推荐使用后者，如果添加、修改、删除数据则推荐使用前者。

##**9. SQLite优化**
- 事务机制Transaction
- 建表优化
- 索引
- 不要关联多表查询，减少链接时间，创建索引、将查询到的数据采用缓存策略等

##**10. sqlite3 工具的使用**
sqlite3 是Android 内置的操作数据库工具，使用该工具可以直接对SQLite 数据库进行操作。操作步骤：

- 在命令行界面使用adb shell 命令进入linux 内核
- 使用cd 命令进入数据库所在目录（数据库的路径为”/data/data/应用包名/databases/数据库”）
- 使用”sqlite3 数据库名”进入数据库操作模式

![这里写图片描述](http://img.blog.csdn.net/20161027125031756)

##11. DAO模式
###11.1 Dao接口
定义操作数据库的增删改查抽象方法，insert()，delete()，update()，select()
###11.2 DaoImpl
Dao接口的实现类
###11.3 DBUtil

###11.4 SQLiteOpenHelper
SQLiteOpenHelper(Context context, String name, CursorFactory factory, int version)

创建数据库
onCreate(SQLiteDatabase db)

升级数据库
onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)
##**12. ORM框架**
ORM是Object Relation Mapping的简称，即对象关系映射，说人话就是用面向对象的编程思想去操作数据库，调用一个方法即可完成数据库的增删改查操作，不用写繁琐的SQL语句，大大提高了开发效率

| 框架名称                                     | 功能描述                                     |
| :--------------------------------------- | :--------------------------------------- |
| [OrmLite](https://sourceforge.net/projects/ormlite/files/releases/com/j256/ormlite/) | JDBC和Android的轻量级ORM java包                |
| [Sugar](https://github.com/satyan/sugar) | 用超级简单的方法处理Android数据库                     |
| [GreenDAO](https://github.com/greenrobot/greenDAO) | 一种轻快地将对象映射到SQLite数据库的ORM解决方案，使用的App有：薄荷，京东 |
| [ActiveAndroid](https://github.com/pardom/ActiveAndroid) | 以活动记录方式为Android SQLite提供持久化              |
| [SQLBrite](https://github.com/square/sqlbrite) | SQLiteOpenHelper 和ContentResolver的轻量级包装  |
| [Realm](https://github.com/jhy/jsoup)    | 移动数据库：一个SQLite和ORM的替换品                   |
| [android-database-sqlcipher](https://github.com/sqlcipher/android-database-sqlcipher) | 数据库加密                                    |
| [storio](https://github.com/pushtorefresh/storio) | Beautiful API for SQLiteDatabase and ContentResolver |
| [realm-java](https://github.com/realm/realm-java) | 高性能数据库，Realm is a mobile database: a replacement for SQLite & ORMs |

##13. SQLite3中自增主键归零方法
当SQLite数据库中包含自增列时，会自动建立一个名为 sqlite_sequence 的表。这个表包含两个列：name和seq。name记录自增列所在的表，seq记录当前序号（下一条记录的编号就是当前序号加1）。如果想把某个自增列的序号归零，只需要修改 sqlite_sequence表就可以了。

```sql
UPDATE sqlite_sequence SET seq = 0 WHERE name='TableName';

<!--也可以直接把该记录删掉-->
DELETE FROM sqlite_sequence WHERE name='TableName';

<!--要想将所有表的自增列都归零，直接清空sqlite_sequence表就可以了-->
DELETE FROM sqlite_sequence;
```

##**14. 扩展阅读**
[SQLite教程](http://www.w3cschool.cn/sqlite)

[性能优化之数据库优化](http://blog.csdn.net/axi295309066/article/details/52663645?locationNum=5&fps=1)

[Android数据库高手秘籍](http://blog.csdn.net/axi295309066/article/details/53136459?locationNum=7&fps=1)

[Android数据存储与持久化](http://blog.csdn.net/axi295309066/article/details/52705843?locationNum=8&fps=1)

[MySQL数据库：SQL语句](http://blog.csdn.net/axi295309066/article/details/52945817)

[MySQL数据库：完整性约束](http://blog.csdn.net/axi295309066/article/details/52947798)