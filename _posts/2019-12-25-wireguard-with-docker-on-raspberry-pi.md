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

In terms of researching how to set this up three articles were particularly useful:

## Installing Wireguard on Debian

First I wanted general information about installing the Wireguard kernel modules and user space tools.

The [Debian wiki](https://wiki.debian.org/Wireguard#Installation) generally outlines the process but leaves some details to the reader in terms of file permissions. [Linode's](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-debian/) covered those in better details. I only care about the client configuration details since I am not running a server. Additionally instead of writing my own configuration file I used the [Mullvad Wireguard configuration tools](https://mullvad.net/en/blog/2018/5/14/wireguard-configuration-tool-has-new-function-download-all/) to generate one.

I also needed to install the following to follow the setup:

{% highlight bash %}
sudo apt install openresolv resolvconf
{% endhighlight %}

## Using Wireguard with Docker
There's a blog post that describes two solutions for using Wireguard with docker. The first involves creating a docker container that runs the Wireguard client. I went with the [second approach](https://nbsoftsolutions.com/blog/routing-select-docker-containers-through-wireguard-vpn#solution-2) instead which covers setting up a Wireguard interface on the host and using in a Docker network.

## Raspberry Pi Specific Changes for Installation
Finally I want to do this on a Rapsberry Pi 4 which requires [additional steps](https://github.com/adrianmihalko/raspberrypiwireguard) to install raspberrypi-kernel-headers and get Wireguard to run. 

# Putting It All Together With Ansible

## Handling the Mullvad config file
The playbook assumes there is a file called "mullvad.conf" in the same directory as playbook which is the configuration file downloaded from the Mullvad configuration generation tool and the vars section uses the [ini plugin for lookups ](https://docs.ansible.com/ansible/latest/plugins/lookup/ini.html) to read the server Address and DNS information from config file:

{% highlight yml %}
{% raw %}
---

  - name: Wireguard
    connection: ssh
    become_user: root
    become: yes
    hosts: rpi
    vars:
      address: "{{ lookup('ini', 'Address section=Interface file=mullvad.conf').split(',')[0] }}"
      dns: "{{ lookup('ini', 'DNS section=Interface file=mullvad.conf') }}"
    tasks:
      - name: Print Mullavd Address
        debug:
          msg: "Address is: {{ address }}"
      - name: Print Mullvad DNS
        debug:
          msg: "DNS is: {{ dns }}"
{% endraw %}
{% endhighlight yml %}

Note the we have to split the address on "," since the INI entry contains both the ipv4 and ipv6 address and we only want the first half

## Installing Wireguard
The installation process is basically a combination of the two posts on installing, but replacing command with ansible tasks to clean it up:

{% highlight yml %}
{% raw %}
      - name: enable unstable
        lineinfile:
          path: /etc/apt/sources.list.d/unstable-wireguard.list
          create: yes
          line: deb http://deb.debian.org/debian/ unstable main
      - name: enable wireguard repo
        blockinfile:
          path: /etc/apt/preferences.d/limit-unstable
          create: yes
          block: |
            Package: *
            Pin: release a=unstable
            Pin-Priority: 150
      - name: Add first key
        apt_key:
          keyserver: keyserver.ubuntu.com
          id: 8B48AD6246925553
      - name: Add second key
        apt_key:
          keyserver: keyserver.ubuntu.com
          id: 7638D0442B90D010
      - name: Add third key
        apt_key:
          keyserver: keyserver.ubuntu.com
          id: 04EE7237B7D453EC
      - name: Install deps
        apt:
          pkg:
            - raspberrypi-kernel-headers
            - dirmngr
            - wireguard-dkms
            - wireguard-tools
            - resolvconf
          update_cache: yes
{% endraw %}
{% endhighlight yml %}

## Configuring the Wireguard Interface
Finally we want to set up the wireguard interface that docker will use. We do this by first copying the Mullvad config over to the machine. Then as noted by the Wireguard on Docker article we remove the "Address" and "DNS" options from the file since we have to manually configure the interface instead of using the wg-quick command. With that done all thats left is to setup up the interface and configure it:

{% highlight yml %}
{% raw %}
      - name: Copy config to server
        synchronize:
          src: mullvad.conf
          dest: /etc/wireguard/wg1.conf
      - name: remove Address entry
        ini_file:
          path: /etc/wireguard/wg1.conf
          section: Interface
          option: Address
          state: absent
      - name: remove DNS entry
        ini_file:
          path: /etc/wireguard/wg1.conf
          section: Interface
          option: DNS
          state: absent
      - name: add wireguard interface
        command: "ip link add dev wg1 type wireguard"
      - name: set wireguard config
        command: "wg setconf wg1 /etc/wireguard/wg1.conf"
      - name: set wireguard address
        command: "ip address add {{ address }} dev wg1"
      - name: put interface up
        command: "ip link set up dev wg1"
      - name: set nameserver
        command: "printf 'nameserver %s\n' '{{ dns }}' | resolvconf -a tun.wg1 -m 0 -x"
      - name: handle reverse path filtering
        command: "sysctl -w net.ipv4.conf.all.rp_filter=2"
{% endraw %}
{% endhighlight yml %}
