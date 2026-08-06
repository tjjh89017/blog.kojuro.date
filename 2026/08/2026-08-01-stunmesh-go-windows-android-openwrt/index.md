# STUNMESH-go Now Runs on Windows, on Android, and on an 8 Year Old MIPS Router


WireGuard is simple and fast because it does one job and stops there. Finding the other peer is not part of that job, and this is on purpose. WireGuard expects you to tell it where the peer is, and leaves the how to you. If both sides sit behind NAT, and neither has a fixed public address, nobody can answer that question, and the tunnel never starts. The usual answers are a VPS in the middle, a port forward on your router, or a service like Tailscale or NetBird that runs a coordination server for you.

[STUNMESH-go](https://github.com/tjjh89017/stunmesh-go) takes a different path. It finds the public address of each node with STUN, encrypts that address, and puts it somewhere both sides can read. You run no server at all. This post explains how it works, and shows three new places it now runs: an 8 year old MIPS access point, Windows, and Android. The code is on [GitHub](https://github.com/tjjh89017/stunmesh-go), and the full documentation is at [docs.stunmesh.dev](https://docs.stunmesh.dev).

<!--more-->

## The problem

A WireGuard peer needs an `Endpoint`, which is just an IP address and a UDP port. On a laptop at home, or on a router behind carrier NAT, that address is not yours. Your NAT gives you a temporary mapping, it can change at any time, and you cannot read it from inside the network.

So people solve it in one of these ways:

- **Port forwarding**: you need control of the router, and a public IP. Many people have neither, because of CGNAT.
- **A VPS in the middle**: it always works, but you pay for it, you maintain it, and all traffic goes through one machine.
- **A coordination server**: Tailscale, NetBird, and headscale do this well. Something still has to run, and it has to know about every node.

All three need infrastructure. The question I wanted to answer is: do you need any of it?

## How STUNMESH-go works

The idea is old and simple. It is the same hole punching that VoIP and games have used for years, applied to WireGuard.

1. **Discover.** STUNMESH sends a STUN request from the *same* UDP port that WireGuard listens on. It uses a raw socket with a BPF filter, so WireGuard keeps the socket and never notices. The STUN server replies with the public IP and port of your NAT mapping.
2. **Publish.** STUNMESH encrypts that endpoint with a Curve25519 sealed box, so only your peer can read it. Then it stores the result under a key made from a SHA-1 digest of the two public keys.
3. **Establish.** It reads the peer's record from the same place, decrypts it, and sets it as the peer endpoint on the WireGuard interface.
4. **Repeat.** It refreshes on a timer, so the tunnel survives a NAT rebinding. It can also refresh when a ping to the peer stops working.

Both sides do this at the same time. After about two refresh intervals, the packets meet in the middle and the handshake completes.

The interesting part is step 2. STUNMESH does not care *where* the endpoint is stored. It only needs a place that both nodes can write to and read from. That place is a plugin.

## The storage is something you already have

This is the design decision that removes the server. Instead of building a rendezvous service, STUNMESH borrows storage that already exists in your life.

- **Cloudflare DNS**: the endpoint goes into a TXT record under a domain you own. If you have a domain, you already have this.
- **OpenDHT**: a public distributed hash table. No account, no token, no quota, because nobody operates it. Savoir-faire Linux runs public proxies for [Jami](https://jami.net/), and you can point STUNMESH at them, or run your own.
- **Your own script**: an `exec` or `shell` plugin gets the encrypted blob on stdin and stores it however you like. A git repo, an S3 bucket, a Redis instance, a pastebin. The protocol is a few lines of text, so a shell script is enough.

The data in storage is a sealed box. The storage operator sees ciphertext and a hash, and learns nothing useful about your mesh.

A minimal two node config looks like this:

```yaml
---
refresh_interval: "1m"
log:
  level: "info"
interfaces:
  wg0:
    peers:
      "PEER_B":
        public_key: "<PEER_B_PUBLIC_KEY_BASE64>"
        plugin: dht
stun:
  addresses: ["stun.l.google.com:19302"]
plugins:
  dht:
    type: builtin
    name: opendht
    endpoints:
      - https://dhtproxy2.jami.net
      - https://dhtproxy3.jami.net
```

Run the same thing on the other node with this node's public key, wait about two minutes, and `wg show` reports a handshake.

## New: an 8 year old MIPS access point

I wanted to know how small a device can be and still join the mesh. So I took an IgniteNet Spark SP-W2M-AC1200, an access point from around 2017:

| | |
|---|---|
| SoC | Realtek RTL8197F, MIPS 24Kc, little endian |
| RAM | 128 MB |
| Firmware | IgniteNet HeliOS 2.3.5, a vendor build based on OpenWrt |
| Kernel | Linux 3.18.29 |

That kernel is far too old to have the WireGuard module, and the vendor firmware has no package manager I want to trust. So the whole stack runs in userspace, as static Go binaries:

- **wireguard-go**, because there is no kernel WireGuard here
- **stunmesh-go**, which does the STUN discovery and sets the peer endpoints
- **a small `wg` like CLI**, to load keys and check handshakes

Cross compiling for this target is one command, with no toolchain to install:

```bash
GOOS=linux GOARCH=mipsle GOMIPS=softfloat CGO_ENABLED=0 go build
```

The result, built with `-s -w -trimpath`:

| Binary | Size |
| --- | ---: |
| wireguard-go | 3.6 MB |
| stunmesh-go | 8.4 MB |
| wg CLI | 3.3 MB |
| **total** | **about 15 MB** |

At runtime the whole stack uses about 20 MB of RAM, on a device with 128 MB. It works. STUN finds the public endpoint, the peers exchange it through the OpenDHT plugin, and the tunnel comes up. No relay, no VPS.

You do not even need free flash for this. You can copy the binaries into `/tmp`, which is tmpfs, and run them from RAM. Nothing is written to the device, and a reboot cleans it up.

If your router is newer, this gets much smaller. Modern OpenWrt ships the WireGuard kernel module and `wg`, so you can drop wireguard-go and the CLI, and add one 8 MB binary that talks to the kernel directly. Kernel WireGuard is also faster than any userspace version.

The point is not this specific access point. The point is that a device from 2017, behind NAT, with no public IP and no port forward, can still be a full mesh node. That is a lot of hardware that does not have to become e-waste.

## New: Windows

Windows needed a different design. On Linux STUNMESH shares WireGuard's UDP port with a raw socket, and on macOS and BSD it uses pcap. Windows has neither, so nothing can look at that socket from the outside.

The answer is to turn the problem around. On Windows, STUNMESH owns the public facing socket itself, and runs a small local UDP proxy:

- STUNMESH opens the outer socket and does STUN discovery on it.
- WireGuard packets are relayed to a loopback listener, one per peer.
- The official [WireGuard for Windows](https://www.wireguard.com/install/) client stays the data plane, and STUNMESH rewrites each peer endpoint to its loopback listener.

You do not configure any of this. The config file is the same as on every other platform, and the proxy needs no settings of its own. There are only three rules:

1. Install the official WireGuard for Windows client and create the tunnel.
2. Activate the tunnel **before** starting STUNMESH.
3. Run STUNMESH from an **Administrator** console, because the WireGuard service needs the same privilege.

Two things to know. If you deactivate and reactivate the tunnel in the WireGuard UI, the service re applies the `.conf` file and wipes the endpoints STUNMESH set, so restart STUNMESH after any toggle. And ping monitoring is not implemented on Windows yet, so STUNMESH logs that and continues without it.

Tested on Windows 11 25H2 with WireGuard for Windows 1.1. Releases ship as `stunmesh-windows-<arch>-<tag>.zip` for amd64 and arm64. The details are in the [Windows guide](https://docs.stunmesh.dev/guides/windows).

The same proxy is available on Linux, macOS, and FreeBSD with `proxy.enabled: true`. It is needed for full tunnel setups on macOS and FreeBSD, and it is useful on Linux when the process cannot get `CAP_NET_RAW`.

## New: Android

The other new platform is [stunmesh-android](https://github.com/tjjh89017/stunmesh-android). Android does not give apps raw sockets, and it does not let two apps own the same UDP port, so the desktop design does not fit. The app solves it by putting everything inside one process:

- The app is a normal VPN app built on Android's `VpnService`, so **no root is needed**.
- The data plane is **wireguard-go embedded in the Go core**, built from the stunmesh-go repository with `gomobile bind` and shipped as an AAR.
- STUNMESH owns the outer UDP socket inside a custom `conn.Bind`. STUN discovery, hole punching, and WireGuard traffic share one socket, and a demux on the receive path decides which packet is which.
- Peer endpoints are applied at run time over WireGuard's UAPI, so **the tunnel never restarts** when an endpoint changes.
- When the network changes, the app hands a fresh tun fd to the running core, and the WireGuard device survives without a restart.

It works on a real device, and I have tested it against the other platforms. It is still early software, so the honest limits are:

- Android allows one active VPN app at a time. STUNMESH cannot run next to another VPN app.
- Only the built in storage plugins are available. The `exec` and `shell` plugins need external processes, so they stay on desktop.
- Doze can delay keepalives in deep sleep. Exempt the app from battery optimization if you want a long lived tunnel.

The source is at [github.com/tjjh89017/stunmesh-android](https://github.com/tjjh89017/stunmesh-android). There is no Play Store or F-Droid listing yet, so the only way to install it today is to sideload the APK from the [releases page](https://github.com/tjjh89017/stunmesh-android/releases). One universal APK covers arm64-v8a, armeabi-v7a, x86, and x86_64. Sideloading needs "install unknown apps" allowed for whatever opens the file, your browser or your file manager.

## What it does not do

That is the good part. Here are the limits, because I would rather tell you than have you find them yourself.

- **Symmetric NAT is hard.** If your NAT gives a new port for every destination, the port that STUN reports is not the port your peer will hit. Full cone, restricted cone, and port restricted cone all work. For a mixed pair, one side behind a cone NAT is usually enough.

  This is not a STUNMESH problem. It is hard for every hole punching solution, because there is nothing to punch when the mapping only exists after you pick a destination. The usual answer is to give up on the direct path and relay the traffic instead. Tailscale does exactly this with its DERP servers, and it is a good answer, but it needs servers that somebody runs.
- **There is no relay.** So when hole punching fails, the tunnel does not come up. STUNMESH has no DERP style fallback on purpose: a fallback needs servers, and having no servers is the whole point of the project. If you need the tunnel to work from anywhere, no matter what the NAT does, use a tool that has a relay.
- **You still exchange public keys yourself.** STUNMESH moves endpoints, not keys. Key distribution, ACLs, and MagicDNS style features are out of scope.
- **It reads the WireGuard device once at startup.** If you recreate the interface or change the listen port, restart STUNMESH. Under systemd, bind it to the WireGuard unit.

So this is not a Tailscale or NetBird replacement for most people. Both have much better UX, and I recommend them if you want things to just work. They focus on that UX, so their agents are bigger, and they need a coordination server behind them. STUNMESH sits at the other end: one static binary, no account, and nothing running anywhere else. It is for the case where you do not want any coordination server to exist at all, or where the device is too small to run a full agent.

## When the big tools are not an option

That last paragraph sounds like a matter of taste, and often it is. But sometimes the choice is made for you.

The common reason is simple. Somebody already has WireGuard running exactly how they want it, and they do not want a second system wrapped around it. Tailscale and NetBird put many things in one box: a control plane, an admin panel, DNS, access rules, single sign on. That box is why they work so well for most people, and it is also the price. You get less room to do things your own way. If the only missing piece is "these two boxes cannot find each other", a whole platform is a large answer to a small question.

The other reason is address space. Tailscale gives each node an address in `100.64.0.0/10`, the CGNAT range from RFC 6598, and that range is popular. Cloudflare reserves parts of it as well, and WARP leaves `100.64.0.0/10` out of its tunnel by default, so the two do not sit together nicely. On macOS this has been an [open issue](https://github.com/tailscale/tailscale/issues/5631) for a long time, and the usual workaround is to exclude the range on one side so the other keeps working.

On Linux there is a second effect. Tailscale installs a firewall rule that drops any packet with a source in `100.64.0.0/10` that does not arrive on the `tailscale0` interface:

```
iptables -A ts-input --source 100.64.0.0/10 ! -i tailscale0 -j DROP
```

The reason is sound. It stops another machine on your LAN from spoofing a Tailscale address. But it also blocks anything else you happen to run in that range, which people [do hit in practice](https://github.com/tailscale/tailscale/issues/12555). You can change it with the [netfilter mode](https://tailscale.com/docs/reference/netfilter-modes), but you have to know it is there first.

STUNMESH does none of this. It picks no addresses, adds no firewall rules, and creates no interface. It writes an endpoint into a WireGuard peer that you configured yourself, and then it stops. Your address plan stays yours. That is a smaller promise, and for some people it is the only one that fits.

## The talk

I presented this design at FOSDEM 2026, in a talk called *STUNMESH-go: Building P2P WireGuard Mesh Without Self-Hosted Infrastructure*. If you prefer a talk to a blog post, it covers the same ground in more depth.

- Event page: <https://fosdem.org/2026/schedule/event/YQWEDC-stunmesh-go_building_p2p_wireguard_mesh_without_self-hosted_infrastructure/>
- Recording: <https://video.fosdem.org/2026/h1302/YQWEDC-stunmesh-go_building_p2p_wireguard_mesh_without_self-hosted_infrastructure.av1.webm>
- Slides: <https://speakerdeck.com/tjjh89017/fosdem-2026-stunmesh-go-building-p2p-wireguard-mesh-without-self-hosted-infrastructure>

## Try it

- Source: <https://github.com/tjjh89017/stunmesh-go>
- Docs: <https://docs.stunmesh.dev>
- Android app: <https://github.com/tjjh89017/stunmesh-android>

Binaries are on the releases page for Linux (amd64, arm, arm64, mipsle), macOS, FreeBSD, and Windows, and there is a container image at `ghcr.io/tjjh89017/stunmesh`.

One static binary per platform is what makes this possible, from a server, to a Windows laptop, to a MIPS access point, to a phone. If you try it on hardware I have not tested, or on a NAT that behaves strangely, I want to hear about it.

