---
title: Android - ToolBar searchView 实现搜索框
date: 2017-05-02 19:48:11
tags: Android
categories: Android
---


在ToolBar上可以很方便的用widget.SearchView实现搜索功能。
一般情况下，SearchView通常有两种实现方案：


* ### 在当前Activity处理搜索逻辑

1. 首先在menu中新增item
    ```
    <item
        android:id="@+id/toolbar_search"
        android:title="Search"
        app:actionViewClass="android.support.v7.widget.SearchView"
        app:showAsAction="always"/>
    ```
2. onCreateOptionsMenu中增加如下代码：
    ```
        public boolean onCreateOptionsMenu(Menu menu){
        getMenuInflater().inflate(R.menu.tool_bar,menu);
        //Toolbar的搜索框
        MenuItem searchItem = menu.findItem(R.id.toolbar_search);
        SearchView searchView = null;
        if (searchItem != null) {
            searchView = (SearchView) searchItem.getActionView();
        }
        
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                //处理搜索结果    
                Toast.makeText(MainActivity.this,"搜索: " + query,Toast.LENGTH_LONG).show();
                return false;
            }
            @Override
            public boolean onQueryTextChange(String s) {
                return false;
            }
        });
        return  true;
    }
    ```


* ### 在新Activity处理搜索逻辑

1. 在menu中新增item（同上）
2. 在onCreateOptionsMenu中增加如下代码：
    ```
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater menuInflater = getMenuInflater();
        menuInflater.inflate(R.menu.dashboard, menu);
    
        MenuItem searchItem = menu.findItem(R.id.action_search);
        SearchManager searchManager = (SearchManager) MainActivity.this.getSystemService(Context.SEARCH_SERVICE);
        SearchView searchView = null;
        if (searchItem != null) {
            searchView = (SearchView) searchItem.getActionView();
        }
        if (searchView != null) {
            searchView.setSearchableInfo(searchManager.getSearchableInfo(MainActivity.this.getComponentName()));
        }
        return super.onCreateOptionsMenu(menu);
    }
    ```
3. 在res/xml/下新建searchable.xml文件，内容如下：
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <searchable xmlns:android="http://schemas.android.com/apk/res/android"
    android:hint="@string/search_hint"
    android:label="@string/app_name" />
    ```
4. 新建处理搜索结果的activity：SearchResultsActivity

5. AndroidManifest文件添加下面的代码：
    ```
    <activity
        android:name="com.example.SearchResultsActivity"
        android:label="@string/app_name">
        <intent-filter>
            <action android:name="android.intent.action.SEARCH" />
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
        </intent-filter>

        <meta-data
            android:name="android.app.searchable"
            android:resource="@xml/searchable" />
    </activity>
    ```
6. 在SearchResultsActivity里面处理搜索结果


