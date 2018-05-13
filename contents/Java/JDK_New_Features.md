
## JDK 1.5 新特性
### 1. 自动装箱与拆箱

### 2. 枚举

### 3. 静态导入

### 4. 可变参数（Varargs）

### 5. 内省（Introspector）

### 6. 泛型(Generic) 

### 7. For-Each循环 


## JDK 1.6 新特性
### 1. 脚本语言支持

### 2. 更简单,更强大的JAX-WS

### 3.轻量级Http Server

### 4.插入式注解处理API(Pluggable Annotation Processing API) 
插入式注解处理API(JSR 269)提供一套标准API来处理Annotations(JSR 175) 


## JDK 1.7 新特性
### 1. switch中可以使用字串

### 2. 泛型实例化类型自动推断

### 3. 自动关闭

### 4. 对Java集合（Collections）的增强支持

在JDK1.7之前的版本中，Java集合容器中存取元素的形式如下：

以List、Set、Map集合容器为例：
```
    //创建List接口对象
    List<String> list=new ArrayList<String>();
    list.add("item"); //用add()方法获取对象
    String Item=list.get(0); //用get()方法获取对象

    //创建Set接口对象
    Set<String> set=new HashSet<String>();
    set.add("item"); //用add()方法添加对象

    //创建Map接口对象
    Map<String,Integer> map=new HashMap<String,Integer>();
    map.put("key",1); //用put()方法添加对象
    int value=map.get("key");
```

在JDK1.7中，摒弃了Java集合接口的实现类，如：ArrayList、HashSet和HashMap。而是直接采用[]、{}的形式存入对象，采用[]的形式按照索引、键值来获取集合中的对象，如下：
```
      List<String> list=["item"]; //向List集合中添加元素
      String item=list[0]; //从List集合中获取元素

      Set<String> set={"item"}; //向Set集合对象中添加元素

      Map<String,Integer> map={"key":1}; //向Map集合中添加对象
      int value=map["key"]; //从Map集合中获取对象
```


## JDK 8 新特性
### 1.允许在接口中有默认方法实现

### 2. Lambda 表达式

### 3. 函数式接口

### 4. 方法和构造函数引用

### 5. Lambda的范围

### 6. 内置函数式接口

### 7. Streams

### 8. Parallel Streams

### 9. Map

### 10. 时间日期API


## JDK 9 新特性
### 1. Jigsaw 项目
模块化源码

### 2. 简化进程API

### 3. 轻量级 JSON API

### 4. 钱和货币的API




