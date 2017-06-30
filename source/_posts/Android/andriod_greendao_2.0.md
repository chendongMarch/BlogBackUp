---
layout: post
title: ORM框架－GreenDao2.0
category: Android
tags:
  - Android
  - 三方框架
description: ORM框架－GreenDao
abbrlink: 3564593419
date: 2016-07-12 00:00:00
keywords:
---

## GreenDao官网
[官网 -> http://greendao-orm.com](http://greendao-orm.com)

## GitHub
[GitHub  -> https://github.com/greenrobot/greenDAO](https://github.com/greenrobot/greenDAO)


## 性能对比
![](http://static.oschina.net/uploads/img/201403/13213257_oGSL.png)


## 介绍
1. GreenDao采用的是用java代码直接生成Bean（实体）和Dao（Data Access Object数据访问对象）的方式，都不用自己写实体了，也是一大好处，不过这也造成了理解上的难度，我刚开始用的时候就有点蒙。
2. 首先创建一个Java的Library,一定是Java的，命名为daogenerator（可以起别的名字，为了后面叙述方便）。在app下面和'java'目录同级创建'java-gen'目录，将会用来放置生成的Bean和Dao。Dao也就是数据访问对象，实体类只能描述一个对象的属性和行为，想与数据库交流就需要Dao对象，它包含对应的数据库字段和增删改查方法。


## 生成文件
- 在daogenerator库里创建'ExampleDaoGenerator.java'文件,包含main方法，构建schema，生成目录和文件

```java
public static void main(String[] args) throws Exception{
        operate();
}

private static void operate() throws Exception {
   // Schema对象，可以用来生成实体和dao
   // 两个参数分别代表：数据库版本号与自动生成bean代码的包路径。
   Schema schema = new Schema(2, "com.march.bean");
   // 默认的dao目录
   schema.setDefaultJavaPackageDao("com.march.dao");
   // 模式（Schema）同时也拥有两个默认的 flags，分别用来标示 entity 是否是 activie 以及是否使用 keep sections。
   // 可以激活预热实体，使读写更迅速
   schema.enableActiveEntitiesByDefault();
   // 这个是为了你可以在自动生成的实体类中添加自己的custom代码
   schema.enableKeepSectionsByDefault();
   // 一旦你拥有了一个 Schema 对象后，你便可以使用它添加实体（Entities）了。这里只是一个假的方法表示一下，生成实体在下一节
   generateBean(schema);
   // 最后我们将使用 DAOGenerator 类的 generateAll() 方法自动生成代码，此处你需要根据自己的情况更改输出目录（既之前创建的 java-gen)。
   // 其实，输出目录的路径可以在 build.gradle 中设置，有兴趣的朋友可以自行搜索，这里就不再详解。
   new DaoGenerator().generateAll(schema, "/Users/march/AndroidPro/Reaper/app/src/main/java-gen");
}
```


## 如何生成实体
```java
private static void addNote(Schema schema) {
        // 一个实体（类）就关联到数据库中的一张表，此处表名为「Note」（既类名）
        Entity note = schema.addEntity("Note");
        // greenDAO 会自动根据实体类的属性值来创建表字段，并赋予默认值
        // 接下来你便可以设置表中的字段,又很多链式编程的方法，结合数据库的create table操作可以设置相关的字段及约束
        note.addIdProperty().autoincrement();
        note.addBooleanProperty("isYes").primaryKey().unique();
        note.addStringProperty("text").notNull();
        // 与在 Java 中使用驼峰命名法不同，默认数据库中的命名是使用大写和下划线来分割单词的。
        // For example, a property called “creationDate” will become a database column “CREATION_DATE”.
        note.addStringProperty("comment");
        note.addDateProperty("date");
}
```


## 自定义生成的代码
- 当重新执行java代码时会覆盖生成新的文件，如果你修改了生成的类，就会被重新覆盖，解决这个问题，设置`note.setHasKeepSections(true);`会在文件中生成一些注释，在注释中间的代码将不会被覆盖，也可以设置继承，实现等。。。这样就不需要每次都修改代码了


```java
// KEEP INCLUDES - put your custom includes here
import com.march.quickrvlibs.inter.RvQuickInterface;
// KEEP INCLUDES END


// KEEP FIELDS - put your custom fields here
public static final int TYPE_SHU = 0;
public static final int TYPE_HENG = 1;
// KEEP FIELDS END


// KEEP METHODS - put your custom methods here
@Override
public int getRvType() {
        if (height > width)
            return 0;
        else
            return 1;
}
// KEEP METHODS END
```


```java
//设置支持自定义代码（或schema.enableKeepSectionsByDefault();）
note.setHasKeepSections(true);
//设置实现的接口
note.implementsInterface("RvQuickInterface", "java.io.Serializable");
//设置继承的父类
note.setSuperclass("Album");
//你也可以重新给表命名
note.setTableName("NODE");
```

## 初始化数据库

```java
private DaoSession mDaoSession;
public void setupDatabase(Context context) {
        // 通过 DaoMaster 的内部类 DevOpenHelper，你可以得到一个便利的 SQLiteOpenHelper 对象。
        // 可能你已经注意到了，你并不需要去编写「CREATE TABLE」这样的 SQL 语句，因为 greenDAO 已经帮你做了。
        // 注意：默认的 DaoMaster.DevOpenHelper 会在数据库升级时，删除所有的表，意味着这将导致数据的丢失。
        // 所以，在正式的项目中，你还应该做一层封装，来实现数据库的安全升级。
        DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(context, "notes-db", null);
        SQLiteDatabase db = helper.getWritableDatabase();
        // 注意：该数据库连接属于 DaoMaster，所以多个 Session 指的是相同的数据库连接。
        DaoMaster daoMaster = new DaoMaster(db);
        mDaoSession = daoMaster.newSession();
        // 在 QueryBuilder 类中内置两个 Flag 用于方便输出执行的 SQL 语句与传递参数的值
        QueryBuilder.LOG_SQL = true;
        QueryBuilder.LOG_VALUES = true;
}
```


## 获取Dao

```java
public WholeAlbumItemDao getWholeAlbumItemDao() {
        return mDaoSession.getWholeAlbumItemDao();
}
```



## 增
```java
//这两个方法提供简单的插入和不存在则插入存在则更新的操作
DaoHelper.get().getAlbumDetailDao().insert();
DaoHelper.get().getAlbumDetailDao().insertOrReplace();
//这两个方法是上面两个方法的加强版，支持iterable类型多个对象的插入和更新，同时是基于事务的。
DaoHelper.get().getAlbumDetailDao().insertInTx();
DaoHelper.get().getAlbumDetailDao().insertOrReplaceInTx();
//官方解释:Insert an entity into the table associated with a concrete DAO <b>without</b> setting key property.
//Warning: This may be faster, but the entity should not be used anymore. The entity also won't be attached to identity scope.
//大概意思就是小心点用，虽然速度快，但是插进去可能就拿不出来了
DaoHelper.get().getAlbumDetailDao().insertWithoutSettingPk()
```

## 删
```java
//与insert方法大同小异
DaoHelper.get().getAlbumDetailDao().delete();
DaoHelper.get().getAlbumDetailDao().deleteInTx();
//根据主键删除元素
DaoHelper.get().getAlbumDetailDao().deleteByKey();
DaoHelper.get().getAlbumDetailDao().deleteByKeyInTx();
//删除全部
DaoHelper.get().getAlbumDetailDao().deleteAll();
//也是删除，具体的区别还没弄明白
DaoHelper.get().getAlbumDetailDao().detach()
DaoHelper.get().getAlbumDetailDao().detachAll();
```

## 改
```java
DaoHelper.get().getAlbumDetailDao().update();
DaoHelper.get().getAlbumDetailDao().updateInTx();
```


## 查
- 关于Query的操作相对复杂， 下面只是比较基本的，更多的使用方法可以参照[Query文档](http://greenrobot.org/greendao/documentation/queries/)


```java
// 获取querybuilder
QueryBuilder<RecommendAlbumItem> queryBuilder = DaoHelper.get().getRecommendAlbumItemDao().queryBuilder();
// 查询条件，大于小于，and,or
queryBuilder.where(
                RecommendAlbumItemDao.Properties.Album_type.eq(""),
                queryBuilder.or(RecommendAlbumItemDao.Properties.Album_cover.gt(""),
                queryBuilder.and(RecommendAlbumItemDao.Properties.Album_cover.ge("")
               ,RecommendAlbumItemDao.Properties.Album_cover.eq(""))));
// offset limmit            
queryBuilder.offset(10).limit(10);
//排序
queryBuilder.orderAsc();
queryBuilder.orderCustom(null,null);
queryBuilder.orderDesc();
queryBuilder.build();
//查询数量
queryBuilder.count();
//返回list
queryBuilder.list();
```
