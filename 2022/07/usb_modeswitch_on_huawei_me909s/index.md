# USB Modeswitch on Huawei ME909s


簡單記錄下， debug 華為 Mini PCIe LTE module ME909s-821 在 Debian 上面的問題，同樣的可以套用到 VyOS 上面。

<!--more-->

## USB Modeswitch

華為的卡幾乎都是兩個以上的模式，有幾個系列的卡可能正常，但是 ME909s 相對起來只要切換設定檔而已，反而會造成 udev 一直重新被觸發，然後就卡住了。

ME909s 大部分都會想把 Configuration 從預設的 2 (cdc_ether) 轉換成 3 (cdc_mbim)，這時候就會需要 usb-modeswitch 的協助。

一般來說我們會新增一個檔案 `/etc/usb_modeswitch.d/12d1:15c1` ，內容如下

```
Configuration=3
```

## Issue

但是 usb-modeswitch 的 udev rule 沒有寫好，導致設定到一半就被中斷重新設定一次，進而導致設定失敗。

`/lib/udev/usb_modeswitch` 中會一直觸發到 `systemctl restart usb-modeswitch@XXX.service` ，所以 ME909s 被設定到一半就被中斷，第二次設定可能依然失敗，進而觸發第三次設定。

## Fix (updated in 2022/07/31)

把 `/lib/udev/rules.d/40-usb_modeswitch.rules` 中的

```
SUBSYSTEM!="usb", ACTION!="add",, GOTO="modeswitch_rules_end"
```

修改成

```
SUBSYSTEM!="usb", GOTO="modeswitch_rules_end"
ACTION!="add", GOTO="modeswitch_rules_end"
```

順便發了個 patch 上 upstream 了，希望會收進去
[https://www.draisberghof.de/usb_modeswitch/bb/viewtopic.php?f=2&t=3034](https://www.draisberghof.de/usb_modeswitch/bb/viewtopic.php?f=2&t=3034)

## Fix (old)

把 `/lib/udev/rules.d/40-usb_modeswitch.rules` 中的

```
# Huawei ME906, ME909 (MBIM, dummy config)
ATTR{idVendor}=="12d1", ATTR{idProduct}=="15c1", RUN+="usb_modeswitch '/%k'"
```

修改成

```
# Huawei ME906, ME909 (MBIM, dummy config)
ACTION=="add", ATTR{idVendor}=="12d1", ATTR{idProduct}=="15c1", RUN+="usb_modeswitch '/%k'"
```



