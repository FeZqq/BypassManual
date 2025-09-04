# One Time Password (OTP) Bypass
<img width="1536" height="1024" alt="otpb" src="https://github.com/user-attachments/assets/267cfdf2-2961-423e-8ce4-75e075b49f5c" />

### >_ Introduction
A One-Time Password (OTP) is a temporary, single-use code generated for authentication, often sent via SMS, email, or an authenticator app, and used to verify a user’s identity during login or transactions, adding an extra security layer beyond static passwords. OTP bypass occurs when an attacker manages to circumvent this protection, typically by exploiting flaws such as intercepting SMS messages, abusing misconfigurations in the verification logic (e.g., accepting reused or predictable codes, or skipping OTP checks entirely), leveraging social engineering to trick users into sharing codes, or manipulating API requests to bypass the OTP validation step.

### >_ OTP Bypass via Brute Force
OTP bypass via brute force happens when an attacker repeatedly tries different numeric or alphanumeric combinations until the correct one is found, taking advantage of weak implementations such as short OTP length (e.g., 4–6 digits), lack of rate limiting, no account lockout mechanism, or predictable code generation; if these protections are missing, the attacker can automate requests and eventually guess the valid OTP, effectively bypassing the verification process.

### >_ OTP Bypass via Response Manipulation
If response manipulation is applied during OTP verification, altering the status codes or messages sent from the server can trick the system into treating an invalid or missing OTP as valid. This works when OTP validation relies on client-side responses instead of being strictly enforced on the server, so changing an error response to a success response effectively bypasses the OTP check. For example; On Burp Proxy, enable the intercept and capture the request. Then right-click the request, choose 'Do intercept' > 'Response to this request' from the menu, and modify the response from 'fail' to 'success'.

Example:

Request:
```
POST /verify-otp HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 45
Cookie: session=abcd1234

{
    "username": "user1",
    "otp": "123456"
}

```

Normal Response:
```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Content-Length: 55

{
    "status": "error",
    "message": "Incorrect OTP. Please try again."
}

```

Modified Response:
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 54

{
    "status": "success",
    "message": "OTP Verified Successfully"
}
```

### >_ Empty OTP / NULL Values / Insert Zeros / True value
Basically, trying "000000", "null", "" or "true".

### >_ Victim & Attacker Number in One Request
Victim & Attacker Number in One Request is an OTP bypass method that exploits the way some applications handle JSON-based OTP requests. If the server does not properly validate that only one phone number can be used per request, combining the victim’s number and the attacker’s number in the same JSON payload can result in the OTP being sent to the attacker’s device while still allowing access to the victim’s account.

Example Request:
```
POST /request-otp HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "phone_no": "victim_number",
    "phone_no": "attacker_number"
}
```


### >_ Parallel OTP Request
Parallel OTP Request is a technique where OTPs for both the attacker and the victim are requested simultaneously, exploiting systems that generate predictable or synchronized codes. If the OTP generation algorithm produces the same code for multiple requests made at the same time (or within a short time window), the attacker can use their own OTP to log in as the victim.

Example Scenario:

1- The attacker uses a tool like Burp Intruder to send two OTP requests at the same time:
```
POST /request-otp HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "phone_no": "victim_number"
}
```

```
POST /request-otp HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "phone_no": "attacker_number"
}
```

2- If the system generates the same OTP for both numbers due to weak randomization or predictable seeds, the attacker can enter their received OTP in the victim’s login flow.

This method relies on poor OTP generation logic, lack of per-user uniqueness, or time-based synchronization flaws.

To do this method:

1- Create both request on repeater.

2- Select "New Tab Group" end of the tabs in repeater.

<img width="1262" height="129" alt="burp1" src="https://github.com/user-attachments/assets/3836d077-98ac-4f5a-a97d-87aaa440f675" />

3- Select "Send Group in Parallel" at send menu.

<img width="389" height="259" alt="burp2" src="https://github.com/user-attachments/assets/f68def11-7499-45c8-a515-f0458e89058d" />

### >_ Infinite OTP Regeneration
Infinite OTP Regeneration is a technique that exploits systems allowing unlimited OTP requests without restrictions. If an application does not implement rate limiting, lockouts, or request throttling, an attacker can repeatedly request new OTPs for a target account and try a set of common or likely OTPs (like 0000, 1234, 1111) until one of them is accepted.

Example Scenario:

1-Send a request to generate a new OTP for the victim:
```
POST /request-otp HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "phone_no": "victim_number"
}
```

2-Receive the OTP (or generate multiple attempts) repeatedly.
3-Submit each OTP guess in the login verification endpoint until the system accepts one:
```
POST /verify-otp HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "phone_no": "victim_number",
    "otp": "1234"
}
```

This method works because the server does not limit OTP generation or attempts, making brute-force or token guessing feasible.

### >_ Referances
- https://hackerone.com/reports/897385

- https://medium.com/@k.raksshitha/otp-bypass-b3ea10091f34

