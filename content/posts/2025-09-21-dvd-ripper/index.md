---
title: "記錄怎麼把有ARccOS保護的DVD備份起來"
subtitle: ""
date: 2025-09-21T20:48:39+08:00
lastmod: 2025-09-21T20:48:39+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: ""
license: ""
images: []

tags: ["life", "dvd", "ripper"]
categories: ["hack"]

featuredImage: ""
featuredImagePreview: ""

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: false
code:
  copy: true
  maxShownLines: 50
math:
  enable: false
  # ...
mapbox:
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # located in "assets/"
    # Or
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # located in "assets/"
    # Or
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
  # ...
---

記錄怎麼把有ARccOS保護的DVD備份起來

<!--more-->

## 緣由

我跑去日本買了一些DVD回台灣，然後想要把它們備份起來到NAS裡面，畢竟我常用的電腦是沒有光碟機的。但是就遇到ARccOS類型的保護，讓光碟機其實可以撥放，但是就沒有辦法備份，所以有了這篇文。

## 感謝

目前看到最成功的方案就是使用[這篇文](https://cmdln.org/2010/01/22/backing-up-disney-dvds/)提到的方案，但是我後續有嘗試其他種類讓成功率更高，效率相對也會高義點的方案。

## 步驟

1. 先用ddrescue把資料盡可能複製出來，而且強調一下，最好用`-R`的參數，讓他從後面往前讀，成功率相對比較高

```bash
ddrescue -n -b 2048 -N -d -R -r3 -c512 /dev/sr0 output.iso output.log
```

2. 然後再用ddrescue正向走一次整個流程，把bad sector盡量複製出來

```bash
ddrescue -n -b 2048 -N -d -r3 -c512 /dev/sr0 output.iso output.log
```

3. 或是

```bash
ddrescue -b 2048 -d -r3 -c1 /dev/sr0 output.iso output.log
```

4. 基本上這樣你就會得到一個相對可用的`output.iso`，然後你可以導入你其他的DVD Ripper或是直接存起來了。

## 其他

### binwalk

如果沒辦法直接把ISO打開放PotPlayer播放，那就可能要靠binwalk把VOB直接取出來

```bash
binwalk -e --dd='mpeg' output.iso
```