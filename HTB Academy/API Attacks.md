---
tags:
  - CWES
---
# API Attacks

Web APIs can be built using various architectural styles, including `REST`, `SOAP`, `GraphQL`, and `gRPC`, each with its own strengths and use cases:
- [Representational State Transfer](https://ics.uci.edu/~fielding/pubs/dissertation/top.htm) (`REST`) is the most popular API style. It uses a `client-server` model where clients make requests to resources on a server using standard HTTP methods (`GET`, `POST`, `PUT`, `DELETE`). `RESTful` APIs are stateless, meaning each request contains all necessary information for the server to process it, and responses are typically serialized as JSON or XML.
- [Simple Object Access Protocol](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/) (`SOAP`) uses XML for message exchange between systems. `SOAP` APIs are highly standardized and offer comprehensive features for security, transactions, and error handling, but they are generally more complex to implement and use than `RESTful` APIs.
- [GraphQL](https://graphql.org/) is an alternative style that provides a more flexible and efficient way to fetch and update data. Instead of returning a fixed set of fields for each resource, `GraphQL` allows clients to specify exactly what data they need, reducing over-fetching and under-fetching of data. `GraphQL` APIs use a single endpoint and a strongly-typed query language to retrieve data.
- [gRPC](https://grpc.io/) is a newer style that uses [Protocol Buffers](https://protobuf.dev/) for message serialization, providing a high-performance, efficient way to communicate between systems. `gRPC` APIs can be developed in a variety of programming languages and are particularly useful for microservices and distributed systems.

## OWASP API Security Top 10

### 1. Broken Object Level Authorization
Failing to properly and securely verify that a user has ownership and permission to view a specific resource. 

### 2. Broken Authentication
We also can use the password reset mechanism and fuzz the OTP

#### Exercise
Use the `api/v1/authentication/customers/passwords/resets/email-otps` to that email, then fuzz with 4-digit token

```sh
seq -w 0 9999 > tokens.txt # can also use from seclists
ffuf -X POST -w ./tokens.txt -u http://94.237.57.211:40872/api/v1/authentication/customers/passwords/resets --data '{"Email": "MasonJenkins@ymail.com", "OTP": "FUZZ", "NewPassword": "HTBPentester3"}' -H 'Content-Type: application/json' -fs 23
```

Note: Don't forget `-H 'Content-Type: application/json'`
### 3. Broken Object Property Level Authorization
- Excessive Data Exposure: Reveals sensitive data to authorized users that are not supposed to access
- Mass Assignment: Permits authorized users to manipulate sensitive object properties beyond (supposed to be) authorized scope

### 4. Unrestricted Resource Consumption
- Upload data superbesar
- Sesering mungkin call API yg butuh jasa 3rd party

### 5. Broken Function Level Authorization
have access to function outside their assigned roles

### 6. Unrestricted Access to Sensitive Business Flows
An API endpoint is vulnerable if it exposes a sensitive business flow without appropriately restricting access to it

### 7. Server Side Request Forgery
A web API is vulnerable to `Server-Side Request Forgery` (`SSRF`) (also known as `Cross-Site Port Attack` (`XPSA`)) if it uses user-controlled input to fetch remote or local resources without validation. SSRF flaws occur when an API fetches a remote resource without validating the user-supplied URL. This allows an attacker to coerce the application to send a crafted request to an unexpected destination (especially local ones), bypassing firewalls or VPNs.

### 8. Security Misconfiguration
- Improper neutralization of special elements --> SQLi
- CORS * HTTP Headers

### 9. Improper Inventory Management
Inadequate versioning, can introduce security misconfigurations and increase the attack surface. This can manifest in various ways, such as outdated or incompatible API versions remaining accessible, creating potential entry points for unauthorized users.

### 10. Unsafe Consumption of APIs
Developers may blindly trust data received from third-party APIs, especially when provided by reputable organizations, leading to relaxed security measures, particularly in input validation and data sanitization.

Several critical vulnerabilities can arise from API-to-API communication:

1. `Insecure Data Transmission`: APIs communicating over unencrypted channels expose sensitive data to interception, compromising confidentiality and integrity.
2. `Inadequate Data Validation`: Failing to properly validate and sanitize data received from external APIs before processing or forwarding it to downstream components can lead to injection attacks, data corruption, or even remote code execution.
3. `Weak Authentication`: Neglecting to implement robust authentication methods when communicating with other APIs can result in unauthorized access to sensitive data or critical functionality.
4. `Insufficient Rate-Limiting`: An API can overwhelm another API by sending a continuous surge of requests, potentially leading to denial-of-service.
5. `Inadequate Monitoring`: Insufficient monitoring of API-to-API interactions can make it difficult to detect and respond to security incidents promptly.