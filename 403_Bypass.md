# 403 Bypass
<img width="2858" height="1502" alt="403-status-code" src="https://github.com/user-attachments/assets/2d1d55d6-ac1f-4c3c-a329-5a1b12575df5" />

### Introduction
The 403 status code indicates that access to the resource is not permitted. In this case, the desired panel, resource, document, etc. can be accessed by changing the code to 200. The steps to follow are listed below. In all examples on this cheat sheet, the endpoint to be accessed with the 200 status code is "api/v2/admin/"

### >_ Proxy/Forwarded Headers
Certain HTTP headers like "X-Forwarded-For" can be exploited to bypass 403 Forbidden restrictions because some web servers or applications rely on these headers to determine the client’s real IP address. If access control is implemented based on IP whitelisting, an attacker can inject a trusted IP into the X-Forwarded-For header, making the server believe the request originates from an allowed source. This can effectively circumvent IP-based restrictions without altering the source IP at the network level. Essentially, the server trusts the header more than the actual connection, creating an opportunity for unauthorized access. Headers list avaible below:

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
```

### >_ Path Normalization
When a request hits a server, there are usually two main steps:

Access Control Check: The server first looks at the requested path and decides whether the client is allowed to access it. For example, it might check if /admin/secret.txt is forbidden.

Resource Resolution (Normalization): After the access control, the server “normalizes” the path—resolving . and .., removing repeated slashes, decoding URL encodings, etc.—to figure out the actual file or resource on disk it should serve.

The vulnerability arises when the access control check happens before normalization, or when the check interprets the path differently than the resource resolution does. An attacker can craft a path like ```/admin/../public/data.txt```. The access control sees /admin/../public/data.txt and thinks it’s under /admin (maybe forbidden) or outside protected paths, but after normalization, the server reads /public/data.txt—thus bypassing restrictions and accessing resources that should have been protected.

Note: Use curl to do this trick. Because browsers convert your url to normalized url.


Example:
```/api/v2/./admin/ >> /api/v2/admin/```

Other Payloads:
```
/api/v2/admin/
/api/v2/admin/.
/api/v2//admin//
/api/v2/./admin/./
/api/v2/./admin/..
/api/v2/;/admin
/api/v2/.;/admin
/api/v2//;//admin
/api/v2/admin..;/
/api/v2/%2e/admin
/api/v2/%252e/admin
/api/v2/%ef%bc%8fadmin
```

### >_ API Versioning
API versioning is typically used to separate endpoints with paths like /v1/, /v2/, and so on. However, in some cases, access control is enforced only under a specific version while other versions may be overlooked. For example, while a request to /api/v2/users may return a 403, an attacker could send the same request to /api/v1/users or /api/v3/users and unexpectedly gain access. This issue arises from inconsistent enforcement of security controls across versions. As a result, an attacker can manipulate the API versioning structure to bypass the 403 restriction and access data they would normally be unauthorized to view.

Example:
```
/api/v2/admin/ >> fail
/api/v1/admin/ >> success
```

### >_ Waybackurl technique
Using tools such as waybackurls or the Wayback Machine, it is possible to retrieve historical versions of an application and list old API endpoints or paths. If the current version of the application returns a 403 Forbidden error for a specific resource, archived endpoints from older versions may still be active and map to the same backend functionality. In such a case, sending the request to the legacy endpoint instead of the restricted one may result in a successful 200 OK response. This situation occurs when legacy endpoints are not properly disabled or deprecated, allowing access to resources that should be restricted.



### >_ Encoding the char
Encoding tricks exploit the fact that different layers of a system (such as WAFs, reverse proxies, and the application server) may interpret encoded characters differently. For example, the word admin can be altered by encoding the last letter n once (/admi%6e) or even twice (/admi%256e). A security filter might not recognize /admi%256e as admin because it only decodes once, but the application server could decode it again, resolving it to admin. This discrepancy allows attackers to bypass access controls by hiding forbidden keywords or paths behind single or double encoding. In short, when the security layer and the application decode input differently, encoding becomes a powerful bypass technique.

```
/api/v2/admin/ >> fail
/api/v2/admi%6E/ >> success >> if it is fail, use double encoding:
/api/v2/admi%256E/ >> success
```

### >_ Using CNAME
A CNAME (Canonical Name) record in DNS acts as an alias, pointing one domain to another. For example, if panel.example.com has a CNAME pointing to login.backend.com, it means that panel.example.com is just an alias and the real resource is login.backend.com. Using a tool like dig, you can reveal this mapping:

dig panel.example.com CNAME

This will show that panel.example.com actually resolves to login.backend.com. The bypass logic comes into play when security restrictions, such as a 403 Forbidden, are applied only on the alias domain (panel.example.com). By accessing the underlying target domain (login.backend.com) directly, an attacker can reach the same resource without hitting the access control, because the restriction was not enforced on the canonical domain. In short, a CNAME points a domain to another, and discovering it can allow bypassing domain-specific access controls.

### >_ Method Change
Method change bypass is a technique used to circumvent HTTP method–based access controls in web applications or APIs. For example, an endpoint might be restricted to only accept GET or POST requests, returning a 403 Forbidden for other methods. However, some servers or applications may process other HTTP methods without proper normalization or filtering on the backend. For instance, an /admin endpoint may allow only GET requests, but sending a HEAD, TRACE, or OPTIONS request might still be processed as if it were a GET, bypassing the restriction. The logic is that the access control enforces method-based filtering, but the backend handles requests differently or more flexibly, allowing attackers to bypass the restriction by changing the HTTP method.


### >_ Referances
- https://www.darkanonsys.com/blogs/E1XTCpTgVy7V5njIXVB9
- https://www.acunetix.com/blog/articles/a-fresh-look-on-reverse-proxy-related-attacks/
- https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf
- https://github.com/zaproxy/zap-extensions/blob/main/addOns/ascanrulesBeta/src/main/java/org/zaproxy/zap/extension/ascanrulesBeta/ForbiddenBypassScanRule.java
- https://medium.com/infosecmatrix/mastering-403-bypass-techniques-a-penetration-testers-guide-f3a1cb16b9a3
- https://www.youtube.com/watch?v=PvpXRBor-Jw






