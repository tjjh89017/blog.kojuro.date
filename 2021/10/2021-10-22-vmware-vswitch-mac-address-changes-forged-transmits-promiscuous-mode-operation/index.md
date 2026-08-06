# VMware vSwitch MAC Address Changes, Forged Transmits, Promiscuous Mode Operation

VMware 的 vSwitch 中有以下三個選項可以讓你使用

  * MAC Address Changes
  * Forged Transmits
  * Promiscuous Mode Operation

主要是為了讓 VM 內的流量可以正常出來而需要選擇的，一般來說如果是要做 VM in VM了話，直接將三個一次打開是最簡單的處理方案。

這邊就依據三個分別去簡單解釋不同意義

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
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2021/10/vmware-vswitch-mac-address-changes-forged-transmits-promiscuous-mode-operation/#Overview" title="Overview">Overview</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2021/10/vmware-vswitch-mac-address-changes-forged-transmits-promiscuous-mode-operation/#MAC_Address_Changes" title="MAC Address Changes">MAC Address Changes</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2021/10/vmware-vswitch-mac-address-changes-forged-transmits-promiscuous-mode-operation/#Forged_Transmits" title="Forged Transmits">Forged Transmits</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-4" href="https://blog.kojuro.date/2021/10/vmware-vswitch-mac-address-changes-forged-transmits-promiscuous-mode-operation/#Promiscuous_Mode_Operation" title="Promiscuous Mode Operation">Promiscuous Mode Operation</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-5" href="https://blog.kojuro.date/2021/10/vmware-vswitch-mac-address-changes-forged-transmits-promiscuous-mode-operation/#VM_in_VM" title="VM in VM">VM in VM</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-6" href="https://blog.kojuro.date/2021/10/vmware-vswitch-mac-address-changes-forged-transmits-promiscuous-mode-operation/#Ref" title="Ref">Ref</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="Overview"></span>Overview<span class="ez-toc-section-end"></span>

首先是，VMware 中的 vSwitch 並沒有 MAC Learning 的功能，所有 MAC 與 Switch Port 的對應，都是由 ESXi 或是 vCenter 靜態寫入的。意思就是，今天如果 VM 裡面有多組 MAC address ，例如說：macvlan, macvtap，vSwitch 並不會學習到這個額外 MAC 與 Switch Port 的對應。這某方面來說是安全考量。

## <span class="ez-toc-section" id="MAC_Address_Changes"></span>MAC Address Changes<span class="ez-toc-section-end"></span>

VM 的描述檔案中會記錄所有網卡的 MAC Address，ESXi 會直接根據這個檔案去填寫 vSwitch 的 MAC Table，進而透過這個表去發送封包。但是今天如果 VM 內部更改了 MAC Address 並發送資料，預設並不會更新 MAC Table，所以資料就無法正確地傳送到 VM 的 Port。

啟用這功能，則可以讓 VM 更改 MAC Address 並發送封包的時候，將 MAC Table 中的原本地址取代為新的地址。

注意：這邊並沒有新增 MAC Table 的紀錄，而是取代原本的紀錄。

## <span class="ez-toc-section" id="Forged_Transmits"></span>Forged Transmits<span class="ez-toc-section-end"></span>

ESXi 預設會比對從 VM 出來至 vSwitch 的來源 MAC Address 是否與 MAC Table 中的對應相符，若不相符則丟棄封包。

啟用這功能，則可以讓 VM 送出非記錄中的 MAC Address 封包，且 vSwitch 會略過檢查直接放行。

注意：這邊只關注 VM 到 vSwitch 的封包，反向的封包則依原本 MAC Table 處理，所以即使可以發送出去，但是因為 MAC Table 沒有更新或是新增紀錄，所以回程的封包則會被 vSwitch 丟棄。

## <span class="ez-toc-section" id="Promiscuous_Mode_Operation"></span>Promiscuous Mode Operation<span class="ez-toc-section-end"></span>

因為 MAC Table 不會更新或新增，所以即使能夠正常送出封包，封包也無法正確回到原本 VM 上面。此選項會關閉所有 RX 方向的 Filter，將所有封包傳送至啟用此功能的網卡中

請用這功能，會設定該 Port Group 或是 vSwitch 的 Mirror 功能，所有流量都會送往啟用該功能的網卡中。大部分用於網路行為的測試與觀察。

注意：會類似關閉 MAC Table 功能，讓 vSwitch 接近 Hub ，可能導致效能降低。

## <span class="ez-toc-section" id="VM_in_VM"></span>VM in VM<span class="ez-toc-section-end"></span>

一般來說會考慮到這些功能的可能性，大部分是 VM in VM，就例如說想要測試不同種類的 Hypervisor ，就會在 VMware 內部再啟用巢狀虛擬化，並且在網卡上面需要分配給 VM 中的 VM。就會需要將上述功能都打開，才會正常運作。

但是 VMware 並不建議 VM in VM。

## <span class="ez-toc-section" id="Ref"></span>Ref<span class="ez-toc-section-end"></span>

  * <https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-3507432E-AFEA-4B6B-B404-17A020575358.html>
  * <https://williamlam.com/2018/04/native-mac-learning-in-vsphere-6-7-removes-the-need-for-promiscuous-mode-for-nested-esxi.html>

