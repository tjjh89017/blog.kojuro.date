# 最近 ARM 開發板的一些心得

因為之前大學時期都在玩一些開發板，例如說 Raspberry Pi 等等的項目，所以朋友 Jalen 買了 Banana Pi W2 的一些問題就跑來問我，所以才有這些心路歷程

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
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2018/12/%e6%9c%80%e8%bf%91-arm-%e9%96%8b%e7%99%bc%e6%9d%bf%e7%9a%84%e4%b8%80%e4%ba%9b%e5%bf%83%e5%be%97/#%E5%89%8D%E6%83%85" title="前情">前情</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2018/12/%e6%9c%80%e8%bf%91-arm-%e9%96%8b%e7%99%bc%e6%9d%bf%e7%9a%84%e4%b8%80%e4%ba%9b%e5%bf%83%e5%be%97/#%E9%96%8B%E7%99%BC%E6%9D%BF%E7%88%86%E7%82%B8%E6%99%82%E4%BB%A3" title="開發板爆炸時代">開發板爆炸時代</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2018/12/%e6%9c%80%e8%bf%91-arm-%e9%96%8b%e7%99%bc%e6%9d%bf%e7%9a%84%e4%b8%80%e4%ba%9b%e5%bf%83%e5%be%97/#%E5%9B%B0%E5%A2%83" title="困境">困境</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2018/12/%e6%9c%80%e8%bf%91-arm-%e9%96%8b%e7%99%bc%e6%9d%bf%e7%9a%84%e4%b8%80%e4%ba%9b%e5%bf%83%e5%be%97/#Open_Source%EF%BC%9F" title="Open Source？">Open Source？</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="%E5%89%8D%E6%83%85"></span>前情<span class="ez-toc-section-end"></span>

Banana Pi W2 的功能有 HDMI Input 與硬體的 H.264 編碼，可以做為擷取卡或是直接當作實況機來用，所以我個人滿有興趣的，畢竟身為開發者，有時候會有開發者的傲氣，認為買現成的產品（例如說：圓剛的擷取卡系列）就輸了，也因為市面上擷取卡等等產品也不便宜，例如說：BlackMagic 的系列產品就不便宜。

而且，如果只買圓剛的產品，就沒有會只有擷取功能，事實上還要搭配電腦做後製（上圖片或是上聊天畫面甚麼的），也要透過電腦做軟體或硬體的 H.264 編碼，才能透過 RTMP 傳送到 Twitch 等等的實況平台，事實上來說也沒有說非常的方便。當然以一個專業的實況主而言，這些都是必要的支出。而我是開發者，有傲氣的開發者，所以我們當然是會選擇比較硬派的方式，雖然說結果可能沒有直接買現成產品來的好，但是算是一種成就。

而後來因為 BPI-W2 使用的 RTD1296 晶片有太多沒 specs 而沒辦法處理的部分，所以我們轉向嘗試 RK3399，至少我們在網路上搜尋到的資訊量而言，RK3399 算是比較有希望的，而 Orange Pi RK3399 有提供一個有 HDMI IN 功能的 Debian Image，當然還是沒有抱太大希望。而真的拿到實品之後測試，確實發現 HDMI IN 並沒有辦法擷取到音訊，花非常多時間研究電路連接設計，參考 AIO-3399J 的電路連接，也參考公板音訊設計，最後還是因為對 ALSA 與 ASoC 的不熟悉而停止。

網路上有些 TC358749XBG 的文章，但大部分都支援 Android 而已，並沒有對 Linux 的支援做描述，更沒有提到該晶片 i2s 與 RT5651 相接的使用方式，即便詢問了 Rockchip 的開發者，卻看到一些與 RK Github 上些許不同的設計，而我沒有經驗與技術可以排除 conflict ，所以只好各種 trial and error，當然最終還是以失敗收場，Toshiba TC358749XBG 的晶片名稱我都 Google 到已經背下來的程度了。

說句實在話，我也沒有真的需要 HDMI IN 來擷取 PS4 遊戲畫面，我實況又不會有人看，真的要實況，直接買圓剛的 Solution 都比這個快速方便，我想這大概就是一種挑戰吧，很自虐的那種。

## <span class="ez-toc-section" id="%E9%96%8B%E7%99%BC%E6%9D%BF%E7%88%86%E7%82%B8%E6%99%82%E4%BB%A3"></span>開發板爆炸時代<span class="ez-toc-section-end"></span>

自從 Raspberry Pi 上市，而引起的自造風潮，不少開發者會認為 Raspberry Pi 的硬體效能不符合需求，進而導致了現在的開發板爆炸的時代，大概就跟羅傑的大航海時代差不多？

好處就是，你想要什麼功能，大概市面上可能都可以找到相應的開發板，來開發自己所需，但是壞處就三言兩語沒辦法講清了。

## <span class="ez-toc-section" id="%E5%9B%B0%E5%A2%83"></span>困境<span class="ez-toc-section-end"></span>

先提一下現在開發板的生態，有分成兩種

  * Linux 主體
  * Android 主體

為什麼會以這兩種為主體呢？是因為在 ARM 世界幾乎以 SoC 方式出現，所以通常會是 CPU Vendor 要準備所有 Driver ，而後 ODM 廠商而基於這些基礎上開發應用。那開發 Driver 的前提通常會以現有 ODM 廠商想要出什麼樣的產品，而決定優先開發 Android Driver 或是 Linux Driver ，這個在現今是非常巨大的差別，這會牽涉到 Android 與 Linux Kernel 授權不同的議題，因為我也沒有非常了解，所以就不細提，不過大致上問題而言，Android Driver 大部分是可以閉源的，就 Android 為了保護或是提供廠商使用的誘因而設計；而 Linux Kernel 在 GPL 授權之下，大部分都會以 GPL 方式釋出 Driver，對於廠商來說，這是一個他們不太喜歡的因素。

而且 Android 開發至今對於 UI/UX 等等的議題已經相當完整，也不需要針對系統方面去處理，所以 ODM 廠商相對會選擇 Android 作為他們開發特定應用的基礎，不會再從 Linux 開始重新造輪子。當然對於 Android 比較耗用資源這點，還是有些廠商認為是個問題，不過這都是相對比較少數，至少在我這邊提到的前提而言是比較少的。

在這個前提之下，CPU Vendor 通常會從 Android Driver 支援起，不一定打算支援 Linux ，也不一定有人力能做支援。Android Driver 不一定能跟 Linux 通用，這點是要注意的，即便 Android 是基於 Linux Kernel 的設計，但是還是有不少部分已經差距過於遙遠了。

所以在 CPU Vendor 沒有提供 Linux 支援的情況下，開發板廠商就會面臨限制，畢竟對於開發板廠商而言，通常他們只具備應用開發，與不同元件之間的軟體支援，但是對於 SoC 本身的支援通常要仰賴 CPU Vendor，所以就會遇到很多開發板，上面只有純 AOSP ，而 Linux Image 爛到一種極點的問題。

## <span class="ez-toc-section" id="Open_Source%EF%BC%9F"></span>Open Source？<span class="ez-toc-section-end"></span>

事實上在 Linux 上支援爛，也不純然是因為 GPL 問題，有非常多方法可以規避 GPL 在 Driver 上的問題。主要還是人力問題，各種 Cost Down 文化，進而導致對 Open Source 又愛又恨，也是現在台灣不少公司所面臨到的難題之一。

對於硬體廠商而言，做支援是賺不了什麼錢的，今天開一顆新的晶片，沒賣出一定數量級，是很可能虧本的。所以不提 Open Source Driver，甚至連 Linux 支援都不一定有，ODM 廠商想要什麼功能，才去開發出什麼功能。永續發展才是真的，沒有利益收入，哪來的支援。

所以對於網路上某些小朋友（對，我用小朋友這詞，因為他們確實年紀都比我小，而且都是剛接觸開源的新人）形容閉源軟體就如邪惡一般，不忍說這已經是對開源的崇拜接近到邪教的程度。當然開源解決了相當多的問題，但是並非萬靈丹，總是會踢到鐵板的。

就我而言，如果說軟體可以完全不靠其他人的貢獻，而做到非常的完美，那我並不需要要求什麼開源，因為我可以不需要自己處理問題，都會有支援幫忙把這些處理完畢。那很明顯的，在商業環境中，就需要極大量的人力與金錢來支持，才有辦法做到。

那就會有不少公司沒有這麼深的口袋可以做到這點，就會站在巨人的肩膀上做更多，所以就會基於 Open Source 軟體，而去開發他們的產品。

但是，就是這個但是，有太多廠商用了 Open Source 卻不照授權方式處理，所以你就會注意到他們用了開源軟體，改了開源軟體但是沒有依授權規定釋出他們修改的內容，這相當令人詬病。不忍說，我會注意到他們修改開源軟體，也是因為遇到 bug ，想尋找問題在哪裡的時候發現居然有太多沒有出現在原始碼的片段。可以很明顯的看到 binary 中有相當大量的 Vendor Name 出現，而沒有看到他們有提出原始碼。

這點就讓我非常的不能接受，我可以接受閉源，畢竟有些還是技術之所在，但是我不能接受在利用社會的貢獻之後，又不依照規則將之回饋給社會。

