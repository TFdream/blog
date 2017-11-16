## 概述
本文主要讲解Guava中的Joiner和Splitter 的用法，通过使用Joiner将集合转换为String，使用Splitter将字符串转换为集合。

## 实战
```
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>19.0</version>
</dependency>
```

### 1.Joiner
```
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    String result = Joiner.on(",").join(names);
```

### 2. Splitter
MapJoinner和MapSplitter
对于MapJoinner和MapSplitter的最好案例就是url的param编码。
生产一个查询id: 123,name: green的学生信息的url。
```
Joiner.on("&").withKeyValueSeparator("=").join(ImmutableMap.of("id", "123", "name", "green"));
```

## 参考资料
[Guava StringsExplained](https://github.com/google/guava/wiki/StringsExplained)
