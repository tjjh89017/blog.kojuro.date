---
title: 核心數太多導致的 Docker 問題
author: Date Huang
date: 2019-03-19T19:11:00+00:00
categories:
  - server
  - arm

---
因為之前在使用 ARM Server ，核心數高達 96 ，所以 Golang 的軟體預設就會使用 96 個程序，有時候就會讓 docker 卡卡的，很難使用，問題會超級多。之前使用 Kolla-Ansible 部署 OpenStack 的時候就經常遇到 Gather Facts 卡住，ARM Server 本身必須要重新啟動才會正常的情況。

所以建議用 GOMAXPROCS 把 docker.service 與 containerd.service 的程序使用量限制在一個合理範圍，目前看起來設定 GOMAXPROCS=4 ，dockerd 與 containerd 都還是會複製成 12 個程序，所以我設定為 GOMAXPROCS=12 ，讓程序使用量為 36 左右，目前測試起來沒有太大的問題。
