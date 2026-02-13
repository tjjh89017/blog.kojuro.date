# 自動備份至 G Suite

因為之前蓋雲的緣故，所以資料總是得備份，而且資料有點大量，所以就寫一篇來說我現在怎麼備份到 G Suite 上面的。

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
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2019/02/%e8%87%aa%e5%8b%95%e5%82%99%e4%bb%bd%e8%87%b3-g-suite/#%E7%9B%AE%E5%89%8D%E6%9E%B6%E6%A7%8B" title="目前架構">目前架構</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2019/02/%e8%87%aa%e5%8b%95%e5%82%99%e4%bb%bd%e8%87%b3-g-suite/#Google_Drive_File_Stream" title="Google Drive File Stream">Google Drive File Stream</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2019/02/%e8%87%aa%e5%8b%95%e5%82%99%e4%bb%bd%e8%87%b3-g-suite/#Windows_Server_2019" title="Windows Server 2019">Windows Server 2019</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2019/02/%e8%87%aa%e5%8b%95%e5%82%99%e4%bb%bd%e8%87%b3-g-suite/#WSL" title="WSL">WSL</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-5" href="https://blog.kojuro.date/2019/02/%e8%87%aa%e5%8b%95%e5%82%99%e4%bb%bd%e8%87%b3-g-suite/#%E7%B8%BD%E7%B5%90" title="總結">總結</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-6" href="https://blog.kojuro.date/2019/02/%e8%87%aa%e5%8b%95%e5%82%99%e4%bb%bd%e8%87%b3-g-suite/#Ref" title="Ref">Ref</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="%E7%9B%AE%E5%89%8D%E6%9E%B6%E6%A7%8B"></span>目前架構<span class="ez-toc-section-end"></span>

我現在是用 OMV 與 zfs 建立一個 NAS，所以我備份的時候只需要透過 zfs snapshot 與 zfs send 的功能，就能當資料寫成一個映像檔，但是因為資料量太大，動輒上 TB ，所以我會把映像檔每 100G 分割成一個檔案，比較有利於儲存等等項目。

主要先用 zfs snapshot ，定期快照，然後透過 zfs send 與 split 把資料分割

<pre class="wp-block-code"><code>zfs snapshot -R zpool@$DATE
zfs send -R zpool@$DATE | split -d -a 3 -b 100G - "zpool@$DATE."</code></pre>

## <span class="ez-toc-section" id="Google_Drive_File_Stream"></span>Google Drive File Stream<span class="ez-toc-section-end"></span>

Google Drive File Stream (後稱：gfs)，這是只有 G Suite 帳號才能使用的功能，就是在 Windows 或是 macOS 底下模擬一個類似硬碟的設計，跟 rclone mount 或是 google-drive-ocamlfuse 有點類似。不過不支援 Linux ，也不支援多使用者操作。那在 Windows Server 上面則要多加一個參數才能正確安裝。

<pre class="wp-block-code"><code>GoogleDriveFSSetup.exe --allow_server_install</code></pre>

在 Windows Server 中只能有一個帳戶啟用 gfs 功能，沒有辦法多人使用 [1]，且必須要設定足夠的 Cache 資料夾暫存，否則 gfs 會拒絕儲存該檔案。

簡單來說 gfs 會將檔案，先透過 FUSE 的方式儲存在 Local Disk 裡面，然後再上傳至 Google Drive，上傳完畢後再將 Local Disk 中的空間釋放，所以如果上傳速度沒有跟上，那就會先佔用到 Local Disk 的空間。

## <span class="ez-toc-section" id="Windows_Server_2019"></span>Windows Server 2019<span class="ez-toc-section-end"></span>

那因為 gfs 無法用在 Linux 環境中，而且手上的 Server 都沒辦法安裝 Windows 10 或是 macOS，所以我就使用 Windows Server 2019 作為上傳工具。先安裝 gfs 之後，順便安裝 WSL，這是 Server 2019 新加入的功能，相當好用。

我的配置大概是，4TB 硬碟六顆做 Soft RAID0，然後把 gfs Cache 設定在上面。因為不想要浪費 NAS 上面的空間來儲存 zfs image ，所以在 Windows Server 中新增一個資料夾，並且開啟 NFS，作為 zfs image 的暫存點，NAS 那邊就會透過 zfs send ，直接儲存 zfs image 到 Windows Server 上面。這邊要注意 Windows Server NFS 會將 wsize 與 rsize 都設定為 1048576 ，所以很可能會超級吃 NAS 上面的 OS Cache，壓縮到 zfs 能使用的記憶體，不過使用上並沒有什麼太多的影響。網路傳輸方面則會累積一定快取之後，才真正寫入 Windows Server ，這點要注意一下。

當複製到 RAID 的 zfs image 暫存點之後，就可以透過 FastCopy [2]，這套在 Windows 上面可以正常複製 UTF-16 檔名、過長檔名的檔案們，而且速度不算太慢，GUI 使用也算方便，不過我還是以 script 使用為主。例如可以這樣

<pre class="wp-block-code"><code>FastCopy.exe /cmd=move /estimate /acl=FALSE /stream=FALSE /reparse=FALSE /verify=FALSE /recreate /error_stop=FALSE /no_ui /ballonn=FALSE /force_start "$SRC_A" "$SRC_B" /to=G:\我的雲端硬碟\zfs_backup\</code></pre>

## <span class="ez-toc-section" id="WSL"></span>WSL<span class="ez-toc-section-end"></span>

接下來介紹 WSL 相對比較好用的使用方式，如果在 Powershell 上面使用，可能會覺得字體等等的不便，所以使用 Cmder 或是單單使用 ConEmu ，搭配 ConEmu 內建的 WSLbridge.exe ，可以正常的使用方向鍵，與愉快的終端機顯示。[3]

WSL 有一個最為終極的好處，針對我們這種喜愛 Linux Script 的使用者，又同時需要 Windows exe 功能的時候。如果你在 WSL 中呼叫一個 Windows EXE ，依然會正常執行，所以既可以享受到方便的 scripting ，又可以直接使用 Native Windows EXE，算是一舉兩得。

<pre class="wp-block-code"><code>/mnt/c/Users/Administrator/FastCopy/FastCopy.exe XXX XXX</code></pre>

## <span class="ez-toc-section" id="%E7%B8%BD%E7%B5%90"></span>總結<span class="ez-toc-section-end"></span>

總結的來說，script 大概會長這個樣子，只要定期在 Windows Server 中執行 autocopy.sh，就可以自動備份到 gfs 上面了。

不過上傳可能會遇到每日上傳 750G 的限制，以及 Cache 的資料夾容量不足等等的問題，也必須要記得將 zfs snapshot 適時的刪除過舊的版本，以免造成效能低落。

上傳速度約為 300 ~ 420Mbps，比 Synology Cloud Sync 20 threads 慢(可以到 1Gbps)，但是 Synology Cloud Sync 容易發生建立成兩個同名資料夾，檔案就分散在兩邊的問題。至於 rclone 等等的 Linux 同步軟體，我都不太確定在遇到上傳限制時，該如何 fallback 。所以才直接採用 Google File Stream，雖然速度相對慢一些，但是資料比較正確上傳。

<pre class="wp-block-code"><code># autocopy.sh in windows wsl
#!/bin/bash
# -*- coding: utf-8 -*-

DATE=`date +%Y%m%d`
echo $DATE

#ssh -i ~/id.pem $USER@$IP "sudo /root/autosnapshot.sh $DATE"
ssh -t -i ~/id.pem $USER@$IP "sudo screen -m /root/autosnapshot.sh $DATE"

SRC_DIR='H:\zfs_backup\'
SRC="$SRC_DIR\\zpool@$DATE"

/mnt/c/Users/Administrator/FastCopy/FastCopy.exe \
        /cmd=move \
        /estimate \
        /acl=FALSE \
        /stream=FALSE \
        /reparse=FALSE \
        /verify=FALSE \
        /recreate \
        /error_stop=FALSE \
        /no_ui \
        /balloon=FALSE \
        /force_start \
        "$SRC" \
        '/to=G:\我的雲端硬碟\zfs_backup\'</code></pre>

<pre class="wp-block-code"><code># /root/autosnapshot.sh in NAS
#!/bin/bash

echo $1
if [ ! -n $1 ]; then
        echo "Need Date YYYYMMDD!"
        exit 1
fi

POOL=zpool
DATE=$1
TARGET=/tmp/zfs_backup

# check if this snapshot exsits
if ! zfs list -t all | grep "$POOL@$DATE" > /dev/null 2>&1 ; then
        echo "There is no snapshot. Snapshot now"
        zfs snapshot -r "$POOL@$DATE"
fi

# send to zfs_backup
mkdir -p "$TARGET"
if ! mountpoint -q -- "$TARGET"; then
        mount -t nfs '$WINDOWS_IP:/zfs_backup' "$TARGET"
fi

mkdir -p "$TARGET/$POOL@$DATE"
zfs send -R "$POOL@$DATE" | pv -trba | split -d -a 3 -b 100G - "$TARGET/$POOL@$DATE/$POOL@$DATE."
#zfs send -R "$POOL@$DATE" | split -d -a 3 -b 100G - "$TARGET/$POOL@$DATE/$POOL@$DATE."</code></pre>

## <span class="ez-toc-section" id="Ref"></span>Ref<span class="ez-toc-section-end"></span>

  1. https://www.reddit.com/r/DataHoarder/comments/8rye2n/there\_is\_a\_simple\_way\_of\_installing\_google\_drive/
  2. https://fastcopy.jp/en/
  3. https://conemu.github.io/en/BashOnWindows.html

