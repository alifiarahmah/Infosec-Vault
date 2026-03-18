# BSCP Cheatsheets

## Server-Side

### [[SQL Injection]]



### [[Authentication]]


### [[Path Traversal]]
### [[Command Injection]]
### [[Business Logic Vulnerabilities]]
### [[Information Disclosure]]
### [[Access Control]]
### [[File Upload Vulnerabilities]]
### [[Race Conditions]]
### [[SSRF]]
### [[XXE Injection]]
### [[NoSQL Injection]]
### [[API Testing]]
### [[Web Cache Deception]]

## Client-Side
### [[XSS]]
### [[CSRF]]
### [[CORS]]
### [[Clickjacking]]
### [[DOM-Based Vulnerabilities]]
### [[WebSockets]]

## Advanced
### [[Insecure Deserialization]]
### [[Web LLM Attacks]]
### [[GraphQL API Vulnerabilities]]
### [[SSTI]]
### [[Web Cache Poisoning]]
### [[HTTP Host Header Attacks]]
### [[HTTP Request Smuggling]]
### [[OAuth]]
### [[JWT]]
### [[Prototype Pollution]]
### [[Essential Skills]]

#### Obfuscating Using Encodings

- 
- URL encoding

#### Using Burp Scanner

- Scan specific request
  Request > HTTP History  > klik kanan di salah 1 req > Do active scan
- Scan custom insertion points
  Highlight value di Request tab > Scan selected insertion point
- Scan non-standard data structures
  Scan single part of parameter `user=04-carlos` highlight di `carlos`