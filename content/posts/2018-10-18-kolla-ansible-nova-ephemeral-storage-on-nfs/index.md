---
title: Kolla-Ansible Nova Ephemeral Storage on NFS
author: Date Huang
date: 2018-10-17T17:23:08+00:00
categories:
  - openstack

---
OpenStack 通常會有兩種儲存 VM 的方式，一個是直接在 Nova-Compute Node 上面的 `/var/lib/nova/instances` ，另外一種則是儲存在 Cinder。前者通常也不會真的直接儲存在 Node 上，而是會透過 Shared Storage 儲存，例如 NFS 等等。

但是因為 OpenStack 會有一些 Lock 機制，偏偏 NFS 不一定實作全部的 Lock 種類，所以會導致 Lock 失敗，進而製作 VM 也跟著失敗。

Nova 在新增 VM 或是複製 Image 的時候，會考慮處理程序處理的問題，所以會使用 Lock ，以確保沒有正確無誤。透過 oslo.concurrency 處理 Lock，底層透過 fasteners 呼叫 fcntl ，進而失敗。所以我們將 Lock 機制暫時取消掉。目前還無法確定取消掉之後會不會有其他問題存在。

<!--more-->

在 `/etc/kolla/nova.conf` 中，加入以下設定，將 Lock 關閉

<pre class="wp-block-code"><code># /etc/kolla/nova.conf
[oslo_concurrency]
disable_process_locking = True</code></pre>

接下來，直接在 Host 中掛載 NFS ，例如：

<pre class="wp-block-code"><code># Nova-Compute Host
mount -t nfs 172.x.x.x:/export/openstack /srv/nfs/openstack</code></pre>

然後設定 `/etc/kolla/globals.yml` ，掛載 NFS 目錄進 Container 之中

<pre class="wp-block-code"><code># /etc/kolla/globals.yml
nova_instance_datadir_volume: "/srv/nfs/openstack/"</code></pre>

並且依據先前調整 [Cinder-Volume NFS Driver][1]，對 QEMU 相關的修改即可。

 [1]: https://blog.kojuro.date/openstack-rocky-cinder-nfs-driver-issue/
