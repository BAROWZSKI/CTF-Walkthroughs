## 0day TryHackMe CTF Walkthrough

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

This CTF was created by 0day (https://tryhackme.com/r/p/0day), who is a professional pentester and one of my hacker idols. In my opinion, this CTF isn’t too difficult if you know the basics of Linux.

I used my Kali Linux setup with OpenVPN to connect to the machine. If you haven’t downloaded your OpenVPN file yet, you can easily find and download your unique OpenVPN file from TryHackMe with a quick search. You could also use TryHackMe’s virtual machines, but they’re pretty slow, so I recommend trying it on your own machine instead.

## 1 - Reconnaissance

I started with the basic Rustscan tool to find open ports. Then, I ran version and script scans with Nmap.

While those were running, I checked the source code but didn’t find anything noteworthy.

```bash
rustscan <machine_ip>
nmap -PN -sS -sV -p 22,80 <machine_ip> -oN VersionScan
```

![0day](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*STQdlhjvPkYYXvAuUxmxYA.png "0day")

So, there are two open ports as usual: 80 (HTTP) and 22 (SSH). I started Dirsearch to look for suspicious directories and ran Nikto for general web vulnerability scanning.

```bash
dirsearch -u http://<machine_ip>
nikto --url http://<machine_ip>
```
Nikto gave me the crucial output for this CTF.

![0day](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*VkTHhmuGz3Uy6SBl7BBrCg.png "0day")

I downloaded the image files and extracted strings from them, thinking I might be able to SSH using one of the lines, but it didn’t work. AperiSolve is a great tool for extracting data from image files:

https://www.aperisolve.com/.

## 2- Exploitation

I looked up “shellshock” online, but it turned out to be a bit complicated to fully understand. It’s a vulnerability in older versions of Bash. I searched for the CVE number on both Searchsploit and Metasploit (msfconsole).

It’s simpler to use msfconsole for enumeration, though Searchsploit works too. I launched msfconsole and searched for the CVE-2014–6278 exploit, which you can also find by searching “shellshock.” I chose the first option, set my RHOSTS, LHOSTS, TARGETURI, and ports, then ran the run command.

![0day](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*apUPfhRZqETvJN3kB_uzNg.png "0day")
![0day](https://miro.medium.com/v2/resize:fit:828/format:webp/1*nPQMq4_d1jbjmGUYykIEpw.png "0day")

Now I have a www-data shell on this machine, so I ran a few commands to make my shell more interactive and responsive.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

The first flag, user.txt, is ready in the /home/ryan directory. Now I need to escalate my privileges to root in order to read the root.txt flag.

## 3- Privilege Escalation

I couldn’t run sudo -l on this machine since I don’t know the password for the www-data user. I tried some privilege escalation techniques from other CTFs I’ve completed, but they didn’t work.

```bash
find / -perm -4000 -type -f 2>/dev/null
```

So, I checked which machine I was working on.

```bash
uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

There are various commands to check the version and distribution, and I found it was running Ubuntu 3.13.0–32-generic — an older version. I searched for any known exploits for this version and tried Searchsploit first.

```bash
Some of them : 

cat /etc/os-release

lsb_release -a

hostnamectl

cat /etc/issue
```
![0day](https://miro.medium.com/v2/resize:fit:828/format:webp/1*I24Tx8FHBaK8xMEdQUvItA.png "0day")

Now that I have the exploit file, I need to transfer it to my www-data multihandler session, then place it in /tmp to run it with gcc. To do this, I started a PHP server on my local machine so I could wget the file in the www-data session.

![0day](https://miro.medium.com/v2/resize:fit:3718/format:webp/1*DNTHF6ry70bXEsjjThOoBw.png "0day")

After downloading, I compiled the .c file using gcc, then ran it. Now I’m root and can cat the root.txt file in /root/root.txt.

![0day](https://miro.medium.com/v2/resize:fit:640/format:webp/1*yEfS9mT_dyNsUY7NzpGTlA.png "0day")

In conclusion, this CTF taught the value of checking for a cgi-bin folder on web servers and the power of Nikto in uncovering such directories. Plus, it provided a practical example of the Shellshock vulnerability in both cybersecurity and real-world contexts.

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)