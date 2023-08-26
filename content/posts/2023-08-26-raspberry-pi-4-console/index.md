---
title: "Raspberry Pi 4 Console"
subtitle: ""
date: 2023-08-26T23:27:55+08:00
lastmod: 2023-08-26T23:27:55+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: ""
license: ""
images: []

tags: ["arm", "arm64", "console", "raspberrypi", "linux"]
categories: ["linux", "arm", "arm64"]

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

快速記錄下 Raspberry Pi 4 的 Console/UART 等等的三小問題

<!--more-->

## Raspberry Pi 4

rpi4 的 console 是有兩個 UART 來選擇，讓你可以接到實體的 GPIO 腳位上面，透過 TTL 來看 console

BCM2711 上面有兩個主要的 UART，我們這邊用 device tree 上面的命名，分別是：
- uart0: `pl011,0xfe201000` (rpi4 預設開啟且連接到藍牙模組上， kernel 內是 ttyAMA0)
- uart1: `uart8250,mmio32,0xfe215040` (rpi4 預設開啟且連接到 GPIO 14,15， kernel 內是 ttyS0)

所以現在要思考成，我們有一個可以修改 uart0/1 的路線，分別到 GPIO 14,15 或是藍牙模組上面

## Device Tree Overlay (dt-overlay)

rpi4 這邊透過 `config.txt` 來設定 `dt-overlay` ，來切換不同的設定

預設情況下，rpi4 的 kernel cmdline 應該要設定成：
```
console=ttyS0 earlycon=uart8250,mmio32,0xfe215040
```

例如說，如果在 `config.txt` 設定 `dt-overlay=disable-bt` ，那這樣 rpi4 會將 uart1 以及藍牙模組關閉，並將 uart0 開啟接上 GPIO 14,15，這時候的 Kernel cmdline 就需要設定成：
```
console=ttyAMA0 earlycon=pl011,0xfe201000
```

另外一種是設定 uart0 為 GPIO 14,15，然後 uart1 跟藍牙模組對接，這時候要選擇 `dt-overlay=miniuart-bt`，這時候的 Kernel cmmdline 就需要設定成：

```
console=ttyAMA0 earlycon=pl011,0xfe201000
```

## 參考資料

1. rpi4 預設的 dtb [這裡](https://github.com/raspberrypi/linux/blob/655fc658a15ae7a6f37103754adb39ba52a9a14e/arch/arm/boot/dts/bcm2711-rpi-4-b.dts#L231)
2. disable-bt.dtbo 可以參照[這裡](https://github.com/raspberrypi/linux/blob/655fc658a15ae7a6f37103754adb39ba52a9a14e/arch/arm/boot/dts/overlays/disable-bt-overlay.dts)
3. miniuart-bt.dtbo 可以參照[這裡](https://github.com/raspberrypi/linux/blob/655fc658a15ae7a6f37103754adb39ba52a9a14e/arch/arm/boot/dts/overlays/miniuart-bt-overlay.dts)
