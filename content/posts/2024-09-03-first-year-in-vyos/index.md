---
title: "在VyOS上班的第一年"
subtitle: ""
date: 2024-09-03T22:58:34+08:00
lastmod: 2024-09-03T22:58:34+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: ""
license: ""
images: []

tags: ["life"]
categories: ["life"]

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

簡單記錄下在VyOS Networks上班的第一年

<!--more-->

## 在這之前

其實我大概在大學跟碩士的時候，其實就已經在用VyOS好一陣子了。那時候在弄DozenCloud的時候因為一些原因，所以需要各種打Tunnel到不同地方，最原本是用pfSense來做一些NAT gateway等等的工作，但是pfSense對於Policy Based Routing沒有說很容易，也沒辦法正確處理，所以就google發現到VyOS是一個完整的Linux based Router，就拿來用了。在Edgecore上班的時候，也會把VyOS拿來測試一些行為啥的。測試跟使用的過程總是會遇到一些bug，也因為是開源的專案，所以我也就順便幫他們修了三四個bug過。總之就用了五年起跳吧。

## 怎麼會進VyOS

2023年的時候，我從前公司離職休息了半年多吧，找工作也沒說很順利，滿多工作都碰壁，也不是很確定自己的職涯發展要往哪個職位走，畢竟我從Presales跑到Software Engineer，用著Presales的角度看軟體開發，有時候滿微妙的，如果專案本身的走向怪怪了話。

那個時間點，我跟Jalen沒事幹在嘗試把VyOS弄上Raspberry Pi的環境上執行，真的搞出來之後，貼到Linkedin上面宣傳一下，順便當找工作用的宣傳。

不知道是不是去媽祖廟跟土地公廟祈求個合適的工作，我的Linkedin被VyOS的老闆看到了，剛好VyOS需要一個Presales Engineer來幫業務們一些技術上的問題，而不用太花RD跟Support的時間，同時間VyOS正在思考跟一些硬體廠商合作，也需要一個曾經在硬體廠也懂一些硬體相關的工程師，這樣聽完我還真的滿合適的角色，畢竟我之前也是做了一堆奇怪的事情。

VyOS老闆就跟我約了一個時間通了一通電話，他在西班牙馬德里，而我在台灣，就聊聊這個職缺的需求跟情況，然後就安排了第二次面試了。第二次面試的面試官有CTO、RD跟Support頭、業務主管，不過講話的幾乎只有RD跟Support的主管跟我問一些問題這樣，說是問問題，我感覺比較像是聊天，因為他就是用各種情境問我會怎麼處理這樣，說實在的我現在覺得這個面試經驗一點參考價值也沒有，感覺他們看我介紹我自己的side project，然後看了下我的Github之後，剩下就是看我的反應適不適合當Presales而已。不過我面試的時候得失心滿重的，因為那時候真的有興趣的沒開缺，不然就是已經把我刷了，因為我不是Software Engineer。

我個人是滿意外會通過這個面試，因為除了RD頭之外，其他人一句話都沒講過，讓我滿緊張的，畢竟有講話會知道對方情緒跟態度，但是沒講話就完全沒辦法猜測了。後來就進入簽約跟談薪水的階段，VyOS也很好的讓我可以再多休一個月，後來就選擇9月1日上班了，工作本身就全遠端模式了。

## VyOS工作型態

進去VyOS之後，發現公司其實算是一個新創，人數沒有很多，加起來才50人上下而已，而且沒半個人會進辦公室，因為住在辦公室附近的，只有一個系統工程師而已，他沒事也不進去。然後東亞只有我一個人，所以我就把自己的上班時區平移到接近歐洲時區了，還有幾個同事在阿根廷，直接跟台灣時區接近日夜顛倒。算是滿彈性的遠端工作，沒有特定時間要在線上，也沒有要求在哪個地點上班。

## 一些工作上的趣事

### 真遠端工作

剛進去的時候，公司指派RD頭當我的入職導師，來帶我的入職的狀態這樣。因為是遠端工作，他也說工作真的很彈性不太要求。我剛好聽朋友提到華航福岡的機票大特價，我就訂了一張機票飛到福岡的機票，周五下午飛，下下周一飛回台灣，總共十天的計畫這樣。跟導師提這件事的時候，說我會早上去玩，晚上工作，畢竟日本晚上時間剛好歐洲上班時間，他就對我說了句：OK啊，我們很彈性的，你在哪上班我們沒有差，反正我們原本原則就是不會預期大家固定的回應時間，總是大家有自己的私事可能會晚點回應什麼的。

後來我也用這個方式，在日本一邊玩好幾天一邊工作好幾天，畢竟我可以有兩個周末在日本放假，雖然機票跟飯店會貴一點。順帶一提，我有一位在日本上班的學弟，他回台灣的時間點，我會跑去他家睡，直接省飯店錢。

還有個有趣的故事，主管主要在西班牙，某天請了兩天假，然後就跟我們說他在峇里島上班兩周，記得用峇里島時區找他，後來他又請假兩天，我們就知道他飛回西班牙了。

### 跟硬體一些合作案

公司突然發現很多他們想要合作的硬體廠，其實總部都在台灣，他們就全部把台灣相關的合作夥伴全部丟給我全權處理，還在會議上的時候跟我說：你可以直接講中文喔，有需要我的時候再用英文跟我講就可以了。不過也因為可以講中文，我很多時後直接打電話給對方快速問一些問題，對方也直接打給我快速問問題，有滿多事件就很快的可以處理過去，而不用信件慢慢來回。

### 聯繫台灣的客戶

業務主管那邊發現某個台灣客戶用VyOS用了很兇而且買了不少加值服務的方案，但從來沒用過那些加值服務，就很好奇是不是客戶不知道有這些服務，但是他們又從email裡面沒辦法聯繫上對方，就問問我有沒有機會找到對方窗口聯繫看看，我就剛好有幾個學弟跟朋友在該公司任職，就打了幾通電話去問這件事，學弟還真的在他們內部系統裡面找到我們公司產品的一些使用說明，然後還真的找到跟我們下單的單位負責人了，覺得這個真的是有夠神奇的路徑。

### 有趣的出包?

說釋出包，但某程度不完全算是我的錯，而是各種溝通誤會疊加之後變成的。這件事發生的時候我剛好在東京，然後太緊急了，直接在東京車站外面，大家常拍照的地方開會，總之最後我發現了這個溝通誤會，業務主管跟技術主管都一致認為我的必要性了，因為大家預設的立場跟選擇不一樣，直接導致了大家的共識不是真的共識，因為有一堆誤會。有趣的點在於我人剛好在東京車站，我還不在飯店，非常克難的一起解決這問題了。

### 聖誕節

幾乎同事都是歐洲美洲地區的，所以聖誕假期是他們最長的假期了，除了輪班的人員之外，全部放假去了。我想找人問個問題也找不到。也遇到有人直接輪班聖誕連假期間，然後下周直接休整周掉。

### 提醒我休假的人資

因為我們公司的規定的緣故，每年的特休是無法變現或是延長使用期限的，所以一定要在該年度使用完畢，所以大概6月的時候，人資跟我說我必須要在幾月幾號開始休假到8/31號，才會用光所有特休，不然就是浪費。同時，因為我們公司沒有加班費，所以人資就直接說我們強烈建議別加班，有一個月我的上班時數確實超過不少，人資就跟我提醒，請不要超時太多，不然很容易burn out的。

我拿這兩件事跟我朋友分享的時候，每個人都覺得很羨慕也覺得很三小。

## 外商的工作壓力

### 身兼多責

因為公司比較小，所以一個人會直接擔上滿多責任的，像我的角色也就很混合。一般人可能會覺得很難專注，因為一直被不同事件打斷。雖然我個人已經習慣這種狀態，但是相對也比較耗費精力跟體力。

### 隨時被裁員

畢竟我們是直接跟美國公司簽約的，隨時都可以裁員。基本上績效不夠的情況，就會被裁掉吧。雖然我確定現在我具有一定的重要性，但是被裁員的壓力可沒因此而消失過。

### 沒有附近的同事

因為台灣只有我一個人，所以沒有什麼距離很近的同事可以當朋友，聊聊公司八卦，幹幹公司問題這種的發洩。某程度上我都還是跟我錢公司的同事們在聊這些議題。也會跟現在同事的距離感稍微遠一點。

### 心理健康

這個比較偏遠端工作的問題，我回台中鄉下老家遠端工作，也離開我原本的社交圈，距離社交圈也超遠，畢竟台中鄉下的交通實在不怎樣，來回移動起來基本上就是半天，台北住宿又很貴，所以變成說社交生活大幅下降，要約要幹嘛都會很懶，原本已經夠懶了，現在變得更懶。跟我更常碰面的朋友可能還會是在東京的朋友，因為我去台北的原因好幾次是因為要去松山機場飛日本。

如果房間沒辦法區分生活跟上班的區域，大腦就更難切換上下班狀態，放鬆跟休息也變得相對得更困難。所以其實一直有打算搬去台北，但是租屋市場太貴，而省下來的房租真的太香了，就變成一直沒有硬需求搬家了。

工作壓力是比一般情況更大的，很多時候沒辦法面對面，沒辦法讀空氣，我本身又相對比較工作狂跟焦慮傾向，就變成相對更負面解讀一些情況。不過還好主管每次文字基上都有使用比較明顯且明確的表達，比較不容易誤解。

## 後記

其實今天(9/3)是我任職滿一年的年度檢驗，才剛跟主管還有人資開完會，開會前一天人資就發了主管給我的評價給我檢視，然後我發現我有幾項缺失，就讓我直接焦慮到睡不好，雖然其他部分都是很符合期待的，但就覺得不踏實。直到今天跟主管開完會看完回顧，他表示公司的「符合期待」的標準是很高的，他說就大概是10分滿分有9分的情況，這才讓我踏實不少，雖然我開會前就知道我不太可能被裁員，因為開會前一天才被指派一個9月跟10月的工作，要被裁員兩周後就不見了，不可能等兩個月的。

雖然薪資沒辦法跟Google台灣以及台積電比，但是彈性真的很很足。也因為公司規模比較小，能跨的就很多，我也有跨到RD那邊提供他們一些有趣的新想法跟幫忙解一些bug。

但不知道為什麼，現在在這間公司時間流動變得有點緩慢，我八月在日本三周，一周遠端工作，兩周使用特休，那三周的體感比過去都還要來的長，長到我三四天會打開公司的Slack看一下的情況，因為總覺得好像我應該已經結束假期。

希望未來安好
