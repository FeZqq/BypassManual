# Rate Limiting Bypass

### >_ Introduction
Rate limiting is a fundamental security mechanism used by web applications and APIs to control the number of requests a client can make within a certain time frame, preventing abuse, brute-force attacks, and resource exhaustion. It is typically enforced based on identifiers such as IP addresses, user accounts, or API keys. However, improper implementation or assumptions about client identity can introduce weaknesses. Attackers can exploit these weaknesses through techniques such as header manipulation, proxy chaining, or distributed requests to bypass rate limiting controls, effectively sending far more requests than intended. Understanding both the purpose of rate limiting and the methods of bypassing it is crucial for designing robust systems and performing effective security assessments. 


### >_ Rate Limiting Control on DNS Server
In some systems, rate limiting is applied based on the requested domain name rather than the client IP. If the server does not enforce strict Host header checks, sending requests directly to the server’s IP address can bypass domain-based rate limiting. Each request to the IP may be treated as a new client, effectively circumventing limits tied to the domain.

Limitations:

- If the server validates the Host header, sending the domain in the header while requesting the IP may still trigger the rate limit.

- Direct IP access may cause TLS/HTTPS certificate mismatches if the certificate is issued for the domain.


### >_ Internal Network via Proxy Headers
Rate limiting is usually applied based on an IP address, but some applications behind a proxy or load balancer rely on HTTP headers like X-Forwarded-For instead of the real client IP. If the server accepts this header without validation and treats internal network IPs as “trusted,” an attacker can set the header to an internal IP, making requests appear to come from the internal network and bypass rate limiting, allowing unlimited requests to the same endpoint. Here are the most common proxy headers:


```
Forwarded: for=127.0.0.1; proto=http; by=127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Host: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Forwarded: 127.0.0.1
Forwarded-For: 127.0.0.1
X-Original-URL: 127.0.0.1
Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
Cluster-Client-IP: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
```

### >_ Fake Ip via Proxy Headers
Proxy headers, such as X-Forwarded-For, X-Forwarded-Host, and Forwarded, convey client information through intermediate proxies, load balancers, or CDNs. In systems where rate limiting is enforced based on the client IP, these headers can be manipulated to specify arbitrary IP addresses for each request. By rotating or spoofing the X-Forwarded-For value, the server may treat each request as originating from a distinct client, effectively bypassing IP-based rate limiting controls. This technique relies on the backend trusting the forwarded headers without proper validation or normalization. Here are the most common proxy headers:

```
Forwarded: for=192.0.2.60; proto=http; by=203.0.113.43
X-Forwarded-For:
X-Custom-IP-Authorization:
X-Originating-IP:
X-Forwarded-For:
X-Remote-IP:
X-Client-IP:
X-Host:
X-Forwarded-Host:
X-ProxyUser-Ip:
X-Remote-Addr:
X-Forwarded:
Forwarded-For:
X-Original-URL:
Client-IP:
True-Client-IP:
Cluster-Client-IP:
X-ProxyUser-Ip:
```

### >_ Bypassing Rate Limiting By Deleting Cookie
Some systems implement rate limiting based on the user’s cookie or session identifier. If an endpoint does not strictly require the cookie to process a request, omitting or deleting the cookie can cause the server to treat the request as originating from a new or unauthenticated user. As a result, the rate limiting logic tied to the original cookie is effectively bypassed, allowing repeated requests without triggering the limit. This behavior occurs because the backend associates the rate limiting state with the presence of the cookie, and requests lacking the cookie are not recognized as belonging to the same client session.


### >_ Bypassing Rate Limiting via Parameter Obfuscation with Non-Printable Characters
If request parameters are not properly normalized or validated, appending special or non-printable characters (e.g., %00 null byte, %09 tab, %0d%0a CRLF) to a parameter can create multiple representations of the same value. The backend processes all variations as identical, but intermediate filters may treat each variation as distinct. By exploiting this, one can effectively bypass checks that rely on exact parameter matching, since the system interprets each modified parameter as a new input while still processing it as the original. 


### >_ Bypassing rate limits via race conditions
If you send multiple requests in parallel using Burp’s Turbo Intruder, the server’s rate-limiting mechanism may fail to block all of them. For example, if the limit is applied per IP or per session, sending requests simultaneously with slight timing differences or without certain cookies can allow more requests than intended, effectively bypassing the restriction. This shows that naive rate-limiting can be evaded simply by increasing concurrency or altering headers that the server uses to track request counts. (https://portswigger.net/web-security/race-conditions/lab-race-conditions-bypassing-rate-limits)

### >_ Referances
- https://hackerone.com/reports/2627062

- https://securitylabs.datadoghq.com/articles/aws-console-rate-limit-bypass/

- https://www.certuscyber.com/insights/rate-limiting-protect-network/

- https://medium.com/@mrxdevil404/how-i-bypassed-rate-limits-to-trigger-account-takeovers-sms-flooding-and-impersonation-9ed42ca1501f

- https://jeetpal2007.medium.com/how-i-approach-account-takeover-due-to-no-rate-limit-on-otp-10a7fe056184

- https://infosecwriteups.com/unique-rate-limit-bypass-worth-1800-6e2947c7d972

- https://portswigger.net/web-security/race-conditions/lab-race-conditions-bypassing-rate-limits