---
title: ZFS 使用的經驗
author: Date Huang
date: 2018-10-20T17:00:53+00:00
url: /2018/10/zfs-使用的經驗/
categories:
  - openstack

---
我透過 OpenMediaVault 搭配 OMV-zfs plugin 使用 zfs ，UI 能使用的功能依舊很少，所以大部分還是得用 CLI 控制 ZFS 本身，不過可以用 OMV UI 控制很多 file sharing 的設定就已經萬謝了。以下分享一些，使用 ZFS 以來的心得。

<!--more-->

<div id="ez-toc-container" class="ez-toc-v2_0_17 counter-hierarchy counter-decimal ez-toc-grey">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title">
      Table of Contents
    </p>
    
    <span class="ez-toc-title-toggle"><a class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" style="display: none;"><i class="ez-toc-glyphicon ez-toc-icon-toggle"></i></a></span>
  </div><nav>
  
  <ul class="ez-toc-list ez-toc-list-level-1">
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#ZFS_on_Debian" title="ZFS on Debian">ZFS on Debian</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#%E7%B3%BB%E7%B5%B1%E9%85%8D%E7%BD%AE" title="系統配置">系統配置</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#%E7%A1%AC%E7%A2%9F%E8%A6%8F%E5%8A%83" title="硬碟規劃">硬碟規劃</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#L2ARC" title="L2ARC">L2ARC</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-5" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#ZIL" title="ZIL">ZIL</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-6" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#VMware_on_ZFS" title="VMware on ZFS">VMware on ZFS</a><ul class="ez-toc-list-level-3">
        <li class="ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-7" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#NFSv3" title="NFSv3">NFSv3</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-8" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#iSCSI" title="iSCSI">iSCSI</a>
        </li>
      </ul>
    </li>
    
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-9" href="https://blog.kojuro.date/2018/10/zfs-%e4%bd%bf%e7%94%a8%e7%9a%84%e7%b6%93%e9%a9%97/#Reference" title="Reference">Reference</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="ZFS_on_Debian"></span>ZFS on Debian<span class="ez-toc-section-end"></span>

Debian 透過 DKMS 的方式動態編譯與載入 ZFS Module，以規避 ZFS 與 Linux Kernel License 不相容的問題，所以每次更新 ZFS 或是 Kernel ，都會有很長一段編譯時間要等待。

## <span class="ez-toc-section" id="%E7%B3%BB%E7%B5%B1%E9%85%8D%E7%BD%AE"></span>系統配置<span class="ez-toc-section-end"></span>

  * CPU: E5-2620v2 x2
  * RAM: DDR3 ECC REG 48G
  * HDD: Toshiba HDD 4T 5900RPM x12
  * HBA: LSI-2308 (有點忘了)
  * Network: Intel Dual 10G

## <span class="ez-toc-section" id="%E7%A1%AC%E7%A2%9F%E8%A6%8F%E5%8A%83"></span>硬碟規劃<span class="ez-toc-section-end"></span>

目前硬碟規劃都是 3 顆硬碟做成 RAIDZ1，用 12 顆硬碟作 4 組，後再做 RAID0

<pre class="wp-block-code"><code>NAME                                      STATE     READ WRITE CKSUM
zpool                                     ONLINE       0     0     0
  raidz1-0                                ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46TAK2K1FNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46T2K63RFNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46T1K2V4FNWA  ONLINE       0     0     0
  raidz1-1                                ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46M4K3ZOFNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46T2K63DFNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46TBK183FNWA  ONLINE       0     0     0
  raidz1-2                                ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46TAK2K5FNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_86F1K3B4FNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46T1K2V7FNWA  ONLINE       0     0     0
  raidz1-3                                ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46TBK184FNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46T1K2V6FNWA  ONLINE       0     0     0
    ata-TOSHIBA_MD03ACA400V_46TAK2KAFNWA  ONLINE       0     0     0 
</code></pre>

## <span class="ez-toc-section" id="L2ARC"></span>L2ARC<span class="ez-toc-section-end"></span>

當你記憶體足夠大的時候，事實上 L2ARC 也沒有甚麼太大作用，對於 4K 讀寫加速也沒有特別顯著，基本上 Cache 的量大約都只有記憶體大小而已，用太大的 SSD 沒什麼幫助以外，甚至可能還會占用一個硬碟空間，記憶體大於 32G 之後，就沒有那麼划算了。除非 Server 背面有 2.5 吋硬碟的槽位，但通常這情況將系統碟做成 RAID1 可能會比較實際。

## <span class="ez-toc-section" id="ZIL"></span>ZIL<span class="ez-toc-section-end"></span>

若使用 async mode 來開啟檔案，ZIL 在這情境就完全不會發揮作用。還沒有測試過 iSCSI 是否也會相同，未來可以測試看看。

測試過後，iSCSI 透過 LIO 處理，並沒有任何使用到 ZIL 的跡象，可能還是因為 async mode 的緣故

## <span class="ez-toc-section" id="VMware_on_ZFS"></span>VMware on ZFS<span class="ez-toc-section-end"></span>

### <span class="ez-toc-section" id="NFSv3"></span>NFSv3<span class="ez-toc-section-end"></span>

透過 NFSv3 來提供 VMware ESXi 存放，以上面那種配置而言，在上面安裝 Windows Server 2016 ，測試速度大概如下：<figure class="wp-block-image">

<img loading="lazy" width="402" height="367" src="https://blog.kojuro.date/wp-content/uploads/2018/12/photo_2018-12-03_11-57-24.jpg" alt="" class="wp-image-137" srcset="https://blog.kojuro.date/wp-content/uploads/2018/12/photo_2018-12-03_11-57-24.jpg 402w, https://blog.kojuro.date/wp-content/uploads/2018/12/photo_2018-12-03_11-57-24-300x274.jpg 300w" sizes="(max-width: 402px) 100vw, 402px" /> </figure> 

基本上 L2ARC 與 ZIL 都沒有發揮作用

### <span class="ez-toc-section" id="iSCSI"></span>iSCSI<span class="ez-toc-section-end"></span>

這邊我就直接測試透過 Windows 10 直接存取 iSCSI 的效能<figure class="wp-block-image">

<img loading="lazy" width="402" height="367" src="https://blog.kojuro.date/wp-content/uploads/2018/12/photo_2018-12-03_12-10-21.jpg" alt="" class="wp-image-138" srcset="https://blog.kojuro.date/wp-content/uploads/2018/12/photo_2018-12-03_12-10-21.jpg 402w, https://blog.kojuro.date/wp-content/uploads/2018/12/photo_2018-12-03_12-10-21-300x274.jpg 300w" sizes="(max-width: 402px) 100vw, 402px" /> </figure> 

## <span class="ez-toc-section" id="Reference"></span>Reference<span class="ez-toc-section-end"></span>

[Configuring ZFS Cache for High Speed IO][1]

 [1]: https://linuxhint.com/configuring-zfs-cache/
