
## JDK Install
### 1. 下载
去Oracle官网下载最新版的安装包：http://www.oracle.com/technetwork/java/javase/downloads/index.html

### 2. 配置Java环境变量

#### 2.1 Windows
![](https://github.com/TFdream/blog/blob/master/docs/image/JAVA_HOME.png)

并在 Path 末尾加上：
```
;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
```
#### 2.2 Linux
在/etc/profile文件中添加Maven环境变量，如下所示：
```
export JAVA_HOME=/usr/share/jdk1.8.0_121
export PATH=$PATH:$JAVA_HOME/bin
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

## Maven Install
### 1. 下载安装包
在Maven官网下载最新版的安装包：http://maven.apache.org/download.cgi

### 2. 配置Maven环境变量
配置M2_HOME环境变量，指向maven的安装目录，并将bin目录追加到PATH路径中，方便在命令行调用。

#### 2.1 Windows
![](https://github.com/TFdream/blog/blob/master/docs/image/maven_home.png)

并在 Path 末尾加上：
```
;%M2_HOME%\bin;
```

#### 2.2 Linux
在/etc/profile文件中添加Maven环境变量，如下所示：
```
export M2_HOME=/opt/maven
export PATH=$PATH:$M2_HOME/bin
```

### 3. 验证
在命令行执行mvn –v，如果显示下图所示信息，说明maven已经安装成功
```
C:\Users\bingbingfeng>mvn -v
Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
Maven home: F:\Maven\apache-maven-3.5.2\bin\..
Java version: 1.8.0_121, vendor: Oracle Corporation
Java home: C:\Program Files\Java\jdk1.8.0_121\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

## 注意事项
Linux下用冒号“:”来分隔路径，Windows下使用分号“;”来分隔路径。

