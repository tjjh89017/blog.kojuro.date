---
title: "About Date Huang"
date: 2026-03-24
draft: false
description: "About Date Huang - Cloud & Network Solution Architect"

lightgallery: true
---

## Hi, I'm Date Huang

I am a Cloud and Network Solution Architect based in Taiwan. I design and build cloud infrastructure, data center networks, and hybrid cloud solutions. I have 7+ years of experience with AWS, Azure, GCP, OpenStack, and Kubernetes. I also maintain several open-source projects and speak at international conferences. If you want to work with me or talk about a project, please check my [resume](./resume_20260313_rev1.pdf), find me on [GitHub](https://github.com/tjjh89017) or [LinkedIn](https://www.linkedin.com/in/tjjh89017/), or send me an email at tjjh89017 [AT] hotmail.com.

## What I Do Now

I currently work as a Cloud and Network Solution Architect. I design BGP EVPN overlay networks on AWS for high-frequency financial trading. I connect AWS EKS clusters with on-premises data centers using VyOS routers and BGP EVPN. I also introduced Cloudflare Zero Trust to replace traditional VPN for better security.

I am also a contributor to the VyOS open-source project, where I work on the Kubernetes operator, ARM64 platform porting, and WireGuard mesh networking.

## Open Source Projects

### Projects I Maintain

- **[vRouter-Operator](https://github.com/tjjh89017/vrouter-operator)** - A Kubernetes Operator written in Go for VyOS router configuration management. It uses a CRD-based architecture with VRouterTemplate, VRouterTarget, VRouterBinding, and VRouterConfig resources. It delivers configuration through QEMU Guest Agent over VirtIO channel, so it does not need SSH or network access to the router. It supports KubeVirt and Proxmox VE as backends, and includes validating webhooks and Helm chart packaging. Released v1.0.0 in March 2026.
- **[STUNMESH-go](https://github.com/tjjh89017/stunmesh-go)** - A tool that builds peer-to-peer WireGuard mesh VPN without self-hosted infrastructure. It uses STUN with raw sockets and cBPF to discover NAT endpoints. It supports pluggable backends including Cloudflare DNS and shell scripts. It works on Linux (amd64, arm, arm64, mipsle), FreeBSD, macOS, and embedded routers like VyOS, EdgeOS, and OpenWrt. Presented at FOSDEM 2026 and AsiaBSDCon 2026.
- **[EZIO](https://github.com/tjjh89017/ezio)** - A BitTorrent-based bare-metal OS deployment tool developed with support from the National Center for High-Performance Computing (NCHC), Taiwan. It uses peer-to-peer data distribution so the deployment time stays nearly constant no matter how many machines you deploy. It streams data directly to the disk without buffering, and supports many file systems (ext2/3/4, NTFS, FAT, Btrfs, XFS, and more). It was integrated into Clonezilla as Lite Server mode since version 2.6.0-31. Written in C++ using libtorrent-rasterbar. Published in MDPI Applied Sciences (2019) and IEEE Access (2021).
- **[VyOS on DPU](https://github.com/tjjh89017/vyos-build-arm64)** - Ported VyOS to NVIDIA BlueField-2 DPU and ARM64 platforms. Built custom ARM64 images with kernel patches. Tested on Ampere eMAG, ARM64 VM, and Raspberry Pi 4.
- **[kvnet](https://github.com/tjjh89017/kvnet)** - A VXLAN overlay network controller for Kubernetes and KubeVirt. It provides OpenStack Neutron-like L2/L3 overlay networking with VXLAN and VPC for multi-tenant network isolation. Built entirely with native Linux network components.
- **DozenCloud Project** - One of the earliest independent efforts to build an OpenStack cloud on ARM64 hardware. Started in 2015 to provide ARM virtual environments for university students using affordable ARM boards (Raspberry Pi, Banana Pi M2). Later ported to ARMv8 64-bit on GIGABYTE R150-T60 servers. Presented at SITCON 2016, OpenStack Day Taiwan 2016, and Open Source Summit North America 2017.

### Projects I Contribute To

- **[VyOS](https://vyos.io/)** - Open-source network operating system (router and firewall)
- **[Harvester HCI](https://harvesterhci.io/)** - Kubernetes-based hyper-converged infrastructure by SUSE
- **[Clonezilla](https://clonezilla.org/)** - Partition and disk imaging/cloning program
- **[Partclone](https://github.com/Thomas-Tsai/partclone)** - Smart partition backup utility
- **[fish-shell](https://github.com/fish-shell/fish-shell)** - User-friendly command line shell
- **[flash-kernel](https://salsa.debian.org/installer-team/flash-kernel)** - Kernel and initramfs installer for embedded devices
- **[Cockpit](https://cockpit-project.org/)** - Web-based server management interface

## Conference Talks

I have given 20+ talks at international and regional conferences:

| Year | Conference | Topic |
|------|-----------|-------|
| 2026 | AsiaBSDCon 2026 (Tokyo) | Porting STUNMESH-go to FreeBSD and macOS: Building Peer-to-Peer WireGuard Networks |
| 2026 | FOSDEM 2026 (Brussels) | STUNMESH-go: Building P2P WireGuard Mesh Without Self-Hosted Infrastructure |
| 2025 | COSCUP 2025 | STUNMESH-go: WireGuard NAT Traversal Tool - Origin and Introduction |
| 2024 | VMUG Australia Meetup | How to Configure and Maintain BGP-EVPN with VyOS in VMware vSphere Solutions |
| 2024 | ALASCA Tech-Talk #19 | Provisioning and Managing your VyOS NFV with Kubernetes |
| 2024 | COSCUP 2024 | Rapidly Deploy NFV with VyOS on Kubernetes |
| 2024 | COSCUP 2024 | A Decade of Open Source Projects: From University to Industry |
| 2024 | COSCUP 2024 | The Business Challenges of Open Source Projects |
| 2024 | OSC Nagoya 2024 (Japan) | Enterprise Networking with VyOS, an Open Source Router |
| 2023 | COSCUP 2023 | Designing Kubernetes Controllers and CRDs in Practice - Using Networking as an Example |
| 2021 | COSCUP 2021 | Cloud Infrastructure Interconnect with WireGuard and OSPF |
| 2019 | TWNOG 4.0 | Next Generation IDC Network Solution Overview |
| 2019 | OSC Tokyo Fall 2019 (Japan) | Build Redundant Gaming Network with WireGuard and BGP |
| 2019 | China Open Source Conference (COSCon '19) | Build Redundant Gaming Network with WireGuard and BGP |
| 2019 | OpenInfra Day Taiwan 2019 | Massive Bare-Metal Operating System Provisioning Improvement |
| 2019 | Hong Kong Open Source Conference 2019 | Decentralized Bare-Metal Operating System Provisioning |
| 2018 | ISC High Performance 2018 (Frankfurt) | The Design and Implementation of Bare Metal Cluster Deployment Using BitTorrent (Poster) |
| 2017 | Open Source Summit North America 2017 | Build Cloud Infrastructure with Cheap ARM Boards (co-speaker) |
| 2017 | OpenStack Day Taiwan 2017 | Combine Continuous Integration (CI) with OpenStack |
| 2016 | OpenStack Day Taiwan 2016 (Invited) | OpenStack on ARM64 |
| 2016 | SITCON 2016 | ARM Cloud Project |

## Publications and Patent

- **US Patent US11146631B1** (Oct 2021): Method for Deploying Computer Operating Environment and Deployment System (Assignee: National Applied Research Laboratories)
- **IEEE Access** (Feb 2021): A BitTorrent Mechanism-Based Solution for Massive System Deployment
- **MDPI Applied Sciences** (Jan 2019): [A Novel Massive Deployment Solution Based on the Peer-to-Peer Protocol](https://www.mdpi.com/2076-3417/9/2/296)
- **Master Thesis** (Dec 2017): Decentralized Image Deployment in Datacenter

## Work Experience

- **Nov 2025 - Now | Solution Architect, Senior Network Engineer - Stranity**
  Design and build EVPN overlay networks on AWS for high-frequency financial trading. Connect AWS EKS Kubernetes clusters with on-premises data centers using VyOS routers and BGP EVPN. Build the entire AWS network infrastructure with 7 Terraform projects. Introduce Cloudflare Zero Trust to replace traditional VPN.

- **Dec 2024 - Now | Contributor - VyOS Networks**
  Contribute to VyOS open-source ecosystem including the vRouter-Operator Kubernetes operator, ARM64 and DPU platform porting, and STUNMESH-go WireGuard mesh networking.

- **Sep 2023 - Dec 2024 | Solution Architect - VyOS Networks**
  Develop solution architecture for data center and cloud networking customers. Design hardware validation plans, implement VyOS on Kubernetes, support hardware partners, and create cloud VPN connection examples for AWS/Azure/GCP. Demo BGP-EVPN with VMware vCenter at VMUG Australia.

- **Jan 2022 - Jan 2023 | Software Engineer - SUSE Rancher**
  Contribute to Harvester HCI, an open-source Kubernetes-based hyper-converged infrastructure. Design VLAN and VXLAN network mechanisms, storage network isolation, and provide internal network training from basic to advanced level. Support European cloud providers with datacenter switch configuration.

- **May 2019 - Dec 2021 | Senior Presales Engineer - Edgecore Networks**
  Develop solution architecture for data center networking with Cumulus Linux and SONiC. Design and configure BGP-EVPN networks for customers across US, Australia, Korea, and Taiwan, including storage vendors, telecom operators, government HPC, and automobile manufacturers.

## Education

- **National Chiao Tung University** - Master's Degree, Institute of Computer Science and Engineering, Software Quality Lab (2016-2018)
- **National Central University** - Bachelor's Degree, Department of Computer Science and Information Engineering (2012-2016)

## Get in Touch

You can find me on [GitHub](https://github.com/tjjh89017) and [LinkedIn](https://www.linkedin.com/in/tjjh89017/). Feel free to reach out if you need help with cloud networking, hybrid cloud architecture, Kubernetes networking, or open-source collaboration.
