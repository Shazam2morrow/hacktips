# HTTP Strict Transport Security (HSTS)

## Description

[HTTP Strict Transport Security (HSTS)](https://www.rfc-editor.org/rfc/rfc6797) is a security feature that safeguards websites from certain types of cyber attacks, such as protocol downgrade attacks and cookie hijacking. It instructs web browsers to only use secure HTTPS connections when interacting with the website.

When a server uses HSTS, it sends a special response header, named `Strict-Transport-Security`, to the browser. This header tells the browser that it should only access the site using HTTPS, not HTTP, for a specified period of time. This period is defined by the `max-age` parameter in the header.

There are also optional parameters like `includeSubDomains` and `preload`. If `includeSubDomains` is specified, the rule applies to all subdomains of the site as well. The `preload` parameter can be used to include the site in a list of sites hardcoded into the browser that are only accessible via HTTPS.

## Key points

The HSTS has the following features: 

1. HSTS protection only kicks in after the user’s first visit to the site, following a principle known as [“trust on first use”](https://en.wikipedia.org/wiki/Trust_on_first_use). This means that the initial visit is still potentially vulnerable to certain types of attacks.

2. HSTS is more secure than simply configuring a HTTP to HTTPS (301) redirect on a server, where the initial HTTP connection is still vulnerable to a man-in-the-middle attack.

3. The `Strict-Transport-Security` header is ignored by the browser when a site has only been accessed using HTTP. Once the site is accessed over HTTPS with no certificate errors, the browser knows that the site is HTTPS capable and will honor the `Strict-Transport-Security` header. Browsers do this as attackers may intercept HTTP connections to the site and inject or remove the header.

4. Whenever the `Strict-Transport-Security` header is delivered to the browser, it will update the expiration time for that site, so sites can refresh this information and prevent the timeout from expiring. Should it be necessary to disable HSTS, setting the `max-age` to 0 (over an HTTPS connection) will immediately expire the `Strict-Transport-Security` header, allowing access via http.

## Checklist

The following checklist can be used to check for issues regarding HSTS:

| **#** | **Status** | **Name** | **Description** | **Vulnerable** | **Notes** |
|:-----:|:------------:|:--------:|:---------------:|:--------------:|:---------:|
| 1 | ❌ | STS header is not used | In case a web server did not utilize the `Strict-Transport-Security` header for encrypted communication in any response | No | Any notes about the issue |
| 2 | ✅ | STS header's expiration time is too small | In case the expiration time of the STS directive was set to a quite small value, like one day, for example. Such an approach could simplify a potential interception of an initial request (when the STS directive is outdated due to the expiration time) to the server sent by HTTP protocol | Yes | Any note about the issue |
| 3 | ❌ | STS header contains incorrect directive | In case the syntax of the directive is not correct a browser will ignore it marking it as an incorrect one effectively providing no protection as a consequence | No | Any notes about the issue |

## Tools

### nmap

Checks for the HTTP response headers related to security given in OWASP Secure Headers Project and gives a brief description of the header and its configuration value:

```
nmap -p <port> --script http-security-headers <target>
```

### curl

Using `-I` or `--head` option you can fetch the headers of a document:

```
curl -I <target>
```

Using `-v` or `--verbose` options you can fetch both the request and response headers:

```
curl -v <target>
```

## Recommendations

A web application should instruct web browsers to only access the application using an encrypted HTTPS channel. For that, HSTS should be enabled by adding a response header with the name `Strict-Transport-Security` and the value `max-age=expireTime`, where `expireTime` is the time in seconds that browsers should remember that the site should only be accessed using HTTPS.

It is also recommended to set the expiration time of STS directive for at least 3 months period. Such an approach should limit the time windows for potential eavesdropping attack during which a malefactor could try to intercept transmitted unencrypted data.

To apply the policy to all subdomains, the `includeSubDomains` flag could also be utilized. As an additional security measure, the domain should be submitted to an [HSTS preload service](https://hstspreload.org/).

## References

1. [Strict-Transport-Security - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
2. [HTTP Strict Transport Security - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
3. [The double-edged sword of HSTS persistence and privacy](https://www.leviathansecurity.com/media/the-double-edged-sword-of-hsts-persistence-and-privacy)
4. [Chromium HSTS](https://www.chromium.org/hsts/)
5. [HTTP Strict Transport Security - Wikipedia](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)