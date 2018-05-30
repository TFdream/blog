
## Java环境配置
### JDK
JDK在Mac系统中，其实有两个路径：一个是默认的，一个是用户自己下载安装的JDK。

想查看默认使用的是哪个JDK，只需在终端中输入whereis java就能看到路径，然后用ls -l则能看到真实路径，命令如下：
```
> whereis java
 /usr/bin/java
> ls -l  /usr/bin/java
```
如果是从Oracle下载的idk，且想要更新的话，则首先需要修改jdk的环境变量。oracle下载的，默认会安装在：/Library/Java/JavaVirtualMachines/中

#### 1. 进入当前用户主目录
打开Mac自带终端Terminal，进入当前用户主目录：
```
> cd ~
```
默认用户目录则不需要

#### 2. 创建.bash_profile文件
如果你是第一次配置环境变量，可以使用
```
> touch .bash_profile
```
创建一个.bash_profile的隐藏配置文件，如果是已存在的配置文件，则使用
```
> open -e .bash_profile
```

#### 3. 打开.bash_profile文件
输入jdk下面的命令，注意根据自己的目录进行调整JAVA_HOME的值，我安装的JDK1.8路径：/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home

完整配置如下：
```
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export CLASSPATH
export PATH
```
#### 4. 重新加载.bash_profile
使用"source .bash_profile"使配置生效，如下：
```
> source .bash_profile
```

#### 5.验证
输入如下命令：
```
> java -version
```
或者：
```
> echo $JAVA_HOME
```

### Maven

### Git

