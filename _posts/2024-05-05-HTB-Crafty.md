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

