# Access Control Vulnerabilities

## What are access control vulnerabilities?

**Access control (AC)** vulnerabilities occur when the mechanisms meant to restrict who can access certain resources or perform specific actions fail. This vulnerability allows users to bypass access restrictions, leading to unauthorized actions or data access.

## What is privilege escalation?

**Privilege escalation (PE)** is a type of security issue where a user gains elevated access rights beyond what they are supposed to have. It is the most common type of unauthorized action.

## How do AC vulnerabilities arise?

These vulnerabilities typically arise from misconfigurations, flawed assumptions, and design errors. For example, unprotected functionality or parameter manipulation can enable unauthorized access.

## What is the impact of AC vulnerabilities?

The impact of these vulnerabilities can be severe, including data breaches, system compromise, and regulatory violations. Such incidents can lead to loss of trust, financial losses, and significant reputational damage.

## What things to consider while searching for AC vulnerabilities?

When searching for such vulnerabilities, consider the questions below to guide your testing. By asking these questions, you can focus your efforts on identifying and understanding **AC** and **PE** vulnerabilities effectively:

1. **What are the critical functions and resources of the application?**

   Identify what functions and data are essential to the application's operation. Look for administrative panels, financial transactions, and sensitive data endpoints.

2. **How does the application handle user roles and permissions?**

   Examine how different user roles are defined and what permissions each role has. Look for inconsistencies or overly permissive roles that might lead to **PE**.

3. **How are user sessions managed?**

   Investigate the session management mechanisms. Check if session tokens are properly validated and if there are protections against session fixation and hijacking.

4. **What mechanisms are in place to prevent unauthorized access?**

   Assess the **AC** mechanisms implemented in the application. Ensure there are proper checks at every access point, and look for any bypass opportunities.

5. **How could a lower-privileged user exploit the system?**

   Think about ways a user with minimal privileges could attempt to access higher-privileged functions or data. Test common attack vectors like URL manipulation and parameter tampering.

6. **What could an attacker gain by escalating privileges?**

   Consider the potential impact of a successful **PE**. Determine what sensitive data or critical functions could be compromised.

## What are examples of broken AC?

Each of these examples below highlights how broken **AC** can lead to severe security breaches, emphasizing the importance of rigorous **AC** implementations and testing.

### Insecure Direct Object Reference (IDOR)

**IDOR** vulnerability is a type of **AC** vulnerability that occurs when an attacker can directly access and manipulate a specific object or resource without proper authorization. This can happen when the application uses user-supplied input to access sensitive data or perform actions.

For example, if a web application uses predictable identifiers in URLs (e.g., `example.com/user?userId=123`) then an attacker might change the `userId` parameter to access other users' data. In this case the impact is unauthorized access to sensitive information, such as personal data, financial records, or private messages.

**IDOR** can be of two types: **state-changing (write)** and **non-state-changing (read)**.

In terms of **state-changing IDOR** (like password reset, password change, account recovery) often have the highest business impact. Say, as compared to a “change email subscription settings” **IDOR**.

As for **non-state-changing IDOR**, look for functionalities that handle the sensitive information in the application. For example, look for functionalities that handle direct messages, sensitive user information, and private content. Consider which functionalities on the application makes use of this information and look for **IDOR** accordingly.

Examples of **IDOR** vulnerabilities include:

- **User data**: Imagine an e-commerce website where users can view their orders by providing an order number. If the website uses sequential order numbers and the order data is not properly protected, an attacker could simply change the order number in the URL to access another user's order data.

- **Private files**: A file-sharing application that allows users to share files with each other may have a vulnerability if the URLs of the files are directly accessible. If an attacker can guess or manipulate the URLs, they can access files that are not intended for them.

- **Restricted pages**: A web application may have pages that are restricted to certain user roles or permissions. If the application uses predictable parameters to control access, an attacker could simply change the parameter in the URL to access the restricted page.

- **API calls**: APIs can also be vulnerable to IDOR attacks if they allow direct object references. For example, if an API call retrieves a user's data based on a user ID, an attacker could simply modify the user ID in the API call to access another user's data.

### Unprotected functionality

Imagine that some administrative functions of your application are accessible without proper authentication. For instance, `yoursite.com/admin` is not restricted and can be accessed by anyone. Even by anonymous users. In this case the attackers might gain control over administrative features, leading to data breaches, system configuration changes, or user account manipulation.

### Misconfigured AC policies

Sometimes incorrectly configured **AC** rules in a firewall or application might allow unauthorized users to access restricted areas. For instance, a web server's configuration file mistakenly grants public access to confidential directories. In this case the exposure of sensitive data and critical system files, might potentially compromise the entire system.

### Vertical Privilege Escalation (VPE)

**VPE** occurs when a lower-privileged users gain access to higher-privileged functions.

Imagine that a low-privileged user performs administrative actions due to flawed **AC**. For instance, a regular user can access `yoursite.com/admin/deleteUser` by modifying the request in a web application. In this case the low-privileged user gets access to unauthorized administrative actions, such as deleting user accounts, altering permissions, or modifying sensitive data.

### Horizontal Privilege Escalation (HPE)

**HPE** occurs when a user accesses resources of other users with the same privilege level.

Imagine that a user can access another user's data by changing the account identifier in a request. For instance, user A can view user B's order details by altering the `orderId` in the URL (`yoursite.com/order?orderId=456`). In this case the impact is unauthorized access to other users' information, leading to privacy violations and potential data leaks.

### Lack of authorization checks

Imagine that an application fails to verify if a user has permission to perform an action. For instance, the application checks if a user is authenticated but does not check if they are authorized to view a specific resource. In this case authenticated users might access resources or perform actions they are not authorized to, leading to data breaches and misuse of application functionalities.

### Forced browsing (direct requests)

Imagine that an attacker manually changes the URL to access protected pages (e.g., `/admin/settings`). In this case the application does not properly restrict access to these pages and it leads to unauthorized access to sensitive application areas, potentially resulting into data breaches and manipulation of critical settings.

### Missing function level AC

Imagine that an application exposes function-level APIs without proper **AC** checks. For example, an API endpoint like `yoursite.com/api/deleteUser` can be accessed by any authenticated user without checking their role. It might lead to unauthorized execution of critical functions, leading to data manipulation, account deletions, or service disruption.

There are many more examples of **AC** vulnerabilities of course but the presented ones are by far the most common I have seen in the wild.

## How to test for AC vulnerabilities?

### Black-Box Testing

When performing black-box testing, you should consider the following key aspects:

1. **Mapping the application**

    - Visit the URL of the application and navigate through all accessible pages within the user context.
    - Make note of all input vectors that could potentially be used to make **AC** decisions in the backend, such as URL parameters, hidden parameters, and cookies.
    - Use tools like **Burp Suite** to intercept and analyze all requests made to the application during navigation.

2. **Requesting multiple accounts**

    - Ensure that you have accounts of different privilege levels to test for **VPE** (e.g., regular user, admin user).
    - Request at least two accounts of each privilege level to test for **HPE** (e.g., two regular user accounts).

3. **Manipulating parameters**

    - Manipulate identified parameters to check if they are used for **AC** decisions.
    - Test parameter manipulation for each privilege level and across different pages to confirm if they are vulnerable.

4. **Using automation tools and extensions**

    - Utilize tools and extensions (like [Authorize](https://github.com/portswigger/autorize), [AuthMatrix](https://github.com/portswigger/auth-matrix), [Auth](https://github.com/portswigger/authz)) for **Burp Suite** to automate a portion of the **AC** testing. Such extensions can make requests from different user perspectives (unauthenticated, low-privileged user, high-privileged user) and compare responses to identify **AC** vulnerabilities.
    - [Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/) is Firefox extension that allows to keep the browser's tabs in separate independent containers. This feature allows you to create a container per each test user and view the differences on the UI.

5. **Comparing responses**

    - Compare the length and content of responses for the same request made from different privilege levels.
    - Look for discrepancies that indicate missing or bypassed **AC** rules.

6. **Test for common AC vulnerabilities**

    - Bypassing **AC** checks by modifying parameters in the URL or HTML.
    - Accessing APIs with missing **AC** on various methods (POST, GET, DELETE).
    - Forceful browsing to authenticated pages as an unauthenticated user.
    - Manipulating metadata like JSON web tokens or cookies to escalate privileges.

### White-Box Testing

When performing white-box testing, you should consider the following key aspects:

1. **Centralized AC mechanism**

    - Verify if the application has a central component that all requests go through to check **AC** rules.
    - Ensure that the application denies access by default unless explicit rules are set to grant access.

2. **Violations of the Principle of Least Privilege**

    - Look for situations where the system defaults to open access rather than defaulting to closed access.
    - Ensure that users only have access to the resources and privileges necessary for their role.

3. **Missing or weak AC checks**

    - Check for functions or resources lacking proper **AC** checks.
    - Identify if any functions allow unauthorized users to perform actions they should not be able to.

4. **AC on API methods**

    - Ensure **AC** rules are applied consistently across all API methods (GET, POST, PUT, DELETE).
    - Verify that each method has appropriate **AC** checks in place.

5. **Client-side input reliance**

    - Identify instances where the application relies on client-side input to make **AC** decisions.
    - Ensure that all client-side inputs are validated on the server side before being used in **AC** checks.

6. **Validate findings on a running application**

    - Test potential vulnerabilities identified in the code on the actual running application to confirm if they are exploitable.
    - Recognize that a vulnerability might appear in the code but might be mitigated by other factors within the application or its configuration.

## Checklist for verifying AC vulnerabilities

Below you can find some basic checks that will allow you to check your application for **AC** and **PE** vulnerabilities.

For additional tests please refer to the latest version of the [interactive checklist](#interactive-checklist) section of this document.

### User account segregation

- Test the application using different user accounts with varying privilege levels (e.g., admin, regular user, guest).

- Use a high-privilege account to locate all available functionalities and then attempt to access those using a lower-privileged account.

- Use two different user accounts with the same privilege level to test access to each other's resources.

### URL and parameter manipulation

- Check if changing URL parameters allows access to unauthorized resources (e.g., `yoursite.com/user?userId=123` to `yoursite.com/user?userId=124`).

- Test if resource identifiers (e.g., user IDs, document IDs) are predictable and can be manipulated to gain unauthorized access.

### HTTP method testing

- Identify privileged requests and test whether modifying the HTTP method (e.g., changing POST to GET) affects **AC**.

- Attempt to use arbitrary invalid HTTP methods to see if the application still processes the request.

### Client-side code analysis

- Inspect client-side code (HTML, JavaScript) for references to hidden or sensitive functionality.

- Check for sensitive data or **AC** information stored in hidden form fields or cookies that can be manipulated.

### Session management

- Ensure that session tokens are properly managed and cannot be easily predicted or reused by attackers.

- Test for session fixation vulnerabilities by attempting to set or fixate session tokens.

### Multistage processes

- Walk through multistage processes (e.g., checkout flows, account setups) to ensure each stage has proper **AC**.

- Switch session tokens between steps to check if **AC** are consistently enforced throughout the process.

### Static resource access

- Verify that static resources (e.g., images, documents) are protected and cannot be accessed directly via their URLs without proper authorization.

- Using different user contexts, attempt to access static resources directly to check if **AC** are applied.

### Administrative function access

- Ensure that administrative pages are not accessible to regular users and require proper authentication and authorization.

- Verify that robust authentication methods (e.g., MFA, IP filtering) are used for sensitive administrative functions.

### Authorization checks

- Confirm that authorization checks are implemented consistently across all application functions and endpoints.

- Ensure that all **AC** decisions are enforced on the server side, not relying on client-side validation.

## How to prevent AC vulnerabilities?

### Generic recommendations

1. **Server-side enforcement**

   Enforce all **AC** decisions on the server side. Client-side controls can be easily bypassed by attackers.

2. **Least privilege principle**

   Grant users the minimum level of access necessary to perform their tasks. This limits the potential damage from compromised accounts.

3. **Regular audits and reviews**

   Conduct regular audits of **AC** policies and configurations. Review access logs to detect and respond to unauthorized access attempts.

4. **Secure session management**

   Implement robust session management practices, such as using secure cookies, enforcing session timeouts, and protecting against session hijacking.

### How to prevent VPE vulnerabilities?

1. **Verify role permissions**

   Ensure that role permissions are properly defined and enforced. Regularly review role definitions and access levels.

2. **Segregate administrative functions**

   Separate administrative functions from regular user functions. Use additional security measures, such as two-factor authentication (2FA) and IP whitelisting, for administrative access.

### How to prevent HPE vulnerabilities?

1. **Validate user ownership**

   Always check that the user requesting access to a resource is the owner or has the necessary permissions. For example, verify that a user can only access their own data.

2. **Use unpredictable identifiers**

   Implement non-predictable identifiers (e.g., GUIDs) for resources to make it harder for attackers to guess and manipulate identifiers.

### Secure coding practices

1. **Centralize AC logic**

   Use a central component to manage **AC** decisions across the application. This ensures consistency and reduces the risk of missing access checks.

2. **Input validation**

   Validate all user inputs to ensure they conform to expected formats and types. Reject any unexpected or malformed inputs.

3. **Parameterized queries**

   Use parameterized queries to prevent SQL injection attacks, which can lead to unauthorized data access and **PE**.

4. **Code reviews and security testing**

   Conduct regular code reviews and security testing (e.g., static code analysis, dynamic application security testing) to identify and fix vulnerabilities early.

5. **Implement indirect object references**

   Use indirect object references or mapping tables to manage access to sensitive resources. This helps prevent direct access to objects using user-supplied identifiers.

## Interactive checklist

- [Google Table - Access Control](https://docs.google.com/spreadsheets/d/1vR7IDd4mE-0_mSVWdO4gFl_S6jhSpcKgIPz6a7t6Pps/edit?gid=862087147#gid=862087147)

## Labs

- [Access Control Vulnerabilities - PortSwigger](https://portswigger.net/web-security/all-labs#access-control-vulnerabilities)

## Videos

- [Broken Access Control - Complete Guide](https://youtu.be/_jz5qFWhLcg?feature=shared)
- [What is Access Control and How Can You Break It?](https://www.youtube.com/watch?v=ub5yaztmmK4&list=PLIK9nm3mu-S4aCVEkK-lTzZFNs5MPXC_F&index=4)
- [How I made 1k in a day with IDORs! (10 Tips!)](https://www.youtube.com/watch?v=hmlkUYJ9MFw)

## References

- [Access Control - PortSwigger](https://portswigger.net/web-security/access-control)
- [Intro to IDOR - Medium](https://vickieli.medium.com/intro-to-idor-9048453a3e5d)
- [How to find more IDORs - Medium](https://vickieli.medium.com/how-to-find-more-idors-ae2db67c9489)
- [IDOR - Hacktricks](https://book.hacktricks.xyz/pentesting-web/idor#8d15)
