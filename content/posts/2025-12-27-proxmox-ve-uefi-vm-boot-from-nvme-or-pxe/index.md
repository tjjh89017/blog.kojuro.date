---
title: "Proxmox VE UEFI VM boot from NVMe Passthrough or PXE"
subtitle: ""
date: 2025-12-27T21:26:04+08:00
lastmod: 2025-12-27T21:26:04+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: ""
license: ""
images: []

tags: ["vm", "pve", "uefi", "edk2"]
categories: ["vm"]

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

紀錄一下被Proxmox VE坑到的情況，如果你需要把NVMe Passthrough給VM，想要透過NVMe開機的情況，或是需要使用UEFI PXE的情境。

<!--more-->

## 緣起

因為最近在測試EZIO的更新，所以要使用Clonezilla Lite Server模式，需要使用PXE開機，同時因為我要測試極限速度在哪，所以直接把NVMe Passthrough給VM來做測試。

畢竟需要做一些e2e測試，來看實際上有沒有成功，所以都會嘗試開機進NVMe，但就發現不能開機，他會直接跳失敗。

同時因為Lite Server mode也要測試一下UEFI PXE開機有沒有成功進入Clonezilla模式，所以發生了問題。

這算是兩個事件，然後剛好我看到有人這樣解UEFI PXE問題，而且看了root cause之後，懷疑NVMe也是同樣問題，測試才發現可以使用相同解法。

在我找到的幾篇文裡面，也有人想要透過NVMe Passthrough開機，而那篇文沒有後續。而UEFI PXE這篇，就有Proxmox VE的staff出來研究一下情況後，有提供解法了。

## 解法

直接在VM上面新增一個`VirtIO RNG`的裝置，這樣功能就全部正常了。

## 原因

EDKII (OVMF)，也就是現在QEMU使用的UEFI loader，在某個版本之後實作了一些安全性相關的功能，而這些功能依賴亂數產生器，如果沒有Entropy來源，就會自動取消相關功能(例如：PXE開機)。這跟Secure Boot無關，而是直接該開機選項會被取消的設計。

From Fabian:
> because EDKII implemented a security hardening measure that means network booting requires a source of entropy, if none is found network booting is disabled.

## 參考

https://forum.proxmox.com/threads/unable-to-boot-vm-with-nvme-drive-passed-through.116114/
https://forum.proxmox.com/threads/uefi-pxe-boot-issues-after-upgrading-from-proxmox-ve-8-3-4-to-8-3-5.164468/
