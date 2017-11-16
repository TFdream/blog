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

#### MapJoinner
对于MapSplitter的最好案例就是url的param编码。
生产一个查询id: 123,name: green的学生信息的url。
```
Joiner.on("&").withKeyValueSeparator("=").join(ImmutableMap.of("id", "123", "name", "green"));
```


### 2. Splitter
#### 1. 基本用法
```
        String str = "a,b,     c,,d";
        Iterable<String> result = Splitter.on(',')//设置分隔符
                        .split(str); //要分割的字符串
        System.out.println("--start--");
        for (String s : result) {
            System.out.println(s);
        }
```

#### 2. 去除分隔结果为空格的数据
```
        String str = "a,b,     c,,d";
        Iterable<String> result = Splitter.on(',')//设置分隔符
                .omitEmptyStrings() //用于去除为空格的分割结果
                .split(str); //要分割的字符串
        System.out.println("--start--");
        for (String s : result) {
            System.out.println(s);
        }
```

#### 3. 去除分隔结果的前后空格
```
        String str = "a, b ,     c,,d";
        Iterable<String> result = Splitter.on(',')//设置分隔符
                .trimResults() //去除前后空格
                .omitEmptyStrings() //用于去除为空格的分割结果
                .split(str); //要分割的字符串
        System.out.println("--start--");
        for (String s : result) {
            System.out.println(s);
        }
```
#### 4. Splitter方法把string转换为list
```
    String input = "apple - banana - orange";
    List<String> result = Splitter.on("-")
                          .trimResults()
                          .splitToList(input);
    for (String string : result) {
        System.out.println(string);
    }
```

#### 5. MapSplitter
```
final Map<String, String> join = Splitter.on("&").withKeyValueSeparator("=").split("id=123&name=green");
```

## 参考资料
[Guava StringsExplained](https://github.com/google/guava/wiki/StringsExplained)
