# 解决hashMap线程不安全的方法

```java
 private static ConcurrentHashMap<String,String> map=new  ConcurrentHashMap<>();

 private static Map<String,String> map=Collections.synchronizedMap(new HashMap<>());

```

