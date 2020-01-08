---
layout: single
classes: wide
title: "Media Center Series Part 3: Simpler  Raspberry Pi 4"
description: "Notes on using Wireguard with Docker on a Raspberry Pi 4"
categories:
  - Projects
---

In a previous post I discussed [using Ansible to install Docker and Docker Compose on Debian]({% post_url _posts/2019-10-01-Example-Ansible %}). Since then I've taken this and made some updates as I've gone and modified for use on a Raspberry Pi 4 instead of just testing on a Linode instance.
 
# Changes

## Docker Installation
As fun (complicated?) as it was manually doing all the steps to install Docker, there is a nice [first party install script](https://github.com/docker/docker-install) you can curl and run instead. It was not usable on the RPi 4 previously which is why I had not originally used it but it has since been updated and works perfectly on Raspbian.

Another upside of this change is it means you don't have to manually check and set the architecture and release when you set up the repos. All that is handled by the script.

## Docker Compose Installation
Docker Compose is now [installed using Pip](https://docs.docker.com/compose/install/#install-using-pip) rather than curling the binary and dropping it on the box. The main downside is this take makes longer to run, especially since the pip install has native code that needs to be compiled and its happening on the RPi itself. The main upside is... well I guess I'd just rather use a package manager in this case? Perhaps the rationale for this change is not as strong as the other one.

Also note that Docker Compose seems to have problems with Python 2's backported libraries  on Debian. There are currently two that gave me problems on Raspbian: 
* python-backports.ssl-match-hostname
* python-backports.shutil_get_terminal_size

You can either (1) uninstall them with pip and then reinstall them with apt or (2) install and use Python 3 instead.

Note that I also made the choice to install into the system Python rather than a virtual environment for simplicity.

## Running the Playbook

Final change is the playbook now uses a ssh connection rather than running locally on the RPi so it must now be provided an inventory when run.

# The updated playbook
Here is the updated playbook:

{% highlight yml %}
{% raw %}

---

  - name: Docker
    connection: ssh
    gather_facts: yes
    hosts: rpi
    vars_prompt:
      - name: password
        prompt: "Password for docker user"
        private: yes
        encrypt: "sha512_crypt"
        confirm: yes
        salt_size: 7
    become_user: root
    become: yes
    tasks:
      - name: Download Docker Install Script
        get_url:
          url: https://get.docker.com
          dest: /tmp/docker_install.sh
      - name: Run the Docker Install Script
        shell: sh /tmp/docker_install.sh
      - name: Create docker user
        user:
            name: docker
            password: "{{password}}"
            update_password: on_create
            groups:
                - sudo
                - render
                - video
                - docker
            shell: /bin/bash
            state: present
      - name: Set Docker to Run on Startup
        systemd:
          state: started
          name: docker
          enabled: true
      - name: Docker Compose Dependencies Workaround
        # See https://stackoverflow.com/a/51071841
        pip:
          name:
            - backports.ssl-match-hostname
            - backports.shutil_get_terminal_size
          state: absent
      - name: Install Docker Compose Dependencies
        apt:
          pkg:
            - python-backports.ssl-match-hostname
            - python-backports.shutil_get_terminal_size
            - python-pip
            - python-dev
            - libffi-dev
            - libssl-dev
            - build-essential
          update_cache: yes
      - name: Install Docker Compose
        pip:
          name: docker-compose
{% endraw %}
{% endhighlight %}
