---
layout: post
section-type: post
has-comments: true
title: Hack The Box - Crafty
category: tech
tags: ["tutorial"]
---

### Reconnaissance and Initial Steps

#### Nmap Results

```
nmap 10.10.11.249 -p-
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/fbc9f2f4-73e7-4f98-9fb3-7856d3a97db9)

```
sudo nmap -p80,25565 -sV -sC 10.10.11.249
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/4a79d415-09b1-46a8-8d79-2dbb2f999681)


I add crafty to `/etc/hosts` with the corresponding ip address given. Then navigate to `http://crafty.htb`

```
echo "10.10.11.249 crafty.htb" | sudo tee -a /etc/hosts
```

When I visited `crafty.htb`, I found a Minecraft introduction page.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/4c649f78-9e8e-42b2-ac1d-7b782e8d63c0)


### Foothold
However, for `port 25565`, I recall there being a log4j vulnerability, `CVE-2021–44228` to be precise. This exploit enables control over log messages and parameters to execute arbitrary code. An exploit for this vulnerability can be found here.

```
git clone https://github.com/kozmer/log4j-shell-poc
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/be2f0fed-4ad2-48b6-bbf9-ccd312818f72)


Modify the `String cmd` variable to ensure compatibility with Windows.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/0f5a7e2e-8431-4bb8-bdca-a37c2d19fcfd)

In order for `poc.py` to run smoothly we need a java archive to be named `jdk1.8.0_20`. I found a java archive here

```
wget https://repo.huaweicloud.com/java/jdk/8u181-b13/jdk-8u181-linux-x64.tar.gz
tar -xf jdk-8u181-linux-x64.tar.gz
mv jdk1.8.0_181 jdk1.8.0_20
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/ea8c6034-046d-4403-be85-10b8293e35c4)

To connect to the server, I needed to download the Minecraft client on my Kali system. After conducting some research, I downloaded TLauncher. https://tlauncher.org/en/download_1/minecraft-1-16-5_12582.html (I don't recommend using this, but I proceeded anyway since I'm installing it on a VM.) 
And I used Minecraft 1.16.5 Java Edition as I knew it's vulnerable to log4j.

one download the TLauncher.zip file, unzip it and open it by runing below command:


```
sudo java -jar TLauncher.jar
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/468caa45-5db4-42da-88ae-68c589cff9ef)


![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/a5013fe2-04a9-4294-a7e9-4e607b6e33a7)

after that select the correct release version and the minecraft account Click on `Enter the game`










