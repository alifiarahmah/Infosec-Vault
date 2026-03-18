- Footprinting - Cheatsheets
- [[Footprinting - Skills Assessment]]

# Footprinting
## Enumeration Principles & Methodology

> [!important]
> OSINT is an independent procedure and should be performed separately from enumeration because **OSINT is based exclusively on passive information gathering** and does not involve active enumeration of the given target.

Our goal is not to get at the systems but to find all the ways to get there.
- What can we see?
- What reasons can we have for seeing it?
- What image does what we see create for us?
- What do we gain from it?
- How can we use it?
- What can we not see?
- What reasons can there be that we do not see?
- What image results for us from what we do not see?

Principles:
1.	There is more than meets the eye. Consider all points of view.
2.	Distinguish between what we see and what we do not see.
3.	There are always ways to gain more information. Understand the target. 

### Methodology

We know that penetration testing, and therefore enumeration, is a dynamic process. Consequently, we have developed a static enumeration methodology for external and internal penetration tests that includes free dynamics and allows for a wide range of changes and adaptations to the given environment

| **Layer**                | **Description**                                                                                        | **Information Categories**                                                                         |
| ------------------------ | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `1. Internet Presence`   | Identification of internet presence and externally accessible infrastructure.                          | Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances, Security Measures      |
| `2. Gateway`             | Identify the possible security measures to protect the company's external and internal infrastructure. | Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare                  |
| `3. Accessible Services` | Identify accessible interfaces and services that are hosted externally or internally.                  | Service Type, Functionality, Configuration, Port, Version, Interface                               |
| `4. Processes`           | Identify the internal processes, sources, and destinations associated with the services.               | PID, Processed Data, Tasks, Source, Destination                                                    |
| `5. Privileges`          | Identification of the internal permissions and privileges to the accessible services.                  | Groups, Users, Permissions, Restrictions, Environment                                              |
| `6. OS Setup`            | Identification of the internal components and systems setup.                                           | OS Type, Patch Level, Network config, OS Environment, Configuration files, sensitive private files |

## Infra-Based Enum

### Domain

#### crt.sh

```sh
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
```

If needed, we can also have them filtered by the unique subdomains.
```sh
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

Identify directly accessible hosts
```sh
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

Once we see which hosts can be investigated further, we can generate a list of IP addresses with a minor adjustment to the `cut` command and run them through `Shodan`. [Shodan](https://www.shodan.io/) can be used to find devices and systems permanently connected to the Internet like `Internet of Things` (`IoT`).
As a result, we can find devices and systems, such as `surveillance cameras`, `servers`, `smart home systems`, `industrial controllers`, `traffic lights` and `traffic controllers`, and various network components.

#### Shodan

```sh
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done

for i in $(cat ip-addresses.txt);do shodan host $i;done
```

#### Dig DNS record

```sh
dig any inlanefreight.com
```

### Cloud

#### Company Hosted Servers

```sh
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

Often cloud storage is added to the DNS list when used for administrative purposes by other employees.
during the IP lookup, we have already seen that one IP address belongs to the `s3-website-us-west-2.amazonaws.com` server

#### Google Dorks

- AWS --> `intext:... inurl:amazonaws.com`
- Azure --> `intext:... inurl:blob.core.windows.net`

#### Target Website - Source Code
- [domain.glass](https://domain.glass)
- https://buckets.grayhatwarfare.com/
	- May found private key here

### Staff
- Linkedin
	- Find out their staff's skill
	- Find github, could has hardcoded JWT token, etc.

## Host-Based Enum

### FTP (21)

File Transfer Protocol

#### TFTP (Trivial FTP)

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits tftp.                                                                                                                            |
| `status`     | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |
Anonymous login
```sh
ftp <IP>
```

Get status
```sh
status
```

vsFTPd Detailed Output
```sh
debug
trace
```

|**Setting**|**Description**|
|---|---|
|`dirmessage_enable=YES`|Show a message when they first enter a new directory?|
|`chown_uploads=YES`|Change ownership of anonymously uploaded files?|
|`chown_username=username`|User who is given ownership of anonymously uploaded files.|
|`local_enable=YES`|Enable local users to login?|
|`chroot_local_user=YES`|Place local users into their home directory?|
|`chroot_list_enable=YES`|Use a list of local users that will be placed in their home directory?|

| **Setting**             | **Description**                                                                  |
| ----------------------- | -------------------------------------------------------------------------------- |
| `hide_ids=YES`          | All user and group information in directory listings will be displayed as "ftp". |
| `ls_recurse_enable=YES` | Allows the use of recurse listings.                                              |

Recursive listing
```sh
ls -R
```

Download
```sh
ls
get <filename>
```

Download all available files
```sh
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
tree .
```

Upload file
```sh
touch testupload.txt
put testupload.txt
ls
```

#### Footprint with Nmap

Update NSE
```sh
sudo nmap --script-updatedb
```

```sh
find / -type f -name ftp* 2>/dev/null | grep scripts

sudo nmap -sV -p21 -sC -A 10.129.14.136
```

Run nmap script with trace
```sh
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

Service interaction
```sh
nc -nv 10.129.14.136 21
```

```sh
telnet 10.129.14.136 21

openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

### SMB (139,445)

Server Message Block

Connect to share
```sh
smbclient -N -L //10.129.14.128
smbclient //10.129.14.128/notes
ls
get <file>
```

#### Status
```sh
smbstatus
```

#### Footprint with Nmap

```sh
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

Output nmap not much, jadi bisa pake rpcclient
```sh
rpcclient -U "" 10.129.14.128
```

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |
| `querygroup <ID>`         |                                                                    |
##### Brute-force User RIDs
```sh
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

##### samrdump.py
```sh
samrdump.py 10.129.14.128
```

##### smbmap
```sh
smbmap -H 10.129.14.128
```

##### CrackMapExec
```sh
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

##### Enum4Linux-ng
```sh
./enum4linux-ng.py 10.129.14.128 -A
```

We need to use more than two tools for enumeration. Because it can happen that due to the programming of the tools, we get different information that we have to check manually. Therefore, we should never rely only on automated tools where we do not know precisely how they were written.

### NFS (111,2049)
Network File System
#### Footprinting
```sh
sudo nmap 10.129.14.128 -p111,2049 -sV -sC

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
...
```

The `rpcinfo` NSE script retrieves a list of all currently running RPC services, their names and descriptions, and the ports they use. This lets us check whether the target share is connected to the network on all required ports. Also, for NFS, Nmap has some NSE scripts that can be used for the scans. These can then show us, for example, the `contents` of the share and its `stats`.

```sh
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
```

Once we have discovered such an NFS service, we can mount it on our local machine.

##### Show available NFS shares
```sh
showmount -e 10.129.14.128
```

##### Mounting NFS Share
```sh
mkdir target-NFS
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
cd target-NFS
tree .
ls -l
```

##### Unmounting
```sh
cd ..
sudo umount ./target-NFS
```


### DNS (53)

#### Footprint

##### Dig ns query
```sh
dig ns inlanefreight.htb @10.129.14.128
```

##### Dig any query
```sh
dig any inlanefreight.htb @10.129.14.128
```

##### Dig axfr
```sh
dig axfr inlanefreight.htb @10.129.14.128
```

##### DNSEnum 
To do subdomain brute forcing
```sh
dnsenum --dnsserver 10.129.9.238 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/SecLists/Discovery/DNS/fierce-hostlist.txt dev.inlanefreight.htb
```

### SMTP (25)

#### Default configuration

Send email
```sh 
telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


EHLO inlanefreight.htb

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING


MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
...

250 2.0.0 Ok: queued as 6E1CF1681AB

QUIT

221 2.0.0 Bye
Connection closed by foreign host.
```

#### Footprinting

##### Nmap
```sh
sudo nmap 10.129.14.128 -sC -sV -p25

sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

##### Enumerate username from wordlist
```sh
smtp-user-enum -U footprinting-wordlist.txt -t 10.129.9.238
```

### IMAP / POP3 (110,143,993,995)
Unlike the `Post Office Protocol` (`POP3`), IMAP allows online management of emails directly on the server and supports folder structures. 

By default, ports `110` and `995` are used for POP3, and ports `143` and `993` are used for IMAP. The higher ports (`993` and `995`) use TLS/SSL to encrypt the communication between the client and server.

```sh
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC
```

If we successfully figure out the access credentials for one of the employees, an attacker could log in to the mail server and read or even send the individual messages.
```sh
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd
```

To interact with the IMAP or POP3 server over SSL, we can use `openssl`, as well as `ncat`.
```sh
openssl s_client -connect 10.129.14.128:pop3s
openssl s_client -connect 10.129.14.128:imaps
```

> [!important] Note
> FQDN = Fully Qualified Domain Name
> A customized version of the POP3 server refers to a modified or tailored implementation of the standard POP3 protocol to meet specific organizational or functional requirements

Custom notes in OpenSSL
- `LOGIN [username] [password]`
- `1 SELECT [yang ada \HasNoChildren]`
- `1 fetch 1 all`: Baca list email
- `1 fetch 1 inbox`: Baca body email

```
> 1 LOGIN robin robin
1 OK [...] Logged in

> 1 LIST "" *
* LIST (\Noselect \HasChildren) "." DEV
* LIST (\Noselect \HasChildren) "." DEV.DEPARTMENT
* LIST (\HasNoChildren) "." DEV.DEPARTMENT.INT
* LIST (\HasNoChildren) "." INBOX

> 1 SELECT DEV.DEPARTMENT.INT

> 1 fetch 1 all
* 1 FETCH (FLAGS (\Seen) INTERNALDATE "08-Nov-2021 23:51:24 +0000" RFC822.SIZE 167 ENVELOPE ("Wed, 03 Nov 2021 16:13:27 +0200" "Flag" (("CTO" NIL "devadmin" "inlanefreight.htb")) (("CTO" NIL "devadmin" "inlanefreight.htb")) (("CTO" NIL "devadmin" "inlanefreight.htb")) (("Robin" NIL "robin" "inlanefreight.htb")) NIL NIL NIL NIL))
1 OK Fetch completed (0.007 + 0.000 + 0.006 secs).

> 1 fetch 1 body[text]
```

### SNMP (udp/161)
`Simple Network Management Protocol` ([SNMP](https://datatracker.ietf.org/doc/html/rfc1157)) was created to monitor network devices. In addition, this protocol can also be used to handle configuration tasks and change settings remotely. SNMP-enabled hardware includes routers, switches, servers, IoT devices, and many other devices that can also be queried and controlled using this standard protocol.

For footprinting SNMP, we can use tools like `snmpwalk`, `onesixtyone`, and `braa`. `Snmpwalk` is used to query the OIDs with their information. `Onesixtyone` can be used to brute-force the names of the community strings since they can be named arbitrarily by the administrator

SNMPWalk
```sh
snmpwalk -v2c -c public 10.129.14.128
```

OneSixtyOne
```sh
sudo apt install onesixtyone
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

Braa: Brute force individual IODs and enumerate info behind them
```sh
sudo apt install braa

# braa <community string>@<IP>:.1.3.6.*
braa public@10.129.14.128:.1.3.6.*
```

### MySQL (3306)

```sh
sudo nmap 10.129.183.68 -sV -sC -p3306 --script mysql*

mysql -u root -h 10.129.14.132
mysql -u robin -probin -h 10.129.183.68 [--disable-ssl]
```

| **Command**                                          | **Description**                                                                                       |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `mysql -u <user> -p<password> -h <IP address>`       | Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password. |
| `show databases;`                                    | Show all databases.                                                                                   |
| `use <database>;`                                    | Select one of the existing databases.                                                                 |
| `show tables;`                                       | Show all available tables in the selected database.                                                   |
| `show columns from <table>;`                         | Show all columns in the selected table.                                                               |
| `select * from <table>;`                             | Show everything in the desired table.                                                                 |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.                                                      |

## MSSQL (1433)

Client: [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15), [SQL Server PowerShell](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15), [HeidiSQL](https://www.heidisql.com/), [SQLPro](https://www.macsqlclient.com/), [Impacket's mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)

| Default System Database | Description                                                                                                                                                                                            |     |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --- |
| `master`                | Tracks all system information for an SQL server instance                                                                                                                                               |     |
| `model`                 | Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database |     |
| `msdb`                  | The SQL Server Agent uses this database to schedule jobs & alerts                                                                                                                                      |     |
| `tempdb`                | Stores temporary objects                                                                                                                                                                               |     |
| `resource`              | Read-only database containing system objects included with SQL server                                                                                                                                  |     |

```sh
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

MSSQL Ping in Metasploit
```sh
use scanner/mssql/mssql_ping
set rhosts 10.129.201.248
run
```

Connecting with Mssqlclient.py
```sh
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth

# Use impacket in kali
impacket-mssqlclient backdoor@10.129.182.6 -windows-auth
```

## Oracle TNS (1521)

The Oracle Transparent Network Substrate (TNS) server is a communication protocol that facilitates communication between Oracle databases and applications over networks. TNS supports various networking protocols between Oracle databases and client applications, such as IPX/SPX and TCP/IP protocol stacks.

Scan Oracle listener port
```sh
sudo nmap -p1521 -sV 10.129.204.235 --open
```

Bruteforcing
```sh
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

[ODAT](https://github.com/quentinhardy/odat) perform a variety of scans to enumerate and gather information about the Oracle database services and its components. Those scans can retrieve database names, versions, running processes, user accounts, vulnerabilities, misconfigurations, etc.
```sh
odat all -s 10.129.204.235
```

SQLPlus to connect to Oracle database
```sh
sqlplus scott/tiger@10.129.204.235/XE  

# if error while loading shared libraries: libsqlplus.so 
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

Oracle RDBMS interaction
```sql
select table_name from all_tables; 
select * from user_role_privs;  # extract password hashes 
select name, password from sys.user$;
```

Oracle RDBMS Database enumeration
```sh
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

Default path for webserver

| **OS**  | **Path**             |
| ------- | -------------------- |
| Linux   | `/var/www/html`      |
| Windows | `C:\inetpub\wwwroot` |

File upload
```sh
echo "Oracle File Upload Test" > testing.txt odat utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt  curl -X GET http://10.129.204.235/testing.txt
```

## IPMI (623)

[Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI_Basics) ( IPMI) is a set of standardized specifications for hardware-based host management systems used for system management and monitoring. Systems that use the IPMI protocol are called Baseboard Management Controllers (BMCs). BMCs are typically implemented as embedded ARM systems running Linux, and connected directly to the host's motherboard.

Many BMCs (including HP iLO, Dell DRAC, and Supermicro IPMI) expose a web-based management console, some sort of command-line remote access protocol such as Telnet or SSH, and the port 623 UDP, which, again, is for the IPMI network protocol.
```sh
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

Metasploit version scan
```
use auxiliary/scanner/ipmi/ipmi_version 
set rhosts 10.129.42.195 
run
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

It is essential to try out known default passwords for ANY services that we discover, as these are often left unchanged and can lead to quick wins.

During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place. This can be leveraged to obtain the password hash for ANY valid user account on the BMC. Use metasploit

```
use auxiliary/scanner/ipmi/ipmi_dumphashes 
set rhosts 10.129.42.195 
run
```

Crack using hashcat mode 7300
```sh
hashcat -m 7300 a.txt /usr/share/wordlists/rockyou.txt
``` 

## Remote Management Protocols

### SSH (22)

[ssh-audit](https://github.com/jtesta/ssh-audit) checks the client-side and server-side configuration and shows some general information and which encryption algorithms are still used by the client and server.

`git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit ./ssh-audit.py 10.129.14.132`

Banner example: in SSH-1.99-OpenSSH_3.9p1, we know that we can use both protocol versions SSH-1 and SSH-2, and we are dealing with OpenSSH server version 3.9p1.

Change authentication method

```sh
ssh -v cry0l1t3@10.129.14.132 ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password
```
### Rsync (873)

[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files. It can be used to copy files locally on a given machine and to/from remote hosts.

`sudo nmap -sV -p 873 127.0.0.1`

Probing for accessible shares

```sh
nc -nv 127.0.0.1 873
```

Enumerating open share
```sh
rsync -av --list-only rsync://127.0.0.1/dev  # sync all files rsync -av rsync://127.0.0.1/dev
```

If Rsync is configured to use SSH to transfer files, we could modify our commands to include the -e ssh flag, or -e "ssh -p2222" if a non-standard port is in use for SSH.

### R-Services (512,513,514)

R-Services are a suite of services hosted to enable remote access or issue commands between Unix hosts over TCP/IP. Much like telnet, it transmit info in unencrypted format

R-services span across the ports 512, 513, and 514 and are only accessible through a suite of programs known as r-commands.

Most frequently abused commands:

- rcp: copy file, but provide no warning for overwriting
- rsh: open shell without login procedure
- rexec: run shell commands on remote machine
- rlogin: login into remote hosts over the network

rsh, rexec, and rlogin relies upon trusted entries in /etc/hosts.equiv and .rhosts.

`sudo nmap -sV -p 512,513,514 10.0.17.2  PORT    STATE SERVICE    VERSION 512/tcp open  exec? 513/tcp open  login? 514/tcp open  tcpwrapped  cat .rhosts htb-student     10.0.17.5 +               10.0.17.10 +               +`

In this example, the + modifier allows any external user to access r-commands from the htb-student user account via the host with the IP address 10.0.17.10.

Login using rlogin

`rlogin 10.0.17.2 -l htb-student`

Listing authenticated users

```
rwho # root     web01:pts/0 ... # htb-student     workstn01:tty1 ..  rusers -al 10.0.17.5  # more detailed   
```

### RDP (3389)

```
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*  nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n
```

we can use --packet-trace to track the individual packages and inspect their contents manually.

A Perl script named [rdp-sec-check.pl](https://github.com/CiscoCXSecurity/rdp-sec-check) can unauthentically identify the security settings of RDP servers based on the handshakes.

`# installation sudo cpan cpan[1]> install Encoding::BER  git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check ./rdp-sec-check.pl 10.129.201.248`

We can connect to RDP servers on Linux using xfreerdp, rdesktop, or Remmina and interact with the GUI of the server accordingly

```
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248
```

### WinRM (5985, 5986)

The Windows Remote Management ( WinRM) is a simple Windows integrated remote management protocol based on the command line.

- Windows Remote Shell (WinRS), which lets us execute arbitrary commands on the remote system. 
- Services like remote sessions using PowerShell and event log merging require WinRM.

Often we will see that only HTTP ( TCP 5985) is used instead of HTTPS ( TCP 5986)

```sh
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
```

If we want to find out whether one or more remote servers can be reached via WinRM:
- Powershell: use `Test-WsMan`
- Linux: use `evil-winrm`

```sh
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

### WMI

Windows Management Instrumentation ( WMI) allows read and write access to almost all settings on Windows systems. WMI is typically accessed via PowerShell, VBScript, or the Windows Management Instrumentation Console ( WMIC).

The initialization of the WMI communication always takes place on TCP port 135, and after the successful establishment of the connection, the communication is moved to a random port.

```sh
impacket-wmiexec Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
```

> [!important]
> Knowledge gained from installing these services and playing around with the configurations on our own Windows Server VM for gaining experience and developing the functional principle and the administrator's point of view cannot be replaced by reading manuals
