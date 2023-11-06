Definition 

IDOR (Insecure Direct Object Reference) vulnerability is a security weakness in web applications that occurs when an attacker can directly access and manipulate a specific object or resource without proper authorization. This can happen when the application uses user-supplied input to access sensitive data or perform actions. 

IDOR is about missing access control! 

How to find 

In order to find IDOR two conditions must be satisfied: 

1. Access control is not properly implemented 
    
2. The references to data objects are predictable 
    

Types 

IDORs can be of two types: state-changing (write) and non-state-changing (read). 

In terms of state-changing (write) IDORs, password reset, password change, account recovery IDORs often have the highest business impact. (Say, as compared to a “change email subscription settings” IDOR.) 

As for non-state-changing (read) IDORs, look for functionalities that handle the sensitive information in the application. For example, look for functionalities that handle direct messages, sensitive user information, and private content. Consider which functionalities on the application makes use of this information and look for IDORs accordingly. 

Examples 

Examples of IDOR vulnerabilities include: 

- User data: Imagine an e-commerce website where users can view their orders by providing an order number. If the website uses sequential order numbers and the order data is not properly protected, an attacker could simply change the order number in the URL to access another user's order data. 
    

- Private files: A file-sharing application that allows users to share files with each other may have a vulnerability if the URLs of the files are directly accessible. If an attacker can guess or manipulate the URLs, they can access files that are not intended for them. 
    

- Restricted pages: A web application may have pages that are restricted to certain user roles or permissions. If the application uses predictable parameters to control access, an attacker could simply change the parameter in the URL to access the restricted page. 
    

- API calls: APIs can also be vulnerable to IDOR attacks if they allow direct object references. For example, if an API call retrieves a user's data based on a user ID, an attacker could simply modify the user ID in the API call to access another user's data. 
    

Prevention 

To prevent IDOR vulnerabilities, it's important to properly validate user input and use indirect object references, such as using a unique identifier or a token that cannot be guessed or manipulated. 

References 

- [https://vickieli.medium.com/intro-to-idor-9048453a3e5d](https://vickieli.medium.com/intro-to-idor-9048453a3e5d) 
    
- [https://vickieli.medium.com/how-to-find-more-idors-ae2db67c9489](https://vickieli.medium.com/how-to-find-more-idors-ae2db67c9489) 
    
- [https://book.hacktricks.xyz/pentesting-web/idor#8d15](https://book.hacktricks.xyz/pentesting-web/idor#8d15) 
    

Checklist 

Check if ID is encoded or hashed and try to decode it 

Check if ID is predictable 

Check if ID can be leaked through another endpoint 

Check if ID can be created or linked manually 

Check if ID is accepted by the endpoint even if it does not ask for it 

Check for HPP (Http Parameter Pollution). Supply multiple values for the same parameter 

Check if information is leaked via another channel (Blind IDOR) 

Check if the request method can be changed (GET, POST, PUT, DELETE, PATCH) 

Check if the requested file type can be changed