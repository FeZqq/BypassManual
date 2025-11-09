<img alt="JWT_Bypass" src="https://github.com/user-attachments/assets/c24a6e16-b32b-40b3-acb2-36114c7407ca" />

### >_ Introduction
A JSON Web Token (JWT) is a compact, URL-safe string used to carry claims between parties — typically for authentication and authorization in web apps and APIs. It consists of three parts (header, payload, signature) encoded in Base64URL and joined by dots. The header declares the token type and algorithm, the payload holds claims (like user id, roles, expiry), and the signature lets the receiver verify the token wasn’t tampered with. Because JWTs are self-contained and stateless, servers can validate them without keeping session state, but they must correctly verify the signature and check claims like expiration and audience.
A JWT bypass is when an attacker gains access to protected resources by exploiting flaws in how a system issues or validates JWTs — for example misconfiguration, weak key management, or skipping proper checks. It means the token-based protections are effectively circumvented, allowing unauthorized access or privilege escalation. (No technical steps provided.)

Note: We will be using Burp Suite JWT Editor in all tutorial. You can download it at Bapp Store.

### >_ Unverified Signature
If the server doesn't verify a JWT's signature, any payload fields—username, role, permissions, etc.—can bealtered by an attacker. For example, changing role: "administrator" and sending the modified token will be accepted, granting elevated access. The result is authentication bypass, privilege escalation, and possible account/data compromise.

<img alt="JWT_Bypass" src="./img/ss1.png" />

<img alt="JWT_Bypass" src="./img/ss2.png" />

### >_ Flawed Signature Verification (None Signature)
If a server accepts alg: "none" and skips signature verification, an attacker can remove the signature and use a forged payload to gain any privileges.JWT verification should confirm the signature proves the token wasn’t tampered with. If the server blindly trusts the token header and treats alg: "none" as “no signature required,” it never checks that the payload is legitimate — so any claims (username, role, permissions) can be changed and accepted.

<img alt="JWT_Bypass" src="./img/ss3.png" />

<img alt="JWT_Bypass" src="./img/ss4.png" />

### >_ Weak Signing Key
It brute‑forces the JWT secret using hashcat. Then, using the recovered secret, change the JWT payload on "jwt.io".

```
 hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list 
```

<img alt="JWT_Bypass" src="./img/ss5.png" />

<img alt="JWT_Bypass" src="./img/ss6.png" />

<img alt="JWT_Bypass" src="./img/ss7.png" />


### >_ JWK Header Injection
A JWK header is a field in a JWT header that can carry information about a JSON Web Key (a public key) used to verify the token. When a server accepts a JWK supplied inside the token header, it may use that attacker-controlled public key to validate the token’s signature.

<img alt="JWT_Bypass" src="./img/ss8.png" />

<img alt="JWT_Bypass" src="./img/ss9.png" />

<img alt="JWT_Bypass" src="./img/ss10.png" />

<img alt="JWT_Bypass" src="./img/ss12.png" />

<img alt="JWT_Bypass" src="./img/ss11.png" />


### >_ JKU (JSON Web Key) Header Injection
JKU header is the URL information located in the header section of the JWT structure and indicating where the JWK cluster is located. So, if the server accept and use JKU header we can change any payload in JWT easily.

Step 1: Creating RSA key and add key on our exploit server as body.

<img alt="JWT_Bypass" src="./img/ss12.png" />

<img alt="JWT_Bypass" src="./img/ss13.png" />

<img alt="JWT_Bypass" src="./img/ss14.png" />

Paste JWK in these structure:
```
{
    "keys": [

    ]
}
```
<img alt="JWT_Bypass" src="./img/ss15.png" />

Step 2: Add "JKU" header, change "kid" value with your RSA key's kid value  and sign with the RSA key with your payload (ex: "sub":"administrator").

<img alt="JWT_Bypass" src="./img/ss16.png" />

### >_ Kid Header Path Traversal
The kid header is just an identifier the server uses to look up the key for verifying a JWT. If the server naively treats kid as a filesystem path and doesn’t sanitize it, an attacker can perform path traversal (e.g. ../../../../../../dev/null) to make the server read an attacker-controlled or empty file as the key. When the server uses a symmetric algorithm like HS256, the same secret is used to sign and verify tokens, so an attacker who can predict or force the server to read a trivial key (such as an empty value) can create a token with sub: "administrator", sign it with that trivial secret, and the server will accept it. This is how the JWT bypass works: unsanitized kid → path traversal → predictable key → attacker-signed HMAC token accepted as valid. (Related failures include algorithm-confusion where alg is trusted blindly; forcing HS256 when the server expected RS256 can enable similar attacks.)

First Status:
<img alt="JWT_Bypass" src="./img/ss17.png" />

Step 1: Create Symmetric Key and Change "k" parameter to "AA==". ("AA==" : null byte) 
<img alt="JWT_Bypass" src="./img/ss18.png" />

Step 2: Change the kid value to empty file and change sub payload to what you want. Then sign the JWT with our symmetric key.
<img alt="JWT_Bypass" src="./img/ss19.png" />


### >_ Algorithm Confusion
The root cause is an algorithm‑confusion vulnerability: the server trusts the JWT’s client‑supplied alg value and/or fails to enforce key‑type checks, so an attacker can trick a verifier that normally uses an asymmetric scheme (e.g. RS256 with a public JWK published at /jwks.json) into accepting a symmetric HMAC token (e.g. HS256) by reusing the publicly available RSA key material as the HMAC secret; the described technique simply converts the published JWK/PEM into a form the signing tool accepts (PEM → Base64 → k in a symmetric JWK) and then crafts a token with an elevated sub and alg: HS256, which the misconfigured server validates because it treats the public key as the HMAC secret — the fix is to stop trusting alg from the client and enforce algorithm/key‑type whitelists and proper JWKS/key handling.

First Request:
<img alt="JWT_Bypass" src="./img/ss20.png" />

Getting Public Key:
<img alt="JWT_Bypass" src="./img/ss21.png" />

Creating HS256 Key via Public Key:
<img alt="JWT_Bypass" src="./img/ss22.png" />

<img alt="JWT_Bypass" src="./img/ss22.png" />

<img alt="JWT_Bypass" src="./img/ss23.png" />

<img alt="JWT_Bypass" src="./img/ss24.png" />

<img alt="JWT_Bypass" src="./img/ss25.png" />

Signing & Result:
<img alt="JWT_Bypass" src="./img/ss26.png" />


### >_ No Exposed Public Key
Will be continue..


