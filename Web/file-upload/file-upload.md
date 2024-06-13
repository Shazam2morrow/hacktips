# File Upload

## What are file upload vulnerabilities?

ToDo

## How do file upload vulnerabilities arise?

ToDo

## What is the impact of file upload vulnerabilities?

ToDo

## What things to consider while searching for file upload vulnerabilities?

ToDo (add info about tips from PortSwigger)

## Checklist for verifying AC and PE vulnerabilities

ToDo

## How to prevent file upload vulnerabilities?

Allowing users to upload files is commonplace and does not have to be dangerous as long as you take the right precautions. Implementing these measures will significantly reduce the risk of file upload vulnerabilities and enhance the overall security of your system.

### Whitelist file extensions

Use a whitelist of permitted file extensions to restrict the types of files users can upload:

- Create a list of allowed file extensions (e.g., `.jpg`, `.png`, `.pdf`).
- Ensure the application checks the file extension against this whitelist before accepting the upload.
- Reject any file that does not match an allowed extension.

It is easier to control and monitor the allowed file types than to predict and block all potential malicious file types.

### Validate filenames

Ensure filenames do not contain substrings that could be interpreted as directory traversal sequences or other dangerous patterns:

- Check for and remove any occurrences of `../` or other directory traversal sequences in filenames.
- Strip any potentially dangerous characters from the filenames (e.g., `..`, `/`, `\`, `:`).

Prevent attackers from navigating outside the intended directory structure and accessing or overwriting unintended files.

### Rename uploaded files

Rename files upon upload to avoid collisions and other potential issues:

- Generate a unique identifier for each uploaded file.
- Rename the file using this identifier or a similar predetermined convention.
- Optionally store the original filename as metadata in a database.

Avoid file overwrites, naming conflicts, and potential injection attacks by ensuring each uploaded file has a unique, system-generated name.

### Utilize a temporary storage for validation

Do not store uploaded files on the permanent filesystem until they have been fully validated:

- Store uploaded files in a temporary location.
- Perform all necessary validation checks (e.g., file type, size, virus scanning) in this temporary location.
- Only move validated files to the permanent storage location.

Ensure that only safe, validated files are stored permanently, reducing the risk of malicious files being executed or accessed.

### Use established frameworks for file uploads

Use established frameworks for handling file uploads rather than custom-built solutions:

- Integrate well-maintained and widely-used libraries or frameworks designed for secure file uploads.
- Regularly update these frameworks to incorporate the latest security patches and features.

Established frameworks are generally more robust and secure, benefiting from the collective experience and scrutiny of the developer community.

### Store files outside the web root

Store uploaded files outside the web root directory to prevent direct access by attackers:

- Configure the application to save files in a directory that is not directly accessible via the web server.
- Implement access controls to ensure only authorized users or processes can retrieve these files.

Minimize the risk of files being accessed or executed directly via URL, reducing the attack surface.

### Restrict file execution rights

Restrict execution permissions on directories where uploaded files are stored:

- Set directory permissions to disallow execution of files.
- Ensure only the application can access and manipulate these files.

Prevent uploaded files from being executed on the server, which could lead to code execution vulnerabilities.

### Enforce maximum file size

Enforce a maximum size limit for uploaded files to prevent resource exhaustion:

- Define a reasonable maximum file size for uploads.
- Reject any files that exceed this size limit.

Protect the server from denial-of-service attacks or other issues caused by excessively large files.

### Scan for viruses

Perform virus scans on all uploaded files before processing them:

- Integrate a reliable virus scanning solution.
- Scan each uploaded file in the temporary storage location.
- Reject any files that are flagged as malicious.

Ensure that uploaded files do not contain malware or other malicious content that could harm the application or its users.

### Set correct content type headers

Ensure that the correct `Content-Type` header is set when files are served to users:

- Determine the actual file type of each uploaded file.
- Set the `Content-Type` header based on this determined file type when serving the file to users.

Prevent content spoofing and other attacks by ensuring that files are interpreted correctly by the user's browser.

## Interactive checklist

**Interactive checklist** is a dynamically updated document (like Google Sheet) that stores information about the checks to be made as well as other important information about an issues like its status, severity, comments and etc.

It allows you to be consistent in your tests and make sure that every possible case is covered.

The list can be found [here](https://docs.google.com/spreadsheets/d/1vR7IDd4mE-0_mSVWdO4gFl_S6jhSpcKgIPz6a7t6Pps/edit?usp=sharing) and you can fine tune it according to the specification of your project.

Please, refer to this document to stay updated.

## Labs

- [File Upload Vulnerabilities - PortSwigger](https://portswigger.net/web-security/all-labs#file-upload-vulnerabilities)

## Videos

- [How File Upload Vulnerabilities Work](https://youtu.be/rPdn88pO7x0?feature=shared)
- [Web Application Hacking - File Upload Attacks Explained](https://youtu.be/YAFVGQ-lBoM?feature=shared)

## References

- [File Upload - PortSwigger](https://portswigger.net/web-security/file-upload)
