# Authorization bypass

### >_ Introduction
Authorization bypass can occur in two ways:

	- Horizontal authorization bypass: The user remains within their own role but gains access to resources that belong to other users.

	- Vertical authorization bypass: A lower-privileged user gains access to functions or resources intended only for higher-privileged users.

In short, horizontal bypass allows unauthorized access between users of the same privilege level, while vertical bypass lets a user act with higher privileges than they should have.

### >_ Vertical Authorization

If you have both user and admin account, test admin functions with user session value. For example:

	```
		GET /admin/dashboard
		Cookie: session=user_token
	```
If the response still returns admin content while using a normal user session, the application is vulnerable to vertical authorization bypass.
Note: Sometimes, developers perform authorization validation at the GUI level only, and leave the functions without authorization validation, this potentially resulting in a vulnerability.

If you don't have an admin or any high authority account you have to find these endpoints. There are many techniques below:

- Directory Fuzzing:
	Directory fuzzing is a web application security testing technique used to discover hidden or unlinked directories and files by systematically sending a predefined set of potential paths (wordlists) to the target server and analyzing the HTTP responses. By leveraging tools such as Gobuster, Dirsearch, or Feroxbuster, security testers can identify accessible resources (e.g., 200 OK), restricted areas (e.g., 403 Forbidden), or misconfigured directories that may expose sensitive information. This method is essentially a form of dictionary-based brute forcing tailored for web paths and is widely employed in penetration testing and vulnerability assessments to uncover attack surfaces that are not visible through standard browsing.

- Endpoint Discovery with Katana (project discovery's tool: https://github.com/projectdiscovery/katana):
	In web security testing, endpoint discovery is the technique of systematically identifying all accessible URLs, parameters, and resources of a target application, including hidden or undocumented paths, by analyzing site structure, sitemaps, and embedded links. Katana automates this process by crawling the application, parsing HTML and JavaScript, reading sitemaps, and extracting potential endpoints, effectively streamlining reconnaissance and providing a comprehensive view of the application’s attack surface.

- Waybackurls (https://github.com/tomnomnom/waybackurls):
	Waybackurls is a reconnaissance tool used in web security testing to gather historical URLs of a target domain from public archives such as the Wayback Machine. By querying these archives, it collects URLs that were previously live, including potentially forgotten or unlinked endpoints, old directories, and files. This allows security testers to identify legacy functionality, hidden resources, or past configurations that could still be exploitable, providing valuable insight into the application’s historical attack surface.

- main.js file on next.js, node.js, javascript projects:
	In some cases, main.js file (it can be found on source code the web html page) has obfuscated endpoints. And these are steps how to find:
	1- The main.js file is obtained from the source code
	2- Search "https://", "/api", "/login" or any other endpoint on the file. If you find a variable like "baseUrl", "apiUrl", "mainUrl" then search this variable.
	3- When you search this variable, you see a combination like "baseUrl + /api/getallusers". Then search for the variable to which this combination is assigned. For example, let's say the variable is getAll. Search getAll variable for discover the http method. You will see a function like .getRequest(getAll) or .postRequest(getAll,body).
	4- Now you can create the request to critical endpoints and try authentication bypass on these endpoints.
	

#### >_ Testing for Special Request Header Handling
Some web applications support non-standard HTTP headers like X-Original-URL or X-Rewrite-URL, which allow the request URL to be overridden by the value specified in the header. This can be abused when the application is behind an access control mechanism that restricts certain paths, such as /admin or /console. By manipulating these headers, an attacker may bypass URL-based access restrictions. For example, sending a request like:

	```
		GET / HTTP/1.1
		Host: example.com
		X-Original-URL: /admin
	```
could cause the server to process the request as if it were targeting /admin, potentially exposing restricted functionality.

#### >_ Other Headers to Consider
Often admin panels or administrative related bits of functionality are only accessible to clients on local networks, therefore it may be possible to abuse various proxy or forwarding related HTTP headers to gain access. Some headers and values to test with are:

    Headers:
        X-Forwarded-For
        X-Forward-For
        X-Remote-IP
        X-Originating-IP
        X-Remote-Addr
        X-Client-IP
    Values
        127.0.0.1 (or anything in the 127.0.0.0/8 or ::1/128 address spaces)
        localhost
        Any RFC1918 address:
            10.0.0.0/8
            172.16.0.0/12
            192.168.0.0/16
        Link local addresses: 169.254.0.0/16

Note: Including a port element along with the address or hostname may also help bypass edge protections such as web application firewalls, etc. For example: 127.0.0.4:80, 127.0.0.4:443, 127.0.0.4:43982

#### >_ Authorization Bypass via Information Disclouse
In some case, trace method is allowed by the enpoint. With this method the header can be discovered to bypass the authorization. In this PortSwigger example, accessing the /admin endpoint with the GET method returns a 401 Unauthorized response, while the TRACE method reflects our own IP address in the X-Custom-Ip-Authorization header. By sending a GET request to /admin and including this header set to 127.0.0.1, we are able to bypass authorization and gain access to the admin panel.

<photo1>
<photo2> 

#### >_ Authorization bypass due to cache misconfiguration
Authorization bypass due to cache misconfiguration occurs when a server temporarily stores responses to requests, including sensitive data, in a cache that can be accessed by other users. If access controls are not properly enforced on cached responses, lower-privileged users may receive data intended for higher-privileged accounts. This creates a short time window during which unauthorized users can exploit the cached data to bypass restrictions, highlighting the importance of properly handling sensitive responses in caching mechanisms. (https://rikeshbaniya.medium.com/authorization-bypass-due-to-cache-misconfiguration-fde8b2332d2d)

#### >_ Exploiting Authorization Bypass via Known CVEs in Web Frameworks
In some web applications, authorization bypass vulnerabilities can be exploited through known CVEs, where improper access control or misconfigured middleware allows attackers to reach protected endpoints. For example, in frameworks like Next.js, poorly implemented middleware that fails to validate user roles or session tokens before granting access can be bypassed if a CVE exists, enabling attackers to perform actions reserved for higher-privileged users. By leveraging these publicly disclosed vulnerabilities, an attacker can systematically exploit weaknesses in authentication or authorization logic, bypassing intended restrictions without needing direct access to credentials.


### >_ Horizontal Authorization

For each role:

    - Register or generate two users with identical privileges.
    - Establish and keep two different sessions active (one for each user).
    - For every request, change the relevant parameters and the session identifier from token one to token two and diagnose the responses for each token.
    - An application will be considered vulnerable if the responses are the same, contain same private data or indicate successful operation on other users’ resource or data.

Example:
	```
	GET /user/profile?id=123
	Cookie: session=token_user1
	```

Change to:
	```
	GET /user/profile?id=123
	Cookie: session=token_user2
	```

If the response still returns user 123’s private data, the application is vulnerable to horizontal authorization bypass (in other words, IDOR).




### >_ Resources
- https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema
- https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-authentication-bypass
- https://rikeshbaniya.medium.com/authorization-bypass-due-to-cache-misconfiguration-fde8b2332d2d
- https://projectdiscovery.io/blog/atlassian-confluence-auth-bypass
- https://medium.com/@manikandannarayanaswamy/how-i-bypassed-authorization-using-expired-jwt-7eef4feca9ca
- https://www.komodosec.com/post/google-groups-authorization-bypass
- https://hackerone.com/reports/205000