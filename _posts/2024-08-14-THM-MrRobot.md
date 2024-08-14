---
layout: post
section-type: post
has-comments: true
title: Try Hack Me - Mr Robot
category: tech
tags: ["tutorial"]
---

### Reconnaissance and Initial Steps

##### Nmap Results

```
sudo nmap -Pn -A 10.10.242.39
```

![image](https://github.com/user-attachments/assets/de811a40-544a-4d62-bafd-f03e64b11c4d)

Port scans revealed that ports 80 and 443 were open, indicating that HTTP and HTTPS services were running on these ports, respectively.

![image](https://github.com/user-attachments/assets/add084e3-351a-4e2a-89a7-c2a17e62f267)

Upon inspecting the source code, nothing useful was found. In addition to searching for the source code, the examination of the `robots.txt` file, which is used by search engine crawlers to determine which files are allowed for parsing, was carried out.

![image](https://github.com/user-attachments/assets/654b72d0-0600-4270-b67f-8c484d9fcb98)

We now have two files: one is a text file containing the first key, and the other is a dictionary file.

