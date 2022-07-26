---
title: "SAF51003I, SAF51002I02, Intel C3000 Esxi Installation"
subtitle: ""
date: 2022-07-26T17:37:55+08:00
lastmod: 2022-07-26T17:37:55+08:00
draft: false
author: ""
authorLink: ""
description: ""
license: ""
images: []

tags: ["esxi", "intel", "com", "console"]
categories: ["workaround"]

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

快速紀錄一下，這幾天玩弄 SAF51003I/SAF51002I02 心得
最主要還是紀錄要怎麼讓 ESXi 可以安裝在這個平台上面

<!--more-->

## ESXi 6.7 serial Installation

先用 Rufus 製作成 USB ，然後在修改 `EFI\boot\boot.cfg`
在 `kernelopt=...` 後面新增 `gdbPort=none logPort=none tty2Port=com2`
就可以存檔，並且使用 EFI 開機了

這邊我只測過 EFI ， Legacy BIOS Boot 不知道為甚麼不太成功

{{< admonition note "Note" >}}
一般來說應該會使用 COM1 ，但是我們遇到下面 Intel C3000 沒有 Legacy COM Port 的問題，而 ESXi 只支援 Legacy COM
{{< /admonition >}}

## Firmware Bug 0x3e2e0000

基本上這問題是因為開啟了 IOMMU / VT-d 的功能所導致的，只是說不知道為什麼 BIOS 推送記憶體的範圍表的時候會推送錯誤的訊息。
尋找以下的 BIOS 設定之後

```
IntelRCSetup >North Bridge Chipset Configuration >Memory config DFX Menu > MMIO Size / BMBOUND Base
```

修改成從 `auto` 修改成 `3072 / 1024M`，基本上就可以完工了。


## Console 按鍵沒反應

因為 Intel C3000 並沒有辦法設定成 Legacy COM (IO=3F8H,IRQ=4) ，從 BIOS SIO 設定會一直被重設回去原本的狀態。
無意間發現，設定成 `IO=2F8H,IRQ=3,4,5,6,7,9,10` 這選項，會自動分配 `IRQ=3` 給 COM1 做使用。

而 `IO=2F8H,IRQ=3` 則是代表 Legacy COM 2 的 IO Port 以及 IRQ ， 所以變成我們直接把 COM1 透過 AMI SIO 的中斷指定，直接偽裝成 OS 中的 COM2 來使用，這樣就能透過原本的 serial console 來做相關的操作了

`Advanced > SIO > Serial 1`
將 `auto` ，修改成 `IO=2F8H,IRQ=4,5,6,7,8,9,10; DMA`

## ESXi 進階設定

ESXi 預設還是會透過 VGA 的方式來給 TTY 跟 Console 操作畫面，所以我們需要先連接網路線，讓我們能先連接到 ESXi 的網頁設定，並從 `進階設定` 中搜尋 `tty2Port` ，修改設定為 `com2` ，並重開機，就能在 serial console 操作 ESXi 相關的設定了。
