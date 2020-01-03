---
layout: single
classes: wide
title: "Wireguard with Docker on Raspberry Pi 4"
description: "Notes on using Wireguard with Docker on a Raspberry Pi 4"
categories:
  - Projects
---


In a [previous post]({% post_url _posts/2019-10-01-Example-Ansible %}) I discussed setting up docker containers that used OpenVPN. Since they I have updated my setup to use [Wireguard](https://www.wireguard.com/) instead and I'm documenting the resources I found useful in case anyone wants to do the same thing. Two main reasons drove me to move from OpenVPN to Wireguard:

The first reason is that my old provider Private Internet Access was [bought out by Kape](https://torrentfreak.com/private-internet-access-to-be-acquired-by-kape/). There haven't been any changes to PIA yet but the new owner is a bit worrying considering their track record. So I have changed to [Mullvad](https://mullvad.net/en/) as my new provider and they have very good Wireguard support.

The second reason is that Wireguard offers some compelling improvements over OpenVPN:
 - OpenVPN is very resource demanding on the RPi. The Model 4 Raspberry Pi has a new I/O architecture which can properly use Gigabit LAN. Unfortunately the CPU overhead for OpenVPN was high enough that it limited throughput. My Pi would regularly have a core pinned near 100% utilization from OpenVpn. In comparison [Wireguard's performance](https://www.wireguard.com/performance/#performance-roadmap) is considerably lighter leading to better throughput.
- OpenVPN runs in user space where as Wireguard will be included in the kernel [eventually](https://www.phoronix.com/scan.php?page=news_item&px=WireGuard-Net-Next-Lands). For now this complicates things a bit since it has to be installed separately but once the changes land it will make setup much easier.
