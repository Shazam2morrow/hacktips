# Access Control

## What is access control?

**Access control (AC)** is a fundamental security mechanism that regulates **who or what can view or use resources** in a computing environment. It involves processes and policies to ensure that only authorized individuals or systems have access to specific data, applications, systems, or physical locations.

**AC** comprises three primary components:

1. **Authentication**

   Verifying the identity of a user or system. It identifies the user and confirms that they say who they say they are. This often involves credentials such as usernames and passwords, biometric data, or security tokens.

2. **Authorization**

   Determining whether a user or system has permission to access a resource or perform an action. This is typically based on predefined access policies or rules.

3. **Accountability**

   Tracking and logging access to resources to ensure that activities can be audited and traced back to the responsible party. This helps in identifying security breaches and maintaining compliance with regulatory requirements.

## What is access control vulnerability?

An **access control (AC)** vulnerability occurs when the mechanisms meant to restrict who can access certain resources or perform specific actions fail. This vulnerability allows users to bypass access restrictions, leading to unauthorized actions or data access.

## What is privilege escalation?

A **privilege escalation (PE)** is a type of security issue where a user gains elevated access rights beyond what they are supposed to have. Basically, there are two types of privilege escalations: **vertical** and **horizontal**.

### Vertical privilege escalation (VPE)

**VPE** occurs when a lower-privileged users gain access to higher-privileged functions.

### Horizontal privilege escalation (HPE)

**HPE** occurs when a user accesses resources of other users with the same privilege level.

## How do AC and PE vulnerabilities arise?

These vulnerabilities typically arise from misconfigurations, flawed assumptions, and design errors. For example, unprotected functionality or parameter manipulation can enable unauthorized access.

## What is the impact of AC and PE vulnerabilities?

The impact of these vulnerabilities can be severe, including data breaches, system compromise, and regulatory violations. Such incidents can lead to loss of trust, financial losses, and significant reputational damage.

## Things to consider while searching for AC and PE vulnerabilities?

When searching for such vulnerabilities, consider the questions below to guide your testing.

By asking these questions, you can focus your efforts on identifying and understanding **AC** and **PE** vulnerabilities effectively.

### What are the critical functions and resources?

Identify what functions and data are essential to the application's operation. Look for administrative panels, financial transactions, and sensitive data endpoints.

### How does the application handle user roles and permissions?

Examine how different user roles are defined and what permissions each role has. Look for inconsistencies or overly permissive roles that might lead to privilege escalation.

### How are user sessions managed?

Investigate the session management mechanisms. Check if session tokens are properly validated and if there are protections against session fixation and hijacking.

### What mechanisms are in place to prevent unauthorized access?

Assess the access control mechanisms implemented in the application. Ensure there are proper checks at every access point, and look for any bypass opportunities.

### How could a lower-privileged user exploit the system?

Think about ways a user with minimal privileges could attempt to access higher-privileged functions or data. Test common attack vectors like URL manipulation and parameter tampering.

### What could an attacker gain by escalating privileges?

Consider the potential impact of a successful privilege escalation. Determine what sensitive data or critical functions could be compromised.

## How to test for access control vulnerabilities?

### Black-Box Testing

When performing black-box testing, the video emphasizes checking the following key aspects to identify access control vulnerabilities:

1. Mapping the Application:

    - Visit the URL of the application and navigate through all accessible pages within the user context.
    - Make note of all input vectors that could potentially be used to make access control decisions in the backend, such as URL parameters, hidden parameters, and cookies.
    - Use tools like Burp Suite to intercept and analyze all requests made to the application during navigation.

2. Requesting Multiple Accounts:
    - Ensure that you have accounts of different privilege levels to test for vertical privilege escalation (e.g., regular user, admin user).
    - Request at least two accounts of each privilege level to test for horizontal privilege escalation (e.g., two regular user accounts).

3. Manipulating Parameters:

    - Manipulate identified parameters to check if they are used for access control decisions.
    - Test parameter manipulation for each privilege level and across different pages to confirm if they are vulnerable.

4. Using Automation Tools:

    - Utilize tools and extensions like the Authorize extension in Burp Suite to automate a portion of the access control testing.
    - The Authorize extension can make requests from different user perspectives (unauthenticated, low-privileged user, high-privileged user) and compare responses to identify access control vulnerabilities.

5. Comparing Responses:

    - Compare the length and content of responses for the same request made from different privilege levels.
    - Look for discrepancies that indicate missing or bypassed access control rules.

6. Test for common access control vulnerabilities, such as:

    - Bypassing access control checks by modifying parameters in the URL or HTML.
    - Accessing APIs with missing access controls on various methods (POST, GET, DELETE).
    - Forceful browsing to authenticated pages as an unauthenticated user.
    - Manipulating metadata like JSON web tokens or cookies to escalate privileges.

### White-Box Testing

When performing white-box testing, the video emphasizes checking several key aspects to identify access control vulnerabilities.

1. Centralized Access Control Mechanism:

    - Verify if the application has a central component that all requests go through to check access control rules.
    - Ensure that the application denies access by default unless explicit rules are set to grant access.

2. Violations of the Principle of Least Privilege:

    - Look for situations where the system defaults to open access rather than defaulting to closed access.
    - Ensure that users only have access to the resources and privileges necessary for their role.

3. Missing or Weak Access Control Checks:

    - Check for functions or resources lacking proper access control checks.
    - Identify if any functions allow unauthorized users to perform actions they shouldn't be able to.

4. Access Control on API Methods:

    - Ensure access control rules are applied consistently across all API methods (GET, POST, PUT, DELETE).
    - Verify that each method has appropriate access control checks in place.

5. Client-Side Input Reliance:

    - Identify instances where the application relies on client-side input to make access control decisions.
    - Ensure that all client-side inputs are validated on the server side before being used in access control checks.

6. Review Code for Access Control Implementation:

    - Examine the source code to understand how access control is implemented.
    - Identify any potential gaps or inconsistencies in the implementation of access control rules.

7. Validate Findings on a Running Application:

    - Test potential vulnerabilities identified in the code on the actual running application to confirm if they are exploitable.
    - Recognize that a vulnerability might appear in the code but might be mitigated by other factors within the application or its configuration.

## Examples of Broken Access Controls

Each of these examples highlights how broken access controls can lead to severe security breaches, emphasizing the importance of rigorous access control implementations and testing.

### Insecure Direct Object References (IDOR)

IDOR are a type of access control vulnerability that occurs when an application uses user-supplied input to access objects directly. This vulnerability allows attackers to bypass authorization checks and gain unauthorized access to data or functionalities by manipulating input parameters.

**Description:**
- **Example:** A web application uses predictable identifiers in URLs (e.g., `example.com/user?userId=123`). An attacker changes the `userId` parameter to access other users' data.
- **Impact:** Unauthorized access to sensitive information, such as personal data, financial records, or private messages.

### Unprotected Functionality

**Description:**
- **Example:** Administrative functions are accessible without proper authentication. For instance, `example.com/admin` is not restricted and can be accessed by anyone.
- **Impact:** Attackers gain control over administrative features, leading to data breaches, system configuration changes, or user account manipulation.

### Misconfigured Access Control Policies

**Description:**
- **Example:** Incorrectly configured access control rules in a firewall or application that allow unauthorized users to access restricted areas. For instance, a web server's configuration file mistakenly grants public access to confidential directories.
- **Impact:** Exposure of sensitive data and critical system files, potentially compromising the entire system.

### Vertical Privilege Escalation

**Description:**
- **Example:** A low-privileged user can perform administrative actions due to flawed access controls. For instance, a regular user can access `/admin/deleteUser` by modifying the request in a web application.
- **Impact:** Unauthorized administrative actions, such as deleting user accounts, altering permissions, or modifying sensitive data.

### Horizontal Privilege Escalation

**Description:**
- **Example:** A user can access another user's data by changing the account identifier in a request. For instance, user A can view user B's order details by altering the order ID in the URL (`example.com/order?orderId=456`).
- **Impact:** Unauthorized access to other users' information, leading to privacy violations and potential data leaks.

### Lack of Authorization Checks

**Description:**
- **Example:** An application fails to verify if a user has permission to perform an action. For instance, the application checks if a user is authenticated but does not check if they are authorized to view a specific resource.
- **Impact:** Authenticated users can access resources or perform actions they are not authorized to, leading to data breaches and misuse of application functionalities.

### Forced Browsing

**Description:**
- **Example:** An attacker manually changes the URL to access unauthorized pages (e.g., `/admin/settings`). The application does not properly restrict access to these pages.
- **Impact:** Unauthorized access to sensitive application areas, potentially leading to data breaches and manipulation of critical settings.

### Missing Function Level Access Control

**Description:**
- **Example:** The application exposes function-level APIs without proper access control checks. For example, an API endpoint like `POST /api/deleteUser` can be accessed by any authenticated user without checking their role.
- **Impact:** Unauthorized execution of critical functions, leading to data manipulation, account deletions, or service disruption.

## General strategies for preventing access control vulnerabilities

1. **Centralize Access Control Logic:**
   - Use a central component to manage access control decisions across the application. This ensures consistency and reduces the risk of missing access checks.

2. **Server-Side Enforcement:**
   - Enforce all access control decisions on the server side. Client-side controls can be easily bypassed by attackers.

3. **Least Privilege Principle:**
   - Grant users the minimum level of access necessary to perform their tasks. This limits the potential damage from compromised accounts.

4. **Role-Based Access Control (RBAC):**
   - Implement RBAC to manage permissions based on user roles. Define roles carefully to ensure appropriate access levels and avoid overly broad permissions.

5. **Regular Audits and Reviews:**
   - Conduct regular audits of access control policies and configurations. Review access logs to detect and respond to unauthorized access attempts.

6. **Secure Session Management:**
   - Implement robust session management practices, such as using secure cookies, enforcing session timeouts, and protecting against session hijacking.

## Specific Measures to Prevent Privilege Escalation

1. **Vertical Privilege Escalation:**
   - **Verify Role Permissions:** Ensure that role permissions are properly defined and enforced. Regularly review role definitions and access levels.
   - **Segregate Administrative Functions:** Separate administrative functions from regular user functions. Use additional security measures, such as two-factor authentication (2FA) and IP whitelisting, for administrative access.

2. **Horizontal Privilege Escalation:**
   - **Validate User Ownership:** Always check that the user requesting access to a resource is the owner or has the necessary permissions. For example, verify that a user can only access their own data.
   - **Use Unpredictable Identifiers:** Implement non-predictable identifiers (e.g., GUIDs) for resources to make it harder for attackers to guess and manipulate identifiers.

## Secure Coding Practices

1. **Input Validation:**
   - Validate all user inputs to ensure they conform to expected formats and types. Reject any unexpected or malformed inputs.

2. **Parameterized Queries:**
   - Use parameterized queries to prevent SQL injection attacks, which can lead to unauthorized data access and privilege escalation.

3. **Code Reviews and Security Testing:**
   - Conduct regular code reviews and security testing (e.g., static code analysis, dynamic application security testing) to identify and fix vulnerabilities early.

4. **Implement Indirect Object References:**
   - Use indirect object references or mapping tables to manage access to sensitive resources. This helps prevent direct access to objects using user-supplied identifiers.

## Additional Recommendations

1. **Multi-Factor Authentication (MFA):**
   - Implement MFA to add an extra layer of security. This reduces the risk of unauthorized access, even if an attacker gains access to a user's credentials.

2. **Access Control Policies:**
   - Develop and enforce comprehensive access control policies. Clearly define who can access what resources and under what conditions.

3. **User Education and Awareness:**
   - Educate users about security best practices and the importance of protecting their credentials. Encourage the use of strong, unique passwords and regular password changes.

4. **Monitoring and Logging:**
   - Implement robust monitoring and logging to track access attempts and detect suspicious activities. Analyze logs regularly to identify potential security incidents.

## Checklist for Verifying Access Control Vulnerabilities

1. User Account Segregation
Use Multiple Accounts: Test the application using different user accounts with varying privilege levels (e.g., admin, regular user, guest).
Vertical Privilege Escalation: Use a high-privilege account to locate all available functionalities and then attempt to access those using a lower-privileged account.
Horizontal Privilege Escalation: Use two different user accounts with the same privilege level to test access to each other's resources.
2. URL and Parameter Manipulation
Direct Object References: Check if changing URL parameters allows access to unauthorized resources (e.g., example.com/user?userId=123 to example.com/user?userId=124).
Predictable Identifiers: Test if resource identifiers (e.g., user IDs, document IDs) are predictable and can be manipulated to gain unauthorized access.
3. HTTP Method Testing
Method Manipulation: Identify privileged requests and test whether modifying the HTTP method (e.g., changing POST to GET) affects access control.
Invalid Verbs: Attempt to use arbitrary invalid HTTP methods to see if the application still processes the request.
4. Client-Side Analysis
Review HTML and Scripts: Inspect client-side code (HTML, JavaScript) for references to hidden or sensitive functionality.
Hidden Fields and Cookies: Check for sensitive data or access control information stored in hidden form fields or cookies that can be manipulated.
5. Session Management
Session Token Analysis: Ensure that session tokens are properly managed and cannot be easily predicted or reused by attackers.
Session Fixation: Test for session fixation vulnerabilities by attempting to set or fixate session tokens.
6. Multistage Processes
Step-by-Step Verification: Walk through multistage processes (e.g., checkout flows, account setups) to ensure each stage has proper access controls.
Session Switching: Switch session tokens between steps to check if access controls are consistently enforced throughout the process.
7. Static Resource Access
Protected Static Content: Verify that static resources (e.g., images, documents) are protected and cannot be accessed directly via their URLs without proper authorization.
URL Access: Using different user contexts, attempt to access static resources directly to check if access controls are applied.
8. Administrative Function Access
Admin Page Protection: Ensure that administrative pages are not accessible to regular users and require proper authentication and authorization.
Authentication Methods: Verify that robust authentication methods (e.g., MFA, IP filtering) are used for sensitive administrative functions.
9. Authorization Checks
Consistent Authorization: Confirm that authorization checks are implemented consistently across all application functions and endpoints.
Server-Side Enforcement: Ensure that all access control decisions are enforced on the server side, not relying on client-side validation.
10. Logging and Monitoring
Access Logs: Review access logs for any unauthorized access attempts or suspicious activities.
Alerting: Implement alerting mechanisms for detecting and responding to potential access control breaches.

## Labs

- [Access Control Vulnerabilities - PortSwigger](https://portswigger.net/web-security/all-labs#access-control-vulnerabilities)

## Videos

- [Web Security Academy - Broken Access Control](https://www.youtube.com/playlist?list=PLuyTk2_mYISId4_l9YET7Gv29cHcNguq-)
- [What is Access Control and How Can You Break It?](https://www.youtube.com/watch?v=ub5yaztmmK4&list=PLIK9nm3mu-S4aCVEkK-lTzZFNs5MPXC_F&index=4)

## References

- [Access Control - PortSwigger](https://portswigger.net/web-security/access-control)