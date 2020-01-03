---
layout: single
classes: wide
title: "Wireguard with Docker on Raspberry Pi 4"
description: "Notes on using Wireguard with Docker on a Raspberry Pi 4"
categories:
  - Projects
---

In a [previous post]({% post_url 2019-10-01-Example-Ansible.md %}) I discussed setting up docker containers that used OpenVPN. Since they I have updated my setup to use [Wireguard](https://www.wireguard.com/) instead and I'm documenting the resources I found useful in case anyone wants to do the same thing. Two main reasons drove me to move from OpenVPN to Wireguard:
