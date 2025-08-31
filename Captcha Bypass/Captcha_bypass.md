# Captcha Bypass
<img width="1536" height="1024" alt="cbs" src="https://github.com/user-attachments/assets/dec469e2-560a-4778-8fbd-fcea8c973fa3" />

### Introduction
Captcha bypass refers to techniques used to circumvent CAPTCHA mechanisms that are designed to distinguish humans from automated bots. This can happen due to weak implementations, such as when CAPTCHA validation is only performed on the client-side without proper server-side checks, allowing attackers to skip or manipulate requests. In other cases, flaws like predictable tokens, reuse of the same challenge, or misconfigured integrations with third-party CAPTCHA services can enable bypassing. Automated tools, OCR (Optical Character Recognition), or machine learning models may also be used to solve CAPTCHAs at scale. Ultimately, a CAPTCHA bypass undermines the intended protection against automated abuse, such as brute-force attacks, credential stuffing, or mass registration.


### >_ Client-Side Capthca
Client-side captcha only validates on the client side and does not include any backend checks. This way, when the request is captured and sent via repeater, it doesn't encounter any limitations.


### >_ Broken Captcha
A broken captcha apparently receives the captcha value and sends it to the backend. However, the captcha value isn't properly validated in the backend, rendering the captcha completely useless. This allows the transaction to be accepted regardless of the captcha value.


### >_ CAPTCHA Replay Vulnerability
CAPTCHA replay is a vulnerability where a previously valid CAPTCHA response can be reused multiple times due to missing or improper server-side invalidation. Instead of treating each CAPTCHA solution as a one-time token, the backend continues to accept the same value for subsequent requests if it is intercepted and replayed, often bypassing the intended frontend flow that would normally invalidate it. This flaw allows attackers to solve a CAPTCHA once and then automate requests with the same response, completely undermining its purpose of preventing bots, brute-force attacks, or automated abuse


### >_ CAPTCHA Replay Vulnerability Due to Missing Server-Side Invalidation
This scenario represents a CAPTCHA replay vulnerability caused by improper server-side validation. Normally, once a CAPTCHA is solved correctly, the value should be invalidated on the server to prevent reuse. However, in this case, the invalidation only occurs when the request is processed through the normal frontend flow. By intercepting the request with a proxy and preventing it from reaching the frontend, the attacker can repeatedly send the same valid CAPTCHA value to the backend, which continues to accept it. This bypasses the intended one-time use property of CAPTCHA and effectively nullifies its protection against automated attacks.


### >_ Client-Controlled Security Flag
A common CAPTCHA implementation flaw occurs when the request includes a client-controlled parameter such as captchaCheck=true/false that determines whether CAPTCHA validation should be enforced. Since this value can be easily modified by an attacker through tools like Burp Suite, the CAPTCHA mechanism can be completely bypassed by simply setting the parameter to the expected value. This design flaw stems from relying on client-side input for a security control, rather than performing the CAPTCHA validation entirely on the server side, and ultimately allows attackers to disable the protection intended to prevent automated abuse.


### >_ Bypassing CAPTCHA via Cookie Manipulation
In some cases, capcha appears after 3 or 5 incorrect attempt. So, if the backend keeps track of how many requests I make based on my session value (for example session:abcd) instead of my IP address, deleting this value will completely prevent the captcha from appearing.


### >_ Bypassing CAPTCHA via Exposed Hash in HTML Source
This implementation exposes the hashed CAPTCHA value directly in the HTML source as a hidden input field, which allows attackers to automate the bypass process. Since the hash is already provided to the client, a script can simply parse the page, extract the hash value, and submit it along with the form request without ever solving the actual CAPTCHA challenge. This design flaw eliminates the need for human interaction and enables automated tools or bots to repeatedly bypass the CAPTCHA protection, defeating its core purpose of preventing automated abuse.


### >_ Disabling CAPTCHA by IP-Based Tracking Flaw
This vulnerability occurs when the CAPTCHA mechanism relies on failed login attempts per IP address while the server trusts the X-Forwarded-For header to determine the client identity. If the value of this header is changed for each request, the server interprets every attempt as coming from a new user. As a result, the system never reaches the threshold for multiple failed attempts and the CAPTCHA challenge is not triggered. This design allows unlimited login attempts without solving a CAPTCHA, showing the risk of relying on client-controlled headers for security decisions.


### >_ CAPTCHA Replay Vulnerability Due to Delayed Expiration
This vulnerability occurs when the CAPTCHA mechanism validates the first request it receives but does not immediately expire the token upon use. By capturing a valid “Forgot Password” request in a proxy and sending it repeatedly through an automated tool, the same CAPTCHA value can be reused multiple times before it expires. As a result, the system treats each replayed request as valid, allowing the CAPTCHA protection to be bypassed and enabling repeated automated actions without solving new challenges.


### >_ Referances
- https://infosecwriteups.com/bypassing-captcha-like-a-boss-d0edcc3a1c1
- https://hackerone.com/reports/210417
- https://medium.com/@abhishekY495/bypassing-captcha-17c59d37f459
- https://infosecwriteups.com/weird-story-of-captcha-to-rate-limit-bypass-c62690db39a

- https://medium.com/@batuhanaydinn/bug-bounty-hunter-captcha-bypass-response-to-this-request-a1438e503db6
