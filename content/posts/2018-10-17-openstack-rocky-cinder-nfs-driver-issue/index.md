---
title: OpenStack Rocky Cinder NFS Driver Issue
author: "Date Huang"
date: 2018-10-17T07:35:59+00:00
categories:
  - openstack

---
最近使用 Kolla-Ansible 部署 OpenStack Rocky on AArch64/ARM64 發生一些問題，這問題並不只是存在 ARM 上，而是因為發行版所維護的 QEMU 版本已經跟到了 commit [ca749954b09b89e22cd69c4949fb7e689b057963][1] ，這個 commit 使用到了 `F_OFD_SETLK`， NFSv3 正好不支援這類型 lock 操作。

而這個修改，並沒有根據不同檔案系統不支援 `F_OFD_SETLK` 回退 `F_SETLK` 的操作，所以只要在現代 Linux 中編譯 QEMU，就會直接使用 `F_OFD_SETLK` ，而造成在 NFSv3 上無法使用 lock 進而導致失敗的現象。

而這問題也不單單只影響 Cinder-Volume，也間接影響 Nova-Compute，如果 VM 要掛載 Volume，而 Cinder 使用 NFS Driver，則會在 Nova-Compute 上也把 NFS 掛載上去給 QEMU 使用。

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
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2018/10/openstack-rocky-cinder-nfs-driver-issue/#Kolla_Cinder_NFSv3_Lock_Failed" title="Kolla Cinder NFSv3 Lock Failed">Kolla Cinder NFSv3 Lock Failed</a><ul class="ez-toc-list-level-3">
        <li class="ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2018/10/openstack-rocky-cinder-nfs-driver-issue/#%E5%95%8F%E9%A1%8C" title="問題">問題</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2018/10/openstack-rocky-cinder-nfs-driver-issue/#%E8%A7%A3%E6%B1%BA%E6%96%B9%E6%B3%95" title="解決方法">解決方法</a>
        </li>
      </ul>
    </li>
    
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2018/10/openstack-rocky-cinder-nfs-driver-issue/#QEMU_Failed_to_lock_byte_100" title="QEMU Failed to lock byte 100">QEMU Failed to lock byte 100</a><ul class="ez-toc-list-level-3">
        <li class="ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-5" href="https://blog.kojuro.date/2018/10/openstack-rocky-cinder-nfs-driver-issue/#%E5%95%8F%E9%A1%8C-2" title="問題">問題</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-3">
          <a class="ez-toc-link ez-toc-heading-6" href="https://blog.kojuro.date/2018/10/openstack-rocky-cinder-nfs-driver-issue/#%E8%A7%A3%E6%B1%BA%E8%BE%A6%E6%B3%95" title="解決辦法">解決辦法</a>
        </li>
      </ul>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="Kolla_Cinder_NFSv3_Lock_Failed"></span>Kolla Cinder NFSv3 Lock Failed<span class="ez-toc-section-end"></span>

### <span class="ez-toc-section" id="%E5%95%8F%E9%A1%8C"></span>問題<span class="ez-toc-section-end"></span>

Kolla Cinder 已經在 commit [17e6e629f5660a364498b2bfd2f5aa6aede79859][2] 中開啟 NFS 功能，但是將 rpcbind 取消，NFSv4 已經不需要 rpcbind ，所以將他移出必要元件之一。

### <span class="ez-toc-section" id="%E8%A7%A3%E6%B1%BA%E6%96%B9%E6%B3%95"></span>解決方法<span class="ez-toc-section-end"></span>

分兩種：

  1. 直接在 Host (包含 Cinder-Volume Host 以及 Nova-Compute Node)上安裝 rpcbind ，然後啟用，確保 rpcbind socket 位置在 `/run` ， Docker Container 也有 Mount Volume
  2. 安裝 rpcbind 的 Container ，直接 Container Volume Bind

## <span class="ez-toc-section" id="QEMU_Failed_to_lock_byte_100"></span>QEMU Failed to lock byte 100<span class="ez-toc-section-end"></span>

### <span class="ez-toc-section" id="%E5%95%8F%E9%A1%8C-2"></span>問題<span class="ez-toc-section-end"></span>

如果欲將 Image 轉換成 Volume 來使用的時候，會透過 qemu-img 來轉換，這過程如果目標地在 NFS 上，則會遇到我在開頭提及的問題，進而產生 `QEMU Failed to lock byte 100`

### <span class="ez-toc-section" id="%E8%A7%A3%E6%B1%BA%E8%BE%A6%E6%B3%95"></span>解決辦法<span class="ez-toc-section-end"></span>

建議是不要用 NFSv3 ，改用其他種 Backend 。雖然講的很不負責任，但是 NFSv3 事實上已經很久的技術了，雖然很成熟，但是已經不太符合現代 HA 等等需求，建議還是替換。個人也是因為經費因素，沒有辦法替換成 Ceph 或是其他的方案，則還是希望使用 NFS 作為 Cinder Backend，甚至 Nova `instances_path` 都希望直接在 NFS Storage 上。

若無法替換則使用下面方法。

重新編譯 QEMU ，強制取消使用 `F_OFD_SETLK` ，若使用 Ubuntu 等等的發行版，建議直接從發行版來源上 patch 重新編譯即可

<pre class="wp-block-code"><code>apt build-dep qemu
apt install -y fakeroot
apt source qemu
cd qemu/
# let macro will never be compiled
# replace all "#ifdef F_OFD_SETLK" to "#if 0"
sed -i.bak 's|def F_OFD_SETLK| 0|g' utils/osdep.c

DEB_BUILD_OPTIONS="parallel=8" fakeroot debian/rules clean binary
# *.deb will place at ../</code></pre>

編譯完之後，將打包好的 deb 放入 Container 中以便安裝，以下 Container 皆需要安裝新的 package ，似乎其他 OpenStack 模組也需要上 patch ，但是我沒有細看，所以只列出我有使用到的部分

  * nova_libvirt
  * nova_compute
  * cinder_api
  * cinder_volume
  * cinder_scheduler

<pre class="wp-block-code"><code>ls *.deb | xargs -i docker cp {} &lt;Container Name>:/var/lib/kolla/</code></pre>

根據不同的 Container 安裝需要的 QEMU package 即可，並不需要全部安裝

<pre class="wp-block-code"><code>docker exec -it -u root &lt;Container Name> bash
# inside Container
# Check installed QEMU package
apt list --installed | grep qemu
cd /var/lib/kolla/
# install patched qemu 
dpkg -i qemu-&lt;some need package name>.deb</code></pre>

可能需要重新啟動 Container Nova-Libvirt，其餘部分就可以直接使用。

 [1]: https://github.com/qemu/qemu/commit/ca749954b09b89e22cd69c4949fb7e689b057963
 [2]: https://github.com/openstack/kolla/commit/17e6e629f5660a364498b2bfd2f5aa6aede79859
