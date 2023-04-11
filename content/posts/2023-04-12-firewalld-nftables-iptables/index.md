---
title: "firewalld nftables iptables 之間的關聯"
subtitle: ""
date: 2023-04-12T00:40:50+08:00
lastmod: 2023-04-12T00:40:50+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: ""
license: ""
images: []

tags: ["linux", "firewall", "iptables", "nftables", "firewalld"]
categories: ["linux"]

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

快速紀錄一下研究 firewalld, nftables, iptables 之間的關聯，還有坑。

<!--more-->

## iptables

最早的 Linux 防火牆的指令，這邊只能算是前端指令跟語法，因為背後是 kernel module netfilter 處理的。這玩意已經被標註要棄用了，但是大家還是很習慣用這玩意。

### iptables-nft

現在很多 Linux distro 都會提供這個指令，把 iptables 的語法翻譯成 nft 的語法，然後再透過 nft 來設定。但是有一些坑，Debian, Ubuntu 這兩個可能還能用 iptables-nft 來設定防火牆，但是 RHEL, SUSE 系的可能就不行了。

## nftables

比較現代的防火牆語法跟指令，一樣是前端的部分而已，後端依然是 kernel module netfilter 處理。語法真的麻煩到靠北。

## firewalld

Redhat 那個系列的，在 nft 之前，又套一個 firewalld 的控制。

## firewalld + nftables 的坑

firewalld 跟 nftables 如果放在一起，然後 distro 又同時提供 iptables-nft 了話，就會有個很大的坑在這邊。

firewalld 跟 nftables 這兩個本身都正常沒有問題，但是他們並沒有試圖設計成跟 iptables-nft 的向下相容，所以產生了坑。

iptables-nft 只能指定 filter/nat/mangle/raw 等等的 table 名稱，且這些 table 名稱就是在 netfilter 那個流程圖中提及的幾個 table，但是 nftables 把 table 名稱跟用途做了改變，可以允許多重且自訂名稱的 table。而 firewalld 直接將自己想要設定的規則，全部放在 table firewalld 裡面處理，也就是說你想要自己額外寫 nft 規則，其實可以開新的 table 同時去 hook 跟過去一樣的 table/chain 等等的。

因為 iptables-nft 是固定限制 table 名稱的，所以像是 RHEL/CentOS/Rocky Linux 這些紅帽系的 Linux 中使用的時候，會發現 `iptables -n -L` 什麼都沒顯示，但實際上有防火牆的運作。這是因為 iptables 沒使用 `-t` 指定 table 名稱的時候，預設是 `filter` ，但是你現在 nftables 裡面是沒有 `table ip filter` ，只有 `table inet firewalld` 的，所以就算看到 iptables 啥也沒有，但其實 nftables 裡面有一堆 firewalld 預先設定好的規則，所以會以為問題在其他地方。

至於 Debian/Ubuntu 系列的可能還能用 iptables 的原因，是因為這系列的防火牆規則直接把 nftables 的 table 結構設計成跟過去 iptables 時期的一樣，所以 iptables-nft 就能簡單的轉換規則顯示跟輸入了。

## 結論

該學 nft 還是得學個 nft。不然哪天被坑死都不知道。
