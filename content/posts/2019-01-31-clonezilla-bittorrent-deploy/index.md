---
title: CloneZilla Bittorrent Deploy
author: Date Huang
date: 2019-01-31T14:48:22+00:00
tags:
  - clonezilla
  - ezio
  - bt
  - bittorrent
categories:
  - clonezilla
  - ezio
  - bt

---
這篇就隨便打打吧。就是篇剛剛幫國網中心 debug 完的心得。

當初就只是很單純覺得 CloneZilla Multicast Mode 為甚麼速度這麼不穩，而研究並思考可以用什麼方案取代，沒想到黃秉鈞學長隨口講講的 BT Solution ，最後真的被我跟我室友顏靖軒做出來了。從 2016 年底開發至今，總算是迎來第一個完整且穩定的 CloneZilla Bittorrent Mode (>=2.6.1-2 or 20190123-* amd64)。附上國網中心蕭志榥研究員做的[教學文件][1]。

可是做完這個之後，我卻沒有任何地方可以真的大規模測試，畢竟我從一開始就不是做網管的，所以也只好請大家幫忙測試看看啦。

<pre class="wp-block-code"><code>Clonezilla live 2.6.1-2

The underlying GNU/Linux operating system was upgraded. This release is based on the Debian Sid repository (as of 2019/Jan/22).
Enable secure boot support when creating Debian live system (create-debian-live). However, it's still not ready for secure boot: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=920144
Add nuttcp as an alternative to netcat in ocs-onthefly. Add dialog for choosing compression algorithm in expert mode. Option "-u" was added in the dialog of expert mode to use nuttcp.
Bug fixed: For CentOS 7, the ncat need the option "--recv-only" in the client.
To avoid OOM killer to kill ezio, we use the multi torrent files support (ezio >= 1.1.6) and limit the cache size. It can be tuned by ezio_cache_ratio in drbl-ocs.conf.
Add a mechanism to reuse image for BT from disk mode. The option -mdst-img can be used to assign the existing pseudo image.
New mechanism was added: instead of using Partclone image as the BT source, the local device (whole disk or partitions) can be as the source, too.
-- Steven Shiau &lt;steven at clonezilla org> Tue, 22 Jan 2019 22:47:00 +0800</code></pre>

 [1]: http://stevenshiau.org/clonezilla-live/bt-from-disk/
