---
layout: single
permalink: /blog/2022/02/01/goodbye-docker-desktop
title:  "Goodbye Docker Desktop"
excerpt: "You were great while you lasted!"
date:   2022-02-01 00:00:00:00 +1000
author: max@aem.design
categories:
- blog
tags:
- docker-desktop
header:
    overlay_image: /assets/images/goodbye/goodbye-docker-desktop.png
---

First of all, if it wasn't for docker, the technology landscape would be very different today. Thank you, docker and the team behind it! I do feel that some technologies that come out as open-source and force other vendors to show their cards. Docker single-handled changed the way a lot of things are done in software development; this is probably an understatement, as it is amazing! I feel that the docker team is doing a great job and I hope this will continue.

But as you all know, docker desktop WAS a great tool for running docker on your desktop, up until it became a thing of the past. It drove itself off the cliff with paid subscription. Most corporate companies will think long and hard before purchasing docker desktop licences. So if you can't use it at work, why would you use it at home? After all, you are what you practice, and you are what you use. Consistency of tools is critical to developers.

There are many alternatives to the docker desktop stack, but let's not throw docker simplicity out the window yet. Obviously, Kube and Helm is the destination, but let's take small steps. For DevOps, using git with docker-compose gives you all the power of git and docker. Yes team behind Docker Desktop have added a lot of front end features to it, and maybe there is a use case for them, but in a pipeline driven world, you can't use them. You don't use docker desktop in production so keeping rest of the stack the same would be a good idea.

So this brings this journey to a crossroads. Do you build a VM and run the docker engine and docker-compose in there, or do you run this semi-natively? If you are on Linux/Unix you are alright; you can google your way out. On windows, however best experience would be attained through Powershell Core7, WSL2 and Windows Terminal. Go ahead try these on; you will never look back! Also, while you are at it, stop using CYGWIN to do this; you are making your life harder than you have to. :P

Now that you have tools from the future installed, lets proceed to the next steps

## WSL Install Ubuntu

This is a ubuntu guide, same as Docker Desktop, Centos ecosystem is dead, so using Ubuntu is the best option we have. (long story post to follow).

### Wsl download and Import ubuntu

Open up Powershell core and run the followig commands.

```powershell
mkdir c:/wsl
cd c:/wsl
curl.exe -L -o impish-server-cloudimg-amd64-root.tar.xz https://cloud-images.ubuntu.com/impish/current/impish-server-cloudimg-amd64-root.tar.xz
wsl --import ubuntu c:/wsl/ubuntu2110 c:/projects/wsl/impish-server-cloudimg-amd64-wsl.rootfs.tar.gz
```

Alternatively you can try this method:

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl.exe --set-default-version 2


dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl --install --distribution Ubuntu
wsl --install --distribution Debian
```

Now you have a ubuntu image in your WSL, you can restart Windows Terminal, and it will appear as a new option.

### Config Ubuntu

Because we did not use MS apx, the new Ubuntu has only a root user; best approach is to add a new user to your liking. Run the new ubuntu terminal in Windows Terminal, and in it run the following commands. (Change user name and password as you like when prompted.)

```bash
adduser -d /home/maxbarrass -m maxbarrass
passwd maxbarrass
addgroup maxbarrass sudo
usermod -aG sudo maxbarrass
sudo echo "$USER ALL=(ALL) NOPASSWD:ALL">>/etc/sudoers
```

### Update Windows Terminal Profile

Now that you have a new user in your Ubuntu, you can update your Windows Terminal profile to use the new user.

This should be in the Command line for your ubuntu profile:

```
wsl.exe -d ubuntu -u maxbarrass
```

### Install docker in Ubuntu

Create a new script `nano instal-docker.sh` with the following content and run it. This will install docker and docker-compose, as well as add docker service start to your `.profile`. This way, when you open your Ubuntu, it will ensure that docker is running.

```bash
# update the package manager and install some prerequisites (all of these aren't technically required)
sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common libssl-dev libffi-dev git wget nano

# create a group named docker and add yourself to it
#   so that we don't have to type sudo docker every time
#   note you will need to logout and login before this takes affect (which we do later)
sudo groupadd docker
sudo usermod -aG docker ${USER}

# add Docker key and repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" -y

# (optional) add kubectl key and repo
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# update the package manager with the new repos
sudo apt-get update

# upgrade the distro
sudo apt-get upgrade -y
sudo apt-get autoremove -y

# install docker
sudo apt-get install -y docker-ce containerd.io

# (optional) install kubectl
sudo apt-get install -y kubectl

# (optional) install latest version of docker compose
sudo curl -sSL https://github.com/docker/compose/releases/download/v`curl -s https://github.com/docker/compose/tags | grep "compose/releases/tag" | sed -r 's|.*([0-9]+\.[0-9]+\.[0-9]+).*|\1|p' | head -n 1`/docker-compose-`uname -s | tr '[:upper:]' '[:lower:]'`-`uname -m` -o /usr/local/bin/docker-compose 
sudo chmod +x /usr/local/bin/docker-compose

# ensure docker does not use iptabels
sudo touch /etc/docker/daemon.json
sudo tee -a /etc/docker/daemon.json <<EOF
{
  "iptables": false
}
EOF

# auto start docker on boot
echo "Starting docker service" 
echo "sudo service docker start" >> ~/.profile

# mount host drives to root /c/ etc.
sudo touch /etc/wsl.conf
sudo tee -a /etc/wsl.conf <<EOF
[automount]
root = /
options = "metadata"
EOF

#allow your account to sudo without password
sudo echo "$USER ALL=(ALL) NOPASSWD:ALL">>/etc/sudoers
```

Reboot, open windows terminal and open bash prompt. You should be prompted for password to start docker. After that you can run `docker ps` to see if docker is running.

### Thank you

I hope you enjoyed this guide. If you have any questions or comments, feel free to contact me. I will be happy to help.

Let me know what you think and don't forget to tell your friends.
