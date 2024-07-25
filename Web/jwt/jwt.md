## Exploiting flawed JWT signature verification
### Accepting arbitrary signatures
1. [Lab: JWT authentication bypass via unverified signature | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)
2. [JWT VII](https://pentesterlab.com/exercises/jwt_vii)
### Accepting tokens with no signature
1. [Lab: JWT authentication bypass via flawed signature verification | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)
2. [JSON Web Token](https://pentesterlab.com/exercises/jwt)
## Weak signing key
1. [Lab: JWT authentication bypass via weak signing key | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key
2. [JWT V](https://pentesterlab.com/exercises/jwt_v)
3. 
## JWT header parameter injections
### jwk header injection
1. [Lab: JWT authentication bypass via jwk header injection | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection)
2. [CVE-2018-0114](https://pentesterlab.com/exercises/cve-2018-0114)
3. 
### jku header injection
1. [Lab: JWT authentication bypass via jku header injection | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection)
2. [JWT VIII](https://pentesterlab.com/exercises/jwt_viii)
3. [JWT IX](https://pentesterlab.com/exercises/jwt_ix)
4. [JWT X](https://pentesterlab.com/exercises/jwt_x)
5. [JWT XI](https://pentesterlab.com/exercises/jwt_xi)
### x5u
1. [JWT XII](https://pentesterlab.com/exercises/jwt_xii)
### Injecting self-signed JWTs via the kid parameter
1. [Lab: JWT authentication bypass via kid header path traversal | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal)
2. [JWT III](https://pentesterlab.com/exercises/jwt_iii)
3. [JWT IV](https://pentesterlab.com/exercises/jwt_iv)
4. [JWT VI](https://pentesterlab.com/exercises/jwt_vi)
## Algorithm confusion
1. [Lab: JWT authentication bypass via algorithm confusion | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion)
2. [Lab: JWT authentication bypass via algorithm confusion with no exposed key | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key)
3. [JSON Web Token II](https://pentesterlab.com/exercises/jwt_ii)
4. [JWT XIII](https://pentesterlab.com/exercises/jwt_xiii)
5. [CVE-2022-21449](https://pentesterlab.com/exercises/cve-2022-21449)
6. [JWT XIV](https://pentesterlab.com/exercises/jwt_xiv)
## Tools

[GitHub - ticarpi/jwt_tool: :snake: A toolkit for testing, tweaking and cracking JSON Web Tokens](https://github.com/ticarpi/jwt_tool)
## Useful Burp Extentensions

JSON Web Tokens
JWT Editor
## Where to practice

[JWT attacks | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt)
https://pentesterlab.com/
## References

[Attack Methodology · ticarpi/jwt_tool Wiki (github.com)](https://github.com/ticarpi/jwt_tool/wiki/Attack-Methodology)
[JWT Vulnerabilities (Json Web Tokens) - HackTricks](https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens)
## Workflow

1. Find JWT
2. Identify a test page
3. Replay it
4. Required?
5. Checked?
6. Persistent?
7. Origin
8. Check claim-processing order
9. Weak HMAC secret used as a key
10. Testing known vulnerabilities
11. URL Tampering attacks
## Algorithm confusion attack

wget https://host-with-public-key/public.pem
./jwt_tool.py -S hs256 -k public.pem -T [JWT]
./jwt_tool.py -X k -pk public.pem [JWT]
## Checklist
### Basic checks

| **#** | **Status** | **Name** | **Description** | **Vulnerable** | **Notes** |
|:-:|:------:|:----:|:-----------:|:----------:|:-----:|
|1| ❌ | Required | Was the token required? | No | Any notes about the issue |
|2| ✅ | Checked | Was the token's signature checked? | ==Yes== | Any notes about the issue |
|3| ❌ | Persistent | Was the token persistent? | No | Any notes about the issue |
|4| ✅ | Origin | Was the token generated on the client side? | ==Yes== | Any notes about the issue |
|5| ❌ | Expired | Was it possible to use the expired token? | No | Any notes about the issue |
|6| ✅ | Claim processing order | Has some of the token's claims been processed before the verification process took place? | ==Yes== | Any notes about the issue |
|7| ❌ | Weak HMAC secret used as a key | Has the token been generated using a weak HMAC secret? | No | Any notes about the issue |
### Advanced checks

| **#** | **Status** | **Name** | **Description** | **Vulnerable** | **Notes** |
|:-:|:------:|:----:|:-----------:|:----------:|:-----:|
| 8 | ✅ | Algorithm 'none' Attack (CVE-2015-9235) | Has the application accepted unsecured token? | ==Yes== | Any notes about the issue |
| 9 | ❌ | Key Confusion Attack (CVE-2016-5431) | Could token generation algorithm be changed? | No | Any notes about the issue |
| 10 | ✅ | Key Injection Attack (CVE-2018-0114) | Could a custom inline public key be included into the token? | ==Yes== | Any notes about the issue |
| 11 | ❌ | Null Signature Attack (CVE-2020-28042) | Could the token be sent without a signature? | No | Any notes about the issue |
| 12 | ✅ | Issues in "kid" claim | Was "kid" claim vulnerable to injection attacks (SQLi, Path Traversal, OS Injection)? | Yes | Any notes about the issue |
| 13 | ❌ | Cross-service relay attacks | Was it possible to use the token generated for another application? | No | Any notes about the issue |
| 14 | ✅ | Issues in "jku" claim | Was it possible to inject a custom URL ? | ==Yes== | Any notes about the issue |
| 15 | ❌ | Issues in "x5u" claim | Was it possible to inject a custom certificate? | No | Any notes about the issue |
| 16 | ✅ | Issues in "x5c" claim | Was is possible to inject a custom certificate in Base64 format? | ==Yes== | Any notes about the issue |
| 17 | ❌ | JTI (JWT ID) | Could token be replayed? | No | Any notes about the issue |
## References

1. [Attack Methodology · ticarpi/jwt_tool Wiki · GitHub](https://github.com/ticarpi/jwt_tool/wiki/Attack-Methodology)
2. [JWT Vulnerabilities (Json Web Tokens) - HackTricks](https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens)
3. [JWT attacks | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/jwt)

