# DC:1 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

DC:6 is a CTF created by DCAU. You can find the technical information about this CTF on the VulnHub page. First of all, make sure to download and install the correct OVA file from VulnHub. (https://download.vulnhub.com/dc/DC-6.zip).

On the VulnHub page, the author provides a hint for brute forcing. He mentions that you don’t have to wait a year to finish the attack, and offer a shorter wordlist from rockyou.txt..

![DC6](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*wWdhzZ--sontnfqjb_qsgg.png "DC6")

## 1 - Recon

I always create a directory on my desktop and take notes in it. As usual, I started with Rustscan and Nmap. But since it’s using WordPress as the CMS, I needed to add the machine’s IP to my /etc/hosts file (the author mentions this as well).

![DC6](https://miro.medium.com/v2/resize:fit:640/format:webp/1*SE0xrM3iZ3sb5wJo6ObICQ.png "DC6")

```bash
rustscan wordy (or nmap -p- -vv wordy)

*and version scan*

nmap -PN -sS -sV --script vuln -p wordy -oN finalscan
```
Instead of the Nmap scan above, you can also use wpscan. It’s more suitable for scanning WordPress websites. Either way, I found some critical information about the website.

![DC6](https://miro.medium.com/v2/resize:fit:828/format:webp/1*KZceBYPzLsN2LJqPzBZCUA.png "DC6")

I checked the website on firefox and there was an interesting paragraph.

```bash
Welcome to Wordy, a world leader in the area of WordPress Plugins and Security.
At Wordy, we know just how important it is to have secure plugins, and for this reason, we endeavour to provide the most secure and up-to-date plugins that are available on the market.
```
The paragraph about plugins. So I checked if plugins up to date or vulnerable somehow.

```bash
nikto -h http://wordy
```
But there was nothing important.

Also I found another not important stuffs with dirsearch.

![DC6](https://miro.medium.com/v2/resize:fit:828/format:webp/1*CoKJqHgrgjgGtOvGzWzWcQ.png "DC6")

![DC6](https://miro.medium.com/v2/resize:fit:640/format:webp/1*A0h7gynFYNyztV2LSjN1uQ.png "DC6")

I searched the images on apperisolve webpage. It is usefull webpage for automized searchs for jpgs, gifs etc…

I created a users.txt with above users list. And created a passwords.txt with author’s command.

![DC6](https://miro.medium.com/v2/resize:fit:720/format:webp/1*pQQfjiKJefFiShCK0dELRQ.png "DC6")

With below command I tried to brute force login page.

```bash
wpscan -U users.txt -P passwords.txt -t 60 --url http://wordy
``` 

I found the mark’s password mark / helpdesk01.

## 2- Getting Reverse Shell

Logged in with mark credentials and found out website uses the plugin “activity monitor “. There must be exploit for it so I searched on searchsploit. And found out one. I copied the exploit file in my current directory.

![DC6](https://miro.medium.com/v2/resize:fit:828/format:webp/1*IS36WCFMqq1mtA9uJwjNwQ.png "DC6")

I modified the exploit file by changing lhost and rhost variables. Searched for this file on browser and when I press the button , I get the shell in my netcat listener.

```bash
rlwrap nc -lvnp <port number>
```
After I get the shell I did the basic shell things.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

After that I searched the directories and there was a “things-to-do.txt” on mark’s directory. It says :

![DC6](https://miro.medium.com/v2/resize:fit:750/format:webp/1*-FmQXrhqD1YWY8YUO68vjw.png "DC6")

So , there is a user named graham and his password is GSo7isUM1D4.

I tried to connect ssh with this credentials. After that I run the command “sudo -l”. And now I got what graham can do as a jens with admin privileges. So i tried to “ sudo -u jen ./backups.sh”.

![DC6](https://miro.medium.com/v2/resize:fit:828/format:webp/1*hRxAVOHiL2TjY9GpUDZjOw.png "DC6")

Jens user can run nmap with super user privileges. In gftobins there is a code for almost every linux command.

```bash
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF
```

Now I am root now and can read “theflag.txt”.

![DC6](https://miro.medium.com/v2/resize:fit:828/format:webp/1*5vI9NzM_lDllQ0V1bx8wAg.png "DC6")

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)