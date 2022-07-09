---
title: Router Loopback IP使用
author: Date Huang
date: 2021-08-27T15:39:18+00:00
categories:
  - network

---
簡單紀錄一下使用Loopback IP的經驗。現在因為我用wireguard把宿舍、家裡還有雲端那邊串連在一起，所以路由變得滿複雜的，借用OSPF的便利把路由直接動態設定，但是這時候就遇到說router-id其實就滿不容易命名的清楚，可以一眼得知現在的路由是哪一個設備處理的。就目前看來以及其他同事給的設定guideline，似乎滿常透過Loopback IP作為這台設備的唯一識別號碼，更精確說是這個VRF的唯一識別。所以在router-id上面設定Loopback IP，就可以確定一定是哪一個設備，不用設定point to point的兩個IP，而且可能還會設定錯誤。

之後可能要研究一下能不能在wireguard上面使用到OSPF的unnumbered特性，這樣應該連IP設定都不太需要了，不過unnumbered這件事似乎是看routing engine決定的，所以如果都是FRR相同版本可能沒問題，但是有其他設備介入了話，就需要花時間測試看看了。
