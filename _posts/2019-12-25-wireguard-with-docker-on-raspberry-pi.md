---
layout: single
classes: wide
title: "Wireguard with Docker on Raspberry Pi 4"
description: "Notes on using Wireguard with Docker on a Raspberry Pi 4"
categories:
  - Projects
---

In a [previous post](/projects/2019/10/01/Example-Ansible.md.html) I discussed setting up docker containers that used OpenVPN. Since they I have updated my setup to use [Wireguard](https://www.wireguard.com/) instead and I'm documenting the resources I found useful in case anyone wants to do the same thing. Two main reasons drove me to move from OpenVPN to Wireguard:

The first reason is that my old provider Private Internet Access was [bought out by Kape](https://torrentfreak.com/private-internet-access-to-be-acquired-by-kape/). There haven't been any changes to PIA yet but the new owner is a bit worrying considering their track record. So I have changed to [Mullvad](https://mullvad.net/en/) as my new provider and they have very good Wireguard support.

The second reason is that Wireguard offers some compelling improvements over OpenVPN:
 - OpenVPN is very resource demanding on the RPi. The Model 4 Raspberry Pi has a new I/O architecture which can properly use Gigabit LAN. Unfortunately the CPU overhead for OpenVPN was high enough that it limited throughput. My Pi would regularly have a core pinned near 100% utilization from OpenVpn. In comparison [Wireguard's performance](https://www.wireguard.com/performance/#performance-roadmap) is considerably lighter leading to better throughput.
- OpenVPN runs in user space where as Wireguard will be included in the kernel [eventually](https://www.phoronix.com/scan.php?page=news_item&px=WireGuard-Net-Next-Lands). For now this complicates things a bit since it has to be installed separately but once the changes land it will make setup much easier.

# Installing Wireguard on Debian

For now we need to install the Wireguard kernel modules and user space tools.

The [Debian wiki](https://wiki.debian.org/Wireguard#Installation) generally outlines the process but leaves some details to the reader in terms of file permissions. [Linode's](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-debian/) covered those in better details. I only care about the client configuration details since I am not running a server. Additionally instead of writing my own configuration file I used the [Mullvad Wireguard configuration tools](https://mullvad.net/en/blog/2018/5/14/wireguard-configuration-tool-has-new-function-download-all/) to generate one.

I also needed to install the following to follow the setup:

{% highlight bash %}
sudo apt install openresolv resolvconf
{% highlight %}

# Using Wireguard with Docker
There's a blog post that describes two solutions for using Wireguard with docker. The first involves creating a docker container that runs the Wireguard client. I went with the [second approach](https://nbsoftsolutions.com/blog/routing-select-docker-containers-through-wireguard-vpn#solution-2) instead which covers setting up a Wireguard interface on the host and using in a Docker network.

# Raspberry Pi Specific Changes for Installation
Finally I want to do this on a Rapsberry Pi 4 which requires [additional steps](https://github.com/adrianmihalko/raspberrypiwireguard) to install raspberrypi-kernel-headers and get Wireguard to run. 
