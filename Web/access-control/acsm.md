# Access Control Security Models (ACSM)

## What is access control?

**Access control (AC)** is a fundamental security mechanism that regulates **who or what can view or use resources** in a computing environment. It involves processes and policies to ensure that only authorized individuals or systems have access to specific data, applications, systems, or physical locations.

## What are the components of AC?

**AC** consists of three primary components:

1. **Authentication**

   Verifying the identity of a user or system. It identifies the user and confirms that they say who they say they are. This often involves credentials such as usernames and passwords, biometric data, or security tokens.

2. **Authorization**

   Determining whether a user or system has permission to access a resource or perform an action. This is typically based on predefined access policies or rules.

3. **Accountability**

   Tracking and logging access to resources to ensure that activities can be audited and traced back to the responsible party. This helps in identifying security breaches and maintaining compliance with regulatory requirements.

## What is ACSM?

**ACSM** define how access rights are assigned and enforced within a system.

Each model has its advantages and disadvantages, and the choice of model depends on the specific security requirements, scale, and nature of the application or organization.

The main types of access control models are presented below.

## Discretionary Access Control (DAC)

In **DAC** model, the owner of a resource (like a file or database entry) determines who can access it. This model relies on the identity of the users and their permissions set by the resource owner.

For example, in many operating systems, such as Windows and Unix, file permissions are set by the file owner. The owner can grant read, write, or execute permissions to other users. A user can create a document and set permissions to allow certain colleagues to read or edit it, while restricting others.

Another example is databases like MySQL or PostgreSQL, the owner of a database object (such as a table) can grant or revoke access to other users. A database administrator can grant select and update privileges on a specific table to a data analyst, while denying access to other tables.

The **DAC** model has the following **advantages**:

- Owners can quickly and easily grant or revoke access to their resources.
- Users have full control over their resources, making it straightforward for personal use or small-scale applications.

On the other hand the **DAC** model also has its own **disadvantages**:

- Because users have full control, there is a higher risk of improper configurations and accidental data leaks.
- As the number of resources and users grows, managing permissions becomes complex and error-prone task.

## Mandatory Access Control (MAC)

**MAC** is a centralized model where access rights are enforced by a central authority based on predefined policies. Users cannot change permissions; access is determined by security labels assigned to users and resources.

For example, military and government systems often use **MAC** to enforce strict access controls based on classification levels (e.g., Confidential, Secret, Top Secret). A military document labeled as "Top Secret" can only be accessed by personnel with the appropriate clearance level, and the permissions are managed centrally without individual overrides.

Another example is SELinux which is an implementation of **MAC** in the Linux operating system, providing a mechanism for enforcing security policies. SELinux restricts the ability of processes to access files, network resources, and other system components based on defined security policies, preventing unauthorized actions even by the root user.

The **MAC** model has the following **advantages**:

- Centralized control reduces the risk of accidental misconfigurations and enforces strict access policies.
- Ensures consistent application of security policies across the entire system.

On the other hand the **MAC** model also has its own **disadvantages**:

- Users have no control over access rights, which can be cumbersome for dynamic or collaborative environments.
- Implementing and maintaining a **MAC** system can be complex and resource-intensive.

## Role-Based Access Control (RBAC)

In **RBAC** model, access rights are assigned based on user roles within an organization. Roles are created for various job functions, and permissions to perform certain operations are assigned to these roles.

For example, many organizations use **RBAC** to manage access to their IT systems based on employee roles and responsibilities. In a company, employees in the HR department have access to the HR system and payroll data, while employees in the finance department have access to financial systems and records.

The **RBAC** model has the following **advantages**:

- Easier to manage permissions as they are grouped by roles rather than individuals.
- Suitable for large organizations where users can be assigned roles based on their responsibilities.

On the other hand **RBAC** model also has its own **disadvantages**:

- In large organizations, the number of roles can become very large, leading to complexity in management.
- Defining appropriate roles and permissions requires careful planning and can be time-consuming.

## Programmatic Access Control (PAC)

**PAC** model involves using a matrix or rules-based approach to manage access rights, typically stored in a storage, like database. Access decisions are made programmatically based on predefined rules and conditions.

For example, web applications often implement programmatic access control to manage permissions based on complex business rules. An online banking application checks if a user has the necessary permissions to view account details or perform transactions based on their account type and status.

The **PAC** model has the following **advantages**:

- Highly customizable to fit specific application needs and can dynamically adapt to changing requirements.
- Allows for fine-grained access control at a detailed level, offering precise control over resources.

On the other hand the **PAC** model also has its own **disadvantages**:

- Developing and maintaining a programmatic access control system can be complex and requires significant effort.
- Depending on the complexity of the rules, it might impact system performance due to the computational overhead of evaluating access decisions.

## Attribute-Based Access Control (ABAC)

**ABAC** model involves assigning access based on attributes or characteristics related to the **user (subject)**, the **resource (object)**, and **environmental conditions** (e.g., geo-location, network or time of day). Policies are created based on these attributes to grant or deny access.

For example, imagine that an auditor at a large accounting firm, needs access to internal documents and client's financial records for a new project. Traditionally, a **RBAC** system would require creating and assigning multiple roles with specific permissions. With **ABAC**, a centralized policy can be established that applies across all projects, existing or new.

In other words, a policy can be created where access is granted only if the project attributes of both the **subject** and **object** match. Another policy ensures that only auditors can access sensitive financial data. These policies allow only project-assigned auditors to access the client's financial data, simplifying management and ensuring security.

The **ABAC** model has the following **advantages**:

- Simplifies access management across applications, projects, and users.
- Centralizes auditing and access policy management.
- Easier to meet strict data segregation requirements.

On the other hand the **ABAC** model also has its own **disadvantages**:

- Initial setup and policy creation can be complex.
- Requires thorough understanding of attributes and their implications for access control.

## Conclusion

As you can see there is no "silver bullet" or "one solution for everything" because each model can work better then the other depending on the context.

## References

- [Attribute Based Access Control](https://www.youtube.com/watch?v=cgTa7YnGfHA)
- [Role-Based Access Control](https://www.youtube.com/watch?v=4Uya_I_Oxjk)
