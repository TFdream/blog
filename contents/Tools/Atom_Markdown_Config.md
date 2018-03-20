为了让中文显示起来跟好看，我使用微软雅黑作为中文字体，Monaco作为英文字体，具体配置如下：

下面这些是和Markdown编辑相关的包：
* language-markdown: Markdown语法支持
* markdown-image-paste: 使用Ctrl-V将图片插入到Markdown文档中
* markdown-preview-plus：对markdown-preview的强化，更好的预览Markdown文档
* markdown-scroll-sync: 同步显示Markdown源文件和预览文件
* markdown-table-editor: 让Markdown编辑表格更轻松
* markdown-themeable-pdf: 为Markdown生成的PDF文件使用自定义主题。
* markdown-writer: 更好的使用Markdown写作
* pdf-view: 基于PDF.js的PDF预览，和markdown-themeable-pdf结合起来，可以在Atom里面直接查看生成的PDF文件

## 修改页眉
~/.atom/markdown-themeable-pdf/header.js: PDF文件页眉，我的设置如下(页眉中只显示文件名称)
```
module.exports = function (info) {
  return {
      height: '2cm',
      contents: '<div style="text-align: right;"><strong>' + info.fileInfo.name + '</strong></div>'
  };
};
```

## 修改页脚
~/.atom/markdown-themeable-pdf/footer.js: PDF文件页脚，我的设置如下(页脚中显示页码信息，Copyright，时间戳和作者名)。
```
module.exports = function (info) {
    var dateFormat = function () {
        var d = new Date();
        var mm = d.getMonth() + 1;
        var dd = d.getDate();
        var yy = d.getFullYear();
        var myDateString = yy + '-' + mm + '-' + dd;
        return  myDateString;
    };
    return {
        height: '1cm',
        contents: '<div style="float:left;">Page {{page}}/{{pages}}</div><div style="float:right;">&copy; Copyright ' + dateFormat() + ' by K.X</div>'
    };
};
```

## 导出字体
~/.atom/markdown-themeable-pdf/styles: PDF文本字体。和Atom一样，我使用微软雅黑作为中文字体，Monaco作为英文字体。
```
* {
  font-family: Monaco, Microsoft YaHei UI;
}
```


