
## 概览
[Apache Tika](http://tika.apache.org/) 是一个内容分析工具，自带全面的parser工具类，能解析基本所有常见格式的文件，得到文件的metadata，content等内容，返回格式化信息。总的来说可以作为一个通用的解析工具。特别对于搜索引擎的数据抓去和处理步骤有重要意义。
官方介绍如下：
> The Apache Tika™ toolkit detects and extracts metadata and text from over a thousand different file types (such as PPT, XLS, and PDF). 
> All of these file types can be parsed through a single interface, making Tika useful for search engine indexing, content analysis, translation, and much more.

## 3分钟入门

### 一、 检测文件类型（The Detector Interface）

如果仅仅是使用Tika 检测文档的类型，只需要添加 ``` tika-core ```依赖即可：
```
    <dependency>
        <groupId>org.apache.tika</groupId>
        <artifactId>tika-core</artifactId>
        <version>1.16</version>
    </dependency>
```

Apache Tika 官方提供的 [Detection](http://tika.apache.org/1.16/detection.html) demo如下：
```
package com.mindflow;

import org.apache.tika.config.TikaConfig;
import org.apache.tika.exception.TikaException;
import org.apache.tika.io.TikaInputStream;
import org.apache.tika.metadata.Metadata;
import org.apache.tika.mime.MediaType;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;

/**
 * @author Ricky Fung
 *
 */
public class App {

    public static void main( String[] args ) throws TikaException, IOException {

        TikaConfig tika = new TikaConfig();

        //1. file
        for (File f : myListOfFiles) {
            Metadata metadata = new Metadata();
            metadata.set(Metadata.RESOURCE_NAME_KEY, f.toString());
            MediaType mimetype = tika.getDetector().detect(
                    TikaInputStream.get(f), metadata);
            System.out.println("File " + f + " is " + mimetype);
        }

        //2. stream
        for (InputStream is : myListOfStreams) {
            MediaType mimetype = tika.getDetector().detect(
                    TikaInputStream.get(is), new Metadata());
            System.out.println("Stream " + is + " is " + mimetype);
        }
    }
}

```

接下来，我们列出指定的目录下的文件的类型，代码如下：
```
package com.mindflow;

import org.apache.tika.config.TikaConfig;
import org.apache.tika.exception.TikaException;
import org.apache.tika.io.TikaInputStream;
import org.apache.tika.metadata.Metadata;
import org.apache.tika.mime.MediaType;
import java.io.File;
import java.io.IOException;

/**
 * @author Ricky Fung
 */
public class TikaDemo {

    public static void main(String[] args) throws TikaException, IOException {

        File dir = new File("F:\\backup");
        File[] files = dir.listFiles();

        TikaConfig tika = new TikaConfig();
        for (File f : files) {
            if (f.isDirectory()) {
                continue;
            }
            Metadata metadata = new Metadata();
            metadata.set(Metadata.RESOURCE_NAME_KEY, f.toString());
            MediaType mimetype = tika.getDetector().detect(TikaInputStream.get(f), metadata);
            System.out.println("File " + f + " is " + mimetype);
        }

    }
}

```

输出结果如下：
```
File F:\backup\40144.jpg is image/jpeg
File F:\backup\40144.png is image/jpeg
File F:\backup\abc.txt is text/plain
File F:\backup\Java面试题.txt is text/plain
File F:\backup\Vegas.rar is application/x-rar-compressed
File F:\backup\架构文档.docx is application/vnd.openxmlformats-officedocument.wordprocessingml.document
File F:\backup\PRD.doc is application/msword
File F:\backup\蔡学镛架构设计方法-2014-8-17.pdf is application/pdf
File F:\backup\demo.txt is text/plain
```

### 二、 解析文件内容（The Parser interface） 

如果你打算使用Tika 解析文档的内容，你需要添加 ```tika-parsers``` 依赖：
```
  <dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>1.16</version>
  </dependency>
````

Apache Tika 文件内容解析的核心API类是 ```org.apache.tika.parser.Parser```，它通过``` Java SPI ```机制来扩展，在[tika-parsers/src/main/resources/META-INF/services/org.apache.tika.parser.Parser](https://gitbox.apache.org/repos/asf?p=tika.git;a=blob;f=tika-parsers/src/main/resources/META-INF/services/org.apache.tika.parser.Parser;hb=refs/heads/master) 中列举了Tika内置的所有Parser实现类，当然如果没有适合我们需求的Parser，可以通过 Java SPI 挂载我们自己的实现类。

Tika Parser API的使用比较简单，固定套路如下：
```
Parser parser = ...;
InputStream stream = ...;      // open the stream
try {
    parser.parse(stream, ...); // parse the stream
} finally {
    stream.close();            // close the stream
}
```

