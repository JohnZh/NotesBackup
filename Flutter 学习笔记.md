# View 和 Widget

Android 界面最小单元 View，Flutter 使用 Widget，Widget 两个状态：

- Stateless：运行时不可变
- Stateful：运行时可变。eg. http 请求得到数据更新 Widget



## 如何更新 Widget

关键在于创建一个 StatefulWidget，以及属于这个 Widget 的 State<StatefulWidget>，

