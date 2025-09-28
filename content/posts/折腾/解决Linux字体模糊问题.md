+++
title = "解决Linux字体模糊问题"
date = "2025-09-27T00:00:00+08:00"
categories = ["折腾"]
tags = ["ArchLinux", "i3", "font"]
+++

## 问题

在使用 Firefox 时, 发现中文字体比较模糊, 受不了。最终发现是字体缩小了, 屏幕的像素不够密集导致的。如果放大一倍, 字体会比较清晰。

## 我的环境

ArchLinux + i3 + 2k,27 寸屏

## 最终方案

觉得苹方的中文字体风格个人比较喜欢。所以使用了[这个配置方案](https://github.com/wxmup/linux-fonts-from-apple)

## 有用的参考链接

-   [字体配置/中文](https://wiki.archlinuxcn.org/wiki/%E5%AD%97%E4%BD%93%E9%85%8D%E7%BD%AE/%E4%B8%AD%E6%96%87): 按照这里配置也不错。
-   [lilydjwg 的字体配置](https://github.com/lilydjwg/dotconfig/tree/base/fontconfig): Dejavu + 思源字体
-   [Linux 上的字体配置与故障排除](https://blog.lilydjwg.me/tag/%E5%AD%97%E4%BD%93)
-   [文泉驿正黑在中文小字体时, 会发虚, 所以使用文泉驿微米雅黑](https://blog.csdn.net/weixin_34433661/article/details/116759676)*

一些用于测试浏览器中文字体渲染效果的测试页:
-   [HTML 文字标记风格](https://ethantw.github.io/Han/latest/test-hans.html)
-   [Unicode CJK 扩展区](https://ctext.org/font-test-page/zhs)
-   [字体大小](https://www.csie.ntu.edu.tw/~piaip/fontsize/)
-   [字重](https://font.yukonga.top/)

发现问题: 测试网页的字体比较清晰, 但是有些网页字体会模糊, 猜测是网页字体大小不正确。
