---
title: "EZIO: Deploying Hundreds of Machines with BitTorrent"
subtitle: "A raw-disk imaging tool that gets faster the more clients you add"
date: 2026-05-23T22:40:00+08:00
lastmod: 2026-05-23T22:40:00+08:00
draft: false
author: "Date Huang"
authorLink: ""
description: "Multicast imaging breaks down at scale: one global sync barrier, one slow client stalls everyone. EZIO flips the model - it distributes raw disk images peer-to-peer with BitTorrent and writes straight to the partition, so deployment gets faster as you add machines."
license: ""
images: []

tags: ["clonezilla", "ezio", "bittorrent", "bt", "deployment", "imaging", "libtorrent", "openstack", "ironic"]
categories: ["ezio"]

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
share:
  enable: true
comment:
  enable: true
seo:
  images: []
---

Back in 2016, my roommate Ching-Hsuan Yen and I started building something that began as an offhand remark from a senior, Ping-Chun Huang: *"why not just use BitTorrent?"* The problem we were staring at was Clonezilla's multicast mode - why was it so unstable, and what could replace it? Years later that idea is [EZIO](https://github.com/tjjh89017/ezio), a BitTorrent-based raw-disk imaging tool that ships inside Clonezilla. I [wrote a short note when the first stable Clonezilla BitTorrent mode landed in 2019]({{< ref "/posts/2019-01-31-clonezilla-bittorrent-deploy" >}}); this post is the proper introduction it never got.

<!--more-->

## First, what "deploying an OS image" means

If you've never run a lab or a server fleet, here's the workflow EZIO lives in. Say you need 200 machines (a computer classroom, a render farm, a rack of servers) to all boot the same operating system, with the same packages, drivers and config. You don't install Windows or Linux 200 times by hand. Instead you set up **one** machine exactly the way you want it, then take a byte-for-byte snapshot of its disk, called a **"golden image"**, and copy that image onto every other machine's disk. Power them on and they're all clones of the original.

This is what tools like [Clonezilla](https://clonezilla.org/) do: capture a disk/partition into an image, then **restore** that image onto many targets at once. The capture is the easy half. The hard half is the *delivery*: getting tens of gigabytes onto hundreds of disks quickly, without the server melting or the network choking. That delivery step is the entire subject of this post, and it's where multicast (Clonezilla's traditional answer) starts to hurt.

## The problem: multicast doesn't scale the way you'd hope

If you've ever had to re-image a computer classroom, a render farm, or a rack of bare-metal servers, you know the drill: capture one golden image, then push it to every machine at once. The classic answer is **multicast** - send each block once on the wire and let every client pick it up simultaneously.

In theory it's elegant. In practice it has three structural weaknesses:

1. **A global sync barrier.** Every client must be ready before transmission starts. The slowest machine to boot sets the pace for everyone.
2. **Retransmission punishes the whole group.** If one client misses a block, the block is resent to *all* of them. One flaky NIC drags down the entire deployment.
3. **No load sharing.** In the ideal case clients wouldn't *need* to share. One transmission reaches everyone at once, so why would they? But the ideal rarely holds. The moment a client falls behind or misses blocks, there is no mechanism for peers to help it catch up. The source pushes 100% of the data, clients stay pure consumers, and all the recovery burden lands back on the single source.

The result is throughput that is, at best, flat as you add machines - and often *worse*, because the weakest link dominates.

## The idea: treat the disk image as a torrent

BitTorrent was designed for exactly the failure modes that hurt multicast. So EZIO turns a disk image into a torrent and lets the clients swarm:

- **Clients seed as they download.** Every machine that has a piece can serve it to others. Load spreads across the swarm instead of bottlenecking on one source. The more clients you add, the more upload capacity the swarm has.
- **Failures are cheap.** A bad transfer costs a single 16 KB block, re-fetched from any peer that has it - not a retransmission to the whole group.
- **No global barrier.** A machine that boots late just joins the swarm and catches up. It doesn't hold anyone back.

This is the same insight behind Twitter's [murder](https://github.com/lg/murder), Uber's [Kraken](https://github.com/uber/kraken), and Alibaba's Dragonfly - except those distribute *container images*. EZIO does it for **raw disk images**.

## Two ways everyone else moves an image, both with a catch

Solving the delivery problem with BitTorrent is only half the story. You still have to decide *what* you are actually transferring and *how* it lands on the target disk. Most existing tools pick one of two approaches, and both have a real cost.

The first approach treats the whole disk or partition as a single opaque blob and transfers every sector of it. It is dead simple, but you ship the empty space too. A 500 GB partition holding 50 GB of real data still moves 500 GB across the wire. The "don't care" free space dominates the transfer, and you pay for it in time and bandwidth every single deployment.

The second approach ships a disk-image format like qcow2 or vmdk. These are compact, because they only record the blocks that are actually used. The catch is in how they get written: the common path stages the image on the target and expands it back into a raw disk in RAM before flushing it out. That means the image cannot be larger than the target's memory. A 64 GB image needs 64 GB of RAM, which is a non-starter for most deploy nodes.

So you are forced to choose: waste bandwidth shipping empty sectors, or stay under the ceiling of however much RAM the target happens to have. EZIO refuses both.

## What makes EZIO different: it writes to raw disk

This is the part that took the most engineering. EZIO doesn't treat the torrent "files" as files at all. It operates directly on a **raw partition** (e.g. `/dev/sda1`):

- The filename of each torrent "file" is just a **hex disk offset**.
- Data is written with direct `pread()` / `pwrite()` to the partition.
- `disk_offset = piece_id * piece_size + block_offset`.

There is no filesystem layer, no fragmentation, no FIEMAP queries. Blocks within a piece are physically contiguous on disk, which is the whole premise that makes the I/O path fast. EZIO implements a custom [libtorrent disk I/O interface](http://libtorrent.org/reference-Custom_Storage.html#overview) to do this, so it streams data straight to the target disk - no need to buffer the entire image in RAM or stage it in scratch space first. Images of any size deploy without extra free space.

Because it works at the block level, EZIO is filesystem-agnostic: ext2/3/4, xfs, btrfs, f2fs, reiserfs, NTFS, FAT, HFS+, UFS and more. Anything it doesn't recognize falls back to a sector-by-sector `dd`-style copy.

Under the hood there's a lock-free, write-through cache with a **1:1 thread-to-partition mapping** - each worker thread exclusively owns one cache partition, so the hot path needs no locks. Consistent hashing routes every piece to a single thread, which is what lets EZIO drop the temporary `store_buffer` that stock libtorrent relies on. The details are in the [repo's architecture notes](https://github.com/tjjh89017/ezio); the short version is that it's tuned to keep a fast NVMe and a 10 GbE link both busy at once.

## Does it actually work? The numbers

Here's the headline result - EZIO versus Clonezilla multicast, deploying a 50 GB Ubuntu image over a 1 GbE network (Cisco 3560G, Dell T1700 nodes). Lower is better:

| Clients | Unicast (s) | EZIO (s) | Multicast (s) | EZIO / Multicast |
| ---: | ---: | ---: | ---: | ---: |
| 1  | 474   | 675  | 390  | 1.73 |
| 2  | 948   | 1273 | 474  | 2.69 |
| 4  | 1896  | 1331 | 638  | 2.09 |
| 8  | 3792  | 1412 | 980  | 1.44 |
| 16 | 7584  | 1005 | 1454 | 0.69 |
| 24 | 11376 | 1048 | 1992 | 0.53 |
| 32 | 15168 | 1143 | 2203 | 0.52 |

Read the trend, not any single row. With **1-4 clients EZIO is slower** than multicast - the swarm has nobody to share with yet, so you're seeing close to its worst case. But notice what happens to the EZIO column: it stays roughly flat (~1000-1400 s) while multicast climbs steadily. By **16 clients EZIO is ~2x faster**, and at 32 clients it finishes in less than half multicast's time - with the gap still widening. Unicast, for reference, scales linearly into oblivion.

That's the whole point: **EZIO is a scale play.** The more machines you deploy to, the more they seed to each other, and the further it pulls ahead.

On modern hardware the per-node throughput is healthy too. A recent end-to-end run deploying a 60.6 GiB partclone image onto raw NVMe (`/dev/nvme0n1p1`, ADATA SX8200 Pro) across separate LAN hosts:

| Scenario | Cache | Time | Download (median/peak) | Disk-write avg |
| --- | ---: | ---: | ---: | ---: |
| 1-on-1 | 512 MB | ~166 s | ~511 / 526 MB/s | ~374 MiB/s |
| 1-on-1 | 4 GB   | ~116 s | ~836 / 874 MB/s | ~536 MiB/s |
| 1-to-3 | 4 GB   | ~210 s | ~375 / 444 MB/s per leecher | ~294 MiB/s per leecher |

With three leechers, aggregate delivered throughput is ~1.1 GB/s - and the seeder still only reads ~one image from disk per run, because the peers offload it by trading pieces among themselves.

## How you actually use it

The easy path: **Clonezilla Live (>= 2.6.0-31), Lite Server Mode.** EZIO is built in and driven through Clonezilla's menus - no compilation, no manual torrent juggling. That's how most people should run it. Steven Shiau at Taiwan's NCHC also maintains a [BT-from-disk tutorial](https://clonezilla.org/fine-print-live-doc.php?path=./clonezilla-live/doc/12_lite_server_BT_from_dev/).

If you want to drive it directly, the workflow is: capture an image with partclone, build a torrent, seed on the source, download on each target:

```bash
# Capture a partition into a partclone image + torrent.info, then build a torrent
utils/partclone_create_torrent.py -i image_dir/torrent.info -o sda1.torrent ...

# On the source: seed
utils/ezio_add_torrent.py -S sda1.torrent /dev/sda1

# On each target: download + write straight to disk
utils/ezio_add_torrent.py sda1.torrent /dev/sda1
```

The `ezio` daemon talks over gRPC and runs in the foreground, so you start it in one terminal and manage torrents from another. For WAN or bottlenecked links you can proxy the swarm through a normal BitTorrent client like qBittorrent.

## Where this could go next: OpenStack Ironic

The use case that keeps nagging at me is bare-metal provisioning at scale. OpenStack Ironic and Metal3 both deploy disk images to many nodes - and their image-distribution story today is essentially *HTTP plus a caching proxy* (Squid / Apache Traffic Server). A caching proxy is a tree of caches; the nodes still don't help each other. The [official Ironic spec for proxy support](https://specs.openstack.org/openstack/ironic-specs/specs/approved/agent-image-proxy.html) acknowledges the bandwidth problem but stops at proxies.

This isn't a new observation. Mirantis published *[Cut Ironic Provisioning Time Using Torrents](https://web.archive.org/web/20211124125644/https://www.mirantis.com/blog/cut-ironic-provisioning-time-using-torrents/)* back in 2016 - the original post is offline now; that link is the Web Archive copy. They proved the idea worked, but it never made it upstream, and the knowledge quietly evaporated. Meanwhile CERN has documented conductor going OOM under parallel deployments, and the container world (Kraken, Dragonfly) long ago settled on P2P as the answer for distributing large blobs at scale.

So there's a decade-old gap sitting open: **P2P distribution of bare-metal disk images.** EZIO already does the hard part - raw-disk, filesystem-agnostic, streaming I/O. If you work on Ironic, Metal3, or any large-scale bare-metal fleet and this resonates, I'd genuinely love to talk. The integration seam (an IPA deploy step / image source) is small, and I think this is worth finishing.

## Try it

- Source & docs: <https://github.com/tjjh89017/ezio>
- Issues / ideas: <https://github.com/tjjh89017/ezio/issues>

If you've ever cursed at a multicast deployment that stalled on one bad machine, give it a spin - and tell me how it scales on hardware bigger than my lab.

Special thanks to the National Center for High-performance Computing (NCHC), Taiwan, for test hardware and support.
