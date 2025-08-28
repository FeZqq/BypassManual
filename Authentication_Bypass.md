# Authentication Bypass
<img width="1536" height="1024" alt="abp" src="https://github.com/user-attachments/assets/807999fd-e17f-4b0f-8dcc-d558c257b5f9" />


### >_ Introduction
Authentication bypass is a security vulnerability that allows an attacker to gain unauthorized access to an application or system by evading or manipulating the authentication mechanism. Instead of providing valid credentials, the attacker exploits flaws such as weak session handling, logic errors, misconfigurations, or improper input validation to directly access restricted areas or assume another user’s identity.Below are several methods for authentication bypass.


### >_ Direct Page Request
If authentication is applied only in the login panel and there is no control on the page after the login process, you can try to access the page directly after the login.
 
For Example:
	- ```"/login"``` >>  authentication endpoint
	- ```"/admin/dashboard"``` >> The endpoint to be accessed after authentication.
	Directly visit on ```"/admin/dashboard"``` endpoint and try to access without any authentication.
	
### >_ Response Manupulation
This method is similar to the previous method. But this time the endpoint that comes after authentication, comes itself with or without any token/cookie. 

On Burp Proxy, enable the intercept and capture the request. Then right-click the request, choose 'Do intercept' > 'Response to this request' from the menu, and modify the response from 'fail' to 'success'.

Example:

	Request:
	```
	{
		"username":"admin",
		"password":"pass"
	}
	```

	Response:
	```
	{
		"login_status":false
	}
	```

	Then change "false" to "true" and access the endpoint that comes after authentication.

SSO note: A similar approach can be applied to SSO systems if the application is misconfigured. For example, if a redirect to the SSO leaks internal responses, it is possible to manipulate the HTTP response—change 302 Found to 200 OK and remove the Location header—allowing access to the application without completing the SSO login.

Note: Response manipulation doesn't always constitute a vulnerability. For example, in this example, if the next panel is accessed after authentication, but no authorization is gained on the panel, it isn't categorized as a vulnerability.

### >_ JSON Manupulation
Some systems that use "application/x-www-form-urlencoded" data accept "application/json" data instead. At this point, these can be try:

- Login with admin or any other account
	```
	{
		"username":"admin",
		"password":true
	}
	```
- Login with "true:true":
	```
	{
		"username":true,
		"password":true
	}
	```
- Login just username:
	```
	{
		"username":"admin"
	}
	```
- Probably not login but it can be information disclouse on error message:

	```
	{
		"username":null,
		"password":null
	}
	```
	
	```
	{
		"username":null
	}
	```

	```
	{
		"password":null
	}
	```

	```
	{
		"username":false,
		"password":false
	}
	```
	
### >_ Refresh Token Endpoint Misconfiguration
1-Login creates a token: When a user logs in with valid credentials, the application generates a Bearer Authentication token. This token is used to access other parts of the application.

2-Token expiration: The token expires after a certain time.

3-Token refresh request: Just before expiration, the application sends a request to the /refresh/tokenlogin endpoint. This request includes:

4-The current token in the Authorization header

5-The username in the HTTP body

6-Vulnerability found: If the Authorization header is removed and the username in the body is changed, the server creates a new valid token for the supplied username.

Impact: This allows an anonymous user to generate a valid authentication token for any username without knowing the original user's credentials.

Conclusion: The token refresh mechanism is incorrectly implemented, making authentication bypass possible.

### >_ Misconfigurated SSO
Single Sign-On (SSO) is a system that allows users to authenticate once and gain access to multiple applications without logging in repeatedly. In one case, an internal application used Microsoft SSO for authentication, redirecting users to the SSO page as expected. However, some JavaScript files were accessible without authentication, and the backend was misconfigured: it did not verify whether a JWT token was generated specifically for that application, only that the token’s signature was valid. As a result, any valid JWT token, including publicly available example tokens from Microsoft’s website, could be accepted by the application, allowing unauthorized access to critical endpoints and sensitive functionality, despite SSO being in place. 

### >_ Parameter Manipulation
Web applications and platforms must be properly configured to prevent unauthorized access to restricted pages or functions. In particular, numeric IDs or endpoint parameters that are meant to be hidden or restricted can be manipulated, allowing direct access to sensitive features. For example, an endpoint like /app?feature_id=10 may normally be accessible only to certain users, but changing the parameter to feature_id=20 could grant access to account creation, admin panels, or other restricted functions. Such misconfigurations can allow users to create accounts, escalate privileges, or access internal parts of the application without authorization, posing serious security risks.	
	
### >_ Parameter Modification
In some case, authentication control on just one basic parameter like "is_authenticated". This parameter can be on cookie header, url parameter or post body parameter. So just a basic change can be effective. For example "is_authenticated=false" to "is_authenticated=true".

### >_ Session ID Prediction
Some systems use predictible session id. So in this case just predict the session id value.

For Example:
	
	```
	session_id=112484697 >> your session id
	session_id=112484724 >> admin session id
	```
	
### >_ Authentication Bypass via SQLi
Many authentication mechanisms use SQL queries to check user credentials. If these queries are not properly sanitized, SQL Injection (SQLi) can occur, allowing login bypass. For example, consider a login form that executes:

	```
	SELECT * FROM users WHERE username = '$username' AND password = '$password';
	```
	
If the username is set to admin'-- and the password is left blank, the query becomes:

	```
	SELECT * FROM users WHERE username = 'admin'--' AND password = '';
	```
The -- comments out the password check, so login succeeds without knowing the actual password.	

### >_ Referances
- https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/04-Testing_for_Bypassing_Authentication_Schema
- https://www.synack.com/exploits-explained/exploits-explained-5-unusual-authentication-bypass-techniques/
- https://portswigger.net/support/using-sql-injection-to-bypass-authentication
- https://www.youtube.com/watch?v=DBNmAJaWcGk


