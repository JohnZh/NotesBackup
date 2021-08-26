# 迁移数据库

```java
Realm.init(context);
Realm.setDefaultConfiguration(new RealmConfiguration.Builder()
                              .schemaVersion(15) //1
                              .migration(realmMigration) //2
                              .deleteRealmIfMigrationNeeded()
                              .modules(new DefaultModule())
                              .build()
                             );
```

迁移 db 至少做两件事：

1. schemaVersion + 1
2. 更新 realmMigration

```java
new RealmMigration() {
  @Override
  public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {
		if (oldVersion == 14){ // newVersion == 15
      ....
    }
  }
}
```



**如果添加新的 RealmObject，还需要更新 DefaultModule**

```java
@RealmModule(classes = {      
  ...
  MenuBanner.class,
  AppMenuBanners.class})
private static class DefaultModule {}
```



# 浏览 db 文件

Mongo Realm Studio 现在暂时不支持直接访问 device 里面的 db 文件，因此首先要把 app 里面的文件 copy 出来：

```shell
adb pull /data/data/<packagename>/files/ .
eg. adb pull /data/data/jp.co.mcdonalds.android.staging/files/ .
```

会导出一个 file/ 进入，默认文件是 default.realm，adb cmd 权限不足，先执行 **adb root**

