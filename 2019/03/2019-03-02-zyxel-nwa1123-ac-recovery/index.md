# Zyxel NWA1123-AC 修復筆記

因為 NWA1123-AC 的 WebUI 在 Chrome 下面沒辦法正常顯示，只能使用 Edge，相當不方便，所以就手賤去更新韌體，然後失敗了，完全開不了機。所以有了這篇修復筆記

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
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2019/03/zyxel-nwa1123-ac-%e4%bf%ae%e5%be%a9%e7%ad%86%e8%a8%98/#%E6%BA%96%E5%82%99" title="準備">準備</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2019/03/zyxel-nwa1123-ac-%e4%bf%ae%e5%be%a9%e7%ad%86%e8%a8%98/#%E6%AD%A3%E9%A1%8C" title="正題">正題</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2019/03/zyxel-nwa1123-ac-%e4%bf%ae%e5%be%a9%e7%ad%86%e8%a8%98/#Ref" title="Ref">Ref</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="%E6%BA%96%E5%82%99"></span>準備<span class="ez-toc-section-end"></span>

要先準備 Serial 線，需要三條杜邦線，以及 USB to TTL 或是 Raspberry Pi 一張，我自己是用 Banana Pi M2 。

Windows 的電腦一台並安裝 tftpd [1]，負責 tftp server，使用 Linux 作為 tftp server 都會失敗，不知道為什麼。

從 Zyxel 官網下載 NWA1123-AC 的 firmware [2]。並將 V212AAOX0C0.bin 重新命名為 V212AAOX0C0.tar.bz2，並解壓縮成 vmlinux\_mi124\_f1e.lzma.uImage 與 mi124_f1e-jffs2 ，並放入 tftpd 的資料夾中。

## <span class="ez-toc-section" id="%E6%AD%A3%E9%A1%8C"></span>正題<span class="ez-toc-section-end"></span>

先將 NWA1123-AC 斷電並拆殼，天線是固定在上蓋，要小心不要扯到線，依照 OpenWrt 的說明 [3] 接上 GND 、 RX 、 TX，Vcc 則忽略，若以 OpenWrt 圖所示，最右邊針腳為 GND ，由右向左依序為 GND、RX、TX。

然後連接 Serial ，並將 NWA1123-AC 上電，並於 Serial 多按幾次任意按鈕，進入 U-Boot 的操作介面。並參考這兩篇教學 [4,5]

<pre class="wp-block-code"><code># setenv ipaddr &lt;AP IP>
# setenv serverip &lt;Windows IP>
# run lk
# run lf</code></pre>

等待完成後就可以將 NWA1123-AC 重開機，並確認是否能使用了。 也可以直接在 U-Boot 操作執行

<pre class="wp-block-code"><code># run bootcmd</code></pre>

就能直接開機測試了。

## <span class="ez-toc-section" id="Ref"></span>Ref<span class="ez-toc-section-end"></span>

  * [1] http://tftpd32.jounin.net/
  * [2] https://www.zyxel.com/tw/zh/support/DownloadLandingSR.shtml?c=tw&l=zh&kbid=M-01949&md=NWA1123-AC
  * [3] https://oldwiki.archive.openwrt.org/toh/zyxel/nwa1123-ac#serial
  * [4] https://blog.edhayes.us/2017/10/06/fixing-my-zyxel-nwa1123-ac/
  * [5] https://kb.zyxel.com/KB/searchArticle!gwsViewDetail.action?articleOid=014292&lang=EN

