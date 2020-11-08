Java 泛型通配符与上下限

---



# 上限限定和下限限定

- <? extend E> E 类型的子类或其本身

- <? super E> E 类型的基类或其本身



# 上下限带来的问题

```java
private static class A {}

private static class B extends A {}

private static class C extends B {}
```

```java
List<? extends A> list = new ArrayList<>();
list.add(new B()); // 编译错误
list.add(new A()); // 编译错误
list.add(new C()); // 编译错误
```

根据声明 list 的元素是继承 A 的类型，那为什么 ABC 都无法加入。因为编译器无法知道 list 最后到底是什么类型，虽然你指明 list 装的是任何继承 A 的元素的类型，但是类型依旧是不确定的，换句话说如果最后 list 被赋值为 List<A>，那么插入 B 和 C 的元素就是不安全的。

那这个 list 有什么用？如下

```java
private static void print(List<? extends A> list) {
  for (int i = 0; i < list.size(); i++) {
    A a = list.get(i);
    System.out.println(a);
  }
}

ArrayList<B> bList = new ArrayList<>();
ArrayList<C> cList = new ArrayList<>();
print(bList);
print(cList);
```

编译器可以知道传入的 list 里面的元素都是 A 或其派生类，那么获取出元素作为 A 类型的对象一定是合法的



与之对应的还要 super 的问题

```java
List<? super B> list = new ArrayList<>();
list.add(new A()); // 编译错误
list.add(new B());
list.add(new C());
```

编译器知道，list 可以是 List<B> 或是其超类的 List，比如 List<A>，那么这个时候如果add A 类型的元素，如果最后实例化的是 List<B> 类型（意味着里面元素都是 B 类型），那么添加 A 类型的元素就行不通了（因为 A 缺少 B 类型的一些 api）。但是添加 B 和 C（转为 B 后只是少了一些 api）是可行的。

但是在获取元素上却和 extends 是相反的，因为数据类型是某个类型的父类，那么到底有哪个超类呢，除了 Object 是肯定安全的，其他的超类型的对象根本拿来做接收都不一定正确，因此：

```java
private static void print(List<? super C> list) {
  for (int i = 0; i < list.size(); i++) {
    C c = list.get(i); // 编译错误
    Object o = list.get(i);
  }
}
```

那么这个 list 又有什么用呢？同样是作为参数使用：

```java
List<A> list = new ArrayList<>();
addB(list); // 编译错误
addC(list); // 编译错误
list.add(new B()); // 没问题
list.add(new C()); // 没问题

private static void addB(List<B> list) {
  list.add(new B());
}

private static void addC(List<C> list) {
  list.add(new C());
}

// 修改 addB 为
private static void addB(List<? super B> list) {
  list.add(new B());
}
// addB(list) 问题 fix
```

> 以上两种性质分别叫作：convariant：协变 和 contravariant：反变

这两种场景也叫作 **PECS**（Producer extends， Consumer super），即当要从集合里面获取元素，那么这个集合就会作为生产者 Producer，使用 extends；当需要往集合里面添加元素，那么集合看作消费者 Consumer，使用 super