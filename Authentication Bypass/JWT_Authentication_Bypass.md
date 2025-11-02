<img width="1536" height="1024" alt="JWT_Bypass" src="https://github.com/user-attachments/assets/c24a6e16-b32b-40b3-acb2-36114c7407ca" />

### >_ Introduction
A JSON Web Token (JWT) is a compact, URL-safe string used to carry claims between parties — typically for authentication and authorization in web apps and APIs. It consists of three parts (header, payload, signature) encoded in Base64URL and joined by dots. The header declares the token type and algorithm, the payload holds claims (like user id, roles, expiry), and the signature lets the receiver verify the token wasn’t tampered with. Because JWTs are self-contained and stateless, servers can validate them without keeping session state, but they must correctly verify the signature and check claims like expiration and audience.
A JWT bypass is when an attacker gains access to protected resources by exploiting flaws in how a system issues or validates JWTs — for example misconfiguration, weak key management, or skipping proper checks. It means the token-based protections are effectively circumvented, allowing unauthorized access or privilege escalation. (No technical steps provided.)

Note: We will be using Burp Suite JWT Editor in all tutorial. You can download it at Bapp Store.

### >_ Unverified Signature
If the server doesn't verify a JWT's signature, any payload fields—username, role, permissions, etc.—can be altered by an attacker. For example, changing role: "administrator" and sending the modified token will be accepted, granting elevated access. The result is authentication bypass, privilege escalation, and possible account/data compromise.

<img width="1536" height="1024" alt="JWT_Bypass" src="./img/ss1.png" />
