+++
title = "解决Linux字体模糊问题"
date = "2025-09-27T00:00:00+08:00"
categories = ["折腾"]
tags = ["ArchLinux", "i3", "font"]
+++

## 问题

在使用 Firefox 时, 发现中文字体比较模糊, 受不了。折腾一下, 解决了问题。

## 我的环境

ArchLinux + i3 + 2k,27寸屏 (高分屏)

字体我还是喜欢 dejavu + wqy-microhei

## 为用户配置字体

创建如下文件:

```sh
~ ❯ tree ~/.config/fontconfig
/home/johan/.config/fontconfig
├── conf.d
│   ├── 00-rendering.conf
│   ├── 10-font-selection.conf
│   ├── 20-aliases.conf
│   ├── 30-font-features.conf
│   ├── 40-hidpi-optimization.conf
│   └── 50-overrides.conf
└── fonts.conf
```

00-rendering.conf:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- ============================================== -->
  <!-- 字体渲染质量设置 - 圆润平滑风格 -->
  <!-- ============================================== -->

  <!-- 启用抗锯齿 -->
  <match target="font">
    <edit name="antialias" mode="assign">
      <bool>true</bool>
    </edit>
  </match>

  <!-- 启用次像素渲染 (RGB顺序) -->
  <match target="font">
    <edit name="rgba" mode="assign">
      <const>rgb</const>
    </edit>
  </match>

  <!-- 设置微调级别为slight - 实现圆润效果 -->
  <match target="font">
    <edit name="hintstyle" mode="assign">
      <const>hintslight</const>
    </edit>
  </match>

  <!-- 启用LCD过滤增强平滑效果 -->
  <match target="font">
    <edit name="lcdfilter" mode="assign">
      <const>lcddefault</const>
    </edit>
  </match>

  <!-- 禁用内嵌点阵字体，强制使用矢量轮廓 -->
  <match target="font">
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

  <!-- 自动提示设为自动模式 -->
  <match target="font">
    <edit name="autohint" mode="assign">
      <bool>false</bool>
    </edit>
  </match>
</fontconfig>
```

10-font-selection.conf:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- ============================================== -->
  <!-- 字体替换策略 - DejaVu + 微米黑优先 -->
  <!-- ============================================== -->

  <!-- 无衬线字体优先级 -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans-serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>DejaVu Sans</string>
      <string>WenQuanYi Micro Hei</string>
      <string>WenQuanYi Zen Hei</string>
    </edit>
  </match>

  <!-- 衬线字体优先级 -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>DejaVu Serif</string>
      <string>WenQuanYi Micro Hei</string>
      <string>WenQuanYi Zen Hei</string>
    </edit>
  </match>

  <!-- 等宽字体优先级 -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>DejaVu Sans Mono</string>
      <string>WenQuanYi Micro Hei</string>
      <string>WenQuanYi Zen Hei</string>
    </edit>
  </match>
</fontconfig>
```

*See [文泉驿正黑在中文小字体时, 会发虚, 所以使用文泉驿微米雅黑](https://blog.csdn.net/weixin_34433661/article/details/116759676)*

20-aliases.conf:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- ============================================== -->
  <!-- 特定字体别名映射 -->
  <!-- ============================================== -->

  <!-- 西文字体别名 -->
  <alias binding="strong">
    <family>Helvetica</family>
    <prefer>
      <family>DejaVu Sans</family>
    </prefer>
  </alias>

  <alias binding="strong">
    <family>Arial</family>
    <prefer>
      <family>DejaVu Sans</family>
    </prefer>
  </alias>

  <alias binding="strong">
    <family>Times New Roman</family>
    <prefer>
      <family>DejaVu Serif</family>
    </prefer>
  </alias>

  <alias binding="strong">
    <family>Courier New</family>
    <prefer>
      <family>DejaVu Sans Mono</family>
    </prefer>
  </alias>

  <!-- 中文字体别名 -->
  <alias binding="strong">
    <family>SimSun</family>
    <prefer>
      <family>WenQuanYi Micro Hei</family>
    </prefer>
  </alias>

  <alias binding="strong">
    <family>SimHei</family>
    <prefer>
      <family>WenQuanYi Micro Hei</family>
    </prefer>
  </alias>

  <alias binding="strong">
    <family>Microsoft YaHei</family>
    <prefer>
      <family>WenQuanYi Micro Hei</family>
    </prefer>
  </alias>

  <alias binding="strong">
    <family>PingFang SC</family>
    <prefer>
      <family>WenQuanYi Micro Hei</family>
    </prefer>
  </alias>
</fontconfig>
```

30-font-features.conf:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- ============================================== -->
  <!-- 字体特性设置 -->
  <!-- ============================================== -->

  <!-- DejaVu 等宽字体连字特性 -->
  <match target="font">
    <test name="family" compare="eq">
      <string>DejaVu Sans Mono</string>
    </test>
    <edit name="fontfeatures" mode="append">
      <string>calt on</string>  <!-- 上下文替代 -->
      <string>liga on</string>  <!-- 标准连字 -->
    </edit>
  </match>

  <!-- 中文斜体修复 -->
  <match target="font">
    <test name="lang" compare="contains">
      <string>zh</string>
    </test>
    <test name="family" compare="contains">
      <string>WenQuanYi</string>
    </test>
    <edit name="globaladvance" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

  <!-- 小字号 hinting 优化 -->
  <match target="font">
    <test name="pixelsize" compare="less_eq">
      <double>10</double>
    </test>
    <edit name="hintstyle" mode="assign">
      <const>hintmedium</const>
    </edit>
  </match>
</fontconfig>
```

40-hidpi-optimization.conf:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- ============================================== -->
  <!-- 高分屏优化 (针对27寸2K) -->
  <!-- ============================================== -->

  <!-- 确保在高DPI下使用矢量字体 -->
  <match target="pattern">
    <test name="pixelsize" compare="more_eq">
      <double>16</double>
    </test>
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

  <!-- 中等字号 hinting 优化 -->
  <match target="font">
    <test name="pixelsize" compare="less_eq">
      <double>12</double>
    </test>
    <edit name="hintstyle" mode="assign">
      <const>hintmedium</const>
    </edit>
  </match>

  <!-- 超小字号使用完整 hinting -->
  <match target="font">
    <test name="pixelsize" compare="less_eq">
      <double>8</double>
    </test>
    <edit name="hintstyle" mode="assign">
      <const>hintfull</const>
    </edit>
  </match>
</fontconfig>
```

50-overrides.conf:

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- ============================================== -->
  <!-- 特殊应用覆盖设置 -->
  <!-- ============================================== -->

  <!-- 终端等宽字体优化 -->
  <match target="pattern">
    <test name="family" compare="eq">
      <string>DejaVu Sans Mono</string>
    </test>
    <test name="pixelsize" compare="less_eq">
      <double>14</double>
    </test>
    <edit name="hintstyle" mode="assign">
      <const>hintmedium</const>
    </edit>
  </match>

  <!-- 中文UI字体优化 -->
  <match target="pattern">
    <test name="lang" compare="contains">
      <string>zh</string>
    </test>
    <test name="family" compare="contains">
      <string>WenQuanYi</string>
    </test>
    <edit name="hintstyle" mode="assign">
      <const>hintslight</const>
    </edit>
  </match>
</fontconfig>
```

---

相关文件在[这里](https://github.com/JohanChane/dotfiles/tree/main/.config/fontconfig)

## 设置 X Windows 的字体渲染

~/.Xresources:

```
! 字体渲染基础设置
Xft.dpi: 96                   ! 使用默认值 96
Xft.antialias: true           ! 启用抗锯齿
Xft.hinting: true             ! 启用微调
Xft.rgba: rgb                 ! 次像素渲染RGB顺序
Xft.lcdfilter: lcddefault     ! LCD过滤，增强平滑效果
```

加载 X Window 系统的资源配置:

```
exec_always --no-startup-id xrdb -merge ~/.Xresources
```

## 使之生效

```sh
fc-cache -fvr
systemctl reboot
```
