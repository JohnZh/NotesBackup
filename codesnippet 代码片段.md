# 反射

匿名子类获取父类的第一个泛型参数类型，适合写 Callback 的时候使用。

```java
public Type getGenericType() {
  Type genericSuperclass = getClass().getGenericSuperclass();
  if (genericSuperclass instanceof Class) {
    throw new RuntimeException("Missing type parameter.");
  }
  ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;
  return parameterizedType.getActualTypeArguments()[0];
}
```

