---
title: OpenStack on AArch64/ARM64 via Kolla-Ansible
date: 2018-10-14T08:10:14+00:00
#featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
#toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#usePageBundles: true # Set to true to group assets like images in the same folder as this post.
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#featureImageAlt: 'Description of image' # Alternative text for featured image.
#featureImageCap: 'This is the featured image.' # Caption (optional).
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
#categories:
#  - Technology
#tags:
#  - Tag_name1
#  - Tag_name2
# comment: false # Disable comment if false.
categories:
  - OpenStack
---

# Openstack on arm64 via kolla-ansible

OpenStack on AArch64/ARM64 已經可以透過 Kolla-Ansible 直接部署，一般在 x86_64 部署 OpenStack 可以參考 [Gene Kuo 的部落格文章][1] ，我這邊就不再贅述。

這篇主要說明如何在 x86_64 與 Arm64 混合雲的情況下，透過 Kolla-Ansible 直接部署 OpenStack Rocky 並且可以正常使用。<!--more-->

<div id="ez-toc-container" class="ez-toc-v2_0_17 counter-hierarchy counter-decimal ez-toc-grey">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title">
      Table of Contents
    </p>

    <span class="ez-toc-title-toggle"><a class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" style="display: none;"><i class="ez-toc-glyphicon ez-toc-icon-toggle"></i></a></span>
  </div><nav>

  <ul class="ez-toc-list ez-toc-list-level-1">
    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E5%9F%BA%E7%A4%8E%E6%9E%B6%E6%A7%8B" title="基礎架構">基礎架構</a><ul class="ez-toc-list-level-2">
        <li class="ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E7%B6%B2%E8%B7%AF%E4%BB%8B%E9%9D%A2" title="網路介面">網路介面</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E4%BC%BA%E6%9C%8D%E5%99%A8" title="伺服器">伺服器</a>
        </li>
      </ul>
    </li>

    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E6%BA%96%E5%82%99" title="準備">準備</a><ul class="ez-toc-list-level-2">
        <li class="ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-5" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#Kolla" title="Kolla">Kolla</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-6" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#Kolla-Ansible" title="Kolla-Ansible">Kolla-Ansible</a>
        </li>
      </ul>
    </li>

    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-7" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E5%AE%89%E8%A3%9D%E8%A8%AD%E5%AE%9A" title="安裝設定">安裝設定</a><ul class="ez-toc-list-level-2">
        <li class="ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-8" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#Kolla-Ansible_%E8%A8%AD%E5%AE%9A" title="Kolla-Ansible 設定">Kolla-Ansible 設定</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-9" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#Ansible_Inventory_%E8%A8%AD%E5%AE%9A" title="Ansible Inventory 設定">Ansible Inventory 設定</a>
        </li>
      </ul>
    </li>

    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-10" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E9%83%A8%E7%BD%B2" title="部署">部署</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-11" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E4%BD%BF%E7%94%A8" title="使用">使用</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-12" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#OpenStack_on_AArch64" title="OpenStack on AArch64">OpenStack on AArch64</a><ul class="ez-toc-list-level-2">
        <li class="ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-13" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E6%98%A0%E5%83%8F%E6%AA%94" title="映像檔">映像檔</a>
        </li>
        <li class="ez-toc-page-1 ez-toc-heading-level-2">
          <a class="ez-toc-link ez-toc-heading-14" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#VNC_Console" title="VNC Console">VNC Console</a>
        </li>
      </ul>
    </li>

    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-15" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#%E5%BE%8C%E8%A8%98" title="後記">後記</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-1">
      <a class="ez-toc-link ez-toc-heading-16" href="https://blog.kojuro.date/2018/10/openstack-on-aarch64-arm64-via-kolla-ansible/#Reference" title="Reference">Reference</a>
    </li>
  </ul></nav>
</div>

# <span class="ez-toc-section" id="%E5%9F%BA%E7%A4%8E%E6%9E%B6%E6%A7%8B"></span>基礎架構<span class="ez-toc-section-end"></span>

一個 HA 的 OpenStack 需要至少 4 個節點，分別為 3 個 control node ，與 1 個 compute node，每個節點至少需要 2 個網路介面，不過我下面會使用 3 個網路介面作為範例，將外部與內部的 API 使用分離。這邊使用 3 個 x86_64 control node，以及 1 個 ARM server 作為 compute node

## <span class="ez-toc-section" id="%E7%B6%B2%E8%B7%AF%E4%BB%8B%E9%9D%A2"></span>網路介面<span class="ez-toc-section-end"></span>

<li style="list-style-type: none;">
  <ul>
    <li style="list-style-type: none;">
      <ul>
        <li>
          <code>api_interface</code> 為內部 API 使用的介面
        </li>
        <li>
          <code>neutron_external_interface</code> 為 VM 或是其他 OpenStack 網路功能所對外的介面，無法使用 IP ，如果使用虛擬機則必須要允許這個介面使用 <code>Promiscuous Mode</code>
        </li>
        <li>
          <code>kolla_external_vip_interface</code> 為 HAProxy 所使用對外服務的介面
        </li>
      </ul>
    </li>
  </ul>
</li>

## <span class="ez-toc-section" id="%E4%BC%BA%E6%9C%8D%E5%99%A8"></span>伺服器<span class="ez-toc-section-end"></span>

  * Mgmt 作為 Kolla-Ansible 執行的環境
  * control01 ~ 03 為 OpenStack Control Node，為 x86
  * compute01 為 ARM server ，為OpenStack Compute Node ，欲執行 ARM VM

# <span class="ez-toc-section" id="%E6%BA%96%E5%82%99"></span>準備<span class="ez-toc-section-end"></span>

## <span class="ez-toc-section" id="Kolla"></span>Kolla<span class="ez-toc-section-end"></span>

Kolla 並沒有在 Docker Hub 上面預先準備 aarch64 的 image 給大家使用，所以我們必須要自行編譯 OpenStack Kolla image 給 aarch64 使用，於 ARM64 server 中執行

<pre>git clone https://github.com/openstack/kolla -b stable/rocky</pre>

因為 OpenStack Rocky 預設會透過 Libvirt 尋找可以 PCI Passthrough 以及 SR-IOV 的設備，但是 Libvirt 並無法正確判讀 Cavium ThunderX 網卡功能，所以我們將該功能直接從 OpenStack 之中去除，於 `kolla/docker/nova/nova-compute/Dockerfile.j2` 中，於 `COPY extend_start.sh /usr/local/bin/kolla_nova_extend_start` 之前加入以下 patch

<pre>RUN rm -f /var/lib/kolla/venv/lib/python2.7/site-packages/nova/virt/libvirt/driver.pyc
RUN sed -i "/data\['pci_passthrough_/d" /var/lib/kolla/venv/lib/python2.7/site-packages/nova/virt/libvirt/driver.py
RUN sed -i "/self._get_pci_passthrough_devices()/d" /var/lib/kolla/venv/lib/python2.7/site-packages/nova/virt/libvirt/driver.py</pre>

並且執行編譯並推送至自架的 Docker Registry 中，在這邊我們推薦來源為 source ，而非 binary (發行版維護)

<pre>cd kolla
tools/build.py --base ubuntu --base-arch aarch64 --tag rocky --type source --registry &lt;Your Registry IP or Hostname&gt;:5000 --push</pre>

## <span class="ez-toc-section" id="Kolla-Ansible"></span>Kolla-Ansible<span class="ez-toc-section-end"></span>

pip 中能夠安裝的 Kolla-Ansible 並沒有我們需要的版本，也因為要修改，所以直接使用 upstream 版本會相對方便許多

<pre>git clone https://github.com/openstack/kolla-ansible -b stable/rocky
pip install -U ansible
cp kolla-ansible/etc/kolla/ /etc/kolla</pre>

該版本似乎存在一些問題，導致 rabbitmq 沒有辦法正確識別參數，所以需要修正。這問題似乎在 commit 5bfcb584275 後修好了，可以考慮直接使用新版的 kolla-ansible

<pre>cd kolla-ansible/ansible/roles
grep -r transport_url | awk -F":" '{print $1}' | xargs -i sed -i 's|transport_url = {{ rpc_transport_url }}|transport_url = {{ rpc_transport_url }}/|g' {}
grep -r transport_url | awk -F":" '{print $1}' | xargs -i sed -i 's|transport_url = {{ notify_transport_url }}|transport_url = {{ notify_transport_url }}/|g' {}</pre>

# <span class="ez-toc-section" id="%E5%AE%89%E8%A3%9D%E8%A8%AD%E5%AE%9A"></span>安裝設定<span class="ez-toc-section-end"></span>

## <span class="ez-toc-section" id="Kolla-Ansible_%E8%A8%AD%E5%AE%9A"></span><span id="Ansible_Inventory" class="ez-toc-section">Kolla-Ansible 設定</span><span class="ez-toc-section-end"></span>

Kolla-Ansible 的設定都會存在 /etc/kolla 之中，主要修改 /etc/kolla/globals.yml 來改變參數，此部分先參照 Gene Kuo 的教學文章，我未來再補上，要注意的是必須關閉 fluentd 功能，這功能目前無法用在 AArch64 上面。

<pre>enable_fluentd="no"</pre>

## <span class="ez-toc-section" id="Ansible_Inventory_%E8%A8%AD%E5%AE%9A"></span><span id="Ansible_Inventory" class="ez-toc-section">Ansible Inventory 設定</span><span class="ez-toc-section-end"></span>

我們先複製預設的多節點部署做為基底修改

<pre>cp kolla-ansible/ansible/inventory/multinode .</pre>

因為我們要部署混合平台的 OpenStack，即為 Control Node 為 x86 ，Compute Node 為 AArch64 ，我們會利用 `multinode` 這個 Inventory 覆蓋相關的設定，並確定你的 ansible user 可以不需要密碼直接使用 `sudo`

<pre>[control]
control[01:03] ansible_ssh_user=&lt;username&gt; ansible_become=True ansible_private_key_file=&lt;priv key&gt; neutron_external_interface=ens224 api_interface=ens192 storage_interface=ens192 tunnel_interface=ens192 remote_src=True

[network:children]
control

[inner-compute]

[external-compute]
compute01 ansible_ssh_user=&lt;username&gt; ansible_become=True ansible_private_key_file=&lt;priv key&gt; neutron_external_interface=enP2p1s0f3 api_interface=enP2p1s0f2 storage_interface=enP2p1s0f2 tunnel_interface=enP2p1s0f2 docker_registry=&lt;Your Registry IP&gt;:5000 remote_src=True
</pre>

# <span class="ez-toc-section" id="%E9%83%A8%E7%BD%B2"></span>部署<span class="ez-toc-section-end"></span>

先產生 OpenStack 部署中所需要的所有密碼，密碼會在 `/etc/kolla/passwords.yml` 之中

<pre>cd kolla-ansible
tools/generate_passwords.py</pre>

然後安裝必要套件，大部分不太會有意外，但是 AArch64 伺服器上不一定有辦法正確執行，所以推薦先行安裝 `docker` 與 `python-dev` 等等套件

<pre>tools/koll-ansible -i ../multinode bootstrap-servers</pre>

部署前測試，會測試相關資源是否被使用到

<pre>tools/kolla-ansible -i ../multinode prechecks</pre>

實際部署，會從 Registry 中拉取 image 使用，並且傳送設定檔至目標伺服器

<pre>tools/kolla-ansible -i ../multinode deploy</pre>

# <span class="ez-toc-section" id="%E4%BD%BF%E7%94%A8"></span>使用<span class="ez-toc-section-end"></span>

若部署沒有出現問題，則可以使用 `/etc/kolla/passwords.yml` 中的 `keystone_admin_password` ，帳號為 admin 登入 OpenStack 介面

透過以下指令可以生產 CLI 使用比較方便的 openrc 予以使用，並存在 `/etc/kolla/admin-openrc.sh` 中

<pre>tools/kolla-ansible -i ../multinode post-deploy</pre>

# <span class="ez-toc-section" id="OpenStack_on_AArch64"></span>OpenStack on AArch64<span class="ez-toc-section-end"></span>

AArch64 會有一些跟 x86 不同的地方，因為 Linaro 已經做了相當多的修改，所以我們必要的修正只有少數一些設定

## <span class="ez-toc-section" id="%E6%98%A0%E5%83%8F%E6%AA%94"></span>映像檔<span class="ez-toc-section-end"></span>

開機映像檔需要新增一些參數，讓 AArch64 VM 可以正常運作，我們必須使用 UEFI 映像檔開機，以 Ubuntu Cloud Image 為例，16.04 需要下載檔名含有 `uefi` 的映像檔，18.04 以後則預設為 UEFI 開機

<pre>openstack image create "image display name" --file "&lt;image file&gt;" --disk-format qcow2 --container-format bare --property hw_scsi_model='virtio-scsi' --property hw_disk_bus='scsi' --property hw_firmware_type='uefi' --public</pre>

## <span class="ez-toc-section" id="VNC_Console"></span>VNC Console<span class="ez-toc-section-end"></span>

AArch64 的顯示並非如 x86 直接使用 vga 設備，而是透過 `virtio-gpu-pci` 這外部裝置，Ubuntu Cloud Image 中的 Linux Kernel 並沒有預設啟用 `virtio-gpu` 這個模組，也並沒有預設安裝，所以在 VM 開機完畢之後，需要安裝 `linux-modules-extra` 的套件，並預設啟動模組以及修改設定。

於 `/etc/default/grub` 中，需修改

<pre>GRUB_CMDLINE_LINUX_DEFAULT="console=tty1"</pre>

然後執行

<pre>sudo update-grub</pre>

安裝並預設啟用 `virtio-gpu` 模組，`linux-image-extra-virtual` 為虛擬套件，他會安裝其他必要模組

<pre>sudo apt install -y linux-image-extra-virtual</pre>

於 `/etc/modules` 中新增 virtio-gpu

<pre># /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
virtio-gpu</pre>

並重新打包 `initramfs`

<pre>sudo update-initramfs -u</pre>

重開機後即可於 AArch64 VM 正常使用 VNC console

# <span class="ez-toc-section" id="%E5%BE%8C%E8%A8%98"></span>後記<span class="ez-toc-section-end"></span>

因為很無聊，所以在 VM 上面安裝了桌面環境來玩看看，但是因為沒有 GPU 加速，所以真的是滿慢的

<img loading="lazy" class="alignnone size-medium wp-image-108" src="https://blog.kojuro.date/wp-content/uploads/2018/10/43878894_2384973168183781_9222119465477472256_o-1-300x193.jpg" alt="" width="300" height="193" srcset="https://blog.kojuro.date/wp-content/uploads/2018/10/43878894_2384973168183781_9222119465477472256_o-1-300x193.jpg 300w, https://blog.kojuro.date/wp-content/uploads/2018/10/43878894_2384973168183781_9222119465477472256_o-1-768x494.jpg 768w, https://blog.kojuro.date/wp-content/uploads/2018/10/43878894_2384973168183781_9222119465477472256_o-1-1024x659.jpg 1024w, https://blog.kojuro.date/wp-content/uploads/2018/10/43878894_2384973168183781_9222119465477472256_o-1.jpg 1442w" sizes="(max-width: 300px) 100vw, 300px" />

&nbsp;

# <span class="ez-toc-section" id="Reference"></span>Reference<span class="ez-toc-section-end"></span>

[透過 Kolla-Ansible 跟 Container 部署 OpenStack][1]

 [1]: https://igene.tw/kolla-ansible-deploy
