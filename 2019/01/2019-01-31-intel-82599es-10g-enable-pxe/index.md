# Intel 82599ES 10G 網卡啟用 PXE

最近在測試 CloneZilla EZIO 模式，想說自己有三台 server 閒置，那就拿來跑看看，結果沒想到有一張網卡沒有辦法 PXE 開機，所以只好研究下怎麼處理了。

<div id="ez-toc-container" class="ez-toc-v2_0_17 counter-hierarchy counter-decimal ez-toc-grey">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title">
      Table of Contents
    </p>
    
    <span class="ez-toc-title-toggle"><a class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" style="display: none;"><i class="ez-toc-glyphicon ez-toc-icon-toggle"></i></a></span>
  </div><nav>
  
  <ul class="ez-toc-list ez-toc-list-level-1">
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2019/01/intel-82599es-10g-%e7%b6%b2%e5%8d%a1%e5%95%9f%e7%94%a8-pxe/#%E8%A7%A3%E6%B1%BA%E6%96%B9%E5%BC%8F" title="解決方式">解決方式</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="%E8%A7%A3%E6%B1%BA%E6%96%B9%E5%BC%8F"></span>解決方式<span class="ez-toc-section-end"></span>

先下載 82599ES 的 firmware，應該會得到一個 IBABUILD.exe 的 DOS 執行檔，然後在透過 DOS 執行編譯還有 flash firmware，就可以搞定了。指令有點忘記了，之後想到在補。

