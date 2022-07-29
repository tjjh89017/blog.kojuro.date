---
title: "USB Modeswitch on Huawei ME909s"
subtitle: ""
date: 2022-07-29T23:13:47+08:00
lastmod: 2022-07-29T23:13:47+08:00
draft: false
author: ""
authorLink: ""
description: ""
license: ""
images: []

tags: ["lte", "modem", "udev"]
categories: ["lte"]

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

簡單記錄下， debug 華為 Mini PCIe LTE module ME909s-821 在 Debian 上面的問題，同樣的可以套用到 VyOS 上面。

<!--more-->

## USB Modeswitch

華為的卡幾乎都是兩個以上的模式，有幾個系列的卡可能正常，但是 ME909s 相對起來只要切換設定檔而已，反而會造成 udev 一直重新被觸發，然後就卡住了。

ME909s 大部分都會想把 Configuration 從預設的 2 (cdc_ether) 轉換成 3 (cdc_mbim)，這時候就會需要 usb-modeswitch 的協助。

一般來說我們會新增一個檔案 `/etc/usb_modeswitch.d/12d1:15c1` ，內容如下

```
Configuration=3
```

## Issue

但是 usb-modeswitch 的 udev rule 沒有寫好，導致設定到一半就被中斷重新設定一次，進而導致設定失敗。

`/lib/udev/usb_modeswitch` 中會一直觸發到 `systemctl restart usb-modeswitch@XXX.service` ，所以 ME909s 被設定到一半就被中斷，第二次設定可能依然失敗，進而觸發第三次設定。

## Fix

把 `/lib/udev/rules.d/40-usb_modeswitch.rules` 中的

```
# Huawei ME906, ME909 (MBIM, dummy config)
ATTR{idVendor}=="12d1", ATTR{idProduct}=="15c1", RUN+="usb_modeswitch '/%k'"
```

修改成

```
# Huawei ME906, ME909 (MBIM, dummy config)
ACTION=="add", ATTR{idVendor}=="12d1", ATTR{idProduct}=="15c1", RUN+="usb_modeswitch '/%k'"
```
