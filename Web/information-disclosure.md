# Information Disclosure
 
## What is information disclosure?
 
Information disclosure, also known as **information leakage**, occurs when a website unintentionally reveals sensitive information to its users. This can include data about other users, sensitive commercial or business data, and technical details about the website and its infrastructure.
 
Examples of information disclosure include revealing hidden directories, providing access to source code files via temporary backups, and hard-coding sensitive information such as API keys and credentials in the source code.
 
## How do information disclosure vulnerabilities arise?
 
Information disclosure vulnerabilities can arise due to failure to remove internal content from public content, insecure configuration of the website and related technologies, and flawed design and behavior of the application.
 
## What is the impact of information disclosure vulnerabilities?
 
The impact of information disclosure vulnerabilities can vary, with the severity depending on the purpose of the website and the potential harm an attacker could do with the leaked information. It is important to assess the severity of information disclosure vulnerabilities based on their impact and exploitability. **Usually you can not report a security bug until it has security impact**.

## What to think about while searching for information disclosure vulnerabilities?

If you are in doubt what information the target treats as valuable you can start by asking the following questions:

- How does your target make money?
- What makes the app special?
- Would a customer of the target be annoyed?
- How could you game the system?
- What does your target value?

## How to test for information disclosure vulnerabilities?
 
When testing for information disclosure vulnerabilities, it is important not to develop "tunnel vision" and focus narrowly on a particular vulnerability. Sensitive data can be leaked in various places, so it is crucial not to miss anything that could be useful later.

### Fuzzing

Fuzzing, or fuzz testing, is an automated software testing technique that involves providing invalid, unexpected, or random data as input to a program. The goal is to find vulnerabilities, crashes, memory leaks, or other unexpected behaviors by observing how the software responds to these inputs.

The typical workflow of a fuzzing is show below:

- Submit unexpected data types and specially crafted strings to parameters.
- Pay attention to subtle changes in responses, such as processing time differences.
- Use tools to automate this process and identify differences in HTTP status codes, response times, lengths, and specific keywords.

##### Payloads

There are also a lot of publicly available wordlists that you can use to automate your tasks:

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [FuzzDB](https://github.com/fuzzdb-project/fuzzdb)
- [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing)

### Analyzing error messages

Study error messages to learn information about the application's internal workings.

Look for details such as database table names, file paths, or technology stack information.

### Reviewing developer comments

Inspect HTML and source code for comments that might reveal sensitive information.

Use developer tools or web security tools to extract these comments.

### Checking directory listings

Attempt to access directories without index files to see if the server lists the contents.

Review files like `/robots.txt` and `/sitemap.xml` for paths that should not be accessible.

### Exploring debugging data

Look for custom error messages or logs that might be left in the production environment.

Analyze any detailed debug information that could provide insights into application behavior.

### Source code disclosure

Search for backup files, temporary files, or version control directories that might be accessible.

Try requesting source code files using known backup file extensions (e.g., `.bak`, `.old`, `~`).

### User account pages

Test whether account pages expose information about other users by manipulating URL parameters or other input fields.

### Verbose output in debug mode

Websites left in debug mode can display detailed stack traces and other diagnostic information that shouldn't be exposed in a production environment.

### Third-party services

Misconfigured integrations with third-party services can leak sensitive information via API responses or webhook URLs.

### Unnecessary information in responses

HTTP responses may include headers or metadata that disclose server versions, framework versions, or other internal details.

## How to prevent information disclosure vulnerabilities?

Preventing information disclosure completely is tricky due to the huge variety of ways in which it can occur. However, there are some general best practices that you can follow to minimize the risk of these kinds of vulnerability creeping into your own websites. 

### Awareness and Education

Ensure that everyone involved in creating the website understands what information is considered sensitive. Highlighting the risks associated with seemingly harmless information can improve overall security practices.

### Code Audits

Regularly audit the code for potential information disclosure during QA or build processes. Automate tasks like stripping developer comments where possible.

### Generic Error Messages

Use generic error messages to avoid providing attackers with clues about the application's behavior.

### Disable Debugging and Diagnostic Features

Verify that any debugging or diagnostic features are disabled in the production environment.

### Thorough Configuration Management

Fully understand the configuration settings and security implications of any third-party technologies you implement. Disable any features and settings that are not needed​.

## Labs

- [Information Disclosure Labs - PortSwigger](https://portswigger.net/web-security/all-labs#information-disclosure)

## Videos

- [Information Disclosure - Bugcrowd](https://www.youtube.com/watch?v=ZM08KoiBkkI)

## References

- [Information Disclosure - Portswigger](https://portswigger.net/web-security/information-disclosure)
