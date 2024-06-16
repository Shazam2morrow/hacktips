# File Upload Vulnerabilities

## What are file upload vulnerabilities?

File upload vulnerabilities are security weaknesses that arise when a web server permits users to upload files without adequately validating their name, type, contents, or size. This lack of proper validation can lead to various security issues, such as allowing the upload of arbitrary and potentially dangerous files. These files could include server-side script files that enable remote code execution. In some instances, the mere act of uploading the file can cause harm, while other attacks might require a follow-up HTTP request to trigger the file's execution by the server.

## How do file upload vulnerabilities arise?

File upload vulnerabilities arise from insufficient validation and insecure handling of uploaded files. This includes not adequately checking the file type, name, size, and contents. Additionally, insecure server configurations and lack of restrictions on uploaded files can enable attackers to upload and execute malicious files, leading to remote code execution or other security breaches.

## What is the impact of file upload vulnerabilities?

File upload vulnerabilities can lead to remote code execution, data overwriting, denial-of-service attacks, and unauthorized access. These vulnerabilities enable attackers to control servers, disrupt services, and manipulate data.

## How to test for file upload vulnerabilities?

These tests provide a comprehensive guide to identifying and testing various file upload vulnerabilities in applications. Each test is designed to uncover potential security risks associated with file uploads.

### Unrestricted file uploads

Verify if the application allows unrestricted file uploads that can execute server-side scripts (e.g., PHP, Python). Test by attempting to upload a script and executing it to check for web shell deployment.

A web shell is a malicious script that enables an attacker to execute arbitrary commands on a remote web server simply by sending HTTP requests to the right endpoint.

For example, the following PHP one-liner could be used to read arbitrary files from the server's filesystem:

```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```

A more versatile web shell may look something like this:

```php
<?php echo system($_GET['command']); ?>
```

This script accepts an arbitrary system command via a query parameter as shown below:

```http
GET /exploit.php?command=id HTTP/1.1
```

### Flawed file type validation

Test file type validation by uploading files with incorrect MIME types using tools like Burp Repeater. Ensure the server validates file contents and not just headers. The `Content-Type` response header may provide clues as to what kind of file the server thinks it has served. If this header hasn't been explicitly set by the application code, it normally contains the result of the file extension/MIME type mapping.

### File execution

Check if the server executes uploaded files. Upload different file types to user-accessible directories and attempt to execute them. Verify the server’s response.

### Insufficient blacklisting

Test for blacklisting of dangerous file types. Upload files with alternative extensions (`.php5`, `.shtml`) or obfuscated names (`exploit.pHp`, `exploit.php.jpg`). Verify if the server still executes these files.

### Overriding server configuration

Attempt to upload server configuration files (`.htaccess` for Apache, `web.config` for IIS) to override or add to global settings. Verify if you can map custom extensions to executable MIME types.

For example, here is an example of an `.htaccess` configuration file that processes files with `.jpeg` or `.jpg` extensions using the PHP interpreter:

```config
<FilesMatch "\.(jpeg|jpg)$">
    SetHandler application/x-httpd-php
</FilesMatch>
```

This configuration uses the `FilesMatch` directive to match files with the `.jpeg` or `.jpg` extensions and sets the handler to `application/x-httpd-php`, which tells the server to process these files with the PHP interpreter.

Or you can even provide the following Apache directive that maps an arbitrary extension (`.l33t`) to the executable MIME type `application/x-httpd-php` and if the server uses the `mod_php` module, it knows how to handle this already:

```.htaccess
AddType application/x-httpd-php .l33t
```

### Obfuscating file extensions

Use obfuscation techniques like multiple extensions (exploit.php.jpg), URL encoding, trailing characters, and multibyte Unicode characters to bypass blacklists:

- Provide multiple extensions. Depending on the algorithm used to parse the filename, the following file may be interpreted as either a PHP file or JPG image: `exploit.php.jpg`.

- Add trailing characters. Some components will strip or ignore trailing whitespaces, dots, and suchlike: `exploit.php`.

- Try using the URL encoding (or double URL encoding) for dots, forward slashes, and backward slashes. If the value isn't decoded when validating the file extension, but is later decoded server-side, this can also allow you to upload malicious files that would otherwise be blocked: `exploit%2Ephp`.

- Add semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in C/C++, for example, this can cause discrepancies in what is treated as the end of the filename: `exploit.asp;.jpg` or `exploit.asp%00.jpg`.

- Try using multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion or normalization. Sequences like `xC0 x2E, xC4 xAE or xC0 xAE` may be translated to `x2E` if the filename parsed as a UTF-8 string, but then converted to ASCII characters before being used in a path.

### Flawed content validation

Validate file contents instead of headers. Upload files with inconsistent headers and contents (e.g., a PHP script with a `.jpg` header). Verify if the server rejects these files.

Certain file types may always contain a specific sequence of bytes in their header or footer. These can be used like a fingerprint or signature to determine whether the contents match the expected type. For example, JPEG files always begin with the bytes `FF D8 FF`.

This is a much more robust way of validating the file type, but even this is not foolproof. Using special tools, such as [ExifTool](https://exiftool.org/), it can be trivial to create a polyglot JPEG file containing malicious code within its metadata.

Here is an example of how you can create a polyglot PHP/JPG file that is fundamentally a normal image, but contains PHP payload in its metadata. A simple way of doing this is to download and run `exiftool` from the command line as follows:

```bash
exiftool -Comment="<?php echo 'START '.file_get_contents('/etc/password').' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php
```

This adds custom PHP payload to the image's `Comment` field, then saves the image with a `.php` extension. `START` and `END` are needed to find out the content boundaries.

### Race condition exploits

Check for race conditions during file upload processing. Upload files quickly before validation is complete to see if execution is possible. Verify server response and timing.

Usually it is very hard to identify such vulnerabilities without access to the application source code but even without one this attack can be automated, for example, using [Turbo Intruder](https://github.com/portswigger/turbo-intruder) which is a Burp Suite extension for sending large numbers of HTTP requests and analyzing the results.

For example, the following script template can be used to send multiple requests to the server very quickly:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,)

    request1 = '''<YOUR-FIRST-REQUEST>'''

    request2 = '''<YOUR-SECOND-REQUEST>'''

    # the 'gate' argument blocks the final byte of each request until openGate is invoked
    engine.queue(request1, gate='race1')
    for x in range(5):
        engine.queue(request2, gate='race1')

    # wait until every 'race1' tagged request is ready
    # then send the final byte of each request
    # (this method is non-blocking, just like queue)
    engine.openGate('race1')

    engine.complete(timeout=60)


def handleResponse(req, interesting):
    table.add(req)
```

If you choose to build the `GET` request manually, make sure you terminate it properly with a `\r\n\r\n` sequence. Also remember that Python will preserve any whitespace within a multiline string, so you need to adjust your indentation accordingly to ensure that a valid request is sent.

### URL-based upload race conditions

Test URL-based file uploads for race conditions. Brute-force temporary directory names if pseudo-random functions are used. Extend processing time by uploading large files.

### Client-side script uploads

Upload files with client-side scripts (e.g., HTML with `<script>` tags or SVG with embedded scripts). Verify if these scripts are executed in the user’s browser, leading to stored XSS vulnerabilities.

### Vulnerabilities in file parsing

Test for vulnerabilities in file parsing. Upload specially crafted files (e.g., XML-based .doc or .xls files) to check for XXE injection or other parsing vulnerabilities.

### PUT method uploads

Verify if the server supports `PUT` requests for file uploads. Use `PUT` to upload malicious files directly to the server and check for execution. You can try sending `OPTIONS` requests to different endpoints to test for any that advertise support for the `PUT` method.

### Archive content validation

Test if the application validates the contents of uploaded archives. Upload archives containing malicious files and verify if they are evaluated or executed.

### Path traversal vulnerability

Check for path traversal vulnerabilities by including traversal characters (`../`) in the uploaded file name. Verify if the application allows accessing files outside the intended directory. Web servers often use the `filename` field in `multipart/form-data` requests to determine the name and location where the file should be saved.

### Large file uploads

Test if the application rejects overly large files to prevent denial of service attacks. Upload large files and observe the server’s response.

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

- [Google Table - File Upload](https://docs.google.com/spreadsheets/d/1vR7IDd4mE-0_mSVWdO4gFl_S6jhSpcKgIPz6a7t6Pps/edit?gid=905183500#gid=905183500)

## Labs

- [File Upload Vulnerabilities - PortSwigger](https://portswigger.net/web-security/all-labs#file-upload-vulnerabilities)

## Videos

- [How File Upload Vulnerabilities Work](https://youtu.be/rPdn88pO7x0?feature=shared)
- [Web Application Hacking - File Upload Attacks Explained](https://youtu.be/YAFVGQ-lBoM?feature=shared)

## References

- [File Upload - PortSwigger](https://portswigger.net/web-security/file-upload)
