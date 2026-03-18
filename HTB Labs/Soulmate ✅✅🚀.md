# Soulmate   
## Finding User Flag   

Scan using Nmap, we found port 22 and 80.

![image](attachments/image_8.png)    

Add `soulmate.htb` to `/etc/hosts`.

Turns out there's nothing worth viewing in soulmate.htb, no XSS or File Upload attacks effective. 
Let's try to fuzz the vhosts

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://soulmate.htb/ -H 'Host: FUZZ.soulmate.htb' -fs 154
```

![image](attachments/image_2.png)    

Found `ftp`. Let's add `ftp.soulmate.htb`  to `/etc/hosts`  and go to the website.

![image](attachments/image_x.png)    

It's difficult to find the version only from the CrushFTP login page, but I found a working exploit for authentication bypass [https://github.com/f4dee-backup/CVE-2025-31161.git](https://github.com/f4dee-backup/CVE-2025-31161.git)    

![image](attachments/image_u.png)    

Yayy can log in   

![image](attachments/image_d.png)    

![image](attachments/image_h.png)    

So, tt has the version 11.3.0 build 2. Let's reset the users passwords as we got admin privilege here.

Based on the folder names, the most promising user is Ben, as in user manager settings, ben have are `webProd`  shares that may be used to upload php files (including shells!). Take note of the newly generated credentials.

`ben:988BFQ`    

![image](attachments/image_f.png)    

![image](attachments/image_0.png)    

Now we can upload file to the prod, let's upload the `webshell/`    

![image](attachments/image_3.png)    
 
 It works!   

![image](attachments/image_1f.png)    

Let's check the data inside, there's `config/config.php`  and `data/soulmate.db`  here.   

![image](attachments/image.png)    

![image](attachments/image_9.png)    

]Wow, interesting code snippet!   

![image](attachments/image_p.png)    

Let's login to `soulmate.htb` using `admin:Crush4dmin990`. This may be naive, but let's SSH using that credentials. NAhh it can't :(  should be obvious from the password string. 

Let's do another classic Linux reconnaissance commands. One of them that gives interesting info is `netstat -tulnp` .   

![image](attachments/image_6.png)    

There are 22, 80, and 4369. 22 is the default SSH port, 80 is the webpage itself, and.. 4369? Let's check on internet. Oh, it's the default port for Erlang stuffs or RabbitMQ stuffs (tbh I'm not familiar with it). But let's do more recon by using the `erl` command.

![image](attachments/image_12.png)    

It works. Let's find out the `erl`  command location   

![image](attachments/image_s.png)    

Not really.. helpful? Based on some searches, the configuration files is in `/usr/local/lib/erlang`.

![image](attachments/image_5.png)    

![image](attachments/image_m.png)    

Eh it seems there's nothing worthy. Let's check `lib/erlang\_login`  folder

![image](attachments/image_1d.png)    

![image](attachments/image_r.png)    

Wow there's another credentials! and based on the code, it seems that it's for… ssh stuffs? Interesting. `ben:HouseH0ldings998`   

![image](attachments/image_c.png)

Found it!
## Finding Root Flag   

In home directory, there's `1299.sh` script.   

![image](attachments/image_t.png)    

The code are indeed suspicious, but when we check what OS we're in…   

![image](attachments/image_v.png)    

ubunyu!  

![image](attachments/image_14.png)    

I'm finding that using linpeas bears no fruit, so let's do recon some more. In our previous `netstat -tuln`  command, there's another port that opens only in localhost (`127.0.0.1:2222` ). Let's try to connect using same credentials.

![image](attachments/image_i.png)    

Oh? TIL Erlang has its own ssh runner, and in help command, there's `os:cmd()`  command…   

![image](attachments/image_y.png)    
