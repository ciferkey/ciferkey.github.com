---
layout: single
classes: wide
title: "Ansible"
description: ""
categories:
  - Projects
---
Today I will demonstrate some DevOps tools with a *simple application of no value*.

The end result of this series we will have:
  * [Debian](https://www.debian.org/) 10 as our OS
  * [Ansible](https://www.ansible.com/) for machine provisioning and configuration management
  * [Docker](https://www.docker.com/)  for containers
  * [Docker Compose](https://docs.docker.com/compose/) for managing containers
  * Containers:
    * [nginx](https://hub.docker.com/_/nginx) as a reverse proxy
    * [OpenVPN preconfigured for PIA](https://hub.docker.com/r/qmcgaw/private-internet-access/) (thats [Private Internet Access](https://www.privateinternetaccess.com/)) for a VPN. _Note this has since been replaced with Wireguard on Mullvad
    * [Deluge](https://hub.docker.com/r/linuxserver/deluge) as a torrent client
    * [Sonarr](https://hub.docker.com/r/linuxserver/sonarr/) a "PVR for Usenet and BitTorrent users"
    * [Radarr](https://hub.docker.com/r/linuxserver/radarr/) a Sonarr fork for movies
    * [Jackett](https://hub.docker.com/r/linuxserver/jackett/) to provide support for popular trackers
    * [Jellyfin](https://hub.docker.com/r/jellyfin/jellyfin/) as a media front end

I'm just using this as an example for demonstration purposes of course.

This page covers using Ansible to configure Docker and Docker Compose on Debian.
In later posts I plan to cover:
 - setting up our containers with Docker Compose.
 - using Ansible to help with the configuration for the containers.

For testing I used a VPS on [Linode](https://www.linode.com/) running Debian 10.

# Ansible
The [full Ansible playbook](https://github.com/ciferkey/media/blob/master/ansible.yml) is available but I'm going to breakdown what went in to it. The file is designed so that you can drop it on to a box as a fresh root user and it will handle everything for you.

I'm not coordinating multiple machines so the script will be run locally on the box you want to configure using the default localhost configuration:
{% highlight yml %}
---

  - name: Docker
    connection: local
    gather_facts: yes
    hosts: localhost
    vars_prompt:
      - name: password
        prompt: "Password for docker user"
        private: yes
        encrypt: "sha512_crypt"
        confirm: yes
        salt_size: 7
{% endhighlight %}

I've configured the playbook to [prompt](https://docs.ansible.com/ansible/latest/user_guide/playbooks_prompts.html) for a password to use with for the docker user since there is no secretes management tool. That could be a nice to have but would be overkill for such a *simple demo application*.


## Docker User
First we need a non root user to run the containers with. Instead of running these commands:
{% highlight shell %}
adduser docker
usermod -aG sudo docker
{% endhighlight %}

We can use the [user module](https://docs.ansible.com/ansible/latest/modules/user_module.html):
{% highlight yml %}
  - name: Create docker user
    user:
        name: docker
        password: "{{password}}"
        update_password: on_create
        groups:
            - sudo
            - render
            - video
        shell: /bin/bash
        state: present
{% endhighlight %}
The docker user will also be added to the render and video groups to enable access to hardware transcoding.

## Install Docker

The Docker [documentation recommends](https://docs.docker.com/install/linux/docker-ce/debian/) the following installation process:
{% highlight shell %}
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
{% endhighlight %}

This involves installing some dependencies, adding a key and a new repository to the package manager and installing docker. Ansible has built in support for many of these operations which simplifies tasks.

The [apt module](https://docs.ansible.com/ansible/latest/modules/apt_module.html) allows us to install dependencies. By using "update_cache: yes" we can skip having a separate update step:
{% highlight yml %}
  - name: Install Docker Deps
    apt:
        name:
            - apt-transport-https
            - ca-certificates
            - curl
            - gnupg2
            - software-properties-common
        update_cache: yes
{% endhighlight %}

The [apt_key module](https://manpages.ubuntu.com/manpages/bionic/man8/apt-key.8.html) lets us pull in a new key from a URL:
{% highlight yml %}
  - name: Install Docker Repository Key
    apt_key:
        url: https://download.docker.com/linux/debian/gpg
{% endhighlight %}

The [apt_repository module](https://docs.ansible.com/ansible/latest/modules/apt_repository_module.html) lets us add a new repository. Since I specified ["gather_facts: yes"](https://docs.ansible.com/ansible/latest/modules/gather_facts_module.html) at the start of the playbook Ansible collected information about the machine before it started. This includes a "ansible_distribution_release" value that lets us use the correct repository for our Debian release:
{% highlight yml %}
- name: Add Docker Respository
apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/debian &#123;&#123; ansible_distribution_release &#125;&#125; stable"
{% endhighlight %}

Then we can install Docker from the new repository:
{% highlight yml %}
- name: Install Docker
    apt:
        name:
            - docker-ce
            - docker-ce-cli
            - containerd.io
        update_cache: yes
{% endhighlight %}

Finally instead having to do this to enable the docker service to run on startup:
{% highlight shell %}
sudo systemctl enable docker
sudo systemctl start docker
{% endhighlight %}

The [systemd module](https://docs.ansible.com/ansible/latest/modules/systemd_module.html) can be used instead:
{% highlight yml %}
- name: Set Docker to Run on Startup
  systemd:
    state: started
    name: docker
    enabled: true
{% endhighlight %}

## Install Docker Compose
The Docker Compose [documentation recommends](https://docs.docker.com/compose/install/) the following installation process:
{% highlight shell %}
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
{% endhighlight %}

This pulls a binary from Github, saves the binary under /usr/local/bin and sets the appropriate permissions for it.

The [get_url module](https://docs.ansible.com/ansible/latest/modules/get_url_module.html) lets us fetch the binary off of GitHub. We use "ansible_system" and "ansible_architecture" from the gathered facts to generate the URL as opposed to [uname](http://man7.org/linux/man-pages/man2/uname.2.html). As a part of getting the binary we can set the mode to 755 for the docker user to avoid doing this in a second step. Also as per the docs "You must either add a leading zero so that Ansible's YAML parser knows it is an octal number (like 0644 or 01777) or quote it (like '644' or '1777') so Ansible receives a string and can do its own conversion from string into number."

{% highlight yml %}
  - name: Install Docker Compose
    get_url:
        url: https://github.com/docker/compose/releases/download/1.24.1/docker-compose-&#123;&#123;ansible_system&#125;&#125;-&#123;&#123;ansible_architecture&#125;&#125;
        dest: /usr/local/bin/docker-compose
        mode: '755'
        owner: docker
{% endhighlight %}

# Running the Playbook
At this point you can curl the playbook to a fresh Debian machine and run the following:
{% highlight shell %}
ansible-playbook ansible.yml
{% endhighlight %}

Now you will have Docker and Docker Compose ready to use on your machine.
