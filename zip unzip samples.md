# zip samples

## zip 文件到具体 zip 文件内

```javascript
zip tmp.zip test/1 // 单个文件（含文件夹）
zip tmp.zip -j test/1 // 忽略 test/ 这个文件夹结构，只会添加 1 这个文件
zip tmp.zip -r test/ //test 路径下的所有文件
zip tmp.zip -r test/ 1.txt // 文件+路径
zip tmp.zip -m test/1 // 移动 1 到 tmp.zip，test/ 下的 1 删除了
zip tmp.zip -rq test/ // -q 不显示指令执行过程
zip tmp.zip -rv test/ // -v 显示指令执行过程和版本详细信息
```

## 追加或更新文件到 zip 文件内

```javascript
zip tmp.zip 2.txt // 如果 tmp.zip 存在就直接追加进去
zip tmp.zip -f test/1 // 只有在 1 有改变的时候才会更新进去，如果 tmp.zip 里面原先没有 1 也不会更新
zip tmp.zip -fj test/1 // -j 忽略文件夹结构
zip tmp.zip -u test/3 // 追加或更新 3，更新的效果和 -f 一样，也会先比较
```

## 删除 zip 文件内的文件

```java
zip tmp.zip -d test/3  // 删除 tmp.zip 里的 3
```

## 压缩率

```java
zip tt.zip -9 2.txt // 压缩率 1-9，最高 9，-0 代表只是存储，即压缩率 0
zip tmp.zip -9 1.txt // 可以更新某个文件的压缩率，每个文件压缩率可以不同
```

## 加密

```java
zip -P password -r tmp.zip test/
```



# unzip samples

## 查看 zip 文件内容

```javascript
unzip -l tmp.zip // 不会解压
```

## unzip 到指定目录

```javascript
unzip tmp.zip -d tmp/
unzip tmp.zip 1.txt -d tmp/
unzip -j tmp.zip -d tmp/ // 不带目录结构
unzip -o tmp.zip -d tmp/ // 不询问用户直接覆盖文件
```

## unzip 到屏幕上

```java
unzip -c t.zip // 解压显示到屏幕上，对字符做适当转换
unzip -p t.zip // 解压到管道 pipe，屏幕显示不友好
```

## unzip 排除部分文件不解压

```java
unzip t.zip -x 2 -d text/ // 排除 2 不解压
```

​	