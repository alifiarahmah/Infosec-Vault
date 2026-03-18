# Web Fuzzing & Attacking Web Applications with Ffuf

## Directory Fuzzing
```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```

## Extension fuzzing
```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

## Page fuzzing
```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

## Recursive scanning
```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

## DNS
```sh
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

## VHost
```sh
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

## Parameter & Value fuzzing

### GET
```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:30239/admin/admin.php?FUZZ=key -fs xxx
```

### POST
Tip: In PHP, "POST" data "content-type" can only accept "application/x-www-form-urlencoded". So, we can set that in "ffuf" with "-H 'Content-Type: application/x-www-form-urlencoded'".

```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
untuk
```sh
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```

```sh
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -X POST -u https://69ef56304f6822c821986b332fe7c1ae.ctf.hacker101.com/login -d 'username=FUZZ\&password=pass' -fs 329
```

ffuf with hashed value of wordlist
```sh
ffuf -w <(cat /usr/share/seclists/Usernames/top-usernames-shortlist.txt | while read line; do echo -n "$line" | md5sum | cut -d' ' -f1; done) -u http://94.237.57.1:54814/skills/ -H "Cookie: cookie=FUZZ"
```

## Value Fuzzing

easiest to type id
```sh
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```

vhost
```sh
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

# Fuzz value using wenum

Setup
```sh
pipx install git+https://github.com/WebFuzzForge/wenum
pipx runpip wenum install setuptools
```

```sh
wenum -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404 -u "http://IP:PORT/get.php?x=FUZZ"
```

```sh
ffuf -u http://IP:PORT/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v
```

## Vhost and subdomain fuzzing

gobuster
```sh
gobuster dns -domain inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## Filtering fuzzing output
  -fc              Filter HTTP status codes from response. Comma separated list of codes and ranges
  -fl              Filter by amount of lines in response. Comma separated list of line counts and ranges
  -fr              Filter regexp
  -fs              Filter HTTP response size. Comma separated list of sizes and ranges
  -fw              Filter by amount of words in response. Comma separated list of word counts and ranges

## Validating findings

## Web APIs

## Identifying endpoints