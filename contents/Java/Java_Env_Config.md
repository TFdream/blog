
## 配置Java环境变量
### 1. 新增JAVA_HOME
![](https://github.com/TFdream/blog/blob/master/docs/image/JAVA_HOME.png)

### 2. Path 末尾加上：
```
;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
```

### 3. 验证
打开命令行窗口，执行``` java -version ```命令：
结果如下：
```
C:\Users\Ricky>java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```
