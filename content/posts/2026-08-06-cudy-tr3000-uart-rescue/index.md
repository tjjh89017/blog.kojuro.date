---
title: "Rescuing a Bricked Cudy TR3000 with a UART Cable"
subtitle: "Why my router could not boot, and how three U-Boot commands fixed it"
date: 2026-08-06T20:30:00+08:00
lastmod: 2026-08-06T20:30:00+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: "I bought a used Cudy TR3000 with a modified bootloader. When I tried to go back to the stock firmware, the router stopped booting. This is the story of how I found the real problem with a UART cable and fixed it without a flash programmer."
license: ""
images: []

tags: ["cudy", "tr3000", "openwrt", "uart", "mtk_uartboot", "mt7981", "ubi", "recovery", "router"]
categories: ["hardware", "network"]

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
  maxShownLines: 200
math:
  enable: false
mapbox:
share:
  enable: true
comment:
  enable: true
---

I bought a used Cudy TR3000 travel router. The seller had already flashed a modified U-Boot (the "ubootmod" style) and LEDE on it. I wanted to go back to the stock Cudy firmware. It did not go well. The router stopped booting, and the normal TFTP recovery did not work.

This post explains what was really broken, and how I fixed it with only a cheap USB-TTL cable. No flash programmer, no soldering.

<!--more-->

## The situation

The TR3000 uses a MediaTek MT7981 SoC with 128 MB of SPI-NAND flash. The stock flash layout looks like this:

```
BL2 | u-boot-env | Factory | bdinfo | FIP | ubi (64 MB)
```

The "ubootmod" build of OpenWrt keeps the first partitions in the same place, but it makes the `ubi` partition much bigger: about 112 MB instead of 64 MB.

I had a stock FIP (bootloader) dump from another device, and I wrote it back to the flash. The bootloader started fine. But the router still could not boot. It rebooted again and again, forever.

## Getting a serial console

To see what was happening, I needed a serial console. The [OpenWrt wiki page for the TR3000](https://openwrt.org/toh/cudy/tr3000) has the UART pinout. The two test points are on the back of the PCB. With the Ethernet port on the left, the left pad is RX and the right pad is TX. Use a 3.3V USB-TTL adapter at 115200 8N1. Do not connect VCC.

You do not need to solder. I just pressed Dupont wires on the pads. One tip that helped a lot: clip the ground wire to the metal shield of the Ethernet port. Then your hand only needs to hold two wires. My first tries gave me garbage data because the contact was not stable, so take your time here.

## What the boot log showed

With the console working, the problem became clear. The kernel booted, found the UBI area, and then failed:

```
ubi0 error: ubi_read_volume_table: the layout volume was not found
ubi0 error: ubi_attach_mtd_dev: failed to attach mtd6, error -22
Kernel panic - not syncing: VFS: Unable to mount root fs
Rebooting in 1 seconds..
```

So the bootloader was fine. The kernel was fine. The problem was the UBI data on the flash.

Here is the interesting part. The U-Boot log said it attached UBI with a size of **112 MB**. But the kernel said the `ubi` partition was only **64 MB**. Two parts of the same system did not agree about the flash layout.

Why? The answer was in the U-Boot environment:

```
mtd_layout=mod-112m
mtdparts=nmbm0:1024k(bl2),512k(u-boot-env),2048k(Factory),256k(bdinfo),2048k(fip),114688k(ubi)
```

The old modified U-Boot had saved its own layout into the `u-boot-env` partition. My stock U-Boot read this old environment and used the wrong 112 MB layout. The stock kernel used its own 64 MB layout from the device tree. The UBI on the flash was created with the big layout, so the kernel could never read it. That is why TFTP recovery also failed: every new firmware wrote into a flash area that still had old UBI data around it.

## Two more traps

While debugging, I found two more things that cost me hours:

**The stock U-Boot has no console window.** It boots with `bootdelay=0`, so you cannot press a key to stop it. The only way to reach the U-Boot shell is: hold the Reset button while powering on, with **no Ethernet cable connected**. The OEM recovery mode starts, waits for the network, times out after about 15 seconds, and then drops to the shell.

**The BootROM is always there.** Even if you destroy everything on the flash, the MT7981 has a small program inside the chip that can load a bootloader over UART. This means you can almost never fully brick this router, as long as you can reach the UART pads.

## The actual fix: three commands

So I used the Reset button trick: hold Reset, power on, no cable, wait for the timeout. After that I was in the stock U-Boot shell, and the fix was very small:

```
mtd erase ubi             # erase the whole 112 MB UBI area, with all old data
mtd erase u-boot-env      # delete the old environment with the wrong layout
reset
```

The order matters. You must erase `ubi` first, while the environment still says it is 112 MB. If you erase the environment first, the layout goes back to 64 MB, and you can no longer reach the old data behind it.

Do **not** touch `Factory` or `bdinfo`. The `Factory` partition holds the Wi-Fi calibration data and MAC addresses of your device. You cannot download it from anywhere.

After the reset, the router had a stock bootloader, a clean default environment, and an empty flash. Now the normal Cudy TFTP recovery worked on the first try: put the stock firmware as `recovery.bin` on a TFTP server, set the PC to `192.168.1.88` (new batches use `192.168.1.2`), connect the LAN port, hold Reset, and power on. A few minutes later, the router booted into the stock Cudy firmware.

By the way: the stock firmware has no SSH. Only the web UI at `192.168.10.1`. If you want a shell, the UART console gives you one.

## Plan B that I did not need: mtk_uartboot

Before I found the Reset button trick, I prepared a bigger hammer: [mtk_uartboot](https://github.com/981213/mtk_uartboot). It talks to the BootROM and loads a BL2 and a full U-Boot directly into RAM over the UART. Nothing is written to the flash, so you can try as many times as you want, even on a router with a fully destroyed flash.

I tested it far enough to see the handshake work (`hw code: 0x7981`) and the FIP arrive in RAM. Two practical notes: keep the baud rate at 115200 for stability, and if your adapter is an FT232R, run the tool in a retry loop — mine sometimes crashed with a write timeout. You need `mt7981-ram-ddr3-bl2.bin` (the TR3000 uses DDR3, not DDR4) and a `bl31-uboot.fip` from the OpenWrt download server.

In the end I did not need it, because the stock U-Boot shell was enough. But it is good to know this door is always open.

## How to run OpenWrt and still go back

After all this pain, I do not want to touch the flash layout ever again. The good news: you do not need to.

**Stock → OpenWrt:**

1. Flash the Cudy-signed OpenWrt image through the stock web UI. Cudy shares these signed images in a Google Drive folder linked from their [OpenWrt download FAQ](https://www.cudy.com/en-us/blogs/faq/openwrt-software-download). The stock UI checks signatures, so a normal OpenWrt image will not work here.
2. From that OpenWrt, run `sysupgrade -n` with the official image `cudy_tr3000-v1-squashfs-sysupgrade.bin` — the **normal** one, not the `ubootmod` one.

**OpenWrt → Stock:**

TFTP recovery, as described above. Because the layout never changed, the recovery writes into exactly the same area, and there is no leftover data problem.

The rule is simple: never flash anything with `ubootmod` in the name, and never touch BL2, FIP, or `u-boot-env`. Keep the stock bootloader, and the TFTP recovery is always there as your safety net.

One warning for buyers: TR3000 units with serial numbers starting from 2543 (made after November 2025) use a different flash chip and need different files. Check the label before you flash anything.

## Lessons learned

- A "brick" is often not one broken thing. Here, every single part worked. The parts just did not agree with each other.
- The boot log tells you everything. Getting the serial console working is worth the effort, even if holding wires on tiny pads is annoying.
- The old U-Boot environment is easy to forget. If you change bootloaders and layouts, `u-boot-env` remembers old settings that can bite you much later.
- Back up the `Factory` partition before you flash anything. It is the only part of the flash you can never replace.
