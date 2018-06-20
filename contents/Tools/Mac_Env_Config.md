# 在 macOS High Sierra 10.13 搭建Java开发环境

## 安装 iTerm2
推荐 [iTerm2](https://www.iterm2.com)，iTerm2 功能强大，可以替代系统默认的命令行终端。下载解压后，将 iTerm2 直接拖入"应用程序"目录。

## 安装 Xcode
[Xcode](https://itunes.apple.com/cn/app/xcode/id497799835) 是苹果出品的包含一系列工具及库的开发套件。通过 AppStore 安装最新版本的 Xcode(9.0)。我们一般不会用 Xcode 来开发后端项目。但这一步也是必须的，因为 Xcode 会附带安装一些如 Git 等必要的软件。

## 安装 Command Line Tools for Xcode
这一步会帮你安装许多常见的基于 Unix 的工具。Xcode 命令行工具作为 Xcode 的一部分，包含了 GCC 编译器。在命令行中执行以下命令即可安装：
```
# 安装 Xcode Command Line Tools
$ xcode-select --install
```

当 Xcode 和 Xcode Command Line Tools 安装完成后，你需要启动 Xcode，并点击同意接受许可协议，然后关闭 Xcode 就可以了。这一步骤也是必须的，否则 Xcode 包含的一系列开发工具都将不可用。

## 安装 Homebrew
[Homebrew](https://brew.sh/index_zh-cn.html) 作为 macOS 不可或缺的套件管理器，用来安装、升级以及卸载常用的软件。在命令行中执行以下命令即可安装：
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
安装后可以修改 Homebrew 源，国外源一直不是很给力，这里我们将 Homebrew 的 git 远程仓库改为中国科学技术大学开源软件镜像：

## 安装一些必要的工具包
```
brew install wget
brew install autoconf
brew install openssl
```

## Java环境配置
### JDK
JDK在Mac系统中，其实有两个路径：一个是默认的，一个是用户自己下载安装的JDK。

想查看默认使用的是哪个JDK，只需在终端中输入whereis java就能看到路径，然后用ls -l则能看到真实路径，命令如下：
```
$ whereis java
/usr/bin/java
$ ls -l  /usr/bin/java

```
如果是从Oracle下载的idk，且想要更新的话，则首先需要修改jdk的环境变量。oracle下载的，默认会安装在：/Library/Java/JavaVirtualMachines/ 目录下。

#### 1. 下载JDK 安装包
[点此](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载JDK 安装包。下载成功后直接点击安装包进行安装。

打开Mac自带终端Terminal，进入当前用户主目录：
```
$ cd ~
```
默认用户目录则不需要

#### 2. 创建.bash_profile文件
如果你是第一次配置环境变量，可以使用
```
$ touch .bash_profile
```
创建一个.bash_profile的隐藏配置文件，如果是已存在的配置文件，则使用
```
$ open -e .bash_profile
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
$ source .bash_profile
```

#### 5.验证
输入如下命令：
```
$ java -version
```
或者：
```
$ echo $JAVA_HOME
```

### Maven
#### 1. 下载Maven
[点此](http://maven.apache.org/download.cgi)下载Maven安装包。

#### 2. 解压缩
解压上一步下载的gz文件，命令如下：
```
$ tar xzvf apache-maven-3.5.3-bin.tar.gz
```

#### 3. 添加环境变量

添加bin到环境变量中，需要编辑 ```.bash_profile```文件，命令如下：
```
$ vi .bash_profile
```

添加内容如下：
```
MAVEN_HOME=/Users/ricky/Data/Software/Maven/apache-maven-3.5.3  
PATH=$PATH:$MAVEN_HOME/bin  
  
export MAVEN_HOME  
export PATH
```
保存并退出。

#### 4. 重新加载.bash_profile
使用"source .bash_profile"使配置生效，如下：
```
$ source .bash_profile
```

#### 5.验证
输入如下命令：
```
$ mvn -v
```
或者：
```
$ echo $MAVEN_HOME
```

### IntelliJ IDEA
#### 1. 下载
[点此](https://www.jetbrains.com/idea/download/) 下载IntelliJ IDEA安装包。

#### 2. 激活方式
激活方式如下：
* 通过License Server：http://intellij.mandroid.cn/。
* 打开 http://idea.lanyus.com/ 获取Activation coce。
* 通过License Server：http://idea.iteblog.com/key.php

#### 3. 启动JVM参数调优

#### 4. 模板配置
```
/**
 * @author Ricky Fung
 * @version 1.0
 * @since ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}
 */
```

## Git
### 1. 下载Git
[点此](https://git-scm.com/)下载Git安装包。

### 2. 初次运行 Git 前的配置
Git 提供了一个叫做 git config 的工具（译注：实际是 git-config 命令，只不过可以通过 git 加一个名字来呼叫此命令。），专门用来配置或读取相应的工作环境变量。而正是由这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

* ```/etc/gitconfig``` 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项，读写的就是这个文件。
* ```~/.gitconfig``` 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。
* 当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

#### 用户信息
第一个要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次 Git 提交时都会引用这两条信息，说明是谁提交了更新，所以会随更新内容一起被永久纳入历史记录：
```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

#### 查看配置信息
要检查已有的配置信息，可以使用 git config --list 命令：
```
$ git config --list
user.name=Scott Chacon
user.email=schacon@gmail.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...
```

### 3. 生成 ssh 密钥
#### 1.查看ssh密钥
查看是否已经有了ssh密钥，命令如下：
```
$ cd ~/.ssh
```
如果没有密钥则不会有此文件夹，有则备份删除。

#### 2.生成密钥
生成ssh密钥，命令如下：
```
$ ssh-keygen -t rsa -C “example@example.com”
```
按3次回车，密码为空。

#### 3 .添加ssh密钥
在gitlab/github个人中心```SSH keys```添加ssh密钥，这要添加的是“id_rsa.pub”里面的公钥。

