---
layout: single
classes: wide
title: "Custom Arch ISO with ZFS and Ansible"
description: "Creating a customized Arch ISO with additional packages."
categories:
  - Projects
---
I'm picking up a X1 Carbon that I'm going to load [Arch Linux](https://www.archlinux.org/) onto. While I've been waiting I've been putting together some Ansible playbooks to help automate the install process.

I'm planning to have a ZFS root partition for my install but due to the [mechanics of the ZFS license](https://marc.info/?l=linux-kernel&m=154714516832389&w=2) ZFS cannot be included in the Linux Kernel. This means you need to go out of your way to add ZFS support. 

In my case I would like bundle the support into the Arch Live CD ISO itself. While I'm at it I'm throwing in Ansible so I can run playbooks locally instead of having to run them over SSH.

The Arch Wiki gives us the basics we need:
 * Follow the [Archiso](https://wiki.archlinux.org/index.php/Archiso) for how to make a custom ISO.
   * We want to use "releng" profile since we want all the packages normally included in the ISO.
   * Add ansible to packages.x86_64
   * This requires an existing Arch install. If you do not have one you can get by building it in a virtual machine.
 * The ZFS page on the wiki has a section explaining what repository to enable and which package to add.
  * The big complication with ZFS not being included in the kernel is that there will be delays between when there is a kernel update and when ZFS on Linux updates which can lead to breaks. I had to work around one of these breaks myself when I first did this. I figured out I was not the only one with the problem since there was a [reddit thread](https://old.reddit.com/r/archlinux/comments/e06kzz/cannot_update_kernel_and_packages_because_of/). To fix the problem I had to add the [archzfs-kernels repository](https://wiki.archlinux.org/index.php/Unofficial_user_repositories#archzfs-kernels) as well.


{% highlight shell %}
sudo ./build.sh -v
{% endhighlight %}
