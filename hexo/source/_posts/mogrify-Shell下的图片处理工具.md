---
title: mogrify - Shell下的图片处理工具
date: 2016-08-21 16:34:41
tags:
categories: 工具
---

**mogrify** 是一个图形处理应用程序，Ubuntu 系统已经集成了此工具。它支持对图片的 `resize`, `blur`, `crop`, `despeckle`, `dither`, `draw on`, `flip`, `re-sample` 等操作。

还有 mogrify 很类似的图形处理工具 [**convert**][convert]，只不过 mogrify 默认是直接在原图上进行修改（除非你使用 `-format` 后缀参数），而 convert 则不是。

**mogrify** 功能强大，并且也容易上手，下面我举一些简单的例子：

- 等比例缩放图片

```
$ mogrify -resize 50% rose.jpg
```

![rose](/images/tools/mogrify/rose.png)
![After 50%](/images/tools/mogrify/缩放后的rose.png)

<!-- more -->

- 随意改变图片大小

```
$ mogrify -resize 256x256 *.jpg
```

- 最后，我们把所有 PNG 的图片转化为 JPEG 的图片

```
$ mogrify -format jpg *.png
$ file *
rose.jpg:     JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 70x46, frames 3
rose.png:     PNG image data, 70 x 46, 8-bit/color RGB, non-interlaced
```

如果你想看更多的例子，有篇英文文章 [Graphics from the command line 命令行下的图形处理][1] 应该可以帮到你。

-------------------

### **Reference**
[Example of mogrify in Imagemagick](http://www.imagemagick.org/script/mogrify.php)

[convert]: http://www.imagemagick.org/script/convert.php
[1]:http://www.ibm.com/developerworks/library/l-graf/?ca=dnt-428
