---
title: Asciidoc的Idea插件版本输出pdf乱码
date: 2024-02-01 11:00:00
keyword: 
- Idea Intellij的插件Asciidoc导出pdf时出现乱码怎么结论
- asciidoctor pdf中文乱码
tags:
- 文档
- Asciidoc
- Idea Intellij
categories:
  - ["文档", "Asciidoc"]
---

最近正在研究Asciidoc这款生成文档的工具，其代码风格和markdown类似，最大的区别就是其有配套的默认工具来识别相应的结构目录，并根据目录结构来生成html, doc, pdf。

语法简单，工具操作简单，可以解决现在markdown碰到的部分问题，且还支持Antora这样的更加灵活的工具，使得文档或者技术书籍的写作变得更加的简单。

今天这篇文章主要来解决使用过程中碰到的问题：非拉丁语系文字在生成pdf时，会变成乱码。

<!--more-->

## 解决方案

官方在文档中其实已经给出了问题的原因和解决方案，原因就是因为字体兼容性，在导出时使用的默认字体不兼容非拉丁语系文字。

那么解决方案也很明确了，更换导出时的字体，使用兼容本地文字的字体，保证导出时显示正常。

下面是官方文档和Idea插件文档中的解释

* [support-for-non-latin-languages](https://docs.asciidoctor.org/pdf-converter/latest/theme/font-support/#support-for-non-latin-languages)
* [Creating PDFs for non-latin languages and extra fonts](https://intellij-asciidoc-plugin.ahus1.de/docs/users-guide/features/advanced/pdf-non-latin-languages)
* [pdf-non-latin-languages](https://github.com/asciidoctor/asciidoctor-intellij-plugin/blob/4858f780f2c0630697643d0d53e5bcd6abd2db9b/doc/users-guide/modules/ROOT/pages/features/advanced/pdf-non-latin-languages.adoc#L52)

Idea插件文档中提到了一个文件`.asciidoctorconfig`,这个文件是给`asciidoctor`使用，也就是导出时使用的。

并且也给出了下载地址 [KaiGenGothicCN字体下载](https://github.com/minjiex/kaigen-gothic/tree/master/dist/CN)

下载完成后形成如下目录
```
├──document
│  ├──config
│  │  ├──fonts
│  │  │  ├──KaiGenGothicCN-Bold.ttf
│  │  │  ├──KaiGenGothicCN-ExtraLight.ttf
│  │  │  ├──KaiGenGothicCN-Heavy.ttf
│  │  │  ├──KaiGenGothicCN-Light.ttf
│  │  │  ├──KaiGenGothicCN-Medium.ttf
│  │  │  ├──KaiGenGothicCN-Normal.ttf
│  │  │  ├──KaiGenGothicCN-Regular.ttf
│  │  ├──themes
│  │  │  ├──zh_CN-theme.yml
--.asciidoctorconfig
```

`.asciidoctorconfig`类容如下
```
:pdf-fontsdir: {asciidoctorconfigdir}/config/fonts
:pdf-themesdir: {asciidoctorconfigdir}/config/themes
:pdf-theme: zh_CN
```

`zh_CN-theme.yml`类容如下
```yml
# default theme at https://github.com/asciidoctor/asciidoctor-pdf/blob/master/data/themes/default-theme.yml
#https://intellij-asciidoc-plugin.ahus1.de/docs/users-guide/features/advanced/pdf-non-latin-languages
extends: default
font:
  fallbacks:
    - kaigen-gothic-cn
  catalog:
    # These are the KaiGen Gothic CN fonts, download them from
    # https://github.com/minjiex/kaigen-gothic/tree/master/dist/CN
    kaigen-gothic-cn:
      normal: KaiGenGothicCN-Regular.ttf
      bold: KaiGenGothicCN-Bold.ttf
      italic: KaiGenGothicCN-Regular.ttf
      bold_italic: KaiGenGothicCN-Bold.ttf
base:
  font_family: kaigen-gothic-cn
```