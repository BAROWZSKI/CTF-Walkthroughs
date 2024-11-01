# DC:1 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

DC:1 is a CTF created by DCAU on Vulnhub. You can find the technical information about this CTF on the Vulnhub page. First of all, make sure to download and install the correct OVA file from Vulnhub (https://download.vulnhub.com/dc/DC-1.zip). The author states that it runs on VirtualBox but it should also work on VMware.

## 1-General Recon
 
After setting your machine’s IP to work on your network, you’re ready to scan. If you can’t reach your machine from another virtual machine or your own computer, you should check your VirtualBox network settings. If you’re trying to reach your machine from another VirtualBox instance, both machines’ network settings should be set to Bridged Adapter or NAT Network. Once that’s configured, you’re ready to start scanning.

```bash
netdiscover -r 10.0.2.0/24 OR nmap -sn 10.0.2.0/24
```

![DC1](/https://miro.medium.com/v2/resize:fit:1100/format:webp/1*qleG8B44yWr622FFIfpGBA.png "DC1")

As you can see above, our IP address is 10.0.2.4 (yours may be different).

I attempted a directory search on this IP, but found nothing special or interesting. I also tried reading the source code, but there was nothing there either.
If you’d like to try it yourself, I’ll give the code below.
(There’s no need to search for subdirectories.)

```bash
dirsearch -u http://10.0.2.10
```
## 2- Port scanning

Normally, with a simple port scan, you should first identify the open ports and then perform a version or script scan on them. You can use Rustscan or Nmap to find the open ports. (If you’d like to save the output, you can append -oG scan.txt at the end of the command.)

```bash
nmap -p- -vv <MACHINE-IP>
```
And then you scan the open ports on this machine by :
```bash
nmap -p 22,80(open-ports) -vv -sV -sC <Machine-IP>
```
On this machine, it’s crucial to scan the ports thoroughly. By adding a simple parameter, you can potentially discover a CVE. The parameter is:

--script vuln -p- <Machine IP>

So, your best scan command would look like this:

```bash
nmap -Pn -sS -sV --script vuln -p- <Machine Ip>
```
![DC1](https://miro.medium.com/v2/resize:fit:828/format:webp/1*G29V3YZdWyH9CMG_CUNDGw.png "DC1")

I discovered that Drupal, specifically version 7, is outdated and has several known vulnerabilities. You can see the CVE code referenced above. Drupal is a Content Management System (CMS) similar to WordPress. Like WordPress, there are numerous vulnerabilities and enumeration techniques available for this CMS.

## 3-Exploiting

By searching for the CVE-2014–3714 vulnerability, you can easily obtain a Meterpreter shell. Searchsploit can be useful for this, but I prefer searching directly on msfconsole. To do this, enter the command “use 0 “and set your rhosts. Then, simply type run in the command line, and it will provide you with a Meterpreter shell. However, this method is quite straightforward, so I prefer not to elaborate on it further.

![DC1](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*JJ8qJsskJ8A2esfvIipxrw.png "DC1")
Set your rhosts and run (use options to see what are you using.)(OPTIONAL)

I search for drupal instead , and used 1. option.

![DC1](https://miro.medium.com/v2/resize:fit:828/format:webp/1*Q-7xukBHeaVWPKeq9X5bqA.png "DC1")

Then i set my rhosts to my dc 1 machine and used the command run.
![DC1](https://miro.medium.com/v2/resize:fit:828/format:webp/1*G4Ls7RJkYrvC2M4FHSMfTA.png "DC1")

This python code makes your bash shell more interactive. Setting enviroment is just and optional and it’s just makes your command line work more properly.
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```
Then you run the command ls and you got the first flag “flag1.txt”. And it says every CMS needs a config file etc. Then you google where is the config file of Drupal. You find out it’s in the
/var/www/sites/default/settings.php

When you look for source code you find credentials.

(Also i don’t know why but i found the flag4 in home directory. )

![DC1](https://miro.medium.com/v2/resize:fit:640/format:webp/1*XS0cfy-ebmqtANIQZDICaQ.png "DC1")

We know website use mysql database from port scanning. So we use this credentials to gather informations.

```bash
mysql -u dbuser -p;
```
Asks for password and you give it.

```bash
show databases;
use drupaldb;
show tables;
select * from users;
```

I used above commands and get admin’s password hash besides Fred. But it was a drupal7 hash and i couldn’t crack it with hashcat or john. So i give up.

 ### 4- Privilege Escalation

I used to command find to become root. And i don’t know how would i know to escalate my privilege with this command. Because sudo -l is not working and as a www-data i don’t know which command i can use with root level. So i search in GTFOBins and i tried find command. I created a notes in /var/www named a.txt and searchd it.

```bash
find / -name a.txt -exec "whoami" \;
```
And because of i can use find command with root level authority. I can use the exec parameter to bring me a bash shell with root privilege.

![DC1](https://miro.medium.com/v2/resize:fit:786/format:webp/1*BcSNHaY0CHpqPCJVbUtSIA.png "DC1")

Finally in the /root directory final flag stands named thefinalflag.txt.

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)