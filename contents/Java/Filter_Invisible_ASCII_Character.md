
今天兄弟团队同事报告了一个BUG，经过调试发现，由于用户输入的字符串中，包含字符0x7F，导致后端服务接口解析出错。于是就想在字符串中过滤掉这些没多大用途的字符，同时又要保留部分常用的字符，例如换行，回车和水平制表符。

上面所说的“不可见字符”，其实属于[ASCII码](https://baike.baidu.com/item/ASCII)中的控制字符，它们是0到31、以及127。

Java代码如下：
```

    public static String filter(String content){
        if (content==null || content.length()==0) {
            return content;
        }
        StringBuilder sb = new StringBuilder(content.length());
        char[] contentCharArr = content.toCharArray();
        for (int i = 0; i < contentCharArr.length; i++) {
            if (contentCharArr[i] < 0x20 || contentCharArr[i] == 0x7F) {
                continue;
            }
            sb.append(contentCharArr[i]);
        }
        return sb.toString();
    }
```
