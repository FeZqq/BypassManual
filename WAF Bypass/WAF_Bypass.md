# Web Application Firewall Bypass
![waf_bypass](https://github.com/user-attachments/assets/abb51831-0a90-4ec1-9cbe-1d646a76e07f)

### >_ Introduction
Web Application Firewalls (WAFs) are a critical layer of defense designed to detect and block malicious web traffic before it reaches an application. However, like any security mechanism, they are not impervious to sophisticated techniques. WAF bypass involves understanding how these systems inspect and filter requests, identifying gaps or weaknesses, and crafting payloads that evade detection. By studying these methods, security professionals, penetration testers, and bug bounty hunters can better evaluate application security, uncover hidden vulnerabilities, and strengthen defenses against potential attacks.

### >_ Types of WAF Bypassing
- Pre-processor Exploitation: Make WAF skip input validation
- Impedance Mismatch: WAF interprets input differently than back end
- Rule Set Bypassing: Use Payloads that are not detected by the WAF

----------------------------------------------------------------------
# Pre-Proccesor Exploitation

### >_ Bypassing Parameter Verification
Pre-processor Exploitation is a technique to bypass filters by exploiting incompatibilities between how the WAF interprets the request and how the backend interprets it. And there are some cases below:

- In PHP language, the spaces are removed by the PHP backend and if WAF was not configurated correctly, it considers the request as safe.
Example:
```
http://www.website.com/products.php?%20productid=select 1,2,3
```
When sending a request like this the WAF sees the parameter name as %20productid (with a leading space) and may ignore it, treating the request as harmless. However, PHP automatically normalizes the input by removing the leading space, recognizing it as productid=select 1,2,3. This discrepancy allows the malicious payload to bypass WAF inspection and still be executed by the backend, which is a classic case of pre-processor exploitation.



- In ASP, ASP removes % character that is not followed by two hexadecimal digits so if WAF was not configurated correctly it considers the request as safe.

Example:
```
http://www.website.com/products.aspx?%productid=select 1,2,3
```

### >_ Proxy/Forwarded Headers
- Some WAFs are configured to trust requests coming from internal IP addresses, meaning input validation is not applied to them. If the WAF determines the client’s IP based on HTTP headers that can be manipulated by the user, such as X-Originating-IP, X-Forwarded-For, X-Remote-IP, or X-Remote-Addr, an attacker can forge these headers to appear as though the request originates from a trusted internal source, effectively bypassing WAF protections.
Headers list avaible below:
```
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
Forwarded: for=192.0.2.60; proto=http; by=203.0.113.43
```


### >_ Malformed HTTP Method
- A WAF that only inspects standard GET and POST requests can be bypassed if the web server is misconfigured to accept uncommon or malformed HTTP methods. For example, instead of sending a normal "GET /index.php HTTP/1.1", an attacker could send "HEAD /index.php HTTP/1.1" or even a non-standard method like "GETS /index.php HTTP/1.1." If the WAF does not analyze these methods but the server still processes them, the malicious request may reach the backend unfiltered.

### >_ WAF Overloading

- A WAF can be bypassed by overloading, where a high volume of malicious requests is sent in a short period. Some WAFs, especially embedded ones, may skip input validation under heavy load to maintain performance, allowing some payloads to reach the backend unfiltered.

- To simulate or trigger this overload, attackers can manipulate the "Content-Length" header. By setting a "Content-Length" that does not match the actual body size, the WAF may misinterpret or partially read the request, while the backend still processes it fully. This approach effectively “tricks” the WAF into skipping validation without sending an actual flood of requests.	

----------------------------------------------------------------------
# Impedance Mismatch

### >_ HTTP Parameter Pollution
This technique involves sending the same parameter twice in a request, where one carries a normal value and the other contains a malicious payload. the WAF may inspect the benign parameter and consider the request safe, while the backend server processes the parameter containing the payload.

Examples:

Url = http://example.com/users?userId=1&userId=2

| Back end | Behavior               | Processed   |
|----------|------------------------|-------------|
| ASP.NET  | Concatenate with comma | userId=1,2  |
| JSP      | First Occurrence       | userId=1    |
| PHP      | Last Occurrence        | userId=2    |


### >_ Impedance Mismatch Example
In an impedance mismatch scenario, a payload can be divided into two or more parameters so that the WAF interprets them as separate values and fails to detect the malicious input. However, when processed by an ASP.NET backend, the values are concatenated into a single parameter, effectively reconstructing the payload and allowing it to bypass the WAF.

Example:

``` 
?userId=Select&userId=*&userId=from&userId=table
```

This payload is equal to:

```
?userId=Select * From Table
```


### >_ HTTP Parameter Fragmention
HTTP Parameter Fragmentation is a technique where an attacker splits a malicious payload across multiple HTTP parameters instead of placing it in a single parameter. This is often used to bypass WAFs or other security mechanisms that inspect individual parameters for suspicious patterns. The web application backend then concatenates these parameters when constructing SQL queries, effectively reassembling the malicious payload. By fragmenting the payload, the attacker can execute SQL Injection or other attacks while evading detection.

Example:

- Backend Codes:
```
sql = "SELECT * FROM table WHERE uid = " + $_GET['uid'] + " and pid = " + $_GET['pid'];
```

- Payload:
```
http://www.website.com/index.php?uid=1+union/*&pid=*/select 1,2,3
```

- Result:
```
SELECT * FROM table WHERE uid = 1 union/* and pid = */select 1,2,3
```


### >_ Double URL Encoding
In some cases, a WAF normalizes URL-encoded characters into ASCII text only once. Therefore, if the payload is encoded twice, the WAF may not detect any malicious code in the parameter.

Example:

- Double Encode:
```
e >> %65 >> %25%36%35
```

- And Payload will be:

```
S%25%36%35lect * from table
```

----------------------------------------------------------------------
# Rule Set Bypassing

There are two methods here:
- Brute force by enumerating payloads
- Reverse-engineer the WAFs rule set


### >_ Brute force by enumerating payloads
Brute forcing a WAF involves systematically sending a large number of different payloads to identify which ones are blocked and which are allowed. By testing many variations of malicious input, an attacker can discover payloads that bypass the WAF and reach the backend application. This method does not require understanding the WAF’s internal rules, only careful trial and error.

Example:
An attacker wants to perform a SQL injection but the WAF blocks the keyword UNION. They try variations like "UN/**/ION", "U%4EION", or "UNI+ON" in requests until one variation successfully bypasses the WAF.


### >_ Reverse-Engineer the WAF’s Rule Set
Reverse-engineering a WAF involves analyzing how the firewall processes input to understand its detection rules. By studying which patterns are blocked, which characters are normalized, and how payloads are parsed, an attacker can craft input that intentionally avoids detection. This is a more sophisticated method than brute force and allows targeted bypasses.

Example:
Suppose a WAF blocks SELECT statements but only when written in uppercase. By analyzing responses, an attacker discovers that lowercase keywords are not filtered. They can then send "select * from users" instead of "SELECT * FROM users", successfully bypassing the WAF.
----------------------------------------------------------------------

### >_ Referances
- https://owasp.org/www-chapter-frankfurt/assets/slides/21_OWASP_Frankfurt_Stammtisch.pdf

- https://claroty.com/team82/research/js-on-security-off-abusing-json-based-sql-to-bypass-waf

- https://www.ukusormus.com/bypassing-cloudflare-waf-xss-via-sql-injection/

- https://claroty.com/team82/research/js-on-security-off-abusing-json-based-sql-to-bypass-wafs

- https://www.pmnh.site/post/writeup_spring_el_waf_bypass/

- https://portswigger.net/research/bypassing-wafs-with-the-phantom-version-cookie



