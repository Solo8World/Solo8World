---
title: Markdown语法与Pandoc转换
date: 2020-09-08 17:02:59
tags:
- 文档
- 工具
categories:
- [技术,工具]
toc: true
description: markdown极简主义标记语言，能帮我们进行便捷的文字排版，文档写作居家必备。pandoc是约翰麦克法兰（一位伯克利哲学系的教授）开发的一个文本转换工具
---
> 前言： markdown极简主义标记语言，能帮我们进行便捷的文字排版，文档写作居家必备。
语法比较简单，本文不再作过多赘述。重点是pandoc，pandoc是约翰麦克法兰（一位伯克利哲学系的教授）开发的一个文本转换工具，
能帮我们将md文件转换为word、pdf、html，甚至ppt，当然，不止如此，你能想到的它都可以做到，号称文档转换界的瑞士军刀。
# 文本标记语言 `Markdown`
 
> 最流行的轻量[文本标记语言](https://baike.baidu.com/item/%E6%A0%87%E8%AE%B0%E8%AF%AD%E8%A8%80/5964436?fr=aladdin)

## 常用语法：
<img src="https://s1.ax1x.com/2020/09/16/wcHHDH.png" width="100%">

## 更多请参考  
[Markdown 教程| 菜鸟教程](https://www.runoob.com/markdown/md-tutorial.html)

***
# 标记语言转换工具`Pandoc`
* **简介** 
  >[Pandoc](https://pandoc.org/) 是由[John MacFarlane](https://johnmacfarlane.net/macfarlane-cv.html) 开发的标记语言转换工具，可实现不同标记语言间的格式转换，堪称该领域中的“瑞士军刀”。  
            
  <img src="https://s1.ax1x.com/2020/09/16/wcbFVs.jpg" width="10%"> 



## **安装**
   * 点击进入二进制包[下载页面](https://github.com/jgm/pandoc/releases/tag/2.10.1) 选取对应包下载安装即可  
  

## **使用**  

   * markdown转word  
   
   ```
   pandoc git-extend.md  -o md_convert_doc.docx
   ```

   * markdown转html
   
   ```
   pandoc git-extend.md  -o md_convert_html.html
   ```

