---
title: "Adding Resizable BAR and UEFI Video to a Huananzhi X99-CH8 BIOS"
subtitle: "Two small drivers, a few Python scripts, and no flash programmer"
date: 2026-10-27T16:38:53+08:00
lastmod: 2026-10-27T16:38:53+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: "My Huananzhi X99-CH8 is a cheap Chinese server board with an Intel C612 chipset. The BIOS has no Resizable BAR and no UEFI driver for the onboard video. I added both by inserting two DXE drivers into the BIOS image. This post explains how I did it, and how I tested it on the real board."
license: ""
images: []

tags: ["x99", "huananzhi", "bios", "uefi", "rebar", "resizable-bar", "aspeed", "ast2400", "gop", "flashrom", "aptio"]
categories: ["hardware"]

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

I have a Huananzhi X99-CH8 board. It is a cheap Chinese server board with an Intel C612 chipset, a Xeon E5 v4 CPU, and an ASPEED AST2400 chip for onboard video. I use it as a home server with an AMD RX 6700 XT.

The stock BIOS has two problems. It does not support Resizable BAR. And the onboard video only has a legacy VBIOS, so when I turn off CSM, the screen stays black. I fixed both by adding two drivers to the BIOS image. All files and scripts are on GitHub: [huananzhi-x99-ch8-bios-mod](https://github.com/tjjh89017/huananzhi-x99-ch8-bios-mod).

<!--more-->

## What I wanted

1. **Resizable BAR.** With ReBAR, the CPU can see the whole VRAM of the GPU, not only a 256 MB window. Linux can do this by itself with the `amdgpu` driver. Windows cannot. It needs the BIOS to set the BAR size at boot.
2. **Onboard video without CSM.** The AST2400 only has a legacy option ROM. A UEFI GOP driver is needed to get a picture when CSM is off.

## What the board has

Before changing anything, I looked at the dump of the BIOS chip.

| Item | Value |
|---|---|
| Board | Huananzhi X99-CH8 v3.0 |
| Chipset | Intel C612 (Wellsburg) |
| BIOS | AMI Aptio IV, version G3X99018, 2022-08-27 |
| Flash chip | Winbond W25Q128, 16 MiB |
| Onboard video | ASPEED AST2400 (PCI rev 30, behind an AST1150 bridge) |
| Boot Guard | No |
| Setup option "Above 4G Decoding" | Yes, already there |

No Boot Guard and no signed BIOS check. This means I can change the image and the board will still boot it.

## Step 1: Dump the BIOS

I did not have a CH341A programmer. But `flashrom` can read and write the chip from Linux on the board itself, because the BIOS does not lock the SPI flash. I read the chip two times and compared the files.

```bash
sudo flashrom -p internal -r dump1.bin
sudo flashrom -p internal -r dump2.bin
sha256sum dump1.bin dump2.bin
```

Both files had the same hash. This dump is also my backup if something goes wrong.

## Step 2: Get the two drivers

**ReBarDxe** comes from the [ReBarUEFI](https://github.com/xCuri0/ReBarUEFI) project. The release has a ready `ReBarDxe.ffs` file. An FFS file is a small container that goes into a UEFI firmware volume.

**AST2400 GOP** comes from the ASPEED UEFI driver package v1.11.03. The file is `uefi_2400_396.efi`. It is a normal PE file. I had to wrap it as a DXE driver FFS file myself. I gave it a new GUID so it does not clash with anything in the image.

## Step 3: Insert the drivers

Most people use UEFITool for this step. I tried UEFITool 0.28, but it broke a padding file in the PEI volume of this image. So I wrote two small Python scripts instead.

- `mkffs.py` builds an FFS file from a PE file. It uses the ReBarDxe FFS as a template.
- `insert_ffs.py` appends an FFS file to the free space at the end of a firmware volume.

The main DXE volume starts at offset `0x880000` in the image. There was a lot of free space at the end. Both drivers went there. Only bytes between `0xB2FE10` and `0xB3B73B` changed. The rest of the image is the same as the dump.

```bash
python3 tools/mkffs.py \
    --template third_party/ReBarUEFI/ReBarDxe.ffs \
    --pe third_party/ASPEED/uefi_2400_396.efi \
    --name AstGopDxe \
    --guid 5AAFD210-7B28-4F20-A12E-C59C0A7C355B \
    --state 0xF8 \
    -o AstGopDxe.ffs

python3 tools/insert_ffs.py dump1.bin 0x880000 \
    third_party/ReBarUEFI/ReBarDxe.ffs AstGopDxe.ffs \
    -o X99-CH8_G3X99018_rebar_astgop.bin
```

One small detail: the FFS file from ReBarUEFI has a state byte of `0x07`. This firmware volume uses the opposite erase polarity, so the state byte must be `0xF8`. The insert script fixes this when it writes the file.

I also made a second image with only ReBarDxe. It was my safer first choice. In the end, I flashed the full image.

## Step 4: Check the image

I did not want to trust my own scripts. So I wrote a second set of small parsers that read the new image from zero: flash descriptor, firmware volumes, FFS files, sections, and GUIDs. They confirmed that every volume still has a correct checksum, that no other file moved, and that both new drivers are in the right place.

The ReBarUEFI project also has some patches for the PCI host bridge code. I did not apply them. They are written for desktop chipsets. On Haswell-EP the module names are different, and the patterns do not match. I decided to test without them first.

## Step 5: Flash

Again, no programmer. I wrote only the BIOS region with `flashrom` on the board. The descriptor and ME regions stay untouched.

```bash
sha256sum -c X99-CH8_G3X99018_rebar_astgop.bin.sha256
sudo flashrom -p internal --ifd -i bios -w X99-CH8_G3X99018_rebar_astgop.bin
```

`flashrom` reads the chip back after writing and prints `VERIFIED`. Then I rebooted.

This step can brick the board. If the write fails in the middle, do not reboot. Write `dump1.bin` back with the same command. If the board does not boot at all, only a programmer can save it.

## Step 6: Turn on ReBAR

ReBarDxe does nothing until you create an NVRAM variable called `ReBarState`. The value `0x20` means "no limit, use the biggest BAR". On Linux, you can write it directly to `efivarfs`. The first four bytes are the variable attributes.

```bash
printf '\x07\x00\x00\x00\x20' | sudo tee \
    /sys/firmware/efi/efivars/ReBarState-a3c5b77a-c88f-4a93-bf1c-4a92a32c65ce
```

On Windows, use `ReBarState.exe` from the ReBarUEFI release.

The write worked on the first try. Some boards block unknown NVRAM variables and need one more patch. This board does not.

## Results

After the next reboot, the kernel log showed this at boot time:

```
pci 0000:05:00.0: BAR 0 [mem 0x383800000000-0x383bffffffff 64bit pref]
```

That is a 16 GB BAR, set by the firmware, before any driver loads. Before the mod, the same line showed 256 MB, and `amdgpu` had to resize it later. The BAR sits above 4 GB, so the "Above 4G Decoding" option in Setup does its job and no extra patch is needed.

The GPU link is PCIe Gen3 x16 all the way to the CPU root port. Nothing dropped to x8.

Then I turned off CSM in Setup. The onboard AST2400 still showed the BIOS screen. The GOP driver works.

## Tips if you want to do this

- A cheap X99 board without Boot Guard is easy to work on. You can change the BIOS and test it, and `flashrom` from Linux is enough. You do not need a programmer, but one is good to have in case the board does not boot.
- Keep the dump. Check its hash two times before you change anything.
- Do not trust one tool. UEFITool broke my image in a quiet way. If you use it, check the result with a second parser.
- Only write the BIOS region. A broken descriptor is much harder to fix than a broken BIOS.
- To check ReBAR, read the kernel log at boot time, not only `lspci`. On Linux, `amdgpu` resizes the BAR by itself, so `lspci` looks the same with or without the mod.

## Next

The board has a real COM port, but the BIOS cannot send its console to it. I want to add `SerialDxe` and `TerminalDxe` from edk2 next, with an NVRAM switch to turn the console on and off. That is a story for another post.
