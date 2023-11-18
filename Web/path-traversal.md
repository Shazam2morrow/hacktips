# Path Traversal

## Description

Path traversal is a common web application security vulnerability. It occurs when an application allows an attacker to navigate outside the intended directory structure and access files or directories that are not supposed to be directly accessible.

This kind of attack is also known as the __dot-dot-slash__ attack, or [__directory traversal__](https://en.wikipedia.org/wiki/Directory_traversal_attack) or __directory climbing__, or __backtracking__.

This can lead to unauthorized access to sensitive information, including configuration files, user data, or even executable scripts.

The attacker typically manipulates input parameters, such as file or directory names, by using special characters like `../` (dot-dot-slash) or encoding schemes to trick the application into navigating to unintended locations.

## Things to remember

1. Different operating systems use different path separators:

    - __Unix-based OS__:
        - root directory: `/`
        - directory separator: `/`
    - __Windows OS__:
        - root directory: `<drive letter>:`
        - directory separator: `\` or `/`
    - __Classic macOS__:
        - root directory: `<drive letter>:`
        - directory separator: `:`

2. Some archive formats (like _zip_) allow for directory traversal attacks: __files in the archive can be written such that they overwrite files on the filesystem by backtracking__.

3. Access to files is limited by system operational access control (such as in the case of locked or in-use files on the Microsoft Windows operating system).

4. Some defense mechanisms can be bypassed using different encoding schemes like double encoding, UTF-8 encoding and other.

## Checklist

### Agenda for symbols

- ✅ denotes that the specific issue has been reviewed or checked
- ❌ indicates that the specific issue has not been reviewed or checked yet
- 🚫 indicates that the specific issue was not applicable
- 🟢 denotes the absence of the vulnerability within the application
- 🔴 signifies the presence of the vulnerability within the application

### Path traversal checklist

The following checklist can be used to track path traversal issues:

| **№** | **Checked?** | **Issue Name** | **Issue Description** | **Vulnerable?** | **Notes** |
|:-----:|:------------:|:--------:|:---------------:|:--------------:|:---------:|
| 1 | ❌ | Arbitrary file access | In case an application allows to read or execute any arbitrary file in the system | 🟢 | Any notes about the issue |
| 2 | ✅ | Absolute path | In case an application allows to use an absolute path from the filesystem root, such as `/etc/passwd`, to directly reference a file without any traversal sequences | 🔴 | Any notes about the issue |
| 3 | 🚫 | Non-recursive stripping | In case an application allows to use nested traversal sequences, such as `....//` or `....\/`, so when the inner sequence is stripped these revert to simple traversal sequences | 🟢 | Any notes about the issue |
| 4 | ❌ | URL encoding | In case an application allows to use encoded version of `../` sequence (results in `%2e%2e%2f`) | 🔴 | Any notes about the issue |
| 5 | ✅ | Double URL encoding | In case an application allows to use doubly-encoded version of `../` sequence (results in `%252e%252e%252f`) | 🟢 | Any notes about the issue |
| 6 | 🚫 | Base folder | In case an application requires the user-supplied filename to start with an expected base folder, so it might be possible to include the required base folder followed by suitable traversal sequences, like `/var/www/static/../../../etc/passwd` | 🔴 | Any notes about the issue |
| 7 | ❌ | Null byte | In case an application requires the user-supplied filename to end with an expected file extension, so it might be possible to use a null byte to effectively terminate the file path before the required extension, like so `../../../etc/passwd%00.jpg` | 🟢 | Any notes about the issue |
| 8 | ✅  | 16-bits Unicode encoding | In case an application allows to use encoded version of `../` sequence, which results in `%u002e%u002e%u2215` | 🔴 | Any notes about the issue |
| 9 | 🚫  | UTF-8 Unicode encoding | In case an application allows to use encoded version of `../` sequence, which results in `%c0%2e%c0%2e%c0%af` | 🟢 | Any notes about the issue |
| 10 | ❌ | Public fuzzing lists | In case an application accepts one or more payloads from the public fuzzing lists | 🔴 | Any notes about the issue |

## Tools

### Burp Suite

Burp Intruder can be used to insert a list of directory traversal fuzz strings into a request either using a built-in `Fuzzing - path traversal` wordlist (available only in Professional edition) or manually add one.

### dotdotpwn

[dotdotpwn](https://github.com/wireghoul/dotdotpwn) is a flexible fuzzer designed to discover directory traversal vulnerabilities in various software and web platforms.

On Kali Linux the tool can be installed using APT:
```
sudo apt install dotdotpwn
```

Or it can be installed manually:

```
# clone from Git repository
git clone https://github.com/wireghoul/dotdotpwn.git

# install missing module
sudo apt install dotdotpwn
```

For example, to check the specific parameter of a URL for path traversal the following command can be used:

```
dotdotpwn -m http-url -u https://victim.com/download?filename=TRAVERSAL -f /etc/passwd -s -b -q -k "root:"
```

## Interesting files

1. [Interesting Linux files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal#interesting-linux-files)
2. [Interesting Windows files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal#interesting-windows-files)

## Fuzzing lists

1. https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI
2. https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_linux.txt
3. https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt
4. https://github.com/xmendez/wfuzz/blob/master/wordlist/vulns/dirTraversal-nix.txt
5. https://github.com/xmendez/wfuzz/blob/master/wordlist/vulns/dirTraversal-win.txt
6. https://github.com/xmendez/wfuzz/blob/master/wordlist/vulns/dirTraversal.txt
7. https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/Traversal.txt

## Recommendations

Ideally, application functionality should be designed in such a way that user-controllable data does not need to be passed to filesystem operations. This can normally be achieved by referencing known files via an index number rather than their name, and using application-generated filenames to save user-supplied file content.

If it is considered unavoidable to pass user-controllable data to a filesystem operation, multiple layers of defense can be employed to prevent path traversal attacks:

1. User-controllable data should be strictly validated before being passed to any filesystem operation. In particular, input containing dot-dot sequences should be blocked.

2. After validating user input, the application can use a suitable filesystem API to verify that the file to be accessed is actually located within the base directory used by the application. In Java, this can be achieved by instantiating a `java.io.File` object using the user-supplied filename and then calling the `getCanonicalPath` method on this object. If the string returned by this method does not begin with the name of the start directory, then the user has somehow bypassed the application's input filters, and the request should be rejected. In ASP.NET, the same check can be performed by passing the user-supplied filename to the `System.Io.Path.GetFullPath` method and checking the returned string in the same way as described for Java.

3. The directory used to store files that are accessed using user-controllable data can be located on a separate logical volume to other sensitive application and operating system files, so that these cannot be reached via path traversal attacks. In Unix-based systems, this can be achieved using a chrooted filesystem; on Windows, this can be achieved by mounting the base directory as a new logical drive and using the associated drive letter to access its contents.

4. Code that extracts archive files can be written to check that the paths of the files in the archive do not engage in path traversal.

5. The application can use chrooted jails and code access policies to restrict where the files can be obtained or saved to.

## Labs

- [PortSwigger - Path Traversal Labs](https://portswigger.net/web-security/all-labs#path-traversal)

## Videos

- [What is directory traversal](https://youtu.be/NQwUDLMOrHo?feature=shared)
- [Directory Traversal attacks are scary easy](https://youtu.be/99yJtmmtrJU?feature=shared)
- [What is Directory Traversal](https://youtu.be/17KYOIf5ZbU?feature=shared)
- [Advanced Directory Traversal Techniques](https://youtu.be/nvITajiF3rs?feature=shared)

## References

1. [PayloadsAllTheThings - Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
2. [HackTricks - File Inclusion/Path Traversal](https://book.hacktricks.xyz/pentesting-web/file-inclusion)
3. [OWASP - Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
4. [OWASP - Testing Directory Traversal File Include](https://github.com/OWASP/wstg/blob/master/document/4-Web_Application_Security_Testing/05-Authorization_Testing/01-Testing_Directory_Traversal_File_Include.md)

