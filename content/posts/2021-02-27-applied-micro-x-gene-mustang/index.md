---
title: Applied Micro X-Gene Mustang 使用心得
author: Date Huang
date: 2021-02-27T10:37:42+00:00
categories:
  - arm

---
之前大學時期因為想玩玩 ARM64 Cloud 的緣故，所以找到了這台 Applied Micro 的 Mustang 原型機，可是大學時期一直沒有機會弄到一台 Mustang 來操作。直到最近廠商在清倉庫的時候，感謝 Richard Liu 直接幫我留了一台下來研究，而有這篇文章

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
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2021/02/applied-micro-x-gene-mustang-%e4%bd%bf%e7%94%a8%e5%bf%83%e5%be%97/#Flash_to_UEFI_bootloader" title="Flash to UEFI bootloader">Flash to UEFI bootloader</a><ul class="ez-toc-list-level-3">
        <li class="ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2021/02/applied-micro-x-gene-mustang-%e4%bd%bf%e7%94%a8%e5%bf%83%e5%be%97/#Setup_MAC_address" title="Setup MAC address">Setup MAC address</a>
        </li>
      </ul>
    </li>
    
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2021/02/applied-micro-x-gene-mustang-%e4%bd%bf%e7%94%a8%e5%bf%83%e5%be%97/#Install_Ubuntu_2004_arm64" title="Install Ubuntu 20.04 arm64">Install Ubuntu 20.04 arm64</a><ul class="ez-toc-list-level-3">
        <li class="ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2021/02/applied-micro-x-gene-mustang-%e4%bd%bf%e7%94%a8%e5%bf%83%e5%be%97/#flash-kernel_bug" title="flash-kernel bug">flash-kernel bug</a>
        </li>
      </ul>
    </li>
    
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-5" href="https://blog.kojuro.date/2021/02/applied-micro-x-gene-mustang-%e4%bd%bf%e7%94%a8%e5%bf%83%e5%be%97/#Result" title="Result">Result</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-6" href="https://blog.kojuro.date/2021/02/applied-micro-x-gene-mustang-%e4%bd%bf%e7%94%a8%e5%bf%83%e5%be%97/#Reference" title="Reference">Reference</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="Flash_to_UEFI_bootloader"></span>Flash to UEFI bootloader<span class="ez-toc-section-end"></span>

認識我的人應該都知道，我極度討厭 ARM 開發板上面的 U-boot 作為開機引導，U-boot 開機的方式太多種類了，導致每次拿到新的板子都要研究一番，就例如說：

  * Raspberry Pi 1
      * U-Boot 使用 kernel.img 作為名稱與格式
  * ORDOID-XU4
      * U-Boot 安裝在 SD 卡最前面的 sector，開機時會直接讀取 SD 卡作為開機引導，仿造 X86 BIOS 模式進行
      * U-Boot 需要指定 uImage 與 initrd.img
  * Other ARM Board
      * 這就要看廠商用哪種模式了

所以 U-Boot 對我來說，是一個討厭到不行的存在，除了 Kernel 需要不同格式跟安裝方式，有時候還需要指定特定的記憶體位置把 Kernel 讀取進去，然後再開機。

感謝 Marcin Juszkiewicz 提供的教學 [[1]][1] ，讓我可以把 Mustang 刷成 UEFI 開機的模式，而不需要浪費生命跟 U-Boot 戰鬥。請直接參閱 Marcin 提供的教學 blog，並且記得要同時更新 UEFI firmware 以及 Slimpro Firmware。

### <span class="ez-toc-section" id="Setup_MAC_address"></span>Setup MAC address<span class="ez-toc-section-end"></span>

發現更新完 UEFI，mac address 在每次開機的時候會填入隨機值，所以需要在 UEFI shell 中設定

<pre class="wp-block-code"><code>set MAC0 aa:bb:cc:dd:ee:f1
set MAC1 aa:bb:cc:dd:ee:f2
set MAC2 aa:bb:cc:dd:ee:f3
set MAC3 aa:bb:cc:dd:ee:f4</code></pre>

需要把 mac address 替換成你機器上面的條碼編號，並依序遞增

## <span class="ez-toc-section" id="Install_Ubuntu_2004_arm64"></span>Install Ubuntu 20.04 arm64<span class="ez-toc-section-end"></span>

安裝 Ubuntu 也是另外一個障礙，Ubuntu 有提供 arm64 安裝光碟，但是我發現 Live Installer 無法正常安裝，目前沒看到 log ，只有看到可能是 flash kernel 的時候遇到問題，目前猜可能是 Ubuntu 判定到是 APM-mustang 的機型之後，會認定使用 U-boot ，所以會順便使用 flash kernel 去安裝 Kernel 跟 DTB，所以 Live server 版本的安裝光碟無法使用。需要使用 Ubuntu arm64 netboot 的安裝 ISO [[2]][2] ，連結2中的 mini.iso 可以使用來安裝，只是安裝過程中必須要連接網路，才有辦法透過網路安裝。直接將 mini.iso 透過 dd 或是 Win32diskimager 寫入隨身碟中，並且插入 Mustang ，執行以下指令就可以進入 Ubuntu 安裝畫面

<pre class="wp-block-code"><code>fsX:\efi\boot\bootaa64.efi</code></pre>

請自行替換 fsX 成 UEFI 中你的隨身碟的代號

### <span class="ez-toc-section" id="flash-kernel_bug"></span>flash-kernel bug<span class="ez-toc-section-end"></span>

flash-kernel bug 經過一番 trace 之後發現很可能是因為 apm-mustang 過去一直都是 U-boot support，所以即便改成 UEFI 之後，flash-kernel 還是會認定需要透過 U-boot 方式開機。所以發了一個可能可以改善的 PR [[3]][3]

結果 flash-kernel bug 並不是 root cause，也不是 fatal error。這問題是 flash-kernel 行為，以及 APT 行為，再加上 Curtin 行為，三者一起才會發生影響的。等待跟開發者討論之後再來決定怎麼紀錄好了。

## <span class="ez-toc-section" id="Result"></span>Result<span class="ez-toc-section-end"></span><figure class="wp-block-image">

<img loading="lazy" width="1024" height="36" src="https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-uname-1024x36.png" alt="" class="wp-image-188" srcset="https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-uname-1024x36.png 1024w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-uname-300x11.png 300w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-uname-768x27.png 768w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-uname.png 1114w" sizes="(max-width: 1024px) 100vw, 1024px" /> </figure> <figure class="wp-block-image"><img loading="lazy" width="1024" height="573" src="https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-dmidecode-1024x573.png" alt="" class="wp-image-187" srcset="https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-dmidecode-1024x573.png 1024w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-dmidecode-300x168.png 300w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-dmidecode-768x430.png 768w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-dmidecode.png 1115w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> <figure class="wp-block-image"><img loading="lazy" width="1024" height="573" src="https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-htop-1024x573.png" alt="" class="wp-image-193" srcset="https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-htop-1024x573.png 1024w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-htop-300x168.png 300w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-htop-768x430.png 768w, https://blog.kojuro.date/wp-content/uploads/2021/02/apm-mustang-htop.png 1115w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

## <span class="ez-toc-section" id="Reference"></span>Reference<span class="ez-toc-section-end"></span>

  1. https://marcin.juszkiewicz.com.pl/2015/11/18/unbricking-apm-mustang/ 
  2. http://free.nchc.org.tw/ubuntu-ports/dists/focal/main/installer-arm64/current/legacy-images/
  3. https://salsa.debian.org/installer-team/flash-kernel/-/merge_requests/27

 [1]: https://marcin.juszkiewicz.com.pl/2015/11/18/unbricking-apm-mustang/
 [2]: http://free.nchc.org.tw/ubuntu-ports/dists/focal/main/installer-arm64/current/legacy-images/
 [3]: https://salsa.debian.org/installer-team/flash-kernel/-/merge_requests/27
