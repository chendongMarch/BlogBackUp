---
layout: post
title: 使用传统Android组件实现高效数据加载
category: Android
tags:
  - Android
description: 使用传统Android组件实现高效数据加载
abbrlink: 3092091804
date: 2015-05-14 00:00:00
keywords:
---



## 前言
- 本文主要介绍使用ContentProvider + Sqlite + Loader等Android的基本组件实现高内聚低耦合的数据加载的数据设计模式，这是一种传统而高效的数据加载，熟悉这种模式同时也是对ContentProvider的更好掌握。


## 优势
1. ContentProvider可以非常简单的调整数据源而不影响其他程序。外部程序不关心数据，只关心返回的Cursor.举个例子，ContentProvider指向的数据库可以随时改变不会影响使用者，同时ContentProvider也可以返回网络数据，使用者只要求结果是Cursor,但是对于数据来自哪里并不关心，这样就实现了数据读取和使用的分离。

2. 提供数据安全保护与权限管理，ContentProvider使用授权机制，对数据在方便访问的同时进行了很好地保护。

3. 强大的API支持，Google官方提供了强大的API.


##Loader特点：

1. 与Activity管理同步，与Activity/Fragment生命周期同步，创建与销毁都会受到Activity/Fragment生命周期的管理。这就意味着我们不需要再去考虑何时去加载数据，使用Loader之后会结合Activity或者Fragment的生命周期自行进行加载和更新。

2. 内部线程异步加载，我们也就不需要再去开启线程获取数据解析数据，大大减少了代码量。

3. 数据源发生改变时实时更新，貌似我们自己的数据数据是无法自动更新的，不过不要紧，只需要一行简单的reStartLoader就可以手动重新加载一下。


## 数据分层
- Loader 写操作
- ContentProvider
- SQLite



## 定义关系类

- 定义一个关系类，声明表结构,Uri,授权

```
public class WebContract {
    
    public static final String Tb_History = "tb_history";
    public static final String Tb_BookMark = "tb_bookmark";
    public static final String Authority = "com.march.db_browser";
    
    public static class History implements BaseColumns {
    //content://com.march.db_browser/tb_history
    public static final Uri ContentUri = Uri.parse(
    "content://" + Authority).buildUpon().appendPath(Tb_History).build();
    public static final String Link = "link";
    public static final String Icon = "icon";
    public static final String Title = "title";
    public static final String Time = "time";
    }
    
    public static class BookMark implements BaseColumns 	{
    //content://com.march.db_browser/tb_history
    public static final Uri ContentUri = Uri.parse(
    "content://" + Authority).buildUpon().appendPath(Tb_BookMark).build();
    public static final String Link = "link";
    public static final String Icon = "icon";
    public static final String Title = "title";
    public static final String Time = "time";
    }
}
```


## 继承ContentProvider
- 实现数据提供者，设定好Code作为表资源的唯一标示，使用UriMacher匹配

```java
    public class WebContentProvider extends ContentProvider {
    
    /*为可以访问的数据库表资源定义代号，区分哪个表可以被访问*/
    private static final int CODE_BOOKMARK = 0x1;
    private static final int CODE_HISTORY = 0x2;
    /*使用UriMacher生成访问的Uri*/
    private static UriMatcher uriMatcher;
    
    /*使用静态代码块初始化UriMacher,添加*/
    static {
    uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    uriMatcher.addURI(WebContract.Authority, WebContract.Tb_BookMark, CODE_BOOKMARK);
    uriMatcher.addURI(WebContract.Authority, WebContract.Tb_History, CODE_HISTORY);
    }
    
    private MySqliteOpenHelper mySqliteOpenHelper;
    private SQLiteDatabase db;
    
    @Override
    public boolean onCreate() {
    mySqliteOpenHelper = new MySqliteOpenHelper(getContext(), "db_browser", 1);
    return false;
    }
    
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
    Cursor cursor = null;
    if (uri != null) {
    int code = uriMatcher.match(uri);
    db = mySqliteOpenHelper.getWritableDatabase();
    switch (code) {
    case CODE_BOOKMARK:
    cursor = db.query(WebContract.Tb_BookMark, null, null, null, null, null, null);
    break;
    case CODE_HISTORY:
    cursor = db.query(WebContract.Tb_History, null, null, null, null, null, null);
    break;
    }
    }
    if (cursor == null) {
    Log.i("chendong", "cursor is null");
    }
       // db.close();
    //读操作不能关闭连接，写操作需要关闭连接
    return cursor;
    }
    
    @Override
    public String getType(Uri uri) {
    return null;
    }
    
    @Override
    public Uri insert(Uri uri, ContentValues values) {
    Uri uri_return = null;
    if (uri != null) {
    int code = uriMatcher.match(uri);
    db = mySqliteOpenHelper.getWritableDatabase();
    long id = -1;
    switch (code) {
    case CODE_BOOKMARK:
    id = db.insert(WebContract.Tb_BookMark, null, values);
    break;
    case CODE_HISTORY:
    id = db.insert(WebContract.Tb_History, null, values);
    break;
    }
    if (id != -1) {
    uri_return = ContentUris.withAppendedId(uri, id);
    }
    db.close();
    }
    return uri_return;
    }
    
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
    // Implement this to handle requests to delete one or more rows.
    throw new UnsupportedOperationException("Not yet implemented");
    }
    
    
    @Override
    public int update(Uri uri, ContentValues values, String selection,
      String[] selectionArgs) {
    // TODO: Implement this to handle requests to update one or more rows.
    throw new UnsupportedOperationException("Not yet implemented");
    }
    }
```

## 使用Loader加载数据

```
    adapter = new SimpleCursorAdapter(this,
    R.layout.item_listview,
    null,
    new String[]{WebContract.History.Title, WebContract.History.Link},
    new int[]{R.id.item_listview_title, R.id.item_listview_url},
    SimpleCursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);
    listView.setAdapter(adapter);
    getLoaderManager().initLoader(0x123, bundle, this);
    
    
    //这是实现的
    implements LoaderManager.LoaderCallbacks<Cursor> 的方法
    @Override
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    int action = args.getInt("action");
    CursorLoader cursorLoader = null;
    Log.i("chendong", "action is " + action);
    if (action == 0) {
    //history
    cursorLoader = new CursorLoader(this, WebContract.History.ContentUri, null, null, null, null);
    } else {
    cursorLoader = new CursorLoader(this, WebContract.BookMark.ContentUri, null, null, null, null);
    }
    return cursorLoader;
    }
    
    @Override
    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
    Log.i("chendong", "get data ");
    adapter.changeCursor(data);
    }
    
    @Override
    public void onLoaderReset(Loader<Cursor> loader) {
    }
    
    使用
    getLoaderManager().restartLoader(0x123, bundle, this);
    重新加载数据
    

